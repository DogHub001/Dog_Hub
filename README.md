-- ============================================
-- DOG HUB V11 - CORRIGIDO (Abas funcionando)
-- Auto Farm Inteligente | Speed 0-300 | Hitbox Invisível
-- ============================================

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local VIM = game:GetService("VirtualInputManager")

-- ========== CORES ==========
local Cores = {
    Fundo = Color3.fromRGB(18, 18, 22),
    Painel = Color3.fromRGB(25, 25, 30),
    Destaque = Color3.fromRGB(79, 70, 229),
    Secundaria = Color3.fromRGB(6, 182, 212),
    Texto = Color3.fromRGB(255, 255, 255),
    TextoEscuro = Color3.fromRGB(160, 160, 180),
    Sucesso = Color3.fromRGB(16, 185, 129),
    Perigo = Color3.fromRGB(239, 68, 68),
}

-- ========== DADOS DOS QUESTS ==========
local Quests = {
    {"Bandit", 0, 10, "Bandit", CFrame.new(-1185, 4, 1350)},
    {"Monkey", 10, 15, "Monkey", CFrame.new(-1250, 8, 380)},
    {"Gorilla", 15, 20, "Gorilla", CFrame.new(-1250, 8, 380)},
    {"Pirate", 30, 40, "Pirate", CFrame.new(-1110, 6, 1000)},
    {"Brute", 40, 55, "Brute", CFrame.new(-1110, 6, 1000)},
    {"Desert Bandit", 60, 75, "Desert Bandit", CFrame.new(1350, 12, -650)},
    {"Desert Officer", 75, 90, "Desert Officer", CFrame.new(1350, 12, -650)},
    {"Snow Bandit", 90, 100, "Snow Bandit", CFrame.new(-4500, 85, -800)},
    {"Snowman", 100, 105, "Snowman", CFrame.new(-4500, 85, -800)},
    {"Chief Petty Officer", 120, 130, "Chief Petty Officer", CFrame.new(-5600, 45, -2800)},
    {"Sky Bandit", 150, 175, "Sky Bandit", CFrame.new(-4850, 750, -2000)},
    {"Dark Master", 175, 190, "Dark Master", CFrame.new(-4850, 750, -2000)},
    {"Prisoner", 190, 210, "Prisoner", CFrame.new(-5250, 280, -2550)},
    {"Dangerous Prisoner", 210, 220, "Dangerous Prisoner", CFrame.new(-5250, 280, -2550)},
    {"Toga Warrior", 250, 275, "Toga Warrior", CFrame.new(1350, 8, 850)},
    {"Gladiator", 275, 300, "Gladiator", CFrame.new(1350, 8, 850)},
    {"Military Soldier", 300, 325, "Military Soldier", CFrame.new(-5300, 45, -1200)},
    {"Military Spy", 325, 350, "Military Spy", CFrame.new(-5300, 45, -1200)},
    {"Fishman Warrior", 375, 400, "Fishman Warrior", CFrame.new(3600, 60, 3100)},
    {"Fishman Commando", 400, 425, "Fishman Commando", CFrame.new(3600, 60, 3100)},
    {"God's Guard", 450, 475, "God's Guard", CFrame.new(5250, 710, -3650)},
    {"Royal Squad", 525, 550, "Royal Squad", CFrame.new(5250, 710, -3650)},
    {"Royal Soldier", 550, 575, "Royal Soldier", CFrame.new(5250, 710, -3650)},
    {"Galley Pirate", 625, 650, "Galley Pirate", CFrame.new(1340, 16, -1570)},
    {"Galley Captain", 650, 675, "Galley Captain", CFrame.new(1340, 16, -1570)},
    {"Cyborg", 675, 700, "Cyborg", CFrame.new(1340, 16, -1570)},
}

-- ========== VARIAVEIS ==========
local JanelaMinimizada = false
local AbaSelecionada = 1  -- 1=Farm, 2=Hitbox, 3=Speed, 4=Fly, 5=Teleport
local AutoFarmOn = false
local HitboxOn = false
local SpeedOn = false
local FlyOn = false
local NivelAtual = 0

