local Config = {
    Admins = {
        [123456789] = true,
    },
    Detection = {
        Enabled = true,
        MaxWalkSpeed = 24,
        MaxJumpPower = 70,
        StrikeLimit = 5,
        StrikeDecayTime = 30,
        SpawnGraceTime = 5,
        TeleportDistance = 100,
        MaxMovementSpeed = 80,
        CheckInterval = 0.25,
    },
    Ban = {
        Enabled = true,
        Permanent = true,
        Reason = "Anti-Cheat detected",
    },
    Remote = {
        DefaultLimit = 15,
        DefaultWindow = 1,
        MaxStringLength = 500,
        MaxTableDepth = 5,
        MaxTableItems = 100,
        KickOnExtremeSpam = false,
    },
    Debug = true,
}
return Config
local PlayerState = {}

function PlayerState.create(player)
    PlayerState[player] = {
        Strikes = 0,
        LastPosition = nil,
        LastCheck = os.clock(),
        SpawnTime = os.clock(),
        ExemptUntil = 0,
        IsBanned = false,
        LastStrikeReason = "",
        LastStrikeTime = 0,
    }
end

function PlayerState.get(player)
    return PlayerState[player]
end

function PlayerState.remove(player)
    PlayerState[player] = nil
end

return PlayerState
local DataStoreService = game:GetService("DataStoreService")
local Config = require(script.Parent.Config)
local PlayerState = require(script.Parent.PlayerState)

local BanStore = DataStoreService:GetDataStore("Kianbest_AntiCheat_Bans_V3")

local BanSystem = {}

function BanSystem.getKey(userId)
    return "BAN_" .. tostring(userId)
end

function BanSystem.check(player)
    local success, result = pcall(function()
        return BanStore:GetAsync(BanSystem.getKey(player.UserId))
    end)
    if success and result then
        return true, result
    end
    return false, nil
end

function BanSystem.apply(player, reason)
    if Config.Admins[player.UserId] then return end
    reason = reason or Config.Ban.Reason
    local state = PlayerState.get(player)
    if state then state.IsBanned = true end

    local banData = {
        UserId = player.UserId,
        Name = player.Name,
        Reason = reason,
        Time = os.time(),
        Permanent = Config.Ban.Permanent,
    }
    pcall(function()
        BanStore:SetAsync(BanSystem.getKey(player.UserId), banData)
    end)
    player:Kick("You have been banned.\nReason: " .. tostring(reason))
end

return BanSystem
local Config = require(script.Parent.Config)
local PlayerState = require(script.Parent.PlayerState)
local BanSystem = require(script.Parent.BanSystem)

local StrikeSystem = {}

-- Thêm strike cho player
function StrikeSystem.add(player, reason)
    local state = PlayerState.get(player)
    if not state then return end
    if Config.Admins[player.UserId] then return end
    if os.clock() < state.ExemptUntil then return end

    state.Strikes += 1
    state.LastStrikeReason = reason
    state.LastStrikeTime = os.clock()

    if Config.Debug then
        print("[AntiCheat] Strike:", player.Name, state.Strikes, reason)
    end

    if state.Strikes >= Config.Detection.StrikeLimit then
        BanSystem.apply(player, reason)
    end
end

-- Giảm strike theo thời gian
function StrikeSystem.decay(player)
    local state = PlayerState.get(player)
    if not state then return end
    if state.Strikes <= 0 then return end

    if os.clock() - state.LastStrikeTime >= Config.Detection.StrikeDecayTime then
        state.Strikes = math.max(0, state.Strikes - 1)
        state.LastStrikeTime = os.clock()
        if Config.Debug then
            print("[AntiCheat] Strike decay:", player.Name, state.Strikes)
        end
    end
end

return StrikeSystem
local Config = require(script.Parent.Config)
local PlayerState = require(script.Parent.PlayerState)
local StrikeSystem = require(script.Parent.StrikeSystem)

local RunService = game:GetService("RunService")

local MovementProtection = {}

