local Players          = game:GetService("Players")
local TweenService     = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local ContextActionService = game:GetService("ContextActionService")
local RunService       = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")

local CFG = {
    WIN_W         = 760,
    WIN_H         = 460,
    SIDEBAR_W     = 240,
    BTN_H         = 44,
    BTN_GAP       = 7,
    BTN_PADX      = 8,
    BTN_PADY_TOP  = 6,
    SCROLL_BAR_W  = 5,
    TOTAL_BTNS    = 15,
    CORNER_WIN    = 0,
    CORNER_BTN    = 0,
    CORNER_PANEL  = 0,

    T_FAST   = 0.14,
    T_MED    = 0.26,
    T_OPEN   = 0.38,
    T_ULTRA  = 0.06,
    T_SPRING = 0.45,

    CLOSE_X_OFFSET = -1,
    CLOSE_Y_OFFSET = -1,
}

local C = {
    WIN_BG        = Color3.fromRGB(10, 10, 16),
    WIN_STROKE    = Color3.fromRGB(55, 55, 75),
    SIDEBAR_BG    = Color3.fromRGB(14, 14, 22),
    BTN_IDLE      = Color3.fromRGB(26, 26, 38),
    BTN_HOVER     = Color3.fromRGB(46, 46, 66),
    BTN_ACTIVE    = Color3.fromRGB(58, 58, 88),
    BTN_PRESS     = Color3.fromRGB(18, 18, 28),
    BTN_STROKE    = Color3.fromRGB(255, 255, 255),
    BTN_STR_HOV   = Color3.fromRGB(255, 255, 255),
    BTN_STR_ACT   = Color3.fromRGB(255, 255, 255),
    BTN_TEXT      = Color3.fromRGB(180, 180, 200),
    BTN_TEXT_ACT  = Color3.fromRGB(230, 230, 245),
    PANEL_BG      = Color3.fromRGB(14, 14, 22),
    PANEL_STROKE  = Color3.fromRGB(255, 255, 255),
    TITLE_TEXT    = Color3.fromRGB(130, 130, 170),
    CLOSE_RED     = Color3.fromRGB(210, 70, 70),
    CLOSE_BG      = Color3.fromRGB(35, 14, 14),
    SCROLL_TRACK  = Color3.fromRGB(30, 30, 44),
    SCROLL_THUMB  = Color3.fromRGB(80, 80, 120),
    SCROLL_HOV    = Color3.fromRGB(120, 120, 170),
}

local function tw(obj, t, style, dir, props)
    local info = TweenInfo.new(t, style or Enum.EasingStyle.Quart, dir or Enum.EasingDirection.Out)
    local tween = TweenService:Create(obj, info, props)
    tween:Play()
    return tween
end

local function twFast(obj, props)   return tw(obj, CFG.T_FAST,  Enum.EasingStyle.Quart, Enum.EasingDirection.Out, props) end
local function twMed(obj, props)    return tw(obj, CFG.T_MED,   Enum.EasingStyle.Quart, Enum.EasingDirection.Out, props) end
local function twUltra(obj, props)  return tw(obj, CFG.T_ULTRA, Enum.EasingStyle.Quad,  Enum.EasingDirection.Out, props) end
local function twSpring(obj, props) return tw(obj, CFG.T_SPRING, Enum.EasingStyle.Elastic, Enum.EasingDirection.Out, props) end

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

local SCROLL_BIND = "DisableScroll"
ContextActionService:BindAction(SCROLL_BIND, function() return Enum.ContextActionResult.Sink end, false, Enum.UserInputType.MouseWheel)

local ScreenGui = New("ScreenGui", {
    Name           = "KeeperMultiTool",
    ResetOnSpawn   = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    IgnoreGuiInset = true,
    Parent         = PlayerGui,
})

local Window = New("Frame", {
    Name             = "Window",
    AnchorPoint      = Vector2.new(0.5, 0.5),
    Position         = UDim2.new(0.5, 0, 0.5, 0),
    Size             = UDim2.new(0, CFG.WIN_W, 0, CFG.WIN_H),
    BackgroundColor3 = C.WIN_BG,
    BorderSizePixel  = 0,
    ZIndex           = 2,
    ClipsDescendants = true,
    Parent           = ScreenGui,
})

