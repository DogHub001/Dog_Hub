-- ============================================
-- DOG HUB V13 - VERSÃO ENXUTA (Delta Mobile)
-- Mouse Fictício + Auto Farm + Hitbox + Speed
-- ============================================

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local VIM = game:GetService("VirtualInputManager")

-- ========== VARIAVEIS ==========
local AutoFarmOn = false
local HitboxOn = false
local SpeedOn = false
local NivelAtual = 0
local TamanhoHitbox = 18
local WalkSpeed = 50
local JumpPower = 80

-- ========== MOUSE FICTÍCIO (O SEGREDO!) ==========
local function ClickMouse()
    pcall(function()
        local vu = game:GetService("VirtualUser")
        vu:Button1Down(Vector2.new(500, 800), Enum.UserInputType.MouseButton1)
        task.wait(0.05)
        vu:Button1Up(Vector2.new(500, 800), Enum.UserInputType.MouseButton1)
    end)
    
    pcall(function()
        VIM:SendMouseButtonEvent(500, 800, 0, true, "Left", false)
        task.wait(0.05)
        VIM:SendMouseButtonEvent(500, 800, 0, false, "Left", false)
    end)
end

-- ========== CLICK DE ATAQUE ==========
local function Atacar()
    pcall(function() VIM:SendKeyEvent(true, "Q", false, game) VIM:SendKeyEvent(false, "Q", false, game) end)
    pcall(function() VIM:SendKeyEvent(true, "E", false, game) VIM:SendKeyEvent(false, "E", false, game) end)
    pcall(function() VIM:SendKeyEvent(true, "R", false, game) VIM:SendKeyEvent(false, "R", false, game) end)
    ClickMouse()
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

-- ========== NPCs POR NÍVEL ==========
local function GetNPCByLevel()
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

-- ========== ACEITAR MISSÃO ==========
local function AceitarQuest()
    pcall(function() VIM:SendKeyEvent(true, "E", false, game) wait(0.1) VIM:SendKeyEvent(false, "E", false, game) end)
    ClickMouse()
    wait(0.5)
end

-- ========== ENCONTRAR NPC ==========
local function FindNPC(nome)
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
    if AutoFarmOn then return end
    AutoFarmOn = true
    
    FarmTask = spawn(function()
        while AutoFarmOn do
            local npcNome, pos = GetNPCByLevel()
            Teleport(pos)
            wait(1)
            
            AceitarQuest()
            wait(1)
            
            local npc, root = FindNPC(npcNome)
            if npc and root then
                local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
                local startTime = tick()
                while npc and npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 and tick() - startTime < 20 do
                    if hrp then hrp.CFrame = root.CFrame * CFrame.new(0, 2, 3) end
                    Atacar()
                    wait(0.1)
                end
            end
            wait(1)
        end
    end)
end

local function StopFarm()
    AutoFarmOn = false
    if FarmTask then task.cancel(FarmTask) FarmTask = nil end
end

-- ========== HITBOX ==========
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

-- ========== SPEED ==========
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

-- ========== GUI SIMPLES E FUNCIONAL ==========
local sg = Instance.new("ScreenGui")
sg.Name = "DogHub"
sg.Parent = LP:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 350, 0, 400)
main.Position = UDim2.new(0.5, -175, 0.5, -200)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
main.BackgroundTransparency = 0.1
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
top.BorderSizePixel = 0
top.Parent = main

