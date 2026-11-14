--[[
	KianBest V5.2 ULTRA  -  Roblox LocalScript
	Đặt vào: StarterPlayer > StarterPlayerScripts (LocalScript)

	CORE .......... Loader, Status Manager, Module Manager
	FLOATING HOME . nút nổi kéo được, mở/đóng dashboard (hoặc phím K)
	DASHBOARD ..... Home, Performance, Players, World, Server, Settings
	MONITOR ....... FPS thật, FPS min/max/avg, Ping, Memory
	BACKGROUND .... ảnh nền anime, độ trong suốt, bật/tắt
	LANGUAGE ...... Tiếng Việt / English
	THEMES ........ Purple, Sakura, Ocean, Emerald, Custom
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Stats = game:GetService("Stats")
local Lighting = game:GetService("Lighting")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

---------------------------------------------------------------------
-- CONFIG
---------------------------------------------------------------------
local CONFIG = {
	Name = "KianBest",
	Version = "V5.2 ULTRA",
	-- Upload ảnh anime lên Roblox rồi dán ID vào đây, ví dụ "rbxassetid://1234567890"
	WallpaperId = "",
	Language = "vi",
	Theme = "Purple",
	BackgroundOn = true,
	Transparency = 0.25,
	Toggle = Enum.KeyCode.K,
	CustomColor = Color3.fromRGB(255, 122, 89),
}

local THEMES = {
	Purple = { Color3.fromRGB(168, 85, 247), Color3.fromRGB(109, 40, 217) },
	Sakura = { Color3.fromRGB(244, 114, 182), Color3.fromRGB(190, 24, 93) },
	Ocean = { Color3.fromRGB(56, 189, 248), Color3.fromRGB(3, 105, 161) },
	Emerald = { Color3.fromRGB(52, 211, 153), Color3.fromRGB(4, 120, 87) },
}
local THEME_ORDER = { "Purple", "Sakura", "Ocean", "Emerald", "Custom" }

local LANG = {
	vi = {
		home = "Trang chủ", performance = "Hiệu năng", players = "Người chơi", world = "Thế giới",
		server = "Máy chủ", settings = "Cài đặt", fps = "FPS", ping = "Ping", memory = "Bộ nhớ",
		min = "FPS thấp nhất", max = "FPS cao nhất", avg = "FPS trung bình", modules = "Mô-đun",
		language = "Ngôn ngữ", theme = "Chủ đề", background = "Hình nền", transparency = "Độ trong suốt",
		running = "Đang chạy", stopped = "Đã dừng", uptime = "Thời gian chạy", count = "Số người chơi",
		clock = "Giờ trong game", gravity = "Trọng lực", brightness = "Độ sáng", streaming = "Streaming",
		place = "Place ID", job = "Job ID", age = "Tuổi tài khoản (ngày)", on = "Bật", off = "Tắt",
		monitor = "Giám sát", bgmod = "Hình nền", plmod = "Danh sách người chơi", hint = "Nhấn K để mở/đóng",
	},
	en = {
		home = "Home", performance = "Performance", players = "Players", world = "World",
		server = "Server", settings = "Settings", fps = "FPS", ping = "Ping", memory = "Memory",
		min = "FPS min", max = "FPS max", avg = "FPS average", modules = "Modules",
		language = "Language", theme = "Theme", background = "Background", transparency = "Transparency",
		running = "Running", stopped = "Stopped", uptime = "Uptime", count = "Players",
		clock = "In-game time", gravity = "Gravity", brightness = "Brightness", streaming = "Streaming",
		place = "Place ID", job = "Job ID", age = "Account age (days)", on = "On", off = "Off",
		monitor = "Monitor", bgmod = "Background", plmod = "Player list", hint = "Press K to open/close",
	},
}

local function tr(key)
	return (LANG[CONFIG.Language] or LANG.vi)[key] or key
end

local function accent()
	if CONFIG.Theme == "Custom" then
		return CONFIG.CustomColor, CONFIG.CustomColor:Lerp(Color3.new(0, 0, 0), 0.4)
	end
	local t = THEMES[CONFIG.Theme]
	return t[1], t[2]
end

local function new(class, props, parent)
	local o = Instance.new(class)
	for k, v in pairs(props) do
		o[k] = v
	end
	o.Parent = parent
	return o
end

---------------------------------------------------------------------
-- CORE: Loader / Status Manager / Module Manager
---------------------------------------------------------------------
local Core = { Modules = {}, Order = {}, Status = {} }

function Core.register(name, mod)
	mod.enabled = true
	Core.Modules[name] = mod
	table.insert(Core.Order, name)
	Core.Status[name] = "stopped"
end

function Core.start(name)
	local mod = Core.Modules[name]
	if not mod then return end
	local ok, err = pcall(function()
		if mod.onStart then mod.onStart() end
	end)
	mod.enabled = ok
	Core.Status[name] = ok and "running" or "error"
	if not ok then warn("[KianBest] " .. name .. ": " .. tostring(err)) end
end

function Core.stop(name)
	local mod = Core.Modules[name]
	if not mod then return end
	pcall(function()
		if mod.onStop then mod.onStop() end
	end)
	mod.enabled = false
	Core.Status[name] = "stopped"
end

function Core.toggle(name)
	if Core.Modules[name].enabled then Core.stop(name) else Core.start(name) end
end

---------------------------------------------------------------------
-- MONITOR data (FPS thật, Ping, Memory)
---------------------------------------------------------------------
local M = { fps = 0, min = math.huge, max = 0, sum = 0, n = 0, ping = 0, mem = 0, hist = {} }
local frames, acc, warm = 0, 0, 0
local monitorConn

local function readPing()
	local ok, v = pcall(function()
		return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
	end)
	return ok and math.floor(v + 0.5) or 0
end

---------------------------------------------------------------------
-- UI
---------------------------------------------------------------------
local gui = new("ScreenGui", {
	Name = "KianBestUI", ResetOnSpawn = false, IgnoreGuiInset = true,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling, DisplayOrder = 999,
}, playerGui)

local panel = new("Frame", {
	Name = "Dashboard", AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5),
	Size = UDim2.fromScale(0.92, 0.78), BackgroundColor3 = Color3.fromRGB(22, 12, 40),
	BackgroundTransparency = CONFIG.Transparency, Visible = false, ClipsDescendants = true,
}, gui)
new("UISizeConstraint", { MaxSize = Vector2.new(520, 380), MinSize = Vector2.new(260, 220) }, panel)
new("UICorner", { CornerRadius = UDim.new(0, 18) }, panel)
local panelStroke = new("UIStroke", { Thickness = 2, Transparency = 0.2 }, panel)
local panelGrad = new("UIGradient", { Rotation = 90 }, panel)

local wallpaper = new("ImageLabel", {
	Name = "Wallpaper", Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
	ScaleType = Enum.ScaleType.Crop, ImageTransparency = 0.55, ZIndex = 0,
}, panel)
new("UICorner", { CornerRadius = UDim.new(0, 18) }, wallpaper)

local title = new("TextLabel", {
	Position = UDim2.fromOffset(14, 8), Size = UDim2.new(1, -60, 0, 26), BackgroundTransparency = 1,
	Font = Enum.Font.GothamBold, TextSize = 18, TextXAlignment = Enum.TextXAlignment.Left,
	Text = CONFIG.Name .. "  " .. CONFIG.Version, TextColor3 = Color3.new(1, 1, 1), ZIndex = 2,
}, panel)

local closeBtn = new("TextButton", {
	AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -10, 0, 8), Size = UDim2.fromOffset(30, 26),
	BackgroundColor3 = Color3.fromRGB(255, 255, 255), BackgroundTransparency = 0.88,
	Text = "X", Font = Enum.Font.GothamBold, TextSize = 14, TextColor3 = Color3.new(1, 1, 1), ZIndex = 2,
}, panel)
new("UICorner", { CornerRadius = UDim.new(0, 8) }, closeBtn)

