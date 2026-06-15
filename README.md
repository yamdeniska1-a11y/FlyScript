local ok, err = pcall(function()

local TweenService     = game:GetService("TweenService")
local Players          = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService       = game:GetService("RunService")

local player    = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid  = character:WaitForChild("Humanoid")
local rootPart  = character:WaitForChild("HumanoidRootPart")

local FlySpeed        = 10
local FlyKey          = Enum.KeyCode.G
local NoClipOn        = false
local FlyOn           = false
local listeningForKey = false
local flyBodyVel, flyBodyGyro
local menuVisible     = true

print("[LG] Script started!")

local BG     = Color3.fromRGB(12, 12, 22)
local CARD   = Color3.fromRGB(25, 25, 42)
local INPUT  = Color3.fromRGB(16, 16, 30)
local ACCENT = Color3.fromRGB(110, 70, 240)
local ACCENT2= Color3.fromRGB(150, 110, 255)
local STROKE = Color3.fromRGB(100, 65, 210)
local WHITE  = Color3.new(1,1,1)
local GRAY   = Color3.fromRGB(140,140,180)

local function tw(o, props, t)
    TweenService:Create(o, TweenInfo.new(t or .2, Enum.EasingStyle.Quad), props):Play()
end
local function corner(p, r)
    local c = Instance.new("UICorner", p)
    c.CornerRadius = UDim.new(0, r or 12)
end
local function addStroke(p, col, th)
    local s = Instance.new("UIStroke", p)
    s.Color = col or STROKE
    s.Thickness = th or 1.5
end

local gui = Instance.new("ScreenGui")
gui.Name           = "LiquidGlassMenu"
gui.ResetOnSpawn   = false
gui.IgnoreGuiInset = true
gui.DisplayOrder   = 999
gui.Parent         = player:WaitForChild("PlayerGui")

print("[LG] GUI created, parent =", gui.Parent)

local win = Instance.new("Frame")
win.Size             = UDim2.new(0, 420, 0, 296)
win.Position         = UDim2.new(0.5, -210, 0.5, -148)
win.BackgroundColor3 = BG
win.BorderSizePixel  = 0
win.Parent           = gui
corner(win, 16)
addStroke(win, STROKE, 2)

print("[LG] Window created, size =", win.Size)

local function toggleMenu()
    menuVisible = not menuVisible
    win.Visible = menuVisible
    print("[LG] Menu toggled:", menuVisible)
end

UserInputService.InputBegan:Connect(function(inp, gpe)
    if inp.KeyCode == Enum.KeyCode.RightAlt then
        toggleMenu()
    end
end)

local xBtn = Instance.new("TextButton")
xBtn.Size             = UDim2.new(0, 26, 0, 26)
xBtn.Position         = UDim2.new(1, -32, 0, 6)
xBtn.BackgroundColor3 = Color3.fromRGB(50, 20, 100)
xBtn.TextColor3       = WHITE
xBtn.Text             = "x"
xBtn.TextSize         = 14
xBtn.Font             = Enum.Font.GothamBold
xBtn.BorderSizePixel  = 0
xBtn.ZIndex           = 20
xBtn.Parent           = win
corner(xBtn, 7)
xBtn.MouseButton1Click:Connect(function() toggleMenu() end)

local _drag, _ds, _dp
win.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        _drag=true _ds=i.Position _dp=win.Position
    end
end)
gui.InputChanged:Connect(function(i)
    if _drag and i.UserInputType == Enum.UserInputType.MouseMovement then
        local d = i.Position - _ds
        win.Position = UDim2.new(_dp.X.Scale, _dp.X.Offset+d.X, _dp.Y.Scale, _dp.Y.Offset+d.Y)
    end
end)
gui.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then _drag=false end
end)

local function makeCard(y, h)
    local f = Instance.new("Frame")
    f.Size             = UDim2.new(0, 388, 0, h)
    f.Position         = UDim2.new(0, 16, 0, y)
    f.BackgroundColor3 = CARD
    f.BorderSizePixel  = 0
    f.Parent           = win
    corner(f, 12)
    addStroke(f, STROKE, 1.5)
    return f
end

