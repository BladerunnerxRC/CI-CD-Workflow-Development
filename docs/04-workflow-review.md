# Review of v2 (user-modified workflow) — 2026-11-23

Source reviewed: [`workflows/history/ci-cd.v2-user-modified.yml`](../workflows/history/ci-cd.v2-user-modified.yml).
All findings were fixed in v3; v4 then changed the production gate for private repos.

## Verdict

v2 was a real improvement on v1: SBOM, build provenance and retained test evidence are the audit
trail regulated industries ask for. But it would not run cleanly: two steps failed outright and
three more failed or misbehaved depending on repo settings.

## Would fail

| # | Finding | Fix (v3) |
|---|---|---|
| 1 | Line 167 had `${GITHUB\_REPOSITORY}` etc. (markdown-escaped underscores). Bash: `bad substitution`. | Dropped the `gh api` lookup; use `upload-artifact`'s `artifact-url` output. |
| 2 | `publish-unit-test-result-action` needs `checks: write`, `pull-requests: write` (+ `contents: read`, `issues: read` on private repos). Job had only `contents`/`packages: read`. | Added the permissions to the test job. |
| 3 | Pages deploy ran on every branch, but the `github-pages` environment only allows the default branch. One site per repo also means branches overwrite each other. | Pages publish limited to `main`. |
| 4 | `if: always()` on publish job: runs when `build` failed (no artifact to download) and on cancel. | `!cancelled() && needs.test.result != 'skipped'`. |
| 5 | Trivy `0.76.0` could not be confirmed to exist (newest found: `0.70.0`). Trivy's own action has had a compromised release. | Pinned `0.70.0`; TODO: append digest. |

## Would misbehave

| # | Finding | Fix (v3) |
|---|---|---|
| 6 | `push` + `pull_request` both on: every branch push with an open PR ran twice (different refs → different concurrency groups) and pushed a second image. | Kept `push` only; checks attach to the SHA so they satisfy PR required checks. |
| 7 | Top-level `pages: write` + `id-token: write` inherited by deploy jobs. | Top level `contents: read` only; per-job grants. |
| 8 | `cancel-in-progress: true` could kill a prod deploy mid-flight on `main` (inherited from v1). | Conditional on non-main; job-level deploy concurrency. |
| 9 | Environment required reviewers aren't available on private repos (Free/Pro/Team): prod would deploy ungated. | v4: manual `workflow_dispatch` promotion. |
| 10 | Container writing to a runner-owned mount fails if the image uses a non-root `USER`. | `--user "$(id -u):$(id -g)"`, `HOME=/tmp`, `-p no:cacheprovider`. |

## Design issues

- Test tooling (pytest, pytest-html) shipped in the production image → attack surface and false
  CVE findings. **Fix:** multi-stage Dockerfile, separate `test` target.
- Trivy left no evidence beyond the log. **Fix:** SARIF report uploaded as an artifact.
- Pages site likely public even for a private repo. **Fix (v4):** opt-in only.
- Minor: `page_url` trailing slash → `//report.html`; no `index.html`; redundant `success()`.

## What was solid (kept)

- `continue-on-error` → publish evidence → explicit fail step.
- `sbom: true`, `provenance` (upgraded to `mode=max`), OCI labels.
- Per-job timeouts, scoped build cache, pinned scanner version.
