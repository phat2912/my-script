-- ==========================================================
-- KIANBEST HUB V17.1 - FIXED iOS DYNAMIC ISLAND EDITION
-- Fix: Đẩy Notification lên sát đỉnh màn hình (DisplayOrder 999999)
-- Fix: Hiệu ứng Fade Out mờ dần & thu nhỏ trơn tru không bị đè khung
-- Feature: Bất Tử GOD MODE, Hitbox VIP, Fly 3D, Super M1, Fix Lag
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
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer

local VirtualUser
pcall(function() VirtualUser = game:GetService("VirtualUser") end)

-- CẤU HÌNH HỆ THỐNG
local Config = {
    AntiBanEnabled = true,
    AntiKick = true,
    AntiLog = true,
    SpoofStats = true,
    AntiAdmin = true,
    AutoHopOnAdmin = true,

    GodMode = false,
    NoclipEnabled = false,
    SpeedEnabled = false,
    SpeedValue = 35,
    FlyEnabled = false,
    FlySpeed = 75,
    
    SelectedTarget = nil,
    AutoLockNearest = false,
    AutoTPTarget = false,
    AutoAttack = false,
    SuperM1Damage = false,
    DamageMultiplier = 25,
    HitboxExpander = false,
    HitboxSize = 25,
    
    GreySkyMode = false,
    PotatoMode = false,
    AutoMemoryClean = true,
    
    AntiAFK = true,
    BgTransparency = 0.2,
    FrameTransparency = 0.15,
    ToggleKey = Enum.KeyCode.RightControl
}

local Themes = {
    ChillPurple = { Name = "Chill Lavender 🔮", Bg = Color3.fromRGB(18, 16, 26), Sidebar = Color3.fromRGB(12, 10, 18), Accent = Color3.fromRGB(165, 120, 255), Button = Color3.fromRGB(28, 22, 38), Text = Color3.fromRGB(240, 235, 255) },
    SoftPink = { Name = "Soft Sakura 🌸", Bg = Color3.fromRGB(24, 16, 22), Sidebar = Color3.fromRGB(16, 10, 15), Accent = Color3.fromRGB(255, 140, 180), Button = Color3.fromRGB(36, 22, 32), Text = Color3.fromRGB(255, 240, 248) },
    OceanBlue = { Name = "Midnight Ocean 🌊", Bg = Color3.fromRGB(12, 18, 28), Sidebar = Color3.fromRGB(8, 12, 20), Accent = Color3.fromRGB(90, 185, 255), Button = Color3.fromRGB(20, 30, 44), Text = Color3.fromRGB(235, 248, 255) },
    MintGreen = { Name = "Chill Matcha 🍃", Bg = Color3.fromRGB(14, 22, 18), Sidebar = Color3.fromRGB(8, 15, 12), Accent = Color3.fromRGB(110, 220, 160), Button = Color3.fromRGB(20, 34, 28), Text = Color3.fromRGB(235, 255, 242) }
}

local CurrentTheme = Themes.ChillPurple
local ParentGui = (gethui and gethui()) or CoreGui or LocalPlayer:WaitForChild("PlayerGui")

if ParentGui:FindFirstChild("KianbestMenuV17_1") then
    ParentGui.KianbestMenuV17_1:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KianbestMenuV17_1"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = ParentGui

----------------------------------------------------------
-- 📲 IOS DYNAMIC ISLAND NOTIFICATION SYSTEM (FIXED V2.2)
----------------------------------------------------------
local NotiGui = ParentGui:FindFirstChild("iOSNotiGui")
if NotiGui then NotiGui:Destroy() end

NotiGui = Instance.new("ScreenGui")
NotiGui.Name = "iOSNotiGui"
NotiGui.DisplayOrder = 999999 -- Đặt đè lên mọi GUI khác
NotiGui.IgnoreGuiInset = true -- Sát mép màn hình chuẩn Dynamic Island
NotiGui.ResetOnSpawn = false
NotiGui.Parent = ParentGui

local NotiContainer = Instance.new("Frame")
NotiContainer.Name = "iOSNotiContainer"
NotiContainer.Size = UDim2.new(0, 280, 0, 300)
NotiContainer.Position = UDim2.new(0.5, -140, 0, 10)
NotiContainer.BackgroundTransparency = 1
NotiContainer.ZIndex = 999999
NotiContainer.Parent = NotiGui

local NotiLayout = Instance.new("UIListLayout", NotiContainer)
NotiLayout.SortOrder = Enum.SortOrder.LayoutOrder
NotiLayout.Padding = UDim.new(0, 6)
NotiLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
NotiLayout.VerticalAlignment = Enum.VerticalAlignment.Top