-- Configurações
local TamanhoHitbox = 18
local WalkSpeed = 16
local JumpPower = 50

-- ========== FUNÇÃO ATUALIZAR NÍVEL ==========
local function AtualizarNivel()
    local playerGui = LP:FindFirstChild("PlayerGui")
    if playerGui then
        local levelText = playerGui:FindFirstChild("LevelText", true)
        if levelText and levelText:IsA("TextLabel") then
            local num = tonumber(levelText.Text:match("%d+"))
            if num then NivelAtual = num end
        end
    end
    return NivelAtual
end

-- ========== FUNÇÃO PARA PEGAR QUEST IDEAL ==========
local function PegarQuestIdeal()
    AtualizarNivel()
    for i, quest in ipairs(Quests) do
        if NivelAtual >= quest[2] and NivelAtual <= quest[3] then
            return i, quest
        end
    end
    return #Quests, Quests[#Quests]
end

-- ========== TELEPORT ==========
local function Teleportar(pos)
    local char = LP.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if hrp then
        for _, v in pairs(char:GetChildren()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
        hrp.CFrame = pos
        wait(0.2)
        for _, v in pairs(char:GetChildren()) do
            if v:IsA("BasePart") then v.CanCollide = true end
        end
    end
end

-- ========== AUTO CLICK ==========
local function ClickRapido(tecla)
    pcall(function() VIM:SendKeyEvent(true, tecla or "Q", false, game) end)
    pcall(function() VIM:SendKeyEvent(false, tecla or "Q", false, game) end)
end

-- ========== AUTO FARM ==========
local FarmTask = nil
local QuestAtualId = 1

local function StartAutoFarm()
    if AutoFarmOn then return end
    AutoFarmOn = true
    
    FarmTask = RS.RenderStepped:Connect(function()
        if not AutoFarmOn then 
            if FarmTask then FarmTask:Disconnect() FarmTask = nil end
            return 
        end
        
        local questId, quest = PegarQuestIdeal()
        if questId ~= QuestAtualId then
            QuestAtualId = questId
            Teleportar(quest[5])
            wait(1)
        end
        
        ClickRapido("Q")
        ClickRapido("E")
    end)
end

local function StopAutoFarm()
    AutoFarmOn = false
    if FarmTask then FarmTask:Disconnect() FarmTask = nil end
end

-- ========== HITBOX INVISÍVEL ==========
local HitboxTask = nil
local function ExpandirHitbox()
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
    HitboxTask = RS.RenderStepped:Connect(function()
        if HitboxOn then ExpandirHitbox() end
    end)
end

local function StopHitbox()
    HitboxOn = false
    if HitboxTask then HitboxTask:Disconnect() HitboxTask = nil end
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") then
            local root = obj:FindFirstChild("HumanoidRootPart")
            if root and root.Size.X > 5 then
                root.Size = Vector3.new(2, 2, 1)
                root.Transparency = 0
                root.CanCollide = true
            end
        end
    end
end

-- ========== SPEED CONTROL ==========
local function AplicarSpeedPulo()
    local char = LP.Character
    if not char then return end
    local hum = char:FindFirstChild("Humanoid")
    if hum then
        hum.WalkSpeed = WalkSpeed
        hum.JumpPower = JumpPower
    end
end

local SpeedTask = nil
local function StartSpeed()
    if SpeedOn then return end
    SpeedOn = true
    SpeedTask = RS.RenderStepped:Connect(function()
        if SpeedOn then AplicarSpeedPulo() end
    end)
end

local function StopSpeed()
    SpeedOn = false
    if SpeedTask then SpeedTask:Disconnect() SpeedTask = nil end
    local char = LP.Character
    if char then
        local hum = char:FindFirstChild("Humanoid")
        if hum then
            hum.WalkSpeed = 16
            hum.JumpPower = 50
        end
    end
end

-- ========== FLY ==========
local FlyActive = false
local bv, bg, flyConn

local function StartFly()
    if FlyActive then return end
    local char = LP.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChild("Humanoid")
    if not hrp then return end
    if hum then hum.PlatformStand = true end
    
    bv = Instance.new("BodyVelocity")
    bv.MaxForce = Vector3.new(1, 1, 1) * 100000
    bv.Parent = hrp
    
    bg = Instance.new("BodyGyro")
    bg.MaxTorque = Vector3.new(1, 1, 1) * 100000
    bg.Parent = hrp
    
    flyConn = RS.RenderStepped:Connect(function()
        if not FlyActive then
            if bv then bv:Destroy() end
            if bg then bg:Destroy() end
            if flyConn then flyConn:Disconnect() end
            return
        end
        if not hrp or not hrp.Parent then return end
        local move = Vector3.new(
            (UIS:IsKeyDown(Enum.KeyCode.D) and 1 or 0) - (UIS:IsKeyDown(Enum.KeyCode.A) and 1 or 0),
            (UIS:IsKeyDown(Enum.KeyCode.Space) and 1 or 0) - (UIS:IsKeyDown(Enum.KeyCode.LeftControl) and 1 or 0),
            (UIS:IsKeyDown(Enum.KeyCode.S) and -1 or 0) + (UIS:IsKeyDown(Enum.KeyCode.W) and 1 or 0)
        )
        if move.Magnitude > 0 then move = move.Unit end
        bv.Velocity = (hrp.CFrame.RightVector * move.X + hrp.CFrame.UpVector * move.Y + hrp.CFrame.LookVector * move.Z) * 100
        bg.CFrame = hrp.CFrame
    end)
    FlyActive = true
end

local function StopFly()
    if not FlyActive then return end
    FlyActive = false
    local char = LP.Character
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            if bv then bv:Destroy() end
            if bg then bg:Destroy() end
        end
        local hum = char:FindFirstChild("Humanoid")
        if hum then hum.PlatformStand = false end
    end
end

-- ========== CRIAR BOTÃO COM INTERAÇÃO ==========
local function CriarBotao(parent, texto, cor, tamanhoX, posX, posY, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(tamanhoX or 0.44, 0, 0, 42)
    btn.Position = UDim2.new(posX or 0.03, 0, posY or 0, 0)
    btn.BackgroundColor3 = cor
    btn.Text = texto
    btn.TextColor3 = Cores.Texto
    btn.TextSize = 14
    btn.Font = Enum.Font.GothamBold
    btn.Parent = parent
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = btn
    
    btn.MouseButton1Click:Connect(function()
        callback()
        btn.TextSize = 12
        task.wait(0.1)
        btn.TextSize = 14
    end)
    
    btn.MouseEnter:Connect(function()
        btn.BackgroundTransparency = 0.3
    end)
    btn.MouseLeave:Connect(function()
        btn.BackgroundTransparency = 0
    end)
    
    return btn
end

-- ========== CRIAR SLIDER ==========
local function CriarSlider(parent, nome, min, max, valorInicial, callback, formato)
    formato = formato or "%.0f"
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 65)
    frame.BackgroundColor3 = Cores.Painel
    frame.BackgroundTransparency = 0.5
    frame.BorderSizePixel = 0
    frame.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = frame
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.5, 0, 0, 25)
    label.Position = UDim2.new(0, 10, 0, 5)
    label.BackgroundTransparency = 1
    label.Text = nome
    label.TextColor3 = Cores.Texto
    label.TextSize = 13
    label.Font = Enum.Font.GothamSemibold
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame
    
    local valueLabel = Instance.new("TextLabel")
    valueLabel.Size = UDim2.new(0.3, 0, 0, 25)
    valueLabel.Position = UDim2.new(0.7, 0, 0, 5)
    valueLabel.BackgroundTransparency = 1
    valueLabel.Text = string.format(formato, valorInicial)
    valueLabel.TextColor3 = Cores.Destaque
    valueLabel.TextSize = 13
    valueLabel.Font = Enum.Font.GothamBold
    valueLabel.TextXAlignment = Enum.TextXAlignment.Right
    valueLabel.Parent = frame
    
    local slider = Instance.new("TextBox")
    slider.Size = UDim2.new(0.9, 0, 0, 32)
    slider.Position = UDim2.new(0.05, 0, 0, 30)
    slider.BackgroundColor3 = Cores.Painel
    slider.Text = string.format(formato, valorInicial)
    slider.TextColor3 = Cores.Texto
    slider.TextSize = 13
    slider.Font = Enum.Font.Gotham
    slider.Parent = frame
    
    local sliderCorner = Instance.new("UICorner")
    sliderCorner.CornerRadius = UDim.new(0, 6)
    sliderCorner.Parent = slider
    
    slider.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local val = tonumber(slider.Text)
            if val then
                local newVal = math.clamp(val, min, max)
                slider.Text = string.format(formato, newVal)
                valueLabel.Text = string.format(formato, newVal)
                callback(newVal)
            else
                slider.Text = string.format(formato, valorInicial)
            end
        end
    end)
    
    return frame
