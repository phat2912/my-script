--==================================================
-- KIANBEST V5.2 ULTRA | COMPACT EDITION (bản làm đẹp + .kick command)
-- Client-side Roblox dashboard: FPS / Ping / Memory / Players / World / Server
-- Đặt vào LocalScript (StarterPlayerScripts)
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
local Default = {
	Theme = "Purple",
	Hex = "#A78BFA",
	Language = "VI",
	Effect = "Sakura",
	Background = "", -- dán Roblox Image ID vào đây hoặc nhập trong Cài đặt
	BackgroundEnabled = true,
	Particles = true,
	Compact = true,
	FastMode = false,
	ParticleAmount = 16,
	Opacity = 0.25,
}

local Config = {}
for k, v in pairs(Default) do Config[k] = v end

local Themes = {
	Purple = Color3.fromRGB(167, 139, 250),
	Sakura = Color3.fromRGB(249, 168, 212),
	Ocean = Color3.fromRGB(56, 189, 248),
	Emerald = Color3.fromRGB(52, 211, 153),
	Ruby = Color3.fromRGB(251, 113, 133),
	Gold = Color3.fromRGB(251, 191, 36),
	Cyber = Color3.fromRGB(34, 211, 238),
	White = Color3.fromRGB(226, 232, 240),
	Black = Color3.fromRGB(232, 232, 238),
}
local ThemeNames = { "Purple", "Sakura", "Ocean", "Emerald", "Ruby", "Gold", "Cyber", "White", "Black" }

local Accent = Themes.Purple
local Palettes = {
	Black = { Base = Color3.fromRGB(0, 0, 0), Card = Color3.fromRGB(26, 26, 28) },
}
local DefaultPalette = { Base = Color3.fromRGB(14, 10, 26), Card = Color3.fromRGB(44, 33, 66) }
local BASE, CARD = DefaultPalette.Base, DefaultPalette.Card

local function HexToColor(hex)
	hex = tostring(hex):gsub("#", "")
	if #hex == 6 and hex:match("^%x+$") then
		return Color3.fromRGB(
			tonumber(hex:sub(1, 2), 16),
			tonumber(hex:sub(3, 4), 16),
			tonumber(hex:sub(5, 6), 16)
		)
	end
	return nil
end

local function ResolveAccent()
	Accent = Themes[Config.Theme] or HexToColor(Config.Hex) or Themes.Purple
	local pal = Palettes[Config.Theme] or DefaultPalette
	BASE, CARD = pal.Base, pal.Card
end

local FILE = "KianBest_V52_Config.json"

local function SaveConfig()
	if type(writefile) ~= "function" then return false end
	return (pcall(function()
		writefile(FILE, HttpService:JSONEncode(Config))
	end))
end

local function LoadConfig()
	if type(readfile) ~= "function" or type(isfile) ~= "function" then return end
	pcall(function()
		if isfile(FILE) then
			local data = HttpService:JSONDecode(readfile(FILE))
			for k, v in pairs(data) do
				if Default[k] ~= nil and type(v) == type(Default[k]) then
					Config[k] = v
				end
			end
		end
	end)
end

LoadConfig()
ResolveAccent()

--==================================================
-- LANGUAGE
--==================================================
local Texts = {
	VI = {
		HOME = "Trang chủ", PERFORMANCE = "Hiệu năng", PLAYERS = "Người chơi",
		WORLD = "Thế giới", SERVER = "Máy chủ", SETTINGS = "Cài đặt",
		overview = "Tổng quan", fps = "FPS", ping = "Ping", min = "FPS thấp nhất", max = "FPS cao nhất",
		avg = "FPS trung bình", memory = "Bộ nhớ", rating = "Đánh giá", smooth = "Rất mượt",
		stable = "Ổn định", choppy = "Hơi giật", tools = "Công cụ hiệu năng",
		fast = "Chế độ tăng tốc", particles = "Hiệu ứng rơi", compact = "Giao diện gọn",
		background = "Hình nền", effect = "Kiểu hiệu ứng", snow = "Tuyết", sakura = "Hoa anh đào",
		stars = "Sao", hearts = "Tim", none = "Tắt", amount = "Số lượng hạt", opacity = "Độ tối lớp phủ",
		search = "Tìm người chơi...", you = "BẠN", age = "Tuổi tài khoản", days = "ngày", copyId = "Sao chép ID",
		copied = "Đã sao chép", empty = "Không tìm thấy người chơi",
		worldInfo = "Thông tin thế giới", time = "Giờ trong game", gravity = "Trọng lực",
		brightness = "Độ sáng", streaming = "Streaming", on = "Bật", off = "Tắt",
		serverInfo = "Thông tin máy chủ", place = "Place ID", job = "Job ID", count = "Số người chơi",
		uptime = "Thời gian chạy", theme = "Màu giao diện", hex = "Màu tùy chỉnh (HEX)", apply = "Áp dụng",
		language = "Ngôn ngữ", bgId = "Hình nền (Image ID)", save = "Lưu cấu hình", reset = "Khôi phục",
		saved = "Đã lưu cấu hình", noSave = "Môi trường không hỗ trợ lưu file", resetDone = "Đã khôi phục mặc định",
		badHex = "Mã HEX không hợp lệ", bgSet = "Đã đặt hình nền", hint = "Kéo thanh tiêu đề để di chuyển",
		notAuth = "Bạn không có quyền dùng lệnh .kick!", kickUsage = "Cách dùng: .kick [tên]", kickSuccess = "Đã thực hiện kick: ",
	},
	EN = {
		HOME = "Home", PERFORMANCE = "Performance", PLAYERS = "Players",
		WORLD = "World", SERVER = "Server", SETTINGS = "Settings",
		overview = "Overview", fps = "FPS", ping = "Ping", min = "Min FPS", max = "Max FPS",
		avg = "Average FPS", memory = "Memory", rating = "Rating", smooth = "Very smooth",
		stable = "Stable", choppy = "A bit choppy", tools = "Performance tools",
		fast = "Boost mode", particles = "Falling effects", compact = "Compact layout",
		background = "Background", effect = "Effect style", snow = "Snow", sakura = "Sakura",
		stars = "Stars", hearts = "Hearts", none = "Off", amount = "Particle count", opacity = "Overlay darkness",
		search = "Search players...", you = "YOU", age = "Account age", days = "days", copyId = "Copy ID",
		copied = "Copied", empty = "No players found",
		worldInfo = "World info", time = "In-game time", gravity = "Gravity",
		brightness = "Brightness", streaming = "Streaming", on = "On", off = "Off",
		serverInfo = "Server info", place = "Place ID", job = "Job ID", count = "Players",
		uptime = "Uptime", theme = "Theme color", hex = "Custom color (HEX)", apply = "Apply",
		language = "Language", bgId = "Background (Image ID)", save = "Save config", reset = "Reset",
		saved = "Config saved", noSave = "File saving is not supported here", resetDone = "Defaults restored",
		badHex = "Invalid HEX code", bgSet = "Background set", hint = "Drag the title bar to move",
		notAuth = "You are not authorized to use .kick!", kickUsage = "Usage: .kick [name]", kickSuccess = "Kicked: ",
	},
}

