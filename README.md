--[[
    SCRIPT: Sistema de Velocidade e Combate
    LOCAL: StarterPlayerScripts ou StarterGui
    AUTOR: Sistema Profissional
    DESCRIÇÃO: UI moderna com controle de velocidade e sistema de combo de socos
]]

-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- Variáveis do jogador
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- Configurações de Velocidade
local NORMAL_SPEED = 16
local FAST_SPEED = 50
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
punchSound.SoundId = "rbxassetid://9120384332" -- Som de soco (substitua pelo ID do seu som)
punchSound.Volume = 0.5
punchSound.Parent = soundService

-- CRIAR UI MODERNA
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ModernUI"
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Container principal (centralizado e responsivo)
local mainContainer = Instance.new("Frame")
mainContainer.Name = "MainContainer"
mainContainer.Size = UDim2.new(0, 320, 0, 400)
mainContainer.Position = UDim2.new(0.5, -160, 0.5, -200)
mainContainer.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
mainContainer.BackgroundTransparency = 0.05
mainContainer.BorderSizePixel = 0
mainContainer.Parent = screenGui

-- Sombra e borda arredondada
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = mainContainer

local shadow = Instance.new("UICorner")
shadow.CornerRadius = UDim.new(0, 12)
shadow.Parent = mainContainer

-- Efeito de vidro (glassmorphism)
local blurEffect = Instance.new("BlurEffect")
blurEffect.Size = 8
blurEffect.Parent = mainContainer

-- Título da UI
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
titleLabel.TextStrokeTransparency = 0.5
titleLabel.Parent = titleFrame

-- Container dos botões
local buttonsContainer = Instance.new("Frame")
buttonsContainer.Size = UDim2.new(1, -40, 0, 120)
buttonsContainer.Position = UDim2.new(0, 20, 0, 70)
buttonsContainer.BackgroundTransparency = 1
buttonsContainer.Parent = mainContainer

-- Layout dos botões (grid)
local gridLayout = Instance.new("UIGridLayout")
gridLayout.CellSize = UDim2.new(0, 130, 0, 50)
gridLayout.CellPadding = UDim2.new(0, 20, 0, 20)
gridLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
gridLayout.Parent = buttonsContainer

-- FUNÇÃO PARA CRIAR BOTÕES MODERNOS
local function createModernButton(text, color, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 130, 0, 50)
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextScaled = true
    button.Font = Enum.Font.GothamSemibold
    button.BackgroundColor3 = color
    button.BorderSizePixel = 0
    button.AutoButtonColor = false
    button.Parent = buttonsContainer
    
    -- Arredondar bordas
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = button
    
    -- Efeito hover
    local originalColor = color
    local hoverColor = Color3.new(color.R + 0.1, color.G + 0.1, color.B + 0.1)
    
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = hoverColor}):Play()
        TweenService:Create(button, TweenInfo.new(0.1), {Size = UDim2.new(0, 135, 0, 52)}):Play()
    end)
    
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = originalColor}):Play()
        TweenService:Create(button, TweenInfo.new(0.1), {Size = UDim2.new(0, 130, 0, 50)}):Play()
    end)
    
    -- Efeito de clique
    button.MouseButton1Click:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.05), {Size = UDim2.new(0, 125, 0, 48)}):Play()
        wait(0.05)
        TweenService:Create(button, TweenInfo.new(0.05), {Size = UDim2.new(0, 130, 0, 50)}):Play()
        callback()
    end)
    
    return button
end