-- Setup nhân vật khi spawn
function MovementProtection.setupCharacter(player, character)
    local state = PlayerState.get(player)
    if not state then return end

    state.SpawnTime = os.clock()
    state.LastPosition = nil
    state.LastCheck = os.clock()

    local humanoid = character:FindFirstChildOfClass("Humanoid")
    local root = character:FindFirstChild("HumanoidRootPart")

    if humanoid then
        humanoid.WalkSpeed = math.min(humanoid.WalkSpeed, Config.Detection.MaxWalkSpeed)
        humanoid.JumpPower = math.min(humanoid.JumpPower, Config.Detection.MaxJumpPower)
    end

    if root then
        state.LastPosition = root.Position
    end
end

-- Kiểm tra tốc độ đi bộ
local function validateWalkSpeed(player, humanoid)
    if humanoid.WalkSpeed > Config.Detection.MaxWalkSpeed then
        StrikeSystem.add(player, "Abnormal WalkSpeed: " .. humanoid.WalkSpeed)
        humanoid.WalkSpeed = Config.Detection.MaxWalkSpeed
    end
end

-- Kiểm tra nhảy
local function validateJump(player, humanoid)
    if humanoid.UseJumpPower then
        if humanoid.JumpPower > Config.Detection.MaxJumpPower then
            StrikeSystem.add(player, "Abnormal JumpPower: " .. humanoid.JumpPower)
            humanoid.JumpPower = Config.Detection.MaxJumpPower
        end
    else
        local maxJumpHeight = 15
        if humanoid.JumpHeight > maxJumpHeight then
            StrikeSystem.add(player, "Abnormal JumpHeight: " .. humanoid.JumpHeight)
            humanoid.JumpHeight = maxJumpHeight
        end
    end
end

-- Kiểm tra di chuyển bất thường
local function validateMovement(player, root, humanoid)
    local state = PlayerState.get(player)
    if not state then return end

    local now = os.clock()
    local previous = state.LastPosition
    state.LastPosition = root.Position
    if not previous then return end

    if now - state.SpawnTime < Config.Detection.SpawnGraceTime then return end

    local dt = now - state.LastCheck
    state.LastCheck = now
    if dt <= 0 or dt > 1 then return end

    local distance = (root.Position - previous).Magnitude
    local expected = math.max(humanoid.WalkSpeed, 0) * dt
    local tolerance = math.max(8, expected * 1.5)
    local maximum = math.max(tolerance, Config.Detection.MaxMovementSpeed * dt + 10)

    if distance > Config.Detection.TeleportDistance then
        StrikeSystem.add(player, "Abnormal teleport distance: " .. distance)
        root.CFrame = CFrame.new(previous)
        return
    end

    if distance > maximum then
        StrikeSystem.add(player, "Abnormal movement speed: " .. math.floor(distance / dt))
        root.CFrame = CFrame.new(previous)
        return
    end
end

-- Heartbeat check
function MovementProtection.checkAll(dt)
    for _, player in ipairs(game.Players:GetPlayers()) do
        local character = player.Character
        if not character then continue end
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        local root = character:FindFirstChild("HumanoidRootPart")
        if not humanoid or not root then continue end

        validateWalkSpeed(player, humanoid)
        validateJump(player, humanoid)
        validateMovement(player, root, humanoid)
    end
end

return MovementProtection
local Config = require(script.Parent.Config)
local PlayerState = require(script.Parent.PlayerState)
local StrikeSystem = require(script.Parent.StrikeSystem)
local BanSystem = require(script.Parent.BanSystem)

local RemoteSecurity = {}
local RemoteState = {}
local ProtectedRemotes = {}

-- Reset remote state khi player rời game
function RemoteSecurity.reset(player)
    RemoteState[player] = nil
end

-- Lấy ID remote
local function getRemoteId(remote)
    if not remote then return "Unknown" end
    return remote:GetFullName()
end

-- Lấy state của player cho remote
local function getRemotePlayerState(player)
    if not RemoteState[player] then
        RemoteState[player] = {}
    end
    return RemoteState[player]
end

-- Làm sạch lịch sử remote
local function cleanRemoteHistory(data, now, window)
    local timestamps = data.Timestamps or {}
    local newList = {}
    for _, t in ipairs(timestamps) do
        if now - t <= window then
            table.insert(newList, t)
        end
    end
    data.Timestamps = newList
