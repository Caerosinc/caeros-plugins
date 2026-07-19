---
name: op-cli
description: Use the 1Password op CLI to inject secrets at runtime with op run, template files with op inject, read values with op read, and automate with service accounts. Use when the user handles secrets in dev workflows.
---

# op CLI patterns

Principle: secrets stay in 1Password; processes receive them at runtime.
Never paste a secret into a file, a shell command, or a chat transcript.

## op run: inject into a process environment

```bash
# .env contains references, not values (see op-secret-references)
op run --env-file=.env -- npm run dev
op run --env-file=.env -- python manage.py runserver
```

`op run` resolves every `op://` reference in the env file, injects the real
values into the child process environment only, and masks them if they appear
in stdout/stderr. Nothing is written to disk. Flags worth knowing:

- `--no-masking` when output masking breaks structured logs (use sparingly).
- Multiple `--env-file` flags merge; later files win.

## op read: one value, straight to the consumer

```bash
op read "op://Dev Vault/Postgres/password"
op read "op://Dev Vault/Deploy Key/private key" > /tmp/key && chmod 600 /tmp/key
DATABASE_URL=$(op read "op://Dev Vault/App/db-url") ./migrate
```

Prefer piping directly into the consumer over exporting into your interactive
shell (exports leak into shell history, child processes, and crash dumps).

## op inject: template config files

```bash
# config.tpl contains {{ op://Dev Vault/App/api-key }} or op:// lines
op inject -i config.tpl -o config.yaml
op inject -i .env.tpl -o .env.local   # generate a local file when a tool insists on one
```

Treat inject output as sensitive: gitignore it, prefer `op run` when the tool
can read from the environment instead.

## Item CRUD

```bash
op vault list
op item list --vault "Dev Vault"
op item get "Postgres" --vault "Dev Vault" --format json
op item get "Postgres" --fields password --reveal
op item create --category=login --vault "Dev Vault" \
  --title "Staging API" username=svc password="$(openssl rand -base64 32)"
op item edit "Staging API" --vault "Dev Vault" password="$(openssl rand -base64 32)"
```

`--format json` makes every command scriptable. Rotation pattern: generate the
new value locally, `op item edit`, then redeploy with `op run` so consumers
pick it up.

## Service accounts (automation and CI)

```bash
export OP_SERVICE_ACCOUNT_TOKEN=<token>   # from your CI secret store
op vault list                             # now authenticated, no app, no user
op run --env-file=.env.ci -- ./deploy.sh
```

Service accounts are the automation path: no desktop app, no biometric
prompts, token-scoped to specific vaults with read or read/write. Rules:

- One service account per pipeline or system, scoped to the minimum vaults.
- The token itself is the only secret your CI stores; everything else resolves
  at runtime.
- Rotate service account tokens like any credential; they are revocable in
  the 1Password admin console.

## Defensive habits

- Audit for plaintext secrets: grep repos for `API_KEY=`, `sk_live`, tokens,
  then move each into 1Password and replace with references.
- Use separate vaults per environment (dev/staging/prod) so a leaked dev
  token cannot read prod.
- `op signin` sessions expire; wrap long scripts with service accounts
  instead of personal sessions.
