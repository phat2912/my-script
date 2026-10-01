loadstring([[
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

local Config = {
    ESPEnabled = false,
    ESPColor = Color3.fromRGB(255, 50, 50),
    ToggleKey = Enum.KeyCode.RightControl
}

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

local ParentGui = (gethui and gethui()) or CoreGui or LocalPlayer:WaitForChild("PlayerGui")
if ParentGui:FindFirstChild("CustomESPMenu") then
    ParentGui.CustomESPMenu:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CustomESPMenu"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = ParentGui

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 480, 0, 300)
MainFrame.Position = UDim2.new(0.5, -240, 0.5, -150)
MainFrame.BackgroundColor3 = Themes.Dark.Bg
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = MainFrame

local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 35)
Header.BackgroundColor3 = Themes.Dark.Sidebar
Header.BorderSizePixel = 0
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
Title.Parent = Header

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 25, 0, 25)
CloseBtn.Position = UDim2.new(1, -30, 0, 5)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 80, 80)
CloseBtn.TextSize = 14
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
CloseBtn.BorderSizePixel = 0
CloseBtn.Parent = Header

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 4)
CloseCorner.Parent = CloseBtn

local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 120, 1, -35)
Sidebar.Position = UDim2.new(0, 0, 0, 35)
Sidebar.BackgroundColor3 = Themes.Dark.Sidebar
Sidebar.BorderSizePixel = 0
Sidebar.Parent = MainFrame

local ContentContainer = Instance.new("Frame")
ContentContainer.Size = UDim2.new(1, -130, 1, -45)
ContentContainer.Position = UDim2.new(0, 125, 0, 40)
ContentContainer.BackgroundTransparency = 1
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
    TabBtn.Parent = Sidebar

    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 4)
    BtnCorner.Parent = TabBtn

    local Page = Instance.new("Frame")
    Page.Size = UDim2.new(1, 0, 1, 0)
    Page.BackgroundTransparency = 1
    Page.Visible = (posIndex == 1)
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

local ESPToggleBtn = Instance.new("TextButton")
ESPToggleBtn.Size = UDim2.new(1, 0, 0, 35)
ESPToggleBtn.Position = UDim2.new(0, 0, 0, 5)
ESPToggleBtn.Text = "ESP Players: OFF"
ESPToggleBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
ESPToggleBtn.Font = Enum.Font.SourceSansBold
ESPToggleBtn.TextSize = 15
ESPToggleBtn.BackgroundColor3 = Themes.Dark.Button
ESPToggleBtn.BorderSizePixel = 0
ESPToggleBtn.Parent = VisualsPage

local ESPCorner = Instance.new("UICorner")
ESPCorner.CornerRadius = UDim.new(0, 6)
ESPCorner.Parent = ESPToggleBtn

local ThemeLabel = Instance.new("TextLabel")
ThemeLabel.Size = UDim2.new(1, 0, 0, 20)
ThemeLabel.Position = UDim2.new(0, 0, 0, 5)
ThemeLabel.Text = "Chọn Theme Menu:"
ThemeLabel.TextColor3 = Themes.Dark.Text
ThemeLabel.Font = Enum.Font.SourceSansBold
ThemeLabel.TextSize = 14
ThemeLabel.TextXAlignment = Enum.TextXAlignment.Left
ThemeLabel.BackgroundTransparency = 1
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

        for _, tabData in pairs(Pages) do
            tabData.Button.BackgroundColor3 = selected.Button
            tabData.Button.TextColor3 = selected.Text
        end
    end)
end

local KeyInfo = Instance.new("TextLabel")
KeyInfo.Size = UDim2.new(1, 0, 0, 20)
KeyInfo.Position = UDim2.new(0, 0, 0, 80)
KeyInfo.Text = "Phím Ẩn/Hiện Menu: [ RightControl ]"
KeyInfo.TextColor3 = Color3.fromRGB(180, 180, 180)
KeyInfo.Font = Enum.Font.SourceSansItalic
KeyInfo.TextSize = 13
KeyInfo.TextXAlignment = Enum.TextXAlignment.Left
KeyInfo.BackgroundTransparency = 1
KeyInfo.Parent = SettingsPage

local function ToggleMenu()
    MainFrame.Visible = not MainFrame.Visible
end

CloseBtn.MouseButton1Click:Connect(ToggleMenu)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Config.ToggleKey then
        ToggleMenu()
    end
end)

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
]])()

