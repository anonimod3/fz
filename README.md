-- =============================================
-- RENAN.mod | FZ - Pequenos Impérios (Tiny Empires)
-- Auto Farm + Menu (Toggle: K)
-- =============================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Workspace = game.Workspace

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

-- ================== CONFIGURAÇÕES ==================
local CONFIG = {
    AutoCollectFarm = true,   -- Coletar da fazenda
    AutoSell = true,          -- Vender recursos
    AutoBuild = true,         -- Construir
    AutoAssignWorkers = true, -- Atribuir trabalhadores
    AutoPullGold = true,      -- Puxar ouro / coletar recursos
    BuildDelay = 0.8,
    SellDelay = 10,
    MaxBuildings = 25,
    Priority = {"Fazenda", "Farm", "Mina", "Casa", "Quartel", "Workshop"},
    MaxPerType = { Fazenda = 10, Farm = 10, Mina = 8, Casa = 6, Quartel = 4 }
}

-- ================== REMOTES (melhor detecção) ==================
local function GetRemote(name)
    return ReplicatedStorage:FindFirstChild(name, true) 
        or (ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild(name))
        or ReplicatedStorage:FindFirstChild(name)
end

-- ================== POSIÇÃO PARA CONSTRUIR ==================
local function GetFreePosition()
    local base = player:FindFirstChild("Base") 
        or Workspace:FindFirstChild("Bases") and Workspace.Bases:FindFirstChild(player.Name)
        or player:FindFirstChild("Plot") or Workspace.Terrain
    
    local origin = (base and base:IsA("BasePart")) and base.Position or Vector3.new(0, 20, 0)
    return origin + Vector3.new(math.random(-35, 35), 10, math.random(-35, 35))
end

-- ================== FUNÇÕES ==================

-- Coletar da Fazenda
local function AutoCollectFarm()
    if not CONFIG.AutoCollectFarm then return end
    local remote = GetRemote("Collect") or GetRemote("Harvest") or GetRemote("CollectFarm") or GetRemote("CollectResource")
    if remote then
        for _, farm in ipairs(Workspace:GetDescendants()) do
            if farm.Name:match("Farm") or farm.Name:match("Fazenda") then
                pcall(function()
                    remote:FireServer(farm)
                end)
            end
        end
    end
end

-- Puxar Ouro / Coletar Recursos
local function AutoPullGold()
    if not CONFIG.AutoPullGold then return end
    local remote = GetRemote("CollectGold") or GetRemote("ClaimGold") or GetRemote("CollectAll") or GetRemote("HarvestAll")
    if remote then
        pcall(function() remote:FireServer() end)
    end
end

-- Construir
local function AutoBuild()
    if not CONFIG.AutoBuild then return end
    local remote = GetRemote("PlaceBuilding") or GetRemote("Build") or GetRemote("BuildStructure") or GetRemote("BuyBuilding")
    if not remote then return end

    local count = 0
    for _, buildingType in ipairs(CONFIG.Priority) do
        if count >= CONFIG.MaxBuildings then break end
        local pos = GetFreePosition()
        pcall(function()
            remote:FireServer(buildingType, pos)
        end)
        count += 1
        task.wait(CONFIG.BuildDelay)
    end
end

-- Atribuir Trabalhadores
local function AutoAssignWorkers()
    if not CONFIG.AutoAssignWorkers then return end
    local remote = GetRemote("AssignWorker") or GetRemote("SetWorkers") or GetRemote("Assign")
    if not remote then return end

    for _, building in ipairs(Workspace:GetDescendants()) do
        if building:FindFirstChild("Owner") and building.Owner.Value == player then
            pcall(function()
                remote:FireServer(building, "max") -- ou "full"
            end)
        end
    end
end

-- Vender
local function AutoSell()
    if not CONFIG.AutoSell then return end
    local remote = GetRemote("Sell") or GetRemote("SellResources") or GetRemote("SellAll") or GetRemote("MarketSell")
    if remote then
        pcall(function() remote:FireServer("all") end)
    end
end

-- ================== GUI (mesma de antes, só adicionei toggles) ==================
local gui = Instance.new("ScreenGui")
gui.Name = "RENAN_FZ"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

-- Cores
local RED = Color3.fromRGB(200, 25, 35)
local RED_DARK = Color3.fromRGB(120, 10, 18)
local BG = Color3.fromRGB(20, 20, 22)
local BG2 = Color3.fromRGB(30, 30, 33)
local TEXT = Color3.fromRGB(240, 240, 240)

-- (O resto da GUI fica igual ao seu código original - só adicionei os toggles novos)

local main = Instance.new("Frame") -- ... (copie o resto da sua GUI aqui)

-- Container
local body = Instance.new("Frame", main) -- ... (manter igual)

local list = Instance.new("UIListLayout", body)

local function makeToggle(text, key)
    local f = Instance.new("Frame", body)
    f.BackgroundColor3 = BG2
    f.Size = UDim2.new(1, 0, 0, 38)
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 6)

    local lbl = Instance.new("TextLabel", f)
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(1, -70, 1, 0)
    lbl.Position = UDim2.new(0, 12, 0, 0)
    lbl.Text = text
    lbl.TextColor3 = TEXT
    lbl.Font = Enum.Font.Gotham
    lbl.TextSize = 14
    lbl.TextXAlignment = Enum.TextXAlignment.Left

    local btn = Instance.new("TextButton", f)
    btn.Size = UDim2.new(0, 50, 0, 22)
    btn.Position = UDim2.new(1, -60, 0.5, -11)
    btn.BackgroundColor3 = CONFIG[key] and RED or Color3.fromRGB(60,60,60)
    btn.Text = CONFIG[key] and "ON" or "OFF"
    btn.TextColor3 = TEXT
    Instance.new("UICorner", btn)

    btn.MouseButton1Click:Connect(function()
        CONFIG[key] = not CONFIG[key]
        btn.Text = CONFIG[key] and "ON" or "OFF"
        btn.BackgroundColor3 = CONFIG[key] and RED or Color3.fromRGB(60,60,60)
    end)
end

-- Toggles novos
makeToggle("Auto Coletar Fazenda", "AutoCollectFarm")
makeToggle("Auto Puxar Ouro", "AutoPullGold")
makeToggle("Auto Build", "AutoBuild")
makeToggle("Auto Assign Workers", "AutoAssignWorkers")
makeToggle("Auto Sell", "AutoSell")

-- Botão manual
local runBtn = Instance.new("TextButton", body) -- ... (manter igual)
runBtn.MouseButton1Click:Connect(function()
    AutoCollectFarm()
    AutoPullGold()
    AutoBuild()
    AutoAssignWorkers()
    AutoSell()
end)

-- ================== LOOP PRINCIPAL ==================
task.spawn(function()
    while true do
        pcall(function()
            AutoCollectFarm()
            AutoPullGold()
            AutoBuild()
            AutoAssignWorkers()
        end)
        
        task.wait(1.5)
        
        if tick() % CONFIG.SellDelay < 2 then
            pcall(AutoSell)
        end
    end
end)

print("✅ RENAN.mod | Tiny Empires carregado! Pressione K para abrir/fechar.")
