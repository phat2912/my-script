-- ==========================================================
-- SCRIPT MENU SYSTEM V6.0 MOVEMENT ULTIMATE (Kianbest Hub)
-- Features: 3D Vector Flying (WASD/Joystick) + AirWalk Pro + Mobile Fly Controls
-- ==========================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local Stats = game:GetService("Stats")
local TeleportService = game:GetService("TeleportService")
local VirtualUser = game:GetService("VirtualUser")
local LocalPlayer = Players.LocalPlayer

-- 1. Cấu hình & Trạng thái (Config)
local Config = {
    ESPEnabled = false,
    ESPNamesEnabled = false,
    ESPHealthEnabled = false,
    ESPColor = Color3.fromRGB(0, 200, 140),
    
    NoclipEnabled = false,
    SpeedEnabled = false,
    SpeedValue = 32,
    
    FlyEnabled = false,
    FlySpeed = 50,
    FlyUp = false,
    FlyDown = false,
    
    AirWalkEnabled = false,
    
    AntiAFK = false,
    CrosshairEnabled = false,
    FOVValue = 70,
    BgTransparency = 0.5,
    ToggleKey = Enum.KeyCode.RightControl
}

-- 2. Bộ chủ đề màu sắc dịu mắt (8 Muted Dark Themes)
local Themes = {
    Obsidian = { Bg = Color3.fromRGB(16, 17, 22), Sidebar = Color3.fromRGB(11, 12, 16), Accent = Color3.fromRGB(90, 155, 255), Button = Color3.fromRGB(24, 26, 34), Text = Color3.fromRGB(230, 230, 235) },
    Midnight = { Bg = Color3.fromRGB(14, 20, 28), Sidebar = Color3.fromRGB(9, 14, 20), Accent = Color3.fromRGB(75, 160, 225), Button = Color3.fromRGB(20, 30, 42), Text = Color3.fromRGB(230, 235, 240) },
    Violet = { Bg = Color3.fromRGB(20, 16, 26), Sidebar = Color3.fromRGB(14, 10, 18), Accent = Color3.fromRGB(150, 100, 220), Button = Color3.fromRGB(30, 24, 38), Text = Color3.fromRGB(235, 230, 240) },
    WineRed = { Bg = Color3.fromRGB(24, 16, 18), Sidebar = Color3.fromRGB(16, 10, 12), Accent = Color3.fromRGB(200, 80, 95), Button = Color3.fromRGB(36, 24, 26), Text = Color3.fromRGB(240, 230, 230) },
    SageEmerald = { Bg = Color3.fromRGB(16, 22, 19), Sidebar = Color3.fromRGB(10, 15, 12), Accent = Color3.fromRGB(75, 175, 125), Button = Color3.fromRGB(24, 34, 28), Text = Color3.fromRGB(230, 240, 235) },
    Amber = { Bg = Color3.fromRGB(24, 21, 16), Sidebar = Color3.fromRGB(16, 14, 10), Accent = Color3.fromRGB(210, 155, 70), Button = Color3.fromRGB(36, 31, 24), Text = Color3.fromRGB(240, 235, 225) },
    CharcoalPink = { Bg = Color3.fromRGB(24, 17, 22), Sidebar = Color3.fromRGB(16, 11, 15), Accent = Color3.fromRGB(200, 110, 160), Button = Color3.fromRGB(36, 25, 32), Text = Color3.fromRGB(240, 230, 235) },
    Slate = { Bg = Color3.fromRGB(18, 20, 24), Sidebar = Color3.fromRGB(12, 14, 17), Accent = Color3.fromRGB(120, 140, 165), Button = Color3.fromRGB(26, 30, 36), Text = Color3.fromRGB(235, 235, 240) }
}

local CurrentTheme = Themes.Obsidian

-- 3. Khởi tạo ScreenGui
local ParentGui = (gethui and gethui()) or CoreGui or LocalPlayer:WaitForChild("PlayerGui")
if ParentGui:FindFirstChild("KianbestMenuV6") then
    ParentGui.KianbestMenuV6:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KianbestMenuV6"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = ParentGui

