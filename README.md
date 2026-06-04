-- ============================================
-- DOG HUB V14 - CORRIGIDO PARA DELTA MOBILE
-- Baseado em Redz Hub + Atherhub
-- Teleport + Click funcionando
-- ============================================

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local VIM = game:GetService("VirtualInputManager")
local TS = game:GetService("TweenService")  -- Usar Tween ao invés de BodyVelocity

-- ========== VARIAVEIS ==========
local AutoFarmOn = false
local HitboxOn = false
local SpeedOn = false
local FlyOn = false
local NivelAtual = 0
local TamanhoHitbox = 18
local WalkSpeed = 50
local JumpPower = 80
local LastAttack = 0

-- ========== MOUSE FICTÍCIO (MULTIPLOS MÉTODOS) ==========
local function ClickMouse()
    -- Método 1: VirtualUser (funciona na maioria)
    pcall(function()
        local vu = game:GetService("VirtualUser")
        vu:Button1Down(Vector2.new(500, 800), Enum.UserInputType.MouseButton1)
        task.wait(0.05)
        vu:Button1Up(Vector2.new(500, 800), Enum.UserInputType.MouseButton1)
    end)
    
    -- Método 2: VirtualInputManager
    pcall(function()
        VIM:SendMouseButtonEvent(500, 800, 0, true, "Left", false)
        task.wait(0.05)
        VIM:SendMouseButtonEvent(500, 800, 0, false, "Left", false)
    end)
    
    -- Método 3: Click em posições diferentes (botões do Blox Fruits)
    local posicoes = {Vector2.new(400, 700), Vector2.new(600, 700), Vector2.new(500, 750)}
    for _, pos in pairs(posicoes) do
        pcall(function()
            VIM:SendMouseButtonEvent(pos.X, pos.Y, 0, true, "Left", false)
            task.wait(0.02)
            VIM:SendMouseButtonEvent(pos.X, pos.Y, 0, false, "Left", false)
        end)
    end
end

-- ========== CLICK DE ATAQUE (COM DELAY) ==========
local function Atacar()
    local now = tick()
    if now - LastAttack < 0.1 then return end
    LastAttack = now
    
    -- Teclas de ataque
    pcall(function() VIM:SendKeyEvent(true, "Q", false, game) task.wait(0.05) VIM:SendKeyEvent(false, "Q", false, game) end)
    pcall(function() VIM:SendKeyEvent(true, "E", false, game) task.wait(0.05) VIM:SendKeyEvent(false, "E", false, game) end)
    pcall(function() VIM:SendKeyEvent(true, "R", false, game) task.wait(0.05) VIM:SendKeyEvent(false, "R", false, game) end)
    
    -- Mouse click
    ClickMouse()
end

-- ========== FUNÇÃO DE TELEPORT (INSTANTÂNEO) ==========
local function Teleport(pos)
    local char = LP.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if hrp then
        -- Desativa colisão temporariamente
        for _, v in pairs(char:GetChildren()) do
            if v:IsA("BasePart") then
                v.CanCollide = false
            end
        end
        hrp.CFrame = pos
        task.wait(0.1)
        for _, v in pairs(char:GetChildren()) do
            if v:IsA("BasePart") then
                v.CanCollide = true
            end
        end
    end
end

-- ========== TWEEN TELEPORT (MOVER SUAVEMENTE) ==========
local function TweenToPosition(targetPos)
    local char = LP.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local tweenInfo = TweenInfo.new(
        (hrp.Position - targetPos.Position).Magnitude / 150,
        Enum.EasingStyle.Linear
    )
    local tween = TS:Create(hrp, tweenInfo, {CFrame = targetPos})
    tween:Play()
    tween.Completed:Wait()
end

-- ========== PEGAR NÍVEL ==========
local function GetLevel()
    local pg = LP:FindFirstChild("PlayerGui")
    if pg then
        local lt = pg:FindFirstChild("LevelText", true)
        if lt and lt:IsA("TextLabel") then
            local num = tonumber(lt.Text:match("%d+"))
            if num then NivelAtual = num end
        end
    end
    return NivelAtual
end

