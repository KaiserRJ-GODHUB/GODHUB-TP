--[[ 
   GOD HUB TP - V5.1
   Sistema de Teleporte, Puxar (Noclip) e Velocidade Ajustável
]]

local Player = game.Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- Limpeza de versões anteriores
if CoreGui:FindFirstChild("GodHub_TP_System") then
    CoreGui.GodHub_TP_System:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GodHub_TP_System"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true

local Cores = {
    TP = Color3.fromRGB(52, 152, 219),
    Registrar = Color3.fromRGB(46, 204, 113),
    Puxar = Color3.fromRGB(155, 89, 182),
    Fundo = Color3.fromRGB(20, 20, 20),
    Texto = Color3.fromRGB(255, 255, 255),
    BotoesVel = Color3.fromRGB(40, 40, 40),
    Destaque = Color3.fromRGB(255, 215, 0) -- Dourado para o nome "GOD"
}

-- Variáveis de Estado
local Waypoints = {}
local Velocidades = { [1] = 50, [2] = 50 }
local noclip = false

-- Lógica de Noclip
RunService.Stepped:Connect(function()
    if noclip and Player.Character then
        for _, part in pairs(Player.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

-- Função de Arrastar GUI
local function MakeDraggable(obj)
    local dragging, dragStart, startPos
    obj.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = obj.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            obj.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

-- Painel Principal
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 260, 0, 380) -- Aumentado levemente para o título
MainFrame.Position = UDim2.new(0.5, -130, 0.5, -190)
MainFrame.BackgroundColor3 = Cores.Fundo
MainFrame.Active = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 15)
MakeDraggable(MainFrame)

-- TÍTULO GOD HUB TP
local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(0, 200, 0, 35)
Title.Position = UDim2.new(0, 15, 0, 5)
Title.Text = "GOD HUB TP"
Title.TextColor3 = Cores.Destaque
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1

-- Botão Fechar (X)
local CloseBtn = Instance.new("TextButton", MainFrame)
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -35, 0, 5)
CloseBtn.Text = "X"
CloseBtn.BackgroundColor3 = Color3.fromRGB(231, 76, 60)
CloseBtn.TextColor3 = Cores.Texto
CloseBtn.Font = Enum.Font.SourceSansBold
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 5)

-- Função para Puxar (Tween)
local function PuxarPara(id)
    local pos = Waypoints[id]
    local char = Player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    
    if hrp and pos then
        noclip = true
        local distancia = (hrp.Position - pos.Position).Magnitude
        local velocidade = Velocidades[id] > 0 and Velocidades[id] or 1
        local tempo = distancia / velocidade
        
        local tween = TweenService:Create(hrp, TweenInfo.new(tempo, Enum.EasingStyle.Linear), {CFrame = pos})
        tween:Play()
        tween.Completed:Connect(function()
            noclip = false
        end)
    end
end