-- 4. HỆ THỐNG THÔNG BÁO (TOAST NOTIFICATION)
local NotificationFrame = Instance.new("Frame")
NotificationFrame.Name = "NotificationFrame"
NotificationFrame.Size = UDim2.new(0, 220, 0, 200)
NotificationFrame.Position = UDim2.new(1, -230, 1, -210)
NotificationFrame.BackgroundTransparency = 1
NotificationFrame.ZIndex = 20
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
    Toast.BackgroundTransparency = 1
    Toast.BorderSizePixel = 0
    Toast.ZIndex = 20
    Toast.Parent = NotificationFrame

    Instance.new("UICorner", Toast).CornerRadius = UDim.new(0, 6)
    local Stroke = Instance.new("UIStroke", Toast)
    Stroke.Color = CurrentTheme.Accent
    Stroke.Thickness = 1.2
    Stroke.Transparency = 1

    local TTitle = Instance.new("TextLabel")
    TTitle.Size = UDim2.new(1, -10, 0, 18)
    TTitle.Position = UDim2.new(0, 8, 0, 4)
    TTitle.Text = title
    TTitle.TextColor3 = CurrentTheme.Accent
    TTitle.TextTransparency = 1
    TTitle.Font = Enum.Font.SourceSansBold
    TTitle.TextSize = 13
    TTitle.TextXAlignment = Enum.TextXAlignment.Left
    TTitle.BackgroundTransparency = 1
    TTitle.ZIndex = 21
    TTitle.Parent = Toast

    local TText = Instance.new("TextLabel")
    TText.Size = UDim2.new(1, -10, 0, 18)
    TText.Position = UDim2.new(0, 8, 0, 22)
    TText.Text = text
    TText.TextColor3 = CurrentTheme.Text
    TText.TextTransparency = 1
    TText.Font = Enum.Font.SourceSans
    TText.TextSize = 12
    TText.TextXAlignment = Enum.TextXAlignment.Left
    TText.BackgroundTransparency = 1
    TText.ZIndex = 21
    TText.Parent = Toast

    local fadeInInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    TweenService:Create(Toast, fadeInInfo, {BackgroundTransparency = 0.1}):Play()
    TweenService:Create(Stroke, fadeInInfo, {Transparency = 0}):Play()
    TweenService:Create(TTitle, fadeInInfo, {TextTransparency = 0}):Play()
    TweenService:Create(TText, fadeInInfo, {TextTransparency = 0}):Play()

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

-- 5. MAIN FRAME MENU
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 520, 0, 330)
MainFrame.Position = UDim2.new(0.5, -260, 0.5, -165)
MainFrame.BackgroundColor3 = CurrentTheme.Bg
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)

local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = CurrentTheme.Accent
MainStroke.Thickness = 1.5

-- HÌNH NỀN ANIME TRẮNG ĐEN
local MainBgImage = Instance.new("ImageLabel")
MainBgImage.Name = "MainBgAnime"
MainBgImage.Size = UDim2.new(1, 0, 1, 0)
MainBgImage.BackgroundTransparency = 1
MainBgImage.ImageTransparency = Config.BgTransparency
MainBgImage.ScaleType = Enum.ScaleType.Crop
MainBgImage.Image = "rbxassetid://6071575925"
MainBgImage.ZIndex = 1
MainBgImage.Parent = MainFrame

-- HEADER BAR
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 38)
Header.BackgroundColor3 = CurrentTheme.Sidebar
Header.BackgroundTransparency = 0.15
Header.BorderSizePixel = 0
Header.ZIndex = 3
Header.Parent = MainFrame

Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 10)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0, 220, 1, 0)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.Text = "Kianbest Hub v6.0 Movement"
Title.TextColor3 = CurrentTheme.Accent
Title.TextSize = 15
Title.Font = Enum.Font.FredokaOne
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1
Title.ZIndex = 4
Title.Parent = Header

local StatsLabel = Instance.new("TextLabel")
StatsLabel.Size = UDim2.new(0, 200, 1, 0)
StatsLabel.Position = UDim2.new(1, -240, 0, 0)
StatsLabel.Text = "FPS: -- | Ping: --ms"
StatsLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
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
CloseBtn.TextColor3 = Color3.fromRGB(220, 90, 90)
CloseBtn.TextSize = 13
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.BackgroundColor3 = Color3.fromRGB(28, 30, 38)
CloseBtn.BorderSizePixel = 0
CloseBtn.ZIndex = 4
CloseBtn.Parent = Header
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

