# Hook Techniques — hookfunction, hookmetamethod, Closures, Signals

> Use when hooking any function, metamethod, or signal. Covers hookfunction / hookmetamethod (__namecall, __index, __newindex), newcclosure / newlclosure, replaceclosure / clonefunction, signal disabling via getconnections, and fire* trigger functions.

**Source:** `skill.md v3` — sections §5  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 5. HOOK TECHNIQUES — FULL REFERENCE

### 5.1 `hookfunction` / `hookfunc` / `detourfunction`

```lua
-- All three are aliases. hookfunction is UNC standard.
local old_td = safe_hook(
    game.Players.LocalPlayer.Character.Humanoid.TakeDamage,
    function(self, amount)
        return old_td(self, 0)  -- zero damage
    end
)
```

### 5.2 Hook Global Functions

```lua
local old_random = safe_hook(math.random, function(min, max)
    if max then return max end
    return 1
end)
```

### 5.3 `hookmetamethod` — `__namecall`

```lua
local mt = getrawmetatable(game)
setreadonly(mt, false)

local old_namecall = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
    local method = getnamecallmethod()
    if method == "TakeDamage" then return end  -- block damage
    if method == "FireServer"  then
        print("[spy] FireServer:", self:GetFullName(), ...)
    end
    return old_namecall(self, ...)
end))

setreadonly(mt, true)
_track_hook(old_namecall, old_namecall)
```

### 5.4 `hookmetamethod` — `__index` / `__newindex`

```lua
local mt = getrawmetatable(game)
setreadonly(mt, false)

-- Intercept property reads
local old_index = mt.__index
mt.__index = newcclosure(function(self, key)
    if key == "Health" and self:IsA("Humanoid") then return math.huge end
    return old_index(self, key)
end)

-- Intercept property writes
local old_newindex = mt.__newindex
mt.__newindex = newcclosure(function(self, key, value)
    if key == "WalkSpeed" then return old_newindex(self, key, 100) end
    return old_newindex(self, key, value)
end)

setreadonly(mt, true)
```

### 5.5 `newcclosure` / `newlclosure`

```lua
-- newcclosure: wraps a Lua function as a C closure — bypasses islclosure checks
local cfn = newcclosure(function() return true end)
print(iscclosure(cfn))  -- true
print(islclosure(cfn))  -- false

-- newlclosure: wraps as a Lua closure (less common use case)
local lfn = newlclosure(function() return true end)
print(islclosure(lfn))  -- true
```

### 5.6 `replaceclosure` / `clonefunction` / `restoreclosure`

```lua
local original = SomeModule.Attack
local cloned   = clonefunction(original)
SomeModule.Attack = cloned  -- same behavior, different pointer

-- In-place body swap
replaceclosure(SomeModule.Attack, function(self, ...)
    print("intercepted")
    return original(self, ...)
end)

-- Restore original
restoreclosure(SomeModule.Attack)
-- or: restorefunction / restorefunc
```

### 5.7 Signal Hooks via `getconnections`

```lua
local conns = getconnections(game.Players.LocalPlayer.Character.Humanoid.Died)
for _, conn in ipairs(conns) do
    conn:Disable()  -- pause AC death handler
end
```

### 5.8 `firesignal` / `firetouchinterest` / `fireclickdetector` / `fireproximityprompt`

```lua
firesignal(game.Players.LocalPlayer.Character.Humanoid.Died)

local root = game.Players.LocalPlayer.Character.HumanoidRootPart
local chest = workspace:FindFirstChild("TreasureChest")
if chest then firetouchinterest(root, chest, 0) end

local detector = workspace:FindFirstChildOfClass("ClickDetector", true)
if detector then fireclickdetector(detector, 0) end

local prompt = workspace:FindFirstChildOfClass("ProximityPrompt", true)
if prompt then fireproximityprompt(prompt) end

-- setproximitypromptduration / getproximitypromptduration
setproximitypromptduration(prompt, 0)  -- instant activation
```

---
