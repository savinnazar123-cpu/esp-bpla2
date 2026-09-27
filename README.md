-- ═══════════════════════════════════════════════════════════════
--   🚀 ULTIMATE FLY SYSTEM v7.1 — ЧАСТЬ 1/2
--   Скопируй ЧАСТЬ 1, затем ЧАСТЬ 2, склей в один скрипт
-- ═══════════════════════════════════════════════════════════════

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Debris = game:GetService("Debris")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local Humanoid = Character:WaitForChild("Humanoid")
local Camera = workspace.CurrentCamera

local CONFIG = {
    flyEnabled = false,
    flySpeed = 50,
    minSpeed = 10,
    maxSpeed = 1000,
    smoothing = 0.15,
    touchControlEnabled = true,
    keyboardControlEnabled = true,
    trailEnabled = true,
    glowEnabled = true,
    particlesEnabled = true,
    soundEnabled = true,
    autoResetEnabled = true,
    fallProtectionEnabled = true,
    maxHeight = 5000,
    minHeight = -50,
    panelVisible = true,
    panelOpen = true,
    themeColor = Color3.fromRGB(0, 200, 255),
    accentColor = Color3.fromRGB(0, 255, 200),
    warningColor = Color3.fromRGB(255, 80, 80),
    flyTime = 0,
    totalDistance = 0,
    maxSpeedReached = 0
}

local STATE = {
    forward = false,
    backward = false,
    left = false,
    right = false,
    up = false,
    down = false,
    boost = false,
    velocity = Vector3.new(0, 0, 0),
    targetVelocity = Vector3.new(0, 0, 0),
    lastPosition = Vector3.new(0, 0, 0),
    startPosition = Vector3.new(0, 0, 0),
    isFlying = false,
    isBoosting = false,
    isMoving = false,
    flyConnection = nil,
    renderConnection = nil,
    currentTrailParts = {},
    startTime = 0,
    lastUpdate = 0,
    frameCount = 0
}

local function clamp(value, min, max)
    if value < min then return min end
    if value > max then return max end
    return value
end

local function round(num)
    return math.floor(num + 0.5)
end

local function formatNumber(num)
    if num >= 1000000 then
        return string.format("%.1fM", num / 1000000)
    elseif num >= 1000 then
        return string.format("%.1fK", num / 1000)
    else
        return tostring(round(num))
    end
end

local function formatTime(seconds)
    local mins = math.floor(seconds / 60)
    local secs = math.floor(seconds % 60)
    return string.format("%02d:%02d", mins, secs)
end

local function getDistance(a, b)
    return (a - b).Magnitude
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "UltimateFlySystem"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = PlayerGui

local MainPanel = Instance.new("Frame")
MainPanel.Name = "MainPanel"
MainPanel.Size = UDim2.new(0, 420, 0, 480)
MainPanel.Position = UDim2.new(0.5, -210, 0.5, -240)
MainPanel.BackgroundColor3 = Color3.fromRGB(8, 6, 20)
MainPanel.BackgroundTransparency = 0.05
MainPanel.BorderSizePixel = 3
MainPanel.BorderColor3 = CONFIG.themeColor
MainPanel.ClipsDescendants = true
MainPanel.Active = true
MainPanel.Draggable = false
MainPanel.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 22)
MainCorner.Parent = MainPanel

local OuterGlow = Instance.new("Frame")
OuterGlow.Name = "OuterGlow"
OuterGlow.Size = UDim2.new(1, 14, 1, 14)
OuterGlow.Position = UDim2.new(0, -7, 0, -7)
OuterGlow.BackgroundTransparency = 1
OuterGlow.BorderSizePixel = 4
OuterGlow.BorderColor3 = CONFIG.themeColor
OuterGlow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
OuterGlow.ZIndex = -1
OuterGlow.Parent = MainPanel

local OuterCorner = Instance.new("UICorner")
OuterCorner.CornerRadius = UDim.new(0, 22)
OuterCorner.Parent = OuterGlow

local InnerGlow = Instance.new("Frame")
InnerGlow.Name = "InnerGlow"
InnerGlow.Size = UDim2.new(1, 6, 1, 6)
InnerGlow.Position = UDim2.new(0, -3, 0, -3)
InnerGlow.BackgroundTransparency = 1
InnerGlow.BorderSizePixel = 2
InnerGlow.BorderColor3 = CONFIG.accentColor
InnerGlow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
InnerGlow.ZIndex = -1
InnerGlow.Parent = MainPanel

local InnerCorner = Instance.new("UICorner")
InnerCorner.CornerRadius = UDim.new(0, 20)
InnerCorner.Parent = InnerGlow

local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 55)
TitleBar.BackgroundTransparency = 1
TitleBar.Parent = MainPanel

local TitleIcon = Instance.new("TextLabel")
TitleIcon.Name = "Icon"
TitleIcon.Size = UDim2.new(0, 40, 0, 40)
TitleIcon.Position = UDim2.new(0, 15, 0, 8)
TitleIcon.BackgroundTransparency = 1
TitleIcon.Text = "🚀"
TitleIcon.TextSize = 32
TitleIcon.Font = Enum.Font.GothamBold
TitleIcon.Parent = TitleBar

local TitleText = Instance.new("TextLabel")
TitleText.Name = "Title"
TitleText.Size = UDim2.new(0.5, 0, 1, 0)
TitleText.Position = UDim2.new(0, 60, 0, 0)
TitleText.BackgroundTransparency = 1
TitleText.Text = "ULTIMATE FLY"
TitleText.TextColor3 = CONFIG.themeColor
TitleText.TextSize = 22
TitleText.Font = Enum.Font.GothamBold
TitleText.TextXAlignment = Enum.TextXAlignment.Left
TitleText.Parent = TitleBar

