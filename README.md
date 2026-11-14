--==================================================
-- KIANBEST V5.3 ULTRA | CLIENT EDITION (1 file, chạy hoàn toàn ở CLIENT)
-- Đặt vào LocalScript (StarterPlayerScripts) hoặc chạy bằng executor.
-- Không cần Script server, không cần RemoteEvent.
-- Lệnh gõ trong chat HOẶC trong ô lệnh ở tab Trang chủ:
--   .tp <tên/id>   .view <tên/id>   .unview   .hide / .show <tên/id>
--   .speed / .jump / .gravity / .fov / .time <số>
--   .esp  .noclip  .ij  .rejoin  .respawn  .leave  .cmds
-- LƯU Ý: client KHÔNG thể kick người khác (Roblox chặn). .kick sẽ báo rõ,
--        dùng .hide để ẩn người đó phía bạn.
--==================================================

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local TextChatService = game:GetService("TextChatService")

local LP = Players.LocalPlayer
local PG = LP:WaitForChild("PlayerGui")
local StartClock = os.clock()

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
	HUD = false,
	FullBright = false,
	NoFog = false,
	Shadows = true,
	FOV = 70,
	UIScale = 1,
	Font = "Builder",
	FPSCap = 0,
	-- mới (V5.3)
	Notify = true,
	Commands = true,
	AntiAFK = false,
	LowGfx = false,
	NoFx = false,
	Speed = 0, -- 0 = không đụng tới
	Jump = 0,
	Gravity = 0,
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
		stars = "Sao", hearts = "Tim", none = "Tắt", amount = "Số lượng hạt",
		opacity = "Độ tối lớp phủ",
		search = "Tìm người chơi...", you = "BẠN", age = "Tuổi tài khoản", days = "ngày", copyId = "Sao chép ID",
		copied = "Đã sao chép", empty = "Không tìm thấy người chơi",
		worldInfo = "Thông tin thế giới", time = "Giờ trong game", gravity = "Trọng lực",
		brightness = "Độ sáng", streaming = "Streaming", on = "Bật", off = "Tắt",
		serverInfo = "Thông tin máy chủ", place = "Place ID", job = "Job ID", count = "Số người chơi",
		uptime = "Thời gian chạy", theme = "Màu giao diện", hex = "Màu tùy chỉnh (HEX)", apply = "Áp dụng",
		language = "Ngôn ngữ", bgId = "Hình nền (Image ID)", save = "Lưu cấu hình", reset = "Khôi phục",
		saved = "Đã lưu cấu hình", noSave = "Môi trường không hỗ trợ lưu file", resetDone = "Đã khôi phục mặc định",
		badHex = "Mã HEX không hợp lệ", bgSet = "Đã bật hình nền", hint = "Kéo thanh tiêu đề để di chuyển",
		espOn = "ESP đã hoạt động", espOff = "ESP đã tắt",
		noServer = "Chưa có Script server (ServerScriptService), lệnh và ESP chưa dùng được",
		nOk = "Thành công", nErr = "Lỗi", nWarn = "Cảnh báo", nInfo = "Thông báo",
		tpOk = "Đã dịch chuyển tới", kickOk = "Đã kick", notFound = "Không tìm thấy người chơi",
		usage = "Cú pháp: .tp <id/tên>  (gõ .cmds để xem tất cả lệnh)", noChar = "Nhân vật chưa sẵn sàng",
		kickSelf = "Không thể tự kick chính mình",
		fpsCap = "Giới hạn FPS", capOff = "Mặc định", capSet = "Đã đặt giới hạn FPS:",
		capNo = "Môi trường này không hỗ trợ đổi giới hạn FPS",
		font = "Kiểu chữ", fontBuilder = "Builder Sans", fontGotham = "Gotham",
		active = "Đang hoạt động", pingChart = "Biểu đồ Ping",
		esp = "ESP người chơi", noAdmin = "Không có quyền",
		worldTools = "Công cụ thế giới", fullbright = "Sáng toàn map", nofog = "Tắt sương mù",
		shadows = "Bóng đổ", fov = "Góc nhìn (FOV)", clock = "Chỉnh giờ trong game",
		ver = "Phiên bản place", sent = "Dữ liệu gửi", recv = "Dữ liệu nhận", localTime = "Giờ thực",
		copyJob = "Sao chép Job ID", rejoin = "Vào lại server", newServer = "Server khác",
		hud = "FPS/Ping trên màn hình", uiscale = "Cỡ giao diện", display = "Hiển thị",
		-- V5.3
		character = "Nhân vật", hp = "Máu", speed = "Tốc độ", pos = "Vị trí", session = "Thời gian chơi",
		run = "Chạy", cmdHint = ".tp <tên/id>   (.cmds = danh sách lệnh)",
		respawn = "Hồi sinh", gc = "Dọn bộ nhớ", gcDone = "Đã dọn bộ nhớ",
		antiafk = "Chống AFK", lowgfx = "Đồ họa thấp", nofx = "Tắt hiệu ứng hình ảnh",
		noclip = "Xuyên tường (Noclip)", infjump = "Nhảy vô hạn", walkspeed = "Tốc độ chạy",
		jumppower = "Lực nhảy", gravityTool = "Chỉnh trọng lực", auto = "Auto",
		tpBtn = "TP", viewBtn = "Xem", unview = "Thoát chế độ xem", showAll = "Hiện tất cả",
		gname = "Tên game", priv = "Server riêng", physfps = "FPS vật lý server",
		copyPlace = "Sao chép Place ID", copyLink = "Sao chép link game",
		notify = "Thông báo", cmdOn = "Lệnh chat (.tp, .view...)", badId = "Image ID không hợp lệ",
		kickNo = "Client không thể kick người khác (Roblox chặn, cần quyền server). Dùng .hide để ẩn họ phía bạn.",
		hidden = "Đã ẩn", shown = "Đã hiện", viewing = "Đang xem", unviewing = "Đã thoát chế độ xem",
		cmdsList = "Lệnh: .tp .view .unview .hide .show .speed .jump .gravity .fov .time .esp .noclip .ij .rejoin .respawn .leave",
		unknownCmd = "Lệnh không tồn tại (gõ .cmds)", numNeed = "Cần nhập một con số",
		setTo = "Đã đặt", noclipOn = "Noclip: bật", noclipOff = "Noclip: tắt",
		ijOn = "Nhảy vô hạn: bật", ijOff = "Nhảy vô hạn: tắt", leaving = "Rời game",
	},
	EN = {
		HOME = "Home", PERFORMANCE = "Performance", PLAYERS = "Players",
		WORLD = "World", SERVER = "Server", SETTINGS = "Settings",
		overview = "Overview", fps = "FPS", ping = "Ping", min = "Min FPS", max = "Max FPS",
		avg = "Average FPS", memory = "Memory", rating = "Rating", smooth = "Very smooth",
		stable = "Stable", choppy = "A bit choppy", tools = "Performance tools",
		fast = "Boost mode", particles = "Falling effects", compact = "Compact layout",
		background = "Background", effect = "Effect style", snow = "Snow", sakura = "Sakura",
		stars = "Stars", hearts = "Hearts", none = "Off", amount = "Particle count",
		opacity = "Overlay darkness",
		search = "Search players...", you = "YOU", age = "Account age", days = "days", copyId = "Copy ID",
		copied = "Copied", empty = "No players found",
		worldInfo = "World info", time = "In-game time", gravity = "Gravity",
		brightness = "Brightness", streaming = "Streaming", on = "On", off = "Off",
		serverInfo = "Server info", place = "Place ID", job = "Job ID", count = "Players",
		uptime = "Uptime", theme = "Theme color", hex = "Custom color (HEX)", apply = "Apply",
		language = "Language", bgId = "Background (Image ID)", save = "Save config", reset = "Reset",
		saved = "Config saved", noSave = "File saving is not supported here", resetDone = "Defaults restored",
		badHex = "Invalid HEX code", bgSet = "Background set", hint = "Drag the title bar to move",
		espOn = "ESP is now active", espOff = "ESP turned off",
		noServer = "Server script missing (ServerScriptService), commands and ESP unavailable",
		nOk = "Success", nErr = "Error", nWarn = "Warning", nInfo = "Info",
		tpOk = "Teleported to", kickOk = "Kicked", notFound = "Player not found",
		usage = "Usage: .tp <id/name>  (type .cmds for all commands)", noChar = "Character not ready",
		kickSelf = "You cannot kick yourself",
		fpsCap = "FPS cap", capOff = "Default", capSet = "FPS cap set to:",
		capNo = "FPS cap is not supported in this environment",
		font = "Font", fontBuilder = "Builder Sans", fontGotham = "Gotham",
		active = "Active", pingChart = "Ping chart",
		esp = "Player ESP", noAdmin = "No permission",
		worldTools = "World tools", fullbright = "Fullbright", nofog = "Remove fog",
		shadows = "Shadows", fov = "Field of view", clock = "Set in-game time",
		ver = "Place version", sent = "Data sent", recv = "Data received", localTime = "Local time",
		copyJob = "Copy Job ID", rejoin = "Rejoin server", newServer = "New server",
		hud = "On-screen FPS/Ping", uiscale = "UI scale", display = "Display",
		-- V5.3
		character = "Character", hp = "Health", speed = "Speed", pos = "Position", session = "Session time",
		run = "Run", cmdHint = ".tp <name/id>   (.cmds = command list)",
		respawn = "Respawn", gc = "Clean memory", gcDone = "Memory cleaned",
		antiafk = "Anti-AFK", lowgfx = "Low graphics", nofx = "Disable visual effects",
		noclip = "Noclip", infjump = "Infinite jump", walkspeed = "Walk speed",
		jumppower = "Jump power", gravityTool = "Set gravity", auto = "Auto",
		tpBtn = "TP", viewBtn = "View", unview = "Stop viewing", showAll = "Show all",
		gname = "Game name", priv = "Private server", physfps = "Server physics FPS",
		copyPlace = "Copy Place ID", copyLink = "Copy game link",
		notify = "Notifications", cmdOn = "Chat commands (.tp, .view...)", badId = "Invalid image ID",
		kickNo = "The client cannot kick other players (Roblox blocks it, needs server rights). Use .hide to hide them on your side.",
		hidden = "Hidden", shown = "Shown", viewing = "Viewing", unviewing = "Stopped viewing",
		cmdsList = "Commands: .tp .view .unview .hide .show .speed .jump .gravity .fov .time .esp .noclip .ij .rejoin .respawn .leave",
		unknownCmd = "Unknown command (type .cmds)", numNeed = "A number is required",
		setTo = "Set to", noclipOn = "Noclip: on", noclipOff = "Noclip: off",
		ijOn = "Infinite jump: on", ijOff = "Infinite jump: off", leaving = "Leaving game",
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

-- Hệ thống font: đổi kiểu chữ toàn bộ giao diện
local Families = {
	Builder = { Enum.Font.BuilderSans, Enum.Font.BuilderSansMedium, Enum.Font.BuilderSansMedium, Enum.Font.BuilderSansBold, Enum.Font.BuilderSansExtraBold },
	Gotham = { Enum.Font.Gotham, Enum.Font.GothamMedium, Enum.Font.GothamSemibold, Enum.Font.GothamBold, Enum.Font.GothamBlack },
}
local function SetFont(d)
	if d:IsA("TextLabel") or d:IsA("TextButton") or d:IsA("TextBox") then
		local w = d:GetAttribute("KBW")
		if not w then
			w = table.find(Families.Gotham, d.Font) or 2
			d:SetAttribute("KBW", w)
		end
		d.Font = (Families[Config.Font] or Families.Builder)[w]
	end
end
Gui.DescendantAdded:Connect(SetFont)
local function ApplyFonts()
	for _, d in ipairs(Gui:GetDescendants()) do SetFont(d) end
end

local TEXT = Color3.fromRGB(246, 240, 255)
local MUTED = Color3.fromRGB(176, 162, 208)
local OFF = Color3.fromRGB(80, 70, 100)
local GREEN = Color3.fromRGB(74, 222, 128)

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

-- Act: gom các hành động dùng chung (tránh vượt giới hạn 200 biến local của Luau)
local Act = { Hidden = {}, fx = {}, noclip = false, ij = false, origGrav = workspace.Gravity }
local RunCommand -- khai báo trước, gán ở phần LỆNH

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

-- noBind = true cho card tạo lại liên tục (danh sách người chơi) để Binds không phình to
local function Card(parent, h, noBind)
	local f = New("Frame", {
		Size = UDim2.new(1, 0, 0, h), BackgroundColor3 = CARD, BackgroundTransparency = 0.22,
		BorderSizePixel = 0, LayoutOrder = O(parent), ZIndex = 5,
	}, parent)
	Corner(f, 12)
	New("UIStroke", { Color = Color3.new(1, 1, 1), Transparency = 0.88, Thickness = 1 }, f)
	if not noBind then
		Bind(function() f.BackgroundColor3 = CARD end)
	end
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
local MainScale = New("UIScale", { Scale = Config.UIScale }, Main)
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
	Text = "V5.3 ULTRA", TextSize = 10, Font = Enum.Font.GothamBold, TextColor3 = BASE,
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

-- Thông báo góc dưới bên phải (có trạng thái rõ ràng)
local NOTE_KIND = {
	ok = { Color3.fromRGB(74, 222, 128), "nOk" },
	err = { Color3.fromRGB(248, 113, 113), "nErr" },
	warn = { Color3.fromRGB(250, 204, 21), "nWarn" },
	info = { Color3.fromRGB(129, 140, 248), "nInfo" },
}
local NotifyHolder = New("Frame", {
	AnchorPoint = Vector2.new(1, 1), Position = UDim2.new(1, -14, 1, UIS.TouchEnabled and -150 or -16),
	Size = UDim2.fromOffset(270, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, ZIndex = 90,
}, Gui)
New("UIListLayout", {
	Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder,
	VerticalAlignment = Enum.VerticalAlignment.Bottom, HorizontalAlignment = Enum.HorizontalAlignment.Right,
}, NotifyHolder)

local Notes, noteId = {}, 0
local function ShowToast(msg, kind)
	if Config.Notify == false and kind ~= "err" then return end
	local k = NOTE_KIND[kind or "info"] or NOTE_KIND.info
	local col = k[1]
	noteId += 1
	local card = New("CanvasGroup", {
		Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundColor3 = BASE,
		BackgroundTransparency = 0.05, BorderSizePixel = 0, GroupTransparency = 1, LayoutOrder = noteId, ZIndex = 91,
	}, NotifyHolder)
	Corner(card, 12)
	New("UIStroke", { Color = col, Transparency = 0.35, Thickness = 1.5 }, card)
	New("UIPadding", { PaddingTop = UDim.new(0, 8), PaddingBottom = UDim.new(0, 9), PaddingLeft = UDim.new(0, 12), PaddingRight = UDim.new(0, 12) }, card)
	New("UIListLayout", { Padding = UDim.new(0, 3), SortOrder = Enum.SortOrder.LayoutOrder }, card)
	local head = New("Frame", { Size = UDim2.new(1, 0, 0, 16), BackgroundTransparency = 1, LayoutOrder = 1, ZIndex = 92 }, card)
	local dot = New("Frame", { Position = UDim2.fromOffset(0, 4), Size = UDim2.fromOffset(8, 8), BackgroundColor3 = col, BorderSizePixel = 0, ZIndex = 93 }, head)
	New("UICorner", { CornerRadius = UDim.new(0.5, 0) }, dot)
	local hl = New("TextLabel", {
		BackgroundTransparency = 1, Position = UDim2.fromOffset(14, 0), Size = UDim2.new(1, -14, 1, 0),
		Text = T(k[2]), TextSize = 11, Font = Enum.Font.GothamBold, TextColor3 = col,
		TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 93,
	}, head)
	New("TextLabel", {
		BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y,
		Text = tostring(msg), TextSize = 12, Font = Enum.Font.GothamMedium, TextColor3 = TEXT,
		TextWrapped = true, TextXAlignment = Enum.TextXAlignment.Left, LayoutOrder = 2, ZIndex = 93,
	}, card)
	table.insert(Notes, card)
	if #Notes > 4 then
		local oldest = table.remove(Notes, 1)
		oldest:Destroy()
	end
	Tween(card, 0.18, { GroupTransparency = 0 })
	task.delay(3.2, function()
		if card.Parent then
			Tween(card, 0.25, { GroupTransparency = 1 })
			task.delay(0.3, function()
				local i = table.find(Notes, card)
				if i then table.remove(Notes, i) end
				card:Destroy()
			end)
		end
	end)
end

--==================================================
-- HÀNH ĐỘNG CLIENT (dùng chung cho nút + lệnh chat)
--==================================================
do
	function Act.find(q)
		q = tostring(q or "")
		if q == "" then return nil end
		local id = tonumber(q)
		if id then return Players:GetPlayerByUserId(id) end
		q = q:lower()
		for _, p in ipairs(Players:GetPlayers()) do
			if p.Name:lower():sub(1, #q) == q or p.DisplayName:lower():sub(1, #q) == q then
				return p
			end
		end
		for _, p in ipairs(Players:GetPlayers()) do
			if p.Name:lower():find(q, 1, true) or p.DisplayName:lower():find(q, 1, true) then
				return p
			end
		end
	end

	function Act.hum(plr)
		local c = (plr or LP).Character
		return c and c:FindFirstChildOfClass("Humanoid")
	end

	function Act.tp(p)
		local mine = LP.Character
		local theirs = p.Character and p.Character:FindFirstChild("HumanoidRootPart")
		if not (mine and theirs) then
			ShowToast(T("noChar"), "err")
			return
		end
		mine:PivotTo(theirs.CFrame * CFrame.new(0, 0, 4))
		ShowToast(T("tpOk") .. " " .. p.Name, "ok")
	end

	function Act.view(p)
		local h = Act.hum(p)
		local cam = workspace.CurrentCamera
		if not (h and cam) then
			ShowToast(T("noChar"), "err")
			return
		end
		cam.CameraSubject = h
		ShowToast(T("viewing") .. " " .. p.Name, "info")
	end

	function Act.unview()
		local h, cam = Act.hum(LP), workspace.CurrentCamera
		if h and cam then cam.CameraSubject = h end
		ShowToast(T("unviewing"), "info")
	end

	function Act.applyHide(p)
		local on = Act.Hidden[p] and true or false
		local ch = p.Character
		if not ch then return end
		for _, d in ipairs(ch:GetDescendants()) do
			if d:IsA("BasePart") then d.LocalTransparencyModifier = on and 1 or 0 end
		end
		local h = ch:FindFirstChildOfClass("Humanoid")
		if h then
			h.DisplayDistanceType = on and Enum.HumanoidDisplayDistanceType.None or Enum.HumanoidDisplayDistanceType.Viewer
		end
	end

	function Act.hide(p, on)
		Act.Hidden[p] = on or nil
		Act.applyHide(p)
		ShowToast((on and T("hidden") or T("shown")) .. " " .. p.Name, "ok")
	end

	function Act.showAll()
		for p in pairs(Act.Hidden) do
			Act.Hidden[p] = nil
			if p.Parent then Act.applyHide(p) end
		end
		ShowToast(T("shown"), "ok")
	end

	-- tốc độ / nhảy (restore = trả về giá trị mặc định khi tắt)
	function Act.applyChar(restore)
		local h = Act.hum(LP)
		if not h then return end
		if Config.Speed > 0 then h.WalkSpeed = Config.Speed elseif restore then h.WalkSpeed = 16 end
		if Config.Jump > 0 then
			h.UseJumpPower = true
			h.JumpPower = Config.Jump
		elseif restore then
			h.JumpPower = 50
		end
	end

	function Act.applyGrav()
		workspace.Gravity = Config.Gravity > 0 and Config.Gravity or Act.origGrav
	end

	-- đồ họa thấp / tắt post-effect (khôi phục đúng giá trị gốc khi tắt)
	function Act.applyPerf()
		pcall(function()
			local r = settings().Rendering
			if Config.LowGfx then
				if not Act.q then Act.q = r.QualityLevel end
				r.QualityLevel = Enum.QualityLevel.Level01
			elseif Act.q then
				r.QualityLevel = Act.q
				Act.q = nil
			end
		end)
		for _, d in ipairs(Lighting:GetDescendants()) do
			if d:IsA("PostEffect") then
				if Config.NoFx then
					if Act.fx[d] == nil then Act.fx[d] = d.Enabled end
					d.Enabled = false
				elseif Act.fx[d] ~= nil then
					d.Enabled = Act.fx[d]
					Act.fx[d] = nil
				end
			end
		end
	end

	-- noclip + nhảy vô hạn + chống AFK
	RunService.Stepped:Connect(function()
		if Act.noclip and LP.Character then
			for _, p in ipairs(LP.Character:GetDescendants()) do
				if p:IsA("BasePart") and p.CanCollide then p.CanCollide = false end
			end
		end
	end)
	UIS.JumpRequest:Connect(function()
		if Act.ij then
			local h = Act.hum(LP)
			if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
		end
	end)
	pcall(function()
		LP.Idled:Connect(function()
			if Config.AntiAFK then
				local vu = game:GetService("VirtualUser")
				vu:CaptureController()
				vu:ClickButton2(Vector2.new())
			end
		end)
	end)
	LP.CharacterAdded:Connect(function()
		task.wait(0.5)
		Act.applyChar(false)
		if Config.NoFx or Config.LowGfx then Act.applyPerf() end
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
		Font = Enum.Font.GothamBold, TextSize = 12, AutoButtonColor = true, ZIndex = 6, LayoutOrder = order or 0,
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

-- Lưới nút 2 cột: defs = { {textKey, fn}, ... }
local function ButtonRow(parent, defs)
	local rows = math.ceil(#defs / 2)
	local g = New("Frame", {
		Size = UDim2.new(1, 0, 0, rows * 34 + (rows - 1) * 8), BackgroundTransparency = 1,
		LayoutOrder = O(parent), ZIndex = 4,
	}, parent)
	New("UIGridLayout", {
		CellSize = UDim2.new(0.5, -4, 0, 34), CellPadding = UDim2.fromOffset(8, 8), SortOrder = Enum.SortOrder.LayoutOrder,
	}, g)
	for i, d in ipairs(defs) do ActionButton(g, d[1], d[2], i) end
	return g
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

-- Biểu đồ đường thời gian thực (dùng cho FPS và Ping)
local MAXH = 40
local function MakeChart(parent, titleKey, minTop, minRange)
	local card = Card(parent, 112)
	local title = Text(card, "", 11, MUTED, Enum.Font.GothamBold)
	title.Position = UDim2.fromOffset(12, 6)
	title.Size = UDim2.new(0.5, 0, 0, 16)
	Loc(title, titleKey)
	local val = Text(card, "-", 12, TEXT, Enum.Font.GothamBold)
	val.TextXAlignment = Enum.TextXAlignment.Right
	val.AnchorPoint = Vector2.new(1, 0)
	val.Position = UDim2.new(1, -12, 0, 6)
	val.Size = UDim2.new(0.5, 0, 0, 16)
	local area = New("Frame", {
		Position = UDim2.fromOffset(36, 28), Size = UDim2.new(1, -48, 1, -40), BackgroundTransparency = 1, ZIndex = 5,
	}, card)
	local hiL = Text(card, "", 9, MUTED)
	hiL.TextXAlignment = Enum.TextXAlignment.Right
	hiL.Position = UDim2.fromOffset(2, 24)
	hiL.Size = UDim2.fromOffset(30, 12)
	local loL = Text(card, "", 9, MUTED)
	loL.TextXAlignment = Enum.TextXAlignment.Right
	loL.AnchorPoint = Vector2.new(0, 1)
	loL.Position = UDim2.new(0, 2, 1, -8)
	loL.Size = UDim2.fromOffset(30, 12)
	local ref = New("Frame", {
		Size = UDim2.new(1, 0, 0, 1), BackgroundColor3 = Color3.new(1, 1, 1), BackgroundTransparency = 0.82,
		BorderSizePixel = 0, Visible = false, ZIndex = 6,
	}, area)

	local hist, segs = {}, {}
	local chart = { Value = val }

	function chart.Push(v)
		table.insert(hist, v)
		if #hist > MAXH then table.remove(hist, 1) end
	end

	function chart.Draw()
		local W, H, n = area.AbsoluteSize.X, area.AbsoluteSize.Y, #hist
		if W <= 0 or H <= 0 or n < 2 then
			for _, s in ipairs(segs) do s.Visible = false end
			return
		end
		local lo, hi = math.huge, 0
		for _, v in ipairs(hist) do
			lo = math.min(lo, v)
			hi = math.max(hi, v)
		end
		hi = math.max(hi * 1.08, minTop) -- tự giãn theo FPS thực, không khóa ở 60
		lo = math.max(0, math.min(lo * 0.9, hi - minRange))
		local range = hi - lo
		hiL.Text, loL.Text = tostring(math.floor(hi)), tostring(math.floor(lo))
		if titleKey == "rating" and lo < 60 and hi > 60 then
			ref.Visible = true
			ref.Position = UDim2.fromOffset(0, H - (60 - lo) / range * H)
		else
			ref.Visible = false
		end
		local function pt(i)
			return (MAXH - n + i - 1) / (MAXH - 1) * W, H - (hist[i] - lo) / range * H
		end
		for i = 1, n - 1 do
			local x1, y1 = pt(i)
			local x2, y2 = pt(i + 1)
			local sg = segs[i]
			if not sg then
				sg = New("Frame", { AnchorPoint = Vector2.new(0.5, 0.5), BorderSizePixel = 0, ZIndex = 7 }, area)
				Corner(sg, 2)
				segs[i] = sg
			end
			local dx, dy = x2 - x1, y2 - y1
			sg.Visible = true
			sg.BackgroundColor3 = Accent
			sg.Size = UDim2.fromOffset(math.sqrt(dx * dx + dy * dy) + 1, 3)
			sg.Position = UDim2.fromOffset((x1 + x2) / 2, (y1 + y2) / 2)
			sg.Rotation = math.deg(math.atan2(dy, dx))
		end
		for i = n, #segs do segs[i].Visible = false end
	end

	Bind(chart.Draw)
	return chart
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
			local sz = math.random(4, 9)
			o = New("Frame", {
				Size = UDim2.fromOffset(sz, sz), BackgroundColor3 = Color3.new(1, 1, 1),
				BackgroundTransparency = math.random() * 0.2, BorderSizePixel = 0, ZIndex = 3,
			}, ParticleLayer)
			New("UICorner", { CornerRadius = UDim.new(0.5, 0) }, o)
			New("UIStroke", { Color = Color3.fromRGB(200, 225, 255), Transparency = 0.55, Thickness = 1.5 }, o)
			p.v = 0.06 + sz * 0.012
		elseif eff == "Sakura" then
			local w = math.random(11, 17)
			o = New("Frame", {
				Size = UDim2.fromOffset(w, math.floor(w * 0.62)), BackgroundColor3 = Color3.new(1, 1, 1),
				BackgroundTransparency = math.random() * 0.15, BorderSizePixel = 0, ZIndex = 3,
			}, ParticleLayer)
			New("UICorner", { CornerRadius = UDim.new(0.5, 0) }, o)
			New("UIGradient", {
				Color = ColorSequence.new(Color3.fromRGB(255, 214, 230):Lerp(Accent, 0.15), Color3.fromRGB(255, 140, 184):Lerp(Accent, 0.15)),
				Rotation = 30,
			}, o)
			New("UIStroke", { Color = Color3.fromRGB(255, 170, 205), Transparency = 0.5, Thickness = 1 }, o)
			p.v = 0.06 + math.random() * 0.06
			p.spin = true
		elseif eff == "Stars" then
			local sz = math.random(5, 9)
			local sc = math.random() < 0.5 and Color3.new(1, 1, 1) or Accent
			o = New("Frame", {
				Size = UDim2.fromOffset(sz, sz), Rotation = 45, BorderSizePixel = 0, ZIndex = 3, BackgroundColor3 = sc,
			}, ParticleLayer)
			New("UIStroke", { Color = sc, Transparency = 0.6, Thickness = 1.5 }, o)
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

do
	local Profile = Card(Home, 58)
	local Avatar = New("ImageLabel", {
		Size = UDim2.fromOffset(40, 40), Position = UDim2.fromOffset(10, 9), BackgroundColor3 = OFF, ZIndex = 6,
		Image = "rbxthumb://type=AvatarHeadShot&id=" .. LP.UserId .. "&w=150&h=150",
	}, Profile)
	Corner(Avatar, 20)
	local PName = Text(Profile, LP.DisplayName, 14, TEXT, Enum.Font.GothamBold)
	PName.Position = UDim2.fromOffset(58, 9)
	PName.Size = UDim2.new(1, -190, 0, 20)
	PName.TextTruncate = Enum.TextTruncate.AtEnd
	local PSub = Text(Profile, "@" .. LP.Name, 11, MUTED)
	PSub.Position = UDim2.fromOffset(58, 29)
	PSub.Size = UDim2.new(1, -190, 0, 17)
	PSub.TextTruncate = Enum.TextTruncate.AtEnd

	-- Trạng thái "Đang hoạt động" (tự co giãn theo chữ, bo tròn đều)
	local Pill = New("Frame", {
		AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -10, 0.5, 0), Size = UDim2.fromOffset(0, 24),
		AutomaticSize = Enum.AutomaticSize.X, BackgroundColor3 = GREEN, BackgroundTransparency = 0.82,
		BorderSizePixel = 0, ZIndex = 6,
	}, Profile)
	New("UICorner", { CornerRadius = UDim.new(0.5, 0) }, Pill)
	New("UIStroke", { Color = GREEN, Transparency = 0.6, Thickness = 1 }, Pill)
	New("UIPadding", { PaddingLeft = UDim.new(0, 10), PaddingRight = UDim.new(0, 12) }, Pill)
	New("UIListLayout", {
		FillDirection = Enum.FillDirection.Horizontal, VerticalAlignment = Enum.VerticalAlignment.Center,
		Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder,
	}, Pill)
	local Dot = New("Frame", {
		Size = UDim2.fromOffset(8, 8), BackgroundColor3 = GREEN, BorderSizePixel = 0, ZIndex = 7, LayoutOrder = 1,
	}, Pill)
	New("UICorner", { CornerRadius = UDim.new(0.5, 0) }, Dot)
	local PillT = New("TextLabel", {
		BackgroundTransparency = 1, Size = UDim2.fromOffset(0, 24), AutomaticSize = Enum.AutomaticSize.X, Text = "",
		TextSize = 11, Font = Enum.Font.GothamBold, TextColor3 = GREEN, ZIndex = 7, LayoutOrder = 2,
	}, Pill)
	Loc(PillT, "active")
	task.spawn(function()
		while Gui.Parent do
			Tween(Dot, 0.8, { BackgroundTransparency = 0.7 })
			task.wait(0.8)
			Tween(Dot, 0.8, { BackgroundTransparency = 0 })
			task.wait(0.8)
		end
	end)
end

-- Ô nhập lệnh (tiện cho mobile, không hiện trong chat)
do
	local cmdBox
	cmdBox = InputRow(Home, ".tp <tên/id>", "run", function(t)
		RunCommand(t, false)
		cmdBox.Text = ""
	end)
	Bind(function() cmdBox.PlaceholderText = T("cmdHint") end)
end

Section(Home, "overview")
local HomeGrid = StatGrid(Home)
local vFps, bFps = StatCard(HomeGrid, "fps")
local vPing, bPing = StatCard(HomeGrid, "ping")
local vMin, bMin = StatCard(HomeGrid, "min")
local vMax, bMax = StatCard(HomeGrid, "max")
local vAvg, bAvg = StatCard(HomeGrid, "avg")
local vMem, bMem = StatCard(HomeGrid, "memory")

local FpsChart = MakeChart(Home, "rating", 60, 30)
local PingChart = MakeChart(Home, "pingChart", 100, 40)
local RatingLabel = FpsChart.Value

-- Nhân vật + thao tác nhanh (V5.3)
do
	Section(Home, "character")
	local CG = StatGrid(Home)
	Act.vHp, Act.bHp = StatCard(CG, "hp")
	Act.vSpd, Act.bSpd = StatCard(CG, "speed")
	Act.vPos = InfoRow(Home, "pos")
	Act.vSess = InfoRow(Home, "session")
	ButtonRow(Home, {
		{ "respawn", function()
			local h = Act.hum(LP)
			if h then h.Health = 0 else ShowToast(T("noChar"), "err") end
		end },
		{ "gc", function()
			pcall(collectgarbage, "collect")
			ShowToast(T("gcDone"), "ok")
		end },
	})
end

--==================================================
-- PERFORMANCE
--==================================================
local Perf = Pages.PERFORMANCE
Section(Perf, "tools")

local ApplyFPSCap
do
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
	-- mới
	Toggle(Perf, "antiafk", function() return Config.AntiAFK end, function(v)
		Config.AntiAFK = v
		SaveConfig()
	end)
	Toggle(Perf, "lowgfx", function() return Config.LowGfx end, function(v)
		Config.LowGfx = v
		Act.applyPerf()
		SaveConfig()
	end)
	Toggle(Perf, "nofx", function() return Config.NoFx end, function(v)
		Config.NoFx = v
		Act.applyPerf()
		SaveConfig()
	end)

	Section(Perf, "fpsCap")
	ApplyFPSCap = function(announce)
		if type(setfpscap) ~= "function" then
			if announce then ShowToast(T("capNo"), "warn") end
			return
		end
		if Config.FPSCap <= 0 then
			-- "Mặc định" trước đây không làm gì: giờ gỡ giới hạn đã đặt
			if announce then
				pcall(setfpscap, 0)
				ShowToast(T("capSet") .. " " .. T("capOff"), "ok")
			end
			return
		end
		pcall(setfpscap, Config.FPSCap)
		if announce then ShowToast(T("capSet") .. " " .. Config.FPSCap, "ok") end
	end
	ChipGrid(Perf, {
		{ 0, "capOff" }, { 30, "30" }, { 60, "60" }, { 90, "90" }, { 120, "120" }, { 144, "144" }, { 240, "240" },
	}, function(c) return Config.FPSCap == c end, function(c)
		Config.FPSCap = c
		ApplyFPSCap(true)
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

	-- Sửa lỗi: trước đây hiển thị độ TRONG SUỐT nhưng gọi là "độ tối" nên bị ngược.
	-- Giờ + = tối hơn, hiển thị đúng % độ tối.
	Stepper(Perf, "opacity", function() return Config.Opacity end, function(d)
		Config.Opacity = math.clamp(math.floor((Config.Opacity - d * 0.05) * 100 + 0.5) / 100, 0, 0.8)
		Overlay.BackgroundTransparency = Config.Opacity
		SaveConfig()
	end, function(v) return math.floor((1 - v) * 100 + 0.5) .. "%" end)
end

--==================================================
-- PLAYERS
--==================================================
local PlayerPage = Pages.PLAYERS
local Search

do
	local SearchRow = Card(PlayerPage, 38)
	Search = New("TextBox", {
		Size = UDim2.new(1, -16, 0, 28), Position = UDim2.fromOffset(8, 5), BackgroundTransparency = 1,
		TextColor3 = TEXT, PlaceholderColor3 = MUTED, Text = "", ClearTextOnFocus = false, TextSize = 12,
		Font = Enum.Font.Gotham, TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 6,
	}, SearchRow)
	Bind(function() Search.PlaceholderText = "🔍  " .. T("search") end)
end

-- ESP: giờ dùng được cho mọi người (không còn phụ thuộc admin/server)
local ESPOn, ESPObjs = false, {}
local function RefreshESP()
	for _, h in ipairs(ESPObjs) do h:Destroy() end
	table.clear(ESPObjs)
	if not ESPOn then return end
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= LP and p.Character then
			local h = Instance.new("Highlight")
			h.FillColor = Accent
			h.OutlineColor = Accent
			h.FillTransparency = 0.7
			h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
			h.Parent = p.Character
			table.insert(ESPObjs, h)
			local head = p.Character:FindFirstChild("Head")
			if head then
				local bb = Instance.new("BillboardGui")
				bb.Size = UDim2.fromOffset(120, 18)
				bb.StudsOffset = Vector3.new(0, 2.6, 0)
				bb.AlwaysOnTop = true
				bb.Adornee = head
				local tl = Instance.new("TextLabel")
				tl.Size = UDim2.fromScale(1, 1)
				tl.BackgroundTransparency = 1
				tl.Text = p.DisplayName
				tl.TextColor3 = Accent
				tl.TextStrokeTransparency = 0.4
				tl.TextSize = 13
				tl.Font = Enum.Font.GothamBold
				tl.Parent = bb
				bb.Parent = p.Character
				table.insert(ESPObjs, bb)
			end
		end
	end
end
function Act.setESP(v)
	ESPOn = v
	RefreshESP()
	ShowToast(v and T("espOn") or T("espOff"), v and "ok" or "info")
end
Toggle(PlayerPage, "esp", function() return ESPOn end, function(v) Act.setESP(v) end)
ButtonRow(PlayerPage, {
	{ "unview", function() Act.unview() end },
	{ "showAll", function() Act.showAll() end },
})

local function WatchChar(p)
	p.CharacterAdded:Connect(function()
		task.wait(0.3)
		RefreshESP()
		if Act.Hidden[p] then Act.applyHide(p) end
	end)
end
for _, p in ipairs(Players:GetPlayers()) do WatchChar(p) end
Players.PlayerAdded:Connect(WatchChar)
Players.PlayerRemoving:Connect(function(p)
	Act.Hidden[p] = nil
	task.defer(RefreshESP)
end)

local PlayerList = New("Frame", {
	Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1,
	LayoutOrder = O(PlayerPage), ZIndex = 4,
}, PlayerPage)
New("UIListLayout", { Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder }, PlayerList)

local function MiniBtn(parent, txt, w, fn)
	local b = New("TextButton", {
		Text = txt, TextSize = 11, Font = Enum.Font.GothamBold, TextColor3 = TEXT, AutoButtonColor = true,
		BackgroundColor3 = OFF, Size = UDim2.fromOffset(w, 22), ZIndex = 6,
	}, parent)
	Corner(b, 8)
	b.Activated:Connect(fn)
	return b
end

local function RefreshPlayers()
	for _, c in ipairs(PlayerList:GetChildren()) do
		if c:IsA("Frame") or c:IsA("TextLabel") then c:Destroy() end
	end
	local q = Search.Text:lower()
	local n = 0
	for _, plr in ipairs(Players:GetPlayers()) do
		if q == "" or plr.Name:lower():find(q, 1, true) or plr.DisplayName:lower():find(q, 1, true) then
			n += 1
			local isMe = plr == LP
			-- noBind = true: card tạo lại liên tục, không được đăng ký vào Binds (sửa rò rỉ)
			local card = Card(PlayerList, isMe and 60 or 88, true)
			card.LayoutOrder = n
			local av = New("ImageLabel", {
				Size = UDim2.fromOffset(42, 42), Position = UDim2.fromOffset(9, 9), BackgroundColor3 = OFF, ZIndex = 6,
				Image = "rbxthumb://type=AvatarHeadShot&id=" .. plr.UserId .. "&w=150&h=150",
			}, card)
			Corner(av, 21)
			local wlab = isMe and -150 or -70
			local nm = Text(card, plr.DisplayName, 13, TEXT, Enum.Font.GothamBold)
			nm.Position = UDim2.fromOffset(60, 8)
			nm.Size = UDim2.new(1, wlab, 0, 18)
			local sub = Text(card, "@" .. plr.Name .. "  ·  ID " .. plr.UserId, 10, MUTED)
			sub.Position = UDim2.fromOffset(60, 27)
			sub.Size = UDim2.new(1, wlab, 0, 14)
			local age = Text(card, T("age") .. ": " .. plr.AccountAge .. " " .. T("days"), 10, MUTED)
			age.Position = UDim2.fromOffset(60, 41)
			age.Size = UDim2.new(1, wlab, 0, 14)
			if isMe then
				local tag = New("TextLabel", {
					Text = T("you"), TextSize = 10, Font = Enum.Font.GothamBold, TextColor3 = BASE,
					BackgroundColor3 = Accent, AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -10, 0, 8),
					Size = UDim2.fromOffset(40, 18), ZIndex = 6,
				}, card)
				Corner(tag, 9)
				if type(setclipboard) == "function" then
					local cp = MiniBtn(card, "ID", 40, function()
						pcall(setclipboard, tostring(plr.UserId))
						ShowToast(T("copied"), "ok")
					end)
					cp.AnchorPoint = Vector2.new(1, 1)
					cp.Position = UDim2.new(1, -10, 1, -8)
				end
			else
				-- hàng nút: TP / Xem / Ẩn / ID
				local bar = New("Frame", {
					Position = UDim2.fromOffset(9, 60), Size = UDim2.new(1, -18, 0, 22), BackgroundTransparency = 1, ZIndex = 6,
				}, card)
				New("UIListLayout", {
					FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder,
				}, bar)
				MiniBtn(bar, T("tpBtn"), 44, function() Act.tp(plr) end).LayoutOrder = 1
				MiniBtn(bar, "👁", 44, function() Act.view(plr) end).LayoutOrder = 2
				MiniBtn(bar, Act.Hidden[plr] and "🙈 ✓" or "🙈", 44, function()
					Act.hide(plr, not Act.Hidden[plr])
					RefreshPlayers()
				end).LayoutOrder = 3
				if type(setclipboard) == "function" then
					MiniBtn(bar, "ID", 40, function()
						pcall(setclipboard, tostring(plr.UserId))
						ShowToast(T("copied"), "ok")
					end).LayoutOrder = 4
				end
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

Section(World, "worldTools")
-- Sửa lỗi: trước đây ApplyWorld ghi đè Lighting của game bằng ảnh chụp lúc khởi động
-- mỗi lần đổi bất kỳ tùy chọn nào. Giờ chỉ lưu giá trị gốc lúc BẬT và trả lại lúc TẮT.
local Saved = {}
local function ApplyWorld(forceFov)
	if Config.FullBright then
		if not Saved.bright then
			Saved.bright = { Lighting.Brightness, Lighting.Ambient, Lighting.OutdoorAmbient }
		end
		Lighting.Brightness = 2
		Lighting.Ambient = Color3.new(1, 1, 1)
		Lighting.OutdoorAmbient = Color3.new(1, 1, 1)
	elseif Saved.bright then
		Lighting.Brightness, Lighting.Ambient, Lighting.OutdoorAmbient = table.unpack(Saved.bright)
		Saved.bright = nil
	end
	if Config.NoFog then
		if not Saved.fog then Saved.fog = Lighting.FogEnd end
		Lighting.FogEnd = 1e6
	elseif Saved.fog then
		Lighting.FogEnd = Saved.fog
		Saved.fog = nil
	end
	if not Config.Shadows then
		if Saved.shadows == nil then Saved.shadows = Lighting.GlobalShadows end
		Lighting.GlobalShadows = false
	elseif Saved.shadows ~= nil then
		Lighting.GlobalShadows = Saved.shadows
		Saved.shadows = nil
	end
	local cam = workspace.CurrentCamera
	if cam then
		if Config.FOV ~= Default.FOV then
			if not Saved.fov then Saved.fov = cam.FieldOfView end
			cam.FieldOfView = Config.FOV
		elseif Saved.fov then
			cam.FieldOfView = Saved.fov
			Saved.fov = nil
		end
	end
end
local function WSet(k)
	return function(v)
		Config[k] = v
		ApplyWorld()
		SaveConfig()
	end
end
Toggle(World, "fullbright", function() return Config.FullBright end, WSet("FullBright"))
Toggle(World, "nofog", function() return Config.NoFog end, WSet("NoFog"))
Toggle(World, "shadows", function() return Config.Shadows end, WSet("Shadows"))
Stepper(World, "fov", function() return Config.FOV end, function(d)
	Config.FOV = math.clamp(Config.FOV + d * 5, 40, 120)
	ApplyWorld(true)
	SaveConfig()
end, function(v) return tostring(v) end)
Stepper(World, "clock", function() return Lighting.ClockTime end, function(d)
	Lighting.ClockTime = (Lighting.ClockTime + d) % 24
end, function(v) return string.format("%02d:00", math.floor(v)) end)

-- Công cụ nhân vật (V5.3)
do
	Toggle(World, "noclip", function() return Act.noclip end, function(v)
		Act.noclip = v
		ShowToast(v and T("noclipOn") or T("noclipOff"), "info")
	end)
	Toggle(World, "infjump", function() return Act.ij end, function(v)
		Act.ij = v
		ShowToast(v and T("ijOn") or T("ijOff"), "info")
	end)
	local function num(v) return v > 0 and tostring(v) or T("auto") end
	Stepper(World, "walkspeed", function() return Config.Speed end, function(d)
		local n = (Config.Speed > 0 and Config.Speed or 16) + d * 4
		Config.Speed = n < 8 and 0 or math.min(n, 200)
		Act.applyChar(true)
		SaveConfig()
	end, num)
	Stepper(World, "jumppower", function() return Config.Jump end, function(d)
		local n = (Config.Jump > 0 and Config.Jump or 50) + d * 10
		Config.Jump = n < 20 and 0 or math.min(n, 300)
		Act.applyChar(true)
		SaveConfig()
	end, num)
	Stepper(World, "gravityTool", function() return Config.Gravity end, function(d)
		local n = (Config.Gravity > 0 and Config.Gravity or Act.origGrav) + d * 20
		Config.Gravity = math.clamp(math.floor(n + 0.5), 20, 400)
		Act.applyGrav()
		SaveConfig()
	end, num)
end

-- chỉ áp dụng lúc khởi động nếu người dùng từng đổi (tránh đè lên cài đặt của game)
if Config.FullBright or Config.NoFog or not Config.Shadows or Config.FOV ~= Default.FOV then
	ApplyWorld()
end
if Config.Gravity > 0 then Act.applyGrav() end
if Config.LowGfx or Config.NoFx then Act.applyPerf() end
task.defer(Act.applyChar, false)

local Server = Pages.SERVER
Section(Server, "serverInfo")
local sPlace = InfoRow(Server, "place")
local sJob = InfoRow(Server, "job")
local sCount = InfoRow(Server, "count")
local sUp = InfoRow(Server, "uptime")
local sPing = InfoRow(Server, "ping")
sPlace.Text = tostring(game.PlaceId)
sJob.Text = game.JobId ~= "" and string.sub(game.JobId, 1, 12) or "Studio"
local sVer = InfoRow(Server, "ver")
sVer.Text = tostring(game.PlaceVersion)
local sSend = InfoRow(Server, "sent")
local sRecv = InfoRow(Server, "recv")
local sLocal = InfoRow(Server, "localTime")
-- mới
Act.sName = InfoRow(Server, "gname")
Act.sPriv = InfoRow(Server, "priv")
Act.sPhys = InfoRow(Server, "physfps")
Act.sPriv.Text = (game.PrivateServerId ~= "" and "✔") or "✘"
task.spawn(function()
	local ok, info = pcall(function()
		return game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId)
	end)
	Act.sName.Text = (ok and info and info.Name) or "-"
end)

do
	local function copy(s)
		if type(setclipboard) == "function" then
			pcall(setclipboard, s)
			ShowToast(T("copied"), "ok")
		else
			ShowToast(T("noSave"), "warn")
		end
	end
	ButtonRow(Server, {
		{ "copyJob", function() copy(game.JobId) end },
		{ "rejoin", function()
			pcall(function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, LP) end)
		end },
		{ "newServer", function()
			pcall(function() TeleportService:Teleport(game.PlaceId, LP) end)
		end },
		{ "copyPlace", function() copy(tostring(game.PlaceId)) end },
		{ "copyLink", function() copy("https://www.roblox.com/games/" .. game.PlaceId) end },
	})
end

--==================================================
-- SETTINGS
--==================================================
local Settings = Pages.SETTINGS

Section(Settings, "theme")
do
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
end

Section(Settings, "hex")
InputRow(Settings, "#A78BFA", "apply", function(txt)
	local c = HexToColor(txt)
	if not c then
		ShowToast(T("badHex"), "err")
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
		ShowToast(T("bgSet"), "ok")
	else
		ShowToast(T("badId"), "err")
	end
end)

Section(Settings, "font")
ChipGrid(Settings, { { "Builder", "fontBuilder" }, { "Gotham", "fontGotham" } }, function(f) return Config.Font == f end, function(f)
	Config.Font = f
	ApplyFonts()
	ApplyAll()
	SaveConfig()
end)

Section(Settings, "display")
-- HUD FPS/Ping trên màn hình
local HUD = New("TextLabel", {
	AnchorPoint = Vector2.new(0.5, 0), Position = UDim2.new(0.5, 0, 0, 6), Size = UDim2.fromOffset(200, 22),
	BackgroundColor3 = BASE, BackgroundTransparency = 0.3, TextColor3 = TEXT, Font = Enum.Font.GothamBold,
	TextSize = 12, ZIndex = 60, Text = "",
}, Gui)
Corner(HUD, 11)
Bind(function()
	HUD.Visible = Config.HUD
	HUD.BackgroundColor3 = BASE
end)
Toggle(Settings, "hud", function() return Config.HUD end, function(v)
	Config.HUD = v
	ApplyAll()
	SaveConfig()
end)
Toggle(Settings, "notify", function() return Config.Notify end, function(v)
	Config.Notify = v
	SaveConfig()
end)
Toggle(Settings, "cmdOn", function() return Config.Commands end, function(v)
	Config.Commands = v
	SaveConfig()
end)
Stepper(Settings, "uiscale", function() return Config.UIScale end, function(d)
	Config.UIScale = math.clamp(math.floor((Config.UIScale + d * 0.1) * 10 + 0.5) / 10, 0.6, 1.4)
	MainScale.Scale = Config.UIScale
	SaveConfig()
end, function(v) return v .. "x" end)

do
	local ActionRow = New("Frame", {
		Size = UDim2.new(1, 0, 0, 36), BackgroundTransparency = 1, LayoutOrder = O(Settings), ZIndex = 4,
	}, Settings)
	New("UIGridLayout", {
		CellSize = UDim2.new(0.5, -4, 0, 36), CellPadding = UDim2.fromOffset(8, 0), SortOrder = Enum.SortOrder.LayoutOrder,
	}, ActionRow)
	ActionButton(ActionRow, "save", function()
		local saved = SaveConfig()
		ShowToast(saved and T("saved") or T("noSave"), saved and "ok" or "warn")
	end, 1)
	ActionButton(ActionRow, "reset", function()
		for k, v in pairs(Default) do Config[k] = v end
		ResolveAccent()
		SpawnParticles()
		MainScale.Scale = Config.UIScale
		ApplyWorld(true)
		Act.applyChar(true)
		Act.applyGrav()
		Act.applyPerf()
		Act.noclip, Act.ij = false, false
		ApplyFonts() -- sửa lỗi: Reset trước đây không áp lại font
		ApplyAll()
		SaveConfig() -- sửa lỗi: Reset trước đây không lưu
		ShowToast(T("resetDone"), "ok")
	end, 2)

	local Hint = Text(Settings, "", 10, MUTED)
	Hint.LayoutOrder = O(Settings)
	Loc(Hint, "hint")
end

--==================================================
-- LỆNH CHAT (.tp / .view / ...): chạy hoàn toàn ở client
--==================================================
do
	local cmds = {}
	Act.cmds = cmds

	local function target(a)
		if a == "" then
			ShowToast(T("usage"), "warn")
			return nil
		end
		local p = Act.find(a)
		if not p then ShowToast(T("notFound") .. ": " .. a, "err") end
		return p
	end
	local function number(a)
		local n = tonumber(a)
		if not n then ShowToast(T("numNeed"), "warn") end
		return n
	end

	cmds.tp = function(a)
		local p = target(a)
		if p then Act.tp(p) end
	end
	cmds["goto"] = cmds.tp
	cmds.view = function(a)
		local p = target(a)
		if p then Act.view(p) end
	end
	cmds.unview = function() Act.unview() end
	cmds.hide = function(a)
		local p = target(a)
		if p then Act.hide(p, true) end
	end
	cmds.show = function(a)
		if a == "" or a == "all" then
			Act.showAll()
			return
		end
		local p = target(a)
		if p then Act.hide(p, false) end
	end
	-- Roblox không cho client kick người khác: báo rõ thay vì giả vờ thành công
	cmds.kick = function() ShowToast(T("kickNo"), "warn") end
	cmds.speed = function(a)
		local n = number(a)
		if n then
			Config.Speed = n <= 0 and 0 or math.clamp(n, 1, 500)
			Act.applyChar(true)
			SaveConfig()
			ShowToast(T("setTo") .. " " .. tostring(Config.Speed), "ok")
		end
	end
	cmds.jump = function(a)
		local n = number(a)
		if n then
			Config.Jump = n <= 0 and 0 or math.clamp(n, 1, 1000)
			Act.applyChar(true)
			SaveConfig()
			ShowToast(T("setTo") .. " " .. tostring(Config.Jump), "ok")
		end
	end
	cmds.gravity = function(a)
		local n = number(a)
		if n then
			Config.Gravity = n <= 0 and 0 or math.clamp(n, 1, 1000)
			Act.applyGrav()
			SaveConfig()
			ShowToast(T("setTo") .. " " .. tostring(workspace.Gravity), "ok")
		end
	end
	cmds.fov = function(a)
		local n = number(a)
		if n then
			Config.FOV = math.clamp(n, 40, 120)
			ApplyWorld(true)
			SaveConfig()
			ShowToast(T("setTo") .. " " .. tostring(Config.FOV), "ok")
		end
	end
	cmds.time = function(a)
		local n = number(a)
		if n then
			Lighting.ClockTime = n % 24
			ShowToast(T("setTo") .. " " .. string.format("%02d:00", math.floor(n % 24)), "ok")
		end
	end
	cmds.esp = function()
		Act.setESP(not ESPOn)
		ApplyAll()
	end
	cmds.noclip = function()
		Act.noclip = not Act.noclip
		ShowToast(Act.noclip and T("noclipOn") or T("noclipOff"), "info")
		ApplyAll()
	end
	cmds.ij = function()
		Act.ij = not Act.ij
		ShowToast(Act.ij and T("ijOn") or T("ijOff"), "info")
		ApplyAll()
	end
	cmds.rejoin = function()
		pcall(function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, LP) end)
	end
	cmds.respawn = function()
		local h = Act.hum(LP)
		if h then h.Health = 0 else ShowToast(T("noChar"), "err") end
	end
	cmds.leave = function()
		ShowToast(T("leaving"), "info")
		task.wait(0.4)
		pcall(function() LP:Kick("KianBest") end)
	end
	cmds.cmds = function() ShowToast(T("cmdsList"), "info") end

	-- silent = true: lệnh sai thì im lặng (dùng cho chat để không báo lỗi khi gõ ".." bình thường)
	RunCommand = function(msg, silent)
		msg = tostring(msg or ""):match("^%s*(.-)%s*$")
		if msg == "" then return false end
		if msg:sub(1, 1) ~= "." then
			if silent then return false end
			msg = "." .. msg
		end
		local c, arg = msg:match("^%.(%a+)%s*(.-)%s*$")
		local fn = c and cmds[c:lower()]
		if not fn then
			if not silent then ShowToast(T("unknownCmd"), "warn") end
			return false
		end
		fn(arg)
		return true
	end

	local lastCmd, lastAt = "", 0
	local function SendCmd(msg)
		if not Config.Commands then return end
		if type(msg) ~= "string" or msg:sub(1, 1) ~= "." then return end
		if msg == lastCmd and os.clock() - lastAt < 1 then return end
		lastCmd, lastAt = msg, os.clock()
		RunCommand(msg, true)
	end
	pcall(function() LP.Chatted:Connect(SendCmd) end)
	pcall(function()
		TextChatService.MessageReceived:Connect(function(m)
			if m.TextSource and m.TextSource.UserId == LP.UserId then SendCmd(m.Text) end
		end)
	end)
	-- ẩn lệnh khỏi khung chat của chính bạn (người khác vẫn thấy tin gốc nếu game không chặn)
	pcall(function()
		local prev = TextChatService.OnIncomingMessage
		TextChatService.OnIncomingMessage = function(m)
			if Config.Commands and m.TextSource and m.TextSource.UserId == LP.UserId then
				local c = m.Text:match("^%.(%a+)")
				if c and cmds[c:lower()] then
					local pr = Instance.new("TextChatMessageProperties")
					pr.Text = ""
					return pr
				end
			end
			if prev then return prev(m) end
		end
	end)
end

--==================================================
-- FLOATING HOME BUTTON + OPEN/CLOSE
--==================================================
local Open = false
local function SetOpen(v)
	Open = v
	if v then
		Main.Visible = true
		MainScale.Scale = Config.UIScale * 0.9
		Tween(MainScale, 0.16, { Scale = Config.UIScale })
		RefreshPlayers()
	else
		Tween(MainScale, 0.12, { Scale = Config.UIScale * 0.9 })
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
New("UIStroke", { Color = Color3.new(1, 1, 1), Thickness = 2, Transparency = 0.2 }, Fab)
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
	local txt, col = Rating()
	vFps.TextColor3 = col
	-- thanh FPS không bị kẹt ở 60: thang đo tự tăng theo FPS cao nhất
	local scale = math.max(60, MaxFPS)
	SetBar(bFps, FPS / scale)
	SetBar(bPing, 1 - Ping / 300)
	SetBar(bMin, (MinFPS == math.huge and 0 or MinFPS) / scale)
	SetBar(bMax, MaxFPS / scale)
	SetBar(bAvg, (Samples > 0 and Sum / Samples or 0) / scale)
	SetBar(bMem, Mem / 1500)
	RatingLabel.Text = txt
	RatingLabel.TextColor3 = col
	PingChart.Value.Text = Ping .. " ms"
	FpsChart.Draw()
	PingChart.Draw()
end

local function UpdateInfo()
	local ok, p = pcall(function()
		return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
	end)
	Ping = ok and math.floor(p + 0.5) or 0
	Mem = math.floor(Stats:GetTotalMemoryUsageMb() + 0.5)
	PingChart.Push(Ping)
	-- game hay reset tốc độ/nhảy: áp lại mỗi giây nếu người dùng đã đặt
	if Config.Speed > 0 or Config.Jump > 0 then Act.applyChar(false) end
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
	pcall(function()
		sSend.Text = string.format("%.1f KB/s", Stats.DataSendKbps)
		sRecv.Text = string.format("%.1f KB/s", Stats.DataReceiveKbps)
	end)
	sLocal.Text = os.date("%H:%M:%S")
	-- mới: nhân vật, phiên chơi, FPS vật lý
	local hum = Act.hum(LP)
	if hum then
		Act.vHp.Text = math.floor(hum.Health + 0.5) .. "/" .. math.floor(hum.MaxHealth + 0.5)
		SetBar(Act.bHp, hum.Health / math.max(hum.MaxHealth, 1))
		Act.vSpd.Text = tostring(math.floor(hum.WalkSpeed + 0.5))
		SetBar(Act.bSpd, hum.WalkSpeed / 100)
	end
	local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
	if root then
		local pos = root.Position
		Act.vPos.Text = string.format("%d, %d, %d", math.floor(pos.X), math.floor(pos.Y), math.floor(pos.Z))
	end
	local ss = math.floor(os.clock() - StartClock)
	Act.vSess.Text = string.format("%02d:%02d:%02d", ss // 3600, (ss % 3600) // 60, ss % 60)
	pcall(function() Act.sPhys.Text = string.format("%.0f", workspace:GetRealPhysicsFPS()) end)
end

local okSig, RenderSig = pcall(function() return RunService.PreRender end)
if not okSig or not RenderSig then RenderSig = RunService.RenderStepped end
RenderSig:Connect(function(dt)
	Frames += 1
	FrameTime += dt
	if FrameTime >= 0.25 then
		FPS = math.floor(Frames / FrameTime + 0.5) -- FPS thực, không giới hạn
		Frames, FrameTime = 0, 0
		Warm += 1
		if Warm > 16 then
			MinFPS = math.min(MinFPS, FPS)
			MaxFPS = math.max(MaxFPS, FPS)
			Sum += FPS
			Samples += 1
		end
		FpsChart.Push(FPS)
		if Config.HUD then HUD.Text = FPS .. " FPS | " .. Ping .. " ms" end
		if Main.Visible then UpdateStats() end
	end

	if Main.Visible and #Particles > 0 then
		PAcc += dt
		if PAcc >= 1 / 60 then
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
ApplyFonts()
ApplyFPSCap(false)
SpawnParticles()
print("[KianBest V5.3 ULTRA | client] loaded | Theme:", Config.Theme, "| Language:", Config.Language)
