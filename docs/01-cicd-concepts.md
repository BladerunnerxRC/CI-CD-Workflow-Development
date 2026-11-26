# CI/CD concepts

## Definition

> **CI/CD is an automated, repeatable path that every code change must take from commit to
> production. It builds, tests and scans the change at each stage, promotes the same verified
> artifact through each environment, and records evidence along the way.**

Three corrections to the narrower "a workflow to standardize and checkpoint early development
changes and bug fixes" reading:

1. **Every change, not just early work and bug fixes.** Features, config, dependency upgrades,
   security patches, infrastructure (the Dockerfile itself) and hotfixes all take the same path.
   Nothing reaches production any other way.
2. **Automation, not just checkpoints.** The checkpoints are scripts that run on every change, the
   same way every time. Humans step in at a few deliberate points: code review and the production gate.
3. **The CD half ships.** CI verifies; CD moves the verified artifact through staging into
   production, monitors it, and rolls it back if needed.

## CI vs continuous delivery vs continuous deployment

| Term | What it means |
|---|---|
| Continuous integration (CI) | Every push is automatically built and tested, so problems surface in minutes. |
| Continuous delivery | Every change that passes CI is *deployable*; a human or a policy decides when it goes to production. |
| Continuous deployment | Every change that passes CI goes to production automatically, with no manual gate. |

The difference between the last two is a single gate before production. This pipeline is
**continuous delivery**: production promotion is a manual `workflow_dispatch`.

## Pipeline stages

| Stage | Purpose | In this pipeline |
|---|---|---|
| Commit / push | A change enters the pipeline | Push to `main`, `dev-*`, `fix-*` |
| Build | Turn source into a runnable artifact | `build` job: runtime + test images, SBOM, provenance |
| Test | Prove the artifact works | `test` job: pytest inside the test image, JUnit + HTML evidence |
| Scan | Prove the artifact is safe | `scan` job: Trivy, SARIF evidence, gate on HIGH/CRITICAL |
| Package | Version the artifact immutably | Image tagged by commit SHA in GHCR |
| Staging | Rehearse in a prod-like environment | `deploy-staging` (main only) + smoke test |
| Approval gate | Human or policy release decision | Manual "Run workflow" → `promote_to_production` |
| Production | Serve real users | `deploy-prod`: same image, verified staged first |
| Monitor | Watch behavior, roll back if needed | Smoke test; rollback = redeploy previous SHA tag |

**Build once, promote many.** The image built from a commit is the exact image that gets
scanned, staged and promoted. Rebuilding between environments means staging validated a
different thing than production runs.

## When each part runs (git flow)

- Every push to a `dev-*` / `fix-*` branch → **CI** (build, test, scan).
- A PR into `main` → reviewed; CI results are **required status checks** (branch protection).
- Merge to `main` → CI again on the merge commit, then **deploy to staging**.
- Manual promotion on `main` → **deploy to production**.

Without branch protection, a red CI run is only advisory: the merge button still works.

## Mapping to a regulated / quality framework

The closest analogy is a validated manufacturing process with in-process controls.

| Manufacturing / QA | CI/CD equivalent |
|---|---|
| Validated, documented process | The pipeline definition (YAML in version control) |
| In-process QC checks | Lint, unit tests, scans |
| Lot / batch number | Commit SHA, image tag |
| Pilot batch | Staging deploy |
| QA release sign-off | Approval gate (manual promotion) |
| Batch record | Pipeline logs, test results, SBOM, provenance, SARIF |
| Deviation / nonconformance | Failed run, which blocks promotion |
| Recall | Rollback |

In regulated industries (pharma, medtech, defense), CI/CD's value is a **controlled, auditable
change-management process** first; speed comes second. The terms that map to GxP (good
practice regulations, as in pharma) and CMMC (the US Defense Department's Cybersecurity Maturity
Model Certification) concerns are the approval gate, SBOM, image signing, environment
protection and DORA metrics.
