# d003 (e02) — mapping reconciliation to accepted analyst IDs — test-engineer

Branch: `cs-sandbox/test-engineer-36f5ca10.task-tracker-05-36f5ca10`
Commit for this dispatch: **b7176fd** (plus 6cecc55, which commits the previously-dirty
`reports/d002-summary.md` left uncommitted at d002 — the dirty-tree fix).
Accepted analyst source: `refs/campaign/orchestrator` = `6f55e3f269532d92aba3ebbfa073491ab564b708`
(rules/*.md, specs/*.md, rules/app-inventory.json, findings/ledger.json).

## 1. Dirty tree committed
- d002 branch head 2fdf5c0 was dirty: `reports/d002-summary.md` was untracked.
- 6cecc55 commits it. Tree now clean; nothing else was uncommitted.

## 2. Mapping reconciled
- All 129 tests in `reports/tests/mapping.json` now carry accepted analyst S-IDs in
  `ear` (string when one, list when several, null when none applies);
  `earProvisional` keeps the old docstring tag; `kind` and `statement` unchanged.
- Test docstrings were NOT renamed: the provisional and accepted numberings are not
  aligned (e.g. provisional S-ACT-003 = approve→DONE but accepted S-LIFE-003 = 404/500
  unknown-task; provisional S-LIST-003 = sort=status but accepted S-QUERY-003 =
  assignee filter), so a mechanical S-ACT-→S-LIFE- / S-LIST-→S-QUERY- rename would
  have produced wrong mappings. Instead the reconciliation table lives in
  `tests/build_mapping.py` (keyed by full test id — provisional ids are not unique,
  e.g. two S-PROJ-009s) and `python3 tests/build_mapping.py` regenerates the
  reconciled mapping.json deterministically, with drift guards: it fails if a
  discovered test has no reconciliation entry, if the table has stale entries, or if
  any accepted id is outside the spec universe (AUTH 1-10, PROJ 1-8, FORM 1-10,
  LIFE 1-9, BULK 1-5, QUERY 1-8, API 1-5, DATA 1-8).
- Note: the dispatch's family tallies (AUTH 9, PROJ 7, FORM 9, QUERY 6) undercount
  the actual accepted specs (AUTH 10, PROJ 8, FORM 10, QUERY 8); the spec files are
  authoritative and were used as-is.

## 3. Conflicts (observed behavior vs accepted spec — tests NOT changed)
Both are spec-text/seed discrepancies, not app defects; the passing tests assert the
app's actual output. Flagged per-entry in mapping.json (`conflicts`) and adjudication
is requested.

1. **S-QUERY-003** (test_s_list_018_assignee_filter_returns_that_users_tasks_only):
   S-QUERY-003's fixtures/observables say `assignee=3` matches "tasks 1,2" →
   "Rows 1,2 only", but `seed.sql:18-20` assigns tasks 1, 2 **and 3** to person 3.
   The app returns [3, 2, 1] (updated DESC), which the test asserts and which passes.
   The EARS statement itself ("only tasks assigned to that person") is not
   contradicted — only the enumerated seed row set.
2. **S-QUERY-005** (test_s_list_004_sort_priority_is_semantic_high_medium_low):
   S-QUERY-005's observable enumerates "Row order 1,2,3,4 (HIGH group newest-first)",
   internally inconsistent with its own EARS statement ("updated_at DESC within a
   group"): seed task 2 (12:00) is newer than task 1 (09:00) within the HIGH group,
   so the app returns and the test asserts [2, 1, 3, 4].

## 4. Tests with no matching accepted spec (ear=null, noted in mapping)
- `test_api...test_s_api_014_api_ignores_post_method` — container 405 on POST
  /api/tasks; accepted API specs describe GET behavior only.
- `test_auth...test_s_auth_014_login_session_is_scoped_to_one_client` — cross-client
  session isolation; accepted specs cover session creation/absence only.
- `test_projects...test_s_proj_002_projects_page_marks_archived_project_without_archive_button`
  — render detail of R-PROJ-006 whose accepted spec (S-PROJ-007) covers only the
  dangling Delete link.
- `test_task_actions...test_s_act_020_action_rejects_get_method` — container 405 for
  GET /tasks/action; LIFE specs cover POST semantics only.
- `test_task_form...test_s_form_024_edit_nonexistent_id_creates_nothing` — zero-row
  UPDATE on nonexistent edit id; S-FORM-009 covers existing ids only.

## 5. Coverage of accepted specs by this suite (informational)
Every accepted spec S-AUTH-001..010, S-PROJ-001..008, S-FORM-001..010, S-LIFE-001..009,
S-BULK-001..005, S-QUERY-001..008, S-API-001..005 and S-DATA-002..006/008 has at least
one mapped test. Uncovered by web tests (as expected/unchanged from d002 findings):
S-DATA-001 (deployment/startup), S-DATA-007 (DAO-level updateStatus, Java-only per
R-DATA-007), and S-PROJ-008's archived-project acceptance is covered only on the
create path (R-PROJ-007 noted as defect D-002/D-003 family in the ledger).

## 6. Commands and results (from ~/outputs)
- `git fetch origin 'refs/campaign/orchestrator:refs/campaign/orchestrator'` →
  "couldn't find remote ref" — the ref was already present locally at exactly
  6f55e3f269532d92aba3ebbfa073491ab564b708, so no fetch was needed.
- `python3 tests/build_mapping.py` → mapped 129 tests (0 unmapped); reconciled:
  2 conflicts, 5 without accepted spec.
- `python3 harness/runtime.py reset` → Database reset: 4 tasks.
- `python3 harness/runtime.py test` → **Ran 129 tests in 25.512s — OK** (0 failures,
  0 errors, exit 0). No test was modified, skipped, or weakened in this dispatch.

## 7. Files changed in b7176fd
- `tests/build_mapping.py` — reconciliation table (ACCEPTED) + drift guards + metadata.
- `reports/tests/mapping.json` — regenerated: accepted S-IDs, earProvisional, notes,
  conflicts, unmappedAccepted, reconciled provenance block.
