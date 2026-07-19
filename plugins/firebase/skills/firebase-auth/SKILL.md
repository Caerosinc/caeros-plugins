---
name: firebase-auth
description: Firebase Authentication, sign-in providers, ID token verification, custom claims for roles, session cookies, and emulator testing. Use when the user adds auth or roles to a Firebase app.
---

# Firebase Authentication

## Providers

Enable in Console -> Authentication -> Sign-in method: Email/Password,
Google, Apple, phone, anonymous, plus SAML/OIDC for enterprise IdPs.
Anonymous auth is underrated: instant uid for guests, upgradeable later via
`linkWithCredential` without losing data.

```js
import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword,
         GoogleAuthProvider, signInWithPopup, onAuthStateChanged } from "firebase/auth"
const auth = getAuth()
await signInWithEmailAndPassword(auth, email, password)
await signInWithPopup(auth, new GoogleAuthProvider())
onAuthStateChanged(auth, (user) => { /* single source of truth for UI state */ })
```

Gotchas:
- `signInWithPopup` breaks under strict browser privacy settings and in
  embedded webviews; `signInWithRedirect` needs correctly configured
  authorized domains. Test both paths.
- The client `currentUser` is UI state, not proof; only verified tokens
  count on the server.

## Verify identity on your server

Client sends its ID token; server verifies with the Admin SDK:

```js
// client
const idToken = await auth.currentUser.getIdToken()
fetch("/api/thing", { headers: { Authorization: `Bearer ${idToken}` } })

// server (firebase-admin)
import { getAuth } from "firebase-admin/auth"
const decoded = await getAuth().verifyIdToken(idToken)   // throws if invalid/expired
// decoded.uid, decoded.email, decoded.<custom claims>
```

ID tokens live ~1 hour and auto-refresh on the client; never cache one
server-side. `verifyIdToken(token, true)` additionally checks revocation
(costs a lookup; use for sensitive routes).

## Custom claims (roles)

Claims ride inside the ID token and are readable by security rules, servers,
and clients, set them ONLY from a trusted environment:

```js
await getAuth().setCustomUserClaims(uid, { admin: true, tenant: "acme" })
```

- Max 1000 bytes of claims per user; claims are authorization data, not a
  profile store.
- Propagation: the change lands on the NEXT token refresh. Force it client
  side with `currentUser.getIdToken(true)` after the server confirms, or the
  user appears unprivileged for up to an hour.
- In Firestore rules: `request.auth.token.admin == true`,
  `request.auth.token.tenant == resource.data.tenant`.
- Typical flow: Cloud Function (or your backend) validates a business rule
  (invite code, Stripe webhook, allowlist) then sets the claim; never expose
  a callable that sets claims from unvalidated client input.

## Sessions for server-rendered apps

For cookie-based SSR instead of bearer tokens:

```js
// exchange a fresh ID token for a long-lived session cookie (max 14 days)
const cookie = await getAuth().createSessionCookie(idToken, { expiresIn: 5 * 864e5 })
res.cookie("__session", cookie, { httpOnly: true, secure: true, sameSite: "strict" })

// per request
const decoded = await getAuth().verifySessionCookie(cookie, true)  // true = check revoked
```

On logout: clear the cookie AND `getAuth().revokeRefreshTokens(uid)`.
On Firebase Hosting + Functions, `__session` is the only cookie name that
survives the CDN cache.

## User admin and emulator

```bash
firebase auth:export users.json --project my-app
firebase auth:import users.json --hash-algo ...   # bring users from elsewhere
```

Admin SDK: `getUser`, `getUserByEmail`, `listUsers` (paged), `updateUser`
(disable with `{ disabled: true }`), `deleteUser`.

The Auth emulator (`firebase emulators:start --only auth`) fakes providers
and issues real-shaped tokens: point clients with
`connectAuthEmulator(auth, "http://localhost:9099")` and the Admin SDK with
`FIREBASE_AUTH_EMULATOR_HOST=localhost:9099`. Emulator tokens are unsigned;
nothing from the emulator must ever reach production verification paths.

## Hardening checklist

1. Enable email enumeration protection and App Check.
2. Multi-tenant or B2B: scope every rule and query by a tenant claim.
3. Blocking functions (beforeUserCreated / beforeUserSignedIn) enforce
   domain allowlists at the source.
4. Treat `email_verified == false` as unverified identity in rules for
   email-gated features.