local tabBar = new("ScrollingFrame", {
	Position = UDim2.fromOffset(10, 40), Size = UDim2.new(1, -20, 0, 34), BackgroundTransparency = 1,
	ScrollBarThickness = 0, CanvasSize = UDim2.new(0, 0, 0, 0), AutomaticCanvasSize = Enum.AutomaticSize.X,
	ScrollingDirection = Enum.ScrollingDirection.X, ZIndex = 2,
}, panel)
new("UIListLayout", {
	FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 6),
	VerticalAlignment = Enum.VerticalAlignment.Center,
}, tabBar)

local content = new("Frame", {
	Position = UDim2.fromOffset(10, 80), Size = UDim2.new(1, -20, 1, -90), BackgroundTransparency = 1, ZIndex = 2,
}, panel)

-- ---------- widgets ----------
local function page(name)
	local p = new("ScrollingFrame", {
		Name = name, Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Visible = false,
		ScrollBarThickness = 3, CanvasSize = UDim2.new(0, 0, 0, 0), AutomaticCanvasSize = Enum.AutomaticSize.Y,
		BorderSizePixel = 0, ZIndex = 2,
	}, content)
	new("UIListLayout", { Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder }, p)
	return p
end

local langLabels = {} -- {label, key}
local function localized(label, key)
	label.Text = tr(key)
	table.insert(langLabels, { label, key })
