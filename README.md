local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

---------------------------------------------------
-- CONFIG
---------------------------------------------------

local Config = {
    MenuOpened = true,
    Enabled    = false,
    FOVEnabled = false,
    FOVRadius  = 180,
    TeamCheck  = false,
    WallCheck  = false,
    HardLock   = false,
    LockPart   = "Head",
    ESP = {
        Box      = false,
        Linha    = false,
        Skeleton = false,
        Vida     = false,
        Dist     = false,
    }
}

---------------------------------------------------
-- SCREEN GUI
---------------------------------------------------

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AimlockUniversalGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.DisplayOrder = 9999
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.IgnoreGuiInset = true
ScreenGui.Parent = PlayerGui

---------------------------------------------------
-- MAIN FRAME
---------------------------------------------------

local menuWidth, menuHeight = 500, 440

local Main = Instance.new("Frame")
Main.Name = "Main"
Main.Size = UDim2.new(0, menuWidth, 0, menuHeight)
Main.AnchorPoint = Vector2.new(0, 0)
Main.Position = UDim2.new(0, 0, 0, 0)
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
Main.BorderSizePixel = 0
Main.Visible = Config.MenuOpened
Main.ClipsDescendants = true
Main.ZIndex = 50
Main.Active = false
Main.Parent = ScreenGui
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 8)

task.defer(function()
    local vp = Camera.ViewportSize
    Main.Position = UDim2.new(0, (vp.X - menuWidth)/2, 0, (vp.Y - menuHeight)/2)
end)

---------------------------------------------------
-- TITLEBAR
---------------------------------------------------

local dragging, dragOffset

local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = Color3.fromRGB(12, 12, 12)
TitleBar.BorderSizePixel = 0
TitleBar.ZIndex = 55
TitleBar.Active = true
TitleBar.Parent = Main

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragOffset = Vector2.new(
            input.Position.X - Main.AbsolutePosition.X,
            input.Position.Y - Main.AbsolutePosition.Y
        )
    end
end)

TitleBar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        Main.Position = UDim2.new(0,
            input.Position.X - dragOffset.X,
            0,
            input.Position.Y - dragOffset.Y
        )
    end
end)

local TitleName = Instance.new("TextLabel")
TitleName.Size = UDim2.new(1, -80, 1, 0)
TitleName.Position = UDim2.new(0, 14, 0, 0)
TitleName.Text = "Aimlock Universal"
TitleName.BackgroundTransparency = 1
TitleName.TextColor3 = Color3.fromRGB(240, 240, 240)
TitleName.Font = Enum.Font.GothamBold
TitleName.TextSize = 15
TitleName.TextXAlignment = Enum.TextXAlignment.Left
TitleName.ZIndex = 56
TitleName.Parent = TitleBar

local MinButton = Instance.new("TextButton")
MinButton.Size = UDim2.new(0, 28, 0, 28)
MinButton.Position = UDim2.new(1, -66, 0.5, -14)
MinButton.Text = "−"
MinButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
MinButton.TextColor3 = Color3.fromRGB(200, 200, 200)
MinButton.Font = Enum.Font.GothamBold
MinButton.TextSize = 16
MinButton.BorderSizePixel = 0
MinButton.ZIndex = 57
MinButton.Active = true
MinButton.Selectable = true
MinButton.Parent = TitleBar
Instance.new("UICorner", MinButton).CornerRadius = UDim.new(0, 6)

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 28, 0, 28)
CloseButton.Position = UDim2.new(1, -34, 0.5, -14)
CloseButton.Text = "✕"
CloseButton.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 13
CloseButton.BorderSizePixel = 0
CloseButton.ZIndex = 57
CloseButton.Active = true
CloseButton.Selectable = true
CloseButton.Parent = TitleBar
Instance.new("UICorner", CloseButton).CornerRadius = UDim.new(0, 6)

---------------------------------------------------
-- MENU LATERAL
---------------------------------------------------

local SideMenu = Instance.new("Frame")
SideMenu.Size = UDim2.new(0, 110, 1, -40)
SideMenu.Position = UDim2.new(0, 0, 0, 40)
SideMenu.BackgroundColor3 = Color3.fromRGB(14, 14, 14)
SideMenu.BorderSizePixel = 0
SideMenu.ZIndex = 51
SideMenu.Active = false
SideMenu.Parent = Main

local SidePad = Instance.new("UIPadding", SideMenu)
SidePad.PaddingTop = UDim.new(0, 8)
SidePad.PaddingLeft = UDim.new(0, 6)
SidePad.PaddingRight = UDim.new(0, 6)

