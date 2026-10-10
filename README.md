
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local old = playerGui:FindFirstChild("HunterSpeedGui")
if old then old:Destroy() end

local gui = Instance.new("ScreenGui")
gui.Name = "HunterSpeedGui"
gui.IgnoreGuiInset = true
gui.ResetOnSpawn = false
gui.DisplayOrder = 100
gui.Parent = playerGui

local BLUE = Color3.fromRGB(80, 210, 255)
local DARK = Color3.fromRGB(7, 10, 24)

local function corner(obj, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius)
    c.Parent = obj
end

local function stroke(obj, color, thickness)
    local s = Instance.new("UIStroke")
    s.Color = color
    s.Thickness = thickness
    s.Parent = obj
    return s
end

local function label(parent, name, text, size, pos, textSize, color)
    local t = Instance.new("TextLabel")
    t.Name = name
    t.Size = size
    t.Position = pos
    t.BackgroundTransparency = 1
    t.Text = text
    t.TextSize = textSize
    t.TextColor3 = color
    t.Font = Enum.Font.GothamBold
    t.Parent = parent
    return t
end

local avatarUrl = "rbxthumb://type=AvatarBust&id="
    .. tostring(player.UserId) .. "&w=420&h=420"

local frame = Instance.new("Frame")
frame.Name = "MainPanel"
frame.Size = UDim2.fromOffset(250, 270)
frame.Position = UDim2.new(0.5, -125, 0.5, -135)
frame.BackgroundColor3 = DARK
frame.BorderSizePixel = 0
frame.Visible = false
frame.ClipsDescendants = true
frame.Parent = gui
corner(frame, 16)
local frameStroke = stroke(frame, BLUE, 2)

local sky = Instance.new("Frame")
sky.Name = "Sky"
sky.Size = UDim2.new(1, -16, 0, 90)
sky.Position = UDim2.fromOffset(8, 8)
sky.BackgroundColor3 = Color3.fromRGB(20, 100, 210)
sky.BorderSizePixel = 0
sky.Parent = frame
corner(sky, 12)

local gradient = Instance.new("UIGradient")
gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(20, 85, 200)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(80, 210, 255))
})
gradient.Rotation = 35
gradient.Parent = sky

local clouds = label(
    sky, "Clouds", "☁     ☁",
    UDim2.fromScale(1, 1), UDim2.fromScale(0, 0),
    25, Color3.fromRGB(240, 248, 255)
)
clouds.TextTransparency = 0.2

local avatar = Instance.new("ImageLabel")
avatar.Name = "PlayerAvatar"
avatar.Size = UDim2.fromOffset(74, 74)
avatar.Position = UDim2.new(0.5, -37, 0, 40)
avatar.BackgroundTransparency = 1
avatar.Image = avatarUrl
avatar.ImageTransparency = 0
avatar.ScaleType = Enum.ScaleType.Crop
avatar.BorderSizePixel = 0
avatar.ZIndex = 5
avatar.Parent = frame
corner(avatar, 100)
stroke(avatar, Color3.fromRGB(120, 70, 255), 3)

label(frame, "PlayerName", player.Name,
    UDim2.new(1, -12, 0, 22), UDim2.fromOffset(6, 118),
    15, BLUE)

label(frame, "Credit", "HUNTER45XB",
    UDim2.new(1, -12, 0, 18), UDim2.fromOffset(6, 140),
    11, Color3.fromRGB(170, 180, 205))

local speed = 16
local speedLabel = label(frame, "SpeedLabel", "Speed: 16",
    UDim2.new(1, -12, 0, 22), UDim2.fromOffset(6, 160),
    15, Color3.new(1, 1, 1))

local function makeButton(name, text, y, height)
    local b = Instance.new("TextButton")
    b.Name = name
    b.Size = UDim2.new(1, -24, 0, height)
    b.Position = UDim2.fromOffset(12, y)
    b.BackgroundColor3 = Color3.fromRGB(20, 30, 55)
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Text = text
    b.TextSize = 14
    b.Font = Enum.Font.GothamBold
    b.BorderSizePixel = 0
    b.Parent = frame
    corner(b, 8)
    return b
end