end

-- ========== GUI PRINCIPAL ==========
local sg = Instance.new("ScreenGui")
sg.Name = "DogHubV11"
sg.Parent = LP:WaitForChild("PlayerGui")

-- Frame principal
local main = Instance.new("Frame")
main.Size = UDim2.new(0, 750, 0, 520)
main.Position = UDim2.new(0.5, -375, 0.5, -260)
main.BackgroundColor3 = Cores.Fundo
main.BackgroundTransparency = 0.05
main.BorderSizePixel = 0
main.Parent = sg

local uic = Instance.new("UICorner")
uic.CornerRadius = UDim.new(0, 12)
uic.Parent = main

local stroke = Instance.new("UIStroke")
stroke.Color = Cores.Destaque
stroke.Thickness = 1.5
stroke.Parent = main

-- Topo
local topBar = Instance.new("Frame")
topBar.Size = UDim2.new(1, 0, 0, 45)
topBar.BackgroundColor3 = Cores.Painel
topBar.BackgroundTransparency = 0.3
topBar.BorderSizePixel = 0
topBar.Parent = main

local topCorner = Instance.new("UICorner")
topCorner.CornerRadius = UDim.new(0, 12)
topCorner.Parent = topBar

local title = Instance.new("TextLabel")
title.Size = UDim2.new(0.6, 0, 1, 0)
title.Position = UDim2.new(0, 15, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🐕 DOG HUB V11 - AUTO FARM INTELIGENTE"
title.TextColor3 = Cores.Destaque
title.TextSize = 18
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = topBar

local nivelLabel = Instance.new("TextLabel")
nivelLabel.Size = UDim2.new(0.3, 0, 1, 0)
nivelLabel.Position = UDim2.new(0.7, 0, 0, 0)
nivelLabel.BackgroundTransparency = 1
nivelLabel.Text = "📊 Nível: " .. NivelAtual
nivelLabel.TextColor3 = Cores.Secundaria
nivelLabel.TextSize = 14
nivelLabel.Font = Enum.Font.GothamBold
nivelLabel.TextXAlignment = Enum.TextXAlignment.Right
nivelLabel.Parent = topBar

-- Botões do topo
local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 35, 0, 35)
minBtn.Position = UDim2.new(1, -85, 0, 5)
minBtn.BackgroundColor3 = Cores.Painel
minBtn.Text = "−"
minBtn.TextColor3 = Cores.Texto
minBtn.TextSize = 22
minBtn.Font = Enum.Font.GothamBold
minBtn.Parent = topBar

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(1, 0)
minCorner.Parent = minBtn

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 35, 0, 35)
closeBtn.Position = UDim2.new(1, -42, 0, 5)
closeBtn.BackgroundColor3 = Cores.Perigo
closeBtn.Text = "✕"
closeBtn.TextColor3 = Cores.Texto
closeBtn.TextSize = 18
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = topBar

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(1, 0)
closeCorner.Parent = closeBtn

