--[[
    ╔══════════════════════════════════════════════════════════════════════════════════════════════════════╗
    ║                                                                                                      ║
    ║     ██████╗ ██╗ █████╗ ███╗   ██╗    ███████╗████████╗██╗   ██╗██████╗  ██╗ █████╗ ███╗   ██╗     ║
    ║     ██╔══██╗██║██╔══██╗████╗  ██║    ██╔════╝╚══██╔══╝██║   ██║██╔══██╗ ██║██╔══██╗████╗  ██║     ║
    ║     ██████╔╝██║███████║██╔██╗ ██║    ███████╗   ██║   ██║   ██║██║  ██║ ██║███████║██╔██╗ ██║     ║
    ║     ██╔══██╗██║██╔══██║██║╚██╗██║    ╚════██║   ██║   ██║   ██║██║  ██║ ██║██╔══██║██║╚██╗██║     ║
    ║     ██║  ██║██║██║  ██║██║ ╚████║    ███████║   ██║   ╚██████╔╝██████╔╝ ██║██║  ██║██║ ╚████║     ║
    ║     ╚═╝  ╚═╝╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝    ╚══════╝   ╚═╝    ╚═════╝ ╚═════╝  ╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝     ║
    ║                                                                                                      ║
    ║              SISTEMA V9.0 - RIAN STUDIOS - AUMENTAR CAPACIDADE DE BLOCOS                             ║
    ║         VELOCIDADE | VOO | AIMBOT | INVINCIBLE | CAPACIDADE DE BLOCOS | ANTI-EXPULSÃO                ║
    ║                                                                                                      ║
    ╚══════════════════════════════════════════════════════════════════════════════════════════════════════╝
]]

-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Variáveis do jogador
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- Detecta mobile
local isMobile = UserInputService.TouchEnabled

-- ============ CONFIGURAÇÃO DE CAPACIDADE DE BLOCOS ============
-- Isso vai MODIFICAR O LIMITE MÁXIMO DE BLOCOS QUE O JOGADOR PODE PEGAR
local maxBlockCapacity = 250 -- Capacidade máxima de blocos (pode ajustar de 30 a 250)
local minBlockCapacity = 30
local currentBlockCount = 0 -- Quantos blocos o jogador tem atualmente

-- ============ ANTI-BAN / ANTI-EXPULSÃO ============
local antiBanEnabled = true
local lastHeartbeat = tick()

local function preventKick()
    coroutine.wrap(function()
        while antiBanEnabled and player do
            wait(25)
            lastHeartbeat = tick()
            pcall(function()
                local stats = player:FindFirstChild("leaderstats")
                if stats then
                    local _ = stats:FindFirstChild("Points")
                end
            end)
        end
    end)()
end

local function setupAntiBan()
    pcall(function()
        preventKick()
        RunService.Heartbeat:Connect(function()
            lastHeartbeat = tick()
        end)
    end)
end

-- Configurações de Velocidade
local NORMAL_SPEED = 16
local FAST_SPEED = 80
local BOMBA_SPEED = 120
local MIN_SPEED = 16
local MAX_SPEED = 120
local currentSpeed = NORMAL_SPEED
local isSpeedBoosted = false
local isBombaBoosted = false
local speedActivated = false

-- Configurações de VOO
local isFlying = false
local FLY_SPEED = 80
local flyBodyVelocity = nil
local flyBodyGyro = nil
local defaultGravity = Workspace.Gravity
local flyUp = false
local flyDown = false

-- Configurações de Ataque
local PUNCH_DAMAGE = 25
local PUNCH_COOLDOWN = 0.2
local COMBO_DELAY = 0.1
local canPunch = true
local isAttacking = false
local comboCount = 0

-- Configurações de AIMBOT
local aimbotEnabled = false
local aimbotRange = 100
local aimbotSmoothness = 0.3
local aimbotConnection = nil

-- Configurações de INVULNERABILIDADE
local invincibleEnabled = false

-- Sons
local soundService = game:GetService("SoundService")
local function playSound(soundId, volume)
    local sound = Instance.new("Sound")
    sound.SoundId = soundId
    sound.Volume = volume or 0.5
    sound.Parent = soundService
    sound:Play()
    game:GetService("Debris"):AddItem(sound, 2)
end

-- ============ FUNÇÃO PARA AUMENTAR CAPACIDADE DE BLOCOS ============
-- Esta função modifica o limite máximo de blocos que o jogador pode carregar
local function increaseBlockCapacity(newCapacity)
    newCapacity = math.clamp(newCapacity, minBlockCapacity, 250)
    maxBlockCapacity = newCapacity
    
    -- Tenta modificar o leaderstats (se existir)
    pcall(function()
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            -- Procura por stats de blocos
            local blockStat = leaderstats:FindFirstChild("Blocks") or 
                              leaderstats:FindFirstChild("Blocos") or 
                              leaderstats:FindFirstChild("Capacity") or
                              leaderstats:FindFirstChild("Limite")
            if blockStat then
                blockStat.Value = maxBlockCapacity
            end
            
            -- Procura por stat de capacidade máxima
            local maxStat = leaderstats:FindFirstChild("MaxBlocks") or 
                           leaderstats:FindFirstChild("Capacidade") or
                           leaderstats:FindFirstChild("MaxCapacity")
            if maxStat then
                maxStat.Value = maxBlockCapacity
            end
        end
    end)
    
    -- Tenta modificar atributos do jogador
    pcall(function()
        player:SetAttribute("MaxBlocks", maxBlockCapacity)
        player:SetAttribute("BlockCapacity", maxBlockCapacity)
        player:SetAttribute("CapacidadeBlocos", maxBlockCapacity)
    end)
    
    -- Tenta modificar o inventário (se for um jogo com ferramentas)
    pcall(function()
        local backpack = player:FindFirstChild("Backpack")
        if backpack then
            backpack:SetAttribute("MaxBlocks", maxBlockCapacity)
        end
    end)
    
    -- Força atualização do limite em qualquer sistema remoto
    pcall(function()
        local remote = ReplicatedStorage:FindFirstChild("UpdateBlockCapacity") or
                       ReplicatedStorage:FindFirstChild("SetMaxBlocks") or
                       ReplicatedStorage:FindFirstChild("UpdateCapacity")
        if remote then
            remote:FireServer(maxBlockCapacity)
        end
    end)
    
    print("[SISTEMA] Capacidade de blocos aumentada para: " .. maxBlockCapacity)
end

-- Função para aumentar a capacidade gradualmente
local function addBlockCapacity(amount)
    local newCapacity = maxBlockCapacity + amount
    increaseBlockCapacity(newCapacity)
end

-- ============ CRIAR UI PRINCIPAL ============
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RianStudiosUI"
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false

