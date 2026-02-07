local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local LocalPlayer = game:GetService("Players").LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local HttpService = game:GetService("HttpService")

local NovaLib = {
    Themes = {
        Dark = {
            Main = Color3.fromRGB(30, 30, 30),
            Secondary = Color3.fromRGB(40, 40, 40),
            Tertiary = Color3.fromRGB(50, 50, 50),
            Text = Color3.fromRGB(255, 255, 255),
            Accent = Color3.fromRGB(100, 100, 255), -- Default accent
            Placeholder = Color3.fromRGB(150, 150, 150)
        },
        Darker = {
            Main = Color3.fromRGB(20, 20, 20),
            Secondary = Color3.fromRGB(30, 30, 30),
            Tertiary = Color3.fromRGB(40, 40, 40),
            Text = Color3.fromRGB(200, 200, 200),
            Accent = Color3.fromRGB(80, 80, 200),
            Placeholder = Color3.fromRGB(120, 120, 120)
        },
        Purple = {
            Main = Color3.fromRGB(35, 25, 45),
            Secondary = Color3.fromRGB(45, 35, 55),
            Tertiary = Color3.fromRGB(55, 45, 65),
            Text = Color3.fromRGB(255, 230, 255),
            Accent = Color3.fromRGB(180, 80, 255),
            Placeholder = Color3.fromRGB(180, 150, 180)
        }
    },
    CurrentTheme = "Dark"
}

-- Utility Functions
local function MakeDraggable(topbarobject, object)
    local Dragging = nil
    local DragInput = nil
    local DragStart = nil
    local StartPosition = nil

    local function Update(input)
        local Delta = input.Position - DragStart
        local pos = UDim2.new(StartPosition.X.Scale, StartPosition.X.Offset + Delta.X, StartPosition.Y.Scale, StartPosition.Y.Offset + Delta.Y)
        object.Position = pos
    end

    topbarobject.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            Dragging = true
            DragStart = input.Position
            StartPosition = object.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    Dragging = false
                end
            end)
        end
    end)

    topbarobject.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            DragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == DragInput and Dragging then
            Update(input)
        end
    end)
end

function NovaLib:SetTheme(themeName)
    if self.Themes[themeName] then
        self.CurrentTheme = themeName
    end
end