end

-- Kiểm tra spam remote
local function checkRemoteRate(player, remote, limit, window)
    local playerData = getRemotePlayerState(player)
    local id = getRemoteId(remote)
    local data = playerData[id]

    if not data then
        data = { Timestamps = {}, Violations = 0 }
        playerData[id] = data
    end

    local now = os.clock()
    cleanRemoteHistory(data, now, window)
    table.insert(data.Timestamps, now)

    if #data.Timestamps <= limit then return true end

    data.Violations += 1
    StrikeSystem.add(player, "Remote spam: " .. remote.Name)

    if Config.Remote.KickOnExtremeSpam and data.Violations >= 10 then
        BanSystem.apply(player, "Extreme remote spam")
        return false
    end
    return false
end

-- Validate giá trị truyền qua remote
local function validateValue(value, depth)
    depth = depth or 0
    if depth > Config.Remote.MaxTableDepth then return false, "table depth too large" end
    local t = typeof(value)

    if t == "nil" or t == "boolean" or t == "number" then
        if value ~= value then return false, "NaN detected" end
        if value == math.huge or value == -math.huge then return false, "infinite number" end
        return true
    elseif t == "string" then
        if #value > Config.Remote.MaxStringLength then return false, "string too long" end
        return true
    elseif t == "Instance" then
        if not value.Parent then return false, "destroyed instance" end
        return true
    elseif t == "Vector3" then
        if value.X ~= value.X or value.Y ~= value.Y or value.Z ~= value.Z then return false, "invalid Vector3" end
        return true
    elseif t == "CFrame" then
        local pos = value.Position
        if pos.X ~= pos.X or pos.Y ~= pos.Y or pos.Z ~= pos.Z then return false, "invalid CFrame" end
        return true
    elseif t == "table" then
        local count = 0
        for k, v in pairs(value) do
            count += 1
            if count > Config.Remote.MaxTableItems then return false, "table too large" end
            local keyValid = validateValue(k, depth + 1)
            if not keyValid then return false, "invalid table key" end
            local childValid, reason = validateValue(v, depth + 1)
            if not childValid then return false, reason end
        end
        return true
    end
    return false, "unsupported type: " .. tostring(t)
end

-- Validate toàn bộ arguments
local function validateArguments(player, remote, args)
    for i, v in ipairs(args) do
        local valid, reason = validateValue(v, 0)
        if not valid then
            StrikeSystem.add(player, "Invalid remote argument: " .. remote.Name .. " [" .. i .. "] " .. reason)
            return false
        end
    end
    return true
end

-- Đăng ký remote cần bảo vệ
function RemoteSecurity.register(remote, limit, window)
    if not remote then return end
    if not (remote:IsA("RemoteEvent") or remote:IsA("RemoteFunction")) then return end
    ProtectedRemotes[remote] = {
        Limit = tonumber(limit) or Config.Remote.DefaultLimit,
        Window = tonumber(window) or Config.Remote.DefaultWindow,
    }
    if Config.Debug then
        print("[AntiCheat] Protected remote:", remote:GetFullName())
    end
end

-- Kiểm tra tất cả remote
function RemoteSecurity.checkAll()
    for remote, settings in pairs(ProtectedRemotes) do
        remote.OnServerEvent:Connect(function(player, ...)
            local args = {...}
            if not checkRemoteRate(player, remote, settings.Limit, settings.Window) then return end
            if not validateArguments(player, remote, args) then return end
        end)
    end
end

return RemoteSecurity
-- Main AntiCheat Loader
local ServerStorage = game:GetService("ServerStorage")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

-- Import modules
local Config = require(ServerStorage.KianbestAntiCheat.Config)
local PlayerState = require(ServerStorage.KianbestAntiCheat.PlayerState)
local BanSystem = require(ServerStorage.KianbestAntiCheat.BanSystem)
local StrikeSystem = require(ServerStorage.KianbestAntiCheat.StrikeSystem)
local MovementProtection = require(ServerStorage.KianbestAntiCheat.MovementProtection)
local RemoteSecurity = require(ServerStorage.KianbestAntiCheat.RemoteSecurity)

