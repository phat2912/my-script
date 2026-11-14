
--========================================================
-- KIANBEST V5.3 ULTRA
-- GALAXY ANIME EDITION
-- Medium Responsive Dashboard
-- Roblox LocalScript | Touch + PC
--========================================================

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")

local LP = Players.LocalPlayer
local PG = LP:WaitForChild("PlayerGui")

--========================================================
-- CONFIGURATION
--========================================================

local Config = {
    Theme = "Galaxy",
    Hex = "#A78BFA",
    Language = "VI",
    Effect = "Stars",
    Background = "",
    BackgroundEnabled = true,
    Particles = true,
    ParticleAmount = 20,
    FastMode = false,
    Compact = false,
    Blur = 20,
}

local Themes = {
    Galaxy  = Color3.fromRGB(167,139,250),
    Purple  = Color3.fromRGB(167,139,250),
    Sakura  = Color3.fromRGB(249,168,212),
    Ocean   = Color3.fromRGB(56,189,248),
    Emerald = Color3.fromRGB(52,211,153),
    Ruby    = Color3.fromRGB(251,113,133),
    Gold    = Color3.fromRGB(251,191,36),
    Cyber   = Color3.fromRGB(34,211,238),
    White   = Color3.fromRGB(226,232,240),
}

local Accent = Themes.Galaxy
local BackgroundColor = Color3.fromRGB(20,15,37)

local function SaveConfig()
    -- Session configuration.
    -- Persistent saving requires a server DataStore.
    Config.Hex = string.format(
        "#%02X%02X%02X",
        math.floor(Accent.R * 255),
        math.floor(Accent.G * 255),
        math.floor(Accent.B * 255)
    )
end

--========================================================
-- LANGUAGE
--========================================================

local Languages = {
    VI = {
        HOME = "Trang chủ",
        PERFORMANCE = "Hiệu năng",
        PLAYERS = "Người chơi",
        WORLD = "Thế giới",
        SERVER = "Máy chủ",
        SETTINGS = "Cài đặt",
        FPS = "FPS thiết bị",
        PING = "Ping kết nối",
        MINFPS = "FPS thấp nhất",
        MAXFPS = "FPS cao nhất",
        ONLINE = "ĐANG HOẠT ĐỘNG",
        OFFLINE = "KHÔNG HOẠT ĐỘNG",
        SEARCH = "Tìm người chơi...",
        THEME = "Bảng màu",
        BACKGROUND = "Hình nền anime",
        EFFECT = "Hiệu ứng",
        SAVE = "Lưu cấu hình phiên",
        FAST = "Chế độ tăng tốc",
        LANGUAGE = "Ngôn ngữ",
        RESET = "Đặt lại",
        UPLOAD = "Áp dụng ảnh",
        CUSTOM = "Màu HEX tùy chỉnh",
        SNOW = "Tuyết rơi",
        SAKURA = "Hoa anh đào",
        STARS = "Sao vũ trụ",
        HEARTS = "Trái tim",
        NONE = "Tắt hiệu ứng",
    },
    EN = {
        HOME = "Home",
        PERFORMANCE = "Performance",
        PLAYERS = "Players",
        WORLD = "World",
        SERVER = "Server",
        SETTINGS = "Settings",
        FPS = "Device FPS",
        PING = "Connection Ping",
        MINFPS = "Minimum FPS",
        MAXFPS = "Maximum FPS",
        ONLINE = "ACTIVE",
        OFFLINE = "INACTIVE",
        SEARCH = "Search players...",
        THEME = "Color themes",
        BACKGROUND = "Anime background",
        EFFECT = "Effects",
        SAVE = "Save session config",
        FAST = "Boost mode",
        LANGUAGE = "Language",
        RESET = "Reset",
        UPLOAD = "Apply image",
        CUSTOM = "Custom HEX color",
        SNOW = "Snow fall",
        SAKURA = "Sakura fall",
        STARS = "Galaxy stars",
        HEARTS = "Floating hearts",
        NONE = "Disable effects",
    }
}

local function T(key)
    return Languages[Config.Language][key] or key
end

--========================================================
-- CLEANUP
--========================================================

local old = PG:FindFirstChild("KianBestV53")
if old then
    old:Destroy()
end

--========================================================
-- GUI ROOT
--========================================================

local Gui = Instance.new("ScreenGui")
Gui.Name = "KianBestV53"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = false
Gui.DisplayOrder = 100
Gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
Gui.Parent = PG

