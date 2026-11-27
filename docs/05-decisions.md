# Decision log

Lightweight decision records. Newest first. Status: **Accepted**, **Proposed**, **Superseded**.

---

## D-009 — Stay on GitHub Actions; Gitea not adopted (Accepted, 2026-11-26)

**Context.** Evaluated whether `ci-cd.yml` v4 would run on Gitea Actions. The YAML structure,
`needs:`, outputs, `workflow_dispatch` inputs, `vars`, job `permissions:` and (Gitea 1.26+)
`concurrency:` carry over. These do not: `environment:` (ignored), GHCR push with the job token
(Gitea's token can't publish to its own registry), Checks API (test-result check and gate 1),
GitHub Pages, stock `upload-artifact@v4`, `type=gha` build cache, and `docker run -v $PWD`
bind mounts inside containerized `act_runner` jobs (evidence lands on the wrong path).

**Decision.** Stay on GitHub (hosted repos + Actions), with work done in a local clone.
No Gitea variant of the workflow will be maintained.

**Consequences.** + One workflow, one platform, no compatibility shims. − Private-repo limits
remain (no environment reviewers on Free/Pro/Team; manual promotion stays the gate).
Revisit only if self-hosting the git server becomes a requirement.

## D-008 — Production promotion is a manual `workflow_dispatch` (Accepted, 2026-11-25)

**Context.** Target repos are private. On GitHub Free/Pro/Team, environment required reviewers
only work on public repos, so an `environment: production` gate would silently not exist.

**Decision.** `deploy-prod` runs only on a manual "Run workflow" on `main` with
`promote_to_production: true`. It deploys the image already built for that commit, after
verifying (1) a successful `deploy-staging` check run for that SHA and (2) the image exists.

**Consequences.** + Real human gate on a private repo. + No rebuild between staging and prod.
− Anyone with write access can promote. − Promotion targets main's HEAD at dispatch time; to
promote an older commit, a SHA input would be needed (see plan backlog).

## D-007 — GitHub Pages test report is opt-in (Accepted, 2026-11-25)

Pages on a private repo needs a paid plan and is public outside Enterprise. The
`test-reports` artifact (private, 30-day retention) is the default evidence store.
Enable Pages with repo variable `PUBLISH_PAGES=true`.

## D-006 — Multi-stage image: `runtime` and `test` targets (Accepted, 2026-11-23)

Test tooling must not ship in the production image. The build job builds both targets;
`test` runs in the test image, `scan` and deploys use the runtime image.

## D-005 — `push` trigger only, no `pull_request` (Accepted, 2026-11-23)

Running both doubled every run on a branch with an open PR. Status checks attach to the commit
SHA, so push runs satisfy PR required checks. Trade-off: branches not matching
`main`/`dev-*`/`fix-*` get no CI.

## D-004 — Least privilege per job (Accepted, 2026-11-23)

Top-level `permissions: contents: read`; each job grants only what it needs.

## D-003 — Never cancel runs on `main` (Accepted, 2026-11-23)

`cancel-in-progress` only for non-main refs; deploy jobs have their own non-cancelling
concurrency groups. Concurrency key includes `event_name` so promotions queue separately.

## D-002 — Evidence is retained for every run (Accepted, 2026-11-23)

JUnit + pytest-html reports, a published check, Trivy SARIF, SBOM and provenance. Rationale:
audit trail / batch-record equivalent for regulated contexts.

## D-001 — Build once, promote by commit SHA (Accepted, 2026-11-22)

Images are tagged `ghcr.io/<owner>/<repo>:<commit-sha>` and the same image is promoted through
every environment. Branch naming follows `dev-` (enhancements) / `fix-` (bug fixes).
