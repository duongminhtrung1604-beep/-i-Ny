-- ====== Rayfield ESP Hoàn Chỉnh Tối Ưu + KeySystem ====== --
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
Rayfield:LoadConfiguration()

-- ====== Window + KeySystem ======
local Window = Rayfield:CreateWindow({
    Name = "ESP Script",
    LoadingTitle = "Rayfield cheats",
    LoadingSubtitle = "by trung",
    ConfigurationSaving = { Enabled = true, FolderName = "RayfieldESP", FileName = "Config" },
    KeySystem = true,
    KeySettings = {
        Title = "ESP Key",
        Subtitle = "Nhập key để sử dụng",
        Note = "Liên hệ admin để lấy key",
        FileName = "UniqueESPKeyFile",
        SaveKey = true,
        GrabKeyFromSite = false,
        Key = {"taoyeumayconcho","taoyeumaycondi"}
    }
})

-- ====== Tab ESP ======
local ESPTab = Window:CreateTab("ESP", 4483362458)

-- ====== Services & Variables ======
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local ESPObjects = {}
local ESPEnabled = true
local TracerEnabled = true
local HealthBarEnabled = true
local MaxDistance = 3000
local UpdateInterval = 0.35 -- update health & billboard

-- ====== Functions ======
local function removeESP(player)
    if ESPObjects[player] then
        local obj = ESPObjects[player]
        if obj.billboard then obj.billboard:Destroy() end
        if obj.highlight then obj.highlight:Destroy() end
        if obj.tracer then obj.tracer:Remove() end
        ESPObjects[player] = nil
    end
end

local function createESP(player, character)
    if player == LocalPlayer then return end
    removeESP(player)

    local hrp = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not hrp or not humanoid then return end

    -- Billboard
    local billboard = Instance.new("BillboardGui")
    billboard.Size = UDim2.new(0, 120, 0, 30)
    billboard.StudsOffset = Vector3.new(0, 10, 0) -- nâng lên 10 studs
    billboard.AlwaysOnTop = true

    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.TextColor3 = Color3.new(1,1,1)
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextSize = 14
    textLabel.Text = player.Name
    textLabel.Parent = billboard

    -- Health Bar (nhỏ hơn)
    local healthBar, healthBarBg
    if HealthBarEnabled then
        healthBarBg = Instance.new("Frame", billboard)
        healthBarBg.Size = UDim2.new(0.8, 0, 0.08, 0) -- nhỏ hơn so với trước
        healthBarBg.Position = UDim2.new(0.1, 0, 0.85, 0)
        healthBarBg.BackgroundColor3 = Color3.fromRGB(50,50,50)
        healthBarBg.BorderSizePixel = 0
        healthBarBg.Visible = HealthBarEnabled

        healthBar = Instance.new("Frame", healthBarBg)
        healthBar.Size = UDim2.new(1,0,1,0)
        healthBar.BackgroundColor3 = Color3.fromRGB(0,255,0)
        healthBar.BorderSizePixel = 0

        humanoid:GetPropertyChangedSignal("Health"):Connect(function()
            if HealthBarEnabled and healthBar then
                local hp,maxHp = humanoid.Health, humanoid.MaxHealth
                if maxHp > 0 then
                    healthBar.Size = UDim2.new(math.clamp(hp/maxHp,0,1),0,1,0)
                end
            end
        end)
    end

    -- Highlight
    local highlight = Instance.new("Highlight")
    highlight.FillTransparency = 1
    highlight.OutlineTransparency = 0
    highlight.OutlineColor = Color3.fromRGB(0,255,0)
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Adornee = character
    highlight.Parent = character

    -- Tracer
    local tracer = Drawing.new("Line")
    tracer.Visible = false
    tracer.Color = Color3.fromRGB(255,255,255)
    tracer.Thickness = 1
    tracer.Transparency = 0.8

    billboard.Adornee = hrp
    billboard.Parent = LocalPlayer:WaitForChild("PlayerGui")

    ESPObjects[player] = {
        billboard = billboard,
        label = textLabel,
        highlight = highlight,
        tracer = tracer,
        hrp = hrp,
        humanoid = humanoid,
        healthBar = healthBar,
        healthBarBg = healthBarBg
    }

    humanoid.Died:Connect(function() removeESP(player) end)
    player.CharacterRemoving:Connect(function() removeESP(player) end)
