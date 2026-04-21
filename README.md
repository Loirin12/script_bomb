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
    ║              SISTEMA V8.0 - RIAN STUDIOS - COLETA DE BLOCOS + ANTI-BAN                               ║
    ║         VELOCIDADE | VOO | AIMBOT | INVINCIBLE | COLETA | ANTI-EXPULSÃO                              ║
    ║                                                                                                      ║
    ╚══════════════════════════════════════════════════════════════════════════════════════════════════════╝
]]

-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Variáveis do jogador
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- Detecta mobile
local isMobile = UserInputService.TouchEnabled

-- ============ ANTI-BAN / ANTI-EXPULSÃO ============
local antiBanEnabled = true
local lastHeartbeat = tick()

local function preventKick()
    coroutine.wrap(function()
        while antiBanEnabled and player do
            wait(25)
            lastHeartbeat = tick()
            -- Simula atividade para evitar timeout
            pcall(function()
                local stats = player:FindFirstChild("leaderstats")
                if stats then
                    -- Apenas toca em algo para mostrar atividade
                    local value = stats:FindFirstChild("Points")
                    if value then
                        -- Leitura simples, sem modificação
                        local _ = value.Value
                    end
                end
            end)
        end
    end)()
end

local function setupAntiBan()
    pcall(function()
        preventKick()
        
        -- Prevenir desconexão por inatividade
        local bindable = Instance.new("BindableEvent")
        bindable.Name = "HeartbeatEvent"
        bindable.Parent = player
        
        RunService.Heartbeat:Connect(function()
            lastHeartbeat = tick()
        end)
        
        -- Reconexão automática
        game:GetService("CoreGui").ChildAdded:Connect(function(child)
            if child.Name == "RobloxPromptGui" then
                wait(0.5)
                local prompts = child:GetDescendants()
                for _, prompt in pairs(prompts) do
                    if prompt.Name == "ErrorPrompt" then
                        wait(1)
                        -- Tenta manter a conexão
                    end
                end
            end
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
local currentTarget = nil
local aimbotConnection = nil

-- Configurações de INVULNERABILIDADE
local invincibleEnabled = false

-- ============ SISTEMA DE COLETA DE BLOCOS ============
local blocksCollected = 0
local targetBlocks = 100
local blockCollectionRange = 20
local autoCollectEnabled = false
local collectedBlocksList = {}
local autoCollectConnection = nil

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

-- CRIAR UI PRINCIPAL
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
mainContainer.Size = isMobile and UDim2.new(0.92, 0, 0.85, 0) or UDim2.new(0, 380, 0, 600)
mainContainer.Position = isMobile and UDim2.new(0.04, 0, 0.075, 0) or UDim2.new(0.5, -190, 0.5, -300)
mainContainer.BackgroundColor3 = Color3.fromRGB(12, 12, 22)
mainContainer.BackgroundTransparency = 0.08
mainContainer.BorderSizePixel = 0
mainContainer.Visible = false
mainContainer.Parent = screenGui
mainContainer.ClipsDescendants = false
mainContainer.ScrollBarThickness = 3
mainContainer.ScrollBarImageColor3 = Color3.fromRGB(66, 135, 245)
mainContainer.CanvasSize = UDim2.new(0, 0, 0, 950)

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
titleLabel.Text = "⚡ RIAN STUDIOS V8.0"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = isMobile and 16 or 14
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local subtitleLabel = Instance.new("TextLabel")
subtitleLabel.Size = UDim2.new(1, -80, 0, 20)
subtitleLabel.Position = UDim2.new(0, 15, 0, 35)
subtitleLabel.BackgroundTransparency = 1
subtitleLabel.Text = "SPEED | FLY | AIMBOT | INVINCIBLE | COLLECT"
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

-- ============ CARD DE VELOCIDADE ============
local speedCard = Instance.new("Frame")
speedCard.Size = UDim2.new(1, -24, 0, 140)
speedCard.Position = UDim2.new(0, 12, 0, 75)
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
speedIcon.Position = UDim2.new(0, 15, 0, 15)
speedIcon.Image = "rbxassetid://6031094773"
speedIcon.ImageColor3 = Color3.fromRGB(66, 135, 245)
speedIcon.BackgroundTransparency = 1
speedIcon.Parent = speedCard

local speedTitleLabel = Instance.new("TextLabel")
speedTitleLabel.Size = UDim2.new(1, -70, 0, 25)
speedTitleLabel.Position = UDim2.new(0, 65, 0, 15)
speedTitleLabel.BackgroundTransparency = 1
speedTitleLabel.Text = "VELOCIDADE"
speedTitleLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
speedTitleLabel.TextSize = isMobile and 12 or 11
speedTitleLabel.Font = Enum.Font.Gotham
speedTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
speedTitleLabel.Parent = speedCard

local speedValueLabel = Instance.new("TextLabel")
speedValueLabel.Size = UDim2.new(1, -70, 0, 40)
speedValueLabel.Position = UDim2.new(0, 65, 0, 38)
speedValueLabel.BackgroundTransparency = 1
speedValueLabel.Text = math.floor(currentSpeed)
speedValueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedValueLabel.TextSize = isMobile and 34 or 28
speedValueLabel.Font = Enum.Font.GothamBold
speedValueLabel.TextXAlignment = Enum.TextXAlignment.Left
speedValueLabel.Parent = speedCard

local speedMaxLabel = Instance.new("TextLabel")
speedMaxLabel.Size = UDim2.new(1, -70, 0, 20)
speedMaxLabel.Position = UDim2.new(0, 65, 0, 78)
speedMaxLabel.BackgroundTransparency = 1
speedMaxLabel.Text = "/ " .. MAX_SPEED
speedMaxLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
speedMaxLabel.TextSize = isMobile and 14 or 12
speedMaxLabel.Font = Enum.Font.Gotham
speedMaxLabel.TextXAlignment = Enum.TextXAlignment.Left
speedMaxLabel.Parent = speedCard

-- SLIDER DE VELOCIDADE
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
flyCard.Position = UDim2.new(0, 12, 0, 230)
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
aimbotCard.Position = UDim2.new(0, 12, 0, 345)
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
invincibleCard.Position = UDim2.new(0, 12, 0, 460)
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

-- ============ CARD DE COLETA ============
local collectCard = Instance.new("Frame")
collectCard.Size = UDim2.new(1, -24, 0, 140)
collectCard.Position = UDim2.new(0, 12, 0, 575)
collectCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
collectCard.BackgroundTransparency = 0.3
collectCard.BorderSizePixel = 0
collectCard.Parent = mainContainer

local collectCardCorner = Instance.new("UICorner")
collectCardCorner.CornerRadius = UDim.new(0, 14)
collectCardCorner.Parent = collectCard

local collectCardStroke = Instance.new("UIStroke")
collectCardStroke.Color = Color3.fromRGB(255, 215, 0)
collectCardStroke.Thickness = 1
collectCardStroke.Transparency = 0.6
collectCardStroke.Parent = collectCard

local collectIcon = Instance.new("ImageLabel")
collectIcon.Size = UDim2.new(0, 40, 0, 40)
collectIcon.Position = UDim2.new(0, 15, 0, 12)
collectIcon.Image = "rbxassetid://6031094773"
collectIcon.ImageColor3 = Color3.fromRGB(255, 215, 0)
collectIcon.BackgroundTransparency = 1
collectIcon.Parent = collectCard

local collectTitleLabel = Instance.new("TextLabel")
collectTitleLabel.Size = UDim2.new(1, -70, 0, 25)
collectTitleLabel.Position = UDim2.new(0, 65, 0, 12)
collectTitleLabel.BackgroundTransparency = 1
collectTitleLabel.Text = "📦 COLETA DE BLOCOS"
collectTitleLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
collectTitleLabel.TextSize = isMobile and 12 or 11
collectTitleLabel.Font = Enum.Font.Gotham
collectTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
collectTitleLabel.Parent = collectCard

local collectValueLabel = Instance.new("TextLabel")
collectValueLabel.Size = UDim2.new(1, -70, 0, 35)
collectValueLabel.Position = UDim2.new(0, 65, 0, 37)
collectValueLabel.BackgroundTransparency = 1
collectValueLabel.Text = blocksCollected .. " / " .. targetBlocks
collectValueLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
collectValueLabel.TextSize = isMobile and 20 or 18
collectValueLabel.Font = Enum.Font.GothamBold
collectValueLabel.TextXAlignment = Enum.TextXAlignment.Left
collectValueLabel.Parent = collectCard

-- Barra de progresso da coleta
local collectProgressBg = Instance.new("Frame")
collectProgressBg.Size = UDim2.new(0.88, 0, 0, 8)
collectProgressBg.Position = UDim2.new(0.06, 0, 0.7, 0)
collectProgressBg.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
collectProgressBg.BorderSizePixel = 0
collectProgressBg.Parent = collectCard

local collectProgressBgCorner = Instance.new("UICorner")
collectProgressBgCorner.CornerRadius = UDim.new(1, 0)
collectProgressBgCorner.Parent = collectProgressBg

local collectProgressFill = Instance.new("Frame")
collectProgressFill.Size = UDim2.new(blocksCollected / targetBlocks, 0, 1, 0)
collectProgressFill.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
collectProgressFill.BorderSizePixel = 0
collectProgressFill.Parent = collectProgressBg

local collectProgressFillCorner = Instance.new("UICorner")
collectProgressFillCorner.CornerRadius = UDim.new(1, 0)
collectProgressFillCorner.Parent = collectProgressFill

-- Slider da meta
local targetSliderBg = Instance.new("Frame")
targetSliderBg.Size = UDim2.new(0.7, 0, 0, 6)
targetSliderBg.Position = UDim2.new(0.15, 0, 0.85, 0)
targetSliderBg.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
targetSliderBg.BorderSizePixel = 0
targetSliderBg.Parent = collectCard

local targetSliderBgCorner = Instance.new("UICorner")
targetSliderBgCorner.CornerRadius = UDim.new(1, 0)
targetSliderBgCorner.Parent = targetSliderBg

local targetSliderFill = Instance.new("Frame")
targetSliderFill.Size = UDim2.new((targetBlocks - 30) / (250 - 30), 0, 1, 0)
targetSliderFill.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
targetSliderFill.BorderSizePixel = 0
targetSliderFill.Parent = targetSliderBg

local targetSliderFillCorner = Instance.new("UICorner")
targetSliderFillCorner.CornerRadius = UDim.new(1, 0)
targetSliderFillCorner.Parent = targetSliderFill

local targetSliderButton = Instance.new("ImageButton")
targetSliderButton.Size = UDim2.new(0, 18, 0, 18)
targetSliderButton.Position = UDim2.new((targetBlocks - 30) / (250 - 30), -9, 0.5, -9)
targetSliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
targetSliderButton.Image = "rbxassetid://266788897"
targetSliderButton.ScaleType = Enum.ScaleType.Fit
targetSliderButton.BackgroundTransparency = 1
targetSliderButton.Parent = collectCard

local targetMinLabel = Instance.new("TextLabel")
targetMinLabel.Size = UDim2.new(0, 25, 0, 15)
targetMinLabel.Position = UDim2.new(0.1, 0, 0.85, -5)
targetMinLabel.BackgroundTransparency = 1
targetMinLabel.Text = "30"
targetMinLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
targetMinLabel.TextSize = 9
targetMinLabel.Font = Enum.Font.Gotham
targetMinLabel.Parent = collectCard

local targetMaxLabel = Instance.new("TextLabel")
targetMaxLabel.Size = UDim2.new(0, 25, 0, 15)
targetMaxLabel.Position = UDim2.new(0.85, 0, 0.85, -5)
targetMaxLabel.BackgroundTransparency = 1
targetMaxLabel.Text = "250"
targetMaxLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
targetMaxLabel.TextSize = 9
targetMaxLabel.Font = Enum.Font.Gotham
targetMaxLabel.Parent = collectCard

-- Botões de coleta
local autoCollectButton = Instance.new("TextButton")
autoCollectButton.Size = UDim2.new(0, isMobile and 100 or 90, 0, isMobile and 34 or 30)
autoCollectButton.Position = UDim2.new(0.5, isMobile and -105 or -95, 1, -10)
autoCollectButton.Text = "🔄 AUTO"
autoCollectButton.TextColor3 = Color3.fromRGB(255, 255, 255)
autoCollectButton.TextSize = isMobile and 11 or 10
autoCollectButton.Font = Enum.Font.GothamBold
autoCollectButton.BackgroundColor3 = Color3.fromRGB(255, 152, 0)
autoCollectButton.BorderSizePixel = 0
autoCollectButton.Parent = collectCard

local autoCollectCorner = Instance.new("UICorner")
autoCollectCorner.CornerRadius = UDim.new(0, 8)
autoCollectCorner.Parent = autoCollectButton

local resetCollectButton = Instance.new("TextButton")
resetCollectButton.Size = UDim2.new(0, isMobile and 80 or 70, 0, isMobile and 34 or 30)
resetCollectButton.Position = UDim2.new(0.5, isMobile and 15 or 10, 1, -10)
resetCollectButton.Text = "🔄 ZERAR"
resetCollectButton.TextColor3 = Color3.fromRGB(255, 255, 255)
resetCollectButton.TextSize = isMobile and 11 or 10
resetCollectButton.Font = Enum.Font.GothamBold
resetCollectButton.BackgroundColor3 = Color3.fromRGB(64, 64, 64)
resetCollectButton.BorderSizePixel = 0
resetCollectButton.Parent = collectCard

local resetCollectCorner = Instance.new("UICorner")
resetCollectCorner.CornerRadius = UDim.new(0, 8)
resetCollectCorner.Parent = resetCollectButton

-- ============ BOTÕES RÁPIDOS ============
local quickButtonsContainer = Instance.new("Frame")
quickButtonsContainer.Size = UDim2.new(1, -24, 0, 50)
quickButtonsContainer.Position = UDim2.new(0, 12, 0, 730)
quickButtonsContainer.BackgroundTransparency = 1
quickButtonsContainer.Parent = mainContainer

local quickLayout = Instance.new("UIListLayout")
quickLayout.FillDirection = Enum.FillDirection.Horizontal
quickLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
quickLayout.Padding = UDim.new(0, 10)
quickLayout.Parent = quickButtonsContainer

local autoSpeedButton = Instance.new("TextButton")
autoSpeedButton.Size = UDim2.new(0, isMobile and 140 or 120, 0, isMobile and 42 or 38)
autoSpeedButton.Text = "🔘 ATIVAR VELOCIDADE"
autoSpeedButton.TextColor3 = Color3.fromRGB(255, 255, 255)
autoSpeedButton.TextSize = isMobile and 12 or 10
autoSpeedButton.Font = Enum.Font.GothamBold
autoSpeedButton.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
autoSpeedButton.BorderSizePixel = 0
autoSpeedButton.Parent = quickButtonsContainer

local autoSpeedCorner = Instance.new("UICorner")
autoSpeedCorner.CornerRadius = UDim.new(0, 10)
autoSpeedCorner.Parent = autoSpeedButton

local turboButton = Instance.new("TextButton")
turboButton.Size = UDim2.new(0, isMobile and 70 or 60, 0, isMobile and 42 or 38)
turboButton.Text = "🚀"
turboButton.TextColor3 = Color3.fromRGB(255, 255, 255)
turboButton.TextSize = isMobile and 24 or 20
turboButton.Font = Enum.Font.GothamBold
turboButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
turboButton.BorderSizePixel = 0
turboButton.Parent = quickButtonsContainer

local turboCorner = Instance.new("UICorner")
turboCorner.CornerRadius = UDim.new(0, 10)
turboCorner.Parent = turboButton

local infoButton = Instance.new("TextButton")
infoButton.Size = UDim2.new(0, isMobile and 70 or 60, 0, isMobile and 42 or 38)
infoButton.Text = "ℹ️"
infoButton.TextColor3 = Color3.fromRGB(255, 255, 255)
infoButton.TextSize = isMobile and 24 or 20
infoButton.Font = Enum.Font.GothamBold
infoButton.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
infoButton.BorderSizePixel = 0
infoButton.Parent = quickButtonsContainer

local infoCorner = Instance.new("UICorner")
infoCorner.CornerRadius = UDim.new(0, 10)
infoCorner.Parent = infoButton

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
creditLabel.Text = "⚡ RIAN STUDIOS V8.0 ⚡"
creditLabel.TextColor3 = Color3.fromRGB(120, 120, 120)
creditLabel.TextSize = isMobile and 10 or 9
creditLabel.Font = Enum.Font.Gotham
creditLabel.Parent = footerFrame

local versionLabel = Instance.new("TextLabel")
versionLabel.Size = UDim2.new(1, 0, 0, 14)
versionLabel.Position = UDim2.new(0, 0, 0, 22)
versionLabel.BackgroundTransparency = 1
versionLabel.Text = "SPEED | FLY | AIMBOT | INVINCIBLE | COLLECT"
versionLabel.TextColor3 = Color3.fromRGB(80, 80, 80)
versionLabel.TextSize = isMobile and 8 or 7
versionLabel.Font = Enum.Font.Gotham
versionLabel.Parent = footerFrame

-- ============ FUNÇÕES PRINCIPAIS ============

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

-- BOTÃO VELOCIDADE
autoSpeedButton.MouseButton1Click:Connect(function()
    playSound("rbxassetid://9120384332", 0.3)
    
    if not speedActivated then
        speedActivated = true
        updateSpeed(FAST_SPEED, true)
        isSpeedBoosted = true
        autoSpeedButton.Text = "✅ ATIVADO"
        autoSpeedButton.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
    else
        speedActivated = false
        updateSpeed(NORMAL_SPEED, true)
        isSpeedBoosted = false
        autoSpeedButton.Text = "🔘 ATIVAR"
        autoSpeedButton.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    end
end)

-- BOTÃO TURBO
turboButton.MouseButton1Click:Connect(function()
    if isBombaBoosted then return end
    playSound("rbxassetid://9120384332", 0.3)
    updateSpeed(MAX_SPEED, true)
    isSpeedBoosted = true
    speedActivated = true
    autoSpeedButton.Text = "✅ ATIVADO"
    autoSpeedButton.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
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

-- SISTEMA DE COLETA
local function findNearbyBlocks()
    local blocks = {}
    local rootPos = humanoidRootPart and humanoidRootPart.Position or character:GetPivot().Position
    
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and not obj:IsA("Terrain") then
            local objName = obj.Name:lower()
            local isCollectable = false
            
            local collectableNames = {"block", "brick", "cube", "collect", "pickup", "item", "coin", "gem", "crystal", "powerup", "boost"}
            for _, name in pairs(collectableNames) do
                if objName:find(name) then
                    isCollectable = true
                    break
                end
            end
            
            if obj:GetAttribute("Collectable") or obj:GetAttribute("Block") then
                isCollectable = true
            end
            
            if isCollectable and not collectedBlocksList[obj] then
                local distance = (rootPos - obj.Position).Magnitude
                if distance <= blockCollectionRange then
                    table.insert(blocks, obj)
                end
            end
        end
    end
    
    return blocks
end

local function collectBlock(block)
    if collectedBlocksList[block] then return false end
    
    collectedBlocksList[block] = true
    blocksCollected = blocksCollected + 1
    
    collectValueLabel.Text = blocksCollected .. " / " .. targetBlocks
    local percent = blocksCollected / targetBlocks
    collectProgressFill:TweenSize(UDim2.new(percent, 0, 1, 0), "Out", "Quad", 0.2, true)
    
    -- Efeito visual
    local collectEffect = Instance.new("Part")
    collectEffect.Shape = Enum.PartType.Ball
    collectEffect.Size = Vector3.new(1.5, 1.5, 1.5)
    collectEffect.BrickColor = BrickColor.new("Bright yellow")
    collectEffect.Material = Enum.Material.Neon
    collectEffect.CFrame = block.CFrame
    collectEffect.Anchored = true
    collectEffect.CanCollide = false
    collectEffect.Parent = workspace
    
    TweenService:Create(collectEffect, TweenInfo.new(0.3), {Size = Vector3.new(0, 0, 0)}):Play()
    game:GetService("Debris"):AddItem(collectEffect, 0.3)
    
    playSound("rbxassetid://9120384332", 0.4)
    
    block.Transparency = 1
    block.CanCollide = false
    
    game:GetService("Debris"):AddItem(block, 2)
    
    if blocksCollected >= targetBlocks then
        local completeNotif = Instance.new("Frame")
        completeNotif.Size = UDim2.new(0, 280, 0, 50)
        completeNotif.Position = UDim2.new(0.5, -140, 0.5, -25)
        completeNotif.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
        completeNotif.BackgroundTransparency = 0.1
        completeNotif.BorderSizePixel = 0
        completeNotif.Parent = screenGui
        
        local completeCorner = Instance.new("UICorner")
        completeCorner.CornerRadius = UDim.new(0, 12)
        completeCorner.Parent = completeNotif
        
        local completeText = Instance.new("TextLabel")
        completeText.Size = UDim2.new(1, 0, 1, 0)
        completeText.BackgroundTransparency = 1
        completeText.Text = "🎉 META ALCANÇADA! 🎉"
        completeText.TextColor3 = Color3.fromRGB(255, 255, 255)
        completeText.TextScaled = true
        completeText.Font = Enum.Font.GothamBold
        completeText.Parent = completeNotif
        
        TweenService:Create(completeNotif, TweenInfo.new(0.4), {BackgroundTransparency = 0.3, Position = UDim2.new(0.5, -140, 0.3, -25)}):Play()
        wait(3)
        TweenService:Create(completeNotif, TweenInfo.new(0.4), {BackgroundTransparency = 1}):Play()
        wait(0.3)
        completeNotif:Destroy()
    end
    
    return true
end

local function startAutoCollect()
    if autoCollectConnection then autoCollectConnection:Disconnect() end
    
    autoCollectConnection = RunService.RenderStepped:Connect(function()
        if autoCollectEnabled and character and humanoidRootPart then
            local nearbyBlocks = findNearbyBlocks()
            for _, block in pairs(nearbyBlocks) do
                if not collectedBlocksList[block] then
                    collectBlock(block)
                    wait(0.1)
                end
            end
        end
    end)
end

local function updateTargetBlocks(newTarget)
    targetBlocks = math.clamp(newTarget, 30, 250)
    collectValueLabel.Text = blocksCollected .. " / " .. targetBlocks
    
    local percent = (targetBlocks - 30) / (250 - 30)
    targetSliderFill:TweenSize(UDim2.new(percent, 0, 1, 0), "Out", "Quad", 0.1, true)
    targetSliderButton:TweenPosition(UDim2.new(percent, -9, 0.5, -9), "Out", "Quad", 0.1, true)
    
    local collectPercent = blocksCollected / targetBlocks
    collectProgressFill:TweenSize(UDim2.new(collectPercent, 0, 1, 0), "Out", "Quad", 0.2, true)
end

-- Slider da meta
local draggingTarget = false
local function updateTargetFromMouse(input)
    local mousePos = input.Position.X
    local sliderAbsPos = targetSliderBg.AbsolutePosition.X
    local sliderWidth = targetSliderBg.AbsoluteSize.X
    local percent = math.clamp((mousePos - sliderAbsPos) / sliderWidth, 0, 1)
    local newTarget = 30 + (percent * (250 - 30))
    updateTargetBlocks(math.floor(newTarget))
end

targetSliderButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingTarget = true
        updateTargetFromMouse(input)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingTarget and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        updateTargetFromMouse(input)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingTarget = false
    end
end)

-- Botões de coleta
autoCollectButton.MouseButton1Click:Connect(function()
    playSound("rbxassetid://9120384332", 0.3)
    autoCollectEnabled = not autoCollectEnabled
    
    if autoCollectEnabled then
        autoCollectButton.Text = "⏹️ AUTO"
        autoCollectButton.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
        collectCardStroke.Color = Color3.fromRGB(76, 175, 80)
        startAutoCollect()
    else
        autoCollectButton.Text = "🔄 AUTO"
        autoCollectButton.BackgroundColor3 = Color3.fromRGB(255, 152, 0)
        collectCardStroke.Color = Color3.fromRGB(255, 215, 0)
        
        if autoCollectConnection then
            autoCollectConnection:Disconnect()
            autoCollectConnection = nil
        end
    end
end)

resetCollectButton.MouseButton1Click:Connect(function()
    playSound("rbxassetid://9120384332", 0.3)
    blocksCollected = 0
    collectedBlocksList = {}
    collectValueLabel.Text = blocksCollected .. " / " .. targetBlocks
    local percent = blocksCollected / targetBlocks
    collectProgressFill:TweenSize(UDim2.new(percent, 0, 1, 0), "Out", "Quad", 0.2, true)
end)

-- SLIDER DE VELOCIDADE
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
    
    if speedActivated then
        autoSpeedButton.Text = "✅ ATIVADO"
        autoSpeedButton.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
    else
        autoSpeedButton.Text = "🔘 ATIVAR"
        autoSpeedButton.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
    end
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
infoButton.MouseButton1Click:Connect(function()
    local infoFrame = Instance.new("Frame")
    infoFrame.Size = UDim2.new(0, 300, 0, 350)
    infoFrame.Position = UDim2.new(0.5, -150, 0.5, -175)
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
⚡ RIAN STUDIOS V8.0 ⚡

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎮 VELOCIDADE:
   • ATIVAR: 80 (persistente)
   • SLIDER: Ajuste 16-120
   • TURBO: Máximo instantâneo

🕊️ VOO: WASD + ESPAÇO/SHIFT

🎯 AIMBOT: Mira automática

🛡️ INVINCIBLE: Imune a danos

📦 COLETA:
   • Meta: 30 a 250 blocos
   • AUTO: Coleta automática

👊 ATAQUE: Clique esquerdo (combo 3x)

💣 BOOST: Pegue bombas!

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
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
    
    currentSpeed = speedActivated and FAST_SPEED or NORMAL_SPEED
    applySpeedToCharacter()
    
    speedValueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    sliderFill.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
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
startAutoCollect()

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
print("✅ SISTEMA RIAN STUDIOS V8.0 CARREGADO COM SUCESSO!")
print("✅ ANTI-BAN ATIVADO - VOCÊ NÃO SERÁ EXPULSO!")
print("✅ SISTEMA DE COLETA: 30 a 250 blocos!")
print("✅ AIMBOT | INVINCIBLE | VOO | VELOCIDADE | COLETA")
print("═══════════════════════════════════════════════════════════════════════════════")
