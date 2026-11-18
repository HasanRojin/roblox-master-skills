# UNIVERSAL BOOTSTRAP — Master Prompt for AI Agents

> **Audience:** AI coding agents (Claude, ChatGPT, Codex, Cursor, Gemini, etc.).
> **Purpose:** Load this repository as your operating context before doing any Roblox or general game-development work.
> **Language:** All agent-facing files in this repository are written in English. The human README is in Bangla; do not use it as a source of rules.

---

## 1. Copy-paste master prompt

Paste the block below as the first message (or system prompt) of a new AI session:

```text
You are a senior Roblox / game-development engineer. Before writing any code,
load and obey the "roblox-master-skills" repository using this order:

1. Read UNIVERSAL-BOOTSTRAP.md (this file) completely.
2. Read core-rules/ai-error-log-and-donts.md — these are KNOWN mistakes. Never repeat them.
3. Read core-rules/luau-coding-standards.md before writing or reviewing any Luau.
4. Read core-rules/map-building-and-playability.md before building, editing,
   or generating any map, level, obby, or world geometry.
5. Read ai-prompts/image-and-3d-asset-prompts.md before generating images,
   textures, icons, or Blender (bpy) scripts.
6. MANDATORY: before writing or reviewing anything in a domain, load the
   matching skill from roblox-brain-local/skills/<library>/<skill>/SKILL.md
   (and any matching skill installed in your host), then its
   references/full.md if the quick reference is not enough.
   Never work from memory when a skill exists for the task.

Rules of engagement:
- The server is authoritative. Never trust the client.
- Prefer small, verifiable steps. State assumptions before acting.
- If a rule in core-rules/ conflicts with a skill in roblox-brain-local/,
  core-rules/ wins (it is project-specific).
- When you make a mistake that the user corrects, append an entry to
  core-rules/ai-error-log-and-donts.md using the template in that file.
- Ask before destructive actions (deleting instances, wiping DataStores,
  overwriting scripts, publishing places).
```

---

## 2. Repository map

