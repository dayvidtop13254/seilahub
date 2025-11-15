-- Blox Fruits Farm Script - Focado em Frutas, Dinheiro e Raids (Nov 2025)
-- Autor: Grok (inspirado em scripts públicos)

getgenv().Config = {
    AutoFarm = true,
    AutoFarmMoney = true,
    SpawnFruits = true,  -- Spawna frutas para "chuva"
    FruitSniper = true,
    AutoRaid = true,
    SelectFruit = "All",  -- Ou especifique: "Leopard", "Kitsune", etc.
    HopServer = false  -- Troca de server se farm lento
}

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local Workspace = game:GetService("Workspace")

local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")

-- Função para Teleport
local function TeleportTo(pos)
    Character:MoveTo(pos)
end

-- Função para Equipar Melhor Ferramenta
local function EquipBestTool()
    local bestTool = nil
    local maxDamage = 0
    for _, tool in pairs(Player.Backpack:GetChildren()) do
        if tool:IsA("Tool") and tool:FindFirstChild("Damage") then
            local damage = tool.Damage.Value
            if damage > maxDamage then
                maxDamage = damage
                bestTool = tool
            end
        end
    end
    if bestTool then
        Humanoid:EquipTool(bestTool)
    end
end

-- Auto Farm Níveis/Dinheiro
spawn(function()
    while getgenv().Config.AutoFarm or getgenv().Config.AutoFarmMoney do
        wait(0.5)
        EquipBestTool()
        for _, mob in pairs(Workspace.Enemies:GetChildren()) do
            if mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
                TeleportTo(mob.HumanoidRootPart.Position + Vector3.new(0, 5, 0))
                game:GetService("VirtualUser"):ClickButton1(Vector2.new())
                Humanoid:MoveTo(mob.HumanoidRootPart.Position)
                wait(0.1)
            end
        end
        -- Farm Bosses para mais dinheiro
        for _, boss in pairs(Workspace.Enemies:GetChildren()) do
            if string.find(boss.Name, "Boss") then
                TeleportTo(boss.HumanoidRootPart.Position)
                game:GetService("VirtualUser"):ClickButton1(Vector2.new())
            end
        end
    end
end)

-- Spawn Frutas e "Chuva" (Spawna múltiplas frutas aleatórias)
spawn(function()
    while getgenv().Config.SpawnFruits do
        wait(2)  -- Delay para evitar lag
        local fruits = {"Bomb-Bomb", "Spike", "Chop", "Spring", "Kilo", "Spin", "Flame", "Ice", "Sand", "Dark", "Light", "Rubber", "Barrier", "Magma", "Leaf", "Earth", "Thunder", "Water", "Venom", "Control", "Dragon", "Leopard", "Dough", "Kitsune"}  -- Lista de frutas
        local randomFruit = fruits[math.random(1, #fruits)]
        local fruitModel = ReplicatedStorage.Remotes.CommF_:InvokeServer("PurchaseFruit", randomFruit)  -- Tenta spawn via remote (pode precisar de bypass)
        if fruitModel then
            -- "Chuva": Spawna em posições aleatórias no céu
            for i = 1, 5 do  -- 5 frutas por vez
                local spawnPos = Vector3.new(math.random(-500, 500), 1000, math.random(-500, 500))  -- Alto no céu
                local fruit = fruitModel:Clone()
                fruit.Parent = Workspace
                fruit:MoveTo(spawnPos)
                -- Coleta automática
                spawn(function()
                    wait(1)
                    TeleportTo(fruit.Position)
                    firetouchinterest(Character.HumanoidRootPart, fruit, 0)
                    wait(0.1)
                    firetouchinterest(Character.HumanoidRootPart, fruit, 1)
                end)
            end
        end
    end
end)

-- Fruit Sniper (Pega frutas que spawnam)
spawn(function()
    while getgenv().Config.FruitSniper do
        wait(0.1)
        for _, fruit in pairs(Workspace:GetChildren()) do
            if string.find(fruit.Name, "Fruit") or string.find(fruit.Name, "Devil") then
                if getgenv().Config.SelectFruit == "All" or string.find(fruit.Name, getgenv().Config.SelectFruit) then
                    TeleportTo(fruit.Position)
                    firetouchinterest(Character.HumanoidRootPart, fruit, 0)
                    wait(0.1)
                    firetouchinterest(Character.HumanoidRootPart, fruit, 1)
                end
            end
        end
    end
end)

-- Auto Raid
spawn(function()
    while getgenv().Config.AutoRaid do
        wait(5)
        -- Inicia Raid
        ReplicatedStorage.Remotes.CommF_:InvokeServer("RaidsNpc", "Select", "Flame")
        wait(2)
        ReplicatedStorage.Remotes.CommF_:InvokeServer("RaidsNpc", "Start")
        -- Durante o Raid, farm automático
        spawn(function()
            while _G.RaidActive do  -- Assume variável global do jogo
                for _, enemy in pairs(Workspace.Enemies:GetChildren()) do
                    if enemy.Parent == Workspace.Raid or string.find(enemy.Name, "Raid") then
                        TeleportTo(enemy.HumanoidRootPart.Position)
                        game:GetService("VirtualUser"):ClickButton1(Vector2.new())
                    end
                end
                wait(0.5)
            end
        end)
        -- Completa e repete
        wait(300)  -- Tempo de um raid típico
    end
end)

-- Hop Server para mais farm (opcional)
spawn(function()
    while getgenv().Config.HopServer do
        wait(600)  -- A cada 10 min
        TeleportService:Teleport(game.PlaceId, Player)
    end
end)

-- GUI Simples para Toggle (opcional, use um GUI library como Kavo)
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
Frame.Parent = ScreenGui
ScreenGui.Parent = game.CoreGui
-- Adicione botões aqui se quiser, mas isso é básico

print("Script carregado! Ative as configs no getgenv() e execute.")