local SideList = Instance.new("UIListLayout", SideMenu)
SideList.Padding = UDim.new(0, 3)
SideList.SortOrder = Enum.SortOrder.LayoutOrder

local tabData = {
    {name = "Home",    icon = ""},
    {name = "ESP",     icon = ""},
    {name = "Jogador", icon = ""},
    {name = "Config",  icon = ""},
}

local tabButtons = {}
local currentTab = "Home"

local function setTabActive(btn, active)
    btn.BackgroundColor3 = active
        and Color3.fromRGB(35, 35, 35)
        or  Color3.fromRGB(20, 20, 20)
    btn.TextColor3 = active
        and Color3.fromRGB(255, 255, 255)
        or  Color3.fromRGB(120, 120, 120)
end

for i, t in ipairs(tabData) do
    local TabBtn = Instance.new("TextButton")
    TabBtn.Size = UDim2.new(1, 0, 0, 34)
    TabBtn.Text = t.name
    TabBtn.LayoutOrder = i
    TabBtn.Font = Enum.Font.Gotham
    TabBtn.TextSize = 13
    TabBtn.TextXAlignment = Enum.TextXAlignment.Left
    TabBtn.BorderSizePixel = 0
    TabBtn.AutoButtonColor = false
    TabBtn.Active = true
    TabBtn.Selectable = true
    TabBtn.ZIndex = 52
    TabBtn.Parent = SideMenu
    Instance.new("UICorner", TabBtn).CornerRadius = UDim.new(0, 5)
    local p = Instance.new("UIPadding", TabBtn)
    p.PaddingLeft = UDim.new(0, 10)
    setTabActive(TabBtn, t.name == currentTab)
    tabButtons[t.name] = TabBtn
end

---------------------------------------------------
-- ÁREA DE CONTEÚDO
---------------------------------------------------

local ContentArea = Instance.new("Frame")
ContentArea.Size = UDim2.new(1, -118, 1, -48)
ContentArea.Position = UDim2.new(0, 114, 0, 44)
ContentArea.BackgroundTransparency = 1
ContentArea.BorderSizePixel = 0
ContentArea.ZIndex = 54
ContentArea.Active = false
ContentArea.Parent = Main

---------------------------------------------------
-- HELPERS — SWITCH TOGGLE
---------------------------------------------------

local function makeSwitch(parent, posY, label)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, -8, 0, 38)
    row.Position = UDim2.new(0, 4, 0, posY)
    row.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    row.BorderSizePixel = 0
    row.ZIndex = 55
    row.Parent = parent
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 6)

    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -70, 1, 0)
    lbl.Position = UDim2.new(0, 12, 0, 0)
    lbl.Text = label
    lbl.BackgroundTransparency = 1
    lbl.Font = Enum.Font.Gotham
    lbl.TextSize = 13
    lbl.TextColor3 = Color3.fromRGB(210, 210, 210)
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.ZIndex = 56
    lbl.Parent = row

    -- Switch track
    local track = Instance.new("Frame")
    track.Size = UDim2.new(0, 44, 0, 24)
    track.Position = UDim2.new(1, -54, 0.5, -12)
    track.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    track.BorderSizePixel = 0
    track.ZIndex = 56
    track.Parent = row
    Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)

    -- Switch knob
    local knob = Instance.new("Frame")
    knob.Size = UDim2.new(0, 18, 0, 18)
    knob.Position = UDim2.new(0, 3, 0.5, -9)
    knob.BackgroundColor3 = Color3.fromRGB(180, 180, 180)
    knob.BorderSizePixel = 0
    knob.ZIndex = 57
    knob.Parent = track
    Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

    local state = false

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.BackgroundTransparency = 1
    btn.Text = ""
    btn.ZIndex = 58
    btn.Active = true
    btn.Selectable = true
    btn.Parent = row

    local function setState(on)
        state = on
        if on then
            track.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
            knob.Position = UDim2.new(1, -21, 0.5, -9)
            knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        else
            track.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
            knob.Position = UDim2.new(0, 3, 0.5, -9)
            knob.BackgroundColor3 = Color3.fromRGB(180, 180, 180)
        end
    end

    btn.MouseButton1Click:Connect(function()
        setState(not state)
    end)

    return btn, setState, function() return state end
end

---------------------------------------------------
-- PÁGINAS
---------------------------------------------------

local pages = {}

-- HOME
local pageHome = Instance.new("Frame")
pageHome.Size = UDim2.new(1, 0, 1, 0)
pageHome.BackgroundTransparency = 1
pageHome.ZIndex = 54
pageHome.Parent = ContentArea

