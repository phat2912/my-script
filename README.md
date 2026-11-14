
--==================================================
-- KIANBEST V5.2 ULTRA | COMPACT EDITION
-- Client-side Roblox Dashboard
-- Default Theme: Purple #A78BFA
--==================================================

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")

local LP = Players.LocalPlayer
local PG = LP:WaitForChild("PlayerGui")

--==================================================
-- CONFIG
--==================================================

local Config = {
    Theme = "Purple",
    Hex = "#A78BFA",
    Language = "VI",
    Effect = "Snow",
    Background = "",
    BackgroundEnabled = true,
    Particles = true,
    Compact = true,
    FastMode = false,
    ParticleAmount = 25,
    Opacity = 0.08,
}

local Themes = {
    Purple  = Color3.fromRGB(167,139,250),
    Sakura  = Color3.fromRGB(249,168,212),
    Ocean   = Color3.fromRGB(56,189,248),
    Emerald = Color3.fromRGB(52,211,153),
    Ruby    = Color3.fromRGB(251,113,133),
    Gold    = Color3.fromRGB(251,191,36),
    Cyber   = Color3.fromRGB(34,211,238),
    White   = Color3.fromRGB(226,232,240),
}

local Accent = Themes.Purple
local Enabled = true
local CurrentTab = "HOME"
local FPS = 0
local MinFPS = math.huge
local MaxFPS = 0
local FPSCounter = 0
local FPSTime = 0

--==================================================
-- OPTIONAL CONFIG SAVE
-- Uses executor file APIs only when available.
--==================================================

local function SaveConfig()
    if type(writefile) ~= "function" then
        return false
    end

    local success = pcall(function()
        writefile(
            "KianBest_V52_Config.json",
            HttpService:JSONEncode(Config)
        )
    end)

    return success
end

local function LoadConfig()
    if type(readfile) ~= "function"
        or type(isfile) ~= "function" then
        return
    end

    pcall(function()
        if isfile("KianBest_V52_Config.json") then
            local data = HttpService:JSONDecode(
                readfile("KianBest_V52_Config.json")
            )

            for k, v in pairs(data) do
                if Config[k] ~= nil then
                    Config[k] = v
                end
            end
        end
    end)

    if Themes[Config.Theme] then
        Accent = Themes[Config.Theme]
    end
end

LoadConfig()

--==================================================
-- LANGUAGE
--==================================================

local Texts = {
    VI = {
        HOME = "Trang chủ",
        PERFORMANCE = "Hiệu năng",
        PLAYERS = "Người chơi",
        WORLD = "Thế giới",
        SERVER = "Máy chủ",
        SETTINGS = "Cài đặt",
        FPS = "FPS thực tế",
        PING = "Ping",
        MIN = "FPS thấp nhất",
        MAX = "FPS cao nhất",
        THEME = "Màu giao diện",
        EFFECT = "Hiệu ứng",
        BACKGROUND = "Hình nền",
        SAVE = "Lưu cấu hình",
        RESET = "Khôi phục",
        FAST = "Chế độ tăng tốc",
        SNOW = "Tuyết rơi",
        SAKURA = "Hoa anh đào",
        STARS = "Sao lấp lánh",
        HEARTS = "Trái tim",
        NONE = "Tắt",
        SEARCH = "Tìm người chơi...",
        ONLINE = "ĐANG HOẠT ĐỘNG",
        OFFLINE = "KHÔNG HOẠT ĐỘNG",
    },

    EN = {
        HOME = "Home",
        PERFORMANCE = "Performance",
        PLAYERS = "Players",
        WORLD = "World",
        SERVER = "Server",
        SETTINGS = "Settings",
        FPS = "Real FPS",
        PING = "Ping",
        MIN = "Minimum FPS",
        MAX = "Maximum FPS",
        THEME = "Theme Color",
        EFFECT = "Effects",
        BACKGROUND = "Background",
        SAVE = "Save Config",
        RESET = "Reset",
        FAST = "Boost Mode",
        SNOW = "Snow Fall",
        SAKURA = "Sakura Fall",
        STARS = "Star Glow",
        HEARTS = "Floating Hearts",
        NONE = "Disabled",
        SEARCH = "Search players...",
        ONLINE = "ACTIVE",
        OFFLINE = "INACTIVE",
    }
}