local UIScale = Instance.new("UIScale")
UIScale.Parent = Gui

local function UpdateScale()
    local camera = workspace.CurrentCamera
    if not camera then return end

    local viewport = camera.ViewportSize

    UIScale.Scale = math.clamp(
        math.min(viewport.X / 430, viewport.Y / 650),
        0.72,
        1
    )
end

UpdateScale()

if workspace.CurrentCamera then
    workspace.CurrentCamera:GetPropertyChangedSignal(
        "ViewportSize"
    ):Connect(UpdateScale)
end

--========================================================
-- UI HELPERS
--========================================================

local function New(class, props, parent)
    local obj = Instance.new(class)

    for k, v in pairs(props or {}) do
        obj[k] = v
    end

    obj.Parent = parent
    return obj
end

local function Corner(obj, radius)
    New("UICorner", {
        CornerRadius = UDim.new(0, radius or 12)
    }, obj)
end

local function AddStroke(obj, color, thickness)
    return New("UIStroke", {
        Color = color or Accent,
        Thickness = thickness or 1,
        Transparency = 0.2
    }, obj)
end

local function Label(parent, text, size, color)
    return New("TextLabel", {
        BackgroundTransparency = 1,
        Text = text,
        TextSize = size or 13,
        TextColor3 = color or Color3.new(1,1,1),
        Font = Enum.Font.GothamMedium,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Center,
        Size = UDim2.new(1,-16,0,25)
    }, parent)
end

local function Button(parent, text, height)
    local b = New("TextButton", {
        Size = UDim2.new(1,-20,0,height or 36),
        BackgroundColor3 = Color3.fromRGB(48,37,69),
        Text = text,
        TextColor3 = Color3.new(1,1,1),
        TextSize = 12,
        Font = Enum.Font.GothamSemibold,
        AutoButtonColor = true,
        BorderSizePixel = 0
    }, parent)

    Corner(b,10)
    return b
end

local function Section(parent,text)
    local label = Label(
        parent,
        text,
        13,
        Color3.fromRGB(209,185,255)
    )
    label.Font = Enum.Font.GothamBold
    return label
end