local btnEnable, setEnable, getEnable = makeSwitch(pageHome, 0, "Ativar Aimlock")
local btnFOV, setFOV, getFOV = makeSwitch(pageHome, 46, "FOV Circle")

-- FOV Slider horizontal
local sliderCard = Instance.new("Frame")
sliderCard.Size = UDim2.new(1, -8, 0, 38)
sliderCard.Position = UDim2.new(0, 4, 0, 92)
sliderCard.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
sliderCard.BorderSizePixel = 0
sliderCard.ZIndex = 55
sliderCard.Parent = pageHome
Instance.new("UICorner", sliderCard).CornerRadius = UDim.new(0, 6)

local sliderLbl = Instance.new("TextLabel")
sliderLbl.Size = UDim2.new(0.4, 0, 0, 16)
sliderLbl.Position = UDim2.new(0, 12, 0, 4)
sliderLbl.Text = "FOV: " .. Config.FOVRadius
sliderLbl.BackgroundTransparency = 1
sliderLbl.Font = Enum.Font.Gotham
sliderLbl.TextSize = 12
sliderLbl.TextColor3 = Color3.fromRGB(160, 160, 160)
sliderLbl.TextXAlignment = Enum.TextXAlignment.Left
sliderLbl.ZIndex = 56
sliderLbl.Parent = sliderCard

local sliderTrack = Instance.new("Frame")
sliderTrack.Size = UDim2.new(1, -24, 0, 6)
sliderTrack.Position = UDim2.new(0, 12, 0.5, 6)
sliderTrack.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
sliderTrack.BorderSizePixel = 0
sliderTrack.ZIndex = 56
sliderTrack.Parent = sliderCard
Instance.new("UICorner", sliderTrack).CornerRadius = UDim.new(1, 0)

local sliderFill = Instance.new("Frame")
sliderFill.Size = UDim2.new(Config.FOVRadius/700, 0, 1, 0)
sliderFill.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
sliderFill.BorderSizePixel = 0
sliderFill.ZIndex = 57
sliderFill.Parent = sliderTrack
Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)

local sliderKnob = Instance.new("Frame")
sliderKnob.Size = UDim2.new(0, 16, 0, 16)
sliderKnob.AnchorPoint = Vector2.new(0.5, 0.5)
sliderKnob.Position = UDim2.new(Config.FOVRadius/700, 0, 0.5, 0)
sliderKnob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
sliderKnob.BorderSizePixel = 0
sliderKnob.ZIndex = 58
sliderKnob.Parent = sliderTrack
Instance.new("UICorner", sliderKnob).CornerRadius = UDim.new(1, 0)

local sliderBtn = Instance.new("TextButton")
sliderBtn.Size = UDim2.new(1, 0, 1, 0)
sliderBtn.BackgroundTransparency = 1
sliderBtn.Text = ""
sliderBtn.ZIndex = 59
sliderBtn.Active = true
sliderBtn.Parent = sliderTrack

local sliding = false

sliderBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        sliding = true
    end
end)

sliderBtn.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        sliding = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if sliding and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        local trackPos = sliderTrack.AbsolutePosition.X
        local trackWidth = sliderTrack.AbsoluteSize.X
        local rel = math.clamp((input.Position.X - trackPos) / trackWidth, 0, 1)
        Config.FOVRadius = math.floor(rel * 700)
        if Config.FOVRadius < 10 then Config.FOVRadius = 10 end
        sliderFill.Size = UDim2.new(rel, 0, 1, 0)
        sliderKnob.Position = UDim2.new(rel, 0, 0.5, 0)
        sliderLbl.Text = "FOV: " .. Config.FOVRadius
    end
end)

local btnTeam, setTeam, getTeam = makeSwitch(pageHome, 138, "Team Check")
local btnWall, setWall, getWall = makeSwitch(pageHome, 184, "Wall Check")
local btnLock, setLock, getLock = makeSwitch(pageHome, 230, "Lock")

-- Seletor de parte do corpo
local partCard = Instance.new("Frame")
partCard.Size = UDim2.new(1, -8, 0, 38)
partCard.Position = UDim2.new(0, 4, 0, 276)
partCard.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
partCard.BorderSizePixel = 0
partCard.ZIndex = 55
partCard.Parent = pageHome
Instance.new("UICorner", partCard).CornerRadius = UDim.new(0, 6)

