# CLAUDE.md — Roblox & Game Development Master Skills

You are working in a repository of rules and skills for Roblox and game development.
Read this file at the start of every session. Read the other files only when the task needs them.

## Load order (cheap first)

1. `UNIVERSAL-BOOTSTRAP.md` — the full map of this repo. Read it once per session.
2. `core-rules/ai-error-log-and-donts.md` — known mistakes. Read it before any code or map work.
3. Then only the files and skills that match the task (table below).

## Task → what to load

| When the task is… | Load |
| :-- | :-- |
| Any Luau / Roblox script | `core-rules/luau-coding-standards.md` + skill `roblox-dev:roblox-security` (remotes) and `roblox-dev:luau-strict-typing` (types) |
| Saving player data, DataStore, purchases | `roblox-dev:roblox-datastores` + `core-rules/luau-coding-standards.md` §5 |
| Lag, memory, frame loops, networking cost | `roblox-dev:roblox-performance` |
| Building maps, obbies, levels, geometry | `core-rules/map-building-and-playability.md` + `roblox-brain-local/skills/gameplay/roblox-building/` |
| Remotes / client–server design | `roblox-brain-local/skills/core/roblox-networking/` |
| Model reference sheet, asset image, 3D prompt | `skills/model-reference-sheet/SKILL.md` |
| Blender `bpy` scripts, image prompts | `ai-prompts/image-and-3d-asset-prompts.md` |
| UI / HUD / frontend design critique or polish | `skills/third-party/impeccable/SKILL.md` |
| UI/UX design system, palettes, typography, styles | `skills/third-party/ui-ux-pro-max/ui-ux-pro-max/SKILL.md` (then `design`, `ui-styling` only if needed) |
| Small, minimal code change (any coding task) | `skills/third-party/ponytail/ponytail/SKILL.md` |
| Review a change before merging | `skills/third-party/ponytail/ponytail-review/SKILL.md` |
| Audit a whole repo for bugs and risk | `skills/third-party/ponytail/ponytail-audit/SKILL.md` |
| Large feature or multi-step plan | `skills/third-party/superpowers/writing-plans/SKILL.md` and `brainstorming/SKILL.md` |
| Bug or failing test | `skills/third-party/superpowers/systematic-debugging/SKILL.md` |
| Claiming work is done | `skills/third-party/superpowers/verification-before-completion/SKILL.md` |

## Session flow (talk → plan → work)

Most sessions start with discussion, not code. Follow these phases:

1. **Discussion:** Read this file and `UNIVERSAL-BOOTSTRAP.md`. Identify the task type from the table above. Do not load a skill's full text yet. Name the skills you expect to need in the plan.
2. **Plan:** Once the user agrees to a plan, load only the skills in that plan, then start. If the user asks which skills they need or whether they are installed, load `skills/project-setup/SKILL.md`.
3. **Work:** Use only the skills named in the agreed plan. Load a new one only if the task changes. Log any correction in `core-rules/ai-error-log-and-donts.md`.

If the user asks for the list of recommended skills or to install them, use `skills/project-setup/SKILL.md`. It asks before copying anything.

## Token-saving rules

- Read a skill's `SKILL.md` first. Open its `references/` or extra files only when the quick section is not enough.
- Do not load several skills for a small task. Pick the one row that matches, plus `ai-error-log-and-donts.md`.
- Do **not** install `using-superpowers` as an always-on skill. Its description asks for a skill check before every reply, which adds cost to every turn.
- For a single-line question, answer directly without loading any skill.

## Hard rules (from core-rules; these override everything else)

- The server is authoritative. Never trust client values for price, damage, currency or position.
- Every remote is validated (type → range/NaN → permission → plausibility) and rate-limited.
- Never write DataStore calls without `pcall`, retry, `UpdateAsync`, session locking and `BindToClose`.
- Never invent Roblox API members. If unsure, say so.
- Never delete or overwrite the user's existing scripts, builds or assets without asking.
- Report what was tested and what was not. Do not claim a play test you did not run.
- When the user corrects a mistake, log it in `core-rules/ai-error-log-and-donts.md`.

## Third-party skills

Skills under `skills/third-party/` are copies of the upstream projects' skill folders (MIT for Ponytail, Superpowers and UI UX Pro Max; Apache-2.0 for Impeccable, see its `NOTICE.md`). Their hooks, CLI tools and plugin code are not included. Only the skill text is included, so these tools work as instructions, not as installed software.
