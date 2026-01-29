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
    ESP_Master = false,
    ESP_Box = false,
    ESP_Health = false,
    ESP_Name = false
}

local FOV_LIST = {25, 45, 60, 100, 180}
local SMOOTH_LIST = {0.1, 0.2, 0.3, 1.0}
local fovIdx, smoothIdx = 3, 2
local ESP_Objects = {}

-- // UI (MENU ÚNICO E ARRASTÁVEL)
local ScreenGui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
ScreenGui.Name = "Kiwi_Defusal_V7"
ScreenGui.IgnoreGuiInset = true
ScreenGui.ResetOnSpawn = false

local OpenBtn = Instance.new("TextButton", ScreenGui)
OpenBtn.Size = UDim2.new(0, 45, 0, 45)
OpenBtn.Position = UDim2.new(0.05, 0, 0.2, 0)
OpenBtn.Text = "⚙️"
OpenBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
OpenBtn.TextColor3 = Color3.new(1,1,1)
OpenBtn.TextSize = 22
Instance.new("UICorner", OpenBtn).CornerRadius = UDim.new(0.5, 0)
Instance.new("UIStroke", OpenBtn).Color = Color3.new(1,1,1)

local MasterFrame = Instance.new("Frame", ScreenGui)
MasterFrame.Size = UDim2.new(0, 340, 0, 300)
MasterFrame.Position = UDim2.new(0.3, 0, 0.3, 0)
MasterFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
MasterFrame.Visible = false
Instance.new("UICorner", MasterFrame).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", MasterFrame).Color = Color3.fromRGB(50, 50, 50)

local TitleBar = Instance.new("Frame", MasterFrame)
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
TitleBar.BorderSizePixel = 0
Instance.new("UICorner", TitleBar).CornerRadius = UDim.new(0, 8)
local TitleTxt = Instance.new("TextLabel", TitleBar)
TitleTxt.Size = UDim2.new(1, 0, 1, 0)
TitleTxt.BackgroundTransparency = 1
TitleTxt.Text = "KIWI TESTES"
TitleTxt.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleTxt.Font = Enum.Font.GothamBlack
TitleTxt.TextSize = 16

-- // FUNÇÃO DE CRIAR BOTÕES
local function createBtn(txt, pos, color)
    local b = Instance.new("TextButton", MasterFrame)
    b.Size = UDim2.new(0.42, 0, 0, 32)
    b.Position = pos
    b.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    b.Text = txt
    b.Font = Enum.Font.SourceSansBold
    b.TextSize = 12
    b.TextColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 4)
    local s = Instance.new("UIStroke", b)
    s.Color = color or Color3.fromRGB(60, 60, 60)
    return b, s
end

-- Colunas (Removido o botão de Smoke)
local AimT, AimS = createBtn("AIM: OFF", UDim2.new(0.05, 0, 0.15, 0), Color3.new(1,0,0))
local PartT, _ = createBtn("ALVO: CABEÇA", UDim2.new(0.05, 0, 0.30, 0))
local WallT, WallS = createBtn("WALLCHECK: ON", UDim2.new(0.05, 0, 0.45, 0), Color3.new(0,1,0))
local FovT, _ = createBtn("FOV: 60", UDim2.new(0.05, 0, 0.60, 0))
local VelT, _ = createBtn("VEL: 0.2", UDim2.new(0.05, 0, 0.75, 0))

local EspMT, EspMS = createBtn("ESP MASTER: OFF", UDim2.new(0.53, 0, 0.15, 0), Color3.new(1,0,0))
local BoxT, BoxS = createBtn("BOX: OFF", UDim2.new(0.53, 0, 0.30, 0), Color3.new(1,0,0))
local NameT, NameS = createBtn("NOMES: OFF", UDim2.new(0.53, 0, 0.45, 0), Color3.new(1,0,0))
local HealthT, HealthS = createBtn("VIDA: OFF", UDim2.new(0.53, 0, 0.60, 0), Color3.new(1,0,0))