local function NewPage(parent)
    local page = New("ScrollingFrame", {
        Size = UDim2.fromScale(1,1),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = Accent,
        CanvasSize = UDim2.new(),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Visible = false
    }, parent)

    New("UIPadding", {
        PaddingTop = UDim.new(0,10),
        PaddingBottom = UDim.new(0,15),
        PaddingLeft = UDim.new(0,10),
        PaddingRight = UDim.new(0,10)
    }, page)

    New("UIListLayout", {
        Padding = UDim.new(0,9),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, page)

    return page
end

--========================================================
-- MAIN WINDOW
--========================================================

local Main = New("Frame", {
    Name = "MainDashboard",
    Size = UDim2.fromOffset(370,500),
    Position = UDim2.new(0.5,-185,0.5,-250),
    BackgroundColor3 = BackgroundColor,
    BorderSizePixel = 0,
    ClipsDescendants = true,
    Visible = false
}, Gui)

Corner(Main,20)

local MainOutline = AddStroke(Main,Accent,1.5)

--========================================================
-- ANIME BACKGROUND
--========================================================

local Background = New("ImageLabel", {
    Name = "AnimeBackground",
    Size = UDim2.fromScale(1,1),
    BackgroundTransparency = 1,
    Image = Config.Background,
    ImageTransparency = 0.35,
    ScaleType = Enum.ScaleType.Crop,
    ZIndex = 0
}, Main)

local DarkOverlay = New("Frame", {
    Size = UDim2.fromScale(1,1),
    BackgroundColor3 = Color3.fromRGB(18,13,34),
    BackgroundTransparency = 0.17,
    BorderSizePixel = 0,
    ZIndex = 1
}, Main)

--========================================================
-- HEADER
--========================================================

local Header = New("Frame", {
    Name = "DragHeader",
    Size = UDim2.new(1,0,0,65),
    BackgroundColor3 = Color3.fromRGB(35,25,58),
    BackgroundTransparency = 0.05,
    BorderSizePixel = 0,
    ZIndex = 3,
    Active = true
}, Main)

local HeaderTitle = New("TextLabel", {
    Size = UDim2.new(1,-85,0,28),
    Position = UDim2.fromOffset(14,7),
    BackgroundTransparency = 1,
    Text = "✦ KianBest V5.3",
    TextColor3 = Color3.fromRGB(245,235,255),
    TextSize = 17,
    Font = Enum.Font.GothamBold,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex = 4
}, Header)

local HeaderSubtitle = New("TextLabel", {
    Size = UDim2.new(1,-85,0,18),
    Position = UDim2.fromOffset(15,35),
    BackgroundTransparency = 1,
    Text = "GALAXY ANIME EDITION",
    TextColor3 = Color3.fromRGB(195,173,230),
    TextSize = 9,
    Font = Enum.Font.GothamBold,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex = 4
}, Header)

local Close = New("TextButton", {
    Size = UDim2.fromOffset(34,34),
    Position = UDim2.new(1,-45,0,14),
    BackgroundColor3 = Color3.fromRGB(62,45,83),
    Text = "×",
    TextSize = 22,
    TextColor3 = Color3.new(1,1,1),
    ZIndex = 5
}, Header)

Corner(Close,12)

--========================================================
-- DRAG SYSTEM
-- Works with mouse and touch.
-- Only the header is draggable.
--========================================================

local function MakeDraggable(handle, target)
    local dragging = false
    local dragStart
    local startPosition
    local dragInput

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

            dragging = true
            dragStart = input.Position
            startPosition = target.Position

            if input.UserInputType == Enum.UserInputType.Touch then
                dragInput = input
            end
        end
    end)

    handle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then
            if input.UserInputType == Enum.UserInputType.Touch then
                dragInput = input
            end
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if not dragging then return end

        if input.UserInputType == Enum.UserInputType.MouseMovement
        or input == dragInput then

            local delta = input.Position - dragStart

            local camera = workspace.CurrentCamera
            if not camera then return end

            local view = camera.ViewportSize
            local scale = UIScale.Scale

            local width = target.AbsoluteSize.X / scale
            local height = target.AbsoluteSize.Y / scale

            local x = math.clamp(
                startPosition.X.Offset + delta.X / scale,
                0,
                math.max(0,view.X / scale - width)
            )

            local y = math.clamp(
                startPosition.Y.Offset + delta.Y / scale,
                0,
                math.max(0,view.Y / scale - height)
            )

            target.Position = UDim2.fromOffset(x,y)
        end
    end)

    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
            dragInput = nil
        end
    end)
end

MakeDraggable(Header,Main)

Close.Activated:Connect(function()
    Main.Visible = false
end)

--========================================================
-- NAVIGATION
--========================================================

local Nav = New("ScrollingFrame", {
    Position = UDim2.fromOffset(0,65),
    Size = UDim2.new(1,0,0,44),
    BackgroundColor3 = Color3.fromRGB(29,22,47),
    BorderSizePixel = 0,
    ScrollBarThickness = 2,
    ScrollingDirection = Enum.ScrollingDirection.X,
    CanvasSize = UDim2.fromOffset(0,0),
    AutomaticCanvasSize = Enum.AutomaticSize.X,
    ZIndex = 3
}, Main)

New("UIListLayout", {
    FillDirection = Enum.FillDirection.Horizontal,
    Padding = UDim.new(0,5),
    SortOrder = Enum.SortOrder.LayoutOrder
}, Nav)

New("UIPadding", {
    PaddingLeft = UDim.new(0,7),
    PaddingTop = UDim.new(0,5),
    PaddingRight = UDim.new(0,7)
}, Nav)

local Content = New("Frame", {
    Position = UDim2.fromOffset(0,109),
    Size = UDim2.new(1,0,1,-109),
    BackgroundTransparency = 1,
    ZIndex = 2
}, Main)

local Tabs = {
    "HOME",
    "PERFORMANCE",
    "PLAYERS",
    "WORLD",
    "SERVER",
    "SETTINGS"
}

local Pages = {}
local TabButtons = {}
local CurrentTab = "HOME"

for i,name in ipairs(Tabs) do
    local page = NewPage(Content)
    page.Name = name
    Pages[name] = page

    local b = New("TextButton", {
        Size = UDim2.fromOffset(100,33),
        BackgroundColor3 = name == "HOME"
            and Accent
            or Color3.fromRGB(48,37,69),
        Text = name,
        TextColor3 = Color3.new(1,1,1),
        TextSize = 10,
        Font = Enum.Font.GothamBold,
        LayoutOrder = i,
        ZIndex = 4
    },Nav)

    Corner(b,9)
    TabButtons[name] = b

    b.Activated:Connect(function()
        CurrentTab = name

        for n,p in pairs(Pages) do
            p.Visible = n == name
        end

        for n,button in pairs(TabButtons) do
            button.BackgroundColor3 =
                n == name and Accent
                or Color3.fromRGB(48,37,69)
        end
    end)
