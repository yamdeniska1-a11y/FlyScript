local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local flying = false
local noclip = false
local speed = 50

local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.ResetOnSpawn = false -- Важно: не удаляется после смерти

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 200, 0, 160)
frame.Position = UDim2.new(0.5, -100, 0.5, -80)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Active = true
frame.Draggable = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

local speedInput = Instance.new("TextBox", frame)
speedInput.Size = UDim2.new(0.8, 0, 0.3, 0)
speedInput.Position = UDim2.new(0.1, 0, 0.1, 0)
speedInput.PlaceholderText = "Скорость (например 50)"
speedInput.Text = tostring(speed)
speedInput.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
speedInput.TextColor3 = Color3.new(1, 1, 1)

local noclipBtn = Instance.new("TextButton", frame)
noclipBtn.Size = UDim2.new(0.8, 0, 0.3, 0)
noclipBtn.Position = UDim2.new(0.1, 0, 0.5, 0)
noclipBtn.Text = "Noclip: OFF"
noclipBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 50)

local attachment = Instance.new("Attachment")
local lv = Instance.new("LinearVelocity")
lv.MaxForce = 100000
lv.VectorVelocity = Vector3.new(0,0,0)
lv.Attachment0 = attachment

local align = Instance.new("AlignOrientation")
align.MaxTorque = 100000
align.Attachment0 = attachment
align.Mode = Enum.OrientationAlignmentMode.OneAttachment

local function updatePhysics()
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChild("Humanoid")
    
    if flying and hrp and hum then
        hum.PlatformStand = true
        attachment.Parent = hrp
        lv.Parent = hrp
        align.Parent = hrp
        align.CFrame = workspace.CurrentCamera.CFrame
        
        local dir = Vector3.new(0,0,0)
        local cam = workspace.CurrentCamera
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir += cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir -= cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir -= cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir += cam.CFrame.RightVector end
        
        lv.VectorVelocity = dir * speed
    else
        if hum then hum.PlatformStand = false end
        lv.Parent = nil
        align.Parent = nil
        attachment.Parent = nil
    end

    if noclip and char then
        for _, v in pairs(char:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end
end

RunService.RenderStepped:Connect(updatePhysics)

UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Enum.KeyCode.RightAlt then
        frame.Visible = not frame.Visible
    elseif input.KeyCode == Enum.KeyCode.LeftAlt then
        flying = not flying
    end
end)

noclipBtn.MouseButton1Click:Connect(function()
    noclip = not noclip
    noclipBtn.Text = noclip and "Noclip: ON" or "Noclip: OFF"
    noclipBtn.BackgroundColor3 = noclip and Color3.fromRGB(50, 150, 50) or Color3.fromRGB(150, 50, 50)
end)

speedInput.FocusLost:Connect(function()
    speed = tonumber(speedInput.Text) or 50
end)
