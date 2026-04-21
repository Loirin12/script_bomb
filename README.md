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
    ║                        ███████╗██╗   ██╗███████╗████████╗███████╗███╗   ███╗ █████╗                  ║
    ║                        ██╔════╝╚██╗ ██╔╝██╔════╝╚══██╔══╝██╔════╝████╗ ████║██╔══██╗                 ║
    ║                        ███████╗ ╚████╔╝ ███████╗   ██║   █████╗  ██╔████╔██║███████║                 ║
    ║                        ╚════██║  ╚██╔╝  ╚════██║   ██║   ██╔══╝  ██║╚██╔╝██║██╔══██║                 ║
    ║                        ███████║   ██║   ███████║   ██║   ███████╗██║ ╚═╝ ██║██║  ██║                 ║
    ║                        ╚══════╝   ╚═╝   ╚══════╝   ╚═╝   ╚══════╝╚═╝     ╚═╝╚═╝  ╚═╝                 ║
    ║                                                                                                      ║
    ║              SISTEMA V7.0 - RIAN STUDIOS - AIMBOT + INVULNERABILIDADE                                 ║
    ║                    VELOCIDADE | VOO | BOOST | COMBATE | AIMBOT | INVINCIBLE                           ║
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

-- Variáveis do jogador
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- Detecta mobile
local isMobile = UserInputService.TouchEnabled
local screenSize = player:GetMouse().ViewSizeX

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
local originalHumanoidState = nil

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

local aimbotSound = Instance.new("Sound")
aimbotSound.SoundId = "rbxassetid://9120384332"
aimbotSound.Volume = 0.5
aimbotSound.Parent = soundService

local invincibleSound = Instance.new("Sound")
invincibleSound.SoundId = "rbxassetid://9120384332"
invincibleSound.Volume = 0.6
invincibleSound.Parent = soundService

-- CRIAR UI MODERNA E BONITA
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RianStudiosUI"
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false

-- Efeito de fundo
local backgroundGradient = Instance.new("Frame")
backgroundGradient.Size = UDim2.new(1, 0, 1, 0)
backgroundGradient.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
backgroundGradient.BackgroundTransparency = 0.85
backgroundGradient.Parent = screenGui

-- BOTÃO FLUTUANTE 3D
local floatingButton = Instance.new("ImageButton")
floatingButton.Name = "FloatingButton"
floatingButton.Size = isMobile and UDim2.new(0, 70, 0, 70) or UDim2.new(0, 60, 0, 60)
floatingButton.Position = isMobile and UDim2.new(0.85, 0, 0.88, 0) or UDim2.new(0.02, 0, 0.85, 0)
floatingButton.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
floatingButton.Image = "rbxassetid://6031094773"
floatingButton.ImageColor3 = Color3.fromRGB(255, 255, 255)
floatingButton.ScaleType = Enum.ScaleType.Fit
floatingButton.Parent = screenGui

local floatingCorner = Instance.new("UICorner")
floatingCorner.CornerRadius = UDim.new(1, 0)
floatingCorner.Parent = floatingButton

local floatingGlow = Instance.new("UIGradient")
floatingGlow.Rotation = 45
floatingGlow.Transparency = NumberSequence.new(0, 0.7)
floatingGlow.Parent = floatingButton

-- CONTAINER PRINCIPAL
local mainContainer = Instance.new("ScrollingFrame")
mainContainer.Name = "MainContainer"
mainContainer.Size = isMobile and UDim2.new(0.92, 0, 0.88, 0) or UDim2.new(0, 380, 0, 600)
mainContainer.Position = isMobile and UDim2.new(0.04, 0, 0.06, 0) or UDim2.new(0.5, -190, 0.5, -300)
mainContainer.BackgroundColor3 = Color3.fromRGB(12, 12, 22)
mainContainer.BackgroundTransparency = 0.08
mainContainer.BorderSizePixel = 0
mainContainer.Visible = false
mainContainer.Parent = screenGui
mainContainer.ClipsDescendants = false
mainContainer.ScrollBarThickness = 3
mainContainer.ScrollBarImageColor3 = Color3.fromRGB(66, 135, 245)

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 24)
mainCorner.Parent = mainContainer

local mainBorder = Instance.new("UIStroke")
mainBorder.Color = Color3.fromRGB(66, 135, 245)
mainBorder.Thickness = 1.5
mainBorder.Transparency = 0.5
mainBorder.Parent = mainContainer

local mainShadow = Instance.new("UIGradient")
mainShadow.Rotation = 90
mainShadow.Transparency = NumberSequence.new(0, 0.9)
mainShadow.Parent = mainContainer

-- Barra de título GLASS
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 65)
titleBar.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainContainer

local titleBarCorner = Instance.new("UICorner")
titleBarCorner.CornerRadius = UDim.new(0, 24)
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
titleLabel.Position = UDim2.new(0, 20, 0, 10)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚡ RIAN STUDIOS V7.0"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = isMobile and 18 or 16
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local subtitleLabel = Instance.new("TextLabel")
subtitleLabel.Size = UDim2.new(1, -80, 0, 20)
subtitleLabel.Position = UDim2.new(0, 20, 0, 38)
subtitleLabel.BackgroundTransparency = 1
subtitleLabel.Text = "AIMBOT | INVINCIBLE | FLY | SPEED"
subtitleLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
subtitleLabel.TextSize = isMobile and 11 or 10
subtitleLabel.Font = Enum.Font.Gotham
subtitleLabel.TextXAlignment = Enum.TextXAlignment.Left
subtitleLabel.Parent = titleBar

local closeButton = Instance.new("ImageButton")
closeButton.Size = UDim2.new(0, 34, 0, 34)
closeButton.Position = UDim2.new(1, -48, 0, 15)
closeButton.Image = "rbxassetid://3926305904"
closeButton.ImageColor3 = Color3.fromRGB(255, 255, 255)
closeButton.BackgroundTransparency = 1
closeButton.Parent = titleBar

