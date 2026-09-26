# ESP System — Full Player Overlay (Boxes, Names, Health, Tracers, Distance)

> Use when building a player ESP from scratch. Contains world_to_screen helper, full ESP object factory with box/name/health bar/tracer/distance label, per-player creation/removal, and a RenderStepped update loop with occlusion and distance filtering.

**Source:** `skill.md v3` — sections §23  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 23. ESP SYSTEM — FROM SCRATCH

### 23.1 World → Screen

```lua
local cam = workspace.CurrentCamera
local function world_to_screen(pos)
    local screen, on_screen = cam:WorldToViewportPoint(pos)
    return Vector2.new(screen.X, screen.Y), on_screen, screen.Z
end
```

### 23.2 Full Player ESP

```lua
local Players    = game:GetService("Players")
local RunService = game:GetService("RunService")
local lp         = Players.LocalPlayer
local cam        = workspace.CurrentCamera
local esp_objects = {}

local ESP_CONFIG = {
    boxes        = true,
    names        = true,
    health       = true,
    tracers      = true,
    distance     = true,
    team_check   = false,
    max_distance = 1000,
}

local function create_esp(player)
    local obj = {
        box    = _track_draw(Drawing.new("Square")),
        name   = _track_draw(Drawing.new("Text")),
        health = _track_draw(Drawing.new("Square")),
        hpfill = _track_draw(Drawing.new("Square")),
        tracer = _track_draw(Drawing.new("Line")),
        dist   = _track_draw(Drawing.new("Text")),
    }
    obj.box.Filled = false; obj.box.Thickness = 1
    obj.box.Color  = Color3.fromRGB(255, 50, 50); obj.box.Transparency = 1
    obj.name.Size  = 13; obj.name.Center = true; obj.name.Outline = true
    obj.name.Color = Color3.fromRGB(255,255,255); obj.name.Transparency = 1
    obj.health.Filled = true; obj.health.Color = Color3.fromRGB(50,50,50)
    obj.health.Transparency = 0.5
    obj.hpfill.Filled = true; obj.hpfill.Color = Color3.fromRGB(0,255,0)
    obj.hpfill.Transparency = 1
    obj.tracer.Thickness = 1; obj.tracer.Color = Color3.fromRGB(255,50,50)
    obj.tracer.Transparency = 0.7
    obj.dist.Size = 11; obj.dist.Center = true; obj.dist.Outline = true
    obj.dist.Color = Color3.fromRGB(200,200,200); obj.dist.Transparency = 1
    esp_objects[player] = obj
end

local function remove_esp(player)
    if not esp_objects[player] then return end
    for _, d in pairs(esp_objects[player]) do pcall(function() d:Remove() end) end
    esp_objects[player] = nil
end

local function hide_esp(obj)
    for _, d in pairs(obj) do d.Visible = false end
end

local function update_esp()
    for _, player in ipairs(Players:GetPlayers()) do
        if player == lp then continue end
        local obj  = esp_objects[player]
        if not obj then continue end
        local char = player.Character
        local hum  = char and char:FindFirstChildOfClass("Humanoid")
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local head = char and char:FindFirstChild("Head")
        if not (char and hum and root and head and hum.Health > 0) then
            hide_esp(obj) continue
        end
        local root_screen, root_on = world_to_screen(root.Position)
        local head_screen           = world_to_screen(head.Position + Vector3.new(0,0.5,0))
        local foot_screen           = world_to_screen(root.Position - Vector3.new(0,3,0))
        local lp_root = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
        if not lp_root then continue end
        local dist = (root.Position - lp_root.Position).Magnitude
        if not root_on or dist > ESP_CONFIG.max_distance then hide_esp(obj) continue end
        local box_h = math.abs(head_screen.Y - foot_screen.Y)
        local box_w = box_h * 0.6
        local box_x = root_screen.X - box_w / 2
        local box_y = head_screen.Y
        if ESP_CONFIG.boxes then
            obj.box.Visible = true
            obj.box.Position = Vector2.new(box_x, box_y)
            obj.box.Size     = Vector2.new(box_w, box_h)
        end
        if ESP_CONFIG.names then
            obj.name.Visible  = true
            obj.name.Text     = player.Name
            obj.name.Position = Vector2.new(root_screen.X, box_y - 16)
        end
        if ESP_CONFIG.distance then
            obj.dist.Visible  = true
            obj.dist.Text     = string.format("[%.0fm]", dist)
            obj.dist.Position = Vector2.new(root_screen.X, box_y + box_h + 2)
        end
        if ESP_CONFIG.health then
            local hp = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
            local bx = box_x - 6
            obj.health.Visible  = true
            obj.health.Position = Vector2.new(bx, box_y)
            obj.health.Size     = Vector2.new(4, box_h)
            obj.hpfill.Visible  = true
            obj.hpfill.Color    = Color3.fromRGB(
                math.floor(255*(1-hp)), math.floor(255*hp), 0)
            obj.hpfill.Position = Vector2.new(bx, box_y + box_h*(1-hp))
            obj.hpfill.Size     = Vector2.new(4, box_h*hp)
        end
        if ESP_CONFIG.tracers then
            local vp = cam.ViewportSize
            obj.tracer.Visible = true
            obj.tracer.From    = Vector2.new(vp.X/2, vp.Y)
            obj.tracer.To      = root_screen
        end
    end
end

_track_conn(Players.PlayerAdded:Connect(create_esp))
_track_conn(Players.PlayerRemoving:Connect(remove_esp))
for _, p in ipairs(Players:GetPlayers()) do
    if p ~= lp then create_esp(p) end
end
_track_conn(RunService.RenderStepped:Connect(update_esp))
```

---
