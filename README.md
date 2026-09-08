local player = game.Players.LocalPlayer
local mouse = player:GetMouse()
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local telekinesisActive = false
local heldPart = nil
local holdDistance = 15
local bv, bg

-- ===== GUI =====
local gui = Instance.new("ScreenGui")
gui.Name = "TelekinesisGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 220, 0, 90)
panel.Position = UDim2.new(0, 20, 0.6, 0)
panel.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
panel.BackgroundTransparency = 0.1
panel.BorderSizePixel = 0
panel.Active = true
panel.Draggable = false -- своя реализация ниже (работает и на тач, и на мыши)
panel.Parent = gui

local panelCorner = Instance.new("UICorner")
panelCorner.CornerRadius = UDim.new(0, 18)
panelCorner.Parent = panel

local panelStroke = Instance.new("UIStroke")
panelStroke.Color = Color3.fromRGB(150, 100, 255)
panelStroke.Thickness = 1.5
panelStroke.Parent = panel

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -50, 0, 30)
title.Position = UDim2.new(0, 15, 0, 5)
title.BackgroundTransparency = 1
title.Text = "TELEKINESIS"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = panel

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 26, 0, 26)
closeBtn.Position = UDim2.new(1, -35, 0, 6)
closeBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 16
closeBtn.Parent = panel

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(1, 0)
closeCorner.Parent = closeBtn

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 190, 0, 40)
toggleBtn.Position = UDim2.new(0, 15, 0, 42)
toggleBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
toggleBtn.Text = "TELEKINESIS: OFF"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 14
toggleBtn.Parent = panel

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 12)
toggleCorner.Parent = toggleBtn

local toggleStroke = Instance.new("UIStroke")
toggleStroke.Color = Color3.fromRGB(150, 100, 255)
toggleStroke.Thickness = 1.5
toggleStroke.Parent = toggleBtn

-- ===== Перетаскивание панели (drag), не мешает кнопкам =====
local dragging = false
local dragStart, startPos

local function beginDrag(input)
    dragging = true
    dragStart = input.Position
    startPos = panel.Position
end

local function updateDrag(input)
    if not dragging then return end
    local delta = input.Position - dragStart
    panel.Position = UDim2.new(
        startPos.X.Scale, startPos.X.Offset + delta.X,
        startPos.Y.Scale, startPos.Y.Offset + delta.Y
    )
end

title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        beginDrag(input)
    end
end)

UIS.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        updateDrag(input)
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- ===== Крестик: скрывает панель и выключает весь скрипт =====
closeBtn.MouseButton1Click:Connect(function()
    telekinesisActive = false
    if heldPart then
        if bv then bv:Destroy() end
        if bg then bg:Destroy() end
        heldPart = nil
    end
    gui:Destroy()
end)

-- ===== Логика телекинеза =====
local function setToggleVisual(active)
    if active then
        toggleBtn.BackgroundColor3 = Color3.fromRGB(150, 100, 255)
        toggleBtn.Text = "TELEKINESIS: ON"
        toggleStroke.Color = Color3.fromRGB(255, 255, 255)
    else
        toggleBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
        toggleBtn.Text = "TELEKINESIS: OFF"
        toggleStroke.Color = Color3.fromRGB(150, 100, 255)
    end
end

toggleBtn.MouseButton1Click:Connect(function()
    telekinesisActive = not telekinesisActive
    setToggleVisual(telekinesisActive)
    if not telekinesisActive and heldPart then
        if bv then bv:Destroy() end
        if bg then bg:Destroy() end
        heldPart = nil
    end
end)

local function grabPart(part)
    if not part or part.Anchored or part:IsA("Terrain") then return end
    heldPart = part

    bv = Instance.new("BodyVelocity")
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Velocity = Vector3.new(0,0,0)
    bv.Parent = part

    bg = Instance.new("BodyGyro")
    bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    bg.P = 3000
    bg.CFrame = part.CFrame
    bg.Parent = part
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "TelekinesisHighlight"
    highlight.FillColor = Color3.fromRGB(150, 100, 255)
    highlight.FillTransparency = 0.6
    highlight.OutlineColor = Color3.fromRGB(200, 150, 255)
    highlight.Parent = part
end

local function releasePart()
    if heldPart then
        local highlight = heldPart:FindFirstChild("TelekinesisHighlight")
        if highlight then highlight:Destroy() end
        if bv then bv:Destroy() end
        if bg then bg:Destroy() end
        heldPart = nil
    end
end

-- Клик/тап по объекту — захват или отпускание
mouse.Button1Down:Connect(function()
    if not telekinesisActive then return end

    if heldPart then
        releasePart()
        return
    end

    local target = mouse.Target
    if target and not target.Anchored then
        grabPart(target)
    end
end)

-- Колесо мыши — приближение/отдаление объекта (только для ПК)
UIS.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseWheel and heldPart then
        holdDistance = math.clamp(holdDistance + input.Position.Z * 2, 5, 50)
    end
end)

-- Обновление позиции объекта перед камерой
RunService.RenderStepped:Connect(function(dt)
    if not telekinesisActive or not heldPart or not bv then return end

    local cam = workspace.CurrentCamera
    local targetPos = cam.CFrame.Position + cam.CFrame.LookVector * holdDistance

    local currentPos = heldPart.Position
    local diff = targetPos - currentPos
    bv.Velocity = diff * 10

    bg.CFrame = CFrame.new(Vector3.new(), cam.CFrame.LookVector)
end)
