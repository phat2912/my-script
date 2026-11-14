
--========================================================
-- KIANBEST V5.5 | GALAXY ANIME EXECUTOR EDITION
-- Single-file client dashboard
--========================================================

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")

local LP = Players.LocalPlayer
local PG = LP:WaitForChild("PlayerGui")

-- CONFIG
local CONFIG = {
    Name = "KIANBEST V5.5",
    Accent = Color3.fromRGB(167, 139, 250),
    Background = Color3.fromRGB(18, 15, 34),
    Panel = Color3.fromRGB(32, 27, 53),
    Text = Color3.fromRGB(245, 240, 255),
    Muted = Color3.fromRGB(170, 160, 195),
    AnimeImage = ""
}

-- CLEAN OLD UI
local old = PG:FindFirstChild("KianBestV55")
if old then old:Destroy() end

local Accent = CONFIG.Accent
local Open = true
local CurrentPage = "HOME"
local EffectsOn = false
local LightingSaved = nil
local Connections = {}
local EffectObjects = {}

local function track(connection)
    table.insert(Connections, connection)
    return connection
end

local function New(class, props, parent)
    local obj = Instance.new(class)
    for k, v in pairs(props or {}) do
        obj[k] = v
    end
    obj.Parent = parent
    return obj
end

local function Round(obj, radius)
    New("UICorner", {
        CornerRadius = UDim.new(0, radius or 10)
    }, obj)
end

local function Outline(obj, color, thickness)
    return New("UIStroke", {
        Color = color or Accent,
        Thickness = thickness or 1,
        Transparency = 0.2
    }, obj)
end

local function MakeLabel(parent, text, size, position, textSize)
    return New("TextLabel", {
        Size = size,
        Position = position,
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = CONFIG.Text,
        Font = Enum.Font.GothamMedium,
        TextSize = textSize or 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextWrapped = true
    }, parent)
end

local function MakeButton(parent, text, callback)
    local b = New("TextButton", {
        Size = UDim2.new(1, -4, 0, 36),
        BackgroundColor3 = CONFIG.Panel,
        Text = text,
        TextColor3 = CONFIG.Text,
        Font = Enum.Font.GothamSemibold,
        TextSize = 12,
        AutoButtonColor = true
    }, parent)

    Round(b, 9)
    Outline(b, Accent, 1)

    b.Activated:Connect(function()
        if callback then callback(b) end
    end)

    return b
end

local function MakeInfo(parent, text)
    local box = New("Frame", {
        Size = UDim2.new(1, -4, 0, 36),
        BackgroundColor3 = CONFIG.Panel,
        BorderSizePixel = 0
    }, parent)

    Round(box, 8)

    local txt = MakeLabel(
        box,
        text,
        UDim2.new(1, -16, 1, 0),
        UDim2.new(0, 9, 0, 0),
        12
    )

    return box, txt
end

-- SCREEN GUI
local GUI = New("ScreenGui", {
    Name = "KianBestV55",
    ResetOnSpawn = false,
    IgnoreGuiInset = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling
}, PG)

-- MAIN WINDOW
local Main = New("Frame", {
    Name = "Dashboard",
    Size = UDim2.fromOffset(520, 355),
    Position = UDim2.new(0.5, -260, 0.5, -177),
    BackgroundColor3 = CONFIG.Background,
    BorderSizePixel = 0,
    ClipsDescendants = true
}, GUI)

Round(Main, 14)
Outline(Main, Accent, 1.5)

local UIScale = New("UIScale", {}, Main)

local function UpdateScale()
    local camera = workspace.CurrentCamera
    if not camera then return end

    local v = camera.ViewportSize
    UIScale.Scale = math.min(1, v.X / 590, v.Y / 430)
end

UpdateScale()

if workspace.CurrentCamera then
    track(workspace.CurrentCamera:GetPropertyChangedSignal(
        "ViewportSize"
    ):Connect(UpdateScale))
end