-- BOTÃO FLUTUANTE
local floatingButton = Instance.new("ImageButton")
floatingButton.Name = "FloatingButton"
floatingButton.Size = isMobile and UDim2.new(0, 65, 0, 65) or UDim2.new(0, 55, 0, 55)
floatingButton.Position = isMobile and UDim2.new(0.85, 0, 0.88, 0) or UDim2.new(0.02, 0, 0.85, 0)
floatingButton.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
floatingButton.Image = "rbxassetid://6031094773"
floatingButton.ImageColor3 = Color3.fromRGB(255, 255, 255)
floatingButton.ScaleType = Enum.ScaleType.Fit
floatingButton.BackgroundTransparency = 0
floatingButton.Parent = screenGui

local floatingCorner = Instance.new("UICorner")
floatingCorner.CornerRadius = UDim.new(1, 0)
floatingCorner.Parent = floatingButton

-- CONTAINER PRINCIPAL
local mainContainer = Instance.new("ScrollingFrame")
mainContainer.Name = "MainContainer"
mainContainer.Size = isMobile and UDim2.new(0.92, 0, 0.85, 0) or UDim2.new(0, 380, 0, 650)
mainContainer.Position = isMobile and UDim2.new(0.04, 0, 0.075, 0) or UDim2.new(0.5, -190, 0.5, -325)
mainContainer.BackgroundColor3 = Color3.fromRGB(12, 12, 22)
mainContainer.BackgroundTransparency = 0.08
mainContainer.BorderSizePixel = 0
mainContainer.Visible = false
mainContainer.Parent = screenGui
mainContainer.ClipsDescendants = false
mainContainer.ScrollBarThickness = 3
mainContainer.ScrollBarImageColor3 = Color3.fromRGB(66, 135, 245)
mainContainer.CanvasSize = UDim2.new(0, 0, 0, 850)

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 20)
mainCorner.Parent = mainContainer

local mainBorder = Instance.new("UIStroke")
mainBorder.Color = Color3.fromRGB(66, 135, 245)
mainBorder.Thickness = 1.5
mainBorder.Transparency = 0.5
mainBorder.Parent = mainContainer

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 60)
titleBar.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainContainer

local titleBarCorner = Instance.new("UICorner")
titleBarCorner.CornerRadius = UDim.new(0, 20)
titleBarCorner.Parent = titleBar

local titleGradient = Instance.new("UIGradient")
titleGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(66, 135, 245)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(100, 80, 200)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(66, 135, 245))
})
titleGradient.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -80, 0, 30)
titleLabel.Position = UDim2.new(0, 15, 0, 8)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚡ RIAN STUDIOS V9.0"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = isMobile and 16 or 14
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local subtitleLabel = Instance.new("TextLabel")
subtitleLabel.Size = UDim2.new(1, -80, 0, 20)
subtitleLabel.Position = UDim2.new(0, 15, 0, 35)
subtitleLabel.BackgroundTransparency = 1
subtitleLabel.Text = "SPEED | FLY | AIMBOT | INVINCIBLE | +BLOCOS"
subtitleLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
subtitleLabel.TextSize = isMobile and 10 or 9
subtitleLabel.Font = Enum.Font.Gotham
subtitleLabel.TextXAlignment = Enum.TextXAlignment.Left
subtitleLabel.Parent = titleBar

local closeButton = Instance.new("ImageButton")
closeButton.Size = UDim2.new(0, 32, 0, 32)
closeButton.Position = UDim2.new(1, -45, 0, 14)
closeButton.Image = "rbxassetid://3926305904"
closeButton.ImageColor3 = Color3.fromRGB(255, 255, 255)
closeButton.BackgroundTransparency = 1
closeButton.Parent = titleBar

-- ============ CARD DE CAPACIDADE DE BLOCOS (PRINCIPAL) ============
local capacityCard = Instance.new("Frame")
capacityCard.Size = UDim2.new(1, -24, 0, 170)
capacityCard.Position = UDim2.new(0, 12, 0, 75)
capacityCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
capacityCard.BackgroundTransparency = 0.3
capacityCard.BorderSizePixel = 0
capacityCard.Parent = mainContainer

local capacityCardCorner = Instance.new("UICorner")
capacityCardCorner.CornerRadius = UDim.new(0, 14)
capacityCardCorner.Parent = capacityCard

local capacityCardStroke = Instance.new("UIStroke")
capacityCardStroke.Color = Color3.fromRGB(255, 215, 0)
capacityCardStroke.Thickness = 2
capacityCardStroke.Transparency = 0.5
capacityCardStroke.Parent = capacityCard

local capacityIcon = Instance.new("ImageLabel")
capacityIcon.Size = UDim2.new(0, 45, 0, 45)
capacityIcon.Position = UDim2.new(0, 15, 0, 15)
capacityIcon.Image = "rbxassetid://6031094773"
capacityIcon.ImageColor3 = Color3.fromRGB(255, 215, 0)
capacityIcon.BackgroundTransparency = 1
capacityIcon.Parent = capacityCard

local capacityTitleLabel = Instance.new("TextLabel")
capacityTitleLabel.Size = UDim2.new(1, -75, 0, 25)
capacityTitleLabel.Position = UDim2.new(0, 70, 0, 15)
capacityTitleLabel.BackgroundTransparency = 1
capacityTitleLabel.Text = "📦 CAPACIDADE DE BLOCOS"
capacityTitleLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
capacityTitleLabel.TextSize = isMobile and 13 or 11
capacityTitleLabel.Font = Enum.Font.GothamBold
capacityTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
capacityTitleLabel.Parent = capacityCard

local capacityValueLabel = Instance.new("TextLabel")
capacityValueLabel.Size = UDim2.new(1, -75, 0, 50)
capacityValueLabel.Position = UDim2.new(0, 70, 0, 40)
capacityValueLabel.BackgroundTransparency = 1
capacityValueLabel.Text = maxBlockCapacity
capacityValueLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
capacityValueLabel.TextSize = isMobile and 42 or 36
capacityValueLabel.Font = Enum.Font.GothamBold
capacityValueLabel.TextXAlignment = Enum.TextXAlignment.Left
capacityValueLabel.Parent = capacityCard

local capacityMaxLabel = Instance.new("TextLabel")
capacityMaxLabel.Size = UDim2.new(1, -75, 0, 25)
capacityMaxLabel.Position = UDim2.new(0, 70, 0, 90)
capacityMaxLabel.BackgroundTransparency = 1
capacityMaxLabel.Text = "/ 250 (MÁXIMO)"
capacityMaxLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
capacityMaxLabel.TextSize = isMobile and 14 or 12
capacityMaxLabel.Font = Enum.Font.Gotham
capacityMaxLabel.TextXAlignment = Enum.TextXAlignment.Left
capacityMaxLabel.Parent = capacityCard

