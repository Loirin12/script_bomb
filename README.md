--[[
    SCRIPT: Sistema de Velocidade e Combate
    LOCAL: StarterPlayerScripts ou StarterGui
    AUTOR: Sistema Profissional
    DESCRIÇÃO: UI moderna com controle de velocidade (SLIDER + BOTÃO RÁPIDO) e sistema de combo de socos
    ATUALIZAÇÃO: Interface inicia minimizada + Slider de velocidade
]]

-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Variáveis do jogador
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- Configurações de Velocidade
local NORMAL_SPEED = 16
local FAST_SPEED = 80
local MIN_SPEED = 16
local MAX_SPEED = 120
local currentSpeed = NORMAL_SPEED
local isSpeedBoosted = false

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

-- CRIAR UI MODERNA
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ModernUI"
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- BOTÃO FLUTUANTE
local floatingButton = Instance.new("ImageButton")
floatingButton.Name = "FloatingButton"
floatingButton.Size = UDim2.new(0, 60, 0, 60)
floatingButton.Position = UDim2.new(0.02, 0, 0.85, 0)
floatingButton.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
floatingButton.BackgroundTransparency = 0.1
floatingButton.Image = "rbxassetid://6031094773"
floatingButton.ImageColor3 = Color3.fromRGB(255, 255, 255)
floatingButton.ScaleType = Enum.ScaleType.Fit
floatingButton.Parent = screenGui

local floatingCorner = Instance.new("UICorner")
floatingCorner.CornerRadius = UDim.new(1, 0)
floatingCorner.Parent = floatingButton

-- CONTAINER PRINCIPAL (INICIA MINIMIZADO)
local mainContainer = Instance.new("Frame")
mainContainer.Name = "MainContainer"
mainContainer.Size = UDim2.new(0, 360, 0, 520)
mainContainer.Position = UDim2.new(0.5, -180, 0.5, -260)
mainContainer.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
mainContainer.BackgroundTransparency = 0.05
mainContainer.BorderSizePixel = 0
mainContainer.Visible = false
mainContainer.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = mainContainer

-- Título
local titleFrame = Instance.new("Frame")
titleFrame.Size = UDim2.new(1, 0, 0, 50)
titleFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
titleFrame.BackgroundTransparency = 0.1
titleFrame.BorderSizePixel = 0
titleFrame.Parent = mainContainer

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = titleFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 1, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚡ CONTROLES AVANÇADOS"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextScaled = true
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Parent = titleFrame

-- CONTAINER DE VELOCIDADE
local speedContainer = Instance.new("Frame")
speedContainer.Size = UDim2.new(1, -40, 0, 120)
speedContainer.Position = UDim2.new(0, 20, 0, 70)
speedContainer.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
speedContainer.BackgroundTransparency = 0.3
speedContainer.BorderSizePixel = 0
speedContainer.Parent = mainContainer

local speedContainerCorner = Instance.new("UICorner")
speedContainerCorner.CornerRadius = UDim.new(0, 8)
speedContainerCorner.Parent = speedContainer

-- Label Velocidade Atual
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(1, -20, 0, 30)
speedLabel.Position = UDim2.new(0, 10, 0, 5)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "⚡ VELOCIDADE ATUAL: " .. math.floor(currentSpeed)
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.TextSize = 14
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Parent = speedContainer

-- SLIDER DE VELOCIDADE
local sliderBg = Instance.new("Frame")
sliderBg.Size = UDim2.new(0.8, 0, 0, 8)
sliderBg.Position = UDim2.new(0.1, 0, 0.5, -4)
sliderBg.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
sliderBg.BorderSizePixel = 0
sliderBg.Parent = speedContainer

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
sliderButton.Size = UDim2.new(0, 20, 0, 20)
sliderButton.Position = UDim2.new((currentSpeed - MIN_SPEED) / (MAX_SPEED - MIN_SPEED), -10, 0.5, -10)
sliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
sliderButton.Image = "rbxassetid://266788897"
sliderButton.ScaleType = Enum.ScaleType.Fit
sliderButton.Parent = speedContainer

local sliderBtnCorner = Instance.new("UICorner")
sliderBtnCorner.CornerRadius = UDim.new(1, 0)
sliderBtnCorner.Parent = sliderButton

