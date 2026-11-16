---
name: roblox-luau
description: Write, review, debug, and refactor Roblox game code in Luau. Use this skill whenever the user mentions Roblox, Roblox Studio, Luau, Lua scripts for a Roblox game, RemoteEvents, DataStores, ModuleScripts, Rojo, game passes, obbies, tycoons, simulators, or any Roblox gameplay, UI, or backend feature, even if they just say "script" or "Lua" without naming Roblox. Also use it for converting old Lua/Roblox code to modern Luau, adding type annotations, fixing client/server bugs, securing remotes against exploiters, and structuring a Roblox project.
---

# Roblox + Luau Scripting

Luau is Roblox's typed, faster dialect of Lua 5.1. Do not write plain Lua 5.3/5.4: Luau has no `goto`, no bitwise operators (use `bit32`), no integer subtype, and no `<const>`/`<close>` attributes.

## Workflow

1. **Clarify where the code runs.** Server Script, LocalScript, or ModuleScript? If unclear, state your assumption in one line.
2. **Say where each script goes** (e.g. `ServerScriptService/Main`) and its type, plus any instances the user must create in Studio.
3. **Write modern Luau.** Use `--!strict` as the first line of modules.
4. **Treat the client as hostile.** Never trust data from `FireServer` / `InvokeServer`.
5. **Handle failure.** Wrap anything that can error (DataStores, HTTP, MarketplaceService) in `pcall` and check the result.
6. After the code, give short test instructions and known limits.

## Client/server model

| Location | Runs on | Notes |
|---|---|---|
| `ServerScriptService` | Server | Server logic. Hidden from clients. |
| `ServerStorage` | Server | Private assets/modules. Clients never see it. |
| `ReplicatedStorage` | Both | Shared modules, remotes, assets. Exploiters can read it. |
| `ReplicatedFirst` | Client (early) | Loading screens. |
| `StarterPlayer/StarterPlayerScripts` | Client | Copied to each player once. |
| `StarterPlayer/StarterCharacterScripts` | Client | Re-copied on every respawn. |
| `StarterGui` | Client | Cloned into `PlayerGui` on spawn unless `ResetOnSpawn = false`. |
| `Workspace` | Both | Server changes replicate down; client changes do NOT replicate up. |

- The **server owns truth**: currency, inventory, damage, spawning, saving, purchases.
- The **client owns presentation**: input, camera, UI, local effects.
- Keep secrets and balance numbers in `ServerScriptService` / `ServerStorage`, never `ReplicatedStorage`.
- A `LocalScript` only runs in a character, `Backpack`, `PlayerGui`, `PlayerScripts`, or `ReplicatedFirst`. A `Script` with `RunContext = Client` runs on the client from anywhere.

## Modern Luau cheat sheet

```lua
--!strict
export type Item = { id: string, price: number, tags: { string }? }

local Cheatsheet = {}

function Cheatsheet.priceLabel(item: Item): string
	return `{item.id}: {item.price} coins`
end

function Cheatsheet.sumEven(values: { number }): number
	local total = 0
	for _, v in values do
		if v % 2 ~= 0 then
			continue
		end
		total += v
	end
	return total
end

function Cheatsheet.tier(total: number): string
	return if total > 50 then "big" else "small"
end

function Cheatsheet.withExtra(values: { number }, extra: number): { number }
	local copy = table.clone(values)
	table.insert(copy, extra)
	return table.freeze(copy)
end

function Cheatsheet.indexOf(values: { number }, target: number): number?
	return table.find(values, target)
end

return table.freeze(Cheatsheet)
```

Requirements to run: ModuleScript at ReplicatedStorage/Shared/Cheatsheet. Pre-created instances: a Shared folder in ReplicatedStorage. Services/settings: none. Standalone as a module; nothing calls it, it is reference only.

Also available: `table.create(n, value)` to preallocate and `table.clear(t)` to empty in place. Iterate with `for k, v in t do` (no `pairs`/`ipairs` needed).

Style:

- `local` everything; avoid `_G`/`shared`.
- Tabs for indentation; camelCase locals/functions, PascalCase types/classes/services, UPPER_SNAKE constants.
- `game:GetService(...)` once at the top of the file.
- Early returns over deep nesting. `:` for methods, `.` for fields. `~=` not `!=`. `..` concatenates. Arrays are 1-indexed.

## Deprecated / discouraged: use the modern replacement

| Avoid | Use instead |
|---|---|
| `wait()`, `spawn()`, `delay()` | `task.wait()`, `task.spawn()`, `task.delay()` |
| `Instance.new("Part", parent)` | Create, set all properties, then set `.Parent` last |
| `BodyVelocity`, `BodyGyro`, `BodyPosition` | `LinearVelocity`, `AlignOrientation`, `AlignPosition`, `VectorForce` |
| `workspace:FindPartOnRay` / `Ray.new` | `workspace:Raycast(origin, direction, RaycastParams)` |
| `while wait() do` | `while true do ... task.wait() end` or `RunService.Heartbeat` |
| `Model:SetPrimaryPartCFrame` | `Model:PivotTo(cframe)` / `Model:GetPivot()` |
| `DataStore:SetAsync` for read-modify-write | `UpdateAsync` |
| Polling a value in a loop | `:GetPropertyChangedSignal()`, `.Changed`, or `:GetAttributeChangedSignal()` |
| `StringValue`/`IntValue` for custom state | Attributes (`:SetAttribute`) or module state |
| `pairs(t)` / `ipairs(t)` | `for k, v in t do` |

## Core patterns

### Players and characters

Handle players who joined before the script ran, and characters that already exist.

```lua
--!strict
local Players = game:GetService("Players")

local function onPlayerAdded(player: Player)
	local function onCharacterAdded(character: Model)
		local humanoid = character:WaitForChild("Humanoid") :: Humanoid
		humanoid.Died:Once(function()
			local deaths = (player:GetAttribute("Deaths") :: number?) or 0
			player:SetAttribute("Deaths", deaths + 1)
		end)
	end

	player.CharacterAdded:Connect(onCharacterAdded)
	if player.Character then
		task.spawn(onCharacterAdded, player.Character)
	end
end

Players.PlayerAdded:Connect(onPlayerAdded)
for _, player in Players:GetPlayers() do
	task.spawn(onPlayerAdded, player)
end
```

Requirements to run: Script at ServerScriptService/PlayerLifecycle. Pre-created instances: none. Services/settings: none. Standalone.

Per-player tables must be cleared in `Players.PlayerRemoving` (see the remote example).

### ModuleScripts

```lua
--!strict
local Config = {
	MAX_HEALTH = 100,
	RESPAWN_TIME = 5,
}
return table.freeze(Config)
```

Requirements to run: ModuleScript at ReplicatedStorage/Shared/Config. Pre-created instances: a Shared folder in ReplicatedStorage. Services/settings: none. Standalone as a module.

Require it from any script with `require(ReplicatedStorage:WaitForChild("Shared"):WaitForChild("Config") :: ModuleScript)`. Use `WaitForChild` (not `.Shared.Config`) so strict mode type-checks and the client does not race replication.

A module runs once per side (server and client each get their own copy); its state is not shared over the network. Avoid circular requires.

Class pattern with metatables:

```lua
--!strict
local Tycoon = {}
Tycoon.__index = Tycoon

export type Tycoon = typeof(setmetatable({} :: { owner: Player, cash: number }, Tycoon))

function Tycoon.new(owner: Player): Tycoon
	return setmetatable({ owner = owner, cash = 0 }, Tycoon)
end

function Tycoon.addCash(self: Tycoon, amount: number)
	self.cash += amount
end

return Tycoon
```

Requirements to run: ModuleScript at ServerScriptService/Tycoon. Pre-created instances: none. Services/settings: none. Host constraint: needs a server Script that calls `require(...).new(player)`.

### Connection cleanup

Every `:Connect()` returns a connection. Destroying an instance disconnects connections to its own events; connections to long-lived objects (`Players`, `RunService`, `workspace`) must be disconnected manually. For several connections, collect them in a table and loop `:Disconnect()`.

```lua
--!strict
local RunService = game:GetService("RunService")

local spinner = workspace:WaitForChild("Spinner") :: BasePart

local connection = RunService.Heartbeat:Connect(function(dt: number)
	spinner.CFrame *= CFrame.Angles(0, math.rad(90) * dt, 0)
end)

spinner.Destroying:Once(function()
	connection:Disconnect()
end)
```

