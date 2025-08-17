-- Script para remover colisões de barcos, deletar bolas e ativar noclip
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- Função para remover colisão de todos os BoatCollide
local function removeBoatCollisions()
    local beachNorth = workspace.Model:FindFirstChild("BeachNorth")
    if beachNorth then
        local boatCollideFolder = beachNorth.Model:FindFirstChild("BoatCollide")
        if boatCollideFolder then
            -- Remove colisão de BoatCollide individual
            local boatCollide = boatCollideFolder:FindFirstChild("BoatCollide")
            if boatCollide then
                boatCollide.CanCollide = false
                print("Colisão removida de BoatCollide individual")
            end
            
            -- Remove colisão de todos os filhos do folder BoatCollide
            for _, child in pairs(boatCollideFolder:GetChildren()) do
                if child:IsA("BasePart") then
                    child.CanCollide = false
                    print("Colisão removida de: " .. child.Name)
                end
            end
        end
    end
end

-- Função para detectar e remover colisão de barcos quando player estiver neles
local function checkPlayerBoats()
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local character = otherPlayer.Character
            local humanoidRootPart = character.HumanoidRootPart
            
            -- Verifica se o player está próximo/em um barco
            local raycast = workspace:Raycast(humanoidRootPart.Position, Vector3.new(0, -10, 0))
            if raycast and raycast.Instance then
                local hit = raycast.Instance
                if hit.Name:lower():find("boat") or hit.Parent.Name:lower():find("boat") then
                    hit.CanCollide = false
                end
            end
        end
    end
end

-- Função para remover todas as SoccerBalls de forma mais agressiva
local function removeSoccerBalls()
    -- Remove do workspace
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and (obj.Name:lower():find("ball") or obj.Name:lower():find("soccer")) then
            obj:Destroy()
            print("Bola deletada do workspace: " .. obj.Name)
        elseif obj:IsA("Tool") and (obj.Name:lower():find("ball") or obj.Name:lower():find("soccer")) then
            obj:Destroy()
            print("Tool bola deletada: " .. obj.Name)
        elseif obj:IsA("Model") and (obj.Name:lower():find("ball") or obj.Name:lower():find("soccer")) then
            obj:Destroy()
            print("Model bola deletada: " .. obj.Name)
        end
    end
    
    -- Remove especificamente do WorkspaceCom
    local workspaceCom = workspace:FindFirstChild("WorkspaceCom")
    if workspaceCom then
        local giveTools = workspaceCom:FindFirstChild("001_GiveTools")
        if giveTools then
            for _, child in pairs(giveTools:GetChildren()) do
                if child.Name == "SoccerBall" or child.Name:lower():find("ball") then
                    child:Destroy()
                    print("SoccerBall removida do GiveTools: " .. child.Name)
                end
            end
        end
    end
    
    -- Remove do player (backpack, character, etc)
    if player then
        -- Remove do Backpack
        if player:FindFirstChild("Backpack") then
            for _, child in pairs(player.Backpack:GetChildren()) do
                if child.Name:lower():find("ball") or child.Name:lower():find("soccer") then
                    child:Destroy()
                    print("Bola removida do Backpack: " .. child.Name)
                end
            end
        end
        
        -- Remove do Character
        if player.Character then
            for _, child in pairs(player.Character:GetChildren()) do
                if child.Name:lower():find("ball") or child.Name:lower():find("soccer") then
                    child:Destroy()
                    print("Bola removida do Character: " .. child.Name)
                end
            end
            
            -- Verifica Humanoid especificamente
            local humanoid = player.Character:FindFirstChild("Humanoid")
            if humanoid then
                for _, child in pairs(humanoid:GetChildren()) do
                    if child.Name:lower():find("ball") or child.Name:lower():find("soccer") then
                        child:Destroy()
                        print("Bola removida do Humanoid: " .. child.Name)
                    end
                end
            end
        end
    end
end

