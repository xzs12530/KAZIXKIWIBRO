local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- // CONFIGURAÇÕES
local Settings = {
    Enabled = false,
    WallCheck = true,
    Smoothing = 0.2,
    FOV = 60,
    AimPart = "Head",
    -- ESP
    ESP_Master = false,
    ESP_Box = false,
    ESP_BoxTransparency = 0.6,
    ESP_Health = false,
    ESP_Name = false,
    ESP_HeadDot = false,
    -- MISC
    RainbowMode = false
}

local ESP_Objects = {}

-- // TEMA
local Theme = {
    MainBG = Color3.fromRGB(12, 12, 14),
    SidebarBG = Color3.fromRGB(15, 15, 18),
    CardBG = Color3.fromRGB(22, 22, 26),
    Accent = Color3.fromRGB(255, 30, 30),
    Separator = Color3.fromRGB(35, 35, 40),
    DropdownBG = Color3.fromRGB(20, 20, 22),
    TextWhite = Color3.fromRGB(255, 255, 255),
    TextGray = Color3.fromRGB(160, 160, 165)
}

-- // UI PRINCIPAL
local ScreenGui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
ScreenGui.Name = "Worldwide_Supreme_V12_MobileFixed"
ScreenGui.IgnoreGuiInset = true
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 540, 0, 420)
MainFrame.Position = UDim2.new(0.5, -270, 0.5, -210)
MainFrame.BackgroundColor3 = Theme.MainBG
MainFrame.Visible = false
MainFrame.Active = true -- Essencial para Mobile
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)
Instance.new("UIStroke", MainFrame).Color = Theme.Separator

local UIScale = Instance.new("UIScale", MainFrame)
UIScale.Scale = 0

-- // BARRA DE NOTIFICAÇÃO SUPERIOR
local NotifBar = Instance.new("Frame", MainFrame)
NotifBar.Size = UDim2.new(1, 0, 0, 22); NotifBar.BackgroundColor3 = Theme.Accent; NotifBar.BorderSizePixel = 0
local NotifText = Instance.new("TextLabel", NotifBar)
NotifText.Size = UDim2.new(1, 0, 1, 0); NotifText.BackgroundTransparency = 1; NotifText.Text = "Worldwide Supreme System V12 | Mobile Drag Fixed."; NotifText.TextColor3 = Color3.new(1,1,1); NotifText.Font = Enum.Font.GothamMedium; NotifText.TextSize = 10
Instance.new("UICorner", NotifBar).CornerRadius = UDim.new(0, 10)

-- // HEADER
local Header = Instance.new("Frame", MainFrame)
Header.Size = UDim2.new(1, 0, 0, 60); Header.Position = UDim2.new(0,0,0,25); Header.BackgroundTransparency = 1

local TitleLabel = Instance.new("TextLabel", Header)
TitleLabel.Size = UDim2.new(0, 300, 1, 0); TitleLabel.Position = UDim2.new(1, -310, 0, 0); TitleLabel.BackgroundTransparency = 1; TitleLabel.Text = "WORLDWIDE DEFUSAL"; TitleLabel.TextColor3 = Color3.new(1,1,1); TitleLabel.Font = Enum.Font.GothamBlack; TitleLabel.TextSize = 20; TitleLabel.TextXAlignment = Enum.TextXAlignment.Right
local TitleGradient = Instance.new("UIGradient", TitleLabel)
TitleGradient.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.new(1,1,1)), ColorSequenceKeypoint.new(0.2, Color3.new(1,1,1)), ColorSequenceKeypoint.new(0.5, Theme.Accent), ColorSequenceKeypoint.new(0.8, Color3.new(1,1,1)), ColorSequenceKeypoint.new(1, Color3.new(1,1,1))})

-- // SIDEBAR
local Sidebar = Instance.new("Frame", MainFrame)
Sidebar.Size = UDim2.new(0, 65, 1, -110); Sidebar.Position = UDim2.new(0, 10, 0, 95); Sidebar.BackgroundColor3 = Theme.SidebarBG; Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 10)

-- // CONTENT AREA
local ContentArea = Instance.new("Frame", MainFrame)
ContentArea.Size = UDim2.new(1, -100, 1, -110); ContentArea.Position = UDim2.new(0, 85, 0, 95); ContentArea.BackgroundTransparency = 1

