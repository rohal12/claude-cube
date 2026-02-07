<img src="src/assets/media/favicon.png" width="120" align="right" />

# Claude-Cube: Interactive Story Generator

An AI-powered interactive fiction generator that uses Claude's multi-agent framework to create complete SugarCube stories through collaborative AI agents.

## 🎨 Features

- **AI-Powered Story Generation** — Multi-agent pipeline creates complete interactive stories
- **Claude Agentic Framework** — Specialized agents for story design, writing, styling, and review
- **Automatic Tweego & SugarCube Setup** — All dependencies installed automatically
- **Live Development Server** — Hot reload with Browser-Sync
- **Custom Theming** — AI-generated SCSS themes based on story genre and mood
- **Production Builds** — Optimized output ready for distribution

## 🤖 Claude Agentic Framework

This project uses Claude's multi-agent system to generate interactive stories through specialized AI agents:

### Story Creation Agents

- **Story Architect** — Designs overall story structure, themes, and narrative arcs
- **Plot Architect** — Creates detailed passage graphs and story flow
- **Passage Writer** — Writes individual passage prose with SugarCube macros
- **Lore Keeper** — Maintains consistency for characters, world, and items
- **Story Stylist** — Generates custom SCSS themes matching story tone
- **Story Reviewer** — Reviews and provides feedback on story quality
- **Story Tester** — Tests story functionality and link validation
- **SugarCube Expert** — Ensures proper macro usage and conventions

### Agent Workflow

1. **Design Phase** — Story Architect creates the story bible with concept, setting, characters
2. **Structure Phase** — Plot Architect designs passage graph and story flow
3. **Writing Phase** — Passage Writer creates prose for each passage
4. **Styling Phase** — Story Stylist generates custom theme
5. **Review Phase** — Story Reviewer and Tester validate quality
6. **Compilation** — Tweego compiles `.twee` files into playable HTML

All agents communicate through structured files in `story-workspace/` and follow conventions defined in `CLAUDE.md`.

## 🗃 Tech Stack

