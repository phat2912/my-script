--[[
    Kianbest Anti-Cheat V3
    PART 1/5
    Server-side protection
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local DataStoreService = game:GetService("DataStoreService")
local ServerStorage = game:GetService("ServerStorage")

--------------------------------------------------
-- CONFIG
--------------------------------------------------

local CONFIG = {
    Admins = {
        -- Thay bằng UserId Roblox của bạn
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

    Debug = false,
}

--------------------------------------------------
-- SERVICES / STORAGE
--------------------------------------------------

local BanStore = DataStoreService:GetDataStore(
    "Kianbest_AntiCheat_Bans_V3"
)

local RootFolder = ServerStorage:FindFirstChild(
    "KianbestAntiCheat"
)

if not RootFolder then
    RootFolder = Instance.new("Folder")
    RootFolder.Name = "KianbestAntiCheat"
    RootFolder.Parent = ServerStorage
end

--------------------------------------------------
-- PLAYER STATE
--------------------------------------------------

local PlayerState = {}

local function createState(player)
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

local function getState(player)
    return PlayerState[player]
end

--------------------------------------------------
-- ADMIN CHECK
--------------------------------------------------

local function isAdmin(player)
    return CONFIG.Admins[player.UserId] == true
end

--------------------------------------------------
-- DEBUG
--------------------------------------------------

local function debugPrint(...)
    if CONFIG.Debug then
        print("[Kianbest Anti-Cheat]", ...)
    end
end

--------------------------------------------------
-- BAN DATA
--------------------------------------------------

local function getBanKey(userId)
    return "BAN_" .. tostring(userId)
end

local function getBanData(player)
    local success, result = pcall(function()
        return BanStore:GetAsync(
            getBanKey(player.UserId)
        )
    end)

    if not success then
        warn(
            "[Kianbest Anti-Cheat] DataStore error:",
            result
        )

        return nil
    end

    return result
end

--------------------------------------------------
-- APPLY BAN
--------------------------------------------------

local function banPlayer(player, reason)
    if not player or not player.Parent then
        return
    end

    if isAdmin(player) then
        debugPrint(
            "Admin ban prevented:",
            player.Name
        )

        return
    end

    reason = reason or CONFIG.Ban.Reason

    local state = getState(player)

    if state then
        state.IsBanned = true
    end

    local banData = {
        UserId = player.UserId,
        Name = player.Name,
        Reason = reason,
        Time = os.time(),
        Permanent = CONFIG.Ban.Permanent,
    }

    pcall(function()
        BanStore:SetAsync(
            getBanKey(player.UserId),
            banData
        )
    end)

    player:Kick(
        "You have been banned.\nReason: "
        .. tostring(reason)
    )
end

--------------------------------------------------
-- CHECK EXISTING BAN
--------------------------------------------------

local function checkBan(player)
    if isAdmin(player) then
        return false
    end

    local data = getBanData(player)

    if not data then
        return false
    end

    return true, data
end

--------------------------------------------------
-- STRIKE SYSTEM
--------------------------------------------------

local function addStrike(player, reason)
    local state = getState(player)

    if not state then
        return
    end

    if isAdmin(player) then
        return
    end

    if os.clock() < state.ExemptUntil then
        return
    end

    state.Strikes += 1
    state.LastStrikeReason = reason
    state.LastStrikeTime = os.clock()

    debugPrint(
        player.Name,
        "Strike:",
        state.Strikes,
        reason
    )

    if state.Strikes >= CONFIG.Detection.StrikeLimit then
        banPlayer(
            player,
            reason
        )
    end
end

--------------------------------------------------
-- STRIKE DECAY
--------------------------------------------------

local function updateStrikeDecay(player)
    local state = getState(player)

    if not state then
        return
    end

    if state.Strikes <= 0 then
        return
    end

    if os.clock() - state.LastStrikeTime
        >= CONFIG.Detection.StrikeDecayTime
    then
        state.Strikes = math.max(
            0,
            state.Strikes - 1
        )

        state.LastStrikeTime = os.clock()

        debugPrint(
            player.Name,
            "Strike decay:",
            state.Strikes
        )
    end
end

--------------------------------------------------
-- CHARACTER SETUP
--------------------------------------------------

local function setupCharacter(player, character)
    local state = getState(player)

    if not state then
        return
    end

    state.SpawnTime = os.clock()
    state.LastPosition = nil
    state.LastCheck = os.clock()

    local humanoid = character:FindFirstChildOfClass(
        "Humanoid"
    )

    local root = character:FindFirstChild(
        "HumanoidRootPart"
    )

    if humanoid then
        humanoid.WalkSpeed =
            math.min(
                humanoid.WalkSpeed,
                CONFIG.Detection.MaxWalkSpeed
            )

        humanoid.JumpPower =
            math.min(
                humanoid.JumpPower,
                CONFIG.Detection.MaxJumpPower
            )
    end

    if root then
        state.LastPosition = root.Position
    end
end

--------------------------------------------------
-- PLAYER JOIN
--------------------------------------------------

Players.PlayerAdded:Connect(function(player)

    createState(player)

    local banned, data = checkBan(player)

    if banned then
        task.defer(function()

            local reason =
                "Existing ban"

            if type(data) == "table"
                and data.Reason
            then
                reason = data.Reason
            end

            player:Kick(
                "You are banned.\nReason: "
                .. tostring(reason)
            )
        end)

        return
    end

    player.CharacterAdded:Connect(
        function(character)
            setupCharacter(
                player,
                character
            )
        end
    )

    if player.Character then
        setupCharacter(
            player,
            player.Character
        )
    end
end)

--------------------------------------------------
-- PLAYER LEAVE
--------------------------------------------------

Players.PlayerRemoving:Connect(function(player)

    PlayerState[player] = nil

end)

--------------------------------------------------
-- SERVER LOOP
--------------------------------------------------

task.spawn(function()

    while true do

        task.wait(
            CONFIG.Detection.CheckInterval
        )

        if not CONFIG.Detection.Enabled then
            continue
        end

        for _, player in ipairs(
            Players:GetPlayers()
        ) do

            updateStrikeDecay(player)

        end
    end

end)

print(
    "[Kianbest Anti-Cheat] V3 Part 1 loaded."
)
--------------------------------------------------
-- KIANBEST ANTI-CHEAT V3
-- PART 2/5
-- MOVEMENT PROTECTION
--------------------------------------------------

local function isExempt(player)
    local state = getState(player)

    if not state then
        return false
    end

    return os.clock() < state.ExemptUntil
end

--------------------------------------------------
-- TEMPORARY EXEMPTION
--------------------------------------------------

local function exemptPlayer(player, duration)
    local state = getState(player)

    if not state then
        return
    end

    duration = tonumber(duration) or 3

    state.ExemptUntil =
        math.max(
            state.ExemptUntil,
            os.clock() + duration
        )

    debugPrint(
        player.Name,
        "movement exemption:",
        duration
    )
end

--------------------------------------------------
-- ALLOW LEGITIMATE TELEPORT
--------------------------------------------------

local function allowTeleport(player, duration)
    if not player then
        return
    end

    exemptPlayer(
        player,
        duration or 5
    )

    local state = getState(player)

    if state then
        local character =
            player.Character

        local root =
            character
            and character:FindFirstChild(
                "HumanoidRootPart"
            )

        if root then
            state.LastPosition =
                root.Position
        end
    end
end

--------------------------------------------------
-- MOVEMENT INFORMATION
--------------------------------------------------

local function getCharacterInfo(player)

    local character =
        player.Character

    if not character then
        return nil
    end

    local humanoid =
        character:FindFirstChildOfClass(
            "Humanoid"
        )

    local root =
        character:FindFirstChild(
            "HumanoidRootPart"
        )

    if not humanoid or not root then
        return nil
    end

    return character, humanoid, root
end

--------------------------------------------------
-- SPEED CHECK
--------------------------------------------------

local function validateWalkSpeed(
    player,
    humanoid
)

    if isExempt(player) then
        return
    end

    if humanoid.WalkSpeed >
        CONFIG.Detection.MaxWalkSpeed
    then

        addStrike(
            player,
            "Abnormal WalkSpeed: "
            .. tostring(
                math.floor(
                    humanoid.WalkSpeed
                )
            )
        )

        humanoid.WalkSpeed =
            CONFIG.Detection.MaxWalkSpeed

        debugPrint(
            player.Name,
            "WalkSpeed corrected"
        )
    end
end

--------------------------------------------------
-- JUMP CHECK
--------------------------------------------------

local function validateJump(
    player,
    humanoid
)

    if isExempt(player) then
        return
    end

    if humanoid.UseJumpPower then

        if humanoid.JumpPower >
            CONFIG.Detection.MaxJumpPower
        then

            addStrike(
                player,
                "Abnormal JumpPower: "
                .. tostring(
                    math.floor(
                        humanoid.JumpPower
                    )
                )
            )

            humanoid.JumpPower =
                CONFIG.Detection.MaxJumpPower
        end

    else

        local maxJumpHeight = 15

        if humanoid.JumpHeight >
            maxJumpHeight
        then

            addStrike(
                player,
                "Abnormal JumpHeight: "
                .. tostring(
                    math.floor(
                        humanoid.JumpHeight
                    )
                )
            )

            humanoid.JumpHeight =
                maxJumpHeight
        end
    end
end

--------------------------------------------------
-- SERVER MOVEMENT CHECK
--------------------------------------------------

local function validateMovement(
    player,
    root,
    humanoid
)

    local state =
        getState(player)

    if not state then
        return
    end

    local now =
        os.clock()

    local previous =
        state.LastPosition

    state.LastPosition =
        root.Position

    if not previous then
        return
    end

    if isExempt(player) then
        return
    end

    ------------------------------------------------
    -- SPAWN GRACE
    ------------------------------------------------

    if now - state.SpawnTime
        < CONFIG.Detection.SpawnGraceTime
    then
        return
    end

    ------------------------------------------------
    -- DELTA TIME
    ------------------------------------------------

    local dt =
        now - state.LastCheck

    state.LastCheck = now

    if dt <= 0 then
        return
    end

    if dt > 1 then
        return
    end

    ------------------------------------------------
    -- DISTANCE
    ------------------------------------------------

    local distance =
        (
            root.Position
            - previous
        ).Magnitude

    ------------------------------------------------
    -- EXPECTED MOVEMENT
    ------------------------------------------------

    local walkSpeed =
        math.max(
            humanoid.WalkSpeed,
            0
        )

    local expected =
        walkSpeed * dt

    ------------------------------------------------
    -- EXTRA PHYSICS TOLERANCE
    ------------------------------------------------

    local tolerance =
        math.max(
            8,
            expected * 1.5
        )

    local maximum =
        math.max(
            tolerance,
            CONFIG.Detection.MaxMovementSpeed
            * dt
            + 10
        )

    ------------------------------------------------
    -- TELEPORT DISTANCE
    ------------------------------------------------

    if distance >
        CONFIG.Detection.TeleportDistance
    then

        addStrike(
            player,
            "Abnormal teleport distance: "
            .. tostring(
                math.floor(distance)
            )
        )

        root.CFrame =
            CFrame.new(previous)

        return
    end

    ------------------------------------------------
    -- HIGH SPEED MOVEMENT
    ------------------------------------------------

    if distance > maximum then

        addStrike(
            player,
            "Abnormal movement speed: "
            .. tostring(
                math.floor(
                    distance / dt
                )
            )
        )

        root.CFrame =
            CFrame.new(previous)

        return
    end
end

--------------------------------------------------
-- RAYCAST PARAMETERS
--------------------------------------------------

local function createRaycastParams(
    character
)

    local params =
        RaycastParams.new()

    params.FilterType =
        Enum.RaycastFilterType.Exclude

    params.FilterDescendantsInstances = {
        character
    }

    params.IgnoreWater = true

    return params
end

--------------------------------------------------
-- NOCLIP CHECK
--------------------------------------------------

local function validateNoclip(
    player,
    character,
    root,
    previousPosition
)

    if isExempt(player) then
        return
    end

    if not previousPosition then
        return
    end

    local movement =
        root.Position
        - previousPosition

    local distance =
        movement.Magnitude

    if distance < 3 then
        return
    end

    local direction =
        movement.Unit

    local params =
        createRaycastParams(
            character
        )

    local result =
        workspace:Raycast(
            previousPosition,
            direction * distance,
            params
        )

    if not result then
        return
    end

    local hit =
        result.Instance

    if not hit then
        return
    end

    ------------------------------------------------
    -- IGNORE NON-COLLIDABLE OBJECTS
    ------------------------------------------------

    if not hit.CanCollide then
        return
    end

    ------------------------------------------------
    -- IGNORE TRANSPARENT EFFECTS
    ------------------------------------------------

    if hit.Transparency >= 0.95 then
        return
    end

    ------------------------------------------------
    -- POSSIBLE NOCLIP
    ------------------------------------------------

    addStrike(
        player,
        "Possible noclip: "
        .. hit:GetFullName()
    )

end

--------------------------------------------------
-- HUMANOID STATE CHECK
--------------------------------------------------

local function validateHumanoidState(
    player,
    humanoid
)

    if isExempt(player) then
        return
    end

    local state =
        humanoid:GetState()

    ------------------------------------------------
    -- SEATED IS LEGITIMATE
    ------------------------------------------------

    if state ==
        Enum.HumanoidStateType.Seated
    then
        return
    end

    ------------------------------------------------
    -- SWIMMING IS LEGITIMATE
    ------------------------------------------------

    if state ==
        Enum.HumanoidStateType.Swimming
    then
        return
    end

    ------------------------------------------------
    -- CLIMBING IS LEGITIMATE
    ------------------------------------------------

    if state ==
        Enum.HumanoidStateType.Climbing
    then
        return
    end

end

--------------------------------------------------
-- MOVEMENT HEARTBEAT
--------------------------------------------------

local movementAccumulator = 0

RunService.Heartbeat:Connect(
    function(deltaTime)

        movementAccumulator +=
            deltaTime

        if movementAccumulator <
            CONFIG.Detection.CheckInterval
        then
            return
        end

        movementAccumulator = 0

        for _, player in ipairs(
            Players:GetPlayers()
        ) do

            if isAdmin(player) then
                continue
            end

            local character,
                humanoid,
                root =
                getCharacterInfo(player)

            if not character
                or not humanoid
                or not root
            then
                continue
            end

            local state =
                getState(player)

            if not state then
                continue
            end

            local previousPosition =
                state.LastPosition

            ------------------------------------------------
            -- BASIC CHECKS
            ------------------------------------------------

            validateWalkSpeed(
                player,
                humanoid
            )

            validateJump(
                player,
                humanoid
            )

            validateHumanoidState(
                player,
                humanoid
            )

            ------------------------------------------------
            -- NOCLIP FIRST
            ------------------------------------------------

            validateNoclip(
                player,
                character,
                root,
                previousPosition
            )

            ------------------------------------------------
            -- MOVEMENT
            ------------------------------------------------

            validateMovement(
                player,
                root,
                humanoid
            )

        end
    end
)

--------------------------------------------------
-- SERVER TELEPORT API
--------------------------------------------------

local TeleportEvent =
    RootFolder:FindFirstChild(
        "AllowTeleport"
    )

if not TeleportEvent then

    TeleportEvent =
        Instance.new("BindableEvent")

    TeleportEvent.Name =
        "AllowTeleport"

    TeleportEvent.Parent =
        RootFolder
end

TeleportEvent.Event:Connect(
    function(player, duration)

        if typeof(player) ~= "Instance"
            or not player:IsA("Player")
        then
            return
        end

        allowTeleport(
            player,
            duration
        )

    end
)

--------------------------------------------------
-- SERVER EXEMPTION API
--------------------------------------------------

local ExemptEvent =
    RootFolder:FindFirstChild(
        "ExemptPlayer"
    )

if not ExemptEvent then

    ExemptEvent =
        Instance.new("BindableEvent")

    ExemptEvent.Name =
        "ExemptPlayer"

    ExemptEvent.Parent =
        RootFolder
end

ExemptEvent.Event:Connect(
    function(player, duration)

        if typeof(player) ~= "Instance"
            or not player:IsA("Player")
        then
            return
        end

        exemptPlayer(
            player,
            duration
        )

    end
)

--------------------------------------------------
-- MOVEMENT API READY
--------------------------------------------------

debugPrint(
    "Movement protection enabled."
)

print(
    "[Kianbest Anti-Cheat] V3 Part 2 loaded."
)
--------------------------------------------------
-- KIANBEST ANTI-CHEAT V3
-- PART 3/5
-- REMOTE SECURITY
--------------------------------------------------

local RemoteState = {}

local RemoteConfig = {
    DefaultLimit = 15,
    DefaultWindow = 1,

    MaxStringLength = 500,
    MaxTableDepth = 5,
    MaxTableItems = 100,

    KickOnExtremeSpam = false,
}

--------------------------------------------------
-- REMOTE IDENTIFIER
--------------------------------------------------

local function getRemoteId(remote)

    if not remote then
        return "Unknown"
    end

    return remote:GetFullName()

end

--------------------------------------------------
-- PLAYER REMOTE STATE
--------------------------------------------------

local function getRemotePlayerState(
    player
)

    if not RemoteState[player] then

        RemoteState[player] = {}

    end

    return RemoteState[player]

end

--------------------------------------------------
-- RESET REMOTE STATE
--------------------------------------------------

local function resetRemoteState(
    player
)

    RemoteState[player] = nil

end

--------------------------------------------------
-- CLEAN REMOTE HISTORY
--------------------------------------------------

local function cleanRemoteHistory(
    data,
    now,
    window
)

    local timestamps =
        data.Timestamps

    if not timestamps then
        data.Timestamps = {}
        return
    end

    local newList = {}

    for _, timestamp in ipairs(
        timestamps
    ) do

        if now - timestamp <= window then

            table.insert(
                newList,
                timestamp
            )

        end
    end

    data.Timestamps =
        newList

end

--------------------------------------------------
-- REMOTE RATE CHECK
--------------------------------------------------

local function checkRemoteRate(
    player,
    remote,
    limit,
    window
)

    if isAdmin(player) then
        return true
    end

    local playerData =
        getRemotePlayerState(
            player
        )

    local id =
        getRemoteId(remote)

    local data =
        playerData[id]

    if not data then

        data = {
            Timestamps = {},
            Violations = 0,
        }

        playerData[id] = data

    end

    local now =
        os.clock()

    cleanRemoteHistory(
        data,
        now,
        window
    )

    table.insert(
        data.Timestamps,
        now
    )

    if #data.Timestamps <= limit then
        return true
    end

    data.Violations += 1

    debugPrint(
        player.Name,
        "Remote spam:",
        id,
        #data.Timestamps,
        "/",
        limit
    )

    ------------------------------------------------
    -- REMOVE OLD REQUEST
    ------------------------------------------------

    while #data.Timestamps > limit do

        table.remove(
            data.Timestamps,
            1
        )

    end

    ------------------------------------------------
    -- STRIKE
    ------------------------------------------------

    addStrike(
        player,
        "Remote spam: "
        .. remote.Name
    )

    ------------------------------------------------
    -- OPTIONAL EXTREME ACTION
    ------------------------------------------------

    if RemoteConfig.KickOnExtremeSpam
        and data.Violations >= 10
    then

        banPlayer(
            player,
            "Extreme remote spam"
        )

        return false
    end

    return false

end

--------------------------------------------------
-- REMOTE ARGUMENT VALIDATOR
--------------------------------------------------

local function validateValue(
    value,
    depth
)

    depth =
        depth or 0

    if depth >
        RemoteConfig.MaxTableDepth
    then
        return false,
            "table depth too large"
    end

    local valueType =
        typeof(value)

    ------------------------------------------------
    -- SAFE PRIMITIVES
    ------------------------------------------------

    if valueType == "nil" then
        return true
    end

    if valueType == "boolean" then
        return true
    end

    if valueType == "number" then

        if value ~= value then
            return false,
                "NaN detected"
        end

        if value == math.huge
            or value == -math.huge
        then
            return false,
                "infinite number"
        end

        return true
    end

    ------------------------------------------------
    -- STRING
    ------------------------------------------------

    if valueType == "string" then

        if #value >
            RemoteConfig.MaxStringLength
        then

            return false,
                "string too long"
        end

        return true
    end

    ------------------------------------------------
    -- INSTANCE
    ------------------------------------------------

    if valueType == "Instance" then

        if not value.Parent then

            return false,
                "destroyed instance"
        end

        return true
    end

    ------------------------------------------------
    -- VECTOR
    ------------------------------------------------

    if valueType == "Vector3" then

        if value.X ~= value.X
            or value.Y ~= value.Y
            or value.Z ~= value.Z
        then

            return false,
                "invalid Vector3"
        end

        return true
    end

    ------------------------------------------------
    -- CFRAME
    ------------------------------------------------

    if valueType == "CFrame" then

        local position =
            value.Position

        if position.X ~= position.X
            or position.Y ~= position.Y
            or position.Z ~= position.Z
        then

            return false,
                "invalid CFrame"
        end

        return true
    end

    ------------------------------------------------
    -- TABLE
    ------------------------------------------------

    if valueType == "table" then

        local count = 0

        for key, child in pairs(value) do

            count += 1

            if count >
                RemoteConfig.MaxTableItems
            then

                return false,
                    "table too large"
            end

            local keyValid =
                validateValue(
                    key,
                    depth + 1
                )

            if not keyValid then

                return false,
                    "invalid table key"
            end

            local childValid,
                childReason =
                validateValue(
                    child,
                    depth + 1
                )

            if not childValid then

                return false,
                    childReason
            end

        end

        return true
    end

    ------------------------------------------------
    -- UNSUPPORTED TYPE
    ------------------------------------------------

    return false,
        "unsupported value type: "
        .. tostring(valueType)

end

--------------------------------------------------
-- VALIDATE REMOTE ARGUMENTS
--------------------------------------------------

local function validateArguments(
    player,
    remote,
    arguments
)

    for index, value in ipairs(
        arguments
    ) do

        local valid,
            reason =
            validateValue(
                value,
                0
            )

        if not valid then

            addStrike(
                player,
                "Invalid remote argument: "
                .. remote.Name
                .. " ["
                .. tostring(index)
                .. "] "
                .. tostring(reason)
            )

            return false
        end
    end

    return true

end

--------------------------------------------------
-- PROTECT REMOTE EVENT
--------------------------------------------------

local ProtectedRemotes = {}

local function registerRemote(
    remote,
    limit,
    window
)

    if not remote then
        return
    end

    if not (
        remote:IsA("RemoteEvent")
        or remote:IsA("RemoteFunction")
    ) then

        return
    end

    ProtectedRemotes[remote] = {
        Limit =
            tonumber(limit)
            or RemoteConfig.DefaultLimit,

        Window =
            tonumber(window)
            or RemoteConfig.DefaultWindow,
    }

    debugPrint(
        "Protected remote:",
        remote:GetFullName()
    )

end

--------------------------------------------------
-- AUTOMATIC REMOTE DISCOVERY
--------------------------------------------------

local function scanRemote(
    instance
)

    if not (
        instance:IsA("RemoteEvent")
        or instance:IsA("RemoteFunction")
    ) then

        return
    end

    ------------------------------------------------
    -- IMPORTANT
    -- We only register the remote here.
    -- We do NOT replace OnServerInvoke.
    ------------------------------------------------

    registerRemote(
        instance,
        RemoteConfig.DefaultLimit,
        RemoteConfig.DefaultWindow
    )

end

for _, instance in ipairs(
    game:GetDescendants()
) do

    scanRemote(instance)

end

--------------------------------------------------
-- NEW REMOTE DETECTION
--------------------------------------------------

game.DescendantAdded:Connect(
    function(instance)

        scanRemote(instance)

    end
)

--------------------------------------------------
-- SERVER VALIDATION API
--------------------------------------------------

local ValidateRemote =
    RootFolder:FindFirstChild(
        "ValidateRemote"
    )

if not ValidateRemote then

    ValidateRemote =
        Instance.new("BindableFunction")

    ValidateRemote.Name =
        "ValidateRemote"

    ValidateRemote.Parent =
        RootFolder

end

ValidateRemote.OnInvoke =
    function(
        player,
        remote,
        ...
    )

        if typeof(player) ~= "Instance"
            or not player:IsA("Player")
        then

            return false
        end

        if typeof(remote) ~= "Instance"
            or not (
                remote:IsA("RemoteEvent")
                or remote:IsA("RemoteFunction")
            )
        then

            return false
        end

        local config =
            ProtectedRemotes[remote]

        if not config then

            config = {
                Limit =
                    RemoteConfig.DefaultLimit,

                Window =
                    RemoteConfig.DefaultWindow,
            }

        end

        local rateOK =
            checkRemoteRate(
                player,
                remote,
                config.Limit,
                config.Window
            )

        if not rateOK then
            return false
        end

        local args = {
            ...
        }

        local argsOK =
            validateArguments(
                player,
                remote,
                args
            )

        if not argsOK then
            return false
        end

        return true

    end

--------------------------------------------------
-- REMOTE REGISTRATION API
--------------------------------------------------

local ProtectRemote =
    RootFolder:FindFirstChild(
        "ProtectRemote"
    )

if not ProtectRemote then

    ProtectRemote =
        Instance.new("BindableFunction")

    ProtectRemote.Name =
        "ProtectRemote"

    ProtectRemote.Parent =
        RootFolder

end

ProtectRemote.OnInvoke =
    function(
        remote,
        limit,
        window
    )

        if typeof(remote) ~= "Instance" then
            return false
        end

        if not (
            remote:IsA("RemoteEvent")
            or remote:IsA("RemoteFunction")
        ) then

            return false
        end

        registerRemote(
            remote,
            limit,
            window
        )

        return true

    end

--------------------------------------------------
-- PLAYER CLEANUP
--------------------------------------------------

Players.PlayerRemoving:Connect(
    function(player)

        resetRemoteState(player)

    end
)

--------------------------------------------------
-- REMOTE SECURITY READY
--------------------------------------------------

print(
    "[Kianbest Anti-Cheat] "
    .. "V3 Part 3 loaded."
)
--------------------------------------------------
-- KIANBEST ANTI-CHEAT V3
-- PART 4/5
-- COMBAT VALIDATION
--------------------------------------------------

local CombatConfig = {

    MaxAttackDistance = 35,

    MaxProjectileDistance = 500,

    DefaultCooldown = 0.15,

    RequireAliveAttacker = true,

    RequireAliveTarget = true,

    RequireLineOfSight = false,

    IgnoreForceField = true,

    MaxTargetsPerRequest = 10,

    Debug = false,
}

--------------------------------------------------
-- COMBAT STATE
--------------------------------------------------

local CombatState = {}

local function getCombatState(player)

    if not CombatState[player] then

        CombatState[player] = {
            Cooldowns = {},
            LastTarget = nil,
            LastAttack = 0,
        }

    end

    return CombatState[player]

end

--------------------------------------------------
-- CLEAN COMBAT STATE
--------------------------------------------------

Players.PlayerRemoving:Connect(
    function(player)

        CombatState[player] = nil

    end
)

--------------------------------------------------
-- CHARACTER VALIDATION
--------------------------------------------------

local function getLivingCharacter(
    player
)

    if not player then
        return nil
    end

    local character =
        player.Character

    if not character then
        return nil
    end

    local humanoid =
        character:FindFirstChildOfClass(
            "Humanoid"
        )

    local root =
        character:FindFirstChild(
            "HumanoidRootPart"
        )

    if not humanoid or not root then
        return nil
    end

    if humanoid.Health <= 0 then
        return nil
    end

    return character,
        humanoid,
        root

end

--------------------------------------------------
-- TARGET CHARACTER VALIDATION
--------------------------------------------------

local function getTargetCharacter(
    target
)

    if not target then
        return nil
    end

    local character

    if target:IsA("Player") then

        character =
            target.Character

    elseif target:IsA("Model") then

        character = target

    elseif target:IsA("BasePart") then

        character =
            target:FindFirstAncestorOfClass(
                "Model"
            )

    else

        return nil

    end

    if not character then
        return nil
    end

    local humanoid =
        character:FindFirstChildOfClass(
            "Humanoid"
        )

    local root =
        character:FindFirstChild(
            "HumanoidRootPart"
        )

    if not humanoid or not root then
        return nil
    end

    if humanoid.Health <= 0 then
        return nil
    end

    return character,
        humanoid,
        root

end

--------------------------------------------------
-- DISTANCE CHECK
--------------------------------------------------

local function isWithinRange(
    attackerRoot,
    targetRoot,
    maxDistance
)

    if not attackerRoot
        or not targetRoot
    then

        return false

    end

    local distance =
        (
            attackerRoot.Position
            - targetRoot.Position
        ).Magnitude

    return distance <= maxDistance

end

--------------------------------------------------
-- LINE OF SIGHT
--------------------------------------------------

local function hasLineOfSight(
    attackerCharacter,
    attackerRoot,
    targetCharacter,
    targetRoot
)

    local direction =
        targetRoot.Position
        - attackerRoot.Position

    local distance =
        direction.Magnitude

    if distance <= 0 then
        return true
    end

    local params =
        RaycastParams.new()

    params.FilterType =
        Enum.RaycastFilterType.Exclude

    params.FilterDescendantsInstances = {
        attackerCharacter,
        targetCharacter,
    }

    params.IgnoreWater = true

    local result =
        workspace:Raycast(
            attackerRoot.Position,
            direction,
            params
        )

    if not result then
        return true
    end

    local hit =
        result.Instance

    if not hit then
        return true
    end

    return hit:IsDescendantOf(
        targetCharacter
    )

end

--------------------------------------------------
-- TARGET IS ENEMY
--------------------------------------------------

local function isValidEnemy(
    attacker,
    targetPlayer
)

    if not targetPlayer then
        return false
    end

    if attacker == targetPlayer then
        return false
    end

    ------------------------------------------------
    -- TEAM CHECK
    ------------------------------------------------

    if attacker.Team
        and targetPlayer.Team
        and attacker.Team ==
            targetPlayer.Team
    then

        return false

    end

    return true

end

--------------------------------------------------
-- COOLDOWN CHECK
--------------------------------------------------

local function checkCombatCooldown(
    player,
    action,
    cooldown
)

    local state =
        getCombatState(player)

    local now =
        os.clock()

    local last =
        state.Cooldowns[action]

    if last
        and now - last < cooldown
    then

        addStrike(
            player,
            "Combat cooldown violation: "
            .. tostring(action)
        )

        return false

    end

    state.Cooldowns[action] =
        now

    state.LastAttack =
        now

    return true

end

--------------------------------------------------
-- FORCEFIELD CHECK
--------------------------------------------------

local function hasForceField(
    character
)

    if not character then
        return false
    end

    return character:FindFirstChild(
        "ForceField"
    ) ~= nil

end

--------------------------------------------------
-- VALIDATE ATTACK
--------------------------------------------------

local function validateAttack(
    player,
    target,
    options
)

    options =
        options or {}

    local attackerCharacter,
        attackerHumanoid,
        attackerRoot =
        getLivingCharacter(
            player
        )

    if CombatConfig.RequireAliveAttacker
        and not attackerCharacter
    then

        return false,
            "attacker_not_alive"

    end

    if not attackerCharacter then
        return false,
            "invalid_attacker"
    end

    ------------------------------------------------
    -- TARGET
    ------------------------------------------------

    local targetCharacter,
        targetHumanoid,
        targetRoot =
        getTargetCharacter(
            target
        )

    if CombatConfig.RequireAliveTarget
        and not targetCharacter
    then

        return false,
            "target_not_alive"

    end

    if not targetCharacter then
        return false,
            "invalid_target"
    end

    ------------------------------------------------
    -- TARGET PLAYER
    ------------------------------------------------

    local targetPlayer =
        Players:GetPlayerFromCharacter(
            targetCharacter
        )

    if targetPlayer then

        if not isValidEnemy(
            player,
            targetPlayer
        ) then

            return false,
                "invalid_enemy"

        end

    end

    ------------------------------------------------
    -- FORCEFIELD
    ------------------------------------------------

    if CombatConfig.IgnoreForceField
        and hasForceField(
            targetCharacter
        )
    then

        return false,
            "target_protected"

    end

    ------------------------------------------------
    -- RANGE
    ------------------------------------------------

    local maxDistance =
        tonumber(
            options.MaxDistance
        )
        or CombatConfig.MaxAttackDistance

    if not isWithinRange(
        attackerRoot,
        targetRoot,
        maxDistance
    ) then

        addStrike(
            player,
            "Attack range violation"
        )

        return false,
            "target_too_far"

    end

    ------------------------------------------------
    -- LINE OF SIGHT
    ------------------------------------------------

    local requireLOS =
        options.RequireLineOfSight

    if requireLOS == nil then

        requireLOS =
            CombatConfig.RequireLineOfSight

    end

    if requireLOS then

        if not hasLineOfSight(
            attackerCharacter,
            attackerRoot,
            targetCharacter,
            targetRoot
        ) then

            return false,
                "no_line_of_sight"

        end

    end

    ------------------------------------------------
    -- COOLDOWN
    ------------------------------------------------

    local action =
        tostring(
            options.Action
            or "DefaultAttack"
        )

    local cooldown =
        tonumber(
            options.Cooldown
        )
        or CombatConfig.DefaultCooldown

    if not checkCombatCooldown(
        player,
        action,
        cooldown
    ) then

        return false,
            "cooldown"

    end

    ------------------------------------------------
    -- SAVE TARGET
    ------------------------------------------------

    local state =
        getCombatState(player)

    state.LastTarget =
        targetCharacter

    ------------------------------------------------
    -- SUCCESS
    ------------------------------------------------

    return true,
        "valid"

end

--------------------------------------------------
-- VALIDATE TARGET ONLY
--------------------------------------------------

local function validateTarget(
    player,
    target,
    maxDistance
)

    local attackerCharacter,
        attackerHumanoid,
        attackerRoot =
        getLivingCharacter(
            player
        )

    if not attackerCharacter then
        return false
    end

    local targetCharacter,
        targetHumanoid,
        targetRoot =
        getTargetCharacter(
            target
        )

    if not targetCharacter then
        return false
    end

    if not isWithinRange(
        attackerRoot,
        targetRoot,
        maxDistance
            or CombatConfig.MaxAttackDistance
    ) then

        return false

    end

    return true

end

--------------------------------------------------
-- DAMAGE LIMIT
--------------------------------------------------

local function validateDamage(
    player,
    target,
    damage,
    maxDamage
)

    if typeof(damage) ~= "number" then
        return false,
            "invalid_damage"
    end

    if damage ~= damage then
        return false,
            "invalid_damage"
    end

    if damage <= 0 then
        return false,
            "invalid_damage"
    end

    maxDamage =
        tonumber(maxDamage)
        or 1000

    if damage > maxDamage then

        addStrike(
            player,
            "Abnormal damage: "
            .. tostring(damage)
        )

        return false,
            "damage_too_high"

    end

    local targetCharacter,
        targetHumanoid =
        getTargetCharacter(
            target
        )

    if not targetCharacter
        or not targetHumanoid
    then

        return false,
            "invalid_target"

    end

    return true,
        "valid"

end

--------------------------------------------------
-- COMBAT API
--------------------------------------------------

local ValidateAttack =
    RootFolder:FindFirstChild(
        "ValidateAttack"
    )

if not ValidateAttack then

    ValidateAttack =
        Instance.new("BindableFunction")

    ValidateAttack.Name =
        "ValidateAttack"

    ValidateAttack.Parent =
        RootFolder

end

ValidateAttack.OnInvoke =
    function(
        player,
        target,
        options
    )

        return validateAttack(
            player,
            target,
            options
        )

    end

--------------------------------------------------
-- TARGET API
--------------------------------------------------

local ValidateTarget =
    RootFolder:FindFirstChild(
        "ValidateTarget"
    )

if not ValidateTarget then

    ValidateTarget =
        Instance.new("BindableFunction")

    ValidateTarget.Name =
        "ValidateTarget"

    ValidateTarget.Parent =
        RootFolder

end

ValidateTarget.OnInvoke =
    function(
        player,
        target,
        maxDistance
    )

        return validateTarget(
            player,
            target,
            maxDistance
        )

    end

--------------------------------------------------
-- DAMAGE API
--------------------------------------------------

local ValidateDamage =
    RootFolder:FindFirstChild(
        "ValidateDamage"
    )

if not ValidateDamage then

    ValidateDamage =
        Instance.new("BindableFunction")

    ValidateDamage.Name =
        "ValidateDamage"

    ValidateDamage.Parent =
        RootFolder

end

ValidateDamage.OnInvoke =
    function(
        player,
        target,
        damage,
        maxDamage
    )

        return validateDamage(
            player,
            target,
            damage,
            maxDamage
        )

    end

--------------------------------------------------
-- LINE OF SIGHT API
--------------------------------------------------

local LineOfSight =
    RootFolder:FindFirstChild(
        "HasLineOfSight"
    )

if not LineOfSight then

    LineOfSight =
        Instance.new("BindableFunction")

    LineOfSight.Name =
        "HasLineOfSight"

    LineOfSight.Parent =
        RootFolder

end

LineOfSight.OnInvoke =
    function(
        player,
        target
    )

        local attackerCharacter,
            attackerHumanoid,
            attackerRoot =
            getLivingCharacter(
                player
            )

        if not attackerCharacter then
            return false
        end

        local targetCharacter,
            targetHumanoid,
            targetRoot =
            getTargetCharacter(
                target
            )

        if not targetCharacter then
            return false
        end

        return hasLineOfSight(
            attackerCharacter,
            attackerRoot,
            targetCharacter,
            targetRoot
        )

    end

--------------------------------------------------
-- COMBAT DEBUG
--------------------------------------------------

if CombatConfig.Debug then

    print(
        "[Kianbest Anti-Cheat] "
        .. "Combat validation enabled."
    )

end

print(
    "[Kianbest Anti-Cheat] "
    .. "V3 Part 4 loaded."
)
--------------------------------------------------
-- KIANBEST ANTI-CHEAT V3
-- PART 5/5
-- ADMIN / BAN / STATUS / FINALIZATION
--------------------------------------------------

--------------------------------------------------
-- ADMIN BAN API
--------------------------------------------------

local AdminBan =
    RootFolder:FindFirstChild(
        "AdminBan"
    )

if not AdminBan then

    AdminBan =
        Instance.new("BindableFunction")

    AdminBan.Name =
        "AdminBan"

    AdminBan.Parent =
        RootFolder

end

AdminBan.OnInvoke =
    function(
        admin,
        target,
        reason
    )

        ------------------------------------------------
        -- CHECK ADMIN
        ------------------------------------------------

        if not admin
            or not admin:IsA("Player")
        then

            return false,
                "invalid_admin"

        end

        if not isAdmin(admin) then

            return false,
                "not_admin"

        end

        ------------------------------------------------
        -- FIND TARGET
        ------------------------------------------------

        local targetPlayer = nil

        if typeof(target) == "Instance"
            and target:IsA("Player")
        then

            targetPlayer = target

        elseif typeof(target) == "number" then

            targetPlayer =
                Players:GetPlayerByUserId(
                    target
                )

        elseif typeof(target) == "string" then

            for _, player in ipairs(
                Players:GetPlayers()
            ) do

                if string.lower(
                    player.Name
                ) == string.lower(
                    target
                ) then

                    targetPlayer =
                        player

                    break

                end

            end

        end

        if not targetPlayer then

            return false,
                "target_not_found"

        end

        ------------------------------------------------
        -- PROTECT OTHER ADMINS
        ------------------------------------------------

        if isAdmin(targetPlayer) then

            return false,
                "target_is_admin"

        end

        ------------------------------------------------
        -- BAN
        ------------------------------------------------

        banPlayer(
            targetPlayer,
            reason
                or "Admin ban"
        )

        return true,
            "banned"

    end

--------------------------------------------------
-- ADMIN UNBAN API
--------------------------------------------------

local AdminUnban =
    RootFolder:FindFirstChild(
        "AdminUnban"
    )

if not AdminUnban then

    AdminUnban =
        Instance.new("BindableFunction")

    AdminUnban.Name =
        "AdminUnban"

    AdminUnban.Parent =
        RootFolder

end

AdminUnban.OnInvoke =
    function(
        admin,
        userId
    )

        ------------------------------------------------
        -- CHECK ADMIN
        ------------------------------------------------

        if not admin
            or not admin:IsA("Player")
        then

            return false,
                "invalid_admin"

        end

        if not isAdmin(admin) then

            return false,
                "not_admin"

        end

        ------------------------------------------------
        -- USER ID
        ------------------------------------------------

        userId =
            tonumber(userId)

        if not userId then

            return false,
                "invalid_user_id"

        end

        ------------------------------------------------
        -- REMOVE BAN
        ------------------------------------------------

        local success,
            errorMessage =
            pcall(function()

                BanStore:RemoveAsync(
                    getBanKey(userId)
                )

            end)

        if not success then

            warn(
                "[Kianbest Anti-Cheat] "
                .. "Unban failed:",
                errorMessage
            )

            return false,
                "datastore_error"

        end

        return true,
            "unbanned"

    end

--------------------------------------------------
-- MANUAL BAN EVENT
--------------------------------------------------

local BanEvent =
    RootFolder:FindFirstChild(
        "BanPlayer"
    )

if not BanEvent then

    BanEvent =
        Instance.new("BindableEvent")

    BanEvent.Name =
        "BanPlayer"

    BanEvent.Parent =
        RootFolder

end

BanEvent.Event:Connect(
    function(
        player,
        reason
    )

        if typeof(player) ~= "Instance"
            or not player:IsA("Player")
        then

            return

        end

        banPlayer(
            player,
            reason
                or "Manual ban"
        )

    end
)

--------------------------------------------------
-- MANUAL UNBAN EVENT
--------------------------------------------------

local UnbanEvent =
    RootFolder:FindFirstChild(
        "UnbanPlayer"
    )

if not UnbanEvent then

    UnbanEvent =
        Instance.new("BindableEvent")

    UnbanEvent.Name =
        "UnbanPlayer"

    UnbanEvent.Parent =
        RootFolder

end

UnbanEvent.Event:Connect(
    function(userId)

        userId =
            tonumber(userId)

        if not userId then
            return
        end

        pcall(function()

            BanStore:RemoveAsync(
                getBanKey(userId)
            )

        end)

    end
)

--------------------------------------------------
-- RESET STRIKES
--------------------------------------------------

local ResetStrikes =
    RootFolder:FindFirstChild(
        "ResetStrikes"
    )

if not ResetStrikes then

    ResetStrikes =
        Instance.new("BindableFunction")

    ResetStrikes.Name =
        "ResetStrikes"

    ResetStrikes.Parent =
        RootFolder

end

ResetStrikes.OnInvoke =
    function(
        admin,
        target
    )

        if not admin
            or not admin:IsA("Player")
        then

            return false,
                "invalid_admin"

        end

        if not isAdmin(admin) then

            return false,
                "not_admin"

        end

        local targetPlayer = nil

        if typeof(target) == "Instance"
            and target:IsA("Player")
        then

            targetPlayer = target

        elseif typeof(target) == "number" then

            targetPlayer =
                Players:GetPlayerByUserId(
                    target
                )

        end

        if not targetPlayer then

            return false,
                "target_not_found"

        end

        local state =
            getState(targetPlayer)

        if not state then

            return false,
                "state_not_found"

        end

        state.Strikes = 0
        state.LastStrikeReason = ""
        state.LastStrikeTime = 0

        return true,
            "reset"

    end

--------------------------------------------------
-- GET PLAYER STATUS
--------------------------------------------------

local GetStatus =
    RootFolder:FindFirstChild(
        "GetStatus"
    )

if not GetStatus then

    GetStatus =
        Instance.new("BindableFunction")

    GetStatus.Name =
        "GetStatus"

    GetStatus.Parent =
        RootFolder

end

GetStatus.OnInvoke =
    function(
        admin,
        target
    )

        if not admin
            or not admin:IsA("Player")
        then

            return nil,
                "invalid_admin"

        end

        if not isAdmin(admin) then

            return nil,
                "not_admin"

        end

        local targetPlayer = nil

        if typeof(target) == "Instance"
            and target:IsA("Player")
        then

            targetPlayer = target

        elseif typeof(target) == "number" then

            targetPlayer =
                Players:GetPlayerByUserId(
                    target
                )

        end

        if not targetPlayer then

            return nil,
                "target_not_found"

        end

        local state =
            getState(targetPlayer)

        if not state then

            return nil,
                "state_not_found"

        end

        return {
            UserId =
                targetPlayer.UserId,

            Name =
                targetPlayer.Name,

            DisplayName =
                targetPlayer.DisplayName,

            Strikes =
                state.Strikes,

            LastStrikeReason =
                state.LastStrikeReason,

            LastStrikeTime =
                state.LastStrikeTime,

            ExemptUntil =
                state.ExemptUntil,

            IsBanned =
                state.IsBanned,

        }

    end

--------------------------------------------------
-- IS ADMIN API
--------------------------------------------------

local IsAdminFunction =
    RootFolder:FindFirstChild(
        "IsAdmin"
    )

if not IsAdminFunction then

    IsAdminFunction =
        Instance.new("BindableFunction")

    IsAdminFunction.Name =
        "IsAdmin"

    IsAdminFunction.Parent =
        RootFolder

end

IsAdminFunction.OnInvoke =
    function(player)

        if typeof(player) ~= "Instance"
            or not player:IsA("Player")
        then

            return false

        end

        return isAdmin(player)

    end

--------------------------------------------------
-- CONFIG API
--------------------------------------------------

local ConfigFolder =
    RootFolder:FindFirstChild(
        "Config"
    )

if not ConfigFolder then

    ConfigFolder =
        Instance.new("Folder")

    ConfigFolder.Name =
        "Config"

    ConfigFolder.Parent =
        RootFolder

end

--------------------------------------------------
-- CONFIG VALUES
--------------------------------------------------

local DetectionEnabled =
    ConfigFolder:FindFirstChild(
        "DetectionEnabled"
    )

if not DetectionEnabled then

    DetectionEnabled =
        Instance.new("BoolValue")

    DetectionEnabled.Name =
        "DetectionEnabled"

    DetectionEnabled.Value =
        CONFIG.Detection.Enabled

    DetectionEnabled.Parent =
        ConfigFolder

end

--------------------------------------------------
-- STRIKE LIMIT
--------------------------------------------------

local StrikeLimit =
    ConfigFolder:FindFirstChild(
        "StrikeLimit"
    )

if not StrikeLimit then

    StrikeLimit =
        Instance.new("IntValue")

    StrikeLimit.Name =
        "StrikeLimit"

    StrikeLimit.Value =
        CONFIG.Detection.StrikeLimit

    StrikeLimit.Parent =
        ConfigFolder

end

--------------------------------------------------
-- MAX WALK SPEED
--------------------------------------------------

local MaxWalkSpeed =
    ConfigFolder:FindFirstChild(
        "MaxWalkSpeed"
    )

if not MaxWalkSpeed then

    MaxWalkSpeed =
        Instance.new("NumberValue")

    MaxWalkSpeed.Name =
        "MaxWalkSpeed"

    MaxWalkSpeed.Value =
        CONFIG.Detection.MaxWalkSpeed

    MaxWalkSpeed.Parent =
        ConfigFolder

end

--------------------------------------------------
-- MAX JUMP POWER
--------------------------------------------------

local MaxJumpPower =
    ConfigFolder:FindFirstChild(
        "MaxJumpPower"
    )

if not MaxJumpPower then

    MaxJumpPower =
        Instance.new("NumberValue")

    MaxJumpPower.Name =
        "MaxJumpPower"

    MaxJumpPower.Value =
        CONFIG.Detection.MaxJumpPower

    MaxJumpPower.Parent =
        ConfigFolder

end

--------------------------------------------------
-- STATUS EVENT
--------------------------------------------------

local StatusEvent =
    RootFolder:FindFirstChild(
        "AntiCheatStatus"
    )

if not StatusEvent then

    StatusEvent =
        Instance.new("BindableEvent")

    StatusEvent.Name =
        "AntiCheatStatus"

    StatusEvent.Parent =
        RootFolder

end

local function fireStatus(
    player,
    message
)

    if not player then
        return
    end

    StatusEvent:Fire(
        player,
        message
    )

end

--------------------------------------------------
-- ADMIN STATUS LOG
--------------------------------------------------

Players.PlayerAdded:Connect(
    function(player)

        task.delay(
            2,
            function()

                if not player.Parent then
                    return
                end

                if isAdmin(player) then

                    debugPrint(
                        "Admin connected:",
                        player.Name
                    )

                    fireStatus(
                        player,
                        "Anti-Cheat admin detected."
                    )

                end

            end
        )

    end
)

--------------------------------------------------
-- SERVER SHUTDOWN
--------------------------------------------------

game:BindToClose(
    function()

        debugPrint(
            "Anti-Cheat shutting down."
        )

        for player, state in pairs(
            PlayerState
        ) do

            if player
                and player.Parent
            then

                state.LastPosition =
                    nil

            end

        end

        table.clear(
            RemoteState
        )

        table.clear(
            CombatState
        )

    end
)

--------------------------------------------------
-- FINAL STATUS
--------------------------------------------------

print(
    "===================================="
)

print(
    "[Kianbest Anti-Cheat V3]"
)

print(
    "Movement Protection : ENABLED"
)

print(
    "Remote Protection   : ENABLED"
)

print(
    "Combat Validation   : ENABLED"
)

print(
    "Ban System           : ENABLED"
)

print(
    "Admin System         : ENABLED"
)

print(
    "Persistent Ban       : ENABLED"
)

print(
    "===================================="
)

print(
    "[Kianbest Anti-Cheat] "
    .. "V3 FULL SYSTEM LOADED."
)