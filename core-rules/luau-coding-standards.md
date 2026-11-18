# Luau Coding Standards (Roblox)

> Agent-facing rules. Mandatory for all Luau written or reviewed in projects using this repository.
> Deep references: `roblox-brain-local/skills/core/` (luau-core, luau-types, luau-patterns, networking, security, data, performance).

---

## 1. File and project layout

| Location | Contains | Script type |
| :-- | :-- | :-- |
| `ServerScriptService/` | All authoritative game logic, validation, DataStore access. | `Script` / server ModuleScripts |
| `ServerStorage/` | Server-only assets and modules (never replicated). | ModuleScripts, models |
| `ReplicatedStorage/Shared/` | Pure, side-effect-free modules used by both sides (types, constants, config, math). | ModuleScripts |
| `ReplicatedStorage/Remotes/` | RemoteEvents / RemoteFunctions / UnreliableRemoteEvents, created by the server. | Instances |
| `StarterPlayer/StarterPlayerScripts/` | Client controllers (input, camera, UI glue). | `LocalScript` / client ModuleScripts |
| `StarterGui/` | ScreenGuis. Logic lives in controllers, not inside each button. | `LocalScript` (minimal) |

Rules:
- One bootstrap `Script` per side; everything else is a `ModuleScript` required by it.
- Never use `_G` or `shared` for cross-script communication. Use `require`.
- Secrets, prices, drop tables and damage formulas live on the server only.
- Rojo-friendly naming: `Name.server.luau`, `Name.client.luau`, `Name.luau`.

---

## 2. Style

- First line: `--!strict`. Fix type errors; do not downgrade to `--!nonstrict` to silence them.
- Get services once at the top: `local Players = game:GetService("Players")`. Never `game.Players`.
- Naming: `PascalCase` for modules, classes, services, types; `camelCase` for locals and functions; `UPPER_SNAKE_CASE` for constants.
- Use `task.wait`, `task.spawn`, `task.defer`, `task.delay`. Never `wait`, `spawn`, `delay`.
- Use `WaitForChild` on the client for replicated instances, with a timeout where the instance might legitimately be missing. With a timeout it returns `Instance?`, so handle `nil`. Do not use it on the server for instances the server itself created.
- Prefer early returns over deep nesting.
- No magic numbers in logic. Put tunables in a `Config` module.
- Deprecated APIs are banned: `Instance.new(class, parent)` (set Parent last), `BodyVelocity`/`BodyPosition`/`BodyGyro` (use constraint movers), `:connect` (lowercase), `Workspace:FindPartOnRay` (use `Workspace:Raycast`).

```luau
--!strict
-- ReplicatedStorage/Shared/Config.luau (ModuleScript)
local Config = {
	MAX_WALK_SPEED = 16,
	SHOP_COOLDOWN_SECONDS = 0.5,
}
return table.freeze(Config)
```

---

## 3. Types

- Annotate every function parameter and return value in public module APIs.
- Narrow `Instance?` before use: `if not part or not part:IsA("BasePart") then return end`.
- Export shared types from a `Types` module in `ReplicatedStorage/Shared/`.
- Prefer `:IsA("ClassName")` over `:: Cast`: it narrows the type and validates at runtime.
- Never cast with `:: any` to escape an error unless you add a comment explaining why. `x :: any :: T` means the types are lying somewhere.
- Types are compile-time only. They never validate network input; remote handlers type incoming arguments as `unknown` and narrow them (Section 4).
- Forward-declare mutually recursive functions with a typed `local name: (args) -> ()` and assign with `name = function(...)`, not `local function name` (which shadows the declaration).

```luau
export type PlayerData = {
	version: number,
	coins: number,
	inventory: { [string]: number },
}
```

---

## 4. RemoteEvent / RemoteFunction security (CRITICAL)

The client is controlled by the player and can be modified by exploiters. Every remote is a public API exposed to attackers.

### 4.1 Hard rules

1. **Server creates remotes.** Clients only find them.
2. **Never trust any argument.** Validate in this order and return early on the first failure: **type → range → permission → plausibility** (distance, state, cooldown). Drop invalid requests silently; never `error()` back to the client.
   - `NaN` passes `typeof(x) == "number"` and defeats `<`/`>`. Check `x ~= x` first, then `math.abs(x) == math.huge`.
   - Never index an argument before type-checking it (`itemId.Name` on a string throws and kills the handler thread).
   - Tables sent through remotes are copies: metatables stripped, functions arrive `nil`, mixed keys mangled, `nil` truncates arrays. Prefer scalar arguments; otherwise validate field by field.
   - Use `utf8.len(s)` to reject malformed UTF-8 strings (they fail DataStore saves later).
