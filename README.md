--[[
    ╔═══════════════════════════════════════════════════════════════════════╗
    ║                                                                       ║
    ║     ██████╗ ██╗ █████╗ ███╗   ██╗    ███████╗████████╗██╗   ██╗██████╗ ║
    ║     ██╔══██╗██║██╔══██╗████╗  ██║    ██╔════╝╚══██╔══╝██║   ██║██╔══██╗║
    ║     ██████╔╝██║███████║██╔██╗ ██║    ███████╗   ██║   ██║   ██║██║  ██║║
    ║     ██╔══██╗██║██╔══██║██║╚██╗██║    ╚════██║   ██║   ██║   ██║██║  ██║║
    ║     ██║  ██║██║██║  ██║██║ ╚████║    ███████║   ██║   ╚██████╔╝██████╔╝║
    ║     ╚═╝  ╚═╝╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝    ╚══════╝   ╚═╝    ╚═════╝ ╚═════╝ ║
    ║                                                                       ║
    ║              SISTEMA V6.0 - OPTIMIZADO PARA MOBILE                    ║
    ║         VELOCIDADE | VOO | BOOST | COMBATE | UI RESPONSIVA            ║
    ║                                                                       ║
    ╚═══════════════════════════════════════════════════════════════════════╝
]]

-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

-- Variáveis do jogador
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- Detecta mobile e tamanho da tela
local isMobile = UserInputService.TouchEnabled
local screenSize = player:GetMouse().ViewSizeX
local isSmallScreen = screenSize < 800

-- Configurações
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

-- CRIAR UI RESPONSIVA
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RianStudiosUI"
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false

-- BOTÃO FLUTUANTE (RESPONSIVO)
local floatingButton = Instance.new("ImageButton")
floatingButton.Name = "FloatingButton"
floatingButton.Size = isMobile and UDim2.new(0, 65, 0, 65) or UDim2.new(0, 55, 0, 55)
floatingButton.Position = isMobile and UDim2.new(0.85, 0, 0.88, 0) or UDim2.new(0.02, 0, 0.85, 0)
floatingButton.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
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

-- CONTAINER PRINCIPAL (RESPONSIVO)
local mainContainer = Instance.new("Frame")
mainContainer.Name = "MainContainer"
mainContainer.Size = isMobile and UDim2.new(0.9, 0, 0.85, 0) or UDim2.new(0, 340, 0, 500)
mainContainer.Position = isMobile and UDim2.new(0.05, 0, 0.075, 0) or UDim2.new(0.5, -170, 0.5, -250)
mainContainer.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
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
mainBorder.Transparency = 0.6
mainBorder.Parent = mainContainer

-- Efeito de blur no fundo (mobile)
if isMobile then
    local backgroundBlur = Instance.new("BlurEffect")
    backgroundBlur.Size = 0
    backgroundBlur.Parent = game:GetService("Lighting")
end

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, isMobile and 55 or 45)
titleBar.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainContainer

local titleBarCorner = Instance.new("UICorner")
titleBarCorner.CornerRadius = UDim.new(0, 20)
titleBarCorner.Parent = titleBar

