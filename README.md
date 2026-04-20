--[[
    ╔═══════════════════════════════════════════════════════════════════════════╗
    ║                                                                           ║
    ║     ██████╗ ██╗ █████╗ ███╗   ██╗    ███████╗████████╗██╗   ██╗██████╗   ║
    ║     ██╔══██╗██║██╔══██╗████╗  ██║    ██╔════╝╚══██╔══╝██║   ██║██╔══██╗  ║
    ║     ██████╔╝██║███████║██╔██╗ ██║    ███████╗   ██║   ██║   ██║██║  ██║  ║
    ║     ██╔══██╗██║██╔══██║██║╚██╗██║    ╚════██║   ██║   ██║   ██║██║  ██║  ║
    ║     ██║  ██║██║██║  ██║██║ ╚████║    ███████║   ██║   ╚██████╔╝██████╔╝  ║
    ║     ╚═╝  ╚═╝╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝    ╚══════╝   ╚═╝    ╚═════╝ ╚═════╝   ║
    ║                                                                           ║
    ║                   SISTEMA AVANÇADO V4.1 - RIAN STUDIOS                     ║
    ║              VELOCIDADE | VOO MOBILE | BOOST | COMBATE                     ║
    ║                                                                           ║
    ╚═══════════════════════════════════════════════════════════════════════════╝
]]

-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")

-- Variáveis do jogador
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- Detecta se é mobile
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled and not UserInputService.MouseEnabled

-- Configurações de Velocidade
local NORMAL_SPEED = 16
local FAST_SPEED = 80
local BOMBA_SPEED = 120  -- Velocidade máxima ao pegar bomba
local MIN_SPEED = 16
local MAX_SPEED = 120
local currentSpeed = NORMAL_SPEED
local isSpeedBoosted = false
local isBombaBoosted = false
local bombaBoostTimer = nil
local originalButtonColor = nil

-- Configurações de VOO
local isFlying = false
local FLY_SPEED = 80
local flyBodyVelocity = nil
local flyBodyGyro = nil
local defaultGravity = Workspace.Gravity
local flyUp = false
local flyDown = false

-- Configurações de Ataque
local PUNCH_DAMAGE = 15
local PUNCH_COOLDOWN = 0.3
local COMBO_DELAY = 0.15
local canPunch = true
local isAttacking = false
local comboCount = 0

-- Sons
local soundService = game:GetService("SoundService")
local punchSound = Instance.new("Sound")
punchSound.SoundId = "rbxassetid://9120384332"
punchSound.Volume = 0.5
punchSound.Parent = soundService

local boostSound = Instance.new("Sound")
boostSound.SoundId = "rbxassetid://9120384332"
boostSound.Volume = 0.7
boostSound.Parent = soundService

local flySound = Instance.new("Sound")
flySound.SoundId = "rbxassetid://9120384332"
flySound.Volume = 0.4
flySound.Looped = true
flySound.Parent = soundService

local clickSound = Instance.new("Sound")
clickSound.SoundId = "rbxassetid://9120384332"
clickSound.Volume = 0.3
clickSound.Parent = soundService

-- CRIAR UI PRINCIPAL
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RianStudiosUI"
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.IgnoreGuiInset = true

-- BOTÃO FLUTUANTE ANIMADO
local floatingButton = Instance.new("ImageButton")
floatingButton.Name = "RianFloatingButton"
floatingButton.Size = UDim2.new(0, 70, 0, 70)
floatingButton.Position = UDim2.new(0.02, 0, 0.82, 0)
floatingButton.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
floatingButton.BackgroundTransparency = 0
floatingButton.Image = "rbxassetid://6031094773"
floatingButton.ImageColor3 = Color3.fromRGB(255, 255, 255)
floatingButton.ScaleType = Enum.ScaleType.Fit
floatingButton.Parent = screenGui

local floatingCorner = Instance.new("UICorner")
floatingCorner.CornerRadius = UDim.new(1, 0)
floatingCorner.Parent = floatingButton

local floatingShadow = Instance.new("UIStroke")
floatingShadow.Color = Color3.fromRGB(255, 255, 255)
floatingShadow.Thickness = 2
floatingShadow.Transparency = 0.5
floatingShadow.Parent = floatingButton

-- CONTAINER PRINCIPAL
local mainContainer = Instance.new("Frame")
mainContainer.Name = "RianMainContainer"
mainContainer.Size = UDim2.new(0, 450, 0, 700)
mainContainer.Position = UDim2.new(0.5, -225, 0.5, -350)
mainContainer.BackgroundColor3 = Color3.fromRGB(18, 18, 28)
mainContainer.BackgroundTransparency = 0.05
mainContainer.BorderSizePixel = 0
mainContainer.Visible = false
mainContainer.Parent = screenGui
mainContainer.ClipsDescendants = true

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 20)
mainCorner.Parent = mainContainer

local mainBorder = Instance.new("UIStroke")
mainBorder.Color = Color3.fromRGB(66, 135, 245)
mainBorder.Thickness = 1.5
mainBorder.Transparency = 0.7
mainBorder.Parent = mainContainer

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 70)
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

local creatorLabel = Instance.new("TextLabel")
creatorLabel.Size = UDim2.new(1, -80, 0, 25)
creatorLabel.Position = UDim2.new(0, 20, 0, 10)
creatorLabel.BackgroundTransparency = 1
creatorLabel.Text = "⚡ RIAN STUDIOS"
creatorLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
creatorLabel.TextSize = 14
creatorLabel.Font = Enum.Font.GothamBold
creatorLabel.TextXAlignment = Enum.TextXAlignment.Left
creatorLabel.TextTransparency = 0.1
creatorLabel.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -80, 0, 30)
titleLabel.Position = UDim2.new(0, 20, 0, 35)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "🎮 SISTEMA DE CONTROLE V4.1"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = 16
titleLabel.Font = Enum.Font.GothamSemibold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local creatorIcon = Instance.new("ImageLabel")
creatorIcon.Size = UDim2.new(0, 40, 0, 40)
creatorIcon.Position = UDim2.new(1, -55, 0, 15)
creatorIcon.Image = "rbxassetid://6031094773"
creatorIcon.ImageColor3 = Color3.fromRGB(255, 255, 255)
creatorIcon.BackgroundTransparency = 1
creatorIcon.Parent = titleBar