3. **The client sends intent, never results.** Send `"BuyItem", "Sword"` — never `coins = 999` or `damage = 100`.
4. **The server recomputes everything:** prices, damage, distance, cooldowns, line of sight.
5. **Rate-limit every remote per player** with a token bucket (Section 4.3). Drop excess calls silently. Log repeat offenders, but **never auto-ban on rate limits alone** (mobile lag and retry loops cause false positives); act only on accumulated evidence.
6. **Check sanity against server state:** is the player alive, in range, owning the item, not on cooldown?
7. **Never use `RemoteFunction:InvokeClient`.** A client can yield forever or error. Use RemoteEvents for server→client.
8. **`OnServerInvoke` must never yield on untrusted work** and must always return a value.
9. **Never pass instances from the client as authority.** If a client sends a `Part`, verify it is a descendant of the expected container and within range.
10. **Never expose admin/debug remotes in production.** Gate every privileged remote by a server-side allowlist of UserIds (or group rank checked on the server), and do not create QA/cheat remotes outside Studio (`RunService:IsStudio()`).
11. **Never read authoritative values from replicated objects** such as `leaderstats` `IntValue`s or Tool attributes the client can see. Server state lives in server modules; look stats up by item ID.
12. **Character movement is client-simulated.** Speed/fly hacks cannot be prevented, only detected: sample `HumanoidRootPart` positions on the server, flag displacement above max speed × elapsed time (with margin), rubber-band before punishing.
13. **`UnreliableRemoteEvent`** is only for replaceable data (VFX triggers, position hints). It can drop messages and has a ~1000-byte payload cap.

### 4.2 Reference pattern

```luau
--!strict
-- ServerScriptService/Shop/ShopServer.luau (ModuleScript)
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Catalog = require(script.Parent.Catalog) -- server-only prices
local PlayerData = require(script.Parent.Parent.Data.PlayerData)

local RateLimit = require(script.Parent.Parent.Net.RateLimit) -- token bucket, Section 4.3

-- The server creates the remotes folder; no WaitForChild needed here.
local remotes = ReplicatedStorage:FindFirstChild("Remotes")
if not remotes then
	remotes = Instance.new("Folder")
	remotes.Name = "Remotes"
	remotes.Parent = ReplicatedStorage
end

local buyRemote = Instance.new("RemoteEvent")
buyRemote.Name = "BuyItem"
buyRemote.Parent = remotes

buyRemote.OnServerEvent:Connect(function(player: Player, itemId: unknown, quantity: unknown)
	if not RateLimit.allow(player, "BuyItem", 3, 5) then return end

	-- 1. TYPE
	if typeof(itemId) ~= "string" or typeof(quantity) ~= "number" then return end
	-- 2. RANGE (NaN first, then integer and bounds)
	if quantity ~= quantity then return end
	if quantity ~= math.floor(quantity) or quantity < 1 or quantity > 100 then return end
	if #itemId == 0 or #itemId > 32 then return end
	-- 3. PERMISSION: item exists, player data loaded
	local item = Catalog[itemId]
	if not item then return end
	local data = PlayerData.get(player)
	if not data then return end
	-- 4. PLAUSIBILITY: server-side price and balance
	local cost = item.price * quantity
	if data.coins < cost then return end

	data.coins -= cost
	data.inventory[itemId] = (data.inventory[itemId] or 0) + quantity
end)

return {}
```

### 4.3 Token-bucket rate limiter

```luau
--!strict
-- ServerScriptService/Net/RateLimit.luau (ModuleScript)
local Players = game:GetService("Players")

type Bucket = { tokens: number, last: number }
local buckets: { [Player]: { [string]: Bucket } } = {}

local RateLimit = {}

function RateLimit.allow(player: Player, key: string, ratePerSec: number, burst: number): boolean
	local now = os.clock()
	local playerBuckets = buckets[player]
	if not playerBuckets then
		playerBuckets = {}
		buckets[player] = playerBuckets
	end
	local b = playerBuckets[key]
	if not b then
		b = { tokens = burst, last = now }
		playerBuckets[key] = b
	end
	b.tokens = math.min(burst, b.tokens + (now - b.last) * ratePerSec)
	b.last = now
	if b.tokens < 1 then
		return false
	end
	b.tokens -= 1
	return true
end

Players.PlayerRemoving:Connect(function(player)
	buckets[player] = nil -- prevent memory leak
end)

return RateLimit
```

### 4.4 Number validation helper

```luau
local function isFiniteNumber(value: unknown, min: number, max: number): boolean
	return typeof(value) == "number" and value == value -- rejects NaN
		and math.abs(value) ~= math.huge
		and value >= min and value <= max
end
```

### 4.5 Distance check for world interactions

```luau
local MAX_REACH = 20 -- studs; tune per interaction

local function canReach(player: Player, target: BasePart): boolean
	local character = player.Character
	local root = character and character:FindFirstChild("HumanoidRootPart")
	if not root or not root:IsA("BasePart") then return false end
	return (root.Position - target.Position).Magnitude <= MAX_REACH
end
```

Also re-check state inside the handler: the chest is not already open, the mob is alive, the item has not been claimed by someone else.

---

## 5. DataStore safety

