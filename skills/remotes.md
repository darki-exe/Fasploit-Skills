# Remote Spy, Replay & Fuzzing — RemoteEvent / RemoteFunction

> Use when intercepting, logging, replaying, or fuzzing RemoteEvent / RemoteFunction calls. Covers remote spy hook, replay with argument override, argument fuzzing with edge-case types, and blocking specific remotes by name.

**Source:** `skill.md v3` — sections §10  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 10. REMOTE EVENT & FUNCTION TECHNIQUES

### 10.1 Remote Spy

```lua
local remote_log = {}

local function hook_remote(remote)
    if remote:IsA("RemoteEvent") then
        local old_fire = remote.FireServer
        remote.FireServer = function(self, ...)
            local args = {...}
            table.insert(remote_log, {
                name = remote.Name,
                path = remote:GetFullName(),
                args = args,
                time = tick()
            })
            print("[RemoteSpy] FireServer:", remote:GetFullName(), unpack(args))
            return old_fire(self, ...)
        end
    elseif remote:IsA("RemoteFunction") then
        local old_invoke = remote.InvokeServer
        remote.InvokeServer = function(self, ...)
            print("[RemoteSpy] InvokeServer:", remote:GetFullName(), ...)
            return old_invoke(self, ...)
        end
    end
end

for _, obj in ipairs(game:GetDescendants()) do
    if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then hook_remote(obj) end
end

_track_conn(game.DescendantAdded:Connect(function(obj)
    if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
        task.wait()
        hook_remote(obj)
    end
end))
```

### 10.2 Replay

```lua
local function replay(entry, override_args)
    local remote = game:FindFirstChild(entry.path, true)
    if not remote then warn("Remote not found:", entry.path) return end
    local args = override_args or entry.args
    if remote:IsA("RemoteEvent") then
        remote:FireServer(unpack(args))
    elseif remote:IsA("RemoteFunction") then
        return remote:InvokeServer(unpack(args))
    end
end

local last = remote_log[#remote_log]
if last then replay(last, { "modified_arg", 9999 }) end
```

### 10.3 Argument Fuzzing

```lua
local fuzz_types = {
    0, -1, math.huge, math.huge * -1, 2^53,
    "", "admin", "nil", "true",
    true, false, nil,
    {}, game.Players.LocalPlayer,
    CFrame.new(), Vector3.new(), UDim2.new()
}

local function fuzz_remote(remote)
    for _, val in ipairs(fuzz_types) do
        pcall(function()
            if remote:IsA("RemoteEvent") then remote:FireServer(val)
            elseif remote:IsA("RemoteFunction") then remote:InvokeServer(val) end
        end)
        task.wait(0.1)
    end
end
```

### 10.4 Block Specific Remotes

```lua
local blocked = { "AntiCheat", "ReportPosition", "HeartbeatSync" }
local function should_block(name)
    for _, b in ipairs(blocked) do
        if name:lower():find(b:lower()) then return true end
    end
    return false
end
for _, obj in ipairs(game:GetDescendants()) do
    if obj:IsA("RemoteEvent") and should_block(obj.Name) then
        obj.FireServer = function() end
    end
end
```

---