local TitleGlow = Instance.new("TextLabel")
TitleGlow.Name = "TitleGlow"
TitleGlow.Size = UDim2.new(0.5, 0, 1, 0)
TitleGlow.Position = UDim2.new(0, 60, 0, 0)
TitleGlow.BackgroundTransparency = 1
TitleGlow.Text = "ULTIMATE FLY"
TitleGlow.TextColor3 = Color3.fromRGB(0, 150, 255)
TitleGlow.TextSize = 22
TitleGlow.Font = Enum.Font.GothamBold
TitleGlow.TextXAlignment = Enum.TextXAlignment.Left
TitleGlow.TextTransparency = 0.5
TitleGlow.Parent = TitleBar

local VersionLabel = Instance.new("TextLabel")
VersionLabel.Name = "Version"
VersionLabel.Size = UDim2.new(0, 60, 0, 15)
VersionLabel.Position = UDim2.new(0, 60, 0, 36)
VersionLabel.BackgroundTransparency = 1
VersionLabel.Text = "v7.1"
VersionLabel.TextColor3 = Color3.fromRGB(150, 150, 180)
VersionLabel.TextSize = 11
VersionLabel.Font = Enum.Font.Gotham
VersionLabel.TextXAlignment = Enum.TextXAlignment.Left
VersionLabel.Parent = TitleBar

local CloseButton = Instance.new("TextButton")
CloseButton.Name = "CloseButton"
CloseButton.Size = UDim2.new(0, 38, 0, 38)
CloseButton.Position = UDim2.new(1, -48, 0, 9)
CloseButton.BackgroundColor3 = CONFIG.warningColor
CloseButton.BackgroundTransparency = 0.15
CloseButton.BorderSizePixel = 2
CloseButton.BorderColor3 = CONFIG.warningColor
CloseButton.Text = "✕"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 22
CloseButton.Font = Enum.Font.GothamBold
CloseButton.Parent = TitleBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 10)
CloseCorner.Parent = CloseButton

local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Size = UDim2.new(0, 38, 0, 38)
ToggleButton.Position = UDim2.new(1, -94, 0, 9)
ToggleButton.BackgroundColor3 = CONFIG.themeColor
ToggleButton.BackgroundTransparency = 0.15
ToggleButton.BorderSizePixel = 2
ToggleButton.BorderColor3 = CONFIG.themeColor
ToggleButton.Text = "−"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextSize = 26
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.Parent = TitleBar

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 10)
ToggleCorner.Parent = ToggleButton

local FlyButton = Instance.new("TextButton")
FlyButton.Name = "FlyButton"
FlyButton.Size = UDim2.new(0.8, 0, 0, 48)
FlyButton.Position = UDim2.new(0.1, 0, 0, 72)
FlyButton.BackgroundColor3 = CONFIG.warningColor
FlyButton.BackgroundTransparency = 0.15
FlyButton.BorderSizePixel = 2
FlyButton.BorderColor3 = CONFIG.warningColor
FlyButton.Text = "🛫 ВЗЛЕТЕТЬ"
FlyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FlyButton.TextSize = 20
FlyButton.Font = Enum.Font.GothamBold
FlyButton.Parent = MainPanel

local FlyButtonCorner = Instance.new("UICorner")
FlyButtonCorner.CornerRadius = UDim.new(0, 12)
FlyButtonCorner.Parent = FlyButton

local StatusFrame = Instance.new("Frame")
StatusFrame.Name = "StatusFrame"
StatusFrame.Size = UDim2.new(0.9, 0, 0, 40)
StatusFrame.Position = UDim2.new(0.05, 0, 0, 128)
StatusFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 30)
StatusFrame.BackgroundTransparency = 0.3
StatusFrame.BorderSizePixel = 1
StatusFrame.BorderColor3 = CONFIG.themeColor
StatusFrame.Parent = MainPanel

local StatusCorner = Instance.new("UICorner")
StatusCorner.CornerRadius = UDim.new(0, 8)
StatusCorner.Parent = StatusFrame

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Name = "StatusLabel"
StatusLabel.Size = UDim2.new(0.5, 0, 1, 0)
StatusLabel.Position = UDim2.new(0, 10, 0, 0)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "СТАТУС: ОЖИДАНИЕ"
StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 200)
StatusLabel.TextSize = 13
StatusLabel.Font = Enum.Font.GothamBold
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
StatusLabel.Parent = StatusFrame

local StatusValue = Instance.new("TextLabel")
StatusValue.Name = "StatusValue"
StatusValue.Size = UDim2.new(0.5, 0, 1, 0)
StatusValue.Position = UDim2.new(0.5, -10, 0, 0)
StatusValue.BackgroundTransparency = 1
StatusValue.Text = "НЕ АКТИВЕН"
StatusValue.TextColor3 = CONFIG.warningColor
StatusValue.TextSize = 13
StatusValue.Font = Enum.Font.GothamBold
StatusValue.TextXAlignment = Enum.TextXAlignment.Right
StatusValue.Parent = StatusFrame

local SpeedFrame = Instance.new("Frame")
SpeedFrame.Name = "SpeedFrame"
SpeedFrame.Size = UDim2.new(0.9, 0, 0, 80)
SpeedFrame.Position = UDim2.new(0.05, 0, 0, 175)
SpeedFrame.BackgroundTransparency = 1
SpeedFrame.Parent = MainPanel

