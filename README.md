-- DOG HUB - CÓDIGO COMPLETO
-- Funciona no Delta Mobile

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local VIM = game:GetService("VirtualInputManager")

-- ========== VARIAVEIS ==========
local AutoFarm = false
local AutoClick = false
local Hitbox = false
local SpeedCtrl = false
local FlyCtrl = false
local HitboxSize = 18
local WalkSpeed = 50
local JumpPower = 80

-- ========== FUNÇÃO DE CLICK ==========
local function Click()
    pcall(function()
        game:GetService("VirtualUser"):Button1Down(Vector2.new(500, 800))
        task.wait(0.05)
        game:GetService("VirtualUser"):Button1Up(Vector2.new(500, 800))
    end)
    pcall(function()
        VIM:SendKeyEvent(true, "Q", false, game)
        task.wait(0.05)
        VIM:SendKeyEvent(false, "Q", false, game)
    end)
    pcall(function()
        VIM:SendKeyEvent(true, "E", false, game)
        task.wait(0.05)
        VIM:SendKeyEvent(false, "E", false, game)
    end)
end

-- ========== PEGAR NÍVEL ==========
local function GetLevel()
    local pg = LP:FindFirstChild("PlayerGui")
    if pg then
        local lt = pg:FindFirstChild("LevelText", true)
        if lt and lt:IsA("TextLabel") then
            local num = tonumber(lt.Text:match("%d+"))
            if num then return num end
        end
    end
    return 0
end

-- ========== NPC POR NÍVEL ==========
local function GetNPC()
    local lvl = GetLevel()
    if lvl <= 10 then return "Bandit", CFrame.new(-1185, 4, 1350)
    elseif lvl <= 20 then return "Gorilla", CFrame.new(-1250, 8, 380)
    elseif lvl <= 40 then return "Pirate", CFrame.new(-1110, 6, 1000)
    elseif lvl <= 60 then return "Brute", CFrame.new(-1110, 6, 1000)
    elseif lvl <= 75 then return "Desert Bandit", CFrame.new(1350, 12, -650)
    elseif lvl <= 90 then return "Desert Officer", CFrame.new(1350, 12, -650)
    elseif lvl <= 100 then return "Snow Bandit", CFrame.new(-4500, 85, -800)
    elseif lvl <= 120 then return "Snowman", CFrame.new(-4500, 85, -800)
    elseif lvl <= 130 then return "Chief Petty Officer", CFrame.new(-5600, 45, -2800)
    elseif lvl <= 175 then return "Sky Bandit", CFrame.new(-4850, 750, -2000)
    elseif lvl <= 190 then return "Dark Master", CFrame.new(-4850, 750, -2000)
    elseif lvl <= 210 then return "Prisoner", CFrame.new(-5250, 280, -2550)
    elseif lvl <= 250 then return "Dangerous Prisoner", CFrame.new(-5250, 280, -2550)
    elseif lvl <= 275 then return "Toga Warrior", CFrame.new(1350, 8, 850)
    elseif lvl <= 300 then return "Gladiator", CFrame.new(1350, 8, 850)
    elseif lvl <= 325 then return "Military Soldier", CFrame.new(-5300, 45, -1200)
    elseif lvl <= 350 then return "Military Spy", CFrame.new(-5300, 45, -1200)
    elseif lvl <= 400 then return "Fishman Warrior", CFrame.new(3600, 60, 3100)
    elseif lvl <= 450 then return "Fishman Commando", CFrame.new(3600, 60, 3100)
    elseif lvl <= 500 then return "God's Guard", CFrame.new(5250, 710, -3650)
    elseif lvl <= 575 then return "Royal Soldier", CFrame.new(5250, 710, -3650)
    else return "Cyborg", CFrame.new(1340, 16, -1570)
    end
end

-- ========== TELEPORT ==========
local function Teleport(pos)
    local char = LP.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = pos
    end
end