-- BOTÃO 1: VELOCIDADE
local speedButton = createModernButton("⚡ VELOCIDADE", Color3.fromRGB(66, 135, 245), function()
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid then
        isSpeedBoosted = not isSpeedBoosted
        local newSpeed = isSpeedBoosted and FAST_SPEED or NORMAL_SPEED
        humanoid.WalkSpeed = newSpeed
        
        -- Efeito visual ao ativar
        local speedColor = isSpeedBoosted and Color3.fromRGB(76, 175, 80) or Color3.fromRGB(66, 135, 245)
        TweenService:Create(speedButton, TweenInfo.new(0.3), {BackgroundColor3 = speedColor}):Play()
        
        -- Notificação visual
        local notif = Instance.new("TextLabel")
        notif.Size = UDim2.new(0, 200, 0, 40)
        notif.Position = UDim2.new(0.5, -100, 0, -50)
        notif.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        notif.BackgroundTransparency = 0.2
        notif.Text = isSpeedBoosted and "⚡ VELOCIDADE ATIVADA! ⚡" or "VELOCIDADE NORMAL"
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

-- BOTÃO 2: INFORMAÇÕES
local infoButton = createModernButton("ℹ️ INFO", Color3.fromRGB(156, 39, 176), function()
    -- Mostrar informações do sistema
    local infoFrame = Instance.new("Frame")
    infoFrame.Size = UDim2.new(0, 250, 0, 150)
    infoFrame.Position = UDim2.new(0.5, -125, 0.5, -75)
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
    infoText.Text = "⚡ VELOCIDADE: " .. (isSpeedBoosted and "RÁPIDA" or "NORMAL") .. "\n\n👊 CLIQUE ESQUERDO: SOCOS RÁPIDOS\n\n💫 COMBO DE 3 SOCOS"
    infoText.TextColor3 = Color3.fromRGB(255, 255, 255)
    infoText.TextScaled = false
    infoText.TextSize = 14
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
    
    -- Auto-fechar após 5 segundos
    wait(5)
    if infoFrame then infoFrame:Destroy() end
end)

-- ANIMAÇÃO DE SOCO
local function playPunchAnimation(character, arm)
    local humanoid = character:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    local animTrack = nil
    if arm == "right" then
        local rightArm = character:FindFirstChild("Right Arm") or character:FindFirstChild("RightHand")
        if rightArm then
            local animation = Instance.new("Animation")
            animation.AnimationId = "rbxassetid://131453564" -- Animação de soco padrão (funciona na maioria dos rigs)
            animTrack = humanoid:LoadAnimation(animation)
        end
    end
    
    if animTrack then
        animTrack:Play()
        wait(0.2)
        animTrack:Stop()
    end
end

-- SISTEMA DE ATAQUE RÁPIDO
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
        
        -- Atualizar personagem (caso tenha morrido)
        local character = player.Character
        local humanoid = character and character:FindFirstChild("Humanoid")
        
        if character and humanoid then
            -- Tocar som de soco
            local soundClone = punchSound:Clone()
            soundClone.Parent = character
            soundClone:Play()
            
            -- Animar soco
            playPunchAnimation(character, "right")
            
            -- Efeito visual de impacto (partículas)
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
            
            -- Detectar hits em inimigos próximos
            for _, enemy in pairs(workspace:GetChildren()) do
                if enemy:IsA("Model") and enemy ~= character then
                    local enemyHumanoid = enemy:FindFirstChild("Humanoid")
                    if enemyHumanoid and enemyHumanoid.Health > 0 then
                        local distance = (character:GetPivot().Position - enemy:GetPivot().Position).Magnitude
                        if distance < 5 then
                            enemyHumanoid:TakeDamage(PUNCH_DAMAGE)
                            
                            -- Efeito de knockback
                            local knockbackForce = Instance.new("BodyVelocity")
                            knockbackForce.MaxForce = Vector3.new(1000, 1000, 1000)
                            knockbackForce.Velocity = (enemy:GetPivot().Position - character:GetPivot().Position).unit * 20 + Vector3.new(0, 10, 0)
                            knockbackForce.Parent = enemy:GetPivot()
                            game:GetService("Debris"):AddItem(knockbackForce, 0.2)
                        end
                    end
                end
            end
        end
        
        wait(PUNCH_COOLDOWN)
        canPunch = true
        
        -- Continuar combo se ainda estiver atacando
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

-- DETECTOR DE CLIQUE DO MOUSE
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    -- Verificar se é clique esquerdo do mouse
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local character = player.Character
        if character and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
            startComboAttack()
        end
    end
end)

-- ATUALIZAR REFERÊNCIA DO PERSONAGEM QUANDO RENASCER
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    
    -- Resetar velocidade ao renascer
    isSpeedBoosted = false
    humanoid.WalkSpeed = NORMAL_SPEED
    TweenService:Create(speedButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(66, 135, 245)}):Play()
end)

-- BOTÃO DE FECHAR UI (opcional)
local closeButton = Instance.new("ImageButton")
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -40, 0, 10)
closeButton.Image = "rbxassetid://3926305904"
closeButton.BackgroundTransparency = 1
closeButton.Parent = mainContainer

closeButton.MouseButton1Click:Connect(function()
    mainContainer.Visible = not mainContainer.Visible
end)

-- ANIMAÇÃO DE ENTRADA DA UI
mainContainer.BackgroundTransparency = 1
mainContainer.Position = UDim2.new(0.5, -160, 0.5, -150)
TweenService:Create(mainContainer, TweenInfo.new(0.5, Enum.EasingStyle.Back), {BackgroundTransparency = 0.05}):Play()
TweenService:Create(mainContainer, TweenInfo.new(0.5, Enum.EasingStyle.Back), {Position = UDim2.new(0.5, -160, 0.5, -200)}):Play()

print("✅ Sistema Moderno de Velocidade e Combate carregado com sucesso!")
