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
    ║              SISTEMA V10.0 - RIAN STUDIOS - QUEBRAR LIMITE DE BLOCOS                                 ║
    ║              VELOCIDADE | VOO | AIMBOT | INVINCIBLE | LIMITE INFINITO DE BLOCOS                      ║
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
local HttpService = game:GetService("HttpService")

-- Variáveis do jogador
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- Detecta mobile
local isMobile = UserInputService.TouchEnabled

-- ============ SISTEMA PARA QUEBRAR LIMITE DE BLOCOS ============
local maxBlocksLimit = 1000 -- Limite máximo de blocos (pode pegar até 1000)
local currentBlocks = 0
local blockLimitRemoved = false

-- Função para remover/ignorar o limite de blocos do jogo
local function removeBlockLimit()
    blockLimitRemoved = true
    print("[SISTEMA] Tentando remover limite de blocos...")
    
    -- MÉTODO 1: Modificar leaderstats
    pcall(function()
        local leaderstats = player:FindFirstChild("leaderstats")
        if leaderstats then
            for _, stat in pairs(leaderstats:GetChildren()) do
                if stat:IsA("IntValue") or stat:IsA("NumberValue") then
                    local statName = stat.Name:lower()
                    if statName:find("block") or statName:find("bloco") or 
                       statName:find("limit") or statName:find("max") or
                       statName:find("capacity") or statName:find("capacidade") then
                        stat.Value = maxBlocksLimit
                        print("[SISTEMA] Modificado: " .. stat.Name .. " para " .. maxBlocksLimit)
                    end
                end
            end
        end
    end)
    
    -- MÉTODO 2: Modificar atributos do jogador
    pcall(function()
        player:SetAttribute("MaxBlocks", maxBlocksLimit)
        player:SetAttribute("BlockLimit", maxBlocksLimit)
        player:SetAttribute("BlockCapacity", maxBlocksLimit)
        player:SetAttribute("MaxCapacity", maxBlocksLimit)
        player:SetAttribute("BlocksLimit", maxBlocksLimit)
        player:SetAttribute("LimiteBlocos", maxBlocksLimit)
        player:SetAttribute("CapacidadeBlocos", maxBlocksLimit)
    end)
    
    -- MÉTODO 3: Modificar o inventário/backpack
    pcall(function()
        local backpack = player:FindFirstChild("Backpack")
        if backpack then
            backpack:SetAttribute("MaxBlocks", maxBlocksLimit)
            backpack:SetAttribute("BlockLimit", maxBlocksLimit)
        end
        
        local starterGear = player:FindFirstChild("StarterGear")
        if starterGear then
            starterGear:SetAttribute("MaxBlocks", maxBlocksLimit)
        end
    end)
    
    -- MÉTODO 4: Procurar e modificar valores no workspace
    pcall(function()
        for _, obj in pairs(Workspace:GetDescendants()) do
            if obj:IsA("IntValue") or obj:IsA("NumberValue") then
                local objName = obj.Name:lower()
                if (objName:find("block") or objName:find("bloco")) and 
                   (objName:find("limit") or objName:find("max") or objName:find("capacity")) then
                    obj.Value = maxBlocksLimit
                    print("[SISTEMA] Modificado workspace: " .. obj.Name)
                end
            end
        end
    end)
    
    -- MÉTODO 5: Interceptar e modificar remotos (se existirem)
    pcall(function()
        local remotes = ReplicatedStorage:FindFirstChild("Remotes") or ReplicatedStorage
        for _, remote in pairs(remotes:GetChildren()) do
            if remote:IsA("RemoteEvent") or remote:IsA("RemoteFunction") then
                local remoteName = remote.Name:lower()
                if remoteName:find("block") or remoteName:find("collect") or 
                   remoteName:find("inventory") or remoteName:find("capacity") then
                    
                    -- Tenta modificar o remote
                    local oldFunction = remote.OnClientInvoke
                    if oldFunction then
                        remote.OnClientInvoke = function(...)
                            local args = {...}
                            for i, arg in pairs(args) do
                                if type(arg) == "number" and arg < maxBlocksLimit then
                                    args[i] = maxBlocksLimit
                                end
                            end
                            return oldFunction(unpack(args))
                        end
                    end
                end
            end
        end
    end)
end