-- ========== NPCs POR NÍVEL (TABELA COMPLETA) ==========
local function GetNPCData()
    local lvl = GetLevel()
    local npcData = {
        {max=10, nome="Bandit", pos=CFrame.new(-1185, 4, 1350)},
        {max=20, nome="Gorilla", pos=CFrame.new(-1250, 8, 380)},
        {max=40, nome="Pirate", pos=CFrame.new(-1110, 6, 1000)},
        {max=60, nome="Brute", pos=CFrame.new(-1110, 6, 1000)},
        {max=75, nome="Desert Bandit", pos=CFrame.new(1350, 12, -650)},
        {max=90, nome="Desert Officer", pos=CFrame.new(1350, 12, -650)},
        {max=100, nome="Snow Bandit", pos=CFrame.new(-4500, 85, -800)},
        {max=120, nome="Snowman", pos=CFrame.new(-4500, 85, -800)},
        {max=130, nome="Chief Petty Officer", pos=CFrame.new(-5600, 45, -2800)},
        {max=175, nome="Sky Bandit", pos=CFrame.new(-4850, 750, -2000)},
        {max=190, nome="Dark Master", pos=CFrame.new(-4850, 750, -2000)},
        {max=210, nome="Prisoner", pos=CFrame.new(-5250, 280, -2550)},
        {max=250, nome="Dangerous Prisoner", pos=CFrame.new(-5250, 280, -2550)},
        {max=275, nome="Toga Warrior", pos=CFrame.new(1350, 8, 850)},
        {max=300, nome="Gladiator", pos=CFrame.new(1350, 8, 850)},
        {max=325, nome="Military Soldier", pos=CFrame.new(-5300, 45, -1200)},
        {max=350, nome="Military Spy", pos=CFrame.new(-5300, 45, -1200)},
        {max=400, nome="Fishman Warrior", pos=CFrame.new(3600, 60, 3100)},
        {max=450, nome="Fishman Commando", pos=CFrame.new(3600, 60, 3100)},
        {max=500, nome="God's Guard", pos=CFrame.new(5250, 710, -3650)},
        {max=575, nome="Royal Soldier", pos=CFrame.new(5250, 710, -3650)},
        {max=999, nome="Cyborg", pos=CFrame.new(1340, 16, -1570)}
    }
    
    for _, data in ipairs(npcData) do
        if lvl <= data.max then
            return data.nome, data.pos
        end
    end
    return "Cyborg", CFrame.new(1340, 16, -1570)
end

-- ========== ACEITAR MISSÃO ==========
local function AceitarQuest()
    -- Tenta interagir com o NPC da quest
    for i = 1, 3 do
        pcall(function() VIM:SendKeyEvent(true, "E", false, game) task.wait(0.1) VIM:SendKeyEvent(false, "E", false, game) end)
        ClickMouse()
        task.wait(0.3)
    end
end

-- ========== ENCONTRAR NPC INIMIGO ==========
local function FindNPC(nome)
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj ~= LP.Character then
            local hum = obj:FindFirstChild("Humanoid")
            local root = obj:FindFirstChild("HumanoidRootPart")
            if hum and hum.Health > 0 and root then
                if obj.Name:lower():find(nome:lower()) then
                    return obj, root
                end
            end
        end
    end
    return nil, nil
end

-- ========== AUTO FARM COMPLETO ==========
local FarmTask = nil
local CurrentNPC = nil
local CurrentRoot = nil

local function StartFarm()
    if AutoFarmOn then return end
    AutoFarmOn = true
    
    FarmTask = spawn(function()
        while AutoFarmOn do
            -- Pega NPC correto baseado no nível
            local npcNome, npcPos = GetNPCData()
            
            -- Teleporta para a ilha do NPC
            Teleport(npcPos)
            task.wait(1)
            
            -- Aceita a missão
            AceitarQuest()
            task.wait(1)
            
            -- Encontra o NPC inimigo
            local npc, root = FindNPC(npcNome)
            
            if npc and root then
                CurrentNPC = npc
                CurrentRoot = root
                
                -- Teleporta para o NPC
                Teleport(root.CFrame * CFrame.new(0, 2, 3))
                task.wait(0.5)
                
                -- Loop de ataque
                local startTime = tick()
                while AutoFarmOn and npc and npc.Parent and npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 and tick() - startTime < 30 do
                    -- Mantém perto do NPC
                    local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
                    if hrp and root and root.Parent then
                        hrp.CFrame = root.CFrame * CFrame.new(0, 2, 3)
                    end
                    
                    -- Ataca
                    for i = 1, 5 do
                        Atacar()
                        task.wait(0.08)
                    end
                end
            else
                -- Se não encontrou, espera um pouco
                task.wait(2)
            end
        end
    end)
