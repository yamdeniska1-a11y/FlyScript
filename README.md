local Players          = game:GetService("Players")
local TweenService     = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService       = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")

local CFG = {
    WIN_W = 400,
    WIN_H = 300,
    CORNER = 0,
}

local C = {
    WIN_BG        = Color3.fromRGB(10, 10, 16),
    WIN_STROKE    = Color3.fromRGB(55, 55, 75),
    TITLE_TEXT    = Color3.fromRGB(130, 130, 170),
    CLOSE_RED     = Color3.fromRGB(210, 70, 70),
    CLOSE_BG      = Color3.fromRGB(35, 14, 14),
    BTN_IDLE      = Color3.fromRGB(40, 40, 55),
    BTN_HOVER     = Color3.fromRGB(60, 60, 80),
    BTN_ACTIVE    = Color3.fromRGB(50, 150, 50),
    BTN_INACTIVE  = Color3.fromRGB(150, 50, 50),
    TEXT_COLOR    = Color3.fromRGB(220, 220, 240),
    SLIDER_TRACK  = Color3.fromRGB(40, 40, 55),
    SLIDER_THUMB  = Color3.fromRGB(80, 80, 120),
}

local function New(class, props, children)
    local inst = Instance.new(class)
    for k, v in pairs(props or {}) do inst[k] = v end
    for _, child in ipairs(children or {}) do child.Parent = inst end
    return inst
end

local function Stroke(color, thick, transparency)
    return New("UIStroke", {
        Color = color,
        Thickness = thick or 1,
        Transparency = transparency or 0,
        ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
    })
end

local ScreenGui = New("ScreenGui", {
    Name = "KeeperFlyTool",
    ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    IgnoreGuiInset = true,
    Parent = PlayerGui,
})

local Window = New("Frame", {
    Name = "Window",
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.new(0.5, 0, 0.5, 0),
    Size = UDim2.new(0, CFG.WIN_W, 0, CFG.WIN_H),
    BackgroundColor3 = C.WIN_BG,
    BorderSizePixel = 0,
    ZIndex = 2,
    ClipsDescendants = true,
    Parent = ScreenGui,
})
Stroke(C.WIN_STROKE, 1, 0.3).Parent = Window

local TitleBar = New("Frame", {
    Size = UDim2.new(1, 0, 0, 36),
    BackgroundColor3 = Color3.fromRGB(16, 16, 24),
    BackgroundTransparency = 0.4,
    BorderSizePixel = 0,
    ZIndex = 4,
    Parent = Window,
})

New("TextLabel", {
    Position = UDim2.new(0, 12, 0, 0),
    Size = UDim2.new(1, -60, 1, 0),
    BackgroundTransparency = 1,
    Font = Enum.Font.GothamMedium,
    Text = "fly control",
    TextColor3 = C.TITLE_TEXT,
    TextSize = 13,
    TextXAlignment = Enum.TextXAlignment.Left,
    ZIndex = 5,
    Parent = TitleBar,
})

local CloseBtn = New("TextButton", {
    AnchorPoint = Vector2.new(1, 0.5),
    Position = UDim2.new(1, -12, 0.5, 0),
    Size = UDim2.new(0, 24, 0, 24),
    BackgroundColor3 = C.CLOSE_BG,
    BackgroundTransparency = 0.3,
    BorderSizePixel = 0,
    Text = "×",
    TextColor3 = C.CLOSE_RED,
    TextSize = 18,
    Font = Enum.Font.GothamBold,
    ZIndex = 6,
    Parent = TitleBar,
})
Stroke(Color3.fromRGB(160, 60, 60), 1).Parent = CloseBtn

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)

local Content = New("Frame", {
    Position = UDim2.new(0, 16, 0, 46),
    Size = UDim2.new(1, -32, 1, -58),
    BackgroundTransparency = 1,
    ZIndex = 3,
    Parent = Window,
})

local FlySystem = {
    flying = false,
    speed = 50,
    hotkey = Enum.KeyCode.LeftAlt,
    hotkeyName = "LeftAlt",
    waitingForKey = false,
}

local speedLabel = New("TextLabel", {
    Size = UDim2.new(0.5, 0, 0, 20),
    Position = UDim2.new(0, 0, 0, 10),
    BackgroundTransparency = 1,
    Font = Enum.Font.Gotham,
    Text = "Speed",
    TextColor3 = Color3.fromRGB(160, 160, 180),
    TextSize = 13,
    TextXAlignment = Enum.TextXAlignment.Left,
    Parent = Content,
})