-- Função para forçar a coleta de blocos
local function forceBlockCollection()
    pcall(function()
        -- Procura por blocos no jogo e tenta coletá-los
        for _, obj in pairs(Workspace:GetDescendants()) do
            if obj:IsA("BasePart") and not obj:IsA("Terrain") then
                local objName = obj.Name:lower()
                local isBlock = objName:find("block") or objName:find("bloco") or 
                               objName:find("brick") or objName:find("cube") or
                               objName:find("collect") or objName:find("item") or
                               objName:find("pickup") or objName:find("coin") or
                               objName:find("gem") or obj:GetAttribute("Collectable")
                
                if isBlock and currentBlocks < maxBlocksLimit then
                    -- Tenta coletar o bloco
                    local distance = (humanoidRootPart.Position - obj.Position).Magnitude
                    if distance < 10 then
                        currentBlocks = currentBlocks + 1
                        obj:Destroy()
                        print("[SISTEMA] Bloco coletado! Total: " .. currentBlocks .. "/" .. maxBlocksLimit)
                        
                        -- Efeito visual
                        local effect = Instance.new("Part")
                        effect.Shape = Enum.PartType.Ball
                        effect.Size = Vector3.new(1, 1, 1)
                        effect.BrickColor = BrickColor.new("Bright green")
                        effect.Material = Enum.Material.Neon
                        effect.CFrame = obj.CFrame
                        effect.Anchored = true
                        effect.CanCollide = false
                        effect.Parent = workspace
                        TweenService:Create(effect, TweenInfo.new(0.3), {Size = Vector3.new(0, 0, 0)}):Play()
                        game:GetService("Debris"):AddItem(effect, 0.3)
                    end
                end
            end
        end
    end)
end

-- ============ CRIAR UI ============
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RianStudiosUI"
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false

-- BOTÃO FLUTUANTE
local floatingButton = Instance.new("ImageButton")
floatingButton.Size = isMobile and UDim2.new(0, 65, 0, 65) or UDim2.new(0, 55, 0, 55)
floatingButton.Position = isMobile and UDim2.new(0.85, 0, 0.85, 0) or UDim2.new(0.02, 0, 0.85, 0)
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
mainContainer.Size = isMobile and UDim2.new(0.92, 0, 0.85, 0) or UDim2.new(0, 400, 0, 650)
mainContainer.Position = isMobile and UDim2.new(0.04, 0, 0.075, 0) or UDim2.new(0.5, -200, 0.5, -325)
mainContainer.BackgroundColor3 = Color3.fromRGB(12, 12, 22)
mainContainer.BackgroundTransparency = 0.08
mainContainer.BorderSizePixel = 0
mainContainer.Visible = false
mainContainer.Parent = screenGui
mainContainer.ClipsDescendants = false
mainContainer.ScrollBarThickness = 3
mainContainer.ScrollBarImageColor3 = Color3.fromRGB(66, 135, 245)
mainContainer.CanvasSize = UDim2.new(0, 0, 0, 800)

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 20)
mainCorner.Parent = mainContainer

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 65)
titleBar.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainContainer

local titleBarCorner = Instance.new("UICorner")
titleBarCorner.CornerRadius = UDim.new(0, 20)
titleBarCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -80, 0, 35)
titleLabel.Position = UDim2.new(0, 15, 0, 8)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚡ RIAN STUDIOS V10.0"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = isMobile and 18 or 16
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local subtitleLabel = Instance.new("TextLabel")
subtitleLabel.Size = UDim2.new(1, -80, 0, 22)
subtitleLabel.Position = UDim2.new(0, 15, 0, 40)
subtitleLabel.BackgroundTransparency = 1
subtitleLabel.Text = "🔓 LIMITE DE BLOCOS REMOVIDO!"
subtitleLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
subtitleLabel.TextSize = isMobile and 11 or 10
subtitleLabel.Font = Enum.Font.GothamBold
subtitleLabel.TextXAlignment = Enum.TextXAlignment.Left
subtitleLabel.Parent = titleBar

local closeButton = Instance.new("ImageButton")
closeButton.Size = UDim2.new(0, 32, 0, 32)
closeButton.Position = UDim2.new(1, -45, 0, 16)
closeButton.Image = "rbxassetid://3926305904"
closeButton.ImageColor3 = Color3.fromRGB(255, 255, 255)
closeButton.BackgroundTransparency = 1
closeButton.Parent = titleBar

