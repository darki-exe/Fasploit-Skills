# Environment Access & GC Scanning — getgenv, getrenv, getgc, filtergc

> Use when reading or patching script environments, scanning all live Lua objects, or finding a function/table by its upvalue or field. Covers getgenv, getrenv, getsenv, getreg, getmenv, getgc, filtergc, and upvalue/table hunting patterns.

**Source:** `skill.md v3` — sections §6, §7  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 6. ENVIRONMENT FUNCTIONS

### 6.1 `getgenv()` — Executor Global

```lua
getgenv().mySharedVar = "hello"
getgenv().myConfig    = { speed = 100 }
getgenv().myUtil      = function(x) return x * 2 end
print(getgenv().mySharedVar)
```

### 6.2 `getrenv()` — Roblox Global

```lua
local old_spawn = getrenv().spawn
getrenv().spawn = function(fn)
    print("[spawn hook]", fn)
    return old_spawn(fn)
end
```

### 6.3 `getsenv(script)` — Script Environment

```lua
for _, s in ipairs(getscripts()) do
    if s.Name == "MainGameScript" then
        local env = getsenv(s)
        env.coins   = 99999
        env.isAdmin = true
        break
    end
end
```

### 6.4 `getreg()` / `debug.getregistry()`

```lua
local reg = getreg()
for k, v in pairs(reg) do
    if type(v) == "function" then
        local info = debug.getinfo(v)
        if info and info.short_src then print(k, info.short_src) end
    end
end
```

### 6.5 `getmenv()` — Module Environment

```lua
-- Returns the environment table of a ModuleScript (Delta-specific)
local mod_env = getmenv(game.ReplicatedStorage.SomeModule)
print(mod_env.someInternalVar)
```

---

---

## 7. GC SCANNING (`getgc` / `filtergc`)

### 7.1 Basic GC Scan

```lua
-- getgc()        — functions + userdata
-- getgc(true)    — also includes tables (larger, slower)
local gc = getgc()
print("GC objects:", #gc)
```

### 7.2 `filtergc` — Targeted GC Scan

```lua
-- filtergc is faster than iterating getgc manually.
-- Filters by type and optional field/upvalue conditions.

-- Find all tables that have a "coins" key
local tables = filtergc("table", { Keys = { "coins" } })
for _, t in ipairs(tables) do
    t.coins = 99999
end

-- Find all functions from a specific source
local fns = filtergc("function", { IgnoreExecutor = true })
for _, fn in ipairs(fns) do
    local info = debug.getinfo(fn)
    if info and info.short_src and info.short_src:find("CombatModule") then
        print("Found:", info.name)
    end
end
```

### 7.3 Find Function by Upvalue

```lua
local function find_by_upvalue(target_name, target_value)
    local results = {}
    for _, fn in ipairs(getgc()) do
        if type(fn) == "function" then
            local i = 1
            while true do
                local name, val = debug.getupvalue(fn, i)
                if not name then break end
                if name == target_name and val == target_value then
                    table.insert(results, fn)
                end
                i = i + 1
            end
        end
    end
    return results
end

-- Patch all functions where upvalue "isAdmin" == false
for _, fn in ipairs(find_by_upvalue("isAdmin", false)) do
    local i = 1
    while true do
        local name = debug.getupvalue(fn, i)
        if not name then break end
        if name == "isAdmin" then debug.setupvalue(fn, i, true) break end
        i = i + 1
    end
end
```

### 7.4 Find Tables by Field in GC

```lua
for _, obj in ipairs(getgc(true)) do
    if type(obj) == "table" then
        if rawget(obj, "PlayerData") ~= nil and rawget(obj, "coins") ~= nil then
            obj.coins = 99999
            obj.gems  = 99999
        end
    end
end
```

---