local AimPage = Instance.new("Frame", ContentArea); local EspPage = Instance.new("Frame", ContentArea); local MiscPage = Instance.new("Frame", ContentArea)
for _, p in pairs({AimPage, EspPage, MiscPage}) do p.Size = UDim2.new(1,0,1,0); p.BackgroundTransparency = 1; p.Visible = false end
AimPage.Visible = true

-- // NOVA FUNÇÃO DE ARRASTE (ESTÁVEL PARA MOBILE)
local function makeDrag(gui)
    local dragging
    local dragInput
    local dragStart
    local startPos

    local function update(input)
        local delta = input.Position - dragStart
        -- Usamos apenas Offset para evitar saltos de escala
        gui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end

    gui.InputBegan:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
            dragging = true
            dragStart = input.Position
            startPos = gui.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    gui.InputChanged:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            update(input)
        end
    end)
end

-- // FUNÇÃO DE SUB-ABAS
local function createSubTabs(parentPage, sections)
    local subBar = Instance.new("Frame", parentPage)
    subBar.Size = UDim2.new(1, 0, 0, 30); subBar.BackgroundTransparency = 1
    local list = Instance.new("UIListLayout", subBar); list.FillDirection = Enum.FillDirection.Horizontal; list.Padding = UDim.new(0, 25)
    
    local container = Instance.new("Frame", parentPage)
    container.Size = UDim2.new(1, 0, 1, -40); container.Position = UDim2.new(0, 0, 0, 40); container.BackgroundTransparency = 1
    
    for name, contentFunc in pairs(sections) do
        local btn = Instance.new("TextButton", subBar)
        btn.Size = UDim2.new(0, 90, 1, 0); btn.BackgroundTransparency = 1; btn.Text = name:upper(); btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; btn.TextColor3 = Theme.TextGray; btn.Name = name
        
        local line = Instance.new("Frame", btn)
        line.Size = UDim2.new(1, 0, 0, 2); line.Position = UDim2.new(0, 0, 1, 0); line.BackgroundColor3 = Theme.Accent; line.Visible = false
        
        local subContent = Instance.new("ScrollingFrame", container)
        subContent.Size = UDim2.new(1, 0, 1, 0); subContent.BackgroundTransparency = 1; subContent.ScrollBarThickness = 0; subContent.Visible = false; subContent.Name = name.."Content"
        Instance.new("UIListLayout", subContent).Padding = UDim.new(0, 10)
        
        contentFunc(subContent)
        
        btn.Activated:Connect(function()
            for _, v in pairs(container:GetChildren()) do if v:IsA("ScrollingFrame") then v.Visible = false end end
            for _, v in pairs(subBar:GetChildren()) do if v:IsA("TextButton") then v.TextColor3 = Theme.TextGray; v:FindFirstChild("Frame").Visible = false end end
            subContent.Visible = true; btn.TextColor3 = Theme.TextWhite; line.Visible = true
        end)
    end
    -- Ativa a primeira aba
    local firstBtn = subBar:FindFirstChildWhichIsA("TextButton")
    if firstBtn then 
        firstBtn.TextColor3 = Theme.TextWhite; firstBtn:FindFirstChild("Frame").Visible = true
        container:FindFirstChildWhichIsA("ScrollingFrame").Visible = true
    end
end

-- // COMPONENTES DE UI
local function createCard(parent, title)
    local card = Instance.new("Frame", parent); card.Size = UDim2.new(1, -5, 0, 30); card.BackgroundColor3 = Theme.CardBG; card.AutomaticSize = Enum.AutomaticSize.Y; Instance.new("UICorner", card).CornerRadius = UDim.new(0, 6)
    local t = Instance.new("TextLabel", card); t.Size = UDim2.new(1, 0, 0, 25); t.Text = "  " .. title:upper(); t.Font = Enum.Font.GothamBold; t.TextSize = 9; t.TextColor3 = Theme.TextGray; t.BackgroundTransparency = 1; t.TextXAlignment = Enum.TextXAlignment.Left
    local container = Instance.new("Frame", card); container.Size = UDim2.new(1, 0, 0, 0); container.Position = UDim2.new(0, 0, 0, 25); container.BackgroundTransparency = 1; container.AutomaticSize = Enum.AutomaticSize.Y; Instance.new("UIListLayout", container).Padding = UDim.new(0, 5)
    return container
end