local partLbl = Instance.new("TextLabel")
partLbl.Size = UDim2.new(0.4, 0, 1, 0)
partLbl.Position = UDim2.new(0, 12, 0, 0)
partLbl.Text = "Parte:"
partLbl.BackgroundTransparency = 1
partLbl.Font = Enum.Font.Gotham
partLbl.TextSize = 13
partLbl.TextColor3 = Color3.fromRGB(210, 210, 210)
partLbl.TextXAlignment = Enum.TextXAlignment.Left
partLbl.ZIndex = 56
partLbl.Parent = partCard

local partOptions = {"Cabeça", "Tronco", "Perna"}
local partIndex = 1
local partMap = {
    ["Cabeça"] = "Head",
    ["Tronco"] = "HumanoidRootPart",
    ["Perna"]  = "LeftFoot",
}

local partDisplay = Instance.new("TextLabel")
partDisplay.Size = UDim2.new(0, 90, 0, 26)
partDisplay.Position = UDim2.new(1, -130, 0.5, -13)
partDisplay.Text = partOptions[partIndex]
partDisplay.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
partDisplay.Font = Enum.Font.Gotham
partDisplay.TextSize = 12
partDisplay.TextColor3 = Color3.fromRGB(220, 220, 220)
partDisplay.BorderSizePixel = 0
partDisplay.ZIndex = 56
partDisplay.Parent = partCard
Instance.new("UICorner", partDisplay).CornerRadius = UDim.new(0, 5)

local partNext = Instance.new("TextButton")
partNext.Size = UDim2.new(0, 26, 0, 26)
partNext.Position = UDim2.new(1, -34, 0.5, -13)
partNext.Text = "›"
partNext.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
partNext.TextColor3 = Color3.fromRGB(220, 220, 220)
partNext.Font = Enum.Font.GothamBold
partNext.TextSize = 16
partNext.BorderSizePixel = 0
partNext.ZIndex = 57
partNext.Active = true
partNext.Selectable = true
partNext.Parent = partCard
Instance.new("UICorner", partNext).CornerRadius = UDim.new(0, 5)

local partPrev = Instance.new("TextButton")
partPrev.Size = UDim2.new(0, 26, 0, 26)
partPrev.Position = UDim2.new(1, -162, 0.5, -13)
partPrev.Text = "‹"
partPrev.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
partPrev.TextColor3 = Color3.fromRGB(220, 220, 220)
partPrev.Font = Enum.Font.GothamBold
partPrev.TextSize = 16
partPrev.BorderSizePixel = 0
partPrev.ZIndex = 57
partPrev.Active = true
partPrev.Selectable = true
partPrev.Parent = partCard
Instance.new("UICorner", partPrev).CornerRadius = UDim.new(0, 5)

local function updatePart()
    partDisplay.Text = partOptions[partIndex]
    Config.LockPart = partMap[partOptions[partIndex]]
end

partNext.MouseButton1Click:Connect(function()
    partIndex = partIndex % #partOptions + 1
    updatePart()
end)