local ActiveNotifications = {}
local MAX_NOTIFICATIONS = 3

local iOSColors = {
    Success = Color3.fromRGB(48, 209, 88),   -- Xanh Lục iOS
    Error   = Color3.fromRGB(255, 69, 58),   -- Đỏ iOS
    Info    = Color3.fromRGB(10, 132, 255),  -- Xanh Dương iOS
    Warning = Color3.fromRGB(255, 159, 10),  -- Cam iOS
    Bg      = Color3.fromRGB(18, 18, 22),    -- Nền Cyber Dark
    Text    = Color3.fromRGB(255, 255, 255)
}

local function Notify(title, text, duration)
    duration = duration or 2.0
    
    local statusColor = iOSColors.Info
    local checkStr = (tostring(title) .. " " .. tostring(text)):lower()
    
    if checkStr:find("bật") or checkStr:find("on") or checkStr:find("🟢") or checkStr:find("thành công") or checkStr:find("kích hoạt") then
        statusColor = iOSColors.Success
    elseif checkStr:find("tắt") or checkStr:find("off") or checkStr:find("🔴") or checkStr:find("chặn") or checkStr:find("lỗi") then
        statusColor = iOSColors.Error
    elseif checkStr:find("cảnh báo") or checkStr:find("⚠️") then
        statusColor = iOSColors.Warning
    end

    while #ActiveNotifications >= MAX_NOTIFICATIONS do
        local oldest = table.remove(ActiveNotifications, 1)
        if oldest and oldest.Frame and oldest.Frame.Parent then
            oldest.Frame:Destroy()
        end
    end

    local Toast = Instance.new("Frame")
    Toast.Name = "iOSToast"
    Toast.Size = UDim2.new(1, 0, 0, 42)
    Toast.BackgroundColor3 = iOSColors.Bg
    Toast.BackgroundTransparency = 1
    Toast.BorderSizePixel = 0
    Toast.ClipsDescendants = true
    Toast.ZIndex = 999999
    Toast.Parent = NotiContainer

    Instance.new("UICorner", Toast).CornerRadius = UDim.new(0, 12)

    local Stroke = Instance.new("UIStroke", Toast)
    Stroke.Color = statusColor
    Stroke.Thickness = 1.2
    Stroke.Transparency = 1

    local Dot = Instance.new("Frame")
    Dot.Size = UDim2.new(0, 8, 0, 8)
    Dot.Position = UDim2.new(0, 12, 0.5, -4)
    Dot.BackgroundColor3 = statusColor
    Dot.BorderSizePixel = 0
    Dot.BackgroundTransparency = 1
    Dot.ZIndex = 1000000
    Dot.Parent = Toast
    Instance.new("UICorner", Dot).CornerRadius = UDim.new(1, 0)

    local TTitle = Instance.new("TextLabel")
    TTitle.Size = UDim2.new(1, -30, 0, 16)
    TTitle.Position = UDim2.new(0, 26, 0, 5)
    TTitle.Text = title or "Thông báo"
    TTitle.TextColor3 = iOSColors.Text
    TTitle.Font = Enum.Font.SourceSansBold
    TTitle.TextSize = 13
    TTitle.TextXAlignment = Enum.TextXAlignment.Left
    TTitle.BackgroundTransparency = 1
    TTitle.TextTransparency = 1
    TTitle.ZIndex = 1000000
    TTitle.Parent = Toast

    local TText = Instance.new("TextLabel")
    TText.Size = UDim2.new(1, -30, 0, 14)
    TText.Position = UDim2.new(0, 26, 0, 21)
    TText.Text = text or ""
    TText.TextColor3 = statusColor
    TText.Font = Enum.Font.SourceSansSemiBold
    TText.TextSize = 12
    TText.TextXAlignment = Enum.TextXAlignment.Left
    TText.BackgroundTransparency = 1
    TText.TextTransparency = 1
    TText.ZIndex = 1000000
    TText.Parent = Toast

    local notiData = {Frame = Toast}
    table.insert(ActiveNotifications, notiData)

    Toast.Size = UDim2.new(0.5, 0, 0, 0)
    local tweenInInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
    TweenService:Create(Toast, tweenInInfo, {Size = UDim2.new(1, 0, 0, 42), BackgroundTransparency = 0.1}):Play()
    TweenService:Create(Stroke, tweenInInfo, {Transparency = 0.3}):Play()
    TweenService:Create(Dot, tweenInInfo, {BackgroundTransparency = 0}):Play()
    TweenService:Create(TTitle, tweenInInfo, {TextTransparency = 0}):Play()
    TweenService:Create(TText, tweenInInfo, {TextTransparency = 0}):Play()

    task.delay(duration, function()
        pcall(function()
            if not Toast or not Toast.Parent then return end
            local tweenOutInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
            
            TweenService:Create(Toast, tweenOutInfo, {Size = UDim2.new(0.5, 0, 0, 0), BackgroundTransparency = 1}):Play()
            TweenService:Create(Stroke, tweenOutInfo, {Transparency = 1}):Play()
            TweenService:Create(Dot, tweenOutInfo, {BackgroundTransparency = 1}):Play()
            TweenService:Create(TTitle, tweenOutInfo, {TextTransparency = 1}):Play()
            local lastTween = TweenService:Create(TText, tweenOutInfo, {TextTransparency = 1})
            lastTween:Play()

            lastTween.Completed:Connect(function()
                for idx, item in ipairs(ActiveNotifications) do
                    if item == notiData then
                        table.remove(ActiveNotifications, idx)
                        break
                    end
                end
                Toast:Destroy()
            end)
        end)
    end)
