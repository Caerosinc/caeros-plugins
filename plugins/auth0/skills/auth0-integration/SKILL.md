---
name: auth0-integration
description: Integrate Auth0 into SPAs, regular web apps, and APIs with the current SDKs (auth0-react, nextjs-auth0 v4, express-openid-connect, express-oauth2-jwt-bearer). Use when the user adds Auth0 login or protects an API.
---

# Auth0 integration

Pick the application type first; it decides the SDK and the token flow:

| App type | SDK | Flow |
|---|---|---|
| SPA (React/vanilla) | `@auth0/auth0-react` / `@auth0/auth0-spa-js` | Auth code + PKCE |
| Regular web app (Next.js) | `@auth0/nextjs-auth0` (v4) | Auth code, server session |
| Regular web app (Express) | `express-openid-connect` | Auth code, server session |
| API | `express-oauth2-jwt-bearer` | Validates bearer JWTs |

## SPA: @auth0/auth0-react

```tsx
<Auth0Provider
  domain="YOUR_DOMAIN.auth0.com"
  clientId="CLIENT_ID"
  authorizationParams={{ redirect_uri: window.location.origin,
                         audience: "https://api.example.com" }}>
  <App />
</Auth0Provider>
```

```tsx
const { loginWithRedirect, logout, user, isAuthenticated, getAccessTokenSilently } = useAuth0();
const token = await getAccessTokenSilently();   // for API calls, sent as Bearer
```

SPA rules: no client secret exists; never store tokens in localStorage when
you can use the SDK's in-memory + refresh token rotation defaults; always set
an `audience` so you get a JWT access token for your API, and keep ID tokens
for identity only (never send an ID token to your API).

## Next.js: @auth0/nextjs-auth0 v4

```bash
npm i @auth0/nextjs-auth0
# .env.local: AUTH0_DOMAIN, AUTH0_CLIENT_ID, AUTH0_CLIENT_SECRET,
#             AUTH0_SECRET (openssl rand -hex 32), APP_BASE_URL
```

```ts
// lib/auth0.ts
import { Auth0Client } from "@auth0/nextjs-auth0/server";
export const auth0 = new Auth0Client();

// middleware.ts (proxy.ts on Next.js 16+): REQUIRED in v4
import { auth0 } from "./lib/auth0";
export async function middleware(request: NextRequest) {
  return await auth0.middleware(request);
}
export const config = { matcher: ["/((?!_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)"] };
```

v4 essentials: auth routes auto-mount at `/auth/*` (`/auth/login`,
`/auth/logout`, `/auth/callback`; the v3 `/api/auth/*` paths are gone). Link
with plain `<a href="/auth/login">` (and `?screen_hint=signup`). Protect
server-side with `const session = await auth0.getSession(); if (!session)
redirect("/auth/login")`; the old `withPageAuthRequired` /
`withApiAuthRequired` helpers were removed. Rolling sessions come from the
middleware, so keep its matcher broad.

## Express web app: express-openid-connect

```js
app.use(auth({ authRequired: false, auth0Logout: true,
  baseURL, clientID, issuerBaseURL: "https://YOUR_DOMAIN.auth0.com",
  secret: LONG_RANDOM }));
app.get("/profile", requiresAuth(), (req, res) => res.json(req.oidc.user));
```

## API: express-oauth2-jwt-bearer

```js
const { auth, requiredScopes } = require("express-oauth2-jwt-bearer");
app.use(auth({ audience: "https://api.example.com",
               issuerBaseURL: "https://YOUR_DOMAIN.auth0.com/" }));
app.post("/orders", requiredScopes("create:orders"), handler);
```

The middleware checks signature (JWKS), `iss`, `aud`, `exp`. You still own
object-level authorization: scope proves what the client may do, your code
must check which resources this `sub` may touch.

## Cross-cutting security

- Register exact callback and logout URLs per environment; wildcards only in
  dev tenants.
- Request the minimum scopes; put roles/permissions in access token claims
  via Actions (see `auth0-actions`) and enforce server-side.
- Custom domain in production so cookies and consent screens match your brand
  and third-party cookie issues disappear.
