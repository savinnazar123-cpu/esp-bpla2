local player = game.Players.LocalPlayer
local mouse = player:GetMouse()
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")
local TweenService = game:GetService("TweenService")

local pullActive = false
local pulling = false
local targetPoint = nil
local bv

local pullSpeed = 15
local minSpeed, maxSpeed = 5, 60
local maxRange = 200

local hookBeam, hookAttachment0, hookAttachment1, pointMarker
local trailAttachment0, trailAttachment1, speedTrail

-- ===== GUI =====
local gui = Instance.new("ScreenGui")
gui.Name = "SelfPullGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 240, 0, 150)
panel.Position = UDim2.new(0, 20, 0.55, 0)
panel.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
panel.BackgroundTransparency = 0.1
panel.BorderSizePixel = 0
panel.Active = true
panel.Parent = gui

local panelCorner = Instance.new("UICorner")
panelCorner.CornerRadius = UDim.new(0, 18)
panelCorner.Parent = panel

local panelStroke = Instance.new("UIStroke")
panelStroke.Color = Color3.fromRGB(100, 200, 255)
panelStroke.Thickness = 1.5
panelStroke.Parent = panel

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -50, 0, 30)
title.Position = UDim2.new(0, 15, 0, 5)
title.BackgroundTransparency = 1
title.Text = "GRAPPLE PULL"
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
toggleBtn.Size = UDim2.new(0, 210, 0, 40)
toggleBtn.Position = UDim2.new(0, 15, 0, 42)
toggleBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
toggleBtn.Text = "GRAPPLE: OFF"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 14
toggleBtn.Parent = panel

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 12)
toggleCorner.Parent = toggleBtn

local toggleStroke = Instance.new("UIStroke")
toggleStroke.Color = Color3.fromRGB(100, 200, 255)
toggleStroke.Thickness = 1.5
toggleStroke.Parent = toggleBtn

local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0, 210, 0, 18)
speedLabel.Position = UDim2.new(0, 15, 0, 90)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Pull Speed: " .. pullSpeed
speedLabel.TextColor3 = Color3.fromRGB(255,255,255)
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextSize = 13
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Parent = panel

local speedBack = Instance.new("Frame")
speedBack.Size = UDim2.new(0, 210, 0, 10)
speedBack.Position = UDim2.new(0, 15, 0, 110)
speedBack.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
speedBack.Parent = panel

local speedBackCorner = Instance.new("UICorner")
speedBackCorner.CornerRadius = UDim.new(1, 0)
speedBackCorner.Parent = speedBack

local speedFill = Instance.new("Frame")
speedFill.Size = UDim2.new((pullSpeed - minSpeed) / (maxSpeed - minSpeed), 0, 1, 0)
speedFill.BackgroundColor3 = Color3.fromRGB(100, 200, 255)
speedFill.Parent = speedBack

local speedFillCorner = Instance.new("UICorner")
speedFillCorner.CornerRadius = UDim.new(1, 0)
speedFillCorner.Parent = speedFill

local speedKnob = Instance.new("Frame")
speedKnob.Size = UDim2.new(0, 16, 0, 16)
speedKnob.AnchorPoint = Vector2.new(0.5, 0.5)
speedKnob.Position = UDim2.new((pullSpeed - minSpeed) / (maxSpeed - minSpeed), 0, 0.5, 0)
speedKnob.BackgroundColor3 = Color3.fromRGB(255,255,255)
speedKnob.Parent = speedBack

local speedKnobCorner = Instance.new("UICorner")
speedKnobCorner.CornerRadius = UDim.new(1, 0)
speedKnobCorner.Parent = speedKnob

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

-- ===== Слайдер =====
local draggingSlider = false

local function updateSpeedSlider(inputPos)
    local relX = math.clamp((inputPos.X - speedBack.AbsolutePosition.X) / speedBack.AbsoluteSize.X, 0, 1)
    pullSpeed = minSpeed + relX * (maxSpeed - minSpeed)
    speedFill.Size = UDim2.new(relX, 0, 1, 0)
    speedKnob.Position = UDim2.new(relX, 0, 0.5, 0)
    speedLabel.Text = "Pull Speed: " .. math.floor(pullSpeed)
end

speedKnob.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = true
    end
end)
speedBack.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = true
        updateSpeedSlider(input.Position)
    end
end)
UIS.InputChanged:Connect(function(input)
    if draggingSlider and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        updateSpeedSlider(input.Position)
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = false
    end
end)

