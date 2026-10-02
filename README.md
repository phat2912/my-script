-- ==========================================================
-- ROBLOX GAME UI FRAMEWORK - KIANBEST EDITION (WITH VISUALS)
-- Sử dụng chuẩn Roblox Luau cho Game Development
-- ==========================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- 1. CẤU HÌNH THEME (GIAO DIỆN)
local Themes = {
    DarkPurple = {
        Bg = Color3.fromRGB(18, 16, 26),
        Sidebar = Color3.fromRGB(12, 10, 18),
        Accent = Color3.fromRGB(165, 120, 255),
        Button = Color3.fromRGB(28, 22, 38),
        Text = Color3.fromRGB(240, 235, 255)
    }
}

local CurrentTheme = Themes.DarkPurple

-- Xóa UI cũ nếu tái khởi chạy
if PlayerGui:FindFirstChild("KianBestUI") then 
    PlayerGui.KianBestUI:Destroy() 
end

-- 2. TẠO SCREENGUI CHÍNH
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KianBestUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = PlayerGui

----------------------------------------------------------
-- 🔔 HỆ THỐNG THÔNG BÁO (TOAST NOTIFICATION SYSTEM)
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
-- 👁️ HỆ THỐNG XỬ LÝ VISUALS (CHAMS & ESP)
----------------------------------------------------------
local VisualState = {
    Chams = false,
    ESP = false
}

local function ApplyChams(player)
    if player == LocalPlayer or not player.Character then return end
    
    local existing = player.Character:FindFirstChild("KianBestChams")
    if VisualState.Chams then
        if not existing then
            local highlight = Instance.new("Highlight")
            highlight.Name = "KianBestChams"
            highlight.FillColor = CurrentTheme.Accent
            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
            highlight.FillTransparency = 0.5
            highlight.OutlineTransparency = 0
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            highlight.Parent = player.Character
        end
    else
        if existing then existing:Destroy() end
    end
end

local function ApplyESP(player)
    if player == LocalPlayer or not player.Character then return end
    local head = player.Character:FindFirstChild("Head")
    if not head then return end

    local existing = head:FindFirstChild("KianBestESP")
    if VisualState.ESP then
        if not existing then
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
        if existing then existing:Destroy() end
    end
end

local function RefreshVisuals()
    for _, player in ipairs(Players:GetPlayers()) do
        ApplyChams(player)
        ApplyESP(player)
    end
end

-- Tự động áp dụng Visuals cho người chơi mới tham gia hoặc hồi sinh
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        task.wait(0.5)
        ApplyChams(player)
        ApplyESP(player)
    end)
end)

for _, player in ipairs(Players:GetPlayers()) do
    player.CharacterAdded:Connect(function()
        task.wait(0.5)
        ApplyChams(player)
        ApplyESP(player)
    end)
end

----------------------------------------------------------
-- 🖥️ BẢNG ĐIỀU KHIỂN CHÍNH (MAIN PANEL)
----------------------------------------------------------
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 500, 0, 320)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -160)
MainFrame.BackgroundColor3 = CurrentTheme.Bg
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = CurrentTheme.Accent
MainStroke.Thickness = 1.5
MainStroke.Transparency = 0.3

-- Thanh Tiêu Đề
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 36)
Header.BackgroundColor3 = CurrentTheme.Sidebar
Header.BorderSizePixel = 0
Header.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0, 200, 1, 0)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.Text = "⚡ KIANBEST DASHBOARD"
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

-- Sidebar & Container
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
        for _, btn in ipairs(TabButtons) do 
            btn.BackgroundColor3 = CurrentTheme.Button 
        end
        Page.Visible = true
        TabBtn.BackgroundColor3 = CurrentTheme.Accent
    end)

    if index == 1 then TabBtn.BackgroundColor3 = CurrentTheme.Accent end
    return Page
end

----------------------------------------------------------
-- 🧩 THÀNH PHẦN GIAO DIỆN (UI COMPONENTS)
----------------------------------------------------------
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
-- 🛠️ KHỞI TẠO CÁC TAB & TÍNH NĂNG
----------------------------------------------------------
local GeneralPage = CreateTab("Cấu Hình", "⚙️", 1)
local VisualPage  = CreateTab("Visuals", "👁️", 2)
local GraphicsPage = CreateTab("Đồ Họa", "🎨", 3)

-- Tab 1: Cấu hình
CreateButton(GeneralPage, "Lưu Cài Đặt", function()
    Notify("KianBest", "Đã lưu cài đặt game thành công!")
end)

-- Tab 2: Visuals (Mới Thêm)
CreateToggle(VisualPage, "Chams (Phát Sáng Nhân Vật)", false, function(enabled)
    VisualState.Chams = enabled
    RefreshVisuals()
end)

CreateToggle(VisualPage, "ESP (Hiển Thị Tên Người Chơi)", false, function(enabled)
    VisualState.ESP = enabled
    RefreshVisuals()
end)

-- Tab 3: Đồ họa
CreateToggle(GraphicsPage, "Bóng Đổ (Shadows)", true, function(enabled)
    game.Lighting.GlobalShadows = enabled
end)

-- Đóng Menu
CloseBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    Notify("KianBest", "Đã ẩn giao diện (Bấm F2 để mở lại)")
end)

-- Phím tắt F2
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.F2 then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

Notify("KianBest", "KianBest Framework + Visuals đã sẵn sàng!")