local titleGradient = Instance.new("UIGradient")
titleGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(66, 135, 245)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(100, 80, 200))
})
titleGradient.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -60, 1, 0)
titleLabel.Position = UDim2.new(0, 15, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚡ RIAN STUDIOS"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = isMobile and 16 or 14
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local closeButton = Instance.new("ImageButton")
closeButton.Size = UDim2.new(0, isMobile and 32 or 28, 0, isMobile and 32 or 28)
closeButton.Position = UDim2.new(1, isMobile and -42 or -38, 0, isMobile and 12 or 8)
closeButton.Image = "rbxassetid://3926305904"
closeButton.ImageColor3 = Color3.fromRGB(255, 255, 255)
closeButton.BackgroundTransparency = 1
closeButton.Parent = titleBar

-- CARD DE VELOCIDADE
local speedCard = Instance.new("Frame")
speedCard.Size = UDim2.new(1, -20, 0, isMobile and 140 or 120)
speedCard.Position = UDim2.new(0, 10, 0, isMobile and 65 or 55)
speedCard.BackgroundColor3 = Color3.fromRGB(25, 25, 38)
speedCard.BackgroundTransparency = 0.2
speedCard.BorderSizePixel = 0
speedCard.Parent = mainContainer

local speedCardCorner = Instance.new("UICorner")
speedCardCorner.CornerRadius = UDim.new(0, 14)
speedCardCorner.Parent = speedCard

local speedCardStroke = Instance.new("UIStroke")
speedCardStroke.Color = Color3.fromRGB(66, 135, 245)
speedCardStroke.Thickness = 0.5
speedCardStroke.Transparency = 0.7
speedCardStroke.Parent = speedCard

-- Ícone
local speedIcon = Instance.new("ImageLabel")
speedIcon.Size = UDim2.new(0, isMobile and 40 or 35, 0, isMobile and 40 or 35)
speedIcon.Position = UDim2.new(0, isMobile and 15 or 12, 0, isMobile and 15 or 12)
speedIcon.Image = "rbxassetid://6031094773"
speedIcon.ImageColor3 = Color3.fromRGB(66, 135, 245)
speedIcon.BackgroundTransparency = 1
speedIcon.Parent = speedCard

-- Valor da velocidade
local speedValueLabel = Instance.new("TextLabel")
speedValueLabel.Size = UDim2.new(1, -70, 0, isMobile and 45 or 40)
speedValueLabel.Position = UDim2.new(0, isMobile and 65 or 55, 0, isMobile and 8 or 5)
speedValueLabel.BackgroundTransparency = 1
speedValueLabel.Text = math.floor(currentSpeed)
speedValueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedValueLabel.TextSize = isMobile and 32 or 28
speedValueLabel.Font = Enum.Font.GothamBold
speedValueLabel.TextXAlignment = Enum.TextXAlignment.Left
speedValueLabel.Parent = speedCard

local speedMaxLabel = Instance.new("TextLabel")
speedMaxLabel.Size = UDim2.new(1, -70, 0, 25)
speedMaxLabel.Position = UDim2.new(0, isMobile and 65 or 55, 0, isMobile and 50 or 42)
speedMaxLabel.BackgroundTransparency = 1
speedMaxLabel.Text = "/ " .. MAX_SPEED
speedMaxLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
speedMaxLabel.TextSize = isMobile and 16 or 14
speedMaxLabel.Font = Enum.Font.Gotham
speedMaxLabel.TextXAlignment = Enum.TextXAlignment.Left
speedMaxLabel.Parent = speedCard

-- Timer da bomba
local bombTimerLabel = Instance.new("TextLabel")
bombTimerLabel.Size = UDim2.new(1, -70, 0, 20)
bombTimerLabel.Position = UDim2.new(0, isMobile and 65 or 55, 0, isMobile and 72 or 62)
bombTimerLabel.BackgroundTransparency = 1
bombTimerLabel.Text = ""
bombTimerLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
bombTimerLabel.TextSize = isMobile and 12 or 10
bombTimerLabel.Font = Enum.Font.GothamBold
bombTimerLabel.TextXAlignment = Enum.TextXAlignment.Left
bombTimerLabel.Visible = false
bombTimerLabel.Parent = speedCard

-- SLIDER
local sliderBg = Instance.new("Frame")
sliderBg.Size = UDim2.new(0.88, 0, 0, isMobile and 8 or 6)
sliderBg.Position = UDim2.new(0.06, 0, 0.85, 0)
sliderBg.BackgroundColor3 = Color3.fromRGB(45, 45, 58)
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
sliderButton.Size = UDim2.new(0, isMobile and 24 or 20, 0, isMobile and 24 or 20)
sliderButton.Position = UDim2.new((currentSpeed - MIN_SPEED) / (MAX_SPEED - MIN_SPEED), -12, 0.5, -12)
sliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
sliderButton.Image = "rbxassetid://266788897"
sliderButton.ScaleType = Enum.ScaleType.Fit
sliderButton.BackgroundTransparency = 1
sliderButton.Parent = speedCard

-- CARD DE VOO
local flyCard = Instance.new("Frame")
flyCard.Size = UDim2.new(1, -20, 0, isMobile and 95 or 85)
flyCard.Position = UDim2.new(0, 10, 0, isMobile and 215 or 185)
flyCard.BackgroundColor3 = Color3.fromRGB(25, 25, 38)
flyCard.BackgroundTransparency = 0.2
flyCard.BorderSizePixel = 0
flyCard.Parent = mainContainer

local flyCardCorner = Instance.new("UICorner")
flyCardCorner.CornerRadius = UDim.new(0, 14)
flyCardCorner.Parent = flyCard

local flyCardStroke = Instance.new("UIStroke")
flyCardStroke.Color = Color3.fromRGB(66, 200, 100)
flyCardStroke.Thickness = 0.5
flyCardStroke.Transparency = 0.7
flyCardStroke.Parent = flyCard

local flyIcon = Instance.new("ImageLabel")
flyIcon.Size = UDim2.new(0, isMobile and 35 or 30, 0, isMobile and 35 or 30)
flyIcon.Position = UDim2.new(0, isMobile and 15 or 12, 0, isMobile and 15 or 12)
flyIcon.Image = "rbxassetid://6031094773"
flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
flyIcon.BackgroundTransparency = 1
flyIcon.Parent = flyCard

local flyStatusLabel = Instance.new("TextLabel")
flyStatusLabel.Size = UDim2.new(1, -60, 0, 30)
flyStatusLabel.Position = UDim2.new(0, isMobile and 60 or 50, 0, isMobile and 12 or 10)
flyStatusLabel.BackgroundTransparency = 1
flyStatusLabel.Text = "VOO: DESLIGADO"
flyStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
flyStatusLabel.TextSize = isMobile and 13 or 12
flyStatusLabel.Font = Enum.Font.GothamBold
flyStatusLabel.TextXAlignment = Enum.TextXAlignment.Left
flyStatusLabel.Parent = flyCard

local flyButton = Instance.new("TextButton")
flyButton.Size = UDim2.new(0, isMobile and 120 or 100, 0, isMobile and 38 or 32)
flyButton.Position = UDim2.new(0.5, isMobile and -60 or -50, 1, -12)
flyButton.Text = "🕊️ VOAR"
flyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
flyButton.TextSize = isMobile and 14 or 12
flyButton.Font = Enum.Font.GothamBold
flyButton.BackgroundColor3 = Color3.fromRGB(66, 200, 100)
flyButton.BorderSizePixel = 0
flyButton.Parent = flyCard

local flyButtonCorner = Instance.new("UICorner")
flyButtonCorner.CornerRadius = UDim.new(0, 10)
flyButtonCorner.Parent = flyButton

-- BOTÕES RÁPIDOS (LAYOUT RESPONSIVO)
local buttonsContainer = Instance.new("Frame")
buttonsContainer.Size = UDim2.new(1, -20, 0, isMobile and 130 or 100)
buttonsContainer.Position = UDim2.new(0, 10, 0, isMobile and 320 or 280)
buttonsContainer.BackgroundTransparency = 1
buttonsContainer.Parent = mainContainer

local buttonsLayout = Instance.new("UIListLayout")
buttonsLayout.FillDirection = isMobile and Enum.FillDirection.Vertical or Enum.FillDirection.Horizontal
buttonsLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
buttonsLayout.VerticalAlignment = Enum.VerticalAlignment.Top
buttonsLayout.Padding = UDim.new(0, isMobile and 10 or 8)
buttonsLayout.Parent = buttonsContainer

-- Botão Ativar Velocidade
local autoSpeedButton = Instance.new("TextButton")
autoSpeedButton.Size = UDim2.new(isMobile and 1 or 0, 0, 0, isMobile and 45 or 40)
autoSpeedButton.Text = "🔘 ATIVAR VELOCIDADE"
autoSpeedButton.TextColor3 = Color3.fromRGB(255, 255, 255)
autoSpeedButton.TextSize = isMobile and 14 or 12
autoSpeedButton.Font = Enum.Font.GothamBold
autoSpeedButton.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
autoSpeedButton.BorderSizePixel = 0
autoSpeedButton.Parent = buttonsContainer

local autoButtonCorner = Instance.new("UICorner")
autoButtonCorner.CornerRadius = UDim.new(0, 10)
autoButtonCorner.Parent = autoSpeedButton

-- Container para botões secundários (mobile fica em linha)
local secondaryButtons = Instance.new("Frame")
secondaryButtons.Size = UDim2.new(isMobile and 1 or 0, 0, 0, isMobile and 45 or 40)
secondaryButtons.BackgroundTransparency = 1
secondaryButtons.Parent = buttonsContainer

local secondaryLayout = Instance.new("UIListLayout")
secondaryLayout.FillDirection = Enum.FillDirection.Horizontal
secondaryLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
secondaryLayout.Padding = UDim.new(0, isMobile and 10 or 8)
secondaryLayout.Parent = secondaryButtons

-- Botão Turbo
local turboButton = Instance.new("TextButton")
turboButton.Size = UDim2.new(0, isMobile and 80 or 70, 0, isMobile and 45 or 40)
turboButton.Text = "🚀"
turboButton.TextColor3 = Color3.fromRGB(255, 255, 255)
turboButton.TextSize = isMobile and 22 or 20
turboButton.Font = Enum.Font.GothamBold
turboButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
turboButton.BorderSizePixel = 0
turboButton.Parent = secondaryButtons

local turboCorner = Instance.new("UICorner")
turboCorner.CornerRadius = UDim.new(0, 10)
turboCorner.Parent = turboButton

-- Botão Info
local infoButton = Instance.new("TextButton")
infoButton.Size = UDim2.new(0, isMobile and 80 or 70, 0, isMobile and 45 or 40)
infoButton.Text = "ℹ️"
infoButton.TextColor3 = Color3.fromRGB(255, 255, 255)
infoButton.TextSize = isMobile and 22 or 20
infoButton.Font = Enum.Font.GothamBold
infoButton.BackgroundColor3 = Color3.fromRGB(156, 39, 176)
infoButton.BorderSizePixel = 0
infoButton.Parent = secondaryButtons

local infoCorner = Instance.new("UICorner")
infoCorner.CornerRadius = UDim.new(0, 10)
infoCorner.Parent = infoButton

-- Footer
local footerFrame = Instance.new("Frame")
footerFrame.Size = UDim2.new(1, 0, 0, isMobile and 35 or 30)
footerFrame.Position = UDim2.new(0, 0, 1, -35)
footerFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 18)
footerFrame.BackgroundTransparency = 0.3
footerFrame.BorderSizePixel = 0
footerFrame.Parent = mainContainer

