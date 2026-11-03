# d002 run summary — rule-analyst, campaign task-tracker-05

Dispatch a01 (d002): fresh whole-app discovery at frozen app commit eee783057ec28d28fb36ec05484de9b4b40ceb9c.

## Status: COMPLETE (all three batches delivered)

## Branch commits
- 13abff6 Batch 1: rules/app-inventory.json (full inventory, all 8 families) + rules/AUTH.md, specs/AUTH.md, rules/PROJ.md, specs/PROJ.md
- 514d4d4 Batch 2: rules/FORM.md, specs/FORM.md, rules/LIFE.md, specs/LIFE.md
- 6f55e3f Batch 3: rules/BULK.md, QUERY, API, DATA + specs for all four + findings/ledger.json

Branch differs from base (empty start; all work committed on cs-sandbox/rule-analyst-36f5ca10.task-tracker-05-36f5ca10).

## What was produced
- 43 rules (R-<FAMILY>-NNN: AUTH 9, PROJ 7, FORM 9, LIFE 9, BULK 5, QUERY 6, API 5, DATA 8 — all with file:line citations, actors/preconditions, trigger, effects/refusal, observed-vs-intended, linked EARS IDs).
- 43 specs (S-<FAMILY>-NNN: EARS statement, linked rules, fixtures, observable assertions, executable test IDs all marked "planned" — engineer reconciles).
- rules/app-inventory.json: every route/method/action/filter/sort token, validation/default/null handling, session/logout, project create/archive, task lifecycle + guards, bulkClose, API readback/escaping/nulls, schema constraints, DAO mappings, JSP-observable behavior, technical Java paths, no-executable-counterpart files.
- findings/ledger.json: 8 defect candidates, each with observed-vs-intended and a reproducible case:
  - D-001 dead Delete link -> 404 (projects.jsp:152, no mapping)
  - D-002 bulkClose approval-policy + role bypass (MEMBER/VIEWER can close PENDING_APPROVAL tasks) — high
  - D-003 ARCHIVED projects accept tasks; unknown form id renders as "New task"
  - D-004 JSP EL not HTML-escaped -> stored-XSS candidate — high
  - D-005 unvalidated numeric params -> HTTP 500 (7 sites cited)
  - D-006 bulkClose silent feedback (skips DONE, ignores affected count)
  - D-007 plaintext password compare; /api/* unauthenticated by design (noted)
  - D-008 API JSON esc() misses control chars -> invalid JSON payload

## Not invented
No execution evidence is claimed; all executable test IDs are "planned" for the engineer/verifier. AUTH-009 explicitly records that VIEWER has no restrictions beyond approve/reject per source (no inference from names).

## Unexamined areas
None — all 18 javaSources, all viewSources, web.xml, and db/*.sql in scope.json were read; scope families fully covered. `evidence/analyst/` left empty intentionally (execution evidence comes via orchestrator from engineer/verifier).

## Durable copies
findings/ledger.json copied to ~/.local/share/cs-campaign/output/ledger-d002.json; this summary to ~/.local/share/cs-campaign/output/summary-d002.md.
