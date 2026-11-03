# Data Objects — task-tracker

## Task (model bean)
Source: `legacy/task-tracker/src/main/java/com/acme/tracker/model/Task.java:3`

| Field | Type | Note |
|---|---|---|
| id | long | PK; read from task.id (TaskDao.java:222); set from the id request param on edit (TaskFormServlet.java:69-70) |
| title | String | Required. Servlet trims it and rejects an empty value (TaskFormServlet.java:41,45-47), matching NOT NULL VARCHAR(256) (schema.sql:30). The servlet does not che… |
| description | String | Optional free text, taken from the request as-is (TaskFormServlet.java:57); VARCHAR(4000) (schema.sql:31) |
| status | String | Plain String, not an enum. Comment lists the values (Task.java:7); constants in TaskStatus. Always set to OPEN on create (TaskFormServlet.java:66); not in the… |
| priority | String | LOW/MEDIUM/HIGH by comment (Task.java:8). Defaults to MEDIUM when blank (TaskFormServlet.java:58). The servlet does no whitelist check; only the DB CHECK enfor… |
| assigneeId | Long (nullable) | Optional FK to person (schema.sql:36,41). Never filtered by role: any person from PersonDao.findAll can be picked (task-form.jsp:90-93). Null is read back via… |
| assigneeName | String (transient) | Display only, from LEFT JOIN person (TaskDao.java:17,20,229) |
| projectId | long | Required. The servlet only checks that the project exists (TaskFormServlet.java:50; ProjectDao.exists ProjectDao.java:45-55), not that it is ACTIVE, so archive… |
| projectName | String (transient) | Display only, from JOIN project (TaskDao.java:17,19,231) |
| dueDate | String (ISO yyyy-MM-dd, nullable) | Comment Task.java:13. Blank is stored as NULL. Otherwise it goes through java.sql.Date.valueOf (TaskDao.java:182-187), which throws IllegalArgumentException on… |
| updatedAt | String (yyyy-MM-dd HH:mm:ss) | Timestamp cut to whole seconds with substring(0,19) (TaskDao.java:234-236). Set to CURRENT_TIMESTAMP on update, updateStatus and bulkClose (TaskDao.java:99,118… |

Used by: New tasks start OPEN and editing never changes status, New-task defaults and what an edit may change, Task save validation, Task data domain constraints, Editing a task cannot move it to another project, Tasks may be created in archived projects, Any user of any role can be assigned a task, Due date is optional, must be an ISO date, and has no range check, Task timestamps shown to whole seconds; due dates as ISO dates, Every task change refreshes its last-updated timestamp, Task status lifecycle (guarded state machine), Status-change precondition: refuse unless the task is in the expected status (c…, Status guards exist only in the web layer; the data layer accepts any status ju…, Order of checks for a single-task action, Only managers and admins may approve or reject tasks, Bulk close marks selected tasks DONE, bypassing the approval workflow, Bulk close: empty selection is ignored and no role is required, Task list filtering by status and assignee, Task list filtering and sort order, Task list ordering options, Transition buttons offered depend only on current status, not on the user's role, Unauthenticated JSON read-back of tasks with optional status filter, Allowed values for status, priority, role and project state

## task (table DDL)
Source: `legacy/task-tracker/src/main/resources/db/schema.sql:28`

| Field | Type | Note |
|---|---|---|
| id | BIGINT IDENTITY PK | schema.sql:29 |
| title | VARCHAR(256) NOT NULL | schema.sql:30 |
| description | VARCHAR(4000) | schema.sql:31 |
| status | VARCHAR(20) NOT NULL DEFAULT 'OPEN' CHECK IN (OPEN,IN_PROGR… | schema.sql:32-33. Checks the domain only, not transitions, so any jump between valid values is accepted at the DB level. The app always passes status explicitl… |
| priority | VARCHAR(16) NOT NULL DEFAULT 'MEDIUM' CHECK IN (LOW,MEDIUM,… | schema.sql:34-35 |
| assignee_id | BIGINT NULL FK person(id) | schema.sql:36,41 |
| project_id | BIGINT NOT NULL FK project(id) | schema.sql:37,40. The FK does not check project state |
| due_date | DATE NULL | schema.sql:38 |
| updated_at | TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP | schema.sql:39. No trigger: the 'refreshed on every mutation' comment holds only because every UPDATE in TaskDao sets it explicitly |

Used by: Task data domain constraints, Allowed values for status, priority, role and project state, Status guards exist only in the web layer; the data layer accepts any status ju…, Every task change refreshes its last-updated timestamp, New-task defaults and what an edit may change, Due date is optional, must be an ISO date, and has no range check, Tasks may be created in archived projects, Initial data: one account per role and tasks seeded in every lifecycle state, Read-only reporting access to all tables, including plaintext passwords

## TaskStatus (string constant holder)
Source: `legacy/task-tracker/src/main/java/com/acme/tracker/model/TaskStatus.java:9`

| Field | Type | Note |
|---|---|---|
| OPEN | static final String | TaskStatus.java:11 |
| IN_PROGRESS | static final String | TaskStatus.java:12 |
| PENDING_APPROVAL | static final String | TaskStatus.java:13 |
| DONE | static final String | TaskStatus.java:14. TaskDao.bulkClose hardcodes 'DONE' as a literal rather than using this constant (TaskDao.java:136-137). The JSPs also hardcode the status s… |

Used by: Task status lifecycle (guarded state machine), Status-change precondition: refuse unless the task is in the expected status (c…, New tasks start OPEN and editing never changes status, Order of checks for a single-task action, Bulk close marks selected tasks DONE, bypassing the approval workflow, Transition buttons offered depend only on current status, not on the user's role, Allowed values for status, priority, role and project state

## Project (model bean)
Source: `legacy/task-tracker/src/main/java/com/acme/tracker/model/Project.java:3`

| Field | Type | Note |
|---|---|---|
| id | long | PK (ProjectDao.java:88) |
| name | String | Required. Trimmed, blank gives 400 (ProjectServlet.java:39-43,46); VARCHAR(128) NOT NULL (schema.sql:20) |
| description | String | Optional (ProjectServlet.java:47; schema.sql:21) |
| state | String | ACTIVE/ARCHIVED by comment (Project.java:7); a String, not an enum. INSERT hardcodes 'ACTIVE' (ProjectDao.java:58). archive() is a conditional UPDATE ... WHERE… |
| ownerId | long | Always the session user's id at create (ProjectServlet.java:44,48); FK person (schema.sql:24-25) |
| ownerName | String (transient) | Display only, from JOIN person (ProjectDao.java:16-17,93) |

Used by: Project lifecycle: create as ACTIVE, archive one way, Project lifecycle: ACTIVE to ARCHIVED, forward only, Project creation defaults, Archive offered only for active projects; Delete link has no backing function, Tasks may be created in archived projects, Allowed values for status, priority, role and project state

## project (table DDL)
Source: `legacy/task-tracker/src/main/resources/db/schema.sql:18`

| Field | Type | Note |
|---|---|---|
| id | BIGINT IDENTITY PK | schema.sql:19 |
| name | VARCHAR(128) NOT NULL | schema.sql:20 |
| description | VARCHAR(1024) | schema.sql:21 |
| state | VARCHAR(16) NOT NULL DEFAULT 'ACTIVE' CHECK IN (ACTIVE,ARCH… | schema.sql:22-23 |
| owner_id | BIGINT NOT NULL FK person(id) | schema.sql:24-25 |

Used by: Project lifecycle: create as ACTIVE, archive one way, Project lifecycle: ACTIVE to ARCHIVED, forward only, Project creation defaults, Allowed values for status, priority, role and project state, Initial data: one account per role and tasks seeded in every lifecycle state, Read-only reporting access to all tables, including plaintext passwords

## Person (model bean; also the session 'user' attribute)
Source: `legacy/task-tracker/src/main/java/com/acme/tracker/model/Person.java:3`

| Field | Type | Note |
|---|---|---|
| id | long | PK (PersonDao.java:48). Used as the project owner (ProjectServlet.java:48) |
| username | String | Login key (PersonDao.java:18) |
| fullName | String | Shown in the header (_header.jspf:13) and in assignee pickers (task-form.jsp:92) |
| role | Role (enum) | Mapped with Role.valueOf (PersonDao.java:51). The DTO has no password field: the password is compared in SQL and never loaded (PersonDao.java:17-18). The Perso… |

Used by: Login required for the application, with plaintext credential match, Sign-in by username and password, Signing out ends the session, Only managers and admins may approve or reject tasks, Access policy: approve/reject is the only role-restricted action, User roles and what VIEWER can actually do, Order of checks for a single-task action, Project creation defaults, Any user of any role can be assigned a task

## person (table DDL)
Source: `legacy/task-tracker/src/main/resources/db/schema.sql:9`

| Field | Type | Note |
|---|---|---|
| id | BIGINT IDENTITY PK | schema.sql:10 |
| username | VARCHAR(64) NOT NULL UNIQUE | schema.sql:11 |
| full_name | VARCHAR(128) NOT NULL | schema.sql:12 |
| password | VARCHAR(128) NOT NULL | schema.sql:13. Stored in plaintext and matched by SQL equality (PersonDao.java:18). Seeded with plaintext values (seed.sql:3-6, e.g. 'ad****'). The read-only D… |
| role | VARCHAR(16) NOT NULL CHECK IN (ADMIN,MANAGER,MEMBER,VIEWER) | schema.sql:14-15. Duplicates the Role enum values (Role.java:4-5) |

Used by: Login required for the application, with plaintext credential match, Sign-in by username and password, User roles and what VIEWER can actually do, Allowed values for status, priority, role and project state, Any user of any role can be assigned a task, Initial data: one account per role and tasks seeded in every lifecycle state, Read-only reporting access to all tables, including plaintext passwords

## Role (enum)
Source: `legacy/task-tracker/src/main/java/com/acme/tracker/model/Role.java:7`

| Field | Type | Note |
|---|---|---|
| ADMIN | enum constant | Role.java:8 |
| MANAGER | enum constant | Role.java:9 |
| MEMBER | enum constant | Role.java:10 |
| VIEWER | enum constant | Role.java:11. The code has no VIEWER-specific restriction. canApprove() is the only role predicate, and it is checked only for approve/reject (AuthFilter.java:… |
| canApprove() | boolean method | True for ADMIN or MANAGER (Role.java:14-16). It is checked twice: in the filter and again in the servlet |

Used by: Only managers and admins may approve or reject tasks, Access policy: approve/reject is the only role-restricted action, User roles and what VIEWER can actually do, Order of checks for a single-task action, Bulk close: empty selection is ignored and no role is required, Transition buttons offered depend only on current status, not on the user's role, Allowed values for status, priority, role and project state

## Task form POST parameters (request DTO)
Source: `legacy/task-tracker/src/main/java/com/acme/tracker/web/TaskFormServlet.java:38`

| Field | Type | Note |
|---|---|---|
| id | String (optional) | Blank means create, present means edit (TaskFormServlet.java:65-71) |
| title | String | Trimmed, required (TaskFormServlet.java:41,45) |
| description | String | TaskFormServlet.java:57 |
| projectId | String parsed to long | Must exist (TaskFormServlet.java:50). Long.parseLong on a non-numeric value throws NumberFormatException (500). Ignored on edit (TaskDao.java:98) |
| assigneeId | String parsed to Long, optional | TaskFormServlet.java:60-61 |
| priority | String | Default MEDIUM (TaskFormServlet.java:58); the options come from the JSP (task-form.jsp:99) |
| dueDate | String (ISO date) optional | TaskFormServlet.java:62; HTML date input (task-form.jsp:107) |

Used by: Task save validation, New-task defaults and what an edit may change, New tasks start OPEN and editing never changes status, Editing a task cannot move it to another project, Tasks may be created in archived projects, Due date is optional, must be an ISO date, and has no range check, Any user of any role can be assigned a task

## Task action POST parameters (request DTO)
Source: `legacy/task-tracker/src/main/java/com/acme/tracker/web/TaskActionServlet.java:23`

| Field | Type | Note |
|---|---|---|
| action | String | start/submit/approve/reject/reopen/bulkClose; any other value gives 400 (TaskActionServlet.java:39-64). bulkClose is routed before any lookup and has no role c… |
| id | String parsed to long | Single-task actions. Order: parse id, then findById (404), then role check for approve/reject (403), then status guard (409), then write (TaskActionServlet.jav… |
| ids | String[] parsed to long[] | Bulk close. Null or empty means redirect with no write (TaskActionServlet.java:77-80). The UPDATE skips rows already DONE (TaskDao.java:136-137) |
| return | String (optional) | 'detail' redirects to the task form page (TaskActionServlet.java:69-71) |

Used by: Task status lifecycle (guarded state machine), Status-change precondition: refuse unless the task is in the expected status (c…, Order of checks for a single-task action, Only managers and admins may approve or reject tasks, Bulk close marks selected tasks DONE, bypassing the approval workflow, Bulk close: empty selection is ignored and no role is required, Status guards exist only in the web layer; the data layer accepts any status ju…

## Task list query parameters
Source: `legacy/task-tracker/src/main/java/com/acme/tracker/web/TaskListServlet.java:22`

| Field | Type | Note |
|---|---|---|
| status | String (optional) | Exact-match filter, parameterized (TaskDao.java:44-47) |
| assignee | String parsed to Long (optional) | TaskListServlet.java:31; TaskDao.java:48-51 |
| sort | String token | Whitelisted tokens: status, priority (HIGH>MEDIUM>LOW), due/due_date, title, id; anything else (including 'updated') sorts by updated_at DESC (TaskDao.java:154… |

Used by: Task list filtering by status and assignee, Task list filtering and sort order, Task list ordering options

## Task JSON payload (GET /api/tasks)
Source: `legacy/task-tracker/src/main/java/com/acme/tracker/web/ApiTaskServlet.java:36`

| Field | Type | Note |
|---|---|---|
| id | number | ApiTaskServlet.java:37 |
| title | string | ApiTaskServlet.java:38. Hand-rolled esc() handles only backslash and double quote, not control characters or newlines (ApiTaskServlet.java:51-54) |
| status | string | ApiTaskServlet.java:39. Optional ?status filter goes through search(status,null,null) (ApiTaskServlet.java:24-27) |
| priority | string | ApiTaskServlet.java:40 |
| assigneeId | number/null | ApiTaskServlet.java:41 |
| projectId | number | ApiTaskServlet.java:42 |
| dueDate | string/null (yyyy-MM-dd) | ApiTaskServlet.java:43 |
| updatedAt | string (yyyy-MM-dd HH:mm:ss) | ApiTaskServlet.java:44. description, assigneeName and projectName are not emitted. The endpoint is unauthenticated because AuthFilter lets /api/* through (Auth… |

Used by: Unauthenticated JSON read-back of tasks with optional status filter, Task timestamps shown to whole seconds; due dates as ISO dates
