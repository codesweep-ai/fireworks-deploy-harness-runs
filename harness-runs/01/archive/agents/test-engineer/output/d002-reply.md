# d002 reply — T1 harness validation + request mechanics + test skeleton

Commit: **b1a2ee6** ("d002: validate harness, record request mechanics, add HTTP test skeleton")
on branch cs-sandbox/test-engineer-077665b6.task-tracker-01-077665b6.

## Runtime status per subcommand (exact logs: evidence/test-engineer/d002-runtime.log)
- `python3 harness/runtime.py start` → exit 0 (~4s), "Ready: http://127.0.0.1:8080"
- `python3 harness/runtime.py reset` → exit 0, "Database reset: 4 tasks"
- `python3 harness/runtime.py sql 'SELECT id,status FROM task ORDER BY id'` → exit 0:
  1|OPEN, 2|IN_PROGRESS, 3|PENDING_APPROVAL, 4|DONE
- HTTP check: `GET /login` → 200, `GET /` → 200 (exit 0)
- `python3 harness/runtime.py test` → exit 1 before skeleton ("No executable tests
  exist yet; refusing an empty passing run" — expected); **exit 0 after skeleton, 4/4 pass**
- `python3 harness/runtime.py coverage` → exit 0; LINE 109/500 covered, writes
  reports/coverage/summary.json + report.xml
- Nothing broken; no platform installs needed.

## Request mechanics (full detail: evidence/test-engineer/d002-request-mechanics.md)
- Login: `POST /login`, form params `username`/`password`. Success → 302 to `/tasks`
  + JSESSIONID session cookie; failure → 200 login page with "Invalid credentials".
  Gotcha: failed login still sets JSESSIONID (JSP creates a session) — auth must be
  proven by reaching `/tasks` (200), not by cookie presence.
- AuthFilter on `/*`: only `/login`, `/api/*`, `*.css` public; rest redirect to /login.
- Task action: `POST /tasks/action`, params `action` + `id` (long); optional
  `return=detail`; `bulkClose` takes repeated `ids`. Approve requires
  PENDING_APPROVAL + approver role (checked in both AuthFilter and servlet).
  Statuses: 400 unknown action / 403 non-approver / 404 unknown id / 409 wrong
  status; success → 302 to /tasks.

## Seeded users/tasks to use
- Users (password == username): admin/ADMIN, mgr/MANAGER (both can approve),
  dev/MEMBER, view/VIEWER (cannot approve).
- Approval target: task id 3 "Migrate invoices", PENDING_APPROVAL (assignee dev,
  project Borealis). Other tasks: 1 OPEN, 2 IN_PROGRESS, 4 DONE.

## Skeleton location
- `tests/httpcase.py` — HttpCase base: cookie-jar session, no-redirect opener,
  `login()`, `task_action()`, 4xx/5xx returned as responses.
- `tests/test_skeleton.py` — 4 runnable smoke tests (login page, anon redirect,
  login success/failure). Discovery via `runtime.py test` (unittest discover) works.
- Ready for EARS-tagged rule tests to drop in on the next dispatch.

## Broken / blocked
Nothing. Awaiting accepted EARS/rule IDs from the rule-analyst dispatch.
