
--========================================================
-- KIANBEST V7.3 | CUTE ANIME EDITION
-- LocalScript
-- StarterPlayer > StarterPlayerScripts
--========================================================

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- THAY BẰNG IMAGE ID CỦA ẢNH ANIME ĐÃ UPLOAD
local ANIME_IMAGE = "rbxassetid://YOUR_IMAGE_ID"

--========================================================
-- COLORS
--========================================================

local C = {
    Background = Color3.fromRGB(20, 15, 32),
    Panel = Color3.fromRGB(36, 27, 53),
    Panel2 = Color3.fromRGB(49, 37, 70),
    Purple = Color3.fromRGB(165, 115, 255),
    Pink = Color3.fromRGB(245, 165, 220),
    White = Color3.fromRGB(250, 245, 255),
    Muted = Color3.fromRGB(185, 170, 205),
    Green = Color3.fromRGB(45, 190, 100),
    Orange = Color3.fromRGB(255, 120, 65),
    Line = Color3.fromRGB(75, 60, 95)
}

--========================================================
-- CLEAN OLD GUI
--========================================================

local old = playerGui:FindFirstChild("KianBestV73")
if old then
    old:Destroy()
end

--========================================================
-- HELPERS
--========================================================

local function make(class, props, parent)
    local obj = Instance.new(class)

    for key, value in pairs(props or {}) do
        obj[key] = value
    end

    obj.Parent = parent
    return obj
end

local function corner(obj, radius)
    make("UICorner", {
        CornerRadius = UDim.new(0, radius)
    }, obj)
end

local function stroke(obj, color, thickness)
    make("UIStroke", {
        Color = color or C.Line,
        Thickness = thickness or 1
    }, obj)
end

local function text(parent, value, size, color, font)
    return make("TextLabel", {
        BackgroundTransparency = 1,
        Text = value,
        TextColor3 = color or C.White,
        TextSize = size or 14,
        Font = font or Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Center,
        TextWrapped = true
    }, parent)
end

local function newButton(parent, value, height)
    local b = make("TextButton", {
        Size = UDim2.new(1, 0, 0, height or 42),
        BackgroundColor3 = C.Panel2,
        Text = value,
        TextColor3 = C.White,
        TextSize = 13,
        Font = Enum.Font.GothamMedium,
        AutoButtonColor = false
    }, parent)

    corner(b, 10)

    b.MouseEnter:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.15), {
            BackgroundColor3 = Color3.fromRGB(78, 57, 108)
        }):Play()
    end)

    b.MouseLeave:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.15), {
            BackgroundColor3 = C.Panel2
        }):Play()
    end)

    return b
end

--========================================================
-- SCREEN GUI
--========================================================

local gui = make("ScreenGui", {
    Name = "KianBestV73",
    ResetOnSpawn = false,
    IgnoreGuiInset = true,
    DisplayOrder = 50
}, playerGui)

--========================================================
-- MAIN WINDOW
--========================================================

local main = make("Frame", {
    Name = "Main",
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromOffset(850, 550),
    BackgroundColor3 = C.Background,
    ClipsDescendants = true
}, gui)

corner(main, 18)
stroke(main, C.Purple, 2)

local uiScale = make("UIScale", {
    Scale = 1
}, main)

local function updateScale()
    local camera = workspace.CurrentCamera
    if not camera then return end

    local size = camera.ViewportSize

    uiScale.Scale = math.clamp(
        math.min(size.X / 900, size.Y / 600),
        0.45,
        1
    )
end

updateScale()

if workspace.CurrentCamera then
    workspace.CurrentCamera:GetPropertyChangedSignal(
        "ViewportSize"
    ):Connect(updateScale)
end

--========================================================
-- HEADER
--========================================================

local header = make("Frame", {
    Size = UDim2.new(1, 0, 0, 65),
    BackgroundColor3 = Color3.fromRGB(43, 29, 65)
}, main)

local logo = text(header, "✦ KIANBEST", 23, C.White,
    Enum.Font.GothamBold)