-- ========== ENCONTRAR INIMIGO ==========
local function FindEnemy(nome)
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj ~= LP.Character then
            local hum = obj:FindFirstChild("Humanoid")
            local root = obj:FindFirstChild("HumanoidRootPart")
            if hum and hum.Health > 0 and root and obj.Name:lower():find(nome:lower()) then
                return obj, root
            end
        end
    end
    return nil, nil
end

-- ========== AUTO FARM ==========
local FarmTask = nil

local function StartFarm()
    if AutoFarm then return end
    AutoFarm = true
    
    FarmTask = spawn(function()
        while AutoFarm do
            local npcName, npcPos = GetNPC()
            Teleport(npcPos)
            task.wait(1)
            
            pcall(function() VIM:SendKeyEvent(true, "E", false, game) task.wait(0.2) VIM:SendKeyEvent(false, "E", false, game) end)
            task.wait(1)
            
            local enemy, root = FindEnemy(npcName)
            if enemy and root then
                local start = tick()
                while enemy and enemy.Parent and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and tick() - start < 25 do
                    local char = LP.Character
                    if char and char:FindFirstChild("HumanoidRootPart") then
                        char.HumanoidRootPart.CFrame = root.CFrame * CFrame.new(0, 2, 3)
                    end
                    for i = 1, 5 do
                        Click()
                        task.wait(0.08)
                    end
                end
            end
            task.wait(0.5)
        end
    end)
end

local function StopFarm()
    AutoFarm = false
    if FarmTask then task.cancel(FarmTask) FarmTask = nil end
end

-- ========== AUTO CLICK ==========
local ClickTask = nil

local function StartAutoClick()
    if AutoClick then return end
    AutoClick = true
    ClickTask = spawn(function()
        while AutoClick do
            Click()
            task.wait(0.08)
        end
    end)
end

local function StopAutoClick()
    AutoClick = false
    if ClickTask then task.cancel(ClickTask) ClickTask = nil end
end

-- ========== HITBOX ==========
local HitTask = nil

local function StartHitbox()
    if Hitbox then return end
    Hitbox = true
    HitTask = RS.RenderStepped:Connect(function()
        if Hitbox then
            for _, obj in pairs(workspace:GetDescendants()) do
                if obj:IsA("Model") and obj ~= LP.Character then
                    local hum = obj:FindFirstChild("Humanoid")
                    local root = obj:FindFirstChild("HumanoidRootPart")
                    if hum and hum.Health > 0 and root then
                        root.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize)
                        root.Transparency = 1
                        root.CanCollide = false
                    end
                end
            end
        end
    end)
end

local function StopHitbox()
    Hitbox = false
    if HitTask then HitTask:Disconnect() HitTask = nil end
end

-- ========== SPEED ==========
local SpeedTask = nil

local function StartSpeed()
    if SpeedCtrl then return end
    SpeedCtrl = true
    SpeedTask = RS.RenderStepped:Connect(function()
        if SpeedCtrl and LP.Character and LP.Character:FindFirstChild("Humanoid") then
            LP.Character.Humanoid.WalkSpeed = WalkSpeed
            LP.Character.Humanoid.JumpPower = JumpPower
        end
    end)
end

local function StopSpeed()
    SpeedCtrl = false
    if SpeedTask then SpeedTask:Disconnect() SpeedTask = nil end
    if LP.Character and LP.Character:FindFirstChild("Humanoid") then
        LP.Character.Humanoid.WalkSpeed = 16
        LP.Character.Humanoid.JumpPower = 50
    end
end

-- ========== FLY ==========
local FlyActive = false
local FlyConn = nil

