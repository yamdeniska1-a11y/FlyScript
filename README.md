local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "LiquidGlass_Menu"

local THEME = {
    Main = Color3.fromRGB(8, 8, 12),
    Accent = Color3.fromRGB(120, 80, 255),
    Glass = Color3.fromRGB(20, 20, 25)
}

local MainWindow = Instance.new("Frame", gui)
MainWindow.Size = UDim2.new(0, 450, 0, 400)
MainWindow.Position = UDim2.new(0.5, -225, 0.5, -200)
MainWindow.BackgroundColor3 = THEME.Main
MainWindow.BorderSizePixel = 0
Instance.new("UICorner", MainWindow).CornerRadius = UDim.new(0, 16)

local stroke = Instance.new("UIStroke", MainWindow)
stroke.Color = THEME.Accent
stroke.Thickness = 2
stroke.Transparency = 0.3

local overlay = Instance.new("Frame", MainWindow)
overlay.Size = UDim2.new(1, 0, 1, 0)
overlay.BackgroundColor3 = Color3.new(1, 1, 1)
overlay.BackgroundTransparency = 0.95
Instance.new("UICorner", overlay).CornerRadius = UDim.new(0, 16)

local container = Instance.new("UIListLayout", MainWindow)
container.Padding = UDim.new(0, 15)
container.HorizontalAlignment = Enum.HorizontalAlignment.Center

local function createInput(text)
    local box = Instance.new("TextBox", MainWindow)
    box.Size = UDim2.new(0.9, 0, 0, 40)
    box.BackgroundColor3 = THEME.Glass
    box.TextColor3 = Color3.new(1,1,1)
    box.Text = text
    Instance.new("UICorner", box).CornerRadius = UDim.new(0, 10)
    Instance.new("UIStroke", box).Color = THEME.Accent
end
