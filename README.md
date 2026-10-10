-- ============================================================
-- Babis | FREE EDITION — Egg Predictor integrado
-- ============================================================

local Players           = game:GetService("Players")
local RunService        = game:GetService("RunService")
local TweenService      = game:GetService("TweenService")
local UserInputService  = game:GetService("UserInputService")
local SoundService      = game:GetService("SoundService")
local HttpService       = game:GetService("HttpService")
local Workspace         = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CoreGui           = game:GetService("CoreGui")
local ProximityPromptService = game:GetService("ProximityPromptService")
local LocalPlayer       = Players.LocalPlayer
local PlayerGui         = LocalPlayer:WaitForChild("PlayerGui")

local LockState = {
    enabled = (getrawmetatable and setreadonly and newcclosure) ~= nil,
    protected = {}, bypass = false,
    props = { Position=true, Size=true, AnchorPoint=true, Parent=true,
        Visible=true, BackgroundTransparency=true, Enabled=true,
        DisplayOrder=true, Active=true, ZIndex=true },
}
if LockState.enabled then
    local ok, err = pcall(function()
        local mt = getrawmetatable(game)
        local oldNewIndex = mt.__newindex
        setreadonly(mt, false)
        mt.__newindex = newcclosure(function(self, key, value)
            if LockState.bypass == false and LockState.props[key]
               and type(self) == "userdata" and LockState.protected[self] then
                return
            end
            return oldNewIndex(self, key, value)
        end)
        setreadonly(mt, true)
    end)
    if not ok then LockState.enabled = false end
end
local function protectInstance(inst)
    if inst and LockState.enabled then LockState.protected[inst] = true end
end
local function safeWrite(inst, prop, value)
    if LockState.enabled then
        LockState.bypass = true; inst[prop] = value; LockState.bypass = false
    else inst[prop] = value end
end

local uiParent = (gethui and gethui()) or PlayerGui
for _, n in ipairs({"BabisUIKit", "BabisUIKitNotif", "BabisIntro", "BabisFlingFloat", "PredictorToggleGui"}) do
    local old = uiParent:FindFirstChild(n); if old then old:Destroy() end
    local oldPg = PlayerGui:FindFirstChild(n); if oldPg then oldPg:Destroy() end
    local oldCg = CoreGui:FindFirstChild(n); if oldCg then oldCg:Destroy() end
end

local CONFIG = {
    appName = "Babis | SURVIVE THE HULK",
    version = "v1.0.0",
    edition = "Free Version",
    discordInvite = "https://discord.gg/gEBbPa5C",
    colors = {
        windowBg=Color3.fromRGB(8,10,14), headerTint=Color3.fromRGB(8,10,14), divider=Color3.fromRGB(65,150,245),
        accent=Color3.fromRGB(90,165,255), accentSoft=Color3.fromRGB(160,210,255), accentDark=Color3.fromRGB(40,90,160),
        gold=Color3.fromRGB(255, 200, 60),
        goldSoft=Color3.fromRGB(255, 225, 140),
        goldDeep=Color3.fromRGB(190, 140, 30),
        navBg=Color3.fromRGB(18,18,19), navBorder=Color3.fromRGB(90,88,92), navIcon=Color3.fromRGB(52,61,69),
        navIconActive=Color3.fromRGB(160,210,255), navActiveBrd=Color3.fromRGB(160,210,255),
        cardBg=Color3.fromRGB(16,16,20), cardBorder=Color3.fromRGB(56,55,62),
        toggleOff=Color3.fromRGB(38,38,44), toggleOn=Color3.fromRGB(90,165,255),
        textPrimary=Color3.fromRGB(240,240,248), textDesc=Color3.fromRGB(150,148,168), textMuted=Color3.fromRGB(130,130,150),
        closeHover=Color3.fromRGB(220,70,90), handIdle=Color3.fromRGB(240,245,255), handHover=Color3.fromRGB(255,255,255),
        cfgIcon=Color3.fromRGB(170,120,255), cfgBtnBg=Color3.fromRGB(24,24,28), cfgBtnBorder=Color3.fromRGB(48,48,56),
        bellBg=Color3.fromRGB(30,38,48),
        lockedCard=Color3.fromRGB(14,14,18), lockedBorder=Color3.fromRGB(40,38,44), lockedText=Color3.fromRGB(120,118,135),
        premium=Color3.fromRGB(255,200,60),
    },
    sizes = {
        windowWidth=460, windowHeight=700, windowRadius=22, headerHeight=82,
        navHeight=88, navBtnSize=70, navBtnRadius=14, navGap=8, navPadX=12,
        contentPadX=14, scrollPadL=6, scrollPadR=8, iconSize=46,
        cardRadius=16, cardPadX=16, cardGap=14,
        toggleCardHeight=150, sliderCardHeight=170,
        toggleW=72, toggleH=38, toggleKnob=30, toggleRight=22,
        controlW=110, controlH=44, controlRight=22, handIconSize=82, handIconY=20,
        cfgHeight=516,
    },
    fonts = { appName=Enum.Font.GothamBold, version=Enum.Font.Gotham, intro=Enum.Font.Code },
    textSizes = {
        appName=22, version=15, intro=20, cardTitle=24, cardDesc=15,
        cfgTitle=22, cfgLabel=17, cfgBtn=17, cfgHint=16, cfgInput=17,
        aboutTitle=30, aboutDesc=16, discordTitle=28, discordSub=18,
    },
    anim = { fast=0.14, normal=0.20, slow=0.32, easing=Enum.EasingStyle.Quint, dir=Enum.EasingDirection.Out },
    tabs = {
        { id="main",     name="Main",     iconId=7733960981,     boost=6 },
        { id="visual",   name="Visuals",  iconId=7733774602,     boost=0 },
        { id="player",   name="Player",   iconId=92187976272467, boost=46 },
        { id="misc",     name="Misc",     iconId=10723387563,    boost=0 },
        { id="settings", name="Settings", iconId=7734053495,     boost=0 },
    },
    defaultTab = "main",
    handIconId="rbxassetid://88060480140568", cfgIconId="rbxassetid://10709791036",
    infoIconId="rbxassetid://7733964719", discordIconId="rbxassetid://100770414662869",
    bellIconId="rbxassetid://7072706001", notifySoundId="rbxassetid://5153734608",
    notifySoundVol=0.5, introImageId="rbxassetid://119251117614023",
    introDuration=5.5, introImgW=156, introImgH=139,
    lockIconId="rbxassetid://113617763566572",
    cfgFolder="Babis/configs", cfgExt=".json", autoloadFile="Babis/autoload.txt",
    positionFile="Babis/position.txt",
}

local function corner(parent, r)
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, r); c.Parent = parent; return c
end
local function stroke(parent, color, thickness, transparency)
    local s = Instance.new("UIStroke")
    s.Color=color; s.Thickness=thickness or 1; s.Transparency=transparency or 0
    s.ApplyStrokeMode=Enum.ApplyStrokeMode.Border; s.Parent=parent; return s
end
local function tween(obj, props, dur, style, dir)
    local info = TweenInfo.new(dur or CONFIG.anim.normal, style or CONFIG.anim.easing, dir or CONFIG.anim.dir)
    local t = TweenService:Create(obj, info, props); t:Play(); return t