end

----------------------------------------------------------
-- 🛡️ ANTI-BAN SHIELD VIP
----------------------------------------------------------
pcall(function()
    if not Config.AntiBanEnabled then return end

    if hookmetamethod then
        local oldNamecall
        oldNamecall = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
            local method = getnamecallmethod()
            if Config.AntiKick and (method == "Kick" or method == "kick") and self == LocalPlayer then
                Notify("🛡️ Anti-Ban VIP", "Đã chặn 1 lệnh KICK!")
                return nil
            end
            if Config.AntiLog and (method == "FireServer" or method == "InvokeServer") then
                local remoteName = tostring(self):lower()
                local blockKeywords = {"ban", "kick", "flag", "cheat", "detect", "log", "ac", "security", "adonis", "anticheat"}
                for _, word in ipairs(blockKeywords) do
                    if remoteName:find(word) then return nil end
                end
            end
            return oldNamecall(self, ...)
        end))

        local oldIndex
        oldIndex = hookmetamethod(game, "__index", newcclosure(function(self, key)
            if Config.SpoofStats and not checkcaller() and self:IsA("Humanoid") then
                if key == "WalkSpeed" then return 16 end
                if key == "JumpPower" then return 50 end
            end
            return oldIndex(self, key)
        end))
    end
end)

----------------------------------------------------------
-- MAIN FRAME & GUI UI
----------------------------------------------------------
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 520, 0, 340)
MainFrame.Position = UDim2.new(0.5, -260, 0.5, -170)
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

local BgImage = Instance.new("ImageLabel")
BgImage.Name = "BgImage"
BgImage.Size = UDim2.new(1, 0, 1, 0)
BgImage.BackgroundTransparency = 1
BgImage.ImageTransparency = Config.BgTransparency
BgImage.ScaleType = Enum.ScaleType.Crop
BgImage.ZIndex = 1
BgImage.Parent = MainFrame
Instance.new("UICorner", BgImage).CornerRadius = UDim.new(0, 12)

local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 36)
Header.BackgroundColor3 = CurrentTheme.Sidebar
Header.BackgroundTransparency = 0.2
Header.BorderSizePixel = 0
Header.ZIndex = 3
Header.Parent = MainFrame
Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 12)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0, 300, 1, 0)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.Text = "⚡ Kianbest Hub v17.1 iOS EDITION"
Title.TextColor3 = CurrentTheme.Accent
Title.TextSize = 13
Title.Font = Enum.Font.FredokaOne
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1
Title.ZIndex = 4
Title.Parent = Header

local StatsLabel = Instance.new("TextLabel")
StatsLabel.Size = UDim2.new(0, 180, 1, 0)
StatsLabel.Position = UDim2.new(1, -210, 0, 0)
StatsLabel.Text = "FPS: -- | Ping: --ms"
StatsLabel.TextColor3 = Color3.fromRGB(220, 210, 235)
StatsLabel.TextSize = 11
StatsLabel.Font = Enum.Font.SourceSansBold
StatsLabel.TextXAlignment = Enum.TextXAlignment.Right
StatsLabel.BackgroundTransparency = 1
StatsLabel.ZIndex = 4
StatsLabel.Parent = Header

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 24, 0, 24)
CloseBtn.Position = UDim2.new(1, -28, 0, 6)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 120, 140)
CloseBtn.TextSize = 12
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 22, 32)
CloseBtn.BorderSizePixel = 0
CloseBtn.ZIndex = 4
CloseBtn.Parent = Header
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Name = "KianToggleButton"
ToggleBtn.Size = UDim2.new(0, 44, 0, 44)
ToggleBtn.Position = UDim2.new(0, 15, 0.4, 0)
ToggleBtn.BackgroundColor3 = CurrentTheme.Sidebar
ToggleBtn.BackgroundTransparency = 0.2
ToggleBtn.Text = "⚡"
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
        TweenService:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 520, 0, 340)}):Play()
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
-- SIDEBAR & PAGES
----------------------------------------------------------
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 130, 1, -36)
Sidebar.Position = UDim2.new(0, 0, 0, 36)
Sidebar.BackgroundColor3 = CurrentTheme.Sidebar
Sidebar.BackgroundTransparency = 0.3
Sidebar.BorderSizePixel = 0
Sidebar.ZIndex = 3
Sidebar.Parent = MainFrame