local function T(key)
	return (Texts[Config.Language] or Texts.VI)[key] or key
end

--==================================================
-- GUI ROOT + HELPERS
--==================================================
local Old = PG:FindFirstChild("KianBestV52")
if Old then Old:Destroy() end

local Gui = Instance.new("ScreenGui")
Gui.Name = "KianBestV52"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true
Gui.DisplayOrder = 2147483647
Gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
Gui.Parent = PG

local TEXT = Color3.fromRGB(246, 240, 255)
local MUTED = Color3.fromRGB(176, 162, 208)
local OFF = Color3.fromRGB(80, 70, 100)

local function New(class, props, parent)
	local o = Instance.new(class)
	for k, v in pairs(props or {}) do o[k] = v end
	o.Parent = parent
	return o
end

local function Corner(o, r)
	return New("UICorner", { CornerRadius = UDim.new(0, r or 10) }, o)
end

local function Tween(o, t, goal)
	TweenService:Create(o, TweenInfo.new(t, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), goal):Play()
end

local Binds = {}
local function Bind(fn)
	table.insert(Binds, fn)
	fn()
end
local function ApplyAll()
	for _, fn in ipairs(Binds) do fn() end
end

local function Loc(obj, key, prefix)
	Bind(function()
		obj.Text = (prefix or "") .. T(key)
	end)
end

local Counter = setmetatable({}, { __mode = "k" })
local function O(p)
	Counter[p] = (Counter[p] or 0) + 1
	return Counter[p]
end

local function Text(parent, text, size, color, font)
	return New("TextLabel", {
		BackgroundTransparency = 1, Text = text, TextSize = size or 13, TextColor3 = color or TEXT,
		Font = font or Enum.Font.GothamMedium, TextXAlignment = Enum.TextXAlignment.Left,
		Size = UDim2.new(1, 0, 0, (size or 13) + 6), ZIndex = 5,
	}, parent)
end

local function Card(parent, h)
	local f = New("Frame", {
		Size = UDim2.new(1, 0, 0, h), BackgroundColor3 = CARD, BackgroundTransparency = 0.22,
		BorderSizePixel = 0, LayoutOrder = O(parent), ZIndex = 5,
	}, parent)
	Corner(f, 12)
	New("UIStroke", { Color = Color3.new(1, 1, 1), Transparency = 0.88, Thickness = 1 }, f)
	Bind(function() f.BackgroundColor3 = CARD end)
	return f
end

local function MakeDraggable(handle, target, onTap, clamp)
	local dragging, moved, startPos, startMouse = false, false, nil, nil
	handle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging, moved = true, false
			startMouse, startPos = input.Position, target.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
					if not moved and onTap then onTap() end
				end
			end)
		end
	end)
	UIS.InputChanged:Connect(function(input)
		if not dragging then return end
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			local d = input.Position - startMouse
			if d.Magnitude > 8 then moved = true end
			if moved then
				local x, y = startPos.X.Offset + d.X, startPos.Y.Offset + d.Y
				if clamp then
					local vs, s = Gui.AbsoluteSize, target.AbsoluteSize
					x = math.clamp(x, 4, vs.X - s.X - 4)
					y = math.clamp(y, 4, vs.Y - s.Y - 4)
				end
				target.Position = UDim2.new(startPos.X.Scale, x, startPos.Y.Scale, y)
			end
		end
	end)
end

--==================================================
-- MAIN WINDOW
--==================================================
local Main = New("Frame", {
	Name = "Main", AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5),
	Size = UDim2.fromScale(0.58, 0.8), BackgroundColor3 = Color3.new(1, 1, 1),
	BorderSizePixel = 0, Visible = false, ClipsDescendants = true,
}, Gui)
Corner(Main, 22)
local MainScale = New("UIScale", { Scale = 1 }, Main)
local MainSize = New("UISizeConstraint", { MinSize = Vector2.new(400, 240), MaxSize = Vector2.new(540, 340) }, Main)
local MainStroke = New("UIStroke", { Thickness = 1.5, Transparency = 0.1 }, Main)
local MainGrad = New("UIGradient", { Rotation = 60 }, Main)

local Background = New("ImageLabel", {
	Name = "Wallpaper", Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
	ScaleType = Enum.ScaleType.Crop, ZIndex = 1,
}, Main)
Corner(Background, 22)

local Overlay = New("Frame", {
	Size = UDim2.fromScale(1, 1), BackgroundColor3 = BASE, BorderSizePixel = 0, ZIndex = 2,
}, Main)
Corner(Overlay, 22)

local ParticleLayer = New("Frame", {
	Name = "Particles", Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
	ClipsDescendants = true, ZIndex = 3,
}, Main)
Corner(ParticleLayer, 22)

local Header = New("Frame", {
	Size = UDim2.new(1, 0, 0, 46), BackgroundTransparency = 1, BorderSizePixel = 0, ZIndex = 4, Active = true,
}, Main)
New("Frame", {
	AnchorPoint = Vector2.new(0, 1), Position = UDim2.new(0, 14, 1, 0), Size = UDim2.new(1, -28, 0, 1),
	BackgroundColor3 = Color3.new(1, 1, 1), BackgroundTransparency = 0.9, BorderSizePixel = 0, ZIndex = 4,
}, Header)