function NovaLib:MakeWindow(options)
    local Title = options.Title or "Nova Hub"
    local SubTitle = options.SubTitle or ""
    local SaveFolder = options.SaveFolder or "NovaLib"

    -- Create ScreenGui
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "NovaLibUI"
    ScreenGui.ResetOnSpawn = false
    
    if pcall(function() ScreenGui.Parent = game:GetService("CoreGui") end) then
    else
        ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    end

    -- Main Window Frame
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 550, 0, 350)
    MainFrame.Position = UDim2.new(0.5, -275, 0.5, -175)
    MainFrame.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Main
    MainFrame.BorderSizePixel = 0
    MainFrame.Parent = ScreenGui

    local MainCorner = Instance.new("UICorner")
    MainCorner.CornerRadius = UDim.new(0, 10)
    MainCorner.Parent = MainFrame

    -- TopBar
    local TopBar = Instance.new("Frame")
    TopBar.Name = "TopBar"
    TopBar.Size = UDim2.new(1, 0, 0, 40)
    TopBar.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    TopBar.BackgroundTransparency = 1
    TopBar.Parent = MainFrame

    local TitleLabel = Instance.new("TextLabel")
    TitleLabel.Name = "Title"
    TitleLabel.Text = Title .. " : " .. SubTitle
    TitleLabel.Size = UDim2.new(1, -50, 1, 0)
    TitleLabel.Position = UDim2.new(0, 15, 0, 0)
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Text
    TitleLabel.TextSize = 18
    TitleLabel.Font = Enum.Font.GothamBold
    TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    TitleLabel.Parent = TopBar

    MakeDraggable(TopBar, MainFrame)

    -- Container for Tabs and Content
    local Container = Instance.new("Frame")
    Container.Name = "Container"
    Container.Size = UDim2.new(1, 0, 1, -40)
    Container.Position = UDim2.new(0, 0, 0, 40)
    Container.BackgroundTransparency = 1
    Container.Parent = MainFrame

    -- Sidebar (Tab Buttons)
    local Sidebar = Instance.new("ScrollingFrame")
    Sidebar.Name = "Sidebar"
    Sidebar.Size = UDim2.new(0, 150, 1, -10)
    Sidebar.Position = UDim2.new(0, 10, 0, 0)
    Sidebar.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Secondary
    Sidebar.BackgroundTransparency = 0.5
    Sidebar.BorderSizePixel = 0
    Sidebar.ScrollBarThickness = 0
    Sidebar.Parent = Container

    local SidebarCorner = Instance.new("UICorner")
    SidebarCorner.CornerRadius = UDim.new(0, 8)
    SidebarCorner.Parent = Sidebar
    
    local SidebarLayout = Instance.new("UIListLayout")
    SidebarLayout.Padding = UDim.new(0, 5)
    SidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    SidebarLayout.SortOrder = Enum.SortOrder.LayoutOrder
    SidebarLayout.Parent = Sidebar
    
    local SidebarPadding = Instance.new("UIPadding")
    SidebarPadding.PaddingTop = UDim.new(0, 10)
    SidebarPadding.PaddingBottom = UDim.new(0, 10)
    SidebarPadding.Parent = Sidebar

    -- Content Area (Where elements go)
    local ContentArea = Instance.new("Frame")
    ContentArea.Name = "ContentArea"
    ContentArea.Size = UDim2.new(1, -170, 1, -10)
    ContentArea.Position = UDim2.new(0, 165, 0, 0)
    ContentArea.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Secondary
    ContentArea.BackgroundTransparency = 0.5
    ContentArea.Parent = Container

    local ContentCorner = Instance.new("UICorner")
    ContentCorner.CornerRadius = UDim.new(0, 8)
    ContentCorner.Parent = ContentArea

    -- Window Object
    local Window = {}
    local Tabs = {}
    local FirstTab = nil

    function Window:AddMinimizeButton(config)
        local ButtonConfig = config.Button or {}
        local CornerConfig = config.Corner or {}
        
        local MinBtn = Instance.new("ImageButton")
        MinBtn.Name = "MinimizeButton"
        MinBtn.Size = UDim2.new(0, 30, 0, 30)
        MinBtn.Position = UDim2.new(1, -40, 0, 5)
        MinBtn.Image = ButtonConfig.Image or "rbxassetid://71014873973869"
        MinBtn.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Secondary
        MinBtn.BackgroundTransparency = ButtonConfig.BackgroundTransparency or 0
        MinBtn.Parent = TopBar
        
        local MinCorner = Instance.new("UICorner")
        MinCorner.CornerRadius = CornerConfig.CornerRadius or UDim.new(0, 6)
        MinCorner.Parent = MinBtn

        local Open = true
        MinBtn.MouseButton1Click:Connect(function()
            Open = not Open
            local targetSize = Open and UDim2.new(0, 550, 0, 350) or UDim2.new(0, 550, 0, 40)
            TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = targetSize}):Play()
            Container.Visible = Open
        end)
    end

    function Window:SelectTab(Tab)
        for _, t in pairs(Tabs) do
            t.Container.Visible = false
            t.Button.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Secondary
            t.Button.Transparency = 1
        end
        Tab.Container.Visible = true
        Tab.Button.Transparency = 0
        Tab.Button.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Accent
    end

    function Window:MakeTab(options)
        local TabName = options[1] or "Tab"
        local TabIcon = options[2] or ""

        -- Tab Button in Sidebar
        local TabBtn = Instance.new("TextButton")
        TabBtn.Name = TabName .. "Btn"
        TabBtn.Size = UDim2.new(0.9, 0, 0, 35)
        TabBtn.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Secondary
        TabBtn.BackgroundTransparency = 1
        TabBtn.Text = TabName
        TabBtn.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Text
        TabBtn.Font = Enum.Font.GothamMedium
        TabBtn.TextSize = 14
        TabBtn.Parent = Sidebar
        
        local TabBtnCorner = Instance.new("UICorner")
        TabBtnCorner.CornerRadius = UDim.new(0, 6)
        TabBtnCorner.Parent = TabBtn

        -- Tab Content Container
        local TabContainer = Instance.new("ScrollingFrame")
        TabContainer.Name = TabName .. "Container"
        TabContainer.Size = UDim2.new(1, -20, 1, -20)
        TabContainer.Position = UDim2.new(0, 10, 0, 10)
        TabContainer.BackgroundTransparency = 1
        TabContainer.ScrollBarThickness = 2
        TabContainer.Visible = false
        TabContainer.Parent = ContentArea
        
        local TabLayout = Instance.new("UIListLayout")
        TabLayout.Padding = UDim.new(0, 10)
        TabLayout.SortOrder = Enum.SortOrder.LayoutOrder
        TabLayout.Parent = TabContainer

        local TabObj = {
            Button = TabBtn,
            Container = TabContainer
        }
        
        table.insert(Tabs, TabObj)
        
        TabBtn.MouseButton1Click:Connect(function()
            Window:SelectTab(TabObj)
        end)

        if #Tabs == 1 then
            FirstTab = TabObj
            Window:SelectTab(TabObj)
        end
        
        -- Tab Functions
        function TabObj:AddSection(options)
            local SectionName = options[1] or "Section"
            local SectionLabel = Instance.new("TextLabel")
            SectionLabel.Text = SectionName
            SectionLabel.Size = UDim2.new(1, 0, 0, 25)
            SectionLabel.BackgroundTransparency = 1
            SectionLabel.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Placeholder
            SectionLabel.Font = Enum.Font.GothamBold
            SectionLabel.TextSize = 14
            SectionLabel.TextXAlignment = Enum.TextXAlignment.Left
            SectionLabel.Parent = TabContainer
            return SectionLabel
        end

        function TabObj:AddParagraph(options)
            local Title = options[1] or "Title"
            local Text = options[2] or "Text"
            
            local ParaFrame = Instance.new("Frame")
            ParaFrame.Size = UDim2.new(1, 0, 0, 0) -- Auto resize
            ParaFrame.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Tertiary
            ParaFrame.Parent = TabContainer
            
            local ParaCorner = Instance.new("UICorner")
            ParaCorner.CornerRadius = UDim.new(0, 6)
            ParaCorner.Parent = ParaFrame
            
            local ParaTitle = Instance.new("TextLabel")
            ParaTitle.Text = Title
            ParaTitle.Size = UDim2.new(1, -20, 0, 25)
            ParaTitle.Position = UDim2.new(0, 10, 0, 5)
            ParaTitle.BackgroundTransparency = 1
            ParaTitle.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Text
            ParaTitle.Font = Enum.Font.GothamBold
            ParaTitle.TextSize = 14
            ParaTitle.TextXAlignment = Enum.TextXAlignment.Left
            ParaTitle.Parent = ParaFrame
            
            local ParaText = Instance.new("TextLabel")
            ParaText.Text = Text
            ParaText.Size = UDim2.new(1, -20, 0, 0)
            ParaText.Position = UDim2.new(0, 10, 0, 30)
            ParaText.BackgroundTransparency = 1
            ParaText.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Placeholder
            ParaText.Font = Enum.Font.Gotham
            ParaText.TextSize = 12
            ParaText.TextXAlignment = Enum.TextXAlignment.Left
            ParaText.TextWrapped = true
            ParaText.AutomaticSize = Enum.AutomaticSize.Y
            ParaText.Parent = ParaFrame
            
            ParaFrame.AutomaticSize = Enum.AutomaticSize.Y
            
            local Pad = Instance.new("UIPadding")
            Pad.PaddingBottom = UDim.new(0, 10)
            Pad.Parent = ParaFrame
        end

        function TabObj:AddButton(options)
            local Text = options[1] or "Button"
            local Callback = options[2] or function() end
            
            local Btn = Instance.new("TextButton")
            Btn.Size = UDim2.new(1, 0, 0, 40)
            Btn.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Tertiary
            Btn.Text = Text
            Btn.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Text
            Btn.Font = Enum.Font.GothamMedium
            Btn.TextSize = 14
            Btn.Parent = TabContainer
            
            local BtnCorner = Instance.new("UICorner")
            BtnCorner.CornerRadius = UDim.new(0, 6)
            BtnCorner.Parent = Btn
            
            Btn.MouseButton1Click:Connect(function()
                pcall(Callback)
                local tween = TweenService:Create(Btn, TweenInfo.new(0.1), {BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Accent})
                tween:Play()
                wait(0.1)
                tween = TweenService:Create(Btn, TweenInfo.new(0.1), {BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Tertiary})
                tween:Play()
            end)
        end
        
        function TabObj:AddToggle(options)
            local Name = options.Name or "Toggle"
            local Description = options.Description or ""
            local Default = options.Default or false
            local Callback = options.Callback or function() end
            
            local ToggleFrame = Instance.new("Frame")
            ToggleFrame.Size = UDim2.new(1, 0, 0, 45)
            ToggleFrame.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Tertiary
            ToggleFrame.Parent = TabContainer
            
            local ToggleCorner = Instance.new("UICorner")
            ToggleCorner.CornerRadius = UDim.new(0, 6)
            ToggleCorner.Parent = ToggleFrame
            
            local ToggleTitle = Instance.new("TextLabel")
            ToggleTitle.Text = Name
            ToggleTitle.Size = UDim2.new(1, -70, 0, 25)
            ToggleTitle.Position = UDim2.new(0, 10, 0, 5)
            ToggleTitle.BackgroundTransparency = 1
            ToggleTitle.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Text
            ToggleTitle.Font = Enum.Font.GothamMedium
            ToggleTitle.TextSize = 14
            ToggleTitle.TextXAlignment = Enum.TextXAlignment.Left
            ToggleTitle.RichText = true
            ToggleTitle.Parent = ToggleFrame

            if Description ~= "" then
                local ToggleDesc = Instance.new("TextLabel")
                ToggleDesc.Text = Description
                ToggleDesc.Size = UDim2.new(1, -70, 0, 15)
                ToggleDesc.Position = UDim2.new(0, 10, 0, 25)
                ToggleDesc.BackgroundTransparency = 1
                ToggleDesc.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Placeholder
                ToggleDesc.Font = Enum.Font.Gotham
                ToggleDesc.TextSize = 12
                ToggleDesc.TextXAlignment = Enum.TextXAlignment.Left
                ToggleDesc.RichText = true
                ToggleDesc.Parent = ToggleFrame
            end
            
            local Switch = Instance.new("TextButton")
            Switch.Text = ""
            Switch.Size = UDim2.new(0, 50, 0, 25)
            Switch.Position = UDim2.new(1, -60, 0, 10)
            Switch.BackgroundColor3 = Default and NovaLib.Themes[NovaLib.CurrentTheme].Accent or Color3.fromRGB(80, 80, 80)
            Switch.Parent = ToggleFrame
            
            local SwitchCorner = Instance.new("UICorner")
            SwitchCorner.CornerRadius = UDim.new(1, 0)
            SwitchCorner.Parent = Switch
            
            local Circle = Instance.new("Frame")
            Circle.Size = UDim2.new(0, 21, 0, 21)
            Circle.Position = Default and UDim2.new(1, -23, 0, 2) or UDim2.new(0, 2, 0, 2)
            Circle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            Circle.Parent = Switch
            
            local CircleCorner = Instance.new("UICorner")
            CircleCorner.CornerRadius = UDim.new(1, 0)
            CircleCorner.Parent = Circle
            
            local Toggled = Default
            
            local function Toggle(value)
                Toggled = value
                
                TweenService:Create(Switch, TweenInfo.new(0.2), {
                    BackgroundColor3 = Toggled and NovaLib.Themes[NovaLib.CurrentTheme].Accent or Color3.fromRGB(80, 80, 80)
                }):Play()
                
                TweenService:Create(Circle, TweenInfo.new(0.2), {
                    Position = Toggled and UDim2.new(1, -23, 0, 2) or UDim2.new(0, 2, 0, 2)
                }):Play()
                
                pcall(Callback, Toggled)
            end
            
            Switch.MouseButton1Click:Connect(function()
                Toggle(not Toggled)
            end)
            
            local ToggleObj = {}
            function ToggleObj:Callback(func)
                Callback = func
            end
            return ToggleObj
        end
        
        function TabObj:AddSlider(options)
            local Name = options.Name or "Slider"
            local Min = options.Min or 0
            local Max = options.Max or 100
            local Default = options.Default or Min
            local Callback = options.Callback or function() end
            
            local SliderFrame = Instance.new("Frame")
            SliderFrame.Size = UDim2.new(1, 0, 0, 55)
            SliderFrame.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Tertiary
            SliderFrame.Parent = TabContainer
            
            local SliderCorner = Instance.new("UICorner")
            SliderCorner.CornerRadius = UDim.new(0, 6)
            SliderCorner.Parent = SliderFrame
            
            local Title = Instance.new("TextLabel")
            Title.Text = Name
            Title.Size = UDim2.new(1, -20, 0, 25)
            Title.Position = UDim2.new(0, 10, 0, 0)
            Title.BackgroundTransparency = 1
            Title.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Text
            Title.Font = Enum.Font.GothamMedium
            Title.TextSize = 14
            Title.TextXAlignment = Enum.TextXAlignment.Left
            Title.Parent = SliderFrame
            
            local ValueLabel = Instance.new("TextLabel")
            ValueLabel.Text = tostring(Default)
            ValueLabel.Size = UDim2.new(0, 50, 0, 25)
            ValueLabel.Position = UDim2.new(1, -60, 0, 0)
            ValueLabel.BackgroundTransparency = 1
            ValueLabel.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Placeholder
            ValueLabel.Font = Enum.Font.Gotham
            ValueLabel.TextSize = 12
            ValueLabel.TextXAlignment = Enum.TextXAlignment.Right
            ValueLabel.Parent = SliderFrame
            
            local SliderBar = Instance.new("Frame")
            SliderBar.Size = UDim2.new(1, -20, 0, 6)
            SliderBar.Position = UDim2.new(0, 10, 0, 35)
            SliderBar.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            SliderBar.Parent = SliderFrame
            
            local SliderBarCorner = Instance.new("UICorner")
            SliderBarCorner.CornerRadius = UDim.new(1, 0)
            SliderBarCorner.Parent = SliderBar
            
            local Fill = Instance.new("Frame")
            Fill.Size = UDim2.new((Default - Min) / (Max - Min), 0, 1, 0)
            Fill.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Accent
            Fill.Parent = SliderBar
            
            local FillCorner = Instance.new("UICorner")
            FillCorner.CornerRadius = UDim.new(1, 0)
            FillCorner.Parent = Fill
            
            local Trigger = Instance.new("TextButton")
            Trigger.Size = UDim2.new(1, 0, 1, 0)
            Trigger.BackgroundTransparency = 1
            Trigger.Text = ""
            Trigger.Parent = SliderBar
            
            local isDragging = false
            
            local function UpdateSlider(input)
                local pos = UDim2.new(math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1), 0, 1, 0)
                Fill.Size = pos
                
                local val = math.floor(Min + ((Max - Min) * pos.X.Scale))
                ValueLabel.Text = tostring(val)
                pcall(Callback, val)
            end
            
            Trigger.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                    isDragging = true
                    UpdateSlider(input)
                end
            end)
            
            UserInputService.InputChanged:Connect(function(input)
                if isDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                    UpdateSlider(input)
                end
            end)
            
            UserInputService.InputEnded:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                    isDragging = false
                end
            end)
        end
        
        function TabObj:AddDropdown(options)
            local Name = options.Name or "Dropdown"
            local Options = options.Options or {}
            local Default = options.Default or Options[1]
            local Callback = options.Callback or function() end
            
            local DropFrame = Instance.new("Frame")
            DropFrame.Size = UDim2.new(1, 0, 0, 45)
            DropFrame.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Tertiary
            DropFrame.Parent = TabContainer
            DropFrame.ClipsDescendants = true
            
            local DropCorner = Instance.new("UICorner")
            DropCorner.CornerRadius = UDim.new(0, 6)
            DropCorner.Parent = DropFrame
            
            local Title = Instance.new("TextLabel")
            Title.Text = Name
            Title.Size = UDim2.new(1, -20, 0, 45)
            Title.Position = UDim2.new(0, 10, 0, 0)
            Title.BackgroundTransparency = 1
            Title.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Text
            Title.Font = Enum.Font.GothamMedium
            Title.TextSize = 14
            Title.TextXAlignment = Enum.TextXAlignment.Left
            Title.RichText = true
            Title.Parent = DropFrame
            
            local Selected = Instance.new("TextLabel")
            Selected.Text = Default
            Selected.Size = UDim2.new(0, 100, 0, 45)
            Selected.Position = UDim2.new(1, -140, 0, 0)
            Selected.BackgroundTransparency = 1
            Selected.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Placeholder
            Selected.Font = Enum.Font.Gotham
            Selected.TextSize = 12
            Selected.TextXAlignment = Enum.TextXAlignment.Right
            Selected.Parent = DropFrame
            
            local Arrow = Instance.new("ImageLabel")
            Arrow.Image = "rbxassetid://6034818372"
            Arrow.Size = UDim2.new(0, 20, 0, 20)
            Arrow.Position = UDim2.new(1, -30, 0, 12)
            Arrow.BackgroundTransparency = 1
            Arrow.Parent = DropFrame
            
            local OpenBtn = Instance.new("TextButton")
            OpenBtn.Size = UDim2.new(1, 0, 0, 45)
            OpenBtn.BackgroundTransparency = 1
            OpenBtn.Text = ""
            OpenBtn.Parent = DropFrame
            
            local ItemList = Instance.new("ScrollingFrame")
            ItemList.Size = UDim2.new(1, -20, 0, 0)
            ItemList.Position = UDim2.new(0, 10, 0, 50)
            ItemList.BackgroundTransparency = 1
            ItemList.ScrollBarThickness = 2
            ItemList.Parent = DropFrame
            
            local ListLayout = Instance.new("UIListLayout")
            ListLayout.Padding = UDim.new(0, 5)
            ListLayout.Parent = ItemList
            
            local isOpen = false
            
            local function RefreshOptions()
                for _, child in pairs(ItemList:GetChildren()) do
                    if child:IsA("TextButton") then child:Destroy() end
                end
                
                for _, opt in pairs(Options) do
                    local Item = Instance.new("TextButton")
                    Item.Size = UDim2.new(1, 0, 0, 30)
                    Item.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Secondary
                    Item.Text = opt
                    Item.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Placeholder
                    Item.Font = Enum.Font.Gotham
                    Item.TextSize = 13
                    Item.Parent = ItemList
                    
                    local ItemCorner = Instance.new("UICorner")
                    ItemCorner.CornerRadius = UDim.new(0, 4)
                    ItemCorner.Parent = Item
                    
                    Item.MouseButton1Click:Connect(function()
                        Selected.Text = opt
                        pcall(Callback, opt)
                        isOpen = false
                        TweenService:Create(DropFrame, TweenInfo.new(0.3), {Size = UDim2.new(1, 0, 0, 45)}):Play()
                        TweenService:Create(Arrow, TweenInfo.new(0.3), {Rotation = 0}):Play()
                    end)
                end
                
                ItemList.CanvasSize = UDim2.new(0, 0, 0, ListLayout.AbsoluteContentSize.Y)
            end
            
            RefreshOptions()
            
            OpenBtn.MouseButton1Click:Connect(function()
                isOpen = not isOpen
                local targetHeight = isOpen and math.min(200, 50 + ItemList.CanvasSize.Y.Offset + 10) or 45
                ItemList.Size = UDim2.new(1, -20, 0, targetHeight - 55)
                TweenService:Create(DropFrame, TweenInfo.new(0.3), {Size = UDim2.new(1, 0, 0, targetHeight)}):Play()
                TweenService:Create(Arrow, TweenInfo.new(0.3), {Rotation = isOpen and 180 or 0}):Play()
            end)
        end
        
        function TabObj:AddTextBox(options)
            local Name = options.Name or "TextBox"
            local Placeholder = options.PlaceholderText or "Input..."
            local Callback = options.Callback or function() end
            
            local BoxFrame = Instance.new("Frame")
            BoxFrame.Size = UDim2.new(1, 0, 0, 45)
            BoxFrame.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Tertiary
            BoxFrame.Parent = TabContainer
            
            local BoxCorner = Instance.new("UICorner")
            BoxCorner.CornerRadius = UDim.new(0, 6)
            BoxCorner.Parent = BoxFrame
            
            local Title = Instance.new("TextLabel")
            Title.Text = Name
            Title.Size = UDim2.new(1, -20, 0, 25)
            Title.Position = UDim2.new(0, 10, 0, 5)
            Title.BackgroundTransparency = 1
            Title.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Text
            Title.Font = Enum.Font.GothamMedium
            Title.TextSize = 14
            Title.TextXAlignment = Enum.TextXAlignment.Left
            Title.Parent = BoxFrame
            
            local Input = Instance.new("TextBox")
            Input.Size = UDim2.new(0, 120, 0, 30)
            Input.Position = UDim2.new(1, -130, 0, 7)
            Input.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Secondary
            Input.PlaceholderText = Placeholder
            Input.Text = ""
            Input.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Text
            Input.Font = Enum.Font.Gotham
            Input.TextSize = 12
            Input.Parent = BoxFrame
            
            local InputCorner = Instance.new("UICorner")
            InputCorner.CornerRadius = UDim.new(0, 4)
            InputCorner.Parent = Input
            
            Input.FocusLost:Connect(function(enter)
                pcall(Callback, Input.Text)
            end)
        end
        
        function TabObj:AddDiscordInvite(options)
            local Name = options.Name or "Discord"
            local Desc = options.Description or "Join"
            local Logo = options.Logo or "rbxassetid://18751483361"
            local Invite = options.Invite or ""
            
            local InviteFrame = Instance.new("Frame")
            InviteFrame.Size = UDim2.new(1, 0, 0, 70)
            InviteFrame.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
            InviteFrame.Parent = TabContainer
            
            local InviteCorner = Instance.new("UICorner")
            InviteCorner.CornerRadius = UDim.new(0, 6)
            InviteCorner.Parent = InviteFrame
            
            local Icon = Instance.new("ImageLabel")
            Icon.Image = Logo
            Icon.Size = UDim2.new(0, 50, 0, 50)
            Icon.Position = UDim2.new(0, 10, 0, 10)
            Icon.BackgroundTransparency = 1
            Icon.Parent = InviteFrame
            
            local Title = Instance.new("TextLabel")
            Title.Text = Name
            Title.Size = UDim2.new(1, -70, 0, 25)
            Title.Position = UDim2.new(0, 70, 0, 10)
            Title.BackgroundTransparency = 1
            Title.TextColor3 = Color3.fromRGB(255, 255, 255)
            Title.Font = Enum.Font.GothamBold
            Title.TextSize = 16
            Title.TextXAlignment = Enum.TextXAlignment.Left
            Title.Parent = InviteFrame
            
            local Description = Instance.new("TextLabel")
            Description.Text = Desc
            Description.Size = UDim2.new(1, -70, 0, 20)
            Description.Position = UDim2.new(0, 70, 0, 35)
            Description.BackgroundTransparency = 1
            Description.TextColor3 = Color3.fromRGB(220, 220, 220)
            Description.Font = Enum.Font.Gotham
            Description.TextSize = 12
            Description.TextXAlignment = Enum.TextXAlignment.Left
            Description.Parent = InviteFrame
            
            local JoinBtn = Instance.new("TextButton")
            JoinBtn.Size = UDim2.new(1, 0, 1, 0)
            JoinBtn.BackgroundTransparency = 1
            JoinBtn.Text = ""
            JoinBtn.Parent = InviteFrame
            
            JoinBtn.MouseButton1Click:Connect(function()
                if setclipboard then
                    setclipboard(Invite)
                end
            end)
        end

        return TabObj
    end

    function Window:Dialog(options)
        local Title = options.Title or "Dialog"
        local Text = options.Text or ""
        local Options = options.Options or {}

        local DialogOverlay = Instance.new("Frame")
        DialogOverlay.Size = UDim2.new(1, 0, 1, 0)
        DialogOverlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        DialogOverlay.BackgroundTransparency = 0.6
        DialogOverlay.ZIndex = 10
        DialogOverlay.Parent = MainFrame
        
        local DialogFrame = Instance.new("Frame")
        DialogFrame.Size = UDim2.new(0, 300, 0, 150)
        DialogFrame.Position = UDim2.new(0.5, -150, 0.5, -75)
        DialogFrame.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Main
        DialogFrame.ZIndex = 11
        DialogFrame.Parent = DialogOverlay
        
        local DialogCorner = Instance.new("UICorner")
        DialogCorner.CornerRadius = UDim.new(0, 10)
        DialogCorner.Parent = DialogFrame
        
        local DialogTitle = Instance.new("TextLabel")
        DialogTitle.Text = Title
        DialogTitle.Size = UDim2.new(1, 0, 0, 30)
        DialogTitle.Position = UDim2.new(0, 0, 0, 10)
        DialogTitle.BackgroundTransparency = 1
        DialogTitle.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Text
        DialogTitle.Font = Enum.Font.GothamBold
        DialogTitle.TextSize = 18
        DialogTitle.ZIndex = 11
        DialogTitle.Parent = DialogFrame
        
        local DialogText = Instance.new("TextLabel")
        DialogText.Text = Text
        DialogText.Size = UDim2.new(1, -20, 0, 60)
        DialogText.Position = UDim2.new(0, 10, 0, 40)
        DialogText.BackgroundTransparency = 1
        DialogText.TextColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Placeholder
        DialogText.Font = Enum.Font.Gotham
        DialogText.TextSize = 14
        DialogText.TextWrapped = true
        DialogText.ZIndex = 11
        DialogText.Parent = DialogFrame
        
        local ButtonContainer = Instance.new("Frame")
        ButtonContainer.Size = UDim2.new(1, -20, 0, 40)
        ButtonContainer.Position = UDim2.new(0, 10, 1, -50)
        ButtonContainer.BackgroundTransparency = 1
        ButtonContainer.ZIndex = 11
        ButtonContainer.Parent = DialogFrame
        
        local BtnLayout = Instance.new("UIListLayout")
        BtnLayout.FillDirection = Enum.FillDirection.Horizontal
        BtnLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
        BtnLayout.Padding = UDim.new(0, 10)
        BtnLayout.Parent = ButtonContainer
        
        for _, opt in pairs(Options) do
            local OptName = opt[1]
            local OptFunc = opt[2]
            
            local OptBtn = Instance.new("TextButton")
            OptBtn.Text = OptName
            OptBtn.Size = UDim2.new(0, 80, 1, 0)
            OptBtn.BackgroundColor3 = NovaLib.Themes[NovaLib.CurrentTheme].Accent
            OptBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            OptBtn.Font = Enum.Font.GothamMedium
            OptBtn.TextSize = 12
            OptBtn.ZIndex = 11
            OptBtn.Parent = ButtonContainer
            
            local OptCorner = Instance.new("UICorner")
            OptCorner.CornerRadius = UDim.new(0, 6)
            OptCorner.Parent = OptBtn
            
            OptBtn.MouseButton1Click:Connect(function()
                pcall(OptFunc)
                DialogOverlay:Destroy()
            end)
        end
    end

    return Window
end

return NovaLib