local footerCorner = Instance.new("UICorner")
footerCorner.CornerRadius = UDim.new(0, 14)
footerCorner.Parent = footerFrame

local creditLabel = Instance.new("TextLabel")
creditLabel.Size = UDim2.new(1, 0, 0, 18)
creditLabel.Position = UDim2.new(0, 0, 0, 4)
creditLabel.BackgroundTransparency = 1
creditLabel.Text = "⚡ RIAN STUDIOS ⚡"
creditLabel.TextColor3 = Color3.fromRGB(120, 120, 120)
creditLabel.TextSize = isMobile and 10 or 9
creditLabel.Font = Enum.Font.Gotham
creditLabel.Parent = footerFrame

local versionLabel = Instance.new("TextLabel")
versionLabel.Size = UDim2.new(1, 0, 0, 14)
versionLabel.Position = UDim2.new(0, 0, 0, 18)
versionLabel.BackgroundTransparency = 1
versionLabel.Text = "V6.0 | MOBILE OPTIMIZED"
versionLabel.TextColor3 = Color3.fromRGB(80, 80, 80)
versionLabel.TextSize = isMobile and 8 or 7
versionLabel.Font = Enum.Font.Gotham
versionLabel.Parent = footerFrame

-- CONTROLES MOBILE PARA VOO (MELHORADOS)
local mobileFlyControls = Instance.new("Frame")
mobileFlyControls.Size = UDim2.new(0, 200, 0, 100)
mobileFlyControls.Position = UDim2.new(0.5, -100, 1, -120)
mobileFlyControls.BackgroundColor3 = Color3.fromRGB(25, 25, 38)
mobileFlyControls.BackgroundTransparency = 0.15
mobileFlyControls.BorderSizePixel = 0
mobileFlyControls.Visible = false
mobileFlyControls.Parent = screenGui