logo.Position = UDim2.fromOffset(20, 5)
logo.Size = UDim2.fromOffset(260, 30)

local subtitle = text(header,
    "V7.3 ULTIMATE ANIME EDITION ♡",
    11, C.Pink)

subtitle.Position = UDim2.fromOffset(22, 35)
subtitle.Size = UDim2.fromOffset(280, 20)

local close = make("TextButton", {
    Size = UDim2.fromOffset(38, 38),
    Position = UDim2.new(1, -50, 0, 13),
    BackgroundColor3 = Color3.fromRGB(100, 55, 90),
    Text = "×",
    TextColor3 = C.White,
    TextSize = 23,
    Font = Enum.Font.GothamBold
}, header)

corner(close, 10)

--========================================================
-- FLOATING ACCESSIBILITY BUTTON
--========================================================

local assist = make("TextButton", {
    Name = "AccessibilityButton",
    Size = UDim2.fromOffset(58, 58),
    Position = UDim2.new(1, -80, 0.65, 0),
    BackgroundColor3 = Color3.fromRGB(115, 70, 185),
    Text = "♡",
    TextColor3 = C.White,
    TextSize = 29,
    Font = Enum.Font.GothamBold,
    Visible = false,
    Active = true,
    AutoButtonColor = true,
    ZIndex = 100
}, gui)

corner(assist, 100)
stroke(assist, C.Pink, 2)

--========================================================
-- SIDEBAR
--========================================================

local sidebar = make("Frame", {
    Position = UDim2.fromOffset(12, 77),
    Size = UDim2.new(0, 175, 1, -89),
    BackgroundColor3 = C.Panel
}, main)

corner(sidebar, 13)

local menuHeading = text(sidebar, "♡ MAIN MENU", 13, C.Pink,
    Enum.Font.GothamBold)

menuHeading.Position = UDim2.fromOffset(13, 8)
menuHeading.Size = UDim2.new(1, -20, 0, 30)

local menu = make("ScrollingFrame", {
    Position = UDim2.fromOffset(8, 43),
    Size = UDim2.new(1, -16, 1, -50),
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    ScrollBarThickness = 3,
    ScrollBarImageColor3 = C.Purple,
    AutomaticCanvasSize = Enum.AutomaticSize.Y,
    CanvasSize = UDim2.new()
}, sidebar)

make("UIListLayout", {
    Padding = UDim.new(0, 8),
    SortOrder = Enum.SortOrder.LayoutOrder
}, menu)

--========================================================
-- CONTENT AREA
--========================================================

local content = make("Frame", {
    Position = UDim2.fromOffset(200, 77),
    Size = UDim2.new(1, -212, 1, -89),
    BackgroundTransparency = 1
}, main)

local pageTitle = text(content, "Welcome back ♡", 21,
    C.White, Enum.Font.GothamBold)

pageTitle.Position = UDim2.fromOffset(5, 0)
pageTitle.Size = UDim2.new(1, -10, 0, 34)

local pageSubtitle = text(content,
    "Cute control center • KianBest V7.3",
    12, C.Muted)

pageSubtitle.Position = UDim2.fromOffset(6, 34)
pageSubtitle.Size = UDim2.new(1, -10, 0, 23)

--========================================================
-- NOTIFICATION
--========================================================

local notification = make("TextLabel", {
    AnchorPoint = Vector2.new(1, 1),
    Position = UDim2.new(1, -15, 1, -12),
    Size = UDim2.fromOffset(245, 42),
    BackgroundColor3 = C.Panel2,
    Text = "",
    TextColor3 = C.White,
    TextSize = 12,
    Font = Enum.Font.GothamMedium,
    Visible = false,
    ZIndex = 120
}, main)

corner(notification, 10)

local noticeToken = 0

local function notify(message)
    noticeToken += 1
    local token = noticeToken

    notification.Text = "♡  " .. message
    notification.Visible = true

    task.delay(2.5, function()
        if token == noticeToken and notification.Parent then
            notification.Visible = false
        end
    end)
