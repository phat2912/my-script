-- ==========================================================
-- SCRIPT MENU SYSTEM V10.5 ULTRA VIP (Kianbest Hub)
-- Updated: Advanced Multi-Mode Fix Lag & Performance Booster
-- ==========================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local Stats = game:GetService("Stats")
local TeleportService = game:GetService("TeleportService")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

-- 1. CẤU HÌNH HỆ THỐNG & FIX LAG MODES
local Config = {
    AntiBanEnabled = true,
    AntiKick = true,
    AntiLog = true,
    DisableClientAC = true,

    -- Fix Lag Extended Modes
    PotatoMode = false,
    RemoveShadows = false,
    RemoveTextures = false,
    FullBright = false,
    LowPoly = false,
    RemoveParticles = false,
    AutoMemoryClean = false,
    HidePlayers = false,
    RemoveVFX = false,

    -- Movement & Combat
    NoclipEnabled = false,
    SpeedEnabled = false,
    SpeedValue = 35,
    FlyEnabled = false,
    FlySpeed = 75,
    SelectedTarget = nil,
    AutoTPTarget = false,
    AutoAttack = false,
    SuperM1Damage = false,
    DamageMultiplier = 40,
    HitboxExpander = false,
    HitboxSize = 18,
    AutoSkills = false,

    AntiAFK = true,
    CurrentAnimeIndex = math.random(1, 10),
    BgTransparency = 0.35,
    FrameTransparency = 0.15,
    ToggleKey = Enum.KeyCode.RightControl
}

-- BỘ 10 ANIME GIRL HD PRESETS
local CuteAnimePresets = {
    "rbxassetid://11702739401",
    "rbxassetid://10023403248",
    "rbxassetid://6071575925",
    "rbxassetid://11414436906",
    "rbxassetid://7043825807",
    "rbxassetid://6985068228",
    "rbxassetid://7305531980",
    "rbxassetid://10878580644",
    "rbxassetid://11414438318",
    "rbxassetid://6880894541"
}

-- THEMES MÀU SẮC
local Themes = {
    CutePink = { Name = "Cute Pink 🌸", Bg = Color3.fromRGB(25, 18, 24), Sidebar = Color3.fromRGB(18, 12, 17), Accent = Color3.fromRGB(255, 120, 170), Button = Color3.fromRGB(38, 24, 34), Text = Color3.fromRGB(255, 240, 245) },
    CyberBlue = { Name = "Cyber Blue 💎", Bg = Color3.fromRGB(15, 22, 32), Sidebar = Color3.fromRGB(10, 15, 24), Accent = Color3.fromRGB(0, 195, 255), Button = Color3.fromRGB(20, 32, 48), Text = Color3.fromRGB(235, 245, 255) },
    BloodRed = { Name = "Blood Red 🩸", Bg = Color3.fromRGB(28, 15, 15), Sidebar = Color3.fromRGB(18, 9, 9), Accent = Color3.fromRGB(255, 60, 60), Button = Color3.fromRGB(42, 20, 20), Text = Color3.fromRGB(255, 235, 235) },
    DarkPurple = { Name = "Dark Purple 🔮", Bg = Color3.fromRGB(22, 15, 30), Sidebar = Color3.fromRGB(14, 9, 20), Accent = Color3.fromRGB(180, 100, 255), Button = Color3.fromRGB(34, 20, 48), Text = Color3.fromRGB(245, 235, 255) }
}

local CurrentTheme = Themes.CutePink

-- KHỞI TẠO SCREENGUI
local ParentGui = (gethui and gethui()) or CoreGui or LocalPlayer:WaitForChild("PlayerGui")
if ParentGui:FindFirstChild("KianbestMenuV10") then
    ParentGui.KianbestMenuV10:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KianbestMenuV10"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = ParentGui

-- NOTIFICATION TOAST SYSTEM
local NotificationFrame = Instance.new("Frame")
NotificationFrame.Name = "NotificationFrame"
NotificationFrame.Size = UDim2.new(0, 240, 0, 220)
NotificationFrame.Position = UDim2.new(1, -250, 1, -230)
NotificationFrame.BackgroundTransparency = 1
NotificationFrame.ZIndex = 50
NotificationFrame.Parent = ScreenGui

local NotificationList = Instance.new("UIListLayout", NotificationFrame)
NotificationList.SortOrder = Enum.SortOrder.LayoutOrder
NotificationList.Padding = UDim.new(0, 6)
NotificationList.VerticalAlignment = Enum.VerticalAlignment.Bottom

