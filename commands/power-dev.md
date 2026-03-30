---
description: PM-driven development workflow with parallel codebase exploration, architecture design, and session handoff
argument-hint: Feature description, 'continue', or 'status'
---

# power-dev

Initial request: $ARGUMENTS

## Argument Routing

Route based on `$ARGUMENTS`:
- If `$ARGUMENTS` is "continue" → jump to **Continue Flow**
- If `$ARGUMENTS` is "status" → jump to **Status Flow**
- Otherwise → this is a new feature request. Start **PM Flow**.

---

## PM Flow (Session 1: Phases 1-4)

### Phase 1: Discovery

**Goal:** Understand what needs to be built through collaborative dialogue.

**Actions:**

1. Use TodoWrite to create a tracker for all 7 phases:
   - [ ] Phase 1: Discovery
   - [ ] Phase 2: Codebase Exploration
   - [ ] Phase 3: Clarifying Questions
   - [ ] Phase 4: Architecture Design
   - [ ] Phase 5: Implementation
   - [ ] Phase 6: Verification & Review
   - [ ] Phase 7: Summary & Archive

2. Treat `$ARGUMENTS` as the initial feature description. Ask clarifying questions **one at a time** to build a complete understanding. Prefer multiple choice when possible. Cover:
   - What problem are we solving?
   - What should the feature do?
   - Any constraints or requirements?
   - Success criteria?

3. After gathering enough information, summarize your understanding of the feature and **wait for the user to confirm** before proceeding.

**Key principle:** One question per message. Do not overwhelm. Wait for answers.

Mark Phase 1 complete in TodoWrite when the user confirms the summary.

---

### Phase 2: Codebase Exploration

**Goal:** Deep understanding of relevant existing code and patterns.

**Actions:**

1. Launch 2-3 code-explorer agents in parallel using the `code-explorer` agent. Each agent targets a different aspect of the codebase relevant to the feature:
   - "Find features similar to [feature] and trace implementation comprehensively"
   - "Map the architecture and abstractions for [feature area] comprehensively"
   - "Analyze current implementation of [related feature/area] comprehensively"

2. After all agents return, read ALL files they identified as essential. Do not skip any.

3. Present a comprehensive summary of findings to the user:
   - Key patterns and conventions discovered
   - Essential files and their roles
   - Integration points for the new feature
   - Anything surprising or concerning

Mark Phase 2 complete in TodoWrite.

---

### Phase 3: Clarifying Questions

**Goal:** Fill in all gaps and resolve ambiguities before designing.

**CRITICAL: DO NOT SKIP THIS PHASE.** This is where design mistakes are caught early.

**Actions:**

1. Review codebase findings from Phase 2 alongside the original feature request from Phase 1.

2. Identify underspecified aspects. Consider:
   - Edge cases and error scenarios
   - Error handling strategy
   - Integration points with existing code
   - Backward compatibility concerns
   - Performance requirements
   - Security implications

3. Present all questions in an organized list, grouped by topic.

4. **WAIT for the user to answer before proceeding.**

If the user says "whatever you think is best" or similar, provide your specific recommendation and get explicit confirmation before moving on.

Mark Phase 3 complete in TodoWrite.

---

### Phase 4: Architecture Design

**Goal:** Design the implementation approach with user input.

**Actions:**

1. Launch 2-3 code-architect agents in parallel using the `code-architect` agent. Each agent explores a different design approach:
   - **Minimal changes approach** — smallest diff, maximum reuse of existing code
   - **Clean architecture approach** — maintainability, elegant abstractions, long-term quality
   - **Pragmatic balance** — speed and quality balanced for the context

2. Review all approaches returned by the agents.

3. Present to the user:
   - Brief summary of each approach (2-3 sentences each)
   - Trade-offs comparison (table or bullet list)
   - **Your recommendation with reasoning**

4. **WAIT for the user to choose an approach** before proceeding.

Mark Phase 4 complete in TodoWrite.

---

### Handoff: Save PM Results

After Phase 4 completes and the user has chosen an approach:

1. Create the handoff directory:
   ```
   mkdir -p .claude/power-dev/history
   ```

2. Write `.claude/power-dev/context.md` with the following format:

   ```markdown
   ---
   feature: "[feature name from discovery]"
   status: implementing
   current_phase: 5
   created: [YYYY-MM-DD]
   last_updated: [YYYY-MM-DD]
   ---

   ## Requirements
   [Phase 1 discovery results — summarized requirements, constraints, success criteria]

   ## Codebase Analysis
   [Phase 2 explorer results — key findings, essential files, patterns discovered]

   ## Clarifications
   [Phase 3 Q&A results — questions asked and answers received]

   ## Design Decisions
   [Phase 4 architecture results — chosen approach, rationale, implementation map, build sequence]
   ```

3. Tell the user:

   ```
   PM session complete. Context saved to .claude/power-dev/context.md

   Next: Open a new session and run /power-dev continue to start implementation.
   ```