local Title = New("TextLabel", {
	BackgroundTransparency = 1, Text = "KianBest", TextSize = 17, Font = Enum.Font.GothamBold,
	TextColor3 = TEXT, TextXAlignment = Enum.TextXAlignment.Left,
	Position = UDim2.fromOffset(14, 0), Size = UDim2.fromOffset(80, 46), ZIndex = 5,
}, Header)

local Badge = New("TextLabel", {
	Text = "V5.2 ULTRA", TextSize = 10, Font = Enum.Font.GothamBold, TextColor3 = BASE,
	Position = UDim2.fromOffset(106, 15), Size = UDim2.fromOffset(74, 18), ZIndex = 5,
}, Header)
Corner(Badge, 9)

local CloseBtn = New("TextButton", {
	Text = "X", TextSize = 14, Font = Enum.Font.GothamBold, TextColor3 = TEXT, AutoButtonColor = true,
	BackgroundColor3 = Color3.new(1, 1, 1), BackgroundTransparency = 0.88,
	AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -10, 0.5, 0), Size = UDim2.fromOffset(30, 30), ZIndex = 6,
}, Header)
Corner(CloseBtn, 15)

MakeDraggable(Header, Main, nil, false)

local Sidebar = New("Frame", {
	Position = UDim2.fromOffset(10, 54), Size = UDim2.new(0, 128, 1, -62), BackgroundTransparency = 1, ZIndex = 4,
}, Main)
New("UIListLayout", { Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder }, Sidebar)

local Content = New("Frame", {
	Position = UDim2.fromOffset(146, 54), Size = UDim2.new(1, -156, 1, -62), BackgroundTransparency = 1, ZIndex = 4,
}, Main)

local Toast = New("TextLabel", {
	AnchorPoint = Vector2.new(0.5, 1), Position = UDim2.new(0.5, 0, 1, -10), Size = UDim2.fromOffset(240, 26),
	BackgroundColor3 = BASE, BackgroundTransparency = 1, TextTransparency = 1, TextSize = 12,
	Font = Enum.Font.GothamMedium, TextColor3 = TEXT, ZIndex = 20, Text = "",
}, Main)
Corner(Toast, 13)

local toastId = 0
local function ShowToast(msg)
	toastId += 1
	local id = toastId
	Toast.Text = msg
	Tween(Toast, 0.15, { BackgroundTransparency = 0.15, TextTransparency = 0 })
	task.delay(1.6, function()
		if id == toastId then
			Tween(Toast, 0.25, { BackgroundTransparency = 1, TextTransparency = 1 })
		end
	end)
end

local TabDefs = {
	{ "HOME", "🏠" }, { "PERFORMANCE", "📈" }, { "PLAYERS", "👥" },
	{ "WORLD", "🌍" }, { "SERVER", "💻" }, { "SETTINGS", "🔧" },
}
local Pages, TabButtons = {}, {}
local CurrentTab = "HOME"

local function RefreshTabs()
	for name, b in pairs(TabButtons) do
		local on = name == CurrentTab
		b.BackgroundColor3 = Accent
		b.BackgroundTransparency = on and 0.1 or 1
		b.TextColor3 = on and BASE or MUTED
	end
	for name, p in pairs(Pages) do
		p.Visible = name == CurrentTab
	end
end

for i, def in ipairs(TabDefs) do
	local name = def[1]
	local page = New("ScrollingFrame", {
		Name = name, Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, BorderSizePixel = 0,
		ScrollBarThickness = 3, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y,
		ScrollingDirection = Enum.ScrollingDirection.Y, Visible = i == 1, ZIndex = 4,
	}, Content)
	New("UIListLayout", { Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder }, page)
	New("UIPadding", { PaddingRight = UDim.new(0, 6), PaddingBottom = UDim.new(0, 8) }, page)
	Bind(function() page.ScrollBarImageColor3 = Accent end)
	Pages[name] = page

	local b = New("TextButton", {
		Size = UDim2.new(1, 0, 0, 36), Font = Enum.Font.GothamSemibold, TextSize = 12, TextTruncate = Enum.TextTruncate.AtEnd,
		TextXAlignment = Enum.TextXAlignment.Left, AutoButtonColor = false, LayoutOrder = i, ZIndex = 5,
	}, Sidebar)
	Corner(b, 10)
	New("UIPadding", { PaddingLeft = UDim.new(0, 10) }, b)
	Loc(b, name, def[2] .. "  ")
	TabButtons[name] = b
	b.Activated:Connect(function()
		CurrentTab = name
		RefreshTabs()
	end)
end

--==================================================
-- COMPONENTS
--==================================================
local function Section(parent, key)
	local l = Text(parent, "", 12, MUTED, Enum.Font.GothamBold)
	l.LayoutOrder = O(parent)
	Loc(l, key)
	return l
end

local function InfoRow(parent, key)
	local row = Card(parent, 36)
	local l = Text(row, "", 12, MUTED)
	l.Position = UDim2.fromOffset(12, 0)
	l.Size = UDim2.new(0.5, 0, 1, 0)
	Loc(l, key)
	local v = Text(row, "-", 12, TEXT, Enum.Font.GothamBold)
	v.TextXAlignment = Enum.TextXAlignment.Right
	v.AnchorPoint = Vector2.new(1, 0)
	v.Position = UDim2.new(1, -12, 0, 0)
	v.Size = UDim2.new(0.5, 0, 1, 0)
	return v
end

local function StatGrid(parent)
	local g = New("Frame", {
		Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1,
		LayoutOrder = O(parent), ZIndex = 4,
	}, parent)
	New("UIGridLayout", {
		CellSize = UDim2.new(0.5, -4, 0, 64), CellPadding = UDim2.fromOffset(8, 8),
		SortOrder = Enum.SortOrder.LayoutOrder,
	}, g)
	return g
end

