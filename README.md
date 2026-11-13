
--[[
    KIANBEST ANIME SECURITY DASHBOARD
    Version: 2.0
    Type: LocalScript
    Location: StarterPlayer > StarterPlayerScripts

    UI only:
    Requires server-side security system for real protection.
]]

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

--==================================================
-- CONFIG
--==================================================

local CONFIG = {
    Title = "KianBest",
    Subtitle = "ANIME SECURITY CENTER",
    ToggleKey = Enum.KeyCode.RightShift,
    Accent = Color3.fromRGB(177, 139, 255),
    Pink = Color3.fromRGB(255, 164, 214),
    Background = Color3.fromRGB(24, 20, 39),
    Panel = Color3.fromRGB(39, 32, 58),
    Text = Color3.fromRGB(245, 239, 255),
    Muted = Color3.fromRGB(169, 157, 194),
    Green = Color3.fromRGB(108, 225, 165),
    Red = Color3.fromRGB(255, 105, 135),
    Orange = Color3.fromRGB(255, 190, 105),
}

--==================================================
-- CLEAN OLD UI
--==================================================

local old = PlayerGui:FindFirstChild("KianBestDashboard")
if old then
    old:Destroy()
end

--==================================================
-- HELPERS
--==================================================

local function create(class, props, parent)
    local object = Instance.new(class)

    for key, value in pairs(props or {}) do
        object[key] = value
    end

    object.Parent = parent
    return object
end

local function corner(parent, radius)
    return create("UICorner", {
        CornerRadius = UDim.new(0, radius or 12)
    }, parent)
end

local function stroke(parent, color, transparency)
    return create("UIStroke", {
        Color = color or CONFIG.Accent,
        Transparency = transparency or 0.5,
        Thickness = 1
    }, parent)
end

local function padding(parent, amount)
    return create("UIPadding", {
        PaddingTop = UDim.new(0, amount),
        PaddingBottom = UDim.new(0, amount),
        PaddingLeft = UDim.new(0, amount),
        PaddingRight = UDim.new(0, amount)
    }, parent)
end

local function label(parent, text, size, color, font)
    return create("TextLabel", {
        BackgroundTransparency = 1,
        Text = text,
        TextSize = size or 14,
        TextColor3 = color or CONFIG.Text,
        Font = font or Enum.Font.GothamMedium,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Center,
        Size = UDim2.new(1, 0, 0, 24),
    }, parent)
end

local function panel(parent, size, position)
    local frame = create("Frame", {
        Size = size,
        Position = position,
        BackgroundColor3 = CONFIG.Panel,
        BorderSizePixel = 0
    }, parent)

    corner(frame, 14)
    stroke(frame, CONFIG.Accent, 0.76)

    return frame
end

local function tween(object, properties, duration)
    TweenService:Create(
        object,
        TweenInfo.new(duration or 0.2, Enum.EasingStyle.Quart,
            Enum.EasingDirection.Out),
        properties
    ):Play()
end

--==================================================
-- MAIN GUI
--==================================================

local Gui = create("ScreenGui", {
    Name = "KianBestDashboard",
    ResetOnSpawn = false,
    IgnoreGuiInset = true,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling
}, PlayerGui)

local Main = create("Frame", {
    Name = "Main",
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.new(0.92, 0, 0.84, 0),
    BackgroundColor3 = CONFIG.Background,
    BorderSizePixel = 0,
    ClipsDescendants = true
}, Gui)

corner(Main, 20)
stroke(Main, CONFIG.Accent, 0.35)

create("UISizeConstraint", {
    MinSize = Vector2.new(310, 350),
    MaxSize = Vector2.new(1050, 720)
}, Main)

-- Background decoration
local BackgroundGlow = create("Frame", {
    Size = UDim2.fromScale(1, 1),
    BackgroundColor3 = CONFIG.Background,
    BorderSizePixel = 0,
    ZIndex = 0
}, Main)

local Gradient = create("UIGradient", {
    Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(35, 27, 57)),
        ColorSequenceKeypoint.new(0.55, Color3.fromRGB(24, 20, 39)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(52, 31, 65))
    }),
    Rotation = 25
}, BackgroundGlow)

--==================================================
-- HEADER
--==================================================

local Header = create("Frame", {
    Size = UDim2.new(1, 0, 0, 66),
    BackgroundTransparency = 1,
    ZIndex = 2
}, Main)