-- Função para Criar Seção de Teleporte
local function CriarSecao(id, posY)
    -- Botão IR TP
    local TP = Instance.new("TextButton", MainFrame)
    TP.Size = UDim2.new(0, 115, 0, 35)
    TP.Position = UDim2.new(0, 10, 0, posY)
    TP.BackgroundColor3 = Cores.TP
    TP.Text = "IR TP " .. id
    TP.TextColor3 = Cores.Texto
    TP.Font = Enum.Font.SourceSansBold
    Instance.new("UICorner", TP)

    -- Botão SALVAR
    local Reg = Instance.new("TextButton", MainFrame)
    Reg.Size = UDim2.new(0, 115, 0, 35)
    Reg.Position = UDim2.new(0, 135, 0, posY)
    Reg.BackgroundColor3 = Cores.Registrar
    Reg.Text = "SALVAR " .. id
    Reg.TextColor3 = Cores.Texto
    Reg.Font = Enum.Font.SourceSansBold
    Instance.new("UICorner", Reg)

    -- Botão PUXAR
    local Puxar = Instance.new("TextButton", MainFrame)
    Puxar.Size = UDim2.new(0, 240, 0, 35)
    Puxar.Position = UDim2.new(0, 10, 0, posY + 40)
    Puxar.BackgroundColor3 = Cores.Puxar
    Puxar.Text = "PUXAR " .. id .. " (NOCLIP)"
    Puxar.TextColor3 = Cores.Texto
    Puxar.Font = Enum.Font.SourceSansBold
    Instance.new("UICorner", Puxar)

    -- CONTROLES DE VELOCIDADE
    local VelFrame = Instance.new("Frame", MainFrame)
    VelFrame.Size = UDim2.new(0, 240, 0, 35)
    VelFrame.Position = UDim2.new(0, 10, 0, posY + 80)
    VelFrame.BackgroundTransparency = 1

    local Menos = Instance.new("TextButton", VelFrame)
    Menos.Size = UDim2.new(0, 40, 0, 35)
    Menos.BackgroundColor3 = Cores.BotoesVel
    Menos.Text = "-"
    Menos.TextColor3 = Cores.Texto
    Menos.Font = Enum.Font.SourceSansBold
    Instance.new("UICorner", Menos)

    local Label = Instance.new("TextLabel", VelFrame)
    Label.Size = UDim2.new(0, 160, 0, 35)
    Label.Position = UDim2.new(0, 40, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = "VEL: " .. Velocidades[id]
    Label.TextColor3 = Cores.Texto
    Label.Font = Enum.Font.SourceSansBold

    local Mais = Instance.new("TextButton", VelFrame)
    Mais.Size = UDim2.new(0, 40, 0, 35)
    Mais.Position = UDim2.new(0, 200, 0, 0)
    Mais.BackgroundColor3 = Cores.BotoesVel
    Mais.Text = "+"
    Mais.TextColor3 = Cores.Texto
    Mais.Font = Enum.Font.SourceSansBold
    Instance.new("UICorner", Mais)

    -- Eventos
    Reg.MouseButton1Click:Connect(function()
        local hrp = Player.Character and Player.Character:FindFirstChild("HumanoidRootPart")
        if hrp then Waypoints[id] = hrp.CFrame end
    end)

    TP.MouseButton1Click:Connect(function()
        local hrp = Player.Character and Player.Character:FindFirstChild("HumanoidRootPart")
        if hrp and Waypoints[id] then hrp.CFrame = Waypoints[id] end
    end)

    Puxar.MouseButton1Click:Connect(function()
        PuxarPara(id)
    end)

    Menos.MouseButton1Click:Connect(function()
        Velocidades[id] = math.max(0, Velocidades[id] - 10)
        Label.Text = "VEL: " .. Velocidades[id]
    end)

    Mais.MouseButton1Click:Connect(function()
        Velocidades[id] = Velocidades[id] + 10
        Label.Text = "VEL: " .. Velocidades[id]
    end)
end

-- Criar as 2 seções (Espaçamento ajustado para começar abaixo do título)
CriarSecao(1, 50)
CriarSecao(2, 190)

-- Botão Minimizado (Menu Ball)
local MinBall = Instance.new("TextButton", ScreenGui)
MinBall.Size = UDim2.new(0, 50, 0, 50)
MinBall.Visible = false
MinBall.Text = "GOD"
MinBall.BackgroundColor3 = Cores.Fundo
MinBall.TextColor3 = Cores.Destaque
MinBall.Font = Enum.Font.SourceSansBold
Instance.new("UICorner", MinBall).CornerRadius = UDim.new(1, 0)
MakeDraggable(MinBall)

CloseBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    MinBall.Position = MainFrame.Position
    MinBall.Visible = true
end)

MinBall.MouseButton1Click:Connect(function()
    MinBall.Visible = false
    MainFrame.Visible = true
    MainFrame.Position = MinBall.Position
end)
