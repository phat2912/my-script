-- ==========================================================
-- ROBLOX GAME UI FRAMEWORK - KIANBEST EDITION (V4.2 BLACK SCREEN)
-- ==========================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local VirtualUser = game:GetService("VirtualUser")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

----------------------------------------------------------
-- 🎨 HỆ THỐNG THEME MÀU
----------------------------------------------------------
local Themes = {
    DarkPurple = {
        Bg = Color3.fromRGB(18, 16, 26),
        Sidebar = Color3.fromRGB(12, 10, 18),
        Accent = Color3.fromRGB(165, 120, 255),
        Button = Color3.fromRGB(28, 22, 38),
        Text = Color3.fromRGB(240, 235, 255)
    },
    OceanBlue = {
        Bg = Color3.fromRGB(12, 18, 28),
        Sidebar = Color3.fromRGB(8, 12, 20),
        Accent = Color3.fromRGB(90, 185, 255),
        Button = Color3.fromRGB(20, 30, 44),
        Text = Color3.fromRGB(235, 248, 255)
    },
    NeonGreen = {
        Bg = Color3.fromRGB(14, 24, 18),
        Sidebar = Color3.fromRGB(8, 16, 12),
        Accent = Color3.fromRGB(50, 220, 120),
        Button = Color3.fromRGB(22, 38, 28),
        Text = Color3.fromRGB(230, 255, 240)
    },
    CrimsonRed = {
        Bg = Color3.fromRGB(26, 14, 16),
        Sidebar = Color3.fromRGB(18, 8, 10),
        Accent = Color3.fromRGB(255, 80, 100),
        Button = Color3.fromRGB(40, 20, 24),
        Text = Color3.fromRGB(255, 235, 238)
    }
}

local CurrentTheme = Themes.DarkPurple

if PlayerGui:FindFirstChild("KianBestUI") then 
    PlayerGui.KianBestUI:Destroy() 
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KianBestUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = PlayerGui

----------------------------------------------------------
-- 🌙 MÀN HÌNH ĐEN TIẾT KIỆM PIN / AFK
----------------------------------------------------------
local BlackScreen = Instance.new("TextButton")
BlackScreen.Name = "BlackScreenOverlay"
BlackScreen.Size = UDim2.new(1, 0, 1, 0)
BlackScreen.Position = UDim2.new(0, 0, 0, 0)
BlackScreen.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
BlackScreen.ZIndex = 999
BlackScreen.Visible = false
BlackScreen.AutoButtonColor = false
BlackScreen.Text = "🌙 CHẾ ĐỘ MÀN HÌNH ĐEN (AFK)\n\n(Bấm vào bất kỳ đâu trên màn hình để bật lại)"
BlackScreen.TextColor3 = Color3.fromRGB(180, 180, 180)
BlackScreen.Font = Enum.Font.SourceSansBold
BlackScreen.TextSize = 15
BlackScreen.Parent = ScreenGui

----------------------------------------------------------
-- 🔔 HỆ THỐNG THÔNG BÁO (NOTIFICATIONS)
----------------------------------------------------------
local NotiContainer = Instance.new("Frame")
NotiContainer.Name = "NotiContainer"
NotiContainer.Size = UDim2.new(0, 260, 0, 300)
NotiContainer.Position = UDim2.new(1, -270, 1, -20)
NotiContainer.AnchorPoint = Vector2.new(0, 1)
NotiContainer.BackgroundTransparency = 1
NotiContainer.ZIndex = 100
NotiContainer.Parent = ScreenGui

local NotiLayout = Instance.new("UIListLayout", NotiContainer)
NotiLayout.SortOrder = Enum.SortOrder.LayoutOrder
NotiLayout.Padding = UDim.new(0, 8)
NotiLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom

local function Notify(title, message, duration)
    duration = duration or 2.5

    local Toast = Instance.new("Frame")
    Toast.Name = "Toast"
    Toast.Size = UDim2.new(1, 0, 0, 40)
    Toast.Position = UDim2.new(0, 0, 0, 20)
    Toast.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
    Toast.BackgroundTransparency = 1
    Toast.BorderSizePixel = 0
    Toast.ClipsDescendants = true
    Toast.Parent = NotiContainer

    Instance.new("UICorner", Toast).CornerRadius = UDim.new(0, 8)

    local Stroke = Instance.new("UIStroke", Toast)
    Stroke.Color = CurrentTheme.Accent
    Stroke.Thickness = 1
    Stroke.Transparency = 1

    local TTitle = Instance.new("TextLabel")
    TTitle.Size = UDim2.new(1, -16, 0, 16)
    TTitle.Position = UDim2.new(0, 12, 0, 4)
    TTitle.Text = title or "KianBest"
    TTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    TTitle.Font = Enum.Font.SourceSansBold
    TTitle.TextSize = 13
    TTitle.TextXAlignment = Enum.TextXAlignment.Left
    TTitle.BackgroundTransparency = 1
    TTitle.TextTransparency = 1
    TTitle.Parent = Toast

    local TMsg = Instance.new("TextLabel")
    TMsg.Size = UDim2.new(1, -16, 0, 14)
    TMsg.Position = UDim2.new(0, 12, 0, 20)
    TMsg.Text = message or ""
    TMsg.TextColor3 = CurrentTheme.Accent
    TMsg.Font = Enum.Font.SourceSans
    TMsg.TextSize = 12
    TMsg.TextXAlignment = Enum.TextXAlignment.Left
    TMsg.BackgroundTransparency = 1
    TMsg.TextTransparency = 1
    TMsg.Parent = Toast

    local tweenIn = TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
    TweenService:Create(Toast, tweenIn, {Position = UDim2.new(0, 0, 0, 0), BackgroundTransparency = 0.1}):Play()
    TweenService:Create(Stroke, tweenIn, {Transparency = 0.3}):Play()
    TweenService:Create(TTitle, tweenIn, {TextTransparency = 0}):Play()
    TweenService:Create(TMsg, tweenIn, {TextTransparency = 0}):Play()

    task.delay(duration, function()
        if Toast and Toast.Parent then
            local tweenOut = TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
            TweenService:Create(Toast, tweenOut, {Position = UDim2.new(0, 0, 0, -20), BackgroundTransparency = 1}):Play()
            TweenService:Create(Stroke, tweenOut, {Transparency = 1}):Play()
            TweenService:Create(TTitle, tweenOut, {TextTransparency = 1}):Play()
            local last = TweenService:Create(TMsg, tweenOut, {TextTransparency = 1})
            last:Play()
            last.Completed:Connect(function()
                Toast:Destroy()
            end)
        end
    end)
end

----------------------------------------------------------
-- 🖥️ BẢNG ĐIỀU KHIỂN CHÍNH
----------------------------------------------------------
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 500, 0, 320)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -160)
MainFrame.BackgroundColor3 = CurrentTheme.Bg
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = CurrentTheme.Accent
MainStroke.Thickness = 1.5
MainStroke.Transparency = 0.3

local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 36)
Header.BackgroundColor3 = CurrentTheme.Sidebar
Header.BorderSizePixel = 0
Header.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0, 200, 1, 0)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.Text = "⚡ KIANBEST DASHBOARD V4.2"
Title.TextColor3 = CurrentTheme.Accent
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1
Title.Parent = Header

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 22, 0, 22)
CloseBtn.Position = UDim2.new(1, -28, 0, 7)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.TextSize = 12
CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 20, 25)
CloseBtn.BorderSizePixel = 0
CloseBtn.Parent = Header
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 130, 1, -36)
Sidebar.Position = UDim2.new(0, 0, 0, 36)
Sidebar.BackgroundColor3 = CurrentTheme.Sidebar
Sidebar.BorderSizePixel = 0
Sidebar.Parent = MainFrame

local ContentContainer = Instance.new("Frame")
ContentContainer.Size = UDim2.new(1, -138, 1, -44)
ContentContainer.Position = UDim2.new(0, 134, 0, 40)
ContentContainer.BackgroundTransparency = 1
ContentContainer.Parent = MainFrame

----------------------------------------------------------
-- 🔘 NÚT TRÒN BẬT / TẮT MENU
----------------------------------------------------------
local OpenToggleBtn = Instance.new("TextButton")
OpenToggleBtn.Name = "KianBestOpenBtn"
OpenToggleBtn.Size = UDim2.new(0, 50, 0, 50)
OpenToggleBtn.Position = UDim2.new(0.05, 0, 0.2, 0)
OpenToggleBtn.BackgroundColor3 = CurrentTheme.Sidebar
OpenToggleBtn.Text = "⚡"
OpenToggleBtn.TextSize = 22
OpenToggleBtn.TextColor3 = CurrentTheme.Accent
OpenToggleBtn.Font = Enum.Font.SourceSansBold
OpenToggleBtn.Active = true
OpenToggleBtn.Parent = ScreenGui