local closeButton = Instance.new("ImageButton")
closeButton.Size = UDim2.new(0, 35, 0, 35)
closeButton.Position = UDim2.new(1, -48, 0, 17)
closeButton.Image = "rbxassetid://3926305904"
closeButton.ImageColor3 = Color3.fromRGB(255, 255, 255)
closeButton.BackgroundTransparency = 1
closeButton.Parent = titleBar

-- CONTAINER DE VELOCIDADE
local speedCard = Instance.new("Frame")
speedCard.Size = UDim2.new(1, -30, 0, 165)
speedCard.Position = UDim2.new(0, 15, 0, 85)
speedCard.BackgroundColor3 = Color3.fromRGB(28, 28, 38)
speedCard.BackgroundTransparency = 0.2
speedCard.BorderSizePixel = 0
speedCard.Parent = mainContainer

local speedCardCorner = Instance.new("UICorner")
speedCardCorner.CornerRadius = UDim.new(0, 14)
speedCardCorner.Parent = speedCard

local speedCardStroke = Instance.new("UIStroke")
speedCardStroke.Color = Color3.fromRGB(66, 135, 245)
speedCardStroke.Thickness = 0.5
speedCardStroke.Transparency = 0.8
speedCardStroke.Parent = speedCard

local speedIcon = Instance.new("ImageLabel")
speedIcon.Size = UDim2.new(0, 40, 0, 40)
speedIcon.Position = UDim2.new(0, 15, 0, 15)
speedIcon.Image = "rbxassetid://6031094773"
speedIcon.ImageColor3 = Color3.fromRGB(66, 135, 245)
speedIcon.BackgroundTransparency = 1
speedIcon.Parent = speedCard

local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(1, -70, 0, 30)
speedLabel.Position = UDim2.new(0, 65, 0, 15)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "VELOCIDADE ATUAL"
speedLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
speedLabel.TextSize = 12
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Parent = speedCard

local speedValueLabel = Instance.new("TextLabel")
speedValueLabel.Size = UDim2.new(1, -70, 0, 35)
speedValueLabel.Position = UDim2.new(0, 65, 0, 35)
speedValueLabel.BackgroundTransparency = 1
speedValueLabel.Text = math.floor(currentSpeed) .. " / " .. MAX_SPEED
speedValueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedValueLabel.TextSize = 22
speedValueLabel.Font = Enum.Font.GothamBold
speedValueLabel.TextXAlignment = Enum.TextXAlignment.Left
speedValueLabel.Parent = speedCard

local bombTimerLabel = Instance.new("TextLabel")
bombTimerLabel.Size = UDim2.new(1, -70, 0, 20)
bombTimerLabel.Position = UDim2.new(0, 65, 0, 70)
bombTimerLabel.BackgroundTransparency = 1
bombTimerLabel.Text = ""
bombTimerLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
bombTimerLabel.TextSize = 11
bombTimerLabel.Font = Enum.Font.GothamBold
bombTimerLabel.TextXAlignment = Enum.TextXAlignment.Left
bombTimerLabel.Visible = false
bombTimerLabel.Parent = speedCard

-- SLIDER
local sliderBg = Instance.new("Frame")
sliderBg.Size = UDim2.new(0.88, 0, 0, 8)
sliderBg.Position = UDim2.new(0.06, 0, 0.72, 0)
sliderBg.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
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

local minLabel = Instance.new("TextLabel")
minLabel.Size = UDim2.new(0, 35, 0, 20)
minLabel.Position = UDim2.new(0.04, 0, 0.85, 0)
minLabel.BackgroundTransparency = 1
minLabel.Text = tostring(MIN_SPEED)
minLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
minLabel.TextSize = 11
minLabel.Font = Enum.Font.Gotham
minLabel.Parent = speedCard

local maxLabel = Instance.new("TextLabel")
maxLabel.Size = UDim2.new(0, 35, 0, 20)
maxLabel.Position = UDim2.new(0.89, 0, 0.85, 0)
maxLabel.BackgroundTransparency = 1
maxLabel.Text = tostring(MAX_SPEED)
maxLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
maxLabel.TextSize = 11
maxLabel.Font = Enum.Font.Gotham
maxLabel.Parent = speedCard

-- Botão Turbo
local turboButton = Instance.new("TextButton")
turboButton.Size = UDim2.new(0, 110, 0, 36)
turboButton.Position = UDim2.new(0.5, -55, 1, -12)
turboButton.Text = "🚀 TURBO"
turboButton.TextColor3 = Color3.fromRGB(255, 255, 255)
turboButton.TextSize = 13
turboButton.Font = Enum.Font.GothamBold
turboButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
turboButton.BorderSizePixel = 0
turboButton.Parent = speedCard

local turboCorner = Instance.new("UICorner")
turboCorner.CornerRadius = UDim.new(0, 8)
turboCorner.Parent = turboButton

-- CONTAINER DE VOO
local flyCard = Instance.new("Frame")
flyCard.Size = UDim2.new(1, -30, 0, 110)
flyCard.Position = UDim2.new(0, 15, 0, 265)
flyCard.BackgroundColor3 = Color3.fromRGB(28, 28, 38)
flyCard.BackgroundTransparency = 0.2
flyCard.BorderSizePixel = 0
flyCard.Parent = mainContainer

local flyCardCorner = Instance.new("UICorner")
flyCardCorner.CornerRadius = UDim.new(0, 14)
flyCardCorner.Parent = flyCard

local flyCardStroke = Instance.new("UIStroke")
flyCardStroke.Color = Color3.fromRGB(66, 200, 100)
flyCardStroke.Thickness = 0.5
flyCardStroke.Transparency = 0.8
flyCardStroke.Parent = flyCard

local flyIcon = Instance.new("ImageLabel")
flyIcon.Size = UDim2.new(0, 40, 0, 40)
flyIcon.Position = UDim2.new(0, 15, 0, 15)
flyIcon.Image = "rbxassetid://6031094773"
flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
flyIcon.BackgroundTransparency = 1
flyIcon.Parent = flyCard