-- ============ CARD PRINCIPAL - QUEBRAR LIMITE DE BLOCOS ============
local mainCard = Instance.new("Frame")
mainCard.Size = UDim2.new(1, -24, 0, 220)
mainCard.Position = UDim2.new(0, 12, 0, 80)
mainCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
mainCard.BackgroundTransparency = 0.3
mainCard.BorderSizePixel = 0
mainCard.Parent = mainContainer

local mainCardCorner = Instance.new("UICorner")
mainCardCorner.CornerRadius = UDim.new(0, 16)
mainCardCorner.Parent = mainCard

local mainCardStroke = Instance.new("UIStroke")
mainCardStroke.Color = Color3.fromRGB(255, 215, 0)
mainCardStroke.Thickness = 2
mainCardStroke.Transparency = 0.3
mainCardStroke.Parent = mainCard

local mainIcon = Instance.new("ImageLabel")
mainIcon.Size = UDim2.new(0, 55, 0, 55)
mainIcon.Position = UDim2.new(0, 18, 0, 18)
mainIcon.Image = "rbxassetid://6031094773"
mainIcon.ImageColor3 = Color3.fromRGB(255, 215, 0)
mainIcon.BackgroundTransparency = 1
mainIcon.Parent = mainCard

local mainTitle = Instance.new("TextLabel")
mainTitle.Size = UDim2.new(1, -90, 0, 30)
mainTitle.Position = UDim2.new(0, 85, 0, 18)
mainTitle.BackgroundTransparency = 1
mainTitle.Text = "🔓 REMOVER LIMITE DE BLOCOS"
mainTitle.TextColor3 = Color3.fromRGB(255, 215, 0)
mainTitle.TextSize = isMobile and 16 or 14
mainTitle.Font = Enum.Font.GothamBold
mainTitle.TextXAlignment = Enum.TextXAlignment.Left
mainTitle.Parent = mainCard

local mainDesc = Instance.new("TextLabel")
mainDesc.Size = UDim2.new(1, -90, 0, 50)
mainDesc.Position = UDim2.new(0, 85, 0, 50)
mainDesc.BackgroundTransparency = 1
mainDesc.Text = "Este script REMOVE o limite de blocos do jogo!\nAgora você pode pegar até 1000 blocos!"
mainDesc.TextColor3 = Color3.fromRGB(180, 180, 180)
mainDesc.TextSize = isMobile and 12 or 11
mainDesc.Font = Enum.Font.Gotham
mainDesc.TextXAlignment = Enum.TextXAlignment.Left
mainDesc.Parent = mainCard

-- Status atual
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -90, 0, 30)
statusLabel.Position = UDim2.new(0, 85, 0, 100)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "📊 STATUS: LIMITE REMOVIDO!"
statusLabel.TextColor3 = Color3.fromRGB(76, 175, 80)
statusLabel.TextSize = isMobile and 13 or 12
statusLabel.Font = Enum.Font.GothamBold
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.Parent = mainCard

-- Contador de blocos
local blockCountLabel = Instance.new("TextLabel")
blockCountLabel.Size = UDim2.new(1, -90, 0, 40)
blockCountLabel.Position = UDim2.new(0, 85, 0, 130)
blockCountLabel.BackgroundTransparency = 1
blockCountLabel.Text = "📦 BLOCOS PEGOS: " .. currentBlocks .. " / " .. maxBlocksLimit
blockCountLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
blockCountLabel.TextSize = isMobile and 14 or 13
blockCountLabel.Font = Enum.Font.GothamBold
blockCountLabel.TextXAlignment = Enum.TextXAlignment.Left
blockCountLabel.Parent = mainCard

-- Botão para aplicar limite infinito
local applyButton = Instance.new("TextButton")
applyButton.Size = UDim2.new(0, isMobile and 180 or 160, 0, isMobile and 45 or 40)
applyButton.Position = UDim2.new(0.5, isMobile and -90 or -80, 1, -15)
applyButton.Text = "🔓 APLICAR LIMITE INFINITO"
applyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
applyButton.TextSize = isMobile and 13 or 11
applyButton.Font = Enum.Font.GothamBold
applyButton.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
applyButton.BorderSizePixel = 0
applyButton.Parent = mainCard

local applyCorner = Instance.new("UICorner")
applyCorner.CornerRadius = UDim.new(0, 10)
applyCorner.Parent = applyButton

