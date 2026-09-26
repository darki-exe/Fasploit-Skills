# Debug Console (rconsole) & Input Simulation

> Use when opening an external console window for debug output, reading user input at runtime, or simulating mouse/keyboard input programmatically. Covers rconsolecreate/print/input/destroy and all mouse/keyboard simulation functions.

**Source:** `skill.md v3` — sections §16, §17  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 16. CONSOLE / RCONSOLE API

```lua
-- Opens an external console window (PC only).
rconsolecreate()           -- create | consolecreate
rconsolesettitle("MyScript")  -- set title | consolesettitle
rconsoleshow()             -- show window
rconsolehide()             -- hide window

-- Output
rconsoleprint("Hello from executor\n")   -- plain text
rconsoleinfo("Info message\n")           -- [INFO] prefix
rconsolewarn("Warning message\n")        -- [WARN] prefix
rconsoleerr("Error message\n")           -- [ERROR] prefix

-- Input (blocking — waits for user to type and press Enter)
local input = rconsoleinput()
print("User typed:", input)

-- Clear and destroy
rconsoleclear()
rconsoledestroy()
```

---

---

## 17. INPUT SIMULATION

```lua
-- Mouse buttons
mouse1click()    -- left click (press + release)
mouse1press()    -- left press
mouse1release()  -- left release
mouse2click()    -- right click
mouse2press()    -- right press
mouse2release()  -- right release

-- Mouse movement
mousemoveabs(960, 540)   -- absolute screen position
mousemoverel(10, -5)     -- relative offset from current position
mousescroll(1)           -- scroll up; -1 = scroll down

-- Keyboard
keypress(0x41)    -- press key (virtual key code; 0x41 = 'A')
keyrelease(0x41)  -- release key
keyclick(0x41)    -- press + release
keytap(0x41)      -- alias for keyclick

-- Virtual Key Code Reference (common):
-- 0x41-0x5A = A-Z
-- 0x30-0x39 = 0-9
-- 0x20 = Space, 0x0D = Enter, 0x1B = Escape
-- 0x10 = Shift, 0x11 = Ctrl, 0x12 = Alt
```

---
