# AI Error Log and Don'ts

> **Living document.** Every AI agent using this repository must read this file before starting work and must append to it whenever the user corrects a mistake.
> Rules in this file override general habits. If you are about to do something listed here, stop.

---

## 1. How to log a new mistake

Append a new entry at the TOP of Section 3 using this exact template. Keep it short and factual.

```markdown
### [YYYY-MM-DD] <Short title of the mistake>
- **Context:** What task was being done.
- **Mistake:** What the AI did wrong.
- **Symptom:** How it showed up (error message, bug, exploit, broken map).
- **Fix:** The correct approach.
- **Rule:** One-sentence rule to prevent recurrence. Start with "Always" or "Never".
- **Agent:** Model name (e.g. Claude, ChatGPT).
```

Logging rules:
- Never delete another agent's entry. Mark it `(superseded by <date> entry)` if a later entry replaces it.
- If the same mistake happens twice, promote its Rule into Section 2 (Permanent Don'ts).
- Do not log speculative or hypothetical mistakes. Only real ones that happened in this project.

---

## 2. Permanent Don'ts (anti-patterns)

### 2.1 Security
- **Never** trust values sent from the client (prices, damage, positions, currency, cooldowns, "isAdmin").
- **Never** put authoritative logic in a `LocalScript`.
- **Never** use `RemoteFunction:InvokeClient`.
- **Never** leave a remote without type validation and rate limiting.
- **Never** ship debug/admin remotes ungated.
- **Never** use `loadstring` or enable `LoadStringEnabled`.
- **Never** insert free models or toolbox assets without inspecting them for hidden scripts (`require(<id>)`, `getfenv`, obfuscated strings).

### 2.2 Luau
- **Never** use deprecated `wait()`, `spawn()`, `delay()`. Use the `task` library.
- **Never** use `game.Workspace.Part` dot-chains for objects that may not exist yet; use `WaitForChild` (client) or create them (server).
- **Never** call `Instance.new(class, parent)` with a parent argument; set `Parent` last.
- **Never** use `_G` / `shared` for module communication.
- **Never** silence type errors with `--!nonstrict` or blanket `:: any`.
- **Never** invent Roblox API members. If unsure, say so.
- **Never** connect events without a cleanup path.

### 2.3 Data
- **Never** call DataStore methods without `pcall` and retry.
- **Never** use `SetAsync` to update data that depends on the previous value; use `UpdateAsync`.
- **Never** let a player continue with default data after a failed load (it overwrites their real save).
- **Never** forget `game:BindToClose`.
- **Never** return `PurchaseGranted` in `ProcessReceipt` before the purchase is persisted.

### 2.4 Building
- **Never** create zero-thickness floors or walls.
- **Never** leave 1–3 stud gaps where players can get stuck.
- **Never** report a jump or obstacle as "working" without a play test.
- **Never** leave static parts unanchored.
- **Never** dump hundreds of unnamed parts directly into `workspace`.
- **Never** delete or overwrite the user's existing builds or scripts without explicit confirmation.

### 2.5 Agent behavior
- **Never** claim something was tested when it was not. State exactly what was and was not verified.
- **Never** output a "simplified" version of an existing script that silently removes features.
- **Never** rewrite large working systems when a targeted fix was requested.
- **Never** guess at the user's project structure; inspect it first (Explorer, `script_search`, file tree).
- **Never** hardcode asset IDs you have not verified exist.
- **Never** commit or publish secrets (API keys, tokens, `.ROBLOSECURITY` cookies, webhook URLs) to code or to this repository.

---

## 3. Incident log (newest first)

### [2026-11-18] Example entry — template demonstration
- **Context:** Building a coin shop.
- **Mistake:** The client sent the item price in the `BuyItem` remote and the server trusted it.
- **Symptom:** Exploiters could buy any item for 0 coins.
- **Fix:** Client sends only `itemId`; server reads the price from a server-only `Catalog` module.
- **Rule:** Never accept prices, rewards, or damage values from the client.
- **Agent:** Template (not a real incident).

<!-- Add new entries directly under the "Incident log" heading, above all existing entries. -->
