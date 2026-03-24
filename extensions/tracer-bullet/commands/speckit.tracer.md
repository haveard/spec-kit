---
description: >
  Identify and implement the thinnest end-to-end vertical slice (tracer bullet)
  through the stack, proving the integration path works before full implementation.
handoffs:
  - label: Implement Remaining Tasks
    agent: speckit.implement
    prompt: Continue implementing remaining tasks. The tracer bullet is complete.
    send: true
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Prerequisites

Before executing, verify these artifacts exist in the current feature's specs directory:

- `tasks.md` (REQUIRED — run `/speckit.tasks` first if missing)
- `plan.md` (REQUIRED)
- `spec.md` (REQUIRED)
- `data-model.md` (REQUIRED if data layer exists)
- `contracts/` (REQUIRED if API contracts exist)
- `/memory/constitution.md` (REQUIRED)

If any required artifact is missing, STOP and report which artifacts are needed.

## Outline

1. **Setup**: Run `{SCRIPT}` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute.

2. **Load context**: Read all prerequisite artifacts. Identify the tech stack, infrastructure, and layer architecture from `plan.md`.

3. **Identify the tracer path**: Analyze `tasks.md` and `plan.md` to find the single thinnest end-to-end path. This means:
   - ONE entry point (the most representative API endpoint, webhook, or event trigger)
   - Every layer between entry and persistence, with the minimum work at each layer
   - ONE exit point (observable proof: HTTP response, DB row, published message, log entry)
   - The path should cross every integration boundary the feature introduces

4. **Define the scope fence**: Explicitly list what the tracer DEFERS. This typically includes:
   - Full validation (tracer validates shape only)
   - Error handling beyond "doesn't crash"
   - Authorization (use hardcoded/test identity)
   - Performance optimization
   - UI polish (if frontend: one unstyled component proving the data round-trip)
   - Edge cases and alternate flows

5. **Implement bottom-up**: Build the tracer layer by layer, starting from persistence:
   - 5a. **Persistence**: Minimal migration with only the columns the tracer touches. Verify insert + read-back.
   - 5b. **Service layer**: Thinnest function that writes to DB and returns result. Hardcode decisions where needed.
   - 5c. **Async layer** (if applicable): Minimal Celery task or NATS consumer proving message → service → DB.
   - 5d. **API / Entry layer**: Endpoint with minimal Pydantic validation. Call service, return result.
   - 5e. **Frontend** (if applicable): One component, one API call, one rendered result. No styling.
   - 5f. **Infrastructure wiring**: Document any new containers, config, dependencies.

6. **Verify the tracer**: Run a smoke test that:
   - Sends the trigger
   - Waits for the expected side effect
   - Asserts the exit point condition
   - This test MUST run against real infrastructure, not mocks

7. **Mark completed tasks**: For every task in `tasks.md` that the tracer fully completed, mark it `[X]`.

8. **Produce `tracer.md`**: Write the tracer report to `specs/[feature]/tracer.md` using the tracer template. Include:
   - The slice diagram (Mermaid sequence diagram)
   - Layer map with what was implemented vs. deferred at each layer
   - Scope fence (what was deferred)
   - Verification results (smoke test pass/fail + evidence)
   - Expansion map: remaining work items grouped by parallelizable track
   - Risk register: anything the tracer revealed that wasn't anticipated

9. **Stop and report**: Do NOT proceed to full implementation. Report:
   - Tracer status (PASS/FAIL)
   - Which tasks in `tasks.md` are now complete
   - Path to `tracer.md`
   - Handoff recommendation: ready for `/speckit.implement` or issues to resolve first

## Tracer Verification Checklist

Before declaring the tracer complete, confirm ALL of these:

- [ ] Entry point receives a request and doesn't crash
- [ ] Every layer in the path is touched — no skipped layers, no shortcuts
- [ ] Persistence layer has real data written by the tracer
- [ ] Exit point produces the observable proof
- [ ] Smoke test passes against real infrastructure (Docker Compose, local server, etc.)
- [ ] No mocks in the integration path (mocks allowed only for external services outside the stack)
- [ ] All new infrastructure is documented
- [ ] `tracer.md` is written with slice diagram, scope fence, verification results, and expansion map
- [ ] Completed tasks are marked `[X]` in `tasks.md`

## Critical Rules

- **Real code, not throwaway.** Every line stays. The tracer is the skeleton that gets fleshed out.
- **Bottom-up construction.** Persistence first, entry point last.
- **One path, no branches.** Single happy-path flow. No conditionals, no error branches.
- **Hardcode over abstract.** Configurable thresholds become constants. Dynamic lookups become hardcoded values.
- **Infrastructure honesty.** If the tracer needs something missing from the current infrastructure, flag it in the risk register. Don't work around it.
- **Stop when the smoke test passes.** Not when the code is clean. When the end-to-end test proves the path works.