local plus = makeButton("PlusButton", "+ Speed", 186, 24)
local minus = makeButton("MinusButton", "- Speed", 214, 24)
local reset = makeButton("ResetButton", "Reset Speed", 242, 23)

local open = Instance.new("TextButton")
open.Name = "OpenButton"
open.Size = UDim2.fromOffset(105, 34)
open.Position = frame.Position
open.BackgroundColor3 = Color3.new(0, 0, 0)
open.Text = "OPEN"
open.TextColor3 = Color3.new(1, 1, 1)
open.TextSize = 14
open.Font = Enum.Font.GothamBold
open.Visible = false
open.BorderSizePixel = 0
open.Parent = gui
corner(open, 10)
stroke(open, BLUE, 1)

local minimize = Instance.new("TextButton")
minimize.Name = "MinimizeButton"
minimize.Size = UDim2.fromOffset(28, 24)
minimize.Position = UDim2.new(1, -34, 0, 4)
minimize.BackgroundColor3 = Color3.new(0, 0, 0)
minimize.Text = "—"
minimize.TextColor3 = Color3.new(1, 1, 1)
minimize.TextSize = 16
minimize.Font = Enum.Font.GothamBold
minimize.BorderSizePixel = 0
minimize.Parent = frame
corner(minimize, 6)

local function getHumanoid()
    local character = player.Character
    return character and character:FindFirstChildOfClass("Humanoid")
end

local function updateSpeed()
    speedLabel.Text = "Speed: " .. speed
    local hum = getHumanoid()
    if hum then hum.WalkSpeed = speed end
end

plus.Activated:Connect(function()
    speed = math.clamp(speed + 4, 0, 200)
    updateSpeed()
end)

minus.Activated:Connect(function()
    speed = math.clamp(speed - 4, 0, 200)
    updateSpeed()
end)

reset.Activated:Connect(function()
    speed = 16
    updateSpeed()
end)

-- تصغير وفتح سينمائي ناعم
local animating = false
local panelSize = UDim2.fromOffset(250, 270)

minimize.Activated:Connect(function()
    if animating or not frame.Visible then return end
    animating = true

    local originalPosition = frame.Position
    local centerPosition = originalPosition + UDim2.fromOffset(125, 135)

    open.Position = centerPosition - UDim2.fromOffset(52.5, 17)
    open.Size = UDim2.fromOffset(0, 0)
    open.TextTransparency = 1
    open.Visible = true

    local shrinkTween = TweenService:Create(
        frame,
        TweenInfo.new(0.48, Enum.EasingStyle.Quint, Enum.EasingDirection.In),
        {
            Position = centerPosition,
            Size = UDim2.fromOffset(0, 0),
            BackgroundTransparency = 1
        }
    )

    local strokeTween = TweenService:Create(
        frameStroke,
        TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {Transparency = 1}
    )

    local buttonTween = TweenService:Create(
        open,
        TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {
            Size = UDim2.fromOffset(105, 34),
            TextTransparency = 0
        }
    )

    shrinkTween:Play()
    strokeTween:Play()

    task.delay(0.18, function()
        if open.Parent then
            buttonTween:Play()
        end
    end)

    shrinkTween.Completed:Wait()

    frame.Visible = false
    frame.Position = originalPosition
    frame.Size = panelSize
    frame.BackgroundTransparency = 0
    frameStroke.Transparency = 0

    animating = false
end)

open.Activated:Connect(function()
    if animating or not open.Visible then return end
    animating = true

    local openPosition = open.Position
    local panelPosition = openPosition - UDim2.fromOffset(72.5, 118)

    frame.Position = openPosition + UDim2.fromOffset(52.5, 17)
    frame.Size = UDim2.fromOffset(0, 0)
    frame.BackgroundTransparency = 1
    frameStroke.Transparency = 1
    frame.Visible = true

    local expandTween = TweenService:Create(
        frame,
        TweenInfo.new(0.55, Enum.EasingStyle.Quint, Enum.EasingDirection.Out),
        {
            Position = panelPosition,
            Size = panelSize,
            BackgroundTransparency = 0
        }
    )

    local strokeTween = TweenService:Create(
        frameStroke,
        TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {Transparency = 0}
    )

    local hideOpenTween = TweenService:Create(
        open,
        TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {
            Size = UDim2.fromOffset(0, 0),
            TextTransparency = 1
        }
    )

    expandTween:Play()
    strokeTween:Play()
    hideOpenTween:Play()

    expandTween.Completed:Wait()

    open.Visible = false
    open.Size = UDim2.fromOffset(105, 34)
    open.TextTransparency = 0

    animating = false
end)

