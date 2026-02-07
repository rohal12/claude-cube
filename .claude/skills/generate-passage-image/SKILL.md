---
name: generate-passage-image
description: >
    Generates an AI illustration for a story passage and inserts it into the .twee file.
    Invoke with /generate-passage-image followed by the passage title.
    Maintains a visual style guide for consistent art across all passages.
---

# Generate Passage Image

Generate an AI illustration for a specific story passage using the Gemini image generation API (direct REST call for reliable aspect ratio control). The illustration is inserted into the `.twee` file using SugarCube wiki markup and styled with CSS.

**Cost note**: Each image generation costs real money (~$0.04 with the default flash model). Always show the user the prompt and get approval before generating.

## Step 1: Parse the Passage Title

Extract the passage title from the user's command argument. For example, `/generate-passage-image The Score` targets the passage titled "The Score".

If no argument was provided, ask the user:

> Which passage would you like to generate an illustration for?

## Step 2: Find the Passage in `.twee` Files

Search all `.twee` files in `src/story/` for a passage header matching `:: <Title>` (with optional `[tags]` suffix).

Use Grep to find the header:

```
pattern: ^:: <Title>
path: src/story/
glob: *.twee
```

**If not found:**

1. List all passage titles from `src/story/*.twee` by searching for lines starting with `:: `
2. Suggest the closest match
3. Ask the user to confirm or provide a corrected title

**If the passage is a special passage** (StoryInit, StoryData, StoryTitle, PassageReady, PassageDone):

- Warn the user: "This is a special SugarCube passage that typically isn't displayed to the reader. Are you sure you want to add an illustration?"
- Only proceed if the user confirms.

## Step 3: Read the Passage Content

1. Read the `.twee` file containing the passage.
2. Extract the full passage content: from the line after `:: Title [tags]` up to the next `:: ` header (or end of file).
3. Also look up the passage in `story-workspace/passage-graph.json` by matching the `title` field. If found, note the `summary` and `briefing` fields for additional context when composing the image prompt.

## Step 4: Check for Existing Image

Search the passage content for `<div class="passage-illustration">`.

If found, ask the user:

> This passage already has an illustration. Would you like to replace it?

Options: "Replace it" / "Keep existing"

If the user chooses to keep the existing image, exit the skill.

## Step 5: Ensure Visual Style Guide Exists

Check if `story-workspace/visual-style.md` exists.

### If it does NOT exist (first use):

1. Read `story-workspace/story-bible.md` for genre, tone, setting, and character descriptions.
2. Read `story-workspace/lore/characters.md` for detailed character appearances.
3. Read `src/assets/app/styles/story-theme.scss` for the color palette (CSS custom properties).

4. Draft a visual style guide following this template:

```markdown
# Visual Style Guide

## Art Style

[Propose a style based on the story's genre and tone, e.g., "Digital illustration, semi-realistic with warm cinematic lighting"]

## Color Palette

- Primary: [from --theme-bg and --theme-text]
- Accent: [from --theme-accent and key story colors]
- Shadows/mood: [atmosphere colors matching the story tone]

## Character Appearance Reference

### [Character Name]

- [Species/physical build]
- [Coloring, distinctive markings]
- [Typical posture/expression]
- [How they're framed from the protagonist's perspective]

[Repeat for each major character]

## Composition Guidelines

- [Perspective notes, e.g., "rat-level perspective, low camera angle"]
- [Lighting, e.g., "warm lamp glow, dramatic shadows"]
- Do NOT include "wide landscape format" or "16:9" in prompt text — aspect ratio is controlled by the API's `imageConfig.aspectRatio` parameter. Including it in the prompt causes black letterbox bars.
- No text, no watermark, no signature, no artist mark
- [Scale/framing notes relevant to the story]

## Prompt Suffix

"[art style], [mood keywords], [perspective], [lighting], no text, no watermark, no signature"
```

5. Present the draft to the user via `AskUserQuestion`:

    > I've drafted a visual style guide based on your story. Here's a summary:
    >
    > - **Art style**: [proposed style]
    > - **Palette**: [proposed colors]
    > - **Perspective**: [proposed perspective]
    >
    > Would you like to use this style, or customize it?

    Options: "Use this style" / "I want to customize it"

6. If the user wants to customize: ask what they'd like to change (art style, perspective, etc.), incorporate their feedback, and finalize.

7. Write the finalized guide to `story-workspace/visual-style.md`.

### If it already exists:

Read `story-workspace/visual-style.md` and proceed.

## Step 6: Identify Key Moment and Generate Image Prompt

1. Read the passage prose carefully. Identify the single most visually striking moment — an action beat, dramatic reveal, or atmospheric scene. **Note which paragraph this moment occurs in**, as it determines where the image will be placed in Step 10.

