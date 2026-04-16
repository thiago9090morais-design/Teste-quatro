--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local LP = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- CONFIG
local cfg = {
    Speed = 16,
    Jump = 50,
    ESP = false,
    Health = false,
    FOV = false,
    FOVSize = 120
}

local aim = {
    Enabled = false,
    Smooth = 0.1,
    FOV = 120
}

-- TOUCH AIM
local aiming = false

UIS.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch then
        aiming = true
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch then
        aiming = false
    end
end)

-- GUI
local gui = Instance.new("ScreenGui", game.CoreGui)

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0,460,0,320)
main.Position = UDim2.new(0.3,0,0.3,0)
main.BackgroundColor3 = Color3.fromRGB(18,18,18)

local top = Instance.new("Frame", main)
top.Size = UDim2.new(1,0,0,40)
top.BackgroundColor3 = Color3.fromRGB(25,25,25)

local title = Instance.new("TextLabel", top)
title.Size = UDim2.new(1,-40,1,0)
title.Text = "th_mods 2.0"
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundTransparency = 1

-- MINIMIZAR
local mini = Instance.new("TextButton", top)
mini.Size = UDim2.new(0,40,1,0)
mini.Position = UDim2.new(1,-40,0,0)
mini.Text = "-"

local bubble = Instance.new("TextButton", gui)
bubble.Size = UDim2.new(0,60,0,30)
bubble.Position = UDim2.new(0.05,0,0.5,0)
bubble.Text = "th"
bubble.Visible = false

mini.MouseButton1Click:Connect(function()
    main.Visible=false
    bubble.Visible=true
end)

bubble.MouseButton1Click:Connect(function()
    main.Visible=true
    bubble.Visible=false
end)

-- DRAG MOBILE
local dragging=false
local startPos,startInput

top.InputBegan:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.Touch then
        dragging=true
        startInput=input.Position
        startPos=main.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if dragging then
        local delta=input.Position-startInput
        main.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+delta.X,startPos.Y.Scale,startPos.Y.Offset+delta.Y)
    end
end)

UIS.InputEnded:Connect(function()
    dragging=false
end)

-- SIDE
local side = Instance.new("Frame", main)
side.Size = UDim2.new(0,130,1,-40)
side.Position = UDim2.new(0,0,0,40)
side.BackgroundColor3 = Color3.fromRGB(22,22,22)

local content = Instance.new("Frame", main)
content.Size = UDim2.new(1,-130,1,-40)
content.Position = UDim2.new(0,130,0,40)

-- ABAS
local tabs={"Main","Visuals","Features"}
local pages={}

for i,n in ipairs(tabs) do
    local b=Instance.new("TextButton",side)
    b.Size=UDim2.new(1,0,0,40)
    b.Position=UDim2.new(0,0,0,(i-1)*40)
    b.Text=n
    b.TextColor3=Color3.new(1,1,1)
    b.BackgroundColor3=Color3.fromRGB(30,30,30)

    local p=Instance.new("Frame",content)
    p.Size=UDim2.new(1,0,1,0)
    p.Visible=(i==1)
    p.BackgroundTransparency=1

    b.MouseButton1Click:Connect(function()
        for _,v in pairs(pages) do v.Visible=false end
        p.Visible=true
    end)

    pages[n]=p
end

-- UI
local function toggle(parent,text,y,callback)
    local b=Instance.new("TextButton",parent)
    b.Size=UDim2.new(1,-10,0,30)
    b.Position=UDim2.new(0,5,0,y)
    b.Text=text.." [OFF]"
    b.BackgroundColor3=Color3.fromRGB(50,50,50)
    b.TextColor3=Color3.new(1,1,1)

    local state=false
    b.MouseButton1Click:Connect(function()
        state=not state
        b.Text=text.." "..(state and "[ON]" or "[OFF]")
        callback(state)
    end)
end

local function slider(parent,text,y,min,max,callback)
    local val=min
    local b=Instance.new("TextButton",parent)
    b.Size=UDim2.new(1,-10,0,30)
    b.Position=UDim2.new(0,5,0,y)
    b.Text=text..": "..val

    b.MouseButton1Click:Connect(function()
        val=val+10
        if val>max then val=min end
        b.Text=text..": "..val
        callback(val)
    end)
end

-- MAIN
slider(pages.Main,"Speed",10,16,100,function(v) cfg.Speed=v end)
slider(pages.Main,"Jump",50,50,150,function(v) cfg.Jump=v end)

-- VISUAL
toggle(pages.Visuals,"ESP",10,function(v) cfg.ESP=v end)
toggle(pages.Visuals,"Health Bar",50,function(v) cfg.Health=v end)
toggle(pages.Visuals,"FOV",90,function(v) cfg.FOV=v end)
slider(pages.Visuals,"FOV Size",130,50,300,function(v) cfg.FOVSize=v end)

-- FEATURES
toggle(pages.Features,"Aim Assist",10,function(v) aim.Enabled=v end)
slider(pages.Features,"Smooth",50,1,10,function(v) aim.Smooth=v/10 end)

-- FOV
local circle = Drawing.new("Circle")
circle.Thickness=2
circle.Color=Color3.fromRGB(0,255,0)
circle.Filled=false

-- ESP + VIDA
local function setupPlayer(p)
    if p==LP then return end

    local function apply(char)
        local hum=char:WaitForChild("Humanoid")
        local head=char:WaitForChild("Head")

        local h=Instance.new("Highlight",char)

        local g=Instance.new("BillboardGui",head)
        g.Size=UDim2.new(4,0,0.5,0)
        g.StudsOffset=Vector3.new(2,2,0)

        local bar=Instance.new("Frame",g)
        bar.BackgroundColor3=Color3.fromRGB(0,255,0)

        hum.HealthChanged:Connect(function()
            bar.Size=UDim2.new(hum.Health/hum.MaxHealth,0,1,0)
        end)

        RunService.RenderStepped:Connect(function()
            h.Enabled=cfg.ESP
            g.Enabled=cfg.Health
        end)
    end

    if p.Character then apply(p.Character) end
    p.CharacterAdded:Connect(apply)
end

for _,p in pairs(Players:GetPlayers()) do setupPlayer(p) end
Players.PlayerAdded:Connect(setupPlayer)

-- AIM TARGET
local function getTarget()
    local closest=nil
    local shortest=aim.FOV

    for _,p in pairs(Players:GetPlayers()) do
        if p~=LP and p.Character then
            local head=p.Character:FindFirstChild("Head")
            if head then
                local pos,vis=Camera:WorldToViewportPoint(head.Position)
                if vis then
                    local dist=(Vector2.new(pos.X,pos.Y)-Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y/2)).Magnitude
                    if dist<shortest then
                        shortest=dist
                        closest=head
                    end
                end
            end
        end
    end

    return closest
end

-- LOOP
RunService.RenderStepped:Connect(function()
    if LP.Character and LP.Character:FindFirstChild("Humanoid") then
        local hum=LP.Character.Humanoid
        hum.WalkSpeed=cfg.Speed
        hum.JumpPower=cfg.Jump
    end

    -- FOV
    circle.Position=Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y/2)
    circle.Radius=cfg.FOVSize
    circle.Visible=cfg.FOV

    -- AIM TOUCH
    if aim.Enabled and aiming then
        local target=getTarget()
        if target then
            local cf=CFrame.new(Camera.CFrame.Position,target.Position)
            Camera.CFrame=Camera.CFrame:Lerp(cf,aim.Smooth)
        end
    end
end)
