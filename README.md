-- SpeedToggle.local.lua
-- Coloque este LocalScript em StarterPlayerScripts (recomendado) ou em StarterGui.
-- Cria um botão GUI que ativa/desativa um aumento de velocidade para o jogador local.

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- Configurações
local BOOST_SPEED = 40        -- velocidade quando ligado
local TOGGLE_KEY = Enum.KeyCode.V -- tecla para alternar (opcional)
local BUTTON_SIZE = UDim2.new(0, 140, 0, 44)

-- Estado
local enabled = false
local debounce = false
local defaultSpeed = nil

-- Cria GUI no PlayerGui do jogador
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SpeedToggleGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local button = Instance.new("TextButton")
button.Name = "SpeedToggleButton"
button.Size = BUTTON_SIZE
button.Position = UDim2.new(0.5, -BUTTON_SIZE.X.Offset/2, 0.9, -BUTTON_SIZE.Y.Offset/2)
button.AnchorPoint = Vector2.new(0.5, 0.5)
button.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
button.BorderSizePixel = 0
button.TextColor3 = Color3.new(1,1,1)
button.Font = Enum.Font.SourceSansBold
button.TextSize = 20
button.Text = "Velocidade: Off"
button.Parent = screenGui

-- Função para atualizar a aparência do botão
local function updateButton()
    if enabled then
        button.Text = "Velocidade: On (" .. tostring(BOOST_SPEED) .. ")"
        TweenService:Create(button, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(0, 170, 0)}):Play()
    else
        button.Text = "Velocidade: Off"
        TweenService:Create(button, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(170, 0, 0)}):Play()
    end
end

-- Aplica a velocidade ao Humanoid atual (se existir)
local function applySpeedToHumanoid(humanoid)
    if not humanoid then return end
    -- se defaultSpeed ainda não foi definido, armazena o valor atual do jogo
    if not defaultSpeed then
        defaultSpeed = humanoid.WalkSpeed or 16
    end
    if enabled then
        humanoid.WalkSpeed = BOOST_SPEED
    else
        humanoid.WalkSpeed = defaultSpeed
    end
end

-- Alterna o estado de velocidade
local function toggleSpeed()
    if debounce then return end
    debounce = true
    enabled = not enabled

    local character = player.Character
    if character then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            applySpeedToHumanoid(humanoid)
        end
    end

    updateButton()
    wait(0.2)
    debounce = false
end

-- Conecta clique do botão
button.MouseButton1Click:Connect(toggleSpeed)

-- Atalho de teclado (V por padrão)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == TOGGLE_KEY then
        toggleSpeed()
    end
end)

-- Ao spawnar o personagem, aplica velocidade conforme o estado atual
local function onCharacterAdded(character)
    local humanoid = character:WaitForChild("Humanoid", 5)
    if humanoid then
        -- atualiza defaultSpeed se ainda não foi definido
        if not defaultSpeed then
            defaultSpeed = humanoid.WalkSpeed or 16
        end
        -- aplica velocidade de acordo com 'enabled'
        applySpeedToHumanoid(humanoid)
    end
end

player.CharacterAdded:Connect(onCharacterAdded)
if player.Character then
    onCharacterAdded(player.Character)
end

-- Final: atualiza visual imediatamente
updateButton()
