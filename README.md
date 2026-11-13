
--==================================================
-- KIANBEST V7.1 - ULTIMATE ANIME DASHBOARD
-- Roblox Studio - LocalScript
-- StarterPlayer > StarterPlayerScripts
--==================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- REMOVE OLD GUI
local old = playerGui:FindFirstChild("KianBestV71")
if old then old:Destroy() end

-- COLORS
local C = {
    BG = Color3.fromRGB(23, 18, 36),
    PANEL = Color3.fromRGB(37, 29, 55),
    PANEL2 = Color3.fromRGB(51, 40, 73),
    PURPLE = Color3.fromRGB(177, 139, 255),
    PINK = Color3.fromRGB(255, 177, 220),
    WHITE = Color3.fromRGB(248, 243, 255),
    MUTED = Color3.fromRGB(180, 167, 204),
    GREEN = Color3.fromRGB(45, 190, 100),
    ORANGE = Color3.fromRGB(255, 116, 55),
    BLACK = Color3.fromRGB(0, 0, 0)
}

--==================================================
-- UI HELPERS
--==================================================

local function create(class, props, parent)
    local obj = Instance.new(class)

    for k, v in pairs(props or {}) do
        obj[k] = v
    end

    obj.Parent = parent
    return obj
end

local function round(obj, radius)
    create("UICorner", {
        CornerRadius = UDim.new(0, radius or 10)
    }, obj)
end

local function outline(obj, color, transparency)
    create("UIStroke", {
        Color = color or C.PURPLE,
        Thickness = 1,
        Transparency = transparency or 0.45
    }, obj)
end

local function label(parent, text, size, color, font)
    return create("TextLabel", {
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = color or C.WHITE,
        TextSize = size or 14,
        Font = font or Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Center
    }, parent)
end

local function button(parent, text, height)
    local b = create("TextButton", {
        Size = UDim2.new(1, 0, 0, height or 40),
        BackgroundColor3 = C.PANEL2,
        Text = text,
        TextColor3 = C.WHITE,
        TextSize = 13,
        Font = Enum.Font.GothamMedium,
        AutoButtonColor = false
    }, parent)

    round(b, 10)
    outline(b)

    b.MouseEnter:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.15), {
            BackgroundColor3 = Color3.fromRGB(78, 59, 110)
        }):Play()
    end)

    b.MouseLeave:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.15), {
            BackgroundColor3 = C.PANEL2
        }):Play()
    end)

    return b
end

local function notify(message)
    noticeId += 1
    local current = noticeId

    notification.Text = "♡  " .. message
    notification.Visible = true

    task.delay(2.5, function()
        if current == noticeId and notification.Parent then
            notification.Visible = false
        end
    end)
end

--==================================================
-- MAIN GUI
--==================================================

local gui = create("ScreenGui", {
    Name = "KianBestV71",
    ResetOnSpawn = false,
    IgnoreGuiInset = true,
    DisplayOrder = 50
}, playerGui)

local main = create("Frame", {
    Name = "Main",
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromOffset(850, 550),
    BackgroundColor3 = C.BG,
    ClipsDescendants = true
}, gui)

round(main, 18)
outline(main, C.PURPLE, 0.2)

-- RESPONSIVE SCALE
local scale = create("UIScale", {
    Scale = 1
}, main)

local function updateScale()
    local camera = workspace.CurrentCamera
    if not camera then return end

    local size = camera.ViewportSize

    scale.Scale = math.clamp(
        math.min(size.X / 900, size.Y / 600),
        0.48,
        1
    )
end

updateScale()

local camera = workspace.CurrentCamera
if camera then
    camera:GetPropertyChangedSignal("ViewportSize"):Connect(updateScale)
end

--==================================================
-- HEADER
--==================================================

local header = create("Frame", {
    Size = UDim2.new(1, 0, 0, 62),
    BackgroundColor3 = Color3.fromRGB(41, 31, 60)
}, main)

