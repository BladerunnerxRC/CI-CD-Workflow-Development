# Workflow changelog

All versions are kept in [`workflows/history/`](workflows/history/). The current version is
[`workflows/ci-cd.yml`](workflows/ci-cd.yml).

## v4 — 2026-11-25 — private-repo promotion gate (current)

Context: the target repos are **private**. On GitHub Free/Pro/Team, environment
"required reviewers" only work on public repos, so the v3 approval gate would silently not exist.

- **Production is promoted manually.** `deploy-prod` runs only on `workflow_dispatch` on `main`
  with `promote_to_production: true`. The click is the human approval gate.
- **Promotion never rebuilds.** On a promotion run, `build` (and therefore test/scan/staging) is
  skipped; `deploy-prod` deploys the image the push run for that commit already built.
- **Two promotion gates:** (1) GitHub check-runs API confirms a successful `deploy-staging` job for
  that exact SHA; (2) `docker manifest inspect` confirms the image still exists in GHCR.
- **Concurrency key includes `event_name`.** A new push can't cancel or replace a pending promotion.
- **Pages publishing is opt-in** (`vars.PUBLISH_PAGES == 'true'`). Pages on a private repo needs a
  paid plan, and on non-Enterprise plans the site is public even when the repo is private.
- Promotion details (commit, image, actor) are written to the run summary as audit evidence.

## v3 — 2026-11-23 — review fixes

Fixes from the review of v2 (details in [`docs/04-workflow-review.md`](docs/04-workflow-review.md)):

- Fixed the bash `bad substitution` error (escaped underscores) by replacing the `gh api` lookup
  with `upload-artifact`'s own `artifact-url` output.
- Added `checks: write`, `pull-requests: write`, `issues: read` for the test-results publisher.
- `publish-test-report` limited to `main`, with `!cancelled()` and a skipped-test guard.
- Trivy pinned to `0.70.0` (newest confirmed), digest pin left as a TODO.
- Dropped the `pull_request` trigger (was double-running every branch push).
- Least-privilege `permissions` at the top level; deploy jobs no longer inherit Pages/OIDC scopes.
- `cancel-in-progress` disabled on `main`; job-level concurrency on each deploy.
- Test container runs as the runner uid so it can write results.
- Split the image into `runtime` (prod) and `test` stages — test tools no longer ship to prod.
- Trivy now writes a retained SARIF report before the pass/fail gate.
- Pages: `index.html`, trailing-slash fix. Optional smoke tests via `STAGING_URL` / `PROD_URL`.

## v2 — 2026-11-23 — user-modified

Added test evidence (JUnit + pytest-html, published check, uploaded artifact, GitHub Pages report),
SBOM + provenance, OCI labels, per-job timeouts, a pinned Trivy version, and a `pull_request`
trigger. Kept verbatim as `history/ci-cd.v2-user-modified.yml`.

## v1 — 2026-11-22 — original teaching example

build → (test ∥ scan) → staging → prod, image tagged by commit SHA and promoted unchanged.