local SpeedLabel = Instance.new("TextLabel")
SpeedLabel.Name = "SpeedLabel"
SpeedLabel.Size = UDim2.new(0.6, 0, 0, 25)
SpeedLabel.Position = UDim2.new(0, 0, 0, 0)
SpeedLabel.BackgroundTransparency = 1
SpeedLabel.Text = "⚡ СКОРОСТЬ"
SpeedLabel.TextColor3 = Color3.fromRGB(150, 200, 255)
SpeedLabel.TextSize = 16
SpeedLabel.Font = Enum.Font.GothamBold
SpeedLabel.TextXAlignment = Enum.TextXAlignment.Left
SpeedLabel.Parent = SpeedFrame

local SpeedValue = Instance.new("TextLabel")
SpeedValue.Name = "SpeedValue"
SpeedValue.Size = UDim2.new(0.4, 0, 0, 25)
SpeedValue.Position = UDim2.new(0.6, 0, 0, 0)
SpeedValue.BackgroundTransparency = 1
SpeedValue.Text = "50"
SpeedValue.TextColor3 = CONFIG.themeColor
SpeedValue.TextSize = 22
SpeedValue.Font = Enum.Font.GothamBold
SpeedValue.TextXAlignment = Enum.TextXAlignment.Right
SpeedValue.Parent = SpeedFrame

local SpeedTrack = Instance.new("Frame")
SpeedTrack.Name = "SpeedTrack"
SpeedTrack.Size = UDim2.new(1, 0, 0, 12)
SpeedTrack.Position = UDim2.new(0, 0, 0, 40)
SpeedTrack.BackgroundColor3 = Color3.fromRGB(20, 30, 60)
SpeedTrack.BorderSizePixel = 0
SpeedTrack.Parent = SpeedFrame

local SpeedTrackCorner = Instance.new("UICorner")
SpeedTrackCorner.CornerRadius = UDim.new(1, 0)
SpeedTrackCorner.Parent = SpeedTrack

local SpeedFill = Instance.new("Frame")
SpeedFill.Name = "SpeedFill"
SpeedFill.Size = UDim2.new(0.04, 0, 1, 0)
SpeedFill.Position = UDim2.new(0, 0, 0, 0)
SpeedFill.BackgroundColor3 = CONFIG.themeColor
SpeedFill.BorderSizePixel = 0
SpeedFill.Parent = SpeedTrack

local SpeedFillCorner = Instance.new("UICorner")
SpeedFillCorner.CornerRadius = UDim.new(1, 0)
SpeedFillCorner.Parent = SpeedFill

local SpeedKnob = Instance.new("ImageButton")
SpeedKnob.Name = "SpeedKnob"
SpeedKnob.Size = UDim2.new(0, 32, 0, 32)
SpeedKnob.Position = UDim2.new(0.04, -16, 0, -10)
SpeedKnob.BackgroundColor3 = CONFIG.themeColor
SpeedKnob.BorderSizePixel = 3
SpeedKnob.BorderColor3 = CONFIG.themeColor
SpeedKnob.Image = "rbxassetid://134687856"
SpeedKnob.ImageColor3 = CONFIG.themeColor
SpeedKnob.ImageTransparency = 0.3
SpeedKnob.Parent = SpeedTrack

local SpeedKnobCorner = Instance.new("UICorner")
SpeedKnobCorner.CornerRadius = UDim.new(1, 0)
SpeedKnobCorner.Parent = SpeedKnob

local ExtraFrame = Instance.new("Frame")
ExtraFrame.Name = "ExtraFrame"
ExtraFrame.Size = UDim2.new(0.9, 0, 0, 80)
ExtraFrame.Position = UDim2.new(0.05, 0, 0, 260)
ExtraFrame.BackgroundTransparency = 1
ExtraFrame.Parent = MainPanel

local function createExtraButton(parent, x, y, text, color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.48, 0, 0, 34)
    btn.Position = UDim2.new(x, 0, y, 0)
    btn.BackgroundColor3 = color
    btn.BackgroundTransparency = 0.2
    btn.BorderSizePixel = 2
    btn.BorderColor3 = color
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 13
    btn.Font = Enum.Font.GothamBold
    btn.Parent = parent
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = btn
    
    return btn
end

local TrailToggle = createExtraButton(ExtraFrame, 0, 0, "✨ СЛЕДЫ ВКЛ", CONFIG.accentColor)
local GlowToggle = createExtraButton(ExtraFrame, 0.52, 0, "💡 СВЕЧЕНИЕ ВКЛ", CONFIG.accentColor)
local ParticlesToggle = createExtraButton(ExtraFrame, 0, 0.5, "✨ ЧАСТИЦЫ ВКЛ", CONFIG.accentColor)
local SoundToggle = createExtraButton(ExtraFrame, 0.52, 0.5, "🔊 ЗВУК ВКЛ", CONFIG.accentColor)

local StatsFrame = Instance.new("Frame")
StatsFrame.Name = "StatsFrame"
StatsFrame.Size = UDim2.new(0.9, 0, 0, 60)
StatsFrame.Position = UDim2.new(0.05, 0, 0, 348)
StatsFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 30)
StatsFrame.BackgroundTransparency = 0.3
StatsFrame.BorderSizePixel = 1
StatsFrame.BorderColor3 = CONFIG.themeColor
StatsFrame.Parent = MainPanel

local StatsCorner = Instance.new("UICorner")
StatsCorner.CornerRadius = UDim.new(0, 8)
StatsCorner.Parent = StatsFrame

