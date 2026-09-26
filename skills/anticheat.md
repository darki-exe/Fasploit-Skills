# Anti-Cheat Detection & Evasion Patterns

> Use when identifying AC scripts in a game, watching for AC probes, disabling AC signal connections, or applying evasion strategies. Contains keyword-based AC scanner, DescendantAdded probe watcher, connection disabler, and a cheat-sheet of flagged vs safe techniques.

**Source:** `skill.md v3` — sections §21  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 21. ANTI-CHEAT DETECTION & EVASION

### 21.1 AC Script Detection

```lua
local ac_keywords = { "anticheat", "ac", "detection", "monitor", "integrity", "watchdog" }
for _, obj in ipairs(game:GetDescendants()) do
    if obj:IsA("ModuleScript") or obj:IsA("Script") or obj:IsA("LocalScript") then
        local name = obj.Name:lower()
        for _, kw in ipairs(ac_keywords) do
            if name:find(kw) then print("[AC Detected]", obj:GetFullName()) end
        end
    end
end
```

### 21.2 AC Probe Watcher

```lua
_track_conn(game.DescendantAdded:Connect(function(obj)
    if obj:IsA("RemoteFunction") then
        task.wait()
        print("[AC PROBE?] New RemoteFunction:", obj:GetFullName())
    end
end))
```

### 21.3 Disable AC Signal Connections

```lua
local target = game.ReplicatedStorage:FindFirstChild("IntegrityCheck")
if target then
    for _, conn in ipairs(getconnections(target.OnClientEvent)) do
        conn:Disable()
    end
end
```

### 21.4 Evasion Cheat-Sheet

```
Speed checks     → BodyVelocity instead of WalkSpeed (see §11.3)
Position delta   → Stay within server thresholds or use server-side teleport remotes
Health checks    → Hook TakeDamage server-side; never set Health client-side
Remote rate      → Jitter between calls (see §29.1)
islclosure check → Always wrap hooks in newcclosure
isexecutorclosure→ Use cloneref + newcclosure combo for stealth
oth detection    → Use oth.hook() to move hook logic off main thread (see §19)
tostring(fn) AC  → newcclosure already wraps; some executors support further spoofing
```

---
