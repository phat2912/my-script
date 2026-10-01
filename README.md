--==========================================================
-- KIANBEST HUB V9.6 - UI / PROTOTYPE
-- Mobile Friendly • Theme • Tabs • FPS/Ping • Notifications
-- Dùng cho Roblox Studio / game của bạn
--==========================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Stats = game:GetService("Stats")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

--==========================================================
-- CONFIG
--==========================================================

local Config = {
    MenuOpen = true,
    AntiAFK = false,
    CurrentTheme = 1,
    CurrentBackground = 1,
    ToggleKey = Enum.KeyCode.RightControl
}

local Themes = {
    {
        Name = "Cute Pink 🌸",
        Bg = Color3.fromRGB(30, 22, 30),
        Sidebar = Color3.fromRGB(22, 16, 23),
        Button = Color3.fromRGB(45, 30, 43),
        Accent = Color3.fromRGB(255, 120, 180),
        Text = Color3.fromRGB(255, 240, 247)
    },

    {
        Name = "Cyber Blue 💎",
        Bg = Color3.fromRGB(20, 27, 38),
        Sidebar = Color3.fromRGB(13, 19, 29),
        Button = Color3.fromRGB(28, 40, 57),
        Accent = Color3.fromRGB(80, 200, 255),
        Text = Color3.fromRGB(235, 248, 255)
    },

    {
        Name = "Purple 🔮",
        Bg = Color3.fromRGB(27, 21, 38),
        Sidebar = Color3.fromRGB(19, 14, 28),
        Button = Color3.fromRGB(41, 30, 55),
        Accent = Color3.fromRGB(190, 120, 255),
        Text = Color3.fromRGB(245, 238, 255)
    }
}

local Backgrounds = {
    "",
    "rbxassetid://11702739401",
    "rbxassetid://10023403248",
    "rbxassetid://6071575925",
    "rbxassetid://11414436906"
}

local Theme = Themes[Config.CurrentTheme]

--==========================================================
-- CLEAN OLD GUI
--==========================================================

local old = PlayerGui:FindFirstChild("KianbestHubV96")
if old then
    old:Destroy()
end

--==========================================================
-- GUI
--==========================================================

local Gui = Instance.new("ScreenGui")
Gui.Name = "KianbestHubV96"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true
Gui.Parent = PlayerGui

--==========================================================
-- NOTIFICATION
--==========================================================

local NotificationHolder = Instance.new("Frame")
NotificationHolder.Size = UDim2.new(0, 260, 0, 200)
NotificationHolder.Position = UDim2.new(1, -275, 1, -215)
NotificationHolder.BackgroundTransparency = 1
NotificationHolder.Parent = Gui

local NotificationLayout = Instance.new("UIListLayout")
NotificationLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom
NotificationLayout.Padding = UDim.new(0, 7)
NotificationLayout.Parent = NotificationHolder

local function Notify(title, message)
    local Toast = Instance.new("Frame")
    Toast.Size = UDim2.new(1, 0, 0, 55)
    Toast.BackgroundColor3 = Theme.Sidebar
    Toast.BorderSizePixel = 0
    Toast.Parent = NotificationHolder

    Instance.new("UICorner", Toast).CornerRadius = UDim.new(0, 10)

    local Stroke = Instance.new("UIStroke")
    Stroke.Color = Theme.Accent
    Stroke.Thickness = 1.5
    Stroke.Parent = Toast

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, -16, 0, 20)
    Title.Position = UDim2.new(0, 8, 0, 5)
    Title.BackgroundTransparency = 1
    Title.Text = title
    Title.TextColor3 = Theme.Accent
    Title.Font = Enum.Font.SourceSansBold
    Title.TextSize = 14
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.Parent = Toast

    local Text = Instance.new("TextLabel")
    Text.Size = UDim2.new(1, -16, 0, 22)
    Text.Position = UDim2.new(0, 8, 0, 27)
    Text.BackgroundTransparency = 1
    Text.Text = message
    Text.TextColor3 = Theme.Text
    Text.Font = Enum.Font.SourceSans
    Text.TextSize = 12
    Text.TextXAlignment = Enum.TextXAlignment.Left
    Text.Parent = Toast

    task.delay(2.5, function()
        if Toast.Parent then
            local t = TweenService:Create(
                Toast,
                TweenInfo.new(.3),
                {BackgroundTransparency = 1}
            )

            t:Play()

            TweenService:Create(
                Title,
                TweenInfo.new(.3),
                {TextTransparency = 1}
            ):Play()

            TweenService:Create(
                Text,
                TweenInfo.new(.3),
                {TextTransparency = 1}
            ):Play()

            t.Completed:Wait()
            Toast:Destroy()
        end
    end)
