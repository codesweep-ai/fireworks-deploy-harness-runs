# Verifier reply — dispatch d002 (independent whole-app verification)

## What I verified
Final integration commit **b38f8dae33762710156ea10ddf29c356130b0787** (= refs/campaign/orchestrator), adopted via merge `73f2281` on my branch; my branch HEAD is now `e040123` and differs from base 8f713fe (branch is non-empty: 107 files, incl. my evidence). Tree diff vs b38f8da is exactly my four verifier-owned files — integration content unmodified.

## Suite results (3 runs, all executed by me on my own podman runtime)
1. reset + test (fresh): **Ran 137 tests, OK**, exit 0 (25.7s)
2. test with NO reset (order/state independence on dirty DB): **Ran 137 tests, OK**, exit 0 (24.6s)
3. reset + test (fresh again): **Ran 137 tests, OK**, exit 0 (24.7s)

Engineer's 137/137 claim independently confirmed. Frozen-source checker passed inside every harness invocation (41 files, commit eee783057ec28d28fb36ec05484de9b4b40ceb9c).

## Coverage (JaCoCo 0.8.12 tcpserver dump) — reproduces engineer's numbers EXACTLY
INSTRUCTION 1993/2155, BRANCH 168/188, LINE 452/500, METHOD 103/108, CLASS 17/17. Denominator: 18/18 scope.json sourcefiles present in report; CLASS=17 because TaskStatus.java has zero executable lines (accounted). Explicitly uncovered: dead DAO methods (TaskDao.findByProject, ProjectDao.findById — no HTTP route), SQLException catches, awaitReady retry arms, defense-in-depth requireApprover arms masked by AuthFilter, esc(null). JSP/SQL/CSS verified behaviorally, outside the Java metric. Note: first two coverage attempts failed (my rm of untracked baseline orphaned the /coverage bind mount; harness/environment, no code touched) — recovered via runtime.py start; recorded.

## Per-family audit
All 8 families: mapping links resolve, 0 ear:null (137 mapped), representative assertions inspected, one-real-request-per-route evidence executed by me (curl + DB). PASS each; details in reports/verification/family-audit.md.

## Defect candidates — independently REPRODUCED and CONFIRMED
- **D-002 CONFIRMED**: MEMBER (dev) bulkClose on a PENDING_APPROVAL task → 302, status DONE (bypasses approve gate; handleBulkClose has no role check).
- **D-010 CONFIRMED**: POST /tasks/form edit with id=999 → 302 `/tasks/form?id=999&saved=1`, zero rows updated (silent no-op reported as success).
- Also confirmed (pre-integration, app behaviors): /api/tasks emits invalid JSON for control chars in title (esc() misses LF/TAB); UTF-8 form POST stored as mojibake (no request.setCharacterEncoding); task titles render unescaped in JSP (EL, no c:out). All are app defects — visible, unfixed (frozen app); intent adjudication is the orchestrator's.

## Files (branch e040123)
- evidence/verifier/source-integrity.md (STEP 0: commit, hashes, images/containers)
- evidence/verifier/probes.md (independent HTTP/DB probes, pre-integration)
- evidence/verifier/integration-run.md (3 runs, coverage exact-match, uncovered branches, D-002/D-010 repros, harness incident)
- evidence/verifier/coverage-audit.md + evidence/verifier/audit-coverage.py (denominator audit tooling)
- evidence/verifier/rows-parser.py (probe helper)
- reports/verification/plan.md (apparatus; written pre-integration)
- reports/verification/family-audit.md (per-family verdicts + gaps)

## Unverified areas
SQLException/awaitReady/fault-injection arms and the two dead DAO methods (no HTTP path) have no executed evidence — named, technical-only. Everything else in scope has executed evidence from my own runtime.