local speedBox = New("TextBox", {
    Size = UDim2.new(0.15, 0, 0, 24),
    Position = UDim2.new(0.7, 0, 0, 8),
    BackgroundColor3 = Color3.fromRGB(40, 40, 55),
    BorderSizePixel = 0,
    Font = Enum.Font.Gotham,
    Text = tostring(FlySystem.speed),
    TextColor3 = C.TEXT_COLOR,
    TextSize = 14,
    TextXAlignment = Enum.TextXAlignment.Center,
    ClearTextOnFocus = false,
    Parent = Content,
})
Stroke(Color3.fromRGB(255,255,255), 0.5, 0.4).Parent = speedBox

local track = New("Frame", {
    Size = UDim2.new(0.75, 0, 0, 6),
    Position = UDim2.new(0, 0, 0, 48),
    BackgroundColor3 = C.SLIDER_TRACK,
    BorderSizePixel = 0,
    Parent = Content,
})

local thumb = New("Frame", {
    Size = UDim2.new(0, 14, 0, 14),
    Position = UDim2.new(0, 0, 0.5, -7),
    BackgroundColor3 = C.SLIDER_THUMB,
    BorderSizePixel = 0,
    Parent = track,
})
Stroke(Color3.fromRGB(150, 150, 200), 1, 0.3).Parent = thumb

New("TextLabel", {
    Size = UDim2.new(0.1, 0, 0, 16),
    Position = UDim2.new(0, 0, 0, 60),
    BackgroundTransparency = 1,
    Font = Enum.Font.Gotham,
    Text = "1",
    TextColor3 = Color3.fromRGB(120, 120, 140),
    TextSize = 11,
    TextXAlignment = Enum.TextXAlignment.Left,
    Parent = Content,
})
New("TextLabel", {
    Size = UDim2.new(0.1, 0, 0, 16),
    Position = UDim2.new(0.75, 0, 0, 60),
    BackgroundTransparency = 1,
    Font = Enum.Font.Gotham,
    Text = "100",
    TextColor3 = Color3.fromRGB(120, 120, 140),
    TextSize = 11,
    TextXAlignment = Enum.TextXAlignment.Left,
    Parent = Content,
})

local hotkeyLabel = New("TextLabel", {
    Size = UDim2.new(0.5, 0, 0, 20),
    Position = UDim2.new(0, 0, 0, 85),
    BackgroundTransparency = 1,
    Font = Enum.Font.Gotham,
    Text = "Hotkey",
    TextColor3 = Color3.fromRGB(160, 160, 180),
    TextSize = 13,
    TextXAlignment = Enum.TextXAlignment.Left,
    Parent = Content,
})

local hotkeyBtn = New("TextButton", {
    Size = UDim2.new(0.3, 0, 0, 26),
    Position = UDim2.new(0.5, 0, 0, 82),
    BackgroundColor3 = Color3.fromRGB(40, 40, 55),
    BorderSizePixel = 0,
    Font = Enum.Font.Gotham,
    Text = FlySystem.hotkeyName,
    TextColor3 = C.TEXT_COLOR,
    TextSize = 13,
    Parent = Content,
})
Stroke(Color3.fromRGB(255,255,255), 0.5, 0.4).Parent = hotkeyBtn

local toggleBtn = New("TextButton", {
    Size = UDim2.new(0.5, 0, 0, 34),
    Position = UDim2.new(0.25, 0, 0, 130),
    BackgroundColor3 = C.BTN_INACTIVE,
    Text = "Fly: OFF",
    TextColor3 = Color3.fromRGB(255, 255, 255),
    TextSize = 15,
    BorderSizePixel = 0,
    Parent = Content,
})
Stroke(Color3.fromRGB(255,255,255), 0.8, 0.2).Parent = toggleBtn

local function updateSliderUI(speed)
    local minSpeed, maxSpeed = 1, 100
    local ratio = math.clamp((speed - minSpeed) / (maxSpeed - minSpeed), 0, 1)
    local trackWidth = track.AbsoluteSize.X
    if trackWidth > 0 then
        thumb.Position = UDim2.new(0, ratio * (trackWidth - 14), 0.5, -7)
    else
        task.wait(0.05)
        trackWidth = track.AbsoluteSize.X
        if trackWidth > 0 then thumb.Position = UDim2.new(0, ratio * (trackWidth - 14), 0.5, -7) end
    end
    speedBox.Text = tostring(math.round(speed))
end

local function updateToggleUI()
    toggleBtn.BackgroundColor3 = FlySystem.flying and C.BTN_ACTIVE or C.BTN_INACTIVE
    toggleBtn.Text = FlySystem.flying and "Fly: ON" or "Fly: OFF"