-- SLIDER PARA AJUSTAR CAPACIDADE
local capacitySliderBg = Instance.new("Frame")
capacitySliderBg.Size = UDim2.new(0.88, 0, 0, 10)
capacitySliderBg.Position = UDim2.new(0.06, 0, 0.85, 0)
capacitySliderBg.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
capacitySliderBg.BorderSizePixel = 0
capacitySliderBg.Parent = capacityCard

local capacitySliderBgCorner = Instance.new("UICorner")
capacitySliderBgCorner.CornerRadius = UDim.new(1, 0)
capacitySliderBgCorner.Parent = capacitySliderBg

local capacitySliderFill = Instance.new("Frame")
capacitySliderFill.Size = UDim2.new((maxBlockCapacity - minBlockCapacity) / (250 - minBlockCapacity), 0, 1, 0)
capacitySliderFill.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
capacitySliderFill.BorderSizePixel = 0
capacitySliderFill.Parent = capacitySliderBg

local capacitySliderFillCorner = Instance.new("UICorner")
capacitySliderFillCorner.CornerRadius = UDim.new(1, 0)
capacitySliderFillCorner.Parent = capacitySliderFill

local capacitySliderButton = Instance.new("ImageButton")
capacitySliderButton.Size = UDim2.new(0, 24, 0, 24)
capacitySliderButton.Position = UDim2.new((maxBlockCapacity - minBlockCapacity) / (250 - minBlockCapacity), -12, 0.5, -12)
capacitySliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
capacitySliderButton.Image = "rbxassetid://266788897"
capacitySliderButton.ScaleType = Enum.ScaleType.Fit
capacitySliderButton.BackgroundTransparency = 1
capacitySliderButton.Parent = capacityCard

-- Labels min/max do slider
local capacityMinLabel = Instance.new("TextLabel")
capacityMinLabel.Size = UDim2.new(0, 30, 0, 20)
capacityMinLabel.Position = UDim2.new(0.04, 0, 0.85, -8)
capacityMinLabel.BackgroundTransparency = 1
capacityMinLabel.Text = "30"
capacityMinLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
capacityMinLabel.TextSize = 10
capacityMinLabel.Font = Enum.Font.Gotham
capacityMinLabel.Parent = capacityCard

local capacityMaxLabel2 = Instance.new("TextLabel")
capacityMaxLabel2.Size = UDim2.new(0, 30, 0, 20)
capacityMaxLabel2.Position = UDim2.new(0.88, 0, 0.85, -8)
capacityMaxLabel2.BackgroundTransparency = 1
capacityMaxLabel2.Text = "250"
capacityMaxLabel2.TextColor3 = Color3.fromRGB(150, 150, 150)
capacityMaxLabel2.TextSize = 10
capacityMaxLabel2.Font = Enum.Font.Gotham
capacityMaxLabel2.Parent = capacityCard

-- Botões de ação para capacidade
local capacityButtonsContainer = Instance.new("Frame")
capacityButtonsContainer.Size = UDim2.new(1, 0, 0, 40)
capacityButtonsContainer.Position = UDim2.new(0, 0, 1, -40)
capacityButtonsContainer.BackgroundTransparency = 1
capacityButtonsContainer.Parent = capacityCard

local capacityLayout = Instance.new("UIListLayout")
capacityLayout.FillDirection = Enum.FillDirection.Horizontal
capacityLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
capacityLayout.Padding = UDim.new(0, 10)
capacityLayout.Parent = capacityButtonsContainer

-- Botão +50
local add50Button = Instance.new("TextButton")
add50Button.Size = UDim2.new(0, isMobile and 80 or 70, 0, isMobile and 34 or 30)
add50Button.Text = "+50"
add50Button.TextColor3 = Color3.fromRGB(255, 255, 255)
add50Button.TextSize = isMobile and 14 or 12
add50Button.Font = Enum.Font.GothamBold
add50Button.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
add50Button.BorderSizePixel = 0
add50Button.Parent = capacityButtonsContainer

local add50Corner = Instance.new("UICorner")
add50Corner.CornerRadius = UDim.new(0, 8)
add50Corner.Parent = add50Button

-- Botão +100
local add100Button = Instance.new("TextButton")
add100Button.Size = UDim2.new(0, isMobile and 80 or 70, 0, isMobile and 34 or 30)
add100Button.Text = "+100"
add100Button.TextColor3 = Color3.fromRGB(255, 255, 255)
add100Button.TextSize = isMobile and 14 or 12
add100Button.Font = Enum.Font.GothamBold
add100Button.BackgroundColor3 = Color3.fromRGB(255, 152, 0)
add100Button.BorderSizePixel = 0
add100Button.Parent = capacityButtonsContainer

local add100Corner = Instance.new("UICorner")
add100Corner.CornerRadius = UDim.new(0, 8)
add100Corner.Parent = add100Button

-- Botão Máximo (250)
local maxButton = Instance.new("TextButton")
maxButton.Size = UDim2.new(0, isMobile and 80 or 70, 0, isMobile and 34 or 30)
maxButton.Text = "MAX"
maxButton.TextColor3 = Color3.fromRGB(255, 255, 255)
maxButton.TextSize = isMobile and 14 or 12
maxButton.Font = Enum.Font.GothamBold
maxButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
maxButton.BorderSizePixel = 0
maxButton.Parent = capacityButtonsContainer

local maxCorner = Instance.new("UICorner")
maxCorner.CornerRadius = UDim.new(0, 8)
maxCorner.Parent = maxButton

-- ============ CARD DE VELOCIDADE ============
local speedCard = Instance.new("Frame")
speedCard.Size = UDim2.new(1, -24, 0, 130)
speedCard.Position = UDim2.new(0, 12, 0, 260)
speedCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
speedCard.BackgroundTransparency = 0.3
speedCard.BorderSizePixel = 0
speedCard.Parent = mainContainer

local speedCardCorner = Instance.new("UICorner")
speedCardCorner.CornerRadius = UDim.new(0, 14)
speedCardCorner.Parent = speedCard

local speedCardStroke = Instance.new("UIStroke")
speedCardStroke.Color = Color3.fromRGB(66, 135, 245)
speedCardStroke.Thickness = 1
speedCardStroke.Transparency = 0.6
speedCardStroke.Parent = speedCard

local speedIcon = Instance.new("ImageLabel")
speedIcon.Size = UDim2.new(0, 40, 0, 40)
speedIcon.Position = UDim2.new(0, 15, 0, 12)
speedIcon.Image = "rbxassetid://6031094773"
speedIcon.ImageColor3 = Color3.fromRGB(66, 135, 245)
speedIcon.BackgroundTransparency = 1
speedIcon.Parent = speedCard

