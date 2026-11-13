
--==================================================
-- KIANBEST V5.5 FULL
-- PURPLE ANIME DASHBOARD
-- UI FRAMEWORK + SAFE MONITORING
--==================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")
local UserInputService = game:GetService("UserInputService")

local Player = Players.LocalPlayer

-- CONFIGURATION
local CONFIG = {
    Name = "KianBest",
    Version = "V5.5",
    Background = Color3.fromRGB(12, 7, 25),
    Panel = Color3.fromRGB(27, 15, 47),
    Purple = Color3.fromRGB(170, 70, 255),
    LightPurple = Color3.fromRGB(225, 160, 255),
    Text = Color3.fromRGB(245, 235, 255),
    Muted = Color3.fromRGB(175, 155, 195),
}

-- GUI PARENT
local GUIParent
pcall(function()
    if gethui then
        GUIParent = gethui()
    end
end)

if not GUIParent then
    GUIParent = game:GetService("CoreGui")
end

-- REMOVE OLD GUI
pcall(function()
    local old = GUIParent:FindFirstChild("KianBestV55")
    if old then old:Destroy() end
end)

-- SCREEN GUI
local Gui = Instance.new("ScreenGui")
Gui.Name = "KianBestV55"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true
Gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
Gui.Parent = GUIParent

-- HELPERS
local function Create(class, properties, parent)
    local obj = Instance.new(class)
    for key, value in pairs(properties or {}) do
        obj[key] = value
    end
    obj.Parent = parent
    return obj
end

local function Round(obj, radius)
    Create("UICorner", {
        CornerRadius = UDim.new(0, radius or 10)
    }, obj)
end

local function Outline(obj, color, thickness)
    Create("UIStroke", {
        Color = color or CONFIG.Purple,
        Thickness = thickness or 1,
        Transparency = 0.35
    }, obj)
end

local function MakeGradient(obj, color1, color2)
    Create("UIGradient", {
        Color = ColorSequence.new(color1, color2),
        Rotation = 40
    }, obj)
end

local function Text(parent, value, size, position, fontSize, color)
    return Create("TextLabel", {
        BackgroundTransparency = 1,
        Size = size,
        Position = position,
        Text = value,
        Font = Enum.Font.GothamMedium,
        TextSize = fontSize or 13,
        TextColor3 = color or CONFIG.Text,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Center,
        TextWrapped = true
    }, parent)
end

local function NewButton(parent, value, size, position)
    local button = Create("TextButton", {
        Size = size,
        Position = position,
        BackgroundColor3 = CONFIG.Panel,
        Text = value,
        TextColor3 = CONFIG.Text,
        Font = Enum.Font.GothamMedium,
        TextSize = 13,
        AutoButtonColor = false
    }, parent)

    Round(button, 9)
    Outline(button)

    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.15), {
            BackgroundColor3 = Color3.fromRGB(66, 29, 100)
        }):Play()
    end)

    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.15), {
            BackgroundColor3 = CONFIG.Panel
        }):Play()
    end)

    return button
end

-- MAIN WINDOW
local Main = Create("Frame", {
    Name = "Main",
    Size = UDim2.fromOffset(900, 570),
    Position = UDim2.new(0.5, -450, 0.5, -285),
    BackgroundColor3 = CONFIG.Background,
    BorderSizePixel = 0,
    ClipsDescendants = true
}, Gui)

Round(Main, 16)
Outline(Main, CONFIG.Purple, 2)

MakeGradient(
    Main,
    Color3.fromRGB(30, 12, 55),
    Color3.fromRGB(9, 7, 21)
)

-- HEADER
local Header = Create("Frame", {
    Size = UDim2.new(1, 0, 0, 65),
    BackgroundColor3 = Color3.fromRGB(40, 15, 68),
    BorderSizePixel = 0,
    ZIndex = 2
}, Main)

MakeGradient(
    Header,
    Color3.fromRGB(95, 25, 145),
    Color3.fromRGB(22, 10, 40)
)

Text(Header, "♛  KianBest", UDim2.fromOffset(230, 35),
    UDim2.fromOffset(20, 4), 25, CONFIG.LightPurple)

Text(Header, "ULTIMATE PURPLE FRAMEWORK  •  V5.5",
    UDim2.fromOffset(300, 20),
    UDim2.fromOffset(22, 39), 10, CONFIG.Muted)

local Minimize = NewButton(
    Header, "—", UDim2.fromOffset(35, 32),
    UDim2.new(1, -85, 0, 16)
)

local Close = NewButton(
    Header, "×", UDim2.fromOffset(35, 32),
    UDim2.new(1, -43, 0, 16)
)