local function makeTextLabel(parent, text, x, y, w, h, size, color, bold)
    local l = Instance.new("TextLabel")
    l.Size                   = UDim2.new(0, w, 0, h)
    l.Position               = UDim2.new(0, x, 0, y)
    l.BackgroundTransparency = 1
    l.TextColor3             = color or GRAY
    l.Text                   = text
    l.TextSize               = size or 13
    l.Font                   = bold and Enum.Font.GothamBold or Enum.Font.Gotham
    l.TextXAlignment         = Enum.TextXAlignment.Left
    l.Parent                 = parent
    return l
end

local c1 = makeCard(10, 76)
makeTextLabel(c1, "Fly Speed", 12, 6, 200, 16, 12, GRAY)
print("[LG] Card1 created at", c1.Position)

local sBox = Instance.new("TextBox")
sBox.Size             = UDim2.new(0, 364, 0, 34)
sBox.Position         = UDim2.new(0, 12, 0, 26)
sBox.BackgroundColor3 = INPUT
sBox.TextColor3       = WHITE
sBox.Text             = "10.0"
sBox.TextSize         = 15
sBox.Font             = Enum.Font.GothamBold
sBox.ClearTextOnFocus = false
sBox.BorderSizePixel  = 0
sBox.TextXAlignment   = Enum.TextXAlignment.Left
sBox.Parent           = c1
corner(sBox, 9)
addStroke(sBox, STROKE)
local p1 = Instance.new("UIPadding", sBox)
p1.PaddingLeft = UDim.new(0, 12)

sBox.FocusLost:Connect(function()
    local v = tonumber(sBox.Text)
    FlySpeed = v and math.clamp(v, 1, 500) or FlySpeed
    sBox.Text = string.format("%.1f", FlySpeed)
end)

local c2 = makeCard(96, 76)
makeTextLabel(c2, "Fly Keyboard", 12, 6, 200, 16, 12, GRAY)
print("[LG] Card2 created at", c2.Position)

local kRow = Instance.new("Frame")
kRow.Size             = UDim2.new(0, 364, 0, 34)
kRow.Position         = UDim2.new(0, 12, 0, 26)
kRow.BackgroundColor3 = INPUT
kRow.BorderSizePixel  = 0
kRow.Parent           = c2
corner(kRow, 9)
addStroke(kRow, STROKE)

local kLbl = makeTextLabel(kRow, FlyKey.Name, 12, 0, 240, 34, 15, WHITE, true)

local cBtn = Instance.new("TextButton")
cBtn.Size             = UDim2.new(0, 80, 0, 26)
cBtn.Position         = UDim2.new(1, -88, 0.5, -13)
cBtn.BackgroundColor3 = ACCENT
cBtn.TextColor3       = WHITE
cBtn.Text             = "Change"
cBtn.TextSize         = 13
cBtn.Font             = Enum.Font.GothamBold
cBtn.BorderSizePixel  = 0
cBtn.Parent           = kRow
corner(cBtn, 7)

cBtn.MouseButton1Click:Connect(function()
    if listeningForKey then return end
    listeningForKey = true
    kLbl.Text = "..."
    tw(cBtn, {BackgroundColor3=ACCENT2})
    local conn
    conn = UserInputService.InputBegan:Connect(function(inp, gpe)
        if gpe then return end
        if inp.UserInputType == Enum.UserInputType.Keyboard
           and inp.KeyCode ~= Enum.KeyCode.RightAlt then
            FlyKey = inp.KeyCode
            kLbl.Text = FlyKey.Name
            listeningForKey = false
            tw(cBtn, {BackgroundColor3=ACCENT})
            conn:Disconnect()
        end
    end)
end)
cBtn.MouseEnter:Connect(function() if not listeningForKey then tw(cBtn,{BackgroundColor3=ACCENT2}) end end)
cBtn.MouseLeave:Connect(function() if not listeningForKey then tw(cBtn,{BackgroundColor3=ACCENT}) end end)

local c3 = makeCard(182, 76)
makeTextLabel(c3, "NoClip", 12, 6, 200, 16, 12, GRAY)
print("[LG] Card3 created at", c3.Position)

