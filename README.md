--[[
    ╔═══════════════════════════════════════════════════════════════╗
    ║                 INFINITYCORE HUB - CS STYLE                   ║
    ║           Updated: Aimbot on RIGHT Mouse Button               ║
    ╚═══════════════════════════════════════════════════════════════╝
]]

-- Services
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")

-- Player Info
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

-- Variables
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local isOpen = true
local toggleKey = Enum.KeyCode.RightControl

-- Settings Storage
local Settings = {
    Fly = false,
    FlySpeed = 50,
    WalkSpeed = 16,
    JumpPower = 50,
    InfiniteJump = false,
    GodMode = false,
    Noclip = false,
    ESP = false,
    ESPColor = Color3.fromRGB(138, 43, 226),
    ESPBox = true,
    ESPName = true,
    ESPHealth = true,
    Aimbot = false,
    AimbotFOV = 100,
    ShowFOV = false,
    AimbotSmooth = 0.5,
    AimPart = "Head",
    FontSize = 14,
    Theme = "Purple"
}

-- Theme Colors
local Themes = {
    Purple = {
        Primary = Color3.fromRGB(138, 43, 226),
        Secondary = Color3.fromRGB(75, 0, 130),
        Background = Color3.fromRGB(15, 15, 20),
        BackgroundSecondary = Color3.fromRGB(25, 25, 35),
        Text = Color3.fromRGB(255, 255, 255),
        TextSecondary = Color3.fromRGB(180, 180, 180),
        Accent = Color3.fromRGB(186, 85, 211)
    },
    Red = {
        Primary = Color3.fromRGB(220, 20, 60),
        Secondary = Color3.fromRGB(139, 0, 0),
        Background = Color3.fromRGB(15, 15, 20),
        BackgroundSecondary = Color3.fromRGB(25, 25, 35),
        Text = Color3.fromRGB(255, 255, 255),
        TextSecondary = Color3.fromRGB(180, 180, 180),
        Accent = Color3.fromRGB(255, 99, 71)
    },
    Blue = {
        Primary = Color3.fromRGB(30, 144, 255),
        Secondary = Color3.fromRGB(0, 0, 139),
        Background = Color3.fromRGB(15, 15, 20),
        BackgroundSecondary = Color3.fromRGB(25, 25, 35),
        Text = Color3.fromRGB(255, 255, 255),
        TextSecondary = Color3.fromRGB(180, 180, 180),
        Accent = Color3.fromRGB(135, 206, 250)
    },
    Green = {
        Primary = Color3.fromRGB(50, 205, 50),
        Secondary = Color3.fromRGB(0, 100, 0),
        Background = Color3.fromRGB(15, 15, 20),
        BackgroundSecondary = Color3.fromRGB(25, 25, 35),
        Text = Color3.fromRGB(255, 255, 255),
        TextSecondary = Color3.fromRGB(180, 180, 180),
        Accent = Color3.fromRGB(144, 238, 144)
    }
}

local CurrentTheme = Themes.Purple

-- Visual FOV Circle
local FOVCircle = Instance.new("ImageLabel")
FOVCircle.Name = "FOVCircle"
FOVCircle.Parent = ScreenGui
FOVCircle.BackgroundTransparency = 1
FOVCircle.Image = "rbxassetid://3570695787"
FOVCircle.ImageColor3 = CurrentTheme.Primary
FOVCircle.ImageTransparency = 0.6
FOVCircle.Visible = false
FOVCircle.AnchorPoint = Vector2.new(0.5, 0.5)
FOVCircle.ZIndex = 0

-- Sound Effects
local function PlayClickSound()
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://6895079853"
    sound.Volume = 0.5
    sound.PlaybackSpeed = 1.2
    sound.Parent = game:GetService("SoundService")
    sound:Play()
    game:GetService("Debris"):AddItem(sound, 1)
end

local function PlayToggleSound()
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://6895079853"
    sound.Volume = 0.3
    sound.PlaybackSpeed = 1.5
    sound.Parent = game:GetService("SoundService")
    sound:Play()
    game:GetService("Debris"):AddItem(sound, 1)
end

local function PlayHoverSound()
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://6895079853"
    sound.Volume = 0.15
    sound.PlaybackSpeed = 2
    sound.Parent = game:GetService("SoundService")
    sound:Play()
    game:GetService("Debris"):AddItem(sound, 1)
end

-- Tween Functions
local function CreateTween(object, properties, duration, easingStyle, easingDirection)
    local tween = TweenService:Create(
        object,
        TweenInfo.new(duration or 0.3, easingStyle or Enum.EasingStyle.Quart, easingDirection or Enum.EasingDirection.Out),
        properties
    )
    return tween
end

-- Create Blur Effect
local BlurEffect = Instance.new("BlurEffect")
BlurEffect.Name = "InfinityCoreBlur"
BlurEffect.Size = 0
BlurEffect.Parent = Lighting

local function SetBlur(enabled)
    local targetSize = enabled and 15 or 0
    CreateTween(BlurEffect, {Size = targetSize}, 0.5):Play()
end

-- Notification System
local function CreateNotification(text, duration)
    local NotifFrame = Instance.new("Frame")
    local NotifText = Instance.new("TextLabel")
    local NotifIcon = Instance.new("ImageLabel")
    local UICorner = Instance.new("UICorner")
    local UIStroke = Instance.new("UIStroke")
    
    NotifFrame.Name = "Notification"
    NotifFrame.Parent = ScreenGui
    NotifFrame.BackgroundColor3 = CurrentTheme.Background
    NotifFrame.Position = UDim2.new(0.5, -150, 0, -60)
    NotifFrame.Size = UDim2.new(0, 300, 0, 50)
    NotifFrame.AnchorPoint = Vector2.new(0.5, 0)
    
    UICorner.CornerRadius = UDim.new(0, 8)
    UICorner.Parent = NotifFrame
    
    UIStroke.Color = CurrentTheme.Primary
    UIStroke.Thickness = 2
    UIStroke.Parent = NotifFrame
    
    NotifIcon.Name = "Icon"
    NotifIcon.Parent = NotifFrame
    NotifIcon.BackgroundTransparency = 1
    NotifIcon.Position = UDim2.new(0, 10, 0.5, -12)
    NotifIcon.Size = UDim2.new(0, 24, 0, 24)
    NotifIcon.Image = "rbxassetid://7733715400"
    NotifIcon.ImageColor3 = CurrentTheme.Primary
    
    NotifText.Name = "Text"
    NotifText.Parent = NotifFrame
    NotifText.BackgroundTransparency = 1
    NotifText.Position = UDim2.new(0, 45, 0, 0)
    NotifText.Size = UDim2.new(1, -55, 1, 0)
    NotifText.Font = Enum.Font.GothamSemibold
    NotifText.Text = text
    NotifText.TextColor3 = CurrentTheme.Text
    NotifText.TextSize = 14
    NotifText.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Animate in
    CreateTween(NotifFrame, {Position = UDim2.new(0.5, -150, 0, 20)}, 0.5, Enum.EasingStyle.Back):Play()
    
    -- Animate out
    task.delay(duration or 3, function()
        local tweenOut = CreateTween(NotifFrame, {Position = UDim2.new(0.5, -150, 0, -60)}, 0.5, Enum.EasingStyle.Back, Enum.EasingDirection.In)
        tweenOut:Play()
        tweenOut.Completed:Connect(function()
            NotifFrame:Destroy()
        end)
    end)