local function Notify(title, text, duration)
    duration = duration or 2.5
    local Toast = Instance.new("Frame")
    Toast.Size = UDim2.new(1, 0, 0, 46)
    Toast.BackgroundColor3 = CurrentTheme.Sidebar
    Toast.BackgroundTransparency = 0.2
    Toast.BorderSizePixel = 0
    Toast.ZIndex = 51
    Toast.Parent = NotificationFrame

    Instance.new("UICorner", Toast).CornerRadius = UDim.new(0, 8)
    local Stroke = Instance.new("UIStroke", Toast)
    Stroke.Color = CurrentTheme.Accent
    Stroke.Thickness = 1.5

    local TTitle = Instance.new("TextLabel")
    TTitle.Size = UDim2.new(1, -10, 0, 18)
    TTitle.Position = UDim2.new(0, 8, 0, 4)
    TTitle.Text = title
    TTitle.TextColor3 = CurrentTheme.Accent
    TTitle.Font = Enum.Font.SourceSansBold
    TTitle.TextSize = 13
    TTitle.TextXAlignment = Enum.TextXAlignment.Left
    TTitle.BackgroundTransparency = 1
    TTitle.ZIndex = 52
    TTitle.Parent = Toast

    local TText = Instance.new("TextLabel")
    TText.Size = UDim2.new(1, -10, 0, 18)
    TText.Position = UDim2.new(0, 8, 0, 22)
    TText.Text = text
    TText.TextColor3 = CurrentTheme.Text
    TText.Font = Enum.Font.SourceSans
    TText.TextSize = 12
    TText.TextXAlignment = Enum.TextXAlignment.Left
    TText.BackgroundTransparency = 1
    TText.ZIndex = 52
    TText.Parent = Toast

    task.delay(duration, function()
        local fadeOutInfo = TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
        TweenService:Create(Toast, fadeOutInfo, {BackgroundTransparency = 1}):Play()
        TweenService:Create(Stroke, fadeOutInfo, {Transparency = 1}):Play()
        TweenService:Create(TTitle, fadeOutInfo, {TextTransparency = 1}):Play()
        local lastTween = TweenService:Create(TText, fadeOutInfo, {TextTransparency = 1})
        lastTween:Play()
        lastTween.Completed:Connect(function() Toast:Destroy() end)
    end)
end

----------------------------------------------------------
-- 🛡️ ANTI-BAN MODULE
----------------------------------------------------------
local function InitAntiBanModule()
    if not Config.AntiBanEnabled then return end
    if hookmetamethod then
        local oldNamecall
        oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
            local method = getnamecallmethod()
            if Config.AntiKick and (method:lower() == "kick") and self == LocalPlayer then
                return nil
            end
            if Config.AntiLog and (method == "FireServer" or method == "InvokeServer") then
                local remoteName = tostring(self.Name):lower()
                if remoteName:find("ban") or remoteName:find("flag") or remoteName:find("cheat") or remoteName:find("detect") or remoteName:find("log") then
                    return nil
                end
            end
            return oldNamecall(self, ...)
        end)
    end
end
InitAntiBanModule()

----------------------------------------------------------
-- MAIN FRAME & INTERFACE
----------------------------------------------------------
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 540, 0, 350)
MainFrame.Position = UDim2.new(0.5, -270, 0.5, -175)
MainFrame.BackgroundColor3 = CurrentTheme.Bg
MainFrame.BackgroundTransparency = Config.FrameTransparency
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)
local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = CurrentTheme.Accent
MainStroke.Thickness = 2

local MainBgImage = Instance.new("ImageLabel")
MainBgImage.Name = "MainBgAnimeGirl"
MainBgImage.Size = UDim2.new(1, 0, 1, 0)
MainBgImage.BackgroundTransparency = 1
MainBgImage.ImageTransparency = Config.BgTransparency
MainBgImage.ScaleType = Enum.ScaleType.Crop
MainBgImage.Image = CuteAnimePresets[Config.CurrentAnimeIndex]
MainBgImage.ZIndex = 1
MainBgImage.Parent = MainFrame

-- HEADER BAR
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 38)
Header.BackgroundColor3 = CurrentTheme.Sidebar
Header.BackgroundTransparency = 0.2
Header.BorderSizePixel = 0
Header.ZIndex = 3
Header.Parent = MainFrame
Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 10)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0, 300, 1, 0)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.Text = "🚀 Kianbest Hub v10.5 ULTRA VIP"
Title.TextColor3 = CurrentTheme.Accent
Title.TextSize = 14
Title.Font = Enum.Font.FredokaOne
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1
Title.ZIndex = 4
Title.Parent = Header

