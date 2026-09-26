# Advanced APIs — Hidden Properties, Thread Identity, Off-Thread Hooks, Parallel Luau

> Use when reading/writing hidden instance properties, escalating thread identity level, hooking off the main thread to reduce AC visibility, or executing code inside Actor parallel contexts. Covers gethiddenproperty, setthreadidentity, oth.hook, get_actors, run_on_actor.

**Source:** `skill.md v3` — sections §18, §19, §20  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 18. HIDDEN PROPERTIES & THREAD IDENTITY

### 18.1 Hidden Properties

```lua
-- gethiddenproperty / get_hidden_property / gethiddenprop / gethiddenprops
local val, is_hidden = gethiddenproperty(workspace.SomePart, "PhysicsReceiveAge")
print("Value:", val, "Is hidden:", is_hidden)

-- get_hidden_properties(instance) -> table
-- Returns ALL hidden properties of an instance at once.
local props = get_hidden_properties(workspace.SomePart)
for k, v in pairs(props) do print(k, v) end

-- sethiddenproperty / set_hidden_property / sethiddenprop
sethiddenproperty(game.Players.LocalPlayer, "SimulationRadius", 1000)
```

### 18.2 Thread Identity

```lua
print(getthreadidentity())  -- 2 = LocalScript level
setthreadidentity(8)        -- CoreScript max
setthreadidentity(8)
local core = game:GetService("CoreGui")
```

---

---

## 19. OFF-THREAD HOOKING (`oth`)

```lua
-- oth (Off-Thread Hook) — hooks a function on a separate thread.
-- Reduces detection by moving hook logic off the main game thread.

-- oth.hook(target_fn, replacement_fn) -> original_fn
local old_attack = oth.hook(SomeModule.Attack, function(self, ...)
    print("[oth hook] Attack called")
    return 0
end)

-- oth.unhook(target_fn)
oth.unhook(SomeModule.Attack)

-- oth.is_hook_thread() -> bool
-- Returns true if the current thread is an oth hook thread.
print(oth.is_hook_thread())

-- oth.get_original_thread() -> thread
-- Returns the original game thread the hook was attached to.
local orig = oth.get_original_thread()

-- oth.get_root_callback(fn) -> function
-- Returns the root (original, unhooked) callback of a function.
local root = oth.get_root_callback(SomeModule.Attack)
```

---

---

## 20. ACTOR / PARALLEL LUAU

```lua
-- get_actors() | getactors() | getallactors()
-- Returns all Actor instances in the game client.
local actors = get_actors()
for _, a in ipairs(actors) do print(a.Name) end

-- get_current_actor() | getcurrentactor()
-- Returns the Actor the current script is running inside (if any).
local actor = get_current_actor()

-- get_deleted_actors() | getdeletedactors() | getdestroyedactors()
-- Returns Actor instances that were deleted during runtime.
local dead = get_deleted_actors()

-- run_on_actor(actor, code: string) | runonactor(actor, code)
-- Executes Lua code on a specific Actor.
run_on_actor(actors[1], [[
    print("Running on actor:", script:GetActor().Name)
]])

-- create_comm_channel() -> id, BindableEvent
-- Creates a communication channel between actors.
local id, event = create_comm_channel()
event.Event:Connect(function(msg) print("Actor message:", msg) end)

-- get_comm_channel(id) -> BindableEvent
local ch = get_comm_channel(id)
ch:Fire("hello from main thread")

-- checkparallel() | isparallel() -> bool
-- Returns true if currently executing in parallel (inside an Actor).
print(isparallel())

-- getallthreads() -> table
-- Returns all coroutine threads currently tracked by the executor.
for _, t in ipairs(getallthreads()) do
    print(coroutine.status(t))
end
```

---
