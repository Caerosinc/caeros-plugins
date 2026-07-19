---
name: aws-cli-auth
description: Set up aws CLI v2, IAM Identity Center SSO profiles, and credential hygiene. Use when the user needs to install, configure, or debug AWS authentication on a machine or in CI.
---

# aws CLI v2 Setup and Auth

## Install and verify

```bash
brew install awscli            # macOS; Linux: official installer, not pip
aws --version                  # must say aws-cli/2.x
aws sts get-caller-identity    # who am I, the first debug command always
```

## SSO profiles (the right way for humans)

Use IAM Identity Center (formerly AWS SSO), not long-lived access keys:

```bash
aws configure sso
# SSO session name: mycompany
# SSO start URL:    https://mycompany.awsapps.com/start
# SSO region:       us-east-1
# then pick account + role, name the profile (e.g. dev, prod)

aws sso login --profile dev            # opens browser, short-lived creds
aws s3 ls --profile dev
export AWS_PROFILE=dev                 # default profile for the shell
```

This writes an `[sso-session]` block plus `[profile dev]` to
`~/.aws/config`. Tokens are cached in `~/.aws/sso/cache` and expire; a
sudden `Error loading SSO Token` or `ExpiredToken` means re-run
`aws sso login`. One `sso-session` can back many account/role profiles.

SDKs resolve the same profiles automatically (`AWS_PROFILE` env or
`--profile`); nothing extra to configure in code.

## Credential resolution order (what wins)

1. Explicit client config in code
2. Env vars: `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` /
   `AWS_SESSION_TOKEN`
3. `AWS_PROFILE` -> `~/.aws/credentials` + `~/.aws/config`
4. Container / IMDS role credentials (ECS task role, EC2 instance profile)

The #1 trap mirrors every cloud: a stale exported `AWS_ACCESS_KEY_ID`
silently overrides your SSO profile. `aws sts get-caller-identity` tells
you which identity actually answered; `env | grep AWS_` finds the culprit.

Region resolution: `AWS_REGION` env beats `region` in the profile. Set a
region per profile so scripts do not fall back to the wrong one.

## Machines and CI (no static keys)

- **GitHub Actions / external CI**: OIDC federation into an IAM role
  (`aws-actions/configure-aws-credentials` with `role-to-assume`), zero
  stored secrets.
- **EC2 / ECS / Lambda**: attached roles, credentials arrive ambiently.
- **Cross-account**: `role_arn` + `source_profile` in `~/.aws/config`, the
  CLI chains the AssumeRole for you:

```ini
[profile prod-admin]
role_arn = arn:aws:iam::222222222222:role/Admin
source_profile = dev
region = us-east-1
```

## Hygiene

- No IAM user access keys for humans, period. If legacy keys exist:
  `aws iam list-access-keys`, rotate, then delete.
- Never commit `~/.aws/credentials` content; keys in a repo mean immediate
  rotation, not deletion of the commit.
- Scope roles per environment; require MFA on the Identity Center login.
- `aws configure list` shows source of each setting when auth behaves
  strangely; add `--debug` to any call to watch the credential chain.

## In Caeros

Caeros shells inherit your environment: export `AWS_PROFILE` (and run
`aws sso login`) in the shell where agents run, and generated code using
default SDK clients will pick the same credentials up.
