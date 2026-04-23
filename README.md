--!native
--!optimize 2

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

--// CONFIG
local Hub = {
    Toggles = { 
        Aimbot = false, SilentAim = false, 
        ESP = false, Tracers = false, 
        Names = true, Chams = true,
        TeamCheck = true, ShowFOV = true,
        SpeedHack = false, Hitboxes = false
    },
    Values = { Smoothness = 0.1, FOV = 120, Speed = 50, HitboxSize = 5 },
    Colors = { 
        Accent = Color3.fromRGB(57, 255, 20), -- Neon Green
        MainBG = Color3.fromRGB(15, 15, 18),
        SectionBG = Color3.fromRGB(25, 25, 30),
        TextCol = Color3.fromRGB(240, 240, 240)
    },
    Visible = true
}

--// CLEANUP OLD UI
if CoreGui:FindFirstChild("Ka2_Final_v22") then CoreGui:FindFirstChild("Ka2_Final_v22"):Destroy() end

--// UI SETUP
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "Ka2_Final_v22"; ScreenGui.Parent = CoreGui; ScreenGui.IgnoreGuiInset = true

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 700, 0, 520) 
Main.Position = UDim2.new(0.5, -350, 0.5, -260)
Main.BackgroundColor3 = Hub.Colors.MainBG
Main.BorderSizePixel = 0; Main.Visible = true; Main.Parent = ScreenGui
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 20)

-- NEON GREEN EDGELINE
local Border = Instance.new("Frame")
Border.Size = UDim2.new(1, 4, 1, 4); Border.Position = UDim2.new(0, -2, 0, -2)
Border.BackgroundColor3 = Hub.Colors.Accent; Border.ZIndex = 0; Border.Parent = Main
Instance.new("UICorner", Border).CornerRadius = UDim.new(0, 22)

-- SIMPLE DRAGGING
local dragToggle, dragStart, startPos
Main.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragToggle = true; dragStart = input.Position; startPos = Main.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragToggle then
        local delta = input.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragToggle = false end
end)

local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 160, 1, 0); Sidebar.BackgroundColor3 = Color3.fromRGB(20, 20, 24); Sidebar.Parent = Main
Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 20)

local Container = Instance.new("ScrollingFrame")
Container.Size = UDim2.new(1, -190, 1, -20); Container.Position = UDim2.new(0, 175, 0, 10)
Container.BackgroundTransparency = 1; Container.ScrollBarThickness = 0; Container.Parent = Main
local Layout = Instance.new("UIListLayout", Container); Layout.Padding = UDim.new(0, 12)

--// UI HELPERS
local function AddToggle(p, n, k)
    local B = Instance.new("TextButton")
    B.Size = UDim2.new(1, -10, 0, 55); B.BackgroundColor3 = Hub.Colors.SectionBG
    B.Text = "  " .. n .. ": " .. (Hub.Toggles[k] and "ON" or "OFF")
    B.TextColor3 = Hub.Toggles[k] and Hub.Colors.Accent or Hub.Colors.TextCol
    B.Font = Enum.Font.ArialBold; B.TextSize = 18; B.TextXAlignment = Enum.TextXAlignment.Left; B.Parent = p
    Instance.new("UICorner", B).CornerRadius = UDim.new(0, 12)
    B.MouseButton1Click:Connect(function()
        Hub.Toggles[k] = not Hub.Toggles[k]
        B.Text = "  " .. n .. ": " .. (Hub.Toggles[k] and "ON" or "OFF")
        B.TextColor3 = Hub.Toggles[k] and Hub.Colors.Accent or Hub.Colors.TextCol
    end)
end

local function AddSlider(p, n, k, min, max)
    local SFrame = Instance.new("Frame")
    SFrame.Size = UDim2.new(1, -10, 0, 80); SFrame.BackgroundTransparency = 1; SFrame.Parent = p
    local L = Instance.new("TextLabel")
    L.Size = UDim2.new(1, 0, 0, 35); L.Text = n .. " [" .. math.floor(Hub.Values[k]) .. "]"; L.TextColor3 = Color3.new(1,1,1); L.Font = Enum.Font.ArialBold; L.TextSize = 17; L.BackgroundTransparency = 1; L.Parent = SFrame
    local R = Instance.new("TextButton")
    R.Size = UDim2.new(1, -10, 0, 14); R.Position = UDim2.new(0, 5, 0, 50); R.BackgroundColor3 = Color3.fromRGB(40,40,45); R.Text = ""; R.Parent = SFrame
    Instance.new("UICorner", R)
    local F = Instance.new("Frame")
    F.Size = UDim2.new((Hub.Values[k]-min)/(max-min), 0, 1, 0); F.BackgroundColor3 = Hub.Colors.Accent; F.Parent = R
    Instance.new("UICorner", F)
    
    local dragging = false
    R.MouseButton1Down:Connect(function() dragging = true end)
    UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
    RunService.RenderStepped:Connect(function()
        if dragging then
            local rel = math.clamp((UserInputService:GetMouseLocation().X - R.AbsolutePosition.X) / R.AbsoluteSize.X, 0, 1)
            Hub.Values[k] = min + (max-min)*rel
            L.Text = n .. " [" .. math.floor(Hub.Values[k]) .. "]"
            F.Size = UDim2.new(rel, 0, 1, 0)
        end
    end)
end