-- ============ CARD DE VELOCIDADE ============
local speedCard = Instance.new("Frame")
speedCard.Size = UDim2.new(1, -24, 0, 150)
speedCard.Position = UDim2.new(0, 12, 0, 80)
speedCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
speedCard.BackgroundTransparency = 0.3
speedCard.BorderSizePixel = 0
speedCard.Parent = mainContainer

local speedCardCorner = Instance.new("UICorner")
speedCardCorner.CornerRadius = UDim.new(0, 16)
speedCardCorner.Parent = speedCard

local speedCardStroke = Instance.new("UIStroke")
speedCardStroke.Color = Color3.fromRGB(66, 135, 245)
speedCardStroke.Thickness = 1
speedCardStroke.Transparency = 0.6
speedCardStroke.Parent = speedCard

local speedIcon = Instance.new("ImageLabel")
speedIcon.Size = UDim2.new(0, 45, 0, 45)
speedIcon.Position = UDim2.new(0, 15, 0, 15)
speedIcon.Image = "rbxassetid://6031094773"
speedIcon.ImageColor3 = Color3.fromRGB(66, 135, 245)
speedIcon.BackgroundTransparency = 1
speedIcon.Parent = speedCard

local speedTitleLabel = Instance.new("TextLabel")
speedTitleLabel.Size = UDim2.new(1, -75, 0, 25)
speedTitleLabel.Position = UDim2.new(0, 70, 0, 15)
speedTitleLabel.BackgroundTransparency = 1
speedTitleLabel.Text = "VELOCIDADE"
speedTitleLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
speedTitleLabel.TextSize = isMobile and 13 or 11
speedTitleLabel.Font = Enum.Font.Gotham
speedTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
speedTitleLabel.Parent = speedCard

local speedValueLabel = Instance.new("TextLabel")
speedValueLabel.Size = UDim2.new(1, -75, 0, 45)
speedValueLabel.Position = UDim2.new(0, 70, 0, 35)
speedValueLabel.BackgroundTransparency = 1
speedValueLabel.Text = math.floor(currentSpeed)
speedValueLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedValueLabel.TextSize = isMobile and 38 or 32
speedValueLabel.Font = Enum.Font.GothamBold
speedValueLabel.TextXAlignment = Enum.TextXAlignment.Left
speedValueLabel.Parent = speedCard

local speedMaxLabel = Instance.new("TextLabel")
speedMaxLabel.Size = UDim2.new(1, -75, 0, 25)
speedMaxLabel.Position = UDim2.new(0, 70, 0, 75)
speedMaxLabel.BackgroundTransparency = 1
speedMaxLabel.Text = "/ " .. MAX_SPEED
speedMaxLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
speedMaxLabel.TextSize = isMobile and 16 or 14
speedMaxLabel.Font = Enum.Font.Gotham
speedMaxLabel.TextXAlignment = Enum.TextXAlignment.Left
speedMaxLabel.Parent = speedCard

local bombTimerLabel = Instance.new("TextLabel")
bombTimerLabel.Size = UDim2.new(1, -75, 0, 20)
bombTimerLabel.Position = UDim2.new(0, 70, 0, 98)
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
sliderButton.Size = UDim2.new(0, 24, 0, 24)
sliderButton.Position = UDim2.new((currentSpeed - MIN_SPEED) / (MAX_SPEED - MIN_SPEED), -12, 0.5, -12)
sliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
sliderButton.Image = "rbxassetid://266788897"
sliderButton.ScaleType = Enum.ScaleType.Fit
sliderButton.BackgroundTransparency = 1
sliderButton.Parent = speedCard

-- ============ CARD DE VOO ============
local flyCard = Instance.new("Frame")
flyCard.Size = UDim2.new(1, -24, 0, 110)
flyCard.Position = UDim2.new(0, 12, 0, 245)
flyCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
flyCard.BackgroundTransparency = 0.3
flyCard.BorderSizePixel = 0
flyCard.Parent = mainContainer

local flyCardCorner = Instance.new("UICorner")
flyCardCorner.CornerRadius = UDim.new(0, 16)
flyCardCorner.Parent = flyCard

local flyCardStroke = Instance.new("UIStroke")
flyCardStroke.Color = Color3.fromRGB(66, 200, 100)
flyCardStroke.Thickness = 1
flyCardStroke.Transparency = 0.6
flyCardStroke.Parent = flyCard

local flyIcon = Instance.new("ImageLabel")
flyIcon.Size = UDim2.new(0, 40, 0, 40)
flyIcon.Position = UDim2.new(0, 15, 0, 15)
flyIcon.Image = "rbxassetid://6031094773"
flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
flyIcon.BackgroundTransparency = 1
flyIcon.Parent = flyCard

local flyTitleLabel = Instance.new("TextLabel")
flyTitleLabel.Size = UDim2.new(1, -70, 0, 25)
flyTitleLabel.Position = UDim2.new(0, 65, 0, 15)
flyTitleLabel.BackgroundTransparency = 1
flyTitleLabel.Text = "SISTEMA DE VOO"
flyTitleLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
flyTitleLabel.TextSize = isMobile and 13 or 11
flyTitleLabel.Font = Enum.Font.Gotham
flyTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
flyTitleLabel.Parent = flyCard

local flyStatusLabel = Instance.new("TextLabel")
flyStatusLabel.Size = UDim2.new(1, -70, 0, 30)
flyStatusLabel.Position = UDim2.new(0, 65, 0, 40)
flyStatusLabel.BackgroundTransparency = 1
flyStatusLabel.Text = "STATUS: DESLIGADO"
flyStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
flyStatusLabel.TextSize = isMobile and 14 or 12
flyStatusLabel.Font = Enum.Font.GothamBold
flyStatusLabel.TextXAlignment = Enum.TextXAlignment.Left
flyStatusLabel.Parent = flyCard

