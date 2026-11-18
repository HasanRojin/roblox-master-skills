# Map Building and Playability Rules

> Agent-facing rules for building maps, levels, obbies, lobbies, and world geometry in Roblox.
> Deep reference: `roblox-brain-local/skills/gameplay/roblox-building/`, `roblox-physics/`, `roblox-lighting/`.
> All measurements are in **studs**. Values assume the default R15 avatar and default `StarterPlayer` settings (`WalkSpeed = 16`, `JumpHeight = 7.2`, `UseJumpPower = false`). If the game changes these, recompute the jump metrics in Section 3.

---

## 1. Grid and scale

| Rule | Value |
| :-- | :-- |
| Dimension increments | Snap sizes and offsets to **0.125 / 0.25 / 0.5 studs** for detail. Never arbitrary decimals like 0.333 or 1.17 (they compound into visible gaps). |
| Structural grid | Place walls, floors and platforms on a **1-stud or 4-stud** grid. |
| Placement method | Offset sub-parts from a named anchor `CFrame`, never from guessed world coordinates. Build complex CSG near the origin, then `PivotTo` the destination. |
| Rotation snap | 15° or 45° (90° for structural). |
| Wall thickness | 1 stud minimum (0.5 only for decorative trim). |
| Floor thickness | 1 stud minimum. Never use zero-thickness or paper-thin floors — characters can clip through. |
| Story height (floor to floor) | 12–16 studs (ceiling height + floor thickness). |
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
| Path / corridor width | 6 studs | 10+ studs for main roads and hubs |
| Ceiling height (rooms) | 10 studs | 10–14 studs (camera needs room) |
| Ceiling height (halls) | 16 studs | 16+ studs |
| Table / counter top | 3.5 studs | 3.5–4 studs from floor |
| Seat height | — | ~1.5 studs from floor |
| Stair step | rise ≤ 1 stud, run ≥ 1 stud | rise 1, run 1.5 |
| Ramp angle | ≤ 45° (walkable up to `Humanoid.MaxSlopeAngle`, default 89°, but steep slopes feel bad) | ≤ 30° |
| Gap between walls (to avoid getting stuck) | ≥ 4 studs or fully closed (0) | Never 1–3 studs |
| Spawn area | 12 × 12 studs clear per `SpawnLocation` | Multiple spawns for >10 players |

Camera clearance: third-person camera sits ~10–15 studs behind the player. Tight rooms must allow the camera to pull in without clipping into geometry; mark decorative clutter `CanQuery = false` if it blocks the camera unhelpfully.

---

## 3. Jump and movement metrics (defaults)

Derived from default physics (`Workspace.Gravity = 196.2`, `JumpHeight = 7.2`, `WalkSpeed = 16`):
- Take-off speed `v = sqrt(2 · g · h) ≈ 53.2 studs/s`
- Air time on flat ground `t = 2v / g ≈ 0.54 s`
- Horizontal travel of the root part `WalkSpeed · t ≈ 8.7 studs` (the edge-to-edge gap can be about 1 stud longer because of foot width)

| Metric | Safe (everyone, incl. mobile) | Hard | Impossible (default settings) |
| :-- | :-- | :-- | :-- |
| Vertical jump onto ledge | ≤ 5 studs | 5–7 | > 7.2 |
| Horizontal gap, same height (edge to edge) | ≤ 6 studs | 6–8 | > ~9.5 |
| Horizontal gap, landing 4 studs higher | ≤ 4 studs | 4–6 | > ~8 |
| Platform size for landing | ≥ 4 × 4 | 2 × 2 | < 1.5 × 1.5 |

- If the game changes `Gravity`, `JumpHeight`/`JumpPower` or `WalkSpeed`, recompute with the formulas above.
- These numbers are planning estimates, not traversal proof. Test every jump in play mode. Never mark a jump "done" from math alone.
- Obby difficulty must ramp gradually. Put checkpoints every 3–6 obstacles.
- Kill bricks must be visually distinct (bright red/neon) and never placed where a player can land without warning.
- Moving platforms: move with `PrismaticConstraint`/`AlignPosition` or `TweenService` on an anchored part plus server-side handling; test that players stay on them.

---

## 4. Layout and navigation

- **Readable landmarks:** every area needs one visually unique landmark visible from a distance.
- **Main path first:** build the critical path (spawn → core loop → reward) before decoration.
- **Width of main path:** 10+ studs in lobbies and hubs.
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
- Before building, inspect the place (Studio state, Workspace, existing assets). Name the root model, origin, and dimensions up front.
- Build in phases, and read back the instance tree after each phase. Never claim a build is done without readback.
- Search project assets and the Toolbox before generating a common prop. Inspect every inserted model's full hierarchy for scripts and remove code the game does not need. Disclose creator, asset ID and price before inserting paid or third-party assets.
- A blank texture or invisible mesh usually means a missing asset-privacy permission, not corruption. Read the Output error first.
- Always build into a named folder so the result can be undone or deleted in one step.
- Record `ChangeHistoryService` waypoints in plugins/MCP edits so the user can undo.
- After generating, inspect the result (screenshot or bounding-box check) before reporting success.
- Never delete or overwrite existing user geometry without asking.

---

## 7. Playability QA checklist

- [ ] Spawn is clear, safe, and faces the main objective.
- [ ] Every required jump tested in play mode on keyboard **and** with mobile emulation.
- [ ] Doorways, ladders, hatches and landings traversed with a real avatar (entry, body clearance, stepping off both ends).
- [ ] Corners, door lintels, and floor/ceiling seams inspected in motion for flicker (Z-fighting) and gaps.
- [ ] No gaps of 1–3 studs where a character can get wedged.
- [ ] No zero-thickness floors or walls; no Z-fighting.
- [ ] Ceilings allow the camera to function.
- [ ] Map boundaries and fall respawn work.
- [ ] Checkpoints save progress correctly.
- [ ] All static parts anchored; decoration non-collidable where appropriate.
- [ ] Workspace organized into named folders and models.
- [ ] Performance check: MicroProfiler / `Stats` acceptable on a low-end device target.