end

local dragging = false
local dragStartX, dragStartPos

thumb.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStartX = input.Position.X
        dragStartPos = thumb.Position.X.Offset
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local trackWidth = track.AbsoluteSize.X
        if trackWidth <= 0 then return end
        local delta = input.Position.X - dragStartX
        local newPos = math.clamp(dragStartPos + delta, 0, trackWidth - 14)
        thumb.Position = UDim2.new(0, newPos, 0.5, -7)
        local ratio = newPos / (trackWidth - 14)
        local speed = math.round(1 + ratio * 99)
        FlySystem.speed = speed
        speedBox.Text = tostring(speed)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

speedBox.FocusLost:Connect(function(enterPressed)
    local val = tonumber(speedBox.Text)
    if val and val > 0 then
        FlySystem.speed = val
        if val >= 1 and val <= 100 then
            updateSliderUI(val)
        elseif val > 100 then
            thumb.Position = UDim2.new(0, track.AbsoluteSize.X - 14, 0.5, -7)
            speedBox.Text = tostring(val)
        else
            thumb.Position = UDim2.new(0, 0, 0.5, -7)
            speedBox.Text = tostring(val)
        end
    else
        speedBox.Text = tostring(FlySystem.speed)
    end
end)

hotkeyBtn.MouseButton1Click:Connect(function()
    if FlySystem.waitingForKey then return end
    FlySystem.waitingForKey = true
    hotkeyBtn.Text = "Press any key..."
    hotkeyBtn.BackgroundColor3 = Color3.fromRGB(80, 50, 50)

    local conn
    conn = UserInputService.InputBegan:Connect(function(input, processed)
        if not FlySystem.waitingForKey then
            if conn then conn:Disconnect() end
            return
        end
        if input.UserInputType == Enum.UserInputType.Keyboard then
            local key = input.KeyCode
            if key then
                FlySystem.hotkey = key
                FlySystem.hotkeyName = tostring(key):gsub("Enum.KeyCode.", "")
                hotkeyBtn.Text = FlySystem.hotkeyName
                hotkeyBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
                FlySystem.waitingForKey = false
                if conn then conn:Disconnect() end
            end
        end
    end)

    task.delay(10, function()
        if FlySystem.waitingForKey then
            FlySystem.waitingForKey = false
            hotkeyBtn.Text = FlySystem.hotkeyName
            hotkeyBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
            if conn then conn:Disconnect() end
        end
    end)
end)

local function toggleFly()
    FlySystem.flying = not FlySystem.flying
    updateToggleUI()
end

toggleBtn.MouseButton1Click:Connect(toggleFly)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.UserInputType == Enum.UserInputType.Keyboard then
        if FlySystem.waitingForKey then return end
        if input.KeyCode == FlySystem.hotkey then
            toggleFly()
        end
    end
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.RightAlt then
        if ScreenGui then ScreenGui:Destroy() end
    end
end)

RunService.RenderStepped:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChild("Humanoid")
    if not hrp or not hum then return end

    local lv = hrp:FindFirstChild("FlyLinearVelocity")
    local align = hrp:FindFirstChild("FlyAlignOrientation")
    local att = hrp:FindFirstChild("FlyAttachment")

    if FlySystem.flying then
        if not lv then
            att = Instance.new("Attachment")
            att.Name = "FlyAttachment"
            att.Parent = hrp
            lv = Instance.new("LinearVelocity")
            lv.Name = "FlyLinearVelocity"
            lv.MaxForce = 100000
            lv.VectorVelocity = Vector3.new(0,0,0)
            lv.Attachment0 = att
            lv.Parent = hrp
            align = Instance.new("AlignOrientation")
            align.Name = "FlyAlignOrientation"
            align.MaxTorque = 100000
            align.Attachment0 = att
            align.Mode = Enum.OrientationAlignmentMode.OneAttachment
            align.Parent = hrp
        end

        hum.PlatformStand = true
        local cam = workspace.CurrentCamera
        if cam then
            local dir = Vector3.new(0,0,0)
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir += cam.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir -= cam.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir -= cam.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir += cam.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then dir += Vector3.new(0,-1,0) end
            lv.VectorVelocity = dir * FlySystem.speed
            align.CFrame = cam.CFrame
        end
    else
        if hum then hum.PlatformStand = false end
        if lv then lv:Destroy() end
        if align then align:Destroy() end
        if att then att:Destroy() end
    end
end)

updateSliderUI(FlySystem.speed)
updateToggleUI()

print("Fly Tool loaded (minimal version) — only flight, no extra buttons.")