-- Labels Min/Max
local minLabel = Instance.new("TextLabel")
minLabel.Size = UDim2.new(0, 30, 0, 20)
minLabel.Position = UDim2.new(0.05, 0, 0.5, 15)
minLabel.BackgroundTransparency = 1
minLabel.Text = tostring(MIN_SPEED)
minLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
minLabel.TextSize = 12
minLabel.Font = Enum.Font.Gotham
minLabel.Parent = speedContainer

local maxLabel = Instance.new("TextLabel")
maxLabel.Size = UDim2.new(0, 30, 0, 20)
maxLabel.Position = UDim2.new(0.88, 0, 0.5, 15)
maxLabel.BackgroundTransparency = 1
maxLabel.Text = tostring(MAX_SPEED)
maxLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
maxLabel.TextSize = 12
maxLabel.Font = Enum.Font.Gotham
maxLabel.Parent = speedContainer

-- Botão Turbo (Velocidade Máxima Instantânea)
local turboButton = Instance.new("TextButton")
turboButton.Size = UDim2.new(0, 120, 0, 35)
turboButton.Position = UDim2.new(0.5, -60, 1, -40)
turboButton.Text = "🚀 TURBO!"
turboButton.TextColor3 = Color3.fromRGB(255, 255, 255)
turboButton.TextScaled = true
turboButton.Font = Enum.Font.GothamBold
turboButton.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
turboButton.BorderSizePixel = 0
turboButton.Parent = speedContainer

local turboCorner = Instance.new("UICorner")
turboCorner.CornerRadius = UDim.new(0, 6)
turboCorner.Parent = turboButton

-- Container dos botões de ação
local buttonsContainer = Instance.new("Frame")
buttonsContainer.Size = UDim2.new(1, -40, 0, 100)
buttonsContainer.Position = UDim2.new(0, 20, 0, 210)
buttonsContainer.BackgroundTransparency = 1
buttonsContainer.Parent = mainContainer

local gridLayout = Instance.new("UIGridLayout")
gridLayout.CellSize = UDim2.new(0, 140, 0, 45)
gridLayout.CellPadding = UDim2.new(0, 15, 0, 10)
gridLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
gridLayout.Parent = buttonsContainer

-- FUNÇÃO PARA ATUALIZAR VELOCIDADE
local function updateSpeed(newSpeed)
    currentSpeed = math.clamp(newSpeed, MIN_SPEED, MAX_SPEED)
    if humanoid then
        humanoid.WalkSpeed = currentSpeed
    end
    speedLabel.Text = "⚡ VELOCIDADE ATUAL: " .. math.floor(currentSpeed)
    
    -- Atualizar barra
    local percent = (currentSpeed - MIN_SPEED) / (MAX_SPEED - MIN_SPEED)
    sliderFill:TweenSize(UDim2.new(percent, 0, 1, 0), "Out", "Quad", 0.1, true)
    sliderButton:TweenPosition(UDim2.new(percent, -10, 0.5, -10), "Out", "Quad", 0.1, true)
end

-- FUNÇÃO SLIDER (DRAG)
local dragging = false
local function updateSliderFromMouse(input)
    local mousePos = input.Position.X
    local sliderAbsPos = sliderBg.AbsolutePosition.X
    local sliderWidth = sliderBg.AbsoluteSize.X
    local percent = math.clamp((mousePos - sliderAbsPos) / sliderWidth, 0, 1)
    local newSpeed = MIN_SPEED + (percent * (MAX_SPEED - MIN_SPEED))
    updateSpeed(newSpeed)
    isSpeedBoosted = false
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
local function createModernButton(text, color, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 140, 0, 45)
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextScaled = true
    button.Font = Enum.Font.GothamSemibold
    button.BackgroundColor3 = color
    button.BorderSizePixel = 0
    button.AutoButtonColor = false
    button.Parent = buttonsContainer
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = button
    
    local originalColor = color
    local hoverColor = Color3.new(color.R + 0.1, color.G + 0.1, color.B + 0.1)
    
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = hoverColor}):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = originalColor}):Play()
    end)
    
    button.MouseButton1Click:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.05), {Size = UDim2.new(0, 135, 0, 42)}):Play()
        wait(0.05)
        TweenService:Create(button, TweenInfo.new(0.05), {Size = UDim2.new(0, 140, 0, 45)}):Play()
        callback()
    end)
    
    return button