local Logo = create("TextLabel", {
    Position = UDim2.fromOffset(20, 10),
    Size = UDim2.fromOffset(42, 42),
    BackgroundColor3 = CONFIG.Accent,
    Text = "K",
    TextSize = 24,
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(1, 1, 1)
}, Header)

corner(Logo, 13)

local Brand = label(Header, CONFIG.Title, 19, CONFIG.Text, Enum.Font.GothamBold)
Brand.Position = UDim2.fromOffset(73, 9)
Brand.Size = UDim2.fromOffset(180, 24)

local Subtitle = label(Header, CONFIG.Subtitle, 9, CONFIG.Muted)
Subtitle.Position = UDim2.fromOffset(74, 33)
Subtitle.Size = UDim2.fromOffset(220, 18)

local Close = create("TextButton", {
    AnchorPoint = Vector2.new(1, 0),
    Position = UDim2.new(1, -15, 0, 15),
    Size = UDim2.fromOffset(35, 35),
    BackgroundColor3 = Color3.fromRGB(72, 49, 77),
    Text = "×",
    TextSize = 24,
    Font = Enum.Font.GothamBold,
    TextColor3 = CONFIG.Text,
    AutoButtonColor = false
}, Header)

corner(Close, 10)

--==================================================
-- SIDEBAR
--==================================================

local Sidebar = create("Frame", {
    Position = UDim2.fromOffset(14, 78),
    Size = UDim2.new(0, 150, 1, -94),
    BackgroundColor3 = Color3.fromRGB(32, 27, 48),
    BorderSizePixel = 0
}, Main)

corner(Sidebar, 14)
padding(Sidebar, 10)

local SidebarList = create("UIListLayout", {
    Padding = UDim.new(0, 8),
    SortOrder = Enum.SortOrder.LayoutOrder
}, Sidebar)

local Content = create("Frame", {
    Position = UDim2.fromOffset(176, 78),
    Size = UDim2.new(1, -190, 1, -94),
    BackgroundTransparency = 1,
    ClipsDescendants = true
}, Main)

local Pages = {}
local CurrentPage = nil
local NavButtons = {}

local function createPage(name)
    local page = create("ScrollingFrame", {
        Name = name,
        Size = UDim2.fromScale(1, 1),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = CONFIG.Accent,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        Visible = false
    }, Content)

    local layout = create("UIListLayout", {
        Padding = UDim.new(0, 12),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, page)

    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        page.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 20)
    end)

    Pages[name] = page
    return page
end

local function switchPage(name)
    for pageName, page in pairs(Pages) do
        page.Visible = pageName == name
    end

    for buttonName, button in pairs(NavButtons) do
        local active = buttonName == name
        button.BackgroundColor3 = active
            and Color3.fromRGB(83, 63, 119)
            or Color3.fromRGB(43, 36, 61)
    end

    CurrentPage = name
end

local function addNav(name, icon)
    local button = create("TextButton", {
        Size = UDim2.new(1, 0, 0, 39),
        BackgroundColor3 = Color3.fromRGB(43, 36, 61),
        Text = icon .. "   " .. name,
        TextColor3 = CONFIG.Text,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        AutoButtonColor = false
    }, Sidebar)

    corner(button, 9)

    button.MouseButton1Click:Connect(function()
        switchPage(name)
    end)

    NavButtons[name] = button
end

--==================================================
-- PAGES
--==================================================

local Overview = createPage("Overview")
local Protection = createPage("Protection")
local Logs = createPage("Logs")
local Settings = createPage("Settings")

addNav("Overview", "⌂")
addNav("Protection", "◇")
addNav("Logs", "≡")
addNav("Settings", "⚙")

--==================================================
-- CARD COMPONENT
--==================================================

local function createCard(parent, title, description, color)
    local card = panel(parent, UDim2.new(1, -8, 0, 94), UDim2.new(0, 4, 0, 0))
    padding(card, 12)

    local titleLabel = label(card, title, 14, color or CONFIG.Text, Enum.Font.GothamBold)
    titleLabel.Size = UDim2.new(1, 0, 0, 24)

    local desc = label(card, description, 11, CONFIG.Muted)
    desc.Position = UDim2.fromOffset(0, 32)
    desc.Size = UDim2.new(1, 0, 0, 40)
    desc.TextWrapped = true

    return card