end

-- Create Main GUI
ScreenGui.Name = "InfinityCoreHub"
ScreenGui.Parent = game:GetService("CoreGui")
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.ResetOnSpawn = false

-- Main Container
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = CurrentTheme.Background
MainFrame.Position = UDim2.new(0.5, -300, 0.5, -200)
MainFrame.Size = UDim2.new(0, 600, 0, 450)
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
MainFrame.ClipsDescendants = true

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = CurrentTheme.Primary
MainStroke.Thickness = 2
MainStroke.Parent = MainFrame

-- Title Bar
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Parent = MainFrame
TitleBar.BackgroundColor3 = CurrentTheme.BackgroundSecondary
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BorderSizePixel = 0

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 10)
TitleCorner.Parent = TitleBar

local TitleFix = Instance.new("Frame")
TitleFix.Parent = TitleBar
TitleFix.BackgroundColor3 = CurrentTheme.BackgroundSecondary
TitleFix.Position = UDim2.new(0, 0, 0.5, 0)
TitleFix.Size = UDim2.new(1, 0, 0.5, 0)
TitleFix.BorderSizePixel = 0

local TitleText = Instance.new("TextLabel")
TitleText.Name = "Title"
TitleText.Parent = TitleBar
TitleText.BackgroundTransparency = 1
TitleText.Position = UDim2.new(0, 15, 0, 0)
TitleText.Size = UDim2.new(0, 200, 1, 0)
TitleText.Font = Enum.Font.GothamBold
TitleText.Text = "🎮 INFINITYCORE"
TitleText.TextColor3 = CurrentTheme.Primary
TitleText.TextSize = 18
TitleText.TextXAlignment = Enum.TextXAlignment.Left

local SubTitle = Instance.new("TextLabel")
SubTitle.Name = "SubTitle"
SubTitle.Parent = TitleBar
SubTitle.BackgroundTransparency = 1
SubTitle.Position = UDim2.new(0, 140, 0, 0)
SubTitle.Size = UDim2.new(0, 100, 1, 0)
SubTitle.Font = Enum.Font.Gotham
SubTitle.Text = "| CS STYLE"
SubTitle.TextColor3 = CurrentTheme.TextSecondary
SubTitle.TextSize = 12
SubTitle.TextXAlignment = Enum.TextXAlignment.Left

-- Close Button
local CloseButton = Instance.new("TextButton")
CloseButton.Name = "Close"
CloseButton.Parent = TitleBar
CloseButton.BackgroundTransparency = 1
CloseButton.Position = UDim2.new(1, -35, 0.5, -12)
CloseButton.Size = UDim2.new(0, 24, 0, 24)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.Text = "×"
CloseButton.TextColor3 = CurrentTheme.Text
CloseButton.TextSize = 24

-- Minimize Button
local MinButton = Instance.new("TextButton")
MinButton.Name = "Minimize"
MinButton.Parent = TitleBar
MinButton.BackgroundTransparency = 1
MinButton.Position = UDim2.new(1, -60, 0.5, -12)
MinButton.Size = UDim2.new(0, 24, 0, 24)
MinButton.Font = Enum.Font.GothamBold
MinButton.Text = "−"
MinButton.TextColor3 = CurrentTheme.Text
MinButton.TextSize = 24

-- Tab Container
local TabContainer = Instance.new("Frame")
TabContainer.Name = "TabContainer"
TabContainer.Parent = MainFrame
TabContainer.BackgroundColor3 = CurrentTheme.BackgroundSecondary
TabContainer.Position = UDim2.new(0, 0, 0, 40)
TabContainer.Size = UDim2.new(0, 120, 1, -40)
TabContainer.BorderSizePixel = 0

-- Content Container
local ContentContainer = Instance.new("Frame")
ContentContainer.Name = "ContentContainer"
ContentContainer.Parent = MainFrame
ContentContainer.BackgroundTransparency = 1
ContentContainer.Position = UDim2.new(0, 130, 0, 50)
ContentContainer.Size = UDim2.new(1, -140, 1, -60)

-- Tab Buttons
local TabList = Instance.new("UIListLayout")
TabList.Parent = TabContainer
TabList.SortOrder = Enum.SortOrder.LayoutOrder
TabList.Padding = UDim.new(0, 5)

local TabPadding = Instance.new("UIPadding")
TabPadding.Parent = TabContainer
TabPadding.PaddingTop = UDim.new(0, 10)
TabPadding.PaddingLeft = UDim.new(0, 10)
TabPadding.PaddingRight = UDim.new(0, 10)

local Tabs = {}
local TabContents = {}
local CurrentTab = "Main"