2. Read the visual style guide for:
    - Character appearance descriptions (for any characters present in the scene)
    - The "Prompt Suffix" (appended to every prompt for consistency)
    - Composition guidelines

3. Compose an image generation prompt (2-4 sentences) that:
    - Describes the key visual moment with specific details (setting, characters, action, lighting, mood)
    - Uses character appearance details from the visual style guide verbatim
    - Ends with the Prompt Suffix from the visual style guide
    - Never includes dialogue or text to render in the image
    - **Do NOT include** "wide landscape format", "16:9", or "widescreen" in the prompt text — aspect ratio is controlled solely by the API's `imageConfig.aspectRatio` parameter. Including aspect ratio hints in the prompt causes the model to render black letterbox bars inside the image.
    - Include "no watermark, no signature, no artist mark" in the prompt — but be aware this is **not fully reliable**. Gemini may still render watermarks stochastically. If the user reports a watermark, simply regenerate (the next attempt often comes out clean).

**Example prompt:**

> A small black-and-white rat with dark markings over his head and shoulders peers through the bars of his cage on a bookshelf, gazing down at an open pizza box on a coffee table far below, golden cheese glistening under warm lamplight in a cozy cluttered apartment. Digital illustration, cinematic composition, rat-level perspective, warm apartment lighting, comedic heist atmosphere, no text, no watermark, no signature

## Step 7: User Approval (Cost Gate)

Present the generated prompt to the user:

> **Image prompt for "[Passage Title]":**
>
> [the prompt]
>
> Estimated cost: ~$0.04 (Gemini flash model)

Ask via `AskUserQuestion`:

> Ready to generate this image?

Options: "Generate" / "Edit the prompt first" / "Cancel"

- **Generate**: proceed to Step 8.
- **Edit the prompt first**: Ask the user for their modified prompt. Use their version. Re-confirm with the same options.
- **Cancel**: Exit the skill without generating.

## Step 8: Generate the Image

1. Ensure the output directory exists:

    ```bash
    mkdir -p src/assets/media/passages/
    ```

2. Compute the filename slug from the passage title:
    - Lowercase the title
    - Replace spaces and non-alphanumeric characters with hyphens
    - Collapse multiple hyphens
    - Trim leading/trailing hyphens
    - Example: "The Counsel of Gerald" → `the-counsel-of-gerald`

3. **Call the Gemini REST API directly** (do NOT use the nano-banana skill — it does not support the `aspectRatio` parameter, causing all images to generate as 1:1 square). Use this curl command:

    **Use a two-step approach** (save response to temp file, then extract). Piping curl directly through jq and base64 can silently produce empty files due to buffering issues with large base64 payloads.

    ```bash
    # Step 1: Call the API and save the full response
    curl -s -X POST \
      "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image:generateContent" \
      -H "x-goog-api-key: $GEMINI_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "contents": [{"parts": [{"text": "<PROMPT>"}]}],
        "generationConfig": {
          "responseModalities": ["IMAGE"],
          "imageConfig": {
            "aspectRatio": "16:9"
          }
        }
      }' -o /tmp/gemini-response.json

    # Step 2: Extract and decode the image
    jq -r '.candidates[0].content.parts[] | select(.inlineData) | .inlineData.data' /tmp/gemini-response.json | base64 -d > "src/assets/media/passages/<slug>.png"
    ```

    **Important notes on the curl command:**
    - **Always use the two-step approach** (save to temp file, then extract). Do NOT pipe curl directly through jq — large base64 responses can produce empty output files.
    - The `$GEMINI_API_KEY` environment variable must be set (same key used by nano-banana)
    - The `aspectRatio` in `imageConfig` is what actually controls the output dimensions — do NOT put aspect ratio hints in the prompt text (causes black letterbox bars)
    - `responseModalities` must be `["IMAGE"]` (not `["TEXT", "IMAGE"]`) to get a pure image response
    - The response contains the image as base64 in `candidates[0].content.parts[].inlineData.data`
    - If the API returns an error, inspect with: `jq . /tmp/gemini-response.json`

4. Verify the image was generated successfully:
    - Check the file exists and has non-zero size
    - If the file is empty or missing, the API call failed — re-run the curl without the jq pipe to see the error response

## Step 9: Show the Image to the User

Read and display the generated image at `src/assets/media/passages/<slug>.png` to the user.

Ask via `AskUserQuestion`:

> Here's the generated illustration for "[Passage Title]". What would you like to do?

Options: "Insert into passage" / "Regenerate with same prompt" / "Edit prompt and regenerate" / "Cancel"