-- NÚT TRÒN SẮC NÉT CHỮ "KIAN"
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Name = "KianToggleButton"
ToggleBtn.Size = UDim2.new(0, 50, 0, 50)
ToggleBtn.Position = UDim2.new(0, 15, 0.4, 0)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(18, 20, 26)
ToggleBtn.Text = "KIAN"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.Font = Enum.Font.FredokaOne
ToggleBtn.TextSize = 15
ToggleBtn.Active = true
ToggleBtn.Draggable = true
ToggleBtn.ZIndex = 100
ToggleBtn.Parent = ScreenGui

Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)

local ButtonStroke = Instance.new("UIStroke", ToggleBtn)
ButtonStroke.Color = CurrentTheme.Accent
ButtonStroke.Thickness = 2.5
ButtonStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

local isMenuOpen = true
local function ToggleMenu()
    isMenuOpen = not isMenuOpen
    if isMenuOpen then
        MainFrame.Visible = true
        TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 520, 0, 330)}):Play()
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

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Config.ToggleKey then ToggleMenu() end
end)

----------------------------------------------------------
-- NÚT ĐIỀU KHIỂN BAY CHO MOBILE (FLY UP / DOWN BUTTONS)
----------------------------------------------------------
local FlyControlsFrame = Instance.new("Frame")
FlyControlsFrame.Name = "FlyControls"
FlyControlsFrame.Size = UDim2.new(0, 50, 0, 110)
FlyControlsFrame.Position = UDim2.new(1, -65, 0.5, -55)
FlyControlsFrame.BackgroundTransparency = 1
FlyControlsFrame.Visible = false
FlyControlsFrame.ZIndex = 90
FlyControlsFrame.Parent = ScreenGui

local FlyUpBtn = Instance.new("TextButton")
FlyUpBtn.Size = UDim2.new(0, 48, 0, 48)
FlyUpBtn.Position = UDim2.new(0, 0, 0, 0)
FlyUpBtn.Text = "▲"
FlyUpBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
FlyUpBtn.Font = Enum.Font.SourceSansBold
FlyUpBtn.TextSize = 20
FlyUpBtn.BackgroundColor3 = Color3.fromRGB(25, 30, 40)
FlyUpBtn.BorderSizePixel = 0
FlyUpBtn.ZIndex = 91
FlyUpBtn.Parent = FlyControlsFrame
Instance.new("UICorner", FlyUpBtn).CornerRadius = UDim.new(0, 10)
local UpStroke = Instance.new("UIStroke", FlyUpBtn)
UpStroke.Color = CurrentTheme.Accent
UpStroke.Thickness = 1.5

local FlyDownBtn = Instance.new("TextButton")
FlyDownBtn.Size = UDim2.new(0, 48, 0, 48)
FlyDownBtn.Position = UDim2.new(0, 0, 0, 58)
FlyDownBtn.Text = "▼"
FlyDownBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
FlyDownBtn.Font = Enum.Font.SourceSansBold
FlyDownBtn.TextSize = 20
FlyDownBtn.BackgroundColor3 = Color3.fromRGB(25, 30, 40)
FlyDownBtn.BorderSizePixel = 0
FlyDownBtn.ZIndex = 91
FlyDownBtn.Parent = FlyControlsFrame
Instance.new("UICorner", FlyDownBtn).CornerRadius = UDim.new(0, 10)
local DownStroke = Instance.new("UIStroke", FlyDownBtn)
DownStroke.Color = CurrentTheme.Accent
DownStroke.Thickness = 1.5

FlyUpBtn.MouseButton1Down:Connect(function() Config.FlyUp = true end)
FlyUpBtn.MouseButton1Up:Connect(function() Config.FlyUp = false end)
FlyDownBtn.MouseButton1Down:Connect(function() Config.FlyDown = true end)
FlyDownBtn.MouseButton1Up:Connect(function() Config.FlyDown = false end)

----------------------------------------------------------
-- SIDEBAR & TAB SYSTEM
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

local VisualsPage = CreateTab("Visuals", "👁️", 1)
local MovementPage = CreateTab("Movement", "⚡", 2)
local FixLagPage = CreateTab("Fix Lag", "🚀", 3)
local SettingsPage = CreateTab("Settings", "⚙️", 4)

