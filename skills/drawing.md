# Drawing Library — 2D Screen Rendering (Line, Circle, Square, Text, Quad)

> Use when rendering any 2D overlay on screen without a Roblox GUI. Covers all Drawing.new() types with every configurable property, font enum, and the _track_draw cleanup pattern. Required reading before building any ESP or HUD.

**Source:** `skill.md v3` — sections §22  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 22. DRAWING LIBRARY — FULL REFERENCE

> Renders 2D shapes directly onto the screen. Lightweight, no GUI overhead.

### Types

```lua
Drawing.new("Line")      -- straight line between two points
Drawing.new("Circle")    -- circle or filled disc
Drawing.new("Square")    -- rectangle (box ESP)
Drawing.new("Quad")      -- 4-point polygon
Drawing.new("Triangle")  -- 3-point polygon
Drawing.new("Text")      -- screen text label
Drawing.new("Image")     -- image from asset/data

-- Drawing.clear() — removes ALL drawing objects at once (nuclear option)
Drawing.clear()
```

### Font Reference

```lua
-- Drawing.Fonts enum:
-- UI        = 0  (default, clean)
-- System    = 1  (system font)
-- Plex      = 2  (IBM Plex Mono)
-- Monospace = 3  (monospaced)
```

### Line

```lua
local line = _track_draw(Drawing.new("Line"))
line.Visible      = true
line.From         = Vector2.new(0, 0)
line.To           = Vector2.new(500, 500)
line.Color        = Color3.fromRGB(255, 0, 0)
line.Thickness    = 1
line.Transparency = 1   -- 0=invisible, 1=opaque
line.ZIndex       = 1
```

### Circle

```lua
local circle = _track_draw(Drawing.new("Circle"))
circle.Visible      = true
circle.Position     = Vector2.new(500, 300)
circle.Radius       = 50
circle.Color        = Color3.fromRGB(0, 255, 0)
circle.Thickness    = 1
circle.Filled       = false   -- true = filled disc
circle.NumSides     = 64      -- smoothness
circle.Transparency = 1
```

### Square (Box)

```lua
local box = _track_draw(Drawing.new("Square"))
box.Visible      = true
box.Position     = Vector2.new(100, 100)  -- top-left
box.Size         = Vector2.new(200, 300)  -- width, height
box.Color        = Color3.fromRGB(255, 255, 0)
box.Thickness    = 1
box.Filled       = false
box.Transparency = 1
```

### Text

```lua
local label = _track_draw(Drawing.new("Text"))
label.Visible       = true
label.Text          = "Player [100 HP] 25m"
label.Position      = Vector2.new(200, 150)
label.Size          = 14
label.Color         = Color3.fromRGB(255, 255, 255)
label.Center        = true
label.Outline       = true
label.OutlineColor  = Color3.fromRGB(0, 0, 0)
label.Font          = Drawing.Fonts.UI
label.Transparency  = 1
```

### Quad

```lua
local quad = _track_draw(Drawing.new("Quad"))
quad.Visible      = true
quad.PointA       = Vector2.new(100, 100)
quad.PointB       = Vector2.new(200, 100)
quad.PointC       = Vector2.new(200, 300)
quad.PointD       = Vector2.new(100, 300)
quad.Color        = Color3.fromRGB(0, 100, 255)
quad.Thickness    = 1
quad.Filled       = false
quad.Transparency = 1
```

---