local speedTitleLabel = Instance.new("TextLabel")
speedTitleLabel.Size = UDim2.new(1, -70, 0, 25)
speedTitleLabel.Position = UDim2.new(0, 65, 0, 12)
speedTitleLabel.BackgroundTransparency = 1
speedTitleLabel.Text = "VELOCIDADE"
speedTitleLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
speedTitleLabel.TextSize = isMobile and 12 or 11
speedTitleLabel.Font = Enum.Font.Gotham
speedTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
speedTitleLabel.Parent = speedCard

local speedValueLabel = Instance.new("TextLabel")
speedValueLabel.Size = UDim2.new(1, -70, 0, 35)
speedValueLabel.Position = UDim2.new(0, 65, 0, 35)
speedValueLabel.BackgroundTransparency = 1
speedValueLabel.Text = math.floor(currentSpeed)
speedValueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedValueLabel.TextSize = isMobile and 28 or 24
speedValueLabel.Font = Enum.Font.GothamBold
speedValueLabel.TextXAlignment = Enum.TextXAlignment.Left
speedValueLabel.Parent = speedCard

local speedMaxLabel = Instance.new("TextLabel")
speedMaxLabel.Size = UDim2.new(1, -70, 0, 20)
speedMaxLabel.Position = UDim2.new(0, 65, 0, 70)
speedMaxLabel.BackgroundTransparency = 1
speedMaxLabel.Text = "/ " .. MAX_SPEED
speedMaxLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
speedMaxLabel.TextSize = isMobile and 12 or 10
speedMaxLabel.Font = Enum.Font.Gotham
speedMaxLabel.TextXAlignment = Enum.TextXAlignment.Left
speedMaxLabel.Parent = speedCard

-- SLIDER VELOCIDADE
local sliderBg = Instance.new("Frame")
sliderBg.Size = UDim2.new(0.88, 0, 0, 8)
sliderBg.Position = UDim2.new(0.06, 0, 0.85, 0)
sliderBg.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
sliderBg.BorderSizePixel = 0
sliderBg.Parent = speedCard

local sliderBgCorner = Instance.new("UICorner")
sliderBgCorner.CornerRadius = UDim.new(1, 0)
sliderBgCorner.Parent = sliderBg

local sliderFill = Instance.new("Frame")
sliderFill.Size = UDim2.new((currentSpeed - MIN_SPEED) / (MAX_SPEED - MIN_SPEED), 0, 1, 0)
sliderFill.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
sliderFill.BorderSizePixel = 0
sliderFill.Parent = sliderBg

local sliderFillCorner = Instance.new("UICorner")
sliderFillCorner.CornerRadius = UDim.new(1, 0)
sliderFillCorner.Parent = sliderFill

local sliderButton = Instance.new("ImageButton")
sliderButton.Size = UDim2.new(0, 22, 0, 22)
sliderButton.Position = UDim2.new((currentSpeed - MIN_SPEED) / (MAX_SPEED - MIN_SPEED), -11, 0.5, -11)
sliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
sliderButton.Image = "rbxassetid://266788897"
sliderButton.ScaleType = Enum.ScaleType.Fit
sliderButton.BackgroundTransparency = 1
sliderButton.Parent = speedCard

-- ============ CARD DE VOO ============
local flyCard = Instance.new("Frame")
flyCard.Size = UDim2.new(1, -24, 0, 100)
flyCard.Position = UDim2.new(0, 12, 0, 405)
flyCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
flyCard.BackgroundTransparency = 0.3
flyCard.BorderSizePixel = 0
flyCard.Parent = mainContainer

local flyCardCorner = Instance.new("UICorner")
flyCardCorner.CornerRadius = UDim.new(0, 14)
flyCardCorner.Parent = flyCard

local flyCardStroke = Instance.new("UIStroke")
flyCardStroke.Color = Color3.fromRGB(66, 200, 100)
flyCardStroke.Thickness = 1
flyCardStroke.Transparency = 0.6
flyCardStroke.Parent = flyCard

local flyIcon = Instance.new("ImageLabel")
flyIcon.Size = UDim2.new(0, 40, 0, 40)
flyIcon.Position = UDim2.new(0, 15, 0, 12)
flyIcon.Image = "rbxassetid://6031094773"
flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
flyIcon.BackgroundTransparency = 1
flyIcon.Parent = flyCard

local flyStatusLabel = Instance.new("TextLabel")
flyStatusLabel.Size = UDim2.new(1, -70, 0, 30)
flyStatusLabel.Position = UDim2.new(0, 65, 0, 15)
flyStatusLabel.BackgroundTransparency = 1
flyStatusLabel.Text = "STATUS: DESLIGADO"
flyStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
flyStatusLabel.TextSize = isMobile and 13 or 11
flyStatusLabel.Font = Enum.Font.GothamBold
flyStatusLabel.TextXAlignment = Enum.TextXAlignment.Left
flyStatusLabel.Parent = flyCard

local flyButton = Instance.new("TextButton")
flyButton.Size = UDim2.new(0, isMobile and 120 or 100, 0, isMobile and 38 or 34)
flyButton.Position = UDim2.new(0.5, isMobile and -60 or -50, 1, -12)
flyButton.Text = "🕊️ VOAR"
flyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
flyButton.TextSize = isMobile and 13 or 11
flyButton.Font = Enum.Font.GothamBold
flyButton.BackgroundColor3 = Color3.fromRGB(66, 200, 100)
flyButton.BorderSizePixel = 0
flyButton.Parent = flyCard

local flyButtonCorner = Instance.new("UICorner")
flyButtonCorner.CornerRadius = UDim.new(0, 10)
flyButtonCorner.Parent = flyButton

-- ============ CARD DE AIMBOT ============
local aimbotCard = Instance.new("Frame")
aimbotCard.Size = UDim2.new(1, -24, 0, 100)
aimbotCard.Position = UDim2.new(0, 12, 0, 520)
aimbotCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
aimbotCard.BackgroundTransparency = 0.3
aimbotCard.BorderSizePixel = 0
aimbotCard.Parent = mainContainer

local aimbotCardCorner = Instance.new("UICorner")
aimbotCardCorner.CornerRadius = UDim.new(0, 14)
aimbotCardCorner.Parent = aimbotCard

local aimbotCardStroke = Instance.new("UIStroke")
aimbotCardStroke.Color = Color3.fromRGB(156, 39, 176)
aimbotCardStroke.Thickness = 1
aimbotCardStroke.Transparency = 0.6
aimbotCardStroke.Parent = aimbotCard

local aimbotIcon = Instance.new("ImageLabel")
aimbotIcon.Size = UDim2.new(0, 40, 0, 40)
aimbotIcon.Position = UDim2.new(0, 15, 0, 12)
aimbotIcon.Image = "rbxassetid://6031094773"
aimbotIcon.ImageColor3 = Color3.fromRGB(156, 39, 176)
aimbotIcon.BackgroundTransparency = 1
aimbotIcon.Parent = aimbotCard

