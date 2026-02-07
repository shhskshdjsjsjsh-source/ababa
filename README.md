local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local LocalPlayer = game:GetService("Players").LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local HttpService = game:GetService("HttpService")

local NovaLib = {
    Themes = {
        Dark = {
            Main = Color3.fromRGB(25, 25, 25),
            Secondary = Color3.fromRGB(35, 35, 35), -- Sidebar/Content BG
            Tertiary = Color3.fromRGB(45, 45, 45), -- Elements BG
            Text = Color3.fromRGB(240, 240, 240),
            Accent = Color3.fromRGB(255, 80, 80), -- Reddish accent like the image
            Placeholder = Color3.fromRGB(160, 160, 160),
            Stroke = Color3.fromRGB(60, 60, 60),
            Transparency = 0.1
        },
        Darker = {
            Main = Color3.fromRGB(15, 15, 15),
            Secondary = Color3.fromRGB(25, 25, 25),
            Tertiary = Color3.fromRGB(35, 35, 35),
            Text = Color3.fromRGB(220, 220, 220),
            Accent = Color3.fromRGB(100, 100, 255),
            Placeholder = Color3.fromRGB(120, 120, 120),
            Stroke = Color3.fromRGB(50, 50, 50),
            Transparency = 0.05
        },
        Purple = {
            Main = Color3.fromRGB(30, 20, 40),
            Secondary = Color3.fromRGB(40, 30, 50),
            Tertiary = Color3.fromRGB(50, 40, 60),
            Text = Color3.fromRGB(255, 230, 255),
            Accent = Color3.fromRGB(180, 80, 255),
            Placeholder = Color3.fromRGB(180, 150, 180),
            Stroke = Color3.fromRGB(80, 60, 100),
            Transparency = 0.1
        }
    },
    CurrentTheme = "Dark"
}

-- Utility Functions
local function AddStroke(parent, color, thickness, transparency)
    local stroke = Instance.new("UIStroke")
    stroke.Color = color or NovaLib.Themes[NovaLib.CurrentTheme].Stroke
    stroke.Thickness = thickness or 1
    stroke.Transparency = transparency or 0
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    stroke.Parent = parent
    return stroke
end