end

--==================================================
-- OVERVIEW
--==================================================

local Welcome = createCard(
    Overview,
    "Welcome back, " .. Player.DisplayName,
    "KianBest Security Center is ready. Monitor your game security from here.",
    CONFIG.Pink
)

local StatusCard = createCard(
    Overview,
    "SYSTEM STATUS",
    "Waiting for server security connection...",
    CONFIG.Orange
)

local StatusText = label(StatusCard, "● DISCONNECTED", 11, CONFIG.Orange)
StatusText.Position = UDim2.fromOffset(0, 60)

local Stats = create("Frame", {
    Size = UDim2.new(1, -8, 0, 105),
    BackgroundTransparency = 1
}, Overview)

local function statCard(position, title, value, color)
    local card = create("Frame", {
        Position = position,
        Size = UDim2.new(0.32, -5, 1, 0),
        BackgroundColor3 = CONFIG.Panel,
        BorderSizePixel = 0
    }, Stats)

    corner(card, 12)
    stroke(card, color, 0.8)

    local t = label(card, title, 10, CONFIG.Muted)
    t.Position = UDim2.fromOffset(10, 12)

    local v = label(card, value, 23, color, Enum.Font.GothamBold)
    v.Position = UDim2.fromOffset(10, 42)

    return v
end

local ModuleCount = statCard(UDim2.fromScale(0, 0), "MODULES", "0", CONFIG.Accent)
local AlertCount = statCard(UDim2.fromScale(0.34, 0), "ALERTS", "0", CONFIG.Orange)
local OnlineCount = statCard(UDim2.fromScale(0.68, 0), "PLAYERS", "0", CONFIG.Green)

createCard(
    Overview,
    "SECURITY NOTICE",
    "Client-side indicators are informational only. Real anti-cheat checks must run on the server.",
    CONFIG.Pink
)

--==================================================
-- PROTECTION PAGE
--==================================================

local moduleNames = {
    "Anti-Speed",
    "Anti-Fly",
    "Anti-Teleport",
    "Anti-Noclip",
    "Anti-Remote",
    "Anti-Tool",
    "Anti-Stat",
    "Anti-Respawn"
}

local ModuleStates = {}

local function addModule(name)
    local card = panel(Protection, UDim2.new(1, -8, 0, 64), UDim2.new(0, 4, 0, 0))
    padding(card, 10)

    local title = label(card, name, 13, CONFIG.Text, Enum.Font.GothamBold)
    title.Size = UDim2.new(0.65, 0, 0, 38)

    local state = label(card, "NOT CONNECTED", 9, CONFIG.Orange)
    state.Position = UDim2.new(0, 0, 0, 35)
    state.Size = UDim2.new(0.7, 0, 0, 18)

    ModuleStates[name] = state
end

for _, name in ipairs(moduleNames) do
    addModule(name)
end

--==================================================
-- LOGS PAGE
--==================================================

local LogHeader = createCard(
    Logs,
    "SECURITY LOGS",
    "Server-generated security events will appear below.",
    CONFIG.Pink
)

local LogList = create("Frame", {
    Size = UDim2.new(1, -8, 0, 0),
    AutomaticSize = Enum.AutomaticSize.Y,
    BackgroundTransparency = 1
}, Logs)

local LogLayout = create("UIListLayout", {
    Padding = UDim.new(0, 7),
    SortOrder = Enum.SortOrder.LayoutOrder
}, LogList)

local logCount = 0

local function addLog(message, severity)
    logCount += 1

    local color = CONFIG.Green

    if severity == "WARNING" then
        color = CONFIG.Orange
    elseif severity == "ERROR" then
        color = CONFIG.Red
    end

    local item = create("Frame", {
        Size = UDim2.new(1, 0, 0, 55),
        BackgroundColor3 = CONFIG.Panel,
        BorderSizePixel = 0,
        LayoutOrder = -logCount
    }, LogList)

    corner(item, 9)
    stroke(item, color, 0.75)
    padding(item, 9)

    local heading = label(item, severity or "INFO", 10, color, Enum.Font.GothamBold)
    heading.Size = UDim2.new(1, 0, 0, 15)

    local body = label(item, tostring(message), 10, CONFIG.Text)
    body.Position = UDim2.fromOffset(0, 18)
    body.Size = UDim2.new(1, 0, 0, 25)
    body.TextTruncate = Enum.TextTruncate.AtEnd

    AlertCount.Text = tostring(logCount)

    if logCount > 40 then
        local children = LogList:GetChildren()

        for _, child in ipairs(children) do
            if child:IsA("Frame") and child.LayoutOrder == -logCount + 40 then
                child:Destroy()
            end
        end
    end