local WinGlass = New("Frame", {
    Name             = "WinGlass",
    Size             = UDim2.new(1, 0, 1, 0),
    BackgroundColor3 = Color3.fromRGB(255, 255, 255),
    BackgroundTransparency = 0.92,
    BorderSizePixel  = 0,
    ZIndex           = 2,
    Parent           = Window,
})

local TitleBar = New("Frame", {
    Name                  = "TitleBar",
    Size                  = UDim2.new(1, 0, 0, 42),
    BackgroundColor3      = Color3.fromRGB(16, 16, 24),
    BackgroundTransparency = 0.4,
    BorderSizePixel       = 0,
    ZIndex                = 4,
    Parent                = Window,
})

New("Frame", {
    Name             = "Sep",
    AnchorPoint      = Vector2.new(0, 1),
    Position         = UDim2.new(0, 0, 1, 0),
    Size             = UDim2.new(1, 0, 0, 1),
    BackgroundColor3 = C.WIN_STROKE,
    BackgroundTransparency = 0.5,
    BorderSizePixel  = 0,
    ZIndex           = 5,
    Parent           = TitleBar,
})

local TitleLabel = New("TextLabel", {
    Name                  = "TitleLabel",
    Position              = UDim2.new(0, 16, 0, 0),
    Size                  = UDim2.new(1, -60, 1, 0),
    BackgroundTransparency = 1,
    Font                  = Enum.Font.GothamMedium,
    Text                  = "keeper | multi-tool",
    TextColor3            = C.TITLE_TEXT,
    TextSize              = 13,
    TextXAlignment        = Enum.TextXAlignment.Left,
    ZIndex                = 5,
    Parent                = TitleBar,
})

local CloseBtnContainer = New("Frame", {
    Name             = "CloseBtnContainer",
    AnchorPoint      = Vector2.new(1, 0.5),
    Position         = UDim2.new(1, -14, 0.5, 0),
    Size             = UDim2.new(0, 26, 0, 26),
    BackgroundColor3 = C.CLOSE_BG,
    BackgroundTransparency = 0.3,
    BorderSizePixel  = 0,
    ZIndex           = 6,
    Parent           = TitleBar,
})

local CloseGlass = New("Frame", {
    Name             = "Glass",
    Size             = UDim2.new(1, 0, 1, 0),
    BackgroundColor3 = Color3.fromRGB(255, 220, 220),
    BackgroundTransparency = 0.88,
    BorderSizePixel  = 0,
    ZIndex           = 7,
    ClipsDescendants = true,
    Parent           = CloseBtnContainer,
})

local CloseStroke = Stroke(Color3.fromRGB(160, 60, 60), 1)
CloseStroke.Parent = CloseBtnContainer

local CloseBtn = New("TextButton", {
    Name                  = "CloseBtn",
    Size                  = UDim2.new(1, 0, 1, 0),
    BackgroundTransparency = 1,
    Text                  = "",
    ZIndex                = 9,
    Parent                = CloseBtnContainer,
})

local CloseLabel = New("TextLabel", {
    Name                  = "CloseLabel",
    AnchorPoint           = Vector2.new(0.5, 0.5),
    Position              = UDim2.new(0.5, CFG.CLOSE_X_OFFSET, 0.5, CFG.CLOSE_Y_OFFSET),
    Size                  = UDim2.new(1, 0, 1, 0),
    BackgroundTransparency = 1,
    Font                  = Enum.Font.GothamBold,
    Text                  = "×",
    TextColor3            = C.CLOSE_RED,
    TextSize              = 18,
    ZIndex                = 10,
    Parent                = CloseBtnContainer,
})

