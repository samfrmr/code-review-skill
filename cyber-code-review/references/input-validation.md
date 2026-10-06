# Area 1: Input validation

Use this checklist for every changed entry point and every changed sink. Start from the boundary map built in SKILL.md step 3. Mark each item checked, not applicable (with reason), or not examined (with reason) in the coverage ledger.

Tags: `[S]` sourced from the cited standard. `[O]` originated by this skill; the anchor is the nearest standard requirement, or "no anchor".

## Items

**IV-1 [S] Server-side enforcement.** Input rules are enforced on the server, at the trusted layer. A change that validates only in client code, a template, or a UI handler, while the server accepts the raw value, is a finding.
- *Look for:* validation that exists only in front-end code, or a server endpoint that trusts a client-supplied flag or pre-validated field.
- *Source:* ASVS 5.0 2.2.2 (client-side checks are not a security control); OWASP-SCR, Input Validation checklist, "server-side validation".

**IV-2 [S] Positive validation.** Accepted input is described by an allowlist of values, a pattern, a range, or a structure. A denylist or "clean-up" pass that strips characters and then continues is not sufficient.
- *Look for:* regex or replace-based sanitizers that remove known-bad characters; code that corrects input rather than rejecting it.
- *Source:* ASVS 5.0 2.2.1 (positive validation against allowlists, patterns, ranges, or structure); OWASP-IV, "denylisting or cleaning input".

**IV-3 [S] Reject, do not repair.** Invalid requests are rejected with an error. Processing does not continue with partly validated data.
- *Look for:* a validator whose failure is logged or ignored and the function proceeds with the original value; default values substituted for invalid input.
- *Source:* OWASP-IV, Input Validation guidance on rejecting invalid requests rather than continuing with partly validated data.

**IV-4 [S] Business-rule validation and step order.** Input satisfies the business expectations of the operation. Multi-step flows run in the expected order for the same user, and a step cannot be skipped by calling a later endpoint directly.
- *Look for:* a changed workflow whose later step accepts a state transition the earlier step should have enforced; a new endpoint that repeats a transition without the preconditions.
- *Source:* ASVS 5.0 2.2.1; ASVS 5.0 2.3.1 (business flows in the expected sequential order without skipping steps).

**IV-5 [S] Query construction.** Data reaches SQL, NoSQL, ORM raw queries, and query languages (for example Cypher or HQL) only through parameters or a protected query API. String concatenation or formatting of user-controlled values into a query is a finding.
- *Look for:* f-strings, `+`, `format`, or template literals that build a query; raw query escape hatches in an ORM; dynamic table or column names taken from input without an allowlist.
- *Source:* ASVS 5.0 1.2.4; OWASP-SCR, Input Validation checklist (SQL injection prevention) and Common Vulnerability Patterns (query string concatenation).

**IV-6 [S] Operating-system commands.** No shell is invoked with user-controlled text. Calls use argument arrays or contextual encoding, not a single command string.
- *Look for:* `shell=True`, `system()`, `exec` with a concatenated string, backticks, subprocess calls that build a command line from input.
- *Source:* ASVS 5.0 1.2.5; OWASP-SCR, Common Vulnerability Patterns (direct command execution with user input).

**IV-7 [S] Dynamic code evaluation.** Input is never evaluated as code or as a dynamic expression language (for example `eval`, `exec`, a template expression, or an expression-language parser over input).
- *Look for:* `eval`, `Function(...)`, `new Function`, dynamic `import` of input, expression languages fed with request data.
- *Source:* ASVS 5.0 1.3.2.

**IV-8 [S] XML parser configuration.** XML parsers changed or added in the diff disable external entity resolution and other unsafe features.
- *Look for:* a new XML parser with default settings; a changed configuration that enables entity resolution or DTD processing.
- *Source:* ASVS 5.0 1.5.1.

**IV-9 [O] Boundary map.** Every changed entry point appears in the boundary map with its source, its trust level, and the validation applied. Any entry point whose field reaches a sink without validation in the boundary map is a candidate finding.
- *Look for:* a field that is read from a request, message, file, or webhook and passed on without a check; a trusted-looking source (an internal queue, a partner feed, a stored record) that is not validated.
- *Source:* OWASP-IV, "Trusting internal sources" pitfall (validate internal and stored data when it crosses a trust boundary). The map itself has no standard; the anchor is ASVS 5.0 2.2.1 and 2.2.2.

**IV-10 [S] Validation errors reveal nothing.** Validation failures return a message that names the rule, not the internal structure, query, path, or stack trace.
- *Look for:* error responses that echo the query, the file path, the schema, or the exception text.
- *Source:* OWASP-SCR, Input Validation checklist (error messages item); ASVS 5.0 16.5.1 (generic message for unexpected errors).

**IV-11 [S] Path and file-name handling.** Paths built from input are resolved and checked against an allowed base directory, and file names from input are not trusted as paths.
- *Look for:* `open`, `read`, `send_file`, or `join` with input; no normalisation or base-directory check.
- *Source:* none verified in this survey. The ASVS 5.0 file-handling chapter was not read. Mark the item as checked with the note "no anchor verified" until the chapter is read.

**IV-12 [O] Deserialisation of input.** Input is not deserialised into objects that can run code or create arbitrary types. Use a data-only format or an allowlist of types.
- *Look for:* native object serialisation (for example pickle, Java object streams, YAML loaders that build arbitrary types) fed with input; a changed parser switched to a permissive mode.
- *Source:* no ASVS anchor verified in this survey. OWASP Top 10:2025 A08 (software or data integrity failures) names the class; detail pages were not read. Anchor: none.
