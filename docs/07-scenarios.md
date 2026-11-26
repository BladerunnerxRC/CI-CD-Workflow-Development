# Scenarios — what happens to a commit

How far a commit gets through `ci-cd.yml` v4 in common situations.
✅ pass · ❌ fail · ⏸ waiting · — skipped

| Scenario | build | test | scan | staging | promote gates | production | Result |
|---|---|---|---|---|---|---|---|
| Push to a `dev-*` branch | ✅ | ✅ | ✅ | — | — | — | CI only. PR shows green checks. |
| Merge to `main` | ✅ | ✅ | ✅ | ✅ | ⏸ | — | Staged and smoke-tested; waits for manual promotion. |
| Failing test | ✅ | ❌ | ✅ | — | — | — | Reports still uploaded; nothing deploys. |
| HIGH/CRITICAL CVE | ✅ | ✅ | ❌ | — | — | — | SARIF saved; nothing deploys until patched. |
| Manual promotion on `main` | — | — | — | — | ✅ | ✅ | Same image deployed; no rebuild. |
| Promotion before staging finished | — | — | — | — | ❌ | — | Gate 1 finds no successful `deploy-staging` for this SHA. |

Notes:

- `test` and `scan` run in parallel; staging needs both.
- A promotion run skips build/test/scan/staging by design. Gate 1 proves the *push* run for
  the same commit already passed staging.
- Rollback = run the deploy for the previous SHA tag (backlog: a `promote_sha` input so this
  goes through the same gates).