-- ANIME BACKGROUND
local Anime = New("ImageLabel", {
    Name = "AnimeBackground",
    Size = UDim2.fromScale(1, 1),
    BackgroundTransparency = 1,
    Image = CONFIG.AnimeImage,
    ImageTransparency = 0.72,
    ScaleType = Enum.ScaleType.Crop,
    ZIndex = 0
}, Main)

local Overlay = New("Frame", {
    Size = UDim2.fromScale(1, 1),
    BackgroundColor3 = CONFIG.Background,
    BackgroundTransparency = 0.12,
    BorderSizePixel = 0,
    ZIndex = 0
}, Main)

-- HEADER
local Header = New("Frame", {
    Size = UDim2.new(1, 0, 0, 48),
    BackgroundColor3 = CONFIG.Panel,
    BorderSizePixel = 0,
    ZIndex = 2
}, Main)

MakeLabel(
    Header,
    "✦  KIANBEST V5.5",
    UDim2.new(0.75, 0, 1, 0),
    UDim2.new(0, 14, 0, 0),
    16
).Font = Enum.Font.GothamBold

local Close = New("TextButton", {
    Size = UDim2.fromOffset(32, 32),
    Position = UDim2.new(1, -42, 0, 8),
    BackgroundColor3 = Color3.fromRGB(95, 55, 100),
    Text = "×",
    TextColor3 = CONFIG.Text,
    TextSize = 22,
    Font = Enum.Font.GothamBold,
    ZIndex = 5
}, Header)

Round(Close, 9)

Close.Activated:Connect(function()
    Main.Visible = false
    Open = false
end)

-- DRAG FUNCTION
local function MakeDraggable(object, handle)
    local dragging = false
    local dragStart
    local startPos
    local activeInput

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            activeInput = input
            dragStart = input.Position
            startPos = object.Position
        end
    end)

    track(UIS.InputChanged:Connect(function(input)
        if not dragging then return end

        if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then
            local delta = input.Position - dragStart

            object.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end))

    track(UIS.InputEnded:Connect(function(input)
        if input == activeInput then
            dragging = false
            activeInput = nil
        elseif input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
            activeInput = nil
        end
    end))
end

MakeDraggable(Main, Header)

-- FLOATING HOME
local Home = New("TextButton", {
    Name = "KianBestHome",
    Size = UDim2.fromOffset(54, 54),
    Position = UDim2.new(0, 18, 0.55, 0),
    BackgroundColor3 = CONFIG.Panel,
    Text = "⌂",
    TextColor3 = CONFIG.Text,
    TextSize = 27,
    Font = Enum.Font.GothamBold
}, GUI)

Round(Home, 50)
Outline(Home, Accent, 2)

Home.Activated:Connect(function()
    Open = not Open
    Main.Visible = Open
end)

MakeDraggable(Home, Home)

-- CONTENT LAYOUT
local Nav = New("Frame", {
    Size = UDim2.new(0, 118, 1, -58),
    Position = UDim2.new(0, 7, 0, 53),
    BackgroundColor3 = CONFIG.Panel,
    BorderSizePixel = 0
}, Main)

Round(Nav, 10)

local Content = New("Frame", {
    Size = UDim2.new(1, -137, 1, -62),
    Position = UDim2.new(0, 128, 0, 53),
    BackgroundColor3 = Color3.fromRGB(24, 20, 41),
    BorderSizePixel = 0,
    ClipsDescendants = true
}, Main)

Round(Content, 10)

local Pages = {}

local function CreatePage(name)
    local page = New("ScrollingFrame", {
        Name = name,
        Size = UDim2.fromScale(1, 1),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = Accent,
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        Visible = false
    }, Content)

    local layout = New("UIListLayout", {
        Padding = UDim.new(0, 8),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, page)

    New("UIPadding", {
        PaddingTop = UDim.new(0, 10),
        PaddingLeft = UDim.new(0, 10),
        PaddingRight = UDim.new(0, 10),
        PaddingBottom = UDim.new(0, 10)
    }, page)

    Pages[name] = page
    return page
end

local function Section(page, text)
    local l = MakeLabel(
        page,
        text,
        UDim2.new(1, -4, 0, 27),
        UDim2.new(),
        14
    )
    l.Font = Enum.Font.GothamBold
    l.TextColor3 = Accent
end

local function ShowPage(name)
    for pageName, page in pairs(Pages) do
        page.Visible = pageName == name
    end
    CurrentPage = name
end

-- HOME PAGE
local HomePage = CreatePage("HOME")

Section(HomePage, "✦ CONTROL CENTER")
MakeInfo(HomePage, "Welcome, " .. LP.DisplayName)
MakeInfo(HomePage, "Username: @" .. LP.Name)
MakeInfo(HomePage, "User ID: " .. LP.UserId)
MakeInfo(HomePage, "Version: V5.5 Galaxy")
MakeInfo(HomePage, "Platform: " ..
    (UIS.TouchEnabled and "Mobile / Touch" or "PC"))

-- PERFORMANCE PAGE
local Performance = CreatePage("PERFORMANCE")

Section(Performance, "✦ PERFORMANCE MONITOR")

local _, FPSText = MakeInfo(Performance, "FPS: Calculating...")
local _, PingText = MakeInfo(Performance, "Ping: Calculating...")

MakeButton(Performance, "Compact Mode", function(b)
    if Main.Size.X.Offset > 400 then
        Main.Size = UDim2.fromOffset(370, 300)
        b.Text = "Restore Size"
    else
        Main.Size = UDim2.fromOffset(520, 355)
        b.Text = "Compact Mode"
    end
end)

MakeButton(Performance, "Reset Window Position", function()
    Main.Position = UDim2.new(0.5, -260, 0.5, -177)
end)

-- PLAYER PAGE
local PlayerPage = CreatePage("PLAYERS")

Section(PlayerPage, "✦ PLAYER DIRECTORY")

local PlayerList = New("Frame", {
    Size = UDim2.new(1, -4, 0, 0),
    AutomaticSize = Enum.AutomaticSize.Y,
    BackgroundTransparency = 1
}, PlayerPage)

New("UIListLayout", {
    Padding = UDim.new(0, 5),
    SortOrder = Enum.SortOrder.LayoutOrder
}, PlayerList)

local function RefreshPlayers()
    for _, child in ipairs(PlayerList:GetChildren()) do
        if child:IsA("TextLabel") then
            child:Destroy()
        end
    end

    for _, target in ipairs(Players:GetPlayers()) do
        local item = New("TextLabel", {
            Size = UDim2.new(1, -4, 0, 30),
            BackgroundColor3 = CONFIG.Panel,
            TextColor3 = CONFIG.Text,
            Font = Enum.Font.Gotham,
            TextSize = 11,
            TextXAlignment = Enum.TextXAlignment.Left,
            Text = "  " .. target.Name .. "  |  " .. target.UserId
        }, PlayerList)

        Round(item, 6)
    end
end

RefreshPlayers()
track(Players.PlayerAdded:Connect(RefreshPlayers))
track(Players.PlayerRemoving:Connect(RefreshPlayers))

MakeButton(PlayerPage, "Refresh Player List", RefreshPlayers)

-- WORLD PAGE
local World = CreatePage("WORLD")

Section(World, "✦ WORLD SETTINGS")

local _, LightingText = MakeInfo(World, "Lighting: Default")

MakeButton(World, "Toggle Galaxy Lighting", function(b)
    if not LightingSaved then
        LightingSaved = {
            Brightness = Lighting.Brightness,
            ClockTime = Lighting.ClockTime,
            Ambient = Lighting.Ambient
        }

        Lighting.Brightness = 2
        Lighting.ClockTime = 0
        Lighting.Ambient = Color3.fromRGB(100, 80, 160)

        LightingText.Text = "Lighting: Galaxy"
        b.Text = "Restore Lighting"
    else
        Lighting.Brightness = LightingSaved.Brightness
        Lighting.ClockTime = LightingSaved.ClockTime
        Lighting.Ambient = LightingSaved.Ambient

        LightingSaved = nil
        LightingText.Text = "Lighting: Default"
        b.Text = "Toggle Galaxy Lighting"
    end
end)

-- EFFECTS PAGE
local Effects = CreatePage("EFFECTS")

Section(Effects, "✦ PARTICLE EFFECTS")

local EffectStatus = MakeInfo(Effects, "Effects: OFF")

local function ClearEffects()
    for _, obj in ipairs(EffectObjects) do
        if obj and obj.Parent then
            obj:Destroy()
        end
    end
    table.clear(EffectObjects)
end

local function CreateParticles(symbol)
    ClearEffects()

    for i = 1, 18 do
        local particle = New("TextLabel", {
            Size = UDim2.fromOffset(25, 25),
            Position = UDim2.new(math.random(), 0, math.random(), 0),
            BackgroundTransparency = 1,
            Text = symbol,
            TextSize = math.random(12, 22),
            TextColor3 = Accent,
            Font = Enum.Font.GothamBold,
            ZIndex = 8
        }, GUI)

        table.insert(EffectObjects, particle)

        task.spawn(function()
            while particle.Parent and EffectsOn do
                local x = math.random(5, 95) / 100
                local y = math.random(5, 90) / 100

                local tween = TweenService:Create(
                    particle,
                    TweenInfo.new(
                        math.random(3, 7),
                        Enum.EasingStyle.Linear
                    ),
                    {Position = UDim2.new(x, 0, y, 0)}
                )

                tween:Play()
                tween.Completed:Wait()
            end
        end)
    end
end

MakeButton(Effects, "Snow ❄", function()
    EffectsOn = true
    EffectStatus:FindFirstChildOfClass("TextLabel").Text = "Effects: Snow"
    CreateParticles("❄")
end)

MakeButton(Effects, "Sakura 🌸", function()
    EffectsOn = true
    EffectStatus:FindFirstChildOfClass("TextLabel").Text = "Effects: Sakura"
    CreateParticles("🌸")
end)

MakeButton(Effects, "Stars ✦", function()
    EffectsOn = true
    EffectStatus:FindFirstChildOfClass("TextLabel").Text = "Effects: Stars"
    CreateParticles("✦")
end)

MakeButton(Effects, "Hearts ♥", function()
    EffectsOn = true
    EffectStatus:FindFirstChildOfClass("TextLabel").Text = "Effects: Hearts"
    CreateParticles("♥")
end)

MakeButton(Effects, "Stop Effects", function()
    EffectsOn = false
    ClearEffects()
    EffectStatus:FindFirstChildOfClass("TextLabel").Text = "Effects: OFF"
end)

-- SERVER PAGE
local Server = CreatePage("SERVER")

Section(Server, "✦ SERVER INFORMATION")

MakeInfo(Server, "Place ID: " .. game.PlaceId)
MakeInfo(Server, "Job ID: " .. game.JobId)

local _, CountText = MakeInfo(
    Server,
    "Players: " .. #Players:GetPlayers()
)

local function UpdateCount()
    CountText.Text = "Players: " .. #Players:GetPlayers()
end

track(Players.PlayerAdded:Connect(UpdateCount))
track(Players.PlayerRemoving:Connect(function()
    task.defer(UpdateCount)
end))

-- SETTINGS PAGE
local Settings = CreatePage("SETTINGS")

Section(Settings, "✦ COLOR STUDIO")

local Colors = {
    {"Galaxy Purple", Color3.fromRGB(167, 139, 250)},
    {"Ocean Blue", Color3.fromRGB(80, 160, 255)},
    {"Rose Pink", Color3.fromRGB(255, 130, 190)},
    {"Mint Green", Color3.fromRGB(100, 230, 180)},
    {"Sunset Orange", Color3.fromRGB(255, 160, 100)},
    {"Ice Cyan", Color3.fromRGB(100, 220, 255)},
    {"Gold", Color3.fromRGB(255, 210, 100)},
    {"White", Color3.fromRGB(235, 235, 245)}
}

local function SetTheme(color)
    Accent = color

    for _, obj in ipairs(GUI:GetDescendants()) do
        if obj:IsA("UIStroke") then
            obj.Color = color
        elseif obj:IsA("ScrollingFrame") then
            obj.ScrollBarImageColor3 = color
        end
    end
end

for _, data in ipairs(Colors) do
    local name, color = data[1], data[2]

    MakeButton(Settings, "● " .. name, function()
        SetTheme(color)
    end)
end

Section(Settings, "✦ ANIME BACKGROUND")

local ImageInput = New("TextBox", {
    Size = UDim2.new(1, -4, 0, 36),
    BackgroundColor3 = CONFIG.Panel,
    TextColor3 = CONFIG.Text,
    PlaceholderColor3 = CONFIG.Muted,
    PlaceholderText = "Image ID or rbxassetid://...",
    Text = "",
    ClearTextOnFocus = false,
    Font = Enum.Font.Gotham,
    TextSize = 12
}, Settings)

Round(ImageInput, 8)

MakeButton(Settings, "Apply Background", function()
    local value = ImageInput.Text:gsub("%s+", "")

    if value:match("^%d+$") then
        value = "rbxassetid://" .. value
    end

    if value:match("^rbxassetid://%d+$") then
        Anime.Image = value
    else
        warn("[KianBest] Invalid image ID")
    end
end)

MakeButton(Settings, "Reset Theme", function()
    SetTheme(CONFIG.Accent)
end)

-- NAVIGATION
local NavItems = {
    {"HOME", "⌂"},
    {"PERFORMANCE", "◈"},
    {"PLAYERS", "♙"},
    {"WORLD", "✧"},
    {"EFFECTS", "❄"},
    {"SERVER", "◎"},
    {"SETTINGS", "⚙"}
}

for index, data in ipairs(NavItems) do
    local name, icon = data[1], data[2]

    local b = MakeButton(Nav, icon .. "  " .. name, function()
        ShowPage(name)
    end)

    b.Position = UDim2.new(0, 5, 0, 5 + (index - 1) * 42)
    b.Size = UDim2.new(1, -10, 0, 35)
    b.TextSize = 10
end

ShowPage("HOME")

-- FPS + PING MONITOR
track(RunService.RenderStepped:Connect(function()
    frameCount += 1

    local now = os.clock()

    if now - lastFPSUpdate >= 1 then
        fps = frameCount
        frameCount = 0
        lastFPSUpdate = now

        FPSText.Text = "FPS: " .. fps

        local success, ping = pcall(function()
            return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
        end)

        PingText.Text = success
            and ("Ping: " .. math.floor(ping) .. " ms")
            or "Ping: Unavailable"
    end
end))

-- NOTIFICATION
local function Notify(message, success)
    local color = success
        and Color3.fromRGB(100, 230, 150)
        or Color3.fromRGB(255, 100, 110)

    local n = New("TextLabel", {
        Size = UDim2.fromOffset(280, 42),
        Position = UDim2.new(0.5, -140, 0, 18),
        BackgroundColor3 = CONFIG.Panel,
        TextColor3 = color,
        Text = tostring(message),
        TextSize = 12,
        Font = Enum.Font.GothamBold,
        ZIndex = 20
    }, GUI)

    Round(n, 10)
    Outline(n, color, 1)

    task.delay(3, function()
        if n.Parent then n:Destroy() end
    end)
end

-- CLEANUP
GUI.Destroying:Connect(function()
    EffectsOn = false
    ClearEffects()

    for _, connection in ipairs(Connections) do
        pcall(function()
            connection:Disconnect()
        end)
    end

    if LightingSaved then
        Lighting.Brightness = LightingSaved.Brightness
        Lighting.ClockTime = LightingSaved.ClockTime
        Lighting.Ambient = LightingSaved.Ambient
    end
end)

print("[KianBest V5.5] Galaxy Anime Dashboard loaded")
Notify("KianBest V5.5 loaded successfully!", true)
