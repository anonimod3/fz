-- =============================================
-- RENAN.mod | FZ - Pequenos Impérios
-- Auto Farm + Menu (Toggle: K)
-- =============================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Workspace = game.Workspace
local player = Players.LocalPlayer

-- ================== CONFIGURAÇÕES ==================
local CONFIG = {
    AutoBuild = true,
    AutoAssignWorkers = true,
    AutoSell = true,
    AutoUpgrade = true,
    BuildDelay = 0.7,
    SellDelay = 8,
    MaxBuildings = 30,
    Priority = {"Fazenda", "Mina", "Casa", "Quartel"},
    MaxPerType = { Fazenda = 12, Mina = 8, Casa = 6, Quartel = 4 }
}

-- ================== REMOTES ==================
local function GetRemote(name)
    return ReplicatedStorage:FindFirstChild(name, true)
        or (ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild(name))
end

-- ================== POSIÇÃO ==================
local function GetFreePosition()
    local base = player:FindFirstChild("Base")
        or (Workspace:FindFirstChild("Bases") and Workspace.Bases:FindFirstChild(player.Name))
        or Workspace:FindFirstChild("Terrain")
    local origin = (base and base:IsA("BasePart")) and base.Position or Vector3.new(0,10,0)
    return origin + Vector3.new(math.random(-40,40), 5, math.random(-40,40))
end

-- ================== FUNÇÕES DE FARM ==================
local function AutoBuild()
    if not CONFIG.AutoBuild then return end
    local remote = GetRemote("PlaceBuilding") or GetRemote("BuildStructure") or GetRemote("Build")
    if not remote then return end
    local count = 0
    for _, buildingType in ipairs(CONFIG.Priority) do
        if count >= CONFIG.MaxBuildings then break end
        local pos = GetFreePosition()
        pcall(function() remote:FireServer(buildingType, pos) end)
        count += 1
        task.wait(CONFIG.BuildDelay)
    end
end

local function AutoAssignWorkers()
    if not CONFIG.AutoAssignWorkers then return end
    local remote = GetRemote("AssignWorker") or GetRemote("SetWorkers") or GetRemote("Assign")
    if not remote or not Workspace:FindFirstChild("Buildings") then return end
    for _, b in ipairs(Workspace.Buildings:GetChildren()) do
        if b:FindFirstChild("Owner") and b.Owner.Value == player then
            pcall(function() remote:FireServer(b, "max") end)
        end
    end
end

local function AutoSell()
    if not CONFIG.AutoSell then return end
    local remote = GetRemote("SellResources") or GetRemote("Sell") or GetRemote("SellAll")
    if remote then pcall(function() remote:FireServer("all") end) end
end

local function AutoUpgrade()
    if not CONFIG.AutoUpgrade then return end
    local remote = GetRemote("UpgradeBuilding")
    if not remote or not Workspace:FindFirstChild("Buildings") then return end
    for _, b in ipairs(Workspace.Buildings:GetChildren()) do
        if b:FindFirstChild("Owner") and b.Owner.Value == player then
            pcall(function() remote:FireServer(b) end)
        end
    end
end

-- ================== GUI ==================
local gui = Instance.new("ScreenGui")
gui.Name = "RENAN_FZ"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = player:WaitForChild("PlayerGui")

local RED       = Color3.fromRGB(200, 25, 35)
local RED_DARK  = Color3.fromRGB(120, 10, 18)
local BG        = Color3.fromRGB(20, 20, 22)
local BG2       = Color3.fromRGB(30, 30, 33)
local TEXT      = Color3.fromRGB(240, 240, 240)

-- Janela principal
local main = Instance.new("Frame")
main.Name = "Main"
main.Size = UDim2.new(0, 320, 0, 360)
main.Position = UDim2.new(0.5, -160, 0.5, -180)
main.BackgroundColor3 = BG
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true
main.Parent = gui

local stroke = Instance.new("UIStroke", main)
stroke.Color = RED; stroke.Thickness = 2
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 10)

-- Header
local header = Instance.new("Frame", main)
header.Size = UDim2.new(1, 0, 0, 36)
header.BackgroundColor3 = RED_DARK
header.BorderSizePixel = 0
Instance.new("UICorner", header).CornerRadius = UDim.new(0, 10)

local headerFix = Instance.new("Frame", header)
headerFix.Size = UDim2.new(1, 0, 0.5, 0)
headerFix.Position = UDim2.new(0, 0, 0.5, 0)
headerFix.BackgroundColor3 = RED_DARK
headerFix.BorderSizePixel = 0

local title = Instance.new("TextLabel", header)
title.BackgroundTransparency = 1
title.Size = UDim2.new(1, -80, 1, 0)
title.Position = UDim2.new(0, 12, 0, 0)
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.TextColor3 = TEXT
title.TextXAlignment = Enum.TextXAlignment.Left
title.Text = "RENAN.mod | FZ"

