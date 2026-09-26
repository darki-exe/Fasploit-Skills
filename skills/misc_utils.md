# Script Loading, File I/O, UNC Portability & Anti-Detection Utilities

> Use when loading scripts from URLs or disk, checking which executor functions are available, writing portable multi-executor code, or applying anti-detection patterns. Covers loadstring, dofile, UNC capability check, portable clipboard/HTTP wrappers, jitter timing, newcclosure hygiene, and cloneref bypass.

**Source:** `skill.md v3` — sections §26, §27, §28, §29  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 26. LOADSTRING & REMOTE CODE EXECUTION

```lua
-- Execute a string as Lua bytecode
local fn, err = loadstring("print('executed') game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = 100")
if fn then fn() else warn(err) end

-- Load from URL (HttpGet / httpget)
local code = game:HttpGet("https://raw.githubusercontent.com/user/repo/main/script.lua")
loadstring(code)()

-- dofile — load and execute from executor workspace
dofile("my_scripts/init.lua")
```

---

---

## 27. FILE SYSTEM (EXECUTOR)

```lua
writefile("config.json",  '{"speed":100,"esp":true}')
local data = readfile("config.json")
print(data)

appendfile("log.txt", os.date() .. " — session started\n")

if isfile("config.json")   then print("File exists")   end
if isfolder("my_scripts")  then print("Folder exists") end

makefolder("my_scripts/configs")

local files = listfiles("./")
for _, f in ipairs(files) do print(f) end

delfile("old_config.json")
delfolder("old_folder")
```

---

---

## 28. UNC PORTABILITY PATTERNS

### 28.1 Check UNC Support

```lua
local function unc_check(name)
    return type(getgenv()[name]) == "function"
end

local supported = {
    hookfunction     = unc_check("hookfunction"),
    newcclosure      = unc_check("newcclosure"),
    getgc            = unc_check("getgc"),
    filtergc         = unc_check("filtergc"),
    getgenv          = unc_check("getgenv"),
    getrenv          = unc_check("getrenv"),
    getsenv          = unc_check("getsenv"),
    getreg           = unc_check("getreg"),
    getinstances     = unc_check("getinstances"),
    getnilinstances  = unc_check("getnilinstances"),
    getconnections   = unc_check("getconnections"),
    firesignal       = unc_check("firesignal"),
    Drawing          = type(Drawing) == "table",
    setreadonly      = unc_check("setreadonly"),
    getrawmetatable  = unc_check("getrawmetatable"),
    oth              = type(getgenv()["oth"]) == "table",
    RakNet           = type(getgenv()["RakNet"]) == "table",
    WebSocket        = type(getgenv()["WebSocket"]) == "table",
    crypt            = type(getgenv()["crypt"]) == "table",
}

for name, ok in pairs(supported) do
    print(string.format("[UNC] %-25s %s", name, ok and "✓" or "✗"))
end
```

### 28.2 Portable Clipboard

```lua
local set_clipboard = setclipboard or toclipboard or set_clipboard
    or (Clipboard and Clipboard.set)
    or function(s) warn("setclipboard not supported") end
set_clipboard("text")
```

### 28.3 Portable HTTP

```lua
local http_request = request or http_request
    or (http and http.request)
    or (syn and syn.request)
    or function() warn("http not supported") end

local res = http_request({ Url = "https://example.com", Method = "GET" })
print(res.StatusCode, res.Body)
```

---

---

## 29. ANTI-DETECTION TIPS

### 29.1 Jitter Timing

```lua
local function jitter(min_ms, max_ms)
    return task.wait(math.random(min_ms, max_ms) / 1000)
end
while true do
    some_remote:FireServer(args)
    jitter(80, 200)
end
```

### 29.2 `newcclosure` for All Hooks

```lua
-- Always wrap hook replacements. Prevents islclosure detection.
-- See §5.5 for examples.
```

### 29.3 BodyVelocity Over WalkSpeed

```lua
-- See §11.3. Direct WalkSpeed writes are flagged server-side.
```

### 29.4 `cloneref` + `compareinstances`

```lua
-- Some ACs check instance identity using == comparisons.
-- cloneref returns a new reference to the same object — bypasses pointer checks.
local safe_ref = cloneref(workspace.SomePart)
print(compareinstances(safe_ref, workspace.SomePart))  -- true, but different pointer
```

### 29.5 Off-Thread Hooking

```lua
-- Use oth.hook() for hooks that need to be less visible to thread-monitoring ACs.
-- See §19 for full reference.
```

---