end
local function popTween(obj, props, dur)
    local info = TweenInfo.new(dur or 0.25, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
    local t = TweenService:Create(obj, info, props); t:Play(); return t
end
local function safeCall(fn, ...)
    if type(fn) ~= "function" then return false end
    local ok, res = pcall(fn, ...); if ok then return res end; return nil
end

local function ensureBabisFolder()
    if not safeCall(isfolder, "Babis") then safeCall(makefolder, "Babis") end
end

local function savePosition(pos)
    if not pos then return end
    ensureBabisFolder()
    local data = { x = pos.X.Scale, y = pos.Y.Scale, xo = pos.X.Offset, yo = pos.Y.Offset }
    local ok, encoded = pcall(function() return HttpService:JSONEncode(data) end)
    if not ok or type(encoded) ~= "string" then return end
    safeCall(writefile, CONFIG.positionFile, encoded)
end

local function loadPosition()
    local content = safeCall(readfile, CONFIG.positionFile)
    if type(content) ~= "string" or content == "" then return nil end
    local ok, data = pcall(function() return HttpService:JSONDecode(content) end)
    if not ok or type(data) ~= "table" then return nil end
    return UDim2.new(
        tonumber(data.x) or 0.5, tonumber(data.xo) or 0,
        tonumber(data.y) or 0.5, tonumber(data.yo) or 0
    )
end

-- ============================================================
-- BYPASS TP V2
-- ============================================================
local BypassTP = (function()
    local API = { Enabled = false, AntiRagdollEnabled = true, AntiTrapEnabled = true }
    local Returning = false
    local AntiRagdollResetCooldown = 0
    local AntiRagdollConnection = nil
    local AntiTrapRan = false
    local DROP_WAIT = 0.4
    local BASE_X = 530
    local STEP_TARGET_X = 600
    local STEP_TARGET_Z = -420
    local STEP_Y = 71
    local FLY_THRESHOLD_X = 2000
    local FLY_SPEED_MULT = 1.25
    local INSTANT_STEAL = true
    local CurrentDropTarget = nil
    local FlyAnimation = nil
    local HeadOriginalSize = nil
    local HeadOriginalMeshScale = nil
    local HeadScaled = false
    local SafeCarry = { StepSpeed = 0.1 }
    local AntiDieConnections = {}
    local HumanoidState = { Character = nil, NormalHumanoid = nil, BypassHumanoid = nil }
    local StashedHumanoids = {}

    local function DisconnectAntiDie()
        for _, connection in ipairs(AntiDieConnections) do
            if connection then pcall(function() connection:Disconnect() end) end
        end
        table.clear(AntiDieConnections)
    end
    local function GetCharacter() return LocalPlayer.Character end
    local function GetRoot()
        local character = GetCharacter()
        if not character then return nil end
        local root = character:FindFirstChild("HumanoidRootPart")
        if root and root:IsDescendantOf(workspace) then return root end
        return nil
    end
    local function GetHumanoid()
        local character = GetCharacter()
        if not character then return nil end
        return character:FindFirstChildOfClass("Humanoid")
    end
    local function StopVelocity(root)
        if not root then return end
        pcall(function()
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
        end)
    end
    local function RestoreNormalHumanoid()
        local character = GetCharacter()
        if not character then return end
        DisconnectAntiDie()
        local stashed = StashedHumanoids[character]
        local current = character:FindFirstChildOfClass("Humanoid")
        if stashed and not stashed:IsDescendantOf(game) and stashed.Parent ~= nil then stashed = nil end
        local target = stashed
        if not target then
            local old = character:FindFirstChild("_OldHumanoid")
            local replaced = character:FindFirstChild("ReplacedHumanoid")
            if old and old:IsA("Humanoid") then target = old
            elseif replaced and replaced:IsA("Humanoid") then target = replaced end
        end
        if not target then target = current end
        if not target then return end
        for _, obj in ipairs(character:GetChildren()) do
            if obj:IsA("Humanoid") and obj ~= target then pcall(function() obj:Destroy() end) end
        end
        target.Name = "Humanoid"
        target:SetAttribute("VxzBypassHumanoid", nil)
        if target.Parent ~= character then target.Parent = character end
        pcall(function()
            target:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
            target.BreakJointsOnDeath = true
            target.Enabled = true
        end)
        if target.Health <= 0 then pcall(function() target.Health = target.MaxHealth end) end
        local camera = workspace.CurrentCamera
        if camera then camera.CameraSubject = target end
        local animate = character:FindFirstChild("Animate")
        if animate and animate:IsA("LocalScript") then animate.Disabled = false end
        HumanoidState.NormalHumanoid = target
        HumanoidState.BypassHumanoid = nil
        HumanoidState.Character = character
        StashedHumanoids[character] = nil
    end
    local function CreateBypassHumanoid(character)
        if not character then return nil end
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return nil end
        if humanoid:GetAttribute("VxzBypassHumanoid") == true then return humanoid end
        local animate = character:FindFirstChild("Animate")
        local walkSpeed = humanoid.WalkSpeed
        local jumpPower = humanoid.JumpPower
        local jumpHeight = humanoid.JumpHeight
        local maxHealth = humanoid.MaxHealth
        local health = humanoid.Health
        local autoRotate = humanoid.AutoRotate
        local animator = humanoid:FindFirstChildOfClass("Animator")
        if animator then
            for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
                pcall(function() track:Stop(0) end)
            end
        end
        if animate and animate:IsA("LocalScript") then animate.Disabled = true end
        humanoid.Archivable = true
        local clone
        local success = pcall(function() clone = humanoid:Clone() end)
        if not success or not clone then
            if animate and animate:IsA("LocalScript") then animate.Disabled = false end
            return nil
        end
        clone.Name = "_BypassHumanoid"
        clone:SetAttribute("VxzBypassHumanoid", true)
        for _, object in ipairs(clone:GetChildren()) do
            if object:IsA("Animator") then object:Destroy() end
        end
        pcall(function()
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
            humanoid.BreakJointsOnDeath = false
            humanoid.Enabled = false
        end)
        humanoid.Name = "_OldHumanoid"
        humanoid:SetAttribute("VxzBypassHumanoid", nil)
        StashedHumanoids[character] = humanoid
        humanoid.Parent = nil
        clone.Name = "Humanoid"
        clone.Parent = character
        clone.WalkSpeed = walkSpeed
        clone.JumpPower = jumpPower
        clone.JumpHeight = jumpHeight
        clone.MaxHealth = maxHealth
        clone.Health = math.clamp(health, 0, maxHealth)
        clone.AutoRotate = autoRotate
        local newAnimator = Instance.new("Animator")
        newAnimator.Parent = clone
        local camera = workspace.CurrentCamera
        if camera then camera.CameraSubject = clone end
        if animate and animate:IsA("LocalScript") then
            task.defer(function()
                if animate.Parent and API.Enabled then
                    animate.Disabled = false
                    task.wait()
                    if animate.Parent and API.Enabled then
                        animate.Disabled = true
                        task.wait()
                        if animate.Parent and API.Enabled then animate.Disabled = false end
                    end
                end
            end)
        end
        HumanoidState.Character = character
        HumanoidState.BypassHumanoid = clone
        HumanoidState.NormalHumanoid = nil
        return clone
    end
    local function ActivateAntiDie()
        if not API.Enabled then return end
        DisconnectAntiDie()
        local character = GetCharacter()
        if not character then return end
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return end
        humanoid:SetAttribute("VxzBypassHumanoid", true)
        pcall(function()
            humanoid.BreakJointsOnDeath = false
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        end)
        local healthConnection = humanoid:GetPropertyChangedSignal("Health"):Connect(function()
            if not API.Enabled then return end
            if humanoid.Parent and humanoid.Health <= 0 then
                pcall(function() humanoid.Health = humanoid.MaxHealth end)
            end
        end)
        table.insert(AntiDieConnections, healthConnection)
        local diedConnection = humanoid.Died:Connect(function()
            if not API.Enabled then return end
            task.defer(function()
                if not API.Enabled then return end
                local currentCharacter = GetCharacter()
                if not currentCharacter or currentCharacter ~= character or not currentCharacter.Parent then return end
                local currentHumanoid = currentCharacter:FindFirstChildOfClass("Humanoid")
                if currentHumanoid and currentHumanoid ~= humanoid then return end
                local newHumanoid = Instance.new("Humanoid")
                newHumanoid.Name = "ReplacedHumanoid"
                newHumanoid:SetAttribute("VxzBypassHumanoid", true)
                newHumanoid.Parent = currentCharacter
                pcall(function()
                    newHumanoid.BreakJointsOnDeath = false
                    newHumanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
                end)
                HumanoidState.BypassHumanoid = newHumanoid
                local camera = workspace.CurrentCamera
                if camera then camera.CameraSubject = newHumanoid end
                pcall(function() humanoid:Destroy() end)
                ActivateAntiDie()
            end)
        end)
        table.insert(AntiDieConnections, diedConnection)
    end
    local function ActivateBypass()
        if not API.Enabled then return end
        local character = GetCharacter()
        if not character then return end
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return end
        if humanoid.Name == "Humanoid" and humanoid:GetAttribute("VxzBypassHumanoid") ~= true then
            CreateBypassHumanoid(character)
        end
        ActivateAntiDie()
    end
    local function DeactivateBypass()
        Returning = false
        CurrentDropTarget = nil
        RestoreNormalHumanoid()
    end
    local function ForceAntiRagdollReset()
        local char = LocalPlayer.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum or not root or hum.Health <= 0 then return end
        pcall(function()
            hum:ChangeState(Enum.HumanoidStateType.GettingUp)
            root.Velocity = Vector3.zero
            root.RotVelocity = Vector3.zero
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
            for _, obj in ipairs(char:GetDescendants()) do
                if obj:IsA("Motor6D") then obj.Enabled = true
                elseif obj:IsA("Constraint") then obj.Enabled = true end
            end
            if workspace.CurrentCamera then workspace.CurrentCamera.CameraSubject = hum end
            local ps = LocalPlayer:FindFirstChild("PlayerScripts")
            local pm = ps and ps:FindFirstChild("PlayerModule")
            if pm then
                local cms = pm:FindFirstChild("ControlModule")
                if cms then
                    local cm = require(cms)
                    if cm then cm:Enable() end
                end
            end
            hum.AutoRotate = true
            hum.PlatformStand = false
            hum.Sit = false
        end)
    end
    local function StartAntiRagdoll()
        if AntiRagdollConnection then AntiRagdollConnection:Disconnect(); AntiRagdollConnection = nil end
        AntiRagdollConnection = RunService.Heartbeat:Connect(function()
            if not API.AntiRagdollEnabled then return end
            local char = LocalPlayer.Character
            if not char then return end
            local hum = char:FindFirstChildOfClass("Humanoid")
            if not hum or hum.Health <= 0 then return end
            local state = hum:GetState()
            local isRagdolled = state == Enum.HumanoidStateType.Physics
                or state == Enum.HumanoidStateType.Ragdoll
                or state == Enum.HumanoidStateType.FallingDown
            if isRagdolled then
                local now = tick()
                if now - AntiRagdollResetCooldown > 0.15 then
                    AntiRagdollResetCooldown = now
                    ForceAntiRagdollReset()
                end
            end
        end)
    end
    local function RunAntiTrap()
        if AntiTrapRan then return end
        AntiTrapRan = true
        task.spawn(function()
            pcall(function()
                loadstring(game:HttpGet("https://raw.githubusercontent.com/Lutosys/opensrc/refs/heads/main/StealAnEggAntiTrap.lua"))()
            end)
        end)
    end
    StartAntiRagdoll()
    function API:SetEnabled(v)
        self.Enabled = v and true or false
        if self.Enabled then
            task.spawn(function()
                task.wait(0.05)
                ActivateBypass()
            end)
        else
            DeactivateBypass()
        end
    end
    function API:SetAntiRagdoll(v)
        self.AntiRagdollEnabled = v and true or false
        if self.AntiRagdollEnabled then ForceAntiRagdollReset() end
    end
    function API:SetAntiTrap(v)
        self.AntiTrapEnabled = v and true or false
        if self.AntiTrapEnabled then RunAntiTrap() end
    end
    return API
end)()

-- ============================================================
-- EGG PREDICTOR MODULE
-- ============================================================
local EggPredictor = (function()
    local M = {
        Priority = "Highest Value",
        Sort = "Value",
        ActiveTab = "Backpack",
        QueuedSteals = {},
        CanceledSteals = {},
    }

    local function safeRequire(pathArray)
        local current = ReplicatedStorage
        for _, name in ipairs(pathArray) do
            current = current and current:FindFirstChild(name)
        end
        if current and current:IsA("ModuleScript") then
            local ok, result = pcall(require, current)
            if ok then return result end
        end
        return nil
    end

    local EggState   = safeRequire({"Client", "EggState"})
    local Mutations  = safeRequire({"Shared", "Modules", "Mutations"})
    local EggRecords = safeRequire({"Shared", "Util", "EggRecords"})
    local Assets     = safeRequire({"Data", "Assets"})

    local function GetAssetInfo(category)
        local assetDir = Assets and rawget(Assets, "Directory")
        local asset = assetDir and assetDir[tostring(category)]
        local rarityObj = asset and asset.Rarity
        local rarityNum = rarityObj and (rarityObj.RarityNumber or rarityObj.Rank) or 0
        local rarityName = rarityObj and tostring(rarityObj.DisplayName or rarityObj._id or "") or ""
        local earningRate = asset and tonumber(asset.EarningRate) or 0
        local icon = asset and asset.Icon or nil
        if icon and tonumber(icon) then icon = "rbxassetid://" .. tostring(icon) end
        local displayName = (asset and tostring(asset.DisplayName or category)) or tostring(category)
        return rarityNum, earningRate, rarityName, icon, displayName
    end

    local function GetMutationMultiplier(mutationData)
        if type(Mutations) == "table" and type(Mutations.EarningsFor) == "function" then
            local ok, result = pcall(Mutations.EarningsFor, mutationData or {})
            if ok and type(result) == "number" then return result end
        end
        return 1
    end

    local function GetEggWeight(category, scale)
        if type(EggRecords) == "table" and type(EggRecords.WeightKgForScale) == "function" then
            local ok, result = pcall(EggRecords.WeightKgForScale, category, scale)
            if ok and type(result) == "number" then return result end
        end
        return 0
    end

    local function GetWeightKg(record)
        if type(EggRecords) == "table" and type(EggRecords.WeightKg) == "function" then
            local ok, result = pcall(EggRecords.WeightKg, record)
            if ok and type(result) == "number" then return result end
        end
        local cat = record.AssetCategory or record.Category
        local scale = tonumber(record.AssetScale or record.Scale) or 1
        return GetEggWeight(cat, scale)
    end

    local function ScaleFactor(scale)
        scale = tonumber(scale) or 1
        if scale > 5 then
            return (scale / 5) ^ 1.2 * 19.637875755794113
        end
        return scale ^ 1.85
    end

    local function GetGrowthStatus(uid, record, serverTime)
        if record.Placement == nil then
            return "Inventory", math.huge, 0
        end
        local ready = false
        if type(EggState) == "table" and type(EggState.IsReadyToHatch) == "function" then
            pcall(function() ready = EggState.IsReadyToHatch(uid) == true end)
        end
        if ready then return "Ready", 0, 100 end
        local remaining, duration = 0, 0
        if type(EggRecords) == "table" then
            local mult = 1
            if type(EggRecords.GrowthSpeedMultiplier) == "function" then
                local ok, r = pcall(EggRecords.GrowthSpeedMultiplier, record.AssetCategory)
                if ok and type(r) == "number" and r > 0 then mult = r end
            end
            if type(EggRecords.GrowthSecondsRemaining) == "function" then
                local ok, r = pcall(EggRecords.GrowthSecondsRemaining, record, serverTime, mult)
                if ok and type(r) == "number" then remaining = r end
            end
            if type(EggRecords.GrowthDuration) == "function" then
                local ok, r = pcall(EggRecords.GrowthDuration, record.AssetCategory)
                if ok and type(r) == "number" then duration = r end
            end
        end
        local percent = 0
        if duration > 0 then
            percent = math.clamp(math.floor((1 - remaining / duration) * 100), 0, 99)
        end
        return "Growing", remaining, percent
    end

    local function ParseRecords(records, context)
        if type(records) ~= "table" then return {} end
        local parsed = {}
        local serverTime = workspace:GetServerTimeNow()
        for uid, record in pairs(records) do
            if type(record) == "table" then
                local rid = record.Uid or record.UID or record.Id or uid
                local category = record.AssetCategory or record.Category or record.ItemCategory
                local validState = true
                if context == "Field" then
                    validState = (record.State == "Slot" or record.State == "Dropped")
                end
                local hasPos = (context == "Backpack") or (typeof(record.BottomCFrame) == "CFrame")
                if rid and category and validState and hasPos and not M.CanceledSteals[rid] then
                    local rarityNum, earningRate, rarityName, icon, displayName = GetAssetInfo(category)
                    local scale = tonumber(record.AssetScale or record.Scale) or 1
                    local mutationMult = GetMutationMultiplier(record.Mutations)
                    local value = earningRate * ScaleFactor(scale) * mutationMult
                    local mutList = {}
                    if type(record.Mutations) == "table" then
                        for _, m in ipairs(record.Mutations) do
                            if type(m) == "string" then table.insert(mutList, m) end
                        end
                    end
                    if type(record.BaseMutation) == "string"
                       and record.BaseMutation ~= "" and record.BaseMutation ~= "Normal" then
                        local found = false
                        for _, m in ipairs(mutList) do if m == record.BaseMutation then found = true break end end
                        if not found then table.insert(mutList, record.BaseMutation) end
                    end
                    local status, remaining, percent = "Bag", math.huge, 0
                    if context == "Backpack" then
                        status, remaining, percent = GetGrowthStatus(rid, record, serverTime)
                    end
                    table.insert(parsed, {
                        Uid = rid, Category = tostring(category),
                        DisplayName = displayName, RarityName = rarityName, Icon = icon,
                        Scale = scale, State = record.State or "Bag",
                        Rarity = rarityNum, Weight = GetWeightKg(record),
                        Mutation = mutationMult, Mutations = mutList,
                        Value = value, CFrame = record.BottomCFrame,
                        AreaId = tostring(record.AreaId or ""),
                        Status = status, Remaining = remaining, Percent = percent,
                    })
                end
            end
        end
        if context == "Backpack" then
            local s = M.Sort
            table.sort(parsed, function(a, b)
                if s == "Rarity" and a.Rarity ~= b.Rarity then return a.Rarity > b.Rarity end
                if s == "Time Left" and a.Remaining ~= b.Remaining then return a.Remaining < b.Remaining end
                if a.Value ~= b.Value then return a.Value > b.Value end
                return a.Rarity > b.Rarity
            end)
        else
            table.sort(parsed, function(a, b)
                if M.Priority == "Biggest Weight" and a.Weight ~= b.Weight then return a.Weight > b.Weight
                elseif M.Priority == "Best Mutation" and a.Mutation ~= b.Mutation then return a.Mutation > b.Mutation
                elseif M.Priority == "Highest Value" and a.Value ~= b.Value then return a.Value > b.Value
                elseif M.Priority == "Lowest Value" and a.Value ~= b.Value then return a.Value < b.Value end
                if a.Rarity ~= b.Rarity then return a.Rarity > b.Rarity end
                return a.Value > b.Value
            end)
        end
        return parsed
    end

    local function GetFieldEggs()
        local records = nil
        if type(EggState) == "table" and type(EggState.ReadFieldEggs) == "function" then
            pcall(function()
                local data = EggState.ReadFieldEggs()
                if data and type(data.Records) == "table" and next(data.Records) ~= nil then
                    records = data.Records
                end
            end)
        end
        if not records then
            local networking = ReplicatedStorage:FindFirstChild("Packages")
                and ReplicatedStorage.Packages:FindFirstChild("Networking")
            local remote = networking and networking:FindFirstChild("RF/EggWorld/AskFieldEggSnapshot")
            if remote and remote:IsA("RemoteFunction") then
                local ok, res = pcall(remote.InvokeServer, remote)
                if ok and type(res) == "table" then records = res.Records end
            end
        end
        return ParseRecords(records, "Field")
    end

    local function GetBackpackEggs()
        local records = nil
        if type(EggState) == "table" and type(EggState.ReadOwnerEggs) == "function" then
            pcall(function()
                local data = EggState.ReadOwnerEggs(LocalPlayer.UserId)
                if type(data) == "table" then
                    local bag = {}
                    for uid, record in pairs(data) do
                        if type(record) == "table" then
                            record.Uid = record.Uid or uid
                            bag[uid] = record
                        end
                    end
                    if next(bag) ~= nil then records = bag end
                end
            end)
        end
        if not records then
            pcall(function()
                local attr = LocalPlayer:GetAttribute("EggInventory")
                          or LocalPlayer:GetAttribute("BackpackEggs")
                if type(attr) == "table" then records = attr end
            end)
        end
        return ParseRecords(records, "Backpack")
    end

    local previewCache = {}

    local function FindEggModel(category)
        local assetDir = Assets and rawget(Assets, "Directory")
        local asset = assetDir and assetDir[tostring(category)]
        if not asset then return nil end
        local names = {}
        if type(asset.Egg) == "table" and asset.Egg.ModelName then table.insert(names, tostring(asset.Egg.ModelName)) end
        if asset.ModelName then table.insert(names, tostring(asset.ModelName)) end
        if asset._id then table.insert(names, tostring(asset._id)) end
        if asset.DisplayName then table.insert(names, tostring(asset.DisplayName)) end
        table.insert(names, tostring(category))
        local containers = {
            ReplicatedStorage,
            ReplicatedStorage:FindFirstChild("Assets"),
            ReplicatedStorage:FindFirstChild("Models"),
            ReplicatedStorage:FindFirstChild("Eggs"),
            ReplicatedStorage:FindFirstChild("Prefabs"),
            ReplicatedStorage:FindFirstChild("Shared"),
            workspace,
        }
        for _, container in ipairs(containers) do
            if container then
                for _, name in ipairs(names) do
                    local hit = container:FindFirstChild(name, true)
                    if hit and (hit:IsA("Model") or hit:IsA("BasePart")) then
                        return hit
                    end
                end
            end
        end
        return nil
    end

    local function PrepareModel(model)
        if not model then return nil end
        local clone
        if model:IsA("BasePart") then
            clone = model:Clone()
        else
            local ok, result = pcall(function() return model:Clone() end)
            if not ok or not result then return nil end
            clone = result
        end
        for _, d in ipairs(clone:GetDescendants()) do
            if d:IsA("Script") or d:IsA("LocalScript") or d:IsA("Sound")
                or d:IsA("ParticleEmitter") or d:IsA("Trail") or d:IsA("Beam")
                or d:IsA("PointLight") or d:IsA("SpotLight") or d:IsA("SurfaceLight")
                or d:IsA("BillboardGui") or d:IsA("SurfaceGui") or d:IsA("ProximityPrompt")
                or d:IsA("ClickDetector") or d:IsA("BodyMover") or d:IsA("Constraint")
                or d:IsA("WeldConstraint") or d:IsA("JointInstance")
            then
                pcall(function() d:Destroy() end)
            elseif d:IsA("BasePart") then
                d.Anchored = true
                d.CanCollide = false
                d.CanQuery = false
                d.CanTouch = false
            end
        end
        return clone
    end

    local function BuildPreview(parent, category)
        local vp = Instance.new("ViewportFrame")
        vp.Name = "EggPreview"
        vp.BackgroundColor3 = Color3.fromRGB(14, 16, 22)
        vp.BorderSizePixel = 0
        vp.Ambient = Color3.fromRGB(190, 190, 200)
        vp.LightColor = Color3.fromRGB(255, 255, 255)
        vp.LightDirection = Vector3.new(-0.4, -1, -0.6)
        vp.Parent = parent
        corner(vp, 6)
        local cam = Instance.new("Camera")
        cam.FieldOfView = 40
        cam.Parent = vp
        vp.CurrentCamera = cam
        local model = previewCache[category]
        if model == nil then
            local raw = FindEggModel(category)
            model = raw and PrepareModel(raw) or false
            previewCache[category] = model
        end
        if not model then
            local placeholder = Instance.new("TextLabel")
            placeholder.Size = UDim2.fromScale(1, 1)
            placeholder.BackgroundTransparency = 1
            placeholder.Text = "No 3D model"
            placeholder.TextColor3 = Color3.fromRGB(160, 168, 184)
            placeholder.TextSize = 10
            placeholder.Font = Enum.Font.Gotham
            placeholder.Parent = vp
            return vp
        end
        local clone = model:Clone()
        clone.Parent = vp
        local modelCFrame, size
        if clone:IsA("Model") then
            local ok, cf, sz = pcall(function() return clone:GetBoundingBox() end)
            if ok then modelCFrame, size = cf, sz end
        elseif clone:IsA("BasePart") then
            modelCFrame, size = clone.CFrame, clone.Size
        end
        if modelCFrame then
            local maxDim = math.max(size.X, size.Y, size.Z, 1)
            local dist = maxDim * 2.4
            local dir = CFrame.Angles(math.rad(-12), math.rad(35), 0)
            cam.CFrame = CFrame.new(modelCFrame.Position + (dir.LookVector * -dist)) * dir
            if clone:IsA("Model") then
                clone:PivotTo(modelCFrame * CFrame.Angles(0, math.rad(25), 0))
            else
                clone.CFrame = modelCFrame * CFrame.Angles(0, math.rad(25), 0)
            end
        else
            cam.CFrame = CFrame.new(Vector3.new(0, 0, -6))
        end
        return vp
    end

    local function formatRate(value)
        value = tonumber(value) or 0
        local a = math.abs(value)
        local c
        if a >= 1e9 then c = string.format("%.1fB", value / 1e9)
        elseif a >= 1e6 then c = string.format("%.1fM", value / 1e6)
        elseif a >= 1e3 then c = string.format("%.1fk", value / 1e3)
        else c = string.format("%.0f", value) end
        c = c:gsub("%.0([kMB])", "%1")
        return c .. "/s"
    end

    local function formatClock(seconds)
        seconds = math.max(0, math.floor(tonumber(seconds) or 0))
        return string.format("%02d:%02d:%02d",
            math.floor(seconds / 3600),
            math.floor(seconds % 3600 / 60),
            seconds % 60)
    end

    function M.BuildInto(card, notify)
        local title = Instance.new("TextLabel")
        title.Size = UDim2.new(1, -24, 0, 24)
        title.Position = UDim2.fromOffset(12, 8)
        title.BackgroundTransparency = 1
        title.Text = "EGG PREDICTOR"
        title.TextColor3 = CONFIG.colors.accent
        title.Font = Enum.Font.GothamBold
        title.TextSize = 18
        title.TextXAlignment = Enum.TextXAlignment.Left
        title.Parent = card

        local status = Instance.new("TextLabel")
        status.Size = UDim2.new(1, -24, 0, 16)
        status.Position = UDim2.fromOffset(12, 34)
        status.BackgroundTransparency = 1
        status.Text = "Tap Refresh to scan."
        status.TextColor3 = CONFIG.colors.textDesc
        status.Font = Enum.Font.Gotham
        status.TextSize = 11
        status.TextXAlignment = Enum.TextXAlignment.Left
        status.TextTruncate = Enum.TextTruncate.AtEnd
        status.Parent = card

        local tabBar = Instance.new("Frame")
        tabBar.Size = UDim2.new(1, -24, 0, 28)
        tabBar.Position = UDim2.fromOffset(12, 54)
        tabBar.BackgroundColor3 = Color3.fromRGB(24, 26, 34)
        tabBar.BorderSizePixel = 0
        tabBar.Parent = card
        corner(tabBar, 6)
        local tabLayout = Instance.new("UIListLayout")
        tabLayout.FillDirection = Enum.FillDirection.Horizontal
        tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
        tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
        tabLayout.Padding = UDim.new(0, 4)
        tabLayout.Parent = tabBar
        local tabPad = Instance.new("UIPadding")
        tabPad.PaddingLeft = UDim.new(0, 4)
        tabPad.PaddingRight = UDim.new(0, 4)
        tabPad.Parent = tabBar

        local function makeTab(text, key, w)
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(0, w or 90, 1, -6)
            btn.BackgroundColor3 = Color3.fromRGB(34, 38, 48)
            btn.Text = text
            btn.TextColor3 = Color3.fromRGB(200, 210, 230)
            btn.TextSize = 11
            btn.Font = Enum.Font.GothamBold
            btn.AutoButtonColor = false
            btn.Parent = tabBar
            corner(btn, 5)
            btn:SetAttribute("TabKey", key)
            return btn
        end

        local backpackTab = makeTab("BACKPACK", "Backpack", 100)
        local fieldTab    = makeTab("FIELD",    "Field",    80)

        local refreshBtn = Instance.new("TextButton")
        refreshBtn.Size = UDim2.new(0, 90, 0, 22)
        refreshBtn.Position = UDim2.new(1, -102, 0, 88)
        refreshBtn.BackgroundColor3 = CONFIG.colors.accent
        refreshBtn.TextColor3 = Color3.fromRGB(15, 20, 25)
        refreshBtn.TextSize = 11
        refreshBtn.Font = Enum.Font.GothamBold
        refreshBtn.Text = "REFRESH"
        refreshBtn.AutoButtonColor = false
        refreshBtn.Parent = card
        corner(refreshBtn, 5)

        local sortBtn = Instance.new("TextButton")
        sortBtn.Size = UDim2.new(0, 180, 0, 22)
        sortBtn.Position = UDim2.fromOffset(12, 88)
        sortBtn.BackgroundColor3 = CONFIG.colors.cardBg
        sortBtn.TextColor3 = CONFIG.colors.accent
        sortBtn.TextSize = 10
        sortBtn.Font = Enum.Font.GothamBold
        sortBtn.Text = "Sort: Value"
        sortBtn.AutoButtonColor = false
        sortBtn.Parent = card
        corner(sortBtn, 5)
        stroke(sortBtn, CONFIG.colors.cardBorder, 1, 0)

        local list = Instance.new("ScrollingFrame")
        list.Size = UDim2.new(1, -24, 0, 230)
        list.Position = UDim2.fromOffset(12, 118)
        list.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
        list.BorderSizePixel = 0
        list.ScrollBarThickness = 4
        list.ScrollBarImageColor3 = CONFIG.colors.accent
        list.CanvasSize = UDim2.new(0, 0, 0, 0)
        list.AutomaticCanvasSize = Enum.AutomaticSize.Y
        list.Parent = card
        corner(list, 8)
        local listLayout = Instance.new("UIListLayout")
        listLayout.Padding = UDim.new(0, 4)
        listLayout.SortOrder = Enum.SortOrder.LayoutOrder
        listLayout.Parent = list
        local listPad = Instance.new("UIPadding")
        listPad.PaddingTop = UDim.new(0, 5)
        listPad.PaddingLeft = UDim.new(0, 5)
        listPad.PaddingRight = UDim.new(0, 5)
        listPad.PaddingBottom = UDim.new(0, 5)
        listPad.Parent = list

        local previewBox = Instance.new("Frame")
        previewBox.Size = UDim2.new(1, -24, 0, 180)
        previewBox.Position = UDim2.fromOffset(12, 354)
        previewBox.BackgroundColor3 = Color3.fromRGB(12, 14, 20)
        previewBox.BorderSizePixel = 0
        previewBox.Parent = card
        corner(previewBox, 8)

        local previewName = Instance.new("TextLabel")
        previewName.Size = UDim2.new(1, -12, 0, 18)
        previewName.Position = UDim2.fromOffset(6, 4)
        previewName.BackgroundTransparency = 1
        previewName.Text = "Select an egg"
        previewName.TextColor3 = CONFIG.colors.textPrimary
        previewName.TextSize = 12
        previewName.Font = Enum.Font.GothamBold
        previewName.TextXAlignment = Enum.TextXAlignment.Left
        previewName.TextTruncate = Enum.TextTruncate.AtEnd
        previewName.Parent = previewBox

        local previewHolder = Instance.new("Frame")
        previewHolder.Size = UDim2.new(0, 120, 1, -30)
        previewHolder.Position = UDim2.fromOffset(6, 26)
        previewHolder.BackgroundTransparency = 1
        previewHolder.Parent = previewBox

        local previewInfo = Instance.new("TextLabel")
        previewInfo.Size = UDim2.new(1, -138, 1, -30)
        previewInfo.Position = UDim2.new(0, 132, 0, 26)
        previewInfo.BackgroundTransparency = 1
        previewInfo.Text = ""
        previewInfo.TextColor3 = CONFIG.colors.textPrimary
        previewInfo.TextSize = 11
        previewInfo.Font = Enum.Font.Gotham
        previewInfo.TextXAlignment = Enum.TextXAlignment.Left
        previewInfo.TextYAlignment = Enum.TextYAlignment.Top
        previewInfo.TextWrapped = true
        previewInfo.Parent = previewBox

        local function updateTabVisuals()
            local active   = CONFIG.colors.accent
            local inactive = Color3.fromRGB(34, 38, 48)
            for _, btn in ipairs({backpackTab, fieldTab}) do
                if btn:GetAttribute("TabKey") == M.ActiveTab then
                    btn.BackgroundColor3 = active
                    btn.TextColor3 = Color3.fromRGB(15, 20, 25)
                else
                    btn.BackgroundColor3 = inactive
                    btn.TextColor3 = Color3.fromRGB(200, 210, 230)
                end
            end
        end

        local fieldPriorities    = {"Highest Value", "Best Rarity", "Biggest Weight", "Best Mutation", "Lowest Value"}
        local backpackPriorities = {"Value", "Rarity", "Time Left"}

        local function updateSortLabel()
            if M.ActiveTab == "Backpack" then
                sortBtn.Text = "Sort: " .. M.Sort .. "  (tap)"
            else
                sortBtn.Text = "Sort: " .. M.Priority .. "  (tap)"
            end
        end

        local function clearList()
            for _, child in ipairs(list:GetChildren()) do
                if child:IsA("Frame") or child:IsA("TextLabel") or child:IsA("TextButton") then
                    child:Destroy()
                end
            end
        end
        local function clearPreview()
            for _, child in ipairs(previewHolder:GetChildren()) do child:Destroy() end
        end

        local function showPreview(egg)
            clearPreview()
            if not egg then
                previewName.Text = "Select an egg"
                previewInfo.Text = ""
                return
            end
            previewName.Text = egg.DisplayName
            local vp = BuildPreview(previewHolder, egg.Category)
            vp.Size = UDim2.fromScale(1, 1)

            local lines = {}
            table.insert(lines, "Value: " .. formatRate(egg.Value))
            table.insert(lines, string.format("Rarity: %s (R%d)",
                egg.RarityName ~= "" and egg.RarityName or "?", egg.Rarity))
            table.insert(lines, string.format("Scale: x%.2f", egg.Scale))
            table.insert(lines, string.format("Weight: %.2f kg", egg.Weight))
            if #egg.Mutations > 0 then
                table.insert(lines, "Mut: " .. table.concat(egg.Mutations, ", "))
            else
                table.insert(lines, "Mut: None")
            end
            if egg.Status == "Ready" then
                table.insert(lines, "")
                table.insert(lines, ">> READY TO HATCH")
            elseif egg.Status == "Growing" then
                table.insert(lines, string.format(">> GROWING %s (%d%%)", formatClock(egg.Remaining), egg.Percent))
            elseif egg.Status == "Inventory" then
                table.insert(lines, ">> In inventory")
            end
            previewInfo.Text = table.concat(lines, "\n")
        end

        local function buildRow(rank, egg)
            local row = Instance.new("TextButton")
            row.Size = UDim2.new(1, -4, 0, 46)
            row.BackgroundColor3 = rank == 1 and Color3.fromRGB(36, 54, 44) or Color3.fromRGB(24, 27, 36)
            row.BorderSizePixel = 0
            row.Text = ""
            row.AutoButtonColor = true
            row.Parent = list
            corner(row, 6)

            local iconSpace = 0
            if egg.Icon then
                local icon = Instance.new("ImageLabel")
                icon.Size = UDim2.fromOffset(30, 30)
                icon.Position = UDim2.fromOffset(6, 8)
                icon.BackgroundTransparency = 1
                icon.Image = egg.Icon
                icon.ScaleType = Enum.ScaleType.Fit
                icon.Parent = row
                iconSpace = 36
            end

            local title = Instance.new("TextLabel")
            title.Size = UDim2.new(1, -10 - iconSpace, 0, 15)
            title.Position = UDim2.fromOffset(5 + iconSpace, 3)
            title.BackgroundTransparency = 1
            title.Text = string.format("#%d  %s%s", rank, egg.DisplayName, rank == 1 and "  *" or "")
            title.TextColor3 = CONFIG.colors.textPrimary
            title.TextSize = 11
            title.Font = Enum.Font.GothamBold
            title.TextXAlignment = Enum.TextXAlignment.Left
            title.TextTruncate = Enum.TextTruncate.AtEnd
            title.Parent = row

            local line2 = Instance.new("TextLabel")
            line2.Size = UDim2.new(1, -10 - iconSpace, 0, 12)
            line2.Position = UDim2.fromOffset(5 + iconSpace, 19)
            line2.BackgroundTransparency = 1
            local rarityText = egg.RarityName ~= "" and egg.RarityName or ("R" .. egg.Rarity)
            line2.Text = string.format("%s | %s | %.1fkg | x%.1f",
                formatRate(egg.Value), rarityText, egg.Weight, egg.Scale)
            line2.TextColor3 = CONFIG.colors.textDesc
            line2.TextSize = 9
            line2.Font = Enum.Font.Gotham
            line2.TextXAlignment = Enum.TextXAlignment.Left
            line2.TextTruncate = Enum.TextTruncate.AtEnd
            line2.Parent = row

            local line3 = Instance.new("TextLabel")
            line3.Size = UDim2.new(1, -10 - iconSpace, 0, 11)
            line3.Position = UDim2.fromOffset(5 + iconSpace, 31)
            line3.BackgroundTransparency = 1
            line3.Text = (#egg.Mutations > 0) and table.concat(egg.Mutations, ", ") or "Normal"
            line3.TextColor3 = Color3.fromRGB(120, 132, 155)
            line3.TextSize = 9
            line3.Font = Enum.Font.Gotham
            line3.TextXAlignment = Enum.TextXAlignment.Left
            line3.TextTruncate = Enum.TextTruncate.AtEnd
            line3.Parent = row

            local chip = Instance.new("TextLabel")
            chip.Size = UDim2.fromOffset(80, 14)
            chip.Position = UDim2.new(1, -84, 0, 4)
            chip.BackgroundTransparency = 1
            chip.TextSize = 9
            chip.Font = Enum.Font.GothamBold
            chip.TextXAlignment = Enum.TextXAlignment.Right
            chip.Parent = row

            if egg.Status == "Ready" then
                chip.Text = "READY"
                chip.TextColor3 = Color3.fromRGB(80, 220, 140)
            elseif egg.Status == "Growing" then
                chip.Text = string.format("%d%%  %s", egg.Percent, formatClock(egg.Remaining):sub(1, 5))
                chip.TextColor3 = Color3.fromRGB(255, 200, 80)
            elseif egg.Status == "Inventory" then
                chip.Text = "IN BAG"
                chip.TextColor3 = Color3.fromRGB(130, 200, 255)
            elseif egg.Status == "Bag" then
                chip.Text = "-"
                chip.TextColor3 = Color3.fromRGB(140, 150, 170)
            else
                chip.Text = tostring(egg.Status):upper()
                chip.TextColor3 = Color3.fromRGB(180, 190, 210)
            end

            row.MouseButton1Click:Connect(function() showPreview(egg) end)
            return row
        end

        local function buildSectionLabel(text, count, color)
            local lbl = Instance.new("TextLabel")
            lbl.Size = UDim2.new(1, -4, 0, 16)
            lbl.BackgroundTransparency = 1
            lbl.Text = string.format("%s  (%d)", text, count)
            lbl.TextColor3 = color
            lbl.TextSize = 10
            lbl.Font = Enum.Font.GothamBold
            lbl.TextXAlignment = Enum.TextXAlignment.Left
            lbl.Parent = list
            return lbl
        end

        local function buildEmpty(text)
            local empty = Instance.new("TextLabel")
            empty.Size = UDim2.new(1, -8, 0, 80)
            empty.BackgroundTransparency = 1
            empty.Text = text
            empty.TextWrapped = true
            empty.TextColor3 = CONFIG.colors.textDesc
            empty.TextSize = 12
            empty.Font = Enum.Font.Gotham
            empty.Parent = list
        end

        local function refresh()
            if not card.Parent then return end
            status.Text = "Scanning " .. M.ActiveTab .. "..."
            clearList()
            clearPreview()

            if M.ActiveTab == "Backpack" then
                local eggs = GetBackpackEggs()
                local ready, growing, bag = 0, 0, 0
                local total = 0
                for _, e in ipairs(eggs) do
                    total = total + e.Value
                    if e.Status == "Ready" then ready = ready + 1
                    elseif e.Status == "Growing" then growing = growing + 1
                    elseif e.Status == "Inventory" then bag = bag + 1 end
                end
                status.Text = string.format("%d eggs · %d ready · %d growing · %d bag · %s",
                    #eggs, ready, growing, bag, formatRate(total))
                if #eggs == 0 then
                    buildEmpty("No eggs in your backpack.\nPlace eggs or add some to your bag and Refresh.")
                    showPreview(nil)
                    return
                end
                local rank = 0
                local function renderGroup(name, color, list_)
                    if #list_ == 0 then return end
                    buildSectionLabel(name, #list_, color)
                    for _, egg in ipairs(list_) do
                        rank = rank + 1
                        buildRow(rank, egg)
                    end
                end
                local readyList, growingList, bagList = {}, {}, {}
                for _, e in ipairs(eggs) do
                    if e.Status == "Ready" then table.insert(readyList, e)
                    elseif e.Status == "Growing" then table.insert(growingList, e)
                    else table.insert(bagList, e) end
                end
                renderGroup("READY TO HATCH", Color3.fromRGB(80, 220, 140), readyList)
                renderGroup("GROWING",        Color3.fromRGB(255, 200, 80),  growingList)
                renderGroup("IN INVENTORY",   Color3.fromRGB(130, 200, 255), bagList)
                showPreview(eggs[1])
            else
                local eggs = GetFieldEggs()
                local total = 0
                for _, e in ipairs(eggs) do total = total + e.Value end
                status.Text = string.format("[Field] %d egg(s) · total %s", #eggs, formatRate(total))
                if #eggs == 0 then
                    buildEmpty("No field eggs found.\nTry refreshing in a moment.")
                    showPreview(nil)
                    return
                end
                local limit = math.min(#eggs, 60)
                for i = 1, limit do buildRow(i, eggs[i]) end
                showPreview(eggs[1])
            end
        end

        M.refresh = refresh

        backpackTab.MouseButton1Click:Connect(function()
            M.ActiveTab = "Backpack"
            updateTabVisuals(); updateSortLabel(); refresh()
        end)
        fieldTab.MouseButton1Click:Connect(function()
            M.ActiveTab = "Field"
            updateTabVisuals(); updateSortLabel(); refresh()
        end)
        sortBtn.MouseButton1Click:Connect(function()
            if M.ActiveTab == "Backpack" then
                local idx = table.find(backpackPriorities, M.Sort) or 1
                idx = idx % #backpackPriorities + 1
                M.Sort = backpackPriorities[idx]
            else
                local idx = table.find(fieldPriorities, M.Priority) or 1
                idx = idx % #fieldPriorities + 1
                M.Priority = fieldPriorities[idx]
            end
            updateSortLabel(); refresh()
        end)
        refreshBtn.MouseButton1Click:Connect(function()
            refreshBtn.Text = "..."
            refresh()
            task.delay(0.6, function()
                if refreshBtn.Parent then refreshBtn.Text = "REFRESH" end
            end)
        end)

        updateTabVisuals()
        updateSortLabel()
        task.defer(refresh)

        task.spawn(function()
            while card.Parent do
                task.wait(3)
                if card.Parent then pcall(refresh) end
            end
        end)
    end

    return M
end)()

-- ============================================================
-- NOTIFICATIONS
-- ============================================================
local function createNotifier()
    local notifGui = Instance.new("ScreenGui")
    notifGui.Name="BabisUIKitNotif"; notifGui.ResetOnSpawn=false; notifGui.IgnoreGuiInset=true
    notifGui.ZIndexBehavior=Enum.ZIndexBehavior.Sibling; notifGui.DisplayOrder=300
    notifGui.Parent=uiParent
    protectInstance(notifGui)
    local sound = Instance.new("Sound")
    sound.SoundId=CONFIG.notifySoundId; sound.Volume=CONFIG.notifySoundVol
    sound.Parent=SoundService
    local container = Instance.new("Frame")
    container.BackgroundTransparency=1; container.Size=UDim2.new(0,320,0,0)
    container.AutomaticSize=Enum.AutomaticSize.Y; container.AnchorPoint=Vector2.new(1,0)
    container.Position=UDim2.new(1,-20,0,20); container.Parent=notifGui
    local layout = Instance.new("UIListLayout")
    layout.FillDirection=Enum.FillDirection.Vertical; layout.Padding=UDim.new(0,10)
    layout.SortOrder=Enum.SortOrder.LayoutOrder; layout.VerticalAlignment=Enum.VerticalAlignment.Top
    layout.HorizontalAlignment=Enum.HorizontalAlignment.Right; layout.Parent=container
    local BLUE = CONFIG.colors.accent
    local order = 0
    local function dismiss(row)
        if not row or not row.Parent then return end
        if row:GetAttribute("Dismissing") then return end
        row:SetAttribute("Dismissing", true)
        local card = row:FindFirstChild("Card")
        if card then
            tween(card, {Position=UDim2.new(1,420,0,0), BackgroundTransparency=0.7}, 0.34, Enum.EasingStyle.Quint, Enum.EasingDirection.In)
            for _, d in ipairs(card:GetDescendants()) do
                if d:IsA("TextLabel") then tween(d, {TextTransparency=1}, 0.28)
                elseif d:IsA("ImageLabel") then tween(d, {ImageTransparency=1}, 0.28)
                elseif d:IsA("Frame") then tween(d, {BackgroundTransparency=1}, 0.28)
                elseif d:IsA("UIStroke") then tween(d, {Transparency=1}, 0.28) end
            end
        end
        tween(row, {Size=UDim2.new(1, 0, 0, 0)}, 0.34, Enum.EasingStyle.Quint, Enum.EasingDirection.In)
        task.delay(0.38, function() if row and row.Parent then row:Destroy() end end)
    end
    local function notify(title, message, kind, duration)
        title    = title    or "Notification"
        message  = message  or ""
        kind     = kind     or "info"
        duration = duration or 2.0
        if type(kind) == "number" then duration = kind; kind = "info" end
        if sound.IsPlaying then sound:Stop() end
        sound.TimePosition=0; sound:Play()
        order = order + 1
        local accentColor = BLUE
        local row = Instance.new("Frame")
        row.Size=UDim2.new(1,0,0,76); row.BackgroundTransparency=1
        row.ClipsDescendants=false; row.LayoutOrder=order; row.Parent=container
        local card = Instance.new("Frame")
        card.Name="Card"; card.Size=UDim2.new(1,0,1,0); card.Position=UDim2.new(1,380,0,0)
        card.BackgroundColor3=Color3.fromRGB(18,20,26); card.BorderSizePixel=0; card.Parent=row
        corner(card,12); stroke(card, Color3.fromRGB(56,55,62), 1.5, 0)
        local bar = Instance.new("Frame")
        bar.Size=UDim2.new(0,5,1,-22); bar.Position=UDim2.new(0,9,0,11)
        bar.BackgroundColor3=accentColor; bar.BorderSizePixel=0; bar.Parent=card; corner(bar,3)
        local bellBg = Instance.new("Frame")
        bellBg.Size=UDim2.new(0,42,0,42); bellBg.Position=UDim2.new(0,20,0.5,-21)
        bellBg.BackgroundColor3=CONFIG.colors.bellBg; bellBg.BorderSizePixel=0; bellBg.Parent=card
        corner(bellBg,21); stroke(bellBg, accentColor, 1.2, 0.4)
        local bell = Instance.new("ImageLabel")
        bell.Size=UDim2.new(0,28,0,28); bell.Position=UDim2.new(0.5,-14,0.5,-14)
        bell.BackgroundTransparency=1; bell.Image=CONFIG.bellIconId
        bell.ImageColor3=accentColor; bell.ScaleType=Enum.ScaleType.Fit; bell.Parent=bellBg
        local titleLbl = Instance.new("TextLabel")
        titleLbl.BackgroundTransparency=1; titleLbl.Text=title
        titleLbl.TextColor3=Color3.fromRGB(245,248,255); titleLbl.Font=Enum.Font.GothamBold
        titleLbl.TextSize=16; titleLbl.TextXAlignment=Enum.TextXAlignment.Left
        titleLbl.TextYAlignment=Enum.TextYAlignment.Top
        titleLbl.Size=UDim2.new(1,-90,0,22); titleLbl.Position=UDim2.new(0,72,0,16)
        titleLbl.Parent=card
        local msgLbl = Instance.new("TextLabel")
        msgLbl.BackgroundTransparency=1; msgLbl.Text=message
        msgLbl.TextColor3=Color3.fromRGB(160,165,185); msgLbl.Font=Enum.Font.Gotham
        msgLbl.TextSize=14; msgLbl.TextXAlignment=Enum.TextXAlignment.Left
        msgLbl.TextYAlignment=Enum.TextYAlignment.Top; msgLbl.TextWrapped=true
        msgLbl.Size=UDim2.new(1,-90,0,34); msgLbl.Position=UDim2.new(0,72,0,38)
        msgLbl.Parent=card
        tween(card, {Position=UDim2.new(0,0,0,0)}, 0.35)
        task.spawn(function()
            task.wait(0.4)
            if not bell.Parent then return end
            bell.Rotation=-12
            tween(bell,{Rotation=12},0.10); task.wait(0.11)
            if not bell.Parent then return end
            tween(bell,{Rotation=-8},0.10); task.wait(0.11)
            if not bell.Parent then return end
            tween(bell,{Rotation=6},0.10); task.wait(0.11)
            if not bell.Parent then return end
            tween(bell,{Rotation=0},0.12)
        end)
        task.spawn(function() task.wait(duration); dismiss(row) end)
        return row
    end
    return notify
end

-- ============================================================
-- MAIN UI
-- ============================================================
local function buildMainUI()
    local notify = createNotifier()
    local savedPos = loadPosition() or UDim2.new(0.5, 0, 0.5, 0)

    local UI = {
        activeTab = CONFIG.defaultTab, connections = {}, isMinimized = false,
        isOpen = false, destroyed = false, options = {}, infoOpen = false,
        cfgSelected = nil, cfgList = {}, savedPos = savedPos, everOpened = false,
        refreshCfgList = nil, updateAutoloadText = nil,
    }
    local function track(conn) table.insert(UI.connections, conn); return conn end
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name="BabisUIKit"; screenGui.ResetOnSpawn=false; screenGui.IgnoreGuiInset=true
    screenGui.ZIndexBehavior=Enum.ZIndexBehavior.Sibling; screenGui.DisplayOrder=10000
    screenGui.Parent=uiParent
    protectInstance(screenGui)
    local scaleWrapper = Instance.new("Frame")
    scaleWrapper.Name = "Root"
    scaleWrapper.BackgroundTransparency=1
    scaleWrapper.Size=UDim2.new(0, CONFIG.sizes.windowWidth, 0, 0)
    scaleWrapper.Position=savedPos
    scaleWrapper.AnchorPoint=Vector2.new(0.5, 0.5)
    scaleWrapper.ClipsDescendants=true
    scaleWrapper.Parent=screenGui
    protectInstance(scaleWrapper)
    local uiScale = Instance.new("UIScale"); uiScale.Parent=scaleWrapper
    local mainWindow = Instance.new("Frame")
    mainWindow.Size=UDim2.new(1,0,1,0); mainWindow.BackgroundColor3=CONFIG.colors.windowBg
    mainWindow.BorderSizePixel=0; mainWindow.ClipsDescendants=true
    mainWindow.BackgroundTransparency=1; mainWindow.Parent=scaleWrapper
    protectInstance(mainWindow)
    corner(mainWindow, CONFIG.sizes.windowRadius)

    local function getViewport()
        local cam = workspace.CurrentCamera
        local vp = cam and cam.ViewportSize
        if not vp or vp.X <= 0 or vp.Y <= 0 then return Vector2.new(1920, 1080) end
        return vp
    end
    local function currentLogicalHeight()
        return UI.isMinimized and (CONFIG.sizes.headerHeight + 2) or CONFIG.sizes.windowHeight
    end
    local function currentPixelSize()
        local s = uiScale.Scale or 1
        return CONFIG.sizes.windowWidth * s, currentLogicalHeight() * s
    end
    local function clampCenter(cx, cy, w, h)
        local vp = getViewport()
        local halfW, halfH = w / 2, h / 2
        local minCX, maxCX = halfW, vp.X - halfW
        local minCY, maxCY = halfH, vp.Y - halfH
        if maxCX < minCX then cx = vp.X / 2 else cx = math.clamp(cx, minCX, maxCX) end
        if maxCY < minCY then cy = vp.Y / 2 else cy = math.clamp(cy, minCY, maxCY) end
        return cx, cy
    end
    local function currentCenterPixels()
        local vp = getViewport()
        local cur = scaleWrapper.Position
        return cur.X.Scale * vp.X + cur.X.Offset, cur.Y.Scale * vp.Y + cur.Y.Offset
    end

    local header = Instance.new("Frame")
    header.Name = "Header"
    header.Size=UDim2.new(1,0,0,CONFIG.sizes.headerHeight)
    header.BackgroundColor3=CONFIG.colors.headerTint; header.BackgroundTransparency=1
    header.BorderSizePixel=0; header.Parent=mainWindow
    protectInstance(header)
    corner(header, CONFIG.sizes.windowRadius)
    local headerFlat = Instance.new("Frame")
    headerFlat.Size=UDim2.new(1,0,0,16); headerFlat.Position=UDim2.new(0,0,1,-16)
    headerFlat.BackgroundColor3=CONFIG.colors.headerTint; headerFlat.BackgroundTransparency=1
    headerFlat.BorderSizePixel=0; headerFlat.Parent=header
    protectInstance(headerFlat)
    local appLabel = Instance.new("TextLabel")
    appLabel.BackgroundTransparency=1; appLabel.Text=CONFIG.appName
    appLabel.TextColor3=CONFIG.colors.accent; appLabel.Font=CONFIG.fonts.appName
    appLabel.TextSize=CONFIG.textSizes.appName; appLabel.TextXAlignment=Enum.TextXAlignment.Left
    appLabel.TextYAlignment=Enum.TextYAlignment.Center
    appLabel.Size=UDim2.new(0,400,0,34); appLabel.Position=UDim2.new(0,22,0,8)
    appLabel.Parent=header
    local subtitleLabel = Instance.new("TextLabel")
    subtitleLabel.BackgroundTransparency=1; subtitleLabel.Text=CONFIG.edition
    subtitleLabel.TextColor3=Color3.fromRGB(255,255,255); subtitleLabel.Font=Enum.Font.GothamBold
    subtitleLabel.TextSize=14; subtitleLabel.TextXAlignment=Enum.TextXAlignment.Left
    subtitleLabel.TextYAlignment=Enum.TextYAlignment.Center
    subtitleLabel.Size=UDim2.new(0,220,0,18); subtitleLabel.Position=UDim2.new(0,24,0,42)
    subtitleLabel.Parent=header
    local versionLabel = Instance.new("TextLabel")
    versionLabel.BackgroundTransparency=1; versionLabel.Text=CONFIG.version
    versionLabel.TextColor3=CONFIG.colors.textMuted; versionLabel.Font=CONFIG.fonts.version
    versionLabel.TextSize=13; versionLabel.TextXAlignment=Enum.TextXAlignment.Left
    versionLabel.TextYAlignment=Enum.TextYAlignment.Center
    versionLabel.Size=UDim2.new(0,200,0,16); versionLabel.Position=UDim2.new(0,24,0,60)
    versionLabel.Parent=header
    local infoBtn = Instance.new("TextButton")
    infoBtn.Size=UDim2.new(0,60,0,60); infoBtn.Position=UDim2.new(1,-182,0,12)
    infoBtn.BackgroundTransparency=1; infoBtn.Text=""; infoBtn.AutoButtonColor=false
    infoBtn.Parent=header
    local infoIcon = Instance.new("ImageLabel")
    infoIcon.Size=UDim2.new(0,30,0,30); infoIcon.Position=UDim2.new(0.5,-15,0.5,-15)
    infoIcon.BackgroundTransparency=1; infoIcon.Image=CONFIG.infoIconId
    infoIcon.ImageColor3=CONFIG.colors.navIcon; infoIcon.ScaleType=Enum.ScaleType.Fit
    infoIcon.Parent=infoBtn
    local minBtn = Instance.new("TextButton")
    minBtn.Size=UDim2.new(0,60,0,60); minBtn.Position=UDim2.new(1,-122,0,12)
    minBtn.BackgroundTransparency=1; minBtn.Text=""; minBtn.AutoButtonColor=false
    minBtn.Parent=header
    local minGlyph = Instance.new("Frame")
    minGlyph.Size=UDim2.new(0,22,0,3); minGlyph.Position=UDim2.new(0.5,-11,0.5,-1)
    minGlyph.BackgroundColor3=CONFIG.colors.textPrimary; minGlyph.BorderSizePixel=0
    minGlyph.Parent=minBtn; corner(minGlyph, 2)
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size=UDim2.new(0,60,0,60); closeBtn.Position=UDim2.new(1,-62,0,12)
    closeBtn.BackgroundTransparency=1; closeBtn.Text=""; closeBtn.AutoButtonColor=false
    closeBtn.Parent=header
    local xLen, xThk = 26, 3
    local xBar1 = Instance.new("Frame")
    xBar1.Size=UDim2.new(0,xLen,0,xThk); xBar1.Position=UDim2.new(0.5,-xLen/2,0.5,-xThk/2)
    xBar1.BackgroundColor3=CONFIG.colors.textPrimary; xBar1.BorderSizePixel=0
    xBar1.Rotation=45; xBar1.Parent=closeBtn; corner(xBar1, 2)
    local xBar2 = Instance.new("Frame")
    xBar2.Size=UDim2.new(0,xLen,0,xThk); xBar2.Position=UDim2.new(0.5,-xLen/2,0.5,-xThk/2)
    xBar2.BackgroundColor3=CONFIG.colors.textPrimary; xBar2.BorderSizePixel=0
    xBar2.Rotation=-45; xBar2.Parent=closeBtn; corner(xBar2, 2)
    local divider = Instance.new("Frame")
    divider.Size=UDim2.new(1,0,0,2); divider.Position=UDim2.new(0,0,0,CONFIG.sizes.headerHeight)
    divider.BackgroundColor3=CONFIG.colors.divider; divider.BorderSizePixel=0
    divider.ZIndex=2; divider.Parent=mainWindow
    protectInstance(divider)
    local divGrad = Instance.new("UIGradient")
    divGrad.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.85), NumberSequenceKeypoint.new(0.5, 0), NumberSequenceKeypoint.new(1, 0.85) })
    divGrad.Parent = divider
    local bodyLayer = Instance.new("Frame")
    bodyLayer.Size=UDim2.new(1,0,1,0); bodyLayer.BackgroundTransparency=1
    bodyLayer.BorderSizePixel=0; bodyLayer.ClipsDescendants=false
    bodyLayer.Visible=false; bodyLayer.Parent=mainWindow
    protectInstance(bodyLayer)
    local navBar = Instance.new("Frame")
    navBar.Size=UDim2.new(1,-CONFIG.sizes.navPadX*2,0,CONFIG.sizes.navHeight)
    navBar.Position=UDim2.new(0,CONFIG.sizes.navPadX,0,CONFIG.sizes.headerHeight+12)
    navBar.BackgroundTransparency=1; navBar.Parent=bodyLayer
    protectInstance(navBar)
    local navLayout = Instance.new("UIListLayout")
    navLayout.FillDirection=Enum.FillDirection.Horizontal
    navLayout.HorizontalAlignment=Enum.HorizontalAlignment.Center
    navLayout.VerticalAlignment=Enum.VerticalAlignment.Center
    navLayout.Padding=UDim.new(0,CONFIG.sizes.navGap)
    navLayout.SortOrder=Enum.SortOrder.LayoutOrder
    navLayout.Parent=navBar
    local contentContainer = Instance.new("Frame")
    contentContainer.Size=UDim2.new(1,-CONFIG.sizes.contentPadX*2,1,-(CONFIG.sizes.headerHeight+CONFIG.sizes.navHeight+30))
    contentContainer.Position=UDim2.new(0,CONFIG.sizes.contentPadX,0,CONFIG.sizes.headerHeight+CONFIG.sizes.navHeight+20)
    contentContainer.BackgroundTransparency=1; contentContainer.Parent=bodyLayer
    protectInstance(contentContainer)
    local scroll = Instance.new("ScrollingFrame")
    scroll.Size=UDim2.new(1,0,1,0); scroll.BackgroundTransparency=1
    scroll.BorderSizePixel=0; scroll.ScrollBarThickness=4
    scroll.ScrollBarImageColor3=CONFIG.colors.accent; scroll.ScrollBarImageTransparency=0.3
    scroll.CanvasSize=UDim2.new(0,0,0,0); scroll.AutomaticCanvasSize=Enum.AutomaticSize.Y
    scroll.ClipsDescendants=true; scroll.Parent=contentContainer
    local contentLayout = Instance.new("UIListLayout")
    contentLayout.FillDirection=Enum.FillDirection.Vertical; contentLayout.Padding=UDim.new(0,CONFIG.sizes.cardGap)
    contentLayout.SortOrder=Enum.SortOrder.LayoutOrder; contentLayout.HorizontalAlignment=Enum.HorizontalAlignment.Center
    contentLayout.Parent=scroll
    local contentPad = Instance.new("UIPadding")
    contentPad.PaddingTop=UDim.new(0,6); contentPad.PaddingBottom=UDim.new(0,16)
    contentPad.PaddingLeft=UDim.new(0,CONFIG.sizes.scrollPadL); contentPad.PaddingRight=UDim.new(0,CONFIG.sizes.scrollPadR)
    contentPad.Parent=scroll

    local infoFrame = Instance.new("Frame")
    infoFrame.Name = "InfoFrame"
    infoFrame.Size=UDim2.new(1,-CONFIG.sizes.contentPadX*2,1,-(CONFIG.sizes.headerHeight+20))
    infoFrame.Position=UDim2.new(0,CONFIG.sizes.contentPadX,0,CONFIG.sizes.headerHeight+10)
    infoFrame.BackgroundColor3=CONFIG.colors.cardBg; infoFrame.BorderSizePixel=0
    infoFrame.Visible=false; infoFrame.BackgroundTransparency=1; infoFrame.Parent=bodyLayer
    corner(infoFrame, CONFIG.sizes.cardRadius)
    local infoStroke = stroke(infoFrame, CONFIG.colors.cardBorder, 1.5, 1)

    local aboutTitle = Instance.new("TextLabel")
    aboutTitle.BackgroundTransparency=1; aboutTitle.Text="About"
    aboutTitle.TextColor3=CONFIG.colors.accent; aboutTitle.Font=Enum.Font.GothamBold
    aboutTitle.TextSize=CONFIG.textSizes.aboutTitle; aboutTitle.TextXAlignment=Enum.TextXAlignment.Left
    aboutTitle.TextYAlignment=Enum.TextYAlignment.Center
    aboutTitle.Size=UDim2.new(1,-32,0,40); aboutTitle.Position=UDim2.new(0,18,0,20)
    aboutTitle.TextTransparency=1; aboutTitle.Parent=infoFrame

    local aboutDesc = Instance.new("TextLabel")
    aboutDesc.BackgroundTransparency=1
    aboutDesc.Text = "Babis | FREE (custom)\n\nMain: Egg Predictor — live scan, sort, 3D preview.\nMisc: Bypass TP · Anti Ragdoll · Anti Trap.\nSettings: keybind + configs."
    aboutDesc.TextColor3=CONFIG.colors.textPrimary; aboutDesc.Font=Enum.Font.Gotham
    aboutDesc.TextSize=CONFIG.textSizes.aboutDesc; aboutDesc.TextXAlignment=Enum.TextXAlignment.Left
    aboutDesc.TextYAlignment=Enum.TextYAlignment.Top; aboutDesc.TextWrapped=true
    aboutDesc.LineHeight=1.2; aboutDesc.Size=UDim2.new(1,-36,1,-160)
    aboutDesc.Position=UDim2.new(0,18,0,70); aboutDesc.TextTransparency=1
    aboutDesc.Parent=infoFrame

    local discordBtn = Instance.new("TextButton")
    discordBtn.Size=UDim2.new(1,-36,0,108); discordBtn.Position=UDim2.new(0,18,1,-128)
    discordBtn.BackgroundColor3=Color3.fromRGB(255,255,255); discordBtn.Text=""
    discordBtn.AutoButtonColor=false; discordBtn.BackgroundTransparency=1
    discordBtn.ClipsDescendants=true
    discordBtn.Parent=infoFrame; corner(discordBtn, 14)
    local dBtnGrad = Instance.new("Frame")
    dBtnGrad.Size=UDim2.new(1,0,1,0); dBtnGrad.BackgroundColor3=Color3.fromRGB(255,255,255)
    dBtnGrad.BorderSizePixel=0; dBtnGrad.ZIndex=1; dBtnGrad.Parent=discordBtn
    corner(dBtnGrad, 14)
    local dBtnGradFill = Instance.new("UIGradient")
    dBtnGradFill.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(88,101,242)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(22,24,45)) })
    dBtnGradFill.Parent = dBtnGrad
    local dBtnStroke = stroke(discordBtn, Color3.fromRGB(88,101,242), 1.5, 0.2)
    local dIcon = Instance.new("ImageLabel")
    dIcon.Size=UDim2.new(0,76,0,76); dIcon.Position=UDim2.new(0,16,0.5,-38)
    dIcon.BackgroundTransparency=1; dIcon.Image=CONFIG.discordIconId
    dIcon.ImageColor3=Color3.fromRGB(255,255,255); dIcon.ScaleType=Enum.ScaleType.Fit
    dIcon.ZIndex=2; dIcon.Parent=discordBtn
    local dTitle = Instance.new("TextLabel")
    dTitle.BackgroundTransparency=1; dTitle.Text="Discord"
    dTitle.TextColor3=Color3.fromRGB(255,255,255); dTitle.Font=Enum.Font.GothamBold
    dTitle.TextSize=CONFIG.textSizes.discordTitle; dTitle.TextXAlignment=Enum.TextXAlignment.Left
    dTitle.TextYAlignment=Enum.TextYAlignment.Center
    dTitle.Size=UDim2.new(1,-130,0,32); dTitle.Position=UDim2.new(0,108,0,22)
    dTitle.ZIndex=2; dTitle.Parent=discordBtn
    local dSub = Instance.new("TextLabel")
    dSub.BackgroundTransparency=1; dSub.Text="Tap to copy the invite"
    dSub.TextColor3=Color3.fromRGB(220,225,245); dSub.Font=Enum.Font.Gotham
    dSub.TextSize=CONFIG.textSizes.discordSub; dSub.TextXAlignment=Enum.TextXAlignment.Left
    dSub.TextYAlignment=Enum.TextYAlignment.Center
    dSub.Size=UDim2.new(1,-130,0,26); dSub.Position=UDim2.new(0,108,0,58)
    dSub.ZIndex=2; dSub.Parent=discordBtn

    discordBtn.MouseEnter:Connect(function()
        tween(dBtnGradFill, {Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(105,118,255)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(30,34,60)) })}, CONFIG.anim.fast)
    end)
    discordBtn.MouseLeave:Connect(function()
        tween(dBtnGradFill, {Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(88,101,242)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(22,24,45)) })}, CONFIG.anim.fast)
    end)

    local discordBusy = false
    discordBtn.Activated:Connect(function()
        if discordBusy then return end
        discordBusy = true
        local ok = pcall(function() setclipboard(CONFIG.discordInvite) end)
        popTween(discordBtn, {Size = UDim2.new(1, -28, 0, 114)}, 0.18)
        task.delay(0.18, function()
            tween(discordBtn, {Size = UDim2.new(1, -36, 0, 108)}, 0.16)
        end)
        local origTitle = dTitle.Text
        local origSub = dSub.Text
        if ok then
            dTitle.Text = "Copied!"
            dSub.Text = CONFIG.discordInvite
        else
            dTitle.Text = "Copy failed"
            dSub.Text = "Manual: " .. CONFIG.discordInvite
        end
        task.delay(1.6, function()
            dTitle.Text = origTitle
            dSub.Text = origSub
            discordBusy = false
        end)
        notify("Discord", ok and ("Invite copied: " .. CONFIG.discordInvite) or "Copy failed", "info")
    end)

    local function closeInfo()
        if not UI.infoOpen then return end
        UI.infoOpen = false
        tween(aboutTitle, {TextTransparency=1}, 0.18)
        tween(aboutDesc, {TextTransparency=1}, 0.18)
        tween(dTitle, {TextTransparency=1}, 0.18)
        tween(dSub, {TextTransparency=1}, 0.18)
        tween(dIcon, {ImageTransparency=1}, 0.18)
        tween(dBtnGrad, {BackgroundTransparency=1}, 0.18)
        tween(discordBtn, {BackgroundTransparency=1}, 0.18)
        tween(dBtnStroke, {Transparency=1}, 0.18)
        tween(infoFrame, {BackgroundTransparency=1}, 0.22)
        tween(infoStroke, {Transparency=1}, 0.22)
        task.delay(0.24, function()
            infoFrame.Visible = false
            if UI.isMinimized then
                safeWrite(navBar, "Visible", false)
                safeWrite(contentContainer, "Visible", false)
            else
                safeWrite(navBar, "Visible", true)
                safeWrite(contentContainer, "Visible", true)
            end
        end)
    end

    local function openInfo()
        if UI.infoOpen then closeInfo() return end
        if UI.isMinimized then
            UI.isMinimized = false
            safeWrite(bodyLayer, "Visible", true)
            local s = uiScale.Scale or 1
            local w, h = CONFIG.sizes.windowWidth * s, CONFIG.sizes.windowHeight * s
            local cx, cy = currentCenterPixels()
            local tcx, tcy = clampCenter(cx, cy, w, h)
            local target = UDim2.new(0, tcx, 0, tcy)
            UI.savedPos = target
            TweenService:Create(scaleWrapper, TweenInfo.new(CONFIG.anim.slow, CONFIG.anim.easing, CONFIG.anim.dir), {
                Size = UDim2.new(0, CONFIG.sizes.windowWidth, 0, CONFIG.sizes.windowHeight),
                Position = target,
            }):Play()
        end
        UI.infoOpen = true
        infoFrame.Visible = true
        infoFrame.BackgroundTransparency = 1
        infoStroke.Transparency = 1
        aboutTitle.TextTransparency = 1
        aboutDesc.TextTransparency = 1
        dTitle.TextTransparency = 1
        dSub.TextTransparency = 1
        dIcon.ImageTransparency = 1
        dBtnGrad.BackgroundTransparency = 1
        discordBtn.BackgroundTransparency = 1
        dBtnStroke.Transparency = 1
        safeWrite(navBar, "Visible", false)
        safeWrite(contentContainer, "Visible", false)
        tween(infoFrame, {BackgroundTransparency=0}, 0.28)
        tween(infoStroke, {Transparency=0}, 0.28)
        tween(aboutTitle, {TextTransparency=0}, 0.30)
        tween(aboutDesc, {TextTransparency=0}, 0.36)
        task.delay(0.10, function()
            tween(dBtnGrad, {BackgroundTransparency=0}, 0.28)
            tween(discordBtn, {BackgroundTransparency=0}, 0.28)
            tween(dBtnStroke, {Transparency=0.2}, 0.28)
            tween(dIcon, {ImageTransparency=0}, 0.30)
            tween(dTitle, {TextTransparency=0}, 0.30)
            tween(dSub, {TextTransparency=0}, 0.32)
        end)
    end
    infoBtn.Activated:Connect(openInfo)

    local tabButtons = {}
    local function createTabButton(tabData, order)
        local btn = Instance.new("TextButton")
        btn.Size=UDim2.new(0,CONFIG.sizes.navBtnSize,0,CONFIG.sizes.navBtnSize)
        btn.BackgroundColor3=CONFIG.colors.navBg; btn.Text=""; btn.AutoButtonColor=false
        btn.ClipsDescendants=false; btn.LayoutOrder=order; btn.Parent=navBar
        corner(btn, CONFIG.sizes.navBtnRadius)
        local btnStroke = stroke(btn, CONFIG.colors.navBorder, 2, 0)
        local baseSize = CONFIG.sizes.iconSize + (tabData.boost or 0)
        local iconGlow = Instance.new("ImageLabel")
        iconGlow.Size=UDim2.new(0,baseSize+8,0,baseSize+8)
        iconGlow.Position=UDim2.new(0.5,-(baseSize+8)/2,0.5,-(baseSize+8)/2)
        iconGlow.BackgroundTransparency=1; iconGlow.Image="rbxassetid://"..tabData.iconId
        iconGlow.ImageColor3=CONFIG.colors.navIcon; iconGlow.ImageTransparency=0.85
        iconGlow.ScaleType=Enum.ScaleType.Fit; iconGlow.ZIndex=2; iconGlow.Parent=btn
        local icon = Instance.new("ImageLabel")
        icon.Size=UDim2.new(0,baseSize,0,baseSize)
        icon.Position=UDim2.new(0.5,-baseSize/2,0.5,-baseSize/2)
        icon.BackgroundTransparency=1; icon.Image="rbxassetid://"..tabData.iconId
        icon.ImageColor3=CONFIG.colors.navIcon; icon.ScaleType=Enum.ScaleType.Fit
        icon.ZIndex=2; icon.Parent=btn
        local hit = Instance.new("TextButton")
        hit.Size=UDim2.new(1,0,1,1); hit.BackgroundTransparency=1; hit.Text=""
        hit.AutoButtonColor=false; hit.ZIndex=3; hit.Parent=btn
        return { button=btn, stroke=btnStroke, icon=icon, iconGlow=iconGlow, hit=hit, id=tabData.id, baseSize=baseSize }
    end
    for i, tab in ipairs(CONFIG.tabs) do tabButtons[tab.id] = createTabButton(tab, i) end

    local function createCard(order, height)
        local card = Instance.new("Frame")
        card.Size=UDim2.new(1,0,0,height or 128); card.BackgroundColor3=CONFIG.colors.cardBg
        card.BorderSizePixel=0; card.LayoutOrder=order; card.Parent=scroll
        corner(card, CONFIG.sizes.cardRadius)
        local cardStroke = stroke(card, CONFIG.colors.cardBorder, 1.5, 0)
        return card, cardStroke
    end

    local function createToggleCard(order, titleText, descText, key, default, onChange)
        local card, cardStroke = createCard(order, CONFIG.sizes.toggleCardHeight)
        local title = Instance.new("TextLabel")
        title.BackgroundTransparency=1; title.TextColor3=CONFIG.colors.textPrimary
        title.Font=Enum.Font.GothamBold; title.TextSize=CONFIG.textSizes.cardTitle
        title.TextXAlignment=Enum.TextXAlignment.Left; title.TextYAlignment=Enum.TextYAlignment.Top
        title.TextWrapped=true
        title.Size=UDim2.new(1,-170,0,34); title.Position=UDim2.new(0,CONFIG.sizes.cardPadX,0,22)
        title.Text=titleText; title.Parent=card
        local desc = Instance.new("TextLabel")
        desc.BackgroundTransparency=1; desc.TextColor3=CONFIG.colors.textDesc
        desc.Font=Enum.Font.Gotham; desc.TextSize=CONFIG.textSizes.cardDesc
        desc.TextXAlignment=Enum.TextXAlignment.Left; desc.TextYAlignment=Enum.TextYAlignment.Top
        desc.TextWrapped=true; desc.LineHeight=1.15
        desc.Size=UDim2.new(1,-170,0,60); desc.Position=UDim2.new(0,CONFIG.sizes.cardPadX,0,62)
        desc.Text=descText; desc.Parent=card
        if UI.options[key] == nil then UI.options[key] = default and true or false end
        local tW = CONFIG.sizes.toggleW; local tH = CONFIG.sizes.toggleH
        local knob = CONFIG.sizes.toggleKnob; local pad = (tH - knob)/2
        local right = CONFIG.sizes.toggleRight
        local track2 = Instance.new("Frame")
        track2.Size=UDim2.new(0,tW,0,tH); track2.Position=UDim2.new(1,-(tW+right),0,26)
        track2.BackgroundColor3=CONFIG.colors.toggleOff; track2.BorderSizePixel=0
        track2.Parent=card; corner(track2, tH/2)
        local trackStroke = stroke(track2, CONFIG.colors.cardBorder, 1.5, 0)
        local knobFrame = Instance.new("Frame")
        knobFrame.Size=UDim2.new(0,knob,0,knob); knobFrame.Position=UDim2.new(0,pad,0.5,-knob/2)
        knobFrame.BackgroundColor3=CONFIG.colors.textPrimary; knobFrame.BorderSizePixel=0
        knobFrame.Parent=track2; corner(knobFrame, knob/2)
        local hit = Instance.new("TextButton")
        hit.Size=UDim2.new(1,0,1,0); hit.BackgroundTransparency=1; hit.Text=""
        hit.AutoButtonColor=false; hit.ZIndex=10; hit.Parent=card
        local offX, onX = pad, tW - knob - pad
        local function render(animate)
            local dur = animate and CONFIG.anim.normal or 0
            if UI.options[key] then
                tween(track2, {BackgroundColor3=CONFIG.colors.toggleOn}, dur)
                tween(trackStroke, {Color=CONFIG.colors.toggleOn}, dur)
                tween(knobFrame, {Position=UDim2.new(0,onX,0.5,-knob/2)}, dur)
            else
                tween(track2, {BackgroundColor3=CONFIG.colors.toggleOff}, dur)
                tween(trackStroke, {Color=CONFIG.colors.cardBorder}, dur)
                tween(knobFrame, {Position=UDim2.new(0,offX,0.5,-knob/2)}, dur)
            end
        end
        render(false)
        track(hit.Activated:Connect(function()
            UI.options[key] = not UI.options[key]
            render(true)
            if onChange then onChange(UI.options[key], titleText) end
            notify(titleText, UI.options[key] and "Enabled" or "Disabled", "info")
        end))
        return card
    end

    local function smallButton(parent, y, text, onClick)
        local btn = Instance.new("TextButton")
        btn.Size=UDim2.new(1,-28,0,34); btn.Position=UDim2.new(0,14,0,y)
        btn.BackgroundColor3=CONFIG.colors.cfgBtnBg; btn.Text=text
        btn.TextColor3=CONFIG.colors.textPrimary; btn.Font=Enum.Font.GothamBold
        btn.TextSize=CONFIG.textSizes.cfgBtn; btn.AutoButtonColor=false
        btn.Parent=parent; corner(btn, 8)
        local s = stroke(btn, CONFIG.colors.cfgBtnBorder, 1.2, 0)
        btn.MouseEnter:Connect(function()
            tween(btn, {BackgroundColor3=CONFIG.colors.navBg}, CONFIG.anim.fast)
            tween(s, {Color=CONFIG.colors.accent}, CONFIG.anim.fast)
        end)
        btn.MouseLeave:Connect(function()
            tween(btn, {BackgroundColor3=CONFIG.colors.cfgBtnBg}, CONFIG.anim.fast)
            tween(s, {Color=CONFIG.colors.cfgBtnBorder}, CONFIG.anim.fast)
        end)
        btn.Activated:Connect(function() if onClick then onClick() end end)
        return btn
    end
    local function smallLabel(parent, y, text)
        local lbl = Instance.new("TextLabel")
        lbl.Size=UDim2.new(1,-28,0,22); lbl.Position=UDim2.new(0,14,0,y)
        lbl.BackgroundTransparency=1; lbl.Text=text
        lbl.TextColor3=CONFIG.colors.textPrimary; lbl.Font=Enum.Font.GothamBold
        lbl.TextSize=CONFIG.textSizes.cfgLabel; lbl.TextXAlignment=Enum.TextXAlignment.Left
        lbl.Parent=parent
        return lbl
    end

    local function buildKeybindCard()
        local card, _ = createCard(0, 110)
        local title = Instance.new("TextLabel")
        title.BackgroundTransparency=1; title.Text="Toggle Keybind"
        title.TextColor3=CONFIG.colors.textPrimary; title.Font=Enum.Font.GothamBold
        title.TextSize=24; title.TextXAlignment=Enum.TextXAlignment.Left
        title.TextYAlignment=Enum.TextYAlignment.Top
        title.Size=UDim2.new(1,-160,0,28); title.Position=UDim2.new(0,16,0,24)
        title.Parent=card
        local desc = Instance.new("TextLabel")
        desc.BackgroundTransparency=1; desc.Text="Key to open/close the interface"
        desc.TextColor3=CONFIG.colors.textDesc; desc.Font=Enum.Font.Gotham
        desc.TextSize=15; desc.TextXAlignment=Enum.TextXAlignment.Left
        desc.TextYAlignment=Enum.TextYAlignment.Top
        desc.Size=UDim2.new(1,-160,0,22); desc.Position=UDim2.new(0,16,0,60)
        desc.Parent=card
        local keyBtn = Instance.new("TextButton")
        keyBtn.Size=UDim2.new(0,130,0,46); keyBtn.Position=UDim2.new(1,-146,0.5,-23)
        keyBtn.BackgroundColor3=CONFIG.colors.navBg; keyBtn.Text=UI.options["toggle_key"] or "None"
        keyBtn.TextColor3=CONFIG.colors.textPrimary; keyBtn.Font=Enum.Font.GothamBold
        keyBtn.TextSize=18; keyBtn.AutoButtonColor=false; keyBtn.Parent=card
        corner(keyBtn, 10); local keyStroke = stroke(keyBtn, CONFIG.colors.cardBorder, 1.5, 0)
        local listenConn = nil
        local listening = false
        local function stopListening()
            listening = false
            if listenConn then pcall(function() listenConn:Disconnect() end) end
            listenConn = nil
            keyBtn.Text = UI.options["toggle_key"] or "None"
            tween(keyStroke, {Color=CONFIG.colors.cardBorder}, CONFIG.anim.fast)
        end
        keyBtn.Activated:Connect(function()
            if listening then return end
            listening = true
            keyBtn.Text = "Press a key..."
            tween(keyStroke, {Color=CONFIG.colors.accent}, CONFIG.anim.fast)
            listenConn = UserInputService.InputBegan:Connect(function(input, gpe)
                if gpe then return end
                if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
                UI.options["toggle_key"] = input.KeyCode.Name
                stopListening()
                notify("Keybind", "Set to " .. input.KeyCode.Name, "info")
            end)
        end)
    end

    local function buildConfigCard()
        local card, _ = createCard(1, CONFIG.sizes.cfgHeight)
        local icon = Instance.new("ImageLabel")
        icon.Size=UDim2.new(0,26,0,26); icon.Position=UDim2.new(0,14,0,14)
        icon.BackgroundTransparency=1; icon.Image=CONFIG.cfgIconId
        icon.ImageColor3=CONFIG.colors.cfgIcon; icon.ScaleType=Enum.ScaleType.Fit
        icon.Parent=card
        local headerLbl = Instance.new("TextLabel")
        headerLbl.Size=UDim2.new(1,-60,0,26); headerLbl.Position=UDim2.new(0,48,0,16)
        headerLbl.BackgroundTransparency=1; headerLbl.Text="Configuration"
        headerLbl.TextColor3=CONFIG.colors.textPrimary; headerLbl.Font=Enum.Font.GothamBold
        headerLbl.TextSize=CONFIG.textSizes.cfgTitle; headerLbl.TextXAlignment=Enum.TextXAlignment.Left
        headerLbl.Parent=card
        smallLabel(card, 54, "Config name")
        local nameBox = Instance.new("TextBox")
        nameBox.Size=UDim2.new(1,-28,0,38); nameBox.Position=UDim2.new(0,14,0,78)
        nameBox.BackgroundColor3=CONFIG.colors.cfgBtnBg; nameBox.Text=""
        nameBox.PlaceholderText="type a name..."; nameBox.PlaceholderColor3=CONFIG.colors.textMuted
        nameBox.TextColor3=CONFIG.colors.textPrimary; nameBox.Font=Enum.Font.GothamBold
        nameBox.TextSize=CONFIG.textSizes.cfgInput; nameBox.TextXAlignment=Enum.TextXAlignment.Left
        nameBox.ClearTextOnFocus=false; nameBox.Parent=card
        corner(nameBox, 8); local nameStroke = stroke(nameBox, CONFIG.colors.cfgBtnBorder, 1.2, 0)
        local padLeft = Instance.new("UIPadding"); padLeft.PaddingLeft=UDim.new(0,12); padLeft.Parent=nameBox
        nameBox.Focused:Connect(function() tween(nameStroke, {Color=CONFIG.colors.accent}, CONFIG.anim.fast) end)
        nameBox.FocusLost:Connect(function() tween(nameStroke, {Color=CONFIG.colors.cfgBtnBorder}, CONFIG.anim.fast) end)
        smallButton(card, 126, "Create config", function()
            local name = nameBox.Text
            if name == "" then notify("Config", "Enter a name first", "info") return end
            local ok, err = ConfigSys.save(name, UI.options)
            if ok then
                notify("Config", "\"" .. name .. "\" created", "info")
                nameBox.Text = ""
                if UI.refreshCfgList then UI.refreshCfgList() end
            else
                notify("Config", "Failed: " .. tostring(err), "info")
            end
        end)
        smallLabel(card, 176, "Config list")
        local ddBtn = Instance.new("TextButton")
        ddBtn.Size=UDim2.new(1,-28,0,38); ddBtn.Position=UDim2.new(0,14,0,200)
        ddBtn.BackgroundColor3=CONFIG.colors.cfgBtnBg
        ddBtn.Text = UI.cfgSelected or "---"
        ddBtn.TextColor3=CONFIG.colors.textPrimary; ddBtn.Font=Enum.Font.GothamBold
        ddBtn.TextSize=CONFIG.textSizes.cfgInput; ddBtn.TextXAlignment=Enum.TextXAlignment.Left
        ddBtn.AutoButtonColor=false; ddBtn.Parent=card
        corner(ddBtn, 8); local ddStroke = stroke(ddBtn, CONFIG.colors.cfgBtnBorder, 1.2, 0)
        local ddPad = Instance.new("UIPadding"); ddPad.PaddingLeft=UDim.new(0,12); ddPad.Parent=ddBtn
        local arrow = Instance.new("TextLabel")
        arrow.Size=UDim2.new(0,24,0,38); arrow.Position=UDim2.new(1,-30,0,0)
        arrow.BackgroundTransparency=1; arrow.Text="^"
        arrow.TextColor3=CONFIG.colors.textDesc; arrow.Font=Enum.Font.GothamBold
        arrow.TextSize=16; arrow.Rotation=180; arrow.Parent=ddBtn
        local ddListOpen = false
        local ddBackdrop, ddList = nil, nil
        local function closeDd()
            if not ddListOpen then return end
            ddListOpen = false
            if ddBackdrop and ddBackdrop.Parent then ddBackdrop:Destroy() end
            if ddList and ddList.Parent then ddList:Destroy() end
            ddBackdrop, ddList = nil, nil
            tween(ddStroke, {Color=CONFIG.colors.cfgBtnBorder}, CONFIG.anim.fast)
            tween(arrow, {Rotation=180}, CONFIG.anim.fast)
        end
        local function openDd()
            if ddListOpen then closeDd() return end
            ddListOpen = true
            ddBackdrop = Instance.new("TextButton")
            ddBackdrop.Size=UDim2.new(1,0,1,0); ddBackdrop.BackgroundTransparency=1
            ddBackdrop.Text=""; ddBackdrop.AutoButtonColor=false; ddBackdrop.ZIndex=60
            ddBackdrop.Parent=scaleWrapper
            ddBackdrop.Activated:Connect(closeDd)
            local list = ConfigSys.list()
            UI.cfgList = list
            if #list == 0 then
                notify("Config", "No configs found", "info")
                closeDd()
                return
            end
            local btnAbs = ddBtn.AbsolutePosition
            local wrapAbs = scaleWrapper.AbsolutePosition
            local s = uiScale.Scale
            local relX = (btnAbs.X - wrapAbs.X) / s
            local relW = ddBtn.AbsoluteSize.X / s
            local btnTopY = (btnAbs.Y - wrapAbs.Y) / s
            local itemH = 34
            local listH = #list * itemH + 8
            local relY = btnTopY - listH - 6
            ddList = Instance.new("Frame")
            ddList.Size=UDim2.new(0, relW, 0, listH)
            ddList.Position=UDim2.new(0, relX, 0, relY)
            ddList.BackgroundColor3=CONFIG.colors.cardBg; ddList.BorderSizePixel=0
            ddList.ZIndex=61; ddList.Parent=scaleWrapper
            corner(ddList, 10); stroke(ddList, CONFIG.colors.accent, 1.5, 0)
            local pad = Instance.new("UIPadding")
            pad.PaddingTop=UDim.new(0,4); pad.PaddingBottom=UDim.new(0,4); pad.Parent=ddList
            local lay = Instance.new("UIListLayout"); lay.FillDirection=Enum.FillDirection.Vertical; lay.Parent=ddList
            for i, name in ipairs(list) do
                local item = Instance.new("TextButton")
                item.Size=UDim2.new(1,-8,0,itemH); item.Position=UDim2.new(0,4,0,0)
                item.BackgroundTransparency=1; item.Text=name
                item.TextColor3 = (name == UI.cfgSelected) and CONFIG.colors.accent or CONFIG.colors.textPrimary
                item.Font=Enum.Font.GothamBold; item.TextSize=16
                item.TextXAlignment=Enum.TextXAlignment.Left; item.AutoButtonColor=false
                item.LayoutOrder=i; item.ZIndex=62; item.Parent=ddList; corner(item, 6)
                local ipad = Instance.new("UIPadding"); ipad.PaddingLeft=UDim.new(0,10); ipad.Parent=item
                item.MouseEnter:Connect(function()
                    tween(item, {BackgroundTransparency=0, BackgroundColor3=CONFIG.colors.navBg}, CONFIG.anim.fast)
                end)
                item.MouseLeave:Connect(function()
                    tween(item, {BackgroundTransparency=1}, CONFIG.anim.fast)
                end)
                item.Activated:Connect(function()
                    UI.cfgSelected = name
                    ddBtn.Text = name
                    closeDd()
                end)
            end
            tween(ddStroke, {Color=CONFIG.colors.accent}, CONFIG.anim.fast)
            tween(arrow, {Rotation=0}, CONFIG.anim.fast)
        end
        ddBtn.Activated:Connect(openDd)
        smallButton(card, 250, "Load config", function()
            local name = UI.cfgSelected
            if not name then notify("Config", "Select a config first", "info") return end
            local data = ConfigSys.load(name)
            if not data then notify("Config", "Failed to load", "info") return end
            for k, v in pairs(data) do UI.options[k] = v end
            if UI.rebuildTab then UI.rebuildTab() end
            notify("Config", "\"" .. name .. "\" loaded", "info")
        end)
        smallButton(card, 288, "Overwrite config", function()
            local name = UI.cfgSelected
            if not name then notify("Config", "Select a config first", "info") return end
            local ok = ConfigSys.save(name, UI.options)
            if ok then notify("Config", "\"" .. name .. "\" overwritten", "info")
            else notify("Config", "Failed", "info") end
        end)
        smallButton(card, 326, "Delete config", function()
            local name = UI.cfgSelected
            if not name then notify("Config", "Select a config first", "info") return end
            ConfigSys.delete(name)
            UI.cfgSelected = nil
            ddBtn.Text = "---"
            if UI.refreshCfgList then UI.refreshCfgList() end
            notify("Config", "\"" .. name .. "\" deleted", "info")
        end)
        smallButton(card, 364, "Refresh list", function()
            if UI.refreshCfgList then UI.refreshCfgList() end
            notify("Config", "List refreshed", "info")
        end)
        smallButton(card, 402, "Set as autoload", function()
            local name = UI.cfgSelected
            if not name then notify("Config", "Select a config first", "info") return end
            ConfigSys.setAutoload(name)
            if UI.updateAutoloadText then UI.updateAutoloadText() end
            notify("Config", "\"" .. name .. "\" set as autoload", "info")
        end)
        smallButton(card, 440, "Reset autoload", function()
            ConfigSys.setAutoload(nil)
            if UI.updateAutoloadText then UI.updateAutoloadText() end
            notify("Config", "Autoload cleared", "info")
        end)
        local autoloadText = Instance.new("TextLabel")
        autoloadText.Size=UDim2.new(1,-28,0,22); autoloadText.Position=UDim2.new(0,14,0,482)
        autoloadText.BackgroundTransparency=1
        autoloadText.Text="Current autoload config: " .. (ConfigSys.getAutoload() or "none")
        autoloadText.TextColor3=CONFIG.colors.textDesc; autoloadText.Font=Enum.Font.GothamBold
        autoloadText.TextSize=CONFIG.textSizes.cfgHint
        autoloadText.TextXAlignment=Enum.TextXAlignment.Left
        autoloadText.Parent=card
        UI.refreshCfgList = function()
            UI.cfgList = ConfigSys.list()
            if UI.cfgSelected then
                local found = false
                for _, n in ipairs(UI.cfgList) do
                    if n == UI.cfgSelected then found = true break end
                end
                if not found then UI.cfgSelected = nil end
            end
            ddBtn.Text = UI.cfgSelected or "---"
        end
        UI.updateAutoloadText = function()
            autoloadText.Text = "Current autoload config: " .. (ConfigSys.getAutoload() or "none")
        end
    end

    local function clearContent()
        for _, c in ipairs(scroll:GetChildren()) do
            if not c:IsA("UIListLayout") and not c:IsA("UIPadding") then c:Destroy() end
        end
    end

    local function buildMainTab()
        clearContent()
        local card = Instance.new("Frame")
        card.Name = "EggPredictorCard"
        card.Size = UDim2.new(1, 0, 0, 546)
        card.BackgroundColor3 = CONFIG.colors.cardBg
        card.BorderSizePixel = 0
        card.LayoutOrder = 1
        card.Parent = scroll
        corner(card, CONFIG.sizes.cardRadius)
        stroke(card, CONFIG.colors.cardBorder, 1.5, 0)
        EggPredictor.BuildInto(card, notify)
    end

    local function buildEmptyTab(message)
        clearContent()
        local card = Instance.new("Frame")
        card.Size = UDim2.new(1, 0, 0, 140); card.BackgroundColor3 = CONFIG.colors.cardBg
        card.BorderSizePixel = 0; card.LayoutOrder = 1; card.Parent = scroll
        corner(card, CONFIG.sizes.cardRadius)
        stroke(card, CONFIG.colors.cardBorder, 1.5, 0)
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(1,-32,1,-24); lbl.Position = UDim2.new(0,16,0,12)
        lbl.BackgroundTransparency = 1; lbl.Text = message
        lbl.TextColor3 = CONFIG.colors.textDesc; lbl.Font = Enum.Font.Gotham
        lbl.TextSize = 15; lbl.TextWrapped = true
        lbl.TextXAlignment = Enum.TextXAlignment.Center; lbl.TextYAlignment = Enum.TextYAlignment.Center
        lbl.Parent = card
    end

    local function buildVisualTab() buildEmptyTab("Coming soon.") end
    local function buildPlayerTab() buildEmptyTab("Coming soon.") end

    local function buildMiscTab()
        clearContent()
        createToggleCard(1, "Bypass TP",
            "Anti-die + humanoid bypass. Quando você agarra um ovo, o script carrega pra um ponto seguro longe, re-agarra, e volta pra base X. Bloqueia morte por queda / colisão.",
            "bypass_tp", false,
            function(v) BypassTP:SetEnabled(v) end)
        createToggleCard(2, "Anti Ragdoll",
            "Força GettingUp no instante que você entra em ragdoll. Restaura Motor6Ds e constraints automaticamente.",
            "anti_ragdoll", true,
            function(v) BypassTP:SetAntiRagdoll(v) end)
        createToggleCard(3, "Anti Trap",
            "Carrega o patch de anti-trap da comunidade uma vez. Bloqueia traps comuns de fake-steal no campo.",
            "anti_trap", true,
            function(v) BypassTP:SetAntiTrap(v) end)
    end

    local function buildSettingsTab()
        clearContent()
        buildKeybindCard()
        buildConfigCard()
    end

    local function buildTab(tabId)
        if tabId == "main" then buildMainTab()
        elseif tabId == "visual" then buildVisualTab()
        elseif tabId == "player" then buildPlayerTab()
        elseif tabId == "misc" then buildMiscTab()
        elseif tabId == "settings" then buildSettingsTab()
        else buildMainTab() end
        scroll.CanvasPosition = Vector2.zero
    end
    UI.rebuildTab = function() buildTab(UI.activeTab) end

    local function animateTabActivate(entry, doPop)
        local baseSize = entry.baseSize
        local activeIconSize = UDim2.new(0, baseSize, 0, baseSize)
        local popSize = UDim2.new(0, baseSize * 1.18, 0, baseSize * 1.18)
        entry.icon.Size = activeIconSize
        tween(entry.button, {BackgroundColor3=CONFIG.colors.navBg}, CONFIG.anim.fast)
        tween(entry.icon, {ImageColor3=CONFIG.colors.navIconActive}, CONFIG.anim.fast)
        tween(entry.stroke, {Color=CONFIG.colors.navActiveBrd, Transparency=0, Thickness=3.6}, 0.18)
        task.delay(0.18, function() tween(entry.stroke, {Thickness=2.4}, 0.22) end)
        tween(entry.iconGlow, {ImageColor3=CONFIG.colors.navIconActive, ImageTransparency=0.35}, CONFIG.anim.fast)
        if doPop then
            entry.icon.Size = UDim2.new(0, baseSize * 0.75, 0, baseSize * 0.75)
            entry.icon.Position = UDim2.new(0.5, -(baseSize * 0.75)/2, 0.5, -(baseSize * 0.75)/2)
            popTween(entry.icon, {
                Size = popSize, Position = UDim2.new(0.5, -(baseSize * 1.18)/2, 0.5, -(baseSize * 1.18)/2) }, 0.22)
            task.delay(0.22, function()
                tween(entry.icon, {Size = activeIconSize, Position = UDim2.new(0.5, -baseSize/2, 0.5, -baseSize/2)}, 0.16)
            end)
        end
    end
    local function animateTabDeactivate(entry)
        local baseSize = entry.baseSize
        tween(entry.button, {BackgroundColor3=CONFIG.colors.navBg}, CONFIG.anim.fast)
        tween(entry.stroke, {Color=CONFIG.colors.navBorder, Transparency=0, Thickness=2}, CONFIG.anim.fast)
        tween(entry.icon, {ImageColor3=CONFIG.colors.navIcon}, CONFIG.anim.fast)
        tween(entry.iconGlow, {ImageColor3=CONFIG.colors.navIcon, ImageTransparency=0.85}, CONFIG.anim.fast)
        entry.icon.Size = UDim2.new(0, baseSize, 0, baseSize)
        entry.icon.Position = UDim2.new(0.5, -baseSize/2, 0.5, -baseSize/2)
    end
    local function selectTab(tabId, doPop)
        UI.activeTab = tabId
        for id, entry in pairs(tabButtons) do
            if id == tabId then animateTabActivate(entry, doPop)
            else animateTabDeactivate(entry) end
        end
        buildTab(tabId)
    end
    for id, entry in pairs(tabButtons) do
        track(entry.hit.Activated:Connect(function()
            if UI.activeTab == id then return end
            selectTab(id, true)
        end))
    end

    local function minimizePanel()
        if not UI.isOpen or UI.isMinimized then return end
        UI.isMinimized = true
        local cxA, cyA = currentCenterPixels()
        local s = uiScale.Scale or 1
        local fullW, fullH = CONFIG.sizes.windowWidth * s, CONFIG.sizes.windowHeight * s
        local px, py = clampCenter(cxA, cyA, fullW, fullH)
        savePosition(UDim2.new(0, px, 0, py))
        UI.savedPos = UDim2.new(0, px, 0, py)
        safeWrite(bodyLayer, "Visible", false)
        safeWrite(navBar, "Visible", false)
        safeWrite(contentContainer, "Visible", false)
        if UI.infoOpen then infoFrame.Visible = false end
        local minH = CONFIG.sizes.headerHeight + 2
        local w, h = CONFIG.sizes.windowWidth * s, minH * s
        local cx, cy = clampCenter(cxA, cyA, w, h)
        TweenService:Create(scaleWrapper, TweenInfo.new(CONFIG.anim.slow, CONFIG.anim.easing, CONFIG.anim.dir), {
            Size = UDim2.new(0, CONFIG.sizes.windowWidth, 0, minH),
            Position = UDim2.new(0, cx, 0, cy),
        }):Play()
    end
    local function restorePanel()
        if not UI.isMinimized then return end
        UI.isMinimized = false
        safeWrite(bodyLayer, "Visible", true)
        if UI.infoOpen then
            infoFrame.Visible = true
            safeWrite(navBar, "Visible", false)
            safeWrite(contentContainer, "Visible", false)
        else
            safeWrite(navBar, "Visible", true)
            safeWrite(contentContainer, "Visible", true)
        end
        local s = uiScale.Scale or 1
        local w, h = CONFIG.sizes.windowWidth * s, CONFIG.sizes.windowHeight * s
        local cx, cy = currentCenterPixels()
        local tcx, tcy = clampCenter(cx, cy, w, h)
        local target = UDim2.new(0, tcx, 0, tcy)
        UI.savedPos = target
        savePosition(target)
        TweenService:Create(scaleWrapper, TweenInfo.new(CONFIG.anim.slow, CONFIG.anim.easing, CONFIG.anim.dir), {
            Size = UDim2.new(0, CONFIG.sizes.windowWidth, 0, CONFIG.sizes.windowHeight),
            Position = target,
        }):Play()
    end
    local function closePanel()
        if not UI.isOpen then return end
        UI.isOpen = false
        local cx, cy = currentCenterPixels()
        local s = uiScale.Scale or 1
        local w, h = CONFIG.sizes.windowWidth * s, CONFIG.sizes.windowHeight * s
        local px, py = clampCenter(cx, cy, w, h)
        savePosition(UDim2.new(0, px, 0, py))
        safeWrite(bodyLayer, "Visible", false)
        safeWrite(navBar, "Visible", false)
        safeWrite(contentContainer, "Visible", false)
        if UI.infoOpen then infoFrame.Visible = false end
        tween(scaleWrapper, { Size = UDim2.new(0, CONFIG.sizes.windowWidth, 0, 0) }, CONFIG.anim.normal)
    end
    local function openPanel(playEntrance)
        if UI.destroyed or UI.isOpen then return end
        UI.isOpen = true
        local vp = getViewport()
        local s = uiScale.Scale or 1
        local w, h = CONFIG.sizes.windowWidth * s, CONFIG.sizes.windowHeight * s
        local usePos = UI.savedPos or UDim2.new(0.5, 0, 0.5, 0)
        local cx = usePos.X.Scale * vp.X + usePos.X.Offset
        local cy = usePos.Y.Scale * vp.Y + usePos.Y.Offset
        local tcx, tcy = clampCenter(cx, cy, w, h)
        local targetPos = UDim2.new(0, tcx, 0, tcy)
        UI.savedPos = targetPos
        safeWrite(scaleWrapper, "Position", targetPos)
        UI.everOpened = true
        if playEntrance ~= false then
            safeWrite(scaleWrapper, "Size", UDim2.new(0, CONFIG.sizes.windowWidth, 0, 0))
            safeWrite(mainWindow, "BackgroundTransparency", 1)
            safeWrite(header, "BackgroundTransparency", 1)
            safeWrite(headerFlat, "BackgroundTransparency", 1)
            safeWrite(bodyLayer, "Visible", false)
            local finalSize = UDim2.new(0, CONFIG.sizes.windowWidth, 0, CONFIG.sizes.windowHeight)
            local tw = TweenService:Create(scaleWrapper, TweenInfo.new(0.55, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size=finalSize})
            tw:Play()
            tween(mainWindow, {BackgroundTransparency=0}, 0.40)
            tween(header, {BackgroundTransparency=0}, 0.40)
            tween(headerFlat, {BackgroundTransparency=0}, 0.40)
            task.delay(0.34, function()
                safeWrite(bodyLayer, "Visible", true)
                if UI.infoOpen then
                    infoFrame.Visible = true
                    safeWrite(navBar, "Visible", false)
                    safeWrite(contentContainer, "Visible", false)
                else
                    safeWrite(navBar, "Visible", true)
                    safeWrite(contentContainer, "Visible", true)
                end
            end)
        else
            safeWrite(scaleWrapper, "Size", UDim2.new(0, CONFIG.sizes.windowWidth, 0, CONFIG.sizes.windowHeight))
            safeWrite(mainWindow, "BackgroundTransparency", 0)
            safeWrite(header, "BackgroundTransparency", 0)
            safeWrite(headerFlat, "BackgroundTransparency", 0)
            safeWrite(bodyLayer, "Visible", true)
            safeWrite(navBar, "Visible", true)
            safeWrite(contentContainer, "Visible", true)
        end
    end

    minBtn.Activated:Connect(function()
        if UI.isMinimized then restorePanel() else minimizePanel() end
    end)
    closeBtn.Activated:Connect(function()
        closePanel()
        task.delay(CONFIG.anim.normal + 0.05, function()
            BypassTP:SetEnabled(false)
            BypassTP:SetAntiRagdoll(false)
            screenGui:Destroy()
            local ng = uiParent:FindFirstChild("BabisUIKitNotif"); if ng then ng:Destroy() end
            local ngPg = PlayerGui:FindFirstChild("BabisUIKitNotif"); if ngPg then ngPg:Destroy() end
        end)
    end)
    minBtn.MouseEnter:Connect(function() tween(minGlyph, {BackgroundColor3=CONFIG.colors.accent}, CONFIG.anim.fast) end)
    minBtn.MouseLeave:Connect(function() tween(minGlyph, {BackgroundColor3=CONFIG.colors.textPrimary}, CONFIG.anim.fast) end)
    closeBtn.MouseEnter:Connect(function()
        tween(xBar1, {BackgroundColor3=CONFIG.colors.closeHover}, CONFIG.anim.fast)
        tween(xBar2, {BackgroundColor3=CONFIG.colors.closeHover}, CONFIG.anim.fast)
    end)
    closeBtn.MouseLeave:Connect(function()
        tween(xBar1, {BackgroundColor3=CONFIG.colors.textPrimary}, CONFIG.anim.fast)
        tween(xBar2, {BackgroundColor3=CONFIG.colors.textPrimary}, CONFIG.anim.fast)
    end)

    local dragging = false
    local dragStartAbs = nil
    local dragStartCenter = nil
    local dragMoved = false
    local DRAG_THRESHOLD = 5
    local DRAG_SPURIOUS = 400

    header.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragMoved = false
            dragStartAbs = input.Position
            local cx, cy = currentCenterPixels()
            dragStartCenter = Vector2.new(cx, cy)
        end
    end)
    track(UserInputService.InputChanged:Connect(function(input)
        if not dragging then return end
        if input.UserInputType ~= Enum.UserInputType.MouseMovement
           and input.UserInputType ~= Enum.UserInputType.Touch then return end
        local delta = input.Position - dragStartAbs
        if delta.Magnitude > DRAG_SPURIOUS then
            dragStartAbs = input.Position
            local cx, cy = currentCenterPixels()
            dragStartCenter = Vector2.new(cx, cy)
            return
        end
        if not dragMoved then
            if delta.Magnitude < DRAG_THRESHOLD then return end
            dragMoved = true
        end
        local w, h = currentPixelSize()
        local tcx, tcy = clampCenter(
            dragStartCenter.X + delta.X,
            dragStartCenter.Y + delta.Y,
            w, h
        )
        local newPos = UDim2.new(0, tcx, 0, tcy)
        safeWrite(scaleWrapper, "Position", newPos)
        UI.savedPos = newPos
    end))
    track(UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType ~= Enum.UserInputType.MouseButton1
           and input.UserInputType ~= Enum.UserInputType.Touch then return end
        if dragging then
            dragging = false
            if dragMoved then savePosition(UI.savedPos) end
        end
    end))

    track(UserInputService.InputBegan:Connect(function(input, gpe)
        if UI.destroyed then return end
        if gpe then return end
        if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
        local savedKey = UI.options["toggle_key"]
        if not savedKey or savedKey == "None" or savedKey == "" then return end
        if input.KeyCode == Enum.KeyCode[savedKey] then
            if UI.isOpen then closePanel() else openPanel() end
        end
    end))

    local function updateScale()
        local vp = getViewport()
        local sx = (vp.X - 16) / CONFIG.sizes.windowWidth
        local sy = (vp.Y - 32) / CONFIG.sizes.windowHeight
        local s = math.min(1.25, math.min(sx, sy))
        s = math.max(s, 0.55)
        uiScale.Scale = s
        local w, h = currentPixelSize()
        local cx, cy = currentCenterPixels()
        local tcx, tcy = clampCenter(cx, cy, w, h)
        local newPos = UDim2.new(0, tcx, 0, tcy)
        safeWrite(scaleWrapper, "Position", newPos)
        UI.savedPos = newPos
    end
    updateScale()
    local cam = workspace.CurrentCamera
    if cam then track(cam:GetPropertyChangedSignal("ViewportSize"):Connect(updateScale)) end

    selectTab(CONFIG.defaultTab, false)
    openPanel(true)