local TimeLabel = Instance.new("TextLabel")
TimeLabel.Name = "Time"
TimeLabel.Size = UDim2.new(0.33, 0, 0, 25)
TimeLabel.Position = UDim2.new(0, 5, 0, 3)
TimeLabel.BackgroundTransparency = 1
TimeLabel.Text = "⏱ 00:00"
TimeLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
TimeLabel.TextSize = 13
TimeLabel.Font = Enum.Font.GothamBold
TimeLabel.TextXAlignment = Enum.TextXAlignment.Center
TimeLabel.Parent = StatsFrame

local DistanceLabel = Instance.new("TextLabel")
DistanceLabel.Name = "Distance"
DistanceLabel.Size = UDim2.new(0.33, 0, 0, 25)
DistanceLabel.Position = UDim2.new(0.33, 0, 0, 3)
DistanceLabel.BackgroundTransparency = 1
DistanceLabel.Text = "📏 0"
DistanceLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
DistanceLabel.TextSize = 13
DistanceLabel.Font = Enum.Font.GothamBold
DistanceLabel.TextXAlignment = Enum.TextXAlignment.Center
DistanceLabel.Parent = StatsFrame

local MaxSpeedLabel = Instance.new("TextLabel")
MaxSpeedLabel.Name = "MaxSpeed"
MaxSpeedLabel.Size = UDim2.new(0.33, 0, 0, 25)
MaxSpeedLabel.Position = UDim2.new(0.66, 0, 0, 3)
MaxSpeedLabel.BackgroundTransparency = 1
MaxSpeedLabel.Text = "🏆 0"
MaxSpeedLabel.TextColor3 = Color3.fromRGB(200, 200, 220)
MaxSpeedLabel.TextSize = 13
MaxSpeedLabel.Font = Enum.Font.GothamBold
MaxSpeedLabel.TextXAlignment = Enum.TextXAlignment.Center
MaxSpeedLabel.Parent = StatsFrame

local SpeedNowLabel = Instance.new("TextLabel")
SpeedNowLabel.Name = "SpeedNow"
SpeedNowLabel.Size = UDim2.new(1, 0, 0, 20)
SpeedNowLabel.Position = UDim2.new(0, 0, 0, 30)
SpeedNowLabel.BackgroundTransparency = 1
SpeedNowLabel.Text = "ТЕКУЩАЯ СКОРОСТЬ: 0"
SpeedNowLabel.TextColor3 = CONFIG.themeColor
SpeedNowLabel.TextSize = 12
SpeedNowLabel.Font = Enum.Font.GothamBold
SpeedNowLabel.TextXAlignment = Enum.TextXAlignment.Center
SpeedNowLabel.Parent = StatsFrame

local ResetButton = Instance.new("TextButton")
ResetButton.Name = "ResetButton"
ResetButton.Size = UDim2.new(0.6, 0, 0, 36)
ResetButton.Position = UDim2.new(0.2, 0, 0, 420)
ResetButton.BackgroundColor3 = CONFIG.warningColor
ResetButton.BackgroundTransparency = 0.15
ResetButton.BorderSizePixel = 2
ResetButton.BorderColor3 = CONFIG.warningColor
ResetButton.Text = "↺ СБРОСИТЬ ВСЁ"
ResetButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ResetButton.TextSize = 15
ResetButton.Font = Enum.Font.GothamBold
ResetButton.Parent = MainPanel

local ResetCorner = Instance.new("UICorner")
ResetCorner.CornerRadius = UDim.new(0, 10)
ResetCorner.Parent = ResetButton

local ClosedButton = Instance.new("ImageButton")
ClosedButton.Name = "ClosedButton"
ClosedButton.Size = UDim2.new(0, 64, 0, 64)
ClosedButton.Position = UDim2.new(1, -80, 1, -80)
ClosedButton.BackgroundColor3 = CONFIG.themeColor
ClosedButton.BackgroundTransparency = 0.15
ClosedButton.BorderSizePixel = 0
ClosedButton.Image = "rbxassetid://134687856"
ClosedButton.ImageColor3 = CONFIG.themeColor
ClosedButton.ImageTransparency = 0.3
ClosedButton.Visible = false
ClosedButton.Parent = ScreenGui

local ClosedCorner = Instance.new("UICorner")
ClosedCorner.CornerRadius = UDim.new(1, 0)
ClosedCorner.Parent = ClosedButton

local ClosedPlus = Instance.new("TextLabel")
ClosedPlus.Name = "Plus"
ClosedPlus.Size = UDim2.new(1, 0, 1, 0)
ClosedPlus.BackgroundTransparency = 1
ClosedPlus.Text = "+"
ClosedPlus.TextColor3 = Color3.fromRGB(255, 255, 255)
ClosedPlus.TextSize = 40
ClosedPlus.Font = Enum.Font.GothamBold
ClosedPlus.Parent = ClosedButton

local Notification = Instance.new("Frame")
Notification.Name = "Notification"
Notification.Size = UDim2.new(0, 300, 0, 50)
Notification.Position = UDim2.new(0.5, -150, 0, 20)
Notification.BackgroundColor3 = Color3.fromRGB(10, 10, 25)
Notification.BackgroundTransparency = 0.1
Notification.BorderSizePixel = 2
Notification.BorderColor3 = CONFIG.themeColor
Notification.Visible = false
Notification.Parent = ScreenGui

local NotifCorner = Instance.new("UICorner")
NotifCorner.CornerRadius = UDim.new(0, 10)
NotifCorner.Parent = Notification

local NotifText = Instance.new("TextLabel")
NotifText.Name = "Text"
NotifText.Size = UDim2.new(1, -20, 1, 0)
NotifText.Position = UDim2.new(0, 10, 0, 0)
NotifText.BackgroundTransparency = 1
NotifText.Text = "Уведомление"
NotifText.TextColor3 = Color3.fromRGB(255, 255, 255)
NotifText.TextSize = 14
NotifText.Font = Enum.Font.GothamBold
NotifText.Parent = Notification