print("[Kianbest Anti-Cheat] V3 Loaded")

-- Player join
Players.PlayerAdded:Connect(function(player)
    PlayerState.create(player)
    local banned, data = BanSystem.check(player)
    if banned then
        player:Kick("You are banned.\nReason: " .. tostring(data.Reason))
        return
    end
    player.CharacterAdded:Connect(function(char)
        MovementProtection.setupCharacter(player, char)
    end)
end)

-- Player leave
Players.PlayerRemoving:Connect(function(player)
    PlayerState.remove(player)
    RemoteSecurity.reset(player)
end)

-- Heartbeat checks
RunService.Heartbeat:Connect(function(dt)
    for _, player in ipairs(Players:GetPlayers()) do
        StrikeSystem.decay(player)
    end
    MovementProtection.checkAll(dt)
end)

-- Remote protection init
RemoteSecurity.checkAll()
local ServerStorage = game:GetService("ServerStorage")

local LoggingSystem = {}
local LogFolder = ServerStorage:FindFirstChild("AntiCheatLogs")

if not LogFolder then
    LogFolder = Instance.new("Folder")
    LogFolder.Name = "AntiCheatLogs"
    LogFolder.Parent = ServerStorage
end

-- Tạo log entry
function LoggingSystem.log(player, action, detail)
    local entry = Instance.new("StringValue")
    entry.Name = player.Name .. "_" .. os.time()
    entry.Value = "[AntiCheat] Player: " .. player.Name .. " | Action: " .. action .. " | Detail: " .. detail
    entry.Parent = LogFolder
    print(entry.Value)
end

-- Lấy toàn bộ log
function LoggingSystem.getAll()
    local logs = {}
    for _, v in ipairs(LogFolder:GetChildren()) do
        table.insert(logs, v.Value)
    end
    return logs
end

-- Xóa log cũ
function LoggingSystem.clearOld(maxEntries)
    local children = LogFolder:GetChildren()
    if #children > maxEntries then
        table.sort(children, function(a, b) return a.Name < b.Name end)
        for i = 1, #children - maxEntries do
            children[i]:Destroy()
        end
    end
end

return LoggingSystem
local Config = require(script.Parent.Config)
local PlayerState = require(script.Parent.PlayerState)
local BanSystem = require(script.Parent.BanSystem)
local LoggingSystem = require(script.Parent.LoggingSystem)

local AdminTools = {}

-- Kiểm tra quyền admin
function AdminTools.isAdmin(player)
    return Config.Admins[player.UserId] == true
end

-- Ban thủ công
function AdminTools.manualBan(admin, target, reason)
    if not AdminTools.isAdmin(admin) then return false, "Not authorized" end
    BanSystem.apply(target, reason or "Manual ban by admin")
    LoggingSystem.log(target, "ManualBan", reason or "No reason")
    return true, "Player banned"
end

-- Unban thủ công
function AdminTools.unban(admin, userId)
    if not AdminTools.isAdmin(admin) then return false, "Not authorized" end
    local key = BanSystem.getKey(userId)
    local DataStoreService = game:GetService("DataStoreService")
    local BanStore = DataStoreService:GetDataStore("Kianbest_AntiCheat_Bans_V3")
    pcall(function()
        BanStore:RemoveAsync(key)
    end)
    LoggingSystem.log(admin, "Unban", "UserId: " .. userId)
    return true, "Player unbanned"
end

-- Xem trạng thái player
function AdminTools.inspect(admin, player)
    if not AdminTools.isAdmin(admin) then return nil, "Not authorized" end
    local state = PlayerState.get(player)
    if not state then return nil, "No state" end
    return {
        Strikes = state.Strikes,
        LastStrikeReason = state.LastStrikeReason,
        LastStrikeTime = state.LastStrikeTime,
        IsBanned = state.IsBanned,
    }
end

-- Xem log
function AdminTools.viewLogs(admin)
    if not AdminTools.isAdmin(admin) then return nil, "Not authorized" end
    return LoggingSystem.getAll()
end

return AdminTools