local flyTitle = Instance.new("TextLabel")
flyTitle.Size = UDim2.new(1, -70, 0, 25)
flyTitle.Position = UDim2.new(0, 65, 0, 15)
flyTitle.BackgroundTransparency = 1
flyTitle.Text = "🕊️ SISTEMA DE VOO"
flyTitle.TextColor3 = Color3.fromRGB(200, 200, 200)
flyTitle.TextSize = 12
flyTitle.Font = Enum.Font.Gotham
flyTitle.TextXAlignment = Enum.TextXAlignment.Left
flyTitle.Parent = flyCard

local flyStatusLabel = Instance.new("TextLabel")
flyStatusLabel.Size = UDim2.new(1, -70, 0, 25)
flyStatusLabel.Position = UDim2.new(0, 65, 0, 38)
flyStatusLabel.BackgroundTransparency = 1
flyStatusLabel.Text = "STATUS: DESLIGADO"
flyStatusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
flyStatusLabel.TextSize = 13
flyStatusLabel.Font = Enum.Font.GothamBold
flyStatusLabel.TextXAlignment = Enum.TextXAlignment.Left
flyStatusLabel.Parent = flyCard

local flyButton = Instance.new("TextButton")
flyButton.Size = UDim2.new(0, 130, 0, 42)
flyButton.Position = UDim2.new(0.5, -65, 1, -12)
flyButton.Text = "🕊️ ATIVAR VOO"
flyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
flyButton.TextSize = 13
flyButton.Font = Enum.Font.GothamBold
flyButton.BackgroundColor3 = Color3.fromRGB(66, 200, 100)
flyButton.BorderSizePixel = 0
flyButton.Parent = flyCard

local flyButtonCorner = Instance.new("UICorner")
flyButtonCorner.CornerRadius = UDim.new(0, 8)
flyButtonCorner.Parent = flyButton

-- Container dos botões
local actionTitle = Instance.new("TextLabel")
actionTitle.Size = UDim2.new(1, -30, 0, 25)
actionTitle.Position = UDim2.new(0, 15, 0, 390)
actionTitle.BackgroundTransparency = 1
actionTitle.Text = "⚡ CONTROLES RÁPIDOS"
actionTitle.TextColor3 = Color3.fromRGB(200, 200, 200)
actionTitle.TextSize = 13
actionTitle.Font = Enum.Font.GothamBold
actionTitle.TextXAlignment = Enum.TextXAlignment.Left
actionTitle.Parent = mainContainer

local buttonsContainer = Instance.new("Frame")
buttonsContainer.Size = UDim2.new(1, -30, 0, 120)
buttonsContainer.Position = UDim2.new(0, 15, 0, 420)
buttonsContainer.BackgroundTransparency = 1
buttonsContainer.Parent = mainContainer

local gridLayout = Instance.new("UIGridLayout")
gridLayout.CellSize = UDim2.new(0, 190, 0, 52)
gridLayout.CellPadding = UDim2.new(0, 12, 0, 12)
gridLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
gridLayout.Parent = buttonsContainer

-- Footer
local footerFrame = Instance.new("Frame")
footerFrame.Size = UDim2.new(1, 0, 0, 45)
footerFrame.Position = UDim2.new(0, 0, 1, -45)
footerFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 22)
footerFrame.BackgroundTransparency = 0.3
footerFrame.BorderSizePixel = 0
footerFrame.Parent = mainContainer

local footerCorner = Instance.new("UICorner")
footerCorner.CornerRadius = UDim.new(0, 14)
footerCorner.Parent = footerFrame

local creditLabel = Instance.new("TextLabel")
creditLabel.Size = UDim2.new(1, 0, 1, 0)
creditLabel.Position = UDim2.new(0, 0, 0, 5)
creditLabel.BackgroundTransparency = 1
creditLabel.Text = "⚡ DESENVOLVIDO POR RIAN ⚡"
creditLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
creditLabel.TextSize = 11
creditLabel.Font = Enum.Font.Gotham
creditLabel.Parent = footerFrame

local versionLabel = Instance.new("TextLabel")
versionLabel.Size = UDim2.new(1, 0, 1, 0)
versionLabel.Position = UDim2.new(0, 0, 0, 22)
versionLabel.BackgroundTransparency = 1
versionLabel.Text = "VERSION 4.1 | BOOST CORRIGIDO | MOBILE COMPATÍVEL"
versionLabel.TextColor3 = Color3.fromRGB(100, 100, 100)
versionLabel.TextSize = 9
versionLabel.Font = Enum.Font.Gotham
versionLabel.Parent = footerFrame

-- Controles mobile para voo
local mobileFlyControls = Instance.new("Frame")
mobileFlyControls.Name = "MobileFlyControls"
mobileFlyControls.Size = UDim2.new(0, 200, 0, 100)
mobileFlyControls.Position = UDim2.new(0.5, -100, 1, -120)
mobileFlyControls.BackgroundColor3 = Color3.fromRGB(28, 28, 38)
mobileFlyControls.BackgroundTransparency = 0.3
mobileFlyControls.BorderSizePixel = 0
mobileFlyControls.Visible = false
mobileFlyControls.Parent = screenGui

local mobileControlsCorner = Instance.new("UICorner")
mobileControlsCorner.CornerRadius = UDim.new(0, 12)
mobileControlsCorner.Parent = mobileFlyControls

local flyUpButton = Instance.new("TextButton")
flyUpButton.Size = UDim2.new(0, 80, 0, 80)
flyUpButton.Position = UDim2.new(0.1, 0, 0.1, 0)
flyUpButton.Text = "⬆️"
flyUpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
flyUpButton.TextSize = 30
flyUpButton.Font = Enum.Font.GothamBold
flyUpButton.BackgroundColor3 = Color3.fromRGB(66, 200, 100)
flyUpButton.BorderSizePixel = 0
flyUpButton.Parent = mobileFlyControls

local flyUpCorner = Instance.new("UICorner")
flyUpCorner.CornerRadius = UDim.new(0, 40)
flyUpCorner.Parent = flyUpButton

