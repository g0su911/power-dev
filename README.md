# power-dev

A PM-driven development workflow plugin for Claude Code that combines the best of multiple plugins into a single, session-based development pipeline.

## Overview

power-dev orchestrates long-running feature development across multiple Claude Code sessions by combining:

- **feature-dev** -- PM-style discovery, parallel codebase exploration, and architecture design
- **superpowers** -- TDD, systematic debugging, and verification-before-completion
- **ralph-loop** -- Autonomous iteration for implementation tasks
- **coderabbit** -- External code review integration

The plugin manages context between sessions via a file-based handoff system, so you can start with discovery in one session, implement in another, and review in a third -- without losing context.

## Session Workflow

power-dev operates across three sessions, each building on the previous one's output.

### Session 1: PM / Discovery

Start a new feature with:

```
/power-dev "feature description"
```

This session runs the PM phase:

1. **Discovery** -- Understands the feature request, identifies scope, and surfaces unknowns.
2. **Codebase Exploration** -- Dispatches parallel agents to explore relevant parts of the codebase (file structure, existing patterns, dependencies, test coverage).
3. **Clarifying Questions** -- Presents questions to the user based on what was discovered, resolving ambiguities before committing to a design.
4. **Architecture Design** -- Produces a concrete implementation plan with file-level changes, data flow, and edge cases.

Output is saved to `.claude/power-dev/context.md` for the next session.

### Session 2: Implementation

Continue into implementation:

```
/power-dev continue
```

The plugin reads the saved context and presents implementation options:

- **TDD** -- Uses superpowers' test-driven development skill. Write tests first, then implement to make them pass.
- **Ralph Loop** -- Hands off to ralph-loop for autonomous iteration. Useful for well-specified tasks where the plan is clear.
- **Manual** -- Step-by-step implementation with human guidance at each stage.

Progress is saved back to `.claude/power-dev/context.md` as implementation proceeds.

### Session 3: Review

Continue into the review phase:

```
/power-dev continue
```

This session finalizes the work:

1. **Verification Before Completion** -- Runs all tests, linters, and type checks. Confirms everything passes with evidence.
2. **CodeRabbit Review** -- If available, requests an external code review via the coderabbit plugin.
3. **Summary** -- Produces a summary of all changes, decisions made, and any remaining follow-ups.
4. **Archive** -- Moves the context file to `.claude/power-dev/archive/` with a timestamp for future reference.

## Commands

| Command | Description |
|---|---|
| `/power-dev "description"` | Start a new feature. Enters the PM/discovery phase. |
| `/power-dev continue` | Continue to the next phase (implementation or review). Reads saved context. |
| `/power-dev status` | Show the current phase, progress, and location of the context file. |

## Prerequisites

power-dev integrates with the following plugins. All are optional -- the plugin falls back gracefully when they are not installed.

| Plugin | Used For | Fallback |
|---|---|---|
| **superpowers** | TDD, systematic debugging, verification-before-completion | Built-in basic test running and manual verification |
| **ralph-loop** | Autonomous iteration during implementation | Manual step-by-step implementation |
| **coderabbit** | External code review in the review phase | Skips external review, relies on verification only |

## Handoff File Structure

Context is persisted between sessions at `.claude/power-dev/context.md`. The file follows this structure:

```markdown
# power-dev: <feature name>

## Status
phase: pm | implementation | review | complete
started: <ISO timestamp>
updated: <ISO timestamp>

## Feature Description
<original user description>

## Discovery Notes
<findings from codebase exploration>

## Clarifying Questions & Answers
- Q: <question>
  A: <answer>

## Architecture Plan
<implementation plan with file paths, changes, data flow>

## Implementation Progress
- [x] Step 1: ...
- [ ] Step 2: ...

## Implementation Mode
<tdd | ralph-loop | manual>

## Review Results
### Verification
<test results, lint output, type check output>

### CodeRabbit
<external review summary, if available>

### Summary
<final summary of changes>
```

Archived context files are moved to `.claude/power-dev/archive/<feature-name>-<timestamp>.md` upon completion.

## License

MIT
