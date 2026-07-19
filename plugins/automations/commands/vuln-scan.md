---
name: vuln-scan
description: Review the working tree for exploitable security issues, validate each finding, and report confirmed ones with severity and fixes.
aliases: security-scan
---

You are running the vuln-scan automation. Goal: find real, exploitable security issues in the working tree, validate each one against the actual code, and report only confirmed findings. This is a report-only pass: make no code changes and open no PRs unless the user explicitly asks for fixes.

## Scope

- Scan the repository currently open. If the user passed arguments (a subdirectory, a language, a vulnerability class, or "fix them"), narrow or extend accordingly.
- Focus on code this project ships and executes. Deprioritize vendored dependencies and generated code, but do flag known-vulnerable dependency versions if manifests reveal them.

## What to hunt for

Work through these classes, using targeted searches (grep/glob) followed by reading the surrounding code:

1. **Injection**: SQL built by string concatenation or formatting, shell commands assembled from user input (`exec`, `system`, `subprocess` with `shell=True`, backticks), template injection, path traversal in file operations fed by request data.
2. **Broken authz/authn**: endpoints or handlers missing permission checks that sibling endpoints have, object-level access without ownership verification (IDOR), trust decisions based on client-supplied fields, JWT verification disabled or algorithm confusion.
3. **Secrets**: hardcoded API keys, passwords, private keys, and tokens in source, config, or committed env files. Check obvious paths (`.env*`, `config/`, CI files) and grep for key-like patterns. Never print a discovered secret's value; identify it by file, line, and type only.
4. **Unsafe deserialization**: `pickle`/`yaml.load` without SafeLoader, Java native deserialization, `eval`/`Function` on external input, prototype-pollution-prone deep merges.
5. **Other high-value classes** as the codebase suggests: SSRF in URL fetchers, XSS in HTML rendering paths, weak crypto (ECB, MD5/SHA1 for passwords, static IVs), permissive CORS, missing CSRF protection on state-changing routes.

## Validation (mandatory)

For every candidate finding, before reporting it:

- Read the full data path: where does the attacker-controlled input enter, and does it actually reach the sink unsanitized? Trace it; do not assume.
- Check for existing mitigations: parameterization, framework auto-escaping, middleware guards, input validation upstream.
- Discard anything you cannot trace to a plausible attacker-controlled trigger. Unreachable code, test fixtures, and developer-only tooling are at most low-severity notes.
- If exploitability is genuinely uncertain after reading the code, either drop it or report it explicitly as "unconfirmed, needs review", never as a confirmed finding.

## Safety rails

- Do not write exploit payloads against live systems, do not exfiltrate or print secret values, do not modify code in this pass.
- If you find a live credential, say so immediately at the top of the report and recommend rotation.

## Output format

If nothing confirmed: say so plainly, list what you checked, and note any unconfirmed items worth a human look.

Otherwise, for each confirmed finding:

1. **Title and severity** (Critical / High / Medium / Low, judged by exploitability and impact).
2. **Location**: file path and line range.
3. **Data path**: entry point to sink, in 2-3 sentences.
4. **Impact**: what an attacker gains.
5. **Fix suggestion**: the concrete change (parameterize, add authz check, safe loader, rotate secret), with a short code sketch where helpful.

Order findings by severity. End with a one-paragraph summary of scanned areas and coverage gaps.