local function CreateTab(name, icon)
    local TabButton = Instance.new("TextButton")
    TabButton.Name = name
    TabButton.Parent = TabContainer
    TabButton.BackgroundColor3 = CurrentTheme.Background
    TabButton.Size = UDim2.new(1, 0, 0, 35)
    TabButton.Font = Enum.Font.GothamSemibold
    TabButton.Text = "  " .. icon .. " " .. name
    TabButton.TextColor3 = CurrentTheme.TextSecondary
    TabButton.TextSize = 13
    TabButton.TextXAlignment = Enum.TextXAlignment.Left
    TabButton.AutoButtonColor = false
    
    local TabCorner = Instance.new("UICorner")
    TabCorner.CornerRadius = UDim.new(0, 6)
    TabCorner.Parent = TabButton
    
    local TabContent = Instance.new("ScrollingFrame")
    TabContent.Name = name .. "Content"
    TabContent.Parent = ContentContainer
    TabContent.BackgroundTransparency = 1
    TabContent.Size = UDim2.new(1, 0, 1, 0)
    TabContent.ScrollBarThickness = 4
    TabContent.ScrollBarImageColor3 = CurrentTheme.Primary
    TabContent.Visible = name == "Main"
    TabContent.CanvasSize = UDim2.new(0, 0, 0, 0)
    TabContent.AutomaticCanvasSize = Enum.AutomaticSize.Y
    
    local ContentList = Instance.new("UIListLayout")
    ContentList.Parent = TabContent
    ContentList.SortOrder = Enum.SortOrder.LayoutOrder
    ContentList.Padding = UDim.new(0, 8)
    
    local ContentPadding = Instance.new("UIPadding")
    ContentPadding.Parent = TabContent
    ContentPadding.PaddingTop = UDim.new(0, 5)
    ContentPadding.PaddingRight = UDim.new(0, 10)
    
    Tabs[name] = TabButton
    TabContents[name] = TabContent
    
    -- Hover Effect
    TabButton.MouseEnter:Connect(function()
        PlayHoverSound()
        if CurrentTab ~= name then
            CreateTween(TabButton, {BackgroundColor3 = CurrentTheme.BackgroundSecondary}, 0.2):Play()
        end
    end)
    
    TabButton.MouseLeave:Connect(function()
        if CurrentTab ~= name then
            CreateTween(TabButton, {BackgroundColor3 = CurrentTheme.Background}, 0.2):Play()
        end
    end)
    
    TabButton.MouseButton1Click:Connect(function()
        PlayClickSound()
        
        for tabName, tabBtn in pairs(Tabs) do
            if tabName == name then
                CreateTween(tabBtn, {BackgroundColor3 = CurrentTheme.Primary}, 0.3):Play()
                CreateTween(tabBtn, {TextColor3 = CurrentTheme.Text}, 0.3):Play()
                TabContents[tabName].Visible = true
            else
                CreateTween(tabBtn, {BackgroundColor3 = CurrentTheme.Background}, 0.3):Play()
                CreateTween(tabBtn, {TextColor3 = CurrentTheme.TextSecondary}, 0.3):Play()
                TabContents[tabName].Visible = false
            end
        end
        CurrentTab = name
    end)
    
    if name == "Main" then
        TabButton.BackgroundColor3 = CurrentTheme.Primary
        TabButton.TextColor3 = CurrentTheme.Text
    end
    
    return TabContent
end

-- UI Elements Creation Functions
local function CreateSection(parent, title)
    local Section = Instance.new("Frame")
    Section.Name = title .. "Section"
    Section.Parent = parent
    Section.BackgroundColor3 = CurrentTheme.BackgroundSecondary
    Section.Size = UDim2.new(1, 0, 0, 35)
    Section.BorderSizePixel = 0
    
    local SectionCorner = Instance.new("UICorner")
    SectionCorner.CornerRadius = UDim.new(0, 6)
    SectionCorner.Parent = Section
    
    local SectionTitle = Instance.new("TextLabel")
    SectionTitle.Parent = Section
    SectionTitle.BackgroundTransparency = 1
    SectionTitle.Position = UDim2.new(0, 12, 0, 0)
    SectionTitle.Size = UDim2.new(1, -24, 1, 0)
    SectionTitle.Font = Enum.Font.GothamBold
    SectionTitle.Text = "⚡ " .. title
    SectionTitle.TextColor3 = CurrentTheme.Primary
    SectionTitle.TextSize = 13
    SectionTitle.TextXAlignment = Enum.TextXAlignment.Left
    
    return Section
end

local function CreateToggle(parent, text, default, callback)
    local Toggle = Instance.new("Frame")
    Toggle.Name = text .. "Toggle"
    Toggle.Parent = parent
    Toggle.BackgroundColor3 = CurrentTheme.BackgroundSecondary
    Toggle.Size = UDim2.new(1, 0, 0, 40)
    Toggle.BorderSizePixel = 0
    
    local ToggleCorner = Instance.new("UICorner")
    ToggleCorner.CornerRadius = UDim.new(0, 6)
    ToggleCorner.Parent = Toggle
    
    local ToggleText = Instance.new("TextLabel")
    ToggleText.Parent = Toggle
    ToggleText.BackgroundTransparency = 1
    ToggleText.Position = UDim2.new(0, 12, 0, 0)
    ToggleText.Size = UDim2.new(1, -70, 1, 0)
    ToggleText.Font = Enum.Font.GothamSemibold
    ToggleText.Text = text
    ToggleText.TextColor3 = CurrentTheme.Text
    ToggleText.TextSize = Settings.FontSize
    ToggleText.TextXAlignment = Enum.TextXAlignment.Left
    
    local ToggleButton = Instance.new("Frame")
    ToggleButton.Name = "Button"
    ToggleButton.Parent = Toggle
    ToggleButton.BackgroundColor3 = default and CurrentTheme.Primary or Color3.fromRGB(60, 60, 70)
    ToggleButton.Position = UDim2.new(1, -55, 0.5, -10)
    ToggleButton.Size = UDim2.new(0, 42, 0, 20)
    
    local ToggleBtnCorner = Instance.new("UICorner")
    ToggleBtnCorner.CornerRadius = UDim.new(1, 0)
    ToggleBtnCorner.Parent = ToggleButton
    
    local ToggleCircle = Instance.new("Frame")
    ToggleCircle.Name = "Circle"
    ToggleCircle.Parent = ToggleButton
    ToggleCircle.BackgroundColor3 = CurrentTheme.Text
    ToggleCircle.Position = default and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)
    ToggleCircle.Size = UDim2.new(0, 16, 0, 16)
    
    local CircleCorner = Instance.new("UICorner")
    CircleCorner.CornerRadius = UDim.new(1, 0)
    CircleCorner.Parent = ToggleCircle
    
    local toggled = default
    
    local ToggleClickArea = Instance.new("TextButton")
    ToggleClickArea.Parent = Toggle
    ToggleClickArea.BackgroundTransparency = 1
    ToggleClickArea.Size = UDim2.new(1, 0, 1, 0)
    ToggleClickArea.Text = ""
    
    ToggleClickArea.MouseEnter:Connect(function()
        PlayHoverSound()
        CreateTween(Toggle, {BackgroundColor3 = Color3.fromRGB(35, 35, 45)}, 0.2):Play()
    end)
    
    ToggleClickArea.MouseLeave:Connect(function()
        CreateTween(Toggle, {BackgroundColor3 = CurrentTheme.BackgroundSecondary}, 0.2):Play()
    end)
    
    ToggleClickArea.MouseButton1Click:Connect(function()
        PlayToggleSound()
        toggled = not toggled
        
        if toggled then
            CreateTween(ToggleButton, {BackgroundColor3 = CurrentTheme.Primary}, 0.3):Play()
            CreateTween(ToggleCircle, {Position = UDim2.new(1, -18, 0.5, -8)}, 0.3, Enum.EasingStyle.Back):Play()
        else
            CreateTween(ToggleButton, {BackgroundColor3 = Color3.fromRGB(60, 60, 70)}, 0.3):Play()
            CreateTween(ToggleCircle, {Position = UDim2.new(0, 2, 0.5, -8)}, 0.3, Enum.EasingStyle.Back):Play()
        end
        
        if callback then
            callback(toggled)
        end
    end)
    
    return Toggle, function() return toggled end
