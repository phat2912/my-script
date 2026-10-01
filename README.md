-- ==========================================================
-- SCRIPT MENU SYSTEM V2.0 PREMIUM (Kianbest)
-- Features: ESP + Movement + FixLag + FPS/Ping + Animations
-- ==========================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer

-- 1. Cấu hình & Trạng thái (Config)
local Config = {
    ESPEnabled = false,
    ESPNamesEnabled = false,
    ESPColor = Color3.fromRGB(0, 255, 150),
    
    SpeedEnabled = false,
    SpeedValue = 50,
    
    FlyEnabled = false,
    FlySpeed = 50,
    
    AirWalkEnabled = false,
    ToggleKey = Enum.KeyCode.RightControl
}

-- 2. Bộ chủ đề giao diện (Themes)
local Themes = {
    Dark = {
        Bg = Color3.fromRGB(20, 22, 28),
        Sidebar = Color3.fromRGB(15, 16, 20),
        Accent = Color3.fromRGB(0, 170, 255),
        Button = Color3.fromRGB(30, 34, 42),
        Text = Color3.fromRGB(240, 240, 240)
    },
    Cyan = {
        Bg = Color3.fromRGB(15, 25, 35),
        Sidebar = Color3.fromRGB(10, 18, 26),
        Accent = Color3.fromRGB(0, 210, 255),
        Button = Color3.fromRGB(22, 38, 52),
        Text = Color3.fromRGB(240, 240, 240)
    },
    Purple = {
        Bg = Color3.fromRGB(25, 18, 35),
        Sidebar = Color3.fromRGB(18, 12, 26),
        Accent = Color3.fromRGB(180, 70, 255),
        Button = Color3.fromRGB(38, 26, 52),
        Text = Color3.fromRGB(240, 240, 240)
    },
    Red = {
        Bg = Color3.fromRGB(30, 18, 18),
        Sidebar = Color3.fromRGB(22, 12, 12),
        Accent = Color3.fromRGB(255, 70, 70),
        Button = Color3.fromRGB(48, 26, 26),
        Text = Color3.fromRGB(240, 240, 240)
    }
}

local CurrentTheme = Themes.Dark

-- 3. Tạo ScreenGui
local ParentGui = (gethui and gethui()) or CoreGui or LocalPlayer:WaitForChild("PlayerGui")
if ParentGui:FindFirstChild("KianbestMenuV2") then
    ParentGui.KianbestMenuV2:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KianbestMenuV2"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = ParentGui

-- 4. HỆ THỐNG THÔNG BÁO (NOTIFICATION TOAST)
local NotificationFrame = Instance.new("Frame")
NotificationFrame.Name = "NotificationFrame"
NotificationFrame.Size = UDim2.new(0, 220, 0, 150)
NotificationFrame.Position = UDim2.new(1, -230, 1, -160)
NotificationFrame.BackgroundTransparency = 1
NotificationFrame.ZIndex = 10
NotificationFrame.Parent = ScreenGui

local function Notify(title, text, duration)
    duration = duration or 2
    local Toast = Instance.new("Frame")
    Toast.Size = UDim2.new(1, 0, 0, 45)
    Toast.Position = UDim2.new(1, 50, 0, 0)
    Toast.BackgroundColor3 = CurrentTheme.Sidebar
    Toast.BorderSizePixel = 0
    Toast.ZIndex = 10
    Toast.Parent = NotificationFrame

    Instance.new("UICorner", Toast).CornerRadius = UDim.new(0, 6)
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
    TTitle.ZIndex = 11
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
    TText.ZIndex = 11
    TText.Parent = Toast

    TweenService:Create(Toast, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Position = UDim2.new(0, 0, 0, 0)}):Play()

    task.delay(duration, function()
        local tween = TweenService:Create(Toast, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {Position = UDim2.new(1, 50, 0, 0)})
        tween:Play()
        tween.Completed:Connect(function()
            Toast:Destroy()
        end)
    end)
end

-- 5. MAIN FRAME
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

