-- Configurações iniciais
local player = game:GetService("Players").LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Cria a interface (fora da função para persistir)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BlackHackUI"
ScreenGui.ResetOnSpawn = false -- Isso impede que o menu desapareça ao respawnar
ScreenGui.Parent = player:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 300, 0, 250)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -125)
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
MainFrame.BackgroundColor3 = Color3.new(0, 0, 0)
MainFrame.BackgroundTransparency = 0.3
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.new(0, 0, 0)
Title.BackgroundTransparency = 0.5
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Text = "Menu de Hacks"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.Parent = MainFrame

local FlyButton = Instance.new("TextButton")
FlyButton.Name = "FlyButton"
FlyButton.Size = UDim2.new(0.9, 0, 0, 40)
FlyButton.Position = UDim2.new(0.05, 0, 0.15, 0)
FlyButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
FlyButton.TextColor3 = Color3.new(1, 1, 1)
FlyButton.Text = "Fly: OFF"
FlyButton.Font = Enum.Font.Gotham
FlyButton.TextSize = 16
FlyButton.Parent = MainFrame

local InfiniteJumpButton = Instance.new("TextButton")
InfiniteJumpButton.Name = "InfiniteJumpButton"
InfiniteJumpButton.Size = UDim2.new(0.9, 0, 0, 40)
InfiniteJumpButton.Position = UDim2.new(0.05, 0, 0.4, 0)
InfiniteJumpButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
InfiniteJumpButton.TextColor3 = Color3.new(1, 1, 1)
InfiniteJumpButton.Text = "Infinite Jump: OFF"
InfiniteJumpButton.Font = Enum.Font.Gotham
InfiniteJumpButton.TextSize = 16
InfiniteJumpButton.Parent = MainFrame

local GodmodeButton = Instance.new("TextButton")
GodmodeButton.Name = "GodmodeButton"
GodmodeButton.Size = UDim2.new(0.9, 0, 0, 40)
GodmodeButton.Position = UDim2.new(0.05, 0, 0.65, 0)
GodmodeButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
GodmodeButton.TextColor3 = Color3.new(1, 1, 1)
GodmodeButton.Text = "Godmode: OFF"
GodmodeButton.Font = Enum.Font.Gotham
GodmodeButton.TextSize = 16
GodmodeButton.Parent = MainFrame

-- Variáveis de controle (fora das funções para manter estado)
local flyEnabled = false
local infiniteJumpEnabled = false
local godmodeEnabled = false
local flySpeed = 50
local bodyVelocity
local originalWalkSpeed
local originalJumpPower
local connections = {} -- Para armazenar conexões e limpá-las adequadamente

-- Função para limpar conexões antigas
local function cleanupConnections()
    for _, connection in pairs(connections) do
        connection:Disconnect()
    end
    connections = {}
end

-- Função para ativar/desativar godmode
local function toggleGodmode()
    godmodeEnabled = not godmodeEnabled
    
    if godmodeEnabled then
        GodmodeButton.Text = "Godmode: ON"
        
        local function setupGodmode(character)
            if not character then return end
            
            local humanoid = character:WaitForChild("Humanoid")
            if not humanoid then return end
            
            -- Salva valores originais
            originalWalkSpeed = humanoid.WalkSpeed
            originalJumpPower = humanoid.JumpPower
            
            -- Configura godmode
            humanoid.MaxHealth = math.huge
            humanoid.Health = math.huge
            
            -- Previne estados indesejados
            humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
            
            -- Monitora saúde
            table.insert(connections, humanoid:GetPropertyChangedSignal("Health"):Connect(function()
                if godmodeEnabled then
                    humanoid.Health = humanoid.MaxHealth
                end
            end))
            
            -- Remove colisão
            for _, child in ipairs(character:GetDescendants()) do
                if child:IsA("BasePart") then
                    child.CanCollide = false
                    table.insert(connections, child:GetPropertyChangedSignal("CanCollide"):Connect(function()
                        if godmodeEnabled then
                            child.CanCollide = false
                        end
                    end))
                end
            end
        end
        
        -- Configura para o personagem atual
        if player.Character then
            setupGodmode(player.Character)
        end
        
        -- Configura para futuros personagens
        table.insert(connections, player.CharacterAdded:Connect(function(character)
            if godmodeEnabled then
                setupGodmode(character)
            end
        end))
    else
        GodmodeButton.Text = "Godmode: OFF"
        
        if player.Character then
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.MaxHealth = 100
                humanoid.Health = 100
                humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
                humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
                humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
                
                if originalWalkSpeed then
                    humanoid.WalkSpeed = originalWalkSpeed
                end
                
                if originalJumpPower then
                    humanoid.JumpPower = originalJumpPower
                end
                
                -- Restaura colisão
                for _, child in ipairs(player.Character:GetDescendants()) do
                    if child:IsA("BasePart") then
                        child.CanCollide = true
                    end
                end
            end
        end
    end