CloseBtn.MouseEnter:Connect(function()
    twFast(CloseBtnContainer, {BackgroundTransparency = 0.05, Size = UDim2.new(0, 30, 0, 30)})
    twFast(CloseStroke, {Color = Color3.fromRGB(220, 80, 80), Thickness = 1.5})
    twFast(CloseLabel, {TextColor3 = Color3.fromRGB(240, 120, 120), TextSize = 20, Position = UDim2.new(0.5, 0, 0.5, CFG.CLOSE_Y_OFFSET)})
    twFast(CloseGlass, {BackgroundTransparency = 0.75})
end)
CloseBtn.MouseLeave:Connect(function()
    twFast(CloseBtnContainer, {BackgroundTransparency = 0.3, Size = UDim2.new(0, 26, 0, 26)})
    twFast(CloseStroke, {Color = Color3.fromRGB(160, 60, 60), Thickness = 1})
    twFast(CloseLabel, {TextColor3 = C.CLOSE_RED, TextSize = 18, Position = UDim2.new(0.5, CFG.CLOSE_X_OFFSET, 0.5, CFG.CLOSE_Y_OFFSET)})
    twFast(CloseGlass, {BackgroundTransparency = 0.88})
end)
CloseBtn.MouseButton1Down:Connect(function()
    twUltra(CloseBtnContainer, {BackgroundTransparency = 0.7, Size = UDim2.new(0, 22, 0, 22)})
    CloseBtnContainer.Position = UDim2.new(1, -12, 0.5, 0)
end)
CloseBtn.MouseButton1Up:Connect(function()
    twFast(CloseBtnContainer, {Size = UDim2.new(0, 26, 0, 26), Position = UDim2.new(1, -14, 0.5, 0)})
end)

local isVisible = true

local function doClose()
    isVisible = false
    ContextActionService:UnbindAction(SCROLL_BIND)
    twMed(Window, {Size = UDim2.new(0, CFG.WIN_W, 0, 0), BackgroundTransparency = 1})
    task.delay(0.35, function() if ScreenGui then ScreenGui:Destroy() end end)
end

local function doToggle()
    if isVisible then
        isVisible = false
        ContextActionService:UnbindAction(SCROLL_BIND)
        twMed(Window, {Size = UDim2.new(0, CFG.WIN_W * 0.92, 0, CFG.WIN_H * 0.92), BackgroundTransparency = 0.98})
        Window.Visible = false
    else
        isVisible = true
        Window.Visible = true
        ContextActionService:BindAction(SCROLL_BIND, function() return Enum.ContextActionResult.Sink end, false, Enum.UserInputType.MouseWheel)
        twMed(Window, {Size = UDim2.new(0, CFG.WIN_W, 0, CFG.WIN_H), BackgroundTransparency = 0})
    end
end

CloseBtn.MouseButton1Click:Connect(doClose)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.RightAlt then doToggle() end
end)

local ContentArea = New("Frame", {
    Name                  = "ContentArea",
    Position              = UDim2.new(0, 12, 0, 46),
    Size                  = UDim2.new(1, -24, 1, -58),
    BackgroundTransparency = 1,
    ZIndex                = 3,
    Parent                = Window,
})

local SidebarContainer = New("Frame", {
    Name             = "SidebarContainer",
    Position         = UDim2.new(0, 0, 0, 0),
    Size             = UDim2.new(0, CFG.SIDEBAR_W, 1, 0),
    BackgroundColor3 = C.SIDEBAR_BG,
    BackgroundTransparency = 0.3,
    BorderSizePixel  = 0,
    ZIndex           = 3,
    ClipsDescendants = true,
    Parent           = ContentArea,
})
local SidebarStroke = Stroke(C.PANEL_STROKE, 1, 0.5)
SidebarStroke.Parent = SidebarContainer

local ScrollClip = New("Frame", {
    Name             = "ScrollClip",
    Position         = UDim2.new(0, CFG.BTN_PADX, 0, CFG.BTN_PADY_TOP),
    Size             = UDim2.new(1, -(CFG.BTN_PADX * 2 + CFG.SCROLL_BAR_W + 6), 1, -(CFG.BTN_PADY_TOP * 2)),
    BackgroundTransparency = 1,
    ClipsDescendants = true,
    ZIndex           = 4,
    Parent           = SidebarContainer,
})

local TotalBtnH  = CFG.TOTAL_BTNS * (CFG.BTN_H + CFG.BTN_GAP) - CFG.BTN_GAP
local ScrollHolder = New("Frame", {
    Name             = "ScrollHolder",
    Position         = UDim2.new(0, 0, 0, 0),
    Size             = UDim2.new(1, 0, 0, TotalBtnH),
    BackgroundTransparency = 1,
    ZIndex           = 4,
    Parent           = ScrollClip,
})

