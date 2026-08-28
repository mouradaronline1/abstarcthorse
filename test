-- =========================================================================
-- ||                   ABSTRACT HORSE - ULTIMATE SCRIPT                  ||
-- =========================================================================

task.spawn(function()
    pcall(function()
        if openurl then
            openurl("https://t.me/abstracthorse")
        elseif syn and syn.open_url then
            syn.open_url("https://t.me/abstracthorse")
        elseif setclipboard then
            setclipboard("https://t.me/abstracthorse")
        end
    end)
end)

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")
local Debris = game:GetService("Debris")

-- ===================== SECURE RAYFIELD LOADER =====================
local Rayfield = nil
local rayfieldMirrors = {
    "https://sirius.menu/rayfield",
    "https://raw.githubusercontent.com/SiriusSoftwareLtd/Rayfield/main/source.lua",
    "https://raw.githubusercontent.com/shlexware/Rayfield/main/source"
}

for _, url in ipairs(rayfieldMirrors) do
    local success, result = pcall(function()
        return loadstring(game:HttpGet(url))()
    end)
    if success and result and typeof(result) == "table" and result.CreateWindow then
        Rayfield = result
        break
    end
end

if not Rayfield then
    warn("[Abstract Horse] Critical: Failed to load Rayfield UI library.")
    return
end

-- ===================== DRAWING API MOCK / FALLBACK =====================
local realDrawing = Drawing or (getgenv and getgenv().Drawing)
local hasDrawing = realDrawing ~= nil

local DrawingMock = {}
function DrawingMock.new(drawType)
    if hasDrawing then
        local ok, obj = pcall(function() return realDrawing.new(drawType) end)
        if ok and obj then return obj end
    end
    local dummy = {
        Visible = false,
        Transparency = 1,
        Color = Color3.new(1, 1, 1),
        Thickness = 1,
        Size = 16,
        Position = Vector2.zero,
        Text = "",
        From = Vector2.zero,
        To = Vector2.zero,
        Filled = false,
        Radius = 1,
        Center = false,
        Outline = false,
        OutlineColor = Color3.new(0, 0, 0),
        Font = 0
    }
    function dummy:Remove() end
    function dummy:Destroy() end
    return dummy
end

-- ===================== CONFIGURATION TABLES =====================
local TeamSettings = {
    ShowName = false,
    ShowDistance = false,
    Teams = {
        ["Defenders"] = { enabled = false, color = Color3.fromRGB(255, 50, 50), box2D = false, box3D = false, cornerBox = false, skeleton = false, snapline = false, healthBar = false, minHeight = 0 },
        ["Operators"] = { enabled = false, color = Color3.fromRGB(50, 150, 255), box2D = false, box3D = false, cornerBox = false, skeleton = false, snapline = false, healthBar = false, minHeight = 0 },
        ["Civilians"] = { enabled = false, color = Color3.fromRGB(50, 255, 50), box2D = false, box3D = false, cornerBox = false, skeleton = false, snapline = false, healthBar = false, minHeight = 0 }
    }
}

local ObjectSettings = { Enabled = false, MinHeight = 50, ShowName = false, ShowDistance = false, Box2D = false, Color = Color3.fromRGB(255, 170, 0) }

local OperatorsSettings = {
    AutoDetonate = false,
    DetonateKey = Enum.KeyCode.F,
    DetonateDangerDist = 220,
    LastDetonate = 0,
    SelectedMap = "Volsk",
    SearchingSpot = false,
    Maps = {
        ["Volsk"] = {
            Vector3.new(9612.16, 3.79, -19217.20),
            Vector3.new(9585.02, 3.81, -19191.89),
            Vector3.new(9592.38, 8.81, -19187.88),
            Vector3.new(9578.83, 3.92, -19196.04),
            Vector3.new(9605.74, 2.75, -19220.97)
        },
        ["Primorsk"] = {
            Vector3.new(-12410.7, 16.0, -1819.3),
            Vector3.new(-12418.2, 16.1, -1814.4),
            Vector3.new(-12410.9, 16.2, -1827.3),
            Vector3.new(-12427.2, 16.1, -1814.4),
            Vector3.new(-12412.4, 15.0, -1834.7)
        }
    }
}

local DefendersSettings = {
    -- Anti-Kamikaze Dodge
    AutoDodge = false,
    DodgeDistance = 70,
    DodgeThreshold = 65,
    DodgeCooldown = 1.5,
    LastDodge = 0,
    
    -- Auto-Update Drone List
    AutoUpdateDrones = true,
    DroneUpdateInterval = 2,
    
    -- Drone Hunter & Actions
    FollowDrone = true,
    FollowDistance = 15,
    FollowHeight = 2,
    FollowDuration = 4,
    RamDuration = 0.6,
    HeightOffset = 0,
    ReturnAfterTP = false,
    ReturnDelay = 1.5,
    PlaySound = true,
    
    -- Auto TP Loop
    AutoTP = false,
    TPDelay = 2,
    CurrentIndex = 1,
    ManualFollowRunning = false,
    ManualRamRunning = false,
    
    -- Target Hunter (SAM / Missiles)
    Pantsir = true,
    Tor = true,
    Strela = true,
    Stinger = true,
    Rockets = true,
    CustomFilter = "",
    MaxDistance = 15000,
    ESPEnabled = false,
    ESPColor = Color3.fromRGB(255, 80, 80),
    ESPBox = true,
    ESPName = true,
    ESPDistance = true
}

local WeaponSettings = {
    WallShot = false,
    AutoAimEnabled = false,
    AutoAimTarget = nil,
    TPAndAim = false,
    TPAndAimDelay = 2,
    ReturnAfterTP = false
}

local defaultLighting = {
    ClockTime = Lighting.ClockTime,
    Brightness = Lighting.Brightness,
    GlobalShadows = Lighting.GlobalShadows,
    Ambient = Lighting.Ambient,
    OutdoorAmbient = Lighting.OutdoorAmbient,
    FogEnd = Lighting.FogEnd,
    FogStart = Lighting.FogStart,
    FogColor = Lighting.FogColor,
    Gravity = workspace.Gravity
}