local flyDownButton = Instance.new("TextButton")
flyDownButton.Size = UDim2.new(0, 80, 0, 80)
flyDownButton.Position = UDim2.new(0.55, 0, 0.1, 0)
flyDownButton.Text = "⬇️"
flyDownButton.TextColor3 = Color3.fromRGB(255, 255, 255)
flyDownButton.TextSize = 30
flyDownButton.Font = Enum.Font.GothamBold
flyDownButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
flyDownButton.BorderSizePixel = 0
flyDownButton.Parent = mobileFlyControls

local flyDownCorner = Instance.new("UICorner")
flyDownCorner.CornerRadius = UDim.new(0, 40)
flyDownCorner.Parent = flyDownButton

local mobileInstruction = Instance.new("TextLabel")
mobileInstruction.Size = UDim2.new(1, 0, 0, 25)
mobileInstruction.Position = UDim2.new(0, 0, 1, -30)
mobileInstruction.BackgroundTransparency = 1
mobileInstruction.Text = "Use os botões para subir/descer"
mobileInstruction.TextColor3 = Color3.fromRGB(200, 200, 200)
mobileInstruction.TextSize = 10
mobileInstruction.Font = Enum.Font.Gotham
mobileInstruction.Parent = mobileFlyControls

-- ============ FUNÇÃO PRINCIPAL DE VELOCIDADE ============
local function applySpeedToCharacter()
    if not humanoid then return end
    
    if isFlying then
        FLY_SPEED = currentSpeed
    else
        humanoid.WalkSpeed = currentSpeed
    end
    
    -- Atualizar UI
    speedValueLabel.Text = math.floor(currentSpeed) .. " / " .. MAX_SPEED
    
    local percent = (currentSpeed - MIN_SPEED) / (MAX_SPEED - MIN_SPEED)
    sliderFill:TweenSize(UDim2.new(percent, 0, 1, 0), "Out", "Quad", 0.1, true)
    sliderButton:TweenPosition(UDim2.new(percent, -11, 0.5, -11), "Out", "Quad", 0.1, true)
end

local function updateSpeed(newSpeed, ignoreBoostLock)
    -- Se tiver boost ativo e não for para ignorar o bloqueio, não permite mudar
    if isBombaBoosted and not ignoreBoostLock then
        return
    end
    
    currentSpeed = math.clamp(newSpeed, MIN_SPEED, MAX_SPEED)
    applySpeedToCharacter()
end

-- ============ BOOST DA BOMBA CORRIGIDO ============
local function activateBombaBoost()
    -- Se já tiver boost ativo, não faz nada
    if isBombaBoosted then 
        return 
    end
    
    print("🔥 BOOST DA BOMBA ACTIVADO! Velocidade máxima por 10 segundos!")
    
    isBombaBoosted = true
    isSpeedBoosted = true
    
    -- Salvar a cor original do botão
    originalButtonColor = speedToggleButton.BackgroundColor3
    
    -- Tocar som
    local soundClone = boostSound:Clone()
    soundClone.Parent = character
    soundClone:Play()
    
    -- Forçar velocidade máxima da bomba (120)
    currentSpeed = BOMBA_SPEED
    applySpeedToCharacter()
    
    -- Mudar cores para indicar boost ativo
    TweenService:Create(speedToggleButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(255, 100, 100)}):Play()
    TweenService:Create(speedIcon, TweenInfo.new(0.3), {ImageColor3 = Color3.fromRGB(255, 100, 100)}):Play()
    TweenService:Create(speedCardStroke, TweenInfo.new(0.3), {Color = Color3.fromRGB(255, 100, 100)}):Play()
    
    speedLabel.Text = "💣 BOOST ACTIVO!"
    speedValueLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
    sliderFill.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
    
    bombTimerLabel.Visible = true
    
    -- Efeitos visuais no personagem
    local glowEffect = Instance.new("PointLight")
    glowEffect.Color = Color3.fromRGB(255, 100, 0)
    glowEffect.Range = 12
    glowEffect.Brightness = 2.5
    glowEffect.Parent = humanoidRootPart or character
    
    local particleEffect = Instance.new("ParticleEmitter")
    particleEffect.Texture = "rbxassetid://284646226"
    particleEffect.Rate = 60
    particleEffect.SpreadAngle = Vector2.new(360, 360)
    particleEffect.VelocityInheritance = 1
    particleEffect.Lifetime = NumberRange.new(0.5)
    particleEffect.Speed = NumberRange.new(6)
    particleEffect.Color = ColorSequence.new(Color3.fromRGB(255, 100, 0))
    particleEffect.Parent = humanoidRootPart or character
    
    -- Notificação visual
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0, 350, 0, 70)
    notif.Position = UDim2.new(0.5, -175, 0, -70)
    notif.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
    notif.BackgroundTransparency = 0.1
    notif.BorderSizePixel = 0
    notif.Parent = screenGui
    
    local notifCorner = Instance.new("UICorner")
    notifCorner.CornerRadius = UDim.new(0, 12)
    notifCorner.Parent = notif
    
    local notifText = Instance.new("TextLabel")
    notifText.Size = UDim2.new(1, 0, 1, 0)
    notifText.BackgroundTransparency = 1
    notifText.Text = "💣 BOOST DE VELOCIDADE ACTIVADO! 💣\n🚀 VELOCIDADE MÁXIMA: " .. BOMBA_SPEED
    notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
    notifText.TextScaled = true
    notifText.Font = Enum.Font.GothamBold
    notifText.Parent = notif
    
    TweenService:Create(notif, TweenInfo.new(0.4, Enum.EasingStyle.Back), {Position = UDim2.new(0.5, -175, 0, 20)}):Play()
    
    -- Timer de 10 segundos
    local startTime = tick()
    local timerConnection
    
    timerConnection = RunService.RenderStepped:Connect(function()
        if not isBombaBoosted then
            if timerConnection then timerConnection:Disconnect() end
            return
        end
        
        local elapsed = tick() - startTime
        local remaining = math.max(0, 10 - elapsed)
        bombTimerLabel.Text = "💣 BOOST ACTIVO! TEMPO RESTANTE: " .. math.ceil(remaining) .. "s"
        
        if remaining <= 0 then
            timerConnection:Disconnect()
            
            -- FINALIZAR BOOST
            print("⏰ Boost da bomba finalizado! Voltando à velocidade normal.")
            
            isBombaBoosted = false
            isSpeedBoosted = false
            
            -- Voltar para velocidade normal
            currentSpeed = NORMAL_SPEED
            applySpeedToCharacter()
            
            -- Restaurar cores originais
            bombTimerLabel.Visible = false
            speedLabel.Text = "⚡ VELOCIDADE ATUAL"
            speedValueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
            sliderFill.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
            
            TweenService:Create(speedToggleButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(66, 135, 245)}):Play()
            TweenService:Create(speedIcon, TweenInfo.new(0.3), {ImageColor3 = Color3.fromRGB(66, 135, 245)}):Play()
            TweenService:Create(speedCardStroke, TweenInfo.new(0.3), {Color = Color3.fromRGB(66, 135, 245)}):Play()
            
            -- Remover efeitos visuais
            if glowEffect then glowEffect:Destroy() end
            if particleEffect then particleEffect:Destroy() end
            
            -- Notificação de fim
            local endNotif = Instance.new("Frame")
            endNotif.Size = UDim2.new(0, 300, 0, 50)
            endNotif.Position = UDim2.new(0.5, -150, 0, -60)
            endNotif.BackgroundColor3 = Color3.fromRGB(64, 64, 64)
            endNotif.BackgroundTransparency = 0.1
            endNotif.BorderSizePixel = 0
            endNotif.Parent = screenGui
            
            local endCorner = Instance.new("UICorner")
            endCorner.CornerRadius = UDim.new(0, 12)
            endCorner.Parent = endNotif
            
            local endText = Instance.new("TextLabel")
            endText.Size = UDim2.new(1, 0, 1, 0)
            endText.BackgroundTransparency = 1
            endText.Text = "⚡ BOOST FINALIZADO! VELOCIDADE NORMAL ⚡"
            endText.TextColor3 = Color3.fromRGB(255, 255, 255)
            endText.TextScaled = true
            endText.Font = Enum.Font.GothamBold
            endText.Parent = endNotif
            
            TweenService:Create(endNotif, TweenInfo.new(0.4), {Position = UDim2.new(0.5, -150, 0, 20)}):Play()
            wait(2)
            TweenService:Create(endNotif, TweenInfo.new(0.4), {Position = UDim2.new(0.5, -150, 0, -60), BackgroundTransparency = 1}):Play()
            wait(0.3)
            endNotif:Destroy()
        end
    end)
    
    wait(2.5)
    TweenService:Create(notif, TweenInfo.new(0.4), {Position = UDim2.new(0.5, -175, 0, -70), BackgroundTransparency = 1}):Play()
    wait(0.3)
    notif:Destroy()
