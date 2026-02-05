---
name: story-review-fixes
description: >
    Applies issues from story-workspace/review/review-report.md to the Twine/SugarCube
    story using the existing multi-agent pipeline. Use after /create-story Phase 8
    when the static review has produced warnings or suggestions.
---

# Story Review Fixes

## Purpose

Use this skill **after** the `story-reviewer` agent has written
`story-workspace/review/review-report.md` (Phase 8 of `/create-story`).
It defines how to interpret the report and orchestrate agents (or make direct
edits) to resolve issues, then verify with build + tests.

## Inputs

-   `story-workspace/review/review-report.md` — primary source of issues
-   `story-workspace/story-bible.md` — for design intent, variables, endings
-   `story-workspace/passage-graph.json` — for structure / outcomeChain intent
-   `src/story/*.twee` — assembled story implementation (do **not** overwrite `StoryData.twee`)
-   Optional:
    -   `story-workspace/review/test-report.md` — if Playwright tests already ran
    -   `story-workspace/state.json` — for phase/progress context

## Issue Types

When reading the review report, classify each item as:

-   **Critical**: broken links, unreachable passages, syntax errors, build failures
-   **Warning**:
    -   **W-logic**: variable init/usage mismatch, unreachable endings, outcomeChain conflicts
    -   **W-structure**: graph vs implementation mismatch, missing convergence behavior
    -   **W-style**: POV/tense violations, inconsistent tone
-   **Suggestion**:
    -   **S-narrative**: flavor, pacing, running gags
    -   **S-docs**: missing comments, unclear priority in conditionals

Treat critical + warnings as **must fix**; suggestions are **nice to have** unless
the user explicitly asks to apply them.

## High-Level Workflow

1. **Read and summarize** `review-report.md`
2. **Prioritize**:
    - Fix logic/variable mismatches first
    - Then structural / outcomeChain issues
    - Then narrative/style suggestions
3. **Plan concrete edits**:
    - Which `.twee` passages
    - Which bible / graph sections
    - Which variables or endings are affected
4. **Apply fixes** using the appropriate agent(s) and files
5. **Rebuild and test**:
    - `npm run build`
    - Re-run `story-tester` if it’s in scope
6. **Update docs** (Story Bible, comments) so design intent matches code

## Using Agents for Fixes

Prefer agents over ad-hoc edits when possible:

-   **Logic / variable issues (W-logic)** → `sugarcube-expert`
-   **Narrative / flavor / pacing (S-narrative)** → `passage-writer`
-   **Structure / outcomeChain intent (W-structure)** → `plot-architect` (for graph) +
    `sugarcube-expert` (for `.twee`)
-   **Styling / visual issues** → `story-stylist`
-   **Docs / comments / bible alignment (S-docs)** → direct edits to
    `story-workspace/story-bible.md` and inline comments in `.twee`

### SugarCube / Twee Fixes (sugarcube-expert)

When a warning involves variables, endings, or outcome chains:

1. Identify the affected passages and files from the report.
2. Invoke the `sugarcube-expert` agent with:

    ```
    Read:
    - story-workspace/story-bible.md
    - story-workspace/passage-graph.json
    - the affected .twee files in src/story/

    The review report at story-workspace/review/review-report.md highlights
    the following issues:
    [paste the relevant warning/suggestion text]

    Update only the affected .twee passages and, if needed, the Story Bible so that:
    - Variables used in logic are all initialized in StoryInit.twee
    - Variables described in the bible/endings table match the actual implementation
    - outcomeChain passages use automatic routing without conflicting player-facing links
    - StoryData.twee is NOT modified
    ```

3. After agent output:
    - Verify `.twee` syntax (balanced macros, valid links)
    - Spot-check the updated passages for narrative consistency

### Narrative / Pacing Fixes (passage-writer)

For suggestions about flavor text, approach differentiation, or running gags:

1. Gather context:
    - The specific passage(s) from `src/story/*.twee`
    - Relevant sections of `story-bible.md` (e.g., Style Guide, character notes)
2. Invoke `passage-writer` with:

    ```
    Read the following for context:
    - story-workspace/story-bible.md (Style Guide + relevant character sections)
    - The target passage(s) from src/story/*.twee

    The static review suggests these narrative improvements:
    [paste suggestion text]

    Lightly revise ONLY the specified passages to address the suggestions while
    preserving POV (2nd person), tense (present), tone, and word-count guidance.
    Return updated passage text to be written back into the .twee files.
    ```

3. Replace only the relevant passage blocks in the `.twee` files.

### Documentation / Commenting (S-docs)

When the report recommends documenting outcome chains or complex conditionals:

-   Add inline comments **inside** the conditional blocks, following the example
    format from the review report:

    ```twee
    <<if $chaos >= 4>>
      <<goto "The Spectacular Failure">> /* Priority 1: Catastrophic chaos */
    <<elseif $caught and $charm >= 2>>
      <<goto "The Negotiator">> /* Priority 2: Charmed capture */
    <</if>>
    ```

-   Update `story-workspace/story-bible.md` when:
    -   An ending’s “Key requirements” no longer match the implemented logic
    -   A variable listed in the bible is no longer used in code

## Specific Patterns from the Sample Review

Use these heuristics for similar future reports:

-   **Unused variable initialized in StoryInit (e.g., `$duchess_betrayed`)**

    -   If the mechanic was dropped from the final graph:
        -   Remove it from `StoryInit.twee`
        -   Remove or update its entry in the bible’s variable table
    -   If the mechanic should exist:
        -   Implement the state changes in the relevant passages and ensure it is
            referenced in at least one conditional or ending requirement.

-   **Design variable in bible not present in code (e.g., `$distraction_used`)**

    -   Prefer aligning the bible to actual code if the implemented logic already
        covers the intended behavior (e.g., using `$chaos` or `$lamp_knocked`).
    -   Only introduce a new variable if it meaningfully clarifies logic that
        cannot be cleanly expressed using existing state.

-   **OutcomeChain passages mixing auto-routing and player choices**
    -   Confirm intent in `passage-graph.json` and the bible.
    -   If the graph flags a pure outcomeChain:
        -   Remove player-visible choice links from that passage chain.
        -   Use `<<if>> / <<elseif>> / <<else>>` with `<<goto>>` to route to endings
            based solely on game state.
    -   If explicit final choices are intentional:
        -   Add a brief comment noting the deviation from the original graph for
            future maintainers.

## Verification & Build

After applying a batch of fixes:

1. Run build:

    ```bash
    npm run build
    ```

2. If build fails:

    - For `.twee` errors: re-invoke `sugarcube-expert` with the exact error text
    - For SCSS/CSS errors: re-invoke `story-stylist` with the error details

3. Optionally re-run Playwright tests via `story-tester`:

    ```
    The dev server is running on port 4321.
    Re-run tests focusing on the paths/endings affected by recent fixes.
    Update story-workspace/review/test-report.md.
    ```

4. Update `story-workspace/state.json` with a brief note that review fixes
   have been applied and built, including a timestamp.

## When to Stop

Consider the review fix cycle complete when:

-   All **critical issues** are resolved
-   All **warnings** are either fixed or consciously documented as intentional
-   The build passes
-   Optional: Playwright tests report no critical path failures
-   Story Bible and variable/endings documentation match the current code