end

-- ====== Update ESP ======
task.spawn(function()
    while true do
        task.wait(UpdateInterval)
        local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if not myHRP then continue end

        for player,obj in pairs(ESPObjects) do
            if obj.hrp and obj.humanoid and obj.humanoid.Health>0 then
                local dist = (myHRP.Position - obj.hrp.Position).Magnitude
                local show = ESPEnabled and dist <= MaxDistance

                -- Billboard + Highlight
                obj.billboard.Enabled = show
                obj.highlight.Enabled = show

                -- Text + HealthBar
                if show then
                    obj.label.Text = player.Name.." ["..math.floor(dist).." studs]"
                    if HealthBarEnabled and obj.healthBar then
                        obj.healthBarBg.Visible = true
                    else
                        if obj.healthBarBg then obj.healthBarBg.Visible = false end
                    end
                else
                    if HealthBarEnabled and obj.healthBarBg then obj.healthBarBg.Visible = false end
                end
            end
        end
    end
end)

-- ====== Update Tracer ======
RunService.RenderStepped:Connect(function()
    local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not myHRP then return end

    for player,obj in pairs(ESPObjects) do
        if obj.hrp and obj.humanoid and obj.humanoid.Health>0 then
            local dist = (myHRP.Position - obj.hrp.Position).Magnitude
            local show = ESPEnabled and TracerEnabled and dist <= MaxDistance

            if show then
                local rootPos,onScreen = Camera:WorldToViewportPoint(obj.hrp.Position)
                local screenCenter = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
                obj.tracer.Visible = onScreen
                if onScreen then
                    obj.tracer.From = screenCenter
                    obj.tracer.To = Vector2.new(rootPos.X, rootPos.Y)
                end
            else
                obj.tracer.Visible = false
            end
        end
    end
end)

-- ====== Tab Player ======
local PlayerTab = Window:CreateTab("Player", 6026663707)

-- ====== Tab Advanced ======
local AdvancedTab = Window:CreateTab("Advanced", 7072456714)

-- ====== Player Features Variables ======
local UserInputService = game:GetService("UserInputService")
local FlyEnabled = false
local FlySpeed = 50
local SpeedEnabled = false
local WalkSpeed = 16
local JumpPowerEnabled = false
local JumpPower = 50
local InfiniteJumpEnabled = false
local AimbotEnabled = false
local AimbotFOV = 100
local HitboxExpanderEnabled = false
local HitboxSize = 2
local originalWalkSpeed = 16
local originalJumpPower = 50
local originalJumpHeight = 7.2
local Flying = false
local FlyVelocity = Vector3.new(0, 0, 0)
local CanInfiniteJump = false

-- ====== Fly Feature ======
local function startFly()
    local character = LocalPlayer.Character
    if not character then return end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not hrp or not humanoid then return end
    
    Flying = true
    local bodyVelocity = Instance.new("BodyVelocity", hrp)
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    
    local bodyGyro = Instance.new("BodyGyro", hrp)
    bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    bodyGyro.CFrame = hrp.CFrame
    
    local connection
    connection = RunService.RenderStepped:Connect(function()
        if not FlyEnabled or not Flying then
            bodyVelocity:Destroy()
            bodyGyro:Destroy()
            connection:Disconnect()
            Flying = false
            return
        end
        
        local moveDirection = Vector3.new(0, 0, 0)
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDirection = moveDirection + (hrp.CFrame.LookVector) end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDirection = moveDirection - (hrp.CFrame.LookVector) end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDirection = moveDirection - (hrp.CFrame.RightVector) end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDirection = moveDirection + (hrp.CFrame.RightVector) end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDirection = moveDirection + Vector3.new(0, 1, 0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then moveDirection = moveDirection - Vector3.new(0, 1, 0) end
        
        if moveDirection.Magnitude > 0 then
            moveDirection = moveDirection.Unit
        end
        
        bodyVelocity.Velocity = moveDirection * FlySpeed
        bodyGyro.CFrame = Camera.CFrame
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Climbing, false)
    end)
end

local function stopFly()
    Flying = false
    FlyEnabled = false
end