local flyButton = Instance.new("TextButton")
flyButton.Size = UDim2.new(0, isMobile and 130 or 110, 0, isMobile and 42 or 38)
flyButton.Position = UDim2.new(0.5, isMobile and -65 or -55, 1, -14)
flyButton.Text = "🕊️ ATIVAR VOO"
flyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
flyButton.TextSize = isMobile and 14 or 12
flyButton.Font = Enum.Font.GothamBold
flyButton.BackgroundColor3 = Color3.fromRGB(66, 200, 100)
flyButton.BorderSizePixel = 0
flyButton.Parent = flyCard

local flyButtonCorner = Instance.new("UICorner")
flyButtonCorner.CornerRadius = UDim.new(0, 10)
flyButtonCorner.Parent = flyButton

-- ============ CARD DE COMBATE ============
local combatCard = Instance.new("Frame")
combatCard.Size = UDim2.new(1, -24, 0, 110)
combatCard.Position = UDim2.new(0, 12, 0, 370)
combatCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
combatCard.BackgroundTransparency = 0.3
combatCard.BorderSizePixel = 0
combatCard.Parent = mainContainer

local combatCardCorner = Instance.new("UICorner")
combatCardCorner.CornerRadius = UDim.new(0, 16)
combatCardCorner.Parent = combatCard

local combatCardStroke = Instance.new("UIStroke")
combatCardStroke.Color = Color3.fromRGB(255, 64, 64)
combatCardStroke.Thickness = 1
combatCardStroke.Transparency = 0.6
combatCardStroke.Parent = combatCard

local combatIcon = Instance.new("ImageLabel")
combatIcon.Size = UDim2.new(0, 40, 0, 40)
combatIcon.Position = UDim2.new(0, 15, 0, 15)
combatIcon.Image = "rbxassetid://6031094773"
combatIcon.ImageColor3 = Color3.fromRGB(255, 64, 64)
combatIcon.BackgroundTransparency = 1
combatIcon.Parent = combatCard

local combatTitleLabel = Instance.new("TextLabel")
combatTitleLabel.Size = UDim2.new(1, -70, 0, 25)
combatTitleLabel.Position = UDim2.new(0, 65, 0, 15)
combatTitleLabel.BackgroundTransparency = 1
combatTitleLabel.Text = "SISTEMA DE COMBATE"
combatTitleLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
combatTitleLabel.TextSize = isMobile and 13 or 11
combatTitleLabel.Font = Enum.Font.Gotham
combatTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
combatTitleLabel.Parent = combatCard

local combatInfoLabel = Instance.new("TextLabel")
combatInfoLabel.Size = UDim2.new(1, -70, 0, 30)
combatInfoLabel.Position = UDim2.new(0, 65, 0, 40)
combatInfoLabel.BackgroundTransparency = 1
combatInfoLabel.Text = "DANO: " .. PUNCH_DAMAGE .. " | COMBO: 3 SOCOS"
combatInfoLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
combatInfoLabel.TextSize = isMobile and 12 or 10
combatInfoLabel.Font = Enum.Font.Gotham
combatInfoLabel.TextXAlignment = Enum.TextXAlignment.Left
combatInfoLabel.Parent = combatCard

local combatStatusLabel = Instance.new("TextLabel")
combatStatusLabel.Size = UDim2.new(1, -70, 0, 20)
combatStatusLabel.Position = UDim2.new(0, 65, 0, 70)
combatStatusLabel.BackgroundTransparency = 1
combatStatusLabel.Text = "✅ PRONTO PARA ATACAR"
combatStatusLabel.TextColor3 = Color3.fromRGB(76, 175, 80)
combatStatusLabel.TextSize = isMobile and 11 or 9
combatStatusLabel.Font = Enum.Font.GothamBold
combatStatusLabel.TextXAlignment = Enum.TextXAlignment.Left
combatStatusLabel.Parent = combatCard

-- ============ CARD DE AIMBOT ============
local aimbotCard = Instance.new("Frame")
aimbotCard.Size = UDim2.new(1, -24, 0, 110)
aimbotCard.Position = UDim2.new(0, 12, 0, 495)
aimbotCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
aimbotCard.BackgroundTransparency = 0.3
aimbotCard.BorderSizePixel = 0
aimbotCard.Parent = mainContainer

local aimbotCardCorner = Instance.new("UICorner")
aimbotCardCorner.CornerRadius = UDim.new(0, 16)
aimbotCardCorner.Parent = aimbotCard

local aimbotCardStroke = Instance.new("UIStroke")
aimbotCardStroke.Color = Color3.fromRGB(156, 39, 176)
aimbotCardStroke.Thickness = 1
aimbotCardStroke.Transparency = 0.6
aimbotCardStroke.Parent = aimbotCard

local aimbotIcon = Instance.new("ImageLabel")
aimbotIcon.Size = UDim2.new(0, 40, 0, 40)
aimbotIcon.Position = UDim2.new(0, 15, 0, 15)
aimbotIcon.Image = "rbxassetid://6031094773"
aimbotIcon.ImageColor3 = Color3.fromRGB(156, 39, 176)
aimbotIcon.BackgroundTransparency = 1
aimbotIcon.Parent = aimbotCard

local aimbotTitleLabel = Instance.new("TextLabel")
aimbotTitleLabel.Size = UDim2.new(1, -70, 0, 25)
aimbotTitleLabel.Position = UDim2.new(0, 65, 0, 15)
aimbotTitleLabel.BackgroundTransparency = 1
aimbotTitleLabel.Text = "🎯 AIMBOT"
aimbotTitleLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
aimbotTitleLabel.TextSize = isMobile and 13 or 11
aimbotTitleLabel.Font = Enum.Font.Gotham
aimbotTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
aimbotTitleLabel.Parent = aimbotCard

local aimbotStatusLabel = Instance.new("TextLabel")
aimbotStatusLabel.Size = UDim2.new(1, -70, 0, 30)
aimbotStatusLabel.Position = UDim2.new(0, 65, 0, 40)
aimbotStatusLabel.BackgroundTransparency = 1
aimbotStatusLabel.Text = "STATUS: DESLIGADO"
aimbotStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
aimbotStatusLabel.TextSize = isMobile and 13 or 11
aimbotStatusLabel.Font = Enum.Font.GothamBold
aimbotStatusLabel.TextXAlignment = Enum.TextXAlignment.Left
aimbotStatusLabel.Parent = aimbotCard

