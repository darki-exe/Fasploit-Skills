# Table & Module Patching — Frozen and Unfrozen

> Use when patching values inside a ModuleScript — whether the table is mutable or frozen. Covers direct key overwrite, proxy tables for frozen modules, metatable hijack, nested table injection, cache.replace spoofing, and require() interception.

**Source:** `skill.md v3` — sections §3, §4  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 3. TABLE MANIPULATION (UNFROZEN & FROZEN)

### 3.1 Check Mutability

```lua
if not table.isfrozen(SomeModule) then
    print("mutable — open for writes")
else
    print("frozen — proxy required")
end
```

### 3.2 Direct Key Overwrite

```lua
local mod = require(game.ReplicatedStorage.SomeModule)
mod.Damage    = 9999
mod.WalkSpeed = 500
mod.MaxHealth = math.huge
mod.CalculateDamage = function() return 99999 end
```

### 3.3 Nested Table Injection

```lua
local mod = require(game.ReplicatedStorage.Config)
for key, value in pairs(mod) do
    if type(value) == "table" then
        for subkey, subval in pairs(value) do
            if type(subval) == "number" then
                value[subkey] = subval * 100
            end
        end
    end
end
```

### 3.4 Metatable Hijack (Unfrozen)

```lua
local mod = require(game.ReplicatedStorage.SomeModule)
local original_meta  = getmetatable(mod) or {}
local original_index = original_meta.__index

setmetatable(mod, {
    __index = function(t, k)
        if k == "Health" then return math.huge end
        if k == "Speed"  then return 500 end
        if type(original_index) == "function" then return original_index(t, k) end
        if type(original_index) == "table"    then return original_index[k] end
        return rawget(t, k)
    end,
    __newindex = function(t, k, v) rawset(t, k, v) end
})
```

### 3.5 Proxy Table (Frozen Module Bypass)

```lua
local frozen_mod = require(game.ReplicatedStorage.FrozenModule)

local proxy = setmetatable({}, {
    __index = function(t, k)
        if k == "Damage"    then return 99999 end
        if k == "WalkSpeed" then return 500   end
        return frozen_mod[k]
    end,
    __newindex = function(t, k, v) end
})
```

### 3.6 Cache Replace (Instance Spoofing)

```lua
-- cache.replace swaps what scripts "see" when they reference an instance.
-- Useful for feeding fake module tables to scripts that require() specific paths.
local fake = Instance.new("ModuleScript")
fake.Name   = "Config"
fake.Parent = nil

cache.replace(game.ReplicatedStorage.Config, fake)
-- Now any script that does require(game.ReplicatedStorage.Config)
-- gets your fake instead — without touching the original.
```

---

---

## 4. MODULE REQUIRE INTERCEPTION

### 4.1 Hook `require()` Globally

```lua
local old_require = require
require = function(module)
    local result = old_require(module)
    if type(result) == "table" and not table.isfrozen(result) then
        if result.Damage then result.Damage = 9999 end
        if result.Speed  then result.Speed  = 500  end
    end
    return result
end
```

### 4.2 Hook Proto Directly

```lua
-- hookproto(fn, replacement) — hooks at the bytecode proto level
-- Affects every closure sharing the same proto (multiple instances of the same function)
hookproto(SomeModule.Attack, newcclosure(function(self, ...)
    print("[proto hook] Attack called")
    return 0
end))
```

---
