-- =============================================
-- RENAN.mod | Pequenos Impérios (Tiny Empires)
-- Auto Farm + Menu (Tecla K)
-- =============================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game.Workspace

local player = Players.LocalPlayer

-- ================== CONFIGURAÇÕES ==================
local CONFIG = {
    AutoCollectFarm = true,     -- Coleta Automática (Fazenda)
    AutoPullGold = true,        -- Puxar Ouro / Recursos
    AutoBuild = true,
    AutoAssignWorkers = true,
    AutoSell = true,
    BuildDelay = 0.75,
    SellDelay = 12,
    MaxBuildings = 25,
    Priority = {"Fazenda", "Farm", "Mina", "Casa", "Quartel", "Workshop", "Mina de Prata", "Mina de Cobre"}
}

-- ================== REMOTES ==================
local function GetRemote(name)
    local rem = ReplicatedStorage:FindFirstChild(name, true)
    if rem then return rem end
    local remotesFolder = ReplicatedStorage:FindFirstChild("Remotes") or ReplicatedStorage:FindFirstChild("RemoteEvents")
    if remotesFolder then
        return remotesFolder:FindFirstChild(name) or remotesFolder:FindFirstChild(name:lower())
    end
    return nil
end

-- ================== FUNÇÕES ==================
local function AutoCollectFarm()
    if not CONFIG.AutoCollectFarm then return end
    local remote = GetRemote("Collect") or GetRemote("Harvest") or GetRemote("CollectFarm") or GetRemote("FarmCollect")
    if remote then
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj.Name:find("Farm") or obj.Name:find("Fazenda") or obj.Name:find("Plant") then
                pcall(function() remote:FireServer(obj) end)
            end
        end
    end
end

local function AutoPullGold()
    if not CONFIG.AutoPullGold then return end
    local remote = GetRemote("Claim") or GetRemote("CollectGold") or GetRemote("CollectAll") or GetRemote("HarvestAll") or GetRemote("Pickup")
    if remote then
        pcall(function() remote:FireServer() end)
    end
end

local function AutoBuild()
    if not CONFIG.AutoBuild then return end
    local remote = GetRemote("PlaceBuilding") or GetRemote("Build") or GetRemote("BuyBuilding") or GetRemote("Construct")
    if not remote then return end

    local count = 0
    for _, bType in ipairs(CONFIG.Priority) do
        if count >= CONFIG.MaxBuildings then break end
        local pos = Vector3.new(math.random(-60,60), 15, math.random(-60,60)) -- Ajuste conforme seu plot
        pcall(function() remote:FireServer(bType, pos) end)
        count += 1
        task.wait(CONFIG.BuildDelay)
    end
end

local function AutoAssignWorkers()
    if not CONFIG.AutoAssignWorkers then return end
    local remote = GetRemote("AssignWorker") or GetRemote("SetWorker") or GetRemote("Assign")
    if not remote then return end

    for _, building in ipairs(Workspace:GetDescendants()) do
        if building:FindFirstChild("Owner") and building.Owner.Value == player then
            pcall(function() remote:FireServer(building, "max") end)
        end
    end
end

local function AutoSell()
    if not CONFIG.AutoSell then return end
    local remote = GetRemote("Sell") or GetRemote("SellAll") or GetRemote("SellResources") or GetRemote("MarketSell")
    if remote then
        pcall(function() remote:FireServer("all") end)
    end
end

-- ================== GUI COMPLETA ==================
local gui = Instance.new("ScreenGui")
gui.Name = "RENAN_PequenosImperios"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local RED = Color3.fromRGB(220, 30, 40)
local RED_DARK = Color3.fromRGB(140, 15, 25)
local BG = Color3.fromRGB(22, 22, 25)
local BG2 = Color3.fromRGB(32, 32, 37)
local TEXT = Color3.fromRGB(245, 245, 245)

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 340, 0, 420)
main.Position = UDim2.new(0.5, -170, 0.5, -210)
main.BackgroundColor3 = BG
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true
main.Parent = gui
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)

local stroke = Instance.new("UIStroke", main)
stroke.Color = RED
stroke.Thickness = 2.5