local ScrollTrack = New("Frame", {
    Name             = "ScrollTrack",
    AnchorPoint      = Vector2.new(1, 0),
    Position         = UDim2.new(1, 0, 0, CFG.BTN_PADY_TOP),
    Size             = UDim2.new(0, CFG.SCROLL_BAR_W, 1, -(CFG.BTN_PADY_TOP * 2)),
    BackgroundColor3 = C.SCROLL_TRACK,
    BackgroundTransparency = 0.4,
    BorderSizePixel  = 0,
    ZIndex           = 6,
    Parent           = SidebarContainer,
})

local ScrollThumb = New("Frame", {
    Name             = "ScrollThumb",
    Position         = UDim2.new(0, 0, 0, 0),
    Size             = UDim2.new(1, 0, 0, 40),
    BackgroundColor3 = C.SCROLL_THUMB,
    BackgroundTransparency = 0,
    BorderSizePixel  = 0,
    ZIndex           = 7,
    Parent           = ScrollTrack,
})

local scrollOffset    = 0
local scrollMaxOffset = 0
local isScrollDragging = false
local scrollDragStartY = 0
local scrollDragStartOffset = 0

local function getVisibleH() return ScrollClip.AbsoluteSize.Y end
local function updateScrollMax()
    local vis = getVisibleH()
    scrollMaxOffset = math.max(0, TotalBtnH - vis)
end
local function clampScroll(v) return math.clamp(v, 0, scrollMaxOffset) end

local function applyScroll(offset, animate)
    scrollOffset = clampScroll(offset)
    local targetY = -scrollOffset
    if animate then twFast(ScrollHolder, {Position = UDim2.new(0, 0, 0, targetY)})
    else ScrollHolder.Position = UDim2.new(0, 0, 0, targetY) end

    local vis = getVisibleH()
    local trackH = ScrollTrack.AbsoluteSize.Y
    local ratio  = vis / math.max(TotalBtnH, 1)
    local thumbH = math.max(24, math.floor(trackH * ratio))
    local thumbY = scrollMaxOffset > 0 and math.floor((scrollOffset / scrollMaxOffset) * (trackH - thumbH)) or 0
    if animate then
        twFast(ScrollThumb, {Size = UDim2.new(1, 0, 0, thumbH), Position = UDim2.new(0, 0, 0, thumbY)})
    else
        ScrollThumb.Size = UDim2.new(1, 0, 0, thumbH)
        ScrollThumb.Position = UDim2.new(0, 0, 0, thumbY)
    end
end

SidebarContainer.InputChanged:Connect(function(input, processed)
    if processed then return end
    if input.UserInputType == Enum.UserInputType.MouseWheel then
        applyScroll(scrollOffset - input.Position.Z * 60, true)
    end
end)

ScrollThumb.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isScrollDragging = true
        scrollDragStartY = input.Position.Y
        scrollDragStartOffset = scrollOffset
        twFast(ScrollThumb, {BackgroundColor3 = C.SCROLL_HOV})
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if isScrollDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position.Y - scrollDragStartY
        local trackH = ScrollTrack.AbsoluteSize.Y
        local ratio = TotalBtnH / math.max(trackH, 1)
        applyScroll(scrollDragStartOffset + delta * ratio, false)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        if isScrollDragging then
            isScrollDragging = false
            twFast(ScrollThumb, {BackgroundColor3 = C.SCROLL_THUMB})
        end
    end
end)

ScrollThumb.MouseEnter:Connect(function()
    if not isScrollDragging then twFast(ScrollThumb, {BackgroundColor3 = C.SCROLL_HOV}) end
end)
ScrollThumb.MouseLeave:Connect(function()
    if not isScrollDragging then twFast(ScrollThumb, {BackgroundColor3 = C.SCROLL_THUMB}) end
end)

