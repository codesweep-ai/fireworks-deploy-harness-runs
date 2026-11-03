# Run 02: whole-application business rules and test coverage

Continue run 01's accepted output commit 5f5751723b921c3ed9e005c4593bccd30de43c22.
Expand discovery and executable tests across the entire frozen task-tracker-demo, not just the
approval slice. Preserve its passing tests and stable rule/spec IDs. Increase meaningful coverage,
reproduce inherited suspected defects, and leave an explicit inventory of every remaining gap.

The app is `~/app`, runtime jars `~/runtime`, and writable output repo `~/outputs`. The app commit
is eee783057ec28d28fb36ec05484de9b4b40ceb9c. `_campaign/scope.json` lists the complete source and
route inventory; `_campaign/baseline.json` records run 01's nine passing tests and 233/500 Java
lines and 75/188 branches. Historical run 01 reports/evidence are under `history/01/`; they are
inputs, never evidence that run 02 executed anything. Current reports must name this run and commit.

## Whole-app scope and acceptance

1. Inventory every public operation and relevant branch across all source files, routes, views,
   and database constraints. `rules/app-inventory.json` maps each operation to source citations,
   rule/EARS IDs, executable test IDs, execution result, and gaps. Include login/logout and access
   exemptions; projects list/create/archive; task form display/create/edit/defaults/validation;
   start/submit/approve/reject/reopen and their guards; bulk close; task status/assignee filters and
   all sort tokens/defaults; JSON readback/filter/escaping/nulls; persistence constraints and mappings.
   Include root/JSP/static behavior and technical Java paths in the inventory, even when a branch
   is unreachable through HTTP. An unexecuted required operation is an unmet goal, not covered
   merely because a gap was documented. Do not shrink scope to obtain `campaign-met`.
2. Extract source-backed rules into `rules/*.md` and EARS test specifications into `specs/*.md`.
   Follow `_campaign/artifact-format.json`; keep existing IDs stable and explain corrections.
   Cover success, refusal, boundary, and no-mutation cases where applicable, without inventing
   authorization from role names or assuming all operations share the approval policy.
3. Extend the external real-HTTP Python unittest suite in `tests/test_*.py`. Keep tests fast and
   isolated with per-test database resets. Assert response status/content/redirects and persisted
   effects through the read-only oracle. Cover each operation family, not just a happy-path tour.
   Maintain `reports/tests/mapping.json` linking every behavior test to rule and EARS IDs.
   The inherited nine tests remain passing unless a source-supported test flaw is independently
   demonstrated; never silently delete, skip, or weaken an assertion to produce green results.
4. Reproduce inherited F-001/F-002 and adjudicate them with evidence. Reproduction establishes
   observed behavior; it does not alone establish intended policy. Preserve confirmed or suspected
   defects and unresolved intent in `findings/ledger.json`. A characterization of defective behavior
   must be labelled as such, with the unmet intended requirement recorded separately. No app fixes.
5. The verifier runs the integrated whole-app suite on its own local app/database, records the
   exact commit, verifies source/spec/test mappings and representative assertions across every
   family, and independently reproduces test results and coverage. Save commands, exit codes,
   logs, failures, and source-integrity checks. Run the suite twice to check isolation. A test that
   fails on a documented app defect stays visible; distinguish it from test/harness defects.
6. Collect production Java line and branch coverage against the same full denominator as run 01
   and report the delta and remaining uncovered files/branches. Account for all Java source files,
   including constants/interfaces with no executable counters. JSP, SQL, and CSS are checked
   behaviorally but are outside the Java coverage metric. Never present Java percentages as
   whole-app business-rule completeness. There is no arbitrary percentage target; push coverage
   as far as useful tests and the time budget permit. Explain unreachable/technical gaps explicitly.
7. Commit current-run `reports/run-summary.json`, `reports/attempts.json`, and `findings/ledger.json`.
   Include breadth by operation family, tests/results, coverage baseline/delta, adjudications,
   initial/integration commit IDs, achieved and unmet criteria, and actionable next-run work.
   Check frozen-app hashes before and after. All source/build/config changes to the app are forbidden.

## Prepared runtime and outputs

Each seat has the same pinned images and runtime jars, but its own running app and PostgreSQL.
From `~/outputs`, run:

```
python3 harness/runtime.py start
python3 harness/runtime.py reset
python3 harness/runtime.py sql 'SELECT id,status FROM task ORDER BY id'
python3 harness/runtime.py fixture 'UPDATE task SET status=... WHERE id=...'
python3 harness/runtime.py test
python3 harness/runtime.py coverage
```

HTTP base is `http://127.0.0.1:8080`. `sql` is read-only, `fixture` writes only this seat's test DB,
and `test` rejects an empty suite. Use inherited `tests/httpcase.py` for sessions and HTTP replies.
No platform installation is needed. Prefer the supplied harness; report infrastructure faults
promptly. The existing source-hash checker must continue to pass. Coverage is runtime instrumentation,
not a POM/source edit. New tests and generated evidence belong in the output repo only.

Stable artifact paths and IDs are required across runs. The host builds a simple HTML review from
the Markdown rules/EARS specs, Python tests, mappings, inventory, and findings after harvest.

## Coordination, attempts, and deadline

The 30-minute clock starts at campaign creation, including readback/provisioning. Work in parallel:
the analyst inventories the whole app and supplies rule/spec batches; the engineer builds family
test batches using source and accepted specs; the verifier starts its own runtime and audits the
inherited baseline while batches are prepared. The orchestrator integrates accepted batches and
pushes the exact integration commit for independent verification. Begin verification early enough
to finish; do not spend all time writing tests and leave verification unfunded.

Each stable work item permits two attempts only: initial submission plus one rework, at most
three minutes each. Track owner, item ID, dispatch ID, attempt number, timestamps, commit, result,
and verification evidence. Readbacks are not work attempts. Relabelling an exhausted item does
not reset its allowance. Distinct operation-family batches are legitimate separate items.

Stop new implementation dispatches no later than eight minutes before the deadline. Aim to finish
verification and commit/reply by three minutes before the deadline, when host closeout begins.
The host can collect partial evidence during that reserve and initiates stopping all nodes at
the 30-minute cutoff. Commit useful partial work and preserve unmet goals honestly; no extension
or new seat may be invented. Use `campaign-met` only if every acceptance criterion is met. Whole-app
coverage that remains incomplete must be stated as such, with a reasoned non-met outcome.