end

--==========================================================
-- MAIN FRAME
--==========================================================

local Main = Instance.new("Frame")
Main.Name = "Main"
Main.Size = UDim2.new(0.92, 0, 0, 390)
Main.Position = UDim2.new(0.04, 0, 0.5, -195)
Main.BackgroundColor3 = Theme.Bg
Main.BorderSizePixel = 0
Main.Active = true
Main.Parent = Gui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 14)
MainCorner.Parent = Main

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Theme.Accent
MainStroke.Thickness = 2
MainStroke.Parent = Main

local SizeConstraint = Instance.new("UISizeConstraint")
SizeConstraint.MinSize = Vector2.new(300, 330)
SizeConstraint.MaxSize = Vector2.new(650, 450)
SizeConstraint.Parent = Main

--==========================================================
-- BACKGROUND
--==========================================================

local Background = Instance.new("ImageLabel")
Background.Size = UDim2.fromScale(1, 1)
Background.BackgroundTransparency = 1
Background.ImageTransparency = 0.72
Background.ScaleType = Enum.ScaleType.Crop
Background.ZIndex = 0
Background.Parent = Main

Instance.new("UICorner", Background).CornerRadius = UDim.new(0, 14)

--==========================================================
-- HEADER
--==========================================================

local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 48)
Header.BackgroundColor3 = Theme.Sidebar
Header.BorderSizePixel = 0
Header.ZIndex = 2
Header.Parent = Main

Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 14)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0.55, 0, 1, 0)
Title.Position = UDim2.new(0, 14, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "🛡️ KIANBEST HUB V9.6"
Title.TextColor3 = Theme.Accent
Title.Font = Enum.Font.FredokaOne
Title.TextSize = 16
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.ZIndex = 3
Title.Parent = Header

local StatsLabel = Instance.new("TextLabel")
StatsLabel.Size = UDim2.new(0.35, 0, 1, 0)
StatsLabel.Position = UDim2.new(0.47, 0, 0, 0)
StatsLabel.BackgroundTransparency = 1
StatsLabel.Text = "FPS: -- | Ping: --"
StatsLabel.TextColor3 = Theme.Text
StatsLabel.Font = Enum.Font.SourceSansBold
StatsLabel.TextSize = 11
StatsLabel.TextXAlignment = Enum.TextXAlignment.Right
StatsLabel.ZIndex = 3
StatsLabel.Parent = Header

local Close = Instance.new("TextButton")
Close.Size = UDim2.new(0, 30, 0, 30)
Close.Position = UDim2.new(1, -38, 0, 9)
Close.Text = "✕"
Close.TextColor3 = Color3.fromRGB(255, 110, 130)
Close.TextSize = 15
Close.Font = Enum.Font.SourceSansBold
Close.BackgroundColor3 = Theme.Button
Close.BorderSizePixel = 0
Close.ZIndex = 4
Close.Parent = Header

Instance.new("UICorner", Close).CornerRadius = UDim.new(0, 8)

--==========================================================
-- SIDEBAR
--==========================================================

local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 125, 1, -48)
Sidebar.Position = UDim2.new(0, 0, 0, 48)
Sidebar.BackgroundColor3 = Theme.Sidebar
Sidebar.BackgroundTransparency = 0.15
Sidebar.BorderSizePixel = 0
Sidebar.ZIndex = 2
Sidebar.Parent = Main

