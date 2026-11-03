# Preflight — task-tracker

- **Date:** 2026-11-02
- **Source:** `legacy/task-tracker` → symlink to `/home/jsdelfino/codesweep-ai/task-tracker-demo`
- **App commit:** `eee783057ec28d28fb36ec05484de9b4b40ceb9c` (clean working tree)
- **Purpose of this pass:** business-rule extraction, characterization tests and coverage only, with no modernization (see `../../PLAN.md`).

## Answers

1. **Scope:** Is this code the complete system, or one slice of a larger codebase? If a slice, what outside depends on it, and is breaking those consumers acceptable?
   > Complete system
2. **Build & test locally:** Can this environment restore, build and run the tests? Roughly how long does the full CI pipeline take?
   > Yes, run-tests.sh
3. **Bespoke build infrastructure:** Is there organization-specific build or dependency machinery a newcomer would not guess? Where is it documented?
   > None
4. **Prior attempts:** Has anyone tried to modernize this before? What went wrong?
   > No prior attempts
5. **Off limits:** Is anything in this code not allowed to change in this pass?
   > Whole app repo

## Check 6: Scope boundary

**Standalone repository.** The git top level is the app directory itself. `pom.xml` has no `<parent>` and no module references. Nothing in any sibling directory under `/home/jsdelfino/codesweep-ai` references `com.acme.tracker` or `task-tracker-demo` in a `pom.xml` or Gradle file. There are no outbound or inbound crossings, which matches answer 1.

## Results

| # | Check | Status | Found | Fix |
|---|---|---|---|---|
| 0 | Ask the human | ✅ | All five answered (above) | — |
| 1 | Stack | ✅ | Java Servlet 3.1 + JSP/JSTL 1.2 + JDBC (PostgreSQL), WAR for Tomcat 9, Maven. 39 tracked files: Java 19 files / 888 LOC (`web` 8, `model` 5, `dao` 3, `db` 2, 1 test), JSP/JSPF 6 / 294, SQL 3 / 54 (`schema.sql`, `seed.sql`, `oracle-ro.sql`), `web.xml`, CSS, Dockerfile, k8s YAML under `deploy/`, 3 shell scripts, GitHub Actions CI | — |
| 2 | Analysis tooling | ⚠️ | `cloc` 2.10 ✅, `python3` 3.14.7 ✅, `scc` missing (assess falls back to cloc), `lizard` missing (only `assess --portfolio` needs it) | Optional: `sudo dnf install scc` (or `go install github.com/boyter/scc/v3@latest`), `pip install --user lizard`. Neither tool is needed by this plan |
| 3a | Build definition | ✅ | CI (`.github/workflows/ci.yml`): JDK 17 Temurin, `mvn -B verify` against a `postgres:16` service, loading `schema.sql`, `seed.sql` **and `oracle-ro.sql`**. Second job: `podman build` of the multi-stage Dockerfile (`maven:3.9-eclipse-temurin-17` → `tomcat:9-jre17`). `maven.compiler.release=8`. Dependencies come from Maven Central only: no `.mvn/`, `settings.xml` or parent POM. Nothing bespoke | — |
| 3b | Smoke test | ✅ | **Level 1:** host `javac` 21 `--release 8` compiled all 18 main sources against `javax.servlet-api` 3.1.0 with no errors. **Level 2:** the app's own `run-tests.sh` flow ran on a `git archive HEAD` copy in the scratch dir (podman, `postgres:16` + `maven:3.9-eclipse-temurin-17`): BUILD SUCCESS, `LifecycleTest` 2/2 pass, 15 s wall. The copy was deleted afterwards and nothing ran inside the app repo. Host also has JDK 21 + Maven 3.9.11 + podman 5.8.7; both images are already pulled | — |
| 4 | Source completeness | ✅ | All imports resolve (`java.*`, `javax.servlet.*`, `com.acme.*`, `org.junit`). The only JSP include is `_header.jspf`, which is present. The only taglib is JSTL core, provided by `jstl` 1.2. Descriptors present: `web.xml` (routing, `AuthFilter` on `/*`, `BootstrapListener`). Data definitions: `schema.sql`, `seed.sql`, `oracle-ro.sql`. One binary: `target/task-tracker.war`, an untracked build output covered by `.gitignore`, not source. Left untouched | — |
| 5 | Optional context | ⚠️ | No APM/telemetry. Git history is a single commit (`eee7830`), so there is no change-frequency signal | None needed for this pass |
| 6 | Scope boundary | ✅ | Standalone (see above) | — |
| 7 | Source protected from edits | ✅ | `.claude/settings.local.json` → `permissions.deny`: `Edit(/legacy/**)` and `Edit(//home/jsdelfino/codesweep-ai/task-tracker-demo/**)`. These cover the link and its real target | — |

**Check 7 caveat:** a permission rule covers Claude's file tools and the shell commands it recognizes (`sed`, `tee`, `> file`), not a script that opens files itself. The hard guarantee is the OS: a read-only mount, or a sandbox that blocks shell writes. This plan's `run-coverage.sh` mounts the app repo `:ro` for that reason.

### Notes for later steps

- CI loads **`oracle-ro.sql`** after schema and seed, but the app's `run-tests.sh` loads only schema + seed. It creates a SELECT-only `oracle_ro` role and changes no app behavior. Extraction should still see it, and `run-coverage.sh` can follow `run-tests.sh` (schema + seed) as the plan says.
- Note for `run-coverage.sh`: podman runs rootful here (`build-war.sh` mentions root-owned files), so files copied into the writable `reports/coverage/` mount may end up root-owned unless ownership is fixed inside the container.

## Verdicts

| Command | Verdict | Why |
|---|---|---|
| `assess` | Ready-with-gaps | `scc` and `lizard` are missing and cloc is the fallback (not used in this plan) |
| `map` | Ready | python3 present, source complete |
| `extract-rules` | **Ready** | Checks 1–2 green for its needs, 0 missing includes |
| `brief` | Ready once discovery artifacts exist | No tooling needed (not used in this plan) |
| `transform` / `reimagine` | Ready-with-gaps | No target stack chosen; the legacy toolchain is green (not used in this plan) |
| `harden` | Ready-with-gaps | No Java SAST tool found, so Claude-derived only (not used in this plan) |
| `uplift` | n/a | No version bump requested |

**Next step for this plan:** `/code-modernization:modernize-extract-rules task-tracker` (Step 2). The plugin's usual next step, `modernize-assess`, is skipped here on purpose.