end

-- ============ SISTEMA DE VOO ============
local function updateFlyMovement()
    if not isFlying or not flyBodyVelocity then return end
    
    local moveDirection = Vector3.new()
    
    if not isMobile then
        if UserInputService:IsKeyDown(Enum.KeyCode.W) or UserInputService:IsKeyDown(Enum.KeyCode.Up) then
            moveDirection = moveDirection + humanoidRootPart.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) or UserInputService:IsKeyDown(Enum.KeyCode.Down) then
            moveDirection = moveDirection - humanoidRootPart.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) or UserInputService:IsKeyDown(Enum.KeyCode.Left) then
            moveDirection = moveDirection - humanoidRootPart.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) or UserInputService:IsKeyDown(Enum.KeyCode.Right) then
            moveDirection = moveDirection + humanoidRootPart.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDirection = moveDirection + Vector3.new(0, 1, 0)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            moveDirection = moveDirection - Vector3.new(0, 1, 0)
        end
    else
        if flyUp then
            moveDirection = moveDirection + Vector3.new(0, 1, 0)
        end
        if flyDown then
            moveDirection = moveDirection - Vector3.new(0, 1, 0)
        end
        if humanoid then
            local moveVector = humanoid.MoveDirection
            if moveVector.Magnitude > 0 then
                moveDirection = moveDirection + moveVector
            end
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
    flyButton.Text = "🕊️ DESATIVAR VOO"
    flyButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
    flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
    flyCardStroke.Color = Color3.fromRGB(66, 200, 100)
    
    if isMobile then
        mobileFlyControls.Visible = true
    end
    
    Workspace.Gravity = 0
    
    flyBodyVelocity = Instance.new("BodyVelocity")
    flyBodyVelocity.MaxForce = Vector3.new(100000, 100000, 100000)
    flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
    flyBodyVelocity.Parent = humanoidRootPart
    
    flyBodyGyro = Instance.new("BodyGyro")
    flyBodyGyro.MaxTorque = Vector3.new(400000, 400000, 400000)
    flyBodyGyro.CFrame = humanoidRootPart.CFrame
    flyBodyGyro.Parent = humanoidRootPart
    
    flySound:Play()
    
    coroutine.wrap(function()
        while isFlying and humanoidRootPart and humanoid do
            updateFlyMovement()
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.GettingUp, false)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Landed, false)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Freefall, false)
            humanoid.PlatformStand = true
            RunService.Heartbeat:Wait()
        end
    end)()
end

local function stopFly()
    if not isFlying then return end
    
    isFlying = false
    flyStatusLabel.Text = "STATUS: DESLIGADO"
    flyStatusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    flyButton.Text = "🕊️ ATIVAR VOO"
    flyButton.BackgroundColor3 = Color3.fromRGB(66, 200, 100)
    flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
    flyCardStroke.Color = Color3.fromRGB(66, 200, 100)
    
    if isMobile then
        mobileFlyControls.Visible = false
        flyUp = false
        flyDown = false
    end
    
    Workspace.Gravity = defaultGravity
    
    if flyBodyVelocity then flyBodyVelocity:Destroy() end
    if flyBodyGyro then flyBodyGyro:Destroy() end
    
    if humanoid then
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.GettingUp, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Landed, true)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Freefall, true)
        humanoid.PlatformStand = false
    end
    
    flySound:Stop()