local function LoadTab(name)
    for _, c in pairs(Container:GetChildren()) do if not c:IsA("UIListLayout") then c:Destroy() end end
    if name == "COMBAT" then
        AddToggle(Container, "Aimbot", "Aimbot")
        AddToggle(Container, "Silent Aim", "SilentAim")
        AddToggle(Container, "Head Expander", "Hitboxes")
        AddSlider(Container, "Smoothness", "Smoothness", 0, 1)
        AddSlider(Container, "FOV Radius", "FOV", 10, 800)
    elseif name == "VISUALS" then
        AddToggle(Container, "Box ESP", "ESP")
        AddToggle(Container, "Neon Chams", "Chams")
        AddToggle(Container, "Show Circle", "ShowFOV")
    elseif name == "MOVEMENT" then
        AddToggle(Container, "Speed Hack", "SpeedHack")
        AddSlider(Container, "Walk Speed", "Speed", 16, 300)
    end
end

local function NewT(n, pos)
    local B = Instance.new("TextButton")
    B.Size = UDim2.new(1, -20, 0, 55); B.Position = UDim2.new(0, 10, 0, 40 + (pos*65))
    B.BackgroundColor3 = Color3.fromRGB(25, 25, 30); B.Text = n; B.TextColor3 = Hub.Colors.TextCol
    B.Font = Enum.Font.ArialBold; B.TextSize = 16; B.Parent = Sidebar
    Instance.new("UICorner", B)
    B.MouseButton1Click:Connect(function() LoadTab(n) end)
end

NewT("COMBAT", 0); NewT("VISUALS", 1); NewT("MOVEMENT", 2)
LoadTab("COMBAT")

--// THE ENGINES
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 1.5; FOVCircle.Color = Hub.Colors.Accent

local ESP_Cache = { Boxes = {}, Highlights = {} }

local function CreateVisuals(P)
    if P == LocalPlayer then return end
    local B = Drawing.new("Square"); B.Thickness = 1.5; B.Color = Hub.Colors.Accent; ESP_Cache.Boxes[P] = B
    local C = Instance.new("Highlight")
    C.FillColor = Hub.Colors.Accent; C.FillTransparency = 0.4; C.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop; ESP_Cache.Highlights[P] = C
end

-- Cleanup function for when players leave
local function RemoveVisuals(P)
    if ESP_Cache.Boxes[P] then ESP_Cache.Boxes[P]:Remove(); ESP_Cache.Boxes[P] = nil end
    if ESP_Cache.Highlights[P] then ESP_Cache.Highlights[P]:Destroy(); ESP_Cache.Highlights[P] = nil end
end

for _, p in pairs(Players:GetPlayers()) do CreateVisuals(p) end
Players.PlayerAdded:Connect(CreateVisuals)
Players.PlayerRemoving:Connect(RemoveVisuals)

--// INSTANT K-TOGGLE
UserInputService.InputBegan:Connect(function(input, g)
    if not g and input.KeyCode == Enum.KeyCode.K then
        Hub.Visible = not Hub.Visible
        Main.Visible = Hub.Visible
    end
end)

--// SILENT AIM & AIMBOT ENGINE
local SilentTarget = nil

RunService.RenderStepped:Connect(function()
    FOVCircle.Visible = Hub.Toggles.ShowFOV
    FOVCircle.Radius = Hub.Values.FOV
    FOVCircle.Position = UserInputService:GetMouseLocation()
    
    if Hub.Toggles.SpeedHack and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = Hub.Values.Speed
    end

    local Target = nil
    local closestDist = Hub.Values.FOV
    
    for p, box in pairs(ESP_Cache.Boxes) do
        if p and p.Parent and p.Character and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0 then
            local Root = p.Character:FindFirstChild("HumanoidRootPart")
            if Root then
                local Pos, On = Camera:WorldToViewportPoint(Root.Position)
                local Enemy = not (p.Team == LocalPlayer.Team and Hub.Toggles.TeamCheck)
                
                -- Update Aim Target
                if On and Enemy then
                    local mag = (Vector2.new(Pos.X, Pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                    if mag < closestDist then 
                        closestDist = mag
                        Target = p.Character
                    end
                end

                -- Update ESP Box
                if Hub.Toggles.ESP and On and Enemy then
                    local dist = (Camera.CFrame.Position - Root.Position).Magnitude
                    local s = (1000/dist)*3
                    box.Visible = true
                    box.Size = Vector2.new(s, s*1.5)
                    box.Position = Vector2.new(Pos.X-s/2, Pos.Y-s/2)
                else
                    box.Visible = false
                end

                -- Update Chams
                if ESP_Cache.Highlights[p] then
                    ESP_Cache.Highlights[p].Parent = p.Character
                    ESP_Cache.Highlights[p].Enabled = Hub.Toggles.Chams and Enemy
                end
            else
                box.Visible = false
            end
        else
            -- Hide if dead or invalid
            box.Visible = false
            if ESP_Cache.Highlights[p] then ESP_Cache.Highlights[p].Enabled = false end
        end
    end
    
    SilentTarget = Target
    if Hub.Toggles.Aimbot and Target and Target:FindFirstChild("Head") and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, Target.Head.Position), Hub.Values.Smoothness)
    end
end)

--// SILENT AIM FIX
local mt = getrawmetatable(game)
local oldIndex = mt.__index
setreadonly(mt, false)

mt.__index = newcclosure(function(t, k)
    if t == Mouse and (k == "Hit" or k == "Target") then
        if Hub.Toggles.SilentAim and SilentTarget and SilentTarget:FindFirstChild("Head") then
            return (k == "Hit" and SilentTarget.Head.CFrame or SilentTarget.Head)
        end
    end
    return oldIndex(t, k)
end)

setreadonly(mt, true)