Requirements to run: LocalScript at StarterPlayer/StarterPlayerScripts/SpinnerVisual. Pre-created instances: an anchored Part named Spinner in Workspace. Services/settings: none. Host constraint: with StreamingEnabled, `WaitForChild` waits until the part streams in.

Use `Heartbeat` (after physics) for most per-frame logic, `RenderStepped` (client only, before render) for camera/visuals, and always scale by `dt`. Prefer events over per-frame polling.

### CollectionService + attributes

Tag instances in Studio and let code react, instead of hard-coding paths.

```lua
--!strict
local CollectionService = game:GetService("CollectionService")

local TAG = "KillBrick"

local function setup(instance: Instance)
	if not instance:IsA("BasePart") then
		return
	end
	instance.Color = Color3.new(1, 0, 0)
	instance:SetAttribute("Damage", 25)
end

CollectionService:GetInstanceAddedSignal(TAG):Connect(setup)
for _, instance in CollectionService:GetTagged(TAG) do
	task.spawn(setup, instance)
end
```

Requirements to run: Script at ServerScriptService/KillBrickSetup. Pre-created instances: one or more Parts tagged KillBrick (Studio Tag Editor). Services/settings: none. Standalone.

### Errors and async

- `local ok, result = pcall(fn, ...)` returns `(success, valueOrError)`. Always check `ok`.
- `task.spawn(fn, ...)` runs now in a new thread; `task.defer(fn, ...)` runs at the end of the current resumption cycle; `task.delay(sec, fn, ...)` runs later.
- `WaitForChild(name, timeout)` returns nil on timeout; use `FindFirstChild` when absence is acceptable.
- With StreamingEnabled, client code must tolerate parts that are not loaded.

## Remotes and security (most important section)

Exploiters control the client. They can fire any remote with any arguments at any rate.

Setup: put remotes in `ReplicatedStorage/Remotes`. Prefer `RemoteEvent` over `RemoteFunction`; never call `InvokeClient` from the server (a client can hang it forever). Use `UnreliableRemoteEvent` for frequent, droppable data.

Validate on the server in every `OnServerEvent`:

1. **Types:** `typeof(arg) == "number"` etc.; reject NaN (`arg ~= arg`) and `math.huge`.
2. **Ranges:** clamp or reject.
3. **Ownership/permission:** does the player own the item, plot, or tool?
4. **State:** is the action legal now (cooldown, alive, in round, has funds)?
5. **Distance:** is the player actually near what they claim to touch?
6. **Rate limit:** per-player cooldown table, cleared on `PlayerRemoving`.
7. **Never accept results from the client** (damage, cash earned, price). Accept intent ("buy X") and compute the outcome on the server.

```lua
--!strict
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local buyRemote = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("BuyItem") :: RemoteEvent
local PRICES: { [string]: number } = { Sword = 100, Shield = 75 }
local COOLDOWN = 0.5
local lastBuy: { [Player]: number } = {}

buyRemote.OnServerEvent:Connect(function(player: Player, itemId: unknown)
	if typeof(itemId) ~= "string" then
		return
	end
	local price = PRICES[itemId]
	if not price then
		return
	end

	local now = os.clock()
	if now - (lastBuy[player] or 0) < COOLDOWN then
		return
	end
	lastBuy[player] = now

	local coins = (player:GetAttribute("Coins") :: number?) or 0
	if coins < price then
		return
	end
	player:SetAttribute("Coins", coins - price)
	player:SetAttribute(`Owns{itemId}`, true)
end)

Players.PlayerRemoving:Connect(function(player: Player)
	lastBuy[player] = nil
end)
```

Requirements to run: Script at ServerScriptService/BuyHandler. Pre-created instances: a Remotes folder in ReplicatedStorage containing a RemoteEvent named BuyItem. Services/settings: none. Host constraint: the `Coins` attribute must be seeded by other server logic for testing (e.g. `player:SetAttribute("Coins", 200)` on join); it defaults to 0, so buys silently fail until seeded. A client LocalScript must call `buyRemote:FireServer("Sword")`.

Also:

- Admin checks use `UserId` on the server, never `player.Name` or client checks.
- Server-side hit detection and damage; the client only requests attacks.
- Set `part:SetNetworkOwner(nil)` for server-critical physics parts.
- Never put API keys or webhooks in code that runs on or replicates to the client.

## Data persistence (DataStoreService)

- Server only. Enable **Studio Access to API Services** (Game Settings > Security) to test in Studio.
- `pcall` every call and retry with backoff (`task.wait` takes seconds).
- Use `UpdateAsync` for atomic writes. For player data (one server-side owner per player), the callback may safely overwrite; for shared keys, merge the existing value inside the callback. Key by `UserId`, not name.
- Save on `PlayerRemoving`, in `game:BindToClose` (wait for all saves), and optionally autosave every few minutes.
- Store plain data only (numbers, strings, booleans, arrays, string-keyed dictionaries). Convert `Vector3`, `Color3`, Instances first.
- Version the schema and merge with defaults so new fields do not break old saves. Validate field types on load: `value or default` is not enough if the stored value is a wrong-typed truthy. For numbers, also reject NaN and infinity (`v == v`, `v < math.huge`).
- Do not save if loading failed; kick the player instead of overwriting their data.
- For high-stakes games use session locking (ProfileService / ProfileStore) rather than hand-rolling.

```lua
--!strict
local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

export type PlayerData = { coins: number, level: number }

local MAX_ATTEMPTS = 3
local DEFAULTS: PlayerData = { coins = 0, level = 1 }
local store = DataStoreService:GetDataStore("PlayerData_v1")
local sessions: { [Player]: PlayerData } = {}
local pendingSaves = 0
local started = false

local function key(userId: number): string
	return `Player_{userId}`
end

local function isFiniteNumber(value: unknown): boolean
	return type(value) == "number" and value == value and value > -math.huge and value < math.huge
end

local function loadData(userId: number): PlayerData?
	for attempt = 1, MAX_ATTEMPTS do
		local ok, stored = pcall(function()
			return store:GetAsync(key(userId))
		end)
		if ok then
			local data = table.clone(DEFAULTS)
			if type(stored) == "table" then
				if isFiniteNumber(stored.coins) then
					data.coins = stored.coins
				end
				if isFiniteNumber(stored.level) then
					data.level = stored.level
				end
			end
			return data
		end
		if attempt < MAX_ATTEMPTS then
			task.wait(2 ^ attempt)
		end
	end
	return nil
end

local function saveData(player: Player)
	local data = sessions[player]
	if not data then
		return
	end
	sessions[player] = nil
	pendingSaves += 1
	local ok, err = pcall(function()
		store:UpdateAsync(key(player.UserId), function()
			return data
		end)
	end)
	pendingSaves -= 1
	if not ok then
		warn(`Save failed for {player.UserId}: {err}`)
	end
end

local function onPlayerAdded(player: Player)
	local data = loadData(player.UserId)
	if not data then
		player:Kick("Could not load your data. Please rejoin.")
		return
	end
	if player.Parent == nil then
		return
	end
	sessions[player] = data
end

local PlayerDataService = {}

function PlayerDataService.get(player: Player): PlayerData?
	return sessions[player]
end

function PlayerDataService.start()
	if started then
		return
	end
	started = true
	Players.PlayerAdded:Connect(onPlayerAdded)
	Players.PlayerRemoving:Connect(saveData)
	for _, player in Players:GetPlayers() do
		task.spawn(onPlayerAdded, player)
	end
	game:BindToClose(function()
		for _, player in Players:GetPlayers() do
			task.spawn(saveData, player)
		end
		while pendingSaves > 0 do
			task.wait()
		end
	end)
end

return PlayerDataService
```

Requirements to run: ModuleScript at ServerScriptService/PlayerDataService. Pre-created instances: none. Services/settings: the place must be published and Studio Access to API Services enabled to test in Studio. Host constraint: a server Script must call `require(game:GetService("ServerScriptService"):WaitForChild("PlayerDataService")).start()` once (repeat calls are ignored).

Other storage: `OrderedDataStore` (leaderboards), `MemoryStoreService` (fast, short-lived, cross-server), `MessagingService` (cross-server pub/sub, not for critical state).

## Monetization