local function T(key)
    return Texts[Config.Language][key] or key
end

--==================================================
-- GUI ROOT
--==================================================

local Old = PG:FindFirstChild("KianBestV52")
if Old then Old:Destroy() end

local Gui = Instance.new("ScreenGui")
Gui.Name = "KianBestV52"
Gui.ResetOnSpawn = false
Gui.DisplayOrder = 999
Gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
Gui.Parent = PG

local Scale = Instance.new("UIScale")
Scale.Parent = Gui

local function UpdateScale()
    local camera = workspace.CurrentCamera
    if not camera then return end

    local size = camera.ViewportSize
    Scale.Scale = math.clamp(
        math.min(size.X / 430, size.Y / 760),
        0.72,
        1.12
    )
end

UpdateScale()

if workspace.CurrentCamera then
    workspace.CurrentCamera:GetPropertyChangedSignal(
        "ViewportSize"
    ):Connect(UpdateScale)
end

--==================================================
-- UI HELPERS
--==================================================

local function New(class, props, parent)
    local obj = Instance.new(class)

    for k, v in pairs(props or {}) do
        obj[k] = v
    end

    obj.Parent = parent
    return obj
end

local function Corner(obj, radius)
    return New("UICorner", {
        CornerRadius = UDim.new(0, radius or 10)
    }, obj)
end

local function Stroke(obj, color, thickness)
    return New("UIStroke", {
        Color = color or Accent,
        Thickness = thickness or 1
    }, obj)
end

local function Label(parent, text, size, position, color)
    return New("TextLabel", {
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = color or Color3.new(1,1,1),
        TextSize = size or 13,
        Font = Enum.Font.GothamMedium,
        Position = position or UDim2.new(),
        Size = UDim2.new(1,0,0,24),
        TextXAlignment = Enum.TextXAlignment.Left
    }, parent)
end

local function Button(parent, text, size, position)
    local b = New("TextButton", {
        Text = text,
        Size = size,
        Position = position,
        BackgroundColor3 = Color3.fromRGB(48,38,67),
        TextColor3 = Color3.new(1,1,1),
        Font = Enum.Font.GothamSemibold,
        TextSize = 12,
        AutoButtonColor = true
    }, parent)

    Corner(b, 9)
    return b
end

--==================================================
-- MAIN WINDOW
--==================================================

local Main = New("Frame", {
    Size = UDim2.fromOffset(340, 430),
    Position = UDim2.new(0.5,-170,0.5,-215),
    BackgroundColor3 = Color3.fromRGB(25,19,40),
    BorderSizePixel = 0,
    Visible = false,
    ClipsDescendants = true
}, Gui)

Corner(Main, 16)
local MainStroke = Stroke(Main, Accent, 1.5)

-- BACKGROUND IMAGE
local Background = New("ImageLabel", {
    Name = "AnimeBackground",
    Size = UDim2.fromScale(1,1),
    BackgroundTransparency = 1,
    Image = Config.Background,
    ImageTransparency = Config.Opacity,
    ScaleType = Enum.ScaleType.Crop,
    Visible = Config.BackgroundEnabled,
    ZIndex = 0
}, Main)

local Overlay = New("Frame", {
    Size = UDim2.fromScale(1,1),
    BackgroundColor3 = Color3.fromRGB(22,15,37),
    BackgroundTransparency = 0.13,
    BorderSizePixel = 0,
    ZIndex = 0
}, Main)

-- HEADER
local Header = New("Frame", {
    Size = UDim2.new(1,0,0,52),
    BackgroundColor3 = Color3.fromRGB(34,24,53),
    BorderSizePixel = 0,
    ZIndex = 2
}, Main)

local Title = Label(
    Header,
    "KianBest V5.2 ULTRA",
    15,
    UDim2.fromOffset(13,7),
    Color3.fromRGB(238,225,255)
)

Title.Font = Enum.Font.GothamBold

local Subtitle = Label(
    Header,
    "COMPACT EDITION",
    9,
    UDim2.fromOffset(14,29),
    Color3.fromRGB(183,160,220)
)

