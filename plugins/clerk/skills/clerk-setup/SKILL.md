---
name: clerk-setup
description: Create Clerk applications, manage API keys, and move safely from development to production instances. Use when keys are missing, auth breaks after deploy, or environments get mixed up.
---

# Clerk setup

## Application and keys

1. Create an application in the Clerk Dashboard (https://dashboard.clerk.com);
   pick the sign-in options (email, OAuth providers, passkeys).
2. Every application has two instances: **Development** and **Production**,
   each with its own key pair:
   - `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`: `pk_test_...` (dev) / `pk_live_...` (prod)
   - `CLERK_SECRET_KEY`: `sk_test_...` (dev) / `sk_live_...` (prod)
3. Copy from Dashboard > API keys into `.env.local`. The publishable key is
   safe to expose; the secret key is server-only.

Key hygiene:

- `.env.local` is gitignored; production keys live only in the host's secret
  store (Vercel/eqv env vars), never in the repo.
- If an `sk_` key ever lands in a commit or a chat, rotate it in the
  Dashboard immediately; old key keeps working during the grace window you
  choose at rotation time.
- Separate Clerk applications for genuinely separate products; instances
  (dev/prod) for environments of the same product.

## Development instance behavior

- Works on localhost with no DNS setup; sessions use a dev browser cookie.
- User limits and a "development mode" banner apply; data is isolated from
  production.
- OAuth providers ship with shared dev credentials so social login works
  instantly; production requires your own OAuth app credentials per provider.

## Going to production

1. Dashboard: activate the Production instance and set your application
   domain.
2. Add the DNS records Clerk shows (frontend API subdomain, typically
   `clerk.yourdomain.com`, plus account portal and email records) and wait for
   verification.
3. Configure real OAuth credentials for each social provider (redirect URIs
   point at your Clerk frontend API domain).
4. Swap env vars on the host to the `pk_live_` / `sk_live_` pair.
5. Re-create webhooks on the production instance; signing secrets differ per
   instance (`CLERK_WEBHOOK_SIGNING_SECRET`).

Production checklist before launch:

- Session token verification uses production JWKS (automatic with live keys).
- Allowed redirect origins configured for native/mobile clients.
- Bot protection and attack protection settings reviewed under
  User & Authentication.
- Test a full sign-up, sign-in, sign-out, and webhook delivery on a staging
  deploy with production keys before DNS cutover.

## Common failures

- Components render but auth loops on deploy: publishable key from one
  instance with secret key from another; keys must be from the same instance.
- `auth()` always signed out in production: middleware/proxy file missing or
  matcher excludes the route.
- OAuth works locally, fails deployed: provider still using shared dev
  credentials, or redirect URI not updated to the production frontend API.
- Webhooks 400: wrong instance's signing secret, or a body parser consumed
  the raw payload before verification.

## MCP note

This plugin connects Clerk's official hosted MCP server
(https://mcp.clerk.com/mcp, Streamable HTTP, no credentials required). It
serves current SDK snippets and implementation guides; it does not touch your
Clerk data, so it is safe to use before any keys exist.