-- // FOV CIRCLE
local FOVCircle = Instance.new("Frame", ScreenGui)
FOVCircle.AnchorPoint = Vector2.new(0.5, 0.5)
FOVCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
FOVCircle.Size = UDim2.new(0, Settings.FOV * 2, 0, Settings.FOV * 2)
FOVCircle.BackgroundTransparency = 1
local CircleStroke = Instance.new("UIStroke", FOVCircle)
CircleStroke.Color = Color3.fromRGB(255, 0, 0)
Instance.new("UICorner", FOVCircle).CornerRadius = UDim.new(1, 0)

-- // ARRASTAR
local function makeDrag(obj, target)
    local drag, start, startPos
    obj.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then drag = true start = i.Position startPos = target.Position end end)
    UserInputService.InputChanged:Connect(function(i) if drag and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then local d = i.Position - start target.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y) end end)
    UserInputService.InputEnded:Connect(function() drag = false end)
end
makeDrag(TitleBar, MasterFrame); makeDrag(MasterFrame, MasterFrame); makeDrag(OpenBtn, OpenBtn)

-- // EVENTOS
OpenBtn.Activated:Connect(function() MasterFrame.Visible = not MasterFrame.Visible end)
AimT.Activated:Connect(function() Settings.Enabled = not Settings.Enabled; AimT.Text = "AIM: "..(Settings.Enabled and "ON" or "OFF"); AimS.Color = Settings.Enabled and Color3.new(0,1,0) or Color3.new(1,0,0); CircleStroke.Color = AimS.Color end)
WallT.Activated:Connect(function() Settings.WallCheck = not Settings.WallCheck; WallT.Text = "WALL: "..(Settings.WallCheck and "ON" or "OFF"); WallS.Color = Settings.WallCheck and Color3.new(0,1,0) or Color3.new(1,0,0) end)
EspMT.Activated:Connect(function() Settings.ESP_Master = not Settings.ESP_Master; EspMT.Text = "ESP: "..(Settings.ESP_Master and "ON" or "OFF"); EspMS.Color = Settings.ESP_Master and Color3.new(0,1,0) or Color3.new(1,0,0) end)
BoxT.Activated:Connect(function() Settings.ESP_Box = not Settings.ESP_Box; BoxT.Text = "BOX: "..(Settings.ESP_Box and "ON" or "OFF"); BoxS.Color = Settings.ESP_Box and Color3.new(0,1,0) or Color3.new(1,0,0) end)
NameT.Activated:Connect(function() Settings.ESP_Name = not Settings.ESP_Name; NameT.Text = "NOMES: "..(Settings.ESP_Name and "ON" or "OFF"); NameS.Color = Settings.ESP_Name and Color3.new(0,1,0) or Color3.new(1,0,0) end)
HealthT.Activated:Connect(function() Settings.ESP_Health = not Settings.ESP_Health; HealthT.Text = "VIDA: "..(Settings.ESP_Health and "ON" or "OFF"); HealthS.Color = Settings.ESP_Health and Color3.new(0,1,0) or Color3.new(1,0,0) end)
FovT.Activated:Connect(function() fovIdx = fovIdx + 1; if fovIdx > #FOV_LIST then fovIdx = 1 end; Settings.FOV = FOV_LIST[fovIdx]; FovT.Text = "FOV: "..Settings.FOV; FOVCircle.Size = UDim2.new(0, Settings.FOV * 2, 0, Settings.FOV * 2) end)
VelT.Activated:Connect(function() smoothIdx = smoothIdx + 1; if smoothIdx > #SMOOTH_LIST then smoothIdx = 1 end; Settings.Smoothing = SMOOTH_LIST[smoothIdx]; VelT.Text = "VEL: "..(Settings.Smoothing == 1 and "INSTANT" or Settings.Smoothing) end)
PartT.Activated:Connect(function() local parts = {"Head", "Torso", "Stomach"}; local pNames = {Head="CABEÇA", Torso="PEITO", Stomach="BARRIGA"}; local idx = table.find(parts, Settings.AimPart) + 1; if idx > 3 then idx = 1 end; Settings.AimPart = parts[idx]; PartT.Text = "ALVO: "..pNames[Settings.AimPart] end)

-- // LOOP ESP E AIMBOT
local function checkVisible(part)
    local rp = RaycastParams.new(); rp.FilterDescendantsInstances = {LocalPlayer.Character, Camera}; rp.FilterType = Enum.RaycastFilterType.Exclude
    local res = workspace:Raycast(Camera.CFrame.Position, (part.Position - Camera.CFrame.Position), rp)
    return not res or res.Instance:IsDescendantOf(part.Parent)
end

RunService.RenderStepped:Connect(function()
    FOVCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
    -- ESP Update
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        local char = plr.Character
        local obj = ESP_Objects[plr]
        if Settings.ESP_Master and char and char:FindFirstChild("HumanoidRootPart") and char.Humanoid.Health > 0 then
            local root = char.HumanoidRootPart; local head = char:FindFirstChild("Head")
            if head then
                local hPos, onS = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
                local fPos = Camera:WorldToViewportPoint(root.Position - Vector3.new(0, 3, 0))
                if onS then
                    if not obj then
                        obj = {Main = Instance.new("Frame", ScreenGui)}
                        obj.Main.BackgroundTransparency = 1; obj.Box = Instance.new("Frame", obj.Main); obj.Box.BackgroundTransparency = 0.7; obj.Box.BorderSizePixel = 0
                        obj.Stroke = Instance.new("UIStroke", obj.Box); obj.HealthBar = Instance.new("Frame", obj.Main); obj.HealthBar.BorderSizePixel = 0
                        obj.NameTag = Instance.new("TextLabel", obj.Main); obj.NameTag.BackgroundTransparency = 1; obj.NameTag.Font = Enum.Font.GothamBold; obj.NameTag.TextSize = 11; Instance.new("UIStroke", obj.NameTag).Thickness = 1
                        ESP_Objects[plr] = obj
                    end
                    local h = math.abs(hPos.Y - fPos.Y); local w = h * 0.6
                    local color = checkVisible(head) and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
                    obj.Main.Visible = true; obj.Box.Visible = Settings.ESP_Box; obj.Box.Size = UDim2.new(0, w, 0, h); obj.Box.Position = UDim2.new(0, hPos.X-w/2, 0, hPos.Y); obj.Box.BackgroundColor3 = color; obj.Stroke.Color = color
                    obj.HealthBar.Visible = Settings.ESP_Health; local hs = char.Humanoid.Health/char.Humanoid.MaxHealth; obj.HealthBar.Size = UDim2.new(0, 2, hs*h, 0); obj.HealthBar.Position = UDim2.new(0, hPos.X-w/2-5, 0, hPos.Y+(h*(1-hs))); obj.HealthBar.BackgroundColor3 = Color3.new(0, 1, 0)
                    obj.NameTag.Visible = Settings.ESP_Name; local d = math.floor((Camera.CFrame.Position-root.Position).Magnitude/3); obj.NameTag.Text = string.format("%s [%dm]", plr.Name, d); obj.NameTag.Size = UDim2.new(0, 100, 0, 15); obj.NameTag.Position = UDim2.new(0, hPos.X-50, 0, hPos.Y-18); obj.NameTag.TextColor3 = color
                else if obj then obj.Main.Visible = false end end
            end
        else if obj then obj.Main.Visible = false end end
    end
    -- AIMBOT Update
    if Settings.Enabled then
        local t = nil; local d = Settings.FOV; local c = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
        for _, v in ipairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("Humanoid") and v.Character.Humanoid.Health > 0 then
                local p = (Settings.AimPart == "Head" and v.Character:FindFirstChild("Head")) or v.Character:FindFirstChild("UpperTorso") or v.Character:FindFirstChild("Torso")
                if p then
                    local pos, onS = Camera:WorldToViewportPoint(p.Position)
                    if onS then
                        local mag = (Vector2.new(pos.X, pos.Y) - c).Magnitude
                        if mag < d and (not Settings.WallCheck or checkVisible(p)) then d = mag; t = p end
                    end
                end
            end
        end
        if t then
            local cf = CFrame.new(Camera.CFrame.Position, t.Position)
            if Settings.Smoothing >= 1 then Camera.CFrame = cf else Camera.CFrame = Camera.CFrame:Lerp(cf, Settings.Smoothing) end
        end
    end
end)
