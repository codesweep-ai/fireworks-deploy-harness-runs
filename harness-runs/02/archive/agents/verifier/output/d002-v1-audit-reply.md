# d002 V1 (run 02) — verifier baseline audit reply

Commit on my branch: `5737f81` — `evidence/verifier/d002-run02/` (commands.md + verbatim logs).

## V1 results (all on my own seat, runtime at 127.0.0.1:8080)

1. **Reset + test**: `runtime.py reset` (4 tasks) then `runtime.py test` → **9/9 OK, exit 0**
   (5 approval-slice S-APR-001..005 + 4 skeleton). Matches baseline.json.
2. **Coverage reproduces baseline EXACTLY** — LINE 233/500, BRANCH 75/188, INSTRUCTION 1021/2155,
   METHOD 64/108, CLASS 12/17, COMPLEXITY 85/119. My regenerated summary.json is byte-identical
   to history/01's. Denominator = frozen app (17 classes / 500 lines); Java-only scope honestly
   stated — not equated with JSP/SQL/CSS/rule coverage.
3. **Source integrity**: `harness/check-frozen-app.py ~/app` → exit 0,
   `eee783057ec28d28fb36ec05484de9b4b40ceb9c (41 files)`; runtime re-verifies internally per command.
4. **Gap audit** — run 01 confirmed F-001 (bulkClose no role/state guard) and F-002 (malformed
   id → 500) were never reproduced on any seat; only the approval slice was verified. Run 02 still
   owes (all to be produced by me post-integration, item V2):
   - F-001 reproduction: dev/dev POST bulkClose ids=2 + before/after status oracle (real HTTP).
   - F-002 reproduction: mgr/mgr POST approve id=abc → expect 500 (real HTTP).
   - All 8 scope families with real HTTP: pre-integration suite covers only family 1 partially
     (no logout, no /api/* or *.css exemption checks) and family 4 partially (no start/submit/
     reopen/not-found/unknown-action); families 2,3,5,6,7,8 have zero executed tests.
   - Mapping verification: every mapping row must point at tests that actually ran in my logs.
   - Double-run isolation: whole integrated suite TWICE with reset between.
   - Coverage delta vs the baseline reproduced above.

Full detail: `evidence/verifier/d002-run02/commands.md` on my branch.

## State

Branch differs from base by the V1 audit commit. No tests/rules/specs/app sources touched.
Runtime left UP. Ready for V2: awaiting the integration commit on `refs/campaign/orchestrator`.
