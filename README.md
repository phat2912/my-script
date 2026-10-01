-- ==========================================================
-- SCRIPT MENU SYSTEM V4.0 SUPREME (Kianbest)
-- Features: Kian Toggle Button + B&W Anime Background + Real Noclip + Multi Fix Lag Modes
-- ==========================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local Stats = game:GetService("Stats")
local LocalPlayer = Players.LocalPlayer

-- 1. Cấu hình & Trạng thái (Config)
local Config = {
    ESPEnabled = false,
    ESPNamesEnabled = false,
    ESPHealthEnabled = false,
    ESPColor = Color3.fromRGB(0, 255, 150),
    
    NoclipEnabled = false,
    SpeedEnabled = false,
    SpeedValue = 32,
    
    FlyEnabled = false,
    FlySpeed = 50,
    
    AirWalkEnabled = false,
    ToggleKey = Enum.KeyCode.RightControl
}

-- 2. Bộ chủ đề giao diện (8 Themes)
local Themes = {
    Dark = { Bg = Color3.fromRGB(20, 22, 28), Sidebar = Color3.fromRGB(15, 16, 20), Accent = Color3.fromRGB(255, 105, 180), Button = Color3.fromRGB(30, 34, 42), Text = Color3.fromRGB(240, 240, 240) },
    Cyan = { Bg = Color3.fromRGB(15, 25, 35), Sidebar = Color3.fromRGB(10, 18, 26), Accent = Color3.fromRGB(0, 210, 255), Button = Color3.fromRGB(22, 38, 52), Text = Color3.fromRGB(240, 240, 240) },
    Purple = { Bg = Color3.fromRGB(25, 18, 35), Sidebar = Color3.fromRGB(18, 12, 26), Accent = Color3.fromRGB(180, 70, 255), Button = Color3.fromRGB(38, 26, 52), Text = Color3.fromRGB(240, 240, 240) },
    Red = { Bg = Color3.fromRGB(30, 18, 18), Sidebar = Color3.fromRGB(22, 12, 12), Accent = Color3.fromRGB(255, 70, 70), Button = Color3.fromRGB(48, 26, 26), Text = Color3.fromRGB(240, 240, 240) },
    Emerald = { Bg = Color3.fromRGB(18, 30, 22), Sidebar = Color3.fromRGB(12, 22, 15), Accent = Color3.fromRGB(50, 220, 120), Button = Color3.fromRGB(26, 48, 32), Text = Color3.fromRGB(240, 240, 240) },
    Gold = { Bg = Color3.fromRGB(30, 28, 18), Sidebar = Color3.fromRGB(22, 20, 12), Accent = Color3.fromRGB(255, 200, 50), Button = Color3.fromRGB(48, 44, 26), Text = Color3.fromRGB(240, 240, 240) },
    Pink = { Bg = Color3.fromRGB(30, 18, 26), Sidebar = Color3.fromRGB(22, 12, 18), Accent = Color3.fromRGB(255, 100, 200), Button = Color3.fromRGB(48, 26, 40), Text = Color3.fromRGB(240, 240, 240) },
    Ocean = { Bg = Color3.fromRGB(15, 20, 32), Sidebar = Color3.fromRGB(10, 14, 24), Accent = Color3.fromRGB(30, 140, 255), Button = Color3.fromRGB(20, 32, 50), Text = Color3.fromRGB(240, 240, 240) }
}

local CurrentTheme = Themes.Dark

-- 3. Khởi tạo Gui
local ParentGui = (gethui and gethui()) or CoreGui or LocalPlayer:WaitForChild("PlayerGui")
if ParentGui:FindFirstChild("KianbestMenuV4") then
    ParentGui.KianbestMenuV4:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KianbestMenuV4"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = ParentGui

