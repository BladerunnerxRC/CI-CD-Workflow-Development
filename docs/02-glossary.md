# Glossary

Grouped by where each term shows up in the pipeline.

## Source control

| Term | Definition |
|---|---|
| Commit | A saved snapshot of code changes, identified by a unique hash (SHA). It exists only locally until pushed. |
| Push | Uploading local commits to the shared remote repository. The most common pipeline trigger. |
| SHA | The 40-character hash identifying a commit. Can't be moved or reused, so it's a reliable build tag. |
| Branch | A separate line of development (e.g. `dev-add-auth`). |
| `main` | The primary branch: releasable or already-deployed code. |
| PR (pull request) | A request to merge one branch into another; where review and required checks happen. GitLab calls it an MR. |
| Merge | Combining one branch's commits into another. |
| Tag | A named, permanent label on a commit (e.g. `v1.4.0`), often marking a release. |
| Branch protection | Rules on a branch: no direct pushes, review required, checks must pass before merge. |
| Required status checks | The specific CI jobs that must succeed before the merge button works. |

## CI: build and verify

| Term | Definition |
|---|---|
| CI (continuous integration) | Automatically building and testing every change as it's pushed. |
| Build | Turning source into something runnable: compiling, installing dependencies, building an image. |
| Compile | Translating source into machine code or bytecode (C, Go, Java, Rust). Python mostly skips this. |
| Lint | Static analysis without running the code: style, unused variables, likely bugs. `ruff`, `eslint`, `shellcheck`, `yamllint`, `hadolint`. |
| Unit test | An automated test of one small piece of code in isolation. Fast; runs on every push. |
| Integration test | Tests several components together (app + database). Slower; often in staging. |
| Smoke test | A minimal "is it alive" check after deploy, e.g. a health endpoint returning 200. |
| SAST | Static application security testing: scanning source for security flaws. Semgrep, CodeQL, Bandit. |
| CVE | A publicly catalogued security flaw with an ID like `CVE-2024-3094`. |
| Trivy | Open-source scanner for container images, filesystems and IaC files. |
| Fail fast | Cheap checks first; the run stops at the first failure. |

## Package

| Term | Definition |
|---|---|
| Artifact | The build output that gets deployed: image, `.jar`, binary, wheel. |
| Container image | Immutable packaged filesystem with the app and its dependencies. A running copy is a container. |
| Registry | Storage server for images: GHCR, Docker Hub, Harbor. |
| Versioning | Labeling artifacts: commit-SHA tags (immutable) and SemVer (`MAJOR.MINOR.PATCH`). |
| Build once, promote many | The same artifact moves staging → production unchanged. |
| Cache | Saved output of unchanged steps (Docker layers, dependencies) to speed later runs. |
| SBOM | Software bill of materials: machine-readable inventory of everything in an artifact. `syft`. |
| Provenance | A signed record of how, where and from what source an artifact was built. |
| Image signing | Cryptographic proof an artifact came from your pipeline unaltered. `cosign`. |

## CD: deploy and release

| Term | Definition |
|---|---|
| Continuous delivery | Every passing change is deployable; a human or policy decides when to release. |
| Continuous deployment | Every passing change goes to production automatically. |
| Environment | A distinct place the app runs (dev, staging, prod) with its own config, secrets and rules. |
| Staging | Mirrors production as closely as possible; the final rehearsal. |
| Production (prod) | The live environment real users depend on. |
| Approval gate | Required checkpoint before prod: human sign-off or automated policy. |
| Canary release | New version to a small slice of traffic first (e.g. 5%); ramp up if healthy, pull back if not. Named for canaries in coal mines. |
| Blue-green deployment | Two identical prod environments; switch traffic from old (blue) to new (green) in one move. |
| Rolling deployment | Replace instances a few at a time. Kubernetes default. |
| Feature flag | Runtime switch that turns code on/off without redeploying. Separates deploy from release. |
| Rollback | Return to the previous known-good version. With SHA tags: redeploy the previous tag. |

## Monitor

| Term | Definition |
|---|---|
| Monitoring | Tracking metrics, logs and traces (Prometheus, Grafana, Checkmk). |
| Observability | How well you can infer internal state from outputs, including unforeseen failures. |
| Alerting | Automatic notification when a metric crosses a threshold. |
| MTTR | Mean time to recovery: average time from failure to restored service. |
| DORA metrics | Deployment frequency, lead time for changes, change failure rate, MTTR. |

## GitHub Actions

| Term | Definition |
|---|---|
| Workflow | A YAML file in `.github/workflows/` defining a pipeline. |
| Trigger (`on:`) | Event that starts a run: push, pull_request, schedule, workflow_dispatch, release. |
| Job | Group of steps on one runner. Parallel by default. |
| Step | One command (`run:`) or action (`uses:`) inside a job. Steps share the job's workspace. |
| Action | Reusable packaged step, e.g. `actions/checkout`. |
| `needs:` | Declares job dependencies; creates ordering and parallel fan-out. |
| Runner | Machine executing a job: GitHub-hosted VM or self-hosted. |
| Outputs | Small values passed from one job to another via `needs.<job>.outputs.*`. |
| Artifacts | Files passed between jobs or kept after a run (`upload-artifact` / `download-artifact`). |
| Contexts / expressions | `${{ }}` values evaluated by GitHub before a step runs: `github.*`, `needs.*`, `secrets.*`, `vars.*`, `inputs.*`. |
| Secrets | Encrypted values injected at runtime and masked in logs. |
| `GITHUB_TOKEN` | Per-run credential, scoped by `permissions:`, expires when the run ends. |
| Environment protection | Per-environment secrets and rules (reviewers, wait timers, allowed branches). Reviewers only on public repos for Free/Pro/Team. |
| Concurrency | Rule preventing overlapping runs; can cancel the older run in a group. |
| SHA pinning | `uses: owner/action@<full commit sha>` instead of a movable tag like `@v4`. |
| `workflow_dispatch` | Manual "Run workflow" trigger, optionally with typed inputs. Used here as the prod gate. |