local SidebarLayout = Instance.new("UIListLayout")
SidebarLayout.Padding = UDim.new(0, 7)
SidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
SidebarLayout.Parent = Sidebar

local SidebarPadding = Instance.new("UIPadding")
SidebarPadding.PaddingTop = UDim.new(0, 10)
SidebarPadding.Parent = Sidebar

--==========================================================
-- CONTENT
--==========================================================

local Content = Instance.new("Frame")
Content.Size = UDim2.new(1, -140, 1, -63)
Content.Position = UDim2.new(0, 135, 0, 55)
Content.BackgroundTransparency = 1
Content.ZIndex = 2
Content.Parent = Main

local Pages = {}

local function CreatePage(name)
    local Page = Instance.new("ScrollingFrame")
    Page.Name = name
    Page.Size = UDim2.fromScale(1, 1)
    Page.BackgroundTransparency = 1
    Page.BorderSizePixel = 0
    Page.ScrollBarThickness = 3
    Page.Visible = false
    Page.CanvasSize = UDim2.new(0, 0, 0, 0)
    Page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    Page.ZIndex = 3
    Page.Parent = Content

    local Layout = Instance.new("UIListLayout")
    Layout.Padding = UDim.new(0, 8)
    Layout.Parent = Page

    Pages[name] = Page

    return Page
end

local CombatPage = CreatePage("Combat")
local VisualPage = CreatePage("Visual")
local MovementPage = CreatePage("Movement")
local FixPage = CreatePage("FixLag")
local SettingsPage = CreatePage("Settings")

CombatPage.Visible = true

--==========================================================
-- TAB CREATOR
--==========================================================

local Tabs = {}

local function CreateTab(text, page)
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(1, -12, 0, 36)
    Button.Text = text
    Button.TextColor3 = Theme.Text
    Button.Font = Enum.Font.SourceSansBold
    Button.TextSize = 13
    Button.TextXAlignment = Enum.TextXAlignment.Left
    Button.BackgroundColor3 = Theme.Button
    Button.BorderSizePixel = 0
    Button.ZIndex = 3
    Button.Parent = Sidebar

    Instance.new("UICorner", Button).CornerRadius = UDim.new(0, 8)

    table.insert(Tabs, Button)

    Button.MouseButton1Click:Connect(function()
        for _, p in pairs(Pages) do
            p.Visible = false
        end

        for _, b in ipairs(Tabs) do
            b.BackgroundColor3 = Theme.Button
        end

        page.Visible = true
        Button.BackgroundColor3 = Theme.Accent
    end)

    return Button
end

local CombatTab = CreateTab("⚔️  Combat", CombatPage)
CreateTab("👁️  Visual", VisualPage)
CreateTab("⚡  Movement", MovementPage)
CreateTab("🚀  Fix Lag", FixPage)
CreateTab("⚙️  Settings", SettingsPage)

CombatTab.BackgroundColor3 = Theme.Accent

--==========================================================
-- UI HELPERS
--==========================================================

local function CreateButton(parent, text, callback)
    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(1, -8, 0, 38)
    Button.BackgroundColor3 = Theme.Button
    Button.BorderSizePixel = 0
    Button.Text = text
    Button.TextColor3 = Theme.Text
    Button.Font = Enum.Font.SourceSansBold
    Button.TextSize = 13
    Button.ZIndex = 4
    Button.Parent = parent

    Instance.new("UICorner", Button).CornerRadius = UDim.new(0, 8)

    Button.MouseButton1Click:Connect(callback)

    return Button
end

local function CreateToggle(parent, text, default, callback)
    local Button = CreateButton(parent, text, function()
        default = not default

        Button.BackgroundColor3 =
            default and Theme.Accent or Theme.Button

        callback(default)
    end)

    Button.BackgroundColor3 =
        default and Theme.Accent or Theme.Button

    return Button
end

--==========================================================
-- COMBAT PAGE
--==========================================================

