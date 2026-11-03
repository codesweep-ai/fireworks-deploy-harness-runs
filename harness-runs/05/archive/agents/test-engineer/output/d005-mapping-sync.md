# d005 / e04 — final mapping sync (attempt 1, done)

Dispatch: link the last ear:null tests to analyst batch 2b09f92 specs, fold the
corrected S-QUERY-003/005 observables into conflict texts, rebuild mapping, run
the suite once, commit.

## What was done

1. Merged the full integrated tree into my branch (fast-forward):
   `refs/campaign/orchestrator` = `065f18d59875a58a777e212fd12d31d61c693e1e`
   (includes analyst batch `2b09f92`). The named ref was already present in my
   clone; the `git fetch` of it from `origin` still fails (origin publishes only
   `main`) — the local ref is what made this work.
2. `tests/build_mapping.py` reconciliation table updated:
   - `test_s_api_014` (api ignores POST) → **S-API-008** (was ear:null)
   - `test_s_act_020` (action rejects GET) → **S-LIFE-010** (was ear:null)
   - `test_s_form_024` (edit nonexistent id creates nothing) → **S-FORM-012** (was ear:null)
   - `test_s_form_009` (create-trims-title) → +**S-FORM-011** (secondary)
   - `test_s_list_024` (titles-unescaped) → +**S-QUERY-009** (secondary)
   - `test_s_api_012` (api HTML-passthrough) → +**S-API-007** (secondary)
   - `ACCEPTED_IDS` extended for the new specs (API-001..008, FORM-001..012,
     LIFE-001..010, QUERY-001..009).
   - S-QUERY-003 / S-QUERY-005 conflict reasons rewritten: they now quote the
     analyst's corrected observables (`Rows 3,2,1` / `Row order 2,1,3,4` at
     2b09f92, matching what the executed tests assert) and mark both conflicts
     RESOLVED instead of quoting the superseded enumerations as current.
   - `reconciled.commit` → 065f18d; added `reconciled.d005` note; docstring
     updated (d004 text marked accepted-by-orchestrator, d005 summary added).
3. Rebuilt `reports/tests/mapping.json`.

## Results

- `python3 tests/build_mapping.py` → **mapped 137 tests (0 unmapped)**;
  2 conflicts (both now recording the 2b09f92 resolution); **2 without accepted
  spec** — see discrepancy below.
- `python3 harness/runtime.py test` → **Ran 137 tests ... OK** (27.2s).

## Discrepancy (dispatch expected 0 ear:null; 2 remain)

Before this dispatch there were **5** ear:null tests, not covered by the
dispatch's link list beyond three:

- `test_auth.LoginTests.test_s_auth_014_login_session_is_scoped_to_one_client`
  — cookie-jar session isolation; no accepted spec covers cross-client scoping
  (S-AUTH-003/004 cover session creation/absence only).
- `test_projects.ProjectListTests.test_s_proj_002_projects_page_marks_archived_project_without_archive_button`
  — archive-button render detail (R-PROJ-006); accepted S-PROJ-007 covers only
  the dangling Delete-link 404.

The 2b09f92 batch adds 7 specs (S-API-006/007/008, S-FORM-011/012, S-LIFE-010,
S-QUERY-009); none covers these two behaviors. I did not fabricate links —
they remain `ear:null`, listed in `unmappedAccepted`.

Observation (not acted on, outside dispatch scope): new spec **S-API-006**
names `test_s_api_004_api_default_order_is_updated_desc` as its executable test;
that test currently maps to S-API-002 (same default-order statement). A
secondary link to S-API-006 could be added if the orchestrator wants it.

## Commit

- `6dd956f` on `cs-sandbox/test-engineer-36f5ca10.task-tracker-05-36f5ca10`
  (branch fast-forwarded to 065f18d first; mapping + table only, no test or app
  changes).

## Commands

- `git merge --no-edit refs/campaign/orchestrator` (fast-forward 56f9601 → 065f18d)
- `python3 tests/build_mapping.py`
- `python3 harness/runtime.py start && python3 harness/runtime.py test`
- `git commit` (6dd956f)