end

local function toggleFly()
    if isFlying then
        stopFly()
    else
        startFly()
    end
end

if isMobile then
    flyUpButton.MouseButton1Down:Connect(function() flyUp = true end)
    flyUpButton.MouseButton1Up:Connect(function() flyUp = false end)
    flyDownButton.MouseButton1Down:Connect(function() flyDown = true end)
    flyDownButton.MouseButton1Up:Connect(function() flyDown = false end)
end

-- DETECTOR DE BOMBA
local function setupBombaDetector()
    local function onTouched(hit)
        if not character then return end
        if not hit or not hit.Parent then return end
        
        local itemName = hit.Name:lower()
        local parentName = hit.Parent and hit.Parent.Name:lower() or ""
        
        local isBomba = itemName:find("bomb") or 
                        itemName:find("bomba") or 
                        itemName:find("power") or 
                        itemName:find("speed") or
                        itemName:find("boost") or
                        parentName:find("bomb") or
                        parentName:find("bomba")
        
        if isBomba then
            activateBombaBoost()
            
            if hit:IsA("BasePart") then
                hit:Destroy()
            elseif hit.Parent and hit.Parent:IsA("Tool") then
                hit.Parent:Destroy()
            end
        end
    end
    
    local function connectDetector()
        character = player.Character
        if character then
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            if rootPart then
                rootPart.Touched:Connect(onTouched)
            end
        end
    end
    
    connectDetector()
    player.CharacterAdded:Connect(function()
        wait(0.5)
        connectDetector()
    end)
end

-- SLIDER DRAG
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
end

sliderButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        updateSliderFromMouse(input)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        updateSliderFromMouse(input)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- FUNÇÃO PARA CRIAR BOTÕES
local function createModernButton(text, color, icon, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 190, 0, 52)
    button.Text = "  " .. text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 13
    button.Font = Enum.Font.GothamSemibold
    button.BackgroundColor3 = color
    button.BorderSizePixel = 0
    button.AutoButtonColor = false
    button.TextXAlignment = Enum.TextXAlignment.Left
    button.Parent = buttonsContainer
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 12)
    btnCorner.Parent = button
    
    local btnIcon = Instance.new("ImageLabel")
    btnIcon.Size = UDim2.new(0, 28, 0, 28)
    btnIcon.Position = UDim2.new(0, 12, 0.5, -14)
    btnIcon.Image = icon
    btnIcon.ImageColor3 = Color3.fromRGB(255, 255, 255)
    btnIcon.BackgroundTransparency = 1
    btnIcon.Parent = button
    
    local btnStroke = Instance.new("UIStroke")
    btnStroke.Color = Color3.fromRGB(255, 255, 255)
    btnStroke.Thickness = 0.5
    btnStroke.Transparency = 0.8
    btnStroke.Parent = button
    
    local originalColor = color
    
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = color}):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = originalColor}):Play()
    end)
    
    button.MouseButton1Click:Connect(function()
        local clickClone = clickSound:Clone()
        clickClone.Parent = button
        clickClone:Play()
        
        TweenService:Create(button, TweenInfo.new(0.05), {Size = UDim2.new(0, 185, 0, 49)}):Play()
        wait(0.05)
        TweenService:Create(button, TweenInfo.new(0.05), {Size = UDim2.new(0, 190, 0, 52)}):Play()
        callback()
    end)
    
    return button
end

-- BOTÃO VELOCIDADE RÁPIDA
local speedToggleButton = createModernButton("⚡ VELOCIDADE RÁPIDA", Color3.fromRGB(66, 135, 245), "rbxassetid://6031094773", function()
    if isBombaBoosted then
        local notif = Instance.new("Frame")
        notif.Size = UDim2.new(0, 250, 0, 40)
        notif.Position = UDim2.new(0.5, -125, 0, -60)
        notif.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
        notif.BackgroundTransparency = 0.2
        notif.BorderSizePixel = 0
        notif.Parent = screenGui
        
        local notifCorner = Instance.new("UICorner")
        notifCorner.CornerRadius = UDim.new(0, 10)
        notifCorner.Parent = notif
        
        local notifText = Instance.new("TextLabel")
        notifText.Size = UDim2.new(1, 0, 1, 0)
        notifText.BackgroundTransparency = 1
        notifText.Text = "💣 BOOST ACTIVO! AGUARDE O FIM DO BOOST..."
        notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
        notifText.TextScaled = true
        notifText.Font = Enum.Font.GothamBold
        notifText.Parent = notif
        
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -125, 0, 20)}):Play()
        wait(2)
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -125, 0, -60), BackgroundTransparency = 1}):Play()
        wait(0.3)
        notif:Destroy()
        return
    end
    
    if isSpeedBoosted then
        updateSpeed(NORMAL_SPEED, true)
        isSpeedBoosted = false
        TweenService:Create(speedToggleButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(66, 135, 245)}):Play()
    else
        updateSpeed(FAST_SPEED, true)
        isSpeedBoosted = true
        TweenService:Create(speedToggleButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(76, 175, 80)}):Play()
    end
end)