Instance.new("UICorner", OpenToggleBtn).CornerRadius = UDim.new(1, 0)

local OpenBtnStroke = Instance.new("UIStroke", OpenToggleBtn)
OpenBtnStroke.Color = CurrentTheme.Accent
OpenBtnStroke.Thickness = 2

local draggingToggle, dragStartToggle, startPosToggle
OpenToggleBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingToggle = true
        dragStartToggle = input.Position
        startPosToggle = OpenToggleBtn.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                draggingToggle = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingToggle and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStartToggle
        OpenToggleBtn.Position = UDim2.new(
            startPosToggle.X.Scale, startPosToggle.X.Offset + delta.X,
            startPosToggle.Y.Scale, startPosToggle.Y.Offset + delta.Y
        )
    end
end)

OpenToggleBtn.MouseButton1Click:Connect(function() MainFrame.Visible = not MainFrame.Visible end)
CloseBtn.MouseButton1Click:Connect(function() MainFrame.Visible = false end)

----------------------------------------------------------
-- 📑 HỆ THỐNG PHÂN TRANG (TABS)
----------------------------------------------------------
local Pages = {}
local TabButtons = {}

local function CreateTab(name, icon, index)
    local TabBtn = Instance.new("TextButton")
    TabBtn.Size = UDim2.new(1, -12, 0, 30)
    TabBtn.Position = UDim2.new(0, 6, 0, 6 + (index - 1) * 34)
    TabBtn.Text = "  " .. icon .. " " .. name
    TabBtn.TextColor3 = CurrentTheme.Text
    TabBtn.Font = Enum.Font.SourceSansBold
    TabBtn.TextSize = 12
    TabBtn.TextXAlignment = Enum.TextXAlignment.Left
    TabBtn.BackgroundColor3 = CurrentTheme.Button
    TabBtn.BorderSizePixel = 0
    TabBtn.Parent = Sidebar
    Instance.new("UICorner", TabBtn).CornerRadius = UDim.new(0, 6)

    local Page = Instance.new("ScrollingFrame")
    Page.Size = UDim2.new(1, 0, 1, 0)
    Page.BackgroundTransparency = 1
    Page.ScrollBarThickness = 3
    Page.ScrollBarImageColor3 = CurrentTheme.Accent
    Page.Visible = (index == 1)
    Page.CanvasSize = UDim2.new(0, 0, 0, 0)
    Page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    Page.Parent = ContentContainer

    local PList = Instance.new("UIListLayout", Page)
    PList.SortOrder = Enum.SortOrder.LayoutOrder
    PList.Padding = UDim.new(0, 6)

    Pages[name] = Page
    table.insert(TabButtons, TabBtn)

    TabBtn.MouseButton1Click:Connect(function()
        for _, p in pairs(Pages) do p.Visible = false end
        for _, btn in ipairs(TabButtons) do btn.BackgroundColor3 = CurrentTheme.Button end
        Page.Visible = true
        TabBtn.BackgroundColor3 = CurrentTheme.Accent
    end)

    if index == 1 then TabBtn.BackgroundColor3 = CurrentTheme.Accent end
    return Page
end

----------------------------------------------------------
-- 🧩 COMPONENTS
----------------------------------------------------------
local function ApplyTheme(newTheme)
    CurrentTheme = newTheme
    MainFrame.BackgroundColor3 = newTheme.Bg
    Header.BackgroundColor3 = newTheme.Sidebar
    Sidebar.BackgroundColor3 = newTheme.Sidebar
    OpenToggleBtn.BackgroundColor3 = newTheme.Sidebar
    OpenToggleBtn.TextColor3 = newTheme.Accent
    OpenBtnStroke.Color = newTheme.Accent
    MainStroke.Color = newTheme.Accent
    Title.TextColor3 = newTheme.Accent

    for _, page in pairs(Pages) do page.ScrollBarImageColor3 = newTheme.Accent end
    for _, btn in ipairs(TabButtons) do
        if btn.BackgroundColor3 ~= CurrentTheme.Button then
            btn.BackgroundColor3 = newTheme.Accent
        else
            btn.BackgroundColor3 = newTheme.Button
        end
        btn.TextColor3 = newTheme.Text
    end
    Notify("KianBest", "Đã đổi Theme giao diện!")
end

