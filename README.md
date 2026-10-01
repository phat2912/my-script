-- ==========================================================
-- SCRIPT MENU SYSTEM V12.7 ULTRA CHILL (Kianbest Hub)
-- Updates: Auto Toggle Notifications, 10 Chill Presets, Reset BG Button
-- Compatibility: Delta, Hydrogen, Fluxus, Solara, Wave, CodeX
-- ==========================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local Stats = game:GetService("Stats")
local TeleportService = game:GetService("TeleportService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

local VirtualInputManager, VirtualUser
pcall(function() VirtualInputManager = game:GetService("VirtualInputManager") end)
pcall(function() VirtualUser = game:GetService("VirtualUser") end)

-- CẤU HÌNH HỆ THỐNG
local Config = {
    AntiBanEnabled = true,
    AntiKick = true,
    AntiLog = true,
    SpoofStats = true,
    DisableClientAC = true,

    ESPEnabled = false,
    ESPNamesEnabled = false,
    ESPHealthEnabled = false,
    
    NoclipEnabled = false,
    SpeedEnabled = false,
    SpeedValue = 35,
    FlyEnabled = false,
    FlySpeed = 75,
    
    PotatoMode = false,
    FullBright = false,
    LowPoly = false,
    AutoMemoryClean = true,
    HidePlayers = false,
    
    SelectedTarget = nil,
    AutoTPTarget = false,
    AutoAttack = false,
    SuperM1Damage = false,
    DamageMultiplier = 30,
    HitboxExpander = false,
    HitboxSize = 18,
    
    AntiAFK = true,
    CurrentChillIndex = 1,
    CustomAssetID = "",
    BgTransparency = 0.45,
    FrameTransparency = 0.25,
    ToggleKey = Enum.KeyCode.RightControl
}

-- DANH SÁCH 10 IMAGE ID CHILL LO-FI / AESTHETIC HD
local ChillPresets = {
    "rbxassetid://6071575925",  -- 1. Lo-Fi Rainy City Night (Mặc định)
    "rbxassetid://7043825807",  -- 2. Soft Pink Sunset Sky
    "rbxassetid://6985068228",  -- 3. Chill Anime Room
    "rbxassetid://11414436906", -- 4. Pastel Purple Clouds
    "rbxassetid://10023403248", -- 5. Anime Cozy Street
    "rbxassetid://11702739401", -- 6. Chill Galaxy Night
    "rbxassetid://6032220401",  -- 7. Cyberpunk Neon City
    "rbxassetid://142410803",   -- 8. Minimalist Dark Lace Pattern
    "rbxassetid://9132200898",  -- 9. Aesthetic Dusk Cloud
    "rbxassetid://7041740322"   -- 10. Retro Purple Horizon
}

local Themes = {
    ChillPurple = { Name = "Chill Lavender 🔮", Bg = Color3.fromRGB(20, 16, 28), Sidebar = Color3.fromRGB(14, 10, 20), Accent = Color3.fromRGB(185, 140, 255), Button = Color3.fromRGB(32, 24, 44), Text = Color3.fromRGB(240, 235, 255) },
    SoftPink = { Name = "Soft Sakura 🌸", Bg = Color3.fromRGB(28, 18, 24), Sidebar = Color3.fromRGB(20, 12, 17), Accent = Color3.fromRGB(255, 150, 190), Button = Color3.fromRGB(42, 26, 36), Text = Color3.fromRGB(255, 240, 248) },
    OceanBlue = { Name = "Midnight Ocean 🌊", Bg = Color3.fromRGB(14, 22, 32), Sidebar = Color3.fromRGB(9, 15, 24), Accent = Color3.fromRGB(100, 200, 255), Button = Color3.fromRGB(22, 34, 48), Text = Color3.fromRGB(235, 248, 255) },
    MintGreen = { Name = "Chill Matcha 🍃", Bg = Color3.fromRGB(16, 26, 22), Sidebar = Color3.fromRGB(10, 18, 15), Accent = Color3.fromRGB(130, 230, 175), Button = Color3.fromRGB(24, 40, 33), Text = Color3.fromRGB(235, 255, 242) }
}

local CurrentTheme = Themes.ChillPurple

local ParentGui = (gethui and gethui()) or CoreGui or LocalPlayer:WaitForChild("PlayerGui")
if ParentGui:FindFirstChild("KianbestMenuV12_7") then
    ParentGui.KianbestMenuV12_7:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KianbestMenuV12_7"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = ParentGui

-- SYSTEM NOTIFICATION TOAST
local NotificationFrame = Instance.new("Frame")
NotificationFrame.Name = "NotificationFrame"
NotificationFrame.Size = UDim2.new(0, 250, 0, 240)
NotificationFrame.Position = UDim2.new(1, -260, 1, -250)
NotificationFrame.BackgroundTransparency = 1
NotificationFrame.ZIndex = 50
NotificationFrame.Parent = ScreenGui

local NotificationList = Instance.new("UIListLayout", NotificationFrame)
NotificationList.SortOrder = Enum.SortOrder.LayoutOrder
NotificationList.Padding = UDim.new(0, 6)
NotificationList.VerticalAlignment = Enum.VerticalAlignment.Bottom

local function Notify(title, text, duration)
    duration = duration or 2.2
    local Toast = Instance.new("Frame")
    Toast.Size = UDim2.new(1, 0, 0, 46)
    Toast.BackgroundColor3 = CurrentTheme.Sidebar
    Toast.BackgroundTransparency = 0.15
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
        pcall(function()
            local fadeOutInfo = TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.In)
            TweenService:Create(Toast, fadeOutInfo, {BackgroundTransparency = 1}):Play()
            TweenService:Create(Stroke, fadeOutInfo, {Transparency = 1}):Play()
            TweenService:Create(TTitle, fadeOutInfo, {TextTransparency = 1}):Play()
            local lastTween = TweenService:Create(TText, fadeOutInfo, {TextTransparency = 1})
            lastTween:Play()
            lastTween.Completed:Connect(function() Toast:Destroy() end)
        end)
    end)
