# Business Rules — task-tracker

At extraction: 34 confirmed rules (4 P0); later steps may add or correct rules below. Each citation was checked by a second agent that read the cited lines; 0 candidate rules were refuted and left out.

| ID | Name | Category | Priority | Source | Confidence |
|---|---|---|---|---|---|
| RULE-001 | Task timestamps shown to whole seconds; due dates as ISO dates | Calculation | P2 | `src/main/java/com/acme/tracker/dao/TaskDao.java:232-236` | High |
| RULE-002 | Any user of any role can be assigned a task | Validation | P1 | `src/main/java/com/acme/tracker/dao/PersonDao.java:31-44` | Medium |
| RULE-003 | Tasks may be created in archived projects | Validation | P1 | `src/main/java/com/acme/tracker/dao/ProjectDao.java:45-55` | Medium |
| RULE-004 | Due date is optional, must be an ISO date, and has no range check | Validation | P1 | `src/main/java/com/acme/tracker/dao/TaskDao.java:182-188` | Medium |
| RULE-005 | Project creation defaults | Validation | P1 | `src/main/java/com/acme/tracker/web/ProjectServlet.java:38-50` | High |
| RULE-006 | Order of checks for a single-task action | Validation | P1 | `src/main/java/com/acme/tracker/web/TaskActionServlet.java:31-36` | High |
| RULE-007 | Task save validation | Validation | P1 | `src/main/java/com/acme/tracker/web/TaskFormServlet.java:44-62` | High |
| RULE-008 | Allowed values for status, priority, role and project state | Validation | P1 | `src/main/resources/db/schema.sql:28-35` | High |
| RULE-009 | Task data domain constraints | Validation | P1 | `src/main/resources/db/schema.sql:28-42` | High |
| RULE-010 | Transition buttons offered depend only on current status, not on the user's role | Validation | P2 | `src/main/webapp/WEB-INF/jsp/tasks.jsp:56-70` | High |
| RULE-011 | Bulk close marks selected tasks DONE, bypassing the approval workflow | Lifecycle | P0 | `src/main/java/com/acme/tracker/dao/TaskDao.java:127-147` | Medium |
| RULE-012 | Task status lifecycle (guarded state machine) | Lifecycle | P0 | `src/main/java/com/acme/tracker/web/TaskActionServlet.java:39-65` | High |
| RULE-013 | Project lifecycle: create as ACTIVE, archive one way | Lifecycle | P1 | `src/main/java/com/acme/tracker/dao/ProjectDao.java:57-84` | Medium |
| RULE-014 | Project lifecycle: ACTIVE to ARCHIVED, forward only | Lifecycle | P1 | `src/main/java/com/acme/tracker/dao/ProjectDao.java:74-84` | Medium |
| RULE-015 | Every task change refreshes its last-updated timestamp | Lifecycle | P1 | `src/main/java/com/acme/tracker/dao/TaskDao.java:97-125` | High |
| RULE-016 | Status-change precondition: refuse unless the task is in the expected status (c… | Lifecycle | P1 | `src/main/java/com/acme/tracker/web/TaskActionServlet.java:100-109` | High |
| RULE-017 | New-task defaults and what an edit may change | Lifecycle | P1 | `src/main/java/com/acme/tracker/web/TaskFormServlet.java:55-72` | High |
| RULE-018 | New tasks start OPEN and editing never changes status | Lifecycle | P1 | `src/main/java/com/acme/tracker/web/TaskFormServlet.java:64-72` | High |
| RULE-019 | Archive offered only for active projects; Delete link has no backing function | Lifecycle | P1 | `src/main/webapp/WEB-INF/jsp/projects.jsp:24-32` | Medium |
| RULE-020 | Status guards exist only in the web layer; the data layer accepts any status ju… | Lifecycle | P1 | `src/test/java/com/acme/tracker/LifecycleTest.java:27-39` | Medium |
| RULE-021 | Signing out ends the session | Lifecycle | P2 | `src/main/java/com/acme/tracker/web/LogoutServlet.java:12-18` | Medium |
| RULE-022 | Only managers and admins may approve or reject tasks | Policy | P0 | `src/main/java/com/acme/tracker/model/Role.java:13-16` | High |
| RULE-023 | Login required for the application, with plaintext credential match | Policy | P0 | `src/main/java/com/acme/tracker/web/AuthFilter.java:28-38` | High |
| RULE-024 | Task list filtering by status and assignee | Policy | P1 | `src/main/java/com/acme/tracker/dao/TaskDao.java:40-56` | High |
| RULE-025 | Editing a task cannot move it to another project | Policy | P1 | `src/main/java/com/acme/tracker/dao/TaskDao.java:97-112` | Medium |
| RULE-026 | User roles and what VIEWER can actually do | Policy | P1 | `src/main/java/com/acme/tracker/model/Role.java:7-16` | Low |
| RULE-027 | Unauthenticated JSON read-back of tasks with optional status filter | Policy | P1 | `src/main/java/com/acme/tracker/web/ApiTaskServlet.java:22-49` | High |
| RULE-028 | Access policy: approve/reject is the only role-restricted action | Policy | P1 | `src/main/java/com/acme/tracker/web/AuthFilter.java:28-49` | Medium |
| RULE-029 | Sign-in by username and password | Policy | P1 | `src/main/java/com/acme/tracker/web/LoginServlet.java:23-35` | Medium |
| RULE-030 | Bulk close: empty selection is ignored and no role is required | Policy | P1 | `src/main/java/com/acme/tracker/web/TaskActionServlet.java:76-88` | High |
| RULE-031 | Read-only reporting access to all tables, including plaintext passwords | Policy | P1 | `src/main/resources/db/oracle-ro.sql:8-12` | Medium |
| RULE-032 | Initial data: one account per role and tasks seeded in every lifecycle state | Policy | P1 | `src/main/resources/db/seed.sql:1-21` | Medium |
| RULE-033 | Task list filtering and sort order | Policy | P2 | `src/main/java/com/acme/tracker/dao/TaskDao.java:149-172` | High |
| RULE-034 | Task list ordering options | Policy | P2 | `src/main/java/com/acme/tracker/dao/TaskDao.java:154-172` | High |

## Calculation

### RULE-001: Task timestamps shown to whole seconds; due dates as ISO dates
**Category:** Calculation
**Priority:** P2
**Source:** `src/main/java/com/acme/tracker/dao/TaskDao.java:232-236`
**Plain English:** A task's last-updated time is truncated (not rounded) to whole seconds, as yyyy-MM-dd HH:mm:ss. Its due date is a plain yyyy-MM-dd date with no time part.
**Specification:**
  Given A task whose updated_at in the database is 2026-07-17 14:23:05.987
  When  The task is read for the list, the detail page or the JSON API
  Then  updatedAt is "2026-07-17 14:23:05" (the first 19 characters of the timestamp text; the .987 is dropped, not rounded up to :06)
**Parameters:** Truncation length 19 characters. Due date format yyyy-MM-dd (java.sql.Date.toString).
**Edge cases handled:** A timestamp on an exact second ("...:05.0") becomes "...:05".; A null updated_at gives null, though the schema makes the column NOT NULL.; Sorting uses the full-precision database value, so two tasks showing the same second can still have a fixed order between them.
**Confidence:** High

## Validation

### RULE-002: Any user of any role can be assigned a task
**Category:** Validation
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/dao/PersonDao.java:31-44`
**Also cited:** src/main/webapp/WEB-INF/jsp/task-form.jsp:86-94
**Plain English:** The assignee list contains every person sorted by full name, whatever their role, and the server only checks that the assignee id is a known person (through the foreign key), so read-only VIEWERs can be given work.
**Specification:**
  Given Person 4 'Vic Viewer' has role VIEWER
  When  A task is saved with assigneeId=4
  Then  The task is saved as assigned to Vic Viewer; leaving assigneeId empty saves it as unassigned (shown as '(unassigned)')
**Parameters:** People list: SELECT ... FROM person ORDER BY full_name, with no role filter. Shown by task-form.jsp:88-93. The assignee is parsed at TaskFormServlet.java:60-61, and the FK fk_task_assignee is at schema.sql:41.
**Edge cases handled:** An assigneeId that does not exist fails the FK and returns a 500, not a friendly message; An assigneeId that is not a number throws NumberFormatException (500); A VIEWER assigned a task cannot be stopped from starting or submitting it, since only approve/reject check the role
**Confidence:** Medium — Should VIEWER-role users (and possibly ADMINs) be excluded from the assignee list, and should a bad assignee give a validation message rather than a server error? / Should read-only VIEWER accounts be assignable as task owners?

### RULE-003: Tasks may be created in archived projects
**Category:** Validation
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/dao/ProjectDao.java:45-55`
**Also cited:** src/main/java/com/acme/tracker/dao/ProjectDao.java:45-55; src/main/java/com/acme/tracker/dao/ProjectDao.java:45-55
**Plain English:** The only project check on a task save is that the project exists. Archived projects appear in the task form's project list and accept new tasks, and archiving a project does nothing to its tasks.
**Specification:**
  Given Project 'Legacy Archive' (id 3) has state ARCHIVED
  When  A user creates a task with projectId=3
  Then  The task is created as OPEN in the archived project. Existing tasks in archived projects can still be started, submitted, approved and reopened.
**Parameters:** exists() runs SELECT 1 FROM project WHERE id = ?, with no filter on state. The form's project list is ProjectDao.findAll(), which includes ARCHIVED projects, ordered by name (lines 19-31).
**Edge cases handled:** Archiving (ProjectServlet.java:33-37) does not close, freeze or reassign the project's tasks.; A non-numeric projectId fails with an unhandled NumberFormatException.
**Suspected defect:** 'Archived' has no effect on work in the project. Users can keep adding tasks to a project described as decommissioned. | Archiving has no business effect on tasks, and it is not authorized beyond being signed in. | Archiving a project does not freeze its tasks. New tasks can be created in an ARCHIVED project, and existing ones edited or moved there.
**Confidence:** Medium — Should archived projects be closed to new tasks and task status changes, or is archiving only a label? / When a project is archived, should its open tasks be closed, blocked, or hidden, and should new tasks be refused in archived projects? Who may archive a project: only its owner, managers/admins, or anyone? / Should creating or moving a task into an ARCHIVED project be blocked, and should archi…

### RULE-004: Due date is optional, must be an ISO date, and has no range check
**Category:** Validation
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/dao/TaskDao.java:182-188`
**Also cited:** src/main/java/com/acme/tracker/dao/TaskDao.java:182-188
**Plain English:** A task may have no due date. If one is given it must be a yyyy-MM-dd date. Any past or future date is accepted, and the system never works out whether a task is overdue.
**Specification:**
  Given A user saves a task with due date "2026-06-10", which is before today's date of 2026-11-02
  When  The task is created or edited
  Then  The due date 2026-06-10 is stored without warning. A blank due date stores no due date. A value such as "10/06/2026" fails parsing with an unhandled error (HTTP 500), not a form error message.
**Parameters:** Parser: java.sql.Date.valueOf (yyyy-[m]m-[d]d). Blank means NULL.
**Edge cases handled:** A blank field clears an existing due date when editing.; Single-digit months and days ("2026-6-5") are probably accepted.; Out-of-range days within 1-31 (for example "2026-02-30") probably roll over to a real date (2026-03-02) instead of being rejected.; No overdue flag, reminder or due-date check exists anywhere in the code.
**Suspected defect:** A due date in the wrong format causes a server error instead of a form validation message.
**Confidence:** Medium — Should the rewrite reject impossible dates (such as Feb 30) and past due dates with a user-facing message, or keep the legacy behavior of accepting any parseable date and failing hard on bad formats?

### RULE-005: Project creation defaults
**Category:** Validation
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/web/ProjectServlet.java:38-50`
**Also cited:** src/main/java/com/acme/tracker/web/ProjectServlet.java:38-50
**Plain English:** A new project needs a non-blank name (leading and trailing spaces are trimmed). It is created ACTIVE and owned by the user who created it.
**Specification:**
  Given signed-in user 'mgr' (id 2)
  When  they create a project named '  Zephyr  ' with no description
  Then  project 'Zephyr' is created in state ACTIVE with owner_id 2. A blank name is refused with HTTP 400 'Project name is required'.
**Parameters:** Initial state ACTIVE (ProjectDao.java:58, schema.sql:22). Owner = session user. Name max 128 characters (schema).
**Edge cases handled:** Any role, including VIEWER, can create projects.; Duplicate project names are allowed (no unique constraint).; The description is not trimmed.
**Confidence:** High

### RULE-006: Order of checks for a single-task action
**Category:** Validation
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/web/TaskActionServlet.java:31-36`
**Plain English:** A single-task action runs these checks in order: the user must be signed in, the task must exist (else 404), approve/reject need a manager or admin (else 403), the task must be in the expected status (else 409), and the action must be known (else 400). On success the user returns to the task list, or to the task detail page if return=detail.
**Specification:**
  Given A signed-in MEMBER
  When  They post action=approve for task id 999 that does not exist
  Then  AuthFilter blocks the request with 403 'Only managers or admins may approve or reject tasks' before any lookup. A MANAGER posting the same request gets 404 'No such task'.
**Parameters:** Existence check at TaskActionServlet.java:31-36, role check in AuthFilter.java:42-49 and again at TaskActionServlet.java:49/54, status guard 409 at :100-109, unknown action 400 at :62-64, redirect target at :66-73 ('/tasks' by default, '/tasks/form?id=<id>' when return=detail).
**Edge cases handled:** A missing or non-numeric id parameter throws NumberFormatException (500), not 400; An unknown action on an existing task gives 400 'Unknown action: <action>'; A missing action parameter is treated as '' and gives 400 'Unknown action: null'
**Confidence:** High

### RULE-007: Task save validation
**Category:** Validation
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/web/TaskFormServlet.java:44-62`
**Also cited:** src/main/java/com/acme/tracker/web/TaskFormServlet.java:38-75; src/main/java/com/acme/tracker/web/TaskFormServlet.java:44-53
**Plain English:** A task cannot be saved without a non-blank title and a project that exists. Priority defaults to MEDIUM and the assignee and due date are optional.
**Specification:**
  Given a user submits the task form with title '   ' and project 1
  When  the form is posted
  Then  nothing is saved and the form is shown again with 'Title is required'. With title 'Fix login' and project 999 (no such project) the error is 'A task must belong to an existing project'.
**Parameters:** Title trimmed; max 256 characters (schema). Default priority MEDIUM (line 58). Assignee empty -> unassigned (null). Due date ISO yyyy-MM-dd or empty -> null.
**Edge cases handled:** An ARCHIVED project passes the existence check.; A priority outside LOW/MEDIUM/HIGH is only caught by the database CHECK constraint (server error, not a friendly message).; A badly formed due date or a non-numeric projectId/assigneeId causes a server error.; A user who does not exist as assignee is only caught by the foreign key.
**Suspected defect:** Tasks can be created in or moved into ARCHIVED projects, because the existence check ignores project state.
**Confidence:** High

### RULE-008: Allowed values for status, priority, role and project state
**Category:** Validation
**Priority:** P1
**Source:** `src/main/resources/db/schema.sql:28-35`
**Also cited:** src/main/resources/db/schema.sql:9-42
**Plain English:** The database is the authority on allowed values: task status (OPEN, IN_PROGRESS, PENDING_APPROVAL, DONE), task priority (LOW, MEDIUM, HIGH), person role (ADMIN, MANAGER, MEMBER, VIEWER) and project state (ACTIVE, ARCHIVED).
**Specification:**
  Given any code path writes a task with status 'CANCELLED'
  When  the row is inserted or updated
  Then  the database rejects the write with a CHECK constraint violation.
**Parameters:** task.status default OPEN; task.priority default MEDIUM; project.state default ACTIVE (schema.sql:22-23); person.role CHECK (schema.sql:14-15). Status strings are also repeated in TaskStatus.java and Role.java.
**Edge cases handled:** The values are duplicated in Java constants and the JSP dropdowns, so changing one place will not update the others.; The list and API status filters accept any text without checking it; an unknown status just returns an empty list.
**Confidence:** High

### RULE-009: Task data domain constraints
**Category:** Validation
**Priority:** P1
**Source:** `src/main/resources/db/schema.sql:28-42`
**Plain English:** Every task must have a title of up to 256 characters, a status from the four lifecycle values, a priority of LOW/MEDIUM/HIGH, and a valid project. The assignee and due date are optional but must reference a real person and be a calendar date.
**Specification:**
  Given An insert with priority 'URGENT'
  When  The task is saved
  Then  The database rejects it with a CHECK violation, which the app shows as a 500 error
**Parameters:** title VARCHAR(256) NOT NULL; description VARCHAR(4000); status IN (OPEN, IN_PROGRESS, PENDING_APPROVAL, DONE) default OPEN; priority IN (LOW, MEDIUM, HIGH) default MEDIUM; project_id NOT NULL FK project; assignee_id nullable FK person; due_date DATE nullable. Person: username VARCHAR(64) unique; full_name VARCHAR(128); role CHECK. Project: name VARCHAR(128) NOT NULL; description VARCHAR(1024).
**Edge cases handled:** Project names are not required to be unique; There is no cascade on delete, so a person or project referenced by tasks cannot be deleted; No constraint stops a due date from being in the past
**Confidence:** High

### RULE-010: Transition buttons offered depend only on current status, not on the user's role
**Category:** Validation
**Priority:** P2
**Source:** `src/main/webapp/WEB-INF/jsp/tasks.jsp:56-70`
**Also cited:** src/main/webapp/WEB-INF/jsp/tasks.jsp:56-70; src/main/webapp/WEB-INF/jsp/tasks.jsp:56-69
**Plain English:** Each task shows only the actions valid from its status: OPEN shows Start, IN_PROGRESS shows Submit for approval, PENDING_APPROVAL shows Approve and Reject, DONE shows Reopen. Approve/Reject are shown to every role even though only managers and admins can use them.
**Specification:**
  Given A MEMBER views task 3 in PENDING_APPROVAL
  When  The task list or detail page is shown
  Then  Approve and Reject buttons are visible, and clicking either returns 403
**Parameters:** The same per-status logic is on the detail page (task-form.jsp:42-59), which posts return=detail.
**Edge cases handled:** A VIEWER sees Start, Submit and Reopen and can use them because they have no role check
**Confidence:** High

## Lifecycle

### RULE-011: Bulk close marks selected tasks DONE, bypassing the approval workflow
**Category:** Lifecycle
**Priority:** P0
**Source:** `src/main/java/com/acme/tracker/dao/TaskDao.java:127-147`
**Also cited:** src/main/java/com/acme/tracker/dao/TaskDao.java:127-147; src/main/java/com/acme/tracker/dao/TaskDao.java:128-147
**Plain English:** A user can select several tasks and close them at once; every selected task that is not already DONE is set straight to DONE regardless of its current status or the user's role.
**Specification:**
  Given A logged-in VIEWER selects task #3 (OPEN), #4 (PENDING_APPROVAL) and #5 (DONE)
  When  They submit action=bulkClose with ids=3,4,5
  Then  Tasks #3 and #4 become DONE with refreshed updated_at; #5 is unchanged (already DONE); the user is redirected to the task list. No role or status check is applied.
**Parameters:** SQL: UPDATE task SET status='DONE' WHERE status <> 'DONE' AND id IN (...). Entry point TaskActionServlet.java:76-88. Empty or missing ids -> silent redirect, no change.
**Edge cases handled:** Empty selection -> no-op redirect to /tasks; Already-DONE tasks are skipped so their updated_at is not touched; Non-existent ids are silently ignored; Non-numeric id -> NumberFormatException (500); AuthFilter's approver check only looks at action=approve/reject, so bulkClose is open to every role
**Suspected defect:** Bulk close lets any role (including MEMBER and VIEWER) take OPEN or PENDING_APPROVAL tasks straight to DONE, which gets around the rule that only managers and admins can approve. The class comment says 'each action checks the task's current status', but bulk close does not. | Bulk close goes around the approval workflow. Any role, including VIEWER, can take OPEN, IN_PROGRESS or PENDING_APPROVAL tasks straight to DONE without manager or admin approval. This contradicts the README rule that only ADMIN/MANAGER may approve. | Bulk close bypasses both the lifecycle guard and the approver-role check. Any logged-in user, VIEWER included, can move OPEN/IN_PROGRESS/PENDING_APPROVAL tasks straight to DONE, which defeats the manager-approval policy. The class comment (TaskActionServlet.java:13-17) says each action checks the expected status, but bulk close does not.
**Confidence:** Medium — Is bulk close meant to be an admin/manager-only override, and should it only apply to tasks in particular statuses (for example PENDING_APPROVAL)? / Is bulk close meant to let any user, including MEMBER and VIEWER, mark tasks DONE without approval? Or should it be limited to approvers, or to tasks that are already PENDING_APPROVAL? / Is bulk close meant as an approver-only administrative override…

### RULE-012: Task status lifecycle (guarded state machine)
**Category:** Lifecycle
**Priority:** P0
**Source:** `src/main/java/com/acme/tracker/web/TaskActionServlet.java:39-65`
**Also cited:** src/main/java/com/acme/tracker/web/TaskActionServlet.java:39-65; src/main/java/com/acme/tracker/web/TaskActionServlet.java:39-65
**Plain English:** A task moves OPEN -> IN_PROGRESS -> PENDING_APPROVAL -> DONE, can be rejected from PENDING_APPROVAL back to IN_PROGRESS, and reopened from DONE to OPEN. Each action is refused unless the task is currently in the one status that action expects.
**Specification:**
  Given Task #7 is in status IN_PROGRESS
  When  a user posts action=start (which expects OPEN)
  Then  the change is refused with HTTP 409 'Cannot start a task in status IN_PROGRESS (expected OPEN)' and the status stays IN_PROGRESS. Posting action=submit instead moves it to PENDING_APPROVAL and refreshes updated_at.
**Parameters:** Transitions: start OPEN->IN_PROGRESS; submit IN_PROGRESS->PENDING_APPROVAL; approve PENDING_APPROVAL->DONE; reject PENDING_APPROVAL->IN_PROGRESS; reopen DONE->OPEN. Guard: TaskActionServlet.java:101-109 (HTTP 409 on mismatch). Unknown action -> HTTP 400; unknown task id -> HTTP 404. After success, redirect to /tasks, or to the task detail page when return=detail.
**Edge cases handled:** There is no direct OPEN->DONE, IN_PROGRESS->DONE or PENDING_APPROVAL->OPEN action. The only way around this is bulk close (separate rule).; Reopen goes all the way back to OPEN, not IN_PROGRESS.; approve/reject check the user's role before the status, so a MEMBER approving a task that is not pending gets 403, not 409.; The check reads and then writes with no lock or conditional UPDATE (TaskDao.updateStatus has no 'AND status = expected'), so two concurrent actions can both pass the guard.; The UI only shows the buttons that are valid for the current status (tasks.jsp:57-69, task-form.jsp:46-58), but the server guard is what actually enforces it.; LifecycleTest.java:27-39 (testDirectOpenToDone) calls the DAO directly to go OPEN->DONE. The test bypasses the servlet guards, so it does not test the business rule.; A request with no id and an action other than bulkClose fails when it parses the id (HTTP 500), before the 400 check for an unknown action.
**Suspected defect:** The guard and the write are not atomic: TaskDao.updateStatus (TaskDao.java:115-125) writes the status without checking the current status, so a race can apply a transition from the wrong state. | The status precondition is not part of the UPDATE (no 'AND status = expected'), so concurrent requests can race past the guard; any code path that calls TaskDao.updateStatus directly bypasses the lifecycle entirely.
**Confidence:** High

### RULE-013: Project lifecycle: create as ACTIVE, archive one way
**Category:** Lifecycle
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/dao/ProjectDao.java:57-84`
**Plain English:** A new project needs a non-blank name, starts ACTIVE, and is owned by the user who created it. An ACTIVE project can be archived. Archived projects cannot be reactivated.
**Specification:**
  Given User 'mgr' (id 2) submits project name '  Zephyr  '
  When  The create form is posted, and later action=archive is posted for that project
  Then  Project 'Zephyr' is created ACTIVE with owner 2. After archiving, its state is ARCHIVED. Archiving it again changes nothing (0 rows)
**Parameters:** Initial state = 'ACTIVE'. Archive SQL: UPDATE project SET state='ARCHIVED' WHERE id=? AND state='ACTIVE'. States allowed by schema.sql:22-23 = ACTIVE, ARCHIVED. Name validation and owner assignment at ProjectServlet.java:31-51 (HTTP 400 'Project name is required').
**Edge cases handled:** There is no role check on create or archive: any logged-in user, VIEWER included, can do both; Archiving has no effect on the project's tasks, and new tasks can still be added to it; There is no un-archive path in the code; projects.jsp:32 links to /projects/delete, but no servlet is mapped for it in web.xml, so delete is not implemented; The project name is trimmed. The description is optional and not trimmed (DB max 1024)
**Confidence:** Medium — Who is allowed to create and archive projects (today anyone, including VIEWER)? Should archiving freeze the project's tasks, and is project deletion a required feature?

### RULE-014: Project lifecycle: ACTIVE to ARCHIVED, forward only
**Category:** Lifecycle
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/dao/ProjectDao.java:74-84`
**Also cited:** src/main/java/com/acme/tracker/dao/ProjectDao.java:74-84
**Plain English:** A project can be archived only while it is ACTIVE. Archiving is one-way: nothing restores a project, and archiving an already-archived project does nothing.
**Specification:**
  Given project Borealis (id 2) is ACTIVE
  When  any signed-in user posts action=archive for id 2 to /projects
  Then  its state becomes ARCHIVED. Repeating the request changes nothing and returns no error.
**Parameters:** UPDATE project SET state='ARCHIVED' WHERE id=? AND state='ACTIVE'. Servlet: ProjectServlet.java:33-37. The Archive button shows only for ACTIVE projects (projects.jsp:25-31).
**Edge cases handled:** No role check: MEMBER and VIEWER can archive any project, not only the owner.; Archiving does not change the project's tasks. They keep their statuses and stay fully actionable.; New tasks can still be created in an ARCHIVED project (TaskFormServlet only checks that the project exists).; projects.jsp:32 shows a 'Delete' link to /projects/delete, but web.xml maps no servlet to that path, so deletion is not a real capability.
**Suspected defect:** Archived projects still accept new tasks and task transitions, so 'archived' carries no business meaning beyond a label. Any user, including VIEWER, can archive any project.
**Confidence:** Medium — What should archiving a project prevent: new tasks, edits or transitions on its tasks? And who may archive (owner, MANAGER/ADMIN only)? / Who may archive a project (owner, managers, admins), and what should happen to open tasks in an archived project?

### RULE-015: Every task change refreshes its last-updated timestamp
**Category:** Lifecycle
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/dao/TaskDao.java:97-125`
**Also cited:** src/main/java/com/acme/tracker/dao/TaskDao.java:97-125
**Plain English:** Any edit or status change to a task sets its updated_at to the current database time, and that timestamp drives the default 'recently updated' ordering.
**Specification:**
  Given task #1 last updated 2026-07-15 09:00:00 and listed last by default
  When  a user starts task #1 at 2026-11-02 10:30:00
  Then  its updated_at becomes 2026-11-02 10:30:00 and it moves to the top of the default task list.
**Parameters:** updated_at = CURRENT_TIMESTAMP in update (line 99), updateStatus (line 118) and bulkClose (line 136). Insert uses the schema default CURRENT_TIMESTAMP (schema.sql:39). Shown truncated to whole seconds (TaskDao.java:234-236).
**Edge cases handled:** Bulk close does not refresh tasks that are already DONE (they are excluded by the WHERE clause).; A form save with no actual changes still refreshes the timestamp.
**Confidence:** High

### RULE-016: Status-change precondition: refuse unless the task is in the expected status (check-then-write, not atomic)
**Category:** Lifecycle
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/web/TaskActionServlet.java:100-109`
**Plain English:** Every single-task status action first reads the task's current status and refuses with a conflict if it is not the status that action requires; but the later write does not re-check the status, so two people acting at the same moment can both succeed.
**Specification:**
  Given Task #3 is PENDING_APPROVAL and a manager presses Approve while another manager presses Reject on the same page
  When  Both requests read the status PENDING_APPROVAL before either one writes
  Then  Both pass the guard. The task ends up DONE or IN_PROGRESS depending on which write lands last. A request that reads a different status gets HTTP 409 with the message 'Cannot <action> a task in status <current> (expected <expected>)'
**Parameters:** Rejection outcome: HTTP 409 CONFLICT. The response text names the current and expected status. Missing task id gives HTTP 404 'No such task' (TaskActionServlet.java:31-36). The write is TaskDao.updateStatus (TaskDao.java:115-125): UPDATE ... WHERE id = ? with no status condition
**Edge cases handled:** A request with an unknown or missing action gets HTTP 400 'Unknown action: <action>' (TaskActionServlet.java:62-64), and this check runs only after the task lookup; After a successful transition the user goes to the task list. With return=detail they go back to that task's detail page instead (TaskActionServlet.java:66-73); A non-numeric id throws NumberFormatException, which the user sees as a server error, not a validation message
**Suspected defect:** Time-of-check/time-of-use race: the guard and the UPDATE are separate steps and the UPDATE has no 'AND status = expected' clause. Two concurrent approve/reject (or approve/reopen) requests can both apply, and the final state depends on timing. The rewrite should make the transition a single conditional update and report failure when 0 rows change.
**Confidence:** High

### RULE-017: New-task defaults and what an edit may change
**Category:** Lifecycle
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/web/TaskFormServlet.java:55-72`
**Plain English:** A new task always starts OPEN with MEDIUM priority unless another priority is chosen, and assignee and due date are optional. Editing a task updates its title, description, priority, assignee and due date but never its status.
**Specification:**
  Given A user creates a task 'Write docs' in project 2 with no priority, no assignee and no due date
  When  The form is saved
  Then  The task is stored with status OPEN, priority MEDIUM, no assignee and no due date, and the user lands on its detail page (/tasks/form?id=N&saved=1). If they later edit it to priority HIGH while it is IN_PROGRESS, it stays IN_PROGRESS and updated_at is refreshed.
**Parameters:** Default status OPEN (TaskFormServlet.java:66); default priority 'MEDIUM' (line 58; also DB default schema.sql:34). Update SQL TaskDao.java:97-112 sets title, description, priority, assignee_id, due_date, updated_at=CURRENT_TIMESTAMP. Every status or field write refreshes updated_at.
**Edge cases handled:** Any submitted status value is ignored on create; it is always OPEN; An empty assignee means unassigned (NULL); On edit, project_id is validated but never written, so the project chosen on the form is silently ignored; Editing a DONE task is allowed and does not change its status
**Suspected defect:** On edit, the form checks the chosen project exists but TaskDao.update never writes project_id, so moving a task to another project silently does nothing.
**Confidence:** High

### RULE-018: New tasks start OPEN and editing never changes status
**Category:** Lifecycle
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/web/TaskFormServlet.java:64-72`
**Also cited:** src/main/java/com/acme/tracker/web/TaskFormServlet.java:64-72
**Plain English:** Every new task starts in status OPEN. Editing a task changes its title, description, priority, assignee and due date, but never its status. Status changes only happen through the transition actions.
**Specification:**
  Given Task #2 is IN_PROGRESS
  When  The user edits task 2, changing priority from HIGH to LOW and saving
  Then  Task 2 keeps status IN_PROGRESS, its priority becomes LOW, and updated_at is refreshed
**Parameters:** Initial status = OPEN (TaskStatus.OPEN). The update SQL at TaskDao.java:97-112 excludes the status and project_id columns.
**Edge cases handled:** Project reassignment: the form sends projectId, but TaskDao.update does not write project_id, so changing a task's project on edit is silently ignored; Editing a non-existent id updates 0 rows but still redirects with saved=1; Editing a DONE task is allowed (no lock on completed tasks)
**Suspected defect:** The edit form lets the user pick a different project and validates it, but the update SQL never persists project_id, so the change is silently lost. | The edit form lets the user pick a different project, but the update never saves it. A user can believe they moved a task to another project when they did not.
**Confidence:** High

### RULE-019: Archive offered only for active projects; Delete link has no backing function
**Category:** Lifecycle
**Priority:** P1
**Source:** `src/main/webapp/WEB-INF/jsp/projects.jsp:24-32`
**Also cited:** src/main/webapp/WEB-INF/jsp/projects.jsp:32; src/main/webapp/WEB-INF/jsp/projects.jsp:24-33
**Plain English:** The projects page shows Archive only for ACTIVE projects, but shows a Delete link for every project. That link goes to /projects/delete, which has no servlet mapping, so projects can never be deleted.
**Specification:**
  Given Project 1 'Apollo' is ACTIVE and project 3 'Legacy Archive' is ARCHIVED
  When  Any signed-in user views /projects
  Then  Apollo shows Archive and Delete, Legacy Archive shows only Delete, and clicking Delete returns 404
**Parameters:** The archive post is handled with no role or owner check at ProjectServlet.java:32-37. The servlet mappings in web.xml:71-78 cover only /projects.
**Edge cases handled:** A forged archive post for an already ARCHIVED or unknown id changes nothing (DAO filter) and still redirects without telling the user; Any role, VIEWER included, can archive any project, not only the owner
**Suspected defect:** The Delete link points to an endpoint that does not exist. Archive has no ownership or role check. | Dead UI action: the Delete link has no backing endpoint. | The UI offers a Delete action with no server-side handler.
**Confidence:** Medium — Is project deletion a required feature, and if so what happens to its tasks? Should archiving be limited to the project owner or managers/admins? / Should projects be deletable at all? If so, what happens to their tasks (block, cascade, or reassign) and which roles may delete? Or should the Delete link be removed in the rewrite? / Is project deletion a required feature that was never built, or sh…

### RULE-020: Status guards exist only in the web layer; the data layer accepts any status jump
**Category:** Lifecycle
**Priority:** P1
**Source:** `src/test/java/com/acme/tracker/LifecycleTest.java:27-39`
**Also cited:** src/test/java/com/acme/tracker/LifecycleTest.java:27-39; src/test/java/com/acme/tracker/LifecycleTest.java:27-39
**Plain English:** The raw status update writes any status to any task without checking where it starts from, and the test suite checks that a task can go straight from OPEN to DONE. The approval workflow is enforced only by the TaskActionServlet checks.
**Specification:**
  Given A new task 'Spike: caching layer' in status OPEN, priority LOW, project 1
  When  TaskDao.updateStatus(id, 'DONE') is called directly
  Then  The task is DONE with no IN_PROGRESS or PENDING_APPROVAL step and no approver check, and the test passes
**Parameters:** Raw write is TaskDao.updateStatus (TaskDao.java:114-125, 'Raw status write'). testNormalFlow (LifecycleTest.java:41-55) also drives OPEN, IN_PROGRESS, PENDING_APPROVAL, DONE without an approver identity.
**Edge cases handled:** Anything that calls the DAO directly (batch jobs, future APIs, tests) skips the 409 guard and the manager/admin rule; Bulk close is a second DAO path that skips the guards
**Suspected defect:** The test treats skipping approval as expected behaviour. A rewrite that moves the guards into the domain layer would fail testDirectOpenToDone. | The test suite codifies the approval bypass as expected behavior. A rewrite that uses these tests as its specification would keep a path to DONE that never goes through approval. | The regression baseline encodes an approval bypass as correct behavior, and the check-then-write guard lets concurrent approve/reject requests conflict.
**Confidence:** Medium — Is the direct OPEN to DONE path in testDirectOpenToDone a real allowed transition (e.g. for admins), or should the domain layer reject it? / Is moving a task from OPEN (or IN_PROGRESS) straight to DONE without manager approval ever allowed, for example for spikes or admin clean-up? If so, who may do it? Or should the rewrite enforce the approval workflow at the domain/data level and treat testDir…

### RULE-021: Signing out ends the session
**Category:** Lifecycle
**Priority:** P2
**Source:** `src/main/java/com/acme/tracker/web/LogoutServlet.java:12-18`
**Also cited:** src/main/java/com/acme/tracker/web/LogoutServlet.java:11-18
**Plain English:** Signing out ends the user's session and returns them to the login page. No explicit session expiry is set.
**Specification:**
  Given Logged-in user Morgan Manager
  When  Morgan opens /logout
  Then  The session is invalidated and Morgan is redirected to /login. Later pages need a new login.
**Parameters:** No <session-config> timeout in web.xml, so the servlet container's default applies.
**Edge cases handled:** Logout without a session still redirects to /login.
**Confidence:** Medium — What idle timeout should sessions have? The legacy app relies on the container default. / Is there a required idle-timeout or maximum session length for this internal tool, and must a role change take effect immediately for active sessions?

## Policy

### RULE-022: Only managers and admins may approve or reject tasks
**Category:** Policy
**Priority:** P0
**Source:** `src/main/java/com/acme/tracker/model/Role.java:13-16`
**Also cited:** src/main/java/com/acme/tracker/web/TaskActionServlet.java:90-98; src/main/java/com/acme/tracker/web/TaskActionServlet.java:90-98
**Plain English:** Approving or rejecting a task that is pending approval is restricted to users whose role is ADMIN or MANAGER. MEMBER and VIEWER users are refused.
**Specification:**
  Given User 'dev' with role MEMBER is logged in and task #3 is PENDING_APPROVAL
  When  dev posts action=approve for task 3
  Then  The request is refused with HTTP 403 and task 3 stays PENDING_APPROVAL. If user 'mgr' (MANAGER) does the same, task 3 becomes DONE
**Parameters:** Approver roles = {ADMIN, MANAGER}. The check is enforced twice: AuthFilter.java:40-47 (403 'Only managers or admins may approve or reject tasks') and TaskActionServlet.java:91-98 (403 'Approval requires a manager or admin').
**Edge cases handled:** Gated actions are only 'approve' and 'reject'. start, submit, reopen and bulkClose have no role gate; A missing session user is treated as not authorized (403) by TaskActionServlet. In practice AuthFilter redirects to /login first; The submitter can approve their own task if they are a MANAGER/ADMIN. There is no segregation-of-duties check; The approver role is read from the session snapshot taken at login, so a role change in the DB does not apply until the user logs in again
**Confidence:** High

### RULE-023: Login required for the application, with plaintext credential match
**Category:** Policy
**Priority:** P0
**Source:** `src/main/java/com/acme/tracker/web/AuthFilter.java:28-38`
**Also cited:** src/main/java/com/acme/tracker/web/AuthFilter.java:28-38
**Plain English:** Every page needs a logged-in user except the login page, stylesheets, and anything under /api/. A user logs in when the username and password exactly match a stored person record.
**Specification:**
  Given An anonymous visitor with no session
  When  They request /tasks
  Then  They are redirected to /login. After posting a matching username and password they get a session holding their Person (id, name, role) and are redirected to /tasks; a wrong pair shows 'Invalid credentials' on the login page.
**Parameters:** Exempt paths: /login, /api/*, *.css. Credential check: PersonDao.java:16-29 (SELECT ... WHERE username=? AND password=?, plaintext). Session attribute 'user'. Logout (LogoutServlet.java:12-18) invalidates the session and redirects to /login. Seed passwords: <credential - masked, see src/main/resources/db/seed.sql:2-6>.
**Edge cases handled:** /api/tasks (ApiTaskServlet.java:23-27) is reachable without login and returns every task, or those with ?status=X, as JSON; Any path ending in .css skips authentication; Passwords are stored and compared in plaintext; there is no lockout or attempt limit; Username match is exact (case-sensitive by DB default); usernames are unique (schema.sql:11)
**Suspected defect:** The /api/ exemption means the whole task list (titles, statuses, assignees, due dates) can be read without logging in. Passwords are stored in plaintext. | The /api/ exemption exposes every task (titles, assignees, statuses, due dates) to unauthenticated callers through ApiTaskServlet (ApiTaskServlet.java:22-49). Passwords are stored and compared in plaintext (PersonDao.java:17-18).
**Confidence:** High

### RULE-024: Task list filtering by status and assignee
**Category:** Policy
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/dao/TaskDao.java:40-56`
**Plain English:** The task list shows every task, optionally narrowed by status and/or assignee. When both filters are given, a task must match both.
**Specification:**
  Given Seed data with 4 tasks, 3 assigned to person 3 and one IN_PROGRESS
  When  /tasks?status=IN_PROGRESS&assignee=3 is requested
  Then  Only 'Wire up SSO' is returned
**Parameters:** Filters: t.status = ? (when non-empty), t.assignee_id = ? (when present). Combined with AND. Wiring at TaskListServlet.java:22-37. The API uses the status filter only (ApiTaskServlet.java:24-27).
**Edge cases handled:** An empty status means all statuses; There is no 'unassigned' filter: unassigned tasks can only be seen with no assignee filter; An unknown status value returns an empty list (no validation); A non-numeric assignee raises a 500; There is no visibility restriction: every logged-in user sees every task in every project, archived ones included
**Confidence:** High

### RULE-025: Editing a task cannot move it to another project
**Category:** Policy
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/dao/TaskDao.java:97-112`
**Plain English:** Editing a task can change its title, description, priority, assignee and due date, but not its project. A different project chosen on the edit form is silently ignored.
**Specification:**
  Given Task #1 belongs to project Apollo (id 1)
  When  A user edits task #1, selects project Borealis (id 2) and saves
  Then  The task stays in Apollo. The page still shows "Saved." because the UPDATE statement never writes project_id.
**Parameters:** Columns updated on edit: title, description, priority, assignee_id, due_date, updated_at. Not updated: project_id, status.
**Edge cases handled:** The edit form still requires a project that exists (TaskFormServlet.java:50-53), even though the value is then discarded.; Editing an id that does not exist updates 0 rows, but the user is still sent to the 'Saved.' page (TaskFormServlet.java:68-74).; An empty description saves as an empty string, not NULL.
**Suspected defect:** The edit form shows an editable project dropdown and reports success, but the project change is never saved.
**Confidence:** Medium — Should a task be movable between projects? The edit form offers a project picker, but the save logic ignores it. Is that an intended lock or a bug?

### RULE-026: User roles and what VIEWER can actually do
**Category:** Policy
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/model/Role.java:7-16`
**Plain English:** There are four user roles: ADMIN, MANAGER, MEMBER and VIEWER. The only permission that differs between them in code is approve/reject (ADMIN, MANAGER).
**Specification:**
  Given User 'view' has role VIEWER
  When  view creates a task, starts it, submits it, bulk-closes tasks, or archives a project
  Then  All of these succeed. Only approve/reject are refused (403)
**Parameters:** Roles: ADMIN, MANAGER, MEMBER, VIEWER (also a DB CHECK at schema.sql:14-15).
**Edge cases handled:** ADMIN and MANAGER have identical permissions in code; MEMBER and VIEWER have identical permissions in code; An unknown role string in the DB would make Role.valueOf throw during login or the people list (PersonDao.java:51)
**Suspected defect:** The role name VIEWER suggests read-only access, but nothing enforces it. Viewers can create, edit, transition, bulk-close tasks and archive projects.
**Confidence:** Low — What is VIEWER intended to be allowed to do? Should it be read-only, and should MEMBER be restricted (e.g., only start/submit tasks assigned to them)?

### RULE-027: Unauthenticated JSON read-back of tasks with optional status filter
**Category:** Policy
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/web/ApiTaskServlet.java:22-49`
**Also cited:** src/main/java/com/acme/tracker/web/ApiTaskServlet.java:23-49; src/main/java/com/acme/tracker/web/ApiTaskServlet.java:22-49
**Plain English:** GET /api/tasks returns every task as JSON with id, title, status, priority, assigneeId, projectId, dueDate and updatedAt (no description), newest-updated first. An optional status parameter limits the list to that exact status.
**Specification:**
  Given The seeded data and no session
  When  A client calls GET /api/tasks?status=DONE
  Then  It receives [{"id":4,"title":"Archive old data","status":"DONE","priority":"LOW","assigneeId":1,"projectId":2,"dueDate":null,"updatedAt":"2026-07-15 18:00:00"}]
**Parameters:** The auth exemption for /api/* is in AuthFilter.java:28-33. The filter uses an exact status match. Ordering is updated_at DESC (TaskDao.findAll at :22-24, or search with the default sort). Timestamps are cut to 'yyyy-MM-dd HH:mm:ss' (TaskDao.java:234-236).
**Edge cases handled:** status=done (lowercase) or an unknown status returns [] rather than an error; An empty status parameter returns all tasks; There is no assignee or project filter and no paging
**Suspected defect:** Anyone who can reach the endpoint can read the whole task list, including assignment and due dates, without signing in. | The hand-written JSON escaping does not escape control characters, so a task title with a line break breaks the JSON for every consumer.
**Confidence:** High

### RULE-028: Access policy: approve/reject is the only role-restricted action
**Category:** Policy
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/web/AuthFilter.java:28-49`
**Plain English:** Every page needs a signed-in user except the login page, stylesheets and the /api/ JSON feed. Apart from approve/reject, any signed-in user, including VIEWER, can create, edit, transition, bulk close and archive.
**Specification:**
  Given user 'view' with role VIEWER is signed in
  When  they post action=start on an OPEN task, or edit a task's title
  Then  the change goes through. Only action=approve or action=reject gets HTTP 403.
**Parameters:** Public paths: /login, /api/*, *.css. Unauthenticated users are redirected to /login.
**Edge cases handled:** GET /api/tasks returns every task (title, status, assignee, project, due date) with no authentication.; The VIEWER role is not read-only in practice, despite its name.
**Suspected defect:** VIEWER has the same write access as MEMBER, and the /api/tasks feed is unauthenticated.
**Confidence:** Medium — Should VIEWER be read-only (no create, edit, transitions, bulk close or archive)? Should MEMBER be allowed to reopen DONE tasks? Is /api/tasks meant to be public?

### RULE-029: Sign-in by username and password
**Category:** Policy
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/web/LoginServlet.java:23-35`
**Also cited:** (session creation, redirect to /tasks, 'Invalid credentials' error) together with src/main/java/com/acme/tracker/dao/PersonDao.java:16-29,46-53 (plaintext username/password match and role mapping)
**Plain English:** A user signs in when the username and password exactly match a stored person. The person and their role are then kept in the session for later permission checks.
**Specification:**
  Given a stored person 'mgr' with role MANAGER
  When  they sign in with the matching password
  Then  a session is created holding the user, including role MANAGER, and they are sent to /tasks. A wrong password shows 'Invalid credentials'.
**Parameters:** Passwords are stored and compared as plain text. Seed passwords: <credential - masked, see src/main/resources/db/seed.sql:2-5> (preview 'ad****'; each seed password equals its username). A read-only reporting DB role is also created with a hardcoded password: <credential - masked, see src/main/resources/db/oracle-ro.sql:8> (preview 'or****').
**Edge cases handled:** The role is cached in the session at sign-in, so a role change does not apply until the user signs in again.; No lockout or retry limit.
**Suspected defect:** Plaintext password storage and comparison. Seed and DB-role passwords are hardcoded.
**Confidence:** Medium — Citation was corrected by referee (The behavior is real, but the cited lines cover only part of it. PersonDao.java:16-29 does the credential match: line 18 runs a parameterized query 'SELECT id, username, full_name, role FROM person WHERE username = ? AND password = ?', and lines 21-22 bind the username and password as given, as plain text with no hashing. The comment at line 17 says the same thi…

### RULE-030: Bulk close: empty selection is ignored and no role is required
**Category:** Policy
**Priority:** P1
**Source:** `src/main/java/com/acme/tracker/web/TaskActionServlet.java:76-88`
**Plain English:** Bulk close with no tasks ticked does nothing and returns to the list. Otherwise any signed-in user, VIEWER included, can close the selected tasks, so PENDING_APPROVAL tasks can be completed without a manager.
**Specification:**
  Given A VIEWER is signed in and task 3 'Migrate invoices' is PENDING_APPROVAL
  When  They tick task 3 and press 'Close selected (bulk)'
  Then  Task 3 becomes DONE with a fresh updated_at and the user is sent back to /tasks. With nothing ticked the redirect happens and no task changes.
**Parameters:** No role check in handleBulkClose. AuthFilter only restricts action=approve/reject (AuthFilter.java:42-49). The UI form is tasks.jsp:38-78.
**Edge cases handled:** A non-numeric id in ids throws NumberFormatException (500) and no tasks close; The user is not told how many tasks were closed; the bulkClose return value is ignored; Tasks already DONE are left alone by the DAO filter
**Suspected defect:** A VIEWER or MEMBER can use bulk close to bypass the manager/admin approval rule.
**Confidence:** High

### RULE-031: Read-only reporting access to all tables, including plaintext passwords
**Category:** Policy
**Priority:** P1
**Source:** `src/main/resources/db/oracle-ro.sql:8-12`
**Also cited:** src/main/resources/db/oracle-ro.sql:8-12
**Plain English:** A separate database login with read-only rights is created for reporting and verification. It can read every table, including the person table with its plaintext passwords, but cannot change data.
**Specification:**
  Given The oracle_ro database login
  When  It queries SELECT username, password FROM person
  Then  The query succeeds and returns every user's plaintext password. Any INSERT, UPDATE or DELETE is denied
**Parameters:** Role: oracle_ro, LOGIN, password <credential — masked, see oracle-ro.sql:8, preview 'or**'>. Grants: CONNECT on database tracker, USAGE on schema public, SELECT on all current tables, and default SELECT on future tables
**Edge cases handled:** The default privileges mean new tables are readable automatically; The role is recreated on every fresh database start (deploy configmap line 81 has an identical copy)
**Suspected defect:** The read-only role can read the credential column, and its own password equals its name.
**Confidence:** Medium — Which reporting consumers use the read-only login, and should it be allowed to see the person table (credentials) at all? / Should the read-only role be allowed to read the person table's password column?

### RULE-032: Initial data: one account per role and tasks seeded in every lifecycle state
**Category:** Policy
**Priority:** P1
**Source:** `src/main/resources/db/seed.sql:1-21`
**Also cited:** src/main/resources/db/seed.sql:2-21; src/main/resources/db/seed.sql:2-6
**Plain English:** Every fresh database starts with four sign-ins, one for each role, whose passwords equal their usernames, plus three projects (two active, one archived) and four tasks, one in each status.
**Specification:**
  Given A fresh deployment (the in-memory DB is re-seeded on every start)
  When  Someone signs in as the seeded manager
  Then  They authenticate with the default password. Seeded state: tasks 1 OPEN, 2 IN_PROGRESS, 3 PENDING_APPROVAL, 4 DONE; project 3 'Legacy Archive' ARCHIVED
**Parameters:** Seeded users: admin/ADMIN, mgr/MANAGER, dev/MEMBER, view/VIEWER. Passwords: <credential — masked, see seed.sql:3-6, preview 'ad**'> (each equals its username). Project owners: Apollo -> person 2, Borealis and Legacy Archive -> person 1
**Edge cases handled:** The seed hard-codes ids 1..4, and the rest of the seed relies on IDENTITY assigning them in that order; Seeded passwords are stored in plaintext in person.password; The README (README.md:76) publishes the default sign-ins
**Suspected defect:** Shipped default accounts use password = username, stored in plaintext. | The default accounts, including ADMIN, ship with guessable plaintext passwords. | Known default credentials, including ADMIN, are stored in plaintext in every deployment that loads the seed.
**Confidence:** Medium — Is the seed reference data that production must also have (e.g. a mandatory admin account), or demo/test data only? Must default credentials be forced to change on first login? / Are these seed accounts loaded in production, and must the rewrite force a password change or remove them? / Is seed.sql loaded in production, or only in demo/test environments?

### RULE-033: Task list filtering and sort order
**Category:** Policy
**Priority:** P2
**Source:** `src/main/java/com/acme/tracker/dao/TaskDao.java:149-172`
**Plain English:** The task list can be filtered by status and/or assignee and sorted by a fixed set of options. By default the most recently updated tasks come first, and priority sorting uses business order HIGH, MEDIUM, LOW.
**Specification:**
  Given Tasks A (LOW, updated 10:00), B (HIGH, updated 09:00), C (MEDIUM, updated 11:00)
  When  The list is requested with sort=priority
  Then  The order is B, C, A; with no sort parameter the order is C, A, B (updated_at descending).
**Parameters:** Sort tokens: updated (default) -> updated_at DESC; status -> status, updated_at DESC; priority -> HIGH=0, MEDIUM=1, other=2, then updated_at DESC; due or due_date -> due_date, updated_at DESC; title -> title, id; id -> id. Filters: TaskDao.search (TaskDao.java:40-56) combines status and assignee with AND; TaskListServlet.java:31-32.
**Edge cases handled:** Unrecognized sort tokens fall back to updated_at DESC; Tasks with no due date sort last under the PostgreSQL default (NULLS LAST ascending); Filtering for unassigned tasks is not possible (a null assignee means no filter); Non-numeric assignee parameter -> NumberFormatException (500)
**Confidence:** High

### RULE-034: Task list ordering options
**Category:** Policy
**Priority:** P2
**Source:** `src/main/java/com/acme/tracker/dao/TaskDao.java:154-172`
**Also cited:** src/main/java/com/acme/tracker/dao/TaskDao.java:154-172
**Plain English:** By default the task list shows the most recently updated tasks first. Users can sort by status, priority (HIGH before MEDIUM before LOW), due date, title or id, with recency as the tie-breaker.
**Specification:**
  Given tasks with priorities LOW, HIGH and MEDIUM
  When  the list is requested with sort=priority
  Then  they appear HIGH, MEDIUM, LOW, and equal priorities appear newest first.
**Parameters:** Tokens: updated (default), status, priority, due/due_date, title, id. Anything else falls back to updated_at DESC.
**Edge cases handled:** Status sorts alphabetically (DONE, IN_PROGRESS, OPEN, PENDING_APPROVAL), not in lifecycle order.; Any priority value other than HIGH or MEDIUM sorts as LOW.; Tasks with no due date sort last (PostgreSQL ascending default).
**Confidence:** High

## Rules requiring SME confirmation

- **RULE-002** (Medium): Should VIEWER-role users (and possibly ADMINs) be excluded from the assignee list, and should a bad assignee give a validation message rather than a server error? / Should read-only VIEWER accounts be assignable as task owners?
- **RULE-003** (Medium): Should archived projects be closed to new tasks and task status changes, or is archiving only a label? / When a project is archived, should its open tasks be closed, blocked, or hidden, and should new tasks be refused in archived projects? Who may archive a project: only its owner, managers/admins, or anyone? / Should creating or moving a task into an ARCHIVED project be blocked, and should archi…
- **RULE-004** (Medium): Should the rewrite reject impossible dates (such as Feb 30) and past due dates with a user-facing message, or keep the legacy behavior of accepting any parseable date and failing hard on bad formats?
- **RULE-011** (Medium): Is bulk close meant to be an admin/manager-only override, and should it only apply to tasks in particular statuses (for example PENDING_APPROVAL)? / Is bulk close meant to let any user, including MEMBER and VIEWER, mark tasks DONE without approval? Or should it be limited to approvers, or to tasks that are already PENDING_APPROVAL? / Is bulk close meant as an approver-only administrative override…
- **RULE-013** (Medium): Who is allowed to create and archive projects (today anyone, including VIEWER)? Should archiving freeze the project's tasks, and is project deletion a required feature?
- **RULE-014** (Medium): What should archiving a project prevent: new tasks, edits or transitions on its tasks? And who may archive (owner, MANAGER/ADMIN only)? / Who may archive a project (owner, managers, admins), and what should happen to open tasks in an archived project?
- **RULE-019** (Medium): Is project deletion a required feature, and if so what happens to its tasks? Should archiving be limited to the project owner or managers/admins? / Should projects be deletable at all? If so, what happens to their tasks (block, cascade, or reassign) and which roles may delete? Or should the Delete link be removed in the rewrite? / Is project deletion a required feature that was never built, or sh…
- **RULE-020** (Medium): Is the direct OPEN to DONE path in testDirectOpenToDone a real allowed transition (e.g. for admins), or should the domain layer reject it? / Is moving a task from OPEN (or IN_PROGRESS) straight to DONE without manager approval ever allowed, for example for spikes or admin clean-up? If so, who may do it? Or should the rewrite enforce the approval workflow at the domain/data level and treat testDir…
- **RULE-021** (Medium): What idle timeout should sessions have? The legacy app relies on the container default. / Is there a required idle-timeout or maximum session length for this internal tool, and must a role change take effect immediately for active sessions?
- **RULE-025** (Medium): Should a task be movable between projects? The edit form offers a project picker, but the save logic ignores it. Is that an intended lock or a bug?
- **RULE-026** (Low): What is VIEWER intended to be allowed to do? Should it be read-only, and should MEMBER be restricted (e.g., only start/submit tasks assigned to them)?
- **RULE-028** (Medium): Should VIEWER be read-only (no create, edit, transitions, bulk close or archive)? Should MEMBER be allowed to reopen DONE tasks? Is /api/tasks meant to be public?
- **RULE-029** (Medium): Citation was corrected by referee (The behavior is real, but the cited lines cover only part of it. PersonDao.java:16-29 does the credential match: line 18 runs a parameterized query 'SELECT id, username, full_name, role FROM person WHERE username = ? AND password = ?', and lines 21-22 bind the username and password as given, as plain text with no hashing. The comment at line 17 says the same thi…
- **RULE-031** (Medium): Which reporting consumers use the read-only login, and should it be allowed to see the person table (credentials) at all? / Should the read-only role be allowed to read the person table's password column?
- **RULE-032** (Medium): Is the seed reference data that production must also have (e.g. a mandatory admin account), or demo/test data only? Must default credentials be forced to change on first login? / Are these seed accounts loaded in production, and must the rewrite force a password change or remove them? / Is seed.sql loaded in production, or only in demo/test environments?

## Rules folded into another

These rules described the same behavior as another rule in a different place, so they were merged into it (the kept rule lists their locations under "Also cited"):

- Bulk close forces selected tasks to DONE (src/main/java/com/acme/tracker/dao/TaskDao.java:127-147) into Bulk close marks selected tasks DONE, bypassing the approval workflow
- Bulk close marks selected tasks DONE, skipping the approval step (src/main/java/com/acme/tracker/dao/TaskDao.java:128-147) into Bulk close marks selected tasks DONE, bypassing the approval workflow
- Task status lifecycle (approval workflow state machine) (src/main/java/com/acme/tracker/web/TaskActionServlet.java:39-65) into Task status lifecycle (guarded state machine)
- Task status lifecycle (single-task transitions) (src/main/java/com/acme/tracker/web/TaskActionServlet.java:39-65) into Task status lifecycle (guarded state machine)
- Approval workflow is enforced only in the web action layer; the data layer accepts any status jump (src/test/java/com/acme/tracker/LifecycleTest.java:27-39) into Status guards exist only in the web layer; the data layer accepts any status jump
- Status rules are enforced only in the web action layer, not on raw status writes (src/test/java/com/acme/tracker/LifecycleTest.java:27-39) into Status guards exist only in the web layer; the data layer accepts any status jump
- New tasks start OPEN; status cannot be edited through the form (src/main/java/com/acme/tracker/web/TaskFormServlet.java:64-72) into New tasks start OPEN and editing never changes status
- Project archiving is one-way ACTIVE to ARCHIVED (src/main/java/com/acme/tracker/dao/ProjectDao.java:74-84) into Project lifecycle: ACTIVE to ARCHIVED, forward only
- Project creation requires a name and is owned by the creator (src/main/java/com/acme/tracker/web/ProjectServlet.java:38-50) into Project creation defaults
- Project deletion is offered in the UI but does not exist (src/main/webapp/WEB-INF/jsp/projects.jsp:32) into Archive offered only for active projects; Delete link has no backing function
- Projects cannot actually be deleted (src/main/webapp/WEB-INF/jsp/projects.jsp:24-33) into Archive offered only for active projects; Delete link has no backing function
- Session lifecycle: sign-out ends the session; entry point lands on the task list (src/main/java/com/acme/tracker/web/LogoutServlet.java:11-18) into Signing out ends the session
- Last-updated timestamp refreshed on every task change (src/main/java/com/acme/tracker/dao/TaskDao.java:97-125) into Every task change refreshes its last-updated timestamp
- Each status offers only its valid next actions, but approve/reject buttons are shown to every role (src/main/webapp/WEB-INF/jsp/tasks.jsp:56-70) into Transition buttons offered depend only on current status, not on the user's role
- Transition buttons offered for each status (src/main/webapp/WEB-INF/jsp/tasks.jsp:56-69) into Transition buttons offered depend only on current status, not on the user's role
- Login required for all pages except login, stylesheets and the JSON API (src/main/java/com/acme/tracker/web/AuthFilter.java:28-38) into Login required for the application, with plaintext credential match
- Only managers and admins may approve or reject (src/main/java/com/acme/tracker/web/TaskActionServlet.java:90-98) into Only managers and admins may approve or reject tasks
- Only managers and admins may approve or reject tasks (src/main/java/com/acme/tracker/web/TaskActionServlet.java:90-98) into Only managers and admins may approve or reject tasks
- Default seed data: one account per role, sample projects and tasks (src/main/resources/db/seed.sql:2-21) into Initial data: one account per role and tasks seeded in every lifecycle state
- Default seeded accounts per role (src/main/resources/db/seed.sql:2-6) into Initial data: one account per role and tasks seeded in every lifecycle state
- Read-only reporting database role (src/main/resources/db/oracle-ro.sql:8-12) into Read-only reporting access to all tables, including plaintext passwords
- Task list sort order (semantic priority ranking) (src/main/java/com/acme/tracker/dao/TaskDao.java:154-172) into Task list ordering options
- Task JSON export: contents, filter and order (src/main/java/com/acme/tracker/web/ApiTaskServlet.java:23-49) into Unauthenticated JSON read-back of tasks with optional status filter
- Unauthenticated JSON read-back of task status (src/main/java/com/acme/tracker/web/ApiTaskServlet.java:22-49) into Unauthenticated JSON read-back of tasks with optional status filter
- Allowed values for roles, task statuses, priorities and project states (src/main/resources/db/schema.sql:9-42) into Allowed values for status, priority, role and project state
- Any person of any role can be assigned a task (src/main/webapp/WEB-INF/jsp/task-form.jsp:86-94) into Any user of any role can be assigned a task
- Due date is optional and must be an ISO calendar date (src/main/java/com/acme/tracker/dao/TaskDao.java:182-188) into Due date is optional, must be an ISO date, and has no range check
- Task create/edit validation and defaults (src/main/java/com/acme/tracker/web/TaskFormServlet.java:38-75) into Task save validation
- Task create/edit validation: title required, project must exist (src/main/java/com/acme/tracker/web/TaskFormServlet.java:44-53) into Task save validation
- Archiving a project does not freeze its tasks; archived projects still accept new and moved-in tasks (src/main/java/com/acme/tracker/dao/ProjectDao.java:45-55) into Tasks may be created in archived projects
- Tasks may be filed under archived projects (src/main/java/com/acme/tracker/dao/ProjectDao.java:45-55) into Tasks may be created in archived projects
