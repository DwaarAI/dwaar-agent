---
name: dwaar-security-review
description: Review and fix the security findings Dwaar has for the code repository you are working in, then prove the fix with a rescan. Use when the user, inside a git repository, asks about its vulnerabilities, CVEs, leaked secrets, security findings or what to fix first, or asks you to fix them.
license: Apache-2.0
---

# Security review of this repository with Dwaar

Rules for the whole review:

- Report only what Dwaar's tools return. Never invent a finding, CVE, severity, EPSS or KEV value. If a tool fails,
  say so and quote its error, including the `· Ref <id>` at its end.
- Cite every finding by its id and link it as `<console>/findings?finding=<id>`.
- Never call a finding fixed until a rescan of the pushed commit no longer lists it as open.
- Ask before you commit, push, or rewrite git history. Never push without the user's consent.
- Waivers, false positives and severity overrides are not tools: send the user to the finding's console page for those.

`<console>` is the console origin the dwaar server names in its instructions ("The console is …"). Without them, it is
`https://console.stg.dwaar.ai` on the shared server; on a dedicated tenant, ask the user for their console address.
Never derive it from the MCP server's URL: the two can be on different domains. If the Dwaar tools are missing or
answer 401, use the `dwaar-connect` skill first.

## 1. Find this repository in Dwaar

1. Read the remote and the current version:
   `git remote get-url origin`, `git rev-parse --abbrev-ref HEAD`, `git rev-parse HEAD`.
   Reduce the remote to `<host>/<owner>/<name>` (drop `git@`, `https://`, `.git`).
2. Call `search` with `q=<name>`. Results of kind `resource` have ids of the form `resource:<id>`: the resource id is
   the part after `resource:`. Or call `resources` with `type=code` and `q=<name>`.
3. Pick the resource whose label or technical name matches `<owner>/<name>` (case-insensitive). If several still match,
   ask the user which one.
4. No match: tell the user this repository is not in Dwaar. Call `connections`: with no source-control connection, use
   `dwaar-connect` to connect one; with one, the repository is outside its discovery scope or discovery has not run
   since it was created, so link the connection at `<console>/integrations/<provider id>`. Stop here.

## 2. Read its state

1. `resource` with `id=<resource id>`: tracked refs (branches, tags), coverage and the newest scan. Note which commit
   was scanned. If it is not the local `HEAD`, say that the findings describe that commit.
2. Never scanned, or a scan still running: `scans` with `resource=<id>`, report what is queued or running, and offer to
   follow it.
3. `findingCounts` with `resource=<id>`: sum the open cells by severity and by type.
4. `findings` with `resource=<id>`, `severity=critical,high`, `first=25` (default order is worst first). Read medium and
   low only when the user asks, or when there are no critical or high findings. Page with `after=<endCursor>`.

Link the resource as `<console>/resources/code/<id>`.

## 3. Explain each finding

Work worst first, at most ten at a time. Put findings on CISA KEV first, then critical and high with a high EPSS.

1. `finding` with `id=<id>`: type, severity, status, the component (PURL and version), vulnerability ids (CVE, GHSA),
   EPSS, KEV, the fix, and the subject (`path`, `line`, and `paths` for every lockfile a package was found in).
2. Find it in this repository:
   - `vulnerability`, `malicious-package`, `license`: the manifest and lockfile that bring the component in
     (`package.json` and its lockfile, `go.mod`/`go.sum`, `requirements*.txt`, `poetry.lock`, `uv.lock`, `pom.xml`,
     `build.gradle`, `Cargo.lock`, `Gemfile.lock`, `composer.lock`). Tell direct from transitive with the package
     manager (`npm ls <pkg>`, `pnpm why <pkg>`, `yarn why <pkg>`, `go mod why -m <module>`, `pip show`, `cargo tree -i <crate>`).
   - `secret`: the file and line. Treat the credential as compromised.
   - `sast`, `misconfiguration`, `ci-workflow`, `code-quality`: the file and line in the subject.
   - `base-image`: the `FROM` line of the Dockerfile.
3. Say in one or two sentences what the risk is here, where it lives, and what fixes it.

## 4. Fix

Propose the fixes first; make them once the user agrees. Keep each change minimal.

- **Dependency**: move to the lowest version that has the fix and stays compatible. Direct: change the manifest.
  Transitive: upgrade the parent that pulls it in, or use the ecosystem's override (`overrides`, `resolutions`,
  `pnpm.overrides`, a `replace` or `require` in `go.mod`, a constraint file). Regenerate the lockfile with the package
  manager (for example `npm install`, `go get <module>@<version> && go mod tidy`, `uv lock --upgrade-package <pkg>`);
  never edit a lockfile by hand.
- **Secret**: remove it from the code and read it from the environment or a secret manager. Tell the user to revoke and
  rotate it at its issuer now: removing it from `HEAD` leaves it in git history, and rewriting history is their call.
- **Code (SAST)**: the smallest change that removes the flaw, with a sentence on why it is safe.
- **CI workflow**: pin third-party actions to a commit SHA, set least-privilege `permissions`, and keep untrusted input
  (issue titles, branch names) out of `run:` steps.
- **Base image**: a patched tag, pinned by digest where the project already pins.
- **No fix available**: say so. The options are another package, a mitigation in code, or accepting the risk, which a
  person records in the console on the finding's page.

Run the project's build and tests after the changes. Summarise what changed per finding id. The findings stay open in
Dwaar until step 5 proves otherwise.

## 5. Prove it with a rescan

1. Ask the user to commit and push. A rescan reads the resource's tracked refs, not the working tree: if the pushed
   branch is not one of them (step 2), say so. The options are merging into a tracked branch, or tracking the branch on
   `<console>/resources/code/<id>`.
2. `rescanResource` with `id=<resource id>`. It needs **Read and write** access. If the tool is missing, have the user
   sign in again and choose **Read and write** (see `dwaar-connect`). If the push is not picked up, `syncConnection`
   with the connection id runs discovery first.
3. Follow the queued scans with `scan` (`id=<scan id>`) or `scans` (`resource=<id>`), waiting about 15 s, 30 s, 60 s,
   then 2 min between checks. A failed scan: show its error, and offer `retryScan` once.
4. When the scan is `completed`, call `findings` with `resource=<id>` again, and `finding` for each one you fixed.
5. Report three lists: fixed (no longer open), still open (with why, if the scan says), and new. Link each finding.
