# Test-engineer findings — dispatch d002 (run 05, fresh whole-app suite)

App commit under test: `eee783057ec28d28fb36ec05484de9b4b40ceb9c` (frozen, verified by
harness integrity checks before and after every run).

## Defect candidates (observed via real HTTP against the frozen app; source-confirmed)

1. **Dangling destructive affordance — Delete link to an unmapped route**
   `src/main/webapp/WEB-INF/jsp/projects.jsp` renders
   `<a href="${contextPath}/projects/delete?id=${p.id}">Delete</a>` for every project,
   but no servlet is mapped to `/projects/delete` in `web.xml` and `ProjectServlet`
   implements no delete. Following the link gives HTTP 404. Test:
   `test_projects.ProjectArchiveTests.test_s_proj_014_projects_delete_link_targets_an_unmapped_route`
   (S-PROJ-014, characterization). Risk: users will click it; on a non-frozen build a
   future mapping could make a GET link destructive (no CSRF token either).

2. **Stored XSS via task title — list page renders titles unescaped**
   `tasks.jsp` (and `task-form.jsp`) output `${t.title}` with plain EL, which does not
   HTML-escape. A task titled `<b>bold</b>` reaches the browser as raw HTML (verified).
   The hand-rolled JSON escaper in `ApiTaskServlet.esc()` likewise only escapes `\` and
   `"` and passes `<`, `>`, `&` through (S-API-012, S-LIST-024, both characterization).
   Any page rendering a user-controlled title without `<c:out>`/`escapeXml` is
   injectable by anyone who can create or edit a task.

3. **Silent fallback for unknown sort tokens (policy ambiguity)**
   `TaskDao.orderClause` maps any non-whitelisted `sort` token to the default
   `updated_at DESC` instead of rejecting it (S-LIST-009, characterization). Injection
   is correctly prevented by the whitelist, but an invalid token is silently accepted.
   Flagging as an ambiguity: should unknown sorts 400 or fall back? Observed behavior
   is documented as characterization, not as an intended requirement.

4. **500 on malformed numeric inputs (uncaught NumberFormatException / IllegalArgumentException)**
   Several handlers parse request parameters with `Long.parseLong` / `Long.valueOf` /
   `java.sql.Date.valueOf` without catching conversion errors, yielding raw HTTP 500:
   - `GET /tasks?assignee=abc` (TaskListServlet L31) — S-LIST-023
   - `GET|POST /tasks/form` with `id=abc`, `projectId=abc`, `dueDate=not-a-date`
     (TaskFormServlet L30/L50/L62) — S-FORM-004, S-FORM-014
   - `POST /tasks/action` with `id=abc`, missing `id`, or `ids` containing `abc`
     (TaskActionServlet L31, L83) — S-ACT-018, S-ACT-019, S-BULK-008
   - `POST /tasks/form` with nonexistent `assigneeId` → FK violation surfaces as 500
     (S-FORM-016)
   Characterization only (no intended requirement states otherwise), but each is a
   user-reachable 500 with a stack trace in the logs — defect candidates for input
   validation rules.

5. **GET /tasks/form?id=999 renders a blank "New task" form** instead of 404
   (`tasks.findById` returns null and the JSP silently falls back to the blank form;
   S-FORM-003, characterization). Same class of issue as the Delete link: silently
   wrong UI state instead of an explicit error.

## Unreachable / remaining uncovered branches (JaCoCo, all production Java)

Final counters (reports/coverage/summary.json): INSTRUCTION 1993/2155, BRANCH 168/188,
LINE 452/500, METHOD 103/108, CLASS 17/17. Remaining gaps are:

- **SQLException catch blocks** in `TaskDao`, `ProjectDao`, `PersonDao`, `Database`
  (and `Database` connection-failure paths) — unreachable without inducing a DB fault;
  the read-only `oracle_ro` connection path is exercised, error branches are not.
- **`TaskDao.findByProject` and `ProjectDao.findById`** — dead from the HTTP surface:
  no servlet calls either (projects page uses `findAll`/`exists`; form uses `findById`
  of Task only). Candidates for dead-code findings.
- **Defensive guards duplicated at DAO level**: `TaskDao.bulkClose` empty/null guard and
  `updated_at` null branch (`map`, L236) — pre-empted by servlet-level checks and NOT
  NULL constraints respectively.
- **Servlet lifecycle / filter lifecycle** (`Filter.init`, `Filter.destroy`,
  `BootstrapListener` internals) — container-driven, not reachable by HTTP tests.
- **`TaskActionServlet.requireApprover` user==null half and L66/L78 committed-response
  halves** — AuthFilter guarantees a user before the servlet runs; error paths already
  committed the response. Unreachable via HTTP.
- **`ApiTaskServlet.esc(null)`** — only called on NOT NULL columns; dueDate null is
  handled by the ternary before `esc`.
- **`LogoutServlet` session==null half** — AuthFilter redirects sessionless requests
  before the servlet can run.

## Policy ambiguities sent to the orchestrator

- Unknown sort token: silent fallback (observed) vs explicit 400 (see item 3).
- Failed login issues a container JSESSIONID cookie (JSP session autocreation) — the
  session authenticates nothing (asserted), but "no session on failed login" is not
  literally true; EARS rules should speak of "no authenticated session".
- `POST /tasks/form` with `id=999` (nonexistent) runs an UPDATE matching no rows and
  answers 302 as if saved (S-FORM-024) — should this be a 404?

## Characterization vs intended requirements

Every test carries `kind` in `reports/tests/mapping.json`:
`intended-requirement` (matches the rule the source implements) vs
`characterization` (observed behavior labelled separately — the 500s, silent fallbacks,
unescaped HTML, blank-form-on-missing-id, ghost-update cases above).