end

--========================================================
-- PAGE SYSTEM
--========================================================

local pages = {}
local menuButtons = {}

local function clearContent()
    for _, child in ipairs(content:GetChildren()) do
        if child ~= pageTitle and child ~= pageSubtitle then
            child:Destroy()
        end
    end
end

local function newPage(name, description)
    clearContent()

    pageTitle.Text = name
    pageSubtitle.Text = description or ""

    local scroll = make("ScrollingFrame", {
        Position = UDim2.fromOffset(0, 65),
        Size = UDim2.new(1, 0, 1, -68),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = C.Purple,
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        CanvasSize = UDim2.new()
    }, content)

    make("UIListLayout", {
        Padding = UDim.new(0, 10),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, scroll)

    return scroll
end

local function addSection(parent, heading)
    local frame = make("Frame", {
        Size = UDim2.new(1, -5, 0, 0),
        AutomaticSize = Enum.AutomaticSize.Y,
        BackgroundColor3 = C.Panel
    }, parent)

    corner(frame, 12)
    stroke(frame, C.Line, 1)

    local title = text(frame, heading, 14, C.Pink,
        Enum.Font.GothamBold)

    title.Position = UDim2.fromOffset(12, 7)
    title.Size = UDim2.new(1, -24, 0, 27)

    local inner = make("Frame", {
        Position = UDim2.fromOffset(12, 40),
        Size = UDim2.new(1, -24, 0, 0),
        AutomaticSize = Enum.AutomaticSize.Y,
        BackgroundTransparency = 1
    }, frame)

    make("UIListLayout", {
        Padding = UDim.new(0, 7),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, inner)

    make("UIPadding", {
        PaddingBottom = UDim.new(0, 12)
    }, inner)

    return inner
end

local function addInfo(parent, value)
    local l = text(parent, value, 12, C.Muted)
    l.Size = UDim2.new(1, 0, 0, 25)
    return l
end

local function addAction(parent, value, callback)
    local b = newButton(parent, value, 39)
    b.MouseButton1Click:Connect(callback)
    return b
end

--========================================================
-- DASHBOARD PAGE
--========================================================

pages["Dashboard"] = function()
    local p = newPage(
        "Welcome back, " .. player.DisplayName .. " ♡",
        "Cute control center • KianBest V7.3"
    )

    -- Anime image panel
    local anime = make("ImageLabel", {
        Size = UDim2.new(1, -5, 0, 175),
        BackgroundColor3 = C.Panel2,
        BackgroundTransparency = 0,
        Image = ANIME_IMAGE,
        ImageTransparency = 0.05,
        ScaleType = Enum.ScaleType.Crop
    }, p)

    corner(anime, 13)
    stroke(anime, C.Purple, 1)

    local animeCaption = make("TextLabel", {
        AnchorPoint = Vector2.new(0, 1),
        Position = UDim2.new(0, 10, 1, -10),
        Size = UDim2.new(1, -20, 0, 30),
        BackgroundColor3 = C.Background,
        BackgroundTransparency = 0.2,
        Text = "Welcome to KianBest ♡",
        TextColor3 = C.White,
        TextSize = 14,
        Font = Enum.Font.GothamBold
    }, anime)

    corner(animeCaption, 8)

    if ANIME_IMAGE:find("YOUR_IMAGE_ID") then
        anime.Image = ""
        animeCaption.Text = "Upload anime image and set your Image ID ♡"
    end

    -- LIVE STATUS
    local live = addSection(p, "✦ LIVE SYSTEM STATUS")

    local fpsLabel = addInfo(live, "FPS: Calculating...")
    local pingLabel = addInfo(live, "Ping: Calculating...")
    local playerLabel = addInfo(live, "Players: Calculating...")

    task.spawn(function()
        local frames = 0
        local elapsed = 0

        while gui.Parent and fpsLabel.Parent do
            local dt = RunService.RenderStepped:Wait()

            frames += 1
            elapsed += dt

            if elapsed >= 1 then
                fpsLabel.Text = "FPS: " ..
                    tostring(math.floor(frames / elapsed))

                local ok, result = pcall(function()
                    return math.floor(player:GetNetworkPing() * 1000)
                end)

                pingLabel.Text = ok and
                    ("Ping: " .. result .. " ms")
                    or "Ping: N/A"

                playerLabel.Text = "Players: " ..
                    tostring(#Players:GetPlayers())

                frames = 0
                elapsed = 0
            end
        end
    end)

    -- QUICK ACCESS
    local quick = addSection(p, "♡ QUICK ACCESS")

    addAction(quick, "☁ Server Manager", function()
        pages["Server Manager"]()
    end)

    addAction(quick, "✧ Visual Settings", function()
        pages["Visual Settings"]()
    end)

    addAction(quick, "◈ Performance", function()
        pages["Performance"]()
    end)

    -- WELCOME
    local welcome = addSection(p, "🌸 WELCOME")

    addInfo(welcome, "Thank you for using KianBest!")
    addInfo(welcome, "Have a wonderful day ♡")
end

--========================================================
-- STATUS PAGE
--========================================================

pages["Trạng thái"] = function()
    local p = newPage("Trạng thái", "Tình trạng các module")

    local box = addSection(p, "SYSTEM STATUS")

    local entries = {
        {"Dashboard", "Hoạt động", true},
        {"FPS / Ping", "Hoạt động", true},
        {"Player Tools", "Cơ bản", true},
        {"Monster Database", "Dữ liệu mẫu", true},
        {"Server Manager", "Chờ tích hợp", false},
        {"Performance", "Cơ bản", true},
        {"Visual Settings", "Cơ bản", true}
    }

    for _, item in ipairs(entries) do
        local row = make("Frame", {
            Size = UDim2.new(1, 0, 0, 53),
            BackgroundTransparency = 1
        }, box)

        make("Frame", {
            Position = UDim2.new(0, 0, 1, -1),
            Size = UDim2.new(1, 0, 0, 1),
            BackgroundColor3 = C.Line,
            BorderSizePixel = 0
        }, row)

        local isReady = item[3]

        local badge = make("TextLabel", {
            Position = UDim2.fromOffset(5, 5),
            Size = UDim2.fromOffset(205, 40),
            BackgroundColor3 = isReady
                and Color3.fromRGB(0, 38, 20)
                or Color3.fromRGB(45, 24, 12),
            Text = item[2],
            TextColor3 = isReady and C.Green or C.Orange,
            TextSize = 17,
            Font = Enum.Font.GothamBold
        }, row)

        corner(badge, 8)
    end
end

--========================================================
-- PLAYER TOOLS
--========================================================

pages["Player Tools"] = function()
    local p = newPage("Player Tools", "Thông tin nhân vật")

    local box = addSection(p, "✦ PLAYER INFORMATION")

    addInfo(box, "Username: " .. player.Name)
    addInfo(box, "Display Name: " .. player.DisplayName)
    addInfo(box, "User ID: " .. player.UserId)

    addAction(box, "Refresh Information", function()
        notify("Đã cập nhật thông tin!")
    end)
end

--========================================================
-- VISUAL SETTINGS
--========================================================

pages["Visual Settings"] = function()
    local p = newPage("Visual Settings", "Tùy chỉnh giao diện")

    local box = addSection(p, "✦ THEME INFORMATION")

    addInfo(box, "Theme: Purple Pastel")
    addInfo(box, "Style: Cute Anime")
    addInfo(box, "Status: Green / Orange")

    addAction(box, "Show theme information", function()
        notify("KianBest Purple Theme ♡")
    end)
end

--========================================================
-- MONSTER DATABASE
--========================================================

pages["Monster Database"] = function()
    local p = newPage("Monster Database", "Dữ liệu NPC mẫu")

    local box = addSection(p, "✦ SAMPLE MONSTERS")

    local names = {
        "Training Dummy",
        "Forest Guardian",
        "Shadow Beast",
        "Crystal Golem",
        "Ancient Dragon"
    }

    for _, name in ipairs(names) do
        addInfo(box, "♡  " .. name)
    end

    addInfo(box, "Đây là dữ liệu mẫu, chưa quét NPC.")
end

--========================================================
-- SERVER MANAGER
--========================================================

pages["Server Manager"] = function()
    local p = newPage("Server Manager", "Thông tin máy chủ hiện tại")

    local box = addSection(p, "✦ CURRENT SERVER")

    addInfo(box, "Place ID: " .. tostring(game.PlaceId))
    addInfo(box, "Players: " .. tostring(#Players:GetPlayers()))

    addInfo(box, "Server finder cần tích hợp API hợp lệ.")

    addAction(box, "Refresh Server Information", function()
        notify("Đã làm mới thông tin server!")
    end)
end

--========================================================
-- PERFORMANCE
--========================================================

pages["Performance"] = function()
    local p = newPage("Performance", "Theo dõi hiệu năng")

    local box = addSection(p, "✦ MONITOR")

    addInfo(box, "FPS và Ping được hiển thị ở Dashboard.")
    addInfo(box, "Số người chơi cập nhật tự động.")

    addAction(box, "Refresh", function()
        notify("Performance page refreshed!")
    end)
end

--========================================================
-- SETTINGS
--========================================================

pages["Settings"] = function()
    local p = newPage("Settings", "Cài đặt KianBest")

    local box = addSection(p, "✦ ACCESSIBILITY")

    addAction(box, "Hide Dashboard", function()
        main.Visible = false
        assist.Visible = true
    end)

    addAction(box, "Show Dashboard", function()
        main.Visible = true
        assist.Visible = false
    end)

    addInfo(box, "Right Shift: Toggle on PC.")
    addInfo(box, "Floating heart: Open on mobile.")
end

--========================================================
-- NAVIGATION
--========================================================

local items = {
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

    for pageName, button in pairs(menuButtons) do
        button.BackgroundColor3 =
            pageName == name
            and Color3.fromRGB(88, 60, 130)
            or C.Panel2
    end
end

for _, item in ipairs(items) do
    local name = item[2]

    local b = newButton(menu, item[1] .. "   " .. name, 40)

    menuButtons[name] = b

    b.MouseButton1Click:Connect(function()
        openPage(name)
    end)
end

--========================================================
-- MINIMIZE / RESTORE
--========================================================

close.MouseButton1Click:Connect(function()
    main.Visible = false
    assist.Visible = true
end)

assist.MouseButton1Click:Connect(function()
    main.Visible = true
    assist.Visible = false
end)

--========================================================
-- DRAG FLOATING BUTTON
--========================================================

local dragging = false
local dragStart
local startPosition
local activeTouch
local moved = false

assist.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true
        moved = false
        dragStart = input.Position
        startPosition = assist.Position

        if input.UserInputType == Enum.UserInputType.Touch then
            activeTouch = input
        end
    end
end)

UIS.InputChanged:Connect(function(input)
    local isMouse =
        input.UserInputType == Enum.UserInputType.MouseMovement

    local isTouch =
        activeTouch and input == activeTouch

    if dragging and (isMouse or isTouch) then
        local delta = input.Position - dragStart

        if math.abs(delta.X) > 5 or math.abs(delta.Y) > 5 then
            moved = true
        end

        assist.Position = UDim2.new(
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
        activeTouch = nil
    end
end)

--========================================================
-- RIGHT SHIFT FOR PC
--========================================================

UIS.InputBegan:Connect(function(input, processed)
    if processed then return end

    if input.KeyCode == Enum.KeyCode.RightShift then
        main.Visible = not main.Visible
        assist.Visible = not main.Visible
    end
end)

--========================================================
-- START
--========================================================

openPage("Dashboard")
notify("KianBest V7.3 đã khởi động!")

print("KianBest V7.3 loaded successfully!")
