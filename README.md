--[=[
    SWILL ESP HUD (Players & Drones Toggle)
    Optimized for Delta Executor
--]=]

local Fluent = loadstring(game:HttpGet("https://github.com"))()

local Window = Fluent:CreateWindow({
    Title = "SWILL ESP | БПЛА",
    SubTitle = "by Swill Way",
    TabWidth = 160,
    Size = Vector2.new(580, 460),
    Acrylic = false,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    ESP = Window:AddTab({ Title = "Визуалы (ESP)", Icon = "eye" })
}

-- Конфигурация переключателей
local ESP_Settings = {
    Players = false,
    Drones = false,
    PlayerColor = Color3.fromRGB(0, 255, 100), -- Зеленый для людей
    DroneColor = Color3.fromRGB(255, 50, 50)   -- Красный для дронов
}

-- Таблицы для хранения активных обводок
local ActivePlayerESP = {}
local ActiveDroneESP = {}

-- Функция очистки ESP
local function removeHighlight(object, tableRef)
    if tableRef[object] then
        if tableRef[object].Highlight then
            tableRef[object].Highlight:Destroy()
        end
        tableRef[object] = nil
    end
end

-- Функция создания подсветки
local function applyHighlight(object, color, tableRef, isEnabled)
    removeHighlight(object, tableRef)
    if not isEnabled then return end

    local highlight = Instance.new("Highlight")
    highlight.FillColor = color
    highlight.FillTransparency = 0.5
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.OutlineTransparency = 0
    highlight.Adornee = object
    highlight.Parent = object

    tableRef[object] = { Highlight = highlight }
end

-- Сканнер игроков (Людей)
local function updatePlayersESP()
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game:GetService("Players").LocalPlayer and player.Character then
            if ESP_Settings.Players then
                applyHighlight(player.Character, ESP_Settings.PlayerColor, ActivePlayerESP, true)
            else
                removeHighlight(player.Character, ActivePlayerESP)
            end
        end
    end
end

-- Сканнер Дронов (ищет модели техники/БПЛА в Workspace)
local function updateDronesESP()
    for _, obj in ipairs(workspace:GetDescendants()) do
        -- Поиск объектов по ключевым названиям в игре "БПЛА"
        if obj:IsA("Model") and (obj.Name:lower():find("drone") or obj.Name:lower():find("бпла") or obj.Name:lower():find("uav") or obj.Name:lower():find("plane")) then
            if not game:GetService("Players"):GetPlayerFromCharacter(obj) then -- Проверка, что это не игрок
                if ESP_Settings.Drones then
                    applyHighlight(obj, ESP_Settings.DroneColor, ActiveDroneESP, true)
                else
                    removeHighlight(obj, ActiveDroneESP)
                end
            end
        end
    end
end

-- Элементы интерфейса управления
Tabs.ESP:AddToggle("TogglePlayers", {
    Title = "Подсветка Людей (Игроков)",
    Default = false,
    Callback = function(Value)
        ESP_Settings.Players = Value
        updatePlayersESP()
    end
})

Tabs.ESP:AddToggle("ToggleDrones", {
    Title = "Подсветка Дронов (Техники)",
    Default = false,
    Callback = function(Value)
        ESP_Settings.Drones = Value
        if not Value then
            for obj, _ in pairs(ActiveDroneESP) do removeHighlight(obj, ActiveDroneESP) end
        else
            updateDronesESP()
        end
    end
})

-- Постоянное обновление в цикле для отслеживания новых игроков и спавна дронов
task.spawn(function()
    while task.wait(1) do
        if ESP_Settings.Players then updatePlayersESP() end
        if ESP_Settings.Drones then updateDronesESP() end
    end
end)

-- Отслеживание выхода игроков для очистки памяти
game:GetService("Players").PlayerRemoving:Connect(function(player)
    if player.Character then removeHighlight(player.Character, ActivePlayerESP) end
end)

Fluent:Notify({
    Title = "SWILL ESP",
    Content = "Скрипт переключения ESP готов к работе!",
    Duration = 5
})
Window:SelectTab(1)