-- ===== Визуальные эффекты крюка =====
local function createHookVisual(hrp, hitPos)
    pointMarker = Instance.new("Part")
    pointMarker.Shape = Enum.PartType.Ball
    pointMarker.Size = Vector3.new(0.6, 0.6, 0.6)
    pointMarker.Position = hitPos
    pointMarker.Anchored = true
    pointMarker.CanCollide = false
    pointMarker.Material = Enum.Material.Neon
    pointMarker.Color = Color3.fromRGB(100, 200, 255)
    pointMarker.Parent = workspace

    local mesh = Instance.new("SpecialMesh")
    mesh.MeshType = Enum.MeshType.Sphere
    mesh.Parent = pointMarker

    local flash = pointMarker:Clone()
    flash.Size = Vector3.new(1,1,1)
    flash.Transparency = 0.2
    flash.Parent = workspace
    local flashTween = TweenService:Create(flash, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Size = Vector3.new(4,4,4),
        Transparency = 1
    })
    flashTween:Play()
    Debris:AddItem(flash, 0.35)

    hookAttachment0 = Instance.new("Attachment")
    hookAttachment0.Parent = hrp

    hookAttachment1 = Instance.new("Attachment")
    hookAttachment1.Parent = pointMarker

    hookBeam = Instance.new("Beam")
    hookBeam.Attachment0 = hookAttachment0
    hookBeam.Attachment1 = hookAttachment1
    hookBeam.Width0 = 0.15
    hookBeam.Width1 = 0.15
    hookBeam.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 200, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(200, 230, 255))
    })
    hookBeam.Transparency = NumberSequence.new(0.2)
    hookBeam.FaceCamera = true
    hookBeam.Parent = hrp
end

local function destroyHookVisual()
    if hookBeam then hookBeam:Destroy(); hookBeam = nil end
    if hookAttachment0 then hookAttachment0:Destroy(); hookAttachment0 = nil end
    if hookAttachment1 then hookAttachment1:Destroy(); hookAttachment1 = nil end
    if pointMarker then pointMarker:Destroy(); pointMarker = nil end
end

local function createSpeedTrail(hrp)
    trailAttachment0 = Instance.new("Attachment")
    trailAttachment0.Position = Vector3.new(0, 1, 0)
    trailAttachment0.Parent = hrp

    trailAttachment1 = Instance.new("Attachment")
    trailAttachment1.Position = Vector3.new(0, -1, 0)
    trailAttachment1.Parent = hrp

    speedTrail = Instance.new("Trail")
    speedTrail.Attachment0 = trailAttachment0
    speedTrail.Attachment1 = trailAttachment1
    speedTrail.Lifetime = 0.3
    speedTrail.Color = ColorSequence.new(Color3.fromRGB(100, 200, 255))
    speedTrail.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(1, 1)
    })
    speedTrail.WidthScale = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(1, 0)
    })
    speedTrail.Parent = hrp
end

local function destroySpeedTrail()
    if speedTrail then speedTrail:Destroy(); speedTrail = nil end
    if trailAttachment0 then trailAttachment0:Destroy(); trailAttachment0 = nil end
    if trailAttachment1 then trailAttachment1:Destroy(); trailAttachment1 = nil end
end

-- ===== Toggle / Stop =====
local function setToggleVisual(active)
    if active then
        toggleBtn.BackgroundColor3 = Color3.fromRGB(100, 200, 255)
        toggleBtn.Text = "GRAPPLE: ON"
        toggleStroke.Color = Color3.fromRGB(255, 255, 255)
    else
        toggleBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
        toggleBtn.Text = "GRAPPLE: OFF"
        toggleStroke.Color = Color3.fromRGB(100, 200, 255)
    end
end

local function stopPull()
    pulling = false
    targetPoint = nil
    local char = player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then
        local existing = hrp:FindFirstChild("PullVelocity")
        if existing then existing:Destroy() end
    end
    destroyHookVisual()
    destroySpeedTrail()
end

toggleBtn.MouseButton1Click:Connect(function()
    pullActive = not pullActive
    setToggleVisual(pullActive)
    if not pullActive then stopPull() end
end)

closeBtn.MouseButton1Click:Connect(function()
    pullActive = false
    stopPull()
    gui:Destroy()
end)

-- ===== Клик — точка притяжения =====
mouse.Button1Down:Connect(function()
    if not pullActive then return end

    local char = player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local hitPos = mouse.Hit and mouse.Hit.Position
    if not hitPos then return end

    if (hitPos - hrp.Position).Magnitude > maxRange then return end

    targetPoint = hitPos
    pulling = true

    local bvExisting = hrp:FindFirstChild("PullVelocity")
    if bvExisting then bvExisting:Destroy() end

    bv = Instance.new("BodyVelocity")
    bv.Name = "PullVelocity"
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Velocity = Vector3.new(0,0,0)
    bv.Parent = hrp

    destroyHookVisual()
    createHookVisual(hrp, hitPos)
    createSpeedTrail(hrp)
end)

mouse.Button2Down:Connect(function()
    if pulling then stopPull() end
end)

-- ===== Полёт к точке =====
RunService.RenderStepped:Connect(function(dt)
    if not pulling or not targetPoint then return end

    local char = player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChild("Humanoid")
    if not hrp or not hum then stopPull(); return end

    local bvCurrent = hrp:FindFirstChild("PullVelocity")
    if not bvCurrent then stopPull(); return end

    local diff = targetPoint - hrp.Position
    local dist = diff.Magnitude

    if dist < 3 then
        stopPull()
        return
    end

    bvCurrent.Velocity = diff.Unit * pullSpeed
end)