-- SIDEBAR
local Sidebar = Create("Frame", {
    Size = UDim2.new(0, 175, 1, -75),
    Position = UDim2.fromOffset(10, 72),
    BackgroundColor3 = Color3.fromRGB(20, 11, 36),
    BorderSizePixel = 0
}, Main)

Round(Sidebar, 12)
Outline(Sidebar, Color3.fromRGB(100, 50, 155))

Create("UIPadding", {
    PaddingTop = UDim.new(0, 10),
    PaddingLeft = UDim.new(0, 9),
    PaddingRight = UDim.new(0, 9)
}, Sidebar)

Create("UIListLayout", {
    Padding = UDim.new(0, 6),
    SortOrder = Enum.SortOrder.LayoutOrder
}, Sidebar)

-- CONTENT
local Content = Create("Frame", {
    Size = UDim2.new(1, -205, 1, -85),
    Position = UDim2.fromOffset(195, 75),
    BackgroundTransparency = 1
}, Main)

local Pages = {}
local CurrentPage

local function CreatePage(name)
    local page = Create("ScrollingFrame", {
        Name = name,
        Size = UDim2.fromScale(1, 1),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = CONFIG.Purple,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        Visible = false
    }, Content)

    local layout = Create("UIListLayout", {
        Padding = UDim.new(0, 10),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, page)

    Create("UIPadding", {
        PaddingTop = UDim.new(0, 5),
        PaddingLeft = UDim.new(0, 5),
        PaddingRight = UDim.new(0, 10)
    }, page)

    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        page.CanvasSize = UDim2.new(
            0, 0, 0, layout.AbsoluteContentSize.Y + 20
        )
    end)

    Pages[name] = page
    return page
end

local function ShowPage(name)
    for pageName, page in pairs(Pages) do
        page.Visible = pageName == name
    end
    CurrentPage = name
end

-- NOTIFICATIONS
local function Notify(title, message)
    local toast = Create("Frame", {
        Size = UDim2.fromOffset(260, 65),
        Position = UDim2.new(1, 280, 1, -90),
        BackgroundColor3 = Color3.fromRGB(39, 19, 65),
        BorderSizePixel = 0,
        ZIndex = 50
    }, Main)

    Round(toast, 10)
    Outline(toast)

    Text(toast, "✦  " .. title,
        UDim2.new(1, -15, 0, 25),
        UDim2.fromOffset(12, 5), 13, CONFIG.LightPurple)

    Text(toast, message,
        UDim2.new(1, -15, 0, 25),
        UDim2.fromOffset(12, 32), 11, CONFIG.Text)

    TweenService:Create(toast, TweenInfo.new(0.3), {
        Position = UDim2.new(1, -275, 1, -90)
    }):Play()

    task.delay(2.5, function()
        if toast.Parent then
            toast:Destroy()
        end
    end)
end

-- SECTION
local function Section(parent, title, description, height)
    local card = Create("Frame", {
        Size = UDim2.new(1, -5, 0, height or 100),
        BackgroundColor3 = Color3.fromRGB(25, 14, 43),
        BorderSizePixel = 0
    }, parent)

    Round(card, 12)
    Outline(card, Color3.fromRGB(105, 55, 155))

    Text(card, title,
        UDim2.new(1, -25, 0, 30),
        UDim2.fromOffset(14, 7), 15, CONFIG.LightPurple)

    if description then
        Text(card, description,
            UDim2.new(1, -25, 0, 35),
            UDim2.fromOffset(14, 39), 11, CONFIG.Muted)
    end

    return card
end

-- STAT CARD
local function StatCard(parent, title, value)
    local card = Create("Frame", {
        Size = UDim2.new(0.32, -5, 0, 82),
        BackgroundColor3 = Color3.fromRGB(36, 19, 58),
        BorderSizePixel = 0
    }, parent)

    Round(card, 10)
    Outline(card)

    Text(card, title,
        UDim2.new(1, -15, 0, 22),
        UDim2.fromOffset(10, 6), 11, CONFIG.Muted)

    local valueLabel = Text(card, value,
        UDim2.new(1, -15, 0, 35),
        UDim2.fromOffset(10, 34), 20, CONFIG.LightPurple)

    return valueLabel
end

-- DASHBOARD PAGE
local Home = CreatePage("Dashboard")

Section(Home, "✦  Welcome to KianBest!",
    "Purple Anime Edition • Play smarter, explore more", 85)

local StatRow = Create("Frame", {
    Size = UDim2.new(1, -5, 0, 85),
    BackgroundTransparency = 1
}, Home)

Create("UIListLayout", {
    FillDirection = Enum.FillDirection.Horizontal,
    Padding = UDim.new(0, 10)
}, StatRow)