round(header, 18)

local title = label(
    header,
    "✦ KIANBEST",
    21,
    C.WHITE,
    Enum.Font.GothamBold
)

title.Position = UDim2.fromOffset(20, 5)
title.Size = UDim2.fromOffset(220, 28)

local subtitle = label(
    header,
    "V7.1 ULTIMATE ANIME EDITION",
    10,
    C.PINK
)

subtitle.Position = UDim2.fromOffset(22, 33)
subtitle.Size = UDim2.fromOffset(240, 18)

local close = create("TextButton", {
    Size = UDim2.fromOffset(36, 36),
    Position = UDim2.new(1, -49, 0, 13),
    BackgroundColor3 = Color3.fromRGB(105, 57, 88),
    Text = "×",
    TextColor3 = C.WHITE,
    TextSize = 23,
    Font = Enum.Font.GothamBold
}, header)

round(close, 10)

close.MouseButton1Click:Connect(function()
    gui.Enabled = false
end)

--==================================================
-- SIDEBAR
--==================================================

local sidebar = create("Frame", {
    Position = UDim2.fromOffset(13, 75),
    Size = UDim2.new(0, 175, 1, -88),
    BackgroundColor3 = C.PANEL
}, main)

round(sidebar, 14)

local menuTitle = label(
    sidebar,
    "♡ MAIN MENU",
    12,
    C.PINK,
    Enum.Font.GothamBold
)

menuTitle.Position = UDim2.fromOffset(14, 12)
menuTitle.Size = UDim2.new(1, -20, 0, 25)

local menu = create("ScrollingFrame", {
    Position = UDim2.fromOffset(8, 45),
    Size = UDim2.new(1, -16, 1, -55),
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    ScrollBarThickness = 3,
    ScrollBarImageColor3 = C.PURPLE,
    AutomaticCanvasSize = Enum.AutomaticSize.Y,
    CanvasSize = UDim2.new()
}, sidebar)

create("UIListLayout", {
    Padding = UDim.new(0, 8),
    SortOrder = Enum.SortOrder.LayoutOrder
}, menu)

--==================================================
-- CONTENT
--==================================================

local content = create("Frame", {
    Position = UDim2.fromOffset(202, 75),
    Size = UDim2.new(1, -215, 1, -88),
    BackgroundTransparency = 1
}, main)

local pageTitle = label(
    content,
    "Welcome back ♡",
    21,
    C.WHITE,
    Enum.Font.GothamBold
)

pageTitle.Position = UDim2.fromOffset(5, 3)
pageTitle.Size = UDim2.new(1, -10, 0, 34)

local pageSubtitle = label(
    content,
    "Your cute control center is ready.",
    12,
    C.MUTED
)

pageSubtitle.Position = UDim2.fromOffset(6, 35)
pageSubtitle.Size = UDim2.new(1, -10, 0, 22)

-- NOTIFICATION
local noticeId = 0

local notification = create("TextLabel", {
    AnchorPoint = Vector2.new(1, 1),
    Position = UDim2.new(1, -15, 1, -12),
    Size = UDim2.fromOffset(260, 42),
    BackgroundColor3 = C.PANEL2,
    Text = "",
    TextColor3 = C.WHITE,
    TextSize = 12,
    Font = Enum.Font.GothamMedium,
    Visible = false,
    ZIndex = 20
}, main)

round(notification, 10)
outline(notification)

--==================================================
-- PAGE SYSTEM
--==================================================

local pages = {}
local menuButtons = {}

local function clearPage()
    for _, child in ipairs(content:GetChildren()) do
        if child ~= pageTitle and child ~= pageSubtitle then
            child:Destroy()
        end
    end
end