-- Ảnh nền Menu
local MainBgImage = Instance.new("ImageLabel")
MainBgImage.Name = "MainBgImage"
MainBgImage.Size = UDim2.new(1, 0, 1, 0)
MainBgImage.BackgroundTransparency = 1
MainBgImage.ImageTransparency = 0.55
MainBgImage.ScaleType = Enum.ScaleType.Crop
MainBgImage.Image = "rbxassetid://6031075931"
MainBgImage.ZIndex = 0
MainBgImage.Parent = MainFrame

-- HEADER BAR
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 38)
Header.BackgroundColor3 = CurrentTheme.Sidebar
Header.BackgroundTransparency = 0.1
Header.BorderSizePixel = 0
Header.ZIndex = 2
Header.Parent = MainFrame

Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 10)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0, 150, 1, 0)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.Text = "Kianbest Hub v2.0"
Title.TextColor3 = CurrentTheme.Accent
Title.TextSize = 16
Title.Font = Enum.Font.SourceSansBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1
Title.ZIndex = 3
Title.Parent = Header

-- Hiển thị FPS & Ping
local StatsLabel = Instance.new("TextLabel")
StatsLabel.Size = UDim2.new(0, 180, 1, 0)
StatsLabel.Position = UDim2.new(1, -220, 0, 0)
StatsLabel.Text = "FPS: 60 | Ping: 0ms"
StatsLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
StatsLabel.TextSize = 12
StatsLabel.Font = Enum.Font.SourceSansBold
StatsLabel.TextXAlignment = Enum.TextXAlignment.Right
StatsLabel.BackgroundTransparency = 1
StatsLabel.ZIndex = 3
StatsLabel.Parent = Header

-- Nút Đóng
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 26, 0, 26)
CloseBtn.Position = UDim2.new(1, -32, 0, 6)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 90, 90)
CloseBtn.TextSize = 14
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 42)
CloseBtn.BorderSizePixel = 0
CloseBtn.ZIndex = 3
CloseBtn.Parent = Header
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

-- 6. NÚT TRÒN MỞ/ĐÓNG MENU (TOGGLE BUTTON)
local ToggleBtn = Instance.new("ImageButton")
ToggleBtn.Name = "ToggleButton"
ToggleBtn.Size = UDim2.new(0, 50, 0, 50)
ToggleBtn.Position = UDim2.new(0, 15, 0.4, 0)
ToggleBtn.BackgroundColor3 = CurrentTheme.Sidebar
ToggleBtn.Image = "rbxassetid://6031075931"
ToggleBtn.Active = true
ToggleBtn.Draggable = true
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
    if not gameProcessed and input.KeyCode == Config.ToggleKey then
        ToggleMenu()
    end
end)

----------------------------------------------------------
-- SIDEBAR & TAB CONTAINER SYSTEM
----------------------------------------------------------
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 130, 1, -38)
Sidebar.Position = UDim2.new(0, 0, 0, 38)
Sidebar.BackgroundColor3 = CurrentTheme.Sidebar
Sidebar.BackgroundTransparency = 0.15
Sidebar.BorderSizePixel = 0
Sidebar.ZIndex = 2
Sidebar.Parent = MainFrame

local ContentContainer = Instance.new("Frame")
ContentContainer.Size = UDim2.new(1, -140, 1, -48)
ContentContainer.Position = UDim2.new(0, 135, 0, 43)
ContentContainer.BackgroundTransparency = 1
ContentContainer.ZIndex = 2
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
    TabBtn.ZIndex = 3
    TabBtn.Parent = Sidebar

    Instance.new("UICorner", TabBtn).CornerRadius = UDim.new(0, 6)

    local Page = Instance.new("Frame")
    Page.Size = UDim2.new(1, 0, 1, 0)
    Page.BackgroundTransparency = 1
    Page.Visible = (posIndex == 1)
    Page.ZIndex = 2
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

    if posIndex == 1 then
        TabBtn.BackgroundColor3 = CurrentTheme.Accent
    end

    return Page
end

-- TẠO CÁC TAB CÓ BIỂU TƯỢNG (ICONS)
local VisualsPage = CreateTab("Visuals", "👁️", 1)
local MovementPage = CreateTab("Movement", "⚡", 2)
local FixLagPage = CreateTab("Fix Lag", "🚀", 3)
local SettingsPage = CreateTab("Settings", "⚙️", 4)

