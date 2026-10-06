# Area 5: Software supply chain

Use this checklist for every changed dependency manifest or lockfile, install or build script, CI workflow, release or packaging configuration, and vendored or copied third-party code. This is a review of the change, not a dependency audit. Do not claim a dependency is free of known vulnerabilities unless the diff or the repository carries that evidence. Otherwise record it as unverified in the ledger.

Tags: `[S]` sourced. `[O]` originated; anchor given or "no anchor".

## Dependencies in the change

**SC-1 [S] New dependencies are justified and integrated.** Each newly added third-party component is needed, comes from the project's approved source, and is recorded in the manifest and lockfile consistently.
- *Look for:* a new package added with no code that uses it; a dependency added from a URL, a git commit, or a personal fork; a manifest changed without the lockfile changing.
- *Source:* OWASP-SCR, Preparation checklist and diff-based review (new integrations); ASVS 5.0 15.1.1 (documented time frames for updating third-party components).

**SC-2 [S] Dependency versions are not known to be vulnerable.** The reviewer checks whether an added or upgraded version is one the repository's own documentation marks as affected, or whether the change moves to a version the project records as fixed. External advisory lookup is outside this skill; if the change's status cannot be verified from the repository, record it as unverified.
- *Look for:* an upgrade that moves back to an earlier version; a manifest pin that the project's own changelog or security notes mark as vulnerable.
- *Source:* ASVS 5.0 15.2.1 (no component that has breached the documented remediation time frame); OWASP-SCR, Configuration and Deployment checklist (dependency management item).

**SC-3 [S] Pinned versions.** Dependencies, container base images, and CI actions are pinned to an exact version or hash, not to a mutable tag, branch, or open-ended range. Where the project already pins by hash, a change that removes the hash is a finding.
- *Look for:* `latest`, `main`, `*`, a floating version range for a build tool; a Docker `FROM` that uses a mutable tag; a CI `uses:` line that points to a branch.
- *Source:* SCORECARD Pinned-Dependencies (dependencies set to a specific hash, not a mutable version or range; checks manifests, Dockerfiles, shell scripts, and workflows).

**SC-4 [O] Install-time and build-time code.** A new or changed hook that runs during install, build, or packaging (install scripts, post-install hooks, build plugins, code generators) is reviewed as code that runs with the build's privileges. A hook that downloads and runs code at build time without a pinned, verified source is a finding.
- *Look for:* a post-install script that fetches a file from the network and executes it; a build step that disables signature or integrity checks; a generator pulled from an unpinned URL.
- *Source:* anchored to ASVS 5.0 15.1.1. No ASVS requirement names install hooks; the check is originated.

**SC-5 [O] Dependency name and origin.** A new dependency name is checked against the origin the project uses. A name that is one character off a well-known package, or that does not match the project's usual registry or namespace, is a candidate for typosquatting or substitution and is reported as unverified until confirmed.
- *Look for:* a package name close to a popular library; a scope or namespace the project does not use elsewhere; a name that appears only in the change and nowhere in the project's docs.
- *Source:* anchored to ASVS 5.0 15.1.1. The check is originated.

**SC-6 [S] Checked-in binaries and generated artifacts.** Compiled binaries, executable archives, and build outputs are not added to source control, because their contents cannot be reviewed.
- *Look for:* `.exe`, `.so`, `.jar`, `.whl`, `.zip` added to the repository; a build output directory added to the change.
- *Source:* SCORECARD Binary-Artifacts (generated executables in the repository increase user risk and are not reviewable).

**SC-7 [S] Source control metadata is not served.** A change does not deploy or expose `.git`, `.svn`, or similar metadata to the application's public paths.
- *Look for:* a static-file route or a copy step that includes the repository directory; a web server configuration that exposes dot-directories.
- *Source:* ASVS 5.0 13.4.1.

## CI and build configuration

**SC-8 [S] Least-privilege CI tokens.** A workflow declares the smallest token permissions it needs. A change that grants write permissions to the default token, or removes a restrictive default, is a finding.
- *Look for:* `permissions: write-all`; a removed `permissions:` block at the top of a workflow; a job that writes to the repository when it only reads.
- *Source:* SCORECARD Token-Permissions (a compromised token with write access can push malicious code).

**SC-9 [S] Dangerous workflow patterns.** A changed workflow does not check out untrusted pull-request code in a privileged context, does not place untrusted input (titles, branch names, comments) into a shell script, and does not log the GitHub context or secrets.
- *Look for:* a `pull_request_target` trigger that checks out the pull-request head and runs it; `${{ github.event.pull_request.title }}` inside a `run:` block; `echo` of `github.event`.
- *Source:* SCORECARD Dangerous-Workflow (untrusted code checkout; untrusted input in scripts; logging of context and secrets).

**SC-10 [S] Build steps keep integrity checks.** A change to a build or release configuration keeps the project's signature, checksum, or provenance steps. A removed or skipped verification step is a finding.
- *Look for:* a removed signature verification; a `continue-on-error` added to a verification job; an environment flag that turns verification off.
- *Source:* SLSA-BUILD L2 (signed provenance from a hosted build platform); SCORECARD Signed-Releases (release assets carry signatures that attest to provenance).

**SC-11 [S] Provenance and release signing.** A change to a release workflow keeps the steps that produce provenance and sign the artifact. The consumer side validates the provenance before use.
- *Look for:* a release job that no longer uploads provenance; a signing key moved from the hosted platform to a developer machine; a verification step removed from the consumer.
- *Source:* SLSA-BUILD L1 (provenance describing how the artifact was built), L2 (signed provenance from a hosted build platform), and the L2 consumer requirement to validate the provenance's authenticity.

**SC-12 [S] Two-party review on protected branches.** A diff-level review cannot see branch protection. Record the review-policy status as unverified unless the repository configuration in the diff shows it. Where the change alters a branch-protection or code-owners file, check that the change itself is consistent with the project's stated policy.
- *Look for:* a change to a CODEOWNERS file that removes a required reviewer; a branch-protection setting codified in the repository that is weakened.
- *Source:* SCORECARD Code-Review (human review before merge); SCORECARD Branch-Protection (protected default and release branches); SLSA-SRC L4 (two trusted persons review protected-branch changes).

**SC-13 [S] Update tooling stays in place.** A change that removes an automated dependency-update tool's configuration reduces update coverage and is a finding of low severity unless it removes a security update path.
- *Look for:* a deleted Dependabot or Renovate configuration file in the change.
- *Source:* SCORECARD Dependency-Update-Tool (Dependabot or Renovate configured).
