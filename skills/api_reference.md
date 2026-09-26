# Executor API — Full Function Reference (Delta v73801)

> Use when looking up what a specific executor function does, its aliases, parameters, or return values. Covers file I/O, HTTP, WebSocket, thread identity, FPS cap, signals, cache, closures, and misc utilities.

**Source:** `skill.md v3` — sections §1  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 1. BASIC EXECUTOR API REFERENCE

> What each executor function actually does — plain descriptions, usage, and examples.
> Sourced from Delta v73801 dump. Most of these exist on all major executors under the same or aliased names.

---

### 1.1 File System

```lua
-- writefile(path: string, content: string)
-- Creates or overwrites a file in the executor workspace folder.
writefile("config.json", '{"speed":100,"esp":true}')

-- readfile(path: string) -> string
-- Reads and returns file contents as a string.
local data = readfile("config.json")
print(data)

-- appendfile(path: string, content: string)
-- Appends text to the end of an existing file. Creates it if it doesn't exist.
appendfile("log.txt", os.date() .. " — session started\n")

-- isfile(path: string) -> bool
-- Returns true if a file exists at the given path.
if isfile("config.json") then
    print("Config exists")
end

-- isfolder(path: string) -> bool
-- Returns true if a folder exists at the given path.
if not isfolder("my_scripts") then
    makefolder("my_scripts")
end

-- makefolder(path: string)
-- Creates a folder in the executor workspace.
makefolder("my_scripts/configs")

-- listfiles(path: string) -> table
-- Returns a list of file/folder paths inside the given directory.
local files = listfiles("./")
for _, f in ipairs(files) do print(f) end

-- delfile(path: string) | delfolder(path: string) | deletefolder(path: string)
-- Deletes a file or folder. All three names are aliases on Delta.
delfile("old_config.json")
delfolder("old_folder")

-- loadfile(path: string) -> function?, string?
-- Loads a file as a Lua chunk (like loadstring but from disk).
local fn, err = loadfile("my_scripts/init.lua")
if fn then fn() else warn(err) end

-- dofile(path: string)
-- Loads and immediately executes a file from the workspace.
dofile("my_scripts/init.lua")
```

---

### 1.2 Clipboard

```lua
-- setclipboard(text: string) | set_clipboard(text) | toclipboard(text)
-- Copies a string to the system clipboard. All three are aliases.
setclipboard("copied text here")

-- set_rbx_clipboard(rbxobj) | setrbxclipboard(rbxobj)
-- Copies a Roblox object reference to the clipboard (for paste into Studio).
set_rbx_clipboard(workspace.SomePart)
```

---

### 1.3 HTTP & Requests

```lua
-- request(config: table) -> table | http_request(config) | http.request(config)
-- Sends an HTTP request from the executor. Returns { StatusCode, Body, Headers }.
local res = request({
    Url     = "https://example.com/api",
    Method  = "GET",
    Headers = { ["Content-Type"] = "application/json" },
    Body    = "",  -- for POST
})
print(res.StatusCode, res.Body)

-- httpget(url: string) -> string
-- Simple GET shorthand. Returns the body directly.
local html = httpget("https://example.com")

-- httppost(url: string, body: string) -> string
-- Simple POST shorthand.
local response = httppost("https://example.com/api", '{"key":"value"}')
```

---

### 1.4 WebSocket

```lua
-- WebSocket.connect(url: string) -> WebSocket
-- Opens a WebSocket connection. Returns a connection object with events.
local ws = WebSocket.connect("ws://localhost:8080")

ws.OnMessage:Connect(function(msg)
    print("Received:", msg)
end)

ws.OnClose:Connect(function()
    print("WebSocket closed")
end)

ws:Send("hello from executor")
ws:Close()
```

---

### 1.5 Identity & Thread Context

```lua
-- getthreadidentity() | get_thread_identity() | getidentity() | get_thread_context()
-- Returns the current execution identity level (2 = LocalScript, 8 = CoreScript max).
print(getthreadidentity())  -- 2

-- setthreadidentity(n) | set_thread_identity(n) | setidentity(n)
-- Sets the execution identity. Level 8 = CoreScript permissions.
setthreadidentity(8)
local core = game:GetService("CoreGui")  -- accessible at level 8
print(core:GetChildren())
```

---

### 1.6 Executor Info

```lua
-- getexecutorname() | identifyexecutor()
-- Returns the name and optionally version of the current executor.
print(getexecutorname())  -- "Delta"
print(identifyexecutor()) -- "Delta", "73801"

-- Delta-specific version info
print(Delta.version())       -- version string
print(Delta.version_num())   -- version number
print(Delta.architecture_str()) -- "x64" / "arm64" / etc.
print(Delta.is_android())    -- true/false
print(Delta.is_ios())        -- true/false
print(Delta.is_mac())        -- true/false
print(Delta.roblox_version()) -- current Roblox version string

-- gethwid() — returns the hardware ID of the device (used for licensing)
print(gethwid())
```

---

### 1.7 FPS Cap

```lua
-- getfpscap() | get_fps_cap() -> number
-- Returns the current FPS cap.
print(getfpscap())  -- e.g. 60

-- setfpscap(n) | set_fps_cap(n)
-- Sets the FPS cap. Useful for unlocking beyond Roblox's default 60.
setfpscap(240)
setfpscap(0)  -- uncapped
```

---

### 1.8 Misc Utility

