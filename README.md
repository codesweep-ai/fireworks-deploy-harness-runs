# Fireworks Deploy - App Discovery Harness Eval Runs

Business-rule discovery of the [task-tracker-demo](https://github.com/codesweep-ai/task-tracker-demo)
app, done two ways for the **Fireworks Deploy Buildathon**:

1. **Baseline:** Claude Code with the Code Modernization plugin, on Claude Opus 5.5.
2. **Codesweep Harness:** a multi-agent campaign running in Codesweep sandboxes, on a mix of
   Fireworks-hosted GLM 5.3 and Kimi K3 models across the campaign agents.

Both analyzed the same frozen app commit, `eee783057ec28d28fb36ec05484de9b4b40ceb9c`, from the
sibling `../task-tracker-demo` repo.

| | Claude Code baseline | Codesweep Harness |
|---|---|---|
| **Business rules discovered** | **34** | **65** (runs 01 + 02) |
| Executable tests reproducing suspected defects | No | Yes |

See [Comparing the two](#comparing-the-two) for how the two rule sets line up.

## Repository layout

```
claude-baseline/analysis/task-tracker/   Claude Code Modernization plugin baseline
harness-runs/01/                         Codesweep Harness run 01 (approval/permission pilot)
harness-runs/02/                         Codesweep Harness run 02 (whole-app expansion)
```

## 1. Claude Code Modernization plugin baseline

The output of `/code-modernization:modernize-extract-rules` on Claude Opus 5.5: **34 business
rules** in `BUSINESS_RULES.md`, with a readable summary in `REPORT.html`.

## 2. Codesweep Harness discovery

The harness then ran the same discovery as a campaign: a team of agents, each in its own
isolated sandbox with its own copy of the app and database. The agents extracted rules, wrote
EARS test specifications and executable HTTP tests, and verified them independently. The
harness relies on these sibling projects from [github.com/codesweep-ai](https://github.com/codesweep-ai):

- [`sandbox`](https://github.com/codesweep-ai/sandbox) provides Firecracker microVM sandboxes for
  each agent, with prewarmed app and database images.
- [`campaign`](https://github.com/codesweep-ai/campaign) handles campaign coordination: profiles,
  dispatch between roles, deadlines, collection and archiving.

Each run used a team of four campaign agents, all on [opencode](https://opencode.ai) with a mix
of Fireworks-hosted models: GLM 5.3 for the orchestrator and rule analyst, Kimi K3 for the test
engineer and verifier.

| Seat | Model |
|---|---|
| orchestrator | `fireworks-ai/accounts/fireworks/models/glm-5p3` |
| rule-analyst | `fireworks-ai/accounts/fireworks/models/glm-5p3` |
| test-engineer | `fireworks-ai/accounts/fireworks/models/kimi-k3` |
| verifier | `fireworks-ai/accounts/fireworks/models/kimi-k3` |

### Runs

| Run | Scope | Duration | Output commit |
|---|---|---|---|
| [01](harness-runs/01/packet/mission.md) | Pilot: one approval/permission slice, 15-minute budget | ~12 min | `5f57517` |
| [02](harness-runs/02/packet/mission.md) | Whole app, continuing from run 01's output | ~24 min | `afd4500` |

Together, runs 01 and 02 discovered an aggregate of **65 business rules**.

Each run directory contains:

| Path | Contents |
|---|---|
| `run.json` | Run metadata: app commit, seats, policy, scope, result |
| `packet/` | Mission, role briefs, profile, resolved plan, toolchain/runtime locks (`packet.html` to browse) |
| `repo/`, `repo.bundle` | Output repository: `rules/`, `specs/`, `tests/`, `findings/`, `reports/`, `evidence/` |
| `archive/` | Per-agent inputs, outputs, dispatch replies, transcripts; orchestrator state; fleet verdict |
| `viewer/` | Dispatch viewer and traces (`viewer/index.html`) |
| `watch/` | Live progress and collection logs (`watch/index.html`) |
| `review/` | Review catalog (`review/index.html`) |
| `cost/` | Cost report (`cost/index.html`) |
| `inputs/` | Frozen app snapshot and runtime jars |

## Comparing the two

`../fireworks-deploy/docs/business-rules-comparison.md` maps the baseline's 34 rules to the
65 rule IDs from harness runs 01 and 02. The two agree on observed behavior almost everywhere.
They differ mainly in judgement (which behaviors count as defects) and coverage. The harness
runs also produced executable tests that reproduce the suspected defects.