-- ====== Speed Feature ======
local function updateSpeed()
    local character = LocalPlayer.Character
    if character then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = SpeedEnabled and WalkSpeed or originalWalkSpeed
        end
    end
end

-- ====== Jump Feature ======
local function updateJumpPower()
    local character = LocalPlayer.Character
    if character then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            -- Handle both JumpPower (old) and JumpHeight (new)
            if humanoid:FindFirstChild("JumpHeight") or humanoid:IsA("Humanoid") and pcall(function() return humanoid.JumpHeight end) then
                humanoid.JumpHeight = JumpPowerEnabled and (JumpPower / 10) or originalJumpHeight
            else
                humanoid.JumpPower = JumpPowerEnabled and JumpPower or originalJumpPower
            end
        end
    end
end

-- ====== Infinite Jump Feature ======
local function setupInfiniteJump()
    local character = LocalPlayer.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.StateChanged:Connect(function(oldState, newState)
            if InfiniteJumpEnabled and newState == Enum.HumanoidStateType.Landed then
                CanInfiniteJump = true
            end
        end)
    end
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if InfiniteJumpEnabled and input.KeyCode == Enum.KeyCode.Space then
        local character = LocalPlayer.Character
        local humanoid = character and character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            CanInfiniteJump = false
        end
    end
end)

-- ====== Aimbot Feature ======
local function getClosestEnemy()
    local closestPlr = nil
    local closestDist = AimbotFOV
    local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not myHRP then return nil end
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local enemyHRP = player.Character:FindFirstChild("HumanoidRootPart")
            local enemyHum = player.Character:FindFirstChildOfClass("Humanoid")
            if enemyHRP and enemyHum and enemyHum.Health > 0 then
                local dist = (myHRP.Position - enemyHRP.Position).Magnitude
                if dist < closestDist then
                    closestDist = dist
                    closestPlr = player
                end
            end
        end
    end
    return closestPlr
end

-- ====== Hitbox Expander Feature ======
local expandedHitboxes = {}
local function expandHitboxes()
    -- This function stores original sizes when called
end

-- ====== Setup players ======
for _,player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        if player.Character then createESP(player,player.Character) end
        player.CharacterAdded:Connect(function(char) createESP(player,char) end)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        player.CharacterAdded:Connect(function(char) createESP(player,char) end)
    end
end)
Players.PlayerRemoving:Connect(removeESP)

-- ====== Rayfield Menu ======
ESPTab:CreateToggle({
    Name = "Bật/Tắt ESP",
    CurrentValue = ESPEnabled,
    Callback = function(Value) ESPEnabled = Value end
})

ESPTab:CreateToggle({
    Name = "Bật/Tắt Tracer",
    CurrentValue = TracerEnabled,
    Callback = function(Value) TracerEnabled = Value end
})

ESPTab:CreateToggle({
    Name = "Hiển thị Health Bar",
    CurrentValue = HealthBarEnabled,
    Callback = function(Value)
        HealthBarEnabled = Value
        for _,obj in pairs(ESPObjects) do
            if obj.healthBarBg then
                obj.healthBarBg.Visible = Value
            end
        end
    end
})

ESPTab:CreateSlider({
    Name = "Khoảng cách ESP",
    Range = {100,5000},
    Increment = 100,
    Suffix = "Studs",
    CurrentValue = MaxDistance,
    Callback = function(Value) MaxDistance = Value end
})

-- ====== Player Features UI ======
PlayerTab:CreateToggle({
    Name = "Fly Hack",
    CurrentValue = FlyEnabled,
    Callback = function(Value)
        FlyEnabled = Value
        if FlyEnabled then
            startFly()
        else
            stopFly()
        end
    end
})

PlayerTab:CreateSlider({
    Name = "Fly Speed",
    Range = {10, 200},
    Increment = 5,
    Suffix = " Speed",
    CurrentValue = FlySpeed,
    Callback = function(Value) FlySpeed = Value end
})

PlayerTab:CreateToggle({
    Name = "Speed Boost",
    CurrentValue = SpeedEnabled,
    Callback = function(Value)
        SpeedEnabled = Value
        updateSpeed()
    end
})

PlayerTab:CreateSlider({
    Name = "Walk Speed",
    Range = {16, 200},
    Increment = 5,
    Suffix = " Speed",
    CurrentValue = WalkSpeed,
    Callback = function(Value)
        WalkSpeed = Value
        updateSpeed()
    end
})