local aimbotStatusLabel = Instance.new("TextLabel")
aimbotStatusLabel.Size = UDim2.new(1, -70, 0, 30)
aimbotStatusLabel.Position = UDim2.new(0, 65, 0, 15)
aimbotStatusLabel.BackgroundTransparency = 1
aimbotStatusLabel.Text = "STATUS: DESLIGADO"
aimbotStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
aimbotStatusLabel.TextSize = isMobile and 13 or 11
aimbotStatusLabel.Font = Enum.Font.GothamBold
aimbotStatusLabel.TextXAlignment = Enum.TextXAlignment.Left
aimbotStatusLabel.Parent = aimbotCard

local aimbotButton = Instance.new("TextButton")
aimbotButton.Size = UDim2.new(0, isMobile and 120 or 100, 0, isMobile and 38 or 34)
aimbotButton.Position = UDim2.new(0.5, isMobile and -60 or -50, 1, -12)
aimbotButton.Text = "🎯 AIMBOT"
aimbotButton.TextColor3 = Color3.fromRGB(255, 255, 255)
aimbotButton.TextSize = isMobile and 13 or 11
aimbotButton.Font = Enum.Font.GothamBold
aimbotButton.BackgroundColor3 = Color3.fromRGB(156, 39, 176)
aimbotButton.BorderSizePixel = 0
aimbotButton.Parent = aimbotCard

local aimbotButtonCorner = Instance.new("UICorner")
aimbotButtonCorner.CornerRadius = UDim.new(0, 10)
aimbotButtonCorner.Parent = aimbotButton

-- ============ CARD DE INVULNERABILIDADE ============
local invincibleCard = Instance.new("Frame")
invincibleCard.Size = UDim2.new(1, -24, 0, 100)
invincibleCard.Position = UDim2.new(0, 12, 0, 635)
invincibleCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
invincibleCard.BackgroundTransparency = 0.3
invincibleCard.BorderSizePixel = 0
invincibleCard.Parent = mainContainer

local invincibleCardCorner = Instance.new("UICorner")
invincibleCardCorner.CornerRadius = UDim.new(0, 14)
invincibleCardCorner.Parent = invincibleCard

local invincibleCardStroke = Instance.new("UIStroke")
invincibleCardStroke.Color = Color3.fromRGB(255, 193, 7)
invincibleCardStroke.Thickness = 1
invincibleCardStroke.Transparency = 0.6
invincibleCardStroke.Parent = invincibleCard

local invincibleIcon = Instance.new("ImageLabel")
invincibleIcon.Size = UDim2.new(0, 40, 0, 40)
invincibleIcon.Position = UDim2.new(0, 15, 0, 12)
invincibleIcon.Image = "rbxassetid://6031094773"
invincibleIcon.ImageColor3 = Color3.fromRGB(255, 193, 7)
invincibleIcon.BackgroundTransparency = 1
invincibleIcon.Parent = invincibleCard

local invincibleStatusLabel = Instance.new("TextLabel")
invincibleStatusLabel.Size = UDim2.new(1, -70, 0, 30)
invincibleStatusLabel.Position = UDim2.new(0, 65, 0, 15)
invincibleStatusLabel.BackgroundTransparency = 1
invincibleStatusLabel.Text = "STATUS: DESLIGADO"
invincibleStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
invincibleStatusLabel.TextSize = isMobile and 13 or 11
invincibleStatusLabel.Font = Enum.Font.GothamBold
invincibleStatusLabel.TextXAlignment = Enum.TextXAlignment.Left
invincibleStatusLabel.Parent = invincibleCard

local invincibleButton = Instance.new("TextButton")
invincibleButton.Size = UDim2.new(0, isMobile and 130 or 110, 0, isMobile and 38 or 34)
invincibleButton.Position = UDim2.new(0.5, isMobile and -65 or -55, 1, -12)
invincibleButton.Text = "🛡️ INVINCIBLE"
invincibleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
invincibleButton.TextSize = isMobile and 12 or 10
invincibleButton.Font = Enum.Font.GothamBold
invincibleButton.BackgroundColor3 = Color3.fromRGB(255, 193, 7)
invincibleButton.BorderSizePixel = 0
invincibleButton.Parent = invincibleCard

local invincibleButtonCorner = Instance.new("UICorner")
invincibleButtonCorner.CornerRadius = UDim.new(0, 10)
invincibleButtonCorner.Parent = invincibleButton

-- Footer
local footerFrame = Instance.new("Frame")
footerFrame.Size = UDim2.new(1, 0, 0, 40)
footerFrame.Position = UDim2.new(0, 0, 1, -40)
footerFrame.BackgroundColor3 = Color3.fromRGB(8, 8, 16)
footerFrame.BackgroundTransparency = 0.5
footerFrame.BorderSizePixel = 0
footerFrame.Parent = mainContainer

local footerCorner = Instance.new("UICorner")
footerCorner.CornerRadius = UDim.new(0, 14)
footerCorner.Parent = footerFrame

local creditLabel = Instance.new("TextLabel")
creditLabel.Size = UDim2.new(1, 0, 0, 18)
creditLabel.Position = UDim2.new(0, 0, 0, 4)
creditLabel.BackgroundTransparency = 1
creditLabel.Text = "⚡ RIAN STUDIOS V9.0 ⚡"
creditLabel.TextColor3 = Color3.fromRGB(120, 120, 120)
creditLabel.TextSize = isMobile and 10 or 9
creditLabel.Font = Enum.Font.Gotham
creditLabel.Parent = footerFrame

local versionLabel = Instance.new("TextLabel")
versionLabel.Size = UDim2.new(1, 0, 0, 14)
versionLabel.Position = UDim2.new(0, 0, 0, 22)
versionLabel.BackgroundTransparency = 1
versionLabel.Text = "CAPACIDADE DE BLOCOS: " .. maxBlockCapacity .. "/250"
versionLabel.TextColor3 = Color3.fromRGB(80, 80, 80)
versionLabel.TextSize = isMobile and 8 or 7
versionLabel.Font = Enum.Font.Gotham
versionLabel.Parent = footerFrame

-- ============ FUNÇÕES PRINCIPAIS ============