local StatsLabel = Instance.new("TextLabel")
StatsLabel.Size = UDim2.new(0, 200, 1, 0)
StatsLabel.Position = UDim2.new(1, -240, 0, 0)
StatsLabel.Text = "FPS: -- | Ping: --ms"
StatsLabel.TextColor3 = Color3.fromRGB(230, 210, 230)
StatsLabel.TextSize = 12
StatsLabel.Font = Enum.Font.SourceSansBold
StatsLabel.TextXAlignment = Enum.TextXAlignment.Right
StatsLabel.BackgroundTransparency = 1
StatsLabel.ZIndex = 4
StatsLabel.Parent = Header

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 26, 0, 26)
CloseBtn.Position = UDim2.new(1, -32, 0, 6)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 100, 120)
CloseBtn.TextSize = 13
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 22, 32)
CloseBtn.BorderSizePixel = 0
CloseBtn.ZIndex = 4
CloseBtn.Parent = Header
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

-- TOGGLE BUTTON
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Name = "KianToggleButton"
ToggleBtn.Size = UDim2.new(0, 50, 0, 50)
ToggleBtn.Position = UDim2.new(0, 15, 0.4, 0)
ToggleBtn.BackgroundColor3 = CurrentTheme.Sidebar
ToggleBtn.Text = "KIAN"
ToggleBtn.TextColor3 = CurrentTheme.Accent
ToggleBtn.Font = Enum.Font.FredokaOne
ToggleBtn.TextSize = 14
ToggleBtn.Active = true
ToggleBtn.Draggable = true
ToggleBtn.ZIndex = 100
ToggleBtn.Parent = ScreenGui
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)

local ButtonAspect = Instance.new("UIAspectRatioConstraint", ToggleBtn)
ButtonAspect.AspectRatio = 1
local ButtonStroke = Instance.new("UIStroke", ToggleBtn)
ButtonStroke.Color = CurrentTheme.Accent
ButtonStroke.Thickness = 2.5

local isMenuOpen = true
local function ToggleMenu()
    isMenuOpen = not isMenuOpen
    if isMenuOpen then
        MainFrame.Visible = true
        TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 540, 0, 350)}):Play()
    else
        local tween = TweenService:Create(MainFrame, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0)})
        tween:Play()
        tween.Completed:Connect(function()
            if not isMenuOpen then MainFrame.Visible = false end
        end)
    end
end

ToggleBtn.MouseButton1Click:Connect(ToggleMenu)
CloseBtn.MouseButton1Click:Connect(ToggleMenu)
UserInputService.InputBegan:Connect(function(input, gp)
    if not gp and input.KeyCode == Config.ToggleKey then ToggleMenu() end
end)

----------------------------------------------------------
-- SIDEBAR & TABS
----------------------------------------------------------
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 130, 1, -38)
Sidebar.Position = UDim2.new(0, 0, 0, 38)
Sidebar.BackgroundColor3 = CurrentTheme.Sidebar
Sidebar.BackgroundTransparency = 0.3
Sidebar.BorderSizePixel = 0
Sidebar.ZIndex = 3
Sidebar.Parent = MainFrame

local ContentContainer = Instance.new("Frame")
ContentContainer.Size = UDim2.new(1, -140, 1, -48)
ContentContainer.Position = UDim2.new(0, 135, 0, 43)
ContentContainer.BackgroundTransparency = 1
ContentContainer.ZIndex = 3
ContentContainer.Parent = MainFrame

local Pages = {}
local TabButtons = {}

