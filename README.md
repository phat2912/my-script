
--========================================================
-- KIANBEST V5.4 GALAXY ANIME EDITION
-- MASTER SOURCE FILE
--
-- GITHUB: KianBestV54.lua
--
-- INSTALL:
-- 1. Copy SERVER SECTION into Script:
--    ServerScriptService/KianBestAdmin
--
-- 2. Copy CLIENT SECTION into LocalScript:
--    StarterPlayer/StarterPlayerScripts/KianBestDashboard
--
-- 3. Change ADMIN USER ID in SERVER SECTION.
--
-- Commands:
-- .tp UserId
-- .kick UserId
--========================================================


--########################################################
--                    SERVER SECTION
--########################################################

-- COPY FROM HERE INTO A NORMAL SCRIPT
-- LOCATION: ServerScriptService

local SERVER_CODE = [==[
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local ADMINS = {
    [123456789] = true, -- CHANGE TO YOUR USER ID
}

local remote = ReplicatedStorage:FindFirstChild("KianBestRemote")

if not remote then
    remote = Instance.new("RemoteEvent")
    remote.Name = "KianBestRemote"
    remote.Parent = ReplicatedStorage
end

local cooldowns = {}
local COOLDOWN = 3

local function notify(player, message, success)
    if player and player.Parent == Players then
        remote:FireClient(player, "STATUS", {
            message = message,
            success = success
        })
    end
end

local function findPlayer(userId)
    for _, target in ipairs(Players:GetPlayers()) do
        if target.UserId == userId then
            return target
        end
    end
end

local function executeCommand(admin, message)
    if not ADMINS[admin.UserId] then
        return
    end

    local command, id = message:match("^%.(%a+)%s+(%d+)%s*$")
    if not command or not id then
        return
    end

    command = command:lower()
    id = tonumber(id)

    if command ~= "tp" and command ~= "kick" then
        return
    end

    local now = os.clock()
    if now - (cooldowns[admin.UserId] or 0) < COOLDOWN then
        notify(admin, "Please wait before using another command.", false)
        return
    end

    cooldowns[admin.UserId] = now

    local target = findPlayer(id)

    if not target then
        notify(admin, "Player not found in this server.", false)
        return
    end

    if target == admin then
        notify(admin, "You cannot target yourself.", false)
        return
    end

    if command == "tp" then
        local character = admin.Character
        local targetCharacter = target.Character

        local targetRoot = targetCharacter
            and targetCharacter:FindFirstChild("HumanoidRootPart")

        if not character or not targetRoot then
            notify(admin, "Character is not ready.", false)
            return
        end

        character:PivotTo(targetRoot.CFrame * CFrame.new(3, 0, 0))
        notify(admin, "Teleported to " .. target.Name, true)

    elseif command == "kick" then
        local targetName = target.Name
        target:Kick("Removed by a game administrator.")
        notify(admin, "Kicked " .. targetName, true)
    end
end

local function setupPlayer(player)
    player.Chatted:Connect(function(message)
        executeCommand(player, message)
    end)
end

Players.PlayerAdded:Connect(setupPlayer)

for _, player in ipairs(Players:GetPlayers()) do
    setupPlayer(player)
end

print("[KianBest V5.4] Server commands ready")
]==]


--########################################################
--                    CLIENT SECTION
--########################################################

-- COPY FROM HERE INTO A LOCALSCRIPT
-- LOCATION: StarterPlayer/StarterPlayerScripts

