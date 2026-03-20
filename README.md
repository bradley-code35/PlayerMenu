local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

-------------------------------------------------
-- CHARACTER HANDLER
-------------------------------------------------
local character = player.Character or player.CharacterAdded:Wait()

player.CharacterAdded:Connect(function(char)
	character = char
end)

local function getHumanoid()
	character = player.Character or player.CharacterAdded:Wait()
	return character:FindFirstChildOfClass("Humanoid")
end

-------------------------------------------------
-- GUI
-------------------------------------------------
local gui = Instance.new("ScreenGui")
gui.Name = "AdminGUI"
gui.Enabled = false
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,320,0,420)
frame.Position = UDim2.new(0.5,-160,0.5,-210)
frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Text = "PLAYER MENU"
title.TextColor3 = Color3.new(1,1,1)
title.TextScaled = true
title.Parent = frame

-------------------------------------------------
-- USERNAME
-------------------------------------------------
local usernameLabel = Instance.new("TextLabel")
usernameLabel.Size = UDim2.new(0.8,0,0,18)
usernameLabel.Position = UDim2.new(0.1,0,0.15,0)
usernameLabel.BackgroundTransparency = 1
usernameLabel.Text = "USERNAME"
usernameLabel.TextColor3 = Color3.new(1,1,1)
usernameLabel.TextScaled = true
usernameLabel.Parent = frame

local box = Instance.new("TextBox")
box.Size = UDim2.new(0.8,0,0,35)
box.Position = UDim2.new(0.1,0,0.20,0)
box.BackgroundColor3 = Color3.fromRGB(25,25,25)
box.TextColor3 = Color3.new(1,1,1)
box.TextScaled = true
box.ClearTextOnFocus = false
box.Parent = frame

-------------------------------------------------
-- BUTTON CREATOR
-------------------------------------------------
local function makeButton(text,y)
	local b = Instance.new("TextButton")
	b.Text = text
	b.Size = UDim2.new(0.8,0,0,35)
	b.Position = UDim2.new(0.1,0,y,0)
	b.BackgroundColor3 = Color3.fromRGB(40,40,40)
	b.TextColor3 = Color3.new(1,1,1)
	b.TextScaled = true
	b.Parent = frame
	return b
end

local tpButton = makeButton("Teleport",0.38)
local flyButton = makeButton("Fly (You)",0.50)
local espButton = makeButton("ESP OFF",0.62)
local noclipButton = makeButton("Noclip",0.74)
local reclipButton = makeButton("Reclip",0.86)