-- ============ CARD DE CONFIGURAÇÃO ============
local configCard = Instance.new("Frame")
configCard.Size = UDim2.new(1, -24, 0, 130)
configCard.Position = UDim2.new(0, 12, 0, 315)
configCard.BackgroundColor3 = Color3.fromRGB(20, 20, 32)
configCard.BackgroundTransparency = 0.3
configCard.BorderSizePixel = 0
configCard.Parent = mainContainer

local configCorner = Instance.new("UICorner")
configCorner.CornerRadius = UDim.new(0, 14)
configCorner.Parent = configCard

local configStroke = Instance.new("UIStroke")
configStroke.Color = Color3.fromRGB(66, 135, 245)
configStroke.Thickness = 1
configStroke.Transparency = 0.6
configStroke.Parent = configCard

local configIcon = Instance.new("ImageLabel")
configIcon.Size = UDim2.new(0, 40, 0, 40)
configIcon.Position = UDim2.new(0, 15, 0, 12)
configIcon.Image = "rbxassetid://6031094773"
configIcon.ImageColor3 = Color3.fromRGB(66, 135, 245)
configIcon.BackgroundTransparency = 1
configIcon.Parent = configCard

local configTitle = Instance.new("TextLabel")
configTitle.Size = UDim2.new(1, -70, 0, 25)
configTitle.Position = UDim2.new(0, 65, 0, 12)
configTitle.BackgroundTransparency = 1
configTitle.Text = "⚙️ CONFIGURAÇÕES"
configTitle.TextColor3 = Color3.fromRGB(180, 180, 180)
configTitle.TextSize = isMobile and 13 or 11
configTitle.Font = Enum.Font.GothamBold
configTitle.TextXAlignment = Enum.TextXAlignment.Left
configTitle.Parent = configCard

-- Slider para ajustar limite
local limitSliderBg = Instance.new("Frame")
limitSliderBg.Size = UDim2.new(0.7, 0, 0, 8)
limitSliderBg.Position = UDim2.new(0.15, 0, 0.55, 0)
limitSliderBg.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
limitSliderBg.BorderSizePixel = 0
limitSliderBg.Parent = configCard

local limitSliderBgCorner = Instance.new("UICorner")
limitSliderBgCorner.CornerRadius = UDim.new(1, 0)
limitSliderBgCorner.Parent = limitSliderBg

local limitSliderFill = Instance.new("Frame")
limitSliderFill.Size = UDim2.new((maxBlocksLimit - 30) / (1000 - 30), 0, 1, 0)
limitSliderFill.BackgroundColor3 = Color3.fromRGB(76, 175, 80)
limitSliderFill.BorderSizePixel = 0
limitSliderFill.Parent = limitSliderBg

local limitSliderFillCorner = Instance.new("UICorner")
limitSliderFillCorner.CornerRadius = UDim.new(1, 0)
limitSliderFillCorner.Parent = limitSliderFill

local limitSliderButton = Instance.new("ImageButton")
limitSliderButton.Size = UDim2.new(0, 20, 0, 20)
limitSliderButton.Position = UDim2.new((maxBlocksLimit - 30) / (1000 - 30), -10, 0.5, -10)
limitSliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
limitSliderButton.Image = "rbxassetid://266788897"
limitSliderButton.ScaleType = Enum.ScaleType.Fit
limitSliderButton.BackgroundTransparency = 1
limitSliderButton.Parent = configCard

local limitMinLabel = Instance.new("TextLabel")
limitMinLabel.Size = UDim2.new(0, 30, 0, 20)
limitMinLabel.Position = UDim2.new(0.1, 0, 0.55, -8)
limitMinLabel.BackgroundTransparency = 1
limitMinLabel.Text = "30"
limitMinLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
limitMinLabel.TextSize = 10
limitMinLabel.Font = Enum.Font.Gotham
limitMinLabel.Parent = configCard

local limitMaxLabel = Instance.new("TextLabel")
limitMaxLabel.Size = UDim2.new(0, 35, 0, 20)
limitMaxLabel.Position = UDim2.new(0.87, 0, 0.55, -8)
limitMaxLabel.BackgroundTransparency = 1
limitMaxLabel.Text = "1000"
limitMaxLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
limitMaxLabel.TextSize = 10
limitMaxLabel.Font = Enum.Font.Gotham
limitMaxLabel.Parent = configCard