end

local function row(parent, key, order)
	local r = new("Frame", {
		Size = UDim2.new(1, -6, 0, 34), BackgroundColor3 = Color3.new(1, 1, 1),
		BackgroundTransparency = 0.92, LayoutOrder = order or 0, ZIndex = 2,
	}, parent)
	new("UICorner", { CornerRadius = UDim.new(0, 10) }, r)
	local l = new("TextLabel", {
		Position = UDim2.fromOffset(10, 0), Size = UDim2.new(0.55, 0, 1, 0), BackgroundTransparency = 1,
		Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left,
		TextColor3 = Color3.fromRGB(230, 220, 250), ZIndex = 3,
	}, r)
	if key then localized(l, key) end
	local v = new("TextLabel", {
		AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -10, 0, 0), Size = UDim2.new(0.45, 0, 1, 0),
		BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 14, Text = "-",
		TextXAlignment = Enum.TextXAlignment.Right, TextColor3 = Color3.new(1, 1, 1), ZIndex = 3,
	}, r)
	return r, l, v
end

local accentButtons = {} -- buttons that follow the accent color
local function button(parent, text, size, onClick)
	local b = new("TextButton", {
		Size = size or UDim2.fromOffset(90, 30), Text = text, Font = Enum.Font.GothamBold, TextSize = 13,
		TextColor3 = Color3.new(1, 1, 1), AutoButtonColor = true, ZIndex = 3,
	}, parent)
	new("UICorner", { CornerRadius = UDim.new(0, 9) }, b)
	table.insert(accentButtons, b)
	b.Activated:Connect(onClick)
	return b
end

-- ---------- pages ----------
local pages, tabButtons, current = {}, {}, "home"

-- HOME
pages.home = page("home")
local _, _, hFps = row(pages.home, "fps", 1)
local _, _, hPing = row(pages.home, "ping", 2)
local _, _, hMem = row(pages.home, "memory", 3)
local modTitle = new("TextLabel", {
	Size = UDim2.new(1, -6, 0, 22), BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 13,
	TextXAlignment = Enum.TextXAlignment.Left, TextColor3 = Color3.fromRGB(190, 170, 230), LayoutOrder = 4, ZIndex = 2,
}, pages.home)
localized(modTitle, "modules")
local modRows = {}

-- PERFORMANCE
pages.performance = page("performance")
local _, _, pFps = row(pages.performance, "fps", 1)
local _, _, pMin = row(pages.performance, "min", 2)
local _, _, pMax = row(pages.performance, "max", 3)
local _, _, pAvg = row(pages.performance, "avg", 4)
local _, _, pPing = row(pages.performance, "ping", 5)
local _, _, pMem = row(pages.performance, "memory", 6)
local graph = new("Frame", {
	Size = UDim2.new(1, -6, 0, 70), BackgroundColor3 = Color3.new(1, 1, 1), BackgroundTransparency = 0.92,
	LayoutOrder = 7, ZIndex = 2,
}, pages.performance)
new("UICorner", { CornerRadius = UDim.new(0, 10) }, graph)
local bars = {}
for i = 1, 30 do
	bars[i] = new("Frame", {
		AnchorPoint = Vector2.new(0, 1), Position = UDim2.new((i - 1) / 30, 2, 1, -3),
		Size = UDim2.new(1 / 30, -3, 0, 2), BorderSizePixel = 0, ZIndex = 3,
	}, graph)
	new("UICorner", { CornerRadius = UDim.new(0, 2) }, bars[i])
end