local aimbotRangeLabel = Instance.new("TextLabel")
aimbotRangeLabel.Size = UDim2.new(1, -70, 0, 20)
aimbotRangeLabel.Position = UDim2.new(0, 65, 0, 70)
aimbotRangeLabel.BackgroundTransparency = 1
aimbotRangeLabel.Text = "ALCANCE: " .. aimbotRange
aimbotRangeLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
aimbotRangeLabel.TextSize = isMobile and 10 or 9
aimbotRangeLabel.Font = Enum.Font.Gotham
aimbotRangeLabel.TextXAlignment = Enum.TextXAlignment.Left
aimbotRangeLabel.Parent = aimbotCard

local aimbotButton = Instance.new("TextButton")
aimbotButton.Size = UDim2.new(0, isMobile and 120 or 100, 0, isMobile and 38 or 34)
aimbotButton.Position = UDim2.new(0.5, isMobile and -60 or -50, 1, -12)
aimbotButton.Text = "🎯 ATIVAR AIMBOT"
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
invincibleCard.Size = UDim2.new(1, -24, 0, 110)
invincibleCard.Position = UDim2.new(0, 12, 0, 620)
invincibleCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
invincibleCard.BackgroundTransparency = 0.3
invincibleCard.BorderSizePixel = 0
invincibleCard.Parent = mainContainer

local invincibleCardCorner = Instance.new("UICorner")
invincibleCardCorner.CornerRadius = UDim.new(0, 16)
invincibleCardCorner.Parent = invincibleCard

local invincibleCardStroke = Instance.new("UIStroke")
invincibleCardStroke.Color = Color3.fromRGB(255, 193, 7)
invincibleCardStroke.Thickness = 1
invincibleCardStroke.Transparency = 0.6
invincibleCardStroke.Parent = invincibleCard

local invincibleIcon = Instance.new("ImageLabel")
invincibleIcon.Size = UDim2.new(0, 40, 0, 40)
invincibleIcon.Position = UDim2.new(0, 15, 0, 15)
invincibleIcon.Image = "rbxassetid://6031094773"
invincibleIcon.ImageColor3 = Color3.fromRGB(255, 193, 7)
invincibleIcon.BackgroundTransparency = 1
invincibleIcon.Parent = invincibleCard

local invincibleTitleLabel = Instance.new("TextLabel")
invincibleTitleLabel.Size = UDim2.new(1, -70, 0, 25)
invincibleTitleLabel.Position = UDim2.new(0, 65, 0, 15)
invincibleTitleLabel.BackgroundTransparency = 1
invincibleTitleLabel.Text = "🛡️ INVULNERABILIDADE"
invincibleTitleLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
invincibleTitleLabel.TextSize = isMobile and 13 or 11
invincibleTitleLabel.Font = Enum.Font.Gotham
invincibleTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
invincibleTitleLabel.Parent = invincibleCard

local invincibleStatusLabel = Instance.new("TextLabel")
invincibleStatusLabel.Size = UDim2.new(1, -70, 0, 30)
invincibleStatusLabel.Position = UDim2.new(0, 65, 0, 40)
invincibleStatusLabel.BackgroundTransparency = 1
invincibleStatusLabel.Text = "STATUS: DESLIGADO"
invincibleStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
invincibleStatusLabel.TextSize = isMobile and 13 or 11
invincibleStatusLabel.Font = Enum.Font.GothamBold
invincibleStatusLabel.TextXAlignment = Enum.TextXAlignment.Left
invincibleStatusLabel.Parent = invincibleCard

local invincibleEffectLabel = Instance.new("TextLabel")
invincibleEffectLabel.Size = UDim2.new(1, -70, 0, 20)
invincibleEffectLabel.Position = UDim2.new(0, 65, 0, 70)
invincibleEffectLabel.BackgroundTransparency = 1
invincibleEffectLabel.Text = "⚠️ IMUNE A DANOS"
invincibleEffectLabel.TextColor3 = Color3.fromRGB(255, 193, 7)
invincibleEffectLabel.TextSize = isMobile and 10 or 9
invincibleEffectLabel.Font = Enum.Font.Gotham
invincibleEffectLabel.TextXAlignment = Enum.TextXAlignment.Left
invincibleEffectLabel.Visible = false
invincibleEffectLabel.Parent = invincibleCard

local invincibleButton = Instance.new("TextButton")
invincibleButton.Size = UDim2.new(0, isMobile and 130 or 110, 0, isMobile and 38 or 34)
invincibleButton.Position = UDim2.new(0.5, isMobile and -65 or -55, 1, -12)
invincibleButton.Text = "🛡️ ATIVAR INVINCIBLE"
invincibleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
invincibleButton.TextSize = isMobile and 12 or 10
invincibleButton.Font = Enum.Font.GothamBold
invincibleButton.BackgroundColor3 = Color3.fromRGB(255, 193, 7)
invincibleButton.BorderSizePixel = 0
invincibleButton.Parent = invincibleCard

local invincibleButtonCorner = Instance.new("UICorner")
invincibleButtonCorner.CornerRadius = UDim.new(0, 10)
invincibleButtonCorner.Parent = invincibleButton

-- ============ BOTÕES RÁPIDOS ============
local quickButtonsContainer = Instance.new("Frame")
quickButtonsContainer.Size = UDim2.new(1, -24, 0, 55)
quickButtonsContainer.Position = UDim2.new(0, 12, 0, 745)
quickButtonsContainer.BackgroundTransparency = 1
quickButtonsContainer.Parent = mainContainer

local quickLayout = Instance.new("UIListLayout")
quickLayout.FillDirection = Enum.FillDirection.Horizontal
quickLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
quickLayout.Padding = UDim.new(0, 10)
quickLayout.Parent = quickButtonsContainer

