-- Silent Aimbot com FOV fixado na mira e ESP 2D Box Pequenas com Interface e Sliders
-- Compatível com executores como Delta (Roblox)

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Interface
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 250, 0, 140)
frame.Position = UDim2.new(0, 10, 0, 10)
frame.BackgroundTransparency = 0.3
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Active = true
frame.Draggable = true

local ESPEnabled = false
local AimbotEnabled = false
local FOVRadius = 78

-- Sliders
local espButton = Instance.new("TextButton", frame)
espButton.Size = UDim2.new(1, -20, 0, 30)
espButton.Position = UDim2.new(0, 10, 0, 10)
espButton.Text = "ESP: OFF"
espButton.MouseButton1Click:Connect(function()
    ESPEnabled = not ESPEnabled
    espButton.Text = "ESP: " .. (ESPEnabled and "ON" or "OFF")
end)

local aimButton = Instance.new("TextButton", frame)
aimButton.Size = UDim2.new(1, -20, 0, 30)
aimButton.Position = UDim2.new(0, 10, 0, 50)
aimButton.Text = "Aimbot: OFF"
aimButton.MouseButton1Click:Connect(function()
    AimbotEnabled = not AimbotEnabled
    aimButton.Text = "Aimbot: " .. (AimbotEnabled and "ON" or "OFF")
end)

-- FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
fovCircle.Radius = FOVRadius
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Thickness = 1
fovCircle.Transparency = 0.3
fovCircle.Filled = false
fovCircle.Visible = true

-- Silent Aimbot
function getClosestToCursor()
    local closest = nil
    local shortest = FOVRadius
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team and player.Character and player.Character:FindFirstChild("Head") then
            local pos, visible = Camera:WorldToViewportPoint(player.Character.Head.Position)
            local mousePos = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
            local dist = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude
            if visible and dist < shortest then
                shortest = dist
                closest = player
            end
        end
    end
    return closest
end

-- Hook de Tiro
local mt = getrawmetatable(game)
setreadonly(mt, false)
local oldNamecall = mt.__namecall
mt.__namecall = newcclosure(function(self, ...)
    local args = {...}
    local method = getnamecallmethod()

    if AimbotEnabled and tostring(self) == "FireServer" and method == "Namecall" and args[1] and typeof(args[1]) == "Vector3" then
        local target = getClosestToCursor()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            args[1] = target.Character.Head.Position
            return oldNamecall(self, unpack(args))
        end
    end
    return oldNamecall(self, ...)
end)

-- ESP 2D Pequeno
function drawESP(player)
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = Color3.fromRGB(255, 0, 0)
    box.Thickness = 1
    box.Transparency = 1
    box.Size = Vector2.new(20, 30)

    game:GetService("RunService").RenderStepped:Connect(function()
        if ESPEnabled and player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Team ~= LocalPlayer.Team then
            local pos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            if onScreen then
                box.Position = Vector2.new(pos.X - 10, pos.Y - 15)
                box.Visible = true
            else
                box.Visible = false
            end
        else
            box.Visible = false
        end
    end)
end

for _, p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        drawESP(p)
    end
end
Players.PlayerAdded:Connect(drawESP)

print("Interface com Silent Aimbot e ESP 2D carregada.")