-- BOTÃO INFORMAÇÕES
local infoButton = createModernButton("ℹ️ INFORMAÇÕES", Color3.fromRGB(156, 39, 176), "rbxassetid://6031094773", function()
    local infoFrame = Instance.new("Frame")
    infoFrame.Size = UDim2.new(0, 380, 0, 320)
    infoFrame.Position = UDim2.new(0.5, -190, 0.5, -160)
    infoFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    infoFrame.BackgroundTransparency = 0.05
    infoFrame.BorderSizePixel = 0
    infoFrame.Parent = screenGui
    
    local infoCorner = Instance.new("UICorner")
    infoCorner.CornerRadius = UDim.new(0, 16)
    infoCorner.Parent = infoFrame
    
    local infoStroke = Instance.new("UIStroke")
    infoStroke.Color = Color3.fromRGB(66, 135, 245)
    infoStroke.Thickness = 1
    infoStroke.Transparency = 0.7
    infoStroke.Parent = infoFrame
    
    local infoTitle = Instance.new("TextLabel")
    infoTitle.Size = UDim2.new(1, 0, 0, 45)
    infoTitle.Position = UDim2.new(0, 0, 0, 0)
    infoTitle.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
    infoTitle.Text = "   INFORMAÇÕES DO SISTEMA"
    infoTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    infoTitle.TextSize = 16
    infoTitle.Font = Enum.Font.GothamBold
    infoTitle.TextXAlignment = Enum.TextXAlignment.Left
    infoTitle.Parent = infoFrame
    
    local infoTitleCorner = Instance.new("UICorner")
    infoTitleCorner.CornerRadius = UDim.new(0, 16)
    infoTitleCorner.Parent = infoTitle
    
    local infoText = Instance.new("TextLabel")
    infoText.Size = UDim2.new(1, -30, 1, -60)
    infoText.Position = UDim2.new(0, 15, 0, 55)
    infoText.BackgroundTransparency = 1
    infoText.Text = [[
⚡ SISTEMA RIAN STUDIOS V4.1 ⚡

🎮 CONTROLES DE VELOCIDADE:
   • SLIDER: Ajuste a velocidade livremente
   • RÁPIDO: Alterna entre 16 ↔ 80
   • TURBO: Velocidade máxima (120)
   • 💣 BOMBA: Boost de 120 por 10s (NÃO INTERROMPÍVEL)

🕊️ SISTEMA DE VOO:
   • Botão ATIVAR VOO para voar
   • PC: WASD + ESPAÇO/SHIFT
   • MOBILE: Botões na tela + joystick
   • Velocidade acompanha o slider

👊 SISTEMA DE COMBATE:
   • Clique ESQUERDO: Combo de 3 socos
   • Dano: 15 por soco

👑 CRIADO POR RIAN
📅 VERSION 4.1 - BOOST CORRIGIDO
    ]]
    infoText.TextColor3 = Color3.fromRGB(220, 220, 220)
    infoText.TextSize = 12
    infoText.Font = Enum.Font.Gotham
    infoText.TextXAlignment = Enum.TextXAlignment.Left
    infoText.TextYAlignment = Enum.TextYAlignment.Top
    infoText.Parent = infoFrame
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 100, 0, 35)
    closeBtn.Position = UDim2.new(0.5, -50, 1, -50)
    closeBtn.Text = "FECHAR"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = 14
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = infoFrame
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 8)
    closeCorner.Parent = closeBtn
    
    closeBtn.MouseButton1Click:Connect(function()
        infoFrame:Destroy()
    end)
end)

-- BOTÃO TURBO
turboButton.MouseButton1Click:Connect(function()
    if isBombaBoosted then
        local notif = Instance.new("Frame")
        notif.Size = UDim2.new(0, 250, 0, 40)
        notif.Position = UDim2.new(0.5, -125, 0, -60)
        notif.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
        notif.BackgroundTransparency = 0.2
        notif.BorderSizePixel = 0
        notif.Parent = screenGui
        
        local notifCorner = Instance.new("UICorner")
        notifCorner.CornerRadius = UDim.new(0, 10)
        notifCorner.Parent = notif
        
        local notifText = Instance.new("TextLabel")
        notifText.Size = UDim2.new(1, 0, 1, 0)
        notifText.BackgroundTransparency = 1
        notifText.Text = "💣 BOOST ACTIVO! AGUARDE..."
        notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
        notifText.TextScaled = true
        notifText.Font = Enum.Font.GothamBold
        notifText.Parent = notif
        
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -125, 0, 20)}):Play()
        wait(1.5)
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -125, 0, -60), BackgroundTransparency = 1}):Play()
        wait(0.3)
        notif:Destroy()
        return
    end
    
    updateSpeed(MAX_SPEED, true)
    isSpeedBoosted = true
    TweenService:Create(speedToggleButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(76, 175, 80)}):Play()
    
    local turboEffect = Instance.new("Frame")
    turboEffect.Size = UDim2.new(0, 320, 0, 60)
    turboEffect.Position = UDim2.new(0.5, -160, 0.5, -30)
    turboEffect.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
    turboEffect.BackgroundTransparency = 0.2
    turboEffect.BorderSizePixel = 0
    turboEffect.Parent = screenGui
    
    local turboEffectCorner = Instance.new("UICorner")
    turboEffectCorner.CornerRadius = UDim.new(0, 12)
    turboEffectCorner.Parent = turboEffect
    
    local turboText = Instance.new("TextLabel")
    turboText.Size = UDim2.new(1, 0, 1, 0)
    turboText.BackgroundTransparency = 1
    turboText.Text = "🚀 TURBO ACTIVADO! 🚀"
    turboText.TextColor3 = Color3.fromRGB(255, 255, 255)
    turboText.TextScaled = true
    turboText.Font = Enum.Font.GothamBold
    turboText.Parent = turboEffect
    
    TweenService:Create(turboEffect, TweenInfo.new(0.3), {BackgroundTransparency = 0.8, Position = UDim2.new(0.5, -160, 0.3, -30)}):Play()
    wait(2)
    TweenService:Create(turboEffect, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
    wait(0.3)
    turboEffect:Destroy()
end)

-- BOTÃO DE VOO
flyButton.MouseButton1Click:Connect(function()
    local clickClone = clickSound:Clone()
    clickClone.Parent = flyButton
    clickClone:Play()
    
    TweenService:Create(flyButton, TweenInfo.new(0.05), {Size = UDim2.new(0, 125, 0, 39)}):Play()
    wait(0.05)
    TweenService:Create(flyButton, TweenInfo.new(0.05), {Size = UDim2.new(0, 130, 0, 42)}):Play()
    
    toggleFly()
end)

-- ANIMAÇÃO DE SOCO
local function playPunchAnimation(character)
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    local animation = Instance.new("Animation")
    animation.AnimationId = "rbxassetid://131453564"
    local animTrack = humanoid:LoadAnimation(animation)
    if animTrack then
        animTrack:Play()
        wait(0.2)
        animTrack:Stop()
    end
end

