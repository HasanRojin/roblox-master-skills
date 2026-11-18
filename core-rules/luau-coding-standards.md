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
- Use `WaitForChild` on the client for replicated instances, with a timeout where the instance might legitimately be missing. Do not use it on the server for instances the server itself created.
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
- Never cast with `:: any` to escape an error unless you add a comment explaining why.

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
2. **Never trust any argument.** Validate type, range, length, NaN/inf, and ownership on every call.
3. **The client sends intent, never results.** Send `"BuyItem", "Sword"` — never `coins = 999` or `damage = 100`.
4. **The server recomputes everything:** prices, damage, distance, cooldowns, line of sight.
5. **Rate-limit every remote per player.** Drop excess calls silently; log repeated abuse.
6. **Check sanity against server state:** is the player alive, in range, owning the item, not on cooldown?
7. **Never use `RemoteFunction:InvokeClient`.** A client can yield forever or error. Use RemoteEvents for server→client.
8. **`OnServerInvoke` must never yield on untrusted work** and must always return a value.
9. **Never pass instances from the client as authority.** If a client sends a `Part`, verify it is a descendant of the expected container and within range.
10. **Never expose admin/debug remotes in production.** Gate them by `game.CreatorId`/group rank **and** `RunService:IsStudio()` where appropriate.

### 4.2 Reference pattern

```luau
--!strict
-- ServerScriptService/Shop/ShopServer.luau (ModuleScript)
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Catalog = require(script.Parent.Catalog) -- server-only prices
local PlayerData = require(script.Parent.Parent.Data.PlayerData)

local remotes = ReplicatedStorage:WaitForChild("Remotes")
local buyRemote = Instance.new("RemoteEvent")
buyRemote.Name = "BuyItem"
buyRemote.Parent = remotes

local COOLDOWN = 0.5
local lastCall: { [Player]: number } = {}

local function isValidItemId(value: unknown): boolean
	return typeof(value) == "string" and #value > 0 and #value <= 32 and Catalog[value] ~= nil
end

buyRemote.OnServerEvent:Connect(function(player: Player, itemId: unknown)
	local now = os.clock()
	if now - (lastCall[player] or 0) < COOLDOWN then
		return -- rate limited
	end
	lastCall[player] = now

	if not isValidItemId(itemId) then
		return
	end
	local id = itemId :: string
	local price = Catalog[id].price -- server decides the price

	local data = PlayerData.get(player)
	if not data or data.coins < price then
		return
	end
	data.coins -= price
	data.inventory[id] = (data.inventory[id] or 0) + 1
end)

Players.PlayerRemoving:Connect(function(player)
	lastCall[player] = nil -- prevent memory leak
end)

return {}
```

### 4.3 Number validation helper

```luau
local function isFiniteNumber(value: unknown, min: number, max: number): boolean
	return typeof(value) == "number" and value == value -- rejects NaN
		and value > -math.huge and value < math.huge
		and value >= min and value <= max
end
```

---

## 5. DataStore safety

- Wrap every DataStore call in `pcall` with retry and exponential backoff (e.g. 3–5 attempts, 1s → 2s → 4s).
- Use `UpdateAsync` for writes that depend on existing data; never read-then-`SetAsync`.
- Implement session locking (or use a proven library such as ProfileStore) to prevent duplication across servers.
- Save on `PlayerRemoving`, on a periodic autosave (60–180 s), and in `game:BindToClose` (yield until saves finish, within the 30 s limit).
- Store a `version` field and migrate old schemas on load.
- If loading fails, do NOT let the player play with default data that will later overwrite their real save. Kick with a clear message or retry.
- Never store `Instance` references or functions in a DataStore.

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
- [ ] Every remote validates types, ranges, ownership, and is rate-limited.
- [ ] Client sends intent only; server computes outcomes.
- [ ] No `InvokeClient`.
- [ ] Every connection and per-player table is cleaned up.
- [ ] DataStore calls use `pcall` + retry + `UpdateAsync` + `BindToClose`.
- [ ] No secrets or authoritative data in `ReplicatedStorage`.
- [ ] Script location and type are stated in a header comment.