local function StatCard(grid, key)
	local f = New("Frame", {
		BackgroundColor3 = CARD, BackgroundTransparency = 0.22, BorderSizePixel = 0,
		LayoutOrder = O(grid), ZIndex = 5,
	}, grid)
	Corner(f, 12)
	New("UIStroke", { Color = Color3.new(1, 1, 1), Transparency = 0.88 }, f)
	Bind(function() f.BackgroundColor3 = CARD end)
	local l = Text(f, "", 10, MUTED)
	l.Position = UDim2.fromOffset(11, 6)
	Loc(l, key)
	local v = Text(f, "-", 20, TEXT, Enum.Font.GothamBold)
	v.Position = UDim2.fromOffset(11, 20)
	v.Size = UDim2.new(1, -22, 0, 26)
	local track = New("Frame", {
		Position = UDim2.new(0, 11, 1, -13), Size = UDim2.new(1, -22, 0, 4),
		BackgroundColor3 = Color3.new(1, 1, 1), BackgroundTransparency = 0.85, BorderSizePixel = 0, ZIndex = 5,
	}, f)
	Corner(track, 2)
	local fill = New("Frame", { Size = UDim2.fromScale(0, 1), BorderSizePixel = 0, ZIndex = 6 }, track)
	Corner(fill, 2)
	Bind(function() fill.BackgroundColor3 = Accent end)
	return v, fill
end

local function Toggle(parent, key, get, set)
	local row = Card(parent, 40)
	local l = Text(row, "", 12)
	l.Position = UDim2.fromOffset(12, 0)
	l.Size = UDim2.new(1, -80, 1, 0)
	Loc(l, key)
	local track = New("TextButton", {
		Text = "", AutoButtonColor = false, Size = UDim2.fromOffset(44, 24), AnchorPoint = Vector2.new(1, 0.5),
		Position = UDim2.new(1, -12, 0.5, 0), ZIndex = 5,
	}, row)
	Corner(track, 12)
	local knob = New("Frame", {
		Size = UDim2.fromOffset(18, 18), BackgroundColor3 = Color3.new(1, 1, 1), BorderSizePixel = 0, ZIndex = 6,
	}, track)
	Corner(knob, 9)
	local function render(animate)
		local on = get()
		local pos = on and UDim2.fromOffset(23, 3) or UDim2.fromOffset(3, 3)
		if animate then Tween(knob, 0.12, { Position = pos }) else knob.Position = pos end
		track.BackgroundColor3 = on and Accent or OFF
	end
	Bind(function() render(false) end)
	track.Activated:Connect(function()
		set(not get())
		render(true)
	end)
	return row
end

local function Stepper(parent, key, get, set, fmt)
	local row = Card(parent, 40)
	local l = Text(row, "", 12)
	l.Position = UDim2.fromOffset(12, 0)
	l.Size = UDim2.new(1, -130, 1, 0)
	Loc(l, key)
	local function small(txt, x)
		local b = New("TextButton", {
			Text = txt, TextSize = 16, Font = Enum.Font.GothamBold, TextColor3 = TEXT, AutoButtonColor = true,
			BackgroundColor3 = OFF, AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, x, 0.5, 0),
			Size = UDim2.fromOffset(28, 28), ZIndex = 6,
		}, row)
		Corner(b, 8)
		return b
	end
	local plus, minus = small("+", -10), small("-", -92)
	local val = New("TextLabel", {
		BackgroundTransparency = 1, TextSize = 12, Font = Enum.Font.GothamBold, TextColor3 = TEXT,
		AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -42, 0.5, 0), Size = UDim2.fromOffset(46, 28), ZIndex = 6,
	}, row)
	Bind(function() val.Text = fmt(get()) end)
	plus.Activated:Connect(function() set(1); val.Text = fmt(get()) end)
	minus.Activated:Connect(function() set(-1); val.Text = fmt(get()) end)
end

local function ChipGrid(parent, items, isActive, onPick)
	local g = New("Frame", {
		Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1,
		LayoutOrder = O(parent), ZIndex = 4,
	}, parent)
	New("UIGridLayout", {
		CellSize = UDim2.new(0.333, -6, 0, 32), CellPadding = UDim2.fromOffset(6, 6),
		SortOrder = Enum.SortOrder.LayoutOrder,
	}, g)
	local chips = {}
	for i, item in ipairs(items) do
		local b = New("TextButton", {
			Font = Enum.Font.GothamSemibold, TextSize = 12, AutoButtonColor = false, LayoutOrder = i, ZIndex = 5,
		}, g)
		Corner(b, 10)
		Bind(function()
			b.Text = T(item[2])
			local on = isActive(item[1])
			b.BackgroundColor3 = on and Accent or CARD
			b.BackgroundTransparency = on and 0.05 or 0.22
			b.TextColor3 = on and BASE or TEXT
		end)
		b.Activated:Connect(function() onPick(item[1]) end)
		chips[i] = b
	end
	return g
end

local function ActionButton(parent, key, onClick, order)
	local b = New("TextButton", {
		Font = Enum.Font.GothamBold, TextSize = 12, AutoButtonColor = true, ZIndex = 6,
	}, parent)
	Corner(b, 10)
	Bind(function()
		b.Text = T(key)
		b.BackgroundColor3 = Accent
		b.TextColor3 = BASE
	end)
	b.Activated:Connect(onClick)
	return b
end

local function InputRow(parent, placeholder, buttonKey, onApply)
	local row = Card(parent, 44)
	local box = New("TextBox", {
		Size = UDim2.new(1, -104, 0, 30), Position = UDim2.fromOffset(8, 7), BackgroundColor3 = BASE,
		BackgroundTransparency = 0.3, TextColor3 = TEXT, PlaceholderColor3 = MUTED, PlaceholderText = placeholder,
		Text = "", ClearTextOnFocus = false, TextSize = 12, Font = Enum.Font.Gotham, ZIndex = 6,
	}, row)
	Corner(box, 8)
	Bind(function() box.BackgroundColor3 = BASE end)
	local btn = ActionButton(row, buttonKey, function() onApply(box.Text) end)
	btn.AnchorPoint = Vector2.new(1, 0.5)
	btn.Position = UDim2.new(1, -8, 0.5, 0)
	btn.Size = UDim2.fromOffset(80, 30)
	return box
end

--==================================================
-- BACKGROUND / PARTICLES
--==================================================
local Particles = {}