local function createToggle(parent, text, key)
    local b = Instance.new("TextButton", parent); b.Size = UDim2.new(1, -10, 0, 30); b.BackgroundTransparency = 1; b.Text = "  "..text; b.Font = Enum.Font.GothamMedium; b.TextColor3 = Theme.TextWhite; b.TextSize = 13; b.TextXAlignment = Enum.TextXAlignment.Left
    local s = Instance.new("Frame", b); s.Size = UDim2.new(0, 30, 0, 16); s.Position = UDim2.new(1, -35, 0.5, -8); s.BackgroundColor3 = Settings[key] and Theme.Accent or Color3.fromRGB(45,45,50); Instance.new("UICorner", s).CornerRadius = UDim.new(1,0)
    b.Activated:Connect(function() Settings[key] = not Settings[key]; TweenService:Create(s, TweenInfo.new(0.2), {BackgroundColor3 = Settings[key] and Theme.Accent or Color3.fromRGB(45,45,50)}):Play() end)
end

local function createSlider(parent, text, min, max, key, dec)
    local c = Instance.new("Frame", parent); c.Size = UDim2.new(1, -10, 0, 45); c.BackgroundTransparency = 1
    local l = Instance.new("TextLabel", c); l.Size = UDim2.new(1, 0, 0, 20); l.Text = text..": "..Settings[key]; l.TextColor3 = Theme.TextWhite; l.BackgroundTransparency = 1; l.Font = Enum.Font.GothamMedium; l.TextSize = 12; l.TextXAlignment = Enum.TextXAlignment.Left
    local bg = Instance.new("Frame", c); bg.Size = UDim2.new(1, -5, 0, 4); bg.Position = UDim2.new(0, 0, 0, 30); bg.BackgroundColor3 = Color3.fromRGB(50, 50, 55); Instance.new("UICorner", bg)
    local f = Instance.new("Frame", bg); f.Size = UDim2.new((Settings[key]-min)/(max-min), 0, 1, 0); f.BackgroundColor3 = Theme.Accent; Instance.new("UICorner", f)
    local function update(input)
        local pos = math.clamp((input.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1); local val = min + (max - min) * pos
        val = dec and math.floor(val * 10) / 10 or math.floor(val); Settings[key] = val; l.Text = text..": "..val; f.Size = UDim2.new(pos, 0, 1, 0)
    end
    bg.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then local conn; conn = UserInputService.InputChanged:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then update(i) end end) UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then conn:Disconnect() end end) update(input) end end)
end

local function createDropdown(parent, text, key)
    local options = {"Head", "Torso", "Stomach"}; local displayNames = {Head = "Head", Torso = "Chest", Stomach = "Belly"}
    local container = Instance.new("Frame", parent); container.Size = UDim2.new(1, -10, 0, 35); container.BackgroundTransparency = 1
    local b = Instance.new("TextButton", container); b.Size = UDim2.new(1, 0, 1, 0); b.BackgroundColor3 = Theme.CardBG; b.Text = "  "..text; b.Font = Enum.Font.GothamMedium; b.TextColor3 = Theme.TextWhite; b.TextSize = 13; b.TextXAlignment = Enum.TextXAlignment.Left; Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
    local sel = Instance.new("TextLabel", b); sel.Size = UDim2.new(0, 100, 1, 0); sel.Position = UDim2.new(1, -135, 0, 0); sel.BackgroundTransparency = 1; sel.Text = displayNames[Settings[key]]; sel.TextColor3 = Theme.Accent; sel.Font = Enum.Font.GothamBold; sel.TextSize = 12; sel.TextXAlignment = Enum.TextXAlignment.Right
    local dropFrame = Instance.new("Frame", b); dropFrame.Size = UDim2.new(1, 0, 0, 0); dropFrame.Position = UDim2.new(0, 0, 1, 2); dropFrame.BackgroundColor3 = Theme.DropdownBG; dropFrame.ClipsDescendants = true; dropFrame.ZIndex = 100; Instance.new("UIStroke", dropFrame).Color = Theme.Separator
    Instance.new("UIListLayout", dropFrame)
    for _, opt in ipairs(options) do
        local o = Instance.new("TextButton", dropFrame); o.Size = UDim2.new(1, 0, 0, 30); o.BackgroundColor3 = Theme.DropdownBG; o.Text = "  "..displayNames[opt]; o.Font = Enum.Font.Gotham; o.TextColor3 = Theme.TextWhite; o.TextSize = 12; o.TextXAlignment = Enum.TextXAlignment.Left; o.ZIndex = 101
        o.Activated:Connect(function() Settings[key] = opt; sel.Text = displayNames[opt]; TweenService:Create(dropFrame, TweenInfo.new(0.2), {Size = UDim2.new(1, 0, 0, 0)}):Play() end)
    end
    b.Activated:Connect(function() local isOpen = dropFrame.Size.Y.Offset > 0; TweenService:Create(dropFrame, TweenInfo.new(0.25), {Size = isOpen and UDim2.new(1, 0, 0, 0) or UDim2.new(1, 0, 0, 90)}):Play() end)