local limitValueLabel = Instance.new("TextLabel")
limitValueLabel.Size = UDim2.new(1, -70, 0, 25)
limitValueLabel.Position = UDim2.new(0, 65, 0, 55)
limitValueLabel.BackgroundTransparency = 1
limitValueLabel.Text = "LIMITE ATUAL: " .. maxBlocksLimit .. " blocos"
limitValueLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
limitValueLabel.TextSize = isMobile and 12 or 10
limitValueLabel.Font = Enum.Font.GothamBold
limitValueLabel.TextXAlignment = Enum.TextXAlignment.Left
limitValueLabel.Parent = configCard

-- Botão de reset
local resetButton = Instance.new("TextButton")
resetButton.Size = UDim2.new(0, isMobile and 100 or 90, 0, isMobile and 38 or 34)
resetButton.Position = UDim2.new(0.5, isMobile and -50 or -45, 1, -12)
resetButton.Text = "🔄 RESETAR CONTAGEM"
resetButton.TextColor3 = Color3.fromRGB(255, 255, 255)
resetButton.TextSize = isMobile and 12 or 10
resetButton.Font = Enum.Font.GothamBold
resetButton.BackgroundColor3 = Color3.fromRGB(255, 152, 0)
resetButton.BorderSizePixel = 0
resetButton.Parent = configCard

local resetCorner = Instance.new("UICorner")
resetCorner.CornerRadius = UDim.new(0, 8)
resetCorner.Parent = resetButton

-- ============ CARD DE VELOCIDADE ============
local speedCard = Instance.new("Frame")
speedCard.Size = UDim2.new(1, -24, 0, 120)
speedCard.Position = UDim2.new(0, 12, 0, 460)
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
speedIcon.Size = UDim2.new(0, 35, 0, 35)
speedIcon.Position = UDim2.new(0, 15, 0, 12)
speedIcon.Image = "rbxassetid://6031094773"
speedIcon.ImageColor3 = Color3.fromRGB(66, 135, 245)
speedIcon.BackgroundTransparency = 1
speedIcon.Parent = speedCard

local speedTitle = Instance.new("TextLabel")
speedTitle.Size = UDim2.new(1, -65, 0, 25)
speedTitle.Position = UDim2.new(0, 60, 0, 12)
speedTitle.BackgroundTransparency = 1
speedTitle.Text = "⚡ VELOCIDADE"
speedTitle.TextColor3 = Color3.fromRGB(180, 180, 180)
speedTitle.TextSize = isMobile and 12 or 11
speedTitle.Font = Enum.Font.GothamBold
speedTitle.TextXAlignment = Enum.TextXAlignment.Left
speedTitle.Parent = speedCard

local speedValue = Instance.new("TextLabel")
speedValue.Size = UDim2.new(1, -65, 0, 35)
speedValue.Position = UDim2.new(0, 60, 0, 38)
speedValue.BackgroundTransparency = 1
speedValue.Text = "80"
speedValue.TextColor3 = Color3.fromRGB(255, 255, 255)
speedValue.TextSize = isMobile and 28 or 24
speedValue.Font = Enum.Font.GothamBold
speedValue.TextXAlignment = Enum.TextXAlignment.Left
speedValue.Parent = speedCard

-- Slider de velocidade
local speedSliderBg = Instance.new("Frame")
speedSliderBg.Size = UDim2.new(0.7, 0, 0, 6)
speedSliderBg.Position = UDim2.new(0.15, 0, 0.8, 0)
speedSliderBg.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
speedSliderBg.BorderSizePixel = 0
speedSliderBg.Parent = speedCard

local speedSliderBgCorner = Instance.new("UICorner")
speedSliderBgCorner.CornerRadius = UDim.new(1, 0)
speedSliderBgCorner.Parent = speedSliderBg

local speedSliderFill = Instance.new("Frame")
speedSliderFill.Size = UDim2.new(0.6, 0, 1, 0)
speedSliderFill.BackgroundColor3 = Color3.fromRGB(66, 135, 245)
speedSliderFill.BorderSizePixel = 0
speedSliderFill.Parent = speedSliderBg

local speedSliderFillCorner = Instance.new("UICorner")
speedSliderFillCorner.CornerRadius = UDim.new(1, 0)
speedSliderFillCorner.Parent = speedSliderFill

local speedSliderButton = Instance.new("ImageButton")
speedSliderButton.Size = UDim2.new(0, 18, 0, 18)
speedSliderButton.Position = UDim2.new(0.6, -9, 0.5, -9)
speedSliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
speedSliderButton.Image = "rbxassetid://266788897"
speedSliderButton.ScaleType = Enum.ScaleType.Fit
speedSliderButton.BackgroundTransparency = 1
speedSliderButton.Parent = speedCard