----------------------------------------------------------
-- HELPER CREATORS (UI CREATION UTILS)
----------------------------------------------------------
local function CreateToggle(parent, text, defaultState, callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, 0, 0, 34)
    Btn.Text = "  " .. text
    Btn.TextColor3 = CurrentTheme.Text
    Btn.Font = Enum.Font.SourceSansBold
    Btn.TextSize = 13
    Btn.TextXAlignment = Enum.TextXAlignment.Left
    Btn.BackgroundColor3 = CurrentTheme.Button
    Btn.BorderSizePixel = 0
    Btn.ZIndex = 3
    Btn.Parent = parent

    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 6)

    local StatusInd = Instance.new("Frame")
    StatusInd.Size = UDim2.new(0, 30, 0, 16)
    StatusInd.Position = UDim2.new(1, -38, 0.5, -8)
    StatusInd.BackgroundColor3 = defaultState and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(80, 80, 90)
    StatusInd.BorderSizePixel = 0
    StatusInd.ZIndex = 4
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

----------------------------------------------------------
-- TAB 1: VISUALS (ESP)
----------------------------------------------------------
local VList = Instance.new("UIListLayout", VisualsPage)
VList.SortOrder = Enum.SortOrder.LayoutOrder
VList.Padding = UDim.new(0, 8)

CreateToggle(VisualsPage, "ESP Highlight (Xuyên Tường)", Config.ESPEnabled, function(state)
    Config.ESPEnabled = state
    Notify("Visuals", state and "Đã bật ESP Highlight" or "Đã tắt ESP Highlight")
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("ESPHighlight") then
            p.Character.ESPHighlight.Enabled = Config.ESPEnabled
        end
    end
end)

CreateToggle(VisualsPage, "ESP Name & Tên Khoảng Cách (Studs)", Config.ESPNamesEnabled, function(state)
    Config.ESPNamesEnabled = state
    Notify("Visuals", state and "Đã bật ESP Tên & Khoảng cách" or "Đã tắt ESP Tên")
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
            local bbg = p.Character.Head:FindFirstChild("ESPNameTag")
            if bbg then bbg.Enabled = Config.ESPNamesEnabled end
        end
    end
end)

----------------------------------------------------------
-- TAB 2: MOVEMENT
----------------------------------------------------------
local MList = Instance.new("UIListLayout", MovementPage)
MList.SortOrder = Enum.SortOrder.LayoutOrder
MList.Padding = UDim.new(0, 8)

