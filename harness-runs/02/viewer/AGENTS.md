# Questions about the campaign run task-tracker-02

This folder is one finished campaign run, written by `cs-dispatch-viewer v0.0.0-20261102055029-b0f0bf56f5cd` for an
agent to answer questions about it. The data is here, in this folder, and nowhere else you need to
look. Read all of this page before answering; it is short. The two rules that matter most:
**measure before you claim**, and **every claim carries a link**.

## The run

- campaign `task-tracker-02`, id `task-tracker-02-7aac623421b3edbe`
- outcome claimed: `campaign-exhausted`, with 1 issues from the viewer, 1 of them severe or worse
- elapsed 0:22:58, 33 protocol events, 10 dispatches, 9 sessions
- traces read by `cs-tracer v0.0.0-20260929195245-961ae414c82a (linux/amd64, go1.27.3)`

| member | role | CLI | model |
|---|---|---|---|
| orchestrator | orchestrator | opencode | fireworks-ai/accounts/fireworks/models/glm-5p3 |
| rule-analyst | agent | opencode | fireworks-ai/accounts/fireworks/models/glm-5p3 |
| test-engineer | agent | opencode | fireworks-ai/accounts/fireworks/models/kimi-k3 |
| verifier | agent | opencode | fireworks-ai/accounts/fireworks/models/kimi-k3 |

## What a campaign is

A **campaign** runs a team of AI coding agents on one **mission**. One member is the
**orchestrator**: it reads the mission, writes briefs, and hands out work to the other
**members** as **dispatches**. A dispatch is one unit of asked work, numbered `d001`, `d002`, and on
per member. The mission itself is dispatch `m1` to the orchestrator. A member's first dispatch is
its **readback**, a check at create time that it read its brief; the host issues and judges it, so
the orchestrator owes it no acceptance. The orchestrator **opens** a dispatch, the member
**replies**, and the orchestrator **accepts** the reply or opens another. When the harness judges a
dispatch stalled it sends a **continue**, and past that a **restart** of the member's session. The
orchestrator ends with a **verdict**, and the run records an **outcome**. The outcome, the verdicts
and every reply are claims. The viewer's **issues** argue with them, and an issue outranks a claim.

Members run on different CLIs, which name their tools differently: `Bash` and `Read` are Claude
Code, `bash`, `read` and `write` are OpenCode, `exec` and `shell` are Codex.

## What is in this folder

- `facts.json`: the numbers asked for most, already computed. **Start here.** It has the elapsed
  time; per member work, idle, idle as a share of that member's own time, tool calls, tool output
  in bytes and model output tokens; every dispatch, longest first; the ten longest idle gaps; every
  failed step, classed; every continue and restart the harness sent, with the gap before it and the
  policy it acted under; every subagent; and every tool call repeated three times or more.
- `run-data.json`: everything the dispatch page draws. Events, dispatches, the orchestrator's log,
  every reply and message, the issues, and each session's steps.
- `traces/<session>.json`: one session's whole trajectory as the tracer wrote it, with `node`
  added. `events` holds every step in full: the tool input, the result text, thinking, tokens.
  These files are large; read the one event you need, by its `i`, rather than the whole file.
- `index.html`: the dispatch page. `tracer/`: one page per session.

Use `jq` or a short script to read the JSON. Every number in an answer should come from one of
these files, and an answer should say which.

## Links

Everything on the page has an address, and the data carries it as `addr`. Put the served URL in
front of it: `<served-url>/#<addr>`. The operator tells you the served URL. If they have not, ask
once, and until then write links as `#<addr>`. Never invent a host.

- `m/<member>/<dNNN>` picks a dispatch and zooms to it.
- `e/<n>` picks a protocol event.
- `traces/e/<n>` picks a step on the traces view. A step's `idleAddr` picks the idle mark after it.
- A step's `page` in `facts.json` is its place on its own trace page, `tracer/traces/<session>.html#ev-<i>`, which
  goes after the served URL with no `#` added. The facts carry it for every step they name; for another, it is the session's `page` with `#ev-<i>` after it.
- `@h:mm:ss-h:mm:ss` after any address sets the visible window, in time since the run began.

Those are the only forms. There are no ranges and no lists. Take every address from the data
rather than working one out.

A step the page does not draw, such as a turn end, has no `addr`. Cite its `page` instead.

A figure has no address: a total, a count, a policy value or a member's idle share. Cite it by file
and field, in words, such as `facts.json` `totals.toolOutputBytes`. To link it, link the file:
`<served-url>/facts.json`. Never put `#` in front of a field name, and never link a figure to an
event it does not describe.

## Answering

State the answer first, in plain words, then the evidence, one claim a line, each with its link.
Short beats complete: the reader will click. Say what you computed and what you read. When you do
not know, say so; never invent a number, a cause or a link.