local ContentContainer = Instance.new("Frame")
ContentContainer.Size = UDim2.new(1, -138, 1, -44)
ContentContainer.Position = UDim2.new(0, 134, 0, 40)
ContentContainer.BackgroundTransparency = 1
ContentContainer.ZIndex = 3
ContentContainer.Parent = MainFrame

local Pages = {}
local TabButtons = {}

local function CreateTab(name, icon, posIndex)
    local TabBtn = Instance.new("TextButton")
    TabBtn.Size = UDim2.new(1, -10, 0, 32)
    TabBtn.Position = UDim2.new(0, 5, 0, 6 + (posIndex - 1) * 36)
    TabBtn.Text = icon .. "  " .. name
    TabBtn.TextColor3 = CurrentTheme.Text
    TabBtn.Font = Enum.Font.SourceSansBold
    TabBtn.TextSize = 12
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
    Page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    Page.CanvasSize = UDim2.new(0, 0, 0, 0)
    Page.Parent = ContentContainer

    local PList = Instance.new("UIListLayout", Page)
    PList.SortOrder = Enum.SortOrder.LayoutOrder
    PList.Padding = UDim.new(0, 6)

    local PPadding = Instance.new("UIPadding", Page)
    PPadding.PaddingTop = UDim.new(0, 4)
    PPadding.PaddingBottom = UDim.new(0, 10)
    PPadding.PaddingRight = UDim.new(0, 6)

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

local AntiBanPage = CreateTab("Anti-Ban Shield", "🛡", 1)
local CombatPage = CreateTab("Combat VIP", "⚔", 2)
local MovementPage = CreateTab("Movement", "⚡", 3)
local FixLagPage = CreateTab("Fix Lag VIP", "🚀", 4)
local SettingsPage = CreateTab("Settings & Chill", "☕", 5)

----------------------------------------------------------
-- UI BUILDERS
----------------------------------------------------------
local function CreateToggle(parent, text, defaultState, callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, 0, 0, 32)
    Btn.Text = "  " .. text
    Btn.TextColor3 = CurrentTheme.Text
    Btn.Font = Enum.Font.SourceSansBold
    Btn.TextSize = 12
    Btn.TextXAlignment = Enum.TextXAlignment.Left
    Btn.BackgroundColor3 = CurrentTheme.Button
    Btn.BackgroundTransparency = 0.25
    Btn.BorderSizePixel = 0
    Btn.ZIndex = 4
    Btn.Parent = parent
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 6)

    local StatusInd = Instance.new("Frame")
    StatusInd.Size = UDim2.new(0, 26, 0, 14)
    StatusInd.Position = UDim2.new(1, -32, 0.5, -7)
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
        cleanTitle = (cleanTitle and cleanTitle ~= "") and cleanTitle or "Tính năng"
        Notify(cleanTitle, state and "Đã BẬT 🟢" or "Đã TẮT 🔴")
        callback(state)
    end)
    return Btn
end

