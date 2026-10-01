-- ==========================================================
-- SCRIPT ESP PLAYER & TAB MENU SYSTEM (WITH CUSTOM BG IMAGE)
-- ==========================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

-- 1. Cấu hình & Trạng thái (Config)
local Config = {
    ESPEnabled = false,
    ESPColor = Color3.fromRGB(255, 50, 50),
    ToggleKey = Enum.KeyCode.RightControl
}

-- 2. Bộ chủ đề giao diện (Themes)
local Themes = {
    Dark = {
        Bg = Color3.fromRGB(30, 30, 35),
        Sidebar = Color3.fromRGB(20, 20, 25),
        Accent = Color3.fromRGB(60, 60, 70),
        Button = Color3.fromRGB(45, 45, 55),
        Text = Color3.fromRGB(255, 255, 255)
    },
    Cyan = {
        Bg = Color3.fromRGB(20, 30, 40),
        Sidebar = Color3.fromRGB(12, 20, 28),
        Accent = Color3.fromRGB(0, 170, 255),
        Button = Color3.fromRGB(25, 45, 60),
        Text = Color3.fromRGB(255, 255, 255)
    },
    Purple = {
        Bg = Color3.fromRGB(30, 20, 40),
        Sidebar = Color3.fromRGB(20, 12, 28),
        Accent = Color3.fromRGB(170, 0, 255),
        Button = Color3.fromRGB(45, 25, 60),
        Text = Color3.fromRGB(255, 255, 255)
    },
    Red = {
        Bg = Color3.fromRGB(40, 20, 20),
        Sidebar = Color3.fromRGB(28, 12, 12),
        Accent = Color3.fromRGB(255, 60, 60),
        Button = Color3.fromRGB(60, 25, 25),
        Text = Color3.fromRGB(255, 255, 255)
    }
}

-- 3. Tạo ScreenGui
local ParentGui = (gethui and gethui()) or CoreGui or LocalPlayer:WaitForChild("PlayerGui")
if ParentGui:FindFirstChild("CustomESPMenu") then
    ParentGui.CustomESPMenu:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CustomESPMenu"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = ParentGui

-- Background Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 480, 0, 300)
MainFrame.Position = UDim2.new(0.5, -240, 0.5, -150)
MainFrame.BackgroundColor3 = Themes.Dark.Bg
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ClipsDescendants = true -- Bo góc toàn bộ ảnh nền
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = MainFrame

-- Ảnh nền Menu (Custom Background Image)
local MainBgImage = Instance.new("ImageLabel")
MainBgImage.Name = "MainBgImage"
MainBgImage.Size = UDim2.new(1, 0, 1, 0)
MainBgImage.Position = UDim2.new(0, 0, 0, 0)
MainBgImage.BackgroundTransparency = 1
MainBgImage.ImageTransparency = 0.5 -- Độ mờ của ảnh (0.5 cho vừa mắt)
MainBgImage.ScaleType = Enum.ScaleType.Crop
MainBgImage.Image = "rbxassetid://6031075931" -- Ảnh mặc định cực ngầu
MainBgImage.ZIndex = 0
MainBgImage.Parent = MainFrame

-- Thanh Tiêu Đề (Header)
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 35)
Header.BackgroundColor3 = Themes.Dark.Sidebar
Header.BackgroundTransparency = 0.2
Header.BorderSizePixel = 0
Header.ZIndex = 2
Header.Parent = MainFrame

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0, 8)
HeaderCorner.Parent = Header

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -40, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.Text = "PLAYER ESP & MENU SYSTEM"
Title.TextColor3 = Themes.Dark.Text
Title.TextSize = 14
Title.Font = Enum.Font.SourceSansBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1
Title.ZIndex = 3
Title.Parent = Header

-- Nút Đóng (Close Button)
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 25, 0, 25)
CloseBtn.Position = UDim2.new(1, -30, 0, 5)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 80, 80)
CloseBtn.TextSize = 14
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
CloseBtn.BorderSizePixel = 0
CloseBtn.ZIndex = 3
CloseBtn.Parent = Header

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 4)
CloseCorner.Parent = CloseBtn

----------------------------------------------------------
-- NÚT TRÒN ĐÓNG/MỞ MENU (TOGGLE BUTTON)
----------------------------------------------------------
local ToggleBtn = Instance.new("ImageButton")
ToggleBtn.Name = "ToggleButton"
ToggleBtn.Size = UDim2.new(0, 50, 0, 50)
ToggleBtn.Position = UDim2.new(0, 15, 0.4, 0)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
ToggleBtn.Image = "rbxassetid://6031075931"
ToggleBtn.Active = true
ToggleBtn.Draggable = true
ToggleBtn.Parent = ScreenGui

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(1, 0)
ButtonCorner.Parent = ToggleBtn

local ButtonStroke = Instance.new("UIStroke")
ButtonStroke.Color = Color3.fromRGB(0, 170, 255)
ButtonStroke.Thickness = 2.5
ButtonStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
ButtonStroke.Parent = ToggleBtn

local function ToggleMenu()
    MainFrame.Visible = not MainFrame.Visible