local CLIENT_CODE = [==[
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local remote = ReplicatedStorage:WaitForChild("KianBestRemote")

-- CONFIGURATION

local CONFIG = {
    Title = "KIANBEST V5.4",
    BackgroundImage = "",
    Accent = Color3.fromRGB(167, 139, 250),
    Background = Color3.fromRGB(20, 17, 35),
    Panel = Color3.fromRGB(34, 29, 55),
    Text = Color3.fromRGB(245, 240, 255),
    Muted = Color3.fromRGB(170, 160, 195)
}

local accent = CONFIG.Accent
local dashboardOpen = true
local frameCount = 0
local fps = 0
local lastFPSUpdate = os.clock()
local oldLighting

-- CLEANUP

local old = playerGui:FindFirstChild("KianBestV54")
if old then
    old:Destroy()
end

local gui = Instance.new("ScreenGui")
gui.Name = "KianBestV54"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = playerGui

-- UI HELPERS

local function round(object, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius or 10)
    c.Parent = object
end

local function outline(object, color, thickness)
    local s = Instance.new("UIStroke")
    s.Color = color or accent
    s.Thickness = thickness or 1
    s.Transparency = 0.2
    s.Parent = object
end

local function frame(parent, size, position, color)
    local f = Instance.new("Frame")
    f.Size = size
    f.Position = position
    f.BackgroundColor3 = color or CONFIG.Panel
    f.BorderSizePixel = 0
    f.Parent = parent
    round(f, 10)
    return f
end

local function label(parent, text, size, position, textSize)
    local l = Instance.new("TextLabel")
    l.Size = size
    l.Position = position
    l.BackgroundTransparency = 1
    l.Text = text
    l.TextColor3 = CONFIG.Text
    l.Font = Enum.Font.GothamMedium
    l.TextSize = textSize or 13
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = parent
    return l
end

local function button(parent, text, callback)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, -4, 0, 36)
    b.BackgroundColor3 = CONFIG.Panel
    b.Text = text
    b.TextColor3 = CONFIG.Text
    b.Font = Enum.Font.GothamSemibold
    b.TextSize = 12
    b.Parent = parent
    round(b, 8)
    outline(b, accent, 1)

    b.Activated:Connect(function()
        if callback then
            callback(b)
        end
    end)

    return b
end

local function info(parent, text)
    local box = frame(
        parent,
        UDim2.new(1, -4, 0, 36),
        UDim2.new(),
        CONFIG.Panel
    )

    local textLabel = label(
        box,
        text,
        UDim2.new(1, -16, 1, 0),
        UDim2.new(0, 9, 0, 0),
        12
    )

    return box, textLabel
end

-- MAIN WINDOW

local main = frame(
    gui,
    UDim2.new(0, 520, 0, 350),
    UDim2.new(0.5, -260, 0.5, -175),
    CONFIG.Background
)

main.Name = "Dashboard"
main.ClipsDescendants = true
outline(main, accent, 1.5)

local sizeLimit = Instance.new("UISizeConstraint")
sizeLimit.MinSize = Vector2.new(300, 280)
sizeLimit.MaxSize = Vector2.new(650, 500)
sizeLimit.Parent = main

local scale = Instance.new("UIScale")
scale.Parent = main

local function updateScale()
    local camera = workspace.CurrentCamera
    if not camera then return end

    local viewport = camera.ViewportSize
    scale.Scale = math.min(1, viewport.X / 600, viewport.Y / 430)
end

updateScale()

if workspace.CurrentCamera then
    workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(updateScale)
end

-- BACKGROUND

local background = Instance.new("ImageLabel")
background.Name = "AnimeBackground"
background.Size = UDim2.fromScale(1, 1)
background.BackgroundTransparency = 1
background.Image = CONFIG.BackgroundImage
background.ImageTransparency = 0.78
background.ScaleType = Enum.ScaleType.Crop
background.ZIndex = 0
background.Parent = main

local overlay = Instance.new("Frame")
overlay.Size = UDim2.fromScale(1, 1)
overlay.BackgroundColor3 = CONFIG.Background
overlay.BackgroundTransparency = 0.12
overlay.BorderSizePixel = 0
overlay.ZIndex = 0
overlay.Parent = main

-- HEADER

local header = frame(
    main,
    UDim2.new(1, 0, 0, 48),
    UDim2.new(),
    CONFIG.Panel
)

header.Name = "Header"
header.ZIndex = 2

local title = label(
    header,
    CONFIG.Title .. "  ✦",
    UDim2.new(0.7, 0, 1, 0),
    UDim2.new(0, 14, 0, 0),
    16
)

title.Font = Enum.Font.GothamBold
title.ZIndex = 3

local close = button(header, "×", function()
    main.Visible = false
    dashboardOpen = false
end)

close.Size = UDim2.new(0, 32, 0, 32)
close.Position = UDim2.new(1, -42, 0, 7)
close.ZIndex = 4

-- DRAG SUPPORT: TOUCH AND MOUSE

local function draggable(object, handle)
    local dragging = false
    local dragStart
    local startPosition
    local activeInput

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            activeInput = input
            dragStart = input.Position
            startPosition = object.Position
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if not dragging then return end

        if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then
            local delta = input.Position - dragStart

            object.Position = UDim2.new(
                startPosition.X.Scale,
                startPosition.X.Offset + delta.X,
                startPosition.Y.Scale,
                startPosition.Y.Offset + delta.Y
            )
        end
    end)

    UIS.InputEnded:Connect(function(input)
        if input == activeInput
        or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
            activeInput = nil
        end
    end)