local function CreateToggle(parent, title, defaultState, callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, -6, 0, 32)
    Btn.Text = "   " .. title
    Btn.TextColor3 = CurrentTheme.Text
    Btn.Font = Enum.Font.SourceSansBold
    Btn.TextSize = 12
    Btn.TextXAlignment = Enum.TextXAlignment.Left
    Btn.BackgroundColor3 = CurrentTheme.Button
    Btn.BorderSizePixel = 0
    Btn.Parent = parent
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 6)

    local Track = Instance.new("Frame")
    Track.Size = UDim2.new(0, 34, 0, 18)
    Track.Position = UDim2.new(1, -40, 0.5, -9)
    Track.BorderSizePixel = 0
    Track.Parent = Btn
    Instance.new("UICorner", Track).CornerRadius = UDim.new(1, 0)

    local Knob = Instance.new("Frame")
    Knob.Size = UDim2.new(0, 14, 0, 14)
    Knob.BorderSizePixel = 0
    Knob.Parent = Track
    Instance.new("UICorner", Knob).CornerRadius = UDim.new(1, 0)

    local state = defaultState

    local function UpdateVisual(animate)
        local targetColor = state and CurrentTheme.Accent or Color3.fromRGB(45, 45, 55)
        local targetPos = state and UDim2.new(1, -16, 0.5, -7) or UDim2.new(0, 2, 0.5, -7)

        if animate then
            TweenService:Create(Track, TweenInfo.new(0.2), {BackgroundColor3 = targetColor}):Play()
            TweenService:Create(Knob, TweenInfo.new(0.2), {Position = targetPos}):Play()
        else
            Track.BackgroundColor3 = targetColor
            Knob.Position = targetPos
        end
    end

    UpdateVisual(false)

    Btn.MouseButton1Click:Connect(function()
        state = not state
        UpdateVisual(true)
        Notify("KianBest", title .. ": " .. (state and "ĐÃ BẬT" or "ĐÃ TẮT"))
        callback(state)
    end)
end

local function CreateAdjuster(parent, title, minVal, maxVal, defaultVal, step, callback)
    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(1, -6, 0, 32)
    Frame.BackgroundColor3 = CurrentTheme.Button
    Frame.BorderSizePixel = 0
    Frame.Parent = parent
    Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 6)

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.6, 0, 1, 0)
    Label.Position = UDim2.new(0, 10, 0, 0)
    Label.Text = title .. ": " .. tostring(defaultVal)
    Label.TextColor3 = CurrentTheme.Text
    Label.Font = Enum.Font.SourceSansBold
    Label.TextSize = 12
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.BackgroundTransparency = 1
    Label.Parent = Frame

    local val = defaultVal

    local Minus = Instance.new("TextButton")
    Minus.Size = UDim2.new(0, 22, 0, 20)
    Minus.Position = UDim2.new(1, -52, 0.5, -10)
    Minus.Text = "-"
    Minus.Font = Enum.Font.SourceSansBold
    Minus.TextColor3 = Color3.fromRGB(255, 255, 255)
    Minus.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    Minus.BorderSizePixel = 0
    Minus.Parent = Frame
    Instance.new("UICorner", Minus).CornerRadius = UDim.new(0, 4)

    local Plus = Instance.new("TextButton")
    Plus.Size = UDim2.new(0, 22, 0, 20)
    Plus.Position = UDim2.new(1, -26, 0.5, -10)
    Plus.Text = "+"
    Plus.Font = Enum.Font.SourceSansBold
    Plus.TextColor3 = Color3.fromRGB(255, 255, 255)
    Plus.BackgroundColor3 = CurrentTheme.Accent
    Plus.BorderSizePixel = 0
    Plus.Parent = Frame
    Instance.new("UICorner", Plus).CornerRadius = UDim.new(0, 4)

    Minus.MouseButton1Click:Connect(function()
        val = math.max(minVal, val - step)
        Label.Text = title .. ": " .. tostring(val)
        callback(val)
    end)

    Plus.MouseButton1Click:Connect(function()
        val = math.min(maxVal, val + step)
        Label.Text = title .. ": " .. tostring(val)
        callback(val)
    end)
end

local function CreateButton(parent, title, callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, -6, 0, 32)
    Btn.Text = title
    Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    Btn.Font = Enum.Font.SourceSansBold
    Btn.TextSize = 12
    Btn.BackgroundColor3 = CurrentTheme.Button
    Btn.BorderSizePixel = 0
    Btn.Parent = parent
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 6)

    Btn.MouseButton1Click:Connect(callback)