-- Barra lateral (ABAS)
local sidebar = Instance.new("Frame")
sidebar.Size = UDim2.new(0, 140, 1, -45)
sidebar.Position = UDim2.new(0, 0, 0, 45)
sidebar.BackgroundColor3 = Cores.Painel
sidebar.BackgroundTransparency = 0.2
sidebar.BorderSizePixel = 0
sidebar.Parent = main

-- Nomes das abas
local nomesAbas = {"⚔️ FARM", "🎯 HITBOX", "⚡ SPEED", "🕊️ FLY", "🌎 TELEPORT"}
local botoesAbas = {}

-- Criar botões das abas
for i = 1, 5 do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9, 0, 0, 45)
    btn.Position = UDim2.new(0.05, 0, 0, 10 + (i-1) * 55)
    btn.BackgroundColor3 = Cores.Destaque
    btn.BackgroundTransparency = 0.8
    btn.Text = nomesAbas[i]
    btn.TextColor3 = Cores.Texto
    btn.TextSize = 13
    btn.Font = Enum.Font.GothamSemibold
    btn.Parent = sidebar
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = btn
    
    botoesAbas[i] = btn
    
    btn.MouseButton1Click:Connect(function()
        AbaSelecionada = i
        for j, b in pairs(botoesAbas) do
            b.BackgroundTransparency = (j == i) and 0.5 or 0.8
        end
        AtualizarConteudo()
    end)