end

ToggleBtn.MouseButton1Click:Connect(ToggleMenu)
CloseBtn.MouseButton1Click:Connect(ToggleMenu)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Config.ToggleKey then
        ToggleMenu()
    end
end)

----------------------------------------------------------
-- SIDEBAR & TAB CONTENT
----------------------------------------------------------
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 120, 1, -35)
Sidebar.Position = UDim2.new(0, 0, 0, 35)
Sidebar.BackgroundColor3 = Themes.Dark.Sidebar
Sidebar.BackgroundTransparency = 0.2
Sidebar.BorderSizePixel = 0
Sidebar.ZIndex = 2
Sidebar.Parent = MainFrame

local ContentContainer = Instance.new("Frame")
ContentContainer.Size = UDim2.new(1, -130, 1, -45)
ContentContainer.Position = UDim2.new(0, 125, 0, 40)
ContentContainer.BackgroundTransparency = 1
ContentContainer.ZIndex = 2
ContentContainer.Parent = MainFrame

local Pages = {}

local function CreateTab(name, posIndex)
    local TabBtn = Instance.new("TextButton")
    TabBtn.Size = UDim2.new(1, -10, 0, 30)
    TabBtn.Position = UDim2.new(0, 5, 0, 5 + (posIndex - 1) * 35)
    TabBtn.Text = name
    TabBtn.TextColor3 = Themes.Dark.Text
    TabBtn.Font = Enum.Font.SourceSans
    TabBtn.TextSize = 14
    TabBtn.BackgroundColor3 = Themes.Dark.Button
    TabBtn.BorderSizePixel = 0
    TabBtn.ZIndex = 3
    TabBtn.Parent = Sidebar

    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 4)
    BtnCorner.Parent = TabBtn

    local Page = Instance.new("Frame")
    Page.Size = UDim2.new(1, 0, 1, 0)
    Page.BackgroundTransparency = 1
    Page.Visible = (posIndex == 1)
    Page.ZIndex = 2
    Page.Parent = ContentContainer

    Pages[name] = {Button = TabBtn, Page = Page}

    TabBtn.MouseButton1Click:Connect(function()
        for _, tab in pairs(Pages) do
            tab.Page.Visible = false
        end
        Page.Visible = true
    end)

    return Page
end

local VisualsPage = CreateTab("Visuals (ESP)", 1)
local SettingsPage = CreateTab("Settings", 2)

-- ESP BUTTON
local ESPToggleBtn = Instance.new("TextButton")
ESPToggleBtn.Size = UDim2.new(1, 0, 0, 35)
ESPToggleBtn.Position = UDim2.new(0, 0, 0, 5)
ESPToggleBtn.Text = "ESP Players: OFF"
ESPToggleBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
ESPToggleBtn.Font = Enum.Font.SourceSansBold
ESPToggleBtn.TextSize = 15
ESPToggleBtn.BackgroundColor3 = Themes.Dark.Button
ESPToggleBtn.BorderSizePixel = 0
ESPToggleBtn.ZIndex = 3
ESPToggleBtn.Parent = VisualsPage

local ESPCorner = Instance.new("UICorner")
ESPCorner.CornerRadius = UDim.new(0, 6)
ESPCorner.Parent = ESPToggleBtn

----------------------------------------------------------
-- THEME & CUSTOM BACKGROUND IMAGE SETTINGS
----------------------------------------------------------
local ThemeLabel = Instance.new("TextLabel")
ThemeLabel.Size = UDim2.new(1, 0, 0, 20)
ThemeLabel.Position = UDim2.new(0, 0, 0, 5)
ThemeLabel.Text = "Chọn Theme Menu:"
ThemeLabel.TextColor3 = Themes.Dark.Text
ThemeLabel.Font = Enum.Font.SourceSansBold
ThemeLabel.TextSize = 14
ThemeLabel.TextXAlignment = Enum.TextXAlignment.Left
ThemeLabel.BackgroundTransparency = 1
ThemeLabel.ZIndex = 3
ThemeLabel.Parent = SettingsPage

local themeList = {"Dark", "Cyan", "Purple", "Red"}
for i, themeName in ipairs(themeList) do
    local ThemeBtn = Instance.new("TextButton")
    ThemeBtn.Size = UDim2.new(0, 75, 0, 30)
    ThemeBtn.Position = UDim2.new(0, (i - 1) * 80, 0, 30)
    ThemeBtn.Text = themeName
    ThemeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    ThemeBtn.Font = Enum.Font.SourceSans
    ThemeBtn.TextSize = 13
    ThemeBtn.BackgroundColor3 = Themes[themeName].Accent
    ThemeBtn.BorderSizePixel = 0
    ThemeBtn.ZIndex = 3
    ThemeBtn.Parent = SettingsPage

    local TCorner = Instance.new("UICorner")
    TCorner.CornerRadius = UDim.new(0, 4)
    TCorner.Parent = ThemeBtn

    ThemeBtn.MouseButton1Click:Connect(function()
        local selected = Themes[themeName]
        MainFrame.BackgroundColor3 = selected.Bg
        Header.BackgroundColor3 = selected.Sidebar
        Sidebar.BackgroundColor3 = selected.Sidebar
        Title.TextColor3 = selected.Text
        ThemeLabel.TextColor3 = selected.Text
        ButtonStroke.Color = selected.Accent

        for _, tabData in pairs(Pages) do
            tabData.Button.BackgroundColor3 = selected.Button
            tabData.Button.TextColor3 = selected.Text
        end
    end)