local function MakeDraggable(topbarobject, object)
    local Dragging = nil
    local DragInput = nil
    local DragStart = nil
    local StartPosition = nil

    local function Update(input)
        local Delta = input.Position - DragStart
        local pos = UDim2.new(StartPosition.X.Scale, StartPosition.X.Offset + Delta.X, StartPosition.Y.Scale, StartPosition.Y.Offset + Delta.Y)
        game:GetService("TweenService"):Create(object, TweenInfo.new(0.25), {Position = pos}):Play()
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
    
    local Theme = NovaLib.Themes[NovaLib.CurrentTheme]

    -- Create ScreenGui
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "NovaLibUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.IgnoreGuiInset = true
    ScreenGui.DisplayOrder = 100 -- Ensure it's on top
    
    if pcall(function() ScreenGui.Parent = game:GetService("CoreGui") end) then
    else
        ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    end

    -- Main Window Frame
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 650, 0, 400)
    MainFrame.Position = UDim2.new(0.5, -325, 0.5, -200)
    MainFrame.BackgroundColor3 = Theme.Main
    MainFrame.BackgroundTransparency = Theme.Transparency
    MainFrame.BorderSizePixel = 0
    MainFrame.ClipsDescendants = false
    MainFrame.Parent = ScreenGui

    local MainCorner = Instance.new("UICorner")
    MainCorner.CornerRadius = UDim.new(0, 8)
    MainCorner.Parent = MainFrame
    
    AddStroke(MainFrame, Theme.Stroke, 1.5, 0.5)

    -- TopBar
    local TopBar = Instance.new("Frame")
    TopBar.Name = "TopBar"
    TopBar.Size = UDim2.new(1, 0, 0, 50)
    TopBar.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    TopBar.BackgroundTransparency = 1
    TopBar.Parent = MainFrame

    local TitleLabel = Instance.new("TextLabel")
    TitleLabel.Name = "Title"
    TitleLabel.Text = string.format("<font color='rgb(%d,%d,%d)'>%s</font> : %s", Theme.Accent.R*255, Theme.Accent.G*255, Theme.Accent.B*255, Title, SubTitle)
    TitleLabel.Size = UDim2.new(1, -200, 1, 0)
    TitleLabel.Position = UDim2.new(0, 20, 0, 0)
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.TextColor3 = Theme.Text
    TitleLabel.TextSize = 16
    TitleLabel.Font = Enum.Font.GothamBold
    TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    TitleLabel.RichText = true
    TitleLabel.Parent = TopBar

    -- Window Controls (MacOS style)
    local Controls = Instance.new("Frame")
    Controls.Name = "Controls"
    Controls.Size = UDim2.new(0, 60, 0, 20)
    Controls.Position = UDim2.new(1, -70, 0, 15)
    Controls.BackgroundTransparency = 1
    Controls.Parent = TopBar
    
    local ListLayout = Instance.new("UIListLayout")
    ListLayout.FillDirection = Enum.FillDirection.Horizontal
    ListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    ListLayout.Padding = UDim.new(0, 8)
    ListLayout.Parent = Controls
    
    local function CreateControlBtn(color, name, callback)
        local Btn = Instance.new("TextButton")
        Btn.Name = name
        Btn.Size = UDim2.new(0, 14, 0, 14)
        Btn.BackgroundColor3 = color
        Btn.Text = ""
        Btn.Parent = Controls
        
        local Corner = Instance.new("UICorner")
        Corner.CornerRadius = UDim.new(1, 0)
        Corner.Parent = Btn
        
        Btn.MouseButton1Click:Connect(callback)
        return Btn
    end
    
    -- Minimize Button (Yellow)
    local isMinimized = false
    local savedSize = MainFrame.Size
    CreateControlBtn(Color3.fromRGB(255, 189, 46), "Minimize", function()
        isMinimized = not isMinimized
        if isMinimized then
            savedSize = MainFrame.Size
            TweenService:Create(MainFrame, TweenInfo.new(0.3), {Size = UDim2.new(0, 650, 0, 50)}):Play()
            -- Hide content
            MainFrame.SidebarArea.Visible = false
            MainFrame.ContentArea.Visible = false
        else
            TweenService:Create(MainFrame, TweenInfo.new(0.3), {Size = savedSize}):Play()
            MainFrame.SidebarArea.Visible = true
            MainFrame.ContentArea.Visible = true
        end
    end)
    
    -- Maximize/Restore Button (Green) - For now just toggles a larger size
    local isMaximized = false
    CreateControlBtn(Color3.fromRGB(39, 200, 63), "Maximize", function()
        isMaximized = not isMaximized
        if isMaximized then
            savedSize = MainFrame.Size -- Save current size before max
            TweenService:Create(MainFrame, TweenInfo.new(0.3), {Size = UDim2.new(0, 800, 0, 500)}):Play()
        else
            TweenService:Create(MainFrame, TweenInfo.new(0.3), {Size = UDim2.new(0, 650, 0, 400)}):Play()
        end
    end)
    
    -- Close Button (Red)
    CreateControlBtn(Color3.fromRGB(254, 94, 86), "Close", function()
        ScreenGui:Destroy()
    end)

    MakeDraggable(TopBar, MainFrame)

    -- Sidebar Area
    local SidebarArea = Instance.new("Frame")
    SidebarArea.Name = "SidebarArea"
    SidebarArea.Size = UDim2.new(0, 180, 1, -50)
    SidebarArea.Position = UDim2.new(0, 0, 0, 50)
    SidebarArea.BackgroundTransparency = 1
    SidebarArea.Parent = MainFrame
    
    local SidebarDiv = Instance.new("Frame")
    SidebarDiv.Size = UDim2.new(0, 1, 1, -20)
    SidebarDiv.Position = UDim2.new(1, 0, 0, 10)
    SidebarDiv.BackgroundColor3 = Theme.Stroke
    SidebarDiv.BackgroundTransparency = 0.5
    SidebarDiv.BorderSizePixel = 0
    SidebarDiv.Parent = SidebarArea

    -- Search Bar in Sidebar
    local SearchFrame = Instance.new("Frame")
    SearchFrame.Name = "SearchFrame"
    SearchFrame.Size = UDim2.new(1, -20, 0, 30)
    SearchFrame.Position = UDim2.new(0, 10, 0, 0)
    SearchFrame.BackgroundColor3 = Theme.Secondary
    SearchFrame.BackgroundTransparency = 0.5
    SearchFrame.Parent = SidebarArea
    
    local SearchCorner = Instance.new("UICorner")
    SearchCorner.CornerRadius = UDim.new(0, 6)
    SearchCorner.Parent = SearchFrame
    
    AddStroke(SearchFrame, Theme.Stroke, 1, 0.7)
    
    local SearchIcon = Instance.new("ImageLabel")
    SearchIcon.Image = "rbxassetid://6031154871"
    SearchIcon.Size = UDim2.new(0, 16, 0, 16)
    SearchIcon.Position = UDim2.new(0, 8, 0, 7)
    SearchIcon.ImageColor3 = Theme.Placeholder
    SearchIcon.BackgroundTransparency = 1
    SearchIcon.Parent = SearchFrame
    
    local SearchBox = Instance.new("TextBox")
    SearchBox.Size = UDim2.new(1, -35, 1, 0)
    SearchBox.Position = UDim2.new(0, 30, 0, 0)
    SearchBox.BackgroundTransparency = 1
    SearchBox.PlaceholderText = "Search..."
    SearchBox.Text = ""
    SearchBox.TextColor3 = Theme.Text
    SearchBox.PlaceholderColor3 = Theme.Placeholder
    SearchBox.Font = Enum.Font.Gotham
    SearchBox.TextSize = 12
    SearchBox.TextXAlignment = Enum.TextXAlignment.Left
    SearchBox.Parent = SearchFrame

    -- Tab Scroll Frame
    local TabList = Instance.new("ScrollingFrame")
    TabList.Name = "TabList"
    TabList.Size = UDim2.new(1, 0, 1, -90) -- Reduced height to fit profile
    TabList.Position = UDim2.new(0, 0, 0, 40)
    TabList.BackgroundTransparency = 1
    TabList.ScrollBarThickness = 2
    TabList.ScrollBarImageColor3 = Theme.Accent
    TabList.Parent = SidebarArea
    TabList.CanvasSize = UDim2.new(0, 0, 0, 0)
    TabList.AutomaticCanvasSize = Enum.AutomaticCanvasSize.Y
    
    local TabListLayout = Instance.new("UIListLayout")
    TabListLayout.Padding = UDim.new(0, 5)
    TabListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    TabListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    TabListLayout.Parent = TabList
    
    -- Profile Area at Bottom of Sidebar
    local ProfileFrame = Instance.new("Frame")
    ProfileFrame.Name = "ProfileFrame"
    ProfileFrame.Size = UDim2.new(1, -20, 0, 40)
    ProfileFrame.Position = UDim2.new(0, 10, 1, -45)
    ProfileFrame.BackgroundColor3 = Theme.Secondary
    ProfileFrame.BackgroundTransparency = 0.5
    ProfileFrame.Parent = SidebarArea
    
    local ProfileCorner = Instance.new("UICorner")
    ProfileCorner.CornerRadius = UDim.new(0, 8)
    ProfileCorner.Parent = ProfileFrame
    
    AddStroke(ProfileFrame, Theme.Stroke, 1, 0.5)
    
    local ProfileImage = Instance.new("ImageLabel")
    ProfileImage.Size = UDim2.new(0, 30, 0, 30)
    ProfileImage.Position = UDim2.new(0, 5, 0, 5)
    ProfileImage.BackgroundTransparency = 1
    local success, content = pcall(function() 
        return game:GetService("Players"):GetUserThumbnailAsync(LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
    end)
    ProfileImage.Image = success and content or "rbxassetid://71014873973869"
    ProfileImage.Parent = ProfileFrame
    
    local PImageCorner = Instance.new("UICorner")
    PImageCorner.CornerRadius = UDim.new(1, 0)
    PImageCorner.Parent = ProfileImage
    
    local ProfileName = Instance.new("TextLabel")
    ProfileName.Text = LocalPlayer.Name
    ProfileName.Size = UDim2.new(1, -45, 0, 20)
    ProfileName.Position = UDim2.new(0, 40, 0, 2)
    ProfileName.BackgroundTransparency = 1
    ProfileName.TextColor3 = Theme.Text
    ProfileName.Font = Enum.Font.GothamBold
    ProfileName.TextSize = 12
    ProfileName.TextXAlignment = Enum.TextXAlignment.Left
    ProfileName.Parent = ProfileFrame

    local ProfileRank = Instance.new("TextLabel")
    ProfileRank.Text = "User" -- Placeholder
    ProfileRank.Size = UDim2.new(1, -45, 0, 15)
    ProfileRank.Position = UDim2.new(0, 40, 0, 20)
    ProfileRank.BackgroundTransparency = 1
    ProfileRank.TextColor3 = Theme.Placeholder
    ProfileRank.Font = Enum.Font.Gotham
    ProfileRank.TextSize = 10
    ProfileRank.TextXAlignment = Enum.TextXAlignment.Left
    ProfileRank.Parent = ProfileFrame

    -- Content Area
    local ContentArea = Instance.new("Frame")
    ContentArea.Name = "ContentArea"
    ContentArea.Size = UDim2.new(1, -190, 1, -60)
    ContentArea.Position = UDim2.new(0, 190, 0, 50)
    ContentArea.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    ContentArea.BackgroundTransparency = 1
    ContentArea.Parent = MainFrame
    
    -- Content Header (Current Tab Name / Search)
    local ContentHeader = Instance.new("Frame")
    ContentHeader.Size = UDim2.new(1, 0, 0, 30)
    ContentHeader.BackgroundTransparency = 1
    ContentHeader.Parent = ContentArea
    
    local CurrentTabLabel = Instance.new("TextLabel")
    CurrentTabLabel.Name = "CurrentTabLabel"
    CurrentTabLabel.Text = "Home"
    CurrentTabLabel.Size = UDim2.new(0.5, 0, 1, 0)
    CurrentTabLabel.BackgroundTransparency = 1
    CurrentTabLabel.TextColor3 = Theme.Text
    CurrentTabLabel.Font = Enum.Font.GothamBold
    CurrentTabLabel.TextSize = 18
    CurrentTabLabel.TextXAlignment = Enum.TextXAlignment.Left
    CurrentTabLabel.Parent = ContentHeader
    
    -- Real Content Container
    local PagesContainer = Instance.new("Frame")
    PagesContainer.Size = UDim2.new(1, 0, 1, -35)
    PagesContainer.Position = UDim2.new(0, 0, 0, 35)
    PagesContainer.BackgroundTransparency = 1
    PagesContainer.Parent = ContentArea

    local Window = {}
    local Tabs = {}
    local FirstTab = nil
    
    function Window:SelectTab(Tab)
        for _, t in pairs(Tabs) do
            t.Container.Visible = false
            -- Reset Tab Button Style
            t.Button.TextLabel.TextColor3 = Theme.Placeholder
            t.Indicator.BackgroundTransparency = 1
            TweenService:Create(t.Indicator, TweenInfo.new(0.2), {Size = UDim2.new(0, 0, 0, 15)}):Play()
        end
        Tab.Container.Visible = true
        CurrentTabLabel.Text = Tab.Name
        
        -- Active Tab Style
        Tab.Button.TextLabel.TextColor3 = Theme.Text
        Tab.Indicator.BackgroundTransparency = 0
        TweenService:Create(Tab.Indicator, TweenInfo.new(0.2), {Size = UDim2.new(0, 3, 0, 15)}):Play()
    end

    function Window:MakeTab(options)
        local TabName = options[1] or "Tab"
        local TabIcon = options[2] -- Optional icon
        
        local TabBtn = Instance.new("TextButton")
        TabBtn.Name = TabName .. "Btn"
        TabBtn.Size = UDim2.new(1, -20, 0, 35)
        TabBtn.BackgroundTransparency = 1
        TabBtn.Text = ""
        TabBtn.Parent = TabList
        
        local Indicator = Instance.new("Frame")
        Indicator.Name = "Indicator"
        Indicator.Size = UDim2.new(0, 0, 0, 15) -- Starts invisible/small
        Indicator.Position = UDim2.new(0, 0, 0.5, -7.5)
        Indicator.BackgroundColor3 = Theme.Accent
        Indicator.BorderSizePixel = 0
        Indicator.BackgroundTransparency = 1
        Indicator.Parent = TabBtn
        
        local IndicatorCorner = Instance.new("UICorner")
        IndicatorCorner.CornerRadius = UDim.new(1, 0)
        IndicatorCorner.Parent = Indicator
        
        local BtnText = Instance.new("TextLabel")
        BtnText.Size = UDim2.new(1, -15, 1, 0)
        BtnText.Position = UDim2.new(0, 15, 0, 0)
        BtnText.BackgroundTransparency = 1
        BtnText.Text = TabName
        BtnText.TextColor3 = Theme.Placeholder
        BtnText.Font = Enum.Font.GothamMedium
        BtnText.TextSize = 14
        BtnText.TextXAlignment = Enum.TextXAlignment.Left
        BtnText.Parent = TabBtn
        
        -- Tab Content
        local TabContainer = Instance.new("ScrollingFrame")
        TabContainer.Name = TabName .. "Container"
        TabContainer.Size = UDim2.new(1, 0, 1, 0)
        TabContainer.BackgroundTransparency = 1
        TabContainer.ScrollBarThickness = 2
        TabContainer.ScrollBarImageColor3 = Theme.Accent
        TabContainer.Visible = false
        TabContainer.Parent = PagesContainer
        TabContainer.CanvasSize = UDim2.new(0, 0, 0, 0) -- Adicionado
        TabContainer.AutomaticCanvasSize = Enum.AutomaticCanvasSize.Y -- Adicionado
        
        local TabLayout = Instance.new("UIListLayout")
        TabLayout.Padding = UDim.new(0, 8)
        TabLayout.SortOrder = Enum.SortOrder.LayoutOrder
        TabLayout.Parent = TabContainer
        
        local TabPadding = Instance.new("UIPadding")
        TabPadding.PaddingTop = UDim.new(0, 5)
        TabPadding.PaddingBottom = UDim.new(0, 10)
        TabPadding.PaddingRight = UDim.new(0, 5)
        TabPadding.Parent = TabContainer
        
        local TabObj = {
            Name = TabName,
            Button = TabBtn,
            Indicator = Indicator,
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

        -- ELEMENTS
        function TabObj:AddSection(options)
            local Text = options[1] or "Section"
            
            local SectionFrame = Instance.new("Frame")
            SectionFrame.Size = UDim2.new(1, 0, 0, 30)
            SectionFrame.BackgroundTransparency = 1
            SectionFrame.Parent = TabContainer
            
            local Label = Instance.new("TextLabel")
            Label.Text = Text
            Label.Size = UDim2.new(1, 0, 1, 0)
            Label.BackgroundTransparency = 1
            Label.TextColor3 = Theme.Accent
            Label.Font = Enum.Font.GothamBold
            Label.TextSize = 14
            Label.TextXAlignment = Enum.TextXAlignment.Left
            Label.Parent = SectionFrame
            
            return SectionFrame
        end

        function TabObj:AddButton(options)
            local Text = options[1] or "Button"
            local Callback = options[2] or function() end
            
            local Btn = Instance.new("TextButton")
            Btn.Size = UDim2.new(1, 0, 0, 35)
            Btn.BackgroundColor3 = Theme.Tertiary
            Btn.BackgroundTransparency = 0.2
            Btn.Text = Text
            Btn.TextColor3 = Theme.Text
            Btn.Font = Enum.Font.GothamMedium
            Btn.TextSize = 13
            Btn.Parent = TabContainer
            
            local BtnCorner = Instance.new("UICorner")
            BtnCorner.CornerRadius = UDim.new(0, 6)
            BtnCorner.Parent = Btn
            
            AddStroke(Btn, Theme.Stroke, 1, 0.5)
            
            Btn.MouseButton1Click:Connect(function()
                pcall(Callback)
                TweenService:Create(Btn, TweenInfo.new(0.1), {BackgroundColor3 = Theme.Accent}):Play()
                wait(0.1)
                TweenService:Create(Btn, TweenInfo.new(0.2), {BackgroundColor3 = Theme.Tertiary}):Play()
            end)
        end
        
        function TabObj:AddToggle(options)
            local Name = options.Name or "Toggle"
            local Description = options.Description or ""
            local Default = options.Default or false
            local Callback = options.Callback or function() end
            
            local ToggleFrame = Instance.new("Frame")
            ToggleFrame.Size = UDim2.new(1, 0, 0, Description ~= "" and 50 or 40)
            ToggleFrame.BackgroundColor3 = Theme.Tertiary
            ToggleFrame.BackgroundTransparency = 0.2
            ToggleFrame.Parent = TabContainer
            
            local ToggleCorner = Instance.new("UICorner")
            ToggleCorner.CornerRadius = UDim.new(0, 6)
            ToggleCorner.Parent = ToggleFrame
            
            AddStroke(ToggleFrame, Theme.Stroke, 1, 0.5)
            
            local Label = Instance.new("TextLabel")
            Label.Text = Name
            Label.Size = UDim2.new(1, -60, 0, 25)
            Label.Position = UDim2.new(0, 10, 0, 5)
            Label.BackgroundTransparency = 1
            Label.TextColor3 = Theme.Text
            Label.Font = Enum.Font.GothamMedium
            Label.TextSize = 13
            Label.TextXAlignment = Enum.TextXAlignment.Left
            Label.Parent = ToggleFrame
            
            if Description ~= "" then
                local Desc = Instance.new("TextLabel")
                Desc.Text = Description
                Desc.Size = UDim2.new(1, -60, 0, 15)
                Desc.Position = UDim2.new(0, 10, 0, 25)
                Desc.BackgroundTransparency = 1
                Desc.TextColor3 = Theme.Placeholder
                Desc.Font = Enum.Font.Gotham
                Desc.TextSize = 11
                Desc.TextXAlignment = Enum.TextXAlignment.Left
                Desc.Parent = ToggleFrame
            end
            
            local Switch = Instance.new("Frame")
            Switch.Size = UDim2.new(0, 40, 0, 20)
            Switch.Position = UDim2.new(1, -50, 0.5, -10)
            Switch.BackgroundColor3 = Default and Theme.Accent or Color3.fromRGB(50, 50, 50)
            Switch.Parent = ToggleFrame
            
            local SwitchCorner = Instance.new("UICorner")
            SwitchCorner.CornerRadius = UDim.new(1, 0)
            SwitchCorner.Parent = Switch
            
            local Dot = Instance.new("Frame")
            Dot.Size = UDim2.new(0, 16, 0, 16)
            Dot.Position = Default and UDim2.new(1, -18, 0, 2) or UDim2.new(0, 2, 0, 2)
            Dot.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            Dot.Parent = Switch
            
            local DotCorner = Instance.new("UICorner")
            DotCorner.CornerRadius = UDim.new(1, 0)
            DotCorner.Parent = Dot
            
            local Button = Instance.new("TextButton")
            Button.Size = UDim2.new(1, 0, 1, 0)
            Button.BackgroundTransparency = 1
            Button.Text = ""
            Button.Parent = ToggleFrame
            
            local Toggled = Default
            
            Button.MouseButton1Click:Connect(function()
                Toggled = not Toggled
                
                TweenService:Create(Switch, TweenInfo.new(0.2), {
                    BackgroundColor3 = Toggled and Theme.Accent or Color3.fromRGB(50, 50, 50)
                }):Play()
                
                TweenService:Create(Dot, TweenInfo.new(0.2), {
                    Position = Toggled and UDim2.new(1, -18, 0, 2) or UDim2.new(0, 2, 0, 2)
                }):Play()
                
                pcall(Callback, Toggled)
            end)
        end
        
        function TabObj:AddSlider(options)
            local Name = options.Name or "Slider"
            local Min = options.Min or 0
            local Max = options.Max or 100
            local Default = options.Default or Min
            local Callback = options.Callback or function() end
            
            local SliderFrame = Instance.new("Frame")
            SliderFrame.Size = UDim2.new(1, 0, 0, 50)
            SliderFrame.BackgroundColor3 = Theme.Tertiary
            SliderFrame.BackgroundTransparency = 0.2
            SliderFrame.Parent = TabContainer
            
            local SliderCorner = Instance.new("UICorner")
            SliderCorner.CornerRadius = UDim.new(0, 6)
            SliderCorner.Parent = SliderFrame
            
            AddStroke(SliderFrame, Theme.Stroke, 1, 0.5)
            
            local Label = Instance.new("TextLabel")
            Label.Text = Name
            Label.Size = UDim2.new(1, -60, 0, 20)
            Label.Position = UDim2.new(0, 10, 0, 5)
            Label.BackgroundTransparency = 1
            Label.TextColor3 = Theme.Text
            Label.Font = Enum.Font.GothamMedium
            Label.TextSize = 13
            Label.TextXAlignment = Enum.TextXAlignment.Left
            Label.Parent = SliderFrame
            
            local ValueLabel = Instance.new("TextLabel")
            ValueLabel.Text = tostring(Default)
            ValueLabel.Size = UDim2.new(0, 40, 0, 20)
            ValueLabel.Position = UDim2.new(1, -50, 0, 5)
            ValueLabel.BackgroundTransparency = 1
            ValueLabel.TextColor3 = Theme.Accent
            ValueLabel.Font = Enum.Font.GothamBold
            ValueLabel.TextSize = 13
            ValueLabel.TextXAlignment = Enum.TextXAlignment.Right
            ValueLabel.Parent = SliderFrame
            
            local SlideBG = Instance.new("Frame")
            SlideBG.Size = UDim2.new(1, -20, 0, 4)
            SlideBG.Position = UDim2.new(0, 10, 0, 35)
            SlideBG.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
            SlideBG.Parent = SliderFrame
            
            local SlideBGCorner = Instance.new("UICorner")
            SlideBGCorner.CornerRadius = UDim.new(1, 0)
            SlideBGCorner.Parent = SlideBG
            
            local Fill = Instance.new("Frame")
            Fill.Size = UDim2.new((Default - Min) / (Max - Min), 0, 1, 0)
            Fill.BackgroundColor3 = Theme.Accent
            Fill.Parent = SlideBG
            
            local FillCorner = Instance.new("UICorner")
            FillCorner.CornerRadius = UDim.new(1, 0)
            FillCorner.Parent = Fill
            
            local Trigger = Instance.new("TextButton")
            Trigger.Size = UDim2.new(1, 0, 1, 0)
            Trigger.BackgroundTransparency = 1
            Trigger.Text = ""
            Trigger.Parent = SliderFrame
            
            local isDragging = false
            
            local function UpdateSlider(input)
                local pos = UDim2.new(math.clamp((input.Position.X - SlideBG.AbsolutePosition.X) / SlideBG.AbsoluteSize.X, 0, 1), 0, 1, 0)
                TweenService:Create(Fill, TweenInfo.new(0.1), {Size = pos}):Play()
                
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
            DropFrame.Size = UDim2.new(1, 0, 0, 40)
            DropFrame.BackgroundColor3 = Theme.Tertiary
            DropFrame.BackgroundTransparency = 0.2
            DropFrame.ClipsDescendants = true
            DropFrame.Parent = TabContainer
            DropFrame.ZIndex = 5 -- Ensure Dropdown is above other elements
            
            local DropCorner = Instance.new("UICorner")
            DropCorner.CornerRadius = UDim.new(0, 6)
            DropCorner.Parent = DropFrame
            
            AddStroke(DropFrame, Theme.Stroke, 1, 0.5)
            
            local Label = Instance.new("TextLabel")
            Label.Text = Name
            Label.Size = UDim2.new(1, -130, 0, 40)
            Label.Position = UDim2.new(0, 10, 0, 0)
            Label.BackgroundTransparency = 1
            Label.TextColor3 = Theme.Text
            Label.Font = Enum.Font.GothamMedium
            Label.TextSize = 13
            Label.TextXAlignment = Enum.TextXAlignment.Left
            Label.Parent = DropFrame
            
            local Status = Instance.new("TextLabel")
            Status.Text = Default
            Status.Size = UDim2.new(0, 100, 0, 40)
            Status.Position = UDim2.new(1, -130, 0, 0)
            Status.BackgroundTransparency = 1
            Status.TextColor3 = Theme.Placeholder
            Status.Font = Enum.Font.Gotham
            Status.TextSize = 12
            Status.TextXAlignment = Enum.TextXAlignment.Right
            Status.Parent = DropFrame
            
            local Arrow = Instance.new("ImageLabel")
            Arrow.Image = "rbxassetid://6034818372"
            Arrow.Size = UDim2.new(0, 20, 0, 20)
            Arrow.Position = UDim2.new(1, -25, 0, 10)
            Arrow.BackgroundTransparency = 1
            Arrow.ImageColor3 = Theme.Text
            Arrow.Parent = DropFrame
            
            local Trigger = Instance.new("TextButton")
            Trigger.Size = UDim2.new(1, 0, 1, 0)
            Trigger.BackgroundTransparency = 1
            Trigger.Text = ""
            Trigger.Parent = DropFrame
            
            local ItemList = Instance.new("ScrollingFrame")
            ItemList.Size = UDim2.new(1, -20, 0, 0)
            ItemList.Position = UDim2.new(0, 10, 0, 45)
            ItemList.BackgroundTransparency = 1
            ItemList.ScrollBarThickness = 2
            ItemList.ScrollBarImageColor3 = Theme.Accent
            ItemList.Parent = DropFrame
            
            local ListLayout = Instance.new("UIListLayout")
            ListLayout.Padding = UDim.new(0, 5)
            ListLayout.Parent = ItemList
            
            local isOpen = false
            
            local function Refresh()
                for _, c in pairs(ItemList:GetChildren()) do
                    if c:IsA("TextButton") then c:Destroy() end
                end
                
                for _, opt in pairs(Options) do
                    local Btn = Instance.new("TextButton")
                    Btn.Size = UDim2.new(1, 0, 0, 25)
                    Btn.BackgroundColor3 = Theme.Secondary
                    Btn.BackgroundTransparency = 0.5
                    Btn.Text = opt
                    Btn.TextColor3 = Theme.Placeholder
                    Btn.Font = Enum.Font.Gotham
                    Btn.TextSize = 12
                    Btn.Parent = ItemList
                    
                    local C = Instance.new("UICorner")
                    C.CornerRadius = UDim.new(0, 4)
                    C.Parent = Btn
                    
                    Btn.MouseButton1Click:Connect(function()
                        Status.Text = opt
                        pcall(Callback, opt)
                        isOpen = false
                        TweenService:Create(DropFrame, TweenInfo.new(0.3), {Size = UDim2.new(1, 0, 0, 40)}):Play()
                        TweenService:Create(Arrow, TweenInfo.new(0.3), {Rotation = 0}):Play()
                    end)
                end
                
                ItemList.CanvasSize = UDim2.new(0, 0, 0, ListLayout.AbsoluteContentSize.Y)
            end
            
            Refresh()
            
            Trigger.MouseButton1Click:Connect(function()
                isOpen = not isOpen
                local h = isOpen and math.min(150, ItemList.CanvasSize.Y.Offset + 50) or 40
                TweenService:Create(DropFrame, TweenInfo.new(0.3), {Size = UDim2.new(1, 0, 0, h)}):Play()
                TweenService:Create(Arrow, TweenInfo.new(0.3), {Rotation = isOpen and 180 or 0}):Play()
            end)
        end
        
        -- Include other elements like TextBox, Paragraph, etc. using similar styles
        function TabObj:AddParagraph(options)
            local Title = options[1] or "Title"
            local Text = options[2] or "Text"
            
            local PFrame = Instance.new("Frame")
            PFrame.Size = UDim2.new(1, 0, 0, 0)
            PFrame.BackgroundColor3 = Theme.Tertiary
            PFrame.BackgroundTransparency = 0.5
            PFrame.AutomaticSize = Enum.AutomaticSize.Y
            PFrame.Parent = TabContainer
            
            local PCorner = Instance.new("UICorner")
            PCorner.CornerRadius = UDim.new(0, 6)
            PCorner.Parent = PFrame
            
            AddStroke(PFrame, Theme.Stroke, 1, 0.5)
            
            local PTitle = Instance.new("TextLabel")
            PTitle.Text = Title
            PTitle.Size = UDim2.new(1, -20, 0, 25)
            PTitle.Position = UDim2.new(0, 10, 0, 5)
            PTitle.BackgroundTransparency = 1
            PTitle.TextColor3 = Theme.Text
            PTitle.Font = Enum.Font.GothamBold
            PTitle.TextSize = 13
            PTitle.TextXAlignment = Enum.TextXAlignment.Left
            PTitle.Parent = PFrame
            
            local PText = Instance.new("TextLabel")
            PText.Text = Text
            PText.Size = UDim2.new(1, -20, 0, 0)
            PText.Position = UDim2.new(0, 10, 0, 30)
            PText.BackgroundTransparency = 1
            PText.TextColor3 = Theme.Placeholder
            PText.Font = Enum.Font.Gotham
            PText.TextSize = 12
            PText.TextXAlignment = Enum.TextXAlignment.Left
            PText.TextWrapped = true
            PText.AutomaticSize = Enum.AutomaticSize.Y
            PText.Parent = PFrame
            
            local Pad = Instance.new("UIPadding")
            Pad.PaddingBottom = UDim.new(0, 10)
            Pad.Parent = PFrame
        end
        
        function TabObj:AddTextBox(options)
            local Name = options.Name or "TextBox"
            local Placeholder = options.PlaceholderText or "Input..."
            local Callback = options.Callback or function() end
            
            local BoxFrame = Instance.new("Frame")
            BoxFrame.Size = UDim2.new(1, 0, 0, 40)
            BoxFrame.BackgroundColor3 = Theme.Tertiary
            BoxFrame.BackgroundTransparency = 0.2
            BoxFrame.Parent = TabContainer
            
            local BoxCorner = Instance.new("UICorner")
            BoxCorner.CornerRadius = UDim.new(0, 6)
            BoxCorner.Parent = BoxFrame
            
            AddStroke(BoxFrame, Theme.Stroke, 1, 0.5)
            
            local Label = Instance.new("TextLabel")
            Label.Text = Name
            Label.Size = UDim2.new(1, -140, 0, 40)
            Label.Position = UDim2.new(0, 10, 0, 0)
            Label.BackgroundTransparency = 1
            Label.TextColor3 = Theme.Text
            Label.Font = Enum.Font.GothamMedium
            Label.TextSize = 13
            Label.TextXAlignment = Enum.TextXAlignment.Left
            Label.Parent = BoxFrame
            
            local Input = Instance.new("TextBox")
            Input.Size = UDim2.new(0, 120, 0, 26)
            Input.Position = UDim2.new(1, -130, 0, 7)
            Input.BackgroundColor3 = Theme.Secondary
            Input.BackgroundTransparency = 0.5
            Input.PlaceholderText = Placeholder
            Input.Text = ""
            Input.TextColor3 = Theme.Text
            Input.PlaceholderColor3 = Theme.Placeholder
            Input.Font = Enum.Font.Gotham
            Input.TextSize = 12
            Input.Parent = BoxFrame
            
            local ICorner = Instance.new("UICorner")
            ICorner.CornerRadius = UDim.new(0, 4)
            ICorner.Parent = Input
            
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
            InviteFrame.Size = UDim2.new(1, 0, 0, 60)
            InviteFrame.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
            InviteFrame.BackgroundTransparency = 0.1
            InviteFrame.Parent = TabContainer
            InviteFrame.ZIndex = 5 -- Fix ZIndex
            
            local ICorner = Instance.new("UICorner")
            ICorner.CornerRadius = UDim.new(0, 8)
            ICorner.Parent = InviteFrame
            
            local Icon = Instance.new("ImageLabel")
            Icon.Image = Logo
            Icon.Size = UDim2.new(0, 40, 0, 40)
            Icon.Position = UDim2.new(0, 10, 0, 10)
            Icon.BackgroundTransparency = 1
            Icon.Parent = InviteFrame
            Icon.ZIndex = 5
            
            local Lbl = Instance.new("TextLabel")
            Lbl.Text = Name
            Lbl.Size = UDim2.new(1, -60, 0, 20)
            Lbl.Position = UDim2.new(0, 60, 0, 10)
            Lbl.BackgroundTransparency = 1
            Lbl.TextColor3 = Color3.fromRGB(255, 255, 255)
            Lbl.Font = Enum.Font.GothamBold
            Lbl.TextSize = 14
            Lbl.TextXAlignment = Enum.TextXAlignment.Left
            Lbl.Parent = InviteFrame
            Lbl.ZIndex = 5
            
            local DLbl = Instance.new("TextLabel")
            DLbl.Text = Desc
            DLbl.Size = UDim2.new(1, -60, 0, 15)
            DLbl.Position = UDim2.new(0, 60, 0, 30)
            DLbl.BackgroundTransparency = 1
            DLbl.TextColor3 = Color3.fromRGB(220, 220, 220)
            DLbl.Font = Enum.Font.Gotham
            DLbl.TextSize = 12
            DLbl.TextXAlignment = Enum.TextXAlignment.Left
            DLbl.Parent = InviteFrame
            DLbl.ZIndex = 5
            
            local Btn = Instance.new("TextButton")
            Btn.Size = UDim2.new(1, 0, 1, 0)
            Btn.BackgroundTransparency = 1
            Btn.Text = ""
            Btn.Parent = InviteFrame
            Btn.ZIndex = 5
            
            Btn.MouseButton1Click:Connect(function()
                 if setclipboard then setclipboard(Invite) end
            end)
        end

        return TabObj
    end
    
    function Window:Dialog(options)
        local Title = options.Title or "Dialog"
        local Text = options.Text or ""
        local Options = options.Options or {}

        local Overlay = Instance.new("Frame")
        Overlay.Size = UDim2.new(1, 0, 1, 0)
        Overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        Overlay.BackgroundTransparency = 0.6
        Overlay.ZIndex = 50 -- Very high ZIndex
        Overlay.Parent = MainFrame
        
        local DFrame = Instance.new("Frame")
        DFrame.Size = UDim2.new(0, 320, 0, 160)
        DFrame.Position = UDim2.new(0.5, -160, 0.5, -80)
        DFrame.BackgroundColor3 = Theme.Main
        DFrame.ZIndex = 51
        DFrame.Parent = Overlay
        
        local DCorner = Instance.new("UICorner")
        DCorner.CornerRadius = UDim.new(0, 10)
        DCorner.Parent = DFrame
        
        AddStroke(DFrame, Theme.Stroke, 1.5, 0)
        
        local DTitle = Instance.new("TextLabel")
        DTitle.Text = Title
        DTitle.Size = UDim2.new(1, 0, 0, 30)
        DTitle.Position = UDim2.new(0, 0, 0, 10)
        DTitle.BackgroundTransparency = 1
        DTitle.TextColor3 = Theme.Text
        DTitle.Font = Enum.Font.GothamBold
        DTitle.TextSize = 16
        DTitle.ZIndex = 52
        DTitle.Parent = DFrame
        
        local DText = Instance.new("TextLabel")
        DText.Text = Text
        DText.Size = UDim2.new(1, -20, 0, 60)
        DText.Position = UDim2.new(0, 10, 0, 40)
        DText.BackgroundTransparency = 1
        DText.TextColor3 = Theme.Placeholder
        DText.Font = Enum.Font.Gotham
        DText.TextSize = 14
        DText.TextWrapped = true
        DText.ZIndex = 52
        DText.Parent = DFrame
        
        local BtnContainer = Instance.new("Frame")
        BtnContainer.Size = UDim2.new(1, -20, 0, 40)
        BtnContainer.Position = UDim2.new(0, 10, 1, -50)
        BtnContainer.BackgroundTransparency = 1
        BtnContainer.ZIndex = 52
        BtnContainer.Parent = DFrame
        
        local UIList = Instance.new("UIListLayout")
        UIList.FillDirection = Enum.FillDirection.Horizontal
        UIList.HorizontalAlignment = Enum.HorizontalAlignment.Center
        UIList.Padding = UDim.new(0, 10)
        UIList.Parent = BtnContainer
        
        for _, o in pairs(Options) do
            local btn = Instance.new("TextButton")
            btn.Text = o[1]
            btn.Size = UDim2.new(0, 90, 1, 0)
            btn.BackgroundColor3 = Theme.Accent
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.Font = Enum.Font.GothamBold
            btn.TextSize = 12
            btn.ZIndex = 53
            btn.Parent = BtnContainer
            
            local c = Instance.new("UICorner")
            c.CornerRadius = UDim.new(0, 6)
            c.Parent = btn
            
            btn.MouseButton1Click:Connect(function()
                pcall(o[2])
                Overlay:Destroy()
            end)
        end
    end

    return Window
end

return NovaLib
