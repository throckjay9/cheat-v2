-- Configurações
local ESP_ENABLED = false
local AIMBOT_ENABLED = false
local ACTIVATE_KEY = Enum.KeyCode.Q

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0.2, 0, 0.3, 0)
Frame.Position = UDim2.new(0.05, 0, 0.7, 0)
Frame.BackgroundTransparency = 0.5
Frame.Visible = false
Frame.Parent = ScreenGui

local ToggleESP = Instance.new("TextButton")
ToggleESP.Size = UDim2.new(0.8, 0, 0.3, 0)
ToggleESP.Position = UDim2.new(0.1, 0, 0.1, 0)
ToggleESP.Text = "ESP: OFF"
ToggleESP.Parent = Frame

local ToggleAimbot = Instance.new("TextButton")
ToggleAimbot.Size = UDim2.new(0.8, 0, 0.3, 0)
ToggleAimbot.Position = UDim2.new(0.1, 0, 0.5, 0)
ToggleAimbot.Text = "Aimbot: OFF"
ToggleAimbot.Parent = Frame

-- Função para desenhar ESP (nomes)
local function DrawESP(player)
    local character = player.Character
    if character and character:FindFirstChild("Head") then
        local head = character.Head
        local BillboardGui = Instance.new("BillboardGui")
        BillboardGui.Adornee = head
        BillboardGui.Size = UDim2.new(0, 100, 0, 50)
        BillboardGui.StudsOffset = Vector3.new(0, 1.5, 0)
        BillboardGui.AlwaysOnTop = true
        BillboardGui.Parent = head

        local TextLabel = Instance.new("TextLabel")
        TextLabel.Size = UDim2.new(1, 0, 1, 0)
        TextLabel.BackgroundTransparency = 1
        TextLabel.Text = player.Name
        TextLabel.TextColor3 = Color3.new(1, 1, 1)
        TextLabel.Parent = BillboardGui
    end
end

-- Função para remover ESP
local function ClearESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            for _, child in ipairs(head:GetChildren()) do
                if child:IsA("BillboardGui") then
                    child:Destroy()
                end
            end
        end
    end
end

-- Aimbot Simulado (mira no jogador mais próximo)
local function FindClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local distance = (player.Character.Head.Position - LocalPlayer.Character.Head.Position).Magnitude
            if distance < shortestDistance then
                shortestDistance = distance
                closestPlayer = player
            end
        end
    end

    return closestPlayer
end

-- Atualizar ESP e Aimbot
RunService.RenderStepped:Connect(function()
    if ESP_ENABLED then
        ClearESP()
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                DrawESP(player)
            end
        end
    end

    if AIMBOT_ENABLED then
        local target = FindClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local head = target.Character.Head
            -- Simula mira (não é um aimbot real)
            LocalPlayer.Character.Humanoid:MoveTo(head.Position)
        end
    end
end)

-- Tecla Q para abrir/fechar GUI
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == ACTIVATE_KEY and not gameProcessed then
        Frame.Visible = not Frame.Visible
    end
end)

-- Botões da GUI
ToggleESP.MouseButton1Click:Connect(function()
    ESP_ENABLED = not ESP_ENABLED
    ToggleESP.Text = ESP_ENABLED and "ESP: ON" or "ESP: OFF"
    if not ESP_ENABLED then
        ClearESP()
    end
end)

ToggleAimbot.MouseButton1Click:Connect(function()
    AIMBOT_ENABLED = not AIMBOT_ENABLED
    ToggleAimbot.Text = AIMBOT_ENABLED and "Aimbot: ON" or "Aimbot: OFF"
end)