-- PLAYERS
pages.players = page("players")
local playerRows = {}
local function rebuildPlayers()
	for _, r in ipairs(playerRows) do r:Destroy() end
	table.clear(playerRows)
	if not Core.Modules.PlayerList.enabled then return end
	for i, p in ipairs(Players:GetPlayers()) do
		local r, l, v = row(pages.players, nil, i)
		l.Text = p.DisplayName
		v.Text = p.AccountAge .. " d"
		table.insert(playerRows, r)
	end
end

-- WORLD
pages.world = page("world")
local _, _, wClock = row(pages.world, "clock", 1)
local _, _, wGrav = row(pages.world, "gravity", 2)
local _, _, wBright = row(pages.world, "brightness", 3)
local _, _, wStream = row(pages.world, "streaming", 4)

-- SERVER
pages.server = page("server")
local _, _, sPlace = row(pages.server, "place", 1)
local _, _, sJob = row(pages.server, "job", 2)
local _, _, sCount = row(pages.server, "count", 3)
local _, _, sUp = row(pages.server, "uptime", 4)
local _, _, sPing = row(pages.server, "ping", 5)
sPlace.Text = tostring(game.PlaceId)
sJob.Text = game.JobId ~= "" and string.sub(game.JobId, 1, 8) or "Studio"

-- SETTINGS
pages.settings = page("settings")
local applyAll -- forward

local function settingsRow(key, order, height)
	local r = new("Frame", {
		Size = UDim2.new(1, -6, 0, height or 40), BackgroundColor3 = Color3.new(1, 1, 1),
		BackgroundTransparency = 0.92, LayoutOrder = order, ZIndex = 2,
	}, pages.settings)
	new("UICorner", { CornerRadius = UDim.new(0, 10) }, r)
	local l = new("TextLabel", {
		Position = UDim2.fromOffset(10, 0), Size = UDim2.new(0.4, 0, 1, 0), BackgroundTransparency = 1,
		Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left,
		TextColor3 = Color3.fromRGB(230, 220, 250), ZIndex = 3,
	}, r)
	localized(l, key)
	local holder = new("Frame", {
		AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -8, 0.5, 0), Size = UDim2.new(0.58, 0, 1, -8),
		BackgroundTransparency = 1, ZIndex = 3,
	}, r)
	new("UIListLayout", {
		FillDirection = Enum.FillDirection.Horizontal, HorizontalAlignment = Enum.HorizontalAlignment.Right,
		VerticalAlignment = Enum.VerticalAlignment.Center, Padding = UDim.new(0, 6),
	}, holder)
	return holder
end

local langHolder = settingsRow("language", 1)
button(langHolder, "VI", UDim2.fromOffset(46, 28), function() CONFIG.Language = "vi"; applyAll() end)
button(langHolder, "EN", UDim2.fromOffset(46, 28), function() CONFIG.Language = "en"; applyAll() end)

local themeHolder = settingsRow("theme", 2, 48)
for _, name in ipairs(THEME_ORDER) do
	local c = name == "Custom" and CONFIG.CustomColor or THEMES[name][1]
	local sw = new("TextButton", {
		Size = UDim2.fromOffset(28, 28), BackgroundColor3 = c, Text = "", ZIndex = 4,
	}, themeHolder)
	new("UICorner", { CornerRadius = UDim.new(1, 0) }, sw)
	sw.Activated:Connect(function() CONFIG.Theme = name; applyAll() end)
end

local bgHolder = settingsRow("background", 3)
local bgToggle = button(bgHolder, "", UDim2.fromOffset(70, 28), function()
	CONFIG.BackgroundOn = not CONFIG.BackgroundOn
	applyAll()
end)

local trHolder = settingsRow("transparency", 4)
local trValue = new("TextLabel", {
	Size = UDim2.fromOffset(44, 28), BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 14,
	TextColor3 = Color3.new(1, 1, 1), ZIndex = 4,
}, trHolder)
button(trHolder, "-", UDim2.fromOffset(30, 28), function()
	CONFIG.Transparency = math.clamp(CONFIG.Transparency - 0.05, 0, 0.9); applyAll()
end)
button(trHolder, "+", UDim2.fromOffset(30, 28), function()
	CONFIG.Transparency = math.clamp(CONFIG.Transparency + 0.05, 0, 0.9); applyAll()
end)

