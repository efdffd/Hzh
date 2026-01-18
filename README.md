local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LP = Players.LocalPlayer
local Mouse = LP:GetMouse()

local ESP_ON = false
local AIM_ON = false
local UI_OPEN = true
local ESP_CACHE = {}

local function clearESP()
    for _,v in pairs(ESP_CACHE) do
        pcall(function() v:Destroy() end)
    end
    ESP_CACHE = {}
end

local function addESP(p)
    if p == LP then return end
    if not p.Character then return end

    if ESP_CACHE[p] then ESP_CACHE[p]:Destroy() end

    local h = Instance.new("Highlight")
    h.Adornee = p.Character
    h.FillTransparency = 1
    h.OutlineTransparency = 0
    h.OutlineColor = Color3.fromRGB(0,255,0)
    h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    h.Parent = game.CoreGui

    ESP_CACHE[p] = h
end

local function enableESP()
    for _,p in ipairs(Players:GetPlayers()) do
        addESP(p)
        p.CharacterAdded:Connect(function()
            task.wait(0.3)
            if ESP_ON then addESP(p) end
        end)
    end
end

local function getClosest()
    local closest, dist = nil, math.huge
    for _,p in ipairs(Players:GetPlayers()) do
        if p ~= LP and p.Character and p.Character:FindFirstChild("Head") then
            local pos, vis = Camera:WorldToViewportPoint(p.Character.Head.Position)
            if vis then
                local d = (Vector2.new(pos.X,pos.Y) - Vector2.new(Mouse.X,Mouse.Y)).Magnitude
                if d < dist then
                    dist = d
                    closest = p
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    if AIM_ON then
        local t = getClosest()
        if t and t.Character and t.Character:FindFirstChild("Head") then
            Camera.CFrame = Camera.CFrame:Lerp(
                CFrame.new(Camera.CFrame.Position, t.Character.Head.Position),
                0.15
            )
        end
    end
end)

local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "XenoUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,200,0,120)
frame.Position = UDim2.new(0.05,0,0.4,0)
frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
frame.Active = true
frame.Draggable = true

local espBtn = Instance.new("TextButton", frame)
espBtn.Size = UDim2.new(1,0,0.5,0)
espBtn.Text = "ESP OFF"
espBtn.BackgroundColor3 = Color3.fromRGB(35,35,35)
espBtn.TextColor3 = Color3.new(1,1,1)

local aimBtn = Instance.new("TextButton", frame)
aimBtn.Size = UDim2.new(1,0,0.5,0)
aimBtn.Position = UDim2.new(0,0,0.5,0)
aimBtn.Text = "AIM OFF"
aimBtn.BackgroundColor3 = Color3.fromRGB(35,35,35)
aimBtn.TextColor3 = Color3.new(1,1,1)

espBtn.MouseButton1Click:Connect(function()
    ESP_ON = not ESP_ON
    espBtn.Text = ESP_ON and "ESP ON" or "ESP OFF"
    if ESP_ON then enableESP() else clearESP() end
end)

aimBtn.MouseButton1Click:Connect(function()
    AIM_ON = not AIM_ON
    aimBtn.Text = AIM_ON and "AIM ON" or "AIM OFF"
end)

UIS.InputBegan:
