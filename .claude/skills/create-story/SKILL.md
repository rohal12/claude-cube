---
name: create-story
description: >
    Orchestrates the multi-agent pipeline for creating an interactive
    SugarCube/Twine story. Invoke with /create-story followed by your story idea.
---

# Create Interactive Story

You are the orchestrator for a multi-agent interactive fiction pipeline. Follow these phases sequentially. Use the Task tool to invoke specialized agents. After each agent completes, verify its output before proceeding.

## Setup

1. Create `story-workspace/` directory if it doesn't exist:
    ```
    mkdir -p story-workspace/lore story-workspace/passages story-workspace/assembly story-workspace/review story-workspace/review/screenshots
    ```
2. Check if `story-workspace/state.json` exists — if so, read it and offer to resume from the last completed phase.

3. **IFID Generation for new stories**:
    - Check if this is a new story (no `story-workspace/story-bible.md` exists, or user chose "Start over")
    - If new story, generate a fresh IFID using: `uuidgen | tr '[:lower:]' '[:upper:]'`
    - Store the IFID in `story-workspace/state.json` under `"ifid"` for use during assembly
    - Each unique story MUST have its own unique IFID

## Phase 1: Story Concept (Interactive — REQUIRES USER DIALOGUE)

**This phase is a creative conversation, not a one-shot generation.**

Invoke the `story-architect` agent:

```
Use the story-architect agent to collaborate with the user on the story concept.
The user's story idea is: [include the user's prompt here]

IMPORTANT: Do NOT write the Story Bible immediately. You must:
1. Propose specific concepts (protagonist, setting, conflict) for the user to react to
2. Wait for the user to respond and refine
3. Only after at least one round of user feedback, ask if they're ready to finalize
4. Write the Story Bible to story-workspace/story-bible.md only after user approval
```

**Verification before proceeding**:

-   The story-architect MUST have engaged in dialogue (not just output the Story Bible)
-   If the Story Bible was written without user interaction, delete it and re-invoke with emphasis on the interactive requirement

After user-approved completion:

-   Verify `story-workspace/story-bible.md` exists and is non-empty
-   Update state: `{ "current_phase": 1, "phase_name": "story_concept_complete" }`
-   Tell the user: "Story Bible complete. Designing the passage structure now."

## Phase 2: Structure Design

Invoke the `plot-architect` agent:

```
Read story-workspace/story-bible.md and design the passage graph.
Write to story-workspace/passage-graph.json.
```

After completion:

-   Verify `story-workspace/passage-graph.json` exists and is valid JSON
-   Read the file to extract metadata
-   Update state: `{ "current_phase": 2, "phase_name": "structure_complete" }`

## Checkpoint 1: Passage Graph Review

Present the passage graph to the user. Read `story-workspace/passage-graph.json` and display:

1. **Story title and stats**: total passages, branch count, ending count, max depth
2. **Graph visualization** — for each act, show passages and their connections:

    ```
    ACT 1 (N passages):
      Start → [follow_path, investigate_sound]
      follow_path → [companion_trust, companion_distrust]
      investigate_sound → [find_cave, retreat]

    ACT 2 (N passages):
      village_gate ← CONVERGENCE [companion_trust, companion_distrust, retreat]
      village_gate → [village_square]
      ...

    ENDINGS (N):
      ending_best "Redemption" (requires: $trust >= 5, $has_key)
      ending_neutral "Survival" (requires: none)
      ending_worst "Tragedy" (requires: $trust < 0)
    ```

3. **Tracked variables**: list each variable with its type, initial value, and purpose

Ask the user using `AskUserQuestion`:

-   "Does this passage structure look good?"
-   Options: "Approve and continue", "I want changes"

If the user wants changes, ask what they'd like to change, then re-invoke `plot-architect` with the feedback appended. Repeat until approved.

## Phase 3: Lore Building

Invoke the `lore-keeper` agent:

```
Read story-workspace/story-bible.md and story-workspace/passage-graph.json.
Build the lore database in story-workspace/lore/.
```

After completion:

-   Verify lore files exist in `story-workspace/lore/`
-   Update state: `{ "current_phase": 3, "phase_name": "lore_complete" }`

## Phase 4: Passage Writing

Read `story-workspace/passage-graph.json` and plan batching:

### Batching Strategy

-   Count total passages (excluding the one with id `start` if handled separately — actually include it)
-   If ≤ 8 passages: write all in one batch
-   If 9-20 passages: one batch per act
-   If > 20 passages: batches of 5-7 passages, grouped by act and narrative order

### For Each Batch

Determine what context the batch needs:

1. **Graph nodes**: Extract the passage objects for this batch from the JSON
2. **Lore files**: Map passage tags to lore categories:
    - Tags mentioning character names or `companion` → `characters.md`
    - Tags mentioning locations or `forest`, `village`, etc. → `world.md`
    - Tags mentioning `faction` or group names → `factions.md`
    - Tags mentioning items → `items.md`
    - When in doubt, include `lore-index.md` and `characters.md`
3. **Preceding passages**: For each passage in the batch, identify which passages link TO it (its "parents"). If those parent passages have already been written, include them for continuity.
4. **Style guide**: Always include reference to the story bible's style guide section.

Invoke the `passage-writer` agent with a detailed prompt:

```
Write passages: [id1, id2, id3, ...].

Here are the passage nodes from the graph:
[paste the relevant JSON nodes]

Read these files for context:
- story-workspace/lore/characters.md (for [character names])
- story-workspace/lore/world.md (for setting details)
- story-workspace/passages/[preceding_id].md (preceding passage for continuity)
- story-workspace/story-bible.md (Style Guide section)

Write output to story-workspace/passages/<id>.md for each passage.
```

After each batch:

-   Verify all expected passage files were created
-   Update state with passage count progress
-   Proceed to next batch

## Phase 5: Assembly

Invoke the `sugarcube-expert` agent:

```
Read all passage files in story-workspace/passages/ and the passage graph
at story-workspace/passage-graph.json.
Assemble valid .twee files in src/story/.
Preserve src/story/StoryData.twee — do not overwrite it.
Write assembly manifest to story-workspace/assembly/assembly-manifest.json.
```

After completion:

-   Verify `.twee` files exist in `src/story/`
-   Verify `StoryData.twee` is unchanged
-   Update state

## Phase 6: Visual Styling

Invoke the `story-stylist` agent:

```
Read story-workspace/story-bible.md to understand the genre, tone, and setting.
Read story-workspace/passage-graph.json for the story title and any tag hints.
Create a custom CSS theme that reflects the story's atmosphere.
Write the stylesheet to src/assets/app/styles/story-theme.scss.
```

After completion:

-   Verify `src/assets/app/styles/story-theme.scss` exists and is non-empty
-   Update state: `{ "current_phase": 6, "phase_name": "styling_complete" }`

## Phase 7: Build Validation

Run the build:

```bash
npm run build
```

Capture the output. If the build fails:

1. Read the error output
2. If CSS/SCSS error: Re-invoke `story-stylist` with the specific error to fix
3. If .twee syntax error: Re-invoke `sugarcube-expert` with the specific error:
    ```
    The build failed with this error:
    [paste error]
    Fix the issue in the affected file(s).
    ```
4. Re-run `npm run build`
5. Repeat up to 3 times. If still failing after 3 attempts, proceed to Checkpoint 2 with the errors.

Update state.

## Phase 8: Static Review

Invoke the `story-reviewer` agent:

```
Review the assembled story.
Read all .twee files in src/story/.
Read story-workspace/passage-graph.json and story-workspace/story-bible.md.
Write your review to story-workspace/review/review-report.md.
```

After completion:

-   Read the review report
-   Note critical issues, warnings, and suggestions

## Phase 9: Playwright Playthrough Testing

### Start the Dev Server

```bash
npm start &
```

Wait a few seconds for the server to start, then verify it's running.

### Run the Tester

Invoke the `story-tester` agent:

```
The dev server is running on port 4321.
Read story-workspace/passage-graph.json to understand the story structure.
Play through ALL paths from Start to every ending using playwright-cli.
Report results to story-workspace/review/test-report.md.
Save error screenshots to story-workspace/review/screenshots/.
```

### Test-Fix Loop

After the tester completes, read `story-workspace/review/test-report.md`:

**If there are errors:**

1. Categorize each error:
    - **Broken links / bad .twee syntax**: Re-invoke `sugarcube-expert` with the specific errors to fix
    - **Missing/wrong prose content**: Re-invoke `passage-writer` for the affected passages, then `sugarcube-expert` to re-assemble
    - **Variable/logic errors**: Re-invoke `sugarcube-expert` with details
2. After fixes: re-run `npm run build`
3. Re-invoke `story-tester` to verify fixes
4. **Repeat until the test report has zero critical errors** (max 5 iterations)
5. If still failing after 5 iterations, proceed to Checkpoint 2 with remaining errors

**If there are no errors**: Proceed to Checkpoint 2.

### Stop the Dev Server

After testing is complete, stop the background server process.

## Checkpoint 2: User Reviews Final Story

Present to the user:

1. **Build status**: Pass/Fail
2. **Playwright test results**: All paths pass / remaining issues
3. **Static review summary** from the review report:
    - Critical issues (count and brief descriptions)
    - Warnings (count and brief descriptions)
    - Suggestions (count)
4. **Story stats**: passage count, path count, ending count, variable count

Ask the user using `AskUserQuestion`:

-   "How would you like to proceed?"
-   Options:
    -   "Fix remaining issues and rebuild" — re-run the fix loop for critical/warning items
    -   "Edit specific passages" — ask which passages, re-invoke passage-writer for those, then sugarcube-expert, rebuild, and retest
    -   "Accept and finalize" — the story is done; tell the user to run `npm start` to play
    -   "Start over" — clear story-workspace/ and restart from Phase 1

## State Tracking

After each phase, write `story-workspace/state.json`:

```json
{
    "current_phase": 4,
    "phase_name": "passage_writing",
    "story_title": "The Forest of Echoes",
    "passages_total": 25,
    "passages_written": 12,
    "batches_completed": 2,
    "batches_total": 4,
    "build_attempts": 0,
    "test_iterations": 0,
    "last_updated": "2026-02-04T10:30:00Z"
}
```

## Important Notes

-   Always verify agent output (files exist, non-empty, valid) before proceeding to the next phase.
-   Never skip checkpoints — the user must approve the passage graph and the final result.
-   Keep the user informed of progress between phases with brief status messages.
-   If any agent fails or produces invalid output, retry once before escalating to the user.
-   The passage-writer uses opus (most expensive model) — batch efficiently to minimize invocations.