local mobileControlsCorner = Instance.new("UICorner")
mobileControlsCorner.CornerRadius = UDim.new(0, 16)
mobileControlsCorner.Parent = mobileFlyControls

local mobileControlsStroke = Instance.new("UIStroke")
mobileControlsStroke.Color = Color3.fromRGB(66, 200, 100)
mobileControlsStroke.Thickness = 1
mobileControlsStroke.Transparency = 0.5
mobileControlsStroke.Parent = mobileFlyControls

local flyUpButton = Instance.new("TextButton")
flyUpButton.Size = UDim2.new(0, 80, 0, 80)
flyUpButton.Position = UDim2.new(0.1, 0, 0.1, 0)
flyUpButton.Text = "⬆️"
flyUpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
flyUpButton.TextSize = 40
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
flyDownButton.TextSize = 40
flyDownButton.Font = Enum.Font.GothamBold
flyDownButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
flyDownButton.BorderSizePixel = 0
flyDownButton.Parent = mobileFlyControls

local flyDownCorner = Instance.new("UICorner")
flyDownCorner.CornerRadius = UDim.new(0, 40)
flyDownCorner.Parent = flyDownButton

local mobileInstruction = Instance.new("TextLabel")
mobileInstruction.Size = UDim2.new(1, 0, 0, 20)
mobileInstruction.Position = UDim2.new(0, 0, 1, -25)
mobileInstruction.BackgroundTransparency = 1
mobileInstruction.Text = "Subir / Descer"
mobileInstruction.TextColor3 = Color3.fromRGB(200, 200, 200)
mobileInstruction.TextSize = 11
mobileInstruction.Font = Enum.Font.Gotham
mobileInstruction.Parent = mobileFlyControls

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
    sliderButton:TweenPosition(UDim2.new(percent, -12, 0.5, -12), "Out", "Quad", 0.1, true)
end

local function updateSpeed(newSpeed, ignoreBoostLock)
    if isBombaBoosted and not ignoreBoostLock then return end
    currentSpeed = math.clamp(newSpeed, MIN_SPEED, MAX_SPEED)
    applySpeedToCharacter()
end

