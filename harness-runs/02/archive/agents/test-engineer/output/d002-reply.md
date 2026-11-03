# Reply — d002 (test-engineer, run 02): whole-app HTTP test batches

ALL SIX ITEMS DONE, committed per item on
`cs-sandbox/test-engineer-7aac6234.task-tracker-02-7aac6234` (base 5493ba9).
Full suite green: **53 tests, 0 failures/errors** (9 inherited preserved
untouched + 44 new). NOTE ON PROTOCOL: one reply closes this dispatch, so
per-item replies were impossible; I committed per item instead (verifier can
fetch any prefix). Log tail in `evidence/test-engineer/d002-test-run.log`.

## Per item: file, commit, tests
- T-AUTH `tests/test_auth_session.py` — ce507a8 — 5 tests: logout invalidation,
  anonymous GET×4/POST×3 → 302 /login with no-mutation oracle, /api/tasks
  anonymous 200 (F-003 characterization), `/` + /index.jsp redirect per auth
  state. Login success/failure left to inherited test_skeleton (non-overlap).
- T-PROJ `tests/test_projects.py` — 6e6efd8 — 5 tests: list incl. archived,
  create valid (oracle: id 4 ACTIVE owner=dev/3, name trimmed), blank/space
  name → 400 + no mutation, duplicate-name persists (characterization: no
  UNIQUE on project.name), archive + re-archive no-op + archived visibility.
- T-FORM `tests/test_task_form.py` — 0c7d3e8 — 6 tests: blank form, edit
  prefill (all 5 fields asserted), create defaults (OPEN/MEDIUM/NULLs),
  blank-title and bad-project refusals re-render 200 + no row, nullable
  round-trip set→clear. Attempt 1 failed on project edit; corrected to
  OBSERVED behavior (see findings below). 2/2 attempts used.
- T-LIFE `tests/test_lifecycle_bulk.py` — e7e36dd — 14 tests: start/submit/
  reopen success (as dev — no role gate) + 409 wrong-state refusals with no
  mutation; 404 approver+missing id; 403 non-approver+missing id (F-004
  ordering, filter-first); unknown action 400; **F-002 REPRODUCED**: id=abc
  and missing id → 500, no mutation; **F-001 REPRODUCED**: dev bulkClose
  ids=3 → 302 and oracle shows task 3 PENDING_APPROVAL→DONE (before/after
  `SELECT id,status FROM task ORDER BY id`); mixed ids=2,4 (4 untouched);
  no-ids 302 no-op. Approve/reject slices left to inherited module.
- T-QUERY `tests/test_filters_sorts.py` — 83ddb52 — 9 tests asserting ACTUAL
  id sequences from /tasks HTML: default [4,3,2,1]; status ×4 + bogus→[];
  assignee 3→[3,2,1], 1→[4]; empty assignee = no filter (characterization:
  no unassigned-only filter exists in source); combined; ALL whitelist
  tokens — updated [4,3,2,1], status [4,2,1,3], priority [2,1,3,4]
  (semantic), due/due_date [3,1,2,4] (ASC NULLS LAST), title [4,1,3,2],
  id [1,2,3,4]; invalid/injection-shaped tokens → default fallback.
- T-API `tests/test_api_json.py` — 8f429ac — 5 tests: readback [4,3,2,1]
  exact key set/types (dueDate null for task 4), status filter, assignee
  param IGNORED (characterization: API reads status only), escaping
  round-trip (`A "quoted" <b>bold</b> \path` byte-exact valid JSON, HTML
  NOT escaped — correct for application/json), null rendering, POST/PUT 405.
- Closeout (mapping + results + coverage + evidence) — 3df5112.

## Commands + results
`python3 harness/runtime.py start|reset|sql|test|coverage` — all exit 0;
frozen-app integrity verified on every call. Full transcripts:
`evidence/test-engineer/d002-test-run.log`, `d002-coverage.log`,
`d002-commands.md`; `reports/tests/result.json` (53/0/0).

## Coverage (whole production-Java denominator, vs run-01 baseline)
LINE 449/500 (was 233/500, **+216**); BRANCH 159/188 (was 75/188, **+84**);
INSTRUCTION 1975/2155; METHOD 103/108; CLASS 17/17.
`reports/coverage/summary.json`.

## Mapping status
`reports/tests/mapping.json` updated: all 44 new tests mapped. **PLACEHOLDER
IDs** (S-AUT/S-PRJ/S-FRM/S-LIF/S-QRY/S-API + R-*) — no analyst rule/spec
batches for these families had reached me by 20:50Z; `idStatus` field says
so. S-APR/R-APR remain final. NOT reconciled (nothing to reconcile against);
needs a pass when analyst batches land.

## Findings (reported, not worked around)
- F-001 confirmed-by-reproduction (observed): MEMBER bulkClose mutates
  PENDING_APPROVAL→DONE, 302. Labelled characterization; intent undecided.
- F-002 confirmed-by-reproduction (observed): id=abc / missing id → 500.
- NEW CANDIDATE 1: GET /tasks?assignee=abc → 500 (unhandled
  NumberFormatException, TaskListServlet.java:31) — same robustness shape as
  F-002, on the list route.
- NEW CANDIDATE 2: TaskDao.update() SQL has no project_id column — editing a
  task silently keeps the old project although the form offers a project
  picker (observed: posted projectId=2, persisted 1).
- No infrastructure faults this run.

## Remaining uncovered operations
/projects/delete (linked in JSP, NO servlet mapping — likely 404; untested),
styles.css exemption, description-field XSS surface in JSP lists (observed
raw EL output; not yet a test), DB-constraint edge cases (family 8:
over-long strings, FK violations), PUT/DELETE on non-API routes, session
fixation/timeout semantics. ~51 missed lines concentrate in
BootstrapListener/Database and error branches.