Things that mislead, and what to say instead:

- **The files record what happened, not why.** A cause no field records is judgement: that a
  policy made the harness wait, or that one member's calls used up another's quota. Label it as
  judgement, or leave it out. A policy value is a limit the harness acted under, not something it
  did.
- **Tool output is not output tokens.** `toolOutputBytes` is what tools returned to a model;
  `outputTokens` is what the model wrote. Answer the one asked.
- **Idle does not sum across members.** Twelve hours of idle in a six-hour run is three members
  waiting through one gap. Use each member's `idleShare`, and name the gap.
- **The orchestrator's work is mostly waiting.** It holds one long turn and blocks in the harness's
  wait call; `wait` says how much. For "who did the most work", compare the members.
- **A continue or restart is the harness, not the orchestrator.** `recovery.events` lists them.
  Each carries the gap before it, the step that preceded the gap, and the policy beside it. Report
  those facts.
- **A resume is not a continue.** After a provider refuses a turn, the same session is resumed and
  no rung is spent. The harness resumes a member; the operator resumes the orchestrator. The
  record says when it happened, not why it happened then.
- **Errors have classes.** `provider` is the model's server, `account` a rate limit, `guard` the
  member's CLI refusing a call, `command` the member's own command failing. Do not lump them.
- **A repeated call is not a loop until you look.** `repeatedCalls` counts them. `polling` marks only
  the harness's wait call, which repeats by design. A member can wait in other ways, such as
  `sleep` and then a read of the same file; those repeat too, and are not marked. Re-reading one
  file three times is normal.
- **The outcome is a claim.** A `campaign-converged` outcome beside a `campaign-not-met` issue
  means the record disagrees with itself. Report both, and lead with the issue.

## Fields

### facts.json