-- BOTÃO DE ATIVAÇÃO AUTOMÁTICA
autoSpeedButton.MouseButton1Click:Connect(function()
    local clickClone = clickSound:Clone()
    clickClone.Parent = autoSpeedButton
    clickClone:Play()
    
    if not speedActivated then
        speedActivated = true
        updateSpeed(FAST_SPEED, true)
        isSpeedBoosted = true
        
        autoSpeedButton.Text = "✅ VELOCIDADE ATIVA"
        autoSpeedButton.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
        
        TweenService:Create(autoSpeedButton, TweenInfo.new(0.1), {Size = UDim2.new(isMobile and 1 or 0, 0, 0, isMobile and 48 or 43)}):Play()
        wait(0.1)
        TweenService:Create(autoSpeedButton, TweenInfo.new(0.1), {Size = UDim2.new(isMobile and 1 or 0, 0, 0, isMobile and 45 or 40)}):Play()
        
        local notif = Instance.new("Frame")
        notif.Size = UDim2.new(0, 220, 0, 40)
        notif.Position = UDim2.new(0.5, -110, 0, -50)
        notif.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
        notif.BackgroundTransparency = 0.1
        notif.BorderSizePixel = 0
        notif.Parent = screenGui
        
        local notifCorner = Instance.new("UICorner")
        notifCorner.CornerRadius = UDim.new(0, 10)
        notifCorner.Parent = notif
        
        local notifText = Instance.new("TextLabel")
        notifText.Size = UDim2.new(1, 0, 1, 0)
        notifText.BackgroundTransparency = 1
        notifText.Text = "⚡ VELOCIDADE ATIVADA! ⚡"
        notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
        notifText.TextScaled = true
        notifText.Font = Enum.Font.GothamBold
        notifText.Parent = notif
        
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -110, 0, 20)}):Play()
        wait(2)
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -110, 0, -50), BackgroundTransparency = 1}):Play()
        wait(0.3)
        notif:Destroy()
    else
        speedActivated = false
        updateSpeed(NORMAL_SPEED, true)
        isSpeedBoosted = false
        
        autoSpeedButton.Text = "🔘 ATIVAR VELOCIDADE"
        autoSpeedButton.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
        
        TweenService:Create(autoSpeedButton, TweenInfo.new(0.1), {Size = UDim2.new(isMobile and 1 or 0, 0, 0, isMobile and 48 or 43)}):Play()
        wait(0.1)
        TweenService:Create(autoSpeedButton, TweenInfo.new(0.1), {Size = UDim2.new(isMobile and 1 or 0, 0, 0, isMobile and 45 or 40)}):Play()
        
        local notif = Instance.new("Frame")
        notif.Size = UDim2.new(0, 220, 0, 40)
        notif.Position = UDim2.new(0.5, -110, 0, -50)
        notif.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
        notif.BackgroundTransparency = 0.1
        notif.BorderSizePixel = 0
        notif.Parent = screenGui
        
        local notifCorner = Instance.new("UICorner")
        notifCorner.CornerRadius = UDim.new(0, 10)
        notifCorner.Parent = notif
        
        local notifText = Instance.new("TextLabel")
        notifText.Size = UDim2.new(1, 0, 1, 0)
        notifText.BackgroundTransparency = 1
        notifText.Text = "⚡ VELOCIDADE DESATIVADA ⚡"
        notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
        notifText.TextScaled = true
        notifText.Font = Enum.Font.GothamBold
        notifText.Parent = notif
        
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -110, 0, 20)}):Play()
        wait(2)
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -110, 0, -50), BackgroundTransparency = 1}):Play()
        wait(0.3)
        notif:Destroy()
    end
end)