end

-- Função para ativar/desativar fly
local function toggleFly()
    flyEnabled = not flyEnabled
    
    if flyEnabled then
        FlyButton.Text = "Fly: ON"
        
        local function setupFly(character)
            if not character then return end
            
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if not humanoid then return end
            
            humanoid.PlatformStand = true
            
            if bodyVelocity then bodyVelocity:Destroy() end
            bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Velocity = Vector3.new(0, 0, 0)
            bodyVelocity.MaxForce = Vector3.new(0, math.huge, 0)
            bodyVelocity.Parent = character:FindFirstChild("HumanoidRootPart")
            
            local flyConnection
            flyConnection = RunService.Heartbeat:Connect(function()
                if not flyEnabled or not player.Character then
                    flyConnection:Disconnect()
                    return
                end
                
                local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
                if not rootPart then return end
                
                local direction = Vector3.new()
                
                if UIS:IsKeyDown(Enum.KeyCode.W) then
                    direction = direction + rootPart.CFrame.LookVector
                end
                if UIS:IsKeyDown(Enum.KeyCode.S) then
                    direction = direction - rootPart.CFrame.LookVector
                end
                if UIS:IsKeyDown(Enum.KeyCode.A) then
                    direction = direction - rootPart.CFrame.RightVector
                end
                if UIS:IsKeyDown(Enum.KeyCode.D) then
                    direction = direction + rootPart.CFrame.RightVector
                end
                
                if direction.Magnitude > 0 then
                    direction = direction.Unit * flySpeed
                end
                
                -- Controle de altura
                if UIS:IsKeyDown(Enum.KeyCode.Space) then
                    direction = direction + Vector3.new(0, flySpeed/2, 0)
                elseif UIS:IsKeyDown(Enum.KeyCode.LeftShift) then
                    direction = direction - Vector3.new(0, flySpeed/2, 0)
                end
                
                bodyVelocity.Velocity = direction
            end)
            
            table.insert(connections, flyConnection)
        end
        
        -- Configura para o personagem atual
        if player.Character then
            setupFly(player.Character)
        end
        
        -- Configura para futuros personagens
        table.insert(connections, player.CharacterAdded:Connect(function(character)
            if flyEnabled then
                setupFly(character)
            end
        end))
    else
        FlyButton.Text = "Fly: OFF"
        if bodyVelocity then
            bodyVelocity:Destroy()
            bodyVelocity = nil
        end
        if player.Character then
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.PlatformStand = false
            end
        end
    end
end

-- Função para ativar/desativar infinite jump
local function toggleInfiniteJump()
    infiniteJumpEnabled = not infiniteJumpEnabled
    
    if infiniteJumpEnabled then
        InfiniteJumpButton.Text = "Infinite Jump: ON"
        
        local jumpConnection
        jumpConnection = UIS.JumpRequest:Connect(function()
            if infiniteJumpEnabled and player.Character then
                local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                end
            end
        end)
        
        table.insert(connections, jumpConnection)
        
        -- Configura para futuros personagens
        table.insert(connections, player.CharacterAdded:Connect(function()
            if infiniteJumpEnabled then
                -- Reconecta o infinite jump para o novo personagem
                toggleInfiniteJump()
                toggleInfiniteJump() -- Alterna para manter ativado
            end
        end))
    else
        InfiniteJumpButton.Text = "Infinite Jump: OFF"
    end
end

-- Conecta os botões às funções
FlyButton.MouseButton1Click:Connect(toggleFly)
InfiniteJumpButton.MouseButton1Click:Connect(toggleInfiniteJump)
GodmodeButton.MouseButton1Click:Connect(toggleGodmode)

-- Função para arrastar a janela
local dragging
local dragInput
local dragStart
local startPos

local function update(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

Title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

Title.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

-- Limpa tudo quando o script é destruído
ScreenGui.AncestryChanged:Connect(function()
    if not ScreenGui:IsDescendantOf(game) then
        cleanupConnections()
        if bodyVelocity then bodyVelocity:Destroy() end
    end
end)