-- Função para monitorar adições de objetos ao personagem
local function setupCharacterMonitoring()
    if player.Character then
        -- Monitora adições diretas ao Character
        player.Character.ChildAdded:Connect(function(child)
            if child.Name:lower():find("ball") or child.Name == "SoccerBall" or child.Name:lower():find("soccer") then
                wait(0.1) -- Pequeno delay para garantir que foi totalmente adicionado
                child:Destroy()
                print("SoccerBall deletada imediatamente do Character: " .. child.Name)
            end
        end)
        
        -- Monitora adições ao Humanoid
        local humanoid = player.Character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.ChildAdded:Connect(function(child)
                if child.Name:lower():find("ball") or child.Name == "SoccerBall" or child.Name:lower():find("soccer") then
                    wait(0.1)
                    child:Destroy()
                    print("SoccerBall deletada imediatamente do Humanoid: " .. child.Name)
                end
            end)
        end
    end
end

-- Conecta quando o personagem spawna/respawna
player.CharacterAdded:Connect(function(character)
    wait(1) -- Espera o personagem carregar completamente
    setupCharacterMonitoring()
    removeSoccerBalls() -- Remove qualquer bola que já esteja presente
    print("Personagem respawnado - monitoramento de SoccerBall ativado")
end)

-- Função para ativar noclip seletivo (mantém colisão com chão)
local function enableSelectiveNoclip()
    if player.Character then
        for _, part in pairs(player.Character:GetChildren()) do
            if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                part.CanCollide = false
            elseif part:IsA("BasePart") and part.Name == "HumanoidRootPart" then
                -- Mantém colisão apenas com partes que estão abaixo (chão)
                local raycast = workspace:Raycast(part.Position, Vector3.new(0, -10, 0))
                if raycast and raycast.Instance then
                    local hit = raycast.Instance
                    -- Se há algo abaixo, mantém colisão mínima para não cair
                    if hit.Name:lower():find("floor") or hit.Name:lower():find("ground") or hit.Name:lower():find("baseplate") or hit.Material == Enum.Material.Grass then
                        -- Mantém colisão com chão
                        part.CanCollide = true
                    else
                        part.CanCollide = false
                    end
                else
                    part.CanCollide = false
                end
            end
        end
    end
end

-- Função para desativar fling do humanoid (mais agressiva)
local function disableFling()
    if player.Character and player.Character:FindFirstChild("Humanoid") and player.Character:FindFirstChild("HumanoidRootPart") then
        local humanoid = player.Character.Humanoid
        local rootPart = player.Character.HumanoidRootPart
        
        -- Remove PlatformStand para permitir movimento
        humanoid.PlatformStand = false
        
        -- Anti-fling mais agressivo - previne qualquer velocidade alta
        if rootPart.AssemblyLinearVelocity.Magnitude > 50 then
            rootPart.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            rootPart.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
            
            -- Remove qualquer BodyMover que possa estar causando fling
            for _, child in pairs(rootPart:GetChildren()) do
                if child:IsA("BodyPosition") or child:IsA("BodyVelocity") or child:IsA("BodyThrust") or child:IsA("BodyForce") then
                    if child.Name ~= "AntiGravity" then -- Mantém anti-gravidade se houver
                        child:Destroy()
                    end
                end
            end
        end
        
        -- Limita a velocidade máxima
        if rootPart.Velocity.Magnitude > 100 then
            rootPart.Velocity = rootPart.Velocity.Unit * 16 -- Velocidade normal de caminhada
        end
    end
end

-- Executa as funções iniciais
removeBoatCollisions()
removeSoccerBalls()

-- Configura monitoramento para o personagem atual (se já existir)
if player.Character then
    setupCharacterMonitoring()
end

-- Loop principal que mantém noclip e anti-fling ativos
local connection
connection = RunService.Heartbeat:Connect(function()
    enableNoclip()
    disableFling()
    checkPlayerBoats()
    checkAndDeleteModifiedDoors()
end)

-- Remove bolas e verifica portas continuamente (mais frequente)
spawn(function()
    while true do
        removeSoccerBalls()
        checkNewDoorsNearPlayer()
        wait(0.5) -- Verifica a cada 0.5 segundos (mais rápido)
    end
end)

