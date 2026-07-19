---
name: auth0-actions
description: Write Auth0 Actions (post-login and other triggers), add namespaced custom claims, use secrets and dependencies, and migrate legacy Rules. Use when the user customizes Auth0 pipelines or tokens.
---

# Auth0 Actions

Actions are Node.js functions that run inside Auth0 flows. Rules and Hooks
are deprecated (end-of-life announced); all new pipeline logic goes into
Actions, and existing Rules should be migrated.

## Triggers

- `post-login`: after every successful auth (most common; claims, MFA,
  access denial, enrichment).
- `pre-user-registration` / `post-user-registration`
- `post-change-password`, `send-phone-message`, `password-reset-post-challenge`
- `credentials-exchange` (M2M token issuance)

## post-login anatomy

```js
exports.onExecutePostLogin = async (event, api) => {
  // event: user, tenant, client, request, transaction, secrets
  // api:   the only way to affect the outcome
};
```

## Custom claims (the right way)

Claims must be collision-safe. Use a namespace URI you control:

```js
exports.onExecutePostLogin = async (event, api) => {
  const ns = "https://example.com";
  const roles = event.authorization?.roles ?? [];
  api.idToken.setCustomClaim(`${ns}/roles`, roles);
  api.accessToken.setCustomClaim(`${ns}/roles`, roles);
  api.accessToken.setCustomClaim(`${ns}/org_id`, event.user.app_metadata?.org_id);
};
```

Rules: never use standard OIDC claim names or `auth0`/`webtask` namespaces;
keep claims small (tokens travel on every request); put authorization data in
the access token, identity data in the ID token; consumers must treat missing
claims as deny.

## Common patterns

```js
// Deny access by policy
if (!event.user.email_verified) {
  api.access.deny("Please verify your email before signing in.");
}

// Require MFA for risky context
if (event.authorization && highRisk(event)) {
  api.authentication.challengeWithAny([{ type: "otp" }, { type: "webauthn-roaming" }]);
}

// Redirect mid-login (custom consent/profiling), then resume
api.redirect.sendUserTo("https://app.example.com/consent",
  { query: { session_token: api.redirect.encodeToken({ secret: event.secrets.SESSION_SECRET, payload: { sub: event.user.user_id } }) } });
// pair with exports.onContinuePostLogin
```

## Secrets and dependencies

- Secrets: defined per Action in the editor, read via `event.secrets.NAME`;
  never hardcode API keys in Action source (source is visible to tenant
  admins and in deploy artifacts).
- Dependencies: declare npm packages with pinned versions in the Action's
  settings; keep the list minimal, every dependency runs inside your login
  path.
- Calling external APIs: set timeouts and fail open or closed deliberately;
  an unhandled throw fails the login.

## Rules migration checklist

1. Inventory Rules; note ordering dependencies (Rules ran sequentially,
   Actions are ordered per trigger in the dashboard flow editor).
2. Port `user`/`context` reads to `event.*`; side effects to `api.*`
   (`context.idToken[...] =` becomes `api.idToken.setCustomClaim`).
3. `configuration` values become Action secrets.
4. Deploy the Action, bind it in the Login flow, test in a dev tenant, then
   disable the Rule in the same change window (both running = duplicate
   claims or double MFA prompts).

## Deploy and observe

- CLI: `auth0 actions create|deploy|list` scriptable from CI; the official
  Auth0 MCP server this plugin ships can also create and deploy Actions on
  your tenant.
- Watch Monitoring > Logs for action execution errors after rollout;
  a broken post-login Action blocks every login, so ship behind a dev tenant
  test first.
