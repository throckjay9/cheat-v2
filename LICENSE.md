-- Configurações
local ESP_ENABLED = false
local AIMBOT_ENABLED = false
local ACTIVATE_KEY = Enum.KeyCode.Q

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0.2, 0, 0.3, 0)
Frame.Position = UDim2.new(0.05, 0, 0.7, 0)
Frame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
Frame.BackgroundTransparency = 0.5
Frame.Visible = false
Frame.Parent = ScreenGui

local ToggleESP = Instance.new("TextButton")
ToggleESP.Size = UDim2.new(0.8, 0, 0.3, 0)
ToggleESP.Position = UDim2.new(0.1, 0, 0.1, 0)
ToggleESP.Text = "ESP: OFF"
ToggleESP.TextColor3 = Color3.new(1, 1, 1)
ToggleESP.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
ToggleESP.Parent = Frame

local ToggleAimbot = Instance.new("TextButton")
ToggleAimbot.Size = UDim2.new(0.8, 0, 0.3, 0)
ToggleAimbot.Position = UDim2.new(0.1, 0, 0.5, 0)
ToggleAimbot.Text = "Aimbot: OFF"
ToggleAimbot.TextColor3 = Color3.new(1, 1, 1)
ToggleAimbot.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
ToggleAimbot.Parent = Frame

-- Variáveis do ESP
local ESPObjects = {}

-- Função para desenhar ESP (Nome, Linha, Caixa)
local function DrawESP(player)
    if ESPObjects[player] then return end -- Evita duplicar ESP

    local character = player.Character
    if not character then return end

    local head = character:FindFirstChild("Head")
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not head or not humanoidRootPart then return end

    -- Nome
    local BillboardGui = Instance.new("BillboardGui")
    BillboardGui.Name = "ESP_Name"
    BillboardGui.Adornee = head
    BillboardGui.Size = UDim2.new(0, 100, 0, 40)
    BillboardGui.StudsOffset = Vector3.new(0, 2.5, 0)
    BillboardGui.AlwaysOnTop = true
    BillboardGui.Parent = head

    local TextLabel = Instance.new("TextLabel")
    TextLabel.Size = UDim2.new(1, 0, 1, 0)
    TextLabel.BackgroundTransparency = 1
    TextLabel.Text = player.Name
    TextLabel.TextColor3 = Color3.new(1, 1, 1)
    TextLabel.TextStrokeTransparency = 0.5
    TextLabel.Parent = BillboardGui

    -- Linha (Tracer)
    local Line = Instance.new("LineHandleAdornment")
    Line.Name = "ESP_Line"
    Line.Adornee = humanoidRootPart
    Line.Length = (humanoidRootPart.Position - Camera.CFrame.Position).Magnitude
    Line.Thickness = 1
    Line.Color3 = Color3.new(1, 0, 0)
    Line.Transparency = 0.5
    Line.ZIndex = 1
    Line.Parent = humanoidRootPart

    -- Caixa (Box)
    local Box = Instance.new("BoxHandleAdornment")
    Box.Name = "ESP_Box"
    Box.Adornee = humanoidRootPart
    Box.Size = Vector3.new(2, 3, 1)
    Box.Color3 = Color3.new(0, 1, 0)
    Box.Transparency = 0.7
    Box.ZIndex = 0
    Box.Parent = humanoidRootPart

    ESPObjects[player] = {BillboardGui, Line, Box}
end

-- Função para remover ESP
local function ClearESP()
    for player, objects in pairs(ESPObjects) do
        for _, obj in ipairs(objects) do
            if obj and obj.Parent then
                obj:Destroy()
            end
        end
    end
    ESPObjects = {}
end

-- Aimbot (Mira na cabeça do inimigo mais próximo)
local function UpdateAimbot()
    if not AIMBOT_ENABLED or not LocalPlayer.Character then return end

    local closestPlayer = nil
    local shortestDistance = math.huge
    local currentCameraPos = Camera.CFrame.Position

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local head = player.Character:FindFirstChild("Head")
            if head then
                local distance = (head.Position - currentCameraPos).Magnitude
                if distance < shortestDistance then
                    shortestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end

    if closestPlayer and closestPlayer.Character then
        local head = closestPlayer.Character:FindFirstChild("Head")
        if head then
            -- Mira na cabeça (ajusta a câmera)
            Camera.CFrame = CFrame.lookAt(Camera.CFrame.Position, head.Position)
        end
    end
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

    UpdateAimbot()
end)

-- Tecla Q para abrir/fechar GUI (corrigido)
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