local function CreateTab(name, icon, posIndex)
    local TabBtn = Instance.new("TextButton")
    TabBtn.Size = UDim2.new(1, -12, 0, 32)
    TabBtn.Position = UDim2.new(0, 6, 0, 8 + (posIndex - 1) * 38)
    TabBtn.Text = icon .. "  " .. name
    TabBtn.TextColor3 = CurrentTheme.Text
    TabBtn.Font = Enum.Font.SourceSansBold
    TabBtn.TextSize = 13
    TabBtn.TextXAlignment = Enum.TextXAlignment.Left
    TabBtn.BackgroundColor3 = CurrentTheme.Button
    TabBtn.BorderSizePixel = 0
    TabBtn.ZIndex = 4
    TabBtn.Parent = Sidebar
    Instance.new("UICorner", TabBtn).CornerRadius = UDim.new(0, 6)

    local Page = Instance.new("ScrollingFrame")
    Page.Size = UDim2.new(1, 0, 1, 0)
    Page.BackgroundTransparency = 1
    Page.ScrollBarThickness = 3
    Page.Visible = (posIndex == 1)
    Page.ZIndex = 3
    Page.Parent = ContentContainer

    Pages[name] = {Button = TabBtn, Page = Page}
    table.insert(TabButtons, TabBtn)

    TabBtn.MouseButton1Click:Connect(function()
        for _, tab in pairs(Pages) do
            tab.Page.Visible = false
            tab.Button.BackgroundColor3 = CurrentTheme.Button
        end
        Page.Visible = true
        TabBtn.BackgroundColor3 = CurrentTheme.Accent
    end)

    if posIndex == 1 then TabBtn.BackgroundColor3 = CurrentTheme.Accent end
    return Page
end

local CombatPage = CreateTab("Combat VIP", "⚔️", 1)
local MovementPage = CreateTab("Movement", "⚡", 2)
local FixLagPage = CreateTab("Fix Lag VIP", "🚀", 3)
local VisualsPage = CreateTab("Visuals", "👁", 4)
local SettingsPage = CreateTab("Settings", "⚙️", 5)

----------------------------------------------------------
-- UI BUILDERS
----------------------------------------------------------
local function CreateToggle(parent, text, defaultState, callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, -6, 0, 34)
    Btn.Text = "  " .. text
    Btn.TextColor3 = CurrentTheme.Text
    Btn.Font = Enum.Font.SourceSansBold
    Btn.TextSize = 13
    Btn.TextXAlignment = Enum.TextXAlignment.Left
    Btn.BackgroundColor3 = CurrentTheme.Button
    Btn.BorderSizePixel = 0
    Btn.ZIndex = 4
    Btn.Parent = parent
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 6)

    local StatusInd = Instance.new("Frame")
    StatusInd.Size = UDim2.new(0, 28, 0, 15)
    StatusInd.Position = UDim2.new(1, -36, 0.5, -7.5)
    StatusInd.BackgroundColor3 = defaultState and CurrentTheme.Accent or Color3.fromRGB(60, 65, 75)
    StatusInd.BorderSizePixel = 0
    StatusInd.ZIndex = 5
    StatusInd.Parent = Btn
    Instance.new("UICorner", StatusInd).CornerRadius = UDim.new(1, 0)

    local state = defaultState
    Btn.MouseButton1Click:Connect(function()
        state = not state
        StatusInd.BackgroundColor3 = state and CurrentTheme.Accent or Color3.fromRGB(60, 65, 75)
        callback(state)
    end)
    return Btn
end

local function CreateValueAdjuster(parent, title, minVal, maxVal, defaultVal, step, callback)
    local Container = Instance.new("Frame")
    Container.Size = UDim2.new(1, -6, 0, 36)
    Container.BackgroundColor3 = CurrentTheme.Button
    Container.BorderSizePixel = 0
    Container.ZIndex = 4
    Container.Parent = parent
    Instance.new("UICorner", Container).CornerRadius = UDim.new(0, 6)

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.55, 0, 1, 0)
    Label.Position = UDim2.new(0, 10, 0, 0)
    Label.Text = title .. ": " .. tostring(defaultVal)
    Label.TextColor3 = CurrentTheme.Text
    Label.Font = Enum.Font.SourceSansBold
    Label.TextSize = 13
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.BackgroundTransparency = 1
    Label.ZIndex = 5
    Label.Parent = Container

    local current = defaultVal
    local MinusBtn = Instance.new("TextButton")
    MinusBtn.Size = UDim2.new(0, 26, 0, 22)
    MinusBtn.Position = UDim2.new(1, -62, 0.5, -11)
    MinusBtn.Text = "-"
    MinusBtn.TextColor3 = Color3.fromRGB(240, 240, 240)
    MinusBtn.Font = Enum.Font.SourceSansBold
    MinusBtn.TextSize = 15
    MinusBtn.BackgroundColor3 = Color3.fromRGB(50, 35, 45)
    MinusBtn.BorderSizePixel = 0
    MinusBtn.ZIndex = 5
    MinusBtn.Parent = Container
    Instance.new("UICorner", MinusBtn).CornerRadius = UDim.new(0, 4)

    local PlusBtn = Instance.new("TextButton")
    PlusBtn.Size = UDim2.new(0, 26, 0, 22)
    PlusBtn.Position = UDim2.new(1, -32, 0.5, -11)
    PlusBtn.Text = "+"
    PlusBtn.TextColor3 = Color3.fromRGB(240, 240, 240)
    PlusBtn.Font = Enum.Font.SourceSansBold
    PlusBtn.TextSize = 15
    PlusBtn.BackgroundColor3 = CurrentTheme.Accent
    PlusBtn.BorderSizePixel = 0
    PlusBtn.ZIndex = 5
    PlusBtn.Parent = Container
    Instance.new("UICorner", PlusBtn).CornerRadius = UDim.new(0, 4)

    MinusBtn.MouseButton1Click:Connect(function()
        current = math.max(minVal, current - step)
        Label.Text = title .. ": " .. tostring(current)
        callback(current)
    end)
    PlusBtn.MouseButton1Click:Connect(function()
        current = math.min(maxVal, current + step)
        Label.Text = title .. ": " .. tostring(current)
        callback(current)
    end)