-- Botão Ativar Velocidade
local autoSpeedButton = Instance.new("TextButton")
autoSpeedButton.Size = UDim2.new(0, isMobile and 160 or 140, 0, isMobile and 48 or 42)
autoSpeedButton.Text = "🔘 ATIVAR VELOCIDADE"
autoSpeedButton.TextColor3 = Color3.fromRGB(255, 255, 255)
autoSpeedButton.TextSize = isMobile and 13 or 11
autoSpeedButton.Font = Enum.Font.GothamBold
autoSpeedButton.BackgroundColor3 = Color3.fromRGB(34, 197, 94)
autoSpeedButton.BorderSizePixel = 0
autoSpeedButton.Parent = quickButtonsContainer

local autoButtonCorner = Instance.new("UICorner")
autoButtonCorner.CornerRadius = UDim.new(0, 12)
autoButtonCorner.Parent = autoSpeedButton

-- Botão Turbo
local turboButton = Instance.new("TextButton")
turboButton.Size = UDim2.new(0, isMobile and 70 or 60, 0, isMobile and 48 or 42)
turboButton.Text = "🚀"
turboButton.TextColor3 = Color3.fromRGB(255, 255, 255)
turboButton.TextSize = isMobile and 24 or 20
turboButton.Font = Enum.Font.GothamBold
turboButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
turboButton.BorderSizePixel = 0
turboButton.Parent = quickButtonsContainer

local turboCorner = Instance.new("UICorner")
turboCorner.CornerRadius = UDim.new(0, 12)
turboCorner.Parent = turboButton

-- Botão Info
local infoButton = Instance.new("TextButton")
infoButton.Size = UDim2.new(0, isMobile and 70 or 60, 0, isMobile and 48 or 42)
infoButton.Text = "ℹ️"
infoButton.TextColor3 = Color3.fromRGB(255, 255, 255)
infoButton.TextSize = isMobile and 24 or 20
infoButton.Font = Enum.Font.GothamBold
infoButton.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
infoButton.BorderSizePixel = 0
infoButton.Parent = quickButtonsContainer

local infoCorner = Instance.new("UICorner")
infoCorner.CornerRadius = UDim.new(0, 12)
infoCorner.Parent = infoButton

-- Footer
local footerFrame = Instance.new("Frame")
footerFrame.Size = UDim2.new(1, 0, 0, 45)
footerFrame.Position = UDim2.new(0, 0, 1, -45)
footerFrame.BackgroundColor3 = Color3.fromRGB(8, 8, 16)
footerFrame.BackgroundTransparency = 0.5
footerFrame.BorderSizePixel = 0
footerFrame.Parent = mainContainer

local footerCorner = Instance.new("UICorner")
footerCorner.CornerRadius = UDim.new(0, 16)
footerCorner.Parent = footerFrame

local creditLabel = Instance.new("TextLabel")
creditLabel.Size = UDim2.new(1, 0, 0, 20)
creditLabel.Position = UDim2.new(0, 0, 0, 5)
creditLabel.BackgroundTransparency = 1
creditLabel.Text = "⚡ RIAN STUDIOS V7.0 - AIMBOT + INVINCIBLE ⚡"
creditLabel.TextColor3 = Color3.fromRGB(120, 120, 120)
creditLabel.TextSize = isMobile and 10 or 9
creditLabel.Font = Enum.Font.Gotham
creditLabel.Parent = footerFrame

local versionLabel = Instance.new("TextLabel")
versionLabel.Size = UDim2.new(1, 0, 0, 16)
versionLabel.Position = UDim2.new(0, 0, 0, 24)
versionLabel.BackgroundTransparency = 1
versionLabel.Text = "SISTEMA PROFISSIONAL | AIMBOT | INVINCIBLE | FLY | SPEED"
versionLabel.TextColor3 = Color3.fromRGB(80, 80, 80)
versionLabel.TextSize = isMobile and 8 or 7
versionLabel.Font = Enum.Font.Gotham
versionLabel.Parent = footerFrame

-- Controles mobile para voo
local mobileFlyControls = Instance.new("Frame")
mobileFlyControls.Size = UDim2.new(0, 200, 0, 100)
mobileFlyControls.Position = UDim2.new(0.5, -100, 1, -120)
mobileFlyControls.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
mobileFlyControls.BackgroundTransparency = 0.2
mobileFlyControls.BorderSizePixel = 0
mobileFlyControls.Visible = false
mobileFlyControls.Parent = screenGui

local mobileControlsCorner = Instance.new("UICorner")
mobileControlsCorner.CornerRadius = UDim.new(0, 16)
mobileControlsCorner.Parent = mobileFlyControls

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

