# Verification verdict — d002 / V1 — approval-slice integration

- Commit verified: `340dc91a4898092b510c473c4890e831feffda9a` ("Integrate A1 + T1/T2"),
  from `refs/campaign/orchestrator`, inspected in ~/outputs.
- Frozen app: `~/app` @ `eee783057ec28d28fb36ec05484de9b4b40ceb9c` — integrity check PASS
  (`harness/check-frozen-app.py`, exit 0, 41 files).
- Evidence: `evidence/verifier/d002/` (commands.md + verbatim logs rt-start/reset/test1/test2/cov).
- Date: 2026-11-02, ~20:10–20:12Z.

## Verdict: ACCEPTED — all five dispatch criteria verified

| Criterion | Result | Evidence |
|---|---|---|
| 1. Source citations accurate; EARS have observable assertions | ACCEPT | 14/14 spot-checked citations confirmed (AuthFilter 28-31/33-38/40-47, web.xml 20-23, TaskActionServlet 31-36/48-57/91-98/100-109, Role 13-16, TaskDao 115-125, schema.sql 14-15 & 33, seed.sql users+tasks). All 5 S-APR statements carry observable assertions (HTTP status/Location/body + oracle query). |
| 2. Frozen-app integrity | ACCEPT | exit 0, "Frozen app verified: eee7830… (41 files)" (also re-verified by every runtime command). |
| 3. Independent run, fresh DB | ACCEPT | start/reset/test all exit 0; 9 tests ran, `OK` (verbatim in rt-test1.log). Coverage: LINE 233/500 (46.6%), BRANCH 75/188 (39.9%); scope "All compiled production Java classes in the frozen WAR; JSP, SQL, and CSS are not measured"; collection command `python3 harness/runtime.py coverage`; 17 classes analyzed, app commit matches. |
| 4. S-APR-003/004 are real filter/servlet round-trips asserting unchanged state | ACCEPT | Tests use stdlib urllib HTTP to 127.0.0.1:8080 (httpcase.py — no mocks, no direct DAO). S-APR-003 asserts the FILTER's 403 message (proving AuthFilter fired); S-APR-004 asserts 409 guard message; both assert full unchanged oracle map via `runtime.py sql`. Per-test reset in setUp. |
| 5. Second run without re-reset | ACCEPT | `runtime.py test` again with no intervening reset: `OK`, exit 0 (rt-test2.log). No dependence on prior test data. |

## Suspected defects in the submitted work (tests/rules/specs)

None observed. Nothing failed; no expected-vs-observed mismatches.

## Notes (not defects in the submission; recorded for the orchestrator)

- Rules file itself flags out-of-slice gaps G-APR-001 (bulkClose has no approver check / no state
  guard) and G-APR-002 (unhandled NumberFormatException → 500 on malformed id). These are app
  behaviors outside the approval slice, already disclosed in rules/R-APR.md; not re-reproduced
  here (3-minute budget). No independent defect reproduction attempted.
- Coverage is partial (LINE 46.6%) — expected for a 5-test slice over 17 classes; scope and
  command are honestly stated in summary.json. Not a gap in reporting, but slice coverage of
  e.g. bulkClose/ApiTaskServlet paths is naturally absent.
- Infrastructure: no failures. Runtime seat started first try; all six commands exit 0.

## What was NOT verified

- G-APR-001/002 not independently reproduced (out of slice, time-boxed).
- No negative check that the jacoco.exec data came solely from this suite run (single session
  file written by the harness's own coverage command; scope statement trusted but consistent
  with the 17-class app).