end

Pages.HOME.Visible = true

--========================================================
-- UI COMPONENTS
--========================================================

local function Card(parent,title,value)
    local frame = New("Frame", {
        Size = UDim2.new(0.5,-5,0,75),
        BackgroundColor3 = Color3.fromRGB(43,33,65),
        BorderSizePixel = 0
    },parent)

    Corner(frame,12)
    AddStroke(frame,Accent,0.7)

    Label(frame,title,10,Color3.fromRGB(195,175,226)).Position =
        UDim2.fromOffset(10,6)

    local number = Label(frame,value,22,Color3.new(1,1,1))
    number.Position = UDim2.fromOffset(10,30)
    number.Name = "Value"

    return number
end

local function Toggle(parent,text,initial,callback)
    local row = New("Frame", {
        Size = UDim2.new(1,0,0,43),
        BackgroundColor3 = Color3.fromRGB(42,32,61),
        BorderSizePixel = 0
    },parent)

    Corner(row,10)

    local title = Label(row,text,11)
    title.Position = UDim2.fromOffset(10,5)
    title.Size = UDim2.new(1,-80,1,-10)

    local enabled = initial == true

    local b = New("TextButton", {
        Size = UDim2.fromOffset(48,25),
        Position = UDim2.new(1,-58,0,9),
        BackgroundColor3 = enabled and Accent
            or Color3.fromRGB(82,72,95),
        Text = enabled and "ON" or "OFF",
        TextColor3 = Color3.new(1,1,1),
        TextSize = 10,
        Font = Enum.Font.GothamBold
    },row)

    Corner(b,20)

    b.Activated:Connect(function()
        enabled = not enabled

        b.Text = enabled and "ON" or "OFF"
        b.BackgroundColor3 = enabled and Accent
            or Color3.fromRGB(82,72,95)

        callback(enabled)
    end)

    return row
end

local function SmallButton(parent,text,callback)
    local b = Button(parent,text,34)

    b.Activated:Connect(callback)

    return b
end

--========================================================
-- HOME PAGE
--========================================================

local HomePage = Pages.HOME

Section(HomePage,"✦ SYSTEM OVERVIEW")

local Metrics = New("Frame", {
    Size = UDim2.new(1,0,0,160),
    BackgroundTransparency = 1
},HomePage)

local Grid = New("UIGridLayout", {
    CellSize = UDim2.new(0.5,-5,0,73),
    CellPadding = UDim2.fromOffset(8,8),
    SortOrder = Enum.SortOrder.LayoutOrder
},Metrics)

local FPSValue = Card(Metrics,"DEVICE FPS","0")
local PingValue = Card(Metrics,"CONNECTION PING","--")
local MinValue = Card(Metrics,"MINIMUM FPS","--")
local MaxValue = Card(Metrics,"MAXIMUM FPS","--")

Section(HomePage,"✦ DEVICE INFORMATION")

local DeviceCard = New("Frame", {
    Size = UDim2.new(1,0,0,90),
    BackgroundColor3 = Color3.fromRGB(42,32,62)
},HomePage)

Corner(DeviceCard,12)

local DeviceType = UIS.TouchEnabled and "MOBILE / TOUCH"
    or "DESKTOP / PC"

Label(DeviceCard,"DEVICE: "..DeviceType,11).Position =
    UDim2.fromOffset(10,5)

local PlayerCount = Label(DeviceCard,"PLAYERS: 0",11)
PlayerCount.Position = UDim2.fromOffset(10,29)

local PlaceInfo = Label(
    DeviceCard,
    "PLACE ID: "..tostring(game.PlaceId),
    10
)

PlaceInfo.Position = UDim2.fromOffset(10,53)

--========================================================
-- PERFORMANCE PAGE
--========================================================

local PerformancePage = Pages.PERFORMANCE

Section(PerformancePage,"✦ PERFORMANCE CONTROL")