local title = Instance.new("TextLabel")
title.Size = UDim2.new(0.7, 0, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🐕 DOG HUB V13"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = top

-- Fechar
local close = Instance.new("TextButton")
close.Size = UDim2.new(0, 30, 0, 30)
close.Position = UDim2.new(1, -40, 0, 10)
close.BackgroundColor3 = Color3.fromRGB(200, 60, 50)
close.Text = "✕"
close.TextColor3 = Color3.fromRGB(255, 255, 255)
close.TextSize = 16
close.Font = Enum.Font.GothamBold
close.Parent = top

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

-- Slider Speed
local speedFrame = Instance.new("Frame")
speedFrame.Size = UDim2.new(0.94, 0, 0, 65)
speedFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
speedFrame.BackgroundTransparency = 0.5
speedFrame.BorderSizePixel = 0
speedFrame.Parent = scroll
speedFrame.Visible = false

local speedSlider = Instance.new("TextBox")
speedSlider.Size = UDim2.new(0.9, 0, 0, 35)
speedSlider.Position = UDim2.new(0.05, 0, 0, 25)
speedSlider.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
speedSlider.Text = tostring(WalkSpeed)
speedSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
speedSlider.TextSize = 14
speedSlider.Font = Enum.Font.Gotham
speedSlider.Parent = speedFrame

-- Slider Pulo
local jumpFrame = Instance.new("Frame")
jumpFrame.Size = UDim2.new(0.94, 0, 0, 65)
jumpFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
jumpFrame.BackgroundTransparency = 0.5
jumpFrame.BorderSizePixel = 0
jumpFrame.Parent = scroll
jumpFrame.Visible = false

local jumpSlider = Instance.new("TextBox")
jumpSlider.Size = UDim2.new(0.9, 0, 0, 35)
jumpSlider.Position = UDim2.new(0.05, 0, 0, 25)
jumpSlider.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
jumpSlider.Text = tostring(JumpPower)
jumpSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
jumpSlider.TextSize = 14
jumpSlider.Font = Enum.Font.Gotham
jumpSlider.Parent = jumpFrame

-- Slider Hitbox
local hitboxFrame = Instance.new("Frame")
hitboxFrame.Size = UDim2.new(0.94, 0, 0, 65)
hitboxFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
hitboxFrame.BackgroundTransparency = 0.5
hitboxFrame.BorderSizePixel = 0
hitboxFrame.Parent = scroll
hitboxFrame.Visible = false

local hitboxSlider = Instance.new("TextBox")
hitboxSlider.Size = UDim2.new(0.9, 0, 0, 35)
hitboxSlider.Position = UDim2.new(0.05, 0, 0, 25)
hitboxSlider.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
hitboxSlider.Text = tostring(TamanhoHitbox)
hitboxSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
hitboxSlider.TextSize = 14
hitboxSlider.Font = Enum.Font.Gotham
hitboxSlider.Parent = hitboxFrame

-- Status
local status = Instance.new("TextLabel")
status.Size = UDim2.new(0.94, 0, 0, 30)
status.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
status.BackgroundTransparency = 0.5
status.Text = "✅ Pronto para farmar!"
status.TextColor3 = Color3.fromRGB(100, 255, 100)
status.TextSize = 12
status.Font = Enum.Font.Gotham
status.Parent = scroll

-- ========== FUNÇÕES DOS BOTÕES ==========
local hitboxVisivel = false
local speedVisivel = false

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
        status.Text = "✅ Auto Farm ATIVADO! (Mouse fictício)"
        status.TextColor3 = Color3.fromRGB(100, 255, 100)
    end
end)

hitBtn.MouseButton1Click:Connect(function()
    if HitboxOn then
        StopHitbox()
        hitBtn.Text = "🎯 ATIVAR HITBOX"
        hitBtn.BackgroundColor3 = Color3.fromRGB(79, 70, 229)
        hitboxFrame.Visible = false
        hitboxVisivel = false
    else
        StartHitbox()
        hitBtn.Text = "❌ DESATIVAR HITBOX"
        hitBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
        hitboxFrame.Visible = true
        hitboxVisivel = true
    end
end)

speedBtn.MouseButton1Click:Connect(function()
    if SpeedOn then
        StopSpeed()
        speedBtn.Text = "⚡ ATIVAR SPEED"
        speedBtn.BackgroundColor3 = Color3.fromRGB(6, 182, 212)
        speedFrame.Visible = false
        jumpFrame.Visible = false
        speedVisivel = false
    else
        StartSpeed()
        speedBtn.Text = "❌ DESATIVAR SPEED"
        speedBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
        speedFrame.Visible = true
        jumpFrame.Visible = true
        speedVisivel = true
    end
end)

speedSlider.FocusLost:Connect(function()
    local val = tonumber(speedSlider.Text)
    if val then
        WalkSpeed = math.clamp(val, 0, 300)
        speedSlider.Text = tostring(WalkSpeed)
        if SpeedOn then ApplySpeed() end
    else
        speedSlider.Text = tostring(WalkSpeed)
    end
end)

jumpSlider.FocusLost:Connect(function()
    local val = tonumber(jumpSlider.Text)
    if val then
        JumpPower = math.clamp(val, 0, 300)
        jumpSlider.Text = tostring(JumpPower)
        if SpeedOn then ApplySpeed() end
    else
        jumpSlider.Text = tostring(JumpPower)
    end
end)

hitboxSlider.FocusLost:Connect(function()
    local val = tonumber(hitboxSlider.Text)
    if val then
        TamanhoHitbox = math.clamp(val, 8, 35)
        hitboxSlider.Text = tostring(TamanhoHitbox)
    else
        hitboxSlider.Text = tostring(TamanhoHitbox)
    end
end)

close.MouseButton1Click:Connect(function()
    StopFarm()
    StopHitbox()
    StopSpeed()
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

print("🐕 DOG HUB V13 CARREGADO!")
print("✅ Mouse Fictício ativado!")
print("✅ Clique em ATIVAR AUTO FARM")