local notifConnection = nil

local function showNotification(text, color)
    if notifConnection then
        notifConnection:Disconnect()
        notifConnection = nil
    end
    
    Notification.Visible = true
    Notification.BorderColor3 = color or CONFIG.themeColor
    NotifText.Text = text
    NotifText.TextColor3 = color or CONFIG.themeColor
    
    local startTime = tick()
    notifConnection = RunService.Heartbeat:Connect(function()
        local elapsed = tick() - startTime
        if elapsed > 2 then
            Notification.Visible = false
            if notifConnection then
                notifConnection:Disconnect()
                notifConnection = nil
            end
        else
            Notification.BackgroundTransparency = 0.1 + (elapsed / 2) * 0.7
        end
    end)
end

-- ═══════════════════════════════════════════════════════════════
--   🚀 ULTIMATE FLY SYSTEM v7.1 — ЧАСТЬ 2/2
--   Вставь СРАЗУ ПОСЛЕ части 1/2
-- ═══════════════════════════════════════════════════════════════

local function createTrailPart()
    if not CONFIG.trailEnabled then return end
    if not Character or not Character.Parent then return end
    if not HumanoidRootPart or not HumanoidRootPart.Parent then return end
    
    local part = Instance.new("Part")
    part.Name = "FlyTrail"
    part.Size = Vector3.new(1.2, 0.25, 2.8)
    part.Shape = Enum.PartType.Block
    part.Anchored = true
    part.CanCollide = false
    part.CanTouch = false
    part.CanQuery = false
    part.Transparency = 0.2
    part.BrickColor = BrickColor.new("Bright blue")
    part.Material = Enum.Material.Neon
    part.TopSurface = Enum.SurfaceType.Smooth
    part.BottomSurface = Enum.SurfaceType.Smooth
    part.Parent = workspace
    
    if CONFIG.glowEnabled then
        local light = Instance.new("PointLight")
        light.Range = 10
        light.Brightness = 5
        light.Color = CONFIG.themeColor
        light.Parent = part
    end
    
    if CONFIG.particlesEnabled then
        local attachment = Instance.new("Attachment")
        attachment.Parent = part
        
        local particles = Instance.new("ParticleEmitter")
        particles.Texture = "rbxasset://textures/particles/sparkles_main.dds"
        particles.Rate = 50
        particles.Lifetime = NumberRange.new(0.3, 0.8)
        particles.SpreadAngle = Vector2.new(180, 180)
        particles.VelocityInheritance = 0
        particles.Speed = NumberRange.new(3, 10)
        particles.Transparency = NumberSequence.new(0, 0.9)
        particles.Color = ColorSequence.new(CONFIG.themeColor)
        particles.Size = NumberSequence.new(0.5, 1.2)
        particles.Enabled = true
        particles.Parent = attachment
    end
    
    local pos = HumanoidRootPart.Position - Vector3.new(0, 1.5, 0)
    local look = HumanoidRootPart.CFrame.LookVector
    part.CFrame = CFrame.new(pos, pos + look)
    
    table.insert(STATE.currentTrailParts, part)
    
    task.spawn(function()
        for i = 1, 16 do
            task.wait(0.05)
            if part and part.Parent then
                part.Transparency = part.Transparency + 0.05
                part.Size = part.Size * 0.96
                if part:FindFirstChildOfClass("PointLight") then
                    part:FindFirstChildOfClass("PointLight").Brightness = part:FindFirstChildOfClass("PointLight").Brightness - 0.3
                end
            end
        end
        if part and part.Parent then
            part:Destroy()
        end
    end)
    
    Debris:AddItem(part, 1)
end