local Close = Button(
    Header,
    "×",
    UDim2.fromOffset(32,32),
    UDim2.new(1,-42,0,10)
)

Close.TextSize = 20
Close.Activated:Connect(function()
    Main.Visible = false
end)

--==================================================
-- NAVIGATION
--==================================================

local Nav = New("ScrollingFrame", {
    Position = UDim2.fromOffset(0,52),
    Size = UDim2.new(1,0,0,43),
    BackgroundColor3 = Color3.fromRGB(30,23,46),
    BorderSizePixel = 0,
    ScrollBarThickness = 2,
    ScrollingDirection = Enum.ScrollingDirection.X,
    CanvasSize = UDim2.fromOffset(600,0),
    ZIndex = 2
}, Main)

local NavLayout = New("UIListLayout", {
    FillDirection = Enum.FillDirection.Horizontal,
    Padding = UDim.new(0,5),
    SortOrder = Enum.SortOrder.LayoutOrder
}, Nav)

local Content = New("Frame", {
    Position = UDim2.fromOffset(0,95),
    Size = UDim2.new(1,0,-0,335),
    BackgroundTransparency = 1,
    ZIndex = 2
}, Main)

local Pages = {}
local TabButtons = {}

local TabNames = {
    "HOME",
    "PERFORMANCE",
    "PLAYERS",
    "WORLD",
    "SERVER",
    "SETTINGS"
}