end

----------------------------------------------------------
-- 🛡️ ULTRA ANTI-BAN ENGINE
----------------------------------------------------------
pcall(function()
    if not Config.AntiBanEnabled then return end

    if hookfunction then
        local oldKick
        oldKick = hookfunction(LocalPlayer.Kick, function(self, ...)
            if Config.AntiKick and self == LocalPlayer then
                Notify("🛡️ Anti-Ban Shield", "Đã chặn 1 yêu cầu Kick từ Server!")
                return nil
            end
            return oldKick(self, ...)
        end)
    end

    if hookmetamethod then
        local oldNamecall
        oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
            local method = getnamecallmethod()
            if Config.AntiKick and (method:lower() == "kick") and self == LocalPlayer then
                return nil
            end

            if Config.AntiLog and (method == "FireServer" or method == "InvokeServer") then
                local remoteName = tostring(self.Name):lower()
                local blockKeywords = {"ban", "kick", "flag", "cheat", "detect", "log", "ac", "check", "security", "report"}
                for _, word in ipairs(blockKeywords) do
                    if remoteName:find(word) then return nil end
                end
            end
            return oldNamecall(self, ...)
        end)

        local oldIndex
        oldIndex = hookmetamethod(game, "__index", function(self, key)
            if Config.SpoofStats and not checkcaller() and self:IsA("Humanoid") then
                if key == "WalkSpeed" then return 16 end
                if key == "JumpPower" then return 50 end
            end
            return oldIndex(self, key)
        end)
    end
end)

----------------------------------------------------------
-- MAIN FRAME & GLASSMORPHISM UI
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

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = CurrentTheme.Accent
MainStroke.Thickness = 1.5
MainStroke.Transparency = 0.3

local MainBgImage = Instance.new("ImageLabel")
MainBgImage.Name = "MainBgChill"
MainBgImage.Size = UDim2.new(1, 0, 1, 0)
MainBgImage.BackgroundTransparency = 1
MainBgImage.ImageTransparency = Config.BgTransparency
MainBgImage.ScaleType = Enum.ScaleType.Crop
MainBgImage.Image = ChillPresets[Config.CurrentChillIndex]
MainBgImage.ZIndex = 1
MainBgImage.Parent = MainFrame

