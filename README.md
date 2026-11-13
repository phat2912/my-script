
--==================================================
-- KIANBEST V7.2 - ACCESSIBILITY EDITION
-- Roblox Studio - LocalScript
-- StarterPlayer > StarterPlayerScripts
--==================================================

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local old = playerGui:FindFirstChild("KianBestV72")
if old then old:Destroy() end

local C = {
    BG = Color3.fromRGB(23,18,36),
    PANEL = Color3.fromRGB(37,29,55),
    PANEL2 = Color3.fromRGB(51,40,73),
    PURPLE = Color3.fromRGB(177,139,255),
    PINK = Color3.fromRGB(255,177,220),
    WHITE = Color3.fromRGB(248,243,255),
    MUTED = Color3.fromRGB(180,167,204),
    GREEN = Color3.fromRGB(45,190,100),
    ORANGE = Color3.fromRGB(255,116,55)
}

local function create(class, props, parent)
    local obj = Instance.new(class)
    for k,v in pairs(props or {}) do
        obj[k] = v
    end
    obj.Parent = parent
    return obj
end

local function round(obj, radius)
    create("UICorner", {
        CornerRadius = UDim.new(0,radius or 10)
    },obj)
end

local function label(parent,text,size,color)
    return create("TextLabel",{
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = color or C.WHITE,
        TextSize = size or 14,
        Font = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left
    },parent)
end

local gui = create("ScreenGui",{
    Name = "KianBestV72",
    ResetOnSpawn = false,
    IgnoreGuiInset = true,
    DisplayOrder = 50
},playerGui)

-- MAIN WINDOW
local main = create("Frame",{
    Name = "Main",
    AnchorPoint = Vector2.new(0.5,0.5),
    Position = UDim2.fromScale(0.5,0.5),
    Size = UDim2.fromOffset(850,550),
    BackgroundColor3 = C.BG,
    ClipsDescendants = true
},gui)

round(main,18)

create("UIStroke",{
    Color = C.PURPLE,
    Thickness = 1.5
},main)

-- HEADER
local header = create("Frame",{
    Size = UDim2.new(1,0,0,62),
    BackgroundColor3 = Color3.fromRGB(41,31,60)
},main)

round(header,18)

local title = label(header,"♡ KIANBEST V7.2",21,C.WHITE)
title.Position = UDim2.fromOffset(20,5)
title.Size = UDim2.fromOffset(300,50)
title.Font = Enum.Font.GothamBold

-- CLOSE BUTTON
local close = create("TextButton",{
    Size = UDim2.fromOffset(38,38),
    Position = UDim2.new(1,-49,0,12),
    BackgroundColor3 = Color3.fromRGB(105,57,88),
    Text = "×",
    TextColor3 = C.WHITE,
    TextSize = 24,
    Font = Enum.Font.GothamBold
},header)

round(close,10)

-- FLOATING ACCESSIBILITY BUTTON
local assist = create("TextButton",{
    Name = "KianBestAccessibility",
    Size = UDim2.fromOffset(58,58),
    Position = UDim2.new(1,-78,0.65,0),
    BackgroundColor3 = Color3.fromRGB(115,75,180),
    Text = "♡",
    TextColor3 = C.WHITE,
    TextSize = 29,
    Font = Enum.Font.GothamBold,
    Visible = false,
    Active = true,
    AutoButtonColor = true,
    ZIndex = 100
},gui)

round(assist,100)

create("UIStroke",{
    Color = Color3.fromRGB(220,180,255),
    Thickness = 2
},assist)

-- SIDEBAR
local sidebar = create("Frame",{
    Position = UDim2.fromOffset(13,75),
    Size = UDim2.new(0,175,1,-88),
    BackgroundColor3 = C.PANEL
},main)

round(sidebar,14)

local menuTitle = label(sidebar,"♡ MAIN MENU",13,C.PINK)
menuTitle.Position = UDim2.fromOffset(13,10)
menuTitle.Size = UDim2.new(1,-20,0,30)

local menu = create("ScrollingFrame",{
    Position = UDim2.fromOffset(8,45),
    Size = UDim2.new(1,-16,1,-53),
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    ScrollBarThickness = 3,
    AutomaticCanvasSize = Enum.AutomaticSize.Y,
    CanvasSize = UDim2.new()
},sidebar)

create("UIListLayout",{
    Padding = UDim.new(0,8)
},menu)

-- CONTENT
local content = create("Frame",{
    Position = UDim2.fromOffset(202,75),
    Size = UDim2.new(1,-215,1,-88),
    BackgroundTransparency = 1
},main)

local pageTitle = label(content,"Welcome back ♡",21,C.WHITE)
pageTitle.Position = UDim2.fromOffset(5,3)
pageTitle.Size = UDim2.new(1,-10,0,34)
pageTitle.Font = Enum.Font.GothamBold

local pageSubtitle = label(content,"KianBest V7.2",12,C.MUTED)
pageSubtitle.Position = UDim2.fromOffset(6,35)
pageSubtitle.Size = UDim2.new(1,-10,0,22)

-- NOTIFICATION
local notice = create("TextLabel",{
    AnchorPoint = Vector2.new(1,1),
    Position = UDim2.new(1,-15,1,-12),
    Size = UDim2.fromOffset(250,40),
    BackgroundColor3 = C.PANEL2,
    Text = "",
    TextColor3 = C.WHITE,
    TextSize = 12,
    Visible = false,
    ZIndex = 110
},main)

round(notice,10)

local function notify(message)
    notice.Text = "♡  "..message
    notice.Visible = true
    task.delay(2.5,function()
        if notice.Parent then
            notice.Visible = false
        end
    end)
end

-- PAGE SYSTEM
local pages = {}
local menuButtons = {}

local function clearPage()
    for _,v in ipairs(content:GetChildren()) do
        if v ~= pageTitle and v ~= pageSubtitle then
            v:Destroy()
        end
    end
end

local function newPage(name,description)
    clearPage()
    pageTitle.Text = name
    pageSubtitle.Text = description or ""

    local scroll = create("ScrollingFrame",{
        Position = UDim2.fromOffset(0,67),
        Size = UDim2.new(1,0,1,-72),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 4,
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        CanvasSize = UDim2.new()
    },content)

    create("UIListLayout",{
        Padding = UDim.new(0,10)
    },scroll)

    return scroll
end

local function section(parent,name)
    local box = create("Frame",{
        Size = UDim2.new(1,-5,0,0),
        AutomaticSize = Enum.AutomaticSize.Y,
        BackgroundColor3 = C.PANEL
    },parent)

    round(box,12)

    local heading = label(box,name,14,C.PINK)
    heading.Position = UDim2.fromOffset(12,7)
    heading.Size = UDim2.new(1,-20,0,28)
    heading.Font = Enum.Font.GothamBold

    local inner = create("Frame",{
        Position = UDim2.fromOffset(12,40),
        Size = UDim2.new(1,-24,0,0),
        AutomaticSize = Enum.AutomaticSize.Y,
        BackgroundTransparency = 1
    },box)

    create("UIListLayout",{
        Padding = UDim.new(0,8)
    },inner)

    create("UIPadding",{
        PaddingBottom = UDim.new(0,12)
    },inner)

    return inner
end

local function info(parent,text)
    local l = label(parent,text,12,C.MUTED)
    l.Size = UDim2.new(1,0,0,25)
    return l
end

local function action(parent,text,callback)
    local b = create("TextButton",{
        Size = UDim2.new(1,0,0,40),
        BackgroundColor3 = C.PANEL2,
        Text = text,
        TextColor3 = C.WHITE,
        TextSize = 13,
        Font = Enum.Font.GothamMedium
    },parent)

    round(b,9)
    b.MouseButton1Click:Connect(callback)
    return b
end

-- DASHBOARD
pages["Dashboard"] = function()
    local p = newPage("Welcome back, "..player.DisplayName.." ♡","Cute control center")

    local live = section(p,"✦ LIVE SYSTEM STATUS")
    local fps = info(live,"FPS: Calculating...")
    local ping = info(live,"Ping: Calculating...")
    local count = info(live,"Players: Calculating...")

    task.spawn(function()
        local frames,elapsed = 0,0
        while fps.Parent do
            local dt = RunService.RenderStepped:Wait()
            frames += 1
            elapsed += dt
            if elapsed >= 1 then
                fps.Text = "FPS: "..math.floor(frames/elapsed)
                local ok,value = pcall(function()
                    return math.floor(player:GetNetworkPing()*1000)
                end)
                ping.Text = ok and "Ping: "..value.." ms" or "Ping: N/A"
                count.Text = "Players: "..#Players:GetPlayers()
                frames,elapsed = 0,0
            end
        end
    end)

    local quick = section(p,"♡ QUICK ACCESS")
    action(quick,"☁ Server Manager",function()
        pages["Server Manager"]()
    end)
    action(quick,"✧ Visual Settings",function()
        pages["Visual Settings"]()
    end)
    action(quick,"◈ Performance",function()
        pages["Performance"]()
    end)
end

-- STATUS
pages["Trạng thái"] = function()
    local p = newPage("Trạng thái","Tình trạng các module")
    local box = section(p,"SYSTEM STATUS")

    local entries = {
        {"Dashboard","Hoạt động",true},
        {"FPS / Ping","Hoạt động",true},
        {"Player Tools","Cơ bản",true},
        {"Monster Database","Dữ liệu mẫu",true},
        {"Server Manager","Chờ tích hợp",false},
        {"Performance","Cơ bản",true},
        {"Visual Settings","Cơ bản",true}
    }

    for _,item in ipairs(entries) do
        local row = create("Frame",{
            Size = UDim2.new(1,0,0,55),
            BackgroundTransparency = 1
        },box)

        create("Frame",{
            Position = UDim2.new(0,0,1,-1),
            Size = UDim2.new(1,0,0,1),
            BackgroundColor3 = Color3.fromRGB(48,48,48)
        },row)

        local badge = create("TextLabel",{
            Position = UDim2.fromOffset(5,5),
            Size = UDim2.fromOffset(210,40),
            BackgroundColor3 = item[3]
                and Color3.fromRGB(0,38,20)
                or Color3.fromRGB(40,20,12),
            Text = item[2],
            TextColor3 = item[3] and C.GREEN or C.ORANGE,
            TextSize = 17,
            Font = Enum.Font.GothamBold
        },row)

        round(badge,8)
    end
end

-- PLAYER TOOLS
pages["Player Tools"] = function()
    local p = newPage("Player Tools","Character information")
    local box = section(p,"✦ PLAYER INFORMATION")
    info(box,"Username: "..player.Name)
    info(box,"Display Name: "..player.DisplayName)
    info(box,"User ID: "..player.UserId)

    action(box,"Refresh Information",function()
        notify("Information refreshed!")
    end)
end

-- VISUAL SETTINGS
pages["Visual Settings"] = function()
    local p = newPage("Visual Settings","Dashboard appearance")
    local box = section(p,"✦ THEME")
    info(box,"Purple pastel theme")
    action(box,"Show theme information",function()
        notify("KianBest Purple Theme")
    end)
end

-- MONSTER DATABASE
pages["Monster Database"] = function()
    local p = newPage("Monster Database","Sample NPC database")
    local box = section(p,"✦ SAMPLE DATA")
    for _,name in ipairs({
        "Training Dummy",
        "Forest Guardian",
        "Shadow Beast",
        "Crystal Golem",
        "Ancient Dragon"
    }) do
        info(box,"♡ "..name)
    end
    info(box,"Sample data only.")
end

-- SERVER MANAGER
pages["Server Manager"] = function()
    local p = newPage("Server Manager","Server information")
    local box = section(p,"✦ CURRENT SERVER")
    info(box,"Place ID: "..game.PlaceId)
    info(box,"Players: "..#Players:GetPlayers())
    info(box,"Server discovery needs authorized integration.")
end

-- PERFORMANCE
pages["Performance"] = function()
    local p = newPage("Performance","Client performance")
    local box = section(p,"✦ MONITOR")
    info(box,"FPS and ping are shown on Dashboard.")
    info(box,"Player count updates automatically.")
end

-- SETTINGS
pages["Settings"] = function()
    local p = newPage("Settings","Dashboard preferences")
    local box = section(p,"✦ ACCESSIBILITY")
    action(box,"Hide dashboard",function()
        main.Visible = false
        assist.Visible = true
    end)
    action(box,"Show dashboard",function()
        main.Visible = true
        assist.Visible = false
    end)
end

-- NAVIGATION
local items = {
    {"⌂","Dashboard"},
    {"◷","Trạng thái"},
    {"♧","Player Tools"},
    {"✧","Visual Settings"},
    {"♡","Monster Database"},
    {"☁","Server Manager"},
    {"◈","Performance"},
    {"⚙","Settings"}
}

local function openPage(name)
    if not pages[name] then return end
    pages[name]()

    for n,b in pairs(menuButtons) do
        b.BackgroundColor3 = n == name
            and Color3.fromRGB(83,62,119)
            or C.PANEL2
    end
end

for _,item in ipairs(items) do
    local b = create("TextButton",{
        Size = UDim2.new(1,0,0,40),
        BackgroundColor3 = C.PANEL2,
        Text = item[1].."   "..item[2],
        TextColor3 = C.WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamMedium
    },menu)

    round(b,9)
    menuButtons[item[2]] = b

    b.MouseButton1Click:Connect(function()
        openPage(item[2])
    end)
end

-- CLOSE = MINIMIZE
close.MouseButton1Click:Connect(function()
    main.Visible = false
    assist.Visible = true
end)

-- FLOATING BUTTON = REOPEN
assist.MouseButton1Click:Connect(function()
    main.Visible = true
    assist.Visible = false
end)

-- DRAG FLOATING BUTTON: MOUSE + TOUCH
local dragging = false
local dragStart
local startPos
local moved = false

assist.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        moved = false
        dragStart = input.Position
        startPos = assist.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if dragging and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        local delta = input.Position - dragStart

        if math.abs(delta.X) > 5 or math.abs(delta.Y) > 5 then
            moved = true
        end

        assist.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- RIGHT SHIFT: PC ONLY
UIS.InputBegan:Connect(function(input,processed)
    if processed then return end

    if input.KeyCode == Enum.KeyCode.RightShift then
        main.Visible = not main.Visible
        assist.Visible = not main.Visible
    end
end)

-- RESPONSIVE SIZE
local function updateScale()
    local camera = workspace.CurrentCamera
    if not camera then return end

    local size = camera.ViewportSize
    local scale = math.clamp(
        math.min(size.X/900,size.Y/600),
        0.48,
        1
    )

    local uiScale = main:FindFirstChildOfClass("UIScale")
    if not uiScale then
        uiScale = Instance.new("UIScale")
        uiScale.Parent = main
    end
    uiScale.Scale = scale
end

updateScale()

if workspace.CurrentCamera then
    workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(updateScale)
end

-- START
openPage("Dashboard")
notify("KianBest V7.2 loaded!")
print("KianBest V7.2 loaded!")