end

-- Área de conteúdo
local contentArea = Instance.new("Frame")
contentArea.Size = UDim2.new(1, -155, 1, -55)
contentArea.Position = UDim2.new(0, 145, 0, 50)
contentArea.BackgroundTransparency = 1
contentArea.Parent = main

local contentScroll = Instance.new("ScrollingFrame")
contentScroll.Size = UDim2.new(1, -20, 1, 0)
contentScroll.Position = UDim2.new(0, 10, 0, 0)
contentScroll.BackgroundTransparency = 1
contentScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
contentScroll.ScrollBarThickness = 4
contentScroll.Parent = contentArea

local contentLayout = Instance.new("UIListLayout")
contentLayout.Padding = UDim.new(0, 10)
contentLayout.Parent = contentScroll

contentLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    contentScroll.CanvasSize = UDim2.new(0, 0, 0, contentLayout.AbsoluteContentSize.Y + 20)
end)

-- ========== FUNÇÃO PARA ATUALIZAR O CONTEÚDO ==========
function AtualizarConteudo()
    -- Limpar conteúdo
    for _, child in pairs(contentScroll:GetChildren()) do
        if child:IsA("Frame") or child:IsA("TextButton") or child:IsA("TextLabel") then
            child:Destroy()
        end
    end
    
    -- ===== ABA FARM =====
    if AbaSelecionada == 1 then
        local titulo = Instance.new("TextLabel")
        titulo.Size = UDim2.new(1, 0, 0, 35)
        titulo.BackgroundColor3 = Cores.Destaque
        titulo.BackgroundTransparency = 0.3
        titulo.Text = "  ⚔️ AUTO FARM INTELIGENTE"
        titulo.TextColor3 = Cores.Texto
        titulo.TextSize = 14
        titulo.Font = Enum.Font.GothamBold
        titulo.TextXAlignment = Enum.TextXAlignment.Left
        titulo.Parent = contentScroll
        
        local statusFrame = Instance.new("Frame")
        statusFrame.Size = UDim2.new(1, 0, 0, 55)
        statusFrame.BackgroundColor3 = Cores.Painel
        statusFrame.BackgroundTransparency = 0.5
        statusFrame.BorderSizePixel = 0
        statusFrame.Parent = contentScroll
        
        local statusText = Instance.new("TextLabel")
        statusText.Size = UDim2.new(0.5, 0, 1, 0)
        statusText.Position = UDim2.new(0, 10, 0, 0)
        statusText.BackgroundTransparency = 1
        statusText.Text = "📊 Status: ❌ Desativado"
        statusText.TextColor3 = Cores.Texto
        statusText.TextSize = 14
        statusText.Font = Enum.Font.GothamSemibold
        statusText.TextXAlignment = Enum.TextXAlignment.Left
        statusText.Parent = statusFrame
        
        local questInfo = Instance.new("TextLabel")
        questInfo.Size = UDim2.new(0.6, 0, 0, 20)
        questInfo.Position = UDim2.new(0, 10, 0, 35)
        questInfo.BackgroundTransparency = 1
        questInfo.Text = "🎯 NPC: " .. (Quests[PegarQuestIdeal()] and Quests[PegarQuestIdeal()][1] or "N/A")
        questInfo.TextColor3 = Cores.TextoEscuro
        questInfo.TextSize = 11
        questInfo.Font = Enum.Font.Gotham
        questInfo.TextXAlignment = Enum.TextXAlignment.Left
        questInfo.Parent = statusFrame
        
        local farmBtn = CriarBotao(statusFrame, "ATIVAR FARM", Cores.Sucesso, 0.28, 0.68, 6, function()
            if AutoFarmOn then
                StopAutoFarm()
                statusText.Text = "📊 Status: ❌ Desativado"
                farmBtn.Text = "ATIVAR FARM"
                farmBtn.BackgroundColor3 = Cores.Sucesso
            else
                StartAutoFarm()
                statusText.Text = "📊 Status: ✅ Ativado"
                farmBtn.Text = "DESATIVAR"
                farmBtn.BackgroundColor3 = Cores.Perigo
            end
        end)
        
        local info = Instance.new("TextLabel")
        info.Size = UDim2.new(1, 0, 0, 40)
        info.BackgroundColor3 = Cores.Painel
        info.BackgroundTransparency = 0.3
        info.Text = "💡 O Auto Farm vai AUTOMATICAMENTE para o NPC correto baseado no seu nível!"
        info.TextColor3 = Cores.TextoEscuro
        info.TextSize = 12
        info.Font = Enum.Font.Gotham
        info.Parent = contentScroll
        
    -- ===== ABA HITBOX =====
    elseif AbaSelecionada == 2 then
        local titulo = Instance.new("TextLabel")
        titulo.Size = UDim2.new(1, 0, 0, 35)
        titulo.BackgroundColor3 = Cores.Destaque
        titulo.BackgroundTransparency = 0.3
        titulo.Text = "  🎯 HITBOX INVISÍVEL"
        titulo.TextColor3 = Cores.Texto
        titulo.TextSize = 14
        titulo.Font = Enum.Font.GothamBold
        titulo.TextXAlignment = Enum.TextXAlignment.Left
        titulo.Parent = contentScroll
        
        local statusFrame = Instance.new("Frame")
        statusFrame.Size = UDim2.new(1, 0, 0, 55)
        statusFrame.BackgroundColor3 = Cores.Painel
        statusFrame.BackgroundTransparency = 0.5
        statusFrame.BorderSizePixel = 0
        statusFrame.Parent = contentScroll
        
        local statusText = Instance.new("TextLabel")
        statusText.Size = UDim2.new(0.5, 0, 1, 0)
        statusText.Position = UDim2.new(0, 10, 0, 0)
        statusText.BackgroundTransparency = 1
        statusText.Text = "🎯 Status: ❌ Desativado"
        statusText.TextColor3 = Cores.Texto
        statusText.TextSize = 14
        statusText.Font = Enum.Font.GothamSemibold
        statusText.TextXAlignment = Enum.TextXAlignment.Left
        statusText.Parent = statusFrame
        
        local hitboxBtn = CriarBotao(statusFrame, "ATIVAR HITBOX", Cores.Sucesso, 0.28, 0.68, 6, function()
            if HitboxOn then
                StopHitbox()
                statusText.Text = "🎯 Status: ❌ Desativado"
                hitboxBtn.Text = "ATIVAR HITBOX"
                hitboxBtn.BackgroundColor3 = Cores.Sucesso
            else
                StartHitbox()
                statusText.Text = "🎯 Status: ✅ Ativado"
                hitboxBtn.Text = "DESATIVAR"
                hitboxBtn.BackgroundColor3 = Cores.Perigo
            end
        end)
        
        CriarSlider(contentScroll, "📏 Tamanho da Hitbox", 8, 35, TamanhoHitbox, function(v)
            TamanhoHitbox = v
        end, "%.0f")
        
        local info = Instance.new("TextLabel")
        info.Size = UDim2.new(1, 0, 0, 50)
        info.BackgroundColor3 = Cores.Painel
        info.BackgroundTransparency = 0.3
        info.Text = "💡 OS NPCS CONTINUAM NORMAIS VISUALMENTE!\nApenas a hitbox invisível deles aumenta para você acertar de longe."
        info.TextColor3 = Cores.TextoEscuro
        info.TextSize = 11
        info.Font = Enum.Font.Gotham
        info.TextWrapped = true
        info.Parent = contentScroll
        
    -- ===== ABA SPEED =====
    elseif AbaSelecionada == 3 then
        local titulo = Instance.new("TextLabel")
        titulo.Size = UDim2.new(1, 0, 0, 35)
        titulo.BackgroundColor3 = Cores.Destaque
        titulo.BackgroundTransparency = 0.3
        titulo.Text = "  ⚡ CONTROLE DE SPEED E PULO"
        titulo.TextColor3 = Cores.Texto
        titulo.TextSize = 14
        titulo.Font = Enum.Font.GothamBold
        titulo.TextXAlignment = Enum.TextXAlignment.Left
        titulo.Parent = contentScroll
        
        local statusFrame = Instance.new("Frame")
        statusFrame.Size = UDim2.new(1, 0, 0, 55)
        statusFrame.BackgroundColor3 = Cores.Painel
        statusFrame.BackgroundTransparency = 0.5
        statusFrame.BorderSizePixel = 0
        statusFrame.Parent = contentScroll
        
        local statusText = Instance.new("TextLabel")
        statusText.Size = UDim2.new(0.5, 0, 1, 0)
        statusText.Position = UDim2.new(0, 10, 0, 0)
        statusText.BackgroundTransparency = 1
        statusText.Text = "⚡ Status: ❌ Desativado"
        statusText.TextColor3 = Cores.Texto
        statusText.TextSize = 14
        statusText.Font = Enum.Font.GothamSemibold
        statusText.TextXAlignment = Enum.TextXAlignment.Left
        statusText.Parent = statusFrame
        
        local speedBtn = CriarBotao(statusFrame, "ATIVAR SPEED", Cores.Sucesso, 0.28, 0.68, 6, function()
            if SpeedOn then
                StopSpeed()
                statusText.Text = "⚡ Status: ❌ Desativado"
                speedBtn.Text = "ATIVAR SPEED"
                speedBtn.BackgroundColor3 = Cores.Sucesso
            else
                StartSpeed()
                statusText.Text = "⚡ Status: ✅ Ativado"
                speedBtn.Text = "DESATIVAR"
                speedBtn.BackgroundColor3 = Cores.Perigo
            end
        end)
        
        CriarSlider(contentScroll, "🏃‍♂️ Velocidade de Andar (0-300)", 0, 300, WalkSpeed, function(v)
            WalkSpeed = v
            if SpeedOn then AplicarSpeedPulo() end
        end, "%.0f")
        
        CriarSlider(contentScroll, "🦘 Força do Pulo (0-300)", 0, 300, JumpPower, function(v)
            JumpPower = v
            if SpeedOn then AplicarSpeedPulo() end
        end, "%.0f")
        
        local info = Instance.new("TextLabel")
        info.Size = UDim2.new(1, 0, 0, 40)
        info.BackgroundColor3 = Cores.Painel
        info.BackgroundTransparency = 0.3
        info.Text = "💡 Speed 300 = muito rápido! Use com cuidado."
        info.TextColor3 = Cores.TextoEscuro
        info.TextSize = 12
        info.Font = Enum.Font.Gotham
        info.Parent = contentScroll
        
    -- ===== ABA FLY =====
    elseif AbaSelecionada == 4 then
        local titulo = Instance.new("TextLabel")
        titulo.Size = UDim2.new(1, 0, 0, 35)
        titulo.BackgroundColor3 = Cores.Destaque
        titulo.BackgroundTransparency = 0.3
        titulo.Text = "  🕊️ FLY (VOAR)"
        titulo.TextColor3 = Cores.Texto
        titulo.TextSize = 14
        titulo.Font = Enum.Font.GothamBold
        titulo.TextXAlignment = Enum.TextXAlignment.Left
        titulo.Parent = contentScroll
        
        local statusFrame = Instance.new("Frame