print("Script ativado! Colisões removidas, noclip ativado e anti-fling ligado")

-- Tabela para armazenar posições originais das portas
local originalDoorPositions = {}
local doorCheckRadius = 50 -- Raio em studs para verificar portas próximas

-- Função para registrar posições originais das portas
local function registerOriginalDoorPositions()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and (obj.Name:lower():find("door") or obj.Name:lower():find("gate")) then
            if obj.PrimaryPart then
                originalDoorPositions[obj] = obj.PrimaryPart.Position
            elseif obj:FindFirstChild("Door") then
                originalDoorPositions[obj] = obj.Door.Position
            elseif obj:FindFirstChildOfClass("BasePart") then
                originalDoorPositions[obj] = obj:FindFirstChildOfClass("BasePart").Position
            end
        elseif obj:IsA("BasePart") and (obj.Name:lower():find("door") or obj.Name:lower():find("gate")) then
            originalDoorPositions[obj] = obj.Position
        end
    end
    print("Posições originais das portas registradas: " .. #originalDoorPositions)
end

-- Função para verificar e deletar portas modificadas
local function checkAndDeleteModifiedDoors()
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
        return
    end
    
    local playerPosition = player.Character.HumanoidRootPart.Position
    
    for doorObj, originalPos in pairs(originalDoorPositions) do
        if doorObj and doorObj.Parent then
            local currentPos
            
            -- Determina a posição atual da porta
            if doorObj:IsA("Model") then
                if doorObj.PrimaryPart then
                    currentPos = doorObj.PrimaryPart.Position
                elseif doorObj:FindFirstChild("Door") then
                    currentPos = doorObj.Door.Position
                elseif doorObj:FindFirstChildOfClass("BasePart") then
                    currentPos = doorObj:FindFirstChildOfClass("BasePart").Position
                end
            elseif doorObj:IsA("BasePart") then
                currentPos = doorObj.Position
            end
            
            if currentPos then
                local distanceFromOriginal = (currentPos - originalPos).Magnitude
                local distanceFromPlayer = (currentPos - playerPosition).Magnitude
                
                -- Se a porta se moveu mais de 5 studs da posição original
                -- OU se está muito perto do player (menos de doorCheckRadius studs)
                if distanceFromOriginal > 5 or distanceFromPlayer < doorCheckRadius then
                    doorObj:Destroy()
                    originalDoorPositions[doorObj] = nil
                    print("Porta deletada por estar fora de posição ou muito perto: " .. doorObj.Name)
                end
            end
        else
            -- Remove da tabela se o objeto foi deletado
            originalDoorPositions[doorObj] = nil
        end
    end
end

-- Função para detectar novas portas e deletar se estiverem próximas
local function checkNewDoorsNearPlayer()
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
        return
    end
    
    local playerPosition = player.Character.HumanoidRootPart.Position
    
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and (obj.Name:lower():find("door") or obj.Name:lower():find("gate")) then
            if not originalDoorPositions[obj] then
                local doorPos
                if obj.PrimaryPart then
                    doorPos = obj.PrimaryPart.Position
                elseif obj:FindFirstChild("Door") then
                    doorPos = obj.Door.Position
                elseif obj:FindFirstChildOfClass("BasePart") then
                    doorPos = obj:FindFirstChildOfClass("BasePart").Position
                end
                
                if doorPos and (doorPos - playerPosition).Magnitude < doorCheckRadius then
                    obj:Destroy()
                    print("Nova porta deletada por estar próxima: " .. obj.Name)
                end
            end
        elseif obj:IsA("BasePart") and (obj.Name:lower():find("door") or obj.Name:lower():find("gate")) then
            if not originalDoorPositions[obj] then
                local doorPos = obj.Position
                if (doorPos - playerPosition).Magnitude < doorCheckRadius then
                    obj:Destroy()
                    print("Nova porta (BasePart) deletada por estar próxima: " .. obj.Name)
                end
            end
        end
    end
end

-- Registra posições originais das portas
registerOriginalDoorPositions()

-- Para parar o script, descomente a linha abaixo:
-- connection:Disconnect()