end

draggable(main, header)

-- FLOATING HOME BUTTON

local home = Instance.new("TextButton")
home.Name = "HomeButton"
home.Size = UDim2.new(0, 52, 0, 52)
home.Position = UDim2.new(0, 18, 0.55, 0)
home.BackgroundColor3 = CONFIG.Panel
home.Text = "⌂"
home.TextColor3 = CONFIG.Text
home.TextSize = 25
home.Font = Enum.Font.GothamBold
home.Parent = gui
round(home, 50)
outline(home, accent, 2)

home.Activated:Connect(function()
    dashboardOpen = not dashboardOpen
    main.Visible = dashboardOpen
end)

draggable(home, home)

-- NAVIGATION AND PAGES

local nav = frame(
    main,
    UDim2.new(0, 115, 1, -58),
    UDim2.new(0, 8, 0, 53),
    CONFIG.Panel
)

local content = frame(
    main,
    UDim2.new(1, -137, 1, -62),
    UDim2.new(0, 128, 0, 53),
    Color3.fromRGB(25, 21, 42)
)

local pages = {}

local function createPage(name)
    local page = Instance.new("ScrollingFrame")
    page.Name = name
    page.Size = UDim2.fromScale(1, 1)
    page.BackgroundTransparency = 1
    page.BorderSizePixel = 0
    page.ScrollBarThickness = 3
    page.ScrollBarImageColor3 = accent
    page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    page.CanvasSize = UDim2.new(0, 0, 0, 0)
    page.Visible = false
    page.Parent = content

    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 8)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = page

    local padding = Instance.new("UIPadding")
    padding.PaddingTop = UDim.new(0, 10)
    padding.PaddingLeft = UDim.new(0, 10)
    padding.PaddingRight = UDim.new(0, 10)
    padding.PaddingBottom = UDim.new(0, 10)
    padding.Parent = page

    pages[name] = page
    return page
end

local function showPage(name)
    for pageName, page in pairs(pages) do
        page.Visible = pageName == name
    end
end

local function section(page, text)
    local l = label(
        page,
        text,
        UDim2.new(1, -4, 0, 27),
        UDim2.new(),
        14
    )
    l.Font = Enum.Font.GothamBold
    l.TextColor3 = accent
end

-- HOME PAGE

local homePage = createPage("HOME")

section(homePage, "✦ KIANBEST CONTROL CENTER")
info(homePage, "Welcome, " .. player.DisplayName)
info(homePage, "User ID: " .. player.UserId)
info(homePage, "Version: V5.4 Galaxy Edition")
info(homePage, "Platform: " ..
    (UIS.TouchEnabled and "Touch" or "Keyboard / Mouse"))
info(homePage, "Dashboard: Online")

-- PERFORMANCE PAGE

local performancePage = createPage("PERFORMANCE")

section(performancePage, "✦ PERFORMANCE MONITOR")

local _, fpsText = info(performancePage, "FPS: Calculating...")
local _, pingText = info(performancePage, "Ping: Calculating...")