-- ============ FUNÇÃO DE INVULNERABILIDADE ============
local function toggleInvincible()
    invincibleEnabled = not invincibleEnabled
    
    if invincibleEnabled then
        -- Ativar invulnerabilidade
        invincibleSound:Play()
        invincibleButton.Text = "🛡️ DESATIVAR INVINCIBLE"
        invincibleButton.BackgroundColor3 = Color3.fromRGB(220, 150, 0)
        invincibleStatusLabel.Text = "STATUS: 🛡️ ATIVADO"
        invincibleStatusLabel.TextColor3 = Color3.fromRGB(255, 193, 7)
        invincibleEffectLabel.Visible = true
        invincibleCardStroke.Color = Color3.fromRGB(255, 193, 7)
        invincibleIcon.ImageColor3 = Color3.fromRGB(255, 193, 7)
        
        -- Aplicar invulnerabilidade
        if humanoid then
            humanoid.BreakJointsOnDeath = false
            humanoid.MaxHealth = math.huge
            humanoid.Health = math.huge
        end
        
        -- Efeito visual de brilho
        local glowEffect = Instance.new("PointLight")
        glowEffect.Color = Color3.fromRGB(255, 193, 7)
        glowEffect.Range = 8
        glowEffect.Brightness = 1.5
        glowEffect.Parent = humanoidRootPart or character
        
        local particleEffect = Instance.new("ParticleEmitter")
        particleEffect.Texture = "rbxassetid://284646226"
        particleEffect.Rate = 30
        particleEffect.SpreadAngle = Vector2.new(360, 360)
        particleEffect.Lifetime = NumberRange.new(0.8)
        particleEffect.Speed = NumberRange.new(3)
        particleEffect.Color = ColorSequence.new(Color3.fromRGB(255, 193, 7))
        particleEffect.Parent = humanoidRootPart or character
        
        -- Conectar detector de dano
        local function onHumanoidDamage(amount)
            if invincibleEnabled then
                humanoid.Health = humanoid.MaxHealth
                return 0
            end
            return amount
        end
        
        humanoid.Damage:Connect(function(amount)
            if invincibleEnabled then
                humanoid.Health = humanoid.MaxHealth
                return 0
            end
        end)
        
        -- Notificação
        local notif = Instance.new("Frame")
        notif.Size = UDim2.new(0, 260, 0, 50)
        notif.Position = UDim2.new(0.5, -130, 0, -50)
        notif.BackgroundColor3 = Color3.fromRGB(255, 193, 7)
        notif.BackgroundTransparency = 0.1
        notif.BorderSizePixel = 0
        notif.Parent = screenGui
        
        local notifCorner = Instance.new("UICorner")
        notifCorner.CornerRadius = UDim.new(0, 12)
        notifCorner.Parent = notif
        
        local notifText = Instance.new("TextLabel")
        notifText.Size = UDim2.new(1, 0, 1, 0)
        notifText.BackgroundTransparency = 1
        notifText.Text = "🛡️ INVULNERABILIDADE ATIVADA! 🛡️"
        notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
        notifText.TextScaled = true
        notifText.Font = Enum.Font.GothamBold
        notifText.Parent = notif
        
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -130, 0, 20)}):Play()
        wait(2)
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -130, 0, -50), BackgroundTransparency = 1}):Play()
        wait(0.3)
        notif:Destroy()
        
    else
        -- Desativar invulnerabilidade
        invincibleSound:Play()
        invincibleButton.Text = "🛡️ ATIVAR INVINCIBLE"
        invincibleButton.BackgroundColor3 = Color3.fromRGB(255, 193, 7)
        invincibleStatusLabel.Text = "STATUS: DESLIGADO"
        invincibleStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        invincibleEffectLabel.Visible = false
        invincibleCardStroke.Color = Color3.fromRGB(255, 193, 7)
        invincibleIcon.ImageColor3 = Color3.fromRGB(255, 193, 7)
        
        -- Restaurar saúde normal
        if humanoid then
            humanoid.BreakJointsOnDeath = true
            humanoid.MaxHealth = 100
            if humanoid.Health > 100 then
                humanoid.Health = 100
            end
        end
        
        -- Remover efeitos visuais
        for _, child in pairs(character:GetChildren()) do
            if child:IsA("PointLight") and child.Color == Color3.fromRGB(255, 193, 7) then
                child:Destroy()
            end
            if child:IsA("ParticleEmitter") and child.Color == ColorSequence.new(Color3.fromRGB(255, 193, 7)) then
                child:Destroy()
            end
        end
        
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
        notifText.Text = "🛡️ INVULNERABILIDADE DESATIVADA! 🛡️"
        notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
        notifText.TextScaled = true
        notifText.Font = Enum.Font.GothamBold
        notifText.Parent = notif
        
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -130, 0, 20)}):Play()
        wait(2)
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -130, 0, -50), BackgroundTransparency = 1}):Play()
        wait(0.3)
        notif:Destroy()
    end
end

-- ============ FUNÇÃO DE AIMBOT ============
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
    
    return nearest, nearestDistance
end

local function updateAimbot()
    if not aimbotEnabled or not character or not humanoidRootPart then return end
    
    local target, distance = getNearestEnemy()
    if target and distance <= aimbotRange then
        -- Calcular ângulo de rotação
        local lookAt = CFrame.lookAt(humanoidRootPart.Position, target.Position)
        local newCFrame = humanoidRootPart.CFrame:Lerp(lookAt, aimbotSmoothness)
        humanoidRootPart.CFrame = newCFrame
    end
end

local function toggleAimbot()
    aimbotEnabled = not aimbotEnabled
    
    if aimbotEnabled then
        aimbotSound:Play()
        aimbotButton.Text = "🎯 DESATIVAR AIMBOT"
        aimbotButton.BackgroundColor3 = Color3.fromRGB(100, 30, 120)
        aimbotStatusLabel.Text = "STATUS: 🎯 ATIVADO"
        aimbotStatusLabel.TextColor3 = Color3.fromRGB(156, 39, 176)
        aimbotCardStroke.Color = Color3.fromRGB(156, 39, 176)
        aimbotIcon.ImageColor3 = Color3.fromRGB(156, 39, 176)
        
        -- Iniciar loop do aimbot
        if aimbotConnection then aimbotConnection:Disconnect() end
        aimbotConnection = RunService.RenderStepped:Connect(updateAimbot)
        
        local notif = Instance.new("Frame")
        notif.Size = UDim2.new(0, 240, 0, 50)
        notif.Position = UDim2.new(0.5, -120, 0, -50)
        notif.BackgroundColor3 = Color3.fromRGB(156, 39, 176)
        notif.BackgroundTransparency = 0.1
        notif.BorderSizePixel = 0
        notif.Parent = screenGui
        
        local notifCorner = Instance.new("UICorner")
        notifCorner.CornerRadius = UDim.new(0, 12)
        notifCorner.Parent = notif
        
        local notifText = Instance.new("TextLabel")
        notifText.Size = UDim2.new(1, 0, 1, 0)
        notifText.BackgroundTransparency = 1
        notifText.Text = "🎯 AIMBOT ATIVADO! 🎯"
        notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
        notifText.TextScaled = true
        notifText.Font = Enum.Font.GothamBold
        notifText.Parent = notif
        
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -120, 0, 20)}):Play()
        wait(2)
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -120, 0, -50), BackgroundTransparency = 1}):Play()
        wait(0.3)
        notif:Destroy()
    else
        aimbotSound:Play()
        aimbotButton.Text = "🎯 ATIVAR AIMBOT"
        aimbotButton.BackgroundColor3 = Color3.fromRGB(156, 39, 176)
        aimbotStatusLabel.Text = "STATUS: DESLIGADO"
        aimbotStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        aimbotCardStroke.Color = Color3.fromRGB(156, 39, 176)
        aimbotIcon.ImageColor3 = Color3.fromRGB(156, 39, 176)
        
        if aimbotConnection then
            aimbotConnection:Disconnect()
            aimbotConnection = nil
        end
        
        local notif = Instance.new("Frame")
        notif.Size = UDim2.new(0, 240, 0, 50)
        notif.Position = UDim2.new(0.5, -120, 0, -50)
        notif.BackgroundColor3 = Color3.fromRGB(64, 64, 64)
        notif.BackgroundTransparency = 0.1
        notif.BorderSizePixel = 0
        notif.Parent = screenGui
        
        local notifCorner = Instance.new("UICorner")
        notifCorner.CornerRadius = UDim.new(0, 12)
        notifCorner.Parent = notif
        
        local notifText = Instance.new("TextLabel")
        notifText.Size = UDim2.new(1, 0, 1, 0)
        notifText.BackgroundTransparency = 1
        notifText.Text = "🎯 AIMBOT DESATIVADO! 🎯"
        notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
        notifText.TextScaled = true
        notifText.Font = Enum.Font.GothamBold
        notifText.Parent = notif
        
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -120, 0, 20)}):Play()
        wait(2)
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -120, 0, -50), BackgroundTransparency = 1}):Play()
        wait(0.3)
        notif:Destroy()
    end
