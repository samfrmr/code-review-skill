# Area 3: Secret hygiene

Use this checklist for every changed file that can hold a secret: source, tests, fixtures, configuration, infrastructure code, CI workflows, environment templates, and logs or error paths. Apply the fail-secure rule and the secret-masking rule from SKILL.md to every finding in this area.

Severity note for this area: a live credential that the code uses at runtime is at least High. Severity is Critical when the credential grants administrative access to production or to other tenants' data. A placeholder that the application never accepts is Informational.

## Items

**SE-1 [S] No hard-coded secrets.** Secret values do not appear in source, tests, fixtures, configuration, or CI files. Test fixtures are in scope.
- *Look for:* string literals that match credential shapes (API key prefixes, private key headers, long high-entropy strings) assigned to names such as `password`, `secret`, `token`, `api_key`, `key`; connection strings with embedded passwords; `.env` files committed with values.
- *Source:* OWASP-SCR, Configuration and Deployment checklist (secrets management item); ASVS 5.0 13.3.1 (secrets held in a managed secrets solution).

**SE-2 [S] No fallback secrets.** The application does not run with a literal fallback when a secret is missing. A missing secret causes a refusal to start, or a refusal of the request, not a boot with a default value.
- *Look for:* `os.environ.get("KEY", "literal")`, `getenv("KEY") or "literal"`, `config.get("secret", "dev-secret")`, a default in a function parameter that becomes the signing key.
- *Source:* ASVS 5.0 16.5.3 (no fail-open behaviour); the decisive question is set out in SKILL.md under the fail-secure rule.

**SE-3 [S] No default or seeded credentials; debug mode off.** Default credentials are absent from the application. Debug features and verbose error pages are off in production configuration.
- *Look for:* `DEBUG = True` in a production config path; a seed script that creates an admin with a fixed password and runs at startup; debug endpoints enabled by default.
- *Source:* ASVS 5.0 13.2.3 (service credentials are not default values); ASVS 5.0 13.4.2 (debug modes disabled in production).

**SE-4 [S] Least privilege for secret access.** A changed component reads only the secrets it needs. A new component that reads a broad secret set, or a secret shared between unrelated services, is a finding.
- *Look for:* a service that loads every secret from the vault; one key reused for signing in two services that do not need the same trust.
- *Source:* ASVS 5.0 13.3.2 (least privilege for secret assets); OWASP-SECRETS, section 2.3 (access control and least privilege).

**SE-5 [S] Rotation and expiry.** A new credential or key has a documented rotation path, and the code can accept a new value without a redeploy where the project already does that. Credentials that the project treats as long-lived are documented as such.
- *Look for:* a token with no expiry field; a key with no version identifier; a rotation function that is defined but not wired in.
- *Source:* ASVS 5.0 13.1.4 (L3, documented schedule for critical secrets); ASVS 5.0 13.3.4 (L3, secrets expire and rotate); OWASP-SECRETS, section 2.7.2 (rotation; user credentials are rotated on suspected compromise rather than on a schedule).

**SE-6 [S] Secrets not in URLs, query strings, or browser storage.** Secrets are sent only in headers or request bodies, and are not stored in client-side browser storage except for session tokens.
- *Look for:* a token placed in a URL or query string; a secret written to `localStorage`, `sessionStorage`, or IndexedDB; a secret embedded in a client bundle.
- *Source:* ASVS 5.0 14.2.1 (no sensitive data in URLs or query strings); ASVS 5.0 14.3.3 (no sensitive data in browser storage other than session tokens).

**SE-7 [S] Secrets not logged or echoed.** Secrets do not appear in log lines, error messages, exception text, debug output, process arguments, or environment dumps.
- *Look for:* `logger.info(f"... {password}")`; an exception that includes the request body; a command built with a secret on the command line; a debug print of `os.environ` or `process.env`.
- *Source:* OWASP-LOG, "Data to exclude" section; ASVS 5.0 16.2.5 (no logging of credentials; masked or hashed where permitted). The process-argument and environment-dump cases are originated under this item.

**SE-8 [S] CI workflows do not leak or expose secrets.** A CI workflow does not print secrets to the log, does not pass them to untrusted pull-request code, and does not log the full GitHub context.
- *Look for:* `echo ${{ secrets.X }}`; a step that prints `github.event` or environment variables; secrets passed to a job that checks out and runs untrusted code from a fork.
- *Source:* SCORECARD Dangerous-Workflow (patterns: logging of context and secrets; untrusted code checkout; untrusted input in scripts); OWASP-SECRETS, section 3.4 (logging and accounting for CI/CD).

**SE-9 [O] Secret scope in new code.** A new secret is scoped to the smallest audience: one environment, one service, one purpose. A secret used in both production and a development path is a finding.
- *Look for:* one credential referenced by both a production and a dev config; a test-only key read by application code.
- *Source:* anchored to ASVS 5.0 13.3.2. The environment-separation check is originated.

**SE-10 [S] Placeholder handling.** A value that is obviously a placeholder (for example `changeme`, `example`, `xxxx`, or a documented sample) and that the application refuses at runtime is Informational. Confirm the refusal in the code before you downgrade. Do not assume a placeholder is safe because of its name.
- *Look for:* a placeholder accepted by a validator, a login path, or a signing function; a placeholder whose surrounding code never checks that it was replaced.
- *Source:* the placeholder rule follows from the fail-secure rule in SKILL.md, with ASVS 5.0 13.2.3 for default credentials.