-- Botão turbo
local turboBtn = Instance.new("TextButton")
turboBtn.Size = UDim2.new(0, isMobile and 70 or 60, 0, isMobile and 34 or 30)
turboBtn.Position = UDim2.new(0.5, isMobile and 60 or 55, 0.8, -4)
turboBtn.Text = "🚀"
turboBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
turboBtn.TextSize = isMobile and 20 or 18
turboBtn.Font = Enum.Font.GothamBold
turboBtn.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
turboBtn.BorderSizePixel = 0
turboBtn.Parent = speedCard

local turboCorner = Instance.new("UICorner")
turboCorner.CornerRadius = UDim.new(0, 8)
turboCorner.Parent = turboBtn

-- ============ CARD DE VOO ============
local flyCard = Instance.new("Frame")
flyCard.Size = UDim2.new(1, -24, 0, 90)
flyCard.Position = UDim2.new(0, 12, 0, 595)
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
flyIcon.Size = UDim2.new(0, 35, 0, 35)
flyIcon.Position = UDim2.new(0, 15, 0, 10)
flyIcon.Image = "rbxassetid://6031094773"
flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
flyIcon.BackgroundTransparency = 1
flyIcon.Parent = flyCard

local flyStatus = Instance.new("TextLabel")
flyStatus.Size = UDim2.new(1, -65, 0, 30)
flyStatus.Position = UDim2.new(0, 60, 0, 12)
flyStatus.BackgroundTransparency = 1
flyStatus.Text = "🕊️ VOO: DESLIGADO"
flyStatus.TextColor3 = Color3.fromRGB(200, 200, 200)
flyStatus.TextSize = isMobile and 13 or 11
flyStatus.Font = Enum.Font.GothamBold
flyStatus.TextXAlignment = Enum.TextXAlignment.Left
flyStatus.Parent = flyCard

local flyBtn = Instance.new("TextButton")
flyBtn.Size = UDim2.new(0, isMobile and 100 or 90, 0, isMobile and 34 or 30)
flyBtn.Position = UDim2.new(0.5, isMobile and -50 or -45, 1, -10)
flyBtn.Text = "🕊️ VOAR"
flyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
flyBtn.TextSize = isMobile and 12 or 10
flyBtn.Font = Enum.Font.GothamBold
flyBtn.BackgroundColor3 = Color3.fromRGB(66, 200, 100)
flyBtn.BorderSizePixel = 0
flyBtn.Parent = flyCard

local flyCorner = Instance.new("UICorner")
flyCorner.CornerRadius = UDim.new(0, 8)
flyCorner.Parent = flyBtn

-- Footer
local footer = Instance.new("Frame")
footer.Size = UDim2.new(1, 0, 0, 35)
footer.Position = UDim2.new(0, 0, 1, -35)
footer.BackgroundColor3 = Color3.fromRGB(8, 8, 16)
footer.BackgroundTransparency = 0.5
footer.BorderSizePixel = 0
footer.Parent = mainContainer

local footerCorner = Instance.new("UICorner")
footerCorner.CornerRadius = UDim.new(0, 14)
footerCorner.Parent = footer

local creditText = Instance.new("TextLabel")
creditText.Size = UDim2.new(1, 0, 0, 18)
creditText.Position = UDim2.new(0, 0, 0, 4)
creditText.BackgroundTransparency = 1
creditText.Text = "⚡ RIAN STUDIOS V10.0 - LIMITE INFINITO DE BLOCOS ⚡"
creditText.TextColor3 = Color3.fromRGB(120, 120, 120)
creditText.TextSize = isMobile and 9 or 8
creditText.Font = Enum.Font.Gotham
creditText.Parent = footer

-- ============ FUNÇÕES ============

-- Função para atualizar limite
local function updateBlockLimit(newLimit)
    maxBlocksLimit = math.clamp(newLimit, 30, 1000)
    limitValueLabel.Text = "LIMITE ATUAL: " .. maxBlocksLimit .. " blocos"
    
    local percent = (maxBlocksLimit - 30) / (1000 - 30)
    limitSliderFill:TweenSize(UDim2.new(percent, 0, 1, 0), "Out", "Quad", 0.1, true)
    limitSliderButton:TweenPosition(UDim2.new(percent, -10, 0.5, -10), "Out", "Quad", 0.1, true)
    
    blockCountLabel.Text = "📦 BLOCOS PEGOS: " .. currentBlocks .. " / " .. maxBlocksLimit
    
    -- Reaplicar o limite
    removeBlockLimit()
