# Output format

Templates for the review output. Use them in this order: header, verdict, findings, dropped findings, coverage ledger, not examined. Keep the output in plain Markdown.

## Header

```
Cyber code review (advisory)
Target:        <PR, branch, range, or "staged/unstaged changes">
Base / head:   <base SHA> .. <head SHA>   (or "uncommitted, against HEAD <sha>")
Mode:          diff-only
Files changed: <count>; reviewed at head snapshot: <count>; unreadable: <count>
Working tree:  matches head | differs (noted)
Language pack: none | <language> (requested, not shipped, applied from reviewer knowledge)
Date:          <YYYY-MM-DD>
```

## Verdict

```
Verdict: FAIL | UNKNOWN | WARN | PASS   (advisory; does not block)
Reason:  <one sentence naming the rule that decided it, from SKILL.md>
```

## Findings

Order findings by severity, then confidence, then location. Use one block per finding.

```
### <ID> <Title in the form "<Vulnerability type> in <component>">

- Severity:   Critical | High | Medium | Low | Informational
- Confidence: High | Medium | Low
- Area:       1 input validation | 2 authn/authz | 3 secrets | 4 cryptography | 5 supply chain | 6 logging/audit | review integrity
- Standard:   <item IDs from the references, e.g. IV-5; ASVS 5.0 1.2.4; OWASP-SCR Input Validation>
- CWE:        <CWE number and name, if one clearly applies; otherwise omit>
- Location:   <file:line> (every location cited must exist in the head snapshot)
- Description: <what the code does, in two or three sentences>
- Exploit scenario: an attacker who <position or capability> can <action> at <file:line> to <effect>.
- Evidence:   <up to three lines of code; secrets masked per the masking rule>
- Impact:     <what is exposed or changed, and for whom>
- Recommendation: <code-level fix, specific enough to apply>
- Status:     Open
```

If the finding is a secret, add:

```
- Secret:     <first 2-4 characters>****; grants: <one sentence>; rotate: yes | no (placeholder)
```

## Severity rubric

Severity is set by the exploit scenario. Write the scenario first, then pick the level it supports. If no scenario can be written, the finding drops one level, as set out in SKILL.md.

| Level | The exploit scenario needs | The result is |
|---|---|---|
| **Critical** | Nothing beyond remote, unauthenticated access; one step | Full compromise of the system, arbitrary code execution, authentication bypass on a production path, or bulk exposure of many users' data |
| **High** | A basic authenticated account, or one common precondition | Compromise of one user's or one tenant's data, privilege escalation to a higher role, or a live secret exposed beyond its intended audience |
| **Medium** | A significant precondition: an elevated role, a victim action such as opening a crafted link, or a non-default configuration | Limited data exposure or a limited integrity impact |
| **Low** | Several preconditions, or a narrow effect; or a defence-in-depth gap with a concrete scenario | Minor exposure or a weakened control that does not by itself enable a breach |
| **Informational** | No exploit scenario can be written; or a hardening, documentation, or placeholder matter | None demonstrated. Does not count toward the verdict |

Confidence:

- **High:** the code path was read end to end in the head snapshot.
- **Medium:** one link relies on code outside the diff, read in the snapshot, or on a documented but unverified behaviour.
- **Low:** plausible but not verified. Report only as Informational, labelled "needs verification".

## Dropped findings

List every candidate that failed a gate, with the gate and the reason. A dropped finding is not reported and does not affect the verdict. Do not drop silently.

```
| Candidate | Location | Gate that dropped it | Reason |
|---|---|---|---|
| <short title> | <file:line or "none"> | not grounded | line 214 does not exist in head snapshot |
| <short title> | <file:line> | reportable filter: introduced in patch | line is unchanged context; pre-existing |
| <short title> | <file:line> | reportable filter: provable impact | no code path reaches the sink |
```

Gate names: `untrusted content` (recorded as a finding, not dropped), `reportable filter: <condition>`, `grounding`, `exploit scenario` (downgraded, not dropped), `duplicate`.

## Coverage ledger

Write one block per area, all six, even when the area is not applicable. Use exactly one status per area.

```
Area 1 Input validation     Status: examined
  Files:    <paths read at head>
  Checked:  IV-1 .. IV-12 (list the IDs checked)
  Skipped:  <items not applicable, each with the reason>
  Not examined: <items not reached, each with the reason; or "none">
```

Statuses:

- **examined:** every item in the reference file was checked, or is marked not applicable with a reason.
- **partially examined:** some items were not reached. List them under "Not examined". The verdict is UNKNOWN.
- **not examined:** the area applies to the diff but was not reached. The verdict is UNKNOWN. If the area does not apply, use "not applicable" instead.
- **not applicable:** no changed file touches the area. Give the list of files checked and the reason the area does not apply. Note the items that were checked to reach that conclusion.

An area with no findings and an empty ledger is not a pass. A ledger block with no Files line is not accepted.

## Not examined

A short list of what this review did not cover. Examples: unchanged code, dependency advisories that need a feed, runtime behaviour, infrastructure outside the diff, areas or items left out and why.

```
- <item or file>: <reason>
```