local FPSValue = StatCard(StatRow, "FPS", "--")
local PingValue = StatCard(StatRow, "PING", "--")
local TimeValue = StatCard(StatRow, "SESSION", "00:00")

Section(Home, "♟  Player Information",
    "Player: " .. Player.Name ..
    "\nUser ID: " .. tostring(Player.UserId), 100)

Section(Home, "♡  Anime Corner",
    "KianBest • Always with you!", 75)

Section(Home, "✦  Quick Information",
    "Dashboard UI loaded successfully.", 75)

-- PLAYER PAGE
local PlayerPage = CreatePage("Player")

Section(PlayerPage, "♟  Player Overview",
    "Thông tin người chơi và nhân vật.", 95)

Section(PlayerPage, "Character Status",
    "Theo dõi thông tin nhân vật hiện tại.", 90)

Section(PlayerPage, "Movement Settings",
    "Khu vực giao diện cài đặt chuyển động.", 90)

-- VISUALS PAGE
local Visuals = CreatePage("Visuals")

Section(Visuals, "◈  Visual Settings",
    "Quản lý tùy chọn hiển thị giao diện.", 90)

Section(Visuals, "Highlight Preview",
    "Khu vực xem trước hiệu ứng màu sắc.", 90)

Section(Visuals, "Theme Colors",
    "Dark Purple • Neon Purple • Pastel Purple", 90)

-- MONSTER DATABASE
local Monster = CreatePage("Monster")

local MonsterCard = Section(Monster, "☠  Monster Database",
    "Tìm kiếm danh sách quái vật của bạn.", 135)

local SearchBox = Create("TextBox", {
    Size = UDim2.new(1, -28, 0, 36),
    Position = UDim2.fromOffset(14, 85),
    BackgroundColor3 = Color3.fromRGB(45, 25, 68),
    PlaceholderText = "Search monster...",
    Text = "",
    TextColor3 = CONFIG.Text,
    PlaceholderColor3 = CONFIG.Muted,
    Font = Enum.Font.Gotham,
    TextSize = 12,
    ClearTextOnFocus = false
}, MonsterCard)

Round(SearchBox, 8)

Section(Monster, "Monster Categories",
    "Normal • Elite • Mini Boss • World Boss", 85)

-- SERVER PAGE
local Server = CreatePage("Server")

Section(Server, "◎  Server Center",
    "Thông tin máy chủ hiện tại.", 90)

Section(Server, "Low Player Server",
    "Giao diện bộ lọc server ít người. Cần API được game hỗ trợ để lấy danh sách server.", 105)

Section(Server, "Low Ping Finder",
    "Hiển thị ping hiện tại. Không tự động chuyển server.", 95)

-- PERFORMANCE PAGE
local Performance = CreatePage("Performance")

Section(Performance, "▥  Performance Monitor",
    "Theo dõi FPS và ping phiên hiện tại.", 90)

Section(Performance, "Memory Monitor",
    "Thông tin bộ nhớ phụ thuộc môi trường hỗ trợ.", 90)

Section(Performance, "Optimization",
    "Các tùy chọn tối ưu giao diện dashboard.", 90)

-- SAFETY PAGE
local Safety = CreatePage("Safety")

Section(Safety, "✓  Safety Center",
    "Kiểm tra trạng thái dashboard và lỗi giao diện.", 90)

Section(Safety, "Script Status",
    "KianBest UI: Loaded", 80)

Section(Safety, "Activity Log",
    "Theo dõi các thông báo thao tác.", 80)

-- SETTINGS PAGE
local SettingsPage = CreatePage("Settings")

Section(SettingsPage, "⚙  Settings",
    "Cài đặt giao diện KianBest.", 90)

Section(SettingsPage, "Theme Studio",
    "Purple Neon • Purple Pastel • Dark Purple", 90)

Section(SettingsPage, "Interface",
    "HOME • Minimize • Close • Drag", 90)

-- SIDEBAR TABS
local Tabs = {
    "Dashboard",
    "Player",
    "Visuals",
    "Monster",
    "Server",
    "Performance",
    "Safety",
    "Settings"
}

for index, name in ipairs(Tabs) do
    local tab = NewButton(
        Sidebar,
        "✦   " .. name,
        UDim2.new(1, 0, 0, 39),
        UDim2.new()
    )

    tab.LayoutOrder = index

    tab.Activated:Connect(function()
        ShowPage(name)
        Notify("Navigation", "Opened " .. name)
    end)
end

ShowPage("Dashboard")

--==================================================
-- FLOATING ROUND HOME BUTTON
--==================================================