-- ===================== HELPER DRAWING BUILDERS =====================
local function createLine(color, thickness)
    local line = DrawingMock.new("Line")
    line.Color = color; line.Thickness = thickness; line.Transparency = 1; line.Visible = false
    return line
end

local function createSquare(color, thickness, filled)
    local square = DrawingMock.new("Square")
    square.Color = color; square.Thickness = thickness; square.Filled = filled or false; square.Transparency = 1; square.Visible = false
    return square
end

local function createText(color, size, center, outline)
    local text = DrawingMock.new("Text")
    text.Color = color; text.Size = size or 16; text.Center = center or true; text.Outline = outline or false
    text.OutlineColor = Color3.fromRGB(0, 0, 0); text.Font = 2; text.Transparency = 1; text.Visible = false
    return text
end

-- ===================== ROBUST ESP ENGINE (NO GHOSTS) =====================
local espPlayersTable = {}
local espObjectsDrawing = {}
local espHunterDrawings = {}

local function createPlayerESP(player)
    if player == LocalPlayer then return end
    local team = player.Team and player.Team.Name or "Unknown"
    local color = (TeamSettings.Teams[team] and TeamSettings.Teams[team].color) or Color3.fromRGB(255, 50, 50)

    local objects = {
        nameText = createText(color, 15, true, true),
        distanceText = createText(color, 13, true, true),
        box2D = createSquare(color, 1, false),
        snapline = createLine(color, 1),
        healthBg = createSquare(Color3.fromRGB(0, 0, 0), 1, true),
        healthFill = createSquare(color, 1, true),
        cornerLines = {}, box3dLines = {}, skeletonLines = {}
    }
    for i = 1, 8 do objects.cornerLines[i] = createLine(color, 2) end
    for i = 1, 12 do objects.box3dLines[i] = createLine(color, 1) end
    for i = 1, 15 do objects.skeletonLines[i] = createLine(color, 1) end

    espPlayersTable[player] = { objects = objects }
end

local function hidePlayerESP(data)
    if not data or not data.objects then return end
    for _, obj in pairs(data.objects) do
        if typeof(obj) == "table" then
            for _, sub in pairs(obj) do pcall(function() sub.Visible = false end) end
        else
            pcall(function() obj.Visible = false end)
        end
    end
end

local function removePlayerESP(player)
    local data = espPlayersTable[player]
    if data and data.objects then
        for _, obj in pairs(data.objects) do
            if typeof(obj) == "table" then
                for _, sub in pairs(obj) do
                    pcall(function() sub.Visible = false; sub:Remove() end)
                end
            else
                pcall(function() obj.Visible = false; obj:Remove() end)
            end
        end
    end
    espPlayersTable[player] = nil
end

local function hideAllESPObjects()
    for _, data in pairs(espPlayersTable) do hidePlayerESP(data) end
    for _, drawings in pairs(espObjectsDrawing) do
        for _, d in pairs(drawings) do pcall(function() d.Visible = false end) end
    end
    for _, drawings in pairs(espHunterDrawings) do
        for _, d in pairs(drawings) do pcall(function() d.Visible = false end) end
    end
end

local function setupPlayerConnection(player)
    if player == LocalPlayer then return end
    removePlayerESP(player)
    createPlayerESP(player)
    
    player.CharacterAdded:Connect(function()
        removePlayerESP(player)
        task.wait(0.2)
        createPlayerESP(player)
    end)
    
    player.CharacterRemoving:Connect(function()
        hidePlayerESP(espPlayersTable[player])
    end)
end

for _, player in ipairs(Players:GetPlayers()) do setupPlayerConnection(player) end
Players.PlayerAdded:Connect(setupPlayerConnection)
Players.PlayerRemoving:Connect(removePlayerESP)

-- Clear ESP completely when Local Player Dies or Character is removed
local function hookLocalCharacter(char)
    hideAllESPObjects()
    local humanoid = char:WaitForChild("Humanoid", 5)
    if humanoid then
        humanoid.Died:Connect(function()
            hideAllESPObjects()
        end)
    end
end

