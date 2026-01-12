local Players=game:GetService("Players")
local RunService=game:GetService("RunService")
local UIS=game:GetService("UserInputService")
local Camera=workspace.CurrentCamera
local Player=Players.LocalPlayer

local aimbot=false
local visibleCheck=true
local smoothing=0.45
local FOV=140
local MAX_FOV=500

local espBox=false
local espName=false

local TARGET_FOLDER=workspace:WaitForChild("NPCs")

local gui=Instance.new("ScreenGui",Player.PlayerGui)
gui.ResetOnSpawn=false

local panel=Instance.new("Frame",gui)
panel.Size=UDim2.fromOffset(420,280)
panel.Position=UDim2.fromScale(0.5,0.5)
panel.AnchorPoint=Vector2.new(0.5,0.5)
panel.BackgroundColor3=Color3.fromRGB(15,15,15)
panel.Visible=false
panel.Active=true
panel.Draggable=true

local title=Instance.new("TextLabel",panel)
title.Size=UDim2.new(1,0,0,30)
title.Text="Script Black"
title.BackgroundColor3=Color3.fromRGB(0,0,0)
title.TextColor3=Color3.new(1,1,1)
title.Font=Enum.Font.SourceSansBold
title.TextSize=18

local function header(txt,x)
	local h=Instance.new("TextLabel",panel)
	h.Position=UDim2.fromOffset(x,40)
	h.Size=UDim2.fromOffset(180,22)
	h.Text=txt
	h.BackgroundTransparency=1
	h.TextColor3=Color3.fromRGB(200,200,200)
	h.Font=Enum.Font.SourceSansBold
	h.TextSize=16
	return h
end

local function btn(txt,x,y)
	local b=Instance.new("TextButton",panel)
	b.Size=UDim2.fromOffset(180,28)
	b.Position=UDim2.fromOffset(x,y)
	b.Text=txt
	b.BackgroundColor3=Color3.fromRGB(25,25,25)
	b.TextColor3=Color3.new(1,1,1)
	b.Font=Enum.Font.SourceSans
	b.TextSize=14
	return b
end

header("AIMBOT",20)
header("ESP",220)

local aimBtn=btn("Aimbot: Off",20,70)

local fovBox=Instance.new("TextBox",panel)
fovBox.Position=UDim2.fromOffset(20,105)
fovBox.Size=UDim2.fromOffset(180,26)
fovBox.Text=tostring(FOV)
fovBox.PlaceholderText="FOV (max 500)"
fovBox.BackgroundColor3=Color3.fromRGB(25,25,25)
fovBox.TextColor3=Color3.new(1,1,1)
fovBox.Font=Enum.Font.SourceSans
fovBox.TextSize=14
fovBox.ClearTextOnFocus=false

local espBoxBtn=btn("ESP Box: Off",220,70)
local espNameBtn=btn("ESP Name: Off",220,105)

local closeBtn=Instance.new("TextButton",panel)
closeBtn.Size=UDim2.fromOffset(420,30)
closeBtn.Position=UDim2.fromOffset(0,250)
closeBtn.Text="Close Menu"
closeBtn.BackgroundColor3=Color3.fromRGB(180,0,0)
closeBtn.TextColor3=Color3.new(1,1,1)
closeBtn.Font=Enum.Font.SourceSansBold
closeBtn.TextSize=14

local fovGui=Instance.new("ScreenGui",Player.PlayerGui)
fovGui.ResetOnSpawn=false

local circle=Instance.new("Frame",fovGui)
circle.AnchorPoint=Vector2.new(0.5,0.5)
circle.Position=UDim2.fromScale(0.5,0.5)
circle.Size=UDim2.fromOffset(FOV*2,FOV*2)
circle.BackgroundTransparency=1
circle.Visible=true
Instance.new("UICorner",circle).CornerRadius=UDim.new(1,0)

local stroke=Instance.new("UIStroke",circle)
stroke.Thickness=2
stroke.Color=Color3.fromRGB(255,255,255)

local espFolder=Instance.new("Folder",gui)

local function updateFOV()
	FOV=math.clamp(FOV,0,MAX_FOV)
	circle.Size=UDim2.fromOffset(FOV*2,FOV*2)
end
updateFOV()

aimBtn.MouseButton1Click:Connect(function()
	aimbot=not aimbot
	aimBtn.Text="Aimbot: "..(aimbot and "On" or "Off")
end)

espBoxBtn.MouseButton1Click:Connect(function()
	espBox=not espBox
	espBoxBtn.Text="ESP Box: "..(espBox and "On" or "Off")
end)

espNameBtn.MouseButton1Click:Connect(function()
	espName=not espName
	espNameBtn.Text="ESP Name: "..(espName and "On" or "Off")
end)

fovBox.FocusLost:Connect(function()
	local n=tonumber(fovBox.Text)
	if n then
		FOV=math.clamp(math.floor(n),0,MAX_FOV)
		updateFOV()
	end
	fovBox.Text=tostring(FOV)
end)

closeBtn.MouseButton1Click:Connect(function()
	panel.Visible=false
end)

local function visible(pos,part)
	if not visibleCheck then return true end
	local params=RaycastParams.new()
	params.FilterDescendantsInstances={Player.Character}
	params.FilterType=Enum.RaycastFilterType.Blacklist
	local r=workspace:Raycast(Camera.CFrame.Position,(pos-Camera.CFrame.Position),params)
	return r and r.Instance:IsDescendantOf(part.Parent)
end

local function getTarget()
	local best,dist=nil,FOV
	for _,npc in pairs(TARGET_FOLDER:GetChildren()) do
		local hum=npc:FindFirstChild("Humanoid")
		local head=npc:FindFirstChild("Head")
		if hum and head and hum.Health>0 then
			local v,on=Camera:WorldToViewportPoint(head.Position)
			if on then
				local d=(Vector2.new(v.X,v.Y)-Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y/2)).Magnitude
				if d<dist and visible(head.Position,head) then
					dist=d
					best=head
				end
			end
		end
	end
	return best
end

local function clearESP()
	for _,v in pairs(espFolder:GetChildren()) do v:Destroy() end
end

local function drawESP(npc)
	local hrp=npc:FindFirstChild("HumanoidRootPart")
	if not hrp then return end
	if espBox then
		local box=Instance.new("BoxHandleAdornment")
		box.Adornee=hrp
		box.Size=npc:GetExtentsSize()
		box.AlwaysOnTop=true
		box.Color3=Color3.fromRGB(0,255,0)
		box.Transparency=0.5
		box.Parent=espFolder
	end
	if espName then
		local bb=Instance.new("BillboardGui",espFolder)
		bb.Adornee=hrp
		bb.Size=UDim2.fromOffset(100,20)
		bb.AlwaysOnTop=true
		local t=Instance.new("TextLabel",bb)
		t.Size=UDim2.new(1,0,1,0)
		t.BackgroundTransparency=1
		t.Text=npc.Name
		t.TextColor3=Color3.new(1,1,1)
		t.Font=Enum.Font.SourceSansBold
		t.TextSize=12
	end
end

RunService.RenderStepped:Connect(function()
	if aimbot then
		local t=getTarget()
		if t then
			Camera.CFrame=Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position,t.Position),smoothing)
		end
	end
	clearESP()
	if espBox or espName then
		for _,npc in pairs(TARGET_FOLDER:GetChildren()) do
			drawESP(npc)
		end
	end
end)

UIS.InputBegan:Connect(function(i,gp)
	if gp then return end
	if i.KeyCode==Enum.KeyCode.G then
		panel.Visible=not panel.Visible
	end
end)