----------------------------------------------------------
-- UI UTILS
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
    StatusInd.BackgroundColor3 = defaultState and Color3.fromRGB(0, 180, 100) or Color3.fromRGB(60, 65, 75)
    StatusInd.BorderSizePixel = 0
    StatusInd.ZIndex = 5
    StatusInd.Parent = Btn

    Instance.new("UICorner", StatusInd).CornerRadius = UDim.new(1, 0)

    local state = defaultState
    Btn.MouseButton1Click:Connect(function()
        state = not state
        StatusInd.BackgroundColor3 = state and Color3.fromRGB(0, 180, 100) or Color3.fromRGB(60, 65, 75)
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
    Label.Size = UDim2.new(0.5, 0, 1, 0)
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
    MinusBtn.BackgroundColor3 = Color3.fromRGB(45, 50, 60)
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
-- TAB 1: VISUALS
----------------------------------------------------------
local VList = Instance.new("UIListLayout", VisualsPage)
VList.SortOrder = Enum.SortOrder.LayoutOrder
VList.Padding = UDim.new(0, 8)

CreateToggle(VisualsPage, "ESP Highlight (Nhìn Xuyên Tường)", Config.ESPEnabled, function(state)
    Config.ESPEnabled = state
    Notify("Visuals", state and "Đã BẬT ESP Highlight" or "Đã TẮT ESP Highlight")
end)

CreateToggle(VisualsPage, "ESP Name (Tên Trên Đầu Trực Quan)", Config.ESPNamesEnabled, function(state)
    Config.ESPNamesEnabled = state
    Notify("Visuals", state and "Đã BẬT Tên người chơi" or "Đã TẮT Tên")
end)

CreateToggle(VisualsPage, "ESP Health Bar (Thanh Máu Địch)", Config.ESPHealthEnabled, function(state)
    Config.ESPHealthEnabled = state
    Notify("Visuals", state and "Đã BẬT Thanh Máu" or "Đã TẮT Thanh Máu")
end)

----------------------------------------------------------
-- TAB 2: MOVEMENT (NÂNG CẤP V6.0 MOVEMENT ULTIMATE)
----------------------------------------------------------
local MList = Instance.new("UIListLayout", MovementPage)
MList.SortOrder = Enum.SortOrder.LayoutOrder
MList.Padding = UDim.new(0, 8)

local bodyVel, bodyGyro

CreateToggle(MovementPage, "Bay 3D Chuẩn (Fly 3D WASD/Joystick)", Config.FlyEnabled, function(state)
    Config.FlyEnabled = state
    FlyControlsFrame.Visible = state
    Notify("Movement", state and "Đã BẬT Bay 3D! Dùng WASD/Cần Gạt để di chuyển 360°" or "Đã TẮT Bay 3D!")

    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")

    if not state then
        if bodyVel then bodyVel:Destroy() bodyVel = nil end
        if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
    elseif hrp then
        bodyVel = Instance.new("BodyVelocity", hrp)
        bodyVel.MaxForce = Vector3.new(1e9, 1e9, 1e9)
        bodyVel.Velocity = Vector3.zero

        bodyGyro = Instance.new("BodyGyro", hrp)
        bodyGyro.MaxTorque = Vector3.new(1e9, 1e9, 1e9)
        bodyGyro.P = 10000
        bodyGyro.CFrame = hrp.CFrame
    end
end)

CreateValueAdjuster(MovementPage, "Tốc Độ Bay", 20, 300, Config.FlySpeed, 10, function(val)
    Config.FlySpeed = val
end)

CreateToggle(MovementPage, "Đi Trên Không 2.0 (Air Walk Pro)", Config.AirWalkEnabled, function(state)
    Config.AirWalkEnabled = state
    Notify("Movement", state and "Đã BẬT Air Walk (Đi trên không mượt mà)" or "Đã TẮT Air Walk")
end)

CreateToggle(MovementPage, "Đi Xuyên Tường (Noclip)", Config.NoclipEnabled, function(state)
    Config.NoclipEnabled = state
    Notify("Movement", state and "Đã BẬT Noclip!" or "Đã TẮT Noclip!")
end)

CreateToggle(MovementPage, "Chạy Nhanh (Speed Hack)", Config.SpeedEnabled, function(state)
    Config.SpeedEnabled = state
    Notify("Movement", state and "Đã BẬT Speed Hack" or "Đã TẮT Speed Hack")
    if not Config.SpeedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 16
    end
end)

CreateValueAdjuster(MovementPage, "Tốc Độ Chạy", 16, 250, Config.SpeedValue, 10, function(val)
    Config.SpeedValue = val
end)

----------------------------------------------------------
-- TAB 3: FIX LAG
----------------------------------------------------------
local FList = Instance.new("UIListLayout", FixLagPage)
FList.SortOrder = Enum.SortOrder.LayoutOrder
FList.Padding = UDim.new(0, 8)

CreateButton(FixLagPage, "🚀 Super FPS Boost (1-Click Tối Ưu Nhanh)", Color3.fromRGB(35, 110, 160), function()
    for _, v in ipairs(Lighting:GetChildren()) do
        if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("PostEffect") then v:Destroy() end
    end
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 9e9
    
    for _, v in ipairs(game:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Material = Enum.Material.SmoothPlastic
            v.CastShadow = false
        elseif v:IsA("Decal") or v:IsA("Texture") then
            v.Transparency = 1
        elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then
            v.Enabled = false
        end
    end
    Notify("Fix Lag", "Đã tối ưu mượt map thành công!")
end)

CreateToggle(FixLagPage, "Xóa Textures Map (Plastic Smooth)", false, function(state)
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Material = state and Enum.Material.SmoothPlastic or Enum.Material.Plastic
        end
    end
    Notify("Fix Lag", state and "Đã bật Smooth Plastic" or "Đã khôi phục Texture")
end)

CreateToggle(FixLagPage, "Đổi Bầu Trời Tối Dịu Mắt (Dark Sky)", false, function(state)
    if state then
        for _, v in ipairs(Lighting:GetChildren()) do
            if v:IsA("Sky") or v:IsA("Atmosphere") then v:Destroy() end
        end
        Lighting.OutdoorAmbient = Color3.fromRGB(25, 25, 28)
        Lighting.Ambient = Color3.fromRGB(25, 25, 28)
    else
        Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
    end
    Notify("Fix Lag", state and "Đã bật Bầu Trời Tối" or "Đã tắt Bầu Trời Tối")
end)

CreateToggle(FixLagPage, "Tắt Bóng Đổ & Sương Mù (No Fog/Shadows)", true, function(state)
    Lighting.GlobalShadows = not state
    Lighting.FogEnd = state and 9e9 or 1000
    Notify("Fix Lag", state and "Đã tắt Bóng & Sương Mù" or "Đã bật Bóng Đổ")
end)

----------------------------------------------------------
-- TAB 4: SETTINGS
----------------------------------------------------------
local SList = Instance.new("UIListLayout", SettingsPage)
SList.SortOrder = Enum.SortOrder.LayoutOrder
SList.Padding = UDim.new(0, 8)

CreateToggle(SettingsPage, "Chống Treo Máy (Anti-AFK 24/7)", Config.AntiAFK, function(state)
    Config.AntiAFK = state
    Notify("Settings", state and "Đã BẬT Anti-AFK" or "Đã TẮT Anti-AFK")
end)

local CrosshairFrame = Instance.new("Frame")
CrosshairFrame.Name = "CustomCrosshair"
CrosshairFrame.Size = UDim2.new(0, 8, 0, 8)
CrosshairFrame.Position = UDim2.new(0.5, -4, 0.5, -4)
CrosshairFrame.BackgroundColor3 = Color3.fromRGB(0, 255, 170)
CrosshairFrame.BorderSizePixel = 0
CrosshairFrame.Visible = false
CrosshairFrame.ZIndex = 100
CrosshairFrame.Parent = ScreenGui
Instance.new("UICorner", CrosshairFrame).CornerRadius = UDim.new(1, 0)

CreateToggle(SettingsPage, "Bật Tâm Bắn Màn Hình (Crosshair)", Config.CrosshairEnabled, function(state)
    Config.CrosshairEnabled = state
    CrosshairFrame.Visible = state
    Notify("Settings", state and "Đã hiện Tâm Bắn" or "Đã ẩn Tâm Bắn")
end)

CreateValueAdjuster(SettingsPage, "Tầm Nhìn Camera (FOV)", 70, 120, Config.FOVValue, 5, function(val)
    Config.FOVValue = val
    workspace.CurrentCamera.FieldOfView = val
end)

CreateValueAdjuster(SettingsPage, "Độ Mờ Ảnh Nền Anime", 0, 10, 5, 1, function(val)
    MainBgImage.ImageTransparency = val / 10
end)

CreateButton(SettingsPage, "🔄 Vào Lại Server (Rejoin)", Color3.fromRGB(45, 55, 70), function()
    Notify("Server", "Đang kết nối lại Server...")
    TeleportService:Teleport(game.PlaceId, LocalPlayer)
end)

CreateButton(SettingsPage, "🔀 Đổi Server Khác (Server Hop)", Color3.fromRGB(45, 55, 70), function()
    Notify("Server", "Đang tìm Server mới...")
    pcall(function()
        local req = game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/0?sortOrder=Asc&limit=100")
        local body = game:GetService("HttpService"):JSONDecode(req)
        if body and body.data then
            for _, v in ipairs(body.data) do
                if v.playing < v.maxPlayers and v.id ~= game.JobId then
                    TeleportService:TeleportToPlaceInstance(game.PlaceId, v.id, LocalPlayer)
                    break
                end
            end
        end
    end)
end)

local ThemeLabel = Instance.new("TextLabel")
ThemeLabel.Size = UDim2.new(1, 0, 0, 18)
ThemeLabel.Text = "Chọn Chủ Đề Màu Tối Dịu Mắt (8 Themes):"
ThemeLabel.TextColor3 = CurrentTheme.Text
ThemeLabel.Font = Enum.Font.SourceSansBold
ThemeLabel.TextSize = 12
ThemeLabel.TextXAlignment = Enum.TextXAlignment.Left
ThemeLabel.BackgroundTransparency = 1
ThemeLabel.ZIndex = 4
ThemeLabel.Parent = SettingsPage

local ThemeGrid = Instance.new("Frame")
ThemeGrid.Size = UDim2.new(1, -6, 0, 65)
ThemeGrid.BackgroundTransparency = 1
ThemeGrid.ZIndex = 4
ThemeGrid.Parent = SettingsPage

local themeList = {"Obsidian", "Midnight", "Violet", "WineRed", "SageEmerald", "Amber", "CharcoalPink", "Slate"}
for i, name in ipairs(themeList) do
    local row = math.floor((i - 1) / 4)
    local col = (i - 1) % 4

    local TBtn = Instance.new("TextButton")
    TBtn.Size = UDim2.new(0, 80, 0, 26)
    TBtn.Position = UDim2.new(0, col * 86, 0, row * 32)
    TBtn.Text = name
    TBtn.TextColor3 = Color3.fromRGB(240, 240, 240)
    TBtn.Font = Enum.Font.SourceSansBold
    TBtn.TextSize = 10
    TBtn.BackgroundColor3 = Themes[name].Accent
    TBtn.BorderSizePixel = 0
    TBtn.ZIndex = 5
    TBtn.Parent = ThemeGrid
    Instance.new("UICorner", TBtn).CornerRadius = UDim.new(0, 4)

    TBtn.MouseButton1Click:Connect(function()
        CurrentTheme = Themes[name]
        MainFrame.BackgroundColor3 = CurrentTheme.Bg
        Header.BackgroundColor3 = CurrentTheme.Sidebar
        Sidebar.BackgroundColor3 = CurrentTheme.Sidebar
        Title.TextColor3 = CurrentTheme.Accent
        MainStroke.Color = CurrentTheme.Accent
        ButtonStroke.Color = CurrentTheme.Accent
        UpStroke.Color = CurrentTheme.Accent
        DownStroke.Color = CurrentTheme.Accent
        CrosshairFrame.BackgroundColor3 = CurrentTheme.Accent
        Notify("Theme", "Đã áp dụng tông màu: " .. name)
    end)
end

----------------------------------------------------------
-- ANTI AFK SYSTEM
----------------------------------------------------------
LocalPlayer.Idled:Connect(function()
    if Config.AntiAFK then
        VirtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(1)
        VirtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    end
end)

----------------------------------------------------------
-- NAMETAG & VISUAL SYSTEM
----------------------------------------------------------
local function CreateNameTag(char, plr)
    if not char then return end
    local head = char:WaitForChild("Head", 5)
    if not head then return end

    local oldTag = head:FindFirstChild("ESPNameTag")
    if oldTag then oldTag:Destroy() end

    local bbg = Instance.new("BillboardGui")
    bbg.Name = "ESPNameTag"
    bbg.Adornee = head
    bbg.Size = UDim2.new(0, 180, 0, 40)
    bbg.StudsOffset = Vector3.new(0, 3, 0)
    bbg.AlwaysOnTop = true
    bbg.Enabled = false
    bbg.Parent = head

    local nameTxt = Instance.new("TextLabel")
    nameTxt.Name = "NameLabel"
    nameTxt.Size = UDim2.new(1, 0, 0, 18)
    nameTxt.Position = UDim2.new(0, 0, 0, 0)
    nameTxt.BackgroundTransparency = 1
    nameTxt.Text = plr.DisplayName .. " (@" .. plr.Name .. ")"
    nameTxt.TextColor3 = Color3.fromRGB(240, 240, 240)
    nameTxt.TextStrokeTransparency = 0.2
    nameTxt.Font = Enum.Font.SourceSansBold
    nameTxt.TextSize = 12
    nameTxt.Parent = bbg

    local distTxt = Instance.new("TextLabel")
    distTxt.Name = "DistLabel"
    distTxt.Size = UDim2.new(1, 0, 0, 14)
    distTxt.Position = UDim2.new(0, 0, 0, 16)
    distTxt.BackgroundTransparency = 1
    distTxt.Text = "0 Studs"
    distTxt.TextColor3 = Color3.fromRGB(120, 180, 240)
    distTxt.TextStrokeTransparency = 0.3
    distTxt.Font = Enum.Font.SourceSans
    distTxt.TextSize = 11
    distTxt.Parent = bbg

    local hpBg = Instance.new("Frame")
    hpBg.Name = "HPBackground"
    hpBg.Size = UDim2.new(0, 100, 0, 4)
    hpBg.Position = UDim2.new(0.5, -50, 0, 32)
    hpBg.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
    hpBg.BorderSizePixel = 0
    hpBg.Visible = false
    hpBg.Parent = bbg
    Instance.new("UICorner", hpBg).CornerRadius = UDim.new(1, 0)

    local hpBar = Instance.new("Frame")
    hpBar.Name = "HPBar"
    hpBar.Size = UDim2.new(1, 0, 1, 0)
    hpBar.BackgroundColor3 = Color3.fromRGB(0, 200, 120)
    hpBar.BorderSizePixel = 0
    hpBar.Parent = hpBg
    Instance.new("UICorner", hpBar).CornerRadius = UDim.new(1, 0)
end

local function SetupPlayerESP(plr)
    if plr == LocalPlayer then return end

    local function OnCharacter(char)
        if not char then return end

        local highlight = char:FindFirstChild("ESPHighlight") or Instance.new("Highlight")
        highlight.Name = "ESPHighlight"
        highlight.Adornee = char
        highlight.FillColor = Config.ESPColor
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.FillTransparency = 0.5
        highlight.Enabled = Config.ESPEnabled
        highlight.Parent = char

        CreateNameTag(char, plr)
    end

    if plr.Character then OnCharacter(plr.Character) end
    plr.CharacterAdded:Connect(OnCharacter)
end

for _, p in ipairs(Players:GetPlayers()) do SetupPlayerESP(p) end
Players.PlayerAdded:Connect(SetupPlayerESP)

----------------------------------------------------------
-- RENDER LOOP & ADVANCED MOVEMENT LOGIC
----------------------------------------------------------
local lastTime = tick()
local frameCount = 0
local airPlatform = nil

RunService.Stepped:Connect(function()
    if Config.NoclipEnabled and LocalPlayer.Character then
        for _, part in ipairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

RunService.RenderStepped:Connect(function(dt)
    -- FPS & Ping Counter
    frameCount = frameCount + 1
    if tick() - lastTime >= 1 then
        local currentFPS = math.floor(frameCount / (tick() - lastTime))
        local pingVal = 0
        pcall(function() pingVal = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue()) end)
        StatsLabel.Text = "FPS: " .. currentFPS .. " | Ping: " .. pingVal .. "ms"
        frameCount = 0
        lastTime = tick()
    end

    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local cam = workspace.CurrentCamera

    -- 1. SPEED HACK SMOOTH
    if Config.SpeedEnabled and hum then
        hum.WalkSpeed = Config.SpeedValue
    end

    -- 2. FLY 3D MULTI-DIRECTIONAL MATH (XỬ LÝ FLY 360 CỰC MƯỢT)
    if Config.FlyEnabled and hrp and hum and cam then
        if not bodyVel or not bodyVel.Parent then
            bodyVel = Instance.new("BodyVelocity", hrp)
            bodyVel.MaxForce = Vector3.new(1e9, 1e9, 1e9)
        end
        if not bodyGyro or not bodyGyro.Parent then
            bodyGyro = Instance.new("BodyGyro", hrp)
            bodyGyro.MaxTorque = Vector3.new(1e9, 1e9, 1e9)
            bodyGyro.P = 10000
        end

        bodyGyro.CFrame = cam.CFrame

        local moveDir = hum.MoveDirection
        local flyVel = Vector3.zero

        if moveDir.Magnitude > 0 then
            -- Tính toán hướng Vector 3D thực tế theo WASD / Joystick gạt
            local camCF = cam.CFrame
            local flatLook = Vector3.new(camCF.LookVector.X, 0, camCF.LookVector.Z).Unit
            local flatRight = Vector3.new(camCF.RightVector.X, 0, camCF.RightVector.Z).Unit

            local forwardDot = moveDir:Dot(flatLook)
            local rightDot = moveDir:Dot(flatRight)

            local flight3D = (camCF.LookVector * forwardDot) + (camCF.RightVector * rightDot)
            if flight3D.Magnitude > 0 then
                flight3D = flight3D.Unit
            end

            flyVel = flight3D * Config.FlySpeed
        end

        -- Thêm điều khiển Nâng / Hạ độ cao (Phím / Touch Mobile)
        local ySpeed = 0
        if Config.FlyUp or UserInputService:IsKeyDown(Enum.KeyCode.E) or UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            ySpeed = ySpeed + Config.FlySpeed
        end
        if Config.FlyDown or UserInputService:IsKeyDown(Enum.KeyCode.Q) or UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            ySpeed = ySpeed - Config.FlySpeed
        end

        bodyVel.Velocity = flyVel + Vector3.new(0, ySpeed, 0)
    end

    -- 3. AIR WALK PRO 2.0 (SÀN TRÊN KHÔNG TỰ ĐỘNG)
    if Config.AirWalkEnabled and hrp then
        if not airPlatform or not airPlatform.Parent then
            airPlatform = Instance.new("Part")
            airPlatform.Name = "AirWalkPlatformPro"
            airPlatform.Size = Vector3.new(7, 0.8, 7)
            airPlatform.Anchored = true
            airPlatform.Transparency = 0.5
            airPlatform.Material = Enum.Material.Neon
            airPlatform.Color = CurrentTheme.Accent
            airPlatform.Parent = workspace
        end
        
        -- Nếu bấm nhảy thì nhích sàn lên cao
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            airPlatform.CFrame = CFrame.new(hrp.Position.X, hrp.Position.Y - 2.8, hrp.Position.Z)
        else
            airPlatform.CFrame = CFrame.new(hrp.Position.X, airPlatform.Position.Y, hrp.Position.Z)
        end
    else
        if airPlatform then airPlatform:Destroy() airPlatform = nil end
    end

    -- 4. ESP & NAMETAGS
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character then
            local pChar = p.Character
            local pHead = pChar:FindFirstChild("Head")
            local pHum = pChar:FindFirstChildOfClass("Humanoid")
            local hl = pChar:FindFirstChild("ESPHighlight")

            if hl then hl.Enabled = Config.ESPEnabled end

            if pHead then
                local bbg = pHead:FindFirstChild("ESPNameTag")
                if bbg then
                    bbg.Enabled = Config.ESPNamesEnabled

                    if Config.ESPNamesEnabled then
                        local distLabel = bbg:FindFirstChild("DistLabel")
                        if distLabel and hrp then
                            local dist = math.floor((pHead.Position - hrp.Position).Magnitude)
                            distLabel.Text = tostring(dist) .. " Studs"
                        end

                        local hpBg = bbg:FindFirstChild("HPBackground")
                        if hpBg then
                            hpBg.Visible = Config.ESPHealthEnabled
                            if Config.ESPHealthEnabled and pHum then
                                local hpBar = hpBg:FindFirstChild("HPBar")
                                if hpBar then
                                    local ratio = math.clamp(pHum.Health / pHum.MaxHealth, 0, 1)
                                    hpBar.Size = UDim2.new(ratio, 0, 1, 0)
                                    hpBar.BackgroundColor3 = Color3.fromRGB(255 * (1 - ratio), 255 * ratio, 0)
                                end
                            end
                        end
                    end
                end
            end
        end
    end
end)

Notify("Kianbest Hub", "Đã nâng cấp v6.0 Movement Ultimate!")