local hint = new("TextLabel", {
	Size = UDim2.new(1, -6, 0, 22), BackgroundTransparency = 1, Font = Enum.Font.Gotham, TextSize = 12,
	TextColor3 = Color3.fromRGB(190, 170, 230), LayoutOrder = 5, ZIndex = 2,
}, pages.settings)
localized(hint, "hint")

-- ---------- tabs ----------
local TAB_ORDER = { "home", "performance", "players", "world", "server", "settings" }
local function showPage(name)
	current = name
	local a = accent()
	for key, p in pairs(pages) do p.Visible = (key == name) end
	for key, b in pairs(tabButtons) do
		b.BackgroundColor3 = (key == name) and a or Color3.new(1, 1, 1)
		b.BackgroundTransparency = (key == name) and 0 or 0.9
	end
end
for _, key in ipairs(TAB_ORDER) do
	local b = new("TextButton", {
		Size = UDim2.fromOffset(0, 28), AutomaticSize = Enum.AutomaticSize.X, Font = Enum.Font.GothamBold,
		TextSize = 13, TextColor3 = Color3.new(1, 1, 1), ZIndex = 3,
	}, tabBar)
	new("UIPadding", { PaddingLeft = UDim.new(0, 12), PaddingRight = UDim.new(0, 12) }, b)
	new("UICorner", { CornerRadius = UDim.new(0, 14) }, b)
	localized(b, key)
	tabButtons[key] = b
	b.Activated:Connect(function() showPage(key) end)
end

-- ---------- modules ----------
local function buildModuleRows()
	local list = { { "Monitor", "monitor" }, { "Background", "bgmod" }, { "PlayerList", "plmod" } }
	for i, m in ipairs(list) do
		local r, l, v = row(pages.home, m[2], 10 + i)
		v.Text = ""
		local status = new("TextLabel", {
			AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -62, 0, 0), Size = UDim2.new(0.3, 0, 1, 0),
			BackgroundTransparency = 1, Font = Enum.Font.Gotham, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Right,
			TextColor3 = Color3.fromRGB(190, 170, 230), ZIndex = 3,
		}, r)
		local btn = button(r, "", UDim2.fromOffset(48, 24), function()
			Core.toggle(m[1])
			applyAll()
		end)
		btn.AnchorPoint = Vector2.new(1, 0.5)
		btn.Position = UDim2.new(1, -8, 0.5, 0)
		modRows[m[1]] = { status = status, btn = btn }
	end
end

-- ---------- apply theme / language / background ----------
function applyAll()
	local a, a2 = accent()
	panelStroke.Color = a
	panelGrad.Color = ColorSequence.new(a2:Lerp(Color3.fromRGB(20, 10, 36), 0.55), Color3.fromRGB(20, 10, 36))
	panel.BackgroundTransparency = CONFIG.Transparency
	title.Text = CONFIG.Name .. "  " .. CONFIG.Version
	for _, b in ipairs(accentButtons) do b.BackgroundColor3 = a end
	for _, b in ipairs(bars) do b.BackgroundColor3 = a end
	for _, item in ipairs(langLabels) do item[1].Text = tr(item[2]) end
	for name, ui in pairs(modRows) do
		local mod = Core.Modules[name]
		ui.status.Text = mod.enabled and tr("running") or tr("stopped")
		ui.btn.Text = mod.enabled and tr("on") or tr("off")
	end
	bgToggle.Text = CONFIG.BackgroundOn and tr("on") or tr("off")
	trValue.Text = math.floor((1 - CONFIG.Transparency) * 100 + 0.5) .. "%"
	wallpaper.Image = CONFIG.WallpaperId
	wallpaper.Visible = CONFIG.BackgroundOn and Core.Modules.Background.enabled and CONFIG.WallpaperId ~= ""
	showPage(current)
	rebuildPlayers()
end

---------------------------------------------------------------------
-- Đăng ký module (Loader)
---------------------------------------------------------------------
Core.register("Monitor", {
	onStart = function()
		frames, acc, warm = 0, 0, 0
		monitorConn = RunService.RenderStepped:Connect(function(dt)
			frames += 1
			acc += dt
			if acc >= 0.5 then
				M.fps = math.floor(frames / acc + 0.5)
				frames, acc = 0, 0
				warm += 1
				if warm > 4 then
					M.min = math.min(M.min, M.fps)
					M.max = math.max(M.max, M.fps)
					M.sum += M.fps
					M.n += 1
				end
				table.insert(M.hist, M.fps)
				if #M.hist > 30 then table.remove(M.hist, 1) end
			end
		end)
	end,
	onStop = function()
		if monitorConn then monitorConn:Disconnect() monitorConn = nil end
	end,
})
Core.register("Background", {})
Core.register("PlayerList", {})

