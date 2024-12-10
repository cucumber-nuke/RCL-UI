local Library = {}
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

Library.Settings = {
    MainColor = Color3.fromRGB(25, 25, 25),
    AccentColor = Color3.fromRGB(0, 170, 255),
    BackgroundColor = Color3.fromRGB(30, 30, 30),
    TextColor = Color3.fromRGB(255, 255, 255),
    ButtonColor = Color3.fromRGB(40, 40, 40),
    ButtonHoverColor = Color3.fromRGB(50, 50, 50),
    GlowColor = Color3.fromRGB(0, 170, 255),
    Font = Enum.Font.GothamBold,
    TextSize = 14,
    EasingStyle = Enum.EasingStyle.Quart,
    AnimationDuration = 0.5
}

local function CreateParticles(position)
    local particles = {}
    for i = 1, 20 do
        local particle = Instance.new("Frame")
        particle.Size = UDim2.new(0, math.random(2, 5), 0, math.random(2, 5))
        particle.BackgroundColor3 = Library.Settings.AccentColor
        particle.Position = position
        particle.BorderSizePixel = 0
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(1, 0)
        corner.Parent = particle
        
        particles[i] = particle
    end
    return particles
end

function Library:CreateWindow(title)
    local Window = {}
    Window.Toggles = {}
    Window.Tasks = {}
    
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Parent = game.CoreGui
    ScreenGui.Name = title
    
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Parent = ScreenGui
    MainFrame.BackgroundColor3 = Library.Settings.MainColor
    MainFrame.Position = UDim2.new(0.5, -150, 0.5, -200)
    MainFrame.Size = UDim2.new(0, 300, 0, 400)
    MainFrame.ClipsDescendants = true
    
    local TitleBar = Instance.new("Frame")
    TitleBar.Name = "TitleBar"
    TitleBar.Parent = MainFrame
    TitleBar.BackgroundColor3 = Library.Settings.BackgroundColor
    TitleBar.Size = UDim2.new(1, 0, 0, 30)
    
    local TitleText = Instance.new("TextLabel")
    TitleText.Parent = TitleBar
    TitleText.BackgroundTransparency = 1
    TitleText.Position = UDim2.new(0, 10, 0, 0)
    TitleText.Size = UDim2.new(1, -50, 1, 0)
    TitleText.Font = Library.Settings.Font
    TitleText.Text = title
    TitleText.TextColor3 = Library.Settings.TextColor
    TitleText.TextSize = 14
    TitleText.TextXAlignment = Enum.TextXAlignment.Left
    
    local MinimizeButton = Instance.new("TextButton")
    MinimizeButton.Parent = TitleBar
    MinimizeButton.BackgroundTransparency = 1
    MinimizeButton.Position = UDim2.new(1, -50, 0, 0)
    MinimizeButton.Size = UDim2.new(0, 25, 1, 0)
    MinimizeButton.Font = Library.Settings.Font
    MinimizeButton.Text = "-"
    MinimizeButton.TextColor3 = Library.Settings.TextColor
    MinimizeButton.TextSize = 20
    
    local CloseButton = Instance.new("TextButton")
    CloseButton.Parent = TitleBar
    CloseButton.BackgroundTransparency = 1
    CloseButton.Position = UDim2.new(1, -25, 0, 0)
    CloseButton.Size = UDim2.new(0, 25, 1, 0)
    CloseButton.Font = Library.Settings.Font
    CloseButton.Text = "X"
    CloseButton.TextColor3 = Library.Settings.TextColor
    CloseButton.TextSize = 14

    local ContentFrame = Instance.new("Frame")
    ContentFrame.Name = "ContentFrame"
    ContentFrame.Parent = MainFrame
    ContentFrame.BackgroundTransparency = 1
    ContentFrame.Position = UDim2.new(0, 0, 0, 30)
    ContentFrame.Size = UDim2.new(1, 0, 1, -30)
    
    local minimized = false
    MinimizeButton.MouseButton1Click:Connect(function()
        minimized = not minimized
        local targetSize = minimized and UDim2.new(0, 300, 0, 30) or UDim2.new(0, 300, 0, 400)
        
        TweenService:Create(MainFrame, 
            TweenInfo.new(0.5, Enum.EasingStyle.Quart), 
            {Size = targetSize}
        ):Play()
    end)
    
    CloseButton.MouseButton1Click:Connect(function()
        local particles = CreateParticles(UDim2.new(0.5, 0, 0.5, 0))
        for _, particle in ipairs(particles) do
            particle.Parent = MainFrame
            
            local angle = math.random() * math.pi * 2
            local speed = math.random(300, 500)
            local dx = math.cos(angle) * speed
            local dy = math.sin(angle) * speed
            
            local startPos = particle.Position
            local targetPos = UDim2.new(
                startPos.X.Scale + (dx / MainFrame.AbsoluteSize.X), 
                startPos.X.Offset,
                startPos.Y.Scale + (dy / MainFrame.AbsoluteSize.Y), 
                startPos.Y.Offset
            )
            
            TweenService:Create(particle, 
                TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
                {
                    Position = targetPos,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(0, 0, 0, 0)
                }
            ):Play()
        end
        
        TweenService:Create(MainFrame, 
            TweenInfo.new(0.5, Enum.EasingStyle.Quad), 
            {
                BackgroundTransparency = 1,
                Size = UDim2.new(0, 0, 0, 0),
                Position = UDim2.new(0.5, 0, 0.5, 0)
            }
        ):Play()
        
        for _, toggle in pairs(Window.Toggles) do
            if toggle.Value then
                toggle:Set(false)
            end
        end
        
        for _, task in pairs(Window.Tasks) do
            if typeof(task) == "RBXScriptConnection" then
                task:Disconnect()
            end
        end
        
        wait(0.5)
        ScreenGui:Destroy()
    end)

    return Window
end

return Library