Toggle(
    PerformancePage,
    "Fast Mode",
    Config.FastMode,
    function(v)
        Config.FastMode = v
        Config.ParticleAmount = v and 3 or 20
        Config.Particles = not v

        Background.ImageTransparency = v and 0.65 or 0.35
    end
)

Toggle(
    PerformancePage,
    "Galaxy Particles",
    Config.Particles,
    function(v)
        Config.Particles = v
    end
)

Toggle(
    PerformancePage,
    "Anime Background",
    Config.BackgroundEnabled,
    function(v)
        Config.BackgroundEnabled = v
        Background.Visible = v
    end
)

Toggle(
    PerformancePage,
    "Compact Layout",
    Config.Compact,
    function(v)
        Config.Compact = v
        Main.Size = v
            and UDim2.fromOffset(340,450)
            or UDim2.fromOffset(370,500)
    end
)

Section(PerformancePage,"✦ EFFECT SETTINGS")

local AmountLabel = Label(
    PerformancePage,
    "Particle amount: "..Config.ParticleAmount,
    11
)

local AmountSlider = New("Frame", {
    Size = UDim2.new(1,0,0,28),
    BackgroundColor3 = Color3.fromRGB(45,35,63)
},PerformancePage)

Corner(AmountSlider,12)

local AmountFill = New("Frame", {
    Size = UDim2.new(Config.ParticleAmount/60,0,1,0),
    BackgroundColor3 = Accent
},AmountSlider)

Corner(AmountFill,12)

local AmountButton = New("TextButton", {
    Size = UDim2.fromScale(1,1),
    BackgroundTransparency = 1,
    Text = "",
    Active = true
},AmountSlider)

local function SetAmount(x)
    local width = AmountSlider.AbsoluteSize.X
    if width <= 0 then return end

    local value = math.clamp(
        math.floor((x-AmountSlider.AbsolutePosition.X)/width*60),
        0,60
    )

    Config.ParticleAmount = value
    AmountFill.Size = UDim2.new(value/60,0,1,0)
    AmountLabel.Text = "Particle amount: "..value
end

local sliding = false

AmountButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        sliding = true
        SetAmount(input.Position.X)
    end
end)

UIS.InputChanged:Connect(function(input)
    if sliding and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        SetAmount(input.Position.X)
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        sliding = false
    end
end)

--========================================================
-- PLAYER MONITOR
--========================================================

local PlayerPage = Pages.PLAYERS

Section(PlayerPage,"✦ PLAYER MONITOR")

local SearchBox = New("TextBox", {
    Size = UDim2.new(1,0,0,37),
    BackgroundColor3 = Color3.fromRGB(43,33,63),
    TextColor3 = Color3.new(1,1,1),
    PlaceholderText = T("SEARCH"),
    Text = "",
    ClearTextOnFocus = false,
    TextSize = 12,
    Font = Enum.Font.Gotham
},PlayerPage)

Corner(SearchBox,10)

local PlayerContainer = New("Frame", {
    Size = UDim2.new(1,0,0,0),
    AutomaticSize = Enum.AutomaticSize.Y,
    BackgroundTransparency = 1
},PlayerPage)

New("UIListLayout", {
    Padding = UDim.new(0,7),
    SortOrder = Enum.SortOrder.LayoutOrder
},PlayerContainer)