end

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

-- Botão Ativar Velocidade
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
        
        local notif = Instance.new("Frame")
        notif.Size = UDim2.new(0, 220, 0, 45)
        notif.Position = UDim2.new(0.5, -110, 0, -50)
        notif.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
        notif.BackgroundTransparency = 0.1
        notif.BorderSizePixel = 0
        notif.Parent = screenGui
        
        local notifCorner = Instance.new("UICorner")
        notifCorner.CornerRadius = UDim.new(0, 12)
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
        
        local notif = Instance.new("Frame")
        notif.Size = UDim2.new(0, 220, 0, 45)
        notif.Position = UDim2.new(0.5, -110, 0, -50)
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
    notif.Size = UDim2.new(0, 280, 0, 55)
    notif.Position = UDim2.new(0.5, -140, 0, -50)
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
    
    TweenService:Create(notif, TweenInfo.new(0.4), {Position = UDim2.new(0.5, -140, 0, 20)}):Play()
    
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
            endNotif.Size = UDim2.new(0, 260, 0, 45)
            endNotif.Position = UDim2.new(0.5, -130, 0, -50)
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
            
            TweenService:Create(endNotif, TweenInfo.new(0.4), {Position = UDim2.new(0.5, -130, 0, 20)}):Play()
            wait(2)
            TweenService:Create(endNotif, TweenInfo.new(0.4), {Position = UDim2.new(0.5, -130, 0, -50), BackgroundTransparency = 1}):Play()
            wait(0.3)
            endNotif:Destroy()
        end
    end)
    
    wait(2)
    TweenService:Create(notif, TweenInfo.new(0.4), {Position = UDim2.new(0.5, -140, 0, -50), BackgroundTransparency = 1}):Play()
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
    flyStatusLabel.Text = "STATUS: 🕊️ VOANDO"
    flyStatusLabel.TextColor3 = Color3.fromRGB(66, 200, 100)
    flyButton.Text = "🛑 PARAR VOO"
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
    flyStatusLabel.Text = "STATUS: DESLIGADO"
    flyStatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
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
    turboEffect.Size = UDim2.new(0, 220, 0, 50)
    turboEffect.Position = UDim2.new(0.5, -110, 0.5, -25)
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
    
    TweenService:Create(turboEffect, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -110, 0.3, -25)}):Play()
    wait(1.5)
    TweenService:Create(turboEffect, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
    wait(0.3)
    turboEffect:Destroy()
end)

infoButton.MouseButton1Click:Connect(function()
    local infoFrame = Instance.new("Frame")
    infoFrame.Size = UDim2.new(0, isMobile and 320 or 300, 0, isMobile and 380 or 350)
    infoFrame.Position = UDim2.new(0.5, isMobile and -160 or -150, 0.5, isMobile and -190 or -175)
    infoFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 28)
    infoFrame.BackgroundTransparency = 0.05
    infoFrame.BorderSizePixel = 0
    infoFrame.Parent = screenGui
    
    local infoCorner = Instance.new("UICorner")
    infoCorner.CornerRadius = UDim.new(0, 20)
    infoCorner.Parent = infoFrame
    
    local infoStroke = Instance.new("UIStroke")
    infoStroke.Color = Color3.fromRGB(66, 135, 245)
    infoStroke.Thickness = 1.5
    infoStroke.Transparency = 0.5
    infoStroke.Parent = infoFrame
    
    local infoTitle = Instance.new("TextLabel")
    infoTitle.Size = UDim2.new(1, 0, 0, 50)
    infoTitle.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
    infoTitle.Text = "   INFORMAÇÕES DO SISTEMA"
    infoTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    infoTitle.TextSize = isMobile and 16 or 14
    infoTitle.Font = Enum.Font.GothamBold
    infoTitle.TextXAlignment = Enum.TextXAlignment.Left
    infoTitle.Parent = infoFrame
    
    local infoTitleCorner = Instance.new("UICorner")
    infoTitleCorner.CornerRadius = UDim.new(0, 20)
    infoTitleCorner.Parent = infoTitle
    
    local infoText = Instance.new("TextLabel")
    infoText.Size = UDim2.new(1, -30, 1, -70)
    infoText.Position = UDim2.new(0, 15, 0, 60)
    infoText.BackgroundTransparency = 1
    infoText.Text = [[
⚡ RIAN STUDIOS V7.0 - SISTEMA COMPLETO ⚡

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎮 CONTROLES DE VELOCIDADE:
   • ATIVAR VELOCIDADE: 80 (persistente)
   • SLIDER: Ajuste livre 16-120
   • TURBO: Máximo instantâneo

🕊️ SISTEMA DE VOO:
   • ATIVAR VOO: WASD + ESPAÇO/SHIFT
   • MOBILE: Botões na tela

🎯 AIMBOT:
   • Mira automática em inimigos
   • Alcance: 100 studs
   • Suavidade: 30%

🛡️ INVULNERABILIDADE:
   • Imune a todos os danos
   • Saúde infinita

👊 COMBATE:
   • Clique ESQUERDO: Combo de 3 socos
   • Dano: 25 por soco

💣 BOOST:
   • Pegue bombas para velocidade 120 por 10s

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
👑 CRIADO POR RIAN | V7.0
    ]]
    infoText.TextColor3 = Color3.fromRGB(220, 220, 220)
    infoText.TextSize = isMobile and 11 or 10
    infoText.Font = Enum.Font.Gotham
    infoText.TextXAlignment = Enum.TextXAlignment.Left
    infoText.TextYAlignment = Enum.TextYAlignment.Top
    infoText.Parent = infoFrame
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 80, 0, 35)
    closeBtn.Position = UDim2.new(0.5, -40, 1, -45)
    closeBtn.Text = "FECHAR"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.TextSize = isMobile and 14 or 12
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