end

----------------------------------------------------------
-- 🛠️ KHỞI TẠO MỤC TAB
----------------------------------------------------------
local GeneralPage  = CreateTab("Cấu Hình", "⚙️", 1)
local VisualPage   = CreateTab("Visuals", "👁️", 2)
local GraphicsPage = CreateTab("Đồ Họa", "🎨", 3)
local ThemePage    = CreateTab("Giao Diện", "🎭", 4)

----------------------------------------------------------
-- TAB 1: CẤU HÌNH, ANTI-BAN, ANTI-AFK & SERVER
----------------------------------------------------------
CreateAdjuster(GeneralPage, "Tốc độ chạy", 16, 200, 16, 5, function(val)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = val
    end
end)

CreateAdjuster(GeneralPage, "Sức nhảy", 50, 300, 50, 10, function(val)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.JumpPower = val
    end
end)

local noclipConn
CreateToggle(GeneralPage, "Xuyên Tường (Noclip)", false, function(enabled)
    if enabled then
        noclipConn = RunService.Stepped:Connect(function()
            if LocalPlayer.Character then
                for _, part in ipairs(LocalPlayer.Character:GetDescendants()) do
                    if part:IsA("BasePart") and part.CanCollide == true then
                        part.CanCollide = false
                    end
                end
            end
        end)
    else
        if noclipConn then noclipConn:Disconnect() end
    end
end)

local antiAfkConn
CreateToggle(GeneralPage, "Chống AFK (Anti-AFK)", true, function(enabled)
    if enabled then
        antiAfkConn = LocalPlayer.Idled:Connect(function()
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new())
            Notify("KianBest", "Đã chặn văng game do AFK!")
        end)
    else
        if antiAfkConn then antiAfkConn:Disconnect() end
    end
end)

local antiAdminEnabled = false
local function IsAdmin(player)
    if player.UserId == game.CreatorId then return true end
    if game.CreatorType == Enum.CreatorType.Group then
        local success, rank = pcall(function() return player:GetRankInGroup(game.CreatorId) end)
        if success and rank and rank >= 200 then return true end
    end
    return false
end

CreateToggle(GeneralPage, "Tự Kick khi Admin vào", false, function(enabled)
    antiAdminEnabled = enabled
    if enabled then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and IsAdmin(p) then
                LocalPlayer:Kick("\n[KianBest Anti-Ban]\nPhát hiện Admin/Moderator (" .. p.Name .. ") trong server!")
                return
            end
        end
    end
end)

Players.PlayerAdded:Connect(function(player)
    if antiAdminEnabled and IsAdmin(player) then
        LocalPlayer:Kick("\n[KianBest Anti-Ban]\nAdmin/Moderator (" .. player.Name .. ") vừa tham gia server!")
    end
end)

CreateButton(GeneralPage, "🔄 Vào Lại Server (Rejoin)", function()
    Notify("KianBest", "Đang kết nối lại server...")
    task.wait(0.5)
    if #Players:GetPlayers() <= 1 then
        TeleportService:Teleport(game.PlaceId, LocalPlayer)
    else
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, LocalPlayer)
    end
end)

CreateButton(GeneralPage, "🌐 Chuyển Server Ít Người", function()
    Notify("KianBest", "Đang tìm server ít người chơi...")
    task.spawn(function()
        local sfUrl = "https://games.roproxy.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
        local success, result = pcall(function()
            local raw = game:HttpGet(sfUrl)
            return HttpService:JSONDecode(raw)
        end)
        
        if success and result and result.data then
            for _, server in ipairs(result.data) do
                if server.playing and server.maxPlayers and server.playing < server.maxPlayers and server.id ~= game.JobId then
                    Notify("KianBest", "Chuyển sang server (" .. server.playing .. " người)...")
                    TeleportService:TeleportToPlaceInstance(game.PlaceId, server.id, LocalPlayer)
                    return
                end
            end
        end
        Notify("KianBest", "Không tìm thấy server khác hoặc lỗi Proxy!")
    end)
end)

----------------------------------------------------------
-- TAB 2: VISUALS
----------------------------------------------------------
local VisualState = { Chams = false, ESP = false }

