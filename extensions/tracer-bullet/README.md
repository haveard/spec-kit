# Tracer Bullet Vertical Slice

A spec-kit extension that adds the `/speckit.tracer` command — a "prove the path, then fill it in" step between `/speckit.tasks` and `/speckit.implement`.

## The Problem

The spec-kit workflow has a structural gap between planning and implementation. `/speckit.implement` tries to build everything at once, and by the time it reaches the integration boundary, layers don't connect. The spec is satisfied on paper — every task checkbox is marked — but the application doesn't work end-to-end.

## The Solution

The tracer bullet forces the integration test to pass **first**, with the thinnest possible code, before any layer gets fleshed out. It inserts a vertical slice step into the workflow:

```
/speckit.specify → /speckit.clarify → /speckit.plan → /speckit.tasks → /speckit.tracer → /speckit.implement
```

## What It Does

`/speckit.tracer` reads existing spec artifacts (`tasks.md`, `plan.md`, `data-model.md`, `contracts/`) and:

1. Identifies the **single thinnest end-to-end path** through the task list
2. Implements that path **bottom-up** (persistence → service → async → API → frontend)
3. Runs a **smoke test** to prove the integration works
4. Produces `tracer.md` documenting the slice, scope fence, and expansion map
5. Marks completed tasks in `tasks.md`
6. Hands off to `/speckit.implement` for the remaining work

## Installation

```bash
specify extension add tracer-bullet
```

## Usage

After running `/speckit.tasks`, run:

```
/speckit.tracer
```

The command will produce a `tracer.md` report and working code that `/speckit.implement` builds on.

## Companion: Constitution Principle

For best results, also add the "Tracer Bullet First" article to your project's `constitution.md`. This ensures all spec-kit commands respect the tracer discipline even outside of the explicit `/speckit.tracer` step. See the spec-kit documentation for the recommended constitution text.

## License

MIT