end

local function CreateButton(parent, text, bgColor, callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, -6, 0, 34)
    Btn.Text = text
    Btn.TextColor3 = Color3.fromRGB(245, 245, 245)
    Btn.Font = Enum.Font.SourceSansBold
    Btn.TextSize = 13
    Btn.BackgroundColor3 = bgColor or CurrentTheme.Button
    Btn.BorderSizePixel = 0
    Btn.ZIndex = 4
    Btn.Parent = parent
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 6)
    Btn.MouseButton1Click:Connect(callback)
    return Btn
end

----------------------------------------------------------
-- TAB 1: COMBAT VIP
----------------------------------------------------------
local CList = Instance.new("UIListLayout", CombatPage)
CList.SortOrder = Enum.SortOrder.LayoutOrder
CList.Padding = UDim.new(0, 8)

local TargetLabel = Instance.new("TextLabel")
TargetLabel.Size = UDim2.new(1, -6, 0, 22)
TargetLabel.Text = "Mục tiêu: Chưa chọn"
TargetLabel.TextColor3 = CurrentTheme.Accent
TargetLabel.Font = Enum.Font.SourceSansBold
TargetLabel.TextSize = 13
TargetLabel.BackgroundTransparency = 1
TargetLabel.ZIndex = 4
TargetLabel.Parent = CombatPage