CreateToggle(MovementPage, "Chạy Nhanh (Speed Hack)", Config.SpeedEnabled, function(state)
    Config.SpeedEnabled = state
    Notify("Movement", state and "Đã BẬT Speed Hack" or "Đã TẮT Speed Hack")
    if not Config.SpeedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 16
    end
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

CreateToggle(MovementPage, "Đi Trên Không (Air Walk)", Config.AirWalkEnabled, function(state)
    Config.AirWalkEnabled = state
    Notify("Movement", state and "Đã BẬT Air Walk" or "Đã TẮT Air Walk")
end)

----------------------------------------------------------
-- TAB 3: FIX LAG & BOOST FPS
----------------------------------------------------------
local FList = Instance.new("UIListLayout", FixLagPage)
FList.SortOrder = Enum.SortOrder.LayoutOrder
FList.Padding = UDim.new(0, 8)

local BoostBtn = Instance.new("TextButton")
BoostBtn.Size = UDim2.new(1, 0, 0, 40)
BoostBtn.Text = "🚀 BẬT TỐI ƯU FPS / FIX LAG CỰC ĐẠI"
BoostBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
BoostBtn.Font = Enum.Font.SourceSansBold
BoostBtn.TextSize = 14
BoostBtn.BackgroundColor3 = Color3.fromRGB(0, 160, 220)
BoostBtn.BorderSizePixel = 0
BoostBtn.ZIndex = 3
BoostBtn.Parent = FixLagPage

Instance.new("UICorner", BoostBtn).CornerRadius = UDim.new(0, 6)

BoostBtn.MouseButton1Click:Connect(function()
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 9e9
    
    local Terrain = workspace:FindFirstChildOfClass('Terrain')
    if Terrain then
        Terrain.WaterWaveSize = 0
        Terrain.WaterWaveSpeed = 0
        Terrain.WaterReflectance = 0
        Terrain.WaterTransparency = 0
    end
    
    for _, v in ipairs(game:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Material = Enum.Material.SmoothPlastic
            v.CastShadow = false
        elseif v:IsA("Decal") or v:IsA("Texture") then
            v.Transparency = 1
        elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") then
            v.Enabled = false
        end
    end
    
    Notify("Fix Lag", "Đã dọn dẹp Đồ Họa & Tăng FPS!")
    BoostBtn.Text = "✔ ĐÃ TỐI ƯU HÓA THÀNH CÔNG!"
    BoostBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 80)
end)

----------------------------------------------------------
-- TAB 4: SETTINGS
----------------------------------------------------------
local SList = Instance.new("UIListLayout", SettingsPage)
SList.SortOrder = Enum.SortOrder.LayoutOrder
SList.Padding = UDim.new(0, 6)

local ThemeLabel = Instance.new("TextLabel")
ThemeLabel.Size = UDim2.new(1, 0, 0, 18)
ThemeLabel.Text = "Đổi Màu Theme:"
ThemeLabel.TextColor3 = CurrentTheme.Text
ThemeLabel.Font = Enum.Font.SourceSansBold
ThemeLabel.TextSize = 13
ThemeLabel.TextXAlignment = Enum.TextXAlignment.Left
ThemeLabel.BackgroundTransparency = 1
ThemeLabel.ZIndex = 3
ThemeLabel.Parent = SettingsPage

local ThemeGrid = Instance.new("Frame")
ThemeGrid.Size = UDim2.new(1, 0, 0, 30)
ThemeGrid.BackgroundTransparency = 1
ThemeGrid.ZIndex = 3
ThemeGrid.Parent = SettingsPage

local themeNames = {"Dark", "Cyan", "Purple", "Red"}
for i, name in ipairs(themeNames) do
    local TBtn = Instance.new("TextButton")
    TBtn.Size = UDim2.new(0, 85, 0, 26)
    TBtn.Position = UDim2.new(0, (i - 1) * 92, 0, 0)
    TBtn.Text = name
    TBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    TBtn.Font = Enum.Font.SourceSansBold
    TBtn.TextSize = 12
    TBtn.BackgroundColor3 = Themes[name].Accent
    TBtn.BorderSizePixel = 0
    TBtn.ZIndex = 4
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
        Notify("Theme", "Đã đổi sang Theme: " .. name)
    end)
end

-- Ô ĐỔI ẢNH NỀN
local ImgLabel = Instance.new("TextLabel")
ImgLabel.Size = UDim2.new(1, 0, 0, 18)
ImgLabel.Text = "Nhập ID Ảnh Nền (Roblox Decal ID):"
ImgLabel.TextColor3 = CurrentTheme.Text
ImgLabel.Font = Enum.Font.SourceSansBold
ImgLabel.TextSize = 13
ImgLabel.TextXAlignment = Enum.TextXAlignment.Left
ImgLabel.BackgroundTransparency = 1
ImgLabel.ZIndex = 3
ImgLabel.Parent = SettingsPage

local ImageBox = Instance.new("TextBox")
ImageBox.Size = UDim2.new(1, 0, 0, 30)
ImageBox.PlaceholderText = "Nhập ID Ảnh tại đây (Ví dụ: 6031075931)..."
ImageBox.Text = ""
ImageBox.TextColor3 = Color3.fromRGB(255, 255, 255)
ImageBox.BackgroundColor3 = CurrentTheme.Button
ImageBox.Font = Enum.Font.SourceSans
ImageBox.TextSize = 13
ImageBox.BorderSizePixel = 0
ImageBox.ZIndex = 3
ImageBox.Parent = SettingsPage
Instance.new("UICorner", ImageBox).CornerRadius = UDim.new(0, 4)