- Set `MarketplaceService.ProcessReceipt` once, on the server. Return `Enum.ProductPurchaseDecision.PurchaseGranted` only after the reward is granted and saved; otherwise `NotProcessedYet`. Make grants idempotent by tracking `receiptInfo.PurchaseId`.
- Game passes: `MarketplaceService:UserOwnsGamePassAsync(userId, passId)` inside `pcall`; cache the result; handle `PromptGamePassPurchaseFinished`.
- Prompt from client or server, but always grant from the server.

## UI (client)

- Reach UI through `Players.LocalPlayer:WaitForChild("PlayerGui")`.
- Use `UDim2` Scale for responsive layouts with `UIAspectRatioConstraint`, `UIListLayout`, `UIGridLayout`, `UIPadding`, `UICorner`, `UIStroke`. Test on phone and tablet emulators; respect `ScreenGui.ScreenInsets`.
- Animate with `TweenService:Create(obj, TweenInfo.new(0.3, Enum.EasingStyle.Quad), { Position = goal }):Play()`.
- Input: `UserInputService` (raw) or `ContextActionService` (rebindable, mobile buttons). Ignore input when `gameProcessedEvent` is true.
- Set `ScreenGui.ResetOnSpawn = false` for persistent UI.

## Physics, raycasts, movement

```lua
--!strict
local function groundBelow(character: Model): RaycastResult?
	local root = character:FindFirstChild("HumanoidRootPart")
	if not root or not root:IsA("BasePart") then
		return nil
	end

	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Exclude
	params.FilterDescendantsInstances = { character }

	return workspace:Raycast(root.Position, Vector3.new(0, -50, 0), params)
end

return groundBelow
```

Requirements to run: ModuleScript at ReplicatedStorage/Shared/GroundCheck. Pre-created instances: a Shared folder in ReplicatedStorage. Services/settings: none. Host constraint: a Script or LocalScript must require it and pass a character.

- Position models with `PivotTo` / `GetPivot`; `Part.Position` ignores rotation.
- Anchored parts do not simulate physics. Set network ownership explicitly when you need server control.
- Use `workspace:GetPartBoundsInBox` / `GetPartBoundsInRadius` with `OverlapParams` for area checks instead of `Touched` spam.
- `Touched` fires repeatedly and for non-player parts; debounce it and verify with `Players:GetPlayerFromCharacter(hit.Parent)`.

## Performance

- Avoid `GetChildren()` / `GetDescendants()` in per-frame code; cache references.
- Prefer signals over `task.wait()` polling loops. Never write `while true do` without a yield.
- Pool or reuse frequently spawned objects (bullets, effects); `Destroy` the rest.
- Use `StreamingEnabled` for large maps.
- Keep remote traffic small: send IDs and deltas, batch updates.
- Profile first: MicroProfiler (Ctrl+F6) and Script Performance. Use `@native` only on measured hot numeric code.

## Debugging

- `print`, `warn`, `error`, `debug.traceback()`; Output window and Developer Console (F9) on live servers.
- Test with Local Server (Start Server + N players); many bugs only appear with a real client/server split.
- "Works in Studio, not live": missing `WaitForChild`, race with `CharacterAdded`, API Services off, assuming client changes replicate, StreamingEnabled.
- "Infinite yield possible" = wrong name or the instance never replicates. `attempt to index nil` = a missing child or table field; names are case-sensitive.

## Project tooling (optional)

- Rojo syncs a filesystem project to Studio. Suggested mapping: `src/server` to `ServerScriptService`, `src/client` to `StarterPlayer/StarterPlayerScripts`, `src/shared` to `ReplicatedStorage/Shared`, configured in `default.project.json`.
- Rojo suffixes: `.server.luau` (Script), `.client.luau` (LocalScript), `.luau` (ModuleScript), `init.luau` (folder as module).
- Wally for packages, Selene for lint, StyLua for formatting, luau-lsp for type checking.

## Response checklist

- [ ] Script type and location stated for every file
- [ ] No deprecated APIs from the table
- [ ] Server validates all remote input; no client-trusted results
- [ ] `pcall` around DataStore/Marketplace/HTTP calls, result checked
- [ ] Connections cleaned up; `PlayerRemoving` handled
- [ ] Handles players/characters that already exist at startup
- [ ] Types on public functions; `--!strict` on modules
- [ ] Short test instructions included
