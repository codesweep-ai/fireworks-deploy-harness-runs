# d004 — planned-only gap closure (dispatch e03, attempt 1)

Member: test-engineer · Branch commit: **56f96014fcb2c6d3b8c401ea4b2b12ac1d940345**
Suite: **137 tests, 0 failures, 0 errors** (was 129; +8 new) · `Ran 137 tests ... OK`

## 1. Infrastructure finding first: ce9c7cb was never delivered

The dispatch says the integrated tree (analyst inventory v2) is at
`refs/campaign/orchestrator` ce9c7cb and to
`git fetch origin 'refs/campaign/orchestrator:refs/campaign/orchestrator'`.

Observed (retried 6× over ~1 minute):

    fatal: couldn't find remote ref refs/campaign/orchestrator

- The origin (`/run/cs-sandbox-repo-1`) publishes only `HEAD` and
  `refs/heads/main` (8f713fe).
- The local `refs/campaign/orchestrator` in `~/outputs` still sits at
  **6f55e3f** (the d003 delivery); ce9c7cb exists nowhere locally
  (`git cat-file -t ce9c7cb` → not found).

So the 12-spec `executionEvidence` list from inventory v2 was **not readable**.
I proceeded using the dispatch's explicit ID list, verifying every accepted
spec's text in `specs/*.md` **at 6f55e3f** (all 8 IDs exist there and their
EARS statements match the dispatch's descriptions). If inventory v2 redefined
any of these specs at ce9c7cb, my mapping may need a follow-up reconciliation
once the ref is actually delivered.

## 2. New tests (8) and their accepted mappings

Provisional docstring tags are new file-local numbers; `ear` (accepted) comes
from the ACCEPTED table in `tests/build_mapping.py`.

