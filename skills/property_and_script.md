# Property Manipulation & Script Introspection

> Use when modifying character stats, blocking TakeDamage, accessing non-scriptable properties, or inspecting running scripts. Covers speed/health persistence, BodyVelocity, setscriptable, render properties, script bytecode, closures, and thread mapping.

**Source:** `skill.md v3` — sections §11, §12  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 11. PROPERTY & INSTANCE MANIPULATION

### 11.1 Block TakeDamage via Metatable

```lua
local mt = getrawmetatable(game)
setreadonly(mt, false)
local old_namecall = mt.__namecall
mt.__namecall = function(self, ...)
    if getnamecallmethod() == "TakeDamage" then return end
    return old_namecall(self, ...)
end
setreadonly(mt, true)
```

### 11.2 Speed, Jump & Health with Persistence

```lua
local lp = game.Players.LocalPlayer

local function apply_stats(char)
    local hum = char:WaitForChild("Humanoid")
    hum.WalkSpeed = 100
    hum.JumpPower = 200
    hum.MaxHealth = math.huge
    hum.Health    = math.huge
end

if lp.Character then apply_stats(lp.Character) end
_track_conn(lp.CharacterAdded:Connect(apply_stats))
```

### 11.3 BodyVelocity Speed (Server-Evasion)

```lua
local root = game.Players.LocalPlayer.Character.HumanoidRootPart
local hum  = game.Players.LocalPlayer.Character.Humanoid
local bv = Instance.new("BodyVelocity")
bv.MaxForce = Vector3.new(math.huge, 0, math.huge)
bv.Velocity = hum.MoveDirection * 100
bv.Parent   = root
```

### 11.4 `setscriptable` — Make Non-Scriptable Properties Accessible

```lua
-- Some Roblox properties are marked "not scriptable" and can't be read/written normally.
-- setscriptable temporarily enables them.
setscriptable(workspace.CurrentCamera, "CameraType", true)
workspace.CurrentCamera.CameraType = Enum.CameraType.Scriptable
```

### 11.5 `getrenderproperty` / `setrenderproperty`

```lua
-- Access render-side properties (not replicated, client-only).
local val = getrenderproperty(workspace.SomePart, "Level0Bias")
print(val)
setrenderproperty(workspace.SomePart, "Level0Bias", 0)
```

---

---

## 12. SCRIPT INTROSPECTION

```lua
-- getscripts() — all scripts in game
for _, s in ipairs(getscripts()) do
    print(s.ClassName, s:GetFullName())
end

-- getloadedmodules() — only required ModuleScripts
for _, m in ipairs(getloadedmodules()) do
    local ok, result = pcall(require, m)
    if ok and type(result) == "table" and not table.isfrozen(result) then
        print("Mutable module:", m.Name)
    end
end

-- getscriptbytecode(script) -> string
local bytecode = getscriptbytecode(
    game.Players.LocalPlayer.PlayerScripts:FindFirstChild("MainScript"))
print("Bytecode length:", #bytecode)

-- getscriptclosure(script) -> function
-- Returns the actual closure of the script — inspect upvalues, constants, protos.
local closure = getscriptclosure(
    game.Players.LocalPlayer.PlayerScripts:FindFirstChild("MainScript"))
dump_upvalues(closure)

-- getscriptfromthread(thread) -> script
-- Returns the script a given coroutine thread belongs to.
local s = getscriptfromthread(coroutine.running())
print(s and s.Name or "no script")

-- getscriptthread(script) -> thread
-- Returns the main coroutine thread of a script.
local thread = getscriptthread(game.Players.LocalPlayer.PlayerScripts.MainScript)
print(coroutine.status(thread))

-- getrunningscripts() | getscriptsthatrun()
-- Returns only scripts that are currently executing (not just present).
for _, s in ipairs(getrunningscripts()) do
    print("Running:", s.Name)
end

-- getcallingscript() -> script
-- Returns the script that invoked the currently executing function.
print(getcallingscript())
```

---