- `schemaVersion`: the version of this document's shape
- `campaign`: the campaign's name
- `elapsed`: from the first protocol event to the last
- `elapsed.from`
- `elapsed.to`
- `elapsed.duration`
- `elapsed.fromAddr`
- `elapsed.toAddr`
- `totals`: the run's totals
- `totals.toolCalls`
- `totals.toolOutputBytes`: bytes the tools returned to the models. Not tokens
- `totals.outputTokens`: tokens the models wrote. Not tool output
- `totals.failedSteps`: failed steps by class
- `members`: one per node, with its sessions
- `members[].node`
- `members[].role`
- `members[].cli`
- `members[].work`: the tracer's work time for each session, summed over the member's sessions
- `members[].idle`: the time between its turns
- `members[].wait`: the part of work spent in the harness's wait call. The orchestrator spends most of its turn there
- `members[].idleShare`: idle as a share of this member's own work plus idle, from 0 to 1
- `members[].steps`
- `members[].toolCalls`
- `members[].toolOutputBytes`
- `members[].outputTokens`
- `members[].sessions`: the same, per session
- `members[].sessions[].id`
- `members[].sessions[].source`
- `members[].sessions[].model`
- `members[].sessions[].work`
- `members[].sessions[].idle`
- `members[].sessions[].steps`
- `members[].sessions[].toolCalls`
- `members[].sessions[].toolOutputBytes`
- `members[].sessions[].outputTokens`
- `members[].sessions[].page`
- `dispatches`: every dispatch, longest first; one never replied to runs to the end of the run
- `dispatches[].node`
- `dispatches[].dispatch`
- `dispatches[].phase`
- `dispatches[].openedAt`
- `dispatches[].repliedAt`
- `dispatches[].duration`
- `dispatches[].continues`
- `dispatches[].restarts`
- `dispatches[].resumes`
- `dispatches[].handoffs`
- `dispatches[].commitAsks`
- `dispatches[].addr`
- `idleGaps`: the ten longest gaps between a member's turns, longest first
- `idleGaps[].idle`
- `idleGaps[].addr`: the idle mark on the page
- `idleGaps[].after`: the step the gap follows
- `failedSteps`: every step that failed, in session order. Each is a step reference plus `class` and `result`
- `failedSteps[].class`: `provider` (the model's server failed), `account` (a rate limit), `guard` (the member's CLI refused the call), `command` (the member's own command failed) or `other`
- `failedSteps[].result`: the start of what the call returned
- `recovery`: every continue, restart, resume, hand-off and ask to commit. The harness sends a continue or a restart when it judges a dispatch stalled, and each spends a rung of the policy. A resume carries the same session on after its provider refused a turn, and spends none. A hand-off starts a fresh session on the node's own handoff after its context crossed its line, and spends none. An ask to commit (`commit`) is a hand-off not taken: the node's work was not committed, or its handoff did not name the commit its branch is at, so the session is kept and asked to commit and write its handoff again. It spends none either. The harness resumes a member; the operator resumes the orchestrator, with `cs-campaign resume`
- `recovery.policy`: the limits it acted under: continues and restarts allowed per dispatch, the seconds of quiet it calls a stall, and the most seconds a dispatch may sit behind a failing provider, after which its node is stuck rather than resumed. Each is a limit, not a record of what the harness did
- `recovery.policy.continueAttempts`
- `recovery.policy.restarts`
- `recovery.policy.stallSeconds`
- `recovery.policy.providerWaitSeconds`
- `recovery.events`: in time order
- `recovery.events[].type`: `continue`, `restart` or `resume`
- `recovery.events[].node`
- `recovery.events[].dispatch`
- `recovery.events[].at`
- `recovery.events[].addr`
- `recovery.events[].sinceLastStep`: the gap from that member's last recorded step to this event
- `recovery.events[].lastStep`: that step, and whether it errored
- `recovery.events[].turnError`: the failed step that ended that step's turn, when one did
- `subagents`: every subagent a member spawned: a step reference plus `child`, the child's session
- `subagents[].child`
- `repeatedCalls`: one member calling one tool with the same input three times or more, most first
- `repeatedCalls[].node`
- `repeatedCalls[].tool`
- `repeatedCalls[].input`: the input, cut short
- `repeatedCalls[].count`
- `repeatedCalls[].polling`: the call is the harness's wait, which repeats by design. Only that call is marked: a member can wait in other ways, such as `sleep` and then a read of the same file
- `repeatedCalls[].addrs`: each call's step

A duration has these fields:

- `ms`: milliseconds
- `hms`: the same, as h:mm:ss

A step reference, in `idleGaps`, `failedSteps`, `recovery.events` and `subagents`, has these
fields:

- `node`: the member
- `session`: its session
- `at`: when
- `kind`: the step's kind, as on the strip
- `label`: the tool name, or the error's kind
- `error`: whether the step failed
- `text`: its first line, or its command, cut short
- `addr`: the step's address on the page; absent for a turn end, which the page does not draw, and then `page` is the link to cite
- `page`: the step on its trace page

### run-data.json

- `schemaVersion`: the version of this document's shape
- `campaign`: the campaign as its record states it
- `campaign.name`
- `campaign.id`
- `campaign.createdAt`: when the campaign was created
- `campaign.updatedAt`: when its record last changed
- `campaign.outcome`: the outcome the orchestrator's mission reply claims, such as `campaign-met`. A claim, which the issues can contradict
- `campaign.outcomeAt`: when that reply was written
- `campaign.policy`: the recovery policy the run used; each field is in the manual's policy table
- `campaign.policy.continueAttempts`
- `campaign.policy.restarts`
- `campaign.policy.elapsedSeconds`
- `campaign.policy.blindProbes`
- `campaign.policy.pollSeconds`
- `campaign.policy.settlingSeconds`
- `campaign.policy.providerWaitSeconds`
- `campaign.policy.stallSeconds`
- `campaign.policy.handoffTokens`
- `campaign.policy.handoffs`
- `campaign.policy.rapidHandoffSeconds`
- `campaign.policy.freshPerDispatch`
- `nodes`: the team, orchestrator first: `name`, `role`, `cli`, `model`, `effort`
- `nodes[].name`
- `nodes[].role`
- `nodes[].cli`
- `nodes[].model`
- `nodes[].effort`
- `nodes[].handoffTokens`: the context line, in tokens, this member hands off at; absent for a seat with none. It can differ from `campaign.policy.handoffTokens`, which is the default a seat's own policy overrides
- `nodes[].freshPerDispatch`: true for an agent that started each dispatch in a fresh session; absent otherwise. Like `handoffTokens`, a seat's own policy can set it where `campaign.policy` does not
- `events`: every protocol event in time order
- `events[].node`: the node whose channel it is on; an accept, plan or assessment is the orchestrator's
- `events[].type`: `open`, `continue`, `restart`, `resume`, `handoff`, `commit`, `reply`, `verdict`, `accept`, `plan` or `assessment`
- `events[].at`: when, to the second
- `events[].dispatch`: the dispatch it belongs to
- `events[].seq`: a continue's or restart's number within its dispatch
- `events[].file`: the message file it came from
- `events[].phase`: a reply's phase, such as `done`
- `events[].outcome`: a verdict's outcome
- `events[].text`: a log entry's text; on an accept, the node accepted
- `events[].addr`: the event's address on the page
- `spans`: one per dispatch
- `spans[].id`: the dispatch, `dNNN` per member or `m1` for the mission
- `spans[].node`
- `spans[].openedAt`
- `spans[].repliedAt`: absent when never replied to
- `spans[].acceptedAt`: when the orchestrator's log accepted it
- `spans[].phase`: the reply's phase
- `spans[].continues`: continues the harness sent on it
- `spans[].restarts`: restarts the harness sent on it
- `spans[].resumes`: resumes sent on it: the same session carried on after its provider refused a turn, spending no rung
- `spans[].handoffs`: hand-offs on it: a fresh session started on the node's own handoff after its context crossed its line, spending no rung
- `spans[].commitAsks`: hand-offs on it that were not taken: the session was kept and asked to commit its work and name the commit in its handoff, spending no rung. The hand-off the kept session makes after one counts against the hand-off bound when it comes within `campaign.policy.rapidHandoffSeconds` of the ask
- `spans[].addr`: the address that picks the dispatch and zooms to it
- `log`: the orchestrator's own log: claims, not channel traffic
- `log[].at`
- `log[].kind`
- `log[].text`
- `log[].member`: a delivery entry only: the member the push named
- `log[].repo`: a delivery entry only: the repository, as the member names it
- `log[].commit`: a delivery entry only: the commit the push named
- `replies`: each reply, by member and dispatch
- `replies.<node>.<dispatch>.dispatch`
- `replies.<node>.<dispatch>.phase`
- `replies.<node>.<dispatch>.note`: the member's own summary of what it did
- `replies.<node>.<dispatch>.outcome`: the mission reply's outcome
- `replies.<node>.<dispatch>.unmet`: what the mission reply says was not met
- `replies.<node>.<dispatch>.at`
- `replies.<node>.<dispatch>.repos`: the repositories the reply names
- `replies.<node>.<dispatch>.raw`: the reply file as written
- `messages`: the text of every dispatch and continue, by member and file name
- `readback`: the host's readback record: which members restated their brief, and in which dispatch
- `fleetVerdict`: the host's audit of the machines after the run
- `issues`: the viewer's findings, most severe first. An issue outranks a claim
- `issues[].severity`: `error`, `severe`, `warning` or `info`
- `issues[].code`: defined in `issueDefs`
- `issues[].message`
- `issues[].node`
- `issues[].dispatch`
- `timelineValid`: false when message times cannot be trusted; the page then draws no timeline
- `issueDefs`: one line defining each issue code
- `traces`: what cs-tracer read from each member's transcript
- `traces.tool`: the tracer's version
- `traces.sessions`: one per session, in the order the page numbers their steps
- `traces.sessions[].id`: the session, and the name of its file in `traces/`
- `traces.sessions[].node`: the member it belongs to
- `traces.sessions[].source`: the CLI: `claude-code`, `codex` or `opencode`
- `traces.sessions[].page`: its trace page
- `traces.sessions[].parent`: the session that spawned it, for a subagent
- `traces.sessions[].parentEventIndex`: the step in the parent that spawned it
- `traces.sessions[].startedAt`
- `traces.sessions[].endedAt`
- `traces.sessions[].model`
- `traces.sessions[].events`: how many events the trajectory holds
- `traces.sessions[].strip`: the session's steps
- `traces.sessions[].strip[].i`: the step's index in its trajectory, and `#ev-<i>` on its trace page
- `traces.sessions[].strip[].kind`: `user`, `assistant`, `thinking`, `tool_call`, `system`, `meta` or `turn_end`
- `traces.sessions[].strip[].ts`: when
- `traces.sessions[].strip[].workMs`: the time this step accounts for: from the end of the previous piece of work to the end of this one. Tool calls running at once overlap, so these do not sum to a session's work
- `traces.sessions[].strip[].idleMs`: time after this step before the next turn began
- `traces.sessions[].strip[].activeMs`: the part of `workMs` the model spent generating, where the CLI records it
- `traces.sessions[].strip[].turnEnd`: the step ends a turn
- `traces.sessions[].strip[].error`: the step failed; `facts.json` classes every one
- `traces.sessions[].strip[].label`: the tool name, or the error's kind such as `server_error`
- `traces.sessions[].strip[].subtask`: the step spawned a subagent
- `traces.sessions[].strip[].childSessionId`: that subagent's session
- `traces.sessions[].strip[].text`: the step's first line, or its command, cut short
- `traces.sessions[].strip[].wait`: the step is a call to the harness's wait
- `traces.sessions[].strip[].addr`: the step's address on the page; absent for a step the page does not draw: one with no time, a turn end, or `meta` and `system` steps
- `traces.sessions[].strip[].idleAddr`: the address of the idle mark after it
- `traces.anchors`: where each dispatch shows in the traces
- `traces.anchors[].node`
- `traces.anchors[].dispatch`
- `traces.anchors[].kind`: `sent`, `arrived`, `replied` or `accepted`
- `traces.anchors[].session`
- `traces.anchors[].i`: the step, by its `i`