local function newPage(name, description)
    clearPage()

    pageTitle.Text = name
    pageSubtitle.Text = description or "KianBest V7.1"

    local scroll = create("ScrollingFrame", {
        Position = UDim2.fromOffset(0, 67),
        Size = UDim2.new(1, 0, 1, -72),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 4,
        ScrollBarImageColor3 = C.PURPLE,
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        CanvasSize = UDim2.new()
    }, content)

    create("UIListLayout", {
        Padding = UDim.new(0, 11),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, scroll)

    return scroll
end

local function section(parent, name)
    local box = create("Frame", {
        Size = UDim2.new(1, -5, 0, 0),
        AutomaticSize = Enum.AutomaticSize.Y,
        BackgroundColor3 = C.PANEL
    }, parent)

    round(box, 13)
    outline(box)

    local heading = label(
        box,
        name,
        14,
        C.PINK,
        Enum.Font.GothamBold
    )

    heading.Position = UDim2.fromOffset(13, 8)
    heading.Size = UDim2.new(1, -25, 0, 25)

    local inner = create("Frame", {
        Position = UDim2.fromOffset(12, 40),
        Size = UDim2.new(1, -24, 0, 0),
        AutomaticSize = Enum.AutomaticSize.Y,
        BackgroundTransparency = 1
    }, box)

    create("UIListLayout", {
        Padding = UDim.new(0, 8)
    }, inner)

    create("UIPadding", {
        PaddingBottom = UDim.new(0, 12)
    }, inner)

    return box, inner
end

local function addInfo(parent, text)
    local l = label(parent, text, 12, C.MUTED)
    l.Size = UDim2.new(1, 0, 0, 25)
    return l
end

local function addAction(parent, text, callback)
    local b = button(parent, text)
    b.MouseButton1Click:Connect(callback)
    return b
end

-- TOGGLE MODULE
local function addToggle(parent, name, default, callback)
    local state = default == true

    local b = create("TextButton", {
        Size = UDim2.new(1, 0, 0, 42),
        BackgroundColor3 = C.PANEL2,
        Text = "",
        AutoButtonColor = false
    }, parent)

    round(b, 10)

    local titleLabel = label(b, name, 13, C.WHITE)
    titleLabel.Position = UDim2.fromOffset(12, 0)
    titleLabel.Size = UDim2.new(0.68, 0, 1, 0)

    local status = create("TextLabel", {
        Size = UDim2.fromOffset(65, 25),
        Position = UDim2.new(1, -76, 0.5, -12),
        BackgroundTransparency = 1,
        TextSize = 11,
        Font = Enum.Font.GothamBold
    }, b)

    local function update()
        if state then
            status.Text = "● ON"
            status.TextColor3 = Color3.fromRGB(80, 255, 160)
            b.BackgroundColor3 = Color3.fromRGB(28, 67, 52)
        else
            status.Text = "● OFF"
            status.TextColor3 = Color3.fromRGB(180, 170, 195)
            b.BackgroundColor3 = C.PANEL2
        end
    end

    b.MouseButton1Click:Connect(function()
        state = not state
        update()

        if callback then
            callback(state)
        end
    end)

    update()
    return b
end

--==================================================
-- DASHBOARD PAGE
--==================================================

pages["Dashboard"] = function()
    local p = newPage(
        "Welcome back, " .. player.DisplayName .. " ♡",
        "Cute control center • KianBest V7.1"
    )

    local box, inner = section(p, "✦ LIVE SYSTEM STATUS")

    local fps = addInfo(inner, "FPS: Calculating...")
    local ping = addInfo(inner, "Ping: Calculating...")
    local playerCount = addInfo(inner, "Players: Calculating...")

    local quickBox, quick = section(p, "♡ QUICK ACCESS")

    addAction(quick, "☁ Server Manager", function()
        pages["Server Manager"]()
    end)

    addAction(quick, "✧ Visual Settings", function()
        pages["Visual Settings"]()
    end)

    addAction(quick, "◈ Performance", function()
        pages["Performance"]()
    end)

    local welcomeBox, welcome = section(p, "🌸 WELCOME")

    addInfo(welcome, "Hello, " .. player.DisplayName .. "!")
    addInfo(welcome, "Welcome to your anime dashboard.")
    addInfo(welcome, "Have a wonderful day ♡")

    task.spawn(function()
        local frames = 0
        local elapsed = 0

        while fps.Parent do
            local dt = RunService.RenderStepped:Wait()

            frames += 1
            elapsed += dt

            if elapsed >= 1 then
                fps.Text = "FPS: " .. math.floor(frames / elapsed)

                local ok, value = pcall(function()
                    return math.floor(player:GetNetworkPing() * 1000)
                end)

                ping.Text = ok and ("Ping: " .. value .. " ms")
                    or "Ping: N/A"

                playerCount.Text = "Players: " .. #Players:GetPlayers()

                frames = 0
                elapsed = 0
            end
        end
    end)
end

--==================================================
-- STATUS PAGE - STYLE FROM YOUR IMAGE
--==================================================

pages["Trạng thái"] = function()
    local p = newPage(
        "Trạng thái",
        "Tình trạng hoạt động của các module"
    )

    local tableFrame = create("Frame", {
        Size = UDim2.new(1, -5, 0, 0),
        AutomaticSize = Enum.AutomaticSize.Y,
        BackgroundColor3 = C.BLACK
    }, p)

    local heading = label(
        tableFrame,
        "Trạng thái",
        26,
        Color3.fromRGB(255, 255, 255),
        Enum.Font.GothamBold
    )

    heading.Position = UDim2.fromOffset(15, 10)
    heading.Size = UDim2.new(1, -30, 0, 48)

    local entries = {
        {"Dashboard", "Hoạt động", "green"},
        {"FPS / Ping", "Hoạt động", "green"},
        {"Player Tools", "Cơ bản", "green"},
        {"Monster Database", "Dữ liệu mẫu", "green"},
        {"Server Manager", "Chờ tích hợp", "orange"},
        {"Performance", "Cơ bản", "green"},
        {"Visual Settings", "Cơ bản", "green"}
    }

    local y = 65

    for _, item in ipairs(entries) do
        local separator = create("Frame", {
            Position = UDim2.fromOffset(0, y),
            Size = UDim2.new(1, 0, 0, 2),
            BackgroundColor3 = Color3.fromRGB(48, 48, 48),
            BorderSizePixel = 0
        }, tableFrame)

        local isOrange = item[3] == "orange"

        local badge = create("TextLabel", {
            Position = UDim2.fromOffset(15, y + 14),
            Size = UDim2.fromOffset(210, 43),
            BackgroundColor3 = isOrange
                and Color3.fromRGB(40, 20, 12)
                or Color3.fromRGB(0, 38, 20),
            Text = item[2],
            TextColor3 = isOrange and C.ORANGE or C.GREEN,
            TextSize = 19,
            Font = Enum.Font.GothamBold
        }, tableFrame)

        round(badge, 8)

        y += 76
    end

    create("Frame", {
        Position = UDim2.fromOffset(0, y),
        Size = UDim2.new(1, 0, 0, 2),
        BackgroundColor3 = Color3.fromRGB(48, 48, 48),
        BorderSizePixel = 0
    }, tableFrame)

    tableFrame.Size = UDim2.new(1, -5, 0, y + 5)
end

--==================================================
-- PLAYER TOOLS
--==================================================

pages["Player Tools"] = function()
    local p = newPage(
        "Player Tools ♡",
        "Character information and safe utilities"
    )

    local box, inner = section(p, "✦ CHARACTER INFORMATION")

    addInfo(inner, "Username: " .. player.Name)
    addInfo(inner, "Display Name: " .. player.DisplayName)
    addInfo(inner, "User ID: " .. player.UserId)

    local actions, actionInner = section(p, "♡ CHARACTER ACTIONS")

    addAction(actionInner, "Reset Character", function()
        local character = player.Character
        local humanoid = character and
            character:FindFirstChildOfClass("Humanoid")

        if humanoid then
            humanoid.Health = 0
        end
    end)

    addAction(actionInner, "Refresh Character Info", function()
        notify("Character information refreshed.")
    end)
end

--==================================================
-- VISUAL SETTINGS
--==================================================

pages["Visual Settings"] = function()
    local p = newPage(
        "Visual Settings ✧",
        "Visual options for your own experience"
    )

    local box, inner = section(p, "✦ VISUAL MODULES")

    addToggle(inner, "Visual Effects UI", false, function(state)
        notify("Visual Effects UI: " .. tostring(state))
    end)

    addToggle(inner, "Cute Theme", true, function(state)
        notify("Cute Theme: " .. tostring(state))
    end)

    addToggle(inner, "Compact Interface", false, function(state)
        notify("Compact Interface: " .. tostring(state))
    end)

    local info, infoInner = section(p, "♡ INFORMATION")

    addInfo(infoInner, "These settings control the dashboard.")
end

--==================================================
-- MONSTER DATABASE
--==================================================

pages["Monster Database"] = function()
    local p = newPage(
        "Monster Database ♡",
        "Manage monster data in your own game"
    )

    local box, inner = section(p, "✦ MONSTER SEARCH")

    local search = create("TextBox", {
        Size = UDim2.new(1, 0, 0, 40),
        BackgroundColor3 = C.PANEL2,
        PlaceholderText = "Search monster...",
        PlaceholderColor3 = C.MUTED,
        Text = "",
        TextColor3 = C.WHITE,
        TextSize = 13,
        Font = Enum.Font.Gotham,
        ClearTextOnFocus = false
    }, inner)

    round(search, 10)

    local list = create("Frame", {
        Size = UDim2.new(1, 0, 0, 0),
        AutomaticSize = Enum.AutomaticSize.Y,
        BackgroundTransparency = 1
    }, inner)

    create("UIListLayout", {
        Padding = UDim.new(0, 7)
    }, list)

    local monsters = {
        "Training Dummy",
        "Forest Guardian",
        "Shadow Beast",
        "Crystal Golem",
        "Ancient Dragon"
    }

    local function refresh()
        for _, child in ipairs(list:GetChildren()) do
            if child:IsA("TextButton") then
                child:Destroy()
            end
        end

        local query = string.lower(search.Text)

        for _, name in ipairs(monsters) do
            if string.find(string.lower(name), query, 1, true) then
                local b = button(list, "✧ " .. name)

                b.MouseButton1Click:Connect(function()
                    notify("Selected: " .. name)
                end)
            end
        end
    end

    search:GetPropertyChangedSignal("Text"):Connect(refresh)
    refresh()

    local info, infoInner = section(p, "♡ NOTE")
    addInfo(infoInner, "Sample data only.")
    addInfo(infoInner, "Connect your own NPC database to populate it.")
end

--==================================================
-- SERVER MANAGER
--==================================================

pages["Server Manager"] = function()
    local p = newPage(
        "Server Manager ☁",
        "Server information and connection tools"
    )

    local box, inner = section(p, "✦ CURRENT SERVER")

    addInfo(inner, "Place ID: " .. tostring(game.PlaceId))
    addInfo(inner, "Players: " .. tostring(#Players:GetPlayers()))

    local job = create("TextBox", {
        Size = UDim2.new(1, 0, 0, 40),
        BackgroundColor3 = C.PANEL2,
        Text = game.JobId,
        TextColor3 = C.WHITE,
        TextSize = 10,
        Font = Enum.Font.Code,
        ClearTextOnFocus = false
    }, inner)

    round(job, 8)

    local server, serverInner = section(p, "♡ SERVER FINDER")

    addInfo(serverInner, "Server discovery needs an authorized")
    addInfo(serverInner, "server listing and teleport integration.")

    addAction(serverInner, "Refresh Server Info", function()
        notify("Server information refreshed.")
    end)

    addAction(serverInner, "Copy Job ID", function()
        job:CaptureFocus()
        job.CursorPosition = #job.Text + 1
        notify("Select the Job ID and copy it.")
    end)
end

--==================================================
-- PERFORMANCE
--==================================================

pages["Performance"] = function()
    local p = newPage(
        "Performance Monitor ◈",
        "Live client performance information"
    )

    local box, inner = section(p, "✦ PERFORMANCE")

    addInfo(inner, "FPS is available on Dashboard.")
    addInfo(inner, "Ping is available on Dashboard.")
    addInfo(inner, "Player count is updated live.")

    local settings, settingsInner = section(p, "♡ DISPLAY OPTIONS")

    addToggle(settingsInner, "Low Detail UI Mode", false, function(state)
        notify("Low Detail UI: " .. tostring(state))
    end)
end

--==================================================
-- SETTINGS
--==================================================

pages["Settings"] = function()
    local p = newPage(
        "Settings ⚙",
        "Customize your dashboard"
    )

    local box, inner = section(p, "✦ INTERFACE")

    addToggle(inner, "Notifications", true, function(state)
        notify("Notifications: " .. tostring(state))
    end)

    addToggle(inner, "Cute Theme", true, function(state)
        notify("Cute Theme: " .. tostring(state))
    end)

    addAction(inner, "Hide Dashboard", function()
        gui.Enabled = false
    end)

    addAction(inner, "Show Dashboard", function()
        gui.Enabled = true
        notify("Welcome back!")
    end)
end

--==================================================
-- MENU NAVIGATION
--==================================================

local menuItems = {
    {"⌂", "Dashboard"},
    {"◷", "Trạng thái"},
    {"♧", "Player Tools"},
    {"✧", "Visual Settings"},
    {"♡", "Monster Database"},
    {"☁", "Server Manager"},
    {"◈", "Performance"},
    {"⚙", "Settings"}
}

local function openPage(name)
    if not pages[name] then return end

    pages[name]()

    for pageName, b in pairs(menuButtons) do
        b.BackgroundColor3 =
            pageName == name
            and Color3.fromRGB(83, 62, 119)
            or C.PANEL2
    end
end

for _, item in ipairs(menuItems) do
    local icon, name = item[1], item[2]

    local b = button(menu, icon .. "   " .. name, 40)
    menuButtons[name] = b

    b.MouseButton1Click:Connect(function()
        openPage(name)
    end)
end

--==================================================
-- DRAG WINDOW - MOUSE + TOUCH
--==================================================

local dragging = false
local dragStart
local startPosition
local activeInput

header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true
        dragStart = input.Position
        startPosition = main.Position

        if input.UserInputType == Enum.UserInputType.Touch then
            activeInput = input
        end
    end
end)

UIS.InputChanged:Connect(function(input)
    local isMouse = input.UserInputType ==
        Enum.UserInputType.MouseMovement

    local isTouch = activeInput and input == activeInput

    if dragging and (isMouse or isTouch) then
        local delta = input.Position - dragStart

        main.Position = UDim2.new(
            startPosition.X.Scale,
            startPosition.X.Offset + delta.X,
            startPosition.Y.Scale,
            startPosition.Y.Offset + delta.Y
        )
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

        dragging = false
        activeInput = nil
    end
end)

--==================================================
-- SHOW / HIDE WITH RIGHT SHIFT
--==================================================

UIS.InputBegan:Connect(function(input, processed)
    if processed then return end

    if input.KeyCode == Enum.KeyCode.RightShift then
        gui.Enabled = not gui.Enabled
    end
end)

--==================================================
-- START
--==================================================

openPage("Dashboard")
notify("KianBest V7.1 loaded successfully!")

print("KianBest V7.1 loaded!")
