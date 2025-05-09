-- Configurações do ESP
local ESP_SETTINGS = {
    Enabled = false,
    TeamColor = true, -- Se true, usa a cor do time. Se false, usa cor fixa.
    DefaultColor = Color3.new(1, 0, 0), -- Vermelho (se TeamColor = false)
    ShowDistance = true, -- Mostra distância do jogador
    ShowName = true, -- Mostra nome do jogador
    RefreshRate = 0.1, -- Atualização a cada 0.1 segundos
}

-- Variáveis
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

local ESPCache = {} -- Armazena os ESPs criados

-- Função para criar um ESP para um jogador
local function createESP(player)
    if ESPCache[player] then return end -- Evita duplicar ESP

    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

    -- Cria um BillboardGui (texto flutuante)
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_" .. player.Name
    billboard.Adornee = humanoidRootPart
    billboard.Size = UDim2.new(0, 200, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = character

    -- Texto do nome e distância
    local espText = Instance.new("TextLabel")
    espText.Name = "ESPLabel"
    espText.Size = UDim2.new(1, 0, 0.5, 0)
    espText.BackgroundTransparency = 1
    espText.TextColor3 = ESP_SETTINGS.TeamColor and player.TeamColor.Color or ESP_SETTINGS.DefaultColor
    espText.TextStrokeTransparency = 0.5
    espText.TextSize = 14
    espText.Font = Enum.Font.SourceSansBold
    espText.Text = player.Name
    espText.Parent = billboard

    -- Texto da distância (opcional)
    if ESP_SETTINGS.ShowDistance then
        local distanceText = Instance.new("TextLabel")
        distanceText.Name = "DistanceLabel"
        distanceText.Size = UDim2.new(1, 0, 0.5, 0)
        distanceText.Position = UDim2.new(0, 0, 0.5, 0)
        distanceText.BackgroundTransparency = 1
        distanceText.TextColor3 = Color3.new(1, 1, 1)
        distanceText.TextStrokeTransparency = 0.5
        distanceText.TextSize = 12
        distanceText.Font = Enum.Font.SourceSans
        distanceText.Parent = billboard
    end

    -- Cria um Highlight (contorno)
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_Highlight"
    highlight.Adornee = character
    highlight.FillTransparency = 0.8
    highlight.OutlineColor = ESP_SETTINGS.TeamColor and player.TeamColor.Color or ESP_SETTINGS.DefaultColor
    highlight.OutlineTransparency = 0
    highlight.Parent = character

    ESPCache[player] = {
        Billboard = billboard,
        Highlight = highlight,
        Character = character
    }
end

-- Função para remover ESP
local function removeESP(player)
    if not ESPCache[player] then return end
    ESPCache[player].Billboard:Destroy()
    ESPCache[player].Highlight:Destroy()
    ESPCache[player] = nil
end

-- Atualiza ESP (distância, etc.)
local function updateESP()
    for player, data in pairs(ESPCache) do
        if player.Character and data.Character and data.Character:IsDescendantOf(workspace) then
            if ESP_SETTINGS.ShowDistance then
                local distance = (LocalPlayer.Character.HumanoidRootPart.Position - data.Character.HumanoidRootPart.Position).Magnitude
                data.Billboard.DistanceLabel.Text = string.format("[%.1f studs]", distance)
            end
        else
            removeESP(player)
        end
    end
end

-- Ativa/Desativa ESP
local function toggleESP(enable)
    ESP_SETTINGS.Enabled = enable
    if enable then
        -- Adiciona ESP para todos os jogadores
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                createESP(player)
            end
        end
        -- Conecta novos jogadores
        Players.PlayerAdded:Connect(function(player)
            createESP(player)
        end)
        -- Atualiza continuamente
        RunService.Heartbeat:Connect(updateESP)
    else
        -- Remove todos os ESPs
        for player in pairs(ESPCache) do
            removeESP(player)
        end
    end
end

-- Comando para ativar/desativar (ex: "/esp on")
LocalPlayer.Chatted:Connect(function(message)
    if message:lower() == "/esp on" then
        toggleESP(true)
    elseif message:lower() == "/esp off" then
        toggleESP(false)
    end
end)

-- Inicializa se já estiver ativado
if ESP_SETTINGS.Enabled then
    toggleESP(true)
end