if LocalPlayer.Character then hookLocalCharacter(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(hookLocalCharacter)
LocalPlayer.CharacterRemoving:Connect(hideAllESPObjects)

-- ===================== SCANNING UTILITIES =====================
local function scanAirborneObjects()
    local objects = {}
    local seen = {}
    local char = LocalPlayer.Character
    local myPos = (char and char:FindFirstChild("HumanoidRootPart")) and char.HumanoidRootPart.Position.Y or 0
    local filterWords = {"prop", "effect", "window", "windows", "desk", "staircase", "model", "wings", "terrain", "tree"}

    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and not obj:FindFirstChildOfClass("Humanoid") then
            local lowerName = string.lower(obj.Name)
            local isJunk = false
            for _, word in ipairs(filterWords) do
                if string.find(lowerName, word) then isJunk = true; break end
            end
            if isJunk then continue end

            local cframe, size = obj:GetBoundingBox()
            if size.Magnitude > 1.5 and size.Magnitude < 300 then
                if cframe.Position.Y >= (myPos + ObjectSettings.MinHeight) then
                    if not seen[obj] then
                        seen[obj] = true
                        table.insert(objects, obj)
                    end
                end
            end
        end
    end
    return objects
end

local function matchesHunterFilter(name)
    local lower = string.lower(name)
    if DefendersSettings.Pantsir and lower:find("pantsir") then return true end
    if DefendersSettings.Tor and (lower:find("torm1") or lower:find("tor")) then return true end
    if DefendersSettings.Strela and (lower:find("strela10") or lower:find("strela")) then return true end
    if DefendersSettings.Stinger and (lower:find("stinger") or lower:find("stringer")) then return true end
    if DefendersSettings.Rockets and (lower:find("rocket") or lower:find("missile") or lower:find("projectile")) then return true end
    if DefendersSettings.CustomFilter ~= "" and lower:find(string.lower(DefendersSettings.CustomFilter)) then return true end
    return false
end

local function scanHunterTargets()
    local list = {}
    local seen = {}
    local char = LocalPlayer.Character
    local myPos = (char and char:FindFirstChild("HumanoidRootPart")) and char.HumanoidRootPart.Position or Vector3.zero

    for _, obj in ipairs(workspace:GetDescendants()) do
        if (obj:IsA("Model") and not obj:FindFirstChildOfClass("Humanoid")) or (obj:IsA("BasePart") and not (obj.Parent and obj.Parent:FindFirstChildOfClass("Humanoid"))) then
            if matchesHunterFilter(obj.Name) then
                local item = obj
                if not seen[item] then
                    seen[item] = true
                    local cf, size
                    if item:IsA("Model") then cf, size = item:GetBoundingBox() else cf, size = item.CFrame, item.Size end
                    local dist = (cf.Position - myPos).Magnitude
                    if dist <= DefendersSettings.MaxDistance then
                        table.insert(list, { object = item, pos = cf.Position, name = item.Name, dist = dist })
                    end
                end
            end
        end
    end
    return list
end

-- ===================== SOUND & TELEPORT HELPERS =====================
local function playBeep()
    pcall(function()
        local sound = Instance.new("Sound")
        sound.SoundId = "rbxassetid://9120263684"
        sound.Volume = 1
        sound.Parent = workspace
        sound:Play()
        Debris:AddItem(sound, 1)
    end)
end

local function teleportTo(position)
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = CFrame.new(position)
    end
end

-- ===================== WEAPON HELPERS =====================
local function findAK74()
    local char = LocalPlayer.Character
    if not char then return nil end
    for _, tool in ipairs(char:GetChildren()) do
        if tool:IsA("Tool") and (string.lower(tool.Name):find("ak74") or string.lower(tool.Name):find("ak47") or string.lower(tool.Name):find("ak")) then
            return tool
        end
    end
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    if backpack then
        for _, tool in ipairs(backpack:GetChildren()) do
            if tool:IsA("Tool") and (string.lower(tool.Name):find("ak74") or string.lower(tool.Name):find("ak47") or string.lower(tool.Name):find("ak")) then
                return tool
            end
        end
    end
    return nil
end

local function equipAK74()
    local ak = findAK74()
    if not ak then return nil end
    if ak.Parent ~= LocalPlayer.Character then
        local char = LocalPlayer.Character
        if char then
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid:EquipTool(ak)
                task.wait(0.1)
            end
        end
    end
    return ak
end

-- ===================== OPERATOR DETONATION (KEY F) =====================
local function triggerDetonation()
    if tick() - OperatorsSettings.LastDetonate < 2 then return end
    OperatorsSettings.LastDetonate = tick()

    pcall(function()
        VirtualInputManager:SendKeyEvent(true, OperatorsSettings.DetonateKey, false, game)
        task.wait(0.03)
        VirtualInputManager:SendKeyEvent(false, OperatorsSettings.DetonateKey, false, game)
    end)

    pcall(function()
        for _, rem in ipairs(ReplicatedStorage:GetDescendants()) do
            if rem:IsA("RemoteEvent") and (string.lower(rem.Name):find("explode") or string.lower(rem.Name):find("detonat") or string.lower(rem.Name):find("selfdestruct")) then
                rem:FireServer()
            end
        end
    end)

    Rayfield:Notify({
        Title = "Operator Alert",
        Content = "Missile incoming! Drone detonated (XP Denied)!",
        Duration = 2.5
    })
end

task.spawn(function()
    while true do
        if OperatorsSettings.AutoDetonate then
            local droneOrCamPos = Camera.CFrame.Position
            local dangerDist = OperatorsSettings.DetonateDangerDist

            for _, obj in ipairs(workspace:GetDescendants()) do
                if (obj:IsA("Model") or obj:IsA("BasePart")) and matchesHunterFilter(obj.Name) then
                    local cf = obj:IsA("Model") and obj:GetBoundingBox() or obj.CFrame
                    if cf then
                        local dist = (cf.Position - droneOrCamPos).Magnitude
                        if dist <= dangerDist then
                            local missileVel = (obj:IsA("BasePart") and obj.AssemblyLinearVelocity) or Vector3.zero
                            local toMe = (droneOrCamPos - cf.Position).Unit
                            local isClosingIn = missileVel.Magnitude > 10 and missileVel.Unit:Dot(toMe) > 0.2

                            if isClosingIn or dist <= (dangerDist * 0.5) then
                                triggerDetonation()
                                break
                            end
                        end
                    end
                end
            end
        end
        task.wait(0.05)
    end
end)

-- ===================== DEFENDER COMBAT MECHANICS =====================
-- 1. Anti-Kamikaze Auto-Dodge
task.spawn(function()
    while true do
        if DefendersSettings.AutoDodge then
            if (tick() - DefendersSettings.LastDodge) >= DefendersSettings.DodgeCooldown then
                local char = LocalPlayer.Character
                local root = char and char:FindFirstChild("HumanoidRootPart")
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                
                if root and hum and hum.Health > 0 then
                    local myPos = root.Position
                    local drones = scanAirborneObjects()

                    for _, drone in ipairs(drones) do
                        if drone and drone.Parent then
                            local cf = drone:GetBoundingBox()
                            local dist = (cf.Position - myPos).Magnitude

                            if dist <= DefendersSettings.DodgeThreshold then
                                local isDiving = cf.Position.Y > myPos.Y + 3
                                local lookAtPlayer = (myPos - cf.Position).Unit
                                local headingTowards = cf.LookVector:Dot(lookAtPlayer) > 0.35

                                if isDiving and headingTowards then
                                    local dodgeVector = (-root.CFrame.LookVector * DefendersSettings.DodgeDistance) + (root.CFrame.RightVector * (math.random() > 0.5 and 25 or -25))
                                    local newPos = root.Position + dodgeVector

                                    local rayResult = workspace:Raycast(newPos + Vector3.new(0, 10, 0), Vector3.new(0, -30, 0))
                                    if rayResult then newPos = rayResult.Position + Vector3.new(0, 3, 0) end

                                    root.CFrame = CFrame.new(newPos)
                                    DefendersSettings.LastDodge = tick()
                                    playBeep()

                                    Rayfield:Notify({
                                        Title = "Defender Evasion",
                                        Content = "Diving drone dodged!",
                                        Duration = 2
                                    })
                                    break
                                end
                            end
                        end
                    end
                end
            end
        end
        task.wait(0.04)
    end
end)

-- 2. Follow & Aim Drone
local function followAndAimDrone(drone)
    if not drone or not drone.Parent then return end
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local root = char.HumanoidRootPart

    local originalCFrame = root.CFrame
    local startTime = tick()
    local duration = DefendersSettings.FollowDuration or 4

    while (tick() - startTime) < duration do
        if not DefendersSettings.AutoTP and not DefendersSettings.ManualFollowRunning then break end
        if not drone or not drone.Parent or not drone:IsDescendantOf(workspace) then break end
        if not char or not char.Parent or not root.Parent then break end

        local droneCF = drone:IsA("Model") and drone:GetBoundingBox() or drone.CFrame
        local dronePos = droneCF.Position
        local lookVec = droneCF.LookVector
        if lookVec.Magnitude == 0 then lookVec = Vector3.new(0, 0, -1) end

        local targetFollowPos = dronePos - (lookVec * DefendersSettings.FollowDistance) + Vector3.new(0, DefendersSettings.FollowHeight, 0)

        root.CFrame = CFrame.lookAt(targetFollowPos, dronePos)
        root.AssemblyLinearVelocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero

        Camera.CFrame = CFrame.lookAt(Camera.CFrame.Position, dronePos)
        task.wait(0.05)
    end

    if DefendersSettings.ReturnAfterTP and char and char:FindFirstChild("HumanoidRootPart") then
        task.wait(DefendersSettings.ReturnDelay or 1)
        root.CFrame = originalCFrame
    end
end

-- 3. Body-Crash / Ram Drone (Instant collision kill)
local function ramAndDestroyDrone(drone)
    if not drone or not drone.Parent then return end
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local root = char.HumanoidRootPart

    local originalCFrame = root.CFrame
    local cf = drone:IsA("Model") and drone:GetBoundingBox() or drone.CFrame

    if DefendersSettings.PlaySound then playBeep() end
    root.CFrame = cf + Vector3.new(0, DefendersSettings.HeightOffset, 0)
    root.AssemblyLinearVelocity = Vector3.new(0, -10, 0)

    task.wait(DefendersSettings.RamDuration or 0.5)

    if DefendersSettings.ReturnAfterTP and char and char:FindFirstChild("HumanoidRootPart") then
        task.wait(DefendersSettings.ReturnDelay or 1)
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = originalCFrame
        end
    end
end

-- ===================== FUN: MORPHS & SKYBOX =====================
local function unmorphPlayer()
    local char = LocalPlayer.Character
    if not char then return end
    local morph = char:FindFirstChild("CustomMorph_AH")
    if morph then morph:Destroy() end
    for _, part in ipairs(char:GetDescendants()) do
        local tag = part:FindFirstChild("OriginalTransparency_AH")
        if tag then part.Transparency = tag.Value; tag:Destroy() end
    end
end

local function morphPlayer(assetId)
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    unmorphPlayer() 
    local morphSuccess, morphResult = pcall(function()
        return game:GetObjects("rbxassetid://" .. tostring(assetId):gsub("%D", ""))[1]
    end)
    if not morphSuccess or not morphResult then
        Rayfield:Notify({ Title = "Morph Error", Content = "Failed to load Asset ID: " .. tostring(assetId), Duration = 4 })
        return
    end
    for _, part in ipairs(char:GetDescendants()) do
        if (part:IsA("BasePart") and part.Name ~= "HumanoidRootPart") or part:IsA("Decal") or part:IsA("Texture") then
            if not part:FindFirstChild("OriginalTransparency_AH") then
                local tag = Instance.new("NumberValue")
                tag.Name = "OriginalTransparency_AH"
                tag.Value = part.Transparency
                tag.Parent = part
            end
            part.Transparency = 1
        end
    end
    morphResult.Name = "CustomMorph_AH"
    morphResult.Parent = char
    local morphRoot = morphResult.PrimaryPart or morphResult:FindFirstChild("HumanoidRootPart") or morphResult:FindFirstChildWhichIsA("BasePart")
    if morphRoot then
        morphRoot.CFrame = char.HumanoidRootPart.CFrame
        local weld = Instance.new("WeldConstraint")
        weld.Part0 = char.HumanoidRootPart
        weld.Part1 = morphRoot
        weld.Parent = morphResult
        for _, part in ipairs(morphResult:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Anchored = false
                part.CanCollide = false
                part.Massless = true
            end
        end
    end
    Rayfield:Notify({ Title = "Morph Applied", Content = "Custom model applied successfully!", Duration = 3 })
end

local function applyCustomSky(assetId)
    if not assetId or assetId == "" then return end
    local formattedId = "rbxassetid://" .. tostring(assetId):gsub("%D", "")
    
    local customSkyInstance = Lighting:FindFirstChild("CustomSky_AH")
    if not customSkyInstance then
        customSkyInstance = Instance.new("Sky")
        customSkyInstance.Name = "CustomSky_AH"
        customSkyInstance.Parent = Lighting
    end
    
    customSkyInstance.SkyboxBk = formattedId
    customSkyInstance.SkyboxDn = formattedId
    customSkyInstance.SkyboxFt = formattedId
    customSkyInstance.SkyboxLf = formattedId
    customSkyInstance.SkyboxRt = formattedId
    customSkyInstance.SkyboxUp = formattedId
    
    Rayfield:Notify({Title = "Skybox Changed", Content = "Applied Skybox ID: " .. assetId, Duration = 3})
end

local function resetCustomSky()
    local sky = Lighting:FindFirstChild("CustomSky_AH")
    if sky then sky:Destroy() end
    Rayfield:Notify({Title = "Skybox Reset", Content = "Skybox returned to default.", Duration = 3})
end

-- ===================== OPERATOR STATION CAPTURE =====================
local function isPlayerInRadius(pos, radius)
    radius = radius or 5
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= LocalPlayer then
            local character = otherPlayer.Character
            if character then
                local rootPart = character:FindFirstChild("HumanoidRootPart")
                if rootPart and (rootPart.Position - pos).Magnitude <= radius then
                    return true
                end
            end
        end
    end
    return false
end

local function startDroneSpotSearch()
    if OperatorsSettings.SearchingSpot then return end
    OperatorsSettings.SearchingSpot = true
    Rayfield:Notify({Title = "Station Finder", Content = "Searching for free drone seat: " .. OperatorsSettings.SelectedMap, Duration = 2})
    
    task.spawn(function()
        local locations = OperatorsSettings.Maps[OperatorsSettings.SelectedMap] or OperatorsSettings.Maps["Volsk"]
        local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        character:WaitForChild("HumanoidRootPart")
        
        while OperatorsSettings.SearchingSpot do
            for _, pos in ipairs(locations) do
                if not OperatorsSettings.SearchingSpot then break end
                if not isPlayerInRadius(pos, 5) then
                    character.HumanoidRootPart.CFrame = CFrame.new(pos)
                    playBeep()
                    OperatorsSettings.SearchingSpot = false
                    Rayfield:Notify({Title = "Success!", Content = "Free operator station secured!", Duration = 3})
                    return
                end
            end
            task.wait(1)
        end
    end)
end

local function stopDroneSpotSearch()
    if OperatorsSettings.SearchingSpot then
        OperatorsSettings.SearchingSpot = false
        Rayfield:Notify({Title = "Station Finder", Content = "Auto-search stopped.", Duration = 2})
    end
end

-- ===================== RENDER STEPPED (ESP) =====================
RunService.RenderStepped:Connect(function()
    local localChar = LocalPlayer.Character
    local localHum = localChar and localChar:FindFirstChildOfClass("Humanoid")
    local localRoot = localChar and localChar:FindFirstChild("HumanoidRootPart")

    -- Check if local player is alive and valid
    if not localChar or not localHum or localHum.Health <= 0 or not localRoot or not hasDrawing then
        hideAllESPObjects()
        return
    end

    -- 1. Player ESP Rendering
    for player, data in pairs(espPlayersTable) do
        if not player or not player.Parent then
            removePlayerESP(player)
            continue
        end

        local team = player.Team and player.Team.Name or "Unknown"
        local teamCfg = TeamSettings.Teams[team]
        local char = player.Character
        local rootPart = char and char:FindFirstChild("HumanoidRootPart")
        local head = char and char:FindFirstChild("Head")
        local humanoid = char and char:FindFirstChildOfClass("Humanoid")

        -- Strict Death / Despawn Check to eliminate ghosts
        local isAlive = char and char.Parent ~= nil and char:IsDescendantOf(workspace) and humanoid and humanoid.Health > 0 and rootPart and head
        if not isAlive or not teamCfg or not teamCfg.enabled then
            hidePlayerESP(data)
            continue
        end

        if teamCfg.minHeight > 0 and rootPart.Position.Y < teamCfg.minHeight then
            hidePlayerESP(data)
            continue
        end

        local headPos, headOnScreen = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
        local rootPos, rootOnScreen = Camera:WorldToViewportPoint(rootPart.Position - Vector3.new(0, 3, 0))

        if not headOnScreen or not rootOnScreen or headPos.Z <= 0 or rootPos.Z <= 0 then
            hidePlayerESP(data)
            continue
        end

        local distance = (localRoot.Position - rootPart.Position).Magnitude

        data.objects.nameText.Text = player.Name; data.objects.nameText.Position = Vector2.new(headPos.X, headPos.Y - 20)
        data.objects.nameText.Color = teamCfg.color; data.objects.nameText.Visible = TeamSettings.ShowName

        data.objects.distanceText.Text = string.format("[%d m]", math.floor(distance)); data.objects.distanceText.Position = Vector2.new(rootPos.X, rootPos.Y + 5)
        data.objects.distanceText.Color = teamCfg.color; data.objects.distanceText.Visible = TeamSettings.ShowDistance

        local height = math.abs(rootPos.Y - headPos.Y); local width = height * 0.55
        local x = headPos.X - width / 2; local y = headPos.Y

        data.objects.box2D.Position = Vector2.new(x, y); data.objects.box2D.Size = Vector2.new(width, height)
        data.objects.box2D.Color = teamCfg.color; data.objects.box2D.Visible = teamCfg.box2D

        if teamCfg.snapline then
            data.objects.snapline.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
            data.objects.snapline.To = Vector2.new(rootPos.X, rootPos.Y)
            data.objects.snapline.Color = teamCfg.color; data.objects.snapline.Visible = true
        else
            data.objects.snapline.Visible = false
        end

        if teamCfg.healthBar then
            local hp = math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1)
            data.objects.healthBg.Position = Vector2.new(x - 6, y); data.objects.healthBg.Size = Vector2.new(4, height); data.objects.healthBg.Visible = true
            local fillH = height * hp
            data.objects.healthFill.Position = Vector2.new(x - 5, y + (height - fillH)); data.objects.healthFill.Size = Vector2.new(2, fillH)
            data.objects.healthFill.Color = Color3.fromRGB(255 - (hp * 255), hp * 255, 0); data.objects.healthFill.Visible = true
        else
            data.objects.healthBg.Visible = false; data.objects.healthFill.Visible = false
        end
    end

    -- 2. SAM / Missiles Target ESP Rendering
    if DefendersSettings.ESPEnabled then
        for obj, drawings in pairs(espHunterDrawings) do
            if obj and obj.Parent and obj:IsDescendantOf(workspace) then
                local cf = obj:IsA("Model") and obj:GetBoundingBox() or obj.CFrame
                local size = obj:IsA("Model") and select(2, obj:GetBoundingBox()) or obj.Size or Vector3.new(2,2,2)
                local screenPos, onScreen = Camera:WorldToViewportPoint(cf.Position)
                
                if onScreen and screenPos.Z > 0 then
                    local dist = (localRoot.Position - cf.Position).Magnitude
                    drawings.nameText.Text = "[🎯] " .. obj.Name; drawings.nameText.Position = Vector2.new(screenPos.X, screenPos.Y - 15)
                    drawings.nameText.Visible = DefendersSettings.ESPName; drawings.nameText.Color = DefendersSettings.ESPColor
                    drawings.distanceText.Text = string.format("[%d m]", math.floor(dist)); drawings.distanceText.Position = Vector2.new(screenPos.X, screenPos.Y + 5)
                    drawings.distanceText.Visible = DefendersSettings.ESPDistance; drawings.distanceText.Color = DefendersSettings.ESPColor

                    if DefendersSettings.ESPBox then
                        local corners = { (cf * CFrame.new(size.X/2, size.Y/2, size.Z/2)).Position, (cf * CFrame.new(-size.X/2, -size.Y/2, -size.Z/2)).Position }
                        local p1 = Camera:WorldToViewportPoint(corners[1])
                        local p2 = Camera:WorldToViewportPoint(corners[2])
                        drawings.box2D.Position = Vector2.new(math.min(p1.X, p2.X), math.min(p1.Y, p2.Y))
                        drawings.box2D.Size = Vector2.new(math.abs(p1.X - p2.X), math.abs(p1.Y - p2.Y))
                        drawings.box2D.Color = DefendersSettings.ESPColor; drawings.box2D.Visible = true
                    else
                        drawings.box2D.Visible = false
                    end
                else
                    drawings.nameText.Visible = false; drawings.distanceText.Visible = false; drawings.box2D.Visible = false
                end
            else
                drawings.nameText.Visible = false; drawings.distanceText.Visible = false; drawings.box2D.Visible = false
            end
        end
    end
end)

local function updateHunterESP()
    for obj, drawings in pairs(espHunterDrawings) do
        for _, d in pairs(drawings) do pcall(function() d:Remove() end) end
    end
    espHunterDrawings = {}

    if not hasDrawing or not DefendersSettings.ESPEnabled then return end

    local list = scanHunterTargets()
    for _, item in ipairs(list) do
        local obj = item.object
        if obj and obj.Parent and obj:IsDescendantOf(workspace) then
            local color = DefendersSettings.ESPColor
            espHunterDrawings[obj] = {
                nameText = createText(color, 15, true, true),
                distanceText = createText(color, 13, true, true),
                box2D = createSquare(color, 1, false)
            }
        end
    end
end

-- ===================== GUI WINDOW CREATION =====================
local Window = Rayfield:CreateWindow({
    Name = "Abstract Horse | Ultimate Suite",
    LoadingTitle = "ABSTRACT HORSE",
    LoadingSubtitle = "Defenders & Operators Edition",
    Theme = "Default",
    ConfigurationSaving = { Enabled = false },
    KeySystem = false
})

-- ===================== TAB 1: OPERATORS =====================
local TabOperators = Window:CreateTab("Operators", 4483362458)

TabOperators:CreateSection("Drone Defense & Anti-XP Denial")
TabOperators:CreateToggle({
    Name = "Auto-Detonate on Threat (Key F)",
    CurrentValue = false,
    Callback = function(v)
        OperatorsSettings.AutoDetonate = v
        Rayfield:Notify({
            Title = "Auto Detonate",
            Content = v and "Active: Will detonate drone before SAM missile hits!" or "Deactivated",
            Duration = 2
        })
    end
})
TabOperators:CreateSlider({
    Name = "Danger Trigger Distance",
    Range = {50, 500},
    Increment = 10,
    CurrentValue = OperatorsSettings.DetonateDangerDist,
    Callback = function(v) OperatorsSettings.DetonateDangerDist = v end
})
TabOperators:CreateButton({
    Name = "Manual Emergency Detonate (F)",
    Callback = function() triggerDetonation() end
})

TabOperators:CreateSection("Operator Station Auto-Takeover")
TabOperators:CreateDropdown({
    Name = "Select Map",
    Options = {"Volsk", "Primorsk"},
    CurrentOption = {"Volsk"},
    MultipleOptions = false,
    Callback = function(Option) OperatorsSettings.SelectedMap = Option[1] end
})
TabOperators:CreateButton({
    Name = "Find & TP to Free Station",
    Callback = function() startDroneSpotSearch() end
})
TabOperators:CreateButton({
    Name = "Stop Station Search",
    Callback = function() stopDroneSpotSearch() end
})

-- ===================== TAB 2: DEFENDERS =====================
local TabDefenders = Window:CreateTab("Defenders", 4483362458)

TabDefenders:CreateSection("Anti-Kamikaze / Dive Drone Evasion")
TabDefenders:CreateToggle({
    Name = "Auto-Dodge Diving Drones",
    CurrentValue = false,
    Callback = function(v)
        DefendersSettings.AutoDodge = v
        Rayfield:Notify({
            Title = "Defender Evasion",
            Content = v and "Active: Will auto-teleport away when a drone dives on you!" or "Deactivated",
            Duration = 2
        })
    end
})
TabDefenders:CreateSlider({
    Name = "Dodge Trigger Range",
    Range = {30, 150},
    Increment = 5,
    CurrentValue = DefendersSettings.DodgeThreshold,
    Callback = function(v) DefendersSettings.DodgeThreshold = v end
})
TabDefenders:CreateSlider({
    Name = "Dodge Distance",
    Range = {30, 200},
    Increment = 5,
    CurrentValue = DefendersSettings.DodgeDistance,
    Callback = function(v) DefendersSettings.DodgeDistance = v end
})

TabDefenders:CreateSection("Airborne Drone Hunting & Destroy")
local defenderDroneDropdown
local defenderDroneList = {}

local function refreshDefenderDrones()
    defenderDroneList = scanAirborneObjects()
    local opts = {}
    for i, obj in ipairs(defenderDroneList) do table.insert(opts, obj.Name .. " [#" .. i .. "]") end
    if #opts == 0 then table.insert(opts, "No Drones Detected") end
    if defenderDroneDropdown then defenderDroneDropdown:Refresh(opts, true) end
end

defenderDroneDropdown = TabDefenders:CreateDropdown({
    Name = "Select Target Drone",
    Options = {"No Drones Detected"},
    CurrentOption = {"No Drones Detected"},
    MultipleOptions = false,
    Callback = function() end
})

TabDefenders:CreateToggle({
    Name = "Auto-Update Drone List",
    CurrentValue = true,
    Callback = function(v) DefendersSettings.AutoUpdateDrones = v end
})
TabDefenders:CreateSlider({
    Name = "Drone List Update Interval (s)",
    Range = {1, 10},
    Increment = 0.5,
    CurrentValue = 2,
    Callback = function(v) DefendersSettings.DroneUpdateInterval = v end
})
TabDefenders:CreateButton({ Name = "Manually Refresh Drone List", Callback = function() refreshDefenderDrones() end })

TabDefenders:CreateButton({
    Name = "💥 Body-Crash / Ram Drone (Touch to Destroy)",
    Callback = function()
        local sel = defenderDroneDropdown.CurrentOption[1]
        if sel and sel ~= "No Drones Detected" then
            local idx = tonumber(sel:match("%[#(%d+)%]"))
            if idx and defenderDroneList[idx] then
                DefendersSettings.ManualRamRunning = true
                task.spawn(function()
                    ramAndDestroyDrone(defenderDroneList[idx])
                    DefendersSettings.ManualRamRunning = false
                end)
            end
        end
    end
})

TabDefenders:CreateButton({
    Name = "✈️ TP Behind Drone & Aim (Follow)",
    Callback = function()
        local sel = defenderDroneDropdown.CurrentOption[1]
        if sel and sel ~= "No Drones Detected" then
            local idx = tonumber(sel:match("%[#(%d+)%]"))
            if idx and defenderDroneList[idx] then
                DefendersSettings.ManualFollowRunning = true
                task.spawn(function()
                    followAndAimDrone(defenderDroneList[idx])
                    DefendersSettings.ManualFollowRunning = false
                end)
            end
        end
    end
})

TabDefenders:CreateButton({
    Name = "Equip AK-74",
    Callback = function() equipAK74() end
})

TabDefenders:CreateSection("Teleport & Return Settings")
TabDefenders:CreateToggle({
    Name = "Return Back After TP",
    CurrentValue = DefendersSettings.ReturnAfterTP,
    Callback = function(v) DefendersSettings.ReturnAfterTP = v end
})
TabDefenders:CreateSlider({
    Name = "Return Delay (Seconds)",
    Range = {0.5, 8},
    Increment = 0.5,
    CurrentValue = DefendersSettings.ReturnDelay,
    Callback = function(v) DefendersSettings.ReturnDelay = v end
})
TabDefenders:CreateSlider({
    Name = "Ram / Touch Duration (Seconds)",
    Range = {0.2, 3},
    Increment = 0.1,
    CurrentValue = DefendersSettings.RamDuration,
    Callback = function(v) DefendersSettings.RamDuration = v end
})
TabDefenders:CreateSlider({
    Name = "Follow Distance Behind (m)",
    Range = {3, 40},
    Increment = 1,
    CurrentValue = DefendersSettings.FollowDistance,
    Callback = function(v) DefendersSettings.FollowDistance = v end
})
TabDefenders:CreateSlider({
    Name = "Follow Height Offset (m)",
    Range = {-5, 15},
    Increment = 0.5,
    CurrentValue = DefendersSettings.FollowHeight,
    Callback = function(v) DefendersSettings.FollowHeight = v end
})
TabDefenders:CreateToggle({
    Name = "Play Beep Sound on TP",
    CurrentValue = DefendersSettings.PlaySound,
    Callback = function(v) DefendersSettings.PlaySound = v end
})

TabDefenders:CreateSection("Auto-Hunt Loop (All Drones)")
TabDefenders:CreateToggle({
    Name = "Continuous Auto TP Loop",
    CurrentValue = false,
    Callback = function(v) DefendersSettings.AutoTP = v; DefendersSettings.CurrentIndex = 1 end
})
TabDefenders:CreateSlider({
    Name = "Delay Between Targets (sec)",
    Range = {0.5, 8},
    Increment = 0.5,
    CurrentValue = 2,
    Callback = function(v) DefendersSettings.TPDelay = v end
})

task.spawn(function()
    while true do
        if DefendersSettings.AutoUpdateDrones then refreshDefenderDrones() end
        task.wait(DefendersSettings.DroneUpdateInterval or 2)
    end
end)

task.spawn(function()
    while true do
        if DefendersSettings.AutoTP then
            local list = defenderDroneList
            if #list > 0 then
                if DefendersSettings.CurrentIndex > #list then DefendersSettings.CurrentIndex = 1 end
                local target = list[DefendersSettings.CurrentIndex]
                if target and target.Parent and target:IsDescendantOf(workspace) then
                    followAndAimDrone(target)
                end
                DefendersSettings.CurrentIndex = DefendersSettings.CurrentIndex + 1
                task.wait(DefendersSettings.TPDelay)
            else
                refreshDefenderDrones()
                task.wait(1)
            end
        else
            task.wait(0.5)
        end
    end
end)

-- ===================== TAB 3: VISUALS (ESP) =====================
local TabESP = Window:CreateTab("Visuals (ESP)", 4483362458)
TabESP:CreateToggle({ Name = "Show Player Names", CurrentValue = false, Callback = function(v) TeamSettings.ShowName = v end })
TabESP:CreateToggle({ Name = "Show Distance", CurrentValue = false, Callback = function(v) TeamSettings.ShowDistance = v end })

local teams = {"Defenders", "Operators", "Civilians"}
for _, tName in ipairs(teams) do
    local tCfg = TeamSettings.Teams[tName]
    TabESP:CreateSection(tName .. " ESP")
    TabESP:CreateToggle({ Name = "Enable " .. tName .. " ESP", CurrentValue = false, Callback = function(v) tCfg.enabled = v end })
    TabESP:CreateColorPicker({ Name = "Color", Color = tCfg.color, Callback = function(v) tCfg.color = v end })
    TabESP:CreateToggle({ Name = "2D Box", CurrentValue = false, Callback = function(v) tCfg.box2D = v end })
    TabESP:CreateToggle({ Name = "Snapline", CurrentValue = false, Callback = function(v) tCfg.snapline = v end })
    TabESP:CreateToggle({ Name = "Health Bar", CurrentValue = false, Callback = function(v) tCfg.healthBar = v end })
end

TabESP:CreateSection("SAM & Missiles ESP (Pantsir / Tor / Strela)")
TabESP:CreateToggle({ Name = "Enable Missiles / SAM ESP", CurrentValue = DefendersSettings.ESPEnabled, Callback = function(v) DefendersSettings.ESPEnabled = v; updateHunterESP() end })
TabESP:CreateColorPicker({ Name = "Target ESP Color", Color = DefendersSettings.ESPColor, Callback = function(v) DefendersSettings.ESPColor = v; updateHunterESP() end })

-- ===================== TAB 4: FUN (MORPHS & SKY) =====================
local TabFun = Window:CreateTab("Fun", 4483362458)
local morphOptions = { ["Fpv Drone"] = "124994246147928", ["Anime Girl"] = "131402196946834", ["Custom Asset"] = "" }
local selectedMorphId = morphOptions["Fpv Drone"]

TabFun:CreateSection("Player Character Morphs")
TabFun:CreateDropdown({
    Name = "Select Model Preset",
    Options = {"Fpv Drone", "Anime Girl", "Custom Asset"},
    CurrentOption = {"Fpv Drone"},
    Callback = function(Option) selectedMorphId = morphOptions[Option[1]] end
})
TabFun:CreateInput({
    Name = "Custom Asset ID (Morph)",
    PlaceholderText = "Paste Model Asset ID",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text) morphOptions["Custom Asset"] = Text; selectedMorphId = Text end
})
TabFun:CreateButton({
    Name = "Apply Morph Model",
    Callback = function() if selectedMorphId and selectedMorphId ~= "" then morphPlayer(selectedMorphId) end end
})
TabFun:CreateButton({
    Name = "Reset / Remove Morph",
    Callback = function() unmorphPlayer() end
})

