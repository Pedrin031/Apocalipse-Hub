local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

local aimbotAtivo = false
local menuAberto = true
local aimbotRadius = 150
local rightClickDown = false

local playerGui = player:WaitForChild("PlayerGui")

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimbotGUI"
screenGui.Parent = playerGui

local menuFrame = Instance.new("Frame")
menuFrame.Size = UDim2.new(0, 200, 0, 100)
menuFrame.Position = UDim2.new(0.5, -100, 0.5, -50)
menuFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
menuFrame.Visible = menuAberto
menuFrame.Parent = screenGui

local menuText = Instance.new("TextLabel")
menuText.Size = UDim2.new(1, 0, 1, 0)
menuText.BackgroundTransparency = 1
menuText.Text = "X: Ativar/Desativar Aimbot\nP: Abrir/Fechar Menu\nScroll: Mudar Raio"
menuText.TextColor3 = Color3.new(1, 1, 1)
menuText.Font = Enum.Font.SourceSans
menuText.TextScaled = true
menuText.Parent = menuFrame

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(0, 300, 0, 50)
statusLabel.Position = UDim2.new(0.5, -150, 0.4, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = ""
statusLabel.TextColor3 = Color3.new(1, 1, 1)
statusLabel.Font = Enum.Font.SourceSansBold
statusLabel.TextScaled = true
statusLabel.Visible = false
statusLabel.Parent = screenGui

local aimCircle = Instance.new("Frame")
aimCircle.Size = UDim2.new(0, aimbotRadius * 2, 0, aimbotRadius * 2)
aimCircle.Position = UDim2.new(0, mouse.X, 0, mouse.Y)
aimCircle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
aimCircle.BackgroundTransparency = 0.7
aimCircle.BorderSizePixel = 0
aimCircle.AnchorPoint = Vector2.new(0.5, 0.5)
aimCircle.ClipsDescendants = true
aimCircle.Parent = screenGui

local uicorner = Instance.new("UICorner")
uicorner.CornerRadius = UDim.new(1, 0)
uicorner.Parent = aimCircle

local function mostrarMensagem(texto, cor)
    statusLabel.Text = texto
    statusLabel.TextColor3 = cor
    statusLabel.Visible = true
    task.spawn(function()
        wait(2)
        statusLabel.Visible = false
    end)
end

local function getClosestTarget()
    local closest = nil
    local shortestDistance = aimbotRadius

    for _, target in ipairs(Players:GetPlayers()) do
        if target ~= player and target.Character and target.Character:FindFirstChild("Head") then
            local head = target.Character.Head
            local pos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(head.Position)
            if onScreen then
                local mousePos = Vector2.new(mouse.X, mouse.Y)
                local targetPos = Vector2.new(pos.X, pos.Y)
                local distance = (mousePos - targetPos).Magnitude

                if distance < shortestDistance then
                    shortestDistance = distance
                    closest = head
                end
            end
        end
    end

    for _, npc in ipairs(workspace:GetChildren()) do
        if npc:IsA("Model") and npc:FindFirstChild("Head") and not Players:GetPlayerFromCharacter(npc) then
            local head = npc.Head
            local pos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(head.Position)
            if onScreen then
                local mousePos = Vector2.new(mouse.X, mouse.Y)
                local targetPos = Vector2.new(pos.X, pos.Y)
                local distance = (mousePos - targetPos).Magnitude

                if distance < shortestDistance then
                    shortestDistance = distance
                    closest = head
                end
            end
        end
    end

    return closest
end

local function aimAt(target)
    if target then
        local camera = workspace.CurrentCamera
        camera.CFrame = CFrame.new(camera.CFrame.Position, target.Position)
    end
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end

    if input.KeyCode == Enum.KeyCode.P then
        menuAberto = not menuAberto
        menuFrame.Visible = menuAberto
    elseif input.KeyCode == Enum.KeyCode.X then
        aimbotAtivo = not aimbotAtivo
        if aimbotAtivo then
            mostrarMensagem("Aimbot Ativado!", Color3.new(0, 1, 0))
        else
            mostrarMensagem("Aimbot Desativado!", Color3.new(1, 0, 0))
        end
    elseif input.UserInputType == Enum.UserInputType.MouseButton2 then
        rightClickDown = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        rightClickDown = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseWheel then
        aimbotRadius = math.clamp(aimbotRadius + input.Position.Z * 5, 50, 500)
        aimCircle.Size = UDim2.new(0, aimbotRadius * 2, 0, aimbotRadius * 2)
    end
end)

RunService.RenderStepped:Connect(function()
    -- Atualiza posição do círculo
    aimCircle.Position = UDim2.new(0, mouse.X, 0, mouse.Y)

    if aimbotAtivo and rightClickDown then
        local target = getClosestTarget()
        if target then
            aimAt(target)
        end
    end
end)