local function CreateValueAdjuster(parent, title, minVal, maxVal, defaultVal, step, callback)
    local Container = Instance.new("Frame")
    Container.Size = UDim2.new(1, 0, 0, 34)
    Container.BackgroundColor3 = CurrentTheme.Button
    Container.BackgroundTransparency = 0.25
    Container.BorderSizePixel = 0
    Container.ZIndex = 4
    Container.Parent = parent
    Instance.new("UICorner", Container).CornerRadius = UDim.new(0, 6)

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.6, 0, 1, 0)
    Label.Position = UDim2.new(0, 8, 0, 0)
    Label.Text = title .. ": " .. tostring(defaultVal)
    Label.TextColor3 = CurrentTheme.Text
    Label.Font = Enum.Font.SourceSansBold
    Label.TextSize = 12
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.BackgroundTransparency = 1
    Label.ZIndex = 5
    Label.Parent = Container

    local current = defaultVal
    local MinusBtn = Instance.new("TextButton")
    MinusBtn.Size = UDim2.new(0, 24, 0, 20)
    MinusBtn.Position = UDim2.new(1, -56, 0.5, -10)
    MinusBtn.Text = "-"
    MinusBtn.TextColor3 = Color3.fromRGB(240, 240, 240)
    MinusBtn.Font = Enum.Font.SourceSansBold
    MinusBtn.TextSize = 14
    MinusBtn.BackgroundColor3 = Color3.fromRGB(50, 35, 45)
    MinusBtn.BorderSizePixel = 0
    MinusBtn.ZIndex = 5
    MinusBtn.Parent = Container
    Instance.new("UICorner", MinusBtn).CornerRadius = UDim.new(0, 4)

    local PlusBtn = Instance.new("TextButton")
    PlusBtn.Size = UDim2.new(0, 24, 0, 20)
    PlusBtn.Position = UDim2.new(1, -28, 0.5, -10)
    PlusBtn.Text = "+"
    PlusBtn.TextColor3 = Color3.fromRGB(240, 240, 240)
    PlusBtn.Font = Enum.Font.SourceSansBold
    PlusBtn.TextSize = 14
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
    Btn.Size = UDim2.new(1, 0, 0, 32)
    Btn.Text = text
    Btn.TextColor3 = Color3.fromRGB(245, 245, 245)
    Btn.Font = Enum.Font.SourceSansBold
    Btn.TextSize = 12
    Btn.BackgroundColor3 = bgColor or CurrentTheme.Button
    Btn.BackgroundTransparency = 0.2
    Btn.BorderSizePixel = 0
    Btn.ZIndex = 4
    Btn.Parent = parent
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 6)
    Btn.MouseButton1Click:Connect(callback)
    return Btn
end

local function CreateTextBox(parent, placeholder, callback)
    local BoxFrame = Instance.new("Frame")
    BoxFrame.Size = UDim2.new(1, 0, 0, 32)
    BoxFrame.BackgroundColor3 = CurrentTheme.Button
    BoxFrame.BackgroundTransparency = 0.25
    BoxFrame.BorderSizePixel = 0
    BoxFrame.ZIndex = 4
    BoxFrame.Parent = parent
    Instance.new("UICorner", BoxFrame).CornerRadius = UDim.new(0, 6)

    local TBox = Instance.new("TextBox")
    TBox.Size = UDim2.new(1, -12, 1, 0)
    TBox.Position = UDim2.new(0, 6, 0, 0)
    TBox.PlaceholderText = placeholder
    TBox.Text = ""
    TBox.TextColor3 = CurrentTheme.Text
    TBox.PlaceholderColor3 = Color3.fromRGB(150, 140, 165)
    TBox.Font = Enum.Font.SourceSansBold
    TBox.TextSize = 11
    TBox.BackgroundTransparency = 1
    TBox.ZIndex = 5
    TBox.Parent = BoxFrame

    TBox.FocusLost:Connect(function(enterPressed)
        if enterPressed and TBox.Text ~= "" then
            callback(TBox.Text)
        end
    end)
end

----------------------------------------------------------
-- TAB 1: ANTI-BAN SHIELD
----------------------------------------------------------
CreateToggle(AntiBanPage, "🛡️ Chống Kick Tối Đa (Anti-Kick Interceptor)", Config.AntiKick, function(state) Config.AntiKick = state end)
CreateToggle(AntiBanPage, "🚫 Chặn Gửi Log/Report Cho Game", Config.AntiLog, function(state) Config.AntiLog = state end)
CreateToggle(AntiBanPage, "🎭 Ngụy Trang Chỉ Số (Spoof Humanoid Stats)", Config.SpoofStats, function(state) Config.SpoofStats = state end)
CreateToggle(AntiBanPage, "👁️ Phát Hiện Admin/Mod (Anti-Staff Alert)", Config.AntiAdmin, function(state) Config.AntiAdmin = state end)
CreateToggle(AntiBanPage, "🏃 Auto Hop Server Khi Gặp Admin", Config.AutoHopOnAdmin, function(state) Config.AutoHopOnAdmin = state end)

----------------------------------------------------------
-- TAB 2: COMBAT VIP & GOD MODE
----------------------------------------------------------
local TargetLabel = Instance.new("TextLabel")
TargetLabel.Size = UDim2.new(1, 0, 0, 22)
TargetLabel.Text = "🎯 Target: Chưa chọn"
TargetLabel.TextColor3 = CurrentTheme.Accent
TargetLabel.Font = Enum.Font.SourceSansBold
TargetLabel.TextSize = 12
TargetLabel.BackgroundTransparency = 1
TargetLabel.ZIndex = 4
TargetLabel.Parent = CombatPage