local HomeButton = Create("TextButton", {
    Name = "KianBestHome",
    Size = UDim2.fromOffset(58, 58),
    Position = UDim2.new(0, 20, 0.5, -29),
    BackgroundColor3 = CONFIG.Purple,
    Text = "⌂",
    TextColor3 = Color3.new(1, 1, 1),
    TextSize = 35,
    Font = Enum.Font.GothamBold,
    AutoButtonColor = false,
    ZIndex = 100
}, Gui)

Round(HomeButton, 100)
Outline(HomeButton, CONFIG.LightPurple, 2)

MakeGradient(
    HomeButton,
    Color3.fromRGB(220, 110, 255),
    Color3.fromRGB(80, 20, 160)
)

-- HOME BUTTON GLOW
local HomeGlow = Create("UIStroke", {
    Color = CONFIG.LightPurple,
    Thickness = 3,
    Transparency = 0.65
}, HomeButton)

-- HOME BUTTON DRAG
local homeDragging = false
local homeMoved = false
local homeStart = nil
local homeStartPosition = nil
local activeInput = nil

HomeButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch
    or input.UserInputType == Enum.UserInputType.MouseButton1 then
        homeDragging = true
        homeMoved = false
        activeInput = input
        homeStart = input.Position
        homeStartPosition = HomeButton.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if not homeDragging then return end

    local isMouseMove =
        input.UserInputType == Enum.UserInputType.MouseMovement

    local isTouchMove = input == activeInput

    if isMouseMove or isTouchMove then
        local delta = input.Position - homeStart

        if delta.Magnitude > 8 then
            homeMoved = true

            HomeButton.Position = UDim2.new(
                homeStartPosition.X.Scale,
                homeStartPosition.X.Offset + delta.X,
                homeStartPosition.Y.Scale,
                homeStartPosition.Y.Offset + delta.Y
            )
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        homeDragging = false
        activeInput = nil
    end
end)

-- SINGLE HOME CLICK HANDLER
HomeButton.Activated:Connect(function()
    if homeMoved then
        homeMoved = false
        return
    end

    Main.Visible = not Main.Visible

    if Main.Visible then
        Notify("KianBest", "Dashboard opened!")
    end
end)

--==================================================
-- MAIN WINDOW DRAG
--==================================================

local mainDragging = false
local mainStart
local mainPosition

Header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        mainDragging = true
        mainStart = input.Position
        mainPosition = Main.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if not mainDragging then return end

    if input.UserInputType == Enum.UserInputType.MouseMovement
    or input.UserInputType == Enum.UserInputType.Touch then
        local delta = input.Position - mainStart

        Main.Position = UDim2.new(
            mainPosition.X.Scale,
            mainPosition.X.Offset + delta.X,
            mainPosition.Y.Scale,
            mainPosition.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        mainDragging = false
    end
end)

--==================================================
-- MINIMIZE / RESTORE
--==================================================

local minimized = false

Minimize.Activated:Connect(function()
    minimized = not minimized

    Sidebar.Visible = not minimized
    Content.Visible = not minimized

    Main.Size = minimized
        and UDim2.fromOffset(900, 65)
        or UDim2.fromOffset(900, 570)

    Minimize.Text = minimized and "□" or "—"
end)

-- CLOSE = HIDE, NOT DESTROY
Close.Activated:Connect(function()
    Main.Visible = false
end)

--==================================================
-- LIVE FPS / PING / SESSION
--==================================================

local startTime = os.clock()
local frameCount = 0
local lastUpdate = os.clock()

RunService.RenderStepped:Connect(function()
    frameCount += 1

    local now = os.clock()

    if now - lastUpdate >= 1 then
        FPSValue.Text = tostring(frameCount)
        frameCount = 0
        lastUpdate = now

        local elapsed = math.floor(now - startTime)
        local minutes = math.floor(elapsed / 60)
        local seconds = elapsed % 60

        TimeValue.Text = string.format(
            "%02d:%02d", minutes, seconds
        )

        local success, ping = pcall(function()
            return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
        end)

        if success and ping then
            PingValue.Text = math.floor(ping) .. " ms"
        else
            PingValue.Text = "N/A"
        end
    end
end)

-- SEARCH DATABASE DEMO
SearchBox:GetPropertyChangedSignal("Text"):Connect(function()
    local query = string.lower(SearchBox.Text)
    print("[KianBest] Monster search:", query)
end)

-- STARTUP
Notify("KianBest V5.5", "Purple Anime Dashboard ready!")

print("===================================")
print("       KIANBEST V5.5 LOADED")
print("       PURPLE ANIME DASHBOARD")
print("       HOME BUTTON ENABLED")
print("===================================")
