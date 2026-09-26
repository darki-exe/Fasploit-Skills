# Silent Aim — FOV-Based Target Snapping via Metatable Hook

> Use when implementing silent aim (bullet magnetism without visible cursor movement). Hooks WorldToViewportPoint on the camera metatable, snaps to the closest player within an FOV radius, and renders an FOV circle overlay.

**Source:** `skill.md v3` — sections §25  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 25. SILENT AIM

```lua
local cam     = workspace.CurrentCamera
local Players = game:GetService("Players")
local lp      = Players.LocalPlayer
local silent_aim = true
local fov_radius = 150

local function get_closest_player()
    local vp     = cam.ViewportSize
    local center = Vector2.new(vp.X/2, vp.Y/2)
    local closest, closest_dist, closest_root = nil, math.huge, nil
    for _, p in ipairs(Players:GetPlayers()) do
        if p == lp or not p.Character then continue end
        local root = p.Character:FindFirstChild("HumanoidRootPart")
        local hum  = p.Character:FindFirstChildOfClass("Humanoid")
        if not root or not hum or hum.Health <= 0 then continue end
        local screen, on = cam:WorldToViewportPoint(root.Position)
        if not on then continue end
        local d = (Vector2.new(screen.X, screen.Y) - center).Magnitude
        if d < fov_radius and d < closest_dist then
            closest = p; closest_dist = d; closest_root = root
        end
    end
    return closest, closest_root
end

local mt = getrawmetatable(cam)
setreadonly(mt, false)
local old_wtvp = mt.__index
mt.__index = newcclosure(function(self, key)
    if key == "WorldToViewportPoint" then
        return function(_, pos)
            if silent_aim then
                local target, root = get_closest_player()
                if target and root then
                    local head = target.Character:FindFirstChild("Head")
                    if head then pos = head.Position end
                end
            end
            return old_wtvp(self, key)(self, pos)
        end
    end
    return old_wtvp(self, key)
end)
setreadonly(mt, true)

-- FOV Circle
local fov_circle = _track_draw(Drawing.new("Circle"))
fov_circle.Radius = 150; fov_circle.Filled = false; fov_circle.Thickness = 1
fov_circle.Color  = Color3.fromRGB(255,255,255); fov_circle.Transparency = 0.8
fov_circle.NumSides = 64; fov_circle.Visible = true

_track_conn(game:GetService("RunService").RenderStepped:Connect(function()
    local vp = cam.ViewportSize
    fov_circle.Position = Vector2.new(vp.X/2, vp.Y/2)
end))
```

---
