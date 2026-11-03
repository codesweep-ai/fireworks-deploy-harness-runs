# d003 — execution-evidence integration (rule-analyst)

**Commit:** `260a80c` on `cs-sandbox/rule-analyst-36f5ca10.task-tracker-05-36f5ca10`
**Evidence source:** `refs/campaign/orchestrator` @ `2fdf5c069b86fdca1094544730e391e5b1c61d26` (engineer d002: 129/129 pass, mapping.json `unmapped: []`, coverage BRANCH 168/188, LINE 452/500, INSTRUCTION 1993/2155, METHOD 103/108, CLASS 17/17).

## 1. Inventory execution links (rules/app-inventory.json, v2)

- New top-level `executionEvidence` maps **all 63 accepted analyst S-IDs** to exact executed engineer test IDs (full `file.Class.test` names from mapping.json):
  - **51 specs executed** — result `pass (run 05, engineer d002, 129/129)`; 122 of the 129 tests link to at least one analyst spec.
  - **12 specs remain planned-only, gap named explicitly**: S-AUTH-003 (session-without-user-attr path), S-AUTH-009 (VIEWER executes mutations), S-AUTH-010 (header render), S-PROJ-008 (ARCHIVED-project acceptance), S-FORM-010 (column-width overflow), S-LIFE-009 (transition-button render), S-BULK-004 (D-002 bypass — engineer suite has no MEMBER-closes-PENDING case), S-BULK-005 (bulk form render), S-QUERY-008 (toolbar render), S-DATA-001 (startup/DB-wait), S-DATA-007 (DAO-level, not HTTP-reachable), S-DATA-008 (masthead/placeholders render).
  - **Partial-coverage notes** where only part of a compound spec executed: S-PROJ-006 (empty-string description), S-FORM-005 (URGENT CHECK-500), S-API-002 (assignee/sort-ignored clause), S-API-005 (newline-raw case), S-DATA-004 (only FK-500 executed), plus extension notes on S-BULK-002 / S-PROJ-001.
- Every one of the 8 route rows plus `auth`, `bulkClose`, `schema`, `daoMappings` now carries an `execution` block linking its S-IDs into `executionEvidence.specs`.
- **7 engineer tests have no matching analyst spec statement** — recorded under `engineerTestsBeyondAnalystSpecs`, not silently mapped: API default order (eng S-API-004), API ignores POST (S-API-014), create-trims-title (S-FORM-009), action rejects GET (S-ACT-020), titles unescaped (S-LIST-024 → D-004 evidence), HTML passthrough (S-API-012 → D-008 evidence), ghost update (S-FORM-024 → D-010 evidence).

## 2. Defect ledger (findings/ledger.json, v2)

- **Merged, not restated:** engineer items 1→D-001 (delete link 404), 2→D-004 (stored XSS), 4→D-005 (user-reachable 500s; scope extended to nonexistent-assigneeId FK-500); item 5 (GET /tasks/form?id=999 blank form) folded into D-003's second half. D-001/D-003/D-004/D-005 gained `engineerCrossRef`, `executedTests` (exact mapping.json names), `executionResult`.
- **New D-009** — unknown sort token silently falls back to default `updated_at DESC` (executed eng S-LIST-009, characterization). Notes the labeling split: analyst S-QUERY-006 states the fallback as intended; engineer labels it characterization. Repro: `GET /tasks?sort=BOGUS` → 200, default order.
- **New D-010** — POST /tasks/form edit with nonexistent `id=999` runs a zero-row UPDATE and answers 302 `saved=1` as if saved (ghost update; executed eng S-FORM-024). Repro included.
- Engineer's failed-login-JSESSIONID ambiguity was left in the engineer's findings (not duplicated into the ledger) — it was not among the dispatch's named candidates.

## 3. Spec-vs-observation mismatches (recorded in inventory `executionEvidence.specMismatches`)

- **ID-space collision** (dispatch's example): engineer provisional S-BULK-002 = "bulk close single selected open task" ≠ analyst S-BULK-002 = "bulk close selected ids; non-numeric → 500". Engineer numbering is provisional throughout (LIST≈QUERY, ACT≈LIFE; AUTH/PROJ/FORM/API numbers don't align 1:1). Specs untouched — engineer reconciles its mapping to analyst IDs.
- **Intendedness label difference** on S-QUERY-006 (see D-009).
- **No behavioral contradiction**: every executed observation agrees with analyst spec statements where both exist (including observed-mislabel specs S-FORM-002, S-PROJ-007).

## Unexamined / remaining

- Nothing re-derived beyond the dispatch scope; specs/*.md and rules/*.md untouched per instruction. Coverage remainder is the engineer's documented unreachable set (SQLException paths, dead DAO finders, container lifecycle, committed-response halves).