local track = Instance.new("Frame")
track.Size             = UDim2.new(0, 364, 0, 34)
track.Position         = UDim2.new(0, 12, 0, 26)
track.BackgroundColor3 = Color3.fromRGB(28, 28, 48)
track.BorderSizePixel  = 0
track.Parent           = c3
corner(track, 9)
addStroke(track, STROKE)

makeTextLabel(track, "Disabled", 14, 0, 150, 34, 13, GRAY)

local enL = Instance.new("TextLabel")
enL.Size               = UDim2.new(0, 150, 0, 34)
enL.Position           = UDim2.new(1, -164, 0, 0)
enL.BackgroundTransparency = 1
enL.TextColor3         = GRAY
enL.Text               = "Enabled"
enL.TextSize           = 13
enL.Font               = Enum.Font.Gotham
enL.TextXAlignment     = Enum.TextXAlignment.Right
enL.Parent             = track

local thumb = Instance.new("Frame")
thumb.Size             = UDim2.new(0, 30, 0, 26)
thumb.Position         = UDim2.new(0, 3, 0.5, -13)
thumb.BackgroundColor3 = ACCENT
thumb.BorderSizePixel  = 0
thumb.Parent           = track
corner(thumb, 7)

local function setNC(state)
    NoClipOn = state
    tw(thumb, {
        Position         = state and UDim2.new(1,-33,0.5,-13) or UDim2.new(0,3,0.5,-13),
        BackgroundColor3 = state and WHITE or ACCENT
    })
    tw(track, {BackgroundColor3 = state and ACCENT or Color3.fromRGB(28,28,48)})
    print("[LG] NoClip:", state)
end

local tBtn = Instance.new("TextButton")
tBtn.Size               = UDim2.new(1,0,1,0)
tBtn.BackgroundTransparency = 1
tBtn.Text               = ""
tBtn.ZIndex             = 5
tBtn.Parent             = track
tBtn.MouseButton1Click:Connect(function() setNC(not NoClipOn) end)

local function startFly()
    FlyOn = true
    local cam = workspace.CurrentCamera
    flyBodyVel = Instance.new("BodyVelocity", rootPart)
    flyBodyVel.MaxForce = Vector3.new(1e5,1e5,1e5)
    flyBodyVel.Velocity = Vector3.zero
    flyBodyGyro = Instance.new("BodyGyro", rootPart)
    flyBodyGyro.MaxTorque = Vector3.new(1e5,1e5,1e5)
    flyBodyGyro.D = 100
    humanoid.PlatformStand = true
    RunService:BindToRenderStep("Fly", Enum.RenderPriority.Input.Value, function()
        if not FlyOn then return end
        local mv = Vector3.zero
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then mv+=cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then mv-=cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then mv-=cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then mv+=cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space)     then mv+=Vector3.yAxis end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then mv-=Vector3.yAxis end
        flyBodyVel.Velocity = mv.Magnitude>0 and mv.Unit*FlySpeed or Vector3.zero
        flyBodyGyro.CFrame  = cam.CFrame
    end)
    print("[LG] Fly started")
end

local function stopFly()
    FlyOn = false
    RunService:UnbindFromRenderStep("Fly")
    if flyBodyVel  then flyBodyVel:Destroy()  flyBodyVel=nil end
    if flyBodyGyro then flyBodyGyro:Destroy() flyBodyGyro=nil end
    humanoid.PlatformStand = false
    print("[LG] Fly stopped")
end

UserInputService.InputBegan:Connect(function(inp, gpe)
    if gpe or listeningForKey then return end
    if inp.KeyCode == Enum.KeyCode.RightAlt then return end
    if inp.KeyCode == FlyKey then
        if FlyOn then stopFly() else startFly() end
    end
end)

RunService.Stepped:Connect(function()
    if not NoClipOn then return end
    for _, p in ipairs(character:GetDescendants()) do
        if p:IsA("BasePart") then p.CanCollide = false end
    end
end)

player.CharacterAdded:Connect(function(char)
    character = char
    humanoid  = char:WaitForChild("Humanoid")
    rootPart  = char:WaitForChild("HumanoidRootPart")
    FlyOn     = false
    setNC(false)
end)

print("[LG] All done! Press RightAlt to toggle menu.")

end) -- pcall end

if not ok then
    print("[LG] ERROR:", err)
end