---

## Continue Flow

Read `.claude/power-dev/context.md`.

If the file does not exist, inform the user:

> No active power-dev session found. Start with `/power-dev 'feature description'`

If the file exists, parse the `status` field from the frontmatter:
- `implementing` → proceed to **Phase 5: Implementation**
- `reviewing` → proceed to **Phase 6: Verification & Review**

---

### Phase 5: Implementation

**Goal:** Build the feature using the chosen architecture.

**Actions:**

1. Display a context summary to orient the session:
   - Feature name
   - Chosen architecture approach
   - Key files to create or modify
   - Build sequence from the design

2. Present implementation method choices:

   ```
   How would you like to implement?

   1. TDD — Red-Green-Refactor cycle (recommended for most features)
      Uses superpowers:test-driven-development

   2. Ralph Loop — Autonomous iteration until completion
      Uses /ralph-loop with completion promise

   3. Manual — Direct implementation, your way
   ```

3. **Wait for the user to choose**, then proceed accordingly:

   **If TDD:**
   - Remind: "Follow TDD discipline: write failing test first, then minimal code to pass, then refactor."
   - Suggest: "Use superpowers:test-driven-development skill for guidance."
   - Begin implementation following the build sequence from context.md.

   **If Ralph Loop:**
   - Generate a ralph-loop prompt from context.md incorporating requirements, architecture, and build sequence.
   - Suggest the command: `/ralph-loop "prompt" --completion-promise "COMPLETE" --max-iterations N`
   - The user runs it.

   **If Manual:**
   - Begin implementation following the build sequence from context.md, pausing at each phase for user input.

4. During implementation, if problems arise:
   - Suggest: "Use superpowers:systematic-debugging for structured diagnosis."

5. After implementation is complete, update `.claude/power-dev/context.md`:
   - Change `status` to `reviewing`
   - Change `current_phase` to `6`
   - Update `last_updated` to today's date
   - Add an `## Implementation Log` section with:
     - Files changed (list with paths)
     - Key implementation decisions made during coding
     - Known issues or concerns
     - Test results

6. Mark Phase 5 complete in TodoWrite.

7. Tell the user:

   ```
   Implementation complete. Context updated.

   Next: Open a new session and run /power-dev continue to start review.
   ```

---

### Phase 6: Verification & Review

**Goal:** Verify the implementation and get external review.

**Actions:**

1. Display a context summary to orient the session.

2. **Verification first:**
   - Run all tests and confirm they pass
   - Run the build and confirm it succeeds
   - Verify no regressions in existing functionality
   - Suggest: "Use superpowers:verification-before-completion for thorough verification."
   - Present verification results to the user with evidence (command output).

3. **External review (if coderabbit is installed):**
   - Suggest running a coderabbit review.
   - Present review findings and ask the user how to proceed:
     - Fix issues now
     - Track issues for later
     - Proceed as-is
   - If coderabbit is not installed, skip this step gracefully and note that external review was skipped.

4. If issues are found, fix them and re-verify. Repeat until clean.

Mark Phase 6 complete in TodoWrite.

---

### Phase 7: Summary & Archive

**Goal:** Close out the feature with a clear record.

**Actions:**

1. Present a summary:
   - What was built (feature description and scope)
   - Key decisions made across all phases
   - Files modified and created (full paths)
   - Test coverage (what is tested, what is not)
   - Suggested next steps (create a PR, update documentation, notify team, etc.)

2. Archive the context file:
   - Move `.claude/power-dev/context.md` to `.claude/power-dev/history/YYYY-MM-DD-[feature-name].md`
   - Use the feature name from the frontmatter, slugified (lowercase, hyphens for spaces).

3. Mark Phase 7 and all remaining todos complete in TodoWrite.

---

## Status Flow

Read `.claude/power-dev/context.md`.

If the file does not exist:

> No active power-dev session.

If the file exists, display:
- **Feature:** name from frontmatter
- **Status:** current status (implementing, reviewing, etc.)
- **Phase:** current phase number and name
- **Last updated:** date from frontmatter
- **Progress:** brief summary of which phases are complete and what comes next

---

## Important Notes

- This is a **guided workflow**. Always wait for user input at decision points. Never auto-proceed past a phase that requires user choice.
- **Never skip Phase 3** (Clarifying Questions). This phase catches design issues early and is essential to the workflow.
- The handoff file (`.claude/power-dev/context.md`) is the **source of truth** between sessions. Always read it at the start of a continue flow and update it at the end of each phase.
- Reference existing plugin skills (superpowers, ralph-loop, coderabbit) but do not require them. If a skill is not installed, skip that option gracefully and note it.
- Use **TodoWrite** to track progress through all 7 phases. Update it as each phase completes.
- Each phase should feel like a **conversation**, not a monologue. Present information, then wait for the user to respond.