partPrev.MouseButton1Click:Connect(function()
    partIndex = ((partIndex - 2) % #partOptions) + 1
    updatePart()
end)

pages["Home"] = pageHome

-- ESP
local pageESP = Instance.new("Frame")
pageESP.Size = UDim2.new(1, 0, 1, 0)
pageESP.BackgroundTransparency = 1
pageESP.ZIndex = 54
pageESP.Visible = false
pageESP.Parent = ContentArea

local btnBox,  setBox,  getBox  = makeSwitch(pageESP, 0,   "Box ESP")
local btnLinha, setLinha, getLinha = makeSwitch(pageESP, 46,  "Linha ESP")
local btnSkel, setSkel, getSkel = makeSwitch(pageESP, 92,  "Esqueleto ESP")
local btnVida, setVida, getVida = makeSwitch(pageESP, 138, "Vida ESP")
local btnDist, setDist, getDist = makeSwitch(pageESP, 184, "Distância ESP")

pages["ESP"] = pageESP

local pageJogador = Instance.new("Frame")
pageJogador.Size = UDim2.new(1, 0, 1, 0)
pageJogador.BackgroundTransparency = 1
pageJogador.ZIndex = 54
pageJogador.Visible = false
pageJogador.Parent = ContentArea
pages["Jogador"] = pageJogador

local pageConfig = Instance.new("Frame")
pageConfig.Size = UDim2.new(1, 0, 1, 0)
pageConfig.BackgroundTransparency = 1
pageConfig.ZIndex = 54
pageConfig.Visible = false
pageConfig.Parent = ContentArea
pages["Config"] = pageConfig

---------------------------------------------------
-- TROCA DE ABAS
---------------------------------------------------

local function switchTab(name)
    currentTab = name
    for tabName, btn in pairs(tabButtons) do
        setTabActive(btn, tabName == name)
    end
    for pageName, page in pairs(pages) do
        page.Visible = pageName == name
    end
end

for _, t in ipairs(tabData) do
    tabButtons[t.name].MouseButton1Click:Connect(function()
        switchTab(t.name)
    end)
end

---------------------------------------------------
-- FOV CIRCLE
---------------------------------------------------

local FOVFrame = Instance.new("Frame")
FOVFrame.BackgroundTransparency = 1
FOVFrame.BorderSizePixel = 0
FOVFrame.ZIndex = 1
FOVFrame.Visible = false
FOVFrame.Parent = ScreenGui

local FOVInner = Instance.new("Frame")
FOVInner.BackgroundTransparency = 1
FOVInner.BorderSizePixel = 0
FOVInner.Parent = FOVFrame
Instance.new("UICorner", FOVInner).CornerRadius = UDim.new(1, 0)

local fovStroke = Instance.new("UIStroke", FOVInner)
fovStroke.Color = Color3.fromRGB(255, 255, 255)
fovStroke.Thickness = 1.5
fovStroke.Transparency = 0.1

---------------------------------------------------
-- ESP ENGINE
---------------------------------------------------

local espObjects = {}

local function getESPFor(player)
    if not espObjects[player] then
        espObjects[player] = {}
    end
    return espObjects[player]
end

local function clearESP(player)
    local t = espObjects[player]
    if not t then return end
    for _, obj in pairs(t) do
        if obj and obj.Remove then
            pcall(function() obj:Remove() end)
        elseif typeof(obj) == "Instance" then
            pcall(function() obj:Destroy() end)
        end
    end
    espObjects[player] = nil
end

local SKELETON_CONNECTIONS = {
    {"Head", "UpperTorso"},
    {"UpperTorso", "LowerTorso"},
    {"UpperTorso", "LeftUpperArm"},
    {"LeftUpperArm", "LeftLowerArm"},
    {"LeftLowerArm", "LeftHand"},
    {"UpperTorso", "RightUpperArm"},
    {"RightUpperArm", "RightLowerArm"},
    {"RightLowerArm", "RightHand"},
    {"LowerTorso", "LeftUpperLeg"},
    {"LeftUpperLeg", "LeftLowerLeg"},
    {"LeftLowerLeg", "LeftFoot"},
    {"LowerTorso", "RightUpperLeg"},
    {"RightUpperLeg", "RightLowerLeg"},
    {"RightLowerLeg", "RightFoot"},
}

RunService.RenderStepped:Connect(function()
    -- FOV circle
    local showFOV = getEnable() and getFOV()
    FOVFrame.Visible = showFOV
    if showFOV then
        local r = Config.FOVRadius
        local cx = Camera.ViewportSize.X / 2
        local cy = Camera.ViewportSize.Y / 2
        FOVFrame.Size = UDim2.new(0, r*2, 0, r*2)
        FOVFrame.Position = UDim2.new(0, cx-r, 0, cy-r)
        FOVInner.Size = UDim2.new(1, 0, 1, 0)
    end

    -- ESP
    local anyESP = getBox() or getLinha() or getSkel() or getVida() or getDist()

    for _, player in ipairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        local char = player.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        local esp = getESPFor(player)

        if not anyESP or not hrp or not hum then
            -- limpa drawings existentes
            for k, obj in pairs(esp) do
                pcall(function()
                    if typeof(obj) == "table" then
                        for _, o in pairs(obj) do pcall(function() o.Visible = false end) end
                    else
                        obj.Visible = false
                    end
                end)
            end
            continue
        end

        local rootPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
        local headPart = char:FindFirstChild("Head")
        local headPos2 = headPart and Camera:WorldToViewportPoint(headPart.Position + Vector3.new(0,0.7,0))

        -- BOX ESP
        if getBox() then
            if not esp.box then
                esp.box = Drawing.new("Square")
                esp.box.Color = Color3.fromRGB(0, 220, 255)
                esp.box.Thickness = 1.5
                esp.box.Filled = false
                esp.name = Drawing.new("Text")
                esp.name.Color = Color3.fromRGB(255, 255, 255)
                esp.name.Size = 13
                esp.name.Center = true
                esp.name.Outline = true
            end
            if onScreen and headPos2 then
                local h = math.abs(rootPos.Y - headPos2.Y) * 2
                local w = h * 0.6
                esp.box.Visible = true
                esp.box.Size = Vector2.new(w, h)
                esp.box.Position = Vector2.new(rootPos.X - w/2, headPos2.Y - 2)
                esp.name.Visible = true
                esp.name.Text = player.Name
                esp.name.Position = Vector2.new(rootPos.X, headPos2.Y - 16)
            else
                esp.box.Visible = false
                if esp.name then esp.name.Visible = false end
            end
        else
            if esp.box then esp.box.Visible = false end
            if esp.name then esp.name.Visible = false end
        end

        -- LINHA ESP
        if getLinha() then
            if not esp.linha then
                esp.linha = Drawing.new("Line")
                esp.linha.Color = Color3.fromRGB(0, 255, 0)
                esp.linha.Thickness = 1
            end
            -- some quando olha pra cima (câmera virada pra cima)
            local camLook = Camera.CFrame.LookVector
            local lookingUp = camLook.Y > 0.5
            if onScreen and not lookingUp then
                esp.linha.Visible = true
                esp.linha.From = Vector2.new(Camera.ViewportSize.X/2, 0)
                esp.linha.To = Vector2.new(rootPos.X, rootPos.Y)
            else
                esp.linha.Visible = false
            end
        else
            if esp.linha then esp.linha.Visible = false end
        end

        -- ESQUELETO ESP
        if getSkel() then
            if not esp.skel then
                esp.skel = {}
                for i = 1, #SKELETON_CONNECTIONS do
                    local line = Drawing.new("Line")
                    line.Color = Color3.fromRGB(255, 165, 0)
                    line.Thickness = 1
                    line.Visible = false
                    esp.skel[i] = line
                end
            end
            for i, conn in ipairs(SKELETON_CONNECTIONS) do
                local p1 = char:FindFirstChild(conn[1])
                local p2 = char:FindFirstChild(conn[2])
                local line = esp.skel[i]
                if p1 and p2 then
                    local s1, vis1 = Camera:WorldToViewportPoint(p1.Position)
                    local s2, vis2 = Camera:WorldToViewportPoint(p2.Position)
                    if vis1 and vis2 then
                        line.Visible = true
                        line.From = Vector2.new(s1.X, s1.Y)
                        line.To = Vector2.new(s2.X, s2.Y)
                    else
                        line.Visible = false
                    end
                else
                    line.Visible = false
                end
            end
        else
            if esp.skel then
                for _, l in pairs(esp.skel) do l.Visible = false end
            end
        end

        -- VIDA ESP
        if getVida() then
            if not esp.vida then
                esp.vida = Drawing.new("Square")
                esp.vida.Filled = true
                esp.vida.Color = Color3.fromRGB(0, 220, 60)
                esp.vida.Thickness = 0
                esp.vidaBg = Drawing.new("Square")
                esp.vidaBg.Filled = true
                esp.vidaBg.Color = Color3.fromRGB(40, 40, 40)
                esp.vidaBg.Thickness = 0
            end
            if onScreen and headPos2 then
                local h = math.abs(rootPos.Y - headPos2.Y) * 2
                local w = h * 0.6
                local barW = 4
                local barX = rootPos.X - w/2 - 6
                local barY = headPos2.Y - 2
                local hp = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
                esp.vidaBg.Visible = true
                esp.vidaBg.Position = Vector2.new(barX, barY)
                esp.vidaBg.Size = Vector2.new(barW, h)
                esp.vida.Visible = true
                esp.vida.Position = Vector2.new(barX, barY + h * (1 - hp))
                esp.vida.Size = Vector2.new(barW, h * hp)
                -- cor muda com hp
                if hp > 0.6 then
                    esp.vida.Color = Color3.fromRGB(0, 220, 60)
                elseif hp > 0.3 then
                    esp.vida.Color = Color3.fromRGB(255, 200, 0)
                else
                    esp.vida.Color = Color3.fromRGB(220, 40, 40)
                end
            else
                esp.vida.Visible = false
                esp.vidaBg.Visible = false
            end
        else
            if esp.vida then esp.vida.Visible = false end
            if esp.vidaBg then esp.vidaBg.Visible = false end
        end

        -- DISTÂNCIA ESP
        if getDist() then
            if not esp.dist then
                esp.dist = Drawing.new("Text")
                esp.dist.Color = Color3.fromRGB(255, 255, 255)
                esp.dist.Size = 12
                esp.dist.Center = true
                esp.dist.Outline = true
            end
            if onScreen then
                local localHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                local distVal = localHRP and math.floor((hrp.Position - localHRP.Position).Magnitude) or 0
                esp.dist.Visible = true
                esp.dist.Text = distVal .. "m"
                -- posiciona abaixo da barra de vida
                local h = headPos2 and math.abs(rootPos.Y - headPos2.Y) * 2 or 60
                esp.dist.Position = Vector2.new(rootPos.X, rootPos.Y + h/2 + 4)
            else
                esp.dist.Visible = false
            end
        else
            if esp.dist then esp.dist.Visible = false end
        end
    end

    -- Aimlock
    if not getEnable() then
        lockedTarget = nil
        return
    end

    local target = getTarget()
    if not target then return end

    local partName = Config.LockPart
    local aimPart = target.Parent and target.Parent:FindFirstChild(partName) or target
    local targetPos = aimPart.Position

    Camera.CameraType = Enum.CameraType.Custom
    if getLock() then
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPos)
    else
        local cf = Camera.CFrame
        Camera.CFrame = cf:Lerp(CFrame.new(cf.Position, targetPos), 0.3)
    end
end)