end

-- // CONFIGURANDO AS SEÇÕES
createSubTabs(AimPage, {
    Aimbot = function(page)
        local combat = createCard(page, "Main Execution")
        createToggle(combat, "Enable Aimbot", "Enabled")
        createToggle(combat, "Wall Check", "WallCheck")
    end,
    ["Configurações"] = function(page)
        local settings = createCard(page, "Advanced Parameters")
        createSlider(settings, "FOV Range", 20, 250, "FOV", false)
        createSlider(settings, "Smooth Speed", 0.1, 1.0, "Smoothing", true)
        createDropdown(settings, "Target Hitbox", "AimPart")
        createToggle(settings, "Head Dot", "ESP_HeadDot")
    end
})

createSubTabs(EspPage, {
    Players = function(page)
        local visuals = createCard(page, "Global ESP")
        createToggle(visuals, "Master ESP", "ESP_Master")
        createToggle(visuals, "Box ESP", "ESP_Box")
        createToggle(visuals, "Player Names", "ESP_Name")
        createToggle(visuals, "Health Bar", "ESP_Health")
    end,
    Settings = function(page)
        local visualSettings = createCard(page, "Visual Options")
        createSlider(visualSettings, "Box Opacity", 0, 1, "ESP_BoxTransparency", true)
    end
})

createSubTabs(MiscPage, {
    Menu = function(page)
        local misc = createCard(page, "Interface Settings")
        createToggle(misc, "Rainbow Mode", "RainbowMode")
    end
})

-- // TABS DA SIDEBAR
local function createMainTab(e, p)
    local b = Instance.new("TextButton", Sidebar); b.Size = UDim2.new(0, 45, 0, 45); b.Position = UDim2.new(0.5, -22, 0, (p=="Aim" and 15 or (p=="Esp" and 70 or 125))); b.BackgroundColor3 = Theme.CardBG; b.Text = e; b.TextSize = 22; Instance.new("UICorner", b).CornerRadius = UDim.new(0, 10)
    b.Activated:Connect(function() AimPage.Visible = (p=="Aim"); EspPage.Visible = (p=="Esp"); MiscPage.Visible = (p=="Misc") end)
end
createMainTab("🎯", "Aim"); createMainTab("👁️", "Esp"); createMainTab("🌍", "Misc")

-- // BOTÃO K
local OpenBtn = Instance.new("TextButton", ScreenGui); OpenBtn.Size = UDim2.new(0, 50, 0, 50); OpenBtn.Position = UDim2.new(0.02, 0, 0.2, 0); OpenBtn.BackgroundColor3 = Theme.MainBG; OpenBtn.Text = "K"; OpenBtn.TextColor3 = Theme.Accent; OpenBtn.Font = Enum.Font.GothamBlack; OpenBtn.TextSize = 25; Instance.new("UICorner", OpenBtn).CornerRadius = UDim.new(1,0); Instance.new("UIStroke", OpenBtn).Color = Theme.Accent
OpenBtn.Activated:Connect(function() MainFrame.Visible = not MainFrame.Visible; UIScale.Scale = MainFrame.Visible and 1 or 0 end)

-- Aplicar o novo Drag
makeDrag(MainFrame)
makeDrag(OpenBtn)

local FOVCircle = Instance.new("Frame", ScreenGui); FOVCircle.AnchorPoint = Vector2.new(0.5, 0.5); FOVCircle.Position = UDim2.new(0.5, 0, 0.5, 0); FOVCircle.BackgroundTransparency = 1; local CircleStroke = Instance.new("UIStroke", FOVCircle); CircleStroke.Color = Theme.Accent; Instance.new("UICorner", FOVCircle).CornerRadius = UDim.new(1, 0)