end

local function CreateSlider(parent, text, min, max, default, callback)
    local Slider = Instance.new("Frame")
    Slider.Name = text .. "Slider"
    Slider.Parent = parent
    Slider.BackgroundColor3 = CurrentTheme.BackgroundSecondary
    Slider.Size = UDim2.new(1, 0, 0, 55)
    Slider.BorderSizePixel = 0
    
    local SliderCorner = Instance.new("UICorner")
    SliderCorner.CornerRadius = UDim.new(0, 6)
    SliderCorner.Parent = Slider
    
    local SliderText = Instance.new("TextLabel")
    SliderText.Parent = Slider
    SliderText.BackgroundTransparency = 1
    SliderText.Position = UDim2.new(0, 12, 0, 5)
    SliderText.Size = UDim2.new(1, -60, 0, 20)
    SliderText.Font = Enum.Font.GothamSemibold
    SliderText.Text = text
    SliderText.TextColor3 = CurrentTheme.Text
    SliderText.TextSize = Settings.FontSize
    SliderText.TextXAlignment = Enum.TextXAlignment.Left
    
    local ValueLabel = Instance.new("TextLabel")
    ValueLabel.Parent = Slider
    ValueLabel.BackgroundTransparency = 1
    ValueLabel.Position = UDim2.new(1, -50, 0, 5)
    ValueLabel.Size = UDim2.new(0, 40, 0, 20)
    ValueLabel.Font = Enum.Font.GothamBold
    ValueLabel.Text = tostring(default)
    ValueLabel.TextColor3 = CurrentTheme.Primary
    ValueLabel.TextSize = 13
    
    local SliderBar = Instance.new("Frame")
    SliderBar.Name = "Bar"
    SliderBar.Parent = Slider
    SliderBar.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    SliderBar.Position = UDim2.new(0, 12, 0, 32)
    SliderBar.Size = UDim2.new(1, -24, 0, 8)
    
    local BarCorner = Instance.new("UICorner")
    BarCorner.CornerRadius = UDim.new(1, 0)
    BarCorner.Parent = SliderBar
    
    local SliderFill = Instance.new("Frame")
    SliderFill.Name = "Fill"
    SliderFill.Parent = SliderBar
    SliderFill.BackgroundColor3 = CurrentTheme.Primary
    SliderFill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    
    local FillCorner = Instance.new("UICorner")
    FillCorner.CornerRadius = UDim.new(1, 0)
    FillCorner.Parent = SliderFill
    
    local SliderKnob = Instance.new("Frame")
    SliderKnob.Name = "Knob"
    SliderKnob.Parent = SliderBar
    SliderKnob.BackgroundColor3 = CurrentTheme.Text
    SliderKnob.Position = UDim2.new((default - min) / (max - min), -8, 0.5, -8)
    SliderKnob.Size = UDim2.new(0, 16, 0, 16)
    SliderKnob.ZIndex = 2
    
    local KnobCorner = Instance.new("UICorner")
    KnobCorner.CornerRadius = UDim.new(1, 0)
    KnobCorner.Parent = SliderKnob
    
    local dragging = false
    local value = default
    
    local function UpdateSlider(input)
        local pos = math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1)
        value = math.floor(min + (max - min) * pos)
        
        CreateTween(SliderFill, {Size = UDim2.new(pos, 0, 1, 0)}, 0.1):Play()
        CreateTween(SliderKnob, {Position = UDim2.new(pos, -8, 0.5, -8)}, 0.1):Play()
        ValueLabel.Text = tostring(value)
        
        if callback then
            callback(value)
        end
    end
    
    SliderBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            PlayClickSound()
            UpdateSlider(input)
        end
    end)
    
    SliderBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            UpdateSlider(input)
        end
    end)
    
    return Slider, function() return value end
end