-- سحب الواجهة
local function draggable(object, handle)
    local dragging = false
    local startPos
    local startInput
    local activeInput

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            startInput = input.Position
            startPos = object.Position
            activeInput = input

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                    activeInput = nil
                end
            end)
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if not dragging then return end
        if input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch then
            local delta = input.Position - startInput
            object.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
        end
    end)
end

draggable(frame, sky)
draggable(open, open)

-- شاشة البداية
local intro = Instance.new("Frame")
intro.Name = "Intro"
intro.Size = UDim2.fromScale(1, 1)
intro.BackgroundColor3 = DARK
intro.BackgroundTransparency = 1
intro.BorderSizePixel = 0
intro.ZIndex = 50
intro.Parent = gui

local portrait = Instance.new("ImageLabel")
portrait.Name = "IntroPortrait"
portrait.Size = UDim2.fromOffset(150, 150)
portrait.AnchorPoint = Vector2.new(0.5, 0.5)
portrait.Position = UDim2.new(0.5, 0, 0.42, 0)
portrait.BackgroundTransparency = 1
portrait.Image = avatarUrl
portrait.ImageTransparency = 0
portrait.ScaleType = Enum.ScaleType.Crop
portrait.BorderSizePixel = 0
portrait.ZIndex = 51
portrait.Parent = intro
corner(portrait, 100)
local ring = stroke(portrait, BLUE, 4)

local title = label(intro, "IntroTitle", "H U N T E R   S P E E D",
    UDim2.new(1, 0, 0, 40), UDim2.new(0, 0, 0.73, 0),
    28, BLUE)
title.ZIndex = 51
title.TextTransparency = 1

local subtitle = label(intro, "IntroSubtitle", "INITIALIZING...",
    UDim2.new(1, 0, 0, 26), UDim2.new(0, 0, 0.78, 0),
    17, Color3.new(1, 1, 1))
subtitle.ZIndex = 51
subtitle.TextTransparency = 1

TweenService:Create(intro, TweenInfo.new(0.6), {
    BackgroundTransparency = 0
}):Play()

portrait.Size = UDim2.fromOffset(110, 110)
TweenService:Create(portrait, TweenInfo.new(
    0.8, Enum.EasingStyle.Back, Enum.EasingDirection.Out
), {
    Size = UDim2.fromOffset(150, 150)
}):Play()

TweenService:Create(title, TweenInfo.new(0.8), {
    TextTransparency = 0
}):Play()

TweenService:Create(subtitle, TweenInfo.new(0.8), {
    TextTransparency = 0
}):Play()

local start = os.clock()
local rotationConnection

rotationConnection = RunService.RenderStepped:Connect(function()
    local elapsed = os.clock() - start
    if elapsed >= 4 then
        portrait.Rotation = 0
        rotationConnection:Disconnect()
    else
        portrait.Rotation = (elapsed * 35) % 360
    end
end)

task.wait(4)

frame.Visible = true
frame.BackgroundTransparency = 1
TweenService:Create(frame, TweenInfo.new(
    0.7, Enum.EasingStyle.Quart, Enum.EasingDirection.Out
), {
    BackgroundTransparency = 0
}):Play()

task.wait(0.7)

TweenService:Create(intro, TweenInfo.new(0.5), {
    BackgroundTransparency = 1
}):Play()

TweenService:Create(portrait, TweenInfo.new(0.5), {
    ImageTransparency = 1
}):Play()

TweenService:Create(title, TweenInfo.new(0.4), {
    TextTransparency = 1
}):Play()

TweenService:Create(subtitle, TweenInfo.new(0.4), {
    TextTransparency = 1
}):Play()

task.wait(0.6)
intro:Destroy()

updateSpeed()

player.CharacterAdded:Connect(function(character)
    local hum = character:WaitForChild("Humanoid", 10)
    if hum then hum.WalkSpeed = speed end
end)
# speed-script-
speed hub
