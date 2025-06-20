--✅ ESP Nick {Team} + Seta Amarela para Amigos
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Tabela para armazenar ESPs ativos
local ESPs = {}

-- Função para criar ESP de texto
local function CreateESP(player)
    if player == LocalPlayer then return end
    if ESPs[player] then ESPs[player]:Destroy() end

    local Billboard = Instance.new("BillboardGui")
    Billboard.Adornee = player.Character and player.Character:FindFirstChild("Head")
    Billboard.Parent = game.CoreGui
    Billboard.Size = UDim2.new(0, 200, 0, 50)
    Billboard.AlwaysOnTop = true

    local Text = Instance.new("TextLabel")
    Text.Parent = Billboard
    Text.Size = UDim2.new(1, 0, 1, 0)
    Text.BackgroundTransparency = 1
    Text.TextStrokeTransparency = 0
    Text.TextSize = 12
    Text.Font = Enum.Font.SourceSansBold
    Text.TextColor3 = Color3.new(0, 1, 0) -- Verde
    Text.TextStrokeColor3 = Color3.new(0, 0, 0)

    local function Update()
        if player.Character and player.Character:FindFirstChild("Head") then
            Billboard.Adornee = player.Character.Head
            local teamColor = player.Team and player.Team.TeamColor.Color or Color3.new(1, 1, 1)
            Text.Text = player.Name.." {"..(player.Team and player.Team.Name or "NoTeam").."}"
            Text.TextColor3 = Color3.new(0, 1, 0)
            Text.TextStrokeColor3 = teamColor
        else
            Billboard.Adornee = nil
        end
    end

    ESPs[player] = Billboard

    local Connection
    Connection = RunService.RenderStepped:Connect(function()
        if not player or not player.Parent then
            Connection:Disconnect()
            if ESPs[player] then
                ESPs[player]:Destroy()
                ESPs[player] = nil
            end
        else
            Update()
        end
    end)
end

-- Função para criar a seta amarela apontando para amigos
local function CreateArrow(friend)
    local Arrow = Instance.new("BillboardGui")
    Arrow.Adornee = friend.Character and friend.Character:FindFirstChild("Head")
    Arrow.Parent = game.CoreGui
    Arrow.Size = UDim2.new(0, 100, 0, 100)
    Arrow.AlwaysOnTop = true

    local Image = Instance.new("ImageLabel")
    Image.Parent = Arrow
    Image.Size = UDim2.new(1, 0, 1, 0)
    Image.BackgroundTransparency = 1
    Image.Image = "rbxassetid://7072718365" -- Setinha amarela (pode trocar o asset ID se quiser)
    Image.ImageColor3 = Color3.new(1, 1, 0) -- Amarelo

    local function UpdateArrow()
        if friend.Character and friend.Character:FindFirstChild("Head") then
            Arrow.Adornee = friend.Character.Head
        else
            Arrow.Adornee = nil
        end
    end

    local Connection
    Connection = RunService.RenderStepped:Connect(function()
        if not friend or not friend.Parent then
            Connection:Disconnect()
            if Arrow then Arrow:Destroy() end
        else
            UpdateArrow()
        end
    end)
end

-- Atualizar ESPs sempre que entrar/sair players
local function RefreshESPs()
    for _,v in pairs(ESPs) do
        if v then v:Destroy() end
    end
    ESPs = {}

    for _,player in pairs(Players:GetPlayers()) do
        CreateESP(player)

        -- Checar se é amigo
        if LocalPlayer:IsFriendsWith(player.UserId) then
            CreateArrow(player)
        end
    end
end

-- Atualizar sempre que um novo jogador entrar
Players.PlayerAdded:Connect(function(player)
    task.wait(1)
    CreateESP(player)

    if LocalPlayer:IsFriendsWith(player.UserId) then
        CreateArrow(player)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    if ESPs[player] then
        ESPs[player]:Destroy()
        ESPs[player] = nil
    end
end)

-- Atualizar no início
RefreshESPs()