local function CreateDropdown(parent, text, options, default, callback)
    local Dropdown = Instance.new("Frame")
    Dropdown.Name = text .. "Dropdown"
    Dropdown.Parent = parent
    Dropdown.BackgroundColor3 = CurrentTheme.BackgroundSecondary
    Dropdown.Size = UDim2.new(1, 0, 0, 40)
    Dropdown.ClipsDescendants = true
    Dropdown.BorderSizePixel = 0
    
    local DropCorner = Instance.new("UICorner")
    DropCorner.CornerRadius = UDim.new(0, 6)
    DropCorner.Parent = Dropdown
    
    local DropText = Instance.new("TextLabel")
    DropText.Parent = Dropdown
    DropText.BackgroundTransparency = 1
    DropText.Position = UDim2.new(0, 12, 0, 0)
    DropText.Size = UDim2.new(0.5, 0, 0, 40)
    DropText.Font = Enum.Font.GothamSemibold
    DropText.Text = text
    DropText.TextColor3 = CurrentTheme.Text
    DropText.TextSize = Settings.FontSize
    DropText.TextXAlignment = Enum.TextXAlignment.Left
    
    local SelectedFrame = Instance.new("Frame")
    SelectedFrame.Parent = Dropdown
    SelectedFrame.BackgroundColor3 = CurrentTheme.Background
    SelectedFrame.Position = UDim2.new(0.5, 5, 0, 7)
    SelectedFrame.Size = UDim2.new(0.5, -17, 0, 26)
    
    local SelCorner = Instance.new("UICorner")
    SelCorner.CornerRadius = UDim.new(0, 4)
    SelCorner.Parent = SelectedFrame
    
    local SelectedText = Instance.new("TextLabel")
    SelectedText.Parent = SelectedFrame
    SelectedText.BackgroundTransparency = 1
    SelectedText.Size = UDim2.new(1, -25, 1, 0)
    SelectedText.Position = UDim2.new(0, 8, 0, 0)
    SelectedText.Font = Enum.Font.Gotham
    SelectedText.Text = default
    SelectedText.TextColor3 = CurrentTheme.Primary
    SelectedText.TextSize = 12
    SelectedText.TextXAlignment = Enum.TextXAlignment.Left
    
    local Arrow = Instance.new("TextLabel")
    Arrow.Parent = SelectedFrame
    Arrow.BackgroundTransparency = 1
    Arrow.Position = UDim2.new(1, -20, 0, 0)
    Arrow.Size = UDim2.new(0, 15, 1, 0)
    Arrow.Font = Enum.Font.GothamBold
    Arrow.Text = "▼"
    Arrow.TextColor3 = CurrentTheme.TextSecondary
    Arrow.TextSize = 10
    
    local OptionsContainer = Instance.new("Frame")
    OptionsContainer.Parent = Dropdown
    OptionsContainer.BackgroundColor3 = CurrentTheme.Background
    OptionsContainer.Position = UDim2.new(0, 10, 0, 45)
    OptionsContainer.Size = UDim2.new(1, -20, 0, #options * 30)
    OptionsContainer.Visible = false
    
    local OptCorner = Instance.new("UICorner")
    OptCorner.CornerRadius = UDim.new(0, 4)
    OptCorner.Parent = OptionsContainer
    
    local OptList = Instance.new("UIListLayout")
    OptList.Parent = OptionsContainer
    OptList.SortOrder = Enum.SortOrder.LayoutOrder
    
    local isOpen = false
    local selected = default
    
    for i, option in ipairs(options) do
        local OptBtn = Instance.new("TextButton")
        OptBtn.Parent = OptionsContainer
        OptBtn.BackgroundTransparency = 1
        OptBtn.Size = UDim2.new(1, 0, 0, 30)
        OptBtn.Font = Enum.Font.Gotham
        OptBtn.Text = option
        OptBtn.TextColor3 = CurrentTheme.Text
        OptBtn.TextSize = 12
        
        OptBtn.MouseEnter:Connect(function()
            PlayHoverSound()
            CreateTween(OptBtn, {TextColor3 = CurrentTheme.Primary}, 0.2):Play()
        end)
        
        OptBtn.MouseLeave:Connect(function()
            CreateTween(OptBtn, {TextColor3 = CurrentTheme.Text}, 0.2):Play()
        end)
        
        OptBtn.MouseButton1Click:Connect(function()
            PlayClickSound()
            selected = option
            SelectedText.Text = option
            isOpen = false
            OptionsContainer.Visible = false
            CreateTween(Dropdown, {Size = UDim2.new(1, 0, 0, 40)}, 0.3):Play()
            CreateTween(Arrow, {Rotation = 0}, 0.3):Play()
            
            if callback then
                callback(option)
            end
        end)
    end
    
    local DropClickArea = Instance.new("TextButton")
    DropClickArea.Parent = SelectedFrame
    DropClickArea.BackgroundTransparency = 1
    DropClickArea.Size = UDim2.new(1, 0, 1, 0)
    DropClickArea.Text = ""
    
    DropClickArea.MouseButton1Click:Connect(function()
        PlayClickSound()
        isOpen = not isOpen
        OptionsContainer.Visible = isOpen
        
        if isOpen then
            CreateTween(Dropdown, {Size = UDim2.new(1, 0, 0, 50 + #options * 30)}, 0.3):Play()
            CreateTween(Arrow, {Rotation = 180}, 0.3):Play()
        else
            CreateTween(Dropdown, {Size = UDim2.new(1, 0, 0, 40)}, 0.3):Play()
            CreateTween(Arrow, {Rotation = 0}, 0.3):Play()
        end
    end)
    
    return Dropdown, function() return selected end
end

local function CreateColorPicker(parent, text, default, callback)
    local ColorPicker = Instance.new("Frame")
    ColorPicker.Name = text .. "ColorPicker"
    ColorPicker.Parent = parent
    ColorPicker.BackgroundColor3 = CurrentTheme.BackgroundSecondary
    ColorPicker.Size = UDim2.new(1, 0, 0, 40)
    ColorPicker.BorderSizePixel = 0
    
    local CPCorner = Instance.new("UICorner")
    CPCorner.CornerRadius = UDim.new(0, 6)
    CPCorner.Parent = ColorPicker
    
    local CPText = Instance.new("TextLabel")
    CPText.Parent = ColorPicker
    CPText.BackgroundTransparency = 1
    CPText.Position = UDim2.new(0, 12, 0, 0)
    CPText.Size = UDim2.new(1, -60, 1, 0)
    CPText.Font = Enum.Font.GothamSemibold
    CPText.Text = text
    CPText.TextColor3 = CurrentTheme.Text
    CPText.TextSize = Settings.FontSize
    CPText.TextXAlignment = Enum.TextXAlignment.Left
    
    local ColorDisplay = Instance.new("Frame")
    ColorDisplay.Parent = ColorPicker
    ColorDisplay.BackgroundColor3 = default
    ColorDisplay.Position = UDim2.new(1, -45, 0.5, -12)
    ColorDisplay.Size = UDim2.new(0, 35, 0, 24)
    
    local CDCorner = Instance.new("UICorner")
    CDCorner.CornerRadius = UDim.new(0, 4)
    CDCorner.Parent = ColorDisplay
    
    local CDStroke = Instance.new("UIStroke")
    CDStroke.Color = CurrentTheme.Primary
    CDStroke.Thickness = 1
    CDStroke.Parent = ColorDisplay
    
    return ColorPicker
end

local function CreateKeybind(parent, text, default, callback)
    local Keybind = Instance.new("Frame")
    Keybind.Name = text .. "Keybind"
    Keybind.Parent = parent
    Keybind.BackgroundColor3 = CurrentTheme.BackgroundSecondary
    Keybind.Size = UDim2.new(1, 0, 0, 40)
    Keybind.BorderSizePixel = 0
    
    local KBCorner = Instance.new("UICorner")
    KBCorner.CornerRadius = UDim.new(0, 6)
    KBCorner.Parent = Keybind
    
    local KBText = Instance.new("TextLabel")
    KBText.Parent = Keybind
    KBText.BackgroundTransparency = 1
    KBText.Position = UDim2.new(0, 12, 0, 0)
    KBText.Size = UDim2.new(1, -100, 1, 0)
    KBText.Font = Enum.Font.GothamSemibold
    KBText.Text = text
    KBText.TextColor3 = CurrentTheme.Text
    KBText.TextSize = Settings.FontSize
    KBText.TextXAlignment = Enum.TextXAlignment.Left
    
    local KeyButton = Instance.new("TextButton")
    KeyButton.Parent = Keybind
    KeyButton.BackgroundColor3 = CurrentTheme.Background
    KeyButton.Position = UDim2.new(1, -85, 0.5, -12)
    KeyButton.Size = UDim2.new(0, 75, 0, 24)
    KeyButton.Font = Enum.Font.GothamBold
    KeyButton.Text = default.Name
    KeyButton.TextColor3 = CurrentTheme.Primary
    KeyButton.TextSize = 11
    KeyButton.AutoButtonColor = false
    
    local KeyCorner = Instance.new("UICorner")
    KeyCorner.CornerRadius = UDim.new(0, 4)
    KeyCorner.Parent = KeyButton
    
    local currentKey = default
    local listening = false
    
    KeyButton.MouseButton1Click:Connect(function()
        PlayClickSound()
        listening = true
        KeyButton.Text = "..."
        CreateTween(KeyButton, {BackgroundColor3 = CurrentTheme.Primary}, 0.2):Play()
        CreateTween(KeyButton, {TextColor3 = CurrentTheme.Text}, 0.2):Play()
    end)
    
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if listening and input.UserInputType == Enum.UserInputType.Keyboard then
            listening = false
            currentKey = input.KeyCode
            KeyButton.Text = input.KeyCode.Name
            CreateTween(KeyButton, {BackgroundColor3 = CurrentTheme.Background}, 0.2):Play()
            CreateTween(KeyButton, {TextColor3 = CurrentTheme.Primary}, 0.2):Play()
            
            if callback then
                callback(currentKey)
            end
        end
    end)
    
    return Keybind, function() return currentKey end
end

local function CreateButton(parent, text, callback)
    local Button = Instance.new("TextButton")
    Button.Name = text .. "Button"
    Button.Parent = parent
    Button.BackgroundColor3 = CurrentTheme.Primary
    Button.Size = UDim2.new(1, 0, 0, 35)
    Button.Font = Enum.Font.GothamBold
    Button.Text = text
    Button.TextColor3 = CurrentTheme.Text
    Button.TextSize = Settings.FontSize
    Button.AutoButtonColor = false
    
    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 6)
    BtnCorner.Parent = Button
    
    Button.MouseEnter:Connect(function()
        PlayHoverSound()
        CreateTween(Button, {BackgroundColor3 = CurrentTheme.Accent}, 0.2):Play()
    end)
    
    Button.MouseLeave:Connect(function()
        CreateTween(Button, {BackgroundColor3 = CurrentTheme.Primary}, 0.2):Play()
    end)
    
    Button.MouseButton1Click:Connect(function()
        PlayClickSound()
        
        -- Click animation
        CreateTween(Button, {Size = UDim2.new(0.98, 0, 0, 33)}, 0.1):Play()
        task.wait(0.1)
        CreateTween(Button, {Size = UDim2.new(1, 0, 0, 35)}, 0.1):Play()
        
        if callback then
            callback()
        end
    end)
    
    return Button
end

local function CreateLabel(parent, text)
    local Label = Instance.new("TextLabel")
    Label.Name = "Label"
    Label.Parent = parent
    Label.BackgroundColor3 = CurrentTheme.BackgroundSecondary
    Label.Size = UDim2.new(1, 0, 0, 30)
    Label.Font = Enum.Font.GothamSemibold
    Label.Text = text
    Label.TextColor3 = CurrentTheme.TextSecondary
    Label.TextSize = 12
    
    local LblCorner = Instance.new("UICorner")
    LblCorner.CornerRadius = UDim.new(0, 6)
    LblCorner.Parent = Label
    
    return Label
end

-- Create Tabs
local MainTab = CreateTab("Main", "🎯")
local MiscTab = CreateTab("Misc", "⚙️")
local SettingsTab = CreateTab("Settings", "🔧")

-- ==================== MAIN TAB ====================
CreateSection(MainTab, "Movement")

-- FLIGHT
CreateToggle(MainTab, "Fly", false, function(enabled)
    Settings.Fly = enabled
    
    if enabled then
        local character = LocalPlayer.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            local hrp = character.HumanoidRootPart
            local bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Name = "FlyVelocity"
            bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bodyVelocity.Velocity = Vector3.new(0, 0, 0)
            bodyVelocity.Parent = hrp
            
            local bodyGyro = Instance.new("BodyGyro")
            bodyGyro.Name = "FlyGyro"
            bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
            bodyGyro.P = 1000000
            bodyGyro.Parent = hrp
            
            RunService:BindToRenderStep("Fly", 1, function()
                if Settings.Fly then
                    local moveDir = Vector3.new(0, 0, 0)
                    
                    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                        moveDir = moveDir + workspace.CurrentCamera.CFrame.LookVector
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                        moveDir = moveDir - workspace.CurrentCamera.CFrame.LookVector
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                        moveDir = moveDir - workspace.CurrentCamera.CFrame.RightVector
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                        moveDir = moveDir + workspace.CurrentCamera.CFrame.RightVector
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                        moveDir = moveDir + Vector3.new(0, 1, 0)
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
                        moveDir = moveDir - Vector3.new(0, 1, 0)
                    end
                    
                    bodyVelocity.Velocity = moveDir * Settings.FlySpeed
                    bodyGyro.CFrame = workspace.CurrentCamera.CFrame
                end
            end)
        end
    else
        RunService:UnbindFromRenderStep("Fly")
        local character = LocalPlayer.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            local hrp = character.HumanoidRootPart
            if hrp:FindFirstChild("FlyVelocity") then
                hrp.FlyVelocity:Destroy()
            end
            if hrp:FindFirstChild("FlyGyro") then
                hrp.FlyGyro:Destroy()
            end
        end
    end
end)

CreateSlider(MainTab, "Fly Speed", 10, 200, 50, function(value)
    Settings.FlySpeed = value
end)

-- INFINITE JUMP
CreateToggle(MainTab, "Infinite Jump", false, function(enabled)
    Settings.InfiniteJump = enabled
end)

-- WALK SPEED
CreateSlider(MainTab, "Walk Speed", 16, 200, 16, function(value)
    Settings.WalkSpeed = value
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = value
    end
end)

-- JUMP POWER
CreateSlider(MainTab, "Jump Power", 50, 300, 50, function(value)
    Settings.JumpPower = value
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.UseJumpPower = true
        LocalPlayer.Character.Humanoid.JumpPower = value
    end
end)

CreateSection(MainTab, "Character")

CreateToggle(MainTab, "God Mode", false, function(enabled)
    Settings.GodMode = enabled
    
    local character = LocalPlayer.Character
    if character then
        local humanoid = character:FindFirstChild("Humanoid")
        if humanoid then
            if enabled then
                humanoid.MaxHealth = math.huge
                humanoid.Health = math.huge
            else
                humanoid.MaxHealth = 100
                humanoid.Health = 100
            end
        end
    end
end)

CreateToggle(MainTab, "Noclip", false, function(enabled)
    Settings.Noclip = enabled
    
    if enabled then
        RunService:BindToRenderStep("Noclip", 1, function()
            if Settings.Noclip then
                local character = LocalPlayer.Character
                if character then
                    for _, part in pairs(character:GetDescendants()) do
                        if part:IsA("BasePart") then
                            part.CanCollide = false
                        end
                    end
                end
            end
        end)
    else
        RunService:UnbindFromRenderStep("Noclip")
        local character = LocalPlayer.Character
        if character then
            for _, part in pairs(character:GetDescendants()) do
                if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                    part.CanCollide = true
                end
            end
        end
    end
end)

-- ==================== MISC TAB ====================
CreateSection(MiscTab, "ESP Settings")

CreateToggle(MiscTab, "ESP Enabled", false, function(enabled)
    Settings.ESP = enabled
    
    if enabled then
        -- Create ESP for all players
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                local function createESP()
                    local character = player.Character
                    if character and character:FindFirstChild("HumanoidRootPart") then
                        -- Remove old ESP
                        if character:FindFirstChild("ESPBillboard") then
                            character.ESPBillboard:Destroy()
                        end
                        
                        local billboard = Instance.new("BillboardGui")
                        billboard.Name = "ESPBillboard"
                        billboard.Adornee = character.HumanoidRootPart
                        billboard.Size = UDim2.new(0, 100, 0, 50)
                        billboard.StudsOffset = Vector3.new(0, 3, 0)
                        billboard.AlwaysOnTop = true
                        billboard.Parent = character
                        
                        if Settings.ESPName then
                            local nameLabel = Instance.new("TextLabel")
                            nameLabel.Name = "NameLabel"
                            nameLabel.Parent = billboard
                            nameLabel.BackgroundTransparency = 1
                            nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
                            nameLabel.Font = Enum.Font.GothamBold
                            nameLabel.Text = player.Name
                            nameLabel.TextColor3 = Settings.ESPColor
                            nameLabel.TextSize = 14
                            nameLabel.TextStrokeTransparency = 0.5
                        end
                        
                        if Settings.ESPHealth and character:FindFirstChild("Humanoid") then
                            local healthLabel = Instance.new("TextLabel")
                            healthLabel.Name = "HealthLabel"
                            healthLabel.Parent = billboard
                            healthLabel.BackgroundTransparency = 1
                            healthLabel.Position = UDim2.new(0, 0, 0.5, 0)
                            healthLabel.Size = UDim2.new(1, 0, 0.5, 0)
                            healthLabel.Font = Enum.Font.Gotham
                            healthLabel.Text = "HP: " .. math.floor(character.Humanoid.Health)
                            healthLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
                            healthLabel.TextSize = 12
                            healthLabel.TextStrokeTransparency = 0.5
                            
                            character.Humanoid.HealthChanged:Connect(function(health)
                                healthLabel.Text = "HP: " .. math.floor(health)
                            end)
                        end
                        
                        if Settings.ESPBox then
                            local highlight = Instance.new("Highlight")
                            highlight.Name = "ESPHighlight"
                            highlight.Adornee = character
                            highlight.FillColor = Settings.ESPColor
                            highlight.OutlineColor = Settings.ESPColor
                            highlight.FillTransparency = 0.7
                            highlight.OutlineTransparency = 0
                            highlight.Parent = character
                        end
                    end
                end
                
                createESP()
                player.CharacterAdded:Connect(function()
                    task.wait(1)
                    if Settings.ESP then
                        createESP()
                    end
                end)
            end
        end
    else
        -- Remove all ESP
        for _, player in pairs(Players:GetPlayers()) do
            if player.Character then
                if player.Character:FindFirstChild("ESPBillboard") then
                    player.Character.ESPBillboard:Destroy()
                end
                if player.Character:FindFirstChild("ESPHighlight") then
                    player.Character.ESPHighlight:Destroy()
                end
            end
        end
    end
end)

CreateColorPicker(MiscTab, "ESP Color", Settings.ESPColor)

CreateToggle(MiscTab, "ESP Box", true, function(enabled)
    Settings.ESPBox = enabled
end)

CreateToggle(MiscTab, "ESP Name", true, function(enabled)
    Settings.ESPName = enabled
end)

CreateToggle(MiscTab, "ESP Health", true, function(enabled)
    Settings.ESPHealth = enabled
end)

CreateSection(MiscTab, "Aimbot Settings")

CreateToggle(MiscTab, "Aimbot (Right Mouse)", false, function(enabled)
    Settings.Aimbot = enabled
end)

CreateToggle(MiscTab, "Show FOV Circle", false, function(enabled)
    Settings.ShowFOV = enabled
    FOVCircle.Visible = enabled
end)

CreateSlider(MiscTab, "Aimbot FOV", 50, 500, 100, function(value)
    Settings.AimbotFOV = value
    FOVCircle.Size = UDim2.new(0, value * 2, 0, value * 2)
end)

CreateSlider(MiscTab, "Aimbot Smoothness", 1, 10, 5, function(value)
    Settings.AimbotSmooth = value / 10
end)

-- ==================== SETTINGS TAB ====================
CreateSection(SettingsTab, "UI Settings")

CreateSlider(SettingsTab, "Font Size", 10, 20, 14, function(value)
    Settings.FontSize = value
end)

CreateDropdown(SettingsTab, "Theme", {"Purple", "Red", "Blue", "Green"}, "Purple", function(theme)
    Settings.Theme = theme
    CurrentTheme = Themes[theme]
    
    -- Update colors
    MainFrame.BackgroundColor3 = CurrentTheme.Background
    MainStroke.Color = CurrentTheme.Primary
    TitleBar.BackgroundColor3 = CurrentTheme.BackgroundSecondary
    TitleFix.BackgroundColor3 = CurrentTheme.BackgroundSecondary
    TitleText.TextColor3 = CurrentTheme.Primary
    TabContainer.BackgroundColor3 = CurrentTheme.BackgroundSecondary
    FOVCircle.ImageColor3 = CurrentTheme.Primary
    
    CreateNotification("Theme changed to " .. theme, 2)
end)

CreateKeybind(SettingsTab, "Toggle Hub Key", Enum.KeyCode.RightControl, function(key)
    toggleKey = key
end)

CreateSection(SettingsTab, "Credits")

CreateLabel(SettingsTab, "💜 Created by Rocha")
CreateLabel(SettingsTab, "🎮 Version 1.1.2")
CreateLabel(SettingsTab, "📅 " .. os.date("%Y"))

-- Player Info Section
CreateSection(SettingsTab, "Player Info")

local PlayerInfoFrame = Instance.new("Frame")
PlayerInfoFrame.Name = "PlayerInfo"
PlayerInfoFrame.Parent = SettingsTab
PlayerInfoFrame.BackgroundColor3 = CurrentTheme.BackgroundSecondary
PlayerInfoFrame.Size = UDim2.new(1, 0, 0, 80)
PlayerInfoFrame.BorderSizePixel = 0

local PICorner = Instance.new("UICorner")
PICorner.CornerRadius = UDim.new(0, 6)
PICorner.Parent = PlayerInfoFrame

local PlayerImage = Instance.new("ImageLabel")
PlayerImage.Name = "Avatar"
PlayerImage.Parent = PlayerInfoFrame
PlayerImage.BackgroundColor3 = CurrentTheme.Background
PlayerImage.Position = UDim2.new(0, 10, 0.5, -25)
PlayerImage.Size = UDim2.new(0, 50, 0, 50)
PlayerImage.Image = Players:GetUserThumbnailAsync(LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size150x150)

local AvatarCorner = Instance.new("UICorner")
AvatarCorner.CornerRadius = UDim.new(1, 0)
AvatarCorner.Parent = PlayerImage

local PlayerName = Instance.new("TextLabel")
PlayerName.Parent = PlayerInfoFrame
PlayerName.BackgroundTransparency = 1
PlayerName.Position = UDim2.new(0, 70, 0, 10)
PlayerName.Size = UDim2.new(1, -80, 0, 20)
PlayerName.Font = Enum.Font.GothamBold
PlayerName.Text = LocalPlayer.Name
PlayerName.TextColor3 = CurrentTheme.Text
PlayerName.TextSize = 14
PlayerName.TextXAlignment = Enum.TextXAlignment.Left

local PlayerId = Instance.new("TextLabel")
PlayerId.Parent = PlayerInfoFrame
PlayerId.BackgroundTransparency = 1
PlayerId.Position = UDim2.new(0, 70, 0, 30)
PlayerId.Size = UDim2.new(1, -80, 0, 15)
PlayerId.Font = Enum.Font.Gotham
PlayerId.Text = "ID: " .. LocalPlayer.UserId
PlayerId.TextColor3 = CurrentTheme.TextSecondary
PlayerId.TextSize = 11
PlayerId.TextXAlignment = Enum.TextXAlignment.Left

local PlayerCreated = Instance.new("TextLabel")
PlayerCreated.Parent = PlayerInfoFrame
PlayerCreated.BackgroundTransparency = 1
PlayerCreated.Position = UDim2.new(0, 70, 0, 45)
PlayerCreated.Size = UDim2.new(1, -80, 0, 15)
PlayerCreated.Font = Enum.Font.Gotham
PlayerCreated.Text = "Account Age: " .. LocalPlayer.AccountAge .. " days"
PlayerCreated.TextColor3 = CurrentTheme.TextSecondary
PlayerCreated.TextSize = 11
PlayerCreated.TextXAlignment = Enum.TextXAlignment.Left

-- Draggable
local dragging = false
local dragStart, startPos

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)

TitleBar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Toggle Hub
local function ToggleHub()
    isOpen = not isOpen
    PlayClickSound()
    
    if isOpen then
        MainFrame.Visible = true
        SetBlur(true)
        MainFrame.Size = UDim2.new(0, 0, 0, 0)
        CreateTween(MainFrame, {Size = UDim2.new(0, 600, 0, 450)}, 0.5, Enum.EasingStyle.Back):Play()
        FOVCircle.Visible = Settings.ShowFOV
    else
        SetBlur(false)
        FOVCircle.Visible = false
        local closeTween = CreateTween(MainFrame, {Size = UDim2.new(0, 0, 0, 0)}, 0.3)
        closeTween:Play()
        closeTween.Completed:Connect(function()
            MainFrame.Visible = false
        end)
        CreateNotification("Press " .. toggleKey.Name .. " to open", 3)
    end
end

-- Close Button
CloseButton.MouseButton1Click:Connect(function()
    ToggleHub()
end)

-- Minimize Button
MinButton.MouseButton1Click:Connect(function()
    ToggleHub()
end)

-- Keybind to toggle
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == toggleKey then
        ToggleHub()
    end
end)