end

-- MỤC ĐỔI NỀN BẰNG ID ẢNH
local ImageLabelHeader = Instance.new("TextLabel")
ImageLabelHeader.Size = UDim2.new(1, 0, 0, 20)
ImageLabelHeader.Position = UDim2.new(0, 0, 0, 75)
ImageLabelHeader.Text = "Đổi Ảnh Nền (Nhập ID Decal/Image):"
ImageLabelHeader.TextColor3 = Themes.Dark.Text
ImageLabelHeader.Font = Enum.Font.SourceSansBold
ImageLabelHeader.TextSize = 14
ImageLabelHeader.TextXAlignment = Enum.TextXAlignment.Left
ImageLabelHeader.BackgroundTransparency = 1
ImageLabelHeader.ZIndex = 3
ImageLabelHeader.Parent = SettingsPage

local ImageBox = Instance.new("TextBox")
ImageBox.Size = UDim2.new(0, 220, 0, 30)
ImageBox.Position = UDim2.new(0, 0, 0, 100)
ImageBox.PlaceholderText = "Nhập ID Ảnh (Ví dụ: 6031075931)..."
ImageBox.Text = ""
ImageBox.TextColor3 = Color3.fromRGB(255, 255, 255)
ImageBox.BackgroundColor3 = Themes.Dark.Button
ImageBox.Font = Enum.Font.SourceSans
ImageBox.TextSize = 13
ImageBox.BorderSizePixel = 0
ImageBox.ZIndex = 3
ImageBox.Parent = SettingsPage

local IBCorner = Instance.new("UICorner")
IBCorner.CornerRadius = UDim.new(0, 4)
IBCorner.Parent = ImageBox

local ApplyImgBtn = Instance.new("TextButton")
ApplyImgBtn.Size = UDim2.new(0, 95, 0, 30)
ApplyImgBtn.Position = UDim2.new(0, 230, 0, 100)
ApplyImgBtn.Text = "Áp dụng"
ApplyImgBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ApplyImgBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 100)
ApplyImgBtn.Font = Enum.Font.SourceSansBold
ApplyImgBtn.TextSize = 13
ApplyImgBtn.BorderSizePixel = 0
ApplyImgBtn.ZIndex = 3
ApplyImgBtn.Parent = SettingsPage

local ABCorner = Instance.new("UICorner")
ABCorner.CornerRadius = UDim.new(0, 4)
ABCorner.Parent = ApplyImgBtn

-- Sự kiện bấm nút Áp dụng Ảnh nền
ApplyImgBtn.MouseButton1Click:Connect(function()
    local text = ImageBox.Text
    local cleanID = string.match(text, "%d+") -- Tự lọc lấy dãy số ID từ văn bản/link
    
    if cleanID then
        MainBgImage.Image = "rbxassetid://" .. cleanID
        ToggleBtn.Image = "rbxassetid://" .. cleanID -- Đổi luôn ảnh ở nút tròn
        ImageBox.Text = "Đã đổi thành công!"
        task.wait(1.5)
        ImageBox.Text = ""
    else
        ImageBox.Text = "ID không hợp lệ!"
        task.wait(1.5)
        ImageBox.Text = ""
    end
end)

----------------------------------------------------------
-- LOGIC ESP HIGHLIGHT
----------------------------------------------------------
local function ApplyESP(player)
    if player == LocalPlayer then return end

    local function SetupCharacter(char)
        if not char then return end
        local highlight = char:FindFirstChild("ESPHighlight") or Instance.new("Highlight")
        highlight.Name = "ESPHighlight"
        highlight.Adornee = char
        highlight.FillColor = Config.ESPColor
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.FillTransparency = 0.5
        highlight.OutlineTransparency = 0
        highlight.Enabled = Config.ESPEnabled
        highlight.Parent = char
    end

    if player.Character then
        SetupCharacter(player.Character)
    end
    player.CharacterAdded:Connect(SetupCharacter)
end

for _, p in ipairs(Players:GetPlayers()) do
    ApplyESP(p)
end
Players.PlayerAdded:Connect(ApplyESP)

ESPToggleBtn.MouseButton1Click:Connect(function()
    Config.ESPEnabled = not Config.ESPEnabled
    if Config.ESPEnabled then
        ESPToggleBtn.Text = "ESP Players: ON"
        ESPToggleBtn.TextColor3 = Color3.fromRGB(100, 255, 100)
    else
        ESPToggleBtn.Text = "ESP Players: OFF"
        ESPToggleBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
    end

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character then
            local highlight = p.Character:FindFirstChild("ESPHighlight")
            if highlight then
                highlight.Enabled = Config.ESPEnabled
            end
        end
    end
end)