local function startFly()
    if STATE.isFlying then return end
    
    if not Character or not Character.Parent then
        Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
        Humanoid = Character:WaitForChild("Humanoid")
    end
    
    if not HumanoidRootPart or not HumanoidRootPart.Parent then return end
    if not Humanoid or not Humanoid.Parent then return end
    
    STATE.isFlying = true
    STATE.startTime = tick()
    STATE.startPosition = HumanoidRootPart.Position
    STATE.lastPosition = HumanoidRootPart.Position
    
    Humanoid.PlatformStand = true
    HumanoidRootPart.Velocity = Vector3.new(0, 0, 0)
    
    FlyButton.Text = "🛬 ПРИЗЕМЛИТЬСЯ"
    FlyButton.BackgroundColor3 = CONFIG.accentColor
    FlyButton.BorderColor3 = CONFIG.accentColor
    StatusValue.Text = "АКТИВЕН"
    StatusValue.TextColor3 = CONFIG.accentColor
    
    showNotification("🚀 ПОЛЁТ АКТИВИРОВАН", CONFIG.accentColor)
    
    STATE.flyConnection = RunService.RenderStepped:Connect(function(deltaTime)
        if not STATE.isFlying then return end
        if not HumanoidRootPart or not HumanoidRootPart.Parent then
            stopFly()
            return
        end
        if not Humanoid or not Humanoid.Parent then
            stopFly()
            return
        end
        
        STATE.frameCount = STATE.frameCount + 1
        
        local moveDirection = Vector3.new(0, 0, 0)
        local cameraCFrame = Camera.CFrame
        
        if STATE.forward then
            moveDirection = moveDirection + cameraCFrame.LookVector
        end
        if STATE.backward then
            moveDirection = moveDirection - cameraCFrame.LookVector
        end
        
        if STATE.left then
            moveDirection = moveDirection - cameraCFrame.RightVector
        end
        if STATE.right then
            moveDirection = moveDirection + cameraCFrame.RightVector
        end
        
        if STATE.up then
            moveDirection = moveDirection + Vector3.new(0, 1, 0)
        end
        if STATE.down then
            moveDirection = moveDirection - Vector3.new(0, 1, 0)
        end
        
        local speed = CONFIG.flySpeed
        if STATE.boost then
            speed = speed * 3
        end
        
        if moveDirection.Magnitude > 0 then
            STATE.isMoving = true
            STATE.targetVelocity = moveDirection.Unit * speed
            
            if CONFIG.smoothing > 0 then
                STATE.velocity = STATE.velocity:Lerp(STATE.targetVelocity, 1 - CONFIG.smoothing)
            else
                STATE.velocity = STATE.targetVelocity
            end
            
            local lookAt = HumanoidRootPart.Position + moveDirection.Unit
            local newCFrame = CFrame.lookAt(HumanoidRootPart.Position, lookAt)
            HumanoidRootPart.CFrame = newCFrame
        else
            STATE.isMoving = false
            STATE.targetVelocity = Vector3.new(0, 0, 0)
            STATE.velocity = STATE.velocity:Lerp(Vector3.new(0, 0, 0), 0.1)
        end
        
        HumanoidRootPart.Velocity = STATE.velocity
        
        local currentPos = HumanoidRootPart.Position
        local distance = getDistance(currentPos, STATE.lastPosition)
        CONFIG.totalDistance = CONFIG.totalDistance + distance
        STATE.lastPosition = currentPos
        
        local currentSpeed = STATE.velocity.Magnitude
        if currentSpeed > CONFIG.maxSpeedReached then
            CONFIG.maxSpeedReached = currentSpeed
        end
        
        CONFIG.flyTime = tick() - STATE.startTime
        
        if CONFIG.trailEnabled and currentSpeed > 10 and STATE.frameCount % 2 == 0 then
            createTrailPart()
        end
        
        if CONFIG.fallProtectionEnabled then
            if currentPos.Y < CONFIG.minHeight then
                HumanoidRootPart.CFrame = CFrame.new(currentPos.X, CONFIG.minHeight + 10, currentPos.Z)
            end
            if currentPos.Y > CONFIG.maxHeight then
                HumanoidRootPart.CFrame = CFrame.new(currentPos.X, CONFIG.maxHeight - 10, currentPos.Z)
            end
        end
    end)
    
    task.spawn(function()
        while STATE.isFlying do
            task.wait(0.1)
            if not STATE.isFlying then break end
            
            TimeLabel.Text = "⏱ " .. formatTime(CONFIG.flyTime)
            DistanceLabel.Text = "📏 " .. formatNumber(CONFIG.totalDistance)
            MaxSpeedLabel.Text = "🏆 " .. formatNumber(CONFIG.maxSpeedReached)
            SpeedNowLabel.Text = "ТЕКУЩАЯ СКОРОСТЬ: " .. formatNumber(STATE.velocity.Magnitude)
        end
    end)
end

function stopFly()
    if not STATE.isFlying then return end
    
    STATE.isFlying = false
    STATE.isMoving = false
    STATE.velocity = Vector3.new(0, 0, 0)
    STATE.targetVelocity = Vector3.new(0, 0, 0)
    
    if STATE.flyConnection then
        STATE.flyConnection:Disconnect()
        STATE.flyConnection = nil
    end
    
    if Humanoid and Humanoid.Parent then
        Humanoid.PlatformStand = false
    end
    
    if HumanoidRootPart and HumanoidRootPart.Parent then
        HumanoidRootPart.Velocity = Vector3.new(0, 0, 0)
    end
    
    FlyButton.Text = "🛫 ВЗЛЕТЕТЬ"
    FlyButton.BackgroundColor3 = CONFIG.warningColor
    FlyButton.BorderColor3 = CONFIG.warningColor
    StatusValue.Text = "НЕ АКТИВЕН"
    StatusValue.TextColor3 = CONFIG.warningColor
    SpeedNowLabel.Text = "ТЕКУЩАЯ СКОРОСТЬ: 0"
    
    showNotification("🛬 ПОЛЁТ ОСТАНОВЛЕН", CONFIG.warningColor)
    
    STATE.forward = false
    STATE.backward = false
    STATE.left = false
    STATE.right = false
    STATE.up = false
    STATE.down = false
    STATE.boost = false
end

local function updateSpeedSlider(value)
    value = clamp(value, CONFIG.minSpeed, CONFIG.maxSpeed)
    CONFIG.flySpeed = value
    SpeedValue.Text = tostring(round(value))
    
    local percent = (value - CONFIG.minSpeed) / (CONFIG.maxSpeed - CONFIG.minSpeed)
    SpeedFill.Size = UDim2.new(percent, 0, 1, 0)
    SpeedKnob.Position = UDim2.new(percent, -16, 0, -10)
end

local sliderDragging = false

SpeedKnob.MouseButton1Down:Connect(function()
    sliderDragging = true
end)

SpeedKnob.MouseButton1Up:Connect(function()
    sliderDragging = false
end)