-- 4. HỆ THỐNG THÔNG BÁO (FADE-IN / FADE-OUT)
local NotificationFrame = Instance.new("Frame")
NotificationFrame.Name = "NotificationFrame"
NotificationFrame.Size = UDim2.new(0, 220, 0, 200)
NotificationFrame.Position = UDim2.new(1, -230, 1, -210)
NotificationFrame.BackgroundTransparency = 1
NotificationFrame.ZIndex = 10
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
    Toast.ZIndex = 10
    Toast.Parent = NotificationFrame

    Instance.new("UICorner", Toast).CornerRadius = UDim.new(0, 6)
    local Stroke = Instance.new("UIStroke", Toast)
    Stroke.Color = CurrentTheme.Accent
    Stroke.Thickness = 1.5
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
    TTitle.ZIndex = 11
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
    TText.ZIndex = 11
    TText.Parent = Toast

    local fadeInInfo = TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    TweenService:Create(Toast, fadeInInfo, {BackgroundTransparency = 0.15}):Play()
    TweenService:Create(Stroke, fadeInInfo, {Transparency = 0}):Play()
    TweenService:Create(TTitle, fadeInInfo, {TextTransparency = 0}):Play()
    TweenService:Create(TText, fadeInInfo, {TextTransparency = 0}):Play()

    task.delay(duration, function()
        local fadeOutInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
        TweenService:Create(Toast, fadeOutInfo, {BackgroundTransparency = 1}):Play()
        TweenService:Create(Stroke, fadeOutInfo, {Transparency = 1}):Play()
        TweenService:Create(TTitle, fadeOutInfo, {TextTransparency = 1}):Play()
        local lastTween = TweenService:Create(TText, fadeOutInfo, {TextTransparency = 1})
        lastTween:Play()
        lastTween.Completed:Connect(function() Toast:Destroy() end)
    end)
end

-- 5. MAIN FRAME MENU (CÓ BACKGROUND ANIME TRẮNG ĐEN MỜ)
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
MainStroke.Thickness = 2
MainStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

-- 🖼️ HÌNH NỀN ANIME TRẮNG ĐEN MỜ MỜ CHUYÊN NGHIỆP
local MainBgImage = Instance.new("ImageLabel")
MainBgImage.Name = "MainBgAnime"
MainBgImage.Size = UDim2.new(1, 0, 1, 0)
MainBgImage.BackgroundTransparency = 1
MainBgImage.ImageTransparency = 0.82 -- Độ mờ nghệ thuật
MainBgImage.ScaleType = Enum.ScaleType.Crop
MainBgImage.Image = "rbxassetid://10023471025" -- Asset Anime Manga B&W
MainBgImage.ImageColor3 = Color3.fromRGB(220, 220, 220)
MainBgImage.ZIndex = 1
MainBgImage.Parent = MainFrame

-- HEADER BAR
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 38)
Header.BackgroundColor3 = CurrentTheme.Sidebar
Header.BackgroundTransparency = 0.1
Header.BorderSizePixel = 0
Header.ZIndex = 3
Header.Parent = MainFrame

Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 10)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0, 180, 1, 0)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.Text = "Kianbest Hub v4.0"
Title.TextColor3 = CurrentTheme.Accent
Title.TextSize = 16
Title.Font = Enum.Font.FredokaOne
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1
Title.ZIndex = 4
Title.Parent = Header

local StatsLabel = Instance.new("TextLabel")
StatsLabel.Size = UDim2.new(0, 200, 1, 0)
StatsLabel.Position = UDim2.new(1, -240, 0, 0)
StatsLabel.Text = "FPS: -- | Ping: --ms"
StatsLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
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
CloseBtn.TextColor3 = Color3.fromRGB(255, 90, 90)
CloseBtn.TextSize = 14
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 42)
CloseBtn.BorderSizePixel = 0
CloseBtn.ZIndex = 4
CloseBtn.Parent = Header
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

-- 🔘 NÚT NỔI TRÒN ĐÓNG/MỞ BẮT MẮT (CHỮ "Kian")
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Name = "KianToggleButton"
ToggleBtn.Size = UDim2.new(0, 52, 0, 52)
ToggleBtn.Position = UDim2.new(0, 15, 0.4, 0)
ToggleBtn.BackgroundColor3 = CurrentTheme.Sidebar
ToggleBtn.Text = "Kian"
ToggleBtn.TextColor3 = CurrentTheme.Accent
ToggleBtn.Font = Enum.Font.FredokaOne
ToggleBtn.TextSize = 16
ToggleBtn.Active = true
ToggleBtn.Draggable = true
ToggleBtn.ZIndex = 100
ToggleBtn.Parent = ScreenGui

Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)