local GlassGradient = Instance.new("UIGradient")
GlassGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 150, 200))
})
GlassGradient.Rotation = 45
GlassGradient.Parent = MainBgImage

-- HEADER BAR
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 38)
Header.BackgroundColor3 = CurrentTheme.Sidebar
Header.BackgroundTransparency = 0.35
Header.BorderSizePixel = 0
Header.ZIndex = 3
Header.Parent = MainFrame
Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 12)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0, 320, 1, 0)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.Text = "☕ Kianbest Hub v12.7 ULTRA CHILL"
Title.TextColor3 = CurrentTheme.Accent
Title.TextSize = 13
Title.Font = Enum.Font.FredokaOne
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1
Title.ZIndex = 4
Title.Parent = Header

local StatsLabel = Instance.new("TextLabel")
StatsLabel.Size = UDim2.new(0, 180, 1, 0)
StatsLabel.Position = UDim2.new(1, -220, 0, 0)
StatsLabel.Text = "FPS: -- | Ping: --ms"
StatsLabel.TextColor3 = Color3.fromRGB(220, 210, 235)
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
CloseBtn.TextColor3 = Color3.fromRGB(255, 120, 140)
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
ToggleBtn.Size = UDim2.new(0, 48, 0, 48)
ToggleBtn.Position = UDim2.new(0, 15, 0.4, 0)
ToggleBtn.BackgroundColor3 = CurrentTheme.Sidebar
ToggleBtn.BackgroundTransparency = 0.2
ToggleBtn.Text = "☕"
ToggleBtn.TextSize = 20
ToggleBtn.Active = true
ToggleBtn.Draggable = true
ToggleBtn.ZIndex = 100
ToggleBtn.Parent = ScreenGui
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)

local ButtonStroke = Instance.new("UIStroke", ToggleBtn)
ButtonStroke.Color = CurrentTheme.Accent
ButtonStroke.Thickness = 2

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
-- SIDEBAR & TABS CONTAINER
----------------------------------------------------------
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 130, 1, -38)
Sidebar.Position = UDim2.new(0, 0, 0, 38)
Sidebar.BackgroundColor3 = CurrentTheme.Sidebar
Sidebar.BackgroundTransparency = 0.45
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
    TabBtn.BackgroundTransparency = 0.3
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
            tab.Button.BackgroundTransparency = 0.3
        end
        Page.Visible = true
        TabBtn.BackgroundColor3 = CurrentTheme.Accent
        TabBtn.BackgroundTransparency = 0.1
    end)

    if posIndex == 1 then 
        TabBtn.BackgroundColor3 = CurrentTheme.Accent 
        TabBtn.BackgroundTransparency = 0.1
    end
    return Page
end

local AntiBanPage = CreateTab("Anti-Ban Shield", "🛡️", 1)
local CombatPage = CreateTab("Combat VIP", "⚔️", 2)
local MovementPage = CreateTab("Movement", "⚡", 3)
local FixLagPage = CreateTab("Fix Lag VIP", "🚀", 4)
local SettingsPage = CreateTab("Settings & Chill", "☕", 5)

----------------------------------------------------------
-- UI BUILDERS WITH AUTOMATIC NOTIFICATIONS
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
    Btn.BackgroundTransparency = 0.25
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
        
        local cleanTitle = text:gsub("^[%s%p%c]+", "")
        cleanTitle = cleanTitle ~= "" and cleanTitle or "Tính năng"
        Notify(cleanTitle, state and "Đã BẬT 🟢" or "Đã TẮT 🔴")
        
        callback(state)
    end)
    return Btn
end

local function CreateValueAdjuster(parent, title, minVal, maxVal, defaultVal, step, callback)
    local Container = Instance.new("Frame")
    Container.Size = UDim2.new(1, -6, 0, 36)
    Container.BackgroundColor3 = CurrentTheme.Button
    Container.BackgroundTransparency = 0.25
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
    Btn.BackgroundTransparency = 0.2
    Btn.BorderSizePixel = 0
    Btn.ZIndex = 4
    Btn.Parent = parent
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 6)
    Btn.MouseButton1Click:Connect(callback)
    return Btn
end