TabFun:CreateSection("Skybox Customization")
local customSkyAssetId = ""
TabFun:CreateInput({
    Name = "Skybox Asset ID",
    PlaceholderText = "Paste Skybox Texture ID",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text) customSkyAssetId = Text end
})
TabFun:CreateButton({ Name = "Apply Custom Skybox", Callback = function() applyCustomSky(customSkyAssetId) end })
TabFun:CreateButton({ Name = "Reset Default Skybox", Callback = function() resetCustomSky() end })

-- ===================== TAB 5: TELEPORTS =====================
local TabTeleport = Window:CreateTab("Teleports", 4483362458)
local selectedPlayerToTP = nil

local function getPlayerList()
    local list = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then table.insert(list, p.Name) end
    end
    if #list == 0 then table.insert(list, "No Players Found") end
    return list
end

local playerDropdown = TabTeleport:CreateDropdown({
    Name = "Select Player",
    Options = getPlayerList(),
    CurrentOption = {getPlayerList()[1]},
    MultipleOptions = false,
    Callback = function(Option) selectedPlayerToTP = Option[1] end
})

TabTeleport:CreateButton({ Name = "Refresh Player List", Callback = function() if playerDropdown then playerDropdown:Refresh(getPlayerList(), true) end end })
TabTeleport:CreateButton({
    Name = "Teleport to Player",
    Callback = function()
        if selectedPlayerToTP and selectedPlayerToTP ~= "No Players Found" then
            local target = Players:FindFirstChild(selectedPlayerToTP)
            if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                teleportTo(target.Character.HumanoidRootPart.Position + Vector3.new(0, 3, 0))
            end
        end
    end
})