-- SISTEMA DE ATAQUE
local function startComboAttack()
    if isAttacking then return end
    isAttacking = true
    comboCount = 0
    
    local function performPunch()
        if not canPunch then return end
        if comboCount >= 3 then
            isAttacking = false
            comboCount = 0
            return
        end
        
        canPunch = false
        comboCount = comboCount + 1
        
        local character = player.Character
        local humanoid = character and character:FindFirstChild("Humanoid")
        
        if character and humanoid then
            local soundClone = punchSound:Clone()
            soundClone.Parent = character
            soundClone:Play()
            
            playPunchAnimation(character)
            
            local impactEffect = Instance.new("Part")
            impactEffect.Shape = Enum.PartType.Ball
            impactEffect.Size = Vector3.new(1, 1, 1)
            impactEffect.BrickColor = BrickColor.new("Bright red")
            impactEffect.Material = Enum.Material.Neon
            impactEffect.CFrame = character:GetPivot() + character:GetPivot().LookVector * 3
            impactEffect.Anchored = true
            impactEffect.CanCollide = false
            impactEffect.Parent = workspace
            
            TweenService:Create(impactEffect, TweenInfo.new(0.3), {Size = Vector3.new(0, 0, 0)}):Play()
            game:GetService("Debris"):AddItem(impactEffect, 0.3)
            
            for _, enemy in pairs(workspace:GetChildren()) do
                if enemy:IsA("Model") and enemy ~= character then
                    local enemyHumanoid = enemy:FindFirstChild("Humanoid")
                    if enemyHumanoid and enemyHumanoid.Health > 0 then
                        local distance = (character:GetPivot().Position - enemy:GetPivot().Position).Magnitude
                        if distance < 5 then
                            enemyHumanoid:TakeDamage(PUNCH_DAMAGE)
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

-- CLIQUE DO MOUSE
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local character = player.Character
        if character and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
            startComboAttack()
        end
    end
    
    if input.UserInputType == Enum.UserInputType.Touch then
        local character = player.Character
        if character and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
            local guiObjects = screenGui:GetDescendants()
            local hitButton = false
            for _, obj in pairs(guiObjects) do
                if obj:IsA("TextButton") or obj:IsA("ImageButton") then
                    if obj.AbsoluteSize.X > 0 and obj.Visible then
                        local mousePos = input.Position
                        local absPos = obj.AbsolutePosition
                        if mousePos.X >= absPos.X and mousePos.X <= absPos.X + obj.AbsoluteSize.X and
                           mousePos.Y >= absPos.Y and mousePos.Y <= absPos.Y + obj.AbsoluteSize.Y then
                            hitButton = true
                            break
                        end
                    end
                end
            end
            if not hitButton then
                startComboAttack()
            end
        end
    end
end)

-- ATUALIZAR REFERÊNCIA DO PERSONAGEM
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    
    if isFlying then
        stopFly()
    end
    
    if isBombaBoosted then
        isBombaBoosted = false
        bombTimerLabel.Visible = false
    end
    
    currentSpeed = NORMAL_SPEED
    isSpeedBoosted = false
    applySpeedToCharacter()
    
    TweenService:Create(speedToggleButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(66, 135, 245)}):Play()
    TweenService:Create(speedIcon, TweenInfo.new(0.3), {ImageColor3 = Color3.fromRGB(66, 135, 245)}):Play()
    TweenService:Create(speedCardStroke, TweenInfo.new(0.3), {Color = Color3.fromRGB(66, 135, 245)}):Play()
    sliderFill.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
    speedLabel.Text = "⚡ VELOCIDADE ATUAL"
    speedValueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
end)

-- FUNÇÃO ABRIR/FECHAR UI
local isUIOpen = false

local function toggleUI()
    isUIOpen = not isUIOpen
    
    if isUIOpen then
        mainContainer.Visible = true
        mainContainer.BackgroundTransparency = 1
        mainContainer.Position = UDim2.new(0.5, -225, 0.5, -300)
        TweenService:Create(mainContainer, TweenInfo.new(0.4, Enum.EasingStyle.Back), {BackgroundTransparency = 0.05}):Play()
        TweenService:Create(mainContainer, TweenInfo.new(0.4, Enum.EasingStyle.Back), {Position = UDim2.new(0.5, -225, 0.5, -350)}):Play()
        TweenService:Create(floatingButton, TweenInfo.new(0.3), {Size = UDim2.new(0, 60, 0, 60), ImageColor3 = Color3.fromRGB(100, 200, 255)}):Play()
    else
        TweenService:Create(mainContainer, TweenInfo.new(0.3), {BackgroundTransparency = 1, Position = UDim2.new(0.5, -225, 0.5, -300)}):Play()
        wait(0.3)
        mainContainer.Visible = false
        TweenService:Create(floatingButton, TweenInfo.new(0.3), {Size = UDim2.new(0, 70, 0, 70), ImageColor3 = Color3.fromRGB(255, 255, 255)}):Play()
    end
end

floatingButton.MouseButton1Click:Connect(toggleUI)
closeButton.MouseButton1Click:Connect(toggleUI)

-- INICIALIZAR
setupBombaDetector()
applySpeedToCharacter()

-- Animação do botão flutuante
coroutine.wrap(function()
    while true do
        wait(1.5)
        if not isUIOpen then
            TweenService:Create(floatingButton, TweenInfo.new(0.5, Enum.EasingStyle.Sine), {Size = UDim2.new(0, 75, 0, 75)}):Play()
            wait(0.25)
            TweenService:Create(floatingButton, TweenInfo.new(0.5, Enum.EasingStyle.Sine), {Size = UDim2.new(0, 70, 0, 70)}):Play()
        end
    end
end)()

print("════════════════════════════════════════════════════════════════")
print("✅ SISTEMA RIAN STUDIOS V4.1 CARREGADO COM SUCESSO!")
print("✅ BOOST DA BOMBA CORRIGIDO - AGORA MANTÉM VELOCIDADE MÁXIMA!")
print("✅ FUNCIONALIDADES: VELOCIDADE | VOO MOBILE | BOOST | COMBATE")
print("════════════════════════════════════════════════════════════════")