button(performancePage, "Toggle Compact Mode", function(b)
    if main.Size.X.Offset > 400 then
        main.Size = UDim2.new(0, 360, 0, 300)
        b.Text = "Restore Dashboard Size"
    else
        main.Size = UDim2.new(0, 520, 0, 350)
        b.Text = "Toggle Compact Mode"
    end
end)

button(performancePage, "Reset Dashboard Position", function()
    main.Position = UDim2.new(0.5, -260, 0.5, -175)
end)

-- PLAYER PAGE

local playerPage = createPage("PLAYERS")

section(playerPage, "✦ PLAYER DIRECTORY")

local playerList = Instance.new("Frame")
playerList.Size = UDim2.new(1, -4, 0, 0)
playerList.AutomaticSize = Enum.AutomaticSize.Y
playerList.BackgroundTransparency = 1
playerList.Parent = playerPage

local playerLayout = Instance.new("UIListLayout")
playerLayout.Padding = UDim.new(0, 5)
playerLayout.Parent = playerList

local function refreshPlayers()
    for _, child in ipairs(playerList:GetChildren()) do
        if child:IsA("TextLabel") then
            child:Destroy()
        end
    end

    for _, target in ipairs(Players:GetPlayers()) do
        local item = Instance.new("TextLabel")
        item.Size = UDim2.new(1, -4, 0, 30)
        item.BackgroundColor3 = CONFIG.Panel
        item.TextColor3 = CONFIG.Text
        item.Font = Enum.Font.Gotham
        item.TextSize = 11
        item.TextXAlignment = Enum.TextXAlignment.Left
        item.Text = "  " .. target.Name .. "  |  " .. target.UserId
        item.Parent = playerList
        round(item, 6)
    end
end

refreshPlayers()
Players.PlayerAdded:Connect(refreshPlayers)
Players.PlayerRemoving:Connect(refreshPlayers)

button(playerPage, "Refresh Player List", refreshPlayers)

-- WORLD PAGE

local worldPage = createPage("WORLD")

section(worldPage, "✦ WORLD SETTINGS")

local _, lightingStatus = info(worldPage, "Lighting: Default")

button(worldPage, "Toggle Galaxy Lighting", function(b)
    if not oldLighting then
        oldLighting = {
            Brightness = Lighting.Brightness,
            ClockTime = Lighting.ClockTime,
            Ambient = Lighting.Ambient
        }

        Lighting.Brightness = 2
        Lighting.ClockTime = 0
        Lighting.Ambient = Color3.fromRGB(100, 80, 160)

        lightingStatus.Text = "Lighting: Galaxy"
        b.Text = "Restore Lighting"
    else
        Lighting.Brightness = oldLighting.Brightness
        Lighting.ClockTime = oldLighting.ClockTime
        Lighting.Ambient = oldLighting.Ambient

        oldLighting = nil
        lightingStatus.Text = "Lighting: Default"
        b.Text = "Toggle Galaxy Lighting"
    end
end)

-- SERVER PAGE

local serverPage = createPage("SERVER")

section(serverPage, "✦ SERVER INFORMATION")

info(serverPage, "Place ID: " .. game.PlaceId)
info(serverPage, "Job ID: " .. game.JobId)

local _, playerCount = info(
    serverPage,
    "Players: " .. #Players:GetPlayers()
)

local function updatePlayerCount()
    playerCount.Text = "Players: " .. #Players:GetPlayers()
end

Players.PlayerAdded:Connect(updatePlayerCount)
Players.PlayerRemoving:Connect(function()
    task.defer(updatePlayerCount)
end)

-- SETTINGS PAGE

local settingsPage = createPage("SETTINGS")

section(settingsPage, "✦ COLOR STUDIO")

local colors = {
    {"Galaxy Purple", Color3.fromRGB(167, 139, 250)},
    {"Ocean Blue", Color3.fromRGB(80, 160, 255)},
    {"Rose Pink", Color3.fromRGB(255, 130, 190)},
    {"Mint Green", Color3.fromRGB(100, 230, 180)},
    {"Sunset Orange", Color3.fromRGB(255, 160, 100)},
    {"Ice Cyan", Color3.fromRGB(100, 220, 255)},
    {"Gold", Color3.fromRGB(255, 210, 100)},
    {"White", Color3.fromRGB(235, 235, 245)}
}