end

-- ============================================================
-- INTRO
-- ============================================================
local function showIntro(onComplete)
    local introGui = Instance.new("ScreenGui")
    introGui.Name="BabisIntro"; introGui.ResetOnSpawn=false; introGui.IgnoreGuiInset=true
    introGui.ZIndexBehavior=Enum.ZIndexBehavior.Sibling; introGui.DisplayOrder=200
    introGui.Parent=uiParent
    protectInstance(introGui)
    local overlay = Instance.new("Frame")
    overlay.Size=UDim2.new(1,0,1,0); overlay.BackgroundColor3=Color3.fromRGB(0,0,0)
    overlay.BackgroundTransparency=1; overlay.BorderSizePixel=0; overlay.Parent=introGui
    tween(overlay, {BackgroundTransparency=0.55}, 0.5)
    task.delay(CONFIG.introDuration, function()
        tween(overlay, {BackgroundTransparency=1}, 0.55)
        task.wait(0.6)
        introGui:Destroy()
        if onComplete then onComplete() end
    end)
end

-- ============================================================
-- CONFIG SYSTEM
-- ============================================================
local ConfigSys = {}
do
    function ConfigSys.ensure()
        if not safeCall(isfolder, "Babis") then safeCall(makefolder, "Babis") end
        if not safeCall(isfolder, CONFIG.cfgFolder) then safeCall(makefolder, CONFIG.cfgFolder) end
    end
    function ConfigSys.list()
        ConfigSys.ensure(); local out = {}
        local files = safeCall(listfiles, CONFIG.cfgFolder)
        if type(files) ~= "table" then return out end
        for _, path in ipairs(files) do
            local name = path:match("([^/\\]+)$") or path
            name = name:gsub(CONFIG.cfgExt:gsub("%.", "%%."), "")
            table.insert(out, name)
        end
        return out
    end
    function ConfigSys.save(name, data)
        ConfigSys.ensure()
        if type(name) ~= "string" or name == "" then return false, "invalid name" end
        local path = CONFIG.cfgFolder .. "/" .. name .. CONFIG.cfgExt
        local okEnc, resEnc = pcall(function() return HttpService:JSONEncode(data) end)
        if not okEnc then return false, "encode failed" end
        local ok = safeCall(writefile, path, resEnc)
        if ok == nil then return false, "writefile failed" end
        return true
    end
    function ConfigSys.load(name)
        ConfigSys.ensure()
        local path = CONFIG.cfgFolder .. "/" .. name .. CONFIG.cfgExt
        local content = safeCall(readfile, path)
        if type(content) ~= "string" then return nil end
        local ok, data = pcall(function() return HttpService:JSONDecode(content) end)
        if not ok then return nil end
        return data
    end
    function ConfigSys.delete(name)
        local path = CONFIG.cfgFolder .. "/" .. name .. CONFIG.cfgExt
        return safeCall(delfile, path) ~= nil
    end
    function ConfigSys.getAutoload()
        local content = safeCall(readfile, CONFIG.autoloadFile)
        if type(content) == "string" and content ~= "" then return content end
        return nil
    end
    function ConfigSys.setAutoload(name)
        ConfigSys.ensure()
        if name == nil or name == "" then safeCall(delfile, CONFIG.autoloadFile)
        else safeCall(writefile, CONFIG.autoloadFile, name) end
    end
end

showIntro(buildMainUI)