----------------------------------------------------------
-- TAB 1: ANTI-BAN SHIELD
----------------------------------------------------------
local AList = Instance.new("UIListLayout", AntiBanPage)
AList.SortOrder = Enum.SortOrder.LayoutOrder
AList.Padding = UDim.new(0, 8)

CreateToggle(AntiBanPage, "🛡️ Chống Kick Tối Đa (Anti-Kick)", Config.AntiKick, function(state) Config.AntiKick = state end)
CreateToggle(AntiBanPage, "🚫 Chặn Gửi Log/Report Cho Game", Config.AntiLog, function(state) Config.AntiLog = state end)
CreateToggle(AntiBanPage, "🎭 Ngụy Trang Chỉ Số (Spoof Humanoid)", Config.SpoofStats, function(state) Config.SpoofStats = state end)
CreateToggle(AntiBanPage, "🧹 Khóa Script Anti-Cheat Của Game", Config.DisableClientAC, function(state) Config.DisableClientAC = state end)

----------------------------------------------------------
-- TAB 2: COMBAT VIP
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
    if #plrs <= 1 then
        Notify("Combat", "Không có đối thủ khác trong server!")
        return
    end
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

CreateToggle(CombatPage, "⚡ M1 Multi-Hit Super Damage", Config.SuperM1Damage, function(state) Config.SuperM1Damage = state end)
CreateValueAdjuster(CombatPage, "Sức Mạnh Nhân Hit M1", 5, 80, Config.DamageMultiplier, 5, function(val) Config.DamageMultiplier = val end)
CreateToggle(CombatPage, "📦 Phóng To Hitbox Kẻ Địch", Config.HitboxExpander, function(state) Config.HitboxExpander = state end)
CreateValueAdjuster(CombatPage, "Kích Thước Hitbox", 5, 35, Config.HitboxSize, 5, function(val) Config.HitboxSize = val end)
CreateToggle(CombatPage, "⚡ Auto TP Áp Sát Lưng Đối Thủ", Config.AutoTPTarget, function(state) Config.AutoTPTarget = state end)
CreateToggle(CombatPage, "🥊 Auto Đấm M1 Tự Động", Config.AutoAttack, function(state) Config.AutoAttack = state end)

----------------------------------------------------------
-- TAB 3: MOVEMENT
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
CreateToggle(MovementPage, "Đi Xuyên Tường Smooth (Noclip)", Config.NoclipEnabled, function(state) Config.NoclipEnabled = state end)
CreateToggle(MovementPage, "Chạy Nhanh (Speed Hack)", Config.SpeedEnabled, function(state) Config.SpeedEnabled = state end)
CreateValueAdjuster(MovementPage, "Tốc Độ Chạy", 16, 200, Config.SpeedValue, 10, function(val) Config.SpeedValue = val end)

----------------------------------------------------------
-- TAB 4: FIX LAG VIP
----------------------------------------------------------
local FList = Instance.new("UIListLayout", FixLagPage)
FList.SortOrder = Enum.SortOrder.LayoutOrder
FList.Padding = UDim.new(0, 8)

CreateButton(FixLagPage, "🚀 Kích Hoạt Max FPS (Siêu Nhẹ Map)", Color3.fromRGB(35, 55, 35), function()
    pcall(function()
        for _, v in ipairs(Lighting:GetChildren()) do
            if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("PostEffect") or v:IsA("BlurEffect") then v:Destroy() end
        end
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 9e9
        Lighting.Brightness = 2
        
        for _, v in ipairs(Workspace:GetDescendants()) do
            if v:IsA("BasePart") then
                v.Material = Enum.Material.SmoothPlastic
                v.CastShadow = false
            elseif v:IsA("Decal") or v:IsA("Texture") then
                v.Transparency = 1
            elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then
                v.Enabled = false
            end
        end
    end)
    Notify("Fix Lag VIP", "Đã bật Max FPS thành công! 🚀")
end)

CreateToggle(FixLagPage, "🥔 Chế Độ Potato Graphics", Config.PotatoMode, function(state)
    Config.PotatoMode = state
    pcall(function()
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
        for _, v in ipairs(Workspace:GetDescendants()) do
            if v:IsA("BasePart") then
                v.Material = state and Enum.Material.SmoothPlastic or Enum.Material.Plastic
                v.CastShadow = not state
            end
        end
    end)
end)