- **Prefer ProfileStore** (successor to ProfileService) for production. It already implements session locking, retries and release. Hand-roll the rules below only with a reason.
- Wrap every DataStore call in `pcall` with retry and exponential backoff (e.g. 3–5 attempts, 1s → 2s → 4s, capped at 16s).
- Use `UpdateAsync` for player data; never `GetAsync` + `SetAsync` (races against other servers and yourself).
- **Never yield inside an `UpdateAsync` transform.** Compute the new value synchronously; return `nil` to abort the write.
- **Session locking:** on load, claim the profile inside `UpdateAsync` by writing this server's `JobId` + timestamp. If another server holds a fresh lock, do not load; retry briefly, then kick with "your data is still saving, rejoin in a moment". Autosaves refresh the lock; a lock older than ~2× the autosave interval is stale.
- **Gameplay code never touches DataStores.** It mutates one in-memory profile table; one persistence module saves it.
- Save on: autosave every 30–60 s (staggered per player, e.g. `UserId % interval`), `PlayerRemoving` (final save + release lock), and `game:BindToClose` (save all players in parallel with `task.spawn`, wait until done or ~25 s; skip the wait in Studio).
- Same-key writes are throttled (about one per ~6 s). One key per player; never save on every coin pickup.
- Store a `version` field and migrate old schemas on load, inside the same `UpdateAsync` that claims the lock. Deep-copy the default profile for new players.
- If loading fails, do NOT let the player play with default data that will later overwrite their real save. Distinguish "no data" (new player) from "call failed", and kick on failure.
- Store only JSON-encodable data: no Instances, functions, CFrames, cyclic or mixed-key tables. Convert at the boundary (`Vector3` → `{x, y, z}`, `Color3` → hex). Cap unbounded arrays (logs, histories); a key holds at most ~4 MB.

---

## 6. Performance and memory

- Every `:Connect` must have an owner that disconnects it (on destroy, on player leave, on character removal). Use a Maid/Trove/Janitor pattern or a simple connection list.
- Clean per-player tables in `Players.PlayerRemoving`.
- Never create instances or tables inside `RunService.Heartbeat`/`RenderStepped` unless pooled.
- Use `RenderStepped` / `BindToRenderStep` only on the client and only for camera or visual work.
- Throttle remote traffic: batch updates, send deltas, use `UnreliableRemoteEvent` for high-frequency cosmetic data.
- Prefer `CollectionService` tags over scanning `workspace:GetDescendants()` repeatedly.
- Enable and design for `StreamingEnabled`: on the client, never assume a distant part exists.
- Set `Anchored = true` on static map parts and `CanCollide/CanTouch/CanQuery = false` where not needed.
- Never tween world objects on the server (each step replicates). Fire one small event and tween on clients.
- Not everything needs 60 Hz: accumulate `dt` and update health bars at ~10 Hz, minimaps at ~4 Hz. Never use `while true do task.wait() end` to poll for something an event can signal.
- Cache `FindFirstChild` results outside frame loops; `GetChildren()` allocates a new table on every call.
- Pool frequently spawned objects (bullets, damage numbers) instead of create/destroy churn.
- Call `SetNetworkOwner(nil)` on server-driven NPCs and parts so exploiters cannot physics-drag them.
- Before teleporting a player far away under `StreamingEnabled`, call `Player:RequestStreamAroundAsync` on the destination.
- Measure before optimizing: MicroProfiler (Ctrl+F6), Developer Console → Memory (F9; steady `LuaHeap` growth while idle means leaks), Script Performance panel.

---

## 7. Error handling and logging

- Use `pcall`/`xpcall` around any call that can fail outside your control: DataStore, HTTP, MarketplaceService, `TeleportService`, `require` of optional modules.
- Log with a prefix: `warn("[Shop]", player.UserId, "invalid item", itemId)`.
- Never `error()` inside a remote handler for invalid client input — return silently.

---

## 8. Monetization

- Handle Developer Products only in `MarketplaceService.ProcessReceipt`. Grant the item, persist it, and only then return `Enum.ProductPurchaseDecision.PurchaseGranted`. Return `NotProcessedYet` on any failure.
- Make receipt processing idempotent: store processed `PurchaseId`s.
- Verify Game Pass ownership on the server with `UserOwnsGamePassAsync` (wrapped in `pcall`, cached per session).

---

## 9. Review checklist (run before delivering code)

- [ ] `--!strict` present and no type errors.
- [ ] All services fetched via `GetService`.
- [ ] No deprecated APIs (`wait`, `spawn`, `BodyVelocity`, `Instance.new(c, parent)`).
- [ ] Every remote validates type → range (NaN/inf) → permission → plausibility, and is rate-limited.
- [ ] Client sends intent only; server computes outcomes.
- [ ] No `InvokeClient`.
- [ ] Every connection and per-player table is cleaned up.
- [ ] DataStore calls use `pcall` + retry + `UpdateAsync` + session lock + `BindToClose`; failed loads kick instead of using defaults.
- [ ] No secrets or authoritative data in `ReplicatedStorage`.
- [ ] Script location and type are stated in a header comment.