local function RefreshPlayerList()
    for _,child in ipairs(PlayerContainer:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end

    local query = SearchBox.Text:lower()

    for _,plr in ipairs(Players:GetPlayers()) do
        if plr.Name:lower():find(query,1,true)
        or plr.DisplayName:lower():find(query,1,true) then

            local card = New("Frame", {
                Size = UDim2.new(1,0,0,76),
                BackgroundColor3 = Color3.fromRGB(43,33,63)
            },PlayerContainer)

            Corner(card,11)

            Label(
                card,
                plr.DisplayName,
                13
            ).Position = UDim2.fromOffset(10,4)

            Label(
                card,
                "@"..plr.Name,
                10,
                Color3.fromRGB(192,170,225)
            ).Position = UDim2.fromOffset(10,24)

            Label(
                card,
                "USER ID: "..plr.UserId,
                9,
                Color3.fromRGB(185,190,220)
            ).Position = UDim2.fromOffset(10,44)

            Label(
                card,
                plr == LP and "LOCAL PLAYER"
                or "CONNECTED",
                9,
                Color3.fromRGB(110,225,165)
            ).Position = UDim2.fromOffset(10,59)

            local copy = New("TextButton", {
                Size = UDim2.fromOffset(65,26),
                Position = UDim2.new(1,-75,0,25),
                BackgroundColor3 = Accent,
                Text = "COPY ID",
                TextColor3 = Color3.new(1,1,1),
                TextSize = 9,
                Font = Enum.Font.GothamBold
            },card)

            Corner(copy,8)

            copy.Activated:Connect(function()
                -- Copy to a visible text field for portability.
                SearchBox.Text = tostring(plr.UserId)
                SearchBox:CaptureFocus()
            end)
        end
    end
end

SearchBox:GetPropertyChangedSignal("Text"):Connect(
    RefreshPlayerList
)

Players.PlayerAdded:Connect(RefreshPlayerList)
Players.PlayerRemoving:Connect(RefreshPlayerList)

RefreshPlayerList()

--========================================================
-- WORLD PAGE
--========================================================

local WorldPage = Pages.WORLD

Section(WorldPage,"✦ WORLD INFORMATION")

local WorldCard = New("Frame", {
    Size = UDim2.new(1,0,0,105),
    BackgroundColor3 = Color3.fromRGB(43,33,63)
},WorldPage)

Corner(WorldCard,12)

Label(
    WorldCard,
    "PLACE NAME: "..game.Name,
    11
).Position = UDim2.fromOffset(10,7)

Label(
    WorldCard,
    "PLACE ID: "..game.PlaceId,
    10
).Position = UDim2.fromOffset(10,34)

Label(
    WorldCard,
    "MONSTER DATABASE: NOT CONNECTED",
    9,
    Color3.fromRGB(255,195,120)
).Position = UDim2.fromOffset(10,60)

local WorldNote = Label(
    WorldPage,
    "NPC and monster data can be connected to an authorized game database.",
    10,
    Color3.fromRGB(195,180,220)
)

WorldNote.TextWrapped = true
WorldNote.Size = UDim2.new(1,0,0,50)

--========================================================
-- SERVER PAGE
--========================================================

local ServerPage = Pages.SERVER

Section(ServerPage,"✦ SERVER INFORMATION")

local ServerFields = {}

local function ServerInfo(name,value)
    local card = New("Frame", {
        Size = UDim2.new(1,0,0,48),
        BackgroundColor3 = Color3.fromRGB(43,33,63)
    },ServerPage)

    Corner(card,10)

    local label = Label(card,name,10)
    label.Position = UDim2.fromOffset(10,2)

    local val = Label(card,value,11,Color3.fromRGB(211,195,245))
    val.Position = UDim2.fromOffset(10,22)

    ServerFields[name] = val
end

ServerInfo("PLACE ID",tostring(game.PlaceId))
ServerInfo("JOB ID",tostring(game.JobId))
ServerInfo("PLAYER COUNT",tostring(#Players:GetPlayers()))

local RefreshServer = Button(
    ServerPage,
    "REFRESH SERVER INFO",
    35
)

RefreshServer.Activated:Connect(function()
    ServerFields["PLAYER COUNT"].Text =
        tostring(#Players:GetPlayers())
end)

--========================================================
-- SETTINGS PAGE
--========================================================

local SettingsPage = Pages.SETTINGS

Section(SettingsPage,"✦ GALAXY COLOR STUDIO")

local function ApplyTheme(name,color)
    Accent = color
    Config.Theme = name

    MainOutline.Color = color
    Nav.ScrollBarImageColor3 = color

    for tabName,b in pairs(TabButtons) do
        b.BackgroundColor3 =
            tabName == CurrentTab and color
            or Color3.fromRGB(48,37,69)
    end

    AmountFill.BackgroundColor3 = color

    for _,child in ipairs(SettingsPage:GetChildren()) do
        if child:IsA("TextButton") then
            child.BackgroundColor3 = color
        end
    end

    SaveConfig()
end

for name,color in pairs(Themes) do
    local b = Button(SettingsPage,name,34)

    b.Activated:Connect(function()
        ApplyTheme(name,color)
    end)
end

Section(SettingsPage,T("CUSTOM"))

local HexBox = New("TextBox", {
    Size = UDim2.new(1,0,0,36),
    BackgroundColor3 = Color3.fromRGB(43,33,63),
    TextColor3 = Color3.new(1,1,1),
    PlaceholderText = "#A78BFA",
    Text = "",
    TextSize = 12,
    ClearTextOnFocus = false
},SettingsPage)

Corner(HexBox,10)

local HexApply = Button(SettingsPage,"APPLY HEX COLOR",35)

HexApply.Activated:Connect(function()
    local hex = HexBox.Text:gsub("#","")

    if #hex == 6 and hex:match("^%x+$") then
        local r = tonumber(hex:sub(1,2),16)
        local g = tonumber(hex:sub(3,4),16)
        local b = tonumber(hex:sub(5,6),16)

        ApplyTheme("Custom",Color3.fromRGB(r,g,b))
        Config.Hex = "#" .. hex:upper()
    end
end)

Section(SettingsPage,T("BACKGROUND"))

local ImageBox = New("TextBox", {
    Size = UDim2.new(1,0,0,36),
    BackgroundColor3 = Color3.fromRGB(43,33,63),
    TextColor3 = Color3.new(1,1,1),
    PlaceholderText = "Roblox Image Asset ID",
    Text = "",
    ClearTextOnFocus = false,
    TextSize = 11
},SettingsPage)

Corner(ImageBox,10)

local ImageApply = Button(SettingsPage,T("UPLOAD"),35)

ImageApply.Activated:Connect(function()
    local id = ImageBox.Text:match("%d+")

    if id then
        Config.Background = "rbxassetid://"..id
        Background.Image = Config.Background
        Background.Visible = true
        Config.BackgroundEnabled = true
    end
end)

SmallButton(SettingsPage,"RESET BACKGROUND",function()
    Config.Background = ""
    Background.Image = ""
    Background.BackgroundColor3 = BackgroundColor
end)

Section(SettingsPage,T("EFFECT"))

local Effects = {
    "Stars",
    "Snow",
    "Sakura",
    "Hearts",
    "None"
}

for _,effect in ipairs(Effects) do
    local b = Button(SettingsPage,effect,32)

    b.Activated:Connect(function()
        Config.Effect = effect
        Config.Particles = effect ~= "None"
    end)
end

Section(SettingsPage,T("LANGUAGE"))

local LanguageButton = Button(
    SettingsPage,
    "VI / EN",
    35
)

LanguageButton.Activated:Connect(function()
    Config.Language =
        Config.Language == "VI" and "EN" or "VI"

    SearchBox.PlaceholderText = T("SEARCH")
    LanguageButton.Text = T("LANGUAGE")..": "..Config.Language
end)

SmallButton(SettingsPage,T("RESET"),function()
    Config.Theme = "Galaxy"
    Config.Hex = "#A78BFA"
    Config.Effect = "Stars"

    ApplyTheme("Galaxy",Themes.Galaxy)
end)

--========================================================
-- FLOATING HOME BUTTON
--========================================================

local HomeButton = New("TextButton", {
    Name = "FloatingHome",
    Size = UDim2.fromOffset(56,56),
    Position = UDim2.new(0,18,0.5,-28),
    BackgroundColor3 = Accent,
    Text = "KB",
    TextColor3 = Color3.fromRGB(36,23,52),
    TextSize = 19,
    Font = Enum.Font.GothamBold,
    ZIndex = 20,
    Active = true
},Gui)

Corner(HomeButton,100)
AddStroke(HomeButton,Color3.fromRGB(235,220,255),2)

-- Independent drag handler for HOME.
local function MakeHomeDraggable(handle)
    local dragging = false
    local moved = false
    local startInput
    local startPos
    local activeTouch

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

            dragging = true
            moved = false
            startInput = input.Position
            startPos = handle.Position

            if input.UserInputType == Enum.UserInputType.Touch then
                activeTouch = input
            end
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if not dragging then return end

        if input.UserInputType == Enum.UserInputType.MouseMovement
        or input == activeTouch then

            local delta = input.Position - startInput

            if delta.Magnitude > 6 then
                moved = true
            end

            if moved then
                local camera = workspace.CurrentCamera
                if not camera then return end

                local view = camera.ViewportSize
                local scale = UIScale.Scale

                local x = math.clamp(
                    startPos.X.Offset + delta.X / scale,
                    0,
                    math.max(0,view.X/scale-56)
                )

                local y = math.clamp(
                    startPos.Y.Offset + delta.Y / scale,
                    0,
                    math.max(0,view.Y/scale-56)
                )

                handle.Position = UDim2.fromOffset(x,y)
            end
        end
    end)

    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input == activeTouch then

            if dragging and not moved then
                Main.Visible = not Main.Visible
            end

            dragging = false
            activeTouch = nil
        end
    end)
end

MakeHomeDraggable(HomeButton)

--========================================================
-- GALAXY PARTICLE ENGINE
--========================================================

local ParticleLayer = New("Frame", {
    Size = UDim2.fromScale(1,1),
    BackgroundTransparency = 1,
    ClipsDescendants = true,
    ZIndex = 2
},Main)

local ParticleObjects = {}

local Symbols = {
    Stars = {"✦","✧","⋆","✴"},
    Snow = {"❄","❅","❆"},
    Sakura = {"🌸","✿","❀"},
    Hearts = {"♡","♥","💜"}
}

local function RebuildParticles()
    for _,p in ipairs(ParticleObjects) do
        p:Destroy()
    end

    table.clear(ParticleObjects)

    if not Config.Particles or Config.Effect == "None" then
        return
    end

    local symbols = Symbols[Config.Effect]
    if not symbols then return end

    local count = Config.FastMode and 3
        or math.clamp(Config.ParticleAmount,0,60)

    for i = 1,count do
        local p = New("TextLabel", {
            BackgroundTransparency = 1,
            Text = symbols[math.random(1,#symbols)],
            TextColor3 = Config.Effect == "Snow"
                and Color3.fromRGB(220,240,255)
                or Accent,
            TextSize = math.random(12,20),
            Size = UDim2.fromOffset(25,25),
            Position = UDim2.new(
                math.random(),0,
                math.random(),0
            ),
            ZIndex = 2
        },ParticleLayer)

        table.insert(ParticleObjects,p)
    end
end

RebuildParticles()

local EffectClock = 0

RunService.RenderStepped:Connect(function(dt)
    if not Main.Visible or not Config.Particles then
        return
    end

    EffectClock += dt

    if EffectClock < 0.04 then return end
    EffectClock = 0

    for _,p in ipairs(ParticleObjects) do
        if p.Parent then
            local speed = Config.FastMode and 0.0008 or 0.0025

            local y = p.Position.Y.Scale + speed
            local x = p.Position.X.Scale

            if y > 1 then
                y = -0.05
                x = math.random()
            end

            p.Position = UDim2.new(x,0,y,0)
        end
    end
end)

--========================================================
-- LIVE LOCAL FPS
--========================================================

local FPSCounter = 0
local FPSTime = 0
local FPS = 0
local MinFPS = math.huge
local MaxFPS = 0

RunService.RenderStepped:Connect(function(dt)
    FPSCounter += 1
    FPSTime += dt

    if FPSTime >= 1 then
        FPS = math.floor(FPSCounter/FPSTime + 0.5)

        MinFPS = math.min(MinFPS,FPS)
        MaxFPS = math.max(MaxFPS,FPS)

        FPSValue.Text = tostring(FPS)
        MinValue.Text = tostring(MinFPS)
        MaxValue.Text = tostring(MaxFPS)

        FPSCounter = 0
        FPSTime = 0
    end
end)

--========================================================
-- LIVE PING
--========================================================

task.spawn(function()
    while Gui.Parent do
        local success,value = pcall(function()
            return Stats.Network.ServerStatsItem[
                "Data Ping"
            ]:GetValue()
        end)

        PingValue.Text = success and value
            and tostring(math.floor(value)).." ms"
            or "--"

        PlayerCount.Text =
            "PLAYERS: "..#Players:GetPlayers()

        task.wait(1)
    end
end)

--========================================================
-- STATUS INDICATOR
--========================================================

local Status = New("TextLabel", {
    Size = UDim2.fromOffset(125,20),
    Position = UDim2.new(1,-135,1,-23),
    BackgroundTransparency = 1,
    Text = "● SYSTEM READY",
    TextColor3 = Color3.fromRGB(74,222,128),
    TextSize = 9,
    Font = Enum.Font.GothamBold,
    TextXAlignment = Enum.TextXAlignment.Right,
    ZIndex = 20
},Main)

--========================================================
-- FINAL INITIALIZATION
--========================================================

ApplyTheme("Galaxy",Accent)

Main.Visible = false

print("====================================")
print("KIANBEST V5.3 ULTRA")
print("GALAXY ANIME EDITION")
print("UI: READY")
print("TOUCH: SUPPORTED")
print("PC: SUPPORTED")
print("THEME: GALAXY")
print("====================================")