local minBtn = Instance.new("TextButton", header)
minBtn.Size = UDim2.new(0, 30, 0, 26)
minBtn.Position = UDim2.new(1, -68, 0, 5)
minBtn.BackgroundColor3 = RED
minBtn.BorderSizePixel = 0
minBtn.Text = "—"
minBtn.Font = Enum.Font.GothamBold
minBtn.TextColor3 = TEXT
minBtn.TextSize = 16
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0, 6)

local closeBtn = Instance.new("TextButton", header)
closeBtn.Size = UDim2.new(0, 30, 0, 26)
closeBtn.Position = UDim2.new(1, -34, 0, 5)
closeBtn.BackgroundColor3 = RED
closeBtn.BorderSizePixel = 0
closeBtn.Text = "✕"
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextColor3 = TEXT
closeBtn.TextSize = 14
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)

-- Container
local body = Instance.new("Frame", main)
body.BackgroundTransparency = 1
body.Position = UDim2.new(0, 12, 0, 48)
body.Size = UDim2.new(1, -24, 1, -100)

local list = Instance.new("UIListLayout", body)
list.Padding = UDim.new(0, 8)
list.SortOrder = Enum.SortOrder.LayoutOrder

-- Função: criar toggle
local function makeToggle(text, key)
    local f = Instance.new("Frame", body)
    f.BackgroundColor3 = BG2
    f.Size = UDim2.new(1, 0, 0, 38)
    f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0, 6)

    local lbl = Instance.new("TextLabel", f)
    lbl.BackgroundTransparency = 1
    lbl.Position = UDim2.new(0, 12, 0, 0)
    lbl.Size = UDim2.new(1, -70, 1, 0)
    lbl.Font = Enum.Font.Gotham
    lbl.TextSize = 14
    lbl.TextColor3 = TEXT
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Text = text

    local btn = Instance.new("TextButton", f)
    btn.Size = UDim2.new(0, 50, 0, 22)
    btn.Position = UDim2.new(1, -60, 0.5, -11)
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Text = CONFIG[key] and "ON" or "OFF"
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    btn.TextColor3 = TEXT
    btn.BackgroundColor3 = CONFIG[key] and RED or Color3.fromRGB(60,60,60)
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)

    btn.MouseButton1Click:Connect(function()
        CONFIG[key] = not CONFIG[key]
        btn.Text = CONFIG[key] and "ON" or "OFF"
        btn.BackgroundColor3 = CONFIG[key] and RED or Color3.fromRGB(60,60,60)
    end)
end

makeToggle("Auto Build",          "AutoBuild")
makeToggle("Auto Assign Workers", "AutoAssignWorkers")
makeToggle("Auto Sell",           "AutoSell")
makeToggle("Auto Upgrade",        "AutoUpgrade")

-- Botão executar manual
local runBtn = Instance.new("TextButton", body)
runBtn.Size = UDim2.new(1, 0, 0, 36)
runBtn.BackgroundColor3 = RED
runBtn.BorderSizePixel = 0
runBtn.Text = "▶ Executar Ciclo Agora"
runBtn.Font = Enum.Font.GothamBold
runBtn.TextSize = 14
runBtn.TextColor3 = TEXT
Instance.new("UICorner", runBtn).CornerRadius = UDim.new(0, 6)

runBtn.MouseButton1Click:Connect(function()
    pcall(AutoBuild); pcall(AutoAssignWorkers); pcall(AutoUpgrade); pcall(AutoSell)
end)

-- Footer
local footer = Instance.new("TextLabel", main)
footer.BackgroundTransparency = 1
footer.Position = UDim2.new(0, 0, 1, -28)
footer.Size = UDim2.new(1, 0, 0, 24)
footer.Font = Enum.Font.Gotham
footer.TextSize = 11
footer.TextColor3 = Color3.fromRGB(150,150,150)
footer.Text = "Pressione [ K ] para abrir/fechar"

-- ================== TOGGLE / MINIMIZAR ==================
local visible = true
local minimized = false

local function setVisible(v)
    visible = v
    main.Visible = v
end

minBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        body.Visible = false
        footer.Visible = false
        TweenService:Create(main, TweenInfo.new(0.2), {Size = UDim2.new(0, 320, 0, 36)}):Play()
    else
        TweenService:Create(main, TweenInfo.new(0.2), {Size = UDim2.new(0, 320, 0, 360)}):Play()
        task.wait(0.2)
        body.Visible = true
        footer.Visible = true
    end
end)

closeBtn.MouseButton1Click:Connect(function() setVisible(false) end)

UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Enum.KeyCode.K then
        setVisible(not visible)
    end
end)

-- ================== LOOP PRINCIPAL ==================
task.spawn(function()
    while true do
        pcall(function()
            AutoBuild()
            AutoAssignWorkers()
            AutoUpgrade()
        end)
        task.wait(1.2)
        if tick() % CONFIG.SellDelay < 1.5 then
            pcall(AutoSell)
        end
    end
end)

print("🚀 RENAN.mod | FZ carregado! Pressione [K] para abrir/fechar o menu.")
