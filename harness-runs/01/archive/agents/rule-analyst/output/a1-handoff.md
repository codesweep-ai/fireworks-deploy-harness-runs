# A1 handoff summary — rule-analyst — task-tracker-01

Commit `d7e6bdf` on branch `cs-sandbox/rule-analyst-077665b6.task-tracker-01-077665b6` (base 679cd02).
All findings verified against frozen ~/app @ eee783057ec28d28fb36ec05484de9b4b40ceb9c.

## Files written (in ~/outputs)
- `rules/R-APR.md` — rules R-APR-001..005 + gaps G-APR-001..003
- `specs/S-APR.md` — EARS specs S-APR-001..005
- `evidence/analyst/fixtures.md` — seeded users/tasks, exact requests, expected responses, oracle expectations

## Key results
- Approver set decided solely by `Role.canApprove()` (Role.java:14-16: ADMIN, MANAGER), checked twice: filter (AuthFilter.java:40-47, fires first, 403 "Only managers or admins…") and servlet (`requireApprover`, TaskActionServlet.java:91-98, 403 "Approval requires a manager or admin"). Same predicate → no outcome disagreement; messages differ and the servlet's 403 is unreachable on the normal route. Ordering: non-approver + bad id → 403 (not 404); approver + bad id → 404.
- Starting-state guard: servlet-only `guard()` (TaskActionServlet.java:48-57, 100-109) — approve/reject require status PENDING_APPROVAL, else 409 "Cannot <action> a task in status <cur> (expected PENDING_APPROVAL)". Neither TaskDao.updateStatus (115-125) nor schema.sql:33 CHECK enforces transitions; single point of enforcement.
- Transitions: approve → DONE, reject → IN_PROGRESS, then 302 to /tasks.

## Fixtures (seed.sql)
- Users: admin/ADMIN, mgr/MANAGER (can approve); dev/MEMBER, view/VIEWER (cannot). Passwords = usernames.
- Tasks: 1 OPEN, 2 IN_PROGRESS, 3 PENDING_APPROVAL, 4 DONE.
- (a) mgr `POST /tasks/action action=approve id=3` → 302, 3→DONE (reject → IN_PROGRESS).
- (b) dev `action=approve id=3` → 403 filter message, 3 unchanged.
- (c) mgr `action=approve id=1` (OPEN) → 409, 1 unchanged. Oracle: `SELECT id,status FROM task ORDER BY id`.

## Gaps / remaining ambiguities
- **G-APR-001 (notable):** `bulkClose` (TaskActionServlet.java:26-29,76-88; TaskDao:128-147) sets any non-DONE task to DONE with no canApprove and no status guard — MEMBER/VIEWER can reach an approve-equivalent state. Out of slice; needs a decision.
- G-APR-002: missing/malformed `id` → NumberFormatException → 500 (TaskActionServlet.java:31).
- /api/* exempt from AuthFilter but ApiTaskServlet is GET-only read-back — no approval bypass (verified).
- No unsupported spec case: all of (a)/(b)/(c) are source-supported.

No tests authored; no app files touched.