local function StartFly()
    if FlyActive then return end
    FlyActive = true
    local char = LP.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.PlatformStand = true
    end
    FlyConn = RS.RenderStepped:Connect(function()
        if not FlyActive then
            if FlyConn then FlyConn:Disconnect() end
            return
        end
        local char = LP.Character
        if not char then return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        local move = Vector3.new(
            (UIS:IsKeyDown(Enum.KeyCode.D) and 1 or 0) - (UIS:IsKeyDown(Enum.KeyCode.A) and 1 or 0),
            (UIS:IsKeyDown(Enum.KeyCode.Space) and 1 or 0) - (UIS:IsKeyDown(Enum.KeyCode.LeftControl) and 1 or 0),
            (UIS:IsKeyDown(Enum.KeyCode.S) and -1 or 0) + (UIS:IsKeyDown(Enum.KeyCode.W) and 1 or 0)
        )
        if move.Magnitude > 0 then move = move.Unit end
        hrp.Velocity = (hrp.CFrame.RightVector * move.X + hrp.CFrame.UpVector * move.Y + hrp.CFrame.LookVector * move.Z) * 100
    end)
end

local function StopFly()
    FlyActive = false
    if FlyConn then FlyConn:Disconnect() FlyConn = nil end
    local char = LP.Character
    if char then
        if char:FindFirstChild("Humanoid") then char.Humanoid.PlatformStand = false end
        if char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.Velocity = Vector3.new(0, 0, 0) end
    end
end

-- ========== GUI ==========
local sg = Instance.new("ScreenGui")
sg.Name = "DogHub"
sg.Parent = LP:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 350, 0, 450)
main.Position = UDim2.new(0.5, -175, 0.5, -225)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
main.BackgroundTransparency = 0.05
main.BorderSizePixel = 0
main.Parent = sg

local uic = Instance.new("UICorner")
uic.CornerRadius = UDim.new(0, 12)
uic.Parent = main

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(255, 100, 50)
stroke.Thickness = 2
stroke.Parent = main

-- Topo
local top = Instance.new("Frame")
top.Size = UDim2.new(1, 0, 0, 50)
top.BackgroundColor3 = Color3.fromRGB(255, 100, 50)
top.BackgroundTransparency = 0.2
top.Parent = main

local title = Instance.new("TextLabel")
title.Size = UDim2.new(0.7, 0, 1, 0)
title.Position = UDim2.new(0, 15, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🐕 DOG HUB"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 22
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = top

local close = Instance.new("TextButton")
close.Size = UDim2.new(0, 35, 0, 35)
close.Position = UDim2.new(1, -45, 0, 8)
close.BackgroundColor3 = Color3.fromRGB(200, 60, 50)
close.Text = "✕"
close.TextColor3 = Color3.fromRGB(255, 255, 255)
close.TextSize = 18
close.Font = Enum.Font.GothamBold
close.Parent = top

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(1, 0)
closeCorner.Parent = close

local min = Instance.new("TextButton")
min.Size = UDim2.new(0, 35, 0, 35)
min.Position = UDim2.new(1, -88, 0, 8)
min.BackgroundColor3 = Color3.fromRGB(80, 80, 90)
min.Text = "−"
min.TextColor3 = Color3.fromRGB(255, 255, 255)
min.TextSize = 24
min.Font = Enum.Font.GothamBold
min.Parent = top

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(1, 0)
minCorner.Parent = min

-- Scroll
local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1, 0, 1, -60)
scroll.Position = UDim2.new(0, 0, 0, 55)
scroll.BackgroundTransparency = 1
scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
scroll.ScrollBarThickness = 4
scroll.Parent = main

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 10)
layout.Parent = scroll

layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    scroll.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 20)
end)

-- Nível
local levelFrame = Instance.new("Frame")
levelFrame.Size = UDim2.new(0.94, 0, 0, 45)
levelFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
levelFrame.BackgroundTransparency = 0.5
levelFrame.Parent = scroll

local levelText = Instance.new("TextLabel")
levelText.Size = UDim2.new(1, 0, 1, 0)
levelText.BackgroundTransparency = 1
levelText.Text = "📊 Nível: " .. GetLevel()
levelText.TextColor3 = Color3.fromRGB(255, 255, 255)
levelText.TextSize = 16
levelText.Font = Enum.Font.GothamBold
levelText.Parent = levelFrame