-- Header
local header = Instance.new("Frame", main)
header.Size = UDim2.new(1, 0, 0, 40)
header.BackgroundColor3 = RED_DARK
header.BorderSizePixel = 0
Instance.new("UICorner", header).CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel", header)
title.Size = UDim2.new(1, -100, 1, 0)
title.Position = UDim2.new(0, 15, 0, 0)
title.BackgroundTransparency = 1
title.Text = "RENAN.mod | Pequenos Impérios"
title.TextColor3 = TEXT
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.TextXAlignment = Enum.TextXAlignment.Left

local closeBtn = Instance.new("TextButton", header)
closeBtn.Size = UDim2.new(0, 32, 0, 32)
closeBtn.Position = UDim2.new(1, -38, 0, 4)
closeBtn.BackgroundColor3 = RED
closeBtn.Text = "✕"
closeBtn.TextColor3 = TEXT
closeBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 8)

-- Body
local body = Instance.new("Frame", main)
body.Position = UDim2.new(0, 15, 0, 50)
body.Size = UDim2.new(1, -30, 1, -90)
body.BackgroundTransparency = 1

local listLayout = Instance.new("UIListLayout", body)
listLayout.Padding = UDim.new(0, 10)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder

local function makeToggle(text, key)
    local frame = Instance.new("Frame", body)
    frame.Size = UDim2.new(1, 0, 0, 42)
    frame.BackgroundColor3 = BG2
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(1, -80, 1, 0)
    label.Position = UDim2.new(0, 15, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = TEXT
    label.Font = Enum.Font.Gotham
    label.TextSize = 15
    label.TextXAlignment = Enum.TextXAlignment.Left

    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, 55, 0, 26)
    btn.Position = UDim2.new(1, -70, 0.5, -13)
    btn.BackgroundColor3 = CONFIG[key] and RED or Color3.fromRGB(70,70,70)
    btn.Text = CONFIG[key] and "ON" or "OFF"
    btn.TextColor3 = TEXT
    btn.Font = Enum.Font.GothamBold
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

    btn.MouseButton1Click:Connect(function()
        CONFIG[key] = not CONFIG[key]
        btn.Text = CONFIG[key] and "ON" or "OFF"
        btn.BackgroundColor3 = CONFIG[key] and RED or Color3.fromRGB(70,70,70)
    end)
end

makeToggle("Coleta Automática (Fazenda)", "AutoCollectFarm")
makeToggle("Puxar Ouro / Recursos", "AutoPullGold")
makeToggle("Auto Construir", "AutoBuild")
makeToggle("Auto Atribuir Trabalhadores", "AutoAssignWorkers")
makeToggle("Auto Vender", "AutoSell")

-- Botão Manual
local runBtn = Instance.new("TextButton", body)
runBtn.Size = UDim2.new(1, 0, 0, 45)
runBtn.BackgroundColor3 = RED
runBtn.Text = "▶ EXECUTAR TUDO AGORA"
runBtn.TextColor3 = TEXT
runBtn.Font = Enum.Font.GothamBold
runBtn.TextSize = 15
Instance.new("UICorner", runBtn).CornerRadius = UDim.new(0, 8)

runBtn.MouseButton1Click:Connect(function()
    AutoCollectFarm()
    AutoPullGold()
    AutoBuild()
    AutoAssignWorkers()
    AutoSell()
end)

-- Footer
local footer = Instance.new("TextLabel", main)
footer.Position = UDim2.new(0, 0, 1, -28)
footer.Size = UDim2.new(1, 0, 0, 25)
footer.BackgroundTransparency = 1
footer.Text = "Pressione K para abrir/fechar • Renan.mod"
footer.TextColor3 = Color3.fromRGB(160,160,160)
footer.Font = Enum.Font.Gotham
footer.TextSize = 12

-- Controles
local visible = true
closeBtn.MouseButton1Click:Connect(function() gui:Destroy() end)

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.K then
        visible = not visible
        main.Visible = visible
    end
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
        task.wait(1.4)
        
        if math.floor(tick()) % CONFIG.SellDelay == 0 then
            pcall(AutoSell)
        end
    end
end)

print("🚀 RENAN.mod | Pequenos Impérios carregado com sucesso! Pressione K")