CreateButton(CombatPage, "ℹ️ Combat Demo", function()
    Notify("Combat", "Trang này dành cho chức năng trong game của bạn.")
end)

CreateButton(CombatPage, "🎯 Target Info", function()
    Notify(
        "Target",
        "Selected: " .. LocalPlayer.DisplayName
    )
end)

--==========================================================
-- VISUAL PAGE
--==========================================================

CreateToggle(
    VisualPage,
    "👁️ Highlight Character",
    false,
    function(enabled)
        local Character = LocalPlayer.Character

        if not Character then
            return
        end

        local Highlight = Character:FindFirstChild("KianHighlight")

        if enabled then
            if not Highlight then
                Highlight = Instance.new("Highlight")
                Highlight.Name = "KianHighlight"
                Highlight.FillTransparency = 0.75
                Highlight.OutlineTransparency = 0
                Highlight.Parent = Character
            end

            Highlight.OutlineColor = Theme.Accent
        else
            if Highlight then
                Highlight:Destroy()
            end
        end
    end
)

CreateToggle(
    VisualPage,
    "🏷️ Display Name",
    false,
    function(enabled)
        local Character = LocalPlayer.Character

        if not Character then
            return
        end

        local Head = Character:FindFirstChild("Head")

        if not Head then
            return
        end

        local old = Head:FindFirstChild("KianName")

        if enabled then
            if not old then
                local Billboard = Instance.new("BillboardGui")
                Billboard.Name = "KianName"
                Billboard.Size = UDim2.new(0, 180, 0, 40)
                Billboard.StudsOffset = Vector3.new(0, 2.5, 0)
                Billboard.AlwaysOnTop = true
                Billboard.Parent = Head

                local Label = Instance.new("TextLabel")
                Label.Size = UDim2.fromScale(1, 1)
                Label.BackgroundTransparency = 1
                Label.Text = LocalPlayer.DisplayName
                Label.TextColor3 = Theme.Accent
                Label.TextStrokeTransparency = 0.3
                Label.Font = Enum.Font.SourceSansBold
                Label.TextSize = 16
                Label.Parent = Billboard
            end
        elseif old then
            old:Destroy()
        end
    end
)

--==========================================================
-- MOVEMENT PAGE
--==========================================================

CreateButton(MovementPage, "📱 Mobile Movement", function()
    Notify(
        "Movement",
        "Dùng Humanoid/Controls của game để điều khiển."
    )
end)

CreateButton(MovementPage, "🏃 Reset WalkSpeed", function()
    local Character = LocalPlayer.Character
    local Humanoid = Character and Character:FindFirstChildOfClass("Humanoid")

    if Humanoid then
        Humanoid.WalkSpeed = 16
        Notify("Movement", "WalkSpeed đã reset.")
    end
end)

--==========================================================
-- FIX LAG PAGE
--==========================================================

CreateButton(FixPage, "🚀 Performance Mode", function()
    local Lighting = game:GetService("Lighting")

    Lighting.GlobalShadows = false
    Lighting.FogEnd = 100000

    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ParticleEmitter") then
            obj.Enabled = false
        elseif obj:IsA("Trail") then
            obj.Enabled = false
        end
    end

    Notify("Fix Lag", "Đã bật Performance Mode.")
end)

CreateButton(FixPage, "✨ Restore Effects", function()
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ParticleEmitter") then
            obj.Enabled = true
        elseif obj:IsA("Trail") then
            obj.Enabled = true
        end
    end

    Notify("Fix Lag", "Đã khôi phục Effect.")
end)

--==========================================================
-- SETTINGS
--==========================================================

CreateButton(SettingsPage, "🎨 Đổi Theme", function()
    Config.CurrentTheme += 1

    if Config.CurrentTheme > #Themes then
        Config.CurrentTheme = 1
    end

    Theme = Themes[Config.CurrentTheme]

    Main.BackgroundColor3 = Theme.Bg
    MainStroke.Color = Theme.Accent
    Title.TextColor3 = Theme.Accent

    Notify(
        "Theme",
        "Đã đổi sang " .. Theme.Name
    )
end)