local function ApplyVisuals(player)
    if player == LocalPlayer or not player.Character then return end
    local existingChams = player.Character:FindFirstChild("KianBestChams")
    if VisualState.Chams then
        if not existingChams then
            local highlight = Instance.new("Highlight")
            highlight.Name = "KianBestChams"
            highlight.FillColor = CurrentTheme.Accent
            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
            highlight.FillTransparency = 0.5
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            highlight.Parent = player.Character
        end
    else
        if existingChams then existingChams:Destroy() end
    end

    local head = player.Character:FindFirstChild("Head")
    if head then
        local existingESP = head:FindFirstChild("KianBestESP")
        if VisualState.ESP then
            if not existingESP then
                local billboard = Instance.new("BillboardGui")
                billboard.Name = "KianBestESP"
                billboard.Size = UDim2.new(0, 150, 0, 30)
                billboard.StudsOffset = Vector3.new(0, 2, 0)
                billboard.AlwaysOnTop = true
                billboard.Parent = head

                local textLabel = Instance.new("TextLabel")
                textLabel.Size = UDim2.new(1, 0, 1, 0)
                textLabel.BackgroundTransparency = 1
                textLabel.Text = player.DisplayName .. " (@" .. player.Name .. ")"
                textLabel.TextColor3 = CurrentTheme.Accent
                textLabel.Font = Enum.Font.SourceSansBold
                textLabel.TextSize = 13
                textLabel.Parent = billboard
            end
        else
            if existingESP then existingESP:Destroy() end
        end
    end
end

CreateToggle(VisualPage, "Chams (Phát Sáng Nhân Vật)", false, function(enabled)
    VisualState.Chams = enabled
    for _, p in ipairs(Players:GetPlayers()) do ApplyVisuals(p) end
end)

CreateToggle(VisualPage, "ESP (Tên Người Chơi)", false, function(enabled)
    VisualState.ESP = enabled
    for _, p in ipairs(Players:GetPlayers()) do ApplyVisuals(p) end
end)

----------------------------------------------------------
-- TAB 3: ĐỒ HỌA & TẮT RENDER + TỐI MÀN HÌNH
----------------------------------------------------------
CreateToggle(GraphicsPage, "Tắt Bóng Đổ (Disable Shadows)", true, function(enabled)
    Lighting.GlobalShadows = not enabled
end)

CreateButton(GraphicsPage, "Xóa Bầu Trời (Remove Sky)", function()
    for _, v in ipairs(Lighting:GetChildren()) do
        if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("PostEffect") then
            v:Destroy()
        end
    end
    Notify("KianBest", "Đã xóa hiệu ứng Bầu Trời!")
end)

CreateButton(GraphicsPage, "Giảm Chi Tiết Đồ Họa (Low Textures)", function()
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Material = Enum.Material.SmoothPlastic
        elseif v:IsA("Decal") or v:IsA("Texture") then
            v:Destroy()
        end
    end
    Notify("KianBest", "Đã hạ vật liệu SmoothPlastic!")
end)

-- CẬP NHẬT TÍNH NĂNG TẮT RENDER 3D VÀ LÀM TỐI MÀN HÌNH
CreateToggle(GraphicsPage, "Tắt Render 3D & Tối Màn Hình", false, function(enabled)
    BlackScreen.Visible = enabled
    pcall(function()
        RunService:Set3dRenderingEnabled(not enabled)
    end)
    if enabled then
        Notify("KianBest", "Đã bật chế độ Tối Màn Hình AFK!")
    else
        Notify("KianBest", "Đã tắt chế độ Tối Màn Hình!")
    end
end)

-- Click vào màn hình đen để tắt chế độ AFK
BlackScreen.MouseButton1Click:Connect(function()
    BlackScreen.Visible = false
    pcall(function()
        RunService:Set3dRenderingEnabled(true)
    end)
    Notify("KianBest", "Đã mở lại màn hình!")
end)

----------------------------------------------------------
-- TAB 4: THEME
----------------------------------------------------------
CreateButton(ThemePage, "🟣 Dark Purple", function() ApplyTheme(Themes.DarkPurple) end)
CreateButton(ThemePage, "🔵 Ocean Blue", function() ApplyTheme(Themes.OceanBlue) end)
CreateButton(ThemePage, "🟢 Neon Green", function() ApplyTheme(Themes.NeonGreen) end)
CreateButton(ThemePage, "🔴 Crimson Red", function() ApplyTheme(Themes.CrimsonRed) end)

Notify("KianBest", "KianBest V4.2 Đã sẵn sàng!")