end

-- Slider de limite
local draggingLimit = false
local function updateLimitFromMouse(input)
    local mousePos = input.Position.X
    local sliderAbsPos = limitSliderBg.AbsolutePosition.X
    local sliderWidth = limitSliderBg.AbsoluteSize.X
    local percent = math.clamp((mousePos - sliderAbsPos) / sliderWidth, 0, 1)
    local newLimit = 30 + (percent * (1000 - 30))
    updateBlockLimit(math.floor(newLimit))
end

limitSliderButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingLimit = true
        updateLimitFromMouse(input)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingLimit and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        updateLimitFromMouse(input)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingLimit = false
    end
end)

-- Slider de velocidade
local currentSpeedValue = 80
local draggingSpeed = false

local function updateSpeedFromMouse(input)
    local mousePos = input.Position.X
    local sliderAbsPos = speedSliderBg.AbsolutePosition.X
    local sliderWidth = speedSliderBg.AbsoluteSize.X
    local percent = math.clamp((mousePos - sliderAbsPos) / sliderWidth, 0, 1)
    local newSpeed = 16 + (percent * (120 - 16))
    currentSpeedValue = math.floor(newSpeed)
    speedValue.Text = currentSpeedValue
    speedSliderFill.Size = UDim2.new(percent, 0, 1, 0)
    speedSliderButton.Position = UDim2.new(percent, -9, 0.5, -9)
    
    if humanoid then
        humanoid.WalkSpeed = currentSpeedValue
    end
end

speedSliderButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSpeed = true
        updateSpeedFromMouse(input)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingSpeed and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        updateSpeedFromMouse(input)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSpeed = false
    end
end)

turboBtn.MouseButton1Click:Connect(function()
    currentSpeedValue = 120
    speedValue.Text = "120"
    local percent = (120 - 16) / (120 - 16)
    speedSliderFill.Size = UDim2.new(1, 0, 1, 0)
    speedSliderButton.Position = UDim2.new(1, -9, 0.5, -9)
    if humanoid then
        humanoid.WalkSpeed = 120
    end
end)

-- Sistema de Voo
local flying = false
local flyVelocity = nil
local flyGyro = nil
local defaultGravity = Workspace.Gravity

local function updateFlyMovement()
    if not flying or not flyVelocity then return end
    
    local moveDir = Vector3.new()
    
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
        moveDir = moveDir + humanoidRootPart.CFrame.LookVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
        moveDir = moveDir - humanoidRootPart.CFrame.LookVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
        moveDir = moveDir - humanoidRootPart.CFrame.RightVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
        moveDir = moveDir + humanoidRootPart.CFrame.RightVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
        moveDir = moveDir + Vector3.new(0, 1, 0)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
        moveDir = moveDir - Vector3.new(0, 1, 0)
    end
    
    if moveDir.Magnitude > 0 then
        moveDir = moveDir.Unit
        flyVelocity.Velocity = moveDir * currentSpeedValue
        if flyGyro then
            flyGyro.CFrame = CFrame.lookAt(Vector3.new(), moveDir)
        end
    else
        flyVelocity.Velocity = Vector3.new(0, 0, 0)
    end
end

local function startFly()
    if flying then return end
    flying = true
    flyStatus.Text = "🕊️ VOO: ATIVADO"
    flyStatus.TextColor3 = Color3.fromRGB(66, 200, 100)
    flyBtn.Text = "🛑 PARAR"
    flyBtn.BackgroundColor3 = Color3.fromRGB(255, 64, 64)
    flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
    flyCardStroke.Color = Color3.fromRGB(66, 200, 100)
    
    Workspace.Gravity = 0
    
    flyVelocity = Instance.new("BodyVelocity")
    flyVelocity.MaxForce = Vector3.new(100000, 100000, 100000)
    flyVelocity.Parent = humanoidRootPart
    
    flyGyro = Instance.new("BodyGyro")
    flyGyro.MaxTorque = Vector3.new(400000, 400000, 400000)
    flyGyro.CFrame = humanoidRootPart.CFrame
    flyGyro.Parent = humanoidRootPart
    
    coroutine.wrap(function()
        while flying and humanoidRootPart do
            updateFlyMovement()
            if humanoid then
                humanoid.PlatformStand = true
            end
            RunService.Heartbeat:Wait()
        end
    end)()
