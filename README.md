local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- // CONFIGURAÇÕES
local Settings = {
    Enabled = false,
    WallCheck = true,
    Smoothing = 0.2,
    FOV = 60,
    AimPart = "Head",
    ESP_Enabled = false,
    ShowHealth = false
}

local FOV_LIST = {25, 45, 60, 100, 180}
local SMOOTH_LIST = {0.1, 0.2, 0.3, 1.0}
local fovIdx, smoothIdx = 3, 2
local ESP_Objects = {}

-- // UI PRINCIPAL
local ScreenGui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
ScreenGui.Name = "Elite_Dual_Final"
ScreenGui.IgnoreGuiInset = true
ScreenGui.ResetOnSpawn = false

-- BOTÃO MASTER
local OpenBtn = Instance.new("TextButton", ScreenGui)
OpenBtn.Size = UDim2.new(0, 45, 0, 45)
OpenBtn.Position = UDim2.new(0.05, 0, 0.2, 0)
OpenBtn.Text = "⚙️"
OpenBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
OpenBtn.TextColor3 = Color3.new(1,1,1)
OpenBtn.TextSize = 22
Instance.new("UICorner", OpenBtn).CornerRadius = UDim.new(0.5, 0)
Instance.new("UIStroke", OpenBtn).Color = Color3.new(1,1,1)

-- MENU MASTER
local MasterFrame = Instance.new("Frame", ScreenGui)
MasterFrame.Size = UDim2.new(0, 360, 0, 280)
MasterFrame.Position = UDim2.new(0.3, 0, 0.3, 0)
MasterFrame.BackgroundTransparency = 1
MasterFrame.Visible = false

local function createPane(title, pos)
    local f = Instance.new("Frame", MasterFrame)
    f.Size = UDim2.new(0, 170, 1, 0)
    f.Position = pos
    f.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    Instance.new("UICorner", f)
    Instance.new("UIStroke", f).Color = Color3.fromRGB(60, 60, 60)
    local t = Instance.new("TextLabel", f)
    t.Size = UDim2.new(1, 0, 0, 35)
    t.Text = title
    t.TextColor3 = Color3.new(1,1,1)
    t.Font = Enum.Font.GothamBold
    t.TextSize = 14
    t.BackgroundTransparency = 1
    return f
end

local LeftMenu = createPane("AIMBOT", UDim2.new(0, 0, 0, 0))
local RightMenu = createPane("ESP VISUAL", UDim2.new(0, 180, 0, 0))

-- CÍRCULO FOV
local FOVCircle = Instance.new("Frame", ScreenGui)
FOVCircle.AnchorPoint = Vector2.new(0.5, 0.5)
FOVCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
FOVCircle.Size = UDim2.new(0, Settings.FOV * 2, 0, Settings.FOV * 2)
FOVCircle.BackgroundTransparency = 1
local CircleStroke = Instance.new("UIStroke", FOVCircle)
CircleStroke.Color = Color3.fromRGB(255, 0, 0)
Instance.new("UICorner", FOVCircle).CornerRadius = UDim.new(1, 0)

-- BOTÕES
local function createBtn(parent, txt, pos, color)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(0.9, 0, 0, 35)
    b.Position = pos
    b.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    b.Text = txt
    b.Font = Enum.Font.SourceSansBold
    b.TextSize = 13
    b.TextColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", b)
    local s = Instance.new("UIStroke", b)
    s.Color = color or Color3.fromRGB(60, 60, 60)
    return b, s
end

local AimT, AimS = createBtn(LeftMenu, "AIM: OFF", UDim2.new(0.05, 0, 0.15, 0), Color3.new(1,0,0))
local PartT, _ = createBtn(LeftMenu, "ALVO: CABEÇA", UDim2.new(0.05, 0, 0.32, 0))
local WallT, WallS = createBtn(LeftMenu, "WALLCHECK: ON", UDim2.new(0.05, 0, 0.49, 0), Color3.new(0,1,0))
local FovT, _ = createBtn(LeftMenu, "FOV: 60", UDim2.new(0.05, 0, 0.66, 0))
local VelT, _ = createBtn(LeftMenu, "VEL: 0.2", UDim2.new(0.05, 0, 0.83, 0))

local EspBoxT, EspBoxS = createBtn(RightMenu, "ESP BOX: OFF", UDim2.new(0.05, 0, 0.15, 0), Color3.new(1,0,0))
local HealthT, HealthS = createBtn(RightMenu, "VIDA: OFF", UDim2.new(0.05, 0, 0.32, 0), Color3.new(1,0,0))

-- // WALLCHECK MELHORADO
local function checkVisible(part)
    local rayParams = RaycastParams.new()
    rayParams.FilterDescendantsInstances = {LocalPlayer.Character, Camera}
    rayParams.FilterType = Enum.RaycastFilterType.Exclude
    
    local rayResult = workspace:Raycast(Camera.CFrame.Position, (part.Position - Camera.CFrame.Position), rayParams)
    
    if rayResult then
        return rayResult.Instance:IsDescendantOf(part.Parent)
    end
    return true
end