CreateButton(CombatPage, "🎯 Chọn Mục Tiêu (Đổi Player)", Color3.fromRGB(45, 25, 40), function()
    local plrs = Players:GetPlayers()
    local currentIndex = 1
    for i, p in ipairs(plrs) do
        if p == Config.SelectedTarget then currentIndex = i break end
    end
    local nextPlr = plrs[(currentIndex % #plrs) + 1]
    if nextPlr == LocalPlayer then nextPlr = plrs[((currentIndex + 1) % #plrs) + 1] end
    Config.SelectedTarget = nextPlr
    if Config.SelectedTarget then
        TargetLabel.Text = "Mục tiêu: " .. Config.SelectedTarget.DisplayName
        Notify("Combat VIP", "Đã chọn: " .. Config.SelectedTarget.DisplayName)
    end
end)

CreateToggle(CombatPage, "⚡ M1 Tử Thần Multi-Hit (Auto Kill Instant)", Config.SuperM1Damage, function(state) Config.SuperM1Damage = state end)
CreateValueAdjuster(CombatPage, "Sức Mạnh Multi-Hit M1", 10, 100, Config.DamageMultiplier, 10, function(val) Config.DamageMultiplier = val end)
CreateToggle(CombatPage, "📦 Phóng To Hitbox Kẻ Địch", Config.HitboxExpander, function(state) Config.HitboxExpander = state end)
CreateValueAdjuster(CombatPage, "Kích Thước Hitbox", 5, 35, Config.HitboxSize, 5, function(val) Config.HitboxSize = val end)
CreateToggle(CombatPage, "⚡ Auto TP Áp Sát Lưng Đối Thủ", Config.AutoTPTarget, function(state) Config.AutoTPTarget = state end)
CreateToggle(CombatPage, "🥊 Auto Đấm Liên Hoàn M1", Config.AutoAttack, function(state) Config.AutoAttack = state end)
CreateToggle(CombatPage, "🔥 Auto Combo Skill (Z, X, C, V)", Config.AutoSkills, function(state) Config.AutoSkills = state end)

----------------------------------------------------------
-- TAB 2: MOVEMENT
----------------------------------------------------------
local MList = Instance.new("UIListLayout", MovementPage)
MList.SortOrder = Enum.SortOrder.LayoutOrder
MList.Padding = UDim.new(0, 8)

local bodyVel, bodyGyro
CreateToggle(MovementPage, "Bay 3D Chuẩn (Fly WASD)", Config.FlyEnabled, function(state)
    Config.FlyEnabled = state
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not state then
        if bodyVel then bodyVel:Destroy() bodyVel = nil end
        if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
    elseif hrp then
        bodyVel = Instance.new("BodyVelocity", hrp)
        bodyVel.MaxForce = Vector3.new(1e9, 1e9, 1e9)
        bodyVel.Velocity = Vector3.zero
        bodyGyro = Instance.new("BodyGyro", hrp)
        bodyGyro.MaxTorque = Vector3.new(1e9, 1e9, 1e9)
        bodyGyro.CFrame = hrp.CFrame
    end
end)
CreateValueAdjuster(MovementPage, "Tốc Độ Bay", 20, 250, Config.FlySpeed, 10, function(val) Config.FlySpeed = val end)
CreateToggle(MovementPage, "Đi Xuyên Tường Smooth (Fix Đứng Yên)", Config.NoclipEnabled, function(state) Config.NoclipEnabled = state end)
CreateToggle(MovementPage, "Chạy Nhanh (Speed Hack)", Config.SpeedEnabled, function(state) Config.SpeedEnabled = state end)
CreateValueAdjuster(MovementPage, "Tốc Độ Chạy", 16, 200, Config.SpeedValue, 10, function(val) Config.SpeedValue = val end)

----------------------------------------------------------
-- TAB 3: FIX LAG VIP (BỔ SUNG NHIỀU CHẾ ĐỘ CHUYÊN SÂU)
----------------------------------------------------------
local FList = Instance.new("UIListLayout", FixLagPage)
FList.SortOrder = Enum.SortOrder.LayoutOrder
FList.Padding = UDim.new(0, 8)

CreateButton(FixLagPage, "🚀 Kích Hoạt Siêu Tối Ưu Toàn Bộ Map", Color3.fromRGB(35, 55, 35), function()
    for _, v in ipairs(Lighting:GetChildren()) do
        if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("PostEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") then v:Destroy() end
    end
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 9e9
    Lighting.Brightness = 2
    
    for _, v in ipairs(Workspace:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Material = Enum.Material.SmoothPlastic
            v.CastShadow = false
            v.Reflectance = 0
        elseif v:IsA("Decal") or v:IsA("Texture") then
            v.Transparency = 1
        elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Fire") or v:IsA("Smoke") or v:IsA("Sparkles") then
            v.Enabled = false
        end
    end
    Notify("Fix Lag VIP", "Đã dọn sạch hiệu ứng, bật max FPS thành công! ⚡")
end)

CreateToggle(FixLagPage, "🥔 Chế Độ Potato Graphics (Đồ Họa Thấp)", Config.PotatoMode, function(state)
    Config.PotatoMode = state
    settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    for _, v in ipairs(Workspace:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Material = state and Enum.Material.SmoothPlastic or Enum.Material.Plastic
            v.CastShadow = not state
        end
    end
    Notify("Fix Lag VIP", state and "Đã Bật chế độ Potato!" | "Đã tắt Potato")
end)

CreateToggle(FixLagPage, "☀️ Full Bright (Sáng Trưng, Xóa Bóng)", Config.FullBright, function(state)
    Config.FullBright = state
    Lighting.GlobalShadows = not state
    Lighting.Brightness = state and 3 or 1
    Notify("Fix Lag", state and "Đã bật Full Bright!" or "Đã tắt Full Bright")
end)

CreateToggle(FixLagPage, "🧱 Chuyển Map Sang Nhựa Trơn (Low-Poly)", Config.LowPoly, function(state)
    Config.LowPoly = state
    for _, v in ipairs(Workspace:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Material = state and Enum.Material.SmoothPlastic or Enum.Material.Plastic
        end
    end
    Notify("Fix Lag", state and "Đã Low-Poly map!" or "Khôi phục vật liệu")
end)

CreateToggle(FixLagPage, "🚫 Tự Động Xóa Hạt Hiệu Ứng (Particles)", Config.RemoveParticles, function(state)
    Config.RemoveParticles = state
    for _, v in ipairs(Workspace:GetDescendants()) do
        if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then
            v.Enabled = not state
        end
    end
    Notify("Fix Lag", state and "Đã tắt tất cả Particles!" or "Đã bật lại Particles")
end)

CreateToggle(FixLagPage, "🧹 Tự Động Dọn Dẹp RAM (Memory Cleaner)", Config.AutoMemoryClean, function(state)
    Config.AutoMemoryClean = state
    Notify("Fix Lag", state and "Đã bật Auto Dọn RAM ngầm!" or "Đã tắt Auto Dọn RAM")
end)

CreateToggle(FixLagPage, "👤 Ẩn Người Chơi Khác (Hide Players)", Config.HidePlayers, function(state)
    Config.HidePlayers = state
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character then
            for _, part in ipairs(p.Character:GetDescendants()) do
                if part:IsA("BasePart") or part:IsA("Decal") then
                    part.Transparency = state and 1 or 0
                end
            end
        end
    end
end)

----------------------------------------------------------
-- TAB 4: VISUALS
----------------------------------------------------------
local VList = Instance.new("UIListLayout", VisualsPage)
VList.SortOrder = Enum.SortOrder.LayoutOrder
VList.Padding = UDim.new(0, 8)

CreateToggle(VisualsPage, "ESP Highlight (Xuyên Tường)", false, function(state) end)
CreateToggle(VisualsPage, "ESP Name (Hiện Tên)", false, function(state) end)
CreateToggle(VisualsPage, "ESP Health Bar (Thanh Máu)", false, function(state) end)

----------------------------------------------------------
-- TAB 5: SETTINGS
----------------------------------------------------------
local SList = Instance.new("UIListLayout", SettingsPage)
SList.SortOrder = Enum.SortOrder.LayoutOrder
SList.Padding = UDim.new(0, 8)

CreateToggle(SettingsPage, "🛡️️ Chống Kick (Anti-Kick)", Config.AntiKick, function(state) Config.AntiKick = state end)
CreateToggle(SettingsPage, "🛡️ Chống Gửi Log Hack (Anti-Report)", Config.AntiLog, function(state) Config.AntiLog = state end)

CreateButton(SettingsPage, "🌸 Đổi Nền Anime Female Cute (1 - 10)", Color3.fromRGB(55, 30, 50), function()
    Config.CurrentAnimeIndex = (Config.CurrentAnimeIndex % #CuteAnimePresets) + 1
    MainBgImage.Image = CuteAnimePresets[Config.CurrentAnimeIndex]
    Notify("Settings", "Đã chuyển sang Anime Nữ Cute #" .. Config.CurrentAnimeIndex .. " 💕")
end)

CreateButton(SettingsPage, "🎨 Đổi Theme Màu (Pink/Blue/Red/Purple)", Color3.fromRGB(40, 25, 45), function()
    if CurrentTheme == Themes.CutePink then CurrentTheme = Themes.CyberBlue
    elseif CurrentTheme == Themes.CyberBlue then CurrentTheme = Themes.BloodRed
    elseif CurrentTheme == Themes.BloodRed then CurrentTheme = Themes.DarkPurple
    else CurrentTheme = Themes.CutePink end

    MainFrame.BackgroundColor3 = CurrentTheme.Bg
    Header.BackgroundColor3 = CurrentTheme.Sidebar
    Sidebar.BackgroundColor3 = CurrentTheme.Sidebar
    Title.TextColor3 = CurrentTheme.Accent
    MainStroke.Color = CurrentTheme.Accent
    ToggleBtn.BackgroundColor3 = CurrentTheme.Sidebar
    ToggleBtn.TextColor3 = CurrentTheme.Accent
    ButtonStroke.Color = CurrentTheme.Accent
    TargetLabel.TextColor3 = CurrentTheme.Accent

    for _, btn in ipairs(TabButtons) do btn.BackgroundColor3 = CurrentTheme.Button end
    Notify("Settings", "Đã đổi theme màu thành công!")
end)

CreateToggle(SettingsPage, "Chống Treo Máy (Anti-AFK 24/7)", Config.AntiAFK, function(state) Config.AntiAFK = state end)
CreateButton(SettingsPage, "🔄 Vào Lại Server (Rejoin)", Color3.fromRGB(50, 30, 45), function()
    TeleportService:Teleport(game.PlaceId, LocalPlayer)
end)

----------------------------------------------------------
-- M1 MULTI-HIT PUNCH PROTOCOL
----------------------------------------------------------
local lastM1Tick = 0
local function ExecuteM1KillProtocol()
    if tick() - lastM1Tick < 0.03 then return end
    lastM1Tick = tick()
    local char = LocalPlayer.Character
    if not char then return end
    local cam = workspace.CurrentCamera
    local tool = char:FindFirstChildOfClass("Tool")
    local hits = Config.SuperM1Damage and Config.DamageMultiplier or 1

    task.spawn(function()
        for i = 1, hits do
            VirtualUser:Button1Down(Vector2.new(0,0), cam.CFrame)
            VirtualUser:Button1Up(Vector2.new(0,0), cam.CFrame)
            if tool then
                tool:Activate()
                for _, child in ipairs(tool:GetDescendants()) do
                    if child:IsA("RemoteEvent") or child:IsA("UnreliableRemoteEvent") then
                        local rName = child.Name:lower()
                        if rName:find("attack") or rName:find("hit") or rName:find("m1") or rName:find("swing") or rName:find("punch") or rName:find("damage") then
                            pcall(function() child:FireServer() end)
                        end
                    end
                end
            end
        end
    end)
end

----------------------------------------------------------
-- RENDER LOOP & MEMORY CLEANER
----------------------------------------------------------
local lastTime = tick()
local frameCount = 0
local skillTimer = 0
local memoryTimer = 0

-- NOCLIP LOOP
RunService.Stepped:Connect(function()
    if Config.NoclipEnabled and LocalPlayer.Character then
        for _, part in ipairs(LocalPlayer.Character:GetChildren()) do
            if part:IsA("BasePart") then part.CanCollide = false end
        end
    end
end)

RunService.RenderStepped:Connect(function(dt)
    frameCount = frameCount + 1
    if tick() - lastTime >= 1 then
        local currentFPS = math.floor(frameCount / (tick() - lastTime))
        local pingVal = 0
        pcall(function() pingVal = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue()) end)
        StatsLabel.Text = "FPS: " .. currentFPS .. " | Ping: " .. pingVal .. "ms"
        frameCount = 0
        lastTime = tick()
    end

    -- Auto Memory Cleaner Ngầm chống tràn RAM
    if Config.AutoMemoryClean then
        memoryTimer = memoryTimer + dt
        if memoryTimer >= 15 then
            memoryTimer = 0
            pcall(function()
                collectgarbage("collect")
                for _, v in ipairs(Workspace:GetDescendants()) do
                    if v:IsA("ParticleEmitter") or v:IsA("Trail") then v.Enabled = false end
                end
            end)
        end
    end

    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")

    -- HITBOX EXPANDER
    if Config.HitboxExpander then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character then
                local eHRP = p.Character:FindFirstChild("HumanoidRootPart")
                if eHRP then
                    eHRP.Size = Vector3.new(Config.HitboxSize, Config.HitboxSize, Config.HitboxSize)
                    eHRP.Transparency = 0.7
                    eHRP.Color = CurrentTheme.Accent
                    eHRP.CanCollide = false
                end
            end
        end
    end

    -- COMBAT TARGET
    if Config.SelectedTarget and Config.SelectedTarget.Character then
        local tHRP = Config.SelectedTarget.Character:FindFirstChild("HumanoidRootPart")
        local tHum = Config.SelectedTarget.Character:FindFirstChildOfClass("Humanoid")
        if tHRP and tHum and tHum.Health > 0 and hrp then
            if Config.AutoTPTarget then hrp.CFrame = tHRP.CFrame * CFrame.new(0, 0, 2.2) end
            if Config.AutoAttack or Config.SuperM1Damage then ExecuteM1KillProtocol() end
            if Config.AutoSkills then
                skillTimer = skillTimer + dt
                if skillTimer >= 0.2 then
                    skillTimer = 0
                    for _, key in ipairs({Enum.KeyCode.Z, Enum.KeyCode.X, Enum.KeyCode.C, Enum.KeyCode.V}) do
                        VirtualInputManager:SendKeyEvent(true, key, false, game)
                        task.wait(0.01)
                        VirtualInputManager:SendKeyEvent(false, key, false, game)
                    end
                end
            end
        end
    end

    if Config.SpeedEnabled and hum then hum.WalkSpeed = Config.SpeedValue end
end)

LocalPlayer.Idled:Connect(function()
    if Config.AntiAFK then
        VirtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(1)
        VirtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    end
end)

Notify("Kianbest Hub", "Đã cập nhật v10.5 ULTRA VIP với Tab Fix Lag Đa Chế Độ!")