---------------------------------------------------
-- AIMLOCK
---------------------------------------------------

local lockedTarget = nil

local function isTargetValid(hrp)
    if not hrp or not hrp.Parent then return false end
    local hum = hrp.Parent:FindFirstChildOfClass("Humanoid")
    if not hum or hum.Health <= 0 then return false end
    if not getLock() then
        local _, onScreen = Camera:WorldToViewportPoint(hrp.Position)
        if not onScreen then return false end
    end
    return true
end

local function getClosestPlayer()
    local closest, closestDist = nil, math.huge
    local screenCenter = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    for _, player in ipairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        local char = player.Character
        if not char then continue end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum or hum.Health <= 0 then continue end
        if getTeam() and player.Team ~= nil and player.Team == LocalPlayer.Team then continue end
        local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
        if not onScreen then continue end
        if getWall() then
            local localChar = LocalPlayer.Character
            local localHRP = localChar and localChar:FindFirstChild("HumanoidRootPart")
            if localHRP then
                local rp = RaycastParams.new()
                rp.FilterDescendantsInstances = {localChar, char}
                rp.FilterType = Enum.RaycastFilterType.Exclude
                local result = workspace:Raycast(localHRP.Position, hrp.Position - localHRP.Position, rp)
                if result then continue end
            end
        end
        local dist = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
        if dist <= Config.FOVRadius and dist < closestDist then
            closestDist = dist
            closest = hrp
        end
    end
    return closest