-- BOOST DA BOMBA
local function activateBombaBoost()
    if isBombaBoosted then return end
    
    isBombaBoosted = true
    isSpeedBoosted = true
    speedActivated = true
    
    autoSpeedButton.Text = "✅ VELOCIDADE ATIVA"
    autoSpeedButton.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
    
    local soundClone = boostSound:Clone()
    soundClone.Parent = character
    soundClone:Play()
    
    currentSpeed = BOMBA_SPEED
    applySpeedToCharacter()
    
    TweenService:Create(speedIcon, TweenInfo.new(0.3), {ImageColor3 = Color3.fromRGB(255, 100, 100)}):Play()
    TweenService:Create(speedCardStroke, TweenInfo.new(0.3), {Color = Color3.fromRGB(255, 100, 100)}):Play()
    
    speedValueLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
    sliderFill.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
    bombTimerLabel.Visible = true
    
    local glowEffect = Instance.new("PointLight")
    glowEffect.Color = Color3.fromRGB(255, 100, 0)
    glowEffect.Range = 10
    glowEffect.Brightness = 2
    glowEffect.Parent = humanoidRootPart or character
    
    local particleEffect = Instance.new("ParticleEmitter")
    particleEffect.Texture = "rbxassetid://284646226"
    particleEffect.Rate = 50
    particleEffect.SpreadAngle = Vector2.new(360, 360)
    particleEffect.Lifetime = NumberRange.new(0.5)
    particleEffect.Speed = NumberRange.new(5)
    particleEffect.Color = ColorSequence.new(Color3.fromRGB(255, 100, 0))
    particleEffect.Parent = humanoidRootPart or character
    
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0, 260, 0, 50)
    notif.Position = UDim2.new(0.5, -130, 0, -50)
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
    notifText.Text = "💣 BOOST DE VELOCIDADE! 💣"
    notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
    notifText.TextScaled = true
    notifText.Font = Enum.Font.GothamBold
    notifText.Parent = notif
    
    TweenService:Create(notif, TweenInfo.new(0.4), {Position = UDim2.new(0.5, -130, 0, 20)}):Play()
    
    local startTime = tick()
    local timerConnection
    
    timerConnection = RunService.RenderStepped:Connect(function()
        if not isBombaBoosted then
            if timerConnection then timerConnection:Disconnect() end
            return
        end
        
        local elapsed = tick() - startTime
        local remaining = math.max(0, 10 - elapsed)
        bombTimerLabel.Text = "💣 " .. math.ceil(remaining) .. "s"
        
        if remaining <= 0 then
            timerConnection:Disconnect()
            
            isBombaBoosted = false
            isSpeedBoosted = false
            
            if not speedActivated then
                currentSpeed = NORMAL_SPEED
                autoSpeedButton.Text = "🔘 ATIVAR VELOCIDADE"
                autoSpeedButton.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
            else
                currentSpeed = FAST_SPEED
            end
            
            applySpeedToCharacter()
            
            bombTimerLabel.Visible = false
            speedValueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
            sliderFill.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
            TweenService:Create(speedIcon, TweenInfo.new(0.3), {ImageColor3 = Color3.fromRGB(66, 135, 245)}):Play()
            TweenService:Create(speedCardStroke, TweenInfo.new(0.3), {Color = Color3.fromRGB(66, 135, 245)}):Play()
            
            glowEffect:Destroy()
            particleEffect:Destroy()
            
            local endNotif = Instance.new("Frame")
            endNotif.Size = UDim2.new(0, 240, 0, 40)
            endNotif.Position = UDim2.new(0.5, -120, 0, -50)
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
            endText.Text = "⚡ BOOST FINALIZADO ⚡"
            endText.TextColor3 = Color3.fromRGB(255, 255, 255)
            endText.TextScaled = true
            endText.Font = Enum.Font.GothamBold
            endText.Parent = endNotif
            
            TweenService:Create(endNotif, TweenInfo.new(0.4), {Position = UDim2.new(0.5, -120, 0, 20)}):Play()
            wait(2)
            TweenService:Create(endNotif, TweenInfo.new(0.4), {Position = UDim2.new(0.5, -120, 0, -50), BackgroundTransparency = 1}):Play()
            wait(0.3)
            endNotif:Destroy()
        end
    end)
    
    wait(2)
    TweenService:Create(notif, TweenInfo.new(0.4), {Position = UDim2.new(0.5, -130, 0, -50), BackgroundTransparency = 1}):Play()
    wait(0.3)
    notif:Destroy()
end

-- SISTEMA DE VOO
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
        if flyUp then moveDirection = moveDirection + Vector3.new(0, 1, 0) end
        if flyDown then moveDirection = moveDirection - Vector3.new(0, 1, 0) end
        if humanoid and humanoid.MoveDirection.Magnitude > 0 then
            moveDirection = moveDirection + humanoid.MoveDirection
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
    flyStatusLabel.Text = "VOO: 🕊️ VOANDO"
    flyStatusLabel.TextColor3 = Color3.fromRGB(66, 200, 100)
    flyButton.Text = "🛑 PARAR"
    flyButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
    flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
    flyCardStroke.Color = Color3.fromRGB(66, 200, 100)
    
    if isMobile then mobileFlyControls.Visible = true end
    
    Workspace.Gravity = 0
    
    flyBodyVelocity = Instance.new("BodyVelocity")
    flyBodyVelocity.MaxForce = Vector3.new(100000, 100000, 100000)
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
            humanoid.PlatformStand = true
            RunService.Heartbeat:Wait()
        end
    end)()
end

local function stopFly()
    if not isFlying then return end
    
    isFlying = false
    flyStatusLabel.Text = "VOO: DESLIGADO"
    flyStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    flyButton.Text = "🕊️ VOAR"
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
        humanoid.PlatformStand = false
    end
    
    flySound:Stop()
end

local function toggleFly()
    if isFlying then stopFly() else startFly() end
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
        if not character or not hit or not hit.Parent then return end
        
        local itemName = hit.Name:lower()
        local parentName = hit.Parent and hit.Parent.Name:lower() or ""
        
        local isBomba = itemName:find("bomb") or itemName:find("bomba") or 
                        itemName:find("power") or itemName:find("speed") or
                        itemName:find("boost") or parentName:find("bomb")
        
        if isBomba then
            activateBombaBoost()
            if hit:IsA("BasePart") then hit:Destroy()
            elseif hit.Parent and hit.Parent:IsA("Tool") then hit.Parent:Destroy() end
        end
    end
    
    local function connectDetector()
        character = player.Character
        if character then
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            if rootPart then rootPart.Touched:Connect(onTouched) end
        end
    end
    
    connectDetector()
    player.CharacterAdded:Connect(function() wait(0.5); connectDetector() end)