Built on **[nijikokun/sugarcube-starter](https://github.com/nijikokun/sugarcube-starter)** with AI agent enhancements:

- [Vite](https://vitejs.dev/) — Fast build tooling
- [TypeScript](https://www.typescriptlang.org/)
- [Sass](https://sass-lang.com/) with [Modern CSS Support](https://github.com/csstools/postcss-preset-env#readme)
- [Browser-Sync](https://browsersync.io/) — Live reloading
- [SugarCube 2.37.3](https://www.motoslave.net/sugarcube/2/) — Interactive fiction engine
- [Tweego 2.1.1](https://www.motoslave.net/tweego/) — Story compiler

## ℹ Requirements

- [Node.js](https://nodejs.org/en/) 18+
- [Claude CLI](https://code.claude.com/docs/en/setup) (Claude Code)
- Any code editor (VSCode, Cursor, or similar)

## 🚀 Getting Started

### 1. Install Claude CLI

Install Claude Code using one of the following methods:

**macOS, Linux, WSL (Recommended):**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell:**

```powershell
irm https://claude.ai/install.ps1 | iex
```

**Alternative installations:**

- **Homebrew**: `brew install --cask claude-code`
- **WinGet**: `winget install Anthropic.ClaudeCode`

For detailed installation instructions and troubleshooting, see the [official Claude Code setup guide](https://code.claude.com/docs/en/setup).

### 2. Authenticate Claude

After installation, authenticate using one of these options:

**For individuals:**

- **Claude Pro or Max plan** (recommended) — Subscribe at [claude.ai/pricing](https://claude.ai/pricing)
- **Claude Console** — Set up billing at [console.anthropic.com](https://console.anthropic.com)

**For teams:**

- **Claude for Teams or Enterprise** — Contact [claude.ai/pricing](https://claude.ai/pricing)

Run `claude` in your terminal and follow the authentication prompts.

### 3. Clone and Set Up the Project

```bash
npx degit https://github.com/rohal12/claude-cube.git <project-name>
cd <project-name>
npm install
```

This will install all Node.js dependencies and set up the project.

### 4. Generate Your First Story

Start Claude CLI in your project directory:

```bash
claude
```

Once Claude starts, run the interactive story creation skill:

```
/create-story
```

The skill will guide you through:

1. **Story Concept** — Define genre, setting, tone, and themes
2. **Character Creation** — Design protagonists and supporting characters
3. **Story Structure** — Choose narrative style (linear, branching, hub-based)
4. **Generation** — AI agents collaborate to create your story
5. **Review & Iteration** — Test and refine the generated content

The agents will create all necessary files in `story-workspace/` and compile the final story to `src/story/`.

### 5. Preview Your Story

Start the development server:

```bash
npm start
```

Open your browser to `http://localhost:4321` to see your generated story in action with live reload.

### 6. Build for Production

When ready to publish:

```bash
npm run build
```

Your compiled story will be in `dist/index.html` — ready to host anywhere!

**New to SugarCube?**

- Check out the [SugarCube Documentation](https://www.motoslave.net/sugarcube/2/docs/)
- Read `CLAUDE.md` for agent-specific conventions and guidelines

## 👩‍💻 Commands

| Command                  | Description                               |
| ------------------------ | ----------------------------------------- |
| `npm start`              | Start development server with live reload |
| `npm run dev`            | Same as `npm start`                       |
| `npm run build`          | Production build to `dist/`               |
| `npm run tweego`         | Run tweego manually                       |
| `npm run tweego:install` | Install/reinstall tweego                  |

## 📁 Directory Structure

```
.claude/                    # Claude AI configuration
├── agents/                 # AI agent definitions
│   ├── story-architect.md
│   ├── plot-architect.md
│   ├── passage-writer.md
│   ├── lore-keeper.md
│   ├── story-stylist.md
│   ├── story-reviewer.md
│   ├── story-tester.md
│   └── sugarcube-expert.md
└── skills/                 # AI skills for story generation
    ├── create-story/       # Interactive story creation
    └── playwright/         # Browser testing capabilities

.build/                     # Build scripts
├── dev.ts                  # Development server
└── tweego.ts               # Tweego installer & runner

src/                        # Source files
├── assets/
│   ├── app/                # Your JS/TS and SCSS
│   │   ├── index.ts
│   │   └── styles/
│   │       ├── index.scss
│   │       └── story-theme.scss  # AI-generated theme
│   ├── fonts/              # Custom fonts
│   ├── media/              # Images and videos
│   └── vendor/             # Third-party scripts
├── story/                  # Twine .twee files (compiled by Tweego)
│   ├── StoryData.twee
│   ├── StoryInit.twee
│   ├── StoryTitle.twee
│   ├── Act1.twee
│   ├── Act2.twee
│   └── Act3.twee
└── head-content.html

story-workspace/            # Agent working files (NOT compiled)
├── story-bible.md          # Story design document
├── passage-graph.json      # Passage structure
├── state.json              # Pipeline progress
├── lore/                   # Character/world/item docs
├── passages/               # Individual passage markdown
└── review/                 # Quality reports

dist/                       # Compiled output (auto-generated)
.tweego/                    # Tweego installation (auto-generated)
```

## 🧠 How the AI Framework Works

### Agent Communication

Agents don't directly modify story files. Instead, they work through a structured pipeline:

1. **Story Bible** (`story-workspace/story-bible.md`) — Master design document containing:
    - Genre, setting, tone
    - Character profiles
    - World lore
    - Style guide (POV, tense, vocabulary)

2. **Passage Graph** (`story-workspace/passage-graph.json`) — Story structure:
    - Passage IDs and titles
    - Choice links and consequences
    - Branching paths and convergence points
    - Variable conditions

3. **Passage Markdown** (`story-workspace/passages/`) — Individual passage files:
    - Prose with SugarCube macros
    - Player choices
    - Variable tracking
4. **Lore Documents** (`story-workspace/lore/`) — Reference material:
    - Character details and relationships
    - World facts and rules
    - Item descriptions

5. **Review Reports** (`story-workspace/review/`) — Quality assurance:
    - Story coherence checks
    - Link validation
    - Macro testing

### Compilation Process

When agents complete their work, the story is compiled:

1. Agents write passage markdown files to `story-workspace/passages/`
2. A compilation agent converts markdown to `.twee` format in `src/story/`
3. Tweego compiles `.twee` files into HTML
4. Vite bundles assets and creates the final build

### Manual Editing

You can edit any generated files:

- **Story content**: Edit `.twee` files in `src/story/`
- **Styling**: Modify `src/assets/app/styles/story-theme.scss`
- **Variables**: Update `src/story/StoryInit.twee`

Just remember: changes to `src/story/` are independent of `story-workspace/`. If you regenerate with AI agents, manual edits will be overwritten.

## 🙋‍♂️ How To

<details>
<summary>How do I iterate on a generated story?</summary>
<p>

You have several options:

1. **Regenerate from scratch**: Run `/create-story` again with a new concept
2. **Manual edits**: Edit `.twee` files in `src/story/` directly
3. **Partial regeneration**: Ask Claude to regenerate specific passages or acts
4. **Style updates**: Ask the Story Stylist agent to revise the theme

Example prompt for iteration:

```
"Regenerate Act 2 with more emphasis on the mystery subplot"
```

</p>
</details>

---

<details>
<summary>How do I customize agent behavior?</summary>
<p>

Agent definitions are in `.claude/agents/`. Each agent has a markdown file defining:

- Personality and expertise
- Task-specific instructions
- Output format requirements
- Quality standards

To customize, edit the relevant agent file (e.g., `.claude/agents/passage-writer.md`) to adjust tone, style preferences, or behavior.

</p>
</details>

---

<details>
<summary>How do I add new story variables?</summary>
<p>

1. Edit `src/story/StoryInit.twee`
2. Add your variables with `<<set>>` macros:

```twee
:: StoryInit
<<set $playerName = "">>
<<set $reputation = 0>>
<<set $hasKey = false>>
```

All story variables must start with `$` and be initialized here before use.

</p>
</details>

---

<details>
<summary>How do I disable Debug View?</summary>
<p>

Debug View is automatically enabled in development and disabled in production builds (`npm run build`).

To disable it in development, create `src/story/PassageReady.twee`:

```js
:: PassageReady
<<run DebugView.disable()>>
```

</p>
</details>

---

<details>
<summary>How should I initialize variables?</summary>
<p>

Use the [`StoryInit`](https://www.motoslave.net/sugarcube/2/docs/#special-passage-storyinit) passage in `src/story/Start.twee`:

```ejs
:: StoryInit
<<set $health = 100>>
<<set $maxHealth = 100>>

:: Start

HP: <<= $health>> / <<= $maxHealth>>
```

</p>
</details>

---

<details>
<summary>How do I install macros?</summary>
<p>

Macros scripts and styles go into `src/assets/vendor`

</p>
</details>

---

<details>
<summary>How do I link to media files?</summary>
<p>

To reference images at `src/assets/media/<asset_path>`:

- `src/assets/media/favicon.png` → `media/favicon.png`

Example in HTML:

```html
<link rel="icon" type="image/png" href="media/favicon.png" />
```

</p>
</details>

---

<details>
<summary>How do I add Google Analytics?</summary>
<p>

Paste into [`src/head-content.html`](./src/head-content.html):

```html
<script
    async
    src="https://www.googletagmanager.com/gtag/js?id=YOUR_TAG_HERE"
></script>
```

Replace `YOUR_TAG_HERE` with your Google Analytics ID.

</p>
</details>

## 🤝 Helpful Resources

**Project Documentation**

- `CLAUDE.md` — Complete guide to the multi-agent framework
- `.claude/agents/` — Individual agent definitions and capabilities
- `.claude/skills/` — Available AI skills for story generation

**Claude CLI & Tools**

- [Claude Code Documentation](https://code.claude.com/docs/en/setup) — Installation and setup
- [Anthropic Claude](https://www.anthropic.com/claude) — Claude AI platform
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) — Connect external tools to Claude

**SugarCube Resources**

- [SugarCube Documentation](https://www.motoslave.net/sugarcube/2/docs/)
- [Tweego Documentation](https://www.motoslave.net/tweego/docs/)
- [Chapel's Custom Macro Collection](https://github.com/ChapelR/custom-macros-for-sugarcube-2)
- [HiEv SugarCube Sample Code](https://twine.hiev-heavy-ind.com/)

**Interactive Fiction**

- [IFDB - Interactive Fiction Database](https://ifdb.org/)
- [Twine Community](https://twinery.org/community)
- [Twine Games Discord](https://discord.com/invite/n5dJvPp) — Active community for help and discussion
- [IntFiction Forum](https://intfiction.org/)

## 💜 Acknowledgements

This project builds upon:

- **[SugarCube Starter](https://github.com/nijikokun/sugarcube-starter)** by [@nijikokun](https://github.com/nijikokun) — Modern starter kit with Vite, TypeScript, and Sass
- **SugarCube** by Thomas M. Edwards — Interactive fiction engine
- **Tweego** by Thomas M. Edwards — Story compiler
- **Claude** by Anthropic — AI agent framework and Claude Code CLI

**Community Thanks:**

- [Twine Games Discord](https://discord.com/invite/n5dJvPp) — Supportive community for interactive fiction creators
- The interactive fiction and AI development communities for their inspiration and tools

## 📝 License

Licensed under the MIT License.