end

local function StopFarm()
    AutoFarmOn = false
    if FarmTask then task.cancel(FarmTask) FarmTask = nil end
end

-- ========== HITBOX (SOMENTE NPCs, SEU BONECO NORMAL) ==========
local HitTask = nil
local function ExpandHitbox()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj ~= LP.Character then
            local hum = obj:FindFirstChild("Humanoid")
            local root = obj:FindFirstChild("HumanoidRootPart")
            if hum and hum.Health > 0 and root then
                root.Size = Vector3.new(TamanhoHitbox, TamanhoHitbox, TamanhoHitbox)
                root.Transparency = 1
                root.CanCollide = false
            end
        end
    end
end

local function StartHitbox()
    if HitboxOn then return end
    HitboxOn = true
    HitTask = RS.RenderStepped:Connect(function()
        if HitboxOn then ExpandHitbox() end
    end)
end

local function StopHitbox()
    HitboxOn = false
    if HitTask then HitTask:Disconnect() HitTask = nil end
end

-- ========== SPEED E PULO ==========
local function ApplySpeed()
    local char = LP.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = WalkSpeed
        char.Humanoid.JumpPower = JumpPower
    end
end

local SpeedTask = nil
local function StartSpeed()
    if SpeedOn then return end
    SpeedOn = true
    SpeedTask = RS.RenderStepped:Connect(function()
        if SpeedOn then ApplySpeed() end
    end)
end

local function StopSpeed()
    SpeedOn = false
    if SpeedTask then SpeedTask:Disconnect() SpeedTask = nil end
    local char = LP.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = 16
        char.Humanoid.JumpPower = 50
    end
end

-- ========== FLY (COM TWEEN - MAIS ESTÁVEL) ==========
local FlyActive = false
local FlyConnection = nil
local FlySpeed = 100

local function StartFly()
    if FlyActive then return end
    FlyActive = true
    
    local char = LP.Character
    if not char then return end
    local hum = char:FindFirstChild("Humanoid")
    if hum then
        hum.PlatformStand = true
    end
    
    FlyConnection = RS.RenderStepped:Connect(function()
        if not FlyActive then
            if FlyConnection then FlyConnection:Disconnect() end
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
        
        if move.Magnitude > 0 then
            move = move.Unit
        end
        
        local vel = (hrp.CFrame.RightVector * move.X + hrp.CFrame.UpVector * move.Y + hrp.CFrame.LookVector * move.Z) * FlySpeed
        hrp.Velocity = vel
    end)
end

local function StopFly()
    FlyActive = false
    if FlyConnection then FlyConnection:Disconnect() FlyConnection = nil end
    local char = LP.Character
    if char then
        local hum = char:FindFirstChild("Humanoid")
        if hum then
            hum.PlatformStand = false
        end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.Velocity = Vector3.new(0, 0, 0)
        end
    end
end

-- ========== GUI ==========
local sg = Instance.new("ScreenGui")
sg.Name = "DogHubV14"
sg.Parent = LP:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 380, 0, 480)
main.Position = UDim2.new(0.5, -190, 0.5, -240)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
main.BackgroundTransparency = 0.05
main.BorderSizePixel = 0
main.Parent = sg

local uic = Instance.new("UICorner")
uic.CornerRadius = UDim.new(0, 12)
uic.Parent = main

-- Topo
local top = Instance.new("Frame")
top.Size = UDim2.new(1, 0, 0, 50)
top.BackgroundColor3 = Color3.fromRGB(79, 70, 229)
top.BackgroundTransparency = 0.2
top.Parent = main

