---
phase: "08"
slug: "container-backed-try-it-out-warm-on-visit"
status: draft
nyquist_compliant: false
wave_0_complete: false
created: "2026-10-08"
---

# Phase 08 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | vitest ^4.1.0 |
| **Config file** | `vitest.config.ts` (node env; components use jsdom; includes `src/**/*.test.ts`, `scripts/**/*.test.ts`) |
| **Quick run command** | `npx vitest run <touched test file>` |
| **Full suite command** | `npm test` |
| **Estimated runtime** | ~30 seconds |

Known baseline: 3 tests in `scripts/set-try-it-out-mode.test.ts` already fail before this phase (pre-existing). The suite is otherwise green.

---

## Sampling Rate

- **After every task commit:** Run the touched test file(s) with `npx vitest run <file>`
- **After every plan wave:** Run `npm test`
- **Before `/gsd-verify-work`:** Full suite green (apart from the 3 known pre-existing failures), plus one recorded live smoke run
- **Max feedback latency:** 60 seconds

---

## Per-Task Verification Map

The planner fills this table per task. The decision-to-test mapping below comes from `08-RESEARCH.md` § Validation Architecture.

| Decision | Behavior | Test Type | Automated Command | File Exists | Status |
|----------|----------|-----------|-------------------|-------------|--------|
| D-08/D-09/D-10 | Lease CAS transitions, concurrent claim vs evict, cap, busy, min idle age, reload reuse, per-session replace, TTL reaper | unit (fake runtime) | `npx vitest run src/lib/server/tryitout/leaseManager.test.ts` | ❌ W0 | ⬜ pending |
| D-11/D-12/D-16 | JSONL parser, event mapping, completion rule (exit 0 + settled + stopReason, retry failure, timeout) | unit | `npx vitest run src/lib/server/tryitout/piEvents.test.ts` | ❌ W0 | ⬜ pending |
| D-11/D-13 | Container job end-to-end with fake runtime, events in order, output tar to zip | unit/integration | `npx vitest run src/lib/server/tryitout/piRunner.test.ts` | ❌ W0 | ⬜ pending |
| D-13 | Zip builder: traversal names rejected, caps, round trip | unit | `npx vitest run src/lib/server/tryitout/artifact.test.ts` | ❌ W0 | ⬜ pending |
| D-14 | Fallback only on engine unreachable or start failure, never at cap; env switch; zip with result.txt | unit | `npx vitest run src/lib/server/tryItOutRunner.test.ts` | update existing | ⬜ pending |
| D-17/D-22 | LLM proxy: token validation, key injection, streaming, budget 429 | unit/route | `npx vitest run src/routes/api/llm` | ❌ W0 | ⬜ pending |
| D-17 | `/api/llm/` bypasses user auth, other `/api/*` stays 401 | unit | `npx vitest run src/hooks.server.test.ts` | ❌ W0 (check for an existing hooks test) | ⬜ pending |
| D-05/D-06 | Page load warms only for runnable agents, does not block render | unit | `npx vitest run "src/routes/agents/[slug]"` | ❌ W0 | ⬜ pending |
| D-02/R-05 | `SKILL.md` frontmatter valid; fallback loader strips it | unit | `npx vitest run src/lib/server/tryItOutPrompts.test.ts` | update existing | ⬜ pending |
| D-21 | Container create options: CapDrop ALL, limits, tmpfs, labels, no ports, internal network | unit | `npx vitest run src/lib/server/tryitout/runtimeDockerode.test.ts` | ❌ W0 | ⬜ pending |
| D-01/D-03/D-19/D-20/D-21 (live) | Socket access, internal network DNS to the app alias, Pi offline start, read-only rootfs, archive on tmpfs, exec demux and exit code, limits, timeout kill, orphan cleanup, real proxy round trip | live smoke (manual) | `scripts/smoke-pi-runner.ts` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] Live spike script (needs Podman): confirm `putArchive` into tmpfs and a read-only rootfs, exec demux and exit code, `--internal` DNS, compat socket access from the app container; its result selects the file-staging variant
- [ ] `src/lib/server/tryitout/runtimeFake.ts` — shared fake runtime (scriptable exec stdout JSONL, injected failures, latency, archive store)
- [ ] Recorded Pi JSONL fixtures (success with read/write tools, model-error with exit 0, retry failure), hand-written from Pi `docs/json.md`
- [ ] Test files listed in the map above
- [ ] `npm install dockerode tar-stream fflate` and `@types/dockerode` (vitest is already present)

---

## Manual-Only Verifications

| Behavior | Decision | Why Manual | Test Instructions |
|----------|----------|------------|-------------------|
| Warm container starts on page open and the first job completes in Pi | D-05, D-11 | Needs Podman, the built image and a real OpenAI key | Run `scripts/smoke-pi-runner.ts`, then open `/agents/demo-rfi-triage`, run the sample RFI and download the zip |
| Container cannot reach anything except the LLM proxy | D-19 | Needs a real internal network | From a warm container, confirm an external host fails and the proxy alias answers |
| App-in-container start with the socket mounted | D-01 | Needs the compose stack | `podman-compose up`, open the page, confirm a Pi container appears and is cleaned up after the idle TTL |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 60s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