local function GetNearestPlayer()
    local closest, maxDist = nil, math.huge
    local myChar = LocalPlayer.Character
    local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myHRP then return nil end

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character then
            local pHRP = p.Character:FindFirstChild("HumanoidRootPart")
            local pHum = p.Character:FindFirstChildOfClass("Humanoid")
            if pHRP and pHum and pHum.Health > 0 then
                local dist = (pHRP.Position - myHRP.Position).Magnitude
                if dist < maxDist then
                    maxDist = dist
                    closest = p
                end
            end
        end
    end
    return closest, math.floor(maxDist)
end

CreateToggle(CombatPage, "👑 Bất Tử GOD MODE (Chống M1 & Chiêu)", Config.GodMode, function(state)
    Config.GodMode = state
    if not state and LocalPlayer.Character then
        for _, part in ipairs(LocalPlayer.Character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanTouch = true
            end
        end
    end
end)

CreateButton(CombatPage, "🎯 Tự Chọn Người Gần Nhất (Auto Nearest)", Color3.fromRGB(45, 25, 40), function()
    local target, dist = GetNearestPlayer()
    if target then
        Config.SelectedTarget = target
        Notify("Combat VIP", "Đã chọn target: " .. target.DisplayName .. " (" .. dist .. "m)")
    else
        Notify("Combat VIP", "Không tìm thấy người chơi ở gần!")
    end
end)

CreateToggle(CombatPage, "🔄 Tự Khóa Target Gần Nhất (Auto Lock)", Config.AutoLockNearest, function(state) Config.AutoLockNearest = state end)
CreateToggle(CombatPage, "⚡ Hit Dame To Mượt (Super Damage M1)", Config.SuperM1Damage, function(state) Config.SuperM1Damage = state end)
CreateValueAdjuster(CombatPage, "Số Hit Nhân Sát Thương", 5, 50, Config.DamageMultiplier, 5, function(val) Config.DamageMultiplier = val end)

CreateToggle(CombatPage, "📦 Hitbox Rộng (Fixed Move Bug)", Config.HitboxExpander, function(state)
    Config.HitboxExpander = state
    if not state then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character then
                local eHRP = p.Character:FindFirstChild("HumanoidRootPart")
                if eHRP then
                    eHRP.Size = Vector3.new(2, 2, 1)
                    eHRP.Transparency = 1
                    eHRP.CanCollide = false
                end
            end
        end
    end
end)
CreateValueAdjuster(CombatPage, "Kích Thước Hitbox Rộng", 5, 50, Config.HitboxSize, 5, function(val) Config.HitboxSize = val end)

CreateToggle(CombatPage, "⚡ Auto TP Áp Sát Lưng (Fixed Hitbox)", Config.AutoTPTarget, function(state) Config.AutoTPTarget = state end)
CreateToggle(CombatPage, "🥊 Auto Đấm M1 Tự Động", Config.AutoAttack, function(state) Config.AutoAttack = state end)

----------------------------------------------------------
-- TAB 3: MOVEMENT & ULTRA 3D FLY
----------------------------------------------------------
local flyVelocity, flyGyro

local function StopFlyEngine()
    if flyVelocity then flyVelocity:Destroy() flyVelocity = nil end
    if flyGyro then flyGyro:Destroy() flyGyro = nil end
    pcall(function()
        if LocalPlayer.Character then
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then hum.PlatformStand = false end
        end
        LocalPlayer.ReplicationFocus = nil
    end)
end

CreateToggle(MovementPage, "Bay 3D Smooth (Fixed Noti)", Config.FlyEnabled, function(state)
    Config.FlyEnabled = state
    if not state then StopFlyEngine() end
end)

CreateValueAdjuster(MovementPage, "Tốc Độ Bay 3D", 20, 250, Config.FlySpeed, 10, function(val) Config.FlySpeed = val end)
CreateToggle(MovementPage, "Đi Xuyên Tường Smooth (Noclip)", Config.NoclipEnabled, function(state) Config.NoclipEnabled = state end)
CreateToggle(MovementPage, "Chạy Nhanh (Speed Hack)", Config.SpeedEnabled, function(state) Config.SpeedEnabled = state end)
CreateValueAdjuster(MovementPage, "Tốc Độ Chạy", 16, 200, Config.SpeedValue, 10, function(val) Config.SpeedValue = val end)

----------------------------------------------------------
-- TAB 4: FIX LAG VIP
----------------------------------------------------------
local function ApplyGreySkyAndLighting()
    pcall(function()
        for _, v in ipairs(Lighting:GetChildren()) do
            if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("PostEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("ColorCorrectionEffect") then
                v:Destroy()
            end
        end

        local GreySky = Instance.new("Sky")
        GreySky.Name = "CleanGreySky"
        GreySky.SkyboxBk = "rbxassetid://152955518"
        GreySky.SkyboxDn = "rbxassetid://152955518"
        GreySky.SkyboxFt = "rbxassetid://152955518"
        GreySky.SkyboxLf = "rbxassetid://152955518"
        GreySky.SkyboxRt = "rbxassetid://152955518"
        GreySky.SkyboxUp = "rbxassetid://152955518"
        GreySky.Parent = Lighting

        Lighting.ClockTime = 12
        Lighting.GlobalShadows = false
        Lighting.Brightness = 1
        Lighting.OutdoorAmbient = Color3.fromRGB(130, 130, 130)
        Lighting.Ambient = Color3.fromRGB(130, 130, 130)
        Lighting.FogEnd = 9e9
    end)
end

CreateToggle(FixLagPage, "☁️ Bầu Trời Xám & Cân Bằng Ánh Sáng", Config.GreySkyMode, function(state)
    Config.GreySkyMode = state
    if state then ApplyGreySkyAndLighting() end
end)

CreateButton(FixLagPage, "🚀 Kích Hoạt Max FPS (Xóa Texture Map)", Color3.fromRGB(35, 55, 35), function()
    pcall(function()
        ApplyGreySkyAndLighting()
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
    Notify("Fix Lag VIP", "Đã tối ưu hóa Map Max FPS! 🚀")
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

----------------------------------------------------------
-- TAB 5: SETTINGS & CHILL
----------------------------------------------------------
CreateButton(SettingsPage, "🎨 Đổi Theme Màu (Purple/Pink/Ocean/Mint)", Color3.fromRGB(35, 30, 48), function()
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

    for _, btn in ipairs(TabButtons) do btn.BackgroundColor3 = CurrentTheme.Button end
    Notify("Settings", "Đã đổi Theme: " .. CurrentTheme.Name)
end)

CreateTextBox(SettingsPage, "Chèn ID Ảnh Background (VD: 6071575925)", function(text)
    local cleanedID = text:gsub("%D", "")
    if cleanedID ~= "" then
        BgImage.Image = "rbxassetid://" .. cleanedID
        Notify("Settings", "Đã chèn ảnh ID: " .. cleanedID)
    else
        Notify("Settings", "ID ảnh không hợp lệ!")
    end
end)

CreateValueAdjuster(SettingsPage, "Độ Trong Suốt Ảnh Nền", 0, 10, math.floor(Config.BgTransparency * 10), 1, function(val)
    Config.BgTransparency = val / 10
    BgImage.ImageTransparency = Config.BgTransparency
end)

CreateToggle(SettingsPage, "Chống Treo Máy (Anti-AFK 24/7)", Config.AntiAFK, function(state) Config.AntiAFK = state end)

CreateButton(SettingsPage, "🌐 Đổi Server Khác (Server Hop)", Color3.fromRGB(28, 42, 55), function()
    Notify("Server Hop", "Đang tìm server khác...", 3)
    pcall(function()
        local sfUrl = "https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
        local req = game:HttpGet(sfUrl)
        local data = HttpService:JSONDecode(req)
        if data and data.data then
            for _, s in ipairs(data.data) do
                if s.id ~= game.JobId and s.playing < s.maxPlayers then
                    TeleportService:TeleportToPlaceInstance(game.PlaceId, s.id, LocalPlayer)
                    return
                end
            end
        end
        TeleportService:Teleport(game.PlaceId, LocalPlayer)
    end)
end)

CreateButton(SettingsPage, "🔄 Vào Lại Server Này (Rejoin)", Color3.fromRGB(45, 25, 35), function()
    TeleportService:Teleport(game.PlaceId, LocalPlayer)
end)

CreateButton(SettingsPage, "🗑️ Tắt Menu Hoàn Toàn (Unload GUI)", Color3.fromRGB(60, 20, 25), function()
    StopFlyEngine()
    if NotiGui then NotiGui:Destroy() end
    ScreenGui:Destroy()
end)

----------------------------------------------------------
-- LOOPS ENGINE & CORE LOGIC
----------------------------------------------------------
local lastM1Tick = 0
local function ExecuteM1KillProtocol()
    if tick() - lastM1Tick < 0.05 then return end
    lastM1Tick = tick()

    local char = LocalPlayer.Character
    if not char then return end
    local cam = Workspace.CurrentCamera
    local tool = char:FindFirstChildOfClass("Tool")
    local hits = Config.SuperM1Damage and Config.DamageMultiplier or 1

    task.spawn(function()
        for i = 1, math.min(hits, 35) do
            if tool then pcall(function() tool:Activate() end) end
            if VirtualUser then
                pcall(function()
                    VirtualUser:Button1Down(Vector2.new(0,0), cam.CFrame)
                    VirtualUser:Button1Up(Vector2.new(0,0), cam.CFrame)
                end)
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
    local cam = Workspace.CurrentCamera

    if Config.GodMode and char then
        for _, part in ipairs(char:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanTouch = false
            end
        end
    end

    if Config.AutoLockNearest then
        local nearPlr = GetNearestPlayer()
        if nearPlr then Config.SelectedTarget = nearPlr end
    end

    if Config.SelectedTarget and Config.SelectedTarget.Character then
        local tHum = Config.SelectedTarget.Character:FindFirstChildOfClass("Humanoid")
        local tHRP = Config.SelectedTarget.Character:FindFirstChild("HumanoidRootPart")
        if tHum and tHRP and hrp then
            local hp = math.floor(tHum.Health)
            local dist = math.floor((tHRP.Position - hrp.Position).Magnitude)
            TargetLabel.Text = "🎯 Target: " .. Config.SelectedTarget.DisplayName .. " [" .. hp .. " HP] (" .. dist .. "m)"
        else
            TargetLabel.Text = "🎯 Target: " .. Config.SelectedTarget.DisplayName .. " [Đã Chết]"
        end
    else
        TargetLabel.Text = "🎯 Target: Chưa chọn"
    end

    if Config.FlyEnabled and hrp and hum and cam then
        hum.PlatformStand = true
        LocalPlayer.ReplicationFocus = hrp
        
        if not flyVelocity or flyVelocity.Parent ~= hrp then
            flyVelocity = Instance.new("BodyVelocity")
            flyVelocity.MaxForce = Vector3.new(1e8, 1e8, 1e8)
            flyVelocity.Parent = hrp
        end
        
        if not flyGyro or flyGyro.Parent ~= hrp then
            flyGyro = Instance.new("BodyGyro")
            flyGyro.MaxTorque = Vector3.new(1e8, 1e8, 1e8)
            flyGyro.P = 12000
            flyGyro.Parent = hrp
        end

        flyGyro.CFrame = cam.CFrame

        if hum.MoveDirection.Magnitude > 0 then
            local relMove = cam.CFrame:VectorToObjectSpace(hum.MoveDirection)
            local flyDir = (cam.CFrame.LookVector * -relMove.Z) + (cam.CFrame.RightVector * relMove.X)
            if flyDir.Magnitude > 0 then
                flyVelocity.Velocity = flyDir.Unit * Config.FlySpeed
            else
                flyVelocity.Velocity = Vector3.zero
            end
        else
            flyVelocity.Velocity = Vector3.zero
        end
    elseif not Config.FlyEnabled and flyVelocity then
        StopFlyEngine()
    end

    if Config.HitboxExpander then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character then
                local eHRP = p.Character:FindFirstChild("HumanoidRootPart")
                if eHRP then
                    eHRP.Size = Vector3.new(Config.HitboxSize, Config.HitboxSize, Config.HitboxSize)
                    eHRP.Transparency = 0.65
                    eHRP.Color = CurrentTheme.Accent
                    eHRP.Material = Enum.Material.Neon
                    eHRP.CanCollide = false
                    eHRP.Massless = true
                    eHRP.CanTouch = true
                end
            end
        end
    end

    if Config.SelectedTarget and Config.SelectedTarget.Character then
        local tHRP = Config.SelectedTarget.Character:FindFirstChild("HumanoidRootPart")
        local tHum = Config.SelectedTarget.Character:FindFirstChildOfClass("Humanoid")
        if tHRP and tHum and tHum.Health > 0 and hrp then
            if Config.AutoTPTarget then
                local behindPos = tHRP.Position - (tHRP.CFrame.LookVector * 2.5)
                hrp.CFrame = CFrame.lookAt(behindPos, tHRP.Position)
            end
            if Config.AutoAttack or Config.SuperM1Damage then ExecuteM1KillProtocol() end
        end
    end

    if Config.SpeedEnabled and hum then hum.WalkSpeed = Config.SpeedValue end
end)

LocalPlayer.Idled:Connect(function()
    if Config.AntiAFK and VirtualUser then
        pcall(function()
            VirtualUser:Button2Down(Vector2.new(0,0), Workspace.CurrentCamera.CFrame)
            task.wait(1)
            VirtualUser:Button2Up(Vector2.new(0,0), Workspace.CurrentCamera.CFrame)
        end)
    end
end)

Notify("Kianbest Hub", "Đã fix thông báo iOS mượt mà! 🟢🔴")
