# d003 reply — mapping reconciliation (rework attempt 2)

**Commit:** f63a5e8 on cs-sandbox/test-engineer-7aac6234.task-tracker-02-7aac6234
**Suite:** `python3 harness/runtime.py test` → Ran 53 tests, OK (53/53, unchanged — name/docstring ID text only; no logic, assertions, or inherited tests touched). Frozen app verified eee7830.

## Mapping rows reconciled: 44 (all placeholders resolved; zero left as placeholders)

- AUTH 5: S-AUT-001→S-AUTH-006 (logout), S-AUT-002/003→S-AUTH-004 (anonymous GET/POST), S-AUT-004→S-AUTH-005 (api exempt), S-AUT-005→S-AUTH-007 (root/index)
- PROJ 5: S-PRJ-001..004→S-PROJ-001..004; S-PRJ-005 (archive+rearchive)→S-PROJ-005+S-PROJ-006
- FORM 6: S-FRM-001..005→S-FORM-001..005; S-FRM-006 (nullable round trip incl. project-not-persisted characterization)→S-FORM-006+S-FORM-003
- LIFE 11: start/submit/reopen success→S-LIFE-001/002/003; three wrong-state conflicts→S-LIFE-004; 404/403 ordering pair→S-LIFE-005; unknown-action→S-LIFE-006; malformed+missing id 500s→S-LIFE-007 (F-002)
- BULK 3 (renumbered out of S-LIF per dispatch): member-bulkclose-pending mutation→S-BULK-003 (F-001 role dim); mixed-states→S-BULK-002 (F-001 state dim); no-ids noop→S-BULK-001
- QUERY 9: S-QRY-001..006→S-QUERY-001..006 (three assignee legs all under S-QUERY-003)
- API 5: S-API-001/002/005 unchanged; assignee-param-ignored→S-API-002 (closest match, note recorded in mapping.json — no dedicated spec exists); escaping+nulls→S-API-003+S-API-004
- APR 5: unchanged (final already)

Rules reconciled alongside specs in mapping.json (R-AUTH/PROJ/FORM/LIFE/BULK/QUERY/API final IDs). Method-name prefixes and docstring IDs rewritten in all six test modules.

## Tests without a spec match
None. Every test maps to at least one final spec; the only approximation is test_s_api_002_assignee_param_ignored → S-API-002 (marked with `note` in mapping.json per instructions).

## Caveat (fast-beats-perfect)
Docstring R-* citations were rewritten with per-file global maps; mapping.json carries the exact curated rule lists. A couple of docstring R- citations (e.g. logout test citing R-AUTH-004 alongside R-AUTH-007) are approximate — names, S-IDs, and mapping.json are exact.

## Files
- tests/test_auth_session.py, test_projects.py, test_task_form.py, test_lifecycle_bulk.py, test_filters_sorts.py, test_api_json.py (ID text only)
- reports/tests/mapping.json (idStatus=FINAL, dispatch=d003)