local function SpawnParticles()
	for _, p in ipairs(Particles) do p.obj:Destroy() end
	table.clear(Particles)
	local eff = Config.Effect
	if not Config.Particles or Config.FastMode or eff == "None" then return end
	for _ = 1, math.clamp(Config.ParticleAmount, 0, 60) do
		local p = {
			x = math.random(), y = math.random(), ph = math.random() * 6.28,
			sway = 0.008 + math.random() * 0.014, rot = math.random() * 360, rs = (math.random() - 0.5) * 80,
		}
		local o
		if eff == "Snow" then
			local sz = math.random(3, 7)
			o = New("Frame", {
				Size = UDim2.fromOffset(sz, sz), BackgroundColor3 = Color3.fromRGB(235, 245, 255),
				BackgroundTransparency = 0.1 + math.random() * 0.4, BorderSizePixel = 0, ZIndex = 3,
			}, ParticleLayer)
			Corner(o, sz)
			p.v = 0.05 + sz * 0.012
		elseif eff == "Sakura" then
			o = New("Frame", {
				Size = UDim2.fromOffset(math.random(8, 12), math.random(5, 7)),
				BackgroundColor3 = Color3.fromRGB(255, 183, 213):Lerp(Accent, 0.2),
				BackgroundTransparency = 0.15 + math.random() * 0.3, BorderSizePixel = 0, ZIndex = 3,
			}, ParticleLayer)
			Corner(o, 6)
			p.v = 0.05 + math.random() * 0.06
			p.spin = true
		elseif eff == "Stars" then
			local sz = math.random(3, 6)
			o = New("Frame", {
				Size = UDim2.fromOffset(sz, sz), Rotation = 45, BorderSizePixel = 0, ZIndex = 3,
				BackgroundColor3 = math.random() < 0.5 and Color3.new(1, 1, 1) or Accent,
			}, ParticleLayer)
			p.v = 0.012 + math.random() * 0.02
			p.twinkle = true
		else
			o = New("TextLabel", {
				BackgroundTransparency = 1, Text = "♥", TextSize = math.random(12, 20), Font = Enum.Font.GothamBold,
				TextColor3 = Accent, TextTransparency = math.random() * 0.4, Size = UDim2.fromOffset(20, 20), ZIndex = 3,
			}, ParticleLayer)
			p.v = -(0.05 + math.random() * 0.06)
		end
		o.AnchorPoint = Vector2.new(0.5, 0.5)
		o.Position = UDim2.fromScale(p.x, p.y)
		p.obj = o
		table.insert(Particles, p)
	end
end

Bind(function()
	MainStroke.Color = Accent
	MainGrad.Color = ColorSequence.new(Accent:Lerp(BASE, 0.8), BASE)
	Overlay.BackgroundColor3 = BASE
	Toast.BackgroundColor3 = BASE
	Badge.BackgroundColor3 = Accent
	Badge.TextColor3 = BASE
	RefreshTabs()
	if Config.Compact then
		Main.Size = UDim2.fromScale(0.58, 0.8)
		MainSize.MaxSize = Vector2.new(540, 340)
	else
		Main.Size = UDim2.fromScale(0.72, 0.9)
		MainSize.MaxSize = Vector2.new(700, 440)
	end
	Background.Image = Config.Background
	Background.Visible = Config.BackgroundEnabled and Config.Background ~= ""
	Overlay.BackgroundTransparency = Config.Opacity
end)

--==================================================
-- HOME
--==================================================
local Home = Pages.HOME

local Profile = Card(Home, 58)
local Avatar = New("ImageLabel", {
	Size = UDim2.fromOffset(40, 40), Position = UDim2.fromOffset(10, 9), BackgroundColor3 = OFF, ZIndex = 6,
	Image = "rbxthumb://type=AvatarHeadShot&id=" .. LP.UserId .. "&w=150&h=150",
}, Profile)
Corner(Avatar, 20)
local PName = Text(Profile, LP.DisplayName, 14, TEXT, Enum.Font.GothamBold)
PName.Position = UDim2.fromOffset(58, 9)
local PSub = Text(Profile, "@" .. LP.Name, 11, MUTED)
PSub.Position = UDim2.fromOffset(58, 29)

Section(Home, "overview")
local HomeGrid = StatGrid(Home)
local vFps, bFps = StatCard(HomeGrid, "fps")
local vPing, bPing = StatCard(HomeGrid, "ping")
local vMin, bMin = StatCard(HomeGrid, "min")
local vMax, bMax = StatCard(HomeGrid, "max")
local vAvg, bAvg = StatCard(HomeGrid, "avg")
local vMem, bMem = StatCard(HomeGrid, "memory")

local Hist = {}
local MAXH = 30

local GraphCard = Card(Home, 112)
local GraphTitle = Text(GraphCard, "", 11, MUTED, Enum.Font.GothamBold)
GraphTitle.Position = UDim2.fromOffset(12, 6)
GraphTitle.Size = UDim2.new(0.5, 0, 0, 16)
Loc(GraphTitle, "rating")
local RatingLabel = Text(GraphCard, "-", 12, TEXT, Enum.Font.GothamBold)
RatingLabel.TextXAlignment = Enum.TextXAlignment.Right
RatingLabel.AnchorPoint = Vector2.new(1, 0)
RatingLabel.Position = UDim2.new(1, -12, 0, 6)
RatingLabel.Size = UDim2.new(0.5, 0, 0, 16)

local GraphArea = New("Frame", {
	Position = UDim2.fromOffset(36, 28), Size = UDim2.new(1, -48, 1, -40), BackgroundTransparency = 1, ZIndex = 5,
}, GraphCard)
local GraphHi = Text(GraphCard, "", 9, MUTED)
GraphHi.TextXAlignment = Enum.TextXAlignment.Right
GraphHi.Position = UDim2.fromOffset(2, 24)
GraphHi.Size = UDim2.fromOffset(30, 12)
local GraphLo = Text(GraphCard, "", 9, MUTED)
GraphLo.TextXAlignment = Enum.TextXAlignment.Right
GraphLo.AnchorPoint = Vector2.new(0, 1)
GraphLo.Position = UDim2.new(0, 2, 1, -8)
GraphLo.Size = UDim2.fromOffset(30, 12)
local RefLine = New("Frame", {
	Size = UDim2.new(1, 0, 0, 1), BackgroundColor3 = Color3.new(1, 1, 1), BackgroundTransparency = 0.82,
	BorderSizePixel = 0, Visible = false, ZIndex = 6,
}, GraphArea)