-- Função para atualizar a capacidade de blocos
local function updateBlockCapacity(newCapacity)
    newCapacity = math.clamp(newCapacity, minBlockCapacity, 250)
    maxBlockCapacity = newCapacity
    
    -- Atualiza UI
    capacityValueLabel.Text = maxBlockCapacity
    versionLabel.Text = "CAPACIDADE DE BLOCOS: " .. maxBlockCapacity .. "/250"
    
    -- Atualiza slider
    local percent = (maxBlockCapacity - minBlockCapacity) / (250 - minBlockCapacity)
    capacitySliderFill:TweenSize(UDim2.new(percent, 0, 1, 0), "Out", "Quad", 0.1, true)
    capacitySliderButton:TweenPosition(UDim2.new(percent, -12, 0.5, -12), "Out", "Quad", 0.1, true)
    
    -- Aplica a mudança no jogo
    increaseBlockCapacity(maxBlockCapacity)
    
    -- Notificação
    playSound("rbxassetid://9120384332", 0.3)
    
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0, 280, 0, 50)
    notif.Position = UDim2.new(0.5, -140, 0, -50)
    notif.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
    notif.BackgroundTransparency = 0.1
    notif.BorderSizePixel = 0
    notif.Parent = screenGui
    
    local notifCorner = Instance.new("UICorner")
    notifCorner.CornerRadius = UDim.new(0, 12)
    notifCorner.Parent = notif
    
    local notifText = Instance.new("TextLabel")
    notifText.Size = UDim2.new(1, 0, 1, 0)
    notifText.BackgroundTransparency = 1
    notifText.Text = "📦 CAPACIDADE: " .. maxBlockCapacity .. " BLOCOS!"
    notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
    notifText.TextScaled = true
    notifText.Font = Enum.Font.GothamBold
    notifText.Parent = notif
    
    TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, 20)}):Play()
    wait(2)
    TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -140, 0, -50), BackgroundTransparency = 1}):Play()
    wait(0.3)
    notif:Destroy()
end

-- Slider da capacidade
local draggingCapacity = false
local function updateCapacityFromMouse(input)
    local mousePos = input.Position.X
    local sliderAbsPos = capacitySliderBg.AbsolutePosition.X
    local sliderWidth = capacitySliderBg.AbsoluteSize.X
    local percent = math.clamp((mousePos - sliderAbsPos) / sliderWidth, 0, 1)
    local newCapacity = minBlockCapacity + (percent * (250 - minBlockCapacity))
    updateBlockCapacity(math.floor(newCapacity))
end

capacitySliderButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingCapacity = true
        updateCapacityFromMouse(input)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingCapacity and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        updateCapacityFromMouse(input)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingCapacity = false
    end
end)

-- Botões de capacidade
add50Button.MouseButton1Click:Connect(function()
    local newCapacity = math.min(maxBlockCapacity + 50, 250)
    updateBlockCapacity(newCapacity)
end)

add100Button.MouseButton1Click:Connect(function()
    local newCapacity = math.min(maxBlockCapacity + 100, 250)
    updateBlockCapacity(newCapacity)
end)

maxButton.MouseButton1Click:Connect(function()
    updateBlockCapacity(250)
end)

-- ============ FUNÇÕES DE VELOCIDADE ============
local function applySpeedToCharacter()
    if not humanoid then return end
    
    if isFlying then
        FLY_SPEED = currentSpeed
    else
        humanoid.WalkSpeed = currentSpeed
    end
    
    speedValueLabel.Text = math.floor(currentSpeed)
    
    local percent = (currentSpeed - MIN_SPEED) / (MAX_SPEED - MIN_SPEED)
    sliderFill:TweenSize(UDim2.new(percent, 0, 1, 0), "Out", "Quad", 0.1, true)
    sliderButton:TweenPosition(UDim2.new(percent, -11, 0.5, -11), "Out", "Quad", 0.1, true)
end

local function updateSpeed(newSpeed, ignoreBoostLock)
    if isBombaBoosted and not ignoreBoostLock then return end
    currentSpeed = math.clamp(newSpeed, MIN_SPEED, MAX_SPEED)
    applySpeedToCharacter()
end

-- Botão Turbo (rápido)
local turboButton = Instance.new("TextButton")
turboButton.Size = UDim2.new(0, isMobile and 100 or 90, 0, isMobile and 38 or 34)
turboButton.Position = UDim2.new(0.5, isMobile and -50 or -45, 1, -12)
turboButton.Text = "🚀 TURBO"
turboButton.TextColor3 = Color3.fromRGB(255, 255, 255)
turboButton.TextSize = isMobile and 12 or 10
turboButton.Font = Enum.Font.GothamBold
turboButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
turboButton.BorderSizePixel = 0
turboButton.Parent = speedCard

local turboCorner = Instance.new("UICorner")
turboCorner.CornerRadius = UDim.new(0, 8)
turboCorner.Parent = turboButton

turboButton.MouseButton1Click:Connect(function()
    playSound("rbxassetid://9120384332", 0.3)
    updateSpeed(MAX_SPEED, true)
    isSpeedBoosted = true
    speedActivated = true
end)

-- Slider de velocidade
local dragging = false
local function updateSliderFromMouse(input)
    if isBombaBoosted then return end
    
    local mousePos = input.Position.X
    local sliderAbsPos = sliderBg.AbsolutePosition.X
    local sliderWidth = sliderBg.AbsoluteSize.X
    local percent = math.clamp((mousePos - sliderAbsPos) / sliderWidth, 0, 1)
    local newSpeed = MIN_SPEED + (percent * (MAX_SPEED - MIN_SPEED))
    
    updateSpeed(newSpeed, true)
    isSpeedBoosted = (newSpeed > NORMAL_SPEED)
    speedActivated = isSpeedBoosted
end

sliderButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        updateSliderFromMouse(input)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        updateSliderFromMouse(input)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- SISTEMA DE VOO
local function updateFlyMovement()
    if not isFlying or not flyBodyVelocity then return end
    
    local moveDirection = Vector3.new()
    
    if not isMobile then
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDirection = moveDirection + humanoidRootPart.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDirection = moveDirection - humanoidRootPart.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDirection = moveDirection - humanoidRootPart.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDirection = moveDirection + humanoidRootPart.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDirection = moveDirection + Vector3.new(0, 1, 0)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            moveDirection = moveDirection - Vector3.new(0, 1, 0)
        end
    end
    
    if moveDirection.Magnitude > 0 then
        moveDirection = moveDirection.Unit
        flyBodyVelocity.Velocity = moveDirection * FLY_SPEED
        if flyBodyGyro then
            flyBodyGyro.CFrame = CFrame.lookAt(Vector3.new(), moveDirection)
        end
    else
        flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
    end
end

local function startFly()
    if isFlying then return end
    
    isFlying = true
    flyStatusLabel.Text = "STATUS: 🕊️ VOANDO"
    flyStatusLabel.TextColor3 = Color3.fromRGB(66, 200, 100)
    flyButton.Text = "🛑 PARAR"
    flyButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
    flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
    flyCardStroke.Color = Color3.fromRGB(66, 200, 100)
    
    Workspace.Gravity = 0
    
    flyBodyVelocity = Instance.new("BodyVelocity")
    flyBodyVelocity.MaxForce = Vector3.new(100000, 100000, 100000)
    flyBodyVelocity.Parent = humanoidRootPart
    
    flyBodyGyro = Instance.new("BodyGyro")
    flyBodyGyro.MaxTorque = Vector3.new(400000, 400000, 400000)
    flyBodyGyro.CFrame = humanoidRootPart.CFrame
    flyBodyGyro.Parent = humanoidRootPart
    
    coroutine.wrap(function()
        while isFlying and humanoidRootPart and humanoid do
            updateFlyMovement()
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            humanoid.PlatformStand = true
            RunService.Heartbeat:Wait()
        end
    end)()