end

-- SLIDER
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
        autoSpeedButton.Text = "✅ VELOCIDADE ATIVA"
        autoSpeedButton.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
    else
        autoSpeedButton.Text = "🔘 ATIVAR VELOCIDADE"
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

-- BOTÕES
turboButton.MouseButton1Click:Connect(function()
    if isBombaBoosted then return end
    updateSpeed(MAX_SPEED, true)
    isSpeedBoosted = true
    speedActivated = true
    autoSpeedButton.Text = "✅ VELOCIDADE ATIVA"
    autoSpeedButton.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
    
    local turboEffect = Instance.new("Frame")
    turboEffect.Size = UDim2.new(0, 200, 0, 45)
    turboEffect.Position = UDim2.new(0.5, -100, 0.5, -22)
    turboEffect.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
    turboEffect.BackgroundTransparency = 0.2
    turboEffect.BorderSizePixel = 0
    turboEffect.Parent = screenGui
    
    local turboCorner = Instance.new("UICorner")
    turboCorner.CornerRadius = UDim.new(0, 12)
    turboCorner.Parent = turboEffect
    
    local turboText = Instance.new("TextLabel")
    turboText.Size = UDim2.new(1, 0, 1, 0)
    turboText.BackgroundTransparency = 1
    turboText.Text = "🚀 TURBO! 🚀"
    turboText.TextColor3 = Color3.fromRGB(255, 255, 255)
    turboText.TextScaled = true
    turboText.Font = Enum.Font.GothamBold
    turboText.Parent = turboEffect
    
    TweenService:Create(turboEffect, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -100, 0.3, -22)}):Play()
    wait(1.5)
    TweenService:Create(turboEffect, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
    wait(0.3)
    turboEffect:Destroy()
end)

infoButton.MouseButton1Click:Connect(function()
    local infoFrame = Instance.new("Frame")
    infoFrame.Size = UDim2.new(0, isMobile and 300 or 280, 0, isMobile and 250 or 220)
    infoFrame.Position = UDim2.new(0.5, isMobile and -150 or -140, 0.5, isMobile and -125 or -110)
    infoFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
    infoFrame.BackgroundTransparency = 0.05
    infoFrame.BorderSizePixel = 0
    infoFrame.Parent = screenGui
    
    local infoCorner = Instance.new("UICorner")
    infoCorner.CornerRadius = UDim.new(0, 16)
    infoCorner.Parent = infoFrame
    
    local infoStroke = Instance.new("UIStroke")
    infoStroke.Color = Color3.fromRGB(66, 135, 245)
    infoStroke.Thickness = 1
    infoStroke.Transparency = 0.6
    infoStroke.Parent = infoFrame
    
    local infoTitle = Instance.new("TextLabel")
    infoTitle.Size = UDim2.new(1, 0, 0, 40)
    infoTitle.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
    infoTitle.Text = "   INFORMAÇÕES"
    infoTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    infoTitle.TextSize = isMobile and 16 or 14
    infoTitle.Font = Enum.Font.GothamBold
    infoTitle.TextXAlignment = Enum.TextXAlignment.Left
    infoTitle.Parent = infoFrame
    
    local infoTitleCorner = Instance.new("UICorner")
    infoTitleCorner.CornerRadius = UDim.new(0, 16)
    infoTitleCorner.Parent = infoTitle
    
    local infoText = Instance.new("TextLabel")
    infoText.Size = UDim2.new(1, -20, 1, -60)
    infoText.Position = UDim2.new(0, 10, 0, 50)
    infoText.BackgroundTransparency = 1
    infoText.Text = [[⚡ RIAN STUDIOS V6.0 ⚡

🎮 VELOCIDADE:
   • ATIVAR: 80 (mantém ativo)
   • Slider: Ajuste 16-120
   • TURBO: Máximo instantâneo

🕊️ VOO: WASD + ESPAÇO/SHIFT
💣 BOOST: Pegue bombas!
👊 ATAQUE: Toque/clique esquerdo

📱 OTIMIZADO PARA MOBILE
    ]]
    infoText.TextColor3 = Color3.fromRGB(220, 220, 220)
    infoText.TextSize = isMobile and 11 or 10
    infoText.Font = Enum.Font.Gotham
    infoText.TextXAlignment = Enum.TextXAlignment.Left
    infoText.TextYAlignment = Enum.TextYAlignment.Top
    infoText.Parent = infoFrame
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 70, 0, 30)
    closeBtn.Position = UDim2.new(0.5, -35, 1, -40)
    closeBtn.Text = "FECHAR"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = isMobile and 13 or 12
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = infoFrame
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 8)
    closeCorner.Parent = closeBtn
    
    closeBtn.MouseButton1Click:Connect(function() infoFrame:Destroy() end)
