-- ==========================================================
-- SCRIPT MENU SYSTEM V10.5 LIGHT (Kianbest Hub - Fix Safe Mode)
-- ==========================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

local Config = {
    PotatoMode = false,
    FullBright = false,
    LowPoly = false,
    NoclipEnabled = false,
    SpeedEnabled = false,
    SpeedValue = 24,
    FlyEnabled = false,
    FlySpeed = 50,
    ToggleKey = Enum.KeyCode.RightControl
}

-- KHỞI TẠO SCREEN GUI AN TOÀN
local ParentGui = (gethui and gethui()) or CoreGui or LocalPlayer:WaitForChild("PlayerGui")
if ParentGui:FindFirstChild("KianbestMenuSafe") then
    ParentGui.KianbestMenuSafe:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KianbestMenuSafe"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = ParentGui

-- MAIN FRAME
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 480, 0, 300)
MainFrame.Position = UDim2.new(0.5, -240, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
MainFrame.BackgroundTransparency = 0.1
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)
local Stroke = Instance.new("UIStroke", MainFrame)
Stroke.Color = Color3.fromRGB(255, 120, 180)
Stroke.Thickness = 2

-- HEADER
local Header = Instance.new("TextLabel")
Header.Size = UDim2.new(1, 0, 0, 35)
Header.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
Header.Text = "  🚀 Kianbest Hub - Safe Fix Lag Mode"
Header.TextColor3 = Color3.fromRGB(255, 120, 180)
Header.Font = Enum.Font.SourceSansBold
Header.TextSize = 14
Header.TextXAlignment = Enum.TextXAlignment.Left
Header.Parent = MainFrame
Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 8)

-- NÚT ĐÓNG / MỞ NHANH
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0, 45, 0, 45)
ToggleBtn.Position = UDim2.new(0, 10, 0.3, 0)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
ToggleBtn.Text = "UI"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 120, 180)
ToggleBtn.Font = Enum.Font.SourceSansBold
ToggleBtn.TextSize = 14
ToggleBtn.Active = true
ToggleBtn.Draggable = true
ToggleBtn.Parent = ScreenGui
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(1, 0)

local isOpen = true
ToggleBtn.MouseButton1Click:Connect(function()
    isOpen = not isOpen
    MainFrame.Visible = isOpen
end)

-- CONTAINER NỘI DUNG FIX LAG
local Content = Instance.new("ScrollingFrame")
Content.Size = UDim2.new(1, -20, 1, -50)
Content.Position = UDim2.new(0, 10, 0, 45)
Content.BackgroundTransparency = 1
Content.ScrollBarThickness = 4
Content.Parent = MainFrame

local UIList = Instance.new("UIListLayout", Content)
UIList.SortOrder = Enum.SortOrder.LayoutOrder
UIList.Padding = UDim.new(0, 6)

local function AddButton(text, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 32)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(240, 240, 240)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 13
    btn.Parent = Content
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    btn.MouseButton1Click:Connect(callback)
end

-- CÁC TÍNH NĂNG FIX LAG AN TOÀN
AddButton("⚡ Xóa Hiệu Ứng & Tăng Max FPS", function()
    Lighting.GlobalShadows = false
    Lighting.Brightness = 2
    for _, v in ipairs(Workspace:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Material = Enum.Material.SmoothPlastic
            v.CastShadow = false
        elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then
            v.Enabled = false
        end
    end
    print("Đã tối ưu hóa map thành công!")
end)

AddButton("🥔 Bật/Tắt Chế Độ Potato Graphics", function()
    Config.PotatoMode = not Config.PotatoMode
    for _, v in ipairs(Workspace:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Material = Config.PotatoMode and Enum.Material.SmoothPlastic or Enum.Material.Plastic
            v.CastShadow = not Config.PotatoMode
        end
    end
end)

AddButton("☀️ Bật Full Brightness (Sáng Mát)", function()
    Config.FullBright = not Config.FullBright
    Lighting.GlobalShadows = not Config.FullBright
    Lighting.Brightness = Config.FullBright and 3 or 1
end)

AddButton("🧹 Dọn Dẹp RAM Thủ Công (Clean Memory)", function()
    pcall(function()
        collectgarbage("collect")
        print("Đã dọn dẹp bộ nhớ RAM rác!")
    end)
end)

-- VÒNG LẶP XỬ LÝ (RENDER)
RunService.Stepped:Connect(function()
    if LocalPlayer.Character then
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum and Config.SpeedEnabled then
            hum.WalkSpeed = Config.SpeedValue
        end
    end
end)

print("Kianbest Safe Script Loaded Successfully!")