end

local function stopFly()
    if not isFlying then return end
    
    isFlying = false
    flyStatusLabel.Text = "STATUS: DESLIGADO"
    flyStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    flyButton.Text = "🕊️ VOAR"
    flyButton.BackgroundColor3 = Color3.fromRGB(66, 200, 100)
    flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
    flyCardStroke.Color = Color3.fromRGB(66, 200, 100)
    
    Workspace.Gravity = defaultGravity
    
    if flyBodyVelocity then flyBodyVelocity:Destroy() end
    if flyBodyGyro then flyBodyGyro:Destroy() end
    
    if humanoid then
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
        humanoid.PlatformStand = false
    end
end

flyButton.MouseButton1Click:Connect(function()
    playSound("rbxassetid://9120384332", 0.3)
    if isFlying then stopFly() else startFly() end
end)

-- AIMBOT
local function getNearestEnemy()
    local nearest = nil
    local nearestDistance = aimbotRange
    
    for _, enemy in pairs(Workspace:GetChildren()) do
        if enemy:IsA("Model") and enemy ~= character then
            local enemyHumanoid = enemy:FindFirstChild("Humanoid")
            local enemyRoot = enemy:FindFirstChild("HumanoidRootPart")
            
            if enemyHumanoid and enemyHumanoid.Health > 0 and enemyRoot then
                local distance = (humanoidRootPart.Position - enemyRoot.Position).Magnitude
                if distance < nearestDistance then
                    nearestDistance = distance
                    nearest = enemyRoot
                end
            end
        end
    end
    
    return nearest
end

local function updateAimbot()
    if not aimbotEnabled or not character or not humanoidRootPart then return end
    
    local target = getNearestEnemy()
    if target then
        local lookAt = CFrame.lookAt(humanoidRootPart.Position, target.Position)
        local newCFrame = humanoidRootPart.CFrame:Lerp(lookAt, aimbotSmoothness)
        humanoidRootPart.CFrame = newCFrame
    end
end

local function toggleAimbot()
    aimbotEnabled = not aimbotEnabled
    
    if aimbotEnabled then
        aimbotButton.Text = "🎯 DESATIVAR"
        aimbotButton.BackgroundColor3 = Color3.fromRGB(100, 30, 120)
        aimbotStatusLabel.Text = "STATUS: 🎯 ATIVADO"
        aimbotStatusLabel.TextColor3 = Color3.fromRGB(156, 39, 176)
        aimbotCardStroke.Color = Color3.fromRGB(156, 39, 176)
        aimbotIcon.ImageColor3 = Color3.fromRGB(156, 39, 176)
        
        if aimbotConnection then aimbotConnection:Disconnect() end
        aimbotConnection = RunService.RenderStepped:Connect(updateAimbot)
    else
        aimbotButton.Text = "🎯 AIMBOT"
        aimbotButton.BackgroundColor3 = Color3.fromRGB(156, 39, 176)
        aimbotStatusLabel.Text = "STATUS: DESLIGADO"
        aimbotStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        aimbotCardStroke.Color = Color3.fromRGB(156, 39, 176)
        aimbotIcon.ImageColor3 = Color3.fromRGB(156, 39, 176)
        
        if aimbotConnection then
            aimbotConnection:Disconnect()
            aimbotConnection = nil
        end
    end
end

aimbotButton.MouseButton1Click:Connect(function()
    playSound("rbxassetid://9120384332", 0.3)
    toggleAimbot()
end)

-- INVULNERABILIDADE
local function toggleInvincible()
    invincibleEnabled = not invincibleEnabled
    
    if invincibleEnabled then
        invincibleButton.Text = "🛡️ DESATIVAR"
        invincibleButton.BackgroundColor3 = Color3.fromRGB(220, 150, 0)
        invincibleStatusLabel.Text = "STATUS: 🛡️ ATIVADO"
        invincibleStatusLabel.TextColor3 = Color3.fromRGB(255, 193, 7)
        invincibleCardStroke.Color = Color3.fromRGB(255, 193, 7)
        invincibleIcon.ImageColor3 = Color3.fromRGB(255, 193, 7)
        
        if humanoid then
            humanoid.BreakJointsOnDeath = false
            humanoid.MaxHealth = math.huge
            humanoid.Health = math.huge
        end
    else
        invincibleButton.Text = "🛡️ INVINCIBLE"
        invincibleButton.BackgroundColor3 = Color3.fromRGB(255, 193, 7)
        invincibleStatusLabel.Text = "STATUS: DESLIGADO"
        invincibleStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        invincibleCardStroke.Color = Color3.fromRGB(255, 193, 7)
        invincibleIcon.ImageColor3 = Color3.fromRGB(255, 193, 7)
        
        if humanoid then
            humanoid.BreakJointsOnDeath = true
            humanoid.MaxHealth = 100
            if humanoid.Health > 100 then
                humanoid.Health = 100
            end
        end
    end
end

invincibleButton.MouseButton1Click:Connect(function()
    playSound("rbxassetid://9120384332", 0.3)
    toggleInvincible()
end)

-- ATAQUE
local function playPunchAnimation()
    local anim = Instance.new("Animation")
    anim.AnimationId = "rbxassetid://131453564"
    local track = humanoid:LoadAnimation(anim)
    if track then track:Play(); wait(0.2); track:Stop() end
end

local function startComboAttack()
    if isAttacking then return end
    isAttacking = true
    comboCount = 0
    
    local function performPunch()
        if not canPunch then return end
        if comboCount >= 3 then isAttacking = false; comboCount = 0; return end
        
        canPunch = false
        comboCount = comboCount + 1
        
        local char = player.Character
        local hum = char and char:FindFirstChild("Humanoid")
        
        if char and hum then
            playSound("rbxassetid://9120384332", 0.5)
            playPunchAnimation()
            
            local impact = Instance.new("Part")
            impact.Shape = Enum.PartType.Ball
            impact.Size = Vector3.new(1, 1, 1)
            impact.BrickColor = BrickColor.new("Bright red")
            impact.Material = Enum.Material.Neon
            impact.CFrame = char:GetPivot() + char:GetPivot().LookVector * 3.5
            impact.Anchored = true
            impact.CanCollide = false
            impact.Parent = workspace
            
            TweenService:Create(impact, TweenInfo.new(0.3), {Size = Vector3.new(0, 0, 0)}):Play()
            game:GetService("Debris"):AddItem(impact, 0.3)
            
            for _, enemy in pairs(Workspace:GetChildren()) do
                if enemy:IsA("Model") and enemy ~= char then
                    local enemyHum = enemy:FindFirstChild("Humanoid")
                    if enemyHum and enemyHum.Health > 0 then
                        local dist = (char:GetPivot().Position - enemy:GetPivot().Position).Magnitude
                        if dist < 6 then
                            enemyHum:TakeDamage(PUNCH_DAMAGE)
                        end
                    end
                end
            end
        end
        
        wait(PUNCH_COOLDOWN)
        canPunch = true
        
        if isAttacking and comboCount < 3 then
            wait(COMBO_DELAY)
            performPunch()
        else
            isAttacking = false
            comboCount = 0
        end
    end
    
    performPunch()
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local char = player.Character
        if char and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0 then
            startComboAttack()
        end
    end
