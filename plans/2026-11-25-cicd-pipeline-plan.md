# Plan: CI/CD pipeline v4 adoption — 2026-11-25

**Goal.** Adopt `workflows/ci-cd.yml` (v4) in a private project repo and prove it end to end:
branch CI → merge → staging → manual production promotion → rollback.

**Status:** Draft. Target project repo not chosen.

## Open decisions

- [ ] **Target repo.** Which private repo gets the pipeline first (an existing Docker project, or a new sandbox repo).
- [ ] **GitHub plan.** Free vs Pro decides whether Pages is even available on a private repo.
- [ ] **Runner.** GitHub-hosted (default) vs self-hosted in the homelab for the deploy jobs (keeps deploy credentials off GitHub-hosted machines).
- [ ] **Deploy mechanism** behind `scripts/deploy.sh` (e.g. SSH + `docker compose pull && up -d`).

## Setup checklist (in the project repo)

- [ ] Multi-stage Dockerfile with `runtime` and `test` stages (`examples/Dockerfile.example`).
- [ ] `requirements-dev.txt` with `pytest` and `pytest-html`; at least one test under `tests/`.
- [ ] `scripts/deploy.sh <environment> <image>`; executable bit set.
- [ ] Copy `workflows/ci-cd.yml` → `.github/workflows/ci-cd.yml`.
- [ ] Settings → Environments: create `staging` and `production`; put deploy secrets there.
- [ ] Settings → Branches: protect `main`; require PR; required checks `build`, `test`, `scan`, `Test results`.
- [ ] Repo variables (optional): `STAGING_URL`, `PROD_URL` (enable smoke tests), `PUBLISH_PAGES=true` (Pages report).
- [ ] Confirm Trivy tag: `docker manifest inspect aquasec/trivy:0.70.0`; append `@sha256:` digest.

## Validation runs

1. [ ] Push a `dev-*` branch → build/test/scan pass; no deploy jobs run.
2. [ ] Push a deliberately failing test → `test` red, evidence still uploaded, check annotated.
3. [ ] Open PR → required checks block merge until green.
4. [ ] Merge → CI on `main` → `deploy-staging` + smoke test.
5. [ ] Run workflow on `main` with `promote_to_production` → gate 1 + gate 2 pass → prod deploy.
6. [ ] Promote while staging still running → gate 1 refuses with a clear error.
7. [ ] Rollback drill: redeploy the previous SHA tag.

## Backlog (proposed improvements)

- [ ] Pin every `uses:` to a full commit SHA (Dependabot can keep them updated).
- [ ] `promote_sha` input to promote/roll back a specific older commit through the same gates.
- [ ] Sign images with `cosign` (keyless, OIDC) and verify the signature before prod deploy.
- [ ] Upload Trivy SARIF to code scanning (needs `security-events: write`; GHAS on private repos).
- [ ] Lint job (ruff, hadolint, actionlint) before build — fail fast.
- [ ] Track DORA metrics from workflow run history.