end

function getTarget()
    if lockedTarget and isTargetValid(lockedTarget) then return lockedTarget end
    lockedTarget = getClosestPlayer()
    return lockedTarget
end

local Mouse = LocalPlayer:GetMouse()
local mt = getrawmetatable(game)
local oldIndex = mt.__index
setreadonly(mt, false)
mt.__index = newcclosure(function(self, key)
    if getEnable() and self == Mouse and key == "Hit" then
        local target = getTarget()
        if target then
            local partName = Config.LockPart
            local aimPart = target.Parent and target.Parent:FindFirstChild(partName) or target
            return CFrame.new(aimPart.Position)
        end
    end
    return oldIndex(self, key)
end)
setreadonly(mt, true)

Players.PlayerRemoving:Connect(function(player)
    clearESP(player)
    if player.Character then
        local hrp = player.Character:FindFirstChild("HumanoidRootPart")
        if hrp == lockedTarget then lockedTarget = nil end
    end
end)

---------------------------------------------------
-- MINI MENU
---------------------------------------------------

local MinimizedFrame = Instance.new("Frame")
MinimizedFrame.Name = "Mini"
MinimizedFrame.Size = UDim2.new(0, 140, 0, 68)
MinimizedFrame.AnchorPoint = Vector2.new(0, 0)
MinimizedFrame.Position = UDim2.new(0, 0, 0, 0)
MinimizedFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 12)
MinimizedFrame.BackgroundTransparency = 0.05
MinimizedFrame.ZIndex = 80
MinimizedFrame.ClipsDescendants = true
MinimizedFrame.Active = false
MinimizedFrame.Visible = not Config.MenuOpened
MinimizedFrame.Parent = ScreenGui
Instance.new("UICorner", MinimizedFrame).CornerRadius = UDim.new(0, 8)

task.defer(function()
    local vp = Camera.ViewportSize
    MinimizedFrame.Position = UDim2.new(0, vp.X - 152, 0, 90)
end)

local miniGrad = Instance.new("UIGradient", MinimizedFrame)
miniGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0,   Color3.fromRGB(80, 20, 160)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(15, 15, 15)),
    ColorSequenceKeypoint.new(1,   Color3.fromRGB(20, 80, 200)),
})
miniGrad.Rotation = 135

local miniTopBar = Instance.new("Frame")
miniTopBar.Size = UDim2.new(1, 0, 0, 46)
miniTopBar.Position = UDim2.new(0, 0, 0, 0)
miniTopBar.BackgroundTransparency = 1
miniTopBar.BorderSizePixel = 0
miniTopBar.ZIndex = 81
miniTopBar.Active = true
miniTopBar.Parent = MinimizedFrame