end)

flyButton.MouseButton1Click:Connect(function()
    local clickClone = clickSound:Clone()
    clickClone.Parent = flyButton
    clickClone:Play()
    toggleFly()
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
            punchSound:Clone().Parent = char
            punchSound:Play()
            playPunchAnimation()
            
            local impact = Instance.new("Part")
            impact.Shape = Enum.PartType.Ball
            impact.Size = Vector3.new(0.8, 0.8, 0.8)
            impact.BrickColor = BrickColor.new("Bright red")
            impact.Material = Enum.Material.Neon
            impact.CFrame = char:GetPivot() + char:GetPivot().LookVector * 3
            impact.Anchored = true
            impact.CanCollide = false
            impact.Parent = workspace
            
            TweenService:Create(impact, TweenInfo.new(0.3), {Size = Vector3.new(0, 0, 0)}):Play()
            game:GetService("Debris"):AddItem(impact, 0.3)
            
            for _, enemy in pairs(workspace:GetChildren()) do
                if enemy:IsA("Model") and enemy ~= char then
                    local enemyHum = enemy:FindFirstChild("Humanoid")
                    if enemyHum and enemyHum.Health > 0 then
                        local dist = (char:GetPivot().Position - enemy:GetPivot().Position).Magnitude
                        if dist < 5 then enemyHum:TakeDamage(PUNCH_DAMAGE) end
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
    
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        local char = player.Character
        if char and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0 then
            -- Verificar se não clicou em botão
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

-- RESPAWN
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    
    if isFlying then stopFly() end
    if isBombaBoosted then isBombaBoosted = false end
    
    currentSpeed = speedActivated and FAST_SPEED or NORMAL_SPEED
    applySpeedToCharacter()
    
    bombTimerLabel.Visible = false
    speedValueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    sliderFill.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
    TweenService:Create(speedIcon, TweenInfo.new(0.3), {ImageColor3 = Color3.fromRGB(66, 135, 245)}):Play()
    TweenService:Create(speedCardStroke, TweenInfo.new(0.3), {Color = Color3.fromRGB(66, 135, 245)}):Play()
end)

-- UI TOGGLE
local isUIOpen = false
local function toggleUI()
    isUIOpen = not isUIOpen
    
    if isUIOpen then
        mainContainer.Visible = true
        mainContainer.BackgroundTransparency = 1
        if not isMobile then
            mainContainer.Position = UDim2.new(0.5, -170, 0.5, -230)
        end
        TweenService:Create(mainContainer, TweenInfo.new(0.3, Enum.EasingStyle.Back), {BackgroundTransparency = 0.05}):Play()
        if not isMobile then
            TweenService:Create(mainContainer, TweenInfo.new(0.3, Enum.EasingStyle.Back), {Position = UDim2.new(0.5, -170, 0.5, -250)}):Play()
        end
        TweenService:Create(floatingButton, TweenInfo.new(0.2), {Size = isMobile and UDim2.new(0, 55, 0, 55) or UDim2.new(0, 50, 0, 50), ImageColor3 = Color3.fromRGB(100, 200, 255)}):Play()
    else
        TweenService:Create(mainContainer, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
        if not isMobile then
            TweenService:Create(mainContainer, TweenInfo.new(0.2), {Position = UDim2.new(0.5, -170, 0.5, -230)}):Play()
        end
        wait(0.2)
        mainContainer.Visible = false
        TweenService:Create(floatingButton, TweenInfo.new(0.2), {Size = isMobile and UDim2.new(0, 65, 0, 65) or UDim2.new(0, 55, 0, 55), ImageColor3 = Color3.fromRGB(255, 255, 255)}):Play()
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
            TweenService:Create(floatingButton, TweenInfo.new(0.4, Enum.EasingStyle.Sine), {Size = isMobile and UDim2.new(0, 72, 0, 72) or UDim2.new(0, 60, 0, 60)}):Play()
            wait(0.2)
            TweenService:Create(floatingButton, TweenInfo.new(0.4, Enum.EasingStyle.Sine), {Size = isMobile and UDim2.new(0, 65, 0, 65) or UDim2.new(0, 55, 0, 55)}):Play()
        end
    end
end)()

print("════════════════════════════════════════════════════════════════")
print("✅ SISTEMA RIAN STUDIOS V6.0 CARREGADO!")
print("✅ UI RESPONSIVA E OTIMIZADA PARA MOBILE!")
print("✅ LAYOUT REDESIGNADO PARA TELAS PEQUENAS!")
print("✅ BOTÃO ATIVAR VELOCIDADE - PERSISTENTE!")
print("════════════════════════════════════════════════════════════════")