-- Botão Farm
local farmBtn = Instance.new("TextButton")
farmBtn.Size = UDim2.new(0.94, 0, 0, 50)
farmBtn.BackgroundColor3 = Color3.fromRGB(16, 185, 129)
farmBtn.Text = "⚔️ ATIVAR AUTO FARM"
farmBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
farmBtn.TextSize = 15
farmBtn.Font = Enum.Font.GothamBold
farmBtn.Parent = scroll

-- Botão Hitbox
local hitBtn = Instance.new("TextButton")
hitBtn.Size = UDim2.new(0.94, 0, 0, 50)
hitBtn.BackgroundColor3 = Color3.fromRGB(79, 70, 229)
hitBtn.Text = "🎯 ATIVAR HITBOX"
hitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
hitBtn.TextSize = 15
hitBtn.Font = Enum.Font.GothamBold
hitBtn.Parent = scroll

-- Botão Speed
local speedBtn = Instance.new("TextButton")
speedBtn.Size = UDim2.new(0.94, 0, 0, 50)
speedBtn.BackgroundColor3 = Color3.fromRGB(6, 182, 212)
speedBtn.Text = "⚡ ATIVAR SPEED"
speedBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
speedBtn.TextSize = 15
speedBtn.Font = Enum.Font.GothamBold
speedBtn.Parent = scroll

-- Botão Fly
local flyBtn = Instance.new("TextButton")
flyBtn.Size = UDim2.new(0.94, 0, 0, 50)
flyBtn.BackgroundColor3 = Color3.fromRGB(139, 92, 246)
flyBtn.Text = "🕊️ ATIVAR FLY"
flyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
flyBtn.TextSize = 15
flyBtn.Font = Enum.Font.GothamBold
flyBtn.Parent = scroll

-- Slider Hitbox
local hitboxSlider = Instance.new("TextBox")
hitboxSlider.Size = UDim2.new(0.94, 0, 0, 40)
hitboxSlider.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
hitboxSlider.Text = "Tamanho Hitbox: " .. HitboxSize
hitboxSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
hitboxSlider.TextSize = 14
hitboxSlider.Font = Enum.Font.Gotham
hitboxSlider.Parent = scroll
hitboxSlider.Visible = false

-- Slider Speed
local speedSlider = Instance.new("TextBox")
speedSlider.Size = UDim2.new(0.94, 0, 0, 40)
speedSlider.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
speedSlider.Text = "Velocidade: " .. WalkSpeed
speedSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
speedSlider.TextSize = 14
speedSlider.Font = Enum.Font.Gotham
speedSlider.Parent = scroll
speedSlider.Visible = false

-- Slider Pulo
local jumpSlider = Instance.new("TextBox")
jumpSlider.Size = UDim2.new(0.94, 0, 0, 40)
jumpSlider.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
jumpSlider.Text = "Pulo: " .. JumpPower
jumpSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
jumpSlider.TextSize = 14
jumpSlider.Font = Enum.Font.Gotham
jumpSlider.Parent = scroll
jumpSlider.Visible = false

-- Status
local status = Instance.new("TextLabel")
status.Size = UDim2.new(0.94, 0, 0, 35)
status.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
status.BackgroundTransparency = 0.5
status.Text = "✅ Dog Hub Carregado!"
status.TextColor3 = Color3.fromRGB(100, 255, 100)
status.TextSize = 12
status.Font = Enum.Font.Gotham
status.Parent = scroll

-- ========== EVENTOS ==========
local minimized = false

min.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        scroll.Visible = false
        main.Size = UDim2.new(0, 350, 0, 70)
        min.Text = "+"
    else
        scroll.Visible = true
        main.Size = UDim2.new(0, 350, 0, 450)
        min.Text = "−"
    end
end)

