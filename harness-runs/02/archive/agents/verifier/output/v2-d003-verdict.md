# V2/d003 — independent verification of run-02 integration (verifier seat)

- Date: 2026-11-02 ~20:46–20:53Z, verifier's own machine and runtime seat (http://127.0.0.1:8080).
- Verified commits: **I1 = d90a91a** (suite executed on I1 tree) and **I2 = 35b9a30**
  (merged as HEAD 261a566; suite re-executed on I2 tree). Frozen app
  `eee783057ec28d28fb36ec05484de9b4b40ceb9c` (41 files).
- Evidence: `evidence/verifier/d003/` (verbatim logs incl. exit codes in commands.md).

## Verdict: ACCEPT (integration verified; app defects confirmed as observed facts, left unfixed)

| # | Check | Result |
|---|-------|--------|
| 1 | Whole suite run TWICE with `reset` between (I1) | **53/53 OK both runs** (rt-test1.log, rt-test2.log; exit 0; 20.4s/18.5s) — order/state independent |
| 2 | Coverage after run 2 | **LINE 449/500, BRANCH 159/188, CLASS 17/17** reproduced exactly (rt-cov.log). Denominator = frozen app: 18 scope sources, 17 with counters; TaskStatus.java present in report.xml with **zero counters/0 lines** (constants holder) — matches. Java-only; JSP/SQL/CSS not measured |
| 3 | Mapping vs execution (I2 rows, re-run on I2) | **49/49 mapped rows ran OK** (rt-test3-i2.log, 53/53 OK). Only unmapped tests = 4 `test_skeleton` smoke tests, declared in mapping.json suites as unmapped. **All 89 cited rule/spec IDs exist** in rules/*.md / specs/*.md |
| 4 | F-001 reproduction | **CONFIRMED**: dev/dev (MEMBER) POST `/tasks/action?action=bulkClose&ids=3` → 302; oracle before `3|PENDING_APPROVAL|3|2`, after `3|DONE|3|2` (f001-repro.txt). No role/state guard. Normative policy question belongs to orchestrator |
| 4 | F-002 reproduction | **CONFIRMED**: mgr/mgr POST approve id=abc → **HTTP 500** Tomcat error page (f001-f002-repro.txt) |
| 5 | F-005 spot-check | **CONFIRMED**: projects.jsp:32 links `/projects/delete`; web.xml has no mapping → GET returns **404** |
| 5 | F-006 spot-check | **CONFIRMED**: POST /tasks/form id=2 projectId=2 → 302 saved=1, but oracle shows `project_id` stays 1 (TaskDao.update omits the column) |
| 5 | F-007 spot-check | **CONFIRMED**: POST /tasks/form id=99999 → **302 /tasks/form?id=99999&saved=1**, nothing persisted (count=0) |
| 5 | F-008 spot-check | **CONFIRMED**: title `ctrl\x01char` persisted, GET /api/tasks emits raw `\x01` inside JSON string → strict `json.loads` fails ("Invalid control character") |
| 5 | F-009 spot-check | **CONFIRMED**: GET /tasks?assignee=abc → **HTTP 500** |
| 6 | Inventory audit | app-inventory.json covers **10 routes / 18 technicalFiles / 8 families**; all 8 families have executed evidence on this seat. Declared gaps are honestly typed (no-test-possible / dead-code / unresolved-intent / capability-gap); GAP-COV-001's 17-vs-18 question resolved by TaskStatus finding above |
| 7 | Integrity | `check-frozen-app.py` PASS before (integrity-before.txt) and after (integrity-after.txt); runtime re-verified the frozen app internally on every command |

## I1→I2 reconciliation check

`git diff d90a91a 35b9a30 -- tests/`: 74 adds / 74 dels. After normalizing
`[SR]-[A-Z]+-\d+` tokens and `test_[sr]_*` names, the only residual changes are
docstring header lines gaining a second ID token and the escaping-test method gaining
its spec number — **ID-text changes only, no logic/assertion changes** (verified by
normalized-diff comparison). Suite re-executed on the I2 tree: 53/53 OK.

## Unverified / notes

- Nothing left unverified from the dispatch. F-003/F-004 were not in scope of d003 spot-checks.
- reports/coverage/ overlap resolved per dispatch: my untracked V1 coverage files were removed before merging; the engineer's committed files stand (regenerated summary.json from MY coverage run matches 449/500 etc.).
- No tests, rules, specs, mappings, or app sources authored/modified by me.