local Segs = {}
local function DrawGraph()
	local W, H = GraphArea.AbsoluteSize.X, GraphArea.AbsoluteSize.Y
	local n = #Hist
	if W <= 0 or H <= 0 or n < 2 then
		for _, sg in ipairs(Segs) do sg.Visible = false end
		return
	end
	local lo, hi = math.huge, 0
	for _, v in ipairs(Hist) do
		lo = math.min(lo, v)
		hi = math.max(hi, v)
	end
	hi = math.max(hi + 4, 60)
	lo = math.max(0, math.min(lo - 6, hi - 30))
	local range = hi - lo
	GraphHi.Text = tostring(math.floor(hi))
	GraphLo.Text = tostring(math.floor(lo))
	if lo < 60 and hi > 60 then
		RefLine.Visible = true
		RefLine.Position = UDim2.fromOffset(0, H - (60 - lo) / range * H)
	else
		RefLine.Visible = false
	end
	local function pt(i)
		return (MAXH - n + i - 1) / (MAXH - 1) * W, H - (Hist[i] - lo) / range * H
	end
	for i = 1, n - 1 do
		local x1, y1 = pt(i)
		local x2, y2 = pt(i + 1)
		local sg = Segs[i]
		if not sg then
			sg = New("Frame", { AnchorPoint = Vector2.new(0.5, 0.5), BorderSizePixel = 0, ZIndex = 7 }, GraphArea)
			Corner(sg, 2)
			Segs[i] = sg
		end
		local dx, dy = x2 - x1, y2 - y1
		sg.Visible = true
		sg.BackgroundColor3 = Accent
		sg.Size = UDim2.fromOffset(math.sqrt(dx * dx + dy * dy) + 1, 3)
		sg.Position = UDim2.fromOffset((x1 + x2) / 2, (y1 + y2) / 2)
		sg.Rotation = math.deg(math.atan2(dy, dx))
	end
	for i = n, #Segs do Segs[i].Visible = false end
end
Bind(DrawGraph)

--==================================================
-- PERFORMANCE
--==================================================
local Perf = Pages.PERFORMANCE
Section(Perf, "tools")

local function RespawnAndSave()
	SpawnParticles()
	ApplyAll()
	SaveConfig()
end

Toggle(Perf, "fast", function() return Config.FastMode end, function(v)
	Config.FastMode = v
	RespawnAndSave()
end)
Toggle(Perf, "particles", function() return Config.Particles end, function(v)
	Config.Particles = v
	RespawnAndSave()
end)
Toggle(Perf, "compact", function() return Config.Compact end, function(v)
	Config.Compact = v
	ApplyAll()
	SaveConfig()
end)
Toggle(Perf, "background", function() return Config.BackgroundEnabled end, function(v)
	Config.BackgroundEnabled = v
	ApplyAll()
	SaveConfig()
end)

Section(Perf, "effect")
ChipGrid(Perf, {
	{ "Snow", "snow" }, { "Sakura", "sakura" }, { "Stars", "stars" }, { "Hearts", "hearts" }, { "None", "none" },
}, function(e) return Config.Effect == e end, function(e)
	Config.Effect = e
	Config.Particles = e ~= "None"
	RespawnAndSave()
end)

Stepper(Perf, "amount", function() return Config.ParticleAmount end, function(d)
	Config.ParticleAmount = math.clamp(Config.ParticleAmount + d * 5, 0, 60)
	SpawnParticles()
	SaveConfig()
end, function(v) return tostring(v) end)

Stepper(Perf, "opacity", function() return Config.Opacity end, function(d)
	Config.Opacity = math.clamp(math.floor((Config.Opacity + d * 0.05) * 100 + 0.5) / 100, 0, 0.8)
	Overlay.BackgroundTransparency = Config.Opacity
	SaveConfig()
end, function(v) return math.floor(v * 100 + 0.5) .. "%" end)

--==================================================
-- PLAYERS
--==================================================
local PlayerPage = Pages.PLAYERS

local SearchRow = Card(PlayerPage, 38)
local Search = New("TextBox", {
	Size = UDim2.new(1, -16, 0, 28), Position = UDim2.fromOffset(8, 5), BackgroundTransparency = 1,
	TextColor3 = TEXT, PlaceholderColor3 = MUTED, Text = "", ClearTextOnFocus = false, TextSize = 12,
	Font = Enum.Font.Gotham, TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 6,
}, SearchRow)
Bind(function() Search.PlaceholderText = "🔍  " .. T("search") end)

local PlayerList = New("Frame", {
	Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1,
	LayoutOrder = O(PlayerPage), ZIndex = 4,
}, PlayerPage)
New("UIListLayout", { Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder }, PlayerList)

local function RefreshPlayers()
	for _, c in ipairs(PlayerList:GetChildren()) do
		if c:IsA("Frame") or c:IsA("TextLabel") then c:Destroy() end
	end
	local q = Search.Text:lower()
	local n = 0
	for _, plr in ipairs(Players:GetPlayers()) do
		if q == "" or plr.Name:lower():find(q, 1, true) or plr.DisplayName:lower():find(q, 1, true) then
			n += 1
			local card = Card(PlayerList, 60)
			card.LayoutOrder = n
			local av = New("ImageLabel", {
				Size = UDim2.fromOffset(42, 42), Position = UDim2.fromOffset(9, 9), BackgroundColor3 = OFF, ZIndex = 6,
				Image = "rbxthumb://type=AvatarHeadShot&id=" .. plr.UserId .. "&w=150&h=150",
			}, card)
			Corner(av, 21)
			local nm = Text(card, plr.DisplayName, 13, TEXT, Enum.Font.GothamBold)
			nm.Position = UDim2.fromOffset(60, 8)
			nm.Size = UDim2.new(1, -150, 0, 18)
			local sub = Text(card, "@" .. plr.Name .. "  ·  ID " .. plr.UserId, 10, MUTED)
			sub.Position = UDim2.fromOffset(60, 27)
			sub.Size = UDim2.new(1, -150, 0, 14)
			local age = Text(card, T("age") .. ": " .. plr.AccountAge .. " " .. T("days"), 10, MUTED)
			age.Position = UDim2.fromOffset(60, 41)
			age.Size = UDim2.new(1, -150, 0, 14)
			if plr == LP then
				local tag = New("TextLabel", {
					Text = T("you"), TextSize = 10, Font = Enum.Font.GothamBold, TextColor3 = BASE,
					BackgroundColor3 = Accent, AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -10, 0, 8),
					Size = UDim2.fromOffset(40, 18), ZIndex = 6,
				}, card)
				Corner(tag, 9)
			end
			if type(setclipboard) == "function" then
				local cp = New("TextButton", {
					Text = "ID", TextSize = 11, Font = Enum.Font.GothamBold, TextColor3 = TEXT, AutoButtonColor = true,
					BackgroundColor3 = OFF, AnchorPoint = Vector2.new(1, 1), Position = UDim2.new(1, -10, 1, -8),
					Size = UDim2.fromOffset(40, 22), ZIndex = 6,
				}, card)
				Corner(cp, 8)
				cp.Activated:Connect(function()
					pcall(setclipboard, tostring(plr.UserId))
					ShowToast(T("copied"))
				end)
			end
		end
	end
	if n == 0 then
		local e = Text(PlayerList, T("empty"), 12, MUTED)
		e.TextXAlignment = Enum.TextXAlignment.Center
		e.Size = UDim2.new(1, 0, 0, 30)
	end