-- // LOOP CORE
RunService.RenderStepped:Connect(function()
    local dynamicColor = Theme.Accent
    if Settings.RainbowMode then dynamicColor = Color3.fromHSV(tick() % 4 / 4, 1, 1) end

    local t = (tick() * 1.2) % 2 - 1
    TitleGradient.Offset = Vector2.new(t, 0)
    TitleGradient.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.new(1,1,1)), ColorSequenceKeypoint.new(0.15, Color3.new(1,1,1)), ColorSequenceKeypoint.new(0.5, dynamicColor), ColorSequenceKeypoint.new(0.85, Color3.new(1,1,1)), ColorSequenceKeypoint.new(1, Color3.new(1,1,1))})

    OpenBtn.TextColor3 = dynamicColor; CircleStroke.Color = dynamicColor; NotifBar.BackgroundColor3 = dynamicColor
    FOVCircle.Visible = Settings.Enabled; FOVCircle.Size = UDim2.new(0, Settings.FOV * 2, 0, Settings.FOV * 2)

    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    local target = nil; local dist = Settings.FOV

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        local char = plr.Character; local obj = ESP_Objects[plr]
        if char and char:FindFirstChild("HumanoidRootPart") and char.Humanoid.Health > 0 then
            local head = char:FindFirstChild("Head")
            if head then
                local hPos, onS = Camera:WorldToViewportPoint(head.Position)
                local visible = #Camera:GetPartsObscuringTarget({Camera.CFrame.Position, head.Position}, {LocalPlayer.Character, char}) == 0
                local espColor = visible and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
                
                if Settings.ESP_Master and onS then
                    if not obj then
                        obj = {Main = Instance.new("Frame", ScreenGui)}
                        obj.Main.BackgroundTransparency = 1; obj.Box = Instance.new("Frame", obj.Main); obj.Box.BorderSizePixel = 0; Instance.new("UIStroke", obj.Box)
                        obj.NameTag = Instance.new("TextLabel", obj.Main); obj.NameTag.BackgroundTransparency = 1; obj.NameTag.Font = Enum.Font.GothamBold; obj.NameTag.TextSize = 10; Instance.new("UIStroke", obj.NameTag)
                        obj.HealthBar = Instance.new("Frame", obj.Main); obj.HeadDot = Instance.new("Frame", obj.Main); obj.HeadDot.Size = UDim2.new(0,6,0,6); Instance.new("UICorner", obj.HeadDot).CornerRadius = UDim.new(1,0)
                        ESP_Objects[plr] = obj
                    end
                    local fPos = Camera:WorldToViewportPoint(char.HumanoidRootPart.Position - Vector3.new(0, 3, 0))
                    local h = math.abs(hPos.Y - fPos.Y); local w = h * 0.6
                    obj.Main.Visible = true; obj.Box.Visible = Settings.ESP_Box; obj.Box.Size = UDim2.new(0, w, 0, h); obj.Box.Position = UDim2.new(0, hPos.X-w/2, 0, hPos.Y); obj.Box.BackgroundColor3 = espColor; obj.Box.BackgroundTransparency = Settings.ESP_BoxTransparency
                    obj.NameTag.Visible = Settings.ESP_Name; obj.NameTag.Text = plr.Name; obj.NameTag.Position = UDim2.new(0, hPos.X-50, 0, hPos.Y-15); obj.NameTag.Size = UDim2.new(0,100,0,15); obj.NameTag.TextColor3 = dynamicColor
                    local hp = char.Humanoid.Health/char.Humanoid.MaxHealth; obj.HealthBar.Visible = Settings.ESP_Health; obj.HealthBar.Size = UDim2.new(0, 2, hp*h, 0); obj.HealthBar.Position = UDim2.new(0, hPos.X-w/2-5, 0, hPos.Y + (h*(1-hp))); obj.HealthBar.BackgroundColor3 = Color3.new(0, 1, 0)
                    obj.HeadDot.Visible = Settings.ESP_HeadDot; obj.HeadDot.Position = UDim2.new(0, hPos.X-3, 0, hPos.Y-3); obj.HeadDot.BackgroundColor3 = dynamicColor
                elseif obj then obj.Main.Visible = false end

                if Settings.Enabled and onS then
                    local mag = (Vector2.new(hPos.X, hPos.Y) - center).Magnitude
                    if mag < dist then 
                        if not Settings.WallCheck or visible then
                            dist = mag
                            target = char:FindFirstChild(Settings.AimPart) or head
                        end
                    end
                end
            end
        elseif obj then obj.Main.Visible = false end
    end
    if target then Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, target.Position), Settings.Smoothing) end
end)