-- ==================== GAME LOGIC LOOPS ====================

-- Infinite Jump Logic
UserInputService.JumpRequest:Connect(function()
    if Settings.InfiniteJump and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid:ChangeState("Jumping")
    end
end)

-- Logic Loop (Aimbot, FOV Circle, Stats Persistence)
RunService.RenderStepped:Connect(function()
    -- Update FOV Circle Position
    if Settings.ShowFOV then
        local mouseLoc = UserInputService:GetMouseLocation()
        FOVCircle.Position = UDim2.new(0, mouseLoc.X, 0, mouseLoc.Y)
    end

    -- Persistent Speed/Jump
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        if LocalPlayer.Character.Humanoid.WalkSpeed ~= Settings.WalkSpeed then
            LocalPlayer.Character.Humanoid.WalkSpeed = Settings.WalkSpeed
        end
        if LocalPlayer.Character.Humanoid.JumpPower ~= Settings.JumpPower then
            LocalPlayer.Character.Humanoid.UseJumpPower = true
            LocalPlayer.Character.Humanoid.JumpPower = Settings.JumpPower
        end
    end

    -- Aimbot Logic (RIGHT MOUSE BUTTON)
    if Settings.Aimbot and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = nil
        local shortestDistance = Settings.AimbotFOV
        local mouseLoc = UserInputService:GetMouseLocation()

        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                local screenPoint, onScreen = Camera:WorldToViewportPoint(player.Character.Head.Position)
                
                if onScreen then
                    local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - mouseLoc).Magnitude
                    if distance < shortestDistance then
                        target = player.Character.Head
                        shortestDistance = distance
                    end
                end
            end
        end

        if target then
            local currentCFrame = Camera.CFrame
            local targetCFrame = CFrame.new(currentCFrame.Position, target.Position)
            -- Apply Smoothness
            Camera.CFrame = currentCFrame:Lerp(targetCFrame, Settings.AimbotSmooth)
        end
    end
end)

-- Initialize
SetBlur(true)
MainFrame.Size = UDim2.new(0, 0, 0, 0)
task.wait(0.3)
CreateTween(MainFrame, {Size = UDim2.new(0, 600, 0, 450)}, 0.5, Enum.EasingStyle.Back):Play()

CreateNotification("🎮 InfinityCore Hub loaded successfully!", 3)

print([[
╔═══════════════════════════════════════╗
║     INFINITYCORE HUB - CS STYLE       ║
║          Created by Rocha             ║
╚═══════════════════════════════════════╝
]])