CreateToggle(FixLagPage, "🧹 Auto Dọn Dẹp RAM Ngầm", Config.AutoMemoryClean, function(state) Config.AutoMemoryClean = state end)
CreateToggle(FixLagPage, "👤 Ẩn Người Chơi Khác (Hide Players)", Config.HidePlayers, function(state)
    Config.HidePlayers = state
    pcall(function()
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
end)

----------------------------------------------------------
-- TAB 5: SETTINGS & CHILL BACKGROUND CUSTOM
----------------------------------------------------------
local SList = Instance.new("UIListLayout", SettingsPage)
SList.SortOrder = Enum.SortOrder.LayoutOrder
SList.Padding = UDim.new(0, 8)

CreateButton(SettingsPage, "🌄 Đổi Ảnh Background Chill (Preset 1 - 10)", Color3.fromRGB(45, 30, 55), function()
    Config.CurrentChillIndex = (Config.CurrentChillIndex % #ChillPresets) + 1
    MainBgImage.Image = ChillPresets[Config.CurrentChillIndex]
    Notify("Chill Bg", "Đã chuyển sang Nền Chill #" .. Config.CurrentChillIndex .. "/" .. #ChillPresets .. " ☕")
end)

-- Ô NHẬP CUSTOM IMAGE ID
local CustomIdContainer = Instance.new("Frame")
CustomIdContainer.Size = UDim2.new(1, -6, 0, 36)
CustomIdContainer.BackgroundColor3 = CurrentTheme.Button
CustomIdContainer.BackgroundTransparency = 0.25
CustomIdContainer.BorderSizePixel = 0
CustomIdContainer.ZIndex = 4
CustomIdContainer.Parent = SettingsPage
Instance.new("UICorner", CustomIdContainer).CornerRadius = UDim.new(0, 6)

local CustomIdBox = Instance.new("TextBox")
CustomIdBox.Size = UDim2.new(1, -80, 1, 0)
CustomIdBox.Position = UDim2.new(0, 10, 0, 0)
CustomIdBox.PlaceholderText = "Nhập ID Ảnh Roblox (Ví dụ: 6071575925)..."
CustomIdBox.Text = ""
CustomIdBox.TextColor3 = CurrentTheme.Text
CustomIdBox.PlaceholderColor3 = Color3.fromRGB(160, 150, 175)
CustomIdBox.Font = Enum.Font.SourceSans
CustomIdBox.TextSize = 12
CustomIdBox.TextXAlignment = Enum.TextXAlignment.Left
CustomIdBox.BackgroundTransparency = 1
CustomIdBox.ZIndex = 5
CustomIdBox.Parent = CustomIdContainer

local ApplyIdBtn = Instance.new("TextButton")
ApplyIdBtn.Size = UDim2.new(0, 60, 0, 26)
ApplyIdBtn.Position = UDim2.new(1, -66, 0.5, -13)
ApplyIdBtn.Text = "Đổi Ảnh"
ApplyIdBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ApplyIdBtn.Font = Enum.Font.SourceSansBold
ApplyIdBtn.TextSize = 12
ApplyIdBtn.BackgroundColor3 = CurrentTheme.Accent
ApplyIdBtn.BorderSizePixel = 0
ApplyIdBtn.ZIndex = 5
ApplyIdBtn.Parent = CustomIdContainer
Instance.new("UICorner", ApplyIdBtn).CornerRadius = UDim.new(0, 4)

ApplyIdBtn.MouseButton1Click:Connect(function()
    local input = CustomIdBox.Text:gsub("%D", "")
    if #input > 3 then
        MainBgImage.Image = "rbxassetid://" .. input
        Notify("Background", "Đã áp dụng Custom Image ID: " .. input)
    else
        Notify("Background Lỗi", "Vui lòng nhập ID hợp lệ!")
    end
end)

-- NÚT RESET KHÔI PHÚC ẢNH BACKGROUND MẶC ĐỊNH
CreateButton(SettingsPage, "🔄 Reset Ảnh Background Về Mặc Định", Color3.fromRGB(55, 30, 35), function()
    Config.CurrentChillIndex = 1
    MainBgImage.Image = ChillPresets[1]
    CustomIdBox.Text = ""
    Notify("Background Reset", "Đã khôi phục ảnh background về mặc định ban đầu! ☕✨")
end)

CreateButton(SettingsPage, "🎨 Đổi Tone Màu Theme (Purple/Pink/Ocean/Mint)", Color3.fromRGB(35, 30, 48), function()
    if CurrentTheme == Themes.ChillPurple then CurrentTheme = Themes.SoftPink
    elseif CurrentTheme == Themes.SoftPink then CurrentTheme = Themes.OceanBlue
    elseif CurrentTheme == Themes.OceanBlue then CurrentTheme = Themes.MintGreen
    else CurrentTheme = Themes.ChillPurple end

    MainFrame.BackgroundColor3 = CurrentTheme.Bg
    Header.BackgroundColor3 = CurrentTheme.Sidebar
    Sidebar.BackgroundColor3 = CurrentTheme.Sidebar
    Title.TextColor3 = CurrentTheme.Accent
    MainStroke.Color = CurrentTheme.Accent
    ToggleBtn.BackgroundColor3 = CurrentTheme.Sidebar
    ButtonStroke.Color = CurrentTheme.Accent
    TargetLabel.TextColor3 = CurrentTheme.Accent
    ApplyIdBtn.BackgroundColor3 = CurrentTheme.Accent

    for _, btn in ipairs(TabButtons) do btn.BackgroundColor3 = CurrentTheme.Button end
    Notify("Settings", "Đã đổi Theme: " .. CurrentTheme.Name)
end)

CreateToggle(SettingsPage, "Chống Treo Máy (Anti-AFK 24/7)", Config.AntiAFK, function(state) Config.AntiAFK = state end)
CreateButton(SettingsPage, "🔄 Vào Lại Server (Rejoin)", Color3.fromRGB(45, 25, 35), function()
    TeleportService:Teleport(game.PlaceId, LocalPlayer)
end)

----------------------------------------------------------
-- SYSTEM LOOPS & CORE ENGINE
----------------------------------------------------------
local lastM1Tick = 0
local function ExecuteM1KillProtocol()
    if tick() - lastM1Tick < 0.04 then return end
    lastM1Tick = tick()
    local char = LocalPlayer.Character
    if not char then return end
    local cam = workspace.CurrentCamera
    local tool = char:FindFirstChildOfClass("Tool")
    local hits = Config.SuperM1Damage and Config.DamageMultiplier or 1

    task.spawn(function()
        for i = 1, hits do
            if VirtualUser then
                pcall(function()
                    VirtualUser:Button1Down(Vector2.new(0,0), cam.CFrame)
                    VirtualUser:Button1Up(Vector2.new(0,0), cam.CFrame)
                end)
            end
            if tool then
                pcall(function() tool:Activate() end)
            end
        end
    end)
end

local lastTime = tick()
local frameCount = 0
local memoryTimer = 0

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

    if Config.AutoMemoryClean then
        memoryTimer = memoryTimer + dt
        if memoryTimer >= 20 then
            memoryTimer = 0
            pcall(function() collectgarbage("collect") end)
        end
    end

    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")

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

    if Config.SelectedTarget and Config.SelectedTarget.Character then
        local tHRP = Config.SelectedTarget.Character:FindFirstChild("HumanoidRootPart")
        local tHum = Config.SelectedTarget.Character:FindFirstChildOfClass("Humanoid")
        if tHRP and tHum and tHum.Health > 0 and hrp then
            if Config.AutoTPTarget then hrp.CFrame = tHRP.CFrame * CFrame.new(0, 0, 2.2) end
            if Config.AutoAttack or Config.SuperM1Damage then ExecuteM1KillProtocol() end
        end
    end

    if Config.SpeedEnabled and hum then hum.WalkSpeed = Config.SpeedValue end
end)

LocalPlayer.Idled:Connect(function()
    if Config.AntiAFK and VirtualUser then
        pcall(function()
            VirtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
            task.wait(1)
            VirtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        end)
    end
end)

Notify("Kianbest Hub", "Đã tải v12.7! Đã thêm nút Reset Background Mặc Định ☕✨")