local function setTheme(color)
    accent = color

    for _, object in ipairs(gui:GetDescendants()) do
        if object:IsA("UIStroke") then
            object.Color = color
        end
    end

    for _, page in pairs(pages) do
        for _, object in ipairs(page:GetChildren()) do
            if object:IsA("TextLabel") then
                object.TextColor3 = color
            end
        end
    end
end

for _, data in ipairs(colors) do
    local name, color = data[1], data[2]

    button(settingsPage, "● " .. name, function()
        setTheme(color)
    end)
end

section(settingsPage, "✦ BACKGROUND IMAGE")

local imageInput = Instance.new("TextBox")
imageInput.Size = UDim2.new(1, -4, 0, 36)
imageInput.BackgroundColor3 = CONFIG.Panel
imageInput.TextColor3 = CONFIG.Text
imageInput.PlaceholderColor3 = CONFIG.Muted
imageInput.PlaceholderText = "Image ID or rbxassetid://..."
imageInput.Text = ""
imageInput.ClearTextOnFocus = false
imageInput.Font = Enum.Font.Gotham
imageInput.TextSize = 12
imageInput.Parent = settingsPage
round(imageInput, 8)

button(settingsPage, "Apply Anime Background", function()
    local value = imageInput.Text:gsub("%s+", "")

    if value:match("^%d+$") then
        value = "rbxassetid://" .. value
    end

    if value:match("^rbxassetid://%d+$") then
        background.Image = value
    else
        warn("[KianBest] Invalid image asset ID")
    end
end)

-- NAV BUTTONS

local navItems = {
    {"HOME", "⌂"},
    {"PERFORMANCE", "◈"},
    {"PLAYERS", "♙"},
    {"WORLD", "✧"},
    {"SERVER", "◎"},
    {"SETTINGS", "⚙"}
}

for index, data in ipairs(navItems) do
    local name, icon = data[1], data[2]

    local b = button(nav, icon .. "  " .. name, function()
        showPage(name)
    end)

    b.Position = UDim2.new(0, 5, 0, 6 + (index - 1) * 43)
    b.Size = UDim2.new(1, -10, 0, 36)
    b.TextSize = 10
end

showPage("HOME")

-- FPS + PING

RunService.RenderStepped:Connect(function()
    frameCount += 1

    local now = os.clock()

    if now - lastFPSUpdate >= 1 then
        fps = frameCount
        frameCount = 0
        lastFPSUpdate = now

        fpsText.Text = "FPS: " .. fps

        local success, ping = pcall(function()
            return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
        end)

        if success then
            pingText.Text = "Ping: " .. math.floor(ping) .. " ms"
        else
            pingText.Text = "Ping: Unavailable"
        end
    end
end)

-- ADMIN COMMAND STATUS NOTIFICATIONS

remote.OnClientEvent:Connect(function(action, data)
    if action ~= "STATUS" or type(data) ~= "table" then
        return
    end

    local color = data.success
        and Color3.fromRGB(100, 230, 150)
        or Color3.fromRGB(255, 100, 110)

    local notification = Instance.new("TextLabel")
    notification.Size = UDim2.new(0, 280, 0, 42)
    notification.Position = UDim2.new(0.5, -140, 0, 20)
    notification.BackgroundColor3 = CONFIG.Panel
    notification.TextColor3 = color
    notification.Text = tostring(data.message)
    notification.TextSize = 12
    notification.Font = Enum.Font.GothamBold
    notification.Parent = gui
    round(notification, 10)
    outline(notification, color, 1)

    task.delay(3, function()
        if notification.Parent then
            notification:Destroy()
        end
    end)
end)

print("[KianBest V5.4] Dashboard loaded")
]==]


--########################################################
-- END OF MASTER SOURCE
--########################################################

-- NOTE:
-- SERVER_CODE and CLIENT_CODE above are source templates.
-- They are intentionally not executed by this master file.
-- Copy each block into its matching Roblox script.