-- // ATUALIZAÇÃO DA ESP
local function UpdateESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        
        local char = player.Character
        local obj = ESP_Objects[player]
        
        if Settings.ESP_Enabled and char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0 then
            local root = char.HumanoidRootPart
            local head = char:FindFirstChild("Head")
            if not head then continue end

            -- Pontos para calcular tamanho exato da caixa
            local headPos, onScreen = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
            local footPos = Camera:WorldToViewportPoint(root.Position - Vector3.new(0, 3, 0))

            if onScreen then
                if not obj then
                    obj = {}
                    obj.Box = Instance.new("Frame", ScreenGui)
                    obj.Box.BackgroundTransparency = 0.7
                    obj.Box.BorderSizePixel = 0
                    obj.Stroke = Instance.new("UIStroke", obj.Box)
                    obj.Stroke.Thickness = 1.5
                    
                    obj.HealthBar = Instance.new("Frame", obj.Box)
                    obj.HealthBar.BorderSizePixel = 0
                    obj.HealthBar.BackgroundColor3 = Color3.new(0, 1, 0)
                    
                    ESP_Objects[player] = obj
                end

                local height = math.abs(headPos.Y - footPos.Y)
                local width = height * 0.6
                
                obj.Box.Visible = true
                obj.Box.Size = UDim2.new(0, width, 0, height)
                obj.Box.Position = UDim2.new(0, headPos.X - width/2, 0, headPos.Y)

                -- Wallcheck Visual
                local isVisible = checkVisible(head)
                local color = isVisible and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
                obj.Box.BackgroundColor3 = color
                obj.Stroke.Color = color

                -- Health Bar Slim
                obj.HealthBar.Visible = Settings.ShowHealth
                local healthScale = char.Humanoid.Health / char.Humanoid.MaxHealth
                obj.HealthBar.Size = UDim2.new(0, 2, healthScale, 0)
                obj.HealthBar.Position = UDim2.new(0, -5, 1 - healthScale, 0)
            else
                if obj then obj.Box.Visible = false end
            end
        else
            if obj then obj.Box.Visible = false end
        end
    end
end

-- // LÓGICA AIMBOT
local function getTarget()
    local target, dist = nil, Settings.FOV
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    for _, v in ipairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("Humanoid") and v.Character.Humanoid.Health > 0 then
            local p = (Settings.AimPart == "Head" and v.Character:FindFirstChild("Head")) or v.Character:FindFirstChild("UpperTorso") or v.Character:FindFirstChild("Torso")
            if p then
                local pos, onS = Camera:WorldToViewportPoint(p.Position)
                if onS then
                    local mag = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                    if mag < dist and (not Settings.WallCheck or checkVisible(p)) then
                        dist = mag; target = p
                    end
                end
            end
        end
    end
    return target
end

-- // DRAG E EVENTOS
local function makeDrag(obj)
    local dragging, start, startPos
    obj.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then dragging = true start = i.Position startPos = obj.Position end end)
    UserInputService.InputChanged:Connect(function(i) if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then local d = i.Position - start obj.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y) end end)
    UserInputService.InputEnded:Connect(function() dragging = false end)
end
makeDrag(MasterFrame); makeDrag(OpenBtn)

OpenBtn.Activated:Connect(function() MasterFrame.Visible = not MasterFrame.Visible end)

AimT.Activated:Connect(function()
    Settings.Enabled = not Settings.Enabled
    AimT.Text = Settings.Enabled and "AIM: ON" or "AIM: OFF"
    AimS.Color = Settings.Enabled and Color3.new(0,1,0) or Color3.new(1,0,0)
    CircleStroke.Color = AimS.Color
end)

EspBoxT.Activated:Connect(function()
    Settings.ESP_Enabled = not Settings.ESP_Enabled
    EspBoxT.Text = "ESP BOX: " .. (Settings.ESP_Enabled and "ON" or "OFF")
    EspBoxS.Color = Settings.ESP_Enabled and Color3.new(0,1,0) or Color3.new(1,0,0)
end)

HealthT.Activated:Connect(function()
    Settings.ShowHealth = not Settings.ShowHealth
    HealthT.Text = "VIDA: " .. (Settings.ShowHealth and "ON" or "OFF")
    HealthS.Color = Settings.ShowHealth and Color3.new(0,1,0) or Color3.new(1,0,0)
end)

WallT.Activated:Connect(function()
    Settings.WallCheck = not Settings.WallCheck
    WallT.Text = "WALLCHECK: " .. (Settings.WallCheck and "ON" or "OFF")
    WallS.Color = Settings.WallCheck and Color3.new(0,1,0) or Color3.new(1,0,0)
end)

VelT.Activated:Connect(function()
    smoothIdx = smoothIdx + 1 if smoothIdx > #SMOOTH_LIST then smoothIdx = 1 end
    Settings.Smoothing = SMOOTH_LIST[smoothIdx]
    VelT.Text = "VEL: " .. (Settings.Smoothing == 1 and "INSTANT" or Settings.Smoothing)
end)

PartT.Activated:Connect(function()
    local pNames = {Head="CABEÇA", Torso="PEITO", Stomach="BARRIGA"}
    local parts = {"Head", "Torso", "Stomach"}
    local idx = table.find(parts, Settings.AimPart) + 1
    if idx > 3 then idx = 1 end
    Settings.AimPart = parts[idx]
    PartT.Text = "ALVO: " .. pNames[Settings.AimPart]
end)

FovT.Activated:Connect(function()
    fovIdx = fovIdx + 1 if fovIdx > #FOV_LIST then fovIdx = 1 end
    Settings.FOV = FOV_LIST[fovIdx]
    FovT.Text = "FOV: " .. Settings.FOV
    FOVCircle.Size = UDim2.new(0, Settings.FOV * 2, 0, Settings.FOV * 2)
end)

-- // LOOP
RunService.RenderStepped:Connect(function()
    UpdateESP()
    FOVCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
    if Settings.Enabled then
        local t = getTarget()
        if t then
            local cf = CFrame.new(Camera.CFrame.Position, t.Position)
            if Settings.Smoothing >= 1 then Camera.CFrame = cf else Camera.CFrame = Camera.CFrame:Lerp(cf, Settings.Smoothing) end
        end
    end
end)