for i, name in ipairs(TabNames) do
    local page = New("ScrollingFrame", {
        Name = name,
        Size = UDim2.fromScale(1,1),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = Accent,
        CanvasSize = UDim2.fromOffset(0,0),
        Visible = i == 1
    }, Content)

    local layout = New("UIListLayout", {
        Padding = UDim.new(0,8),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, page)

    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(
        function()
            page.CanvasSize = UDim2.fromOffset(
                0, layout.AbsoluteContentSize.Y + 15
            )
        end
    )

    Pages[name] = page

    local tab = Button(
        Nav,
        name,
        UDim2.fromOffset(94,34),
        UDim2.new()
    )

    tab.LayoutOrder = i
    TabButtons[name] = tab

    tab.Activated:Connect(function()
        CurrentTab = name

        for n, p in pairs(Pages) do
            p.Visible = n == name
        end

        for n, b in pairs(TabButtons) do
            b.BackgroundColor3 =
                n == name and Accent
                or Color3.fromRGB(48,38,67)
        end
    end)
end

--==================================================
-- COMPONENTS
--==================================================

local function Section(parent, text)
    local l = Label(parent, text, 13)
    l.Size = UDim2.new(1,-20,0,27)
    l.Position = UDim2.fromOffset(10,0)
    l.Font = Enum.Font.GothamBold
    l.TextColor3 = Color3.fromRGB(205,178,246)
    return l
end

local function InfoCard(parent, title, value)
    local card = New("Frame", {
        Size = UDim2.fromOffset(145,65),
        BackgroundColor3 = Color3.fromRGB(43,33,62),
        BorderSizePixel = 0
    }, parent)

    Corner(card, 10)

    Label(
        card,title,10,
        UDim2.fromOffset(9,7),
        Color3.fromRGB(185,164,220)
    )

    local v = Label(
        card,value,21,
        UDim2.fromOffset(9,28),
        Color3.fromRGB(255,255,255)
    )

    v.Name = "Value"
    return v
end

local function MakeToggle(parent, text, callback)
    local row = New("Frame", {
        Size = UDim2.new(1,-20,0,42),
        BackgroundColor3 = Color3.fromRGB(42,32,59),
        BorderSizePixel = 0
    }, parent)

    Corner(row, 9)

    Label(row,text,12,UDim2.fromOffset(10,8))

    local on = false

    local toggle = Button(
        row,"OFF",
        UDim2.fromOffset(55,27),
        UDim2.new(1,-65,0,7)
    )

    toggle.Activated:Connect(function()
        on = not on

        toggle.Text = on and "ON" or "OFF"
        toggle.BackgroundColor3 = on
            and Accent
            or Color3.fromRGB(75,65,87)

        callback(on)
    end)

    return row
end

--==================================================
-- HOME PAGE
--==================================================

local HomePage = Pages.HOME

Section(HomePage, "SYSTEM OVERVIEW")

local Metrics = New("Frame", {
    Size = UDim2.new(1,-20,0,145),
    BackgroundTransparency = 1,
    Position = UDim2.fromOffset(10,0)
}, HomePage)

local FPSValue = InfoCard(Metrics,"REAL FPS","0")
FPSValue.Parent.Position = UDim2.fromOffset(0,0)

local PingValue = InfoCard(Metrics,"YOUR PING","--")
PingValue.Parent.Position = UDim2.fromOffset(155,0)

local MinValue = InfoCard(Metrics,"MIN FPS","--")
MinValue.Parent.Position = UDim2.fromOffset(0,75)

local MaxValue = InfoCard(Metrics,"MAX FPS","--")
MaxValue.Parent.Position = UDim2.fromOffset(155,75)

local SystemInfo = New("Frame", {
    Size = UDim2.new(1,-20,0,75),
    BackgroundColor3 = Color3.fromRGB(42,32,59)
}, HomePage)

Corner(SystemInfo,10)

Label(
    SystemInfo,
    "SYSTEM INFORMATION",
    11,
    UDim2.fromOffset(10,5)
)

local DeviceLabel = Label(
    SystemInfo,
    UIS.TouchEnabled and "DEVICE: TOUCH" or "DEVICE: DESKTOP",
    10,
    UDim2.fromOffset(10,29),
    Color3.fromRGB(200,180,230)
)

local PlayerCount = Label(
    SystemInfo,
    "PLAYERS: 0",
    10,
    UDim2.fromOffset(10,48),
    Color3.fromRGB(200,180,230)
)

--==================================================
-- PERFORMANCE PAGE
--==================================================

local PerformancePage = Pages.PERFORMANCE

Section(PerformancePage,"PERFORMANCE TOOLS")

MakeToggle(
    PerformancePage,
    "Fast Mode - Reduce UI Effects",
    function(enabled)
        Config.FastMode = enabled

        Overlay.BackgroundTransparency =
            enabled and 0.04 or 0.13

        Config.ParticleAmount =
            enabled and 5 or 25

        SaveConfig()
    end
)

MakeToggle(
    PerformancePage,
    "Reduce Decorative Particles",
    function(enabled)
        Config.Particles = not enabled
    end
)

MakeToggle(
    PerformancePage,
    "Compact Interface",
    function(enabled)
        Config.Compact = enabled
        Main.Size = enabled
            and UDim2.fromOffset(340,430)
            or UDim2.fromOffset(390,500)
    end
)

MakeToggle(
    PerformancePage,
    "Background Image",
    function(enabled)
        Config.BackgroundEnabled = enabled
        Background.Visible = enabled
    end
)

--==================================================
-- PLAYER PAGE
--==================================================

local PlayerPage = Pages.PLAYERS

Section(PlayerPage,"PLAYER MONITOR")

local Search = New("TextBox", {
    Size = UDim2.new(1,-20,0,35),
    BackgroundColor3 = Color3.fromRGB(43,33,62),
    TextColor3 = Color3.new(1,1,1),
    PlaceholderText = "Search players...",
    Text = "",
    ClearTextOnFocus = false,
    TextSize = 12,
    Font = Enum.Font.Gotham
}, PlayerPage)

Corner(Search,9)

local PlayerContainer = New("Frame", {
    Size = UDim2.new(1,-20,0,0),
    AutomaticSize = Enum.AutomaticSize.Y,
    BackgroundTransparency = 1
}, PlayerPage)

local PlayerLayout = New("UIListLayout", {
    Padding = UDim.new(0,7)
}, PlayerContainer)

local function ClearPlayerCards()
    for _, obj in ipairs(PlayerContainer:GetChildren()) do
        if obj:IsA("Frame") then
            obj:Destroy()
        end
    end
end

local function RefreshPlayers()
    ClearPlayerCards()

    local query = Search.Text:lower()

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.Name:lower():find(query,1,true)
        or plr.DisplayName:lower():find(query,1,true) then

            local card = New("Frame", {
                Size = UDim2.new(1,0,0,77),
                BackgroundColor3 = Color3.fromRGB(43,33,62),
                BorderSizePixel = 0
            }, PlayerContainer)

            Corner(card,9)

            Label(
                card,
                plr.DisplayName,
                13,
                UDim2.fromOffset(10,5)
            )

            Label(
                card,
                "@"..plr.Name.." | ID: "..plr.UserId,
                9,
                UDim2.fromOffset(10,25),
                Color3.fromRGB(188,169,216)
            )

            Label(
                card,
                plr == LP and "YOU | FPS: Local Monitor"
                or "FPS: Not available",
                9,
                UDim2.fromOffset(10,44),
                Color3.fromRGB(160,220,180)
            )

            Label(
                card,
                "PING: Not available",
                9,
                UDim2.fromOffset(10,59),
                Color3.fromRGB(180,200,240)
            )

            local copy = Button(
                card,
                "COPY ID",
                UDim2.fromOffset(65,25),
                UDim2.new(1,-75,0,25)
            )

            copy.Activated:Connect(function()
                if type(setclipboard) == "function" then
                    pcall(setclipboard,tostring(plr.UserId))
                end
            end)
        end
    end
end

Search:GetPropertyChangedSignal("Text"):Connect(RefreshPlayers)

Players.PlayerAdded:Connect(RefreshPlayers)
Players.PlayerRemoving:Connect(RefreshPlayers)

RefreshPlayers()

--==================================================
-- WORLD PAGE
--==================================================

local WorldPage = Pages.WORLD

Section(WorldPage,"WORLD INFORMATION")

local WorldText = Label(
    WorldPage,
    "NPC / MONSTER INFORMATION\n\nThis panel is ready for authorized game data.",
    12
)

WorldText.Size = UDim2.new(1,-20,0,70)
WorldText.Position = UDim2.fromOffset(10,0)
WorldText.TextWrapped = true

local MonsterNote = New("Frame", {
    Size = UDim2.new(1,-20,0,60),
    BackgroundColor3 = Color3.fromRGB(43,33,62)
}, WorldPage)

Corner(MonsterNote,9)

Label(
    MonsterNote,
    "Monster Database",
    13,
    UDim2.fromOffset(10,7)
)

Label(
    MonsterNote,
    "Connect your own authorized NPC data source.",
    9,
    UDim2.fromOffset(10,30),
    Color3.fromRGB(190,175,220)
)

--==================================================
-- SERVER PAGE
--==================================================

local ServerPage = Pages.SERVER

Section(ServerPage,"SERVER INFORMATION")

local function ServerCard(text)
    local box = New("Frame", {
        Size = UDim2.new(1,-20,0,47),
        BackgroundColor3 = Color3.fromRGB(43,33,62)
    }, ServerPage)

    Corner(box,9)

    Label(
        box,text,10,
        UDim2.fromOffset(10,12)
    )

    return box
end

ServerCard("PLACE ID: "..tostring(game.PlaceId))
ServerCard("JOB ID: "..tostring(game.JobId))
ServerCard("PLAYERS: "..#Players:GetPlayers())

local ServerRefresh = Button(
    ServerPage,
    "REFRESH INFORMATION",
    UDim2.new(1,-20,0,35),
    UDim2.new()
)

ServerRefresh.Activated:Connect(function()
    for _, item in ipairs(ServerPage:GetChildren()) do
        if item:IsA("Frame") then
            local label = item:FindFirstChildOfClass("TextLabel")
            if label and label.Text:find("PLAYERS:") then
                label.Text = "PLAYERS: "..#Players:GetPlayers()
            end
        end
    end
end)

--==================================================
-- SETTINGS PAGE
--==================================================

local SettingsPage = Pages.SETTINGS

Section(SettingsPage,"COLOR STUDIO")

local ThemeNames = {
    "Purple","Sakura","Ocean","Emerald",
    "Ruby","Gold","Cyber","White"
}

for _, name in ipairs(ThemeNames) do
    local b = Button(
        SettingsPage,
        name,
        UDim2.new(1,-20,0,32),
        UDim2.new()
    )

    b.Activated:Connect(function()
        Accent = Themes[name]
        Config.Theme = name
        Config.Hex = name

        MainStroke.Color = Accent

        for _, button in pairs(TabButtons) do
            button.BackgroundColor3 =
                button.Text == CurrentTab
                and Accent
                or Color3.fromRGB(48,38,67)
        end

        SaveConfig()
    end)
end

Section(SettingsPage,"CUSTOM HEX COLOR")

local HexInput = New("TextBox", {
    Size = UDim2.new(1,-20,0,35),
    BackgroundColor3 = Color3.fromRGB(43,33,62),
    TextColor3 = Color3.new(1,1,1),
    PlaceholderText = "#A78BFA",
    Text = "",
    ClearTextOnFocus = false
}, SettingsPage)

Corner(HexInput,8)

local ApplyHex = Button(
    SettingsPage,
    "APPLY CUSTOM COLOR",
    UDim2.new(1,-20,0,34),
    UDim2.new()
)

ApplyHex.Activated:Connect(function()
    local hex = HexInput.Text:gsub("#","")

    if #hex == 6 and hex:match("^%x+$") then
        local r = tonumber(hex:sub(1,2),16)
        local g = tonumber(hex:sub(3,4),16)
        local b = tonumber(hex:sub(5,6),16)

        Accent = Color3.fromRGB(r,g,b)
        Config.Theme = "Custom"
        Config.Hex = "#" .. hex:upper()

        MainStroke.Color = Accent
        SaveConfig()
    end
end)

Section(SettingsPage,"LANGUAGE")

local LanguageButton = Button(
    SettingsPage,
    "VI / EN",
    UDim2.new(1,-20,0,35),
    UDim2.new()
)

LanguageButton.Activated:Connect(function()
    Config.Language =
        Config.Language == "VI" and "EN" or "VI"

    LanguageButton.Text =
        Config.Language == "VI" and "Tiếng Việt"
        or "English"

    SaveConfig()
end)

Section(SettingsPage,"BACKGROUND")

local ImageInput = New("TextBox", {
    Size = UDim2.new(1,-20,0,35),
    BackgroundColor3 = Color3.fromRGB(43,33,62),
    TextColor3 = Color3.new(1,1,1),
    PlaceholderText = "Roblox Image Asset ID",
    Text = "",
    ClearTextOnFocus = false
}, SettingsPage)

Corner(ImageInput,8)

local SetImage = Button(
    SettingsPage,
    "APPLY IMAGE",
    UDim2.new(1,-20,0,35),
    UDim2.new()
)

SetImage.Activated:Connect(function()
    local id = ImageInput.Text:match("%d+")

    if id then
        Config.Background = "rbxassetid://"..id
        Background.Image = Config.Background
        Background.Visible = true
        SaveConfig()
    end
end)

--==================================================
-- HOME FLOATING BUTTON
--==================================================

local HomeButton = New("TextButton", {
    Name = "FloatingHome",
    Size = UDim2.fromOffset(54,54),
    Position = UDim2.new(0,18,0.5,-27),
    BackgroundColor3 = Accent,
    Text = "KB",
    TextColor3 = Color3.fromRGB(36,24,52),
    Font = Enum.Font.GothamBold,
    TextSize = 18,
    ZIndex = 20
}, Gui)

Corner(HomeButton,100)
Stroke(HomeButton,Color3.fromRGB(235,215,255),2)

-- DRAG AND TAP
local dragging = false
local moved = false
local startPosition
local startInput

HomeButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true
        moved = false
        startInput = input.Position
        startPosition = HomeButton.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if not dragging then return end

    if input.UserInputType == Enum.UserInputType.MouseMovement
    or input.UserInputType == Enum.UserInputType.Touch then

        local delta = input.Position - startInput

        if delta.Magnitude > 8 then
            moved = true
        end

        if moved then
            HomeButton.Position = UDim2.new(
                startPosition.X.Scale,
                startPosition.X.Offset + delta.X,
                startPosition.Y.Scale,
                startPosition.Y.Offset + delta.Y
            )
        end
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then

        if dragging and not moved then
            Main.Visible = not Main.Visible
        end

        dragging = false
    end
end)

--==================================================
-- PARTICLE EFFECTS
--==================================================

local ParticleLayer = New("Frame", {
    Name = "Particles",
    Size = UDim2.fromScale(1,1),
    BackgroundTransparency = 1,
    ClipsDescendants = true,
    ZIndex = 10
}, Main)

local ParticleObjects = {}

local Symbols = {
    Snow = {"❄","❅","✦"},
    Sakura = {"🌸","✿","❀"},
    Stars = {"✦","✧","⋆"},
    Hearts = {"♥","♡","💜"}
}

local function ClearParticles()
    for _, p in ipairs(ParticleObjects) do
        if p.Parent then p:Destroy() end
    end
    table.clear(ParticleObjects)
end

local function SpawnParticles()
    ClearParticles()

    if not Config.Particles or Config.Effect == "None" then
        return
    end

    local list = Symbols[Config.Effect]
    if not list then return end

    local amount = Config.FastMode and 5
        or Config.ParticleAmount

    for i = 1, math.clamp(amount,0,60) do
        local particle = New("TextLabel", {
            BackgroundTransparency = 1,
            Text = list[math.random(1,#list)],
            TextSize = math.random(12,23),
            TextColor3 = Config.Effect == "Snow"
                and Color3.fromRGB(220,240,255)
                or Accent,
            Position = UDim2.new(
                math.random(),0,
                math.random(),0
            ),
            Size = UDim2.fromOffset(25,25),
            ZIndex = 10
        }, ParticleLayer)

        table.insert(ParticleObjects,particle)
    end
end

SpawnParticles()

local EffectClock = 0

RunService.RenderStepped:Connect(function(dt)
    if not Main.Visible or not Config.Particles then
        return
    end

    EffectClock += dt

    if EffectClock >= 0.035 then
        EffectClock = 0

        for _, p in ipairs(ParticleObjects) do
            if p.Parent then
                local y = p.Position.Y.Scale + 0.0025
                local x = p.Position.X.Scale

                if y > 1 then
                    y = -0.04
                    x = math.random()
                end

                p.Position = UDim2.new(x,0,y,0)
            end
        end
    end
end)

--==================================================
-- REAL LOCAL FPS MONITOR
--==================================================

RunService.RenderStepped:Connect(function(dt)
    FPSCounter += 1
    FPSTime += dt

    if FPSTime >= 1 then
        FPS = math.floor(FPSCounter / FPSTime + 0.5)

        MinFPS = math.min(MinFPS,FPS)
        MaxFPS = math.max(MaxFPS,FPS)

        FPSValue.Text = tostring(FPS)
        MinValue.Text = tostring(MinFPS)
        MaxValue.Text = tostring(MaxFPS)

        FPSCounter = 0
        FPSTime = 0
    end
end)

--==================================================
-- LOCAL PING MONITOR
--==================================================

task.spawn(function()
    while Gui.Parent do
        local success, result = pcall(function()
            return Stats.Network.ServerStatsItem[
                "Data Ping"
            ]:GetValue()
        end)

        if success and result then
            PingValue.Text = tostring(math.floor(result)).." ms"
        else
            PingValue.Text = "--"
        end

        PlayerCount.Text =
            "PLAYERS: "..#Players:GetPlayers()

        task.wait(1)
    end
end)

--==================================================
-- STATUS
--==================================================

local Status = New("TextLabel", {
    Size = UDim2.fromOffset(100,20),
    Position = UDim2.new(1,-110,1,-25),
    BackgroundTransparency = 1,
    Text = "● READY",
    TextColor3 = Color3.fromRGB(74,222,128),
    TextSize = 10,
    Font = Enum.Font.GothamBold,
    TextXAlignment = Enum.TextXAlignment.Right,
    ZIndex = 15
}, Main)

--==================================================
-- STARTUP
--==================================================

Main.Visible = false

print("================================")
print("KianBest V5.2 ULTRA")
print("Dashboard initialized")
print("Theme:",Config.Theme)
print("Language:",Config.Language)
print("================================")
