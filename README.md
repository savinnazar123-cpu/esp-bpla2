local player = game.Players.LocalPlayer
local mouse = player:GetMouse()
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local telekinesisActive = false
local heldPart = nil
local bv, bg

local holdDistance = 15
local minDistance, maxDistance = 5, 60
local moveSpeed = 10
local minSpeed, maxSpeed = 2, 30

-- ===== GUI =====
local gui = Instance.new("ScreenGui")
gui.Name = "TelekinesisGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 260, 0, 220)
panel.Position = UDim2.new(0, 20, 0.45, 0)
panel.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
panel.BackgroundTransparency = 0.1
panel.BorderSizePixel = 0
panel.Active = true
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
toggleBtn.Size = UDim2.new(0, 230, 0, 40)
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

-- Кнопка "Оттолкнуть"
local pushBtn = Instance.new("TextButton")
pushBtn.Size = UDim2.new(0, 230, 0, 40)
pushBtn.Position = UDim2.new(0, 15, 0, 90)
pushBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
pushBtn.Text = "PUSH 💥"
pushBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
pushBtn.Font = Enum.Font.GothamBold
pushBtn.TextSize = 14
pushBtn.Parent = panel

local pushCorner = Instance.new("UICorner")
pushCorner.CornerRadius = UDim.new(0, 12)
pushCorner.Parent = pushBtn

local pushStroke = Instance.new("UIStroke")
pushStroke.Color = Color3.fromRGB(255, 120, 80)
pushStroke.Thickness = 1.5
pushStroke.Parent = pushBtn

-- ===== Слайдер: Скорость =====
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0, 230, 0, 18)
speedLabel.Position = UDim2.new(0, 15, 0, 140)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Speed: " .. moveSpeed
speedLabel.TextColor3 = Color3.fromRGB(255,255,255)
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextSize = 13
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Parent = panel

local speedBack = Instance.new("Frame")
speedBack.Size = UDim2.new(0, 230, 0, 10)
speedBack.Position = UDim2.new(0, 15, 0, 160)
speedBack.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
speedBack.Parent = panel

local speedBackCorner = Instance.new("UICorner")
speedBackCorner.CornerRadius = UDim.new(1, 0)
speedBackCorner.Parent = speedBack

local speedFill = Instance.new("Frame")
speedFill.Size = UDim2.new((moveSpeed - minSpeed) / (maxSpeed - minSpeed), 0, 1, 0)
speedFill.BackgroundColor3 = Color3.fromRGB(150, 100, 255)
speedFill.Parent = speedBack

local speedFillCorner = Instance.new("UICorner")
speedFillCorner.CornerRadius = UDim.new(1, 0)
speedFillCorner.Parent = speedFill

local speedKnob = Instance.new("Frame")
speedKnob.Size = UDim2.new(0, 16, 0, 16)
speedKnob.AnchorPoint = Vector2.new(0.5, 0.5)
speedKnob.Position = UDim2.new((moveSpeed - minSpeed) / (maxSpeed - minSpeed), 0, 0.5, 0)
speedKnob.BackgroundColor3 = Color3.fromRGB(255,255,255)
speedKnob.Parent = speedBack

local speedKnobCorner = Instance.new("UICorner")
speedKnobCorner.CornerRadius = UDim.new(1, 0)
speedKnobCorner.Parent = speedKnob

-- ===== Слайдер: Дальность =====
local distLabel = Instance.new("TextLabel")
distLabel.Size = UDim2.new(0, 230, 0, 18)
distLabel.Position = UDim2.new(0, 15, 0, 180)
distLabel.BackgroundTransparency = 1
distLabel.Text = "Distance: " .. holdDistance
distLabel.TextColor3 = Color3.fromRGB(255,255,255)
distLabel.Font = Enum.Font.Gotham
distLabel.TextSize = 13
distLabel.TextXAlignment = Enum.TextXAlignment.Left
distLabel.Parent = panel

local distBack = Instance.new("Frame")
distBack.Size = UDim2.new(0, 230, 0, 10)
distBack.Position = UDim2.new(0, 15, 0, 200)
distBack.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
distBack.Parent = panel

local distBackCorner = Instance.new("UICorner")
distBackCorner.CornerRadius = UDim.new(1, 0)
distBackCorner.Parent = distBack

local distFill = Instance.new("Frame")
distFill.Size = UDim2.new((holdDistance - minDistance) / (maxDistance - minDistance), 0, 1, 0)
distFill.BackgroundColor3 = Color3.fromRGB(150, 100, 255)
distFill.Parent = distBack

local distFillCorner = Instance.new("UICorner")
distFillCorner.CornerRadius = UDim.new(1, 0)
distFillCorner.Parent = distFill

local distKnob = Instance.new("Frame")
distKnob.Size = UDim2.new(0, 16, 0, 16)
distKnob.AnchorPoint = Vector2.new(0.5, 0.5)
distKnob.Position = UDim2.new((holdDistance - minDistance) / (maxDistance - minDistance), 0, 0.5, 0)
distKnob.BackgroundColor3 = Color3.fromRGB(255,255,255)
distKnob.Parent = distBack

