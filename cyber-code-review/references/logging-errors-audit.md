# Area 6: Logging, errors, and audit

Use this checklist for every changed logging call, audit call, exception handler, error response, and security-check failure path.

Scope note on absence: a missing log line is not a finding by itself. Report a missing audit event only under LG-12, and only when the operation is sensitive and the patch introduced it or removed it.

Tags: `[S]` sourced. `[O]` originated; anchor given or "no anchor".

## Security events

**LG-1 [S] Authentication events are logged.** Successful and failed authentication attempts are recorded, along with the account identifier where that is permitted.
- *Look for:* a changed login or token-exchange path that returns without a log call on either outcome.
- *Source:* ASVS 5.0 16.3.1 (all authentication operations logged, successful and unsuccessful); OWASP-LOG, "Which events to log" section (authentication successes and failures).

**LG-2 [S] Authorization failures are logged.** Failed permission checks are recorded.
- *Look for:* a new denial path that returns 403 without a log entry; a changed permission check that silently returns false.
- *Source:* ASVS 5.0 16.3.2 (failed authorization attempts logged); OWASP-LOG, "Which events to log" section (authorization failures).

**LG-3 [S] Validation failures and bypass attempts are logged.** Input-validation failures, and attempts to bypass a control (such as business-logic or anti-automation checks), are logged.
- *Look for:* a validator whose rejection path does not log; a changed check that now drops a rejected request without a record.
- *Source:* ASVS 5.0 16.3.3 (logs security events and bypass attempts); OWASP-LOG, "Which events to log" section (input validation failures).

**LG-4 [S] Sensitive data is not logged.** Credentials, payment details, session tokens, connection strings, and encryption keys are not written to logs. Data that may be logged, such as session tokens, is hashed or masked.
- *Look for:* a log call that takes the whole request body, a headers dict containing an authorization header, an exception string containing a password; a log of a full card number or a bearer token.
- *Source:* ASVS 5.0 16.2.5 (logging based on protection level; credentials not logged; tokens hashed or masked); OWASP-LOG, "data to exclude".

**LG-5 [S] Log entries are encoded.** User-controlled values written to logs are encoded so that they cannot forge new lines or fields.
- *Look for:* `logger.info("user " + name)` where `name` can contain a newline; a structured log field that is built by string concatenation with raw input.
- *Source:* ASVS 5.0 16.4.1 (log injection prevented by encoding).

**LG-6 [S] Log entries carry the context needed for investigation.** Entries record when, where, who, and what, so that a timeline can be built.
- *Look for:* a new security event with no timestamp, actor, or resource identifier.
- *Source:* ASVS 5.0 16.2.1 (log entries include metadata for investigation).

**LG-7 [S] Standard event names.** Security events use the event names the project documents, following a consistent vocabulary. A new event that breaks the vocabulary is a low-severity finding only when it makes a documented alert unreliable.
- *Look for:* a new event named differently from the project's existing security events; a changed event name that breaks an existing alert.
- *Source:* OWASP-LOGVOCAB, standardised event names (for example the authentication-failure and authorization-failure forms). ASVS 5.0 16.1.1 (inventory of logged events).

## Errors and failure paths

**LG-8 [S] Generic errors to the caller.** An unexpected or security-sensitive error returns a generic message. Stack traces, queries, keys, and tokens are logged on the server, not returned.
- *Look for:* `return str(exception)` in a response; a changed error handler that adds the query text to the response; a traceback rendered in production.
- *Source:* ASVS 5.0 16.5.1 (generic message for unexpected errors); OWASP-ERR, error handling section (generic response to the caller; details logged server side).

**LG-9 [S] Failure is secure.** When a security check fails or raises an error, the operation is denied or stops. An error path that proceeds as success is a finding. This includes a validation check that is skipped on error.
- *Look for:* `try: verify(token) except Exception: pass` followed by the protected action; a function that returns True on error; a check whose result is ignored.
- *Source:* ASVS 5.0 16.5.3 (fail gracefully and securely; no fail-open condition such as processing despite a validation error); OWASP-AUTHZ, "exit safely when authorization checks fail".

**LG-10 [S] No silent failure around security operations.** An empty or swallowed exception handler around a security operation (authentication, authorization, cryptography, audit write, validation) is a finding. The handler must log the failure and fail closed.
- *Look for:* an empty `catch` or `except` around a token check, a signature verification, an audit write, or a permission lookup.
- *Source:* ASVS 5.0 16.5.3 (fail-open prevention). The rule that the handler must log is originated.

**LG-11 [S] Last-resort handler.** At L3 scope, a defined last-resort error handler catches all unhandled exceptions. Apply this item only when the project targets L3.
- *Look for:* a changed application bootstrap that removes the global error handler; a worker that crashes without a handler, leaving partial state.
- *Source:* ASVS 5.0 16.5.4 (L3).

## Log protection and audit trail

**LG-12 [O] Audit-trail completeness for sensitive operations.** For each changed sensitive operation, the code writes an audit event that records who acted, what they did, which resource was affected, when, and whether it succeeded. Sensitive operations include money movement, role or permission changes, data export or bulk read, deletion, configuration changes, secret or key access, and administrative actions. A changed sensitive operation with no audit event is a finding.
- *Look for:* a new admin action with no audit write; a bulk export endpoint that logs nothing; a deletion path that removes the record without a trace.
- *Source:* OWASP-SCR, Security Monitoring checklist (audit trails item; the requirement exists, and the per-operation check is originated); anchored to ASVS 5.0 16.3.2 and 16.3.3.
- *Note:* this item is the one place a lack of logs is reportable. Some existing review tools exclude missing audit logs outright. This skill does not, but only for sensitive operations changed by the patch.

**LG-13 [O] Audit event not removed or suppressed.** A change does not remove, rename, or disable an existing audit event, or move a sensitive operation off the audited path (for example, by adding a new route that skips the audit call).
- *Look for:* a deleted audit call in the diff; a new code path to the same operation that does not call the audit function.
- *Source:* anchored to ASVS 5.0 16.3.1 and 16.3.2. The regression check is originated.

**LG-14 [S] Logs are protected.** Logs cannot be modified or deleted by the application's ordinary users, and are sent to a system separate from the application for analysis.
- *Look for:* a change that gives the application write-delete access to its own log store; a log path inside a writable web directory.
- *Source:* ASVS 5.0 16.4.2 (logs protected from unauthorized access and modification); ASVS 5.0 16.4.3 (logs sent to a logically separate system); OWASP-LOG, "protection".

**LG-15 [S] Logging inventory.** A new logging layer or logger is documented in the project's logging inventory, including what it logs, where it is stored, and how long it is kept.
- *Look for:* a new logging sink (a new file, a new service) with no matching entry in the project's logging documentation.
- *Source:* ASVS 5.0 16.1.1 (inventory of logging per layer).