SpeedKnob.MouseLeave:Connect(function()
    sliderDragging = false
end)

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch then
        local pos = input.Position
        local knobPos = SpeedKnob.AbsolutePosition
        local knobSize = SpeedKnob.AbsoluteSize
        if pos.X >= knobPos.X - 30 and pos.X <= knobPos.X + knobSize.X + 30 and
           pos.Y >= knobPos.Y - 30 and pos.Y <= knobPos.Y + knobSize.Y + 30 then
            sliderDragging = true
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch then
        sliderDragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if sliderDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local pos = input.Position
        local trackPos = SpeedTrack.AbsolutePosition
        local trackSize = SpeedTrack.AbsoluteSize
        local percent = clamp((pos.X - trackPos.X) / trackSize.X, 0, 1)
        local value = CONFIG.minSpeed + percent * (CONFIG.maxSpeed - CONFIG.minSpeed)
        updateSpeedSlider(value)
    end
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if not STATE.isFlying then return end
    
    if input.KeyCode == Enum.KeyCode.W then
        STATE.forward = true
    elseif input.KeyCode == Enum.KeyCode.S then
        STATE.backward = true
    elseif input.KeyCode == Enum.KeyCode.A then
        STATE.left = true
    elseif input.KeyCode == Enum.KeyCode.D then
        STATE.right = true
    elseif input.KeyCode == Enum.KeyCode.Space then
        STATE.up = true
    elseif input.KeyCode == Enum.KeyCode.LeftShift then
        STATE.down = true
    elseif input.KeyCode == Enum.KeyCode.LeftControl then
        STATE.boost = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if not STATE.isFlying then return end
    
    if input.KeyCode == Enum.KeyCode.W then
        STATE.forward = false
    elseif input.KeyCode == Enum.KeyCode.S then
        STATE.backward = false
    elseif input.KeyCode == Enum.KeyCode.A then
        STATE.left = false
    elseif input.KeyCode == Enum.KeyCode.D then
        STATE.right = false
    elseif input.KeyCode == Enum.KeyCode.Space then
        STATE.up = false
    elseif input.KeyCode == Enum.KeyCode.LeftShift then
        STATE.down = false
    elseif input.KeyCode == Enum.KeyCode.LeftControl then
        STATE.boost = false
    end
end)

UserInputService.TouchStarted:Connect(function(touch, gameProcessed)
    if gameProcessed then return end
    if not STATE.isFlying then return end
    if not CONFIG.touchControlEnabled then return end
    
    local screenSize = Camera.ViewportSize
    local touchX = touch.Position.X
    
    if touchX < screenSize.X / 2 then
        STATE.up = true
        STATE.down = false
    else
        STATE.down = true
        STATE.up = false
    end
end)

UserInputService.TouchEnded:Connect(function(touch, gameProcessed)
    if gameProcessed then return end
    if not STATE.isFlying then return end
    
    STATE.up = false
    STATE.down = false
end)

FlyButton.MouseButton1Click:Connect(function()
    if STATE.isFlying then
        stopFly()
    else
        startFly()
    end
end)

TrailToggle.MouseButton1Click:Connect(function()
    CONFIG.trailEnabled = not CONFIG.trailEnabled
    if CONFIG.trailEnabled then
        TrailToggle.Text = "✨ СЛЕДЫ ВКЛ"
        TrailToggle.BorderColor3 = CONFIG.accentColor
        TrailToggle.BackgroundColor3 = CONFIG.accentColor
    else
        TrailToggle.Text = "✨ СЛЕДЫ ВЫКЛ"
        TrailToggle.BorderColor3 = CONFIG.warningColor
        TrailToggle.BackgroundColor3 = CONFIG.warningColor
        for _, part in pairs(STATE.currentTrailParts) do
            if part and part.Parent then
                part:Destroy()
            end
        end
        STATE.currentTrailParts = {}
    end
end)

GlowToggle.MouseButton1Click:Connect(function()
    CONFIG.glowEnabled = not CONFIG.glowEnabled
    if CONFIG.glowEnabled then
        GlowToggle.Text = "💡 СВЕЧЕНИЕ ВКЛ"
        GlowToggle.BorderColor3 = CONFIG.accentColor
        GlowToggle.BackgroundColor3 = CONFIG.accentColor
    else
        GlowToggle.Text = "💡 СВЕЧЕНИЕ ВЫКЛ"
        GlowToggle.BorderColor3 = CONFIG.warningColor
        GlowToggle.BackgroundColor3 = CONFIG.warningColor
    end
end)

ParticlesToggle.MouseButton1Click:Connect(function()
    CONFIG.particlesEnabled = not CONFIG.particlesEnabled
    if CONFIG.particlesEnabled then
        ParticlesToggle.Text = "✨ ЧАСТИЦЫ ВКЛ"
        ParticlesToggle.BorderColor3 = CONFIG.accentColor
        ParticlesToggle.BackgroundColor3 = CONFIG.accentColor
    else
        ParticlesToggle.Text = "✨ ЧАСТИЦЫ ВЫКЛ"
        ParticlesToggle.BorderColor3 = CONFIG.warningColor
        ParticlesToggle.BackgroundColor3 = CONFIG.warningColor
    end
end)

SoundToggle.MouseButton1Click:Connect(function()
    CONFIG.soundEnabled = not CONFIG.soundEnabled
    if CONFIG.soundEnabled then
        SoundToggle.Text = "🔊 ЗВУК ВКЛ"
        SoundToggle.BorderColor3 = CONFIG.accentColor
        SoundToggle.BackgroundColor3 = CONFIG.accentColor
    else
        SoundToggle.Text = "🔇 ЗВУК ВЫКЛ"
        SoundToggle.BorderColor3 = CONFIG.warningColor
        SoundToggle.BackgroundColor3 = CONFIG.warningColor
    end
end)