ImageBox.FocusLost:Connect(function(enter)
    if enter then
        local cleanID = string.match(ImageBox.Text, "%d+")
        if cleanID then
            MainBgImage.Image = "rbxassetid://" .. cleanID
            ToggleBtn.Image = "rbxassetid://" .. cleanID
            Notify("Background", "Đổi ảnh nền thành công!")
            ImageBox.Text = ""
        end
    end
end)

----------------------------------------------------------
-- LOGIC ESP, RUNSERVICE, RENDER LOOPS
----------------------------------------------------------
local FrameCount = 0
local LastUpdate = tick()

RunService.RenderStepped:Connect(function()
    -- Tính FPS & Ping
    FrameCount = FrameCount + 1
    if tick() - LastUpdate >= 1 then
        local fps = math.floor(FrameCount / (tick() - LastUpdate))
        local ping = math.floor(workspace:GetRealPhysicalResponseTime() * 1000)
        StatsLabel.Text = "FPS: " .. fps .. " | Ping: " .. ping .. "ms"
        FrameCount = 0
        LastUpdate = tick()
    end

    -- Update Speed
    if Config.SpeedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = Config.SpeedValue
    end

    -- Update Fly
    if Config.FlyEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local cam = workspace.CurrentCamera
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if bodyVel and hum then
            bodyVel.Velocity = cam.CFrame.LookVector * (hum.MoveDirection.Magnitude > 0 and Config.FlySpeed or 0)
        end
        if bodyGyro then bodyGyro.CFrame = cam.CFrame end
    end

    -- Update AirWalk
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
        if airPlatform then
            airPlatform:Destroy()
            airPlatform = nil
        end
    end

    -- Update ESP Names & Distance
    if Config.ESPNamesEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local myPos = LocalPlayer.Character.HumanoidRootPart.Position
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") and p.Character:FindFirstChild("HumanoidRootPart") then
                local bbg = p.Character.Head:FindFirstChild("ESPNameTag")
                if bbg and bbg:FindFirstChild("TextLabel") then
                    local dist = math.floor((p.Character.HumanoidRootPart.Position - myPos).Magnitude)
                    bbg.TextLabel.Text = p.Name .. " [" .. tostring(dist) .. " Studs]"
                end
            end
        end
    end
end)

-- SETUP ESP PLAYER
local function SetupESP(player)
    if player == LocalPlayer then return end

    local function CharacterAdded(char)
        if not char then return end
        
        local highlight = char:FindFirstChild("ESPHighlight") or Instance.new("Highlight")
        highlight.Name = "ESPHighlight"
        highlight.Adornee = char
        highlight.FillColor = Config.ESPColor
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.FillTransparency = 0.4
        highlight.Enabled = Config.ESPEnabled
        highlight.Parent = char

        local head = char:WaitForChild("Head", 5)
        if head then
            local bbg = head:FindFirstChild("ESPNameTag") or Instance.new("BillboardGui")
            bbg.Name = "ESPNameTag"
            bbg.Adornee = head
            bbg.Size = UDim2.new(0, 150, 0, 30)
            bbg.StudsOffset = Vector3.new(0, 2.5, 0)
            bbg.AlwaysOnTop = true
            bbg.Enabled = Config.ESPNamesEnabled
            bbg.Parent = head

            local txt = bbg:FindFirstChild("TextLabel") or Instance.new("TextLabel")
            txt.Name = "TextLabel"
            txt.Size = UDim2.new(1, 0, 1, 0)
            txt.BackgroundTransparency = 1
            txt.TextColor3 = Color3.fromRGB(255, 255, 255)
            txt.TextStrokeTransparency = 0
            txt.Font = Enum.Font.SourceSansBold
            txt.TextSize = 13
            txt.Parent = bbg
        end
    end

    if player.Character then CharacterAdded(player.Character) end
    player.CharacterAdded:Connect(CharacterAdded)
end

for _, p in ipairs(Players:GetPlayers()) do SetupESP(p) end
Players.PlayerAdded:Connect(SetupESP)

Notify("Kianbest Hub", "Đã tải thành công v2.0 Premium!")