local PanelX = CFG.SIDEBAR_W + 12
local MainPanel = New("Frame", {
    Name             = "MainPanel",
    Position         = UDim2.new(0, PanelX, 0, 0),
    Size             = UDim2.new(1, -PanelX, 1, 0),
    BackgroundColor3 = C.PANEL_BG,
    BackgroundTransparency = 0.3,
    BorderSizePixel  = 0,
    ZIndex           = 3,
    ClipsDescendants = true,
    Parent           = ContentArea,
})
local MainPanelStroke = Stroke(C.PANEL_STROKE, 1, 0.5)
MainPanelStroke.Parent = MainPanel

local PanelContent = New("TextLabel", {
    Name                  = "PanelContent",
    AnchorPoint           = Vector2.new(0.5, 0.5),
    Position              = UDim2.new(0.5, 0, 0.5, 0),
    Size                  = UDim2.new(0.85, 0, 0.8, 0),
    BackgroundTransparency = 1,
    Font                  = Enum.Font.Gotham,
    Text                  = "",
    TextColor3            = Color3.fromRGB(150, 150, 180),
    TextSize              = 13,
    RichText              = true,
    TextWrapped           = true,
    ZIndex                = 6,
    Parent                = MainPanel,
})

local DynamicContainer = New("Frame", {
    Name             = "DynamicContainer",
    Size             = UDim2.new(1, 0, 1, 0),
    BackgroundTransparency = 1,
    ZIndex           = 7,
    Visible          = false,
    Parent           = MainPanel,
})

local FlySystem = {}
FlySystem.flying = false
FlySystem.speed = 50
FlySystem.hotkey = Enum.KeyCode.LeftAlt
FlySystem.hotkeyName = "LeftAlt"
FlySystem.ui = {}
FlySystem.waitingForKey = false
local mouseConnections = {}