ResetButton.MouseButton1Click:Connect(function()
    if STATE.isFlying then
        stopFly()
    end
    updateSpeedSlider(50)
    CONFIG.totalDistance = 0
    CONFIG.flyTime = 0
    CONFIG.maxSpeedReached = 0
    TimeLabel.Text = "⏱ 00:00"
    DistanceLabel.Text = "📏 0"
    MaxSpeedLabel.Text = "🏆 0"
    showNotification("↺ ВСЁ СБРОШЕНО", CONFIG.themeColor)
end)

CloseButton.MouseButton1Click:Connect(function()
    if STATE.isFlying then
        stopFly()
    end
    ScreenGui:Destroy()
end)

local panelOpen = true

local function togglePanel()
    panelOpen = not panelOpen
    if panelOpen then
        ToggleButton.Text = "−"
        ClosedButton.Visible = false
        MainPanel.Visible = true
        local cx, cy = MainPanel.Position.X.Offset, MainPanel.Position.Y.Offset
        for i = 1, 10 do
            local s = i / 10
            MainPanel.Size = UDim2.new(0, 420 * s, 0, 480 * s)
            MainPanel.Position = UDim2.new(0, cx + 210 * (1 - s), 0, cy + 240 * (1 - s))
            MainPanel.BackgroundTransparency = 0.05 * (1 - s) + 0.05
            task.wait(0.012)
        end
        MainPanel.Size = UDim2.new(0, 420, 0, 480)
        MainPanel.Position = UDim2.new(0, cx, 0, cy)
        MainPanel.BackgroundTransparency = 0.05
    else
        ToggleButton.Text = "+"
        local cx, cy = MainPanel.Position.X.Offset, MainPanel.Position.Y.Offset
        for i = 10, 1, -1 do
            local s = i / 10
            MainPanel.Size = UDim2.new(0, 420 * s, 0, 480 * s)
            MainPanel.Position = UDim2.new(0, cx + 210 * (1 - s), 0, cy + 240 * (1 - s))
            MainPanel.BackgroundTransparency = 1 - s * 0.95
            task.wait(0.012)
        end
        MainPanel.Size = UDim2.new(0, 0, 0, 0)
        MainPanel.Visible = false
        ClosedButton.Visible = true
    end
end

ToggleButton.MouseButton1Click:Connect(togglePanel)
ClosedButton.MouseButton1Click:Connect(togglePanel)

local panelDragging = false
local panelDragOffset = Vector2.new(0, 0)

local function startPanelDrag(input)
    if input.UserInputType ~= Enum.UserInputType.Touch and input.UserInputType ~= Enum.UserInputType.MouseButton1 then
        return
    end
    local pos = input.Position
    local panelPos = MainPanel.AbsolutePosition
    local panelSize = MainPanel.AbsoluteSize
    
    if pos.X >= panelPos.X and pos.X <= panelPos.X + panelSize.X and
       pos.Y >= panelPos.Y and pos.Y <= panelPos.Y + 55 then
        panelDragging = true
        panelDragOffset = Vector2.new(pos.X - panelPos.X, pos.Y - panelPos.Y)
    end
end

local function movePanelDrag(input)
    if not panelDragging then return end
    if input.UserInputType ~= Enum.UserInputType.Touch and input.UserInputType ~= Enum.UserInputType.MouseMovement then
        return
    end
    
    local pos = input.Position
    local screenSize = Camera.ViewportSize
    
    local newX = clamp(pos.X - panelDragOffset.X, 0, screenSize.X - MainPanel.AbsoluteSize.X)
    local newY = clamp(pos.Y - panelDragOffset.Y, 0, screenSize.Y - MainPanel.AbsoluteSize.Y)
    
    MainPanel.Position = UDim2.new(0, newX, 0, newY)
end

local function stopPanelDrag()
    panelDragging = false
end

UserInputService.InputBegan:Connect(startPanelDrag)
UserInputService.InputChanged:Connect(movePanelDrag)
UserInputService.InputEnded:Connect(stopPanelDrag)

LocalPlayer.CharacterAdded:Connect(function(newCharacter)
    if STATE.isFlying then
        stopFly()
    end
    
    Character = newCharacter
    HumanoidRootPart = newCharacter:WaitForChild("HumanoidRootPart")
    Humanoid = newCharacter:WaitForChild("Humanoid")
    
    task.wait(0.5)
    
    if Humanoid and Humanoid.Parent then
        Humanoid.PlatformStand = false
    end
    
    showNotification("👤 Персонаж перезагружен", CONFIG.themeColor)
end)

updateSpeedSlider(50)

if not Character or not Character.Parent then
    Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
    Humanoid = Character:WaitForChild("Humanoid")
end

task.wait(0.5)
showNotification("🚀 ULTIMATE FLY v7.1 ЗАГРУЖЕН", CONFIG.accentColor)

print("╔══════════════════════════════════════════════════╗")
print("║   🚀 ULTIMATE FLY SYSTEM v7.1 ЗАГРУЖЕН          ║")
print("╠══════════════════════════════════════════════════╣")
print("║   ✅ Полёт с сенсорным управлением              ║")
print("║   ✅ Регулировка скорости (10-1000)             ║")
print("║   ✅ Следы, свечение, частицы                   ║")
print("║   ✅ Статистика полёта                          ║")
print("║   ✅ Защита от падения                          ║")
print("╚══════════════════════════════════════════════════╝")
print("")
print("🎮 УПРАВЛЕНИЕ:")
print("   📱 Телефон: левая половина = вверх, правая = вниз")
print("   🖥 ПК: WASD + Space (вверх) + Shift (вниз) + Ctrl (буст)")
print("")

-- ═══════════════════════════════════════════════════════════════
--   КОНЕЦ СКРИПТА
-- ═══════════════════════════════════════════════════════════════