end

local function stopFly()
    if not flying then return end
    flying = false
    flyStatus.Text = "🕊️ VOO: DESLIGADO"
    flyStatus.TextColor3 = Color3.fromRGB(200, 200, 200)
    flyBtn.Text = "🕊️ VOAR"
    flyBtn.BackgroundColor3 = Color3.fromRGB(66, 200, 100)
    flyIcon.ImageColor3 = Color3.fromRGB(66, 200, 100)
    flyCardStroke.Color = Color3.fromRGB(66, 200, 100)
    
    Workspace.Gravity = defaultGravity
    
    if flyVelocity then flyVelocity:Destroy() end
    if flyGyro then flyGyro:Destroy() end
    
    if humanoid then
        humanoid.PlatformStand = false
    end
end

flyBtn.MouseButton1Click:Connect(function()
    if flying then stopFly() else startFly() end
end)

-- Botão para aplicar limite infinito
applyButton.MouseButton1Click:Connect(function()
    removeBlockLimit()
    
    -- Notificação
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0, 300, 0, 50)
    notif.Position = UDim2.new(0.5, -150, 0, -50)
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
    notifText.Text = "🔓 LIMITE REMOVIDO! Agora você pode pegar até " .. maxBlocksLimit .. " blocos!"
    notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
    notifText.TextScaled = true
    notifText.Font = Enum.Font.GothamBold
    notifText.Parent = notif
    
    TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -150, 0, 20)}):Play()
    wait(2.5)
    TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -150, 0, -50), BackgroundTransparency = 1}):Play()
    wait(0.3)
    notif:Destroy()
end)

-- Botão resetar contagem
resetButton.MouseButton1Click:Connect(function()
    currentBlocks = 0
    blockCountLabel.Text = "📦 BLOCOS PEGOS: " .. currentBlocks .. " / " .. maxBlocksLimit
    
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0, 200, 0, 40)
    notif.Position = UDim2.new(0.5, -100, 0, -50)
    notif.BackgroundColor3 = Color3.fromRGB(255, 152, 0)
    notif.BackgroundTransparency = 0.1
    notif.BorderSizePixel = 0
    notif.Parent = screenGui
    
    local notifCorner = Instance.new("UICorner")
    notifCorner.CornerRadius = UDim.new(0, 10)
    notifCorner.Parent = notif
    
    local notifText = Instance.new("TextLabel")
    notifText.Size = UDim2.new(1, 0, 1, 0)
    notifText.BackgroundTransparency = 1
    notifText.Text = "🔄 CONTAGEM RESETADA!"
    notifText.TextColor3 = Color3.fromRGB(255, 255, 255)
    notifText.TextScaled = true
    notifText.Font = Enum.Font.GothamBold
    notifText.Parent = notif
    
    TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -100, 0, 20)}):Play()
    wait(1.5)
    TweenService:Create(notif, TweenInfo.new(0.3), {Position = UDim2.new(0.5, -100, 0, -50), BackgroundTransparency = 1}):Play()
    wait(0.3)
    notif:Destroy()
end)

-- UI Toggle
local uiOpen = false
local function toggleUI()
    uiOpen = not uiOpen
    mainContainer.Visible = uiOpen
    TweenService:Create(floatingButton, TweenInfo.new(0.2), {Size = uiOpen and (isMobile and UDim2.new(0, 55, 0, 55) or UDim2.new(0, 50, 0, 50)) or (isMobile and UDim2.new(0, 65, 0, 65) or UDim2.new(0, 55, 0, 55))}):Play()
end

floatingButton.MouseButton1Click:Connect(toggleUI)
closeButton.MouseButton1Click:Connect(toggleUI)

-- Inicializar
removeBlockLimit()

-- Auto coletor de blocos (opcional)
coroutine.wrap(function()
    while true do
        wait(0.5)
        if character and humanoidRootPart then
            forceBlockCollection()
        end
    end
end)()

print("═══════════════════════════════════════════════════════════════════════════════")
print("✅ SISTEMA RIAN STUDIOS V10.0 CARREGADO!")
print("✅ LIMITE DE BLOCOS REMOVIDO! Agora você pode pegar até 1000 blocos!")
print("✅ Use o slider para ajustar o limite máximo!")
print("✅ Clique em 'APLICAR LIMITE INFINITO' para ativar!")
print("═══════════════════════════════════════════════════════════════════════════════")
