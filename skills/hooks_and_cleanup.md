# Hook Infrastructure — Cleanup Registry & Safe Hook Wrapper

> Use when building any script that hooks functions, creates Drawing objects, or registers connections. Contains the _cleanup registry, _G.UNLOAD teardown, safe_hook pcall wrapper, and deferred module patching pattern.

**Source:** `skill.md v3` — sections §2  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 2. SAFE HOOK UTILITIES & CLEANUP SYSTEM

> Always build with cleanup in mind. Scripts without unload logic leak Drawing objects,
> RBXScriptConnections, and hooked functions permanently — detectable and memory-expensive.

### 2.1 Global Cleanup Registry

```lua
local _cleanup = {
    connections = {},
    drawings    = {},
    hooks       = {},
}

local function _track_conn(conn)
    table.insert(_cleanup.connections, conn)
    return conn
end

local function _track_draw(obj)
    table.insert(_cleanup.drawings, obj)
    return obj
end

local function _track_hook(fn, original)
    table.insert(_cleanup.hooks, { fn = fn, original = original })
end

_G.UNLOAD = function()
    for _, conn in ipairs(_cleanup.connections) do
        pcall(function() conn:Disconnect() end)
    end
    for _, d in ipairs(_cleanup.drawings) do
        pcall(function() d:Remove() end)
    end
    for _, h in ipairs(_cleanup.hooks) do
        pcall(function() hookfunction(h.fn, h.original) end)
    end
    table.clear(_cleanup.connections)
    table.clear(_cleanup.drawings)
    table.clear(_cleanup.hooks)
    print("[Unload] Clean.")
end
```

### 2.2 Safe Hook Wrapper

```lua
local function safe_hook(target_fn, replacement_fn)
    local ok, result = pcall(hookfunction, target_fn, newcclosure(replacement_fn))
    if not ok then
        warn("[safe_hook] hookfunction failed:", result)
        return nil
    end
    _track_hook(target_fn, result)
    return result
end
```

### 2.3 Deferred Patch with Per-Module Isolation

```lua
task.defer(function()
    task.wait(2)
    local rs = game:GetService("ReplicatedStorage")
    for _, obj in ipairs(rs:GetDescendants()) do
        if obj:IsA("ModuleScript") then
            local ok, mod = pcall(require, obj)
            if ok and type(mod) == "table" and not table.isfrozen(mod) then
                for k, v in pairs(mod) do
                    if type(v) == "number" and k:lower():find("damage") then
                        local prev = mod[k]
                        mod[k] = 99999
                        print(string.format("[Patch] %s.%s: %s → 99999",
                            obj:GetFullName(), k, tostring(prev)))
                    end
                end
            end
        end
    end
end)
```

---