```lua
-- checkcaller() -> bool
-- Returns true if the current function was called from the executor environment.
print(checkcaller())

-- isrbxactive() | isgameactive() | iswindowactive() -> bool
-- Returns true if the Roblox window is in focus.
if isrbxactive() then
    -- safe to fire inputs
end

-- getsimulationradius() -> number
-- Returns the current simulation radius of the local player.
print(getsimulationradius())

-- setsimulationradius(n)
-- Sets simulation radius — affects what the client simulates.
setsimulationradius(1000)

-- setfflag(name: string, value: string)
-- Overrides a Roblox FFlag (feature flag) value.
setfflag("DFIntTaskSchedulerTargetFps", "240")

-- getfflag(name: string) -> string
-- Reads a Roblox FFlag value.
print(getfflag("DFIntTaskSchedulerTargetFps"))

-- messagebox(text: string, title: string, flags: number) -> number
-- Shows a Windows message box dialog (PC only).
messagebox("Script loaded!", "Notice", 0)

-- getmousepos() -> Vector2
-- Returns the current mouse position on screen.
local pos = getmousepos()
print(pos.X, pos.Y)

-- getfunctionhash(fn: function) -> string
-- Returns a hash of a function's bytecode. Useful for identity checking.
print(getfunctionhash(SomeModule.Attack))

-- getscripthash(script: LuaSourceContainer) -> string
-- Returns a hash of a script's bytecode.
print(getscripthash(game.Players.LocalPlayer.PlayerScripts.MainScript))

-- queue_on_teleport(code: string) | queueonteleport(code)
-- Queues a script string to execute after the player teleports to a new place.
queue_on_teleport([[
    print("Loaded after teleport")
]])

-- clear_teleport_queue() | clearteleportqueue()
-- Clears all queued teleport scripts.
clear_teleport_queue()

-- setnamecallmethod(name: string)
-- Sets the __namecall method name (used inside __namecall hooks).
-- Normally read with getnamecallmethod() inside a hook.

-- cloneref(obj: Instance) -> Instance | clonereference(obj)
-- Returns a separate reference to the same Instance.
-- Helps bypass =="comparison checks used by some ACs.
local cloned = cloneref(workspace.SomePart)

-- compareinstances(a: Instance, b: Instance) -> bool
-- Returns true if two references point to the same underlying instance.
print(compareinstances(cloned, workspace.SomePart))  -- true

-- isnetworkowner(part: BasePart) -> bool
-- Returns true if the local client has network ownership of the part.
print(isnetworkowner(workspace.SomePart))
```

---

### 1.9 Closure Inspection

```lua
-- islclosure(fn) -> bool    — is it a Lua closure?
-- iscclosure(fn) -> bool    — is it a C closure?
-- isexecutorclosure(fn) -> bool | is_executor_closure(fn) | isourclosure(fn)
-- Returns true if the function belongs to the executor environment.
print(islclosure(print))           -- false (C function)
print(iscclosure(print))           -- true
print(isexecutorclosure(newcclosure(function() end)))  -- true

-- checkclosure(fn) -> bool
-- Checks if the function is a valid callable closure.
print(checkclosure(SomeModule.Attack))

-- isfunctionhooked(fn) -> bool | is_function_hooked(fn) | ishooked(fn)
-- Returns true if the function has been hooked.
print(isfunctionhooked(SomeModule.Attack))

-- isprotohooked(fn) -> bool
-- Returns true if the function's proto (bytecode body) has been hooked.
print(isprotohooked(SomeModule.Attack))

-- restoreclosure(fn) | restorefunction(fn) | restorefunc(fn)
-- Restores a hooked function to its original state.
restoreclosure(SomeModule.Attack)

-- restoreproto(fn)
-- Restores the proto of a hooked function.
restoreproto(SomeModule.Attack)
```

---

### 1.10 Cache

```lua
-- cache.invalidate(obj: Instance)
-- Removes an instance from the internal instance cache.
-- After this, require() or FindFirstChild() returns a fresh reference.
cache.invalidate(game.ReplicatedStorage.SomeModule)

-- cache.iscached(obj: Instance) -> bool
-- Returns true if the instance is currently in the cache.
print(cache.iscached(game.ReplicatedStorage.SomeModule))

-- cache.replace(obj: Instance, newObj: Instance)
-- Replaces a cached instance reference with another.
-- Scripts that reference the old object now silently get the new one.
cache.replace(game.ReplicatedStorage.SomeModule, myFakeModule)
```

---

### 1.11 Signals

```lua
-- getconnections(signal) -> table
-- Returns all RBXScriptConnections on a signal.
local conns = getconnections(game.Players.LocalPlayer.Character.Humanoid.Died)
for _, conn in ipairs(conns) do
    print(conn.Function, conn.Thread)
    conn:Disable()   -- pause
    conn:Enable()    -- resume
    conn:Fire()      -- trigger manually
end

-- isconnectionenabled(conn) -> bool
print(isconnectionenabled(conns[1]))

-- setconnectionenabled(conn, bool)
setconnectionenabled(conns[1], false)

-- cansignalreplicate(signal) -> bool
-- Returns true if the signal can replicate to the server.
print(cansignalreplicate(someRemote.OnClientEvent))

-- replicatesignal(signal, ...)
-- Replicates a signal firing to the server.
replicatesignal(someRemote.OnClientEvent, arg1, arg2)

-- firesignal(signal, ...) — fires a signal directly
firesignal(game.Players.LocalPlayer.Character.Humanoid.Died)

-- DeltaSignal.new() — creates a custom executor-side signal
local sig = DeltaSignal.new()
sig:Connect(function(val) print("Got:", val) end)
sig:Fire(42)
```

---