Players.PlayerAdded:Connect(function() task.defer(rebuildPlayers) end)
Players.PlayerRemoving:Connect(function() task.defer(rebuildPlayers) end)

---------------------------------------------------------------------
-- Cập nhật số liệu (mỗi 0.5 giây khi dashboard đang mở)
---------------------------------------------------------------------
local function fmtTime(s)
	s = math.floor(s)
	return string.format("%02d:%02d:%02d", s // 3600, (s % 3600) // 60, s % 60)
end

task.spawn(function()
	while gui.Parent do
		task.wait(0.5)
		if panel.Visible then
			M.ping = readPing()
			M.mem = math.floor(Stats:GetTotalMemoryUsageMb() + 0.5)
			local fps = Core.Modules.Monitor.enabled and tostring(M.fps) or "-"
			hFps.Text, hPing.Text, hMem.Text = fps, M.ping .. " ms", M.mem .. " MB"
			if current == "performance" then
				pFps.Text = fps
				pMin.Text = M.min == math.huge and "-" or tostring(M.min)
				pMax.Text = M.max == 0 and "-" or tostring(M.max)
				pAvg.Text = M.n > 0 and tostring(math.floor(M.sum / M.n + 0.5)) or "-"
				pPing.Text, pMem.Text = M.ping .. " ms", M.mem .. " MB"
				for i, bar in ipairs(bars) do
					local v = M.hist[#M.hist - 30 + i]
					bar.Size = UDim2.new(1 / 30, -3, 0, v and math.max(2, math.min(v / 120, 1) * 62) or 2)
				end
			elseif current == "world" then
				wClock.Text = string.format("%.1f", Lighting.ClockTime)
				wGrav.Text = string.format("%.1f", workspace.Gravity)
				wBright.Text = string.format("%.1f", Lighting.Brightness)
				wStream.Text = workspace.StreamingEnabled and tr("on") or tr("off")
			elseif current == "server" then
				sCount.Text = #Players:GetPlayers() .. "/" .. Players.MaxPlayers
				sUp.Text = fmtTime(workspace.DistributedGameTime)
				sPing.Text = M.ping .. " ms"
			end
		end
	end
end)

---------------------------------------------------------------------
-- FLOATING HOME (nút nổi kéo được)
---------------------------------------------------------------------
local fab = new("TextButton", {
	Name = "FloatingHome", Size = UDim2.fromOffset(54, 54), Position = UDim2.new(0, 16, 0.4, 0),
	Text = "KB", Font = Enum.Font.GothamBlack, TextSize = 18, TextColor3 = Color3.new(1, 1, 1),
	AutoButtonColor = false, ZIndex = 50,
}, gui)
new("UICorner", { CornerRadius = UDim.new(1, 0) }, fab)
local fabStroke = new("UIStroke", { Thickness = 2, Color = Color3.new(1, 1, 1), Transparency = 0.3 }, fab)
table.insert(accentButtons, fab)

local function setOpen(v)
	panel.Visible = v
end
closeBtn.Activated:Connect(function() setOpen(false) end)

do
	local dragging, moved, startInput, startPos = false, false, nil, nil
	fab.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging, moved = true, false
			startInput, startPos = input.Position, fab.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
					if not moved then setOpen(not panel.Visible) end
				end
			end)
		end
	end)
	UIS.InputChanged:Connect(function(input)
		if not dragging then return end
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			local d = input.Position - startInput
			if d.Magnitude > 6 then moved = true end
			if moved then
				fab.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
			end
		end
	end)
end

UIS.InputBegan:Connect(function(input, processed)
	if not processed and input.KeyCode == CONFIG.Toggle then
		setOpen(not panel.Visible)
	end
end)

---------------------------------------------------------------------
-- START
---------------------------------------------------------------------
buildModuleRows()
for _, name in ipairs(Core.Order) do Core.start(name) end
applyAll()
print(("[%s %s] loaded"):format(CONFIG.Name, CONFIG.Version))