end

-- BOTÃO VELOCIDADE RÁPIDA (ALTERNA ENTRE NORMAL E RÁPIDO)
local speedToggleButton = createModernButton("⚡ RÁPIDO", Color3.fromRGB(66, 135, 245), function()
    if isSpeedBoosted then
        -- Voltar para velocidade normal
        updateSpeed(NORMAL_SPEED)
        isSpeedBoosted = false
        TweenService:Create(speedToggleButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(66, 135, 245)}):Play()
        
        local notif = Instance.new("TextLabel")
        notif.Size = UDim2.new(0, 200, 0, 40)
        notif.Position = UDim2.new(0.5, -100, 0, -50)
        notif.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        notif.BackgroundTransparency = 0.2
        notif.Text = "VELOCIDADE NORMAL"
        notif.TextColor3 = Color3.fromRGB(255, 255, 255)
        notif.TextScaled = true
        notif.Font = Enum.Font.GothamBold
        notif.Parent = screenGui
        
        local notifCorner = Instance.new("UICorner")
        notifCorner.CornerRadius = UDim.new(0, 8)
        notifCorner.Parent = notif
        
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -100, 0, 20)}):Play()
        wait(1.5)
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -100, 0, -50), BackgroundTransparency = 1}):Play()
        wait(0.3)
        notif:Destroy()
    else
        -- Ativar velocidade rápida
        updateSpeed(FAST_SPEED)
        isSpeedBoosted = true
        TweenService:Create(speedToggleButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(76, 175, 80)}):Play()
        
        local notif = Instance.new("TextLabel")
        notif.Size = UDim2.new(0, 200, 0, 40)
        notif.Position = UDim2.new(0.5, -100, 0, -50)
        notif.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        notif.BackgroundTransparency = 0.2
        notif.Text = "⚡ VELOCIDADE RÁPIDA! ⚡"
        notif.TextColor3 = Color3.fromRGB(255, 255, 255)
        notif.TextScaled = true
        notif.Font = Enum.Font.GothamBold
        notif.Parent = screenGui
        
        local notifCorner = Instance.new("UICorner")
        notifCorner.CornerRadius = UDim.new(0, 8)
        notifCorner.Parent = notif
        
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -100, 0, 20)}):Play()
        wait(1.5)
        TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -100, 0, -50), BackgroundTransparency = 1}):Play()
        wait(0.3)
        notif:Destroy()
    end
end)

-- BOTÃO INFORMAÇÕES
local infoButton = createModernButton("ℹ️ INFO", Color3.fromRGB(156, 39, 176), function()
    local infoFrame = Instance.new("Frame")
    infoFrame.Size = UDim2.new(0, 280, 0, 170)
    infoFrame.Position = UDim2.new(0.5, -140, 0.5, -85)
    infoFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    infoFrame.BackgroundTransparency = 0.05
    infoFrame.BorderSizePixel = 0
    infoFrame.Parent = screenGui
    
    local infoCorner = Instance.new("UICorner")
    infoCorner.CornerRadius = UDim.new(0, 12)
    infoCorner.Parent = infoFrame
    
    local infoText = Instance.new("TextLabel")
    infoText.Size = UDim2.new(1, -20, 1, -20)
    infoText.Position = UDim2.new(0, 10, 0, 10)
    infoText.BackgroundTransparency = 1
    infoText.Text = "⚡ VELOCIDADE:\n   • Slider: Ajuste livre\n   • RÁPIDO: 16 ↔ 80\n   • TURBO: Máximo instantâneo\n\n👊 ATAQUE:\n   • Clique ESQUERDO: Combo de 3 socos\n   • Dano: " .. PUNCH_DAMAGE .. " por soco\n\n💫 Slider vai de " .. MIN_SPEED .. " até " .. MAX_SPEED
    infoText.TextColor3 = Color3.fromRGB(255, 255, 255)
    infoText.TextSize = 13
    infoText.Font = Enum.Font.Gotham
    infoText.TextXAlignment = Enum.TextXAlignment.Left
    infoText.Parent = infoFrame
    
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 80, 0, 30)
    closeBtn.Position = UDim2.new(0.5, -40, 1, -40)
    closeBtn.Text = "FECHAR"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = infoFrame
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 6)
    closeCorner.Parent = closeBtn
    
    closeBtn.MouseButton1Click:Connect(function()
        infoFrame:Destroy()
    end)
    
    wait(6)
    if infoFrame then infoFrame:Destroy() end
