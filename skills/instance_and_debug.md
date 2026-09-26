# Instance Inspection & Debug Library — Upvalues, Constants, Protos

> Use when enumerating instances (including hidden and nil-parented), or when reading/patching a function's internal state. Covers getinstances, getnilinstances, gethui, debug.getupvalue/setupvalue, debug.getconstants/setconstant, protos, stack, decompile.

**Source:** `skill.md v3` — sections §8, §9  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 8. INSTANCE INSPECTION

### 8.1 `getinstances()` — All Instances

```lua
for _, inst in ipairs(getinstances()) do
    if inst:IsA("ScreenGui") and
       not inst:IsDescendantOf(game.Players.LocalPlayer.PlayerGui) then
        print("Hidden GUI:", inst:GetFullName())
    end
    if inst:IsA("RemoteEvent") and
       not inst:IsDescendantOf(game.ReplicatedStorage) then
        print("Hidden remote:", inst:GetFullName())
    end
end
```

### 8.2 `getnilinstances()` / `get_nil_instances()` — Nil-Parented

```lua
for _, inst in ipairs(getnilinstances()) do
    print("[nil parent]", inst.ClassName, inst.Name)
    if inst:IsA("LocalScript") or inst:IsA("Script") then
        print("  >> SCRIPT in nil:", inst.Name)
    end
    if inst:IsA("RemoteEvent") or inst:IsA("RemoteFunction") then
        print("  >> REMOTE in nil:", inst.Name)
    end
end
```

### 8.3 `getinstancecache()` — Internal Instance Cache

```lua
-- Returns the executor's internal cache of Roblox instances.
-- Useful for finding references that have been cache.replace()'d.
local cache_table = getinstancecache()
for _, inst in ipairs(cache_table) do
    print(inst.ClassName, inst.Name)
end
```

### 8.4 `gethui()` / `get_hidden_gui()` — Hidden CoreGui

```lua
-- Returns a hidden ScreenGui parented outside normal PlayerGui hierarchy.
-- Used by executors to render GUIs that survive game GUI resets.
local hidden_gui = gethui()
local frame = Instance.new("Frame")
frame.Parent = hidden_gui
```

---

---

## 9. DEBUG LIBRARY — FULL EXPLOIT USAGE

### 9.1 Dump & Patch Upvalues

```lua
local function dump_upvalues(fn)
    -- getupvalues returns a table of all upvalues at once (Delta extension)
    local ups = debug.getupvalues(fn)
    for i, val in pairs(ups) do
        print(string.format("  [%d] = %s (%s)", i, tostring(val), type(val)))
    end
end

-- Patch a specific upvalue by name
local function patch_upvalue(fn, target_name, new_value)
    local i = 1
    while true do
        local name, _ = debug.getupvalue(fn, i)
        if not name then break end
        if name == target_name then
            debug.setupvalue(fn, i, new_value)
            return true
        end
        i = i + 1
    end
    return false
end

-- Multiply all numeric upvalues
local function patch_numeric_upvalues(fn, multiplier)
    local i = 1
    while true do
        local name, val = debug.getupvalue(fn, i)
        if not name then break end
        if type(val) == "number" then
            debug.setupvalue(fn, i, val * multiplier)
            print("Patched:", name, val, "→", val * multiplier)
        end
        i = i + 1
    end
end
```

### 9.2 Constants

```lua
local function dump_constants(fn)
    local consts = debug.getconstants(fn)
    for i, c in ipairs(consts) do
        print(string.format("  [%d] %s (%s)", i, tostring(c), type(c)))
    end
end

local function patch_constant(fn, old_val, new_val)
    for i, c in ipairs(debug.getconstants(fn)) do
        if c == old_val then
            debug.setconstant(fn, i, new_val)
            print("Patched constant:", old_val, "→", new_val)
        end
    end
end
```

### 9.3 Protos (Sub-Function References)

```lua
local function walk_protos(fn, depth)
    depth = depth or 0
    for i, proto in ipairs(debug.getprotos(fn)) do
        local info = debug.getinfo(proto)
        print(string.rep("  ", depth) .. "[proto "..i.."]", info and info.short_src or "?")
        walk_protos(proto, depth + 1)
    end
end
```

### 9.4 Stack

```lua
local function dump_stack(level)
    level = level or 1
    local i = 1
    while true do
        local val = debug.getstack(level, i)
        if val == nil then break end
        print(string.format("  stack[%d][%d] = %s", level, i, tostring(val)))
        i = i + 1
    end
end

-- debug.getcallstack() — returns a string traceback of the current call stack
print(debug.getcallstack())
```

### 9.5 `decompile` / `dumpbytecode`

```lua
-- decompile(fn or script) -> string
-- Attempts to decompile a function or script back to readable Lua source.
local src = decompile(game.Players.LocalPlayer.PlayerScripts.MainScript)
print(src)

-- dumpbytecode / dumpstring — dumps raw bytecode as a string
local bytecode = dumpbytecode(SomeModule.Attack)
print(#bytecode, "bytes")
```

---