local ButtonStroke = Instance.new("UIStroke", ToggleBtn)
ButtonStroke.Color = CurrentTheme.Accent
ButtonStroke.Thickness = 3

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
-- SIDEBAR & TAB SYSTEM
----------------------------------------------------------
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 130, 1, -38)
Sidebar.Position = UDim2.new(0, 0, 0, 38)
Sidebar.BackgroundColor3 = CurrentTheme.Sidebar
Sidebar.BackgroundTransparency = 0.25
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
    StatusInd.Size = UDim2.new(0, 30, 0, 16)
    StatusInd.Position = UDim2.new(1, -38, 0.5, -8)
    StatusInd.BackgroundColor3 = defaultState and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(80, 80, 90)
    StatusInd.BorderSizePixel = 0
    StatusInd.ZIndex = 5
    StatusInd.Parent = Btn

    Instance.new("UICorner", StatusInd).CornerRadius = UDim.new(1, 0)

    local state = defaultState
    Btn.MouseButton1Click:Connect(function()
        state = not state
        StatusInd.BackgroundColor3 = state and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(80, 80, 90)
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
    MinusBtn.Size = UDim2.new(0, 28, 0, 24)
    MinusBtn.Position = UDim2.new(1, -66, 0.5, -12)
    MinusBtn.Text = "-"
    MinusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    MinusBtn.Font = Enum.Font.SourceSansBold
    MinusBtn.TextSize = 16
    MinusBtn.BackgroundColor3 = Color3.fromRGB(50, 55, 65)
    MinusBtn.BorderSizePixel = 0
    MinusBtn.ZIndex = 5
    MinusBtn.Parent = Container
    Instance.new("UICorner", MinusBtn).CornerRadius = UDim.new(0, 4)

    local PlusBtn = Instance.new("TextButton")
    PlusBtn.Size = UDim2.new(0, 28, 0, 24)
    PlusBtn.Position = UDim2.new(1, -34, 0.5, -12)
    PlusBtn.Text = "+"
    PlusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    PlusBtn.Font = Enum.Font.SourceSansBold
    PlusBtn.TextSize = 16
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
-- TAB 2: MOVEMENT (CÓ THÊM NOCLIP ĐI XUYÊN TƯỜNG THẬT)
----------------------------------------------------------
local MList = Instance.new("UIListLayout", MovementPage)
MList.SortOrder = Enum.SortOrder.LayoutOrder
MList.Padding = UDim.new(0, 8)

-- CHỨC NĂNG NOCLIP THẬT
CreateToggle(MovementPage, "Đi Xuyên Tường (Noclip)", Config.NoclipEnabled, function(state)
    Config.NoclipEnabled = state
    Notify("Movement", state and "Đã BẬT Đi Xuyên Tường!" or "Đã TẮT Đi Xuyên Tường!")
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

CreateToggle(MovementPage, "Chế độ Bay (Fly Mode)", Config.FlyEnabled, function(state)
    Config.FlyEnabled = state
    Notify("Movement", state and "Đã BẬT Fly Mode" or "Đã TẮT Fly Mode")
    
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not Config.FlyEnabled then
        if bodyVel then bodyVel:Destroy() end
        if bodyGyro then bodyGyro:Destroy() end
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

CreateValueAdjuster(MovementPage, "Tốc Độ Bay", 20, 300, Config.FlySpeed, 15, function(val)
    Config.FlySpeed = val
end)

CreateToggle(MovementPage, "Đi Trên Không (Air Walk)", Config.AirWalkEnabled, function(state)
    Config.AirWalkEnabled = state
    Notify("Movement", state and "Đã BẬT Air Walk" or "Đã TẮT Air Walk")
end)

----------------------------------------------------------
-- TAB 3: FIX LAG (NÂNG CẤP ĐA DẠNG CHẾ ĐỘ TỐI ƯU)
----------------------------------------------------------
local FList = Instance.new("UIListLayout", FixLagPage)
FList.SortOrder = Enum.SortOrder.LayoutOrder
FList.Padding = UDim.new(0, 8)

-- 1. Full Auto Boost
local BoostBtn = Instance.new("TextButton")
BoostBtn.Size = UDim2.new(1, -6, 0, 36)
BoostBtn.Text = "🚀 Super FPS Boost (1-Click Tối Ưu Nhanh)"
BoostBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
BoostBtn.Font = Enum.Font.SourceSansBold
BoostBtn.TextSize = 13
BoostBtn.BackgroundColor3 = Color3.fromRGB(0, 160, 220)
BoostBtn.BorderSizePixel = 0
BoostBtn.ZIndex = 4
BoostBtn.Parent = FixLagPage
Instance.new("UICorner", BoostBtn).CornerRadius = UDim.new(0, 6)

BoostBtn.MouseButton1Click:Connect(function()
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
    Notify("Fix Lag", "Đã bật Super FPS Boost thành công!")
end)

-- 2. Toggle Xóa Texture (Plastic Mode)
CreateToggle(FixLagPage, "Xóa Textures Map (Plastic Smooth)", false, function(state)
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Material = state and Enum.Material.SmoothPlastic or Enum.Material.Plastic
        end
    end
    Notify("Fix Lag", state and "Đã bật Smooth Plastic" or "Đã khôi phục Texture")
end)

-- 3. Toggle Đổi Bầu Trời Đen / Xám (Black Skybox)
CreateToggle(FixLagPage, "Đổi Bầu Trời Tối Đơn Sắc (Black Sky)", false, function(state)
    if state then
        for _, v in ipairs(Lighting:GetChildren()) do
            if v:IsA("Sky") or v:IsA("Atmosphere") then v:Destroy() end
        end
        Lighting.OutdoorAmbient = Color3.fromRGB(30, 30, 30)
        Lighting.Ambient = Color3.fromRGB(30, 30, 30)
    else
        Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
    end
    Notify("Fix Lag", state and "Đã bật Bầu Trời Tối" or "Đã tắt Bầu Trời Tối")
end)

-- 4. Toggle Tắt Bóng Đổ & Sương Mù (No Fog & Shadows)
CreateToggle(FixLagPage, "Tắt Bóng Đổ & Sương Mù (No Shadow/Fog)", true, function(state)
    Lighting.GlobalShadows = not state
    Lighting.FogEnd = state and 9e9 or 1000
    Notify("Fix Lag", state and "Đã tắt Bóng & Sương Mù" or "Đã bật Bóng Đổ")
end)

----------------------------------------------------------
-- TAB 4: SETTINGS
----------------------------------------------------------
local SList = Instance.new("UIListLayout", SettingsPage)
SList.SortOrder = Enum.SortOrder.LayoutOrder
SList.Padding = UDim.new(0, 6)

local ThemeLabel = Instance.new("TextLabel")
ThemeLabel.Size = UDim2.new(1, 0, 0, 18)
ThemeLabel.Text = "Chọn Chủ Đề Màu Sắc (8 Themes):"
ThemeLabel.TextColor3 = CurrentTheme.Text
ThemeLabel.Font = Enum.Font.SourceSansBold
ThemeLabel.TextSize = 13
ThemeLabel.TextXAlignment = Enum.TextXAlignment.Left
ThemeLabel.BackgroundTransparency = 1
ThemeLabel.ZIndex = 4
ThemeLabel.Parent = SettingsPage

local ThemeGrid = Instance.new("Frame")
ThemeGrid.Size = UDim2.new(1, -6, 0, 65)
ThemeGrid.BackgroundTransparency = 1
ThemeGrid.ZIndex = 4
ThemeGrid.Parent = SettingsPage

local themeList = {"Dark", "Cyan", "Purple", "Red", "Emerald", "Gold", "Pink", "Ocean"}
for i, name in ipairs(themeList) do
    local row = math.floor((i - 1) / 4)
    local col = (i - 1) % 4

    local TBtn = Instance.new("TextButton")
    TBtn.Size = UDim2.new(0, 80, 0, 26)
    TBtn.Position = UDim2.new(0, col * 86, 0, row * 32)
    TBtn.Text = name
    TBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    TBtn.Font = Enum.Font.SourceSansBold
    TBtn.TextSize = 11
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
        ToggleBtn.TextColor3 = CurrentTheme.Accent
        Notify("Theme", "Đã đổi sang Theme: " .. name)
    end)
end

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
    nameTxt.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameTxt.TextStrokeTransparency = 0.2
    nameTxt.Font = Enum.Font.SourceSansBold
    nameTxt.TextSize = 13
    nameTxt.Parent = bbg

    local distTxt = Instance.new("TextLabel")
    distTxt.Name = "DistLabel"
    distTxt.Size = UDim2.new(1, 0, 0, 14)
    distTxt.Position = UDim2.new(0, 0, 0, 16)
    distTxt.BackgroundTransparency = 1
    distTxt.Text = "0 Studs"
    distTxt.TextColor3 = Color3.fromRGB(0, 230, 255)
    distTxt.TextStrokeTransparency = 0.3
    distTxt.Font = Enum.Font.SourceSans
    distTxt.TextSize = 11
    distTxt.Parent = bbg

    local hpBg = Instance.new("Frame")
    hpBg.Name = "HPBackground"
    hpBg.Size = UDim2.new(0, 100, 0, 5)
    hpBg.Position = UDim2.new(0.5, -50, 0, 32)
    hpBg.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    hpBg.BorderSizePixel = 0
    hpBg.Visible = false
    hpBg.Parent = bbg
    Instance.new("UICorner", hpBg).CornerRadius = UDim.new(1, 0)

    local hpBar = Instance.new("Frame")
    hpBar.Name = "HPBar"
    hpBar.Size = UDim2.new(1, 0, 1, 0)
    hpBar.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
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
        highlight.FillTransparency = 0.4
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
-- REALTIME RENDER LOOP & NOCLIP STEPPED
----------------------------------------------------------
local lastTime = tick()
local frameCount = 0

-- VÒNG LẶP NOCLIP CHUẨN XÁC
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
    -- Calculate FPS/Ping
    frameCount = frameCount + 1
    if tick() - lastTime >= 1 then
        local currentFPS = math.floor(frameCount / (tick() - lastTime))
        local pingVal = 0
        pcall(function() pingVal = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue()) end)
        StatsLabel.Text = "FPS: " .. currentFPS .. " | Ping: " .. pingVal .. "ms"
        frameCount = 0
        lastTime = tick()
    end

    -- Speed Hack
    if Config.SpeedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = Config.SpeedValue
    end

    -- Fly Mode
    if Config.FlyEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local cam = workspace.CurrentCamera
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if bodyVel and hum then
            bodyVel.Velocity = cam.CFrame.LookVector * (hum.MoveDirection.Magnitude > 0 and Config.FlySpeed or 0)
        end
        if bodyGyro then bodyGyro.CFrame = cam.CFrame end
    end

    -- AirWalk
    if Config.AirWalkEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        if not airPlatform or not airPlatform.Parent then
            airPlatform = Instance.new("Part")
            airPlatform.Size = Vector3.new(6, 1, 6)
            airPlatform.Anchored = true
            airPlatform.Transparency = 0.5
            airPlatform.Material = Enum.Material.Neon
            airPlatform.Color = CurrentTheme.Accent
            airPlatform.Parent = workspace
        end
        local hrp = LocalPlayer.Character.HumanoidRootPart
        airPlatform.CFrame = CFrame.new(hrp.Position.X, hrp.Position.Y - 3.5, hrp.Position.Z)
    else
        if airPlatform then airPlatform:Destroy() airPlatform = nil end
    end

    -- ESP & Nametags
    local myChar = LocalPlayer.Character
    local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character then
            local char = p.Character
            local head = char:FindFirstChild("Head")
            local hum = char:FindFirstChildOfClass("Humanoid")
            local hl = char:FindFirstChild("ESPHighlight")

            if hl then hl.Enabled = Config.ESPEnabled end

            if head then
                local bbg = head:FindFirstChild("ESPNameTag")
                if bbg then
                    bbg.Enabled = Config.ESPNamesEnabled

                    if Config.ESPNamesEnabled then
                        local distLabel = bbg:FindFirstChild("DistLabel")
                        if distLabel and myHRP then
                            local dist = math.floor((head.Position - myHRP.Position).Magnitude)
                            distLabel.Text = tostring(dist) .. " Studs"
                        end

                        local hpBg = bbg:FindFirstChild("HPBackground")
                        if hpBg then
                            hpBg.Visible = Config.ESPHealthEnabled
                            if Config.ESPHealthEnabled and hum then
                                local hpBar = hpBg:FindFirstChild("HPBar")
                                if hpBar then
                                    local ratio = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
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

Notify("Kianbest Hub", "Đã cập nhật phiên bản v4.0 Supreme!")