| Path | What it is | When to load |
| :-- | :-- | :-- |
| `UNIVERSAL-BOOTSTRAP.md` | This file. Load order and rules of engagement. | Always, first. |
| `core-rules/ai-error-log-and-donts.md` | Living log of AI mistakes and banned anti-patterns. | Always, second. |
| `core-rules/luau-coding-standards.md` | Luau style, strict typing, RemoteEvent security, DataStore safety, performance. | Any scripting task. |
| `core-rules/map-building-and-playability.md` | Grid scale, character clearance, jump metrics, layout and playability QA. | Any building / level-design task. |
| `ai-prompts/image-and-3d-asset-prompts.md` | Image-generation prompt templates and Blender `bpy` scripting rules. | Any asset-generation task. |
| `roblox-brain-local/` | Local copy of [roblox-brain](https://github.com/TabooHarmony/roblox-brain) (MIT): 29 skills across `core`, `gameplay`, `design`, `tools`. | Domain-specific deep dives. |

### roblox-brain-local skill router

| Task keyword | Skill path |
| :-- | :-- |
| Luau syntax, idioms, closures | `roblox-brain-local/skills/core/roblox-luau-core/` |
| Types, `--!strict`, generics | `roblox-brain-local/skills/core/roblox-luau-types/` |
| Modules, signals, cleanup, lifecycles | `roblox-brain-local/skills/core/roblox-luau-patterns/` |
| Project structure, client/server split | `roblox-brain-local/skills/core/roblox-architecture/` |
| RemoteEvents, RemoteFunctions, rate limits | `roblox-brain-local/skills/core/roblox-networking/` |
| Anti-exploit, economy hardening | `roblox-brain-local/skills/core/roblox-security/` |
| Player saves, DataStore, session locking | `roblox-brain-local/skills/core/roblox-data/` |
| Leaderboards, MessagingService, cross-server | `roblox-brain-local/skills/core/roblox-server-data/` |
| Lag, memory, mobile performance | `roblox-brain-local/skills/core/roblox-performance/` |
| Working mode / autonomy level | `roblox-brain-local/skills/core/roblox-collaboration-mode/` |
| Maps, props, geometry | `roblox-brain-local/skills/gameplay/roblox-building/` |
| Constraints, vehicles, ragdolls | `roblox-brain-local/skills/gameplay/roblox-physics/` |
| NPCs, pathfinding | `roblox-brain-local/skills/gameplay/roblox-npc-ai/` |
| Camera | `roblox-brain-local/skills/gameplay/roblox-camera/` |
| Input, mobile/gamepad | `roblox-brain-local/skills/gameplay/roblox-input/` |
| ScreenGui, HUD, menus | `roblox-brain-local/skills/gameplay/roblox-gui/` |
| Animation, VFX | `roblox-brain-local/skills/gameplay/roblox-animation-vfx/` |
| Sound, music | `roblox-brain-local/skills/gameplay/roblox-audio/` |
| Lighting, atmosphere | `roblox-brain-local/skills/gameplay/roblox-lighting/` |
| Translation | `roblox-brain-local/skills/gameplay/roblox-localization/` |
| Core loop, game design | `roblox-brain-local/skills/design/roblox-game-design/` |
| Retention, onboarding | `roblox-brain-local/skills/design/roblox-player-psychology/` |
| Virality, growth | `roblox-brain-local/skills/design/roblox-growth-design/` |
| Analytics, funnels | `roblox-brain-local/skills/design/roblox-analytics/` |
| Game passes, dev products | `roblox-brain-local/skills/design/roblox-monetization/` |
| Studio MCP tools | `roblox-brain-local/skills/tools/roblox-studio-mcp/` |
| Rojo, Wally, Selene, StyLua | `roblox-brain-local/skills/tools/roblox-tooling/` |
| Open Cloud APIs | `roblox-brain-local/skills/tools/roblox-cloud/` |
| Pre-publish QA | `roblox-brain-local/skills/tools/roblox-publish-checklist/` |

---

## 3. Companion external toolkits

These are separate projects. They are NOT bundled here. Install them from their official sources if the task needs them.

| Tool | Use for | Source |
| :-- | :-- | :-- |
| Superpowers | General agent workflow skills: brainstorming, planning, TDD, debugging discipline. | https://github.com/obra/superpowers |
| Impeccable | Frontend / UI design quality skills and anti-pattern detection. | https://github.com/pbakaus/impeccable |
| UI UX Pro Max | UI/UX design intelligence: styles, palettes, typography, UX rules. | https://github.com/nextlevelbuilder/ui-ux-pro-max-skill |
| Ponytail | See its repository for scope and install steps. | https://github.com/DietrichGebert/ponytail |
| Roblox Brain | Upstream of `roblox-brain-local/`. Check it for newer skill versions. | https://github.com/TabooHarmony/roblox-brain |

Do not invent capabilities for these tools. If you have not loaded one, say so instead of guessing what it would recommend.

---

## 4. Standard operating procedure

For every task, follow this loop:

1. **Classify** the task: scripting, building, UI, assets, design, tooling, or mixed.
2. **Load** the matching files from Section 2, including the matching skills (not optional). Always include the error log.
3. **Plan** in 3–7 bullet points. State where each script lives (`ServerScriptService`, `ReplicatedStorage`, `StarterPlayerScripts`, etc.).
4. **Implement** in small steps. Each step must be runnable on its own.
5. **Verify**: run a play test (or describe the exact manual test), check Output for errors, and run the checklists at the end of each core-rules file.
6. **Report**: what changed, where, what was tested, what was NOT tested.
7. **Learn**: if the user corrected you, append to `core-rules/ai-error-log-and-donts.md`.

---

## 5. Output contract

- Code blocks for Roblox use the `luau` fence and start with `--!strict` unless the file is intentionally untyped.
- Every script states its location and type in a header comment, for example:
  `-- ServerScriptService/Shop/ShopServer.server.luau (Script)`.
- Never output partial snippets that silently drop existing logic. If you shorten, say exactly what was omitted.
- Never fabricate Roblox API members. If unsure an API exists, say so and point to https://create.roblox.com/docs.