end)

-- BOTÃO TURBO (VELOCIDADE MÁXIMA)
turboButton.MouseButton1Click:Connect(function()
    updateSpeed(MAX_SPEED)
    isSpeedBoosted = true
    TweenService:Create(speedToggleButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(76, 175, 80)}):Play()
    
    -- Efeito turbo
    local turboEffect = Instance.new("TextLabel")
    turboEffect.Size = UDim2.new(0, 300, 0, 60)
    turboEffect.Position = UDim2.new(0.5, -150, 0.5, -30)
    turboEffect.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
    turboEffect.BackgroundTransparency = 0.2
    turboEffect.Text = "🚀 TURBO ATIVADO! 🚀"
    turboEffect.TextColor3 = Color3.fromRGB(255, 255, 255)
    turboEffect.TextScaled = true
    turboEffect.Font = Enum.Font.GothamBold
    turboEffect.Parent = screenGui
    
    local turboEffectCorner = Instance.new("UICorner")
    turboEffectCorner.CornerRadius = UDim.new(0, 12)
    turboEffectCorner.Parent = turboEffect
    
    TweenService:Create(turboEffect, TweenInfo.new(0.3), {BackgroundTransparency = 0.8, Position = UDim2.new(0.5, -150, 0.3, -30)}):Play()
    wait(2)
    TweenService:Create(turboEffect, TweenInfo.new(0.3), {BackgroundTransparency = 1}):Play()
    wait(0.3)
    turboEffect:Destroy()
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
            
            -- Efeito visual
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
            
            -- Hit detection
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
            startCombatAttack()
        end
    end
end)

-- ATUALIZAR REFERÊNCIA DO PERSONAGEM
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    updateSpeed(currentSpeed)
    isSpeedBoosted = false
    TweenService:Create(speedToggleButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(66, 135, 245)}):Play()
end)

-- FUNÇÃO ABRIR/FECHAR UI
local isUIOpen = false

local function toggleUI()
    isUIOpen = not isUIOpen
    
    if isUIOpen then
        mainContainer.Visible = true
        mainContainer.BackgroundTransparency = 1
        mainContainer.Position = UDim2.new(0.5, -180, 0.5, -200)
        TweenService:Create(mainContainer, TweenInfo.new(0.4, Enum.EasingStyle.Back), {BackgroundTransparency = 0.05}):Play()
        TweenService:Create(mainContainer, TweenInfo.new(0.4, Enum.EasingStyle.Back), {Position = UDim2.new(0.5, -180, 0.5, -260)}):Play()
        TweenService:Create(floatingButton, TweenInfo.new(0.3), {Size = UDim2.new(0, 55, 0, 55), ImageColor3 = Color3.fromRGB(100, 200, 255)}):Play()
    else
        TweenService:Create(mainContainer, TweenInfo.new(0.3), {BackgroundTransparency = 1, Position = UDim2.new(0.5, -180, 0.5, -200)}):Play()
        wait(0.3)
        mainContainer.Visible = false
        TweenService:Create(floatingButton, TweenInfo.new(0.3), {Size = UDim2.new(0, 60, 0, 60), ImageColor3 = Color3.fromRGB(255, 255, 255)}):Play()
    end
end

floatingButton.MouseButton1Click:Connect(toggleUI)

-- BOTÃO DE FECHAR NA UI
local closeButton = Instance.new("ImageButton")
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -40, 0, 10)
closeButton.Image = "rbxassetid://3926305904"
closeButton.BackgroundTransparency = 1
closeButton.Parent = mainContainer

closeButton.MouseButton1Click:Connect(function()
    toggleUI()
end)

-- INICIALIZAR VELOCIDADE
updateSpeed(NORMAL_SPEED)

print("✅ Sistema Moderno carregado! UI inicia minimizada. Use o botão flutuante para abrir.")