aimbotButton.MouseButton1Click:Connect(function()
    local clickClone = clickSound:Clone()
    clickClone.Parent = aimbotButton
    clickClone:Play()
    toggleAimbot()
end)

invincibleButton.MouseButton1Click:Connect(function()
    local clickClone = clickSound:Clone()
    clickClone.Parent = invincibleButton
    clickClone:Play()
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
            punchSound:Clone().Parent = char
            punchSound:Play()
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
            
            for _, enemy in pairs(workspace:GetChildren()) do
                if enemy:IsA("Model") and enemy ~= char then
                    local enemyHum = enemy:FindFirstChild("Humanoid")
                    if enemyHum and enemyHum.Health > 0 then
                        local dist = (char:GetPivot().Position - enemy:GetPivot().Position).Magnitude
                        if dist < 6 then
                            enemyHum:TakeDamage(PUNCH_DAMAGE)
                            
                            -- Knockback
                            local knockback = Instance.new("BodyVelocity")
                            knockback.MaxForce = Vector3.new(1000, 1000, 1000)
                            knockback.Velocity = (enemy:GetPivot().Position - char:GetPivot().Position).unit * 30 + Vector3.new(0, 15, 0)
                            knockback.Parent = enemy:GetPivot()
                            game:GetService("Debris"):AddItem(knockback, 0.2)
                        end
                    end
                end
            end
            
            -- Atualizar status do combate
            combatStatusLabel.Text = "🔥 ATACANDO! COMBO: " .. comboCount .. "/3"
            combatStatusLabel.TextColor3 = Color3.fromRGB(255, 64, 64)
        end
        
        wait(PUNCH_COOLDOWN)
        canPunch = true
        
        if isAttacking and comboCount < 3 then
            wait(COMBO_DELAY)
            performPunch()
        else
            isAttacking = false
            comboCount = 0
            combatStatusLabel.Text = "✅ PRONTO PARA ATACAR"
            combatStatusLabel.TextColor3 = Color3.fromRGB(76, 175, 80)
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
    
    if invincibleEnabled then
        humanoid.BreakJointsOnDeath = false
        humanoid.MaxHealth = math.huge
        humanoid.Health = math.huge
    end
    
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
            mainContainer.Position = UDim2.new(0.5, -190, 0.5, -280)
        end
        TweenService:Create(mainContainer, TweenInfo.new(0.3, Enum.EasingStyle.Back), {BackgroundTransparency = 0.08}):Play()
        if not isMobile then
            TweenService:Create(mainContainer, TweenInfo.new(0.3, Enum.EasingStyle.Back), {Position = UDim2.new(0.5, -190, 0.5, -300)}):Play()
        end
        TweenService:Create(floatingButton, TweenInfo.new(0.2), {Size = isMobile and UDim2.new(0, 60, 0, 60) or UDim2.new(0, 50, 0, 50), ImageColor3 = Color3.fromRGB(100, 200, 255)}):Play()
    else
        TweenService:Create(mainContainer, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
        if not isMobile then
            TweenService:Create(mainContainer, TweenInfo.new(0.2), {Position = UDim2.new(0.5, -190, 0.5, -280)}):Play()
        end
        wait(0.2)
        mainContainer.Visible = false
        TweenService:Create(floatingButton, TweenInfo.new(0.2), {Size = isMobile and UDim2.new(0, 70, 0, 70) or UDim2.new(0, 60, 0, 60), ImageColor3 = Color3.fromRGB(255, 255, 255)}):Play()
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
            TweenService:Create(floatingButton, TweenInfo.new(0.4, Enum.EasingStyle.Sine), {Size = isMobile and UDim2.new(0, 78, 0, 78) or UDim2.new(0, 65, 0, 65)}):Play()
            wait(0.2)
            TweenService:Create(floatingButton, TweenInfo.new(0.4, Enum.EasingStyle.Sine), {Size = isMobile and UDim2.new(0, 70, 0, 70) or UDim2.new(0, 60, 0, 60)}):Play()
        end
    end
end)()

print("═══════════════════════════════════════════════════════════════════════════")
print("✅ SISTEMA RIAN STUDIOS V7.0 CARREGADO COM SUCESSO!")
print("✅ LAYOUT PROFISSIONAL REDESIGNADO!")
print("✅ AIMBOT ATIVADO! - MIRA AUTOMÁTICA EM INIMIGOS")
print("✅ INVULNERABILIDADE ATIVADA! - IMUNE A DANOS")
print("✅ VELOCIDADE | VOO | BOOST | COMBATE | AIMBOT | INVINCIBLE")
print("═══════════════════════════════════════════════════════════════════════════")
