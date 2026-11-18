# Map Building and Playability Rules

> Agent-facing rules for building maps, levels, obbies, lobbies, and world geometry in Roblox.
> Deep reference: `roblox-brain-local/skills/gameplay/roblox-building/`, `roblox-physics/`, `roblox-lighting/`.
> All measurements are in **studs**. Values assume the default R15 avatar and default `StarterPlayer` settings (`WalkSpeed = 16`, `JumpHeight = 7.2`, `UseJumpPower = false`). If the game changes these, recompute the jump metrics in Section 3.

---

## 1. Grid and scale

| Rule | Value |
| :-- | :-- |
| Base build grid | **1 stud** for detail, **4 studs** for structural pieces (walls, floors, platforms). |
| Rotation snap | 15° or 45° (90° for structural). |
| Wall thickness | 1 stud minimum (0.5 only for decorative trim). |
| Floor thickness | 1 stud minimum. Never use zero-thickness or paper-thin floors — characters can clip through. |
| Story height (floor to floor) | 14–16 studs. |
| Real-world reference | 1 stud ≈ 0.28 m; R15 character ≈ 5 studs tall ≈ human height. Build doors, stairs, furniture to character scale, not to "realistic meters". |
| Positions | Snap part positions to the grid. Avoid values like `12.3471`. Seams cause Z-fighting and snag points. |
| Z-fighting | Never place two coplanar faces at the same position. Offset by ≥ 0.01 stud or merge the parts. |

---

## 2. Player clearance

| Element | Minimum | Recommended |
| :-- | :-- | :-- |
| Character height (R15) | ~5 studs | — |
| Character width | ~2 studs (shoulders), ~4 studs with arms/accessories | — |
| Doorway | 4 w × 7 h | 6 w × 9 h |
| Corridor width | 6 studs | 8–10 studs (2+ players pass) |
| Ceiling height (indoor) | 9 studs | 12+ studs (camera needs room) |
| Stair step | rise ≤ 1 stud, run ≥ 1 stud | rise 1, run 2 |
| Ramp angle | ≤ 45° (walkable up to `Humanoid.MaxSlopeAngle`, default 89°, but steep slopes feel bad) | ≤ 30° |
| Gap between walls (to avoid getting stuck) | ≥ 4 studs or fully closed (0) | Never 1–3 studs |
| Spawn area | 12 × 12 studs clear per `SpawnLocation` | Multiple spawns for >10 players |

Camera clearance: third-person camera sits ~10–15 studs behind the player. Tight rooms must allow the camera to pull in without clipping into geometry; mark decorative clutter `CanQuery = false` if it blocks the camera unhelpfully.

---

## 3. Jump and movement metrics (defaults)

| Metric | Safe (everyone) | Hard | Impossible |
| :-- | :-- | :-- | :-- |
| Vertical jump onto ledge | ≤ 6 studs | 6–7 | > 7.2 |
| Horizontal gap, same height | ≤ 10 studs | 10–13 | > 14 |
| Horizontal gap, landing 4 studs higher | ≤ 6 studs | 6–8 | > 9 |
| Platform size for landing | ≥ 4 × 4 | 2 × 2 | < 1.5 × 1.5 |

- Test every jump in play mode. Never mark a jump "done" from math alone.
- Obby difficulty must ramp gradually. Put checkpoints every 3–6 obstacles.
- Kill bricks must be visually distinct (bright red/neon) and never placed where a player can land without warning.
- Moving platforms: move with `PrismaticConstraint`/`AlignPosition` or `TweenService` on an anchored part plus server-side handling; test that players stay on them.

---

## 4. Layout and navigation

- **Readable landmarks:** every area needs one visually unique landmark visible from a distance.
- **Main path first:** build the critical path (spawn → core loop → reward) before decoration.
- **Width of main path:** 12+ studs in lobbies and hubs.
- **No dead ends** without a reward or reason.
- **Lead the eye** with lighting, color contrast and leading lines toward objectives.
- **Boundaries:** use invisible walls (`Transparency = 1`, `CanCollide = true`) plus a visible cue (fence, cliff, water). Add a fall-void catcher or `FallenPartsDestroyHeight` respawn.
- **Mobile players** have a smaller screen and less precise input: avoid pixel-perfect jumps on the critical path, keep important UI-world prompts large (`ProximityPrompt` max activation distance 8–12 studs).
- **Spawn safety:** players must not spawn inside geometry, on kill bricks, or in view of PvP without protection (`ForceField` duration).

---

## 5. Part hygiene and performance

- `Anchored = true` for all static geometry.
- `CanCollide = false`, `CanTouch = false`, `CanQuery = false` for purely decorative parts.
- `CastShadow = false` on small or interior detail parts.
- Group logical objects into `Model`s with a `PrimaryPart`. Name every model and part meaningfully (no `Part`, `Part`, `Part`).
- Organize `workspace` into folders: `Map/`, `Spawns/`, `Interactables/`, `Effects/`.
- Use `CollectionService` tags (e.g. `KillBrick`, `Checkpoint`) instead of per-part scripts.
- Keep part count reasonable: prefer MeshParts or unions for complex shapes but check collision fidelity (`CollisionFidelity = Box` or `Hull` for decorations).
- Design for `StreamingEnabled`: put critical gameplay objects near spawn or mark models `ModelStreamingMode = Persistent` only when required.

---

## 6. Building via code or Studio MCP

When generating geometry from scripts (Command Bar, plugin, or Studio MCP `execute_luau`):

```luau
--!strict
-- Snap-safe platform builder (run in Studio Command Bar or MCP)
local GRID = 4

local function snap(v: number): number
	return math.round(v / GRID) * GRID
end

local function makePlatform(name: string, pos: Vector3, size: Vector3, parent: Instance): Part
	local p = Instance.new("Part")
	p.Name = name
	p.Anchored = true
	p.Size = size
	p.Position = Vector3.new(snap(pos.X), pos.Y, snap(pos.Z))
	p.Material = Enum.Material.SmoothPlastic
	p.TopSurface = Enum.SurfaceType.Smooth
	p.BottomSurface = Enum.SurfaceType.Smooth
	p.Parent = parent -- set Parent LAST
	return p
end
```

Rules:
- Always build into a named folder so the result can be undone or deleted in one step.
- Record `ChangeHistoryService` waypoints in plugins/MCP edits so the user can undo.
- After generating, inspect the result (screenshot or bounding-box check) before reporting success.
- Never delete or overwrite existing user geometry without asking.

---

## 7. Playability QA checklist

- [ ] Spawn is clear, safe, and faces the main objective.
- [ ] Every required jump tested in play mode on keyboard **and** with mobile emulation.
- [ ] No gaps of 1–3 studs where a character can get wedged.
- [ ] No zero-thickness floors or walls; no Z-fighting.
- [ ] Ceilings allow the camera to function.
- [ ] Map boundaries and fall respawn work.
- [ ] Checkpoints save progress correctly.
- [ ] All static parts anchored; decoration non-collidable where appropriate.
- [ ] Workspace organized into named folders and models.
- [ ] Performance check: MicroProfiler / `Stats` acceptable on a low-end device target.
