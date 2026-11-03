# d003 / T2 reply — approval-slice tests

**Commit:** `8566315` on `cs-sandbox/test-engineer-077665b6.task-tracker-01-077665b6`
(includes merge `1175e63` of orchestrator d7e6bdf: R-APR/S-APR/fixtures — inspected, not re-litigated).

## Test command & result
- `python3 harness/runtime.py test` → **exit 0**, 9 tests, all OK (log: `evidence/test-engineer/t2-test.log`).
- New tests in `tests/test_approval_slice.py` (built on `httpcase.py`; per-test `reset`, oracle via `runtime.py sql`, never HTML scraping; no reliance on ids/order).

## Pass/fail per spec (all PASS)
| Test | Spec | Rules | Result |
|---|---|---|---|
| test_s_apr_001_mgr_approve_task3 | S-APR-001 | R-APR-001,003,004,005 | PASS — 302 /tasks; oracle 3 DONE, others unchanged |
| test_s_apr_002_admin_reject_task3 | S-APR-002 | R-APR-001,003,004,005 | PASS — 302 /tasks; oracle 3 IN_PROGRESS |
| test_s_apr_003_dev_approve_forbidden | S-APR-003 | R-APR-001 | PASS — 403 + exact refusal body; task 3 still PENDING_APPROVAL |
| test_s_apr_004_approve_open_task_conflict | S-APR-004 | R-APR-004 | PASS — 409 + exact "Cannot approve a task in status OPEN (expected PENDING_APPROVAL)"; task 1 still OPEN |
| test_s_apr_005_anonymous_redirected_to_login | S-APR-005 | R-APR-001 | PASS — 302 Location /login; task 3 unchanged |

No suspected defects: observed behavior matches all accepted specs.

## Mapping
`reports/tests/mapping.json` — test name → spec ID(s) + rule ID(s), machine-readable.

## Coverage (`python3 harness/runtime.py coverage`, exit 0; log `evidence/test-engineer/t2-coverage.log`)
Measured on frozen production sources @ `eee7830…`; scope per `reports/coverage/summary.json`:
**all compiled production Java classes in the frozen WAR; JSP, SQL, CSS not measured.**
- **LINE: 233 covered / 267 missed (500 total, ~46.6%)**
- **BRANCH: 75 covered / 113 missed (188 total, ~39.9%)**
- (METHOD 64/44, CLASS 12/5.) Totals include d002 smoke/skeleton tests plus the new slice.

## Evidence
`t2-test.log`, `t2-coverage.log` in `evidence/test-engineer/` (d002 evidence untouched); full JaCoCo report + `summary.json` under `reports/coverage/`.

## Gaps / notes
- Ordering cross-checks from fixtures.md (id=999 403/404 variants) not yet implemented — candidates for a later slice.
- Coverage above is whole-suite; per-spec attribution not measured.