-------------------------------------------------
-- TOGGLE GUI
-------------------------------------------------
UIS.InputBegan:Connect(function(input,gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.U then
		gui.Enabled = not gui.Enabled
	end
end)

-------------------------------------------------
-- FIND PLAYER
-------------------------------------------------
local function findPlayer(name)
	name = string.lower(name)
	for _,plr in pairs(Players:GetPlayers()) do
		if string.sub(string.lower(plr.Name),1,#name) == name then
			return plr
		end
	end
end

-------------------------------------------------
-- TELEPORT
-------------------------------------------------
tpButton.MouseButton1Click:Connect(function()
	local target = findPlayer(box.Text)
	if not target then return end

	local c1 = player.Character
	local c2 = target.Character
	if not (c1 and c2) then return end

	local h1 = c1:FindFirstChild("HumanoidRootPart")
	local h2 = c2:FindFirstChild("HumanoidRootPart")

	if h1 and h2 then
		h1.CFrame = h2.CFrame * CFrame.new(3,0,0)
	end
end)

-------------------------------------------------
-- FLY (FULLY FIXED)
-------------------------------------------------
local flying = false
local flyBV, flyBG
local moveDir = Vector3.zero

UIS.InputBegan:Connect(function(input,gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.W then moveDir += Vector3.new(0,0,-1) end
	if input.KeyCode == Enum.KeyCode.S then moveDir += Vector3.new(0,0,1) end
	if input.KeyCode == Enum.KeyCode.A then moveDir += Vector3.new(-1,0,0) end
	if input.KeyCode == Enum.KeyCode.D then moveDir += Vector3.new(1,0,0) end
	if input.KeyCode == Enum.KeyCode.Space then moveDir += Vector3.new(0,1,0) end
	if input.KeyCode == Enum.KeyCode.LeftControl then moveDir += Vector3.new(0,-1,0) end
end)

UIS.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then moveDir -= Vector3.new(0,0,-1) end
	if input.KeyCode == Enum.KeyCode.S then moveDir -= Vector3.new(0,0,1) end
	if input.KeyCode == Enum.KeyCode.A then moveDir -= Vector3.new(-1,0,0) end
	if input.KeyCode == Enum.KeyCode.D then moveDir -= Vector3.new(1,0,0) end
	if input.KeyCode == Enum.KeyCode.Space then moveDir -= Vector3.new(0,1,0) end
	if input.KeyCode == Enum.KeyCode.LeftControl then moveDir -= Vector3.new(0,-1,0) end
end)

local function stopFly()
	flying = false
	RunService:UnbindFromRenderStep("FlyMove")

	local hum = getHumanoid()
	if hum then hum.PlatformStand = false end

	if flyBV then flyBV:Destroy() flyBV=nil end
	if flyBG then flyBG:Destroy() flyBG=nil end
end

local function startFly()
	local char = player.Character
	if not char then return end

	local hrp = char:WaitForChild("HumanoidRootPart")
	local hum = getHumanoid()
	if not hum then return end

	flying = true
	hum.PlatformStand = true

	flyBV = Instance.new("BodyVelocity")
	flyBV.Name = "FlyVelocity"
	flyBV.MaxForce = Vector3.new(9e9,9e9,9e9)
	flyBV.Velocity = Vector3.zero
	flyBV.Parent = hrp

	flyBG = Instance.new("BodyGyro")
	flyBG.Name = "FlyGyro"
	flyBG.MaxTorque = Vector3.new(9e9,9e9,9e9)
	flyBG.P = 1e5
	flyBG.CFrame = hrp.CFrame
	flyBG.Parent = hrp

	RunService:BindToRenderStep("FlyMove",Enum.RenderPriority.Character.Value,function()
		if not flying then return end

		local cam = workspace.CurrentCamera
		flyBG.CFrame = cam.CFrame
		flyBV.Velocity = cam.CFrame:VectorToWorldSpace(moveDir) * 70
	end)
end

flyButton.MouseButton1Click:Connect(function()
	if flying then
		stopFly()
	else
		startFly()
	end
end)

-------------------------------------------------
-- NOCLIP / RECLIP
-------------------------------------------------
local noclip = false
local noclipConnection

local function setCollision(state)
	local char = player.Character
	if not char then return end

	for _,v in ipairs(char:GetDescendants()) do
		if v:IsA("BasePart") then
			v.CanCollide = state
		end
	end
end

local function startNoclip()
	if noclip then return end
	noclip = true

	noclipConnection = RunService.Stepped:Connect(function()
		setCollision(false)
	end)
end

local function stopNoclip()
	noclip = false
	if noclipConnection then
		noclipConnection:Disconnect()
		noclipConnection = nil
	end
	setCollision(true)
end

noclipButton.MouseButton1Click:Connect(startNoclip)
reclipButton.MouseButton1Click:Connect(stopNoclip)

player.CharacterAdded:Connect(function()
	task.wait(0.5)
	if noclip then startNoclip() end
end)

-------------------------------------------------
-- ESP
-------------------------------------------------
local espEnabled = false
local espFolder = Instance.new("Folder")
espFolder.Parent = gui

local function addESP(plr)
	if plr == player then return end

	local function apply(char)
		if not espEnabled then return end

		local hl = Instance.new("Highlight")
		hl.Adornee = char
		hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
		hl.FillTransparency = 1
		hl.OutlineTransparency = 0
		hl.OutlineColor = Color3.fromRGB(255,0,0)
		hl.Parent = espFolder
	end

	if plr.Character then apply(plr.Character) end
	plr.CharacterAdded:Connect(function(c)
		task.wait(0.5)
		apply(c)
	end)
end

local function enableESP()
	for _,plr in ipairs(Players:GetPlayers()) do
		addESP(plr)
	end
end

espButton.MouseButton1Click:Connect(function()
	espEnabled = not espEnabled
	espButton.Text = espEnabled and "ESP ON" or "ESP OFF"

	if espEnabled then
		enableESP()
	else
		espFolder:ClearAllChildren()
	end
end)

Players.PlayerAdded:Connect(function(plr)
	if espEnabled then addESP(plr) end
end)