| # | Test (file · class) | Accepted ID | Kind | Asserts (real HTTP + persistence readback) |
|---|---|---|---|---|
| 1 | `test_bulk_close.BulkCloseTests.test_s_bulk_010_member_bulk_close_bypasses_approval_on_pending_task` | **S-BULK-004** | **characterization** (defect D-002) | dev (MEMBER) POST approve id=3 → **403**, task 3 still PENDING_APPROVAL; then dev POST bulkClose ids=3 → **302**, DB readback task 3 = **DONE** (observed approval bypass; no 403 anywhere) |
| 2 | `test_bulk_close.BulkCloseTests.test_s_bulk_011_bulk_close_form_renders_checkbox_per_row_and_submit_button` | **S-BULK-005** | intended (render-only) | GET /tasks: hidden `action=bulkClose`, one `ids` checkbox per row (1–4, any order), "Close selected (bulk)" button |
| 3 | `test_projects.ProjectTaskStateTests.test_s_proj_015_task_create_and_edit_in_archived_project_are_accepted` | **S-PROJ-008** | **characterization** | POST /tasks/form title=Ghost&projectId=3 (ARCHIVED) → 302 `…&saved=1`; SQL join readback: row in 'Legacy Archive' (ARCHIVED); then POST edit of the same task → 302, title persisted (covers dispatch's "create/edit") |
| 4 | `test_auth.LoginTests.test_s_auth_023_authenticated_header_shows_full_name_role_and_logout` | **S-AUTH-010** | intended | login mgr; GET /tasks shows "Morgan Manager (MANAGER)" + `href="/logout"` |
| 5 | `test_task_actions.TaskLifecycleTests.test_s_act_021_list_shows_exactly_the_valid_transition_buttons_per_status` | **S-LIFE-009** | intended | GET /tasks rows: 1→[start], 2→[submit], 3→[approve, reject], 4→[reopen] — exactly, per row |
| 6 | `test_task_list.TaskFilterTests.test_s_list_027_toolbar_renders_six_sort_options_and_reflects_selection` | **S-QUERY-008** | intended | GET /tasks?sort=priority&status=OPEN: sort options exactly [updated, status, priority, due, title, id]; priority selected; OPEN selected |
| 7 | `test_task_list.TaskFilterTests.test_s_data_001_masthead_status_badge_and_null_placeholders_render` | **S-DATA-008** | intended | GET /tasks as dev: brand masthead, "Devon Member (MEMBER)", logout link; GET /tasks/form?id=4: `statusbadge status-DONE`, "(unassigned)", em-dash placeholder for null due date |
| 8 | `test_task_form.TaskCreateTests.test_s_form_025_title_at_column_width_boundary_persists_and_over_boundary_is_refused` | **S-FORM-010** | intended (boundary) | title=256 chars (VARCHAR(256) boundary) → 302, persists at full length, displays on detail page; title=257 → **500**, `tasks_table()` before==after (no row) |

Notes on 8: the dispatch phrased S-FORM-010 as "title at column-width boundary
persists and displays"; the accepted spec text (6f55e3f) specifies the
over-boundary refusal (500, nothing persisted). The test covers **both sides**
of the boundary, so the executed evidence matches the spec statement and the
dispatch's clause.

## 3. Skipped, as instructed (stay documented gaps)

- **S-AUTH-003** (session-without-user-attr), **S-DATA-001** (startup),
  **S-DATA-007** (DAO-level) — HTTP-unreachable.

## 4. Commands and results (from ~/outputs)

- `python3 harness/runtime.py start` → Ready (frozen app verified eee7830).
- `python3 tests/build_mapping.py` → "mapped 137 tests (0 unmapped);
  reconciled to accepted analyst ids: 2 conflicts, 5 without accepted spec".
- `python3 harness/runtime.py test` → **Ran 137 tests … OK** (reports/tests/result.json:
  testsRun 137, failures 0, errors 0, successful true).
- `python3 harness/runtime.py coverage` → counters **identical to d002**
  (INSTRUCTION 1993/2155, BRANCH 168/188, LINE 452/500, METHOD 103/108,
  CLASS 17/17): all 8 new tests exercise code paths (JSP render, the bulkClose
  UPDATE, form INSERT/UPDATE) already executed by the earlier suite.
- Two iterations were needed on my own test code before green (row-parsing
  regex case, checkbox order under updated-DESC list order); no test was
  weakened — assertions were corrected to the app's actual (and spec-legal)
  render order, and the "exactly the valid buttons" / "one checkbox per row"
  semantics were kept strict.

## 5. Ambiguities / findings for the orchestrator

1. **ce9c7cb not delivered** (§1) — fetch fails; remote publishes no campaign
   refs. All 8 mappings were made against spec texts at 6f55e3f + the dispatch.
2. **"7 ear:null tests" vs 5**: the dispatch says the analyst is adding specs
   for "your 7 ear:null tests"; the reconciled mapping has exactly **5** ear:null
   tests (unchanged from d003, kept null as instructed): api_014 POST-405,
   auth_014 session scoping, proj_002 archive-button render, act_020 GET-405,
   form_024 ghost-edit. Count discrepancy may come from counting the 2
   spec-conflict cases (S-QUERY-003 fixture rows, S-QUERY-005 priority order) —
   those are mapped, not null.
3. **D-002 confirmed again under stricter evidence**: S-BULK-004 executes the
   exact bypass sequence (approve 403 → bulkClose 302 → DONE readback) in one
   session on one task; `TaskDao.bulkClose` (frozen source) indeed has no role
   check and no status guard besides `status <> 'DONE'`.
4. **S-DATA-008 render contract detail**: the em-dash placeholder and the
   status badge exist only on the task **form/detail** page (task-form.jsp), not
   in the list rows (list due cell renders raw EL, empty for null — the D-004
   family). Test asserts each on the page where the spec's observables live.

## 6. Artifacts

- Branch: `cs-sandbox/test-engineer-36f5ca10.task-tracker-05-36f5ca10` at
  56f9601 (base b7176fd + this commit; 12 files changed).
- `reports/tests/mapping.json` — 137 entries, d004 note in `reconciled.d004`;
  `reports/tests/result.json`; `reports/coverage/*` refreshed.