local miniVer = Instance.new("TextLabel")
miniVer.Size = UDim2.new(0, 28, 0, 16)
miniVer.Position = UDim2.new(0, 8, 0, 5)
miniVer.Text = "v1.0"
miniVer.BackgroundTransparency = 1
miniVer.Font = Enum.Font.Gotham
miniVer.TextSize = 10
miniVer.TextColor3 = Color3.fromRGB(140, 140, 140)
miniVer.ZIndex = 82
miniVer.Parent = miniTopBar

local miniNameLbl = Instance.new("TextLabel")
miniNameLbl.Size = UDim2.new(1, -50, 0, 16)
miniNameLbl.Position = UDim2.new(0, 36, 0, 5)
miniNameLbl.Text = "aimlock universal"
miniNameLbl.BackgroundTransparency = 1
miniNameLbl.Font = Enum.Font.GothamBold
miniNameLbl.TextSize = 12
miniNameLbl.TextColor3 = Color3.fromRGB(240, 240, 240)
miniNameLbl.TextXAlignment = Enum.TextXAlignment.Left
miniNameLbl.ZIndex = 82
miniNameLbl.Parent = miniTopBar

local miniDot = Instance.new("Frame")
miniDot.Size = UDim2.new(0, 8, 0, 8)
miniDot.Position = UDim2.new(1, -13, 0, 7)
miniDot.BackgroundColor3 = Color3.fromRGB(0, 220, 100)
miniDot.ZIndex = 82
miniDot.Parent = miniTopBar
Instance.new("UICorner", miniDot).CornerRadius = UDim.new(1, 1)

local miniArrow = Instance.new("TextLabel")
miniArrow.Size = UDim2.new(1, 0, 0, 22)
miniArrow.Position = UDim2.new(0, 0, 0, 22)
miniArrow.Text = "🏹"
miniArrow.TextScaled = true
miniArrow.BackgroundTransparency = 1
miniArrow.ZIndex = 82
miniArrow.Parent = miniTopBar

local miniDragging, miniDragOffset, miniDragMoved

miniTopBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        miniDragging = true
        miniDragMoved = false
        miniDragOffset = Vector2.new(
            input.Position.X - MinimizedFrame.AbsolutePosition.X,
            input.Position.Y - MinimizedFrame.AbsolutePosition.Y
        )
    end
end)

miniTopBar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        miniDragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if miniDragging and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        miniDragMoved = true
        MinimizedFrame.Position = UDim2.new(0,
            input.Position.X - miniDragOffset.X,
            0,
            input.Position.Y - miniDragOffset.Y
        )
    end
end)

local miniOpenBar = Instance.new("Frame")
miniOpenBar.Size = UDim2.new(1, 0, 0, 22)
miniOpenBar.Position = UDim2.new(0, 0, 1, -22)
miniOpenBar.BackgroundColor3 = Color3.fromRGB(8, 8, 8)
miniOpenBar.BackgroundTransparency = 0.2
miniOpenBar.BorderSizePixel = 0
miniOpenBar.ZIndex = 81
miniOpenBar.Active = false
miniOpenBar.Parent = MinimizedFrame

local OpenButton = Instance.new("TextButton")
OpenButton.Size = UDim2.new(1, 0, 1, 0)
OpenButton.Text = "OPEN ›"
OpenButton.Font = Enum.Font.Gotham
OpenButton.TextSize = 11
OpenButton.TextColor3 = Color3.fromRGB(180, 180, 180)
OpenButton.BackgroundTransparency = 1
OpenButton.BorderSizePixel = 0
OpenButton.Active = true
OpenButton.Selectable = true
OpenButton.ZIndex = 82
OpenButton.Parent = miniOpenBar

OpenButton.MouseButton1Click:Connect(function()
    if miniDragMoved then return end
    Config.MenuOpened = true
    Main.Visible = true
    MinimizedFrame.Visible = false
end)

---------------------------------------------------
-- MINIMIZAR / ABRIR
---------------------------------------------------

local function UpdateMenu()
    Main.Visible = Config.MenuOpened
    MinimizedFrame.Visible = not Config.MenuOpened
end

MinButton.MouseButton1Click:Connect(function()
    Config.MenuOpened = false
    UpdateMenu()
end)

CloseButton.MouseButton1Click:Connect(function()
    Config.MenuOpened = false
    UpdateMenu()
end)

UpdateMenu()
switchTab("Home")
