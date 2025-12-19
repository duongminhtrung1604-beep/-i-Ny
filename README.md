-- ====== Rayfield ESP Hoàn Chỉnh Tối Ưu + KeySystem ====== --

**Link cá nhân:** [https://raw.githubusercontent.com/duongminhtrung1604-beep/-i-Ny/refs/heads/main/README.md](https://raw.githubusercontent.com/duongminhtrung1604-beep/-i-Ny/refs/heads/main/README.md)

```lua
loadstring(game:HttpGet("https://raw.githubusercontent.com/duongminhtrung1604-beep/-i-Ny/refs/heads/main/README.md"))()
```

---

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
Rayfield:LoadConfiguration()

-- ====== Window + KeySystem ======
local Window = Rayfield:CreateWindow({
    Name = "ESP Script",
    LoadingTitle = "ESP nhìn dú",
    LoadingSubtitle = "by trungsaygex",
    ConfigurationSaving = { Enabled = true, FolderName = "RayfieldESP", FileName = "Config" },
    KeySystem = true,
    KeySettings = {
        Title = "tôi đã từng yêu cô ấy",
        Subtitle = "Nhập key để sử dụng",
        Note = "Liên hệ trung dz top 1 để lấy key",
        FileName = "UniqueESPKeyFile",
        SaveKey = true,
        GrabKeyFromSite = false,
        Key = {"dungyeungoc","baosaygexyeutran","longfemini","datgay","trungsaygex"}
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
local MaxDistance = 10000
local UpdateInterval = 0.35 -- update health & billboard

-- ====== RGB Settings ======
local RGBEnabled = true     -- bật/tắt hiệu ứng RGB
local RGBSpeed = 0.25       -- tốc độ thay đổi hue (hue per second)
local hue = 0               -- giá trị hue hiện tại (0..1)

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
    textLabel.TextColor3 = Color3.fromRGB(255,182,193)
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
        healthBar.BackgroundColor3 = Color3.fromRGB(255,182,193)
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
    highlight.OutlineColor = Color3.fromRGB(255,182,193)
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Adornee = character
    highlight.Parent = character

    -- Tracer
    local tracer = Drawing.new("Line")
    tracer.Visible = false
    tracer.Color = Color3.fromRGB(255,182,193)
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

-- ====== Update Tracer (và RGB Billboard) ======
RunService.RenderStepped:Connect(function(dt)
    -- cập nhật hue nếu bật RGB
    if RGBEnabled then
        hue = (hue + RGBSpeed * dt) % 1
    end
    local currentColor = Color3.fromHSV(hue, 1, 1)

    local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not myHRP then return end

    for player,obj in pairs(ESPObjects) do
        if obj.hrp and obj.humanoid and obj.humanoid.Health>0 then
            local dist = (myHRP.Position - obj.hrp.Position).Magnitude
            local show = ESPEnabled and TracerEnabled and dist <= MaxDistance

            -- Tracer
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

            -- Áp dụng màu RGB nếu bật
            if RGBEnabled then
                -- Label (billboard text)
                if obj.label then
                    obj.label.TextColor3 = currentColor
                end
                -- Highlight outline
                if obj.highlight then
                    obj.highlight.OutlineColor = currentColor
                end
                -- Tracer color
                if obj.tracer then
                    obj.tracer.Color = currentColor
                end
                -- Health bar
                if obj.healthBar then
                    obj.healthBar.BackgroundColor3 = currentColor
                end
            end
        end
    end
end)

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
    Name = "Khoảng cách giữa Trung và lọ",
    Range = {1000,10000},
    Increment = 100,
    Suffix = "Studs",
    CurrentValue = MaxDistance,
    Callback = function(Value) MaxDistance = Value end
})

-- ====== RGB Controls ======
ESPTab:CreateToggle({
    Name = "RGB Billboard",
    CurrentValue = RGBEnabled,
    Callback = function(Value) RGBEnabled = Value end
})

ESPTab:CreateSlider({
    Name = "RGB Speed",
    Range = {1, 200}, -- percent
    Increment = 5,
    Suffix = "%",
    CurrentValue = math.floor(RGBSpeed * 100),
    Callback = function(Value)
        RGBSpeed = Value / 100
    end
})

    