local distKnobCorner = Instance.new("UICorner")
distKnobCorner.CornerRadius = UDim.new(1, 0)
distKnobCorner.Parent = distKnob

-- ===== Перетаскивание панели =====
local dragging = false
local dragStart, startPos

title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = panel.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        panel.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- ===== Логика слайдеров (обобщённая функция) =====
local function setupSlider(back, fill, knob, minVal, maxVal, onChange)
    local draggingThis = false

    local function update(inputPos)
        local relX = math.clamp((inputPos.X - back.AbsolutePosition.X) / back.AbsoluteSize.X, 0, 1)
        local val = minVal + relX * (maxVal - minVal)
        fill.Size = UDim2.new(relX, 0, 1, 0)
        knob.Position = UDim2.new(relX, 0, 0.5, 0)
        onChange(val)
    end

    knob.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            draggingThis = true
        end
    end)
    back.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            draggingThis = true
            update(input.Position)
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if draggingThis and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            update(input.Position)
        end
    end)
    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            draggingThis = false
        end
    end)
end

setupSlider(speedBack, speedFill, speedKnob, minSpeed, maxSpeed, function(val)
    moveSpeed = val
    speedLabel.Text = "Speed: " .. math.floor(val)
end)

setupSlider(distBack, distFill, distKnob, minDistance, maxDistance, function(val)
    holdDistance = val
    distLabel.Text = "Distance: " .. math.floor(val)
end)

-- ===== Крестик =====
local function releasePart()
    if heldPart then
        local highlight = heldPart:FindFirstChild("TelekinesisHighlight")
        if highlight then highlight:Destroy() end
        if bv then bv:Destroy() end
        if bg then bg:Destroy() end
        heldPart = nil
    end
end

closeBtn.MouseButton1Click:Connect(function()
    telekinesisActive = false
    releasePart()
    gui:Destroy()
end)

-- ===== Toggle =====
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
    if not telekinesisActive then releasePart() end
end)

-- ===== Захват объекта =====
local function grabPart(part)
    if not part or part.Anchored or part:IsA("Terrain") then return end

    local model = part:FindFirstAncestorOfClass("Model")
    if model and model.PrimaryPart then
        part = model.PrimaryPart
    end

    heldPart = part

    local totalMass = part:GetMass()
    if totalMass < 1 then totalMass = 1 end

    bv = Instance.new("BodyVelocity")
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Velocity = Vector3.new(0,0,0)
    bv.Parent = part

    bg = Instance.new("BodyGyro")
    bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    bg.P = math.max(3000, totalMass * 500)
    bg.CFrame = part.CFrame
    bg.Parent = part

    local highlight = Instance.new("Highlight")
    highlight.Name = "TelekinesisHighlight"
    highlight.FillColor = Color3.fromRGB(150, 100, 255)
    highlight.FillTransparency = 0.6
    highlight.OutlineColor = Color3.fromRGB(200, 150, 255)
    highlight.Parent = part
end

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

-- Колесо мыши — тоже меняет дальность (для ПК, вдобавок к слайдеру)
UIS.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseWheel and heldPart then
        holdDistance = math.clamp(holdDistance + input.Position.Z * 2, minDistance, maxDistance)
        local relX = (holdDistance - minDistance) / (maxDistance - minDistance)
        distFill.Size = UDim2.new(relX, 0, 1, 0)
        distKnob.Position = UDim2.new(relX, 0, 0.5, 0)
        distLabel.Text = "Distance: " .. math.floor(holdDistance)
    end
end)

-- ===== Кнопка PUSH (отталкивание) =====
pushBtn.MouseButton1Click:Connect(function()
    if not heldPart or not bv then return end
    local cam = workspace.CurrentCamera
    local pushForce = 80 + moveSpeed * 5

    -- Импульс от камеры вперёд
    bv.Velocity = cam.CFrame.LookVector * pushForce

    -- Отпускаем объект сразу после толчка, чтобы он улетел свободно
    task.delay(0.05, function()
        releasePart()
    end)
end)

-- ===== Плавное перемещение объекта (пружина, без рывков) =====
local currentVelocity = Vector3.new(0,0,0)

RunService.RenderStepped:Connect(function(dt)
    if not telekinesisActive or not heldPart or not bv then return end

    local cam = workspace.CurrentCamera
    local targetPos = cam.CFrame.Position + cam.CFrame.LookVector * holdDistance
    local currentPos = heldPart.Position

    local diff = targetPos - currentPos
    local mass = heldPart:GetMass()

    -- Пружинная модель: сила пропорциональна расстоянию, демпфирование гасит колебания
    local springStrength = moveSpeed * 4
    local damping = moveSpeed * 0.6

    local desiredVelocity = diff * springStrength
    currentVelocity = currentVelocity:Lerp(desiredVelocity, math.clamp(dt * damping, 0, 1))

    bv.Velocity = currentVelocity

    bg.CFrame = CFrame.new(Vector3.new(), cam.CFrame.LookVector)
end)