PlayerTab:CreateToggle({
    Name = "Jump Boost",
    CurrentValue = JumpPowerEnabled,
    Callback = function(Value)
        JumpPowerEnabled = Value
        updateJumpPower()
    end
})

PlayerTab:CreateSlider({
    Name = "Jump Power",
    Range = {50, 500},
    Increment = 10,
    Suffix = " Power",
    CurrentValue = JumpPower,
    Callback = function(Value)
        JumpPower = Value
        updateJumpPower()
    end
})

PlayerTab:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = InfiniteJumpEnabled,
    Callback = function(Value)
        InfiniteJumpEnabled = Value
        setupInfiniteJump()
    end
})

-- ====== Advanced Features UI ======
AdvancedTab:CreateToggle({
    Name = "Aimbot",
    CurrentValue = AimbotEnabled,
    Callback = function(Value)
        AimbotEnabled = Value
    end
})

AdvancedTab:CreateSlider({
    Name = "Aimbot FOV",
    Range = {50, 500},
    Increment = 10,
    Suffix = " Studs",
    CurrentValue = AimbotFOV,
    Callback = function(Value) AimbotFOV = Value end
})

AdvancedTab:CreateToggle({
    Name = "Hitbox Expander",
    CurrentValue = HitboxExpanderEnabled,
    Callback = function(Value)
        HitboxExpanderEnabled = Value
        if Value then
            expandHitboxes()
        end
    end
})

AdvancedTab:CreateSlider({
    Name = "Hitbox Size",
    Range = {1, 10},
    Increment = 0.5,
    Suffix = " x",
    CurrentValue = HitboxSize,
    Callback = function(Value)
        HitboxSize = Value
        if HitboxExpanderEnabled then
            expandHitboxes()
        end
    end
})

-- ====== Update Speed on Character Respawn ======
LocalPlayer.CharacterAdded:Connect(function(character)
    character:WaitForChild("Humanoid")
    task.wait(0.1)
    updateSpeed()
    updateJumpPower()
    setupInfiniteJump()
end)

-- Initial setup
if LocalPlayer.Character then
    setupInfiniteJump()
end

-- ====== Infinite Jump Loop ======
task.spawn(function()
    while true do
        task.wait(0.01)
        if InfiniteJumpEnabled then
            local character = LocalPlayer.Character
            if character then
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                local hrp = character:FindFirstChild("HumanoidRootPart")
                if humanoid and hrp and humanoid.Health > 0 then
                    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                        if humanoid:GetState() == Enum.HumanoidStateType.Landed or humanoid:GetState() == Enum.HumanoidStateType.Flying or humanoid:GetState() == Enum.HumanoidStateType.Falling then
                            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                        end
                    end
                end
            end
        end
    end
end)

-- ====== Aimbot Loop ======
task.spawn(function()
    while true do
        task.wait(0.05)
        if AimbotEnabled then
            local target = getClosestEnemy()
            if target and target.Character then
                local targetHRP = target.Character:FindFirstChild("HumanoidRootPart")
                if targetHRP then
                    Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetHRP.Position + targetHRP.Velocity * 0.1)
                end
            end
        end
    end
end)

-- ====== Hitbox Expander Loop ======
task.spawn(function()
    while true do
        task.wait(0.1)
        if HitboxExpanderEnabled then
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character then
                    local hrp = player.Character:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        if not expandedHitboxes[player] then
                            expandedHitboxes[player] = {
                                originalSize = hrp.Size
                            }
                        end
                        hrp.Size = expandedHitboxes[player].originalSize * HitboxSize
                    end
                end
            end
        else
            -- Restore hitboxes when disabled
            for player, data in pairs(expandedHitboxes) do
                if player.Character then
                    local hrp = player.Character:FindFirstChild("HumanoidRootPart")
                    if hrp and data.originalSize then
                        hrp.Size = data.originalSize
                    end
                end
            end
            expandedHitboxes = {}
        end
    end
end)

-- ====== New Player ESP Setup ======
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        player.CharacterAdded:Connect(function(char)
            task.wait(0.1)
            createESP(player, char)
        end)
        if player.Character then
            createESP(player, player.Character)
        end
    end
end)