function FlySystem.createPanel(parent)
    for _, child in ipairs(parent:GetChildren()) do child:Destroy() end
    for _, conn in ipairs(mouseConnections) do if conn then conn:Disconnect() end end
    mouseConnections = {}

    local function buildUI()
        local title = Instance.new("TextLabel")
        title.Size = UDim2.new(1, -20, 0, 30)
        title.Position = UDim2.new(0, 10, 0, 10)
        title.BackgroundTransparency = 1
        title.Font = Enum.Font.GothamBold
        title.Text = "Fly Settings"
        title.TextColor3 = Color3.fromRGB(200, 200, 220)
        title.TextSize = 18
        title.TextXAlignment = Enum.TextXAlignment.Left
        title.Parent = parent

        local speedLabel = Instance.new("TextLabel")
        speedLabel.Size = UDim2.new(0.5, 0, 0, 20)
        speedLabel.Position = UDim2.new(0, 10, 0, 50)
        speedLabel.BackgroundTransparency = 1
        speedLabel.Font = Enum.Font.Gotham
        speedLabel.Text = "Fly Speed"
        speedLabel.TextColor3 = Color3.fromRGB(160, 160, 180)
        speedLabel.TextSize = 13
        speedLabel.TextXAlignment = Enum.TextXAlignment.Left
        speedLabel.Parent = parent

        local speedBox = Instance.new("TextBox")
        speedBox.Size = UDim2.new(0.15, 0, 0, 24)
        speedBox.Position = UDim2.new(0.72, 0, 0, 48)
        speedBox.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
        speedBox.BorderSizePixel = 0
        speedBox.Font = Enum.Font.Gotham
        speedBox.Text = tostring(FlySystem.speed)
        speedBox.TextColor3 = Color3.fromRGB(220, 220, 240)
        speedBox.TextSize = 14
        speedBox.TextXAlignment = Enum.TextXAlignment.Center
        speedBox.ClearTextOnFocus = false
        speedBox.Parent = parent
        local boxStroke = Instance.new("UIStroke")
        boxStroke.Color = Color3.fromRGB(255,255,255)
        boxStroke.Thickness = 0.5
        boxStroke.Transparency = 0.4
        boxStroke.Parent = speedBox

        
        speedBox.Focused:Connect(function()
            speedBox.Text = ""
            if FlySystem.ui.updateSlider then
                FlySystem.ui.updateSlider(1)   -- minimum = 1
            end
        end)

        local track = Instance.new("Frame")
        track.Size = UDim2.new(0.6, 0, 0, 6)
        track.Position = UDim2.new(0, 10, 0, 85)
        track.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
        track.BorderSizePixel = 0
        track.Parent = parent

        local thumb = Instance.new("Frame")
        thumb.Size = UDim2.new(0, 14, 0, 14)
        thumb.Position = UDim2.new(0, 0, 0.5, -7)
        thumb.BackgroundColor3 = Color3.fromRGB(80, 80, 120)
        thumb.BorderSizePixel = 0
        thumb.Parent = track
        local thumbStroke = Instance.new("UIStroke")
        thumbStroke.Color = Color3.fromRGB(150, 150, 200)
        thumbStroke.Thickness = 1
        thumbStroke.Transparency = 0.3
        thumbStroke.Parent = thumb

        local minLabel = Instance.new("TextLabel")
        minLabel.Size = UDim2.new(0.1, 0, 0, 16)
        minLabel.Position = UDim2.new(0, 10, 0, 95)
        minLabel.BackgroundTransparency = 1
        minLabel.Font = Enum.Font.Gotham
        minLabel.Text = "Min"
        minLabel.TextColor3 = Color3.fromRGB(120, 120, 140)
        minLabel.TextSize = 11
        minLabel.TextXAlignment = Enum.TextXAlignment.Left
        minLabel.Parent = parent

        local maxLabel = Instance.new("TextLabel")
        maxLabel.Size = UDim2.new(0.1, 0, 0, 16)
        maxLabel.Position = UDim2.new(0.6, 0, 0, 95)
        maxLabel.BackgroundTransparency = 1
        maxLabel.Font = Enum.Font.Gotham
        maxLabel.Text = "Max"
        maxLabel.TextColor3 = Color3.fromRGB(120, 120, 140)
        maxLabel.TextSize = 11
        maxLabel.TextXAlignment = Enum.TextXAlignment.Left
        maxLabel.Parent = parent

        local hotkeyLabel = Instance.new("TextLabel")
        hotkeyLabel.Size = UDim2.new(0.5, 0, 0, 20)
        hotkeyLabel.Position = UDim2.new(0, 10, 0, 125)
        hotkeyLabel.BackgroundTransparency = 1
        hotkeyLabel.Font = Enum.Font.Gotham
        hotkeyLabel.Text = "Hotkey"
        hotkeyLabel.TextColor3 = Color3.fromRGB(160, 160, 180)
        hotkeyLabel.TextSize = 13
        hotkeyLabel.TextXAlignment = Enum.TextXAlignment.Left
        hotkeyLabel.Parent = parent

        local hotkeyBtn = Instance.new("TextButton")
        hotkeyBtn.Size = UDim2.new(0.25, 0, 0, 26)
        hotkeyBtn.Position = UDim2.new(0.5, 0, 0, 122)
        hotkeyBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 55)
        hotkeyBtn.BorderSizePixel = 0
        hotkeyBtn.Font = Enum.Font.Gotham
        hotkeyBtn.Text = FlySystem.hotkeyName
        hotkeyBtn.TextColor3 = Color3.fromRGB(220, 220, 240)
        hotkeyBtn.TextSize = 13
        hotkeyBtn.Parent = parent
        local hbStroke = Instance.new("UIStroke")
        hbStroke.Color = Color3.fromRGB(255,255,255)
        hbStroke.Thickness = 0.5
        hbStroke.Transparency = 0.4
        hbStroke.Parent = hotkeyBtn

        local toggleBtn = Instance.new("TextButton")
        toggleBtn.Size = UDim2.new(0.4, 0, 0, 30)
        toggleBtn.Position = UDim2.new(0.3, 0, 0, 160)
        toggleBtn.BackgroundColor3 = FlySystem.flying and Color3.fromRGB(50, 150, 50) or Color3.fromRGB(150, 50, 50)
        toggleBtn.Text = FlySystem.flying and "Fly: ON" or "Fly: OFF"
        toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        toggleBtn.TextSize = 14
        toggleBtn.BorderSizePixel = 0
        toggleBtn.Parent = parent
        local togStroke = Instance.new("UIStroke")
        togStroke.Color = Color3.fromRGB(255, 255, 255)
        togStroke.Thickness = 0.8
        togStroke.Transparency = 0.3
        togStroke.Parent = toggleBtn

        
        FlySystem.ui.track = track
        FlySystem.ui.thumb = thumb
        FlySystem.ui.speedBox = speedBox
        FlySystem.ui.hotkeyBtn = hotkeyBtn
        FlySystem.ui.toggleBtn = toggleBtn
        FlySystem.ui.toggleStroke = togStroke
        FlySystem.ui.panel = parent

        
        local function updateSlider(speed)
            local minSpeed = 1; local maxSpeed = 100
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
        FlySystem.ui.updateSlider = updateSlider

        
        local function setSliderPosition(speed)
            local minSpeed = 1; local maxSpeed = 100
            local ratio = math.clamp((speed - minSpeed) / (maxSpeed - minSpeed), 0, 1)
            local trackWidth = track.AbsoluteSize.X
            if trackWidth > 0 then
                thumb.Position = UDim2.new(0, ratio * (trackWidth - 14), 0.5, -7)
            else
                task.wait(0.05)
                trackWidth = track.AbsoluteSize.X
                if trackWidth > 0 then thumb.Position = UDim2.new(0, ratio * (trackWidth - 14), 0.5, -7) end
            end
            
        end
        FlySystem.ui.setSliderPosition = setSliderPosition

        
        updateSlider(FlySystem.speed)

        
        local dragging = false
        local dragStartX, dragStartPos
        thumb.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = true
                dragStartX = input.Position.X
                dragStartPos = thumb.Position.X.Offset
            end
        end)

        local mouseConn = UserInputService.InputChanged:Connect(function(input)
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
                print("[FLY] Speed changed to:", speed)
            end
        end)
        table.insert(mouseConnections, mouseConn)

        local mouseEndConn = UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = false
            end
        end)
        table.insert(mouseConnections, mouseEndConn)

        
        speedBox.FocusLost:Connect(function(enterPressed)
            local val = tonumber(speedBox.Text)
            if val and val > 0 then
                FlySystem.speed = val
                
                if val >= 1 and val <= 100 then
                    updateSlider(val)
                elseif val > 100 then
                    setSliderPosition(100)
                elseif val < 1 then
                    setSliderPosition(1)
                end
                
                speedBox.Text = tostring(val)
                print("[FLY] Speed set via input (unlimited):", val)
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
                        print("[FLY] Hotkey bound to:", FlySystem.hotkeyName)
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

        
        toggleBtn.MouseButton1Click:Connect(function()
            print("[FLY] Toggle button clicked")
            FlySystem.toggleFly()
        end)

        print("Fly panel built successfully")
    end

    buildUI()
end

function FlySystem.toggleFly()
    FlySystem.flying = not FlySystem.flying
    FlySystem.updateUI()
    print("[FLY] Toggled to:", FlySystem.flying)
end

function FlySystem.updateUI()
    if FlySystem.ui.toggleBtn then
        FlySystem.ui.toggleBtn.BackgroundColor3 = FlySystem.flying and Color3.fromRGB(50, 150, 50) or Color3.fromRGB(150, 50, 50)
        FlySystem.ui.toggleBtn.Text = FlySystem.flying and "Fly: ON" or "Fly: OFF"
    end
end

FlySystem.createPanel(DynamicContainer)
DynamicContainer.Visible = false

local flyRenderConn = RunService.RenderStepped:Connect(function()
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

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.UserInputType == Enum.UserInputType.Keyboard then
        if FlySystem.waitingForKey then return end
        if input.KeyCode == FlySystem.hotkey then
            FlySystem.toggleFly()
        end
    end
end)

local LABELS = {
    "Main ",
    "Fly ",
    "NoClip ",
    "", "", "", "", "", "", "", "", "", "", "", ""
}

local buttons   = {}
local activeBtn = nil

for i = 1, CFG.TOTAL_BTNS do
    local yOff = (i - 1) * (CFG.BTN_H + CFG.BTN_GAP)

    local Btn = New("TextButton", {
        Name             = "Btn_" .. i,
        Position         = UDim2.new(0, 1, 0, yOff + 1),
        Size             = UDim2.new(1, -2, 0, CFG.BTN_H - 2),
        BackgroundColor3 = C.BTN_IDLE,
        BackgroundTransparency = 0.2,
        BorderSizePixel  = 0,
        AutoButtonColor  = false,
        Font             = Enum.Font.Gotham,
        Text             = LABELS[i] or "",
        TextColor3       = C.BTN_TEXT,
        TextSize         = 12,
        TextXAlignment   = Enum.TextXAlignment.Center,
        TextTruncate     = Enum.TextTruncate.None,
        ZIndex           = 5,
        Parent           = ScrollHolder,
    })

    local BtnStroke = Stroke(C.BTN_STROKE, 0.8, 0.2)
    BtnStroke.Parent = Btn

    Btn.MouseEnter:Connect(function()
        if Btn ~= activeBtn then
            twFast(Btn, {BackgroundColor3 = C.BTN_HOVER, BackgroundTransparency = 0.1})
            twFast(BtnStroke, {Color = C.BTN_STR_HOV, Thickness = 1.0, Transparency = 0.1})
            twFast(Btn, {TextColor3 = C.BTN_TEXT_ACT})
        end
    end)
    Btn.MouseLeave:Connect(function()
        if Btn ~= activeBtn then
            twFast(Btn, {BackgroundColor3 = C.BTN_IDLE, BackgroundTransparency = 0.2})
            twFast(BtnStroke, {Color = C.BTN_STROKE, Thickness = 0.8, Transparency = 0.2})
            twFast(Btn, {TextColor3 = C.BTN_TEXT})
        end
    end)
    Btn.MouseButton1Down:Connect(function()
        twUltra(Btn, {BackgroundColor3 = C.BTN_PRESS, BackgroundTransparency = 0.1})
    end)
    Btn.MouseButton1Up:Connect(function()
        if Btn == activeBtn then
            twFast(Btn, {BackgroundColor3 = C.BTN_ACTIVE, BackgroundTransparency = 0.1})
        else
            twFast(Btn, {BackgroundColor3 = C.BTN_IDLE, BackgroundTransparency = 0.2})
        end
    end)

    Btn.MouseButton1Click:Connect(function()
        if activeBtn and activeBtn ~= Btn then
            local prev = activeBtn
            local prevStroke = prev:FindFirstChildOfClass("UIStroke")
            twFast(prev, {BackgroundColor3 = C.BTN_IDLE, BackgroundTransparency = 0.2})
            twFast(prev, {TextColor3 = C.BTN_TEXT})
            if prevStroke then twFast(prevStroke, {Color = C.BTN_STROKE, Thickness = 0.8, Transparency = 0.2}) end
        end

        activeBtn = Btn
        twFast(Btn, {BackgroundColor3 = C.BTN_ACTIVE, BackgroundTransparency = 0.1})
        twFast(BtnStroke, {Color = C.BTN_STR_ACT, Thickness = 1.0, Transparency = 0.05})
        twFast(Btn, {TextColor3 = C.BTN_TEXT_ACT})

        if i == 2 then  -- Fly button
            PanelContent.Visible = false
            DynamicContainer.Visible = true
            FlySystem.updateUI()
            if FlySystem.ui.updateSlider then
                FlySystem.ui.updateSlider(FlySystem.speed)
            end
        else
            PanelContent.Visible = true
            DynamicContainer.Visible = false
            twUltra(PanelContent, {TextTransparency = 1})
            task.delay(0.09, function()
                PanelContent.Text = ""
                twMed(PanelContent, {TextTransparency = 0})
            end)
        end
    end)

    buttons[i] = Btn
end

task.defer(function()
    updateScrollMax()
    applyScroll(0, false)
end)

Window.Size = UDim2.new(0, CFG.WIN_W * 0.86, 0, CFG.WIN_H * 0.86)
Window.BackgroundTransparency = 0.95
twSpring(Window, {Size = UDim2.new(0, CFG.WIN_W, 0, CFG.WIN_H), BackgroundTransparency = 0})

for i, btn in ipairs(buttons) do
    btn.BackgroundTransparency = 1
    btn.TextTransparency = 1
    task.delay(0.04 + i * 0.025, function()
        if btn and btn.Parent then
            twSpring(btn, {BackgroundTransparency = 0.2, TextTransparency = 0})
        end
    end)
end

print("Keeper Multi-Tool v4.9 loaded — Unlimited speed input, English UI, exact value display.")