TabTeleport:CreateSection("Map Points")
local teleports = {
    {"Airbase Spawn", Vector3.new(-2328.2, 2.4, -9159.5)},
    {"Ammunition Depot", Vector3.new(-3142.5, 2.4, -8926.9)},
    {"Pantsir Position", Vector3.new(-4401.5, 46.9, -7204.0)},
    {"Tor M1 Position", Vector3.new(-4401.5, 46.9, -7204.0)}
}
for _, tp in ipairs(teleports) do
    TabTeleport:CreateButton({ Name = tp[1], Callback = function() teleportTo(tp[2]) end })
end

-- ===================== TAB 6: WORLD & PLAYER =====================
local TabWorld = Window:CreateTab("World & Player", 4483362458)
local fullbrightConn

TabWorld:CreateToggle({
    Name = "Fullbright (Night Vision)",
    CurrentValue = false,
    Callback = function(v)
        if v then
            fullbrightConn = RunService.RenderStepped:Connect(function()
                Lighting.Brightness = 2; Lighting.ClockTime = 14; Lighting.FogEnd = 100000; Lighting.GlobalShadows = false
                Lighting.Ambient = Color3.fromRGB(255, 255, 255); Lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)
            end)
        else
            if fullbrightConn then fullbrightConn:Disconnect(); fullbrightConn = nil end
            Lighting.Brightness = defaultLighting.Brightness; Lighting.ClockTime = defaultLighting.ClockTime
            Lighting.FogEnd = defaultLighting.FogEnd; Lighting.GlobalShadows = defaultLighting.GlobalShadows
            Lighting.Ambient = defaultLighting.Ambient; Lighting.OutdoorAmbient = defaultLighting.OutdoorAmbient
        end
    end
})

TabWorld:CreateToggle({
    Name = "No Fog",
    CurrentValue = false,
    Callback = function(v)
        Lighting.FogEnd = v and 1000000 or defaultLighting.FogEnd
        Lighting.FogStart = v and 0 or defaultLighting.FogStart
    end
})

TabWorld:CreateSlider({
    Name = "WalkSpeed",
    Range = {16, 150},
    Increment = 2,
    CurrentValue = 16,
    Callback = function(v)
        local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.WalkSpeed = v end
    end
})

TabWorld:CreateSlider({
    Name = "Field of View (FOV)",
    Range = {50, 120},
    Increment = 2,
    CurrentValue = 70,
    Callback = function(v) Camera.FieldOfView = v end
})

-- ===================== TAB 7: SETTINGS & LANGUAGE =====================
local TabSettings = Window:CreateTab("Settings", 4483362458)
TabSettings:CreateSection("Language & Configuration")
TabSettings:CreateDropdown({
    Name = "Interface Language (Язык интерфейса)",
    Options = {"English (Default)", "Русский (Russian)"},
    CurrentOption = {"English (Default)"},
    MultipleOptions = false,
    Callback = function(opt)
        Rayfield:Notify({
            Title = "Language",
            Content = opt[1] == "Русский (Russian)" and "Выбран русский язык. Все модули во вкладках активны!" or "English layout active.",
            Duration = 3
        })
    end
})

Rayfield:Notify({
    Title = "Abstract Horse Ready",
    Content = "All features, Fun Morphs, and Anti-Ghost ESP loaded successfully!",
    Duration = 5
})