farmBtn.MouseButton1Click:Connect(function()
    if AutoFarm then
        StopFarm()
        farmBtn.Text = "⚔️ ATIVAR AUTO FARM"
        farmBtn.BackgroundColor3 = Color3.fromRGB(16, 185, 129)
        status.Text = "❌ Auto Farm desativado"
        status.TextColor3 = Color3.fromRGB(255, 100, 100)
    else
        StartFarm()
        farmBtn.Text = "⏸️ DESATIVAR FARM"
        farmBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
        status.Text = "✅ Auto Farm ATIVADO!"
        status.TextColor3 = Color3.fromRGB(100, 255, 100)
    end
end)

hitBtn.MouseButton1Click:Connect(function()
    if Hitbox then
        StopHitbox()
        hitBtn.Text = "🎯 ATIVAR HITBOX"
        hitBtn.BackgroundColor3 = Color3.fromRGB(79, 70, 229)
        hitboxSlider.Visible = false
    else
        StartHitbox()
        hitBtn.Text = "❌ DESATIVAR HITBOX"
        hitBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
        hitboxSlider.Visible = true
    end
end)

speedBtn.MouseButton1Click:Connect(function()
    if SpeedCtrl then
        StopSpeed()
        speedBtn.Text = "⚡ ATIVAR SPEED"
        speedBtn.BackgroundColor3 = Color3.fromRGB(6, 182, 212)
        speedSlider.Visible = false
        jumpSlider.Visible = false
    else
        StartSpeed()
        speedBtn.Text = "❌ DESATIVAR SPEED"
        speedBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
        speedSlider.Visible = true
        jumpSlider.Visible = true
    end
end)

flyBtn.MouseButton1Click:Connect(function()
    if FlyActive then
        StopFly()
        flyBtn.Text = "🕊️ ATIVAR FLY"
        flyBtn.BackgroundColor3 = Color3.fromRGB(139, 92, 246)
    else
        StartFly()
        flyBtn.Text = "❌ DESATIVAR FLY"
        flyBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
    end
end)

hitboxSlider.FocusLost:Connect(function()
    local val = tonumber(hitboxSlider.Text:match("%d+"))
    if val then
        HitboxSize = math.clamp(val, 8, 35)
        hitboxSlider.Text = "Tamanho Hitbox: " .. HitboxSize
    else
        hitboxSlider.Text = "Tamanho Hitbox: " .. HitboxSize
    end
end)

speedSlider.FocusLost:Connect(function()
    local val = tonumber(speedSlider.Text:match("%d+"))
    if val then
        WalkSpeed = math.clamp(val, 16, 300)
        speedSlider.Text = "Velocidade: " .. WalkSpeed
        if SpeedCtrl and LP.Character and LP.Character:FindFirstChild("Humanoid") then
            LP.Character.Humanoid.WalkSpeed = WalkSpeed
        end
    else
        speedSlider.Text = "Velocidade: " .. WalkSpeed
    end
end)

jumpSlider.FocusLost:Connect(function()
    local val = tonumber(jumpSlider.Text:match("%d+"))
    if val then
        JumpPower = math.clamp(val, 50, 300)
        jumpSlider.Text = "Pulo: " .. JumpPower
        if SpeedCtrl and LP.Character and LP.Character:FindFirstChild("Humanoid") then
            LP.Character.Humanoid.JumpPower = JumpPower
        end
    else
        jumpSlider.Text = "Pulo: " .. JumpPower
    end
end)

close.MouseButton1Click:Connect(function()
    StopFarm()
    StopHitbox()
    StopSpeed()
    StopFly()
    sg:Destroy()
end)

-- Arrastar
local dragging = false
local dragStart, startPos

top.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = main.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Atualizar nível
spawn(function()
    while true do
        levelText.Text = "📊 Nível: " .. GetLevel()
        wait(3)
    end
end)

print("🐕 DOG HUB CARREGADO COM SUCESSO!")
