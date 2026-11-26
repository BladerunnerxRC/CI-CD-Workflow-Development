# How GitHub Actions workflows work

## Run lifecycle

1. **An event happens**: push, pull request, tag, schedule, or a manual "Run workflow".
2. **GitHub reads the workflow files** in `.github/workflows/` **at the commit that triggered
   the event**. Editing a workflow on a `dev-*` branch only affects runs on that branch until merged.
3. **Triggers are matched** against `on:` and its filters (`branches:`, `paths:`, `tags:`).
   No match, no run.
4. **A workflow run is created.** Jobs are queued; `needs:` sets their order. Jobs without
   dependencies start in parallel.
5. **Each job gets a runner** (`runs-on:`). GitHub-hosted runners are fresh VMs, discarded after
   the job. A `GITHUB_TOKEN` scoped by `permissions:` is injected.
6. **Steps execute in order.** `uses:` calls an action; `run:` executes shell. The first failing
   step stops the job and its dependents are skipped, unless overridden with `if:` status
   functions (`!cancelled()`, `always()`, `failure()`) or `continue-on-error`.
7. **Results are reported**: logs, a status check per job on the commit/PR, uploaded artifacts,
   and the run summary. Required status checks can block a PR merge.

See [`diagrams.md`](diagrams.md) for the flowchart.

## File anatomy

```
workflow (ci-cd.yml)
├── on:            triggers and inputs
├── permissions:   default GITHUB_TOKEN scope
├── concurrency:   overlap rules
└── jobs:
    ├── build      runs-on: ubuntu-latest      (own VM)
    │   └── steps: checkout → login → build image
    └── test       needs: build                (own VM)
        └── steps: login → pytest → upload report
```

## Building blocks

- **Workflow.** One YAML file. A repo can have many; each triggers independently.
- **Event (`on:`).** `push`, `pull_request`, `schedule` (cron), `workflow_dispatch` (manual),
  `release`, and others. Filters narrow it down.
- **Job.** Runs on one runner. Jobs can't see each other's files.
- **Step.** Runs in order inside a job; steps share the workspace.
- **Runner.** GitHub-hosted (`ubuntu-latest`) or self-hosted (e.g. a homelab box).
- **Passing data.** Small values via `outputs:` → `needs.<job>.outputs.*`. Files via artifacts.
  A job can only read outputs of jobs listed directly in its own `needs:`.
- **Expressions and contexts.** `${{ }}` is evaluated before the step runs: `github.ref`,
  `needs.*`, `secrets.*`, `vars.*`, `inputs.*`. Drives `if:` conditions. Pass values into shell
  through `env:` rather than inlining `${{ }}` in scripts (script-injection hygiene).
- **`GITHUB_TOKEN`.** Created per run, expires at the end, scoped by `permissions:`. When a job
  declares `permissions:`, any scope it doesn't list is `none`.
- **Environments.** Named deploy targets with their own secrets, variables and protection rules.
- **Status checks.** Each job reports pass/fail on the commit. Branch protection can require them.

## Private-repo constraints (relevant to this pipeline)

- On GitHub Free, Pro and Team plans, **environment required reviewers and wait timers only work
  on public repos.** That's why production promotion here is a manual `workflow_dispatch`.
- **GitHub Pages** for a private repo needs a paid plan, and outside Enterprise the published
  site is public. Pages publishing is therefore opt-in (`vars.PUBLISH_PAGES`).
- The default `github-pages` environment only allows deployments from the default branch.