- **Insert into passage**: proceed to Step 10.
- **Regenerate with same prompt**: go back to Step 8 (run the curl command again).
- **Edit prompt and regenerate**: ask user for their changes, then go back to Step 8.
- **Cancel**: The image file remains in `src/assets/media/passages/` but the `.twee` file is not modified. Inform the user of the file location.

## Step 10: Insert Image into the `.twee` File

1. Construct the image markup:

    ```
    <div class="passage-illustration">[img[<brief alt text>|passages/<slug>.png]]</div>
    ```

    The alt text should be a concise (5-15 word) description of what the image depicts.

2. **Determine placement:**
    - In Step 6, you identified which paragraph contains the key moment. Insert the image **on its own line directly above that paragraph**.
    - If the key moment is in the very first paragraph (or no clear paragraph match), place the image at the top of the passage content, **after** any leading `<<set>>`, `<<nobr>>`, or other macro-only lines.
    - Always ensure the image line is separated from surrounding prose by blank lines.

3. **If replacing an existing image** (from Step 4): Find and replace the entire `<div class="passage-illustration">...</div>` line with the new one.

4. Use the Edit tool to make the change in the `.twee` file.

**Example — new image at top of passage:**

```twee
:: The Score [opening cage]
<div class="passage-illustration">[img[A rat peers through cage bars at a distant pizza box|passages/the-score.png]]</div>

The moment arrives the way all great moments do...
```

**Example — image placed mid-passage above a key moment:**

```twee
:: Shadows and Silence [stealth floor]
<<set $stealth += 1>>
You flatten yourself against the baseboard and consider your options.

<div class="passage-illustration">[img[A tiny rat pressed flat against the baseboard in deep shadow|passages/shadows-and-silence.png]]</div>

The hallway stretches before you like a canyon...
```

## Step 11: Add CSS if Missing

Check `src/assets/app/styles/story-theme.scss` for the `.passage-illustration` class.

If not found, append this CSS block to the end of the file:

```scss
// ===========================================
// Passage Illustrations
// ===========================================

.passage-illustration {
    margin: 1em 0 1.5em;
    line-height: 0;

    img {
        display: block;
        width: 100%;
        max-width: 100%;
        height: auto;
        border-radius: 8px;
        box-shadow: 0 2px 12px rgba(58, 52, 41, 0.15);
        animation: img-fade-in 0.4s ease-out;
    }
}

@keyframes img-fade-in {
    from {
        opacity: 0;
        transform: translateY(4px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@media screen and (max-width: 480px) {
    .passage-illustration img {
        border-radius: 6px;
    }
    .passage-illustration {
        margin: 0.75em 0 1em;
    }
}
```

**Note:** The shadow color `rgba(58, 52, 41, 0.15)` is derived from the story theme's `--theme-text` value. If the story has a different theme, adjust the shadow color to match. Read the theme file's CSS custom properties and use the appropriate text color.

## Step 12: Verify Build

Run the build:

```bash
npm run build
```

- **If the build succeeds**: Report to the user:

    > Illustration added successfully.
    >
    > - **Image**: `src/assets/media/passages/<slug>.png`
    > - **Passage**: `<filename>.twee` :: `<Passage Title>`
    > - **Build**: Passing

- **If the build fails**: Read the error output. The most likely causes are:
    - Malformed markup in the `.twee` file — fix the insertion
    - SCSS syntax error if CSS was added — fix the CSS
    - If unfixable, revert the `.twee` change and report the error to the user

## Important Notes

- **Never generate an image without user approval of the prompt.** This is the primary cost control mechanism.
- **Character appearance must come from visual-style.md**, not from re-reading lore files each time. This ensures visual consistency.
- **The Prompt Suffix is critical** — it anchors the art style, perspective, and mood across all passage images.
- **One image per passage.** If a passage already has an image, the user must explicitly choose to replace it.
- **Alt text is required** for accessibility. Keep it concise and descriptive.

## Known Gemini Pitfalls

- **"Hooded" is misinterpreted.** The term "hooded rat" (a breed with dark markings on the head/shoulders) causes Gemini to draw a rat wearing a literal fabric hood. Always describe the markings explicitly instead: "dark markings over head and shoulders, white body".
- **Aspect ratio in prompt text causes letterboxing.** Phrases like "wide landscape format", "16:9 widescreen composition" in the prompt text cause Gemini to render black bars inside the image. The `imageConfig.aspectRatio` API parameter is the only reliable way to control dimensions — never duplicate it in the prompt.
- **"No watermark" is unreliable.** Including "no watermark, no signature" in the prompt reduces but does not eliminate watermarks. Gemini may still render them stochastically. If a watermark appears, simply regenerate — the next attempt often comes out clean.
- **Piping curl directly can produce empty files.** Large base64 API responses can cause silent failures when piped through `jq | base64 -d`. Always save the response to a temp file first, then extract in a second step.
