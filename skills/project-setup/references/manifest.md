# Recommended skill manifest

Each row: skill name → source path inside this repo → destination folder name in the personal skills folder.
Destination base: `~/.claude/skills/`. Copy the **whole folder**, including subfolders such as `references/` and `scripts/`.

## Roblox Brain (`roblox-brain-local/`)

| Skill | Source | Note |
| :-- | :-- | :-- |
| roblox-networking | roblox-brain-local/skills/core/roblox-networking | remotes, validation, rate limits |
| roblox-server-data | roblox-brain-local/skills/core/roblox-server-data | OrderedDataStore, MessagingService |
| roblox-luau-core | roblox-brain-local/skills/core/roblox-luau-core | Luau language traps |
| roblox-luau-patterns | roblox-brain-local/skills/core/roblox-luau-patterns | modules, signals, cleanup |
| roblox-building | roblox-brain-local/skills/gameplay/roblox-building | maps, props, geometry |
| roblox-gui | roblox-brain-local/skills/gameplay/roblox-gui | ScreenGui, HUD, menus |
| roblox-input | roblox-brain-local/skills/gameplay/roblox-input | keyboard, mobile, gamepad |
| roblox-camera | roblox-brain-local/skills/gameplay/roblox-camera | camera |
| roblox-physics | roblox-brain-local/skills/gameplay/roblox-physics | constraints, vehicles, ragdolls |
| roblox-animation-vfx | roblox-brain-local/skills/gameplay/roblox-animation-vfx | animation, effects |
| roblox-lighting | roblox-brain-local/skills/gameplay/roblox-lighting | lighting, atmosphere |
| roblox-audio | roblox-brain-local/skills/gameplay/roblox-audio | sound |
| roblox-npc-ai | roblox-brain-local/skills/gameplay/roblox-npc-ai | NPC behavior |
| roblox-studio-mcp | roblox-brain-local/skills/tools/roblox-studio-mcp | Studio MCP workflow |
| roblox-tooling | roblox-brain-local/skills/tools/roblox-tooling | Rojo, Wally, Selene, StyLua |
| roblox-publish-checklist | roblox-brain-local/skills/tools/roblox-publish-checklist | pre-publish QA |

**Skip (already provided by the Roblox plugin):** roblox-security, roblox-data, roblox-performance, roblox-luau-types, roblox-architecture. Installing them too would duplicate the same guidance.

## Superpowers (`skills/third-party/superpowers/`)

| Skill | Source |
| :-- | :-- |
| brainstorming | skills/third-party/superpowers/brainstorming |
| writing-plans | skills/third-party/superpowers/writing-plans |
| systematic-debugging | skills/third-party/superpowers/systematic-debugging |
| verification-before-completion | skills/third-party/superpowers/verification-before-completion |

**Skip:** `using-superpowers` (it asks for a skill check before every reply, which adds cost to every turn).

## Ponytail (`skills/third-party/ponytail/`)

| Skill | Source |
| :-- | :-- |
| ponytail | skills/third-party/ponytail/ponytail |
| ponytail-review | skills/third-party/ponytail/ponytail-review |

## Impeccable (`skills/third-party/impeccable/`)

| Skill | Source |
| :-- | :-- |
| impeccable | skills/third-party/impeccable |

## UI UX Pro Max (`skills/third-party/ui-ux-pro-max/`)

| Skill | Source |
| :-- | :-- |
| ui-ux-pro-max | skills/third-party/ui-ux-pro-max/ui-ux-pro-max |
| design-system | skills/third-party/ui-ux-pro-max/design-system |

**Note:** `ui-ux-pro-max` reads `data/` and `scripts/` from its own folder. When copying it, copy the whole `ui-ux-pro-max/` folder (the parent with `ui-ux-pro-max/` and `design-system/` inside), not just one subfolder, so the shared data stays reachable. Copy `design-system` in the same way.

## Already installed (do not copy again)

- model-reference-sheet (from `skills/model-reference-sheet/`)