end

addLog("Dashboard initialized.", "INFO")

--==================================================
-- SETTINGS PAGE
--==================================================

createCard(
    Settings,
    "INTERFACE",
    "KianBest Anime Security Dashboard",
    CONFIG.Pink
)

local function addSettingButton(text, callback)
    local button = create("TextButton", {
        Size = UDim2.new(1, -8, 0, 42),
        BackgroundColor3 = Color3.fromRGB(71, 53, 99),
        Text = text,
        TextColor3 = CONFIG.Text,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        AutoButtonColor = false
    }, Settings)

    corner(button, 10)

    button.MouseButton1Click:Connect(callback)
end

addSettingButton("Add test log", function()
    addLog("Test security event.", "WARNING")
end)

addSettingButton("Reset local log display", function()
    for _, child in ipairs(LogList:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end

    logCount = 0
    AlertCount.Text = "0"
    addLog("Local log display reset.", "INFO")
end)

--==================================================
-- ANIME GIRL DECORATION
--==================================================

-- Replace this URL with an image you own or have permission to use.
local ANIME_IMAGE = ""

if ANIME_IMAGE ~= "" then
    local anime = create("ImageLabel", {
        Name = "AnimeGirl",
        AnchorPoint = Vector2.new(1, 1),
        Position = UDim2.new(1, -8, 1, -8),
        Size = UDim2.new(0, 150, 0, 190),
        BackgroundTransparency = 1,
        Image = ANIME_IMAGE,
        ScaleType = Enum.ScaleType.Fit,
        ImageTransparency = 0.08,
        ZIndex = 1
    }, Main)
end

--==================================================
-- SERVER CONNECTION
--==================================================

local securityFolder = ReplicatedStorage:FindFirstChild("KianBestSecurity")
local getLogs = securityFolder and securityFolder:FindFirstChild("GetLogs")
local logUpdate = securityFolder and securityFolder:FindFirstChild("LogUpdate")

if getLogs and getLogs:IsA("RemoteFunction") then
    local success, result = pcall(function()
        return getLogs:InvokeServer()
    end)

    if success and type(result) == "table" then
        StatusText.Text = "● CONNECTED"
        StatusText.TextColor3 = CONFIG.Green

        for _, entry in ipairs(result) do
            if type(entry) == "table" then
                addLog(entry.message or "Security event", entry.severity or "INFO")
            end
        end
    else
        StatusText.Text = "● CONNECTION ERROR"
        StatusText.TextColor3 = CONFIG.Red
    end
end

if logUpdate and logUpdate:IsA("RemoteEvent") then
    logUpdate.OnClientEvent:Connect(function(entry)
        if type(entry) == "table" then
            addLog(entry.message or "Security event", entry.severity or "INFO")
        end
    end)
end

--==================================================
-- NAVIGATION
--==================================================

switchPage("Overview")

--==================================================
-- OPEN / CLOSE
--==================================================

local minimized = false

Close.MouseButton1Click:Connect(function()
    minimized = not minimized
    Main.Visible = false
end)

local Toggle = create("TextButton", {
    Name = "KianBestToggle",
    Position = UDim2.new(0, 15, 0.5, 0),
    Size = UDim2.fromOffset(48, 48),
    BackgroundColor3 = CONFIG.Accent,
    Text = "K",
    TextSize = 22,
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.new(1, 1, 1),
    Visible = false
}, Gui)

corner(Toggle, 16)

Toggle.MouseButton1Click:Connect(function()
    Main.Visible = true
    Toggle.Visible = false
    minimized = false
end)

Close.MouseButton1Click:Connect(function()
    Main.Visible = false
    Toggle.Visible = true
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end

    if input.KeyCode == CONFIG.ToggleKey then
        Main.Visible = not Main.Visible
        Toggle.Visible = not Main.Visible
    end
end)

--==================================================
-- DRAG WINDOW
--==================================================

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
    if not dragging then return end

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

print("[KianBest] Anime Security Dashboard loaded.")