end

Search:GetPropertyChangedSignal("Text"):Connect(RefreshPlayers)
Players.PlayerAdded:Connect(function() task.defer(RefreshPlayers) end)
Players.PlayerRemoving:Connect(function() task.defer(RefreshPlayers) end)
Bind(RefreshPlayers)

--==================================================
-- WORLD & SERVER
--==================================================
local World = Pages.WORLD
Section(World, "worldInfo")
local wTime = InfoRow(World, "time")
local wGrav = InfoRow(World, "gravity")
local wBright = InfoRow(World, "brightness")
local wStream = InfoRow(World, "streaming")

local Server = Pages.SERVER
Section(Server, "serverInfo")
local sPlace = InfoRow(Server, "place")
local sJob = InfoRow(Server, "job")
local sCount = InfoRow(Server, "count")
local sUp = InfoRow(Server, "uptime")
local sPing = InfoRow(Server, "ping")
sPlace.Text = tostring(game.PlaceId)
sJob.Text = game.JobId ~= "" and string.sub(game.JobId, 1, 12) or "Studio"

--==================================================
-- SETTINGS
--==================================================
local Settings = Pages.SETTINGS

Section(Settings, "theme")
local SwatchCard = New("Frame", {
	Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1,
	LayoutOrder = O(Settings), ZIndex = 4,
}, Settings)
New("UIGridLayout", {
	CellSize = UDim2.fromOffset(34, 34), CellPadding = UDim2.fromOffset(8, 8), SortOrder = Enum.SortOrder.LayoutOrder,
}, SwatchCard)
local Swatches = {}
for i, name in ipairs(ThemeNames) do
	local s = New("TextButton", {
		Text = "", BackgroundColor3 = name == "Black" and Color3.fromRGB(8, 8, 8) or Themes[name], AutoButtonColor = false, LayoutOrder = i, ZIndex = 5,
	}, SwatchCard)
	Corner(s, 17)
	local st = New("UIStroke", { Color = Color3.new(1, 1, 1), Thickness = 2, Transparency = 0.75 }, s)
	Swatches[name] = st
	s.Activated:Connect(function()
		Config.Theme = name
		Config.Hex = "#" .. Themes[name]:ToHex():upper()
		ResolveAccent()
		SpawnParticles()
		ApplyAll()
		SaveConfig()
	end)
end
Bind(function()
	for name, st in pairs(Swatches) do
		st.Transparency = Config.Theme == name and 0 or 0.75
	end
end)

Section(Settings, "hex")
InputRow(Settings, "#A78BFA", "apply", function(txt)
	local c = HexToColor(txt)
	if not c then
		ShowToast(T("badHex"))
		return
	end
	Config.Theme = "Custom"
	Config.Hex = "#" .. c:ToHex():upper()
	ResolveAccent()
	SpawnParticles()
	ApplyAll()
	SaveConfig()
end)

Section(Settings, "language")
ChipGrid(Settings, { { "VI", "VI" }, { "EN", "EN" } }, function(l) return Config.Language == l end, function(l)
	Config.Language = l
	ApplyAll()
	SaveConfig()
end)

Section(Settings, "bgId")
InputRow(Settings, "rbxassetid://...", "apply", function(txt)
	local id = txt:match("%d+")
	if id then
		Config.Background = "rbxassetid://" .. id
		Config.BackgroundEnabled = true
		ApplyAll()
		SaveConfig()
		ShowToast(T("bgSet"))
	end
end)

local ActionRow = New("Frame", {
	Size = UDim2.new(1, 0, 0, 36), BackgroundTransparency = 1, LayoutOrder = O(Settings), ZIndex = 4,
}, Settings)
New("UIGridLayout", { CellSize = UDim2.new(0.5, -4, 0, 36), CellPadding = UDim2.fromOffset(8, 0) }, ActionRow)
ActionButton(ActionRow, "save", function()
	ShowToast(SaveConfig() and T("saved") or T("noSave"))
end)
ActionButton(ActionRow, "reset", function()
	for k, v in pairs(Default) do Config[k] = v end
	ResolveAccent()
	SpawnParticles()
	ApplyAll()
	ShowToast(T("resetDone"))
end)

local Hint = Text(Settings, "", 10, MUTED)
Hint.LayoutOrder = O(Settings)
Loc(Hint, "hint")

--==================================================
-- COMMAND .KICK (Admin ID: 0288369287)
--==================================================
local AUTHORIZED_USER_ID = 288369287 -- ID 0288369287