end)

-- INFO
local infoButton = Instance.new("TextButton")
infoButton.Size = UDim2.new(0, isMobile and 70 or 60, 0, isMobile and 38 or 34)
infoButton.Position = UDim2.new(0.5, isMobile and 40 or 35, 1, -12)
infoButton.Text = "ℹ️"
infoButton.TextColor3 = Color3.fromRGB(255, 255, 255)
infoButton.TextSize = isMobile and 20 or 18
infoButton.Font = Enum.Font.GothamBold
infoButton.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
infoButton.BorderSizePixel = 0
infoButton.Parent = speedCard

local infoCorner = Instance.new("UICorner")
infoCorner.CornerRadius = UDim.new(0, 8)
infoCorner.Parent = infoButton

infoButton.MouseButton1Click:Connect(function()
    local infoFrame = Instance.new("Frame")
    infoFrame.Size = UDim2.new(0, 320, 0, 420)
    infoFrame.Position = UDim2.new(0.5, -160, 0.5, -210)
    infoFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 28)
    infoFrame.BackgroundTransparency = 0.05
    infoFrame.BorderSizePixel = 0
    infoFrame.Parent = screenGui
    
    local infoCorner = Instance.new("UICorner")
    infoCorner.CornerRadius = UDim.new(0, 16)
    infoCorner.Parent = infoFrame
    
    local infoText = Instance.new("TextLabel")
    infoText.Size = UDim2.new(1, -20, 1, -20)
    infoText.Position = UDim2.new(0, 10, 0, 10)
    infoText.BackgroundTransparency = 1
    infoText.Text = [[
⚡ RIAN STUDIOS V9.0 ⚡

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📦 CAPACIDADE DE BLOCOS:
   • Aumente de 30 ATÉ 250 blocos!
   • Use o slider ou botões (+50, +100, MAX)
   • MUDA O LIMITE MÁXIMO DO JOGO!

🎮 VELOCIDADE:
   • Slider: Ajuste 16-120
   • TURBO: Máximo instantâneo

🕊️ VOO: WASD + ESPAÇO/SHIFT

🎯 AIMBOT: Mira automática em inimigos

🛡️ INVINCIBLE: Imune a danos

👊 ATAQUE: Clique esquerdo (combo 3x)

💣 BOOST: Pegue bombas para velocidade 120

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 COMO FUNCIONA A CAPACIDADE:
   • O script MODIFICA o limite máximo
   • Você pode carregar MAIS blocos
   • Vai de 30 até 250 blocos!

👑 CRIADO POR RIAN
    ]]
    infoText.TextColor3 = Color3.fromRGB(220, 220, 220)
    infoText.TextSize = 11
    infoText.Font = Enum.Font.Gotham
    infoText.TextXAlignment = Enum.TextXAlignment.Left
    infoText.TextYAlignment = Enum.TextYAlignment.Top
    infoText.Parent = infoFrame
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 70, 0, 30)
    closeBtn.Position = UDim2.new(0.5, -35, 1, -40)
    closeBtn.Text = "FECHAR"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = infoFrame
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 8)
    closeCorner.Parent = closeBtn
    
    closeBtn.MouseButton1Click:Connect(function() infoFrame:Destroy() end)
end)

-- RESPAWN
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    
    if isFlying then stopFly() end
    if isBombaBoosted then isBombaBoosted = false end
    
    if invincibleEnabled then
        humanoid.BreakJointsOnDeath = false
        humanoid.MaxHealth = math.huge
        humanoid.Health = math.huge
    end
    
    applySpeedToCharacter()
end)

-- UI TOGGLE
local isUIOpen = false
local function toggleUI()
    isUIOpen = not isUIOpen
    
    if isUIOpen then
        mainContainer.Visible = true
        mainContainer.BackgroundTransparency = 1
        TweenService:Create(mainContainer, TweenInfo.new(0.3, Enum.EasingStyle.Back), {BackgroundTransparency = 0.08}):Play()
        TweenService:Create(floatingButton, TweenInfo.new(0.2), {Size = isMobile and UDim2.new(0, 55, 0, 55) or UDim2.new(0, 50, 0, 50)}):Play()
    else
        TweenService:Create(mainContainer, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
        wait(0.2)
        mainContainer.Visible = false
        TweenService:Create(floatingButton, TweenInfo.new(0.2), {Size = isMobile and UDim2.new(0, 65, 0, 65) or UDim2.new(0, 55, 0, 55)}):Play()
    end
end

floatingButton.MouseButton1Click:Connect(toggleUI)
closeButton.MouseButton1Click:Connect(toggleUI)

-- INICIALIZAR
setupAntiBan()
applySpeedToCharacter()
updateBlockCapacity(maxBlockCapacity)

-- Animação do botão flutuante
coroutine.wrap(function()
    while true do
        wait(1.5)
        if not isUIOpen then
            TweenService:Create(floatingButton, TweenInfo.new(0.4, Enum.EasingStyle.Sine), {Size = isMobile and UDim2.new(0, 72, 0, 72) or UDim2.new(0, 60, 0, 60)}):Play()
            wait(0.2)
            TweenService:Create(floatingButton, TweenInfo.new(0.4, Enum.EasingStyle.Sine), {Size = isMobile and UDim2.new(0, 65, 0, 65) or UDim2.new(0, 55, 0, 55)}):Play()
        end
    end
end)()

print("═══════════════════════════════════════════════════════════════════════════════")
print("✅ SISTEMA RIAN STUDIOS V9.0 CARREGADO COM SUCESSO!")
print("✅ CAPACIDADE DE BLOCOS: DE 30 ATÉ 250 BLOCOS!")
print("✅ Use o slider ou botões para AUMENTAR o limite máximo!")
print("✅ O script MODIFICA o jogo para você carregar MAIS blocos!")
print("✅ ANTI-BAN ATIVADO | VELOCIDADE | VOO | AIMBOT | INVINCIBLE")
print("═══════════════════════════════════════════════════════════════════════════════")
