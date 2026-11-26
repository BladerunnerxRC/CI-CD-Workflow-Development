# Diagrams

Illustrations (SVG, adapt to light/dark mode) followed by Mermaid sources; GitHub renders both inline.

## Illustrations

| | |
|---|---|
| **v4 overall flow** — push path vs manual promotion | ![Overall flow](visuals/01-overall-flow.svg) |
| **Build once, promote many** — one SHA-tagged image through every stage | ![Image promotion](visuals/02-image-promotion.svg) |
| **Test job evidence pattern** — evidence first, verdict last | ![Test evidence pattern](visuals/03-test-evidence-pattern.svg) |

Per-job details: [`06-job-reference.md`](06-job-reference.md). Scenario walk-through: [`07-scenarios.md`](07-scenarios.md).

## Mermaid sources

## 1. Pipeline overview

The approval gate is what separates continuous **delivery** (gate kept) from continuous
**deployment** (gate removed). A failure at any stage stops the run.

```mermaid
flowchart LR
    subgraph CI["Continuous integration"]
        direction LR
        C[Commit<br/>push / PR] --> B[Build<br/>compile, lint] --> T[Test<br/>tests + scans] --> P[Package<br/>image + SHA tag]
    end
    subgraph CD["Continuous delivery"]
        direction LR
        S[Staging<br/>prod-like env] --> G{{Approval gate<br/>manual promotion}} --> PR[Production<br/>same image] --> M[Monitor<br/>observe, roll back]
    end
    P --> S
```

## 2. When CI and CD run (git flow)

```mermaid
gitGraph
    commit id: "base"
    branch dev-add-auth
    checkout dev-add-auth
    commit id: "CI pass"
    commit id: "CI fail" type: REVERSE
    commit id: "CI pass (fix)"
    checkout main
    merge dev-add-auth id: "PR merged -> staging" type: HIGHLIGHT
    commit id: "manual promote -> prod"
```

- Every push to `dev-*` / `fix-*` → CI.
- PR into `main` → required checks + review gate the merge.
- Merge to `main` → CI + deploy to staging.
- Manual "Run workflow" on `main` → production.

## 3. Job dependency graph (v4)

```mermaid
flowchart LR
    TR([push]) --> BU[build]
    BU --> TE[test]
    BU --> SC[scan]
    TE --> ST[deploy-staging<br/>main only]
    SC --> ST
    TE --> PG[publish-test-report<br/>main + PUBLISH_PAGES]
    WD([workflow_dispatch<br/>promote_to_production]) --> DP[deploy-prod]
    ST -. "gate 1: staging passed<br/>for this SHA" .-> DP
```

## 4. GitHub Actions run lifecycle

```mermaid
flowchart TD
    E[Event happens<br/>push, PR, tag, schedule, manual] --> R[Read .github/workflows/*.yml<br/>at that commit]
    R --> M{on: triggers<br/>and filters match?}
    M -- no --> X[No run]
    M -- yes --> W[Create workflow run<br/>jobs queued, needs: sets order]
    W --> A[Assign a runner per job<br/>fresh VM, GITHUB_TOKEN injected]
    A --> S[Execute steps in order<br/>uses: actions, run: shell]
    S --> O[Report results<br/>logs, status checks, artifacts]
    O -. required checks .-> PRM[Allow or block PR merge]
```

## 5. Production promotion (v4)

```mermaid
sequenceDiagram
    actor You
    participant GH as GitHub Actions
    participant API as Checks API
    participant REG as GHCR
    participant PROD as Production
    You->>GH: Run workflow on main, promote_to_production = true
    GH->>GH: build/test/scan/staging skipped (no rebuild)
    GH->>API: deploy-staging succeeded for this SHA?
    API-->>GH: yes (else fail with error)
    GH->>REG: image :SHA exists?
    REG-->>GH: yes (else fail)
    GH->>PROD: deploy.sh production image:SHA
    GH->>PROD: smoke test /health (if PROD_URL set)
    GH-->>You: run summary: commit, image, promoted by
```