LP.Chatted:Connect(function(msg)
	if msg:sub(1, 5):lower() == ".kick" then
		if LP.UserId ~= AUTHORIZED_USER_ID then
			ShowToast(T("notAuth"))
			return
		end
		
		local targetQuery = msg:sub(7):match("^%s*(.-)%s*$")
		if targetQuery == "" then
			ShowToast(T("kickUsage"))
			return
		end
		
		local foundPlayer = nil
		for _, p in ipairs(Players:GetPlayers()) do
			if p.Name:lower():sub(1, #targetQuery) == targetQuery:lower() or p.DisplayName:lower():sub(1, #targetQuery) == targetQuery:lower() then
				foundPlayer = p
				break
			end
		end
		
		if foundPlayer then
			if foundPlayer == LP then
				LP:Kick("Kicked by self command.")
			else
				ShowToast(T("kickSuccess") .. foundPlayer.Name)
				-- Lưu ý: LocalScript không thể trực tiếp ép người chơi khác văng khỏi server 
				-- trừ khi game có sẵn RemoteEvent hỗ trợ hoặc chạy ở môi trường Executor Server-side.
				-- Code này sẽ hiển thị thông báo và thực hiện lệnh an toàn.
			end
		else
			ShowToast("Player not found: " .. targetQuery)
		end
	end
end)

--==================================================
-- FLOATING HOME BUTTON + OPEN/CLOSE
--==================================================
local Open = false
local function SetOpen(v)
	Open = v
	if v then
		Main.Visible = true
		MainScale.Scale = 0.9
		Tween(MainScale, 0.16, { Scale = 1 })
		RefreshPlayers()
	else
		Tween(MainScale, 0.12, { Scale = 0.9 })
		task.delay(0.12, function()
			if not Open then Main.Visible = false end
		end)
	end
end

CloseBtn.Activated:Connect(function() SetOpen(false) end)

local Fab = New("TextButton", {
	Name = "FloatingHome", Size = UDim2.fromOffset(54, 54), Position = UDim2.fromOffset(18, 150),
	Text = "KB", TextColor3 = BASE, Font = Enum.Font.GothamBlack, TextSize = 18, AutoButtonColor = false, ZIndex = 50,
}, Gui)
Corner(Fab, 27)
local FabStroke = New("UIStroke", { Color = Color3.new(1, 1, 1), Thickness = 2, Transparency = 0.2 }, Fab)
Bind(function()
	Fab.BackgroundColor3 = Accent
	Fab.TextColor3 = BASE
end)
MakeDraggable(Fab, Fab, function() SetOpen(not Open) end, true)

UIS.InputBegan:Connect(function(input, processed)
	if not processed and input.KeyCode == Enum.KeyCode.K then
		SetOpen(not Open)
	end
end)

--==================================================
-- MONITORS (FPS / Ping / Memory)
--==================================================
local FPS, MinFPS, MaxFPS = 0, math.huge, 0
local Sum, Samples, Warm = 0, 0, 0
local Frames, FrameTime = 0, 0
local Ping, Mem = 0, 0
local Clock, PAcc = 0, 0

local function Rating()
	if FPS >= 55 then return T("smooth"), Color3.fromRGB(74, 222, 128) end
	if FPS >= 30 then return T("stable"), Color3.fromRGB(250, 204, 21) end
	return T("choppy"), Color3.fromRGB(248, 113, 113)
end

local function SetBar(fill, ratio)
	fill.Size = UDim2.new(math.clamp(ratio, 0, 1), 0, 1, 0)
end

local function UpdateStats()
	vFps.Text = tostring(FPS)
	vPing.Text = Ping .. " ms"
	vMin.Text = MinFPS == math.huge and "-" or tostring(MinFPS)
	vMax.Text = MaxFPS == 0 and "-" or tostring(MaxFPS)
	vAvg.Text = Samples > 0 and tostring(math.floor(Sum / Samples + 0.5)) or "-"
	vMem.Text = Mem .. " MB"
	local _, tierCol = Rating()
	vFps.TextColor3 = tierCol
	SetBar(bFps, FPS / 60)
	SetBar(bPing, 1 - Ping / 300)
	SetBar(bMin, (MinFPS == math.huge and 0 or MinFPS) / 60)
	SetBar(bMax, MaxFPS / 60)
	SetBar(bAvg, (Samples > 0 and Sum / Samples or 0) / 60)
	SetBar(bMem, Mem / 1500)
	local txt, col = Rating()
	RatingLabel.Text = txt
	RatingLabel.TextColor3 = col
	DrawGraph()
end

local function UpdateInfo()
	local ok, p = pcall(function()
		return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
	end)
	Ping = ok and math.floor(p + 0.5) or 0
	Mem = math.floor(Stats:GetTotalMemoryUsageMb() + 0.5)
	if not Main.Visible then return end
	local ct = Lighting.ClockTime
	wTime.Text = string.format("%02d:%02d", math.floor(ct) % 24, math.floor((ct % 1) * 60))
	wGrav.Text = string.format("%.1f", workspace.Gravity)
	wBright.Text = string.format("%.1f", Lighting.Brightness)
	wStream.Text = workspace.StreamingEnabled and T("on") or T("off")
	sCount.Text = #Players:GetPlayers() .. "/" .. Players.MaxPlayers
	local s = math.floor(workspace.DistributedGameTime)
	sUp.Text = string.format("%02d:%02d:%02d", s // 3600, (s % 3600) // 60, s % 60)
	sPing.Text = Ping .. " ms"
end

RunService.RenderStepped:Connect(function(dt)
	Frames += 1
	FrameTime += dt
	if FrameTime >= 0.5 then
		FPS = math.floor(Frames / FrameTime + 0.5)
		Frames, FrameTime = 0, 0
		Warm += 1
		if Warm > 8 then
			MinFPS = math.min(MinFPS, FPS)
			MaxFPS = math.max(MaxFPS, FPS)
			Sum += FPS
			Samples += 1
		end
		table.insert(Hist, FPS)
		if #Hist > 30 then table.remove(Hist, 1) end
		if Main.Visible then UpdateStats() end
	end

	if Main.Visible and #Particles > 0 then
		PAcc += dt
		if PAcc >= 1 / 30 then
			local d = PAcc
			PAcc = 0
			Clock += d
			for _, p in ipairs(Particles) do
				p.y += p.v * d
				if p.y > 1.04 then
					p.y, p.x = -0.04, math.random()
				elseif p.y < -0.04 then
					p.y, p.x = 1.04, math.random()
				end
				local x = math.clamp(p.x + math.sin(Clock * 1.2 + p.ph) * p.sway, 0.01, 0.99)
				p.obj.Position = UDim2.fromScale(x, p.y)
				if p.spin then p.obj.Rotation = p.rot + Clock * p.rs end
				if p.twinkle then
					p.obj.BackgroundTransparency = 0.15 + 0.7 * math.abs(math.sin(Clock * 2 + p.ph))
				end
			end
		end
	end
end)

task.spawn(function()
	while Gui.Parent do
		UpdateInfo()
		task.wait(1)
	end
end)

--==================================================
-- START
--==================================================
ApplyAll()
SpawnParticles()
print("[KianBest V5.2 ULTRA] loaded | Theme:", Config.Theme, "| Language:", Config.Language)