local title = Instance.new("TextLabel")
title.Size = UDim2.new(0.7, 0, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🐕 DOG HUB V14 - CORRIGIDO"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 16
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = top

local close = Instance.new("TextButton")
close.Size = UDim2.new(0, 32, 0, 32)
close.Position = UDim2.new(1, -42, 0, 9)
close.BackgroundColor3 = Color3.fromRGB(200, 60, 50)
close.Text = "✕"
close.TextColor3 = Color3.fromRGB(255, 255, 255)
close.TextSize = 18
close.Font = Enum.Font.GothamBold
close.Parent = top

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
levelFrame.BorderSizePixel = 0
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
farmBtn.Size = UDim2.new(0.94, 0, 0, 55)
farmBtn.BackgroundColor3 = Color3.fromRGB(16, 185, 129)
farmBtn.Text = "⚔️ ATIVAR AUTO FARM"
farmBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
farmBtn.TextSize = 16
farmBtn.Font = Enum.Font.GothamBold
farmBtn.Parent = scroll

-- Botão Hitbox
local hitBtn = Instance.new("TextButton")
hitBtn.Size = UDim2.new(0.94, 0, 0, 55)
hitBtn.BackgroundColor3 = Color3.fromRGB(79, 70, 229)
hitBtn.Text = "🎯 ATIVAR HITBOX"
hitBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
hitBtn.TextSize = 16
hitBtn.Font = Enum.Font.GothamBold
hitBtn.Parent = scroll

-- Botão Speed
local speedBtn = Instance.new("TextButton")
speedBtn.Size = UDim2.new(0.94, 0, 0, 55)
speedBtn.BackgroundColor3 = Color3.fromRGB(6, 182, 212)
speedBtn.Text = "⚡ ATIVAR SPEED"
speedBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
speedBtn.TextSize = 16
speedBtn.Font = Enum.Font.GothamBold
speedBtn.Parent = scroll

-- Botão Fly
local flyBtn = Instance.new("TextButton")
flyBtn.Size = UDim2.new(0.94, 0, 0, 55)
flyBtn.BackgroundColor3 = Color3.fromRGB(139, 92, 246)
flyBtn.Text = "🕊️ ATIVAR FLY"
flyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
flyBtn.TextSize = 16
flyBtn.Font = Enum.Font.GothamBold
flyBtn.Parent = scroll

-- Slider Hitbox
local hitboxSlider = Instance.new("TextBox")
hitboxSlider.Size = UDim2.new(0.94, 0, 0, 40)
hitboxSlider.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
hitboxSlider.Text = "Tamanho Hitbox: " .. TamanhoHitbox
hitboxSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
hitboxSlider.TextSize = 14
hitboxSlider.Font = Enum.Font.Gotham
hitboxSlider.Parent = scroll
hitboxSlider.Visible = false

-- Status
local status = Instance.new("TextLabel")
status.Size = UDim2.new(0.94, 0, 0, 35)
status.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
status.BackgroundTransparency = 0.5
status.Text = "✅ Dog Hub V14 Pronto!"
status.TextColor3 = Color3.fromRGB(100, 255, 100)
status.TextSize = 12
status.Font = Enum.Font.Gotham
status.Parent = scroll

-- ========== FUNÇÕES DOS BOTÕES ==========
farmBtn.MouseButton1Click:Connect(function()
    if AutoFarmOn then
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
    if HitboxOn then
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
    if SpeedOn then
        StopSpeed()
        speedBtn.Text = "⚡ ATIVAR SPEED"
        speedBtn.BackgroundColor3 = Color3.fromRGB(6, 182, 212)
    else
        StartSpeed()
        speedBtn.Text = "❌ DESATIVAR SPEED"
        speedBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
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
        TamanhoHitbox = math.clamp(val, 8, 35)
        hitboxSlider.Text = "Tamanho Hitbox: " .. TamanhoHitbox
    else
        hitboxSlider.Text = "Tamanho Hitbox: " .. TamanhoHitbox
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
local drag = false
local dragStart, startPos

top.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        drag = true
        dragStart = input.Position
        startPos = main.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if drag and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        drag = false
    end
end)

-- Atualizar nível
spawn(function()
    while true do
        levelText.Text = "📊 Nível: " .. GetLevel()
        wait(3)
    end
end)

print("🐕 DOG HUB V14 CARREGADO!")
print("✅ Teleport corrigido (instantâneo)")
print("✅ Click com múltiplos métodos")
print("✅ Clique em ATIVAR AUTO FARM")