CreateButton(SettingsPage, "🖼️ Đổi Anime Background", function()
    Config.CurrentBackground += 1

    if Config.CurrentBackground > #Backgrounds then
        Config.CurrentBackground = 1
    end

    Background.Image = Backgrounds[Config.CurrentBackground]

    Notify(
        "Background",
        "Anime Background #" ..
            Config.CurrentBackground
    )
end)

CreateToggle(
    SettingsPage,
    "💤 Anti-AFK",
    Config.AntiAFK,
    function(enabled)
        Config.AntiAFK = enabled

        Notify(
            "Anti-AFK",
            enabled and "Đã bật" or "Đã tắt"
        )
    end
)

CreateButton(SettingsPage, "🧹 Cleanup GUI", function()
    Gui:Destroy()
end)

--==========================================================
-- TOGGLE BUTTON
--==========================================================

local Toggle = Instance.new("TextButton")
Toggle.Name = "KianToggle"
Toggle.Size = UDim2.new(0, 58, 0, 58)
Toggle.Position = UDim2.new(0, 15, 0.45, 0)
Toggle.BackgroundColor3 = Theme.Sidebar
Toggle.Text = "KIAN"
Toggle.TextColor3 = Theme.Accent
Toggle.Font = Enum.Font.FredokaOne
Toggle.TextSize = 14
Toggle.BorderSizePixel = 0
Toggle.ZIndex = 100
Toggle.Parent = Gui

Instance.new("UICorner", Toggle).CornerRadius = UDim.new(1, 0)

local ToggleStroke = Instance.new("UIStroke")
ToggleStroke.Color = Theme.Accent
ToggleStroke.Thickness = 2
ToggleStroke.Parent = Toggle

local function ToggleMenu()
    Config.MenuOpen = not Config.MenuOpen

    if Config.MenuOpen then
        Main.Visible = true

        Main.Size = UDim2.new(0.92, 0, 0, 0)

        TweenService:Create(
            Main,
            TweenInfo.new(.3, Enum.EasingStyle.Back),
            {Size = UDim2.new(0.92, 0, 0, 390)}
        ):Play()
    else
        local Tween = TweenService:Create(
            Main,
            TweenInfo.new(.2),
            {Size = UDim2.new(0.92, 0, 0, 0)}
        )

        Tween:Play()

        Tween.Completed:Connect(function()
            if not Config.MenuOpen then
                Main.Visible = false
            end
        end)
    end
end

Toggle.MouseButton1Click:Connect(ToggleMenu)
Close.MouseButton1Click:Connect(ToggleMenu)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then
        return
    end

    if input.KeyCode == Config.ToggleKey then
        ToggleMenu()
    end
end)

--==========================================================
-- DRAG SUPPORT
--==========================================================

local dragging = false
local dragStart
local startPosition

Header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true
        dragStart = input.Position
        startPosition = Main.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if not dragging then
        return
    end

    if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then

        local delta = input.Position - dragStart

        Main.Position = UDim2.new(
            startPosition.X.Scale,
            startPosition.X.Offset + delta.X,
            startPosition.Y.Scale,
            startPosition.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

        dragging = false
    end
end)

--==========================================================
-- FPS / PING
--==========================================================

local Last = os.clock()
local Frames = 0

RunService.RenderStepped:Connect(function()
    Frames += 1

    local now = os.clock()

    if now - Last >= 1 then
        local FPS = math.floor(
            Frames / (now - Last)
        )

        local Ping = 0

        pcall(function()
            Ping = math.floor(
                Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
            )
        end)

        StatsLabel.Text =
            "FPS: " .. FPS ..
            " | Ping: " .. Ping .. "ms"

        Frames = 0
        Last = now
    end
end)

--==========================================================
-- START
--==========================================================

Notify(
    "Kianbest Hub V9.6",
    "Menu đã khởi động thành công!"
)