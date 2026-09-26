# Movement & Physics Bypass — Noclip, Fly, Infinite Jump, Teleport

> Use when implementing movement cheats. Covers noclip (CanCollide loop), fly with BodyVelocity/BodyGyro (WASD+Q/E, F to toggle), infinite jump via HumanoidStateType, teleport to player, and teleport to coordinates.

**Source:** `skill.md v3` — sections §24  
**Executor reference:** Delta v73801 (semi-universal, UNC-compliant naming)

---

## 24. MOVEMENT HACKS

### 24.1 Noclip

```lua
local noclip = false
_track_conn(game:GetService("RunService").Stepped:Connect(function()
    if not noclip then return end
    local char = game.Players.LocalPlayer.Character
    if not char then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then part.CanCollide = false end
    end
end))
noclip = true
```

### 24.2 Fly (WASD + Q/E, F to toggle)

```lua
local lp         = game.Players.LocalPlayer
local UIS        = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local flying = false; local fly_speed = 80
local body_vel, body_gyro

local function start_fly()
    local char = lp.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    char:FindFirstChildOfClass("Humanoid").PlatformStand = true
    body_vel = Instance.new("BodyVelocity", root)
    body_vel.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    body_vel.Velocity = Vector3.zero
    body_gyro = Instance.new("BodyGyro", root)
    body_gyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    body_gyro.D = 100
end

local function stop_fly()
    flying = false
    if body_vel  then body_vel:Destroy()  end
    if body_gyro then body_gyro:Destroy() end
    local char = lp.Character
    local hum  = char and char:FindFirstChildOfClass("Humanoid")
    if hum then hum.PlatformStand = false end
end

_track_conn(RunService.RenderStepped:Connect(function()
    if not flying or not body_vel or not body_gyro then return end
    local cam = workspace.CurrentCamera
    local dir = Vector3.zero
    if UIS:IsKeyDown(Enum.KeyCode.W) then dir = dir + cam.CFrame.LookVector  end
    if UIS:IsKeyDown(Enum.KeyCode.S) then dir = dir - cam.CFrame.LookVector  end
    if UIS:IsKeyDown(Enum.KeyCode.A) then dir = dir - cam.CFrame.RightVector end
    if UIS:IsKeyDown(Enum.KeyCode.D) then dir = dir + cam.CFrame.RightVector end
    if UIS:IsKeyDown(Enum.KeyCode.E) then dir = dir + Vector3.new(0,1,0)     end
    if UIS:IsKeyDown(Enum.KeyCode.Q) then dir = dir - Vector3.new(0,1,0)     end
    body_vel.Velocity = dir.Magnitude > 0 and dir.Unit * fly_speed or Vector3.zero
    body_gyro.CFrame  = cam.CFrame
end))

_track_conn(UIS.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F then
        flying = not flying
        if flying then start_fly() else stop_fly() end
    end
end))
```

### 24.3 Infinite Jump

```lua
_track_conn(game:GetService("UserInputService").JumpRequest:Connect(function()
    local char = game.Players.LocalPlayer.Character
    local hum  = char and char:FindFirstChildOfClass("Humanoid")
    if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
end))
```

### 24.4 Teleport

```lua
local function teleport_to(player_name)
    local target = game.Players:FindFirstChild(player_name)
    if not target or not target.Character then return end
    local root   = game.Players.LocalPlayer.Character
                   and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    local t_root = target.Character:FindFirstChild("HumanoidRootPart")
    if root and t_root then root.CFrame = t_root.CFrame + Vector3.new(3,0,0) end
end

local function teleport_pos(x, y, z)
    local root = game.Players.LocalPlayer.Character
                 and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if root then root.CFrame = CFrame.new(x, y, z) end
end
```

---
