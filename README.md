local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local isMobile = UserInputService:GetLastInputType() == Enum.UserInputType.Touch

if player.PlayerGui:FindFirstChild("StoryGUI") then
	player.PlayerGui:FindFirstChild("StoryGUI"):Destroy()
end

local SaveSystem = {
	FileName = "WaveDuelsConfig.json",
	DefaultConfig = {
		Speed = 58,
		StealSpeed = 29,
		Jump = 80,
		AutoStealRange = 50,
		InfiniteJumpEnabled = false,
		AutoBatEnabled = false,
		XrayBaseEnabled = false,
		NoAnimEnabled = false,
		MeleeAimbotEnabled = false,
		AntiRagdollEnabled = false,
		AutoStealEnabled = false,
		GUISize = isMobile and "small" or "normal",
	}
}

local function loadConfig()
	local success, config = pcall(function()
		local file = readfile(SaveSystem.FileName)
		return game:GetService("HttpService"):JSONDecode(file)
	end)
	if success and config then
		return config
	else
		return SaveSystem.DefaultConfig
	end
end

local function saveConfig(config)
	pcall(function()
		local json = game:GetService("HttpService"):JSONEncode(config)
		writefile(SaveSystem.FileName, json)
	end)
end

local Config = loadConfig()

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "StoryGUI"
ScreenGui.Parent = player.PlayerGui
ScreenGui.ResetOnSpawn = false
ScreenGui.SafeAreaCompatible = true

local IsStealing = false
local StealProgress = 0

-- Ajustar tamanho da GUI baseado no dispositivo
local guiScale = isMobile and 0.7 or 1
local mainFrameSize = isMobile and UDim2.new(0, 250, 0, 380) or UDim2.new(0, 350, 0, 500)
local mainFramePos = isMobile and UDim2.new(0.5, -125, 0.5, -190) or UDim2.new(0.5, -175, 0.5, -250)

local progressBarBg = Instance.new("Frame")
progressBarBg.Name = "ProgressBarBg"
progressBarBg.Size = UDim2.new(0.4, 0, 0, 12)
progressBarBg.Position = UDim2.new(0.3, 0, 0, 8)
progressBarBg.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
progressBarBg.BorderSizePixel = 0
progressBarBg.Parent = ScreenGui
Instance.new("UICorner", progressBarBg).CornerRadius = UDim.new(0, 6)

local progressBarFill = Instance.new("Frame")
progressBarFill.Name = "ProgressBarFill"
progressBarFill.Size = UDim2.new(0, 0, 1, 0)
progressBarFill.BackgroundColor3 = Color3.fromRGB(0, 140, 255)
progressBarFill.BorderSizePixel = 0
progressBarFill.Parent = progressBarBg
Instance.new("UICorner", progressBarFill).CornerRadius = UDim.new(0, 6)

local progressLabel = Instance.new("TextLabel")
progressLabel.Size = UDim2.new(1, 0, 1, 0)
progressLabel.BackgroundTransparency = 1
progressLabel.Text = "0%"
progressLabel.TextColor3 = Color3.new(1, 1, 1)
progressLabel.Font = Enum.Font.GothamBold
progressLabel.TextSize = 10
progressLabel.Parent = progressBarFill

local MainFrame = Instance.new("Frame")
MainFrame.Size = mainFrameSize
MainFrame.Position = mainFramePos
MainFrame.BackgroundColor3 = Color3.fromRGB(8, 20, 50)
MainFrame.BackgroundTransparency = 0.1
MainFrame.Parent = ScreenGui
MainFrame.Active = true
MainFrame.Visible = false

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 20)

local Stroke = Instance.new("UIStroke", MainFrame)
Stroke.Color = Color3.fromRGB(0, 140, 255)
Stroke.Transparency = 0.35
Stroke.Thickness = 2

local UIScale = Instance.new("UIScale", MainFrame)
UIScale.Scale = 0

local function openGui()
	MainFrame.Visible = true
	TweenService:Create(UIScale, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1}):Play()
	TweenService:Create(MainFrame, TweenInfo.new(0.25), {BackgroundTransparency = 0.08}):Play()
end

local function closeGui()
	local tween = TweenService:Create(UIScale, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Scale = 0})
	tween:Play()
	tween.Completed:Wait()
	MainFrame.Visible = false
end

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 45)
Title.BackgroundTransparency = 1
Title.Text = "WAVE DUELS"
Title.TextSize = isMobile and 16 or 22
Title.Font = Enum.Font.GothamBold
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Parent = MainFrame

local Underline = Instance.new("Frame")
Underline.Size = UDim2.new(0.5, 0, 0, 2)
Underline.Position = UDim2.new(0.25, 0, 1, 0)
Underline.BackgroundColor3 = Color3.fromRGB(0, 140, 255)
Underline.Parent = Title

local UnderlineStroke = Instance.new("UIStroke", Underline)
UnderlineStroke.Color = Color3.fromRGB(0, 140, 255)
UnderlineStroke.Transparency = 0.5
UnderlineStroke.Thickness = 1

local GearButton = Instance.new("TextButton")
GearButton.Size = UDim2.new(0, 30, 0, 30)
GearButton.Position = UDim2.new(1, -40, 0, 7)
GearButton.BackgroundTransparency = 1
GearButton.Text = "⚙️"
GearButton.Font = Enum.Font.SourceSans
GearButton.TextSize = 24
GearButton.TextColor3 = Color3.new(1, 1, 1)
GearButton.Parent = MainFrame

local SettingsFrame = Instance.new("Frame")
SettingsFrame.Size = isMobile and UDim2.new(0, 250, 0, 400) or UDim2.new(0, 350, 0, 520)
SettingsFrame.Position = isMobile and UDim2.new(0.5, 130, 0.5, -200) or UDim2.new(0.5, 200, 0.5, -260)
SettingsFrame.BackgroundColor3 = Color3.fromRGB(8, 20, 50)
SettingsFrame.BackgroundTransparency = 0.1
SettingsFrame.Parent = ScreenGui
SettingsFrame.Active = true
SettingsFrame.Visible = false

Instance.new("UICorner", SettingsFrame).CornerRadius = UDim.new(0, 20)

local SettingsStroke = Instance.new("UIStroke", SettingsFrame)
SettingsStroke.Color = Color3.fromRGB(0, 140, 255)
SettingsStroke.Transparency = 0.35
SettingsStroke.Thickness = 2

local SettingsUIScale = Instance.new("UIScale", SettingsFrame)
SettingsUIScale.Scale = 0

local function openSettingsGui()
	SettingsFrame.Visible = true
	TweenService:Create(SettingsUIScale, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Scale = 1}):Play()
	TweenService:Create(SettingsFrame, TweenInfo.new(0.25), {BackgroundTransparency = 0.08}):Play()
end

local function closeSettingsGui()
	local tween = TweenService:Create(SettingsUIScale, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Scale = 0})
	tween:Play()
	tween.Completed:Wait()
	SettingsFrame.Visible = false
end

local SettingsTitle = Instance.new("TextLabel")
SettingsTitle.Size = UDim2.new(1, 0, 0, 40)
SettingsTitle.BackgroundTransparency = 1
SettingsTitle.Text = "⚡ SPEED"
SettingsTitle.TextSize = isMobile and 14 or 22
SettingsTitle.Font = Enum.Font.GothamBold
SettingsTitle.TextColor3 = Color3.new(1, 1, 1)
SettingsTitle.Parent = SettingsFrame

local SettingsUnderline = Instance.new("Frame")
SettingsUnderline.Size = UDim2.new(0.5, 0, 0, 2)
SettingsUnderline.Position = UDim2.new(0.25, 0, 1, 0)
SettingsUnderline.BackgroundColor3 = Color3.fromRGB(0, 140, 255)
SettingsUnderline.Parent = SettingsTitle

local SettingsUnderlineStroke = Instance.new("UIStroke", SettingsUnderline)
SettingsUnderlineStroke.Color = Color3.fromRGB(0, 140, 255)
SettingsUnderlineStroke.Transparency = 0.5
SettingsUnderlineStroke.Thickness = 1

-- BOTÃO SALVAR CONFIGURAÇÃO
local SaveButton = Instance.new("TextButton")
SaveButton.Size = UDim2.new(1, -30, 0, 35)
SaveButton.Position = UDim2.new(0, 15, 1, -50)
SaveButton.BackgroundColor3 = Color3.fromRGB(0, 140, 255)
SaveButton.TextColor3 = Color3.new(1, 1, 1)
SaveButton.Text = "💾 SALVAR CONFIG"
SaveButton.Font = Enum.Font.GothamBold
SaveButton.TextSize = isMobile and 11 or 14
SaveButton.Parent = SettingsFrame
Instance.new("UICorner", SaveButton).CornerRadius = UDim.new(0, 8)

SaveButton.MouseButton1Click:Connect(function()
	saveConfig(Config)
	SaveButton.Text = "✅ SALVO!"
	task.wait(1.5)
	SaveButton.Text = "💾 SALVAR CONFIG"
end)

local SpeedCustomizer = {
	Enabled = false,
	Speed = Config.Speed,
	StealSpeed = Config.StealSpeed,
	Jump = Config.Jump,
	HeartbeatConn = nil,
	JumpConn = nil,
	character = nil,
	hrp = nil,
	hum = nil
}

local function setupSpeedCharacter(char)
	SpeedCustomizer.character = char
	SpeedCustomizer.hrp = char:FindFirstChild("HumanoidRootPart")
	SpeedCustomizer.hum = char:FindFirstChildOfClass("Humanoid")
end

local function createSpeedInput(y, labelText, defaultValue)
	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, 0, 0, 16)
	label.Position = UDim2.new(0, 15, 0, y)
	label.BackgroundTransparency = 1
	label.Text = labelText
	label.TextColor3 = Color3.fromRGB(0, 140, 255)
	label.TextSize = isMobile and 10 or 12
	label.Font = Enum.Font.GothamBold
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = SettingsFrame
	
	local box = Instance.new("TextBox")
	box.Size = UDim2.new(1, -30, 0, 28)
	box.Position = UDim2.new(0, 15, 0, y + 18)
	box.BackgroundColor3 = Color3.fromRGB(20, 25, 35)
	box.Text = tostring(defaultValue)
	box.TextColor3 = Color3.fromRGB(255, 255, 255)
	box.Font = Enum.Font.Gotham
	box.TextSize = isMobile and 12 or 14
	box.Parent = SettingsFrame
	Instance.new("UICorner", box).CornerRadius = UDim.new(0, 6)
	
	local boxStroke = Instance.new("UIStroke", box)
	boxStroke.Color = Color3.fromRGB(0, 140, 255)
	boxStroke.Transparency = 0.6
	boxStroke.Thickness = 1.5
	
	return box, label
end

local speedBox, speedLabel = createSpeedInput(45, "🚀 Walk Speed", SpeedCustomizer.Speed)
local stealBox, stealLabel = createSpeedInput(100, "👻 Steal Speed", SpeedCustomizer.StealSpeed)
local jumpBox, jumpLabel = createSpeedInput(155, "⬆️ Jump Power", SpeedCustomizer.Jump)

for _, data in ipairs({
	{speedBox, "Speed", speedLabel},
	{stealBox, "Steal", stealLabel},
	{jumpBox, "Jump", jumpLabel}
}) do
	data[1].FocusLost:Connect(function()
		local val = tonumber(data[1].Text)
		if val and val > 0 then
			val = math.clamp(val, 1, 250)
			if data[2] == "Speed" then
				SpeedCustomizer.Speed = val
				Config.Speed = val
			elseif data[2] == "Steal" then
				SpeedCustomizer.StealSpeed = val
				Config.StealSpeed = val
			elseif data[2] == "Jump" then
				SpeedCustomizer.Jump = val
				Config.Jump = val
			end
			data[1].Text = tostring(val)
			saveConfig(Config)
		else
			if data[2] == "Speed" then data[1].Text = tostring(SpeedCustomizer.Speed)
			elseif data[2] == "Steal" then data[1].Text = tostring(SpeedCustomizer.StealSpeed)
			elseif data[2] == "Jump" then data[1].Text = tostring(SpeedCustomizer.Jump)
			end
		end
	end)
end

local autoStealRange = Config.AutoStealRange or 50
local autoStealRangeBox = nil

local rangeLabel = Instance.new("TextLabel")
rangeLabel.Size = UDim2.new(1, 0, 0, 16)
rangeLabel.Position = UDim2.new(0, 15, 0, 210)
rangeLabel.BackgroundTransparency = 1
rangeLabel.Text = "🎯 Steal Range"
rangeLabel.TextColor3 = Color3.fromRGB(0, 140, 255)
rangeLabel.TextSize = isMobile and 10 or 12
rangeLabel.Font = Enum.Font.GothamBold
rangeLabel.TextXAlignment = Enum.TextXAlignment.Left
rangeLabel.Parent = SettingsFrame

autoStealRangeBox = Instance.new("TextBox")
autoStealRangeBox.Size = UDim2.new(1, -30, 0, 28)
autoStealRangeBox.Position = UDim2.new(0, 15, 0, 228)
autoStealRangeBox.BackgroundColor3 = Color3.fromRGB(20, 25, 35)
autoStealRangeBox.Text = tostring(autoStealRange)
autoStealRangeBox.TextColor3 = Color3.fromRGB(255, 255, 255)
autoStealRangeBox.Font = Enum.Font.Gotham
autoStealRangeBox.TextSize = isMobile and 12 or 14
autoStealRangeBox.Parent = SettingsFrame
Instance.new("UICorner", autoStealRangeBox).CornerRadius = UDim.new(0, 6)

local rangeStroke = Instance.new("UIStroke", autoStealRangeBox)
rangeStroke.Color = Color3.fromRGB(0, 140, 255)
rangeStroke.Transparency = 0.6
rangeStroke.Thickness = 1.5

autoStealRangeBox.FocusLost:Connect(function()
	local val = tonumber(autoStealRangeBox.Text)
	if val and val > 0 then
		val = math.clamp(val, 5, 200)
		autoStealRange = val
		Config.AutoStealRange = val
		autoStealRangeBox.Text = tostring(val)
		saveConfig(Config)
	else
		autoStealRangeBox.Text = tostring(autoStealRange)
	end
end)

local function enableSpeed()
	if SpeedCustomizer.Enabled then return end
	SpeedCustomizer.Enabled = true
	
	setupSpeedCharacter(player.Character or player.CharacterAdded:Wait())
	player.CharacterAdded:Connect(setupSpeedCharacter)
	
	SpeedCustomizer.HeartbeatConn = RunService.Heartbeat:Connect(function()
		if not SpeedCustomizer.Enabled then return end
		
		local c, h, hrp = SpeedCustomizer.character, SpeedCustomizer.hum, SpeedCustomizer.hrp
		if not (c and h and hrp) then return end
		
		local moveDir = h.MoveDirection
		if moveDir.Magnitude > 0 then
			local isSteal = h.WalkSpeed < 25
			local curSpeed = isSteal and SpeedCustomizer.StealSpeed or SpeedCustomizer.Speed
			
			hrp.AssemblyLinearVelocity = Vector3.new(
				moveDir.X * curSpeed,
				hrp.AssemblyLinearVelocity.Y,
				moveDir.Z * curSpeed
			)
		end
	end)
	
	SpeedCustomizer.JumpConn = UserInputService.JumpRequest:Connect(function()
		if SpeedCustomizer.Enabled then
			local c, h, hrp = SpeedCustomizer.character, SpeedCustomizer.hum, SpeedCustomizer.hrp
			if c and h and hrp and h.FloorMaterial ~= Enum.Material.Air then
				hrp.AssemblyLinearVelocity = Vector3.new(
					hrp.AssemblyLinearVelocity.X,
					SpeedCustomizer.Jump,
					hrp.AssemblyLinearVelocity.Z
				)
			end
		end
	end)
end

local function disableSpeed()
	if not SpeedCustomizer.Enabled then return end
	SpeedCustomizer.Enabled = false
	
	if SpeedCustomizer.HeartbeatConn then
		SpeedCustomizer.HeartbeatConn:Disconnect()
		SpeedCustomizer.HeartbeatConn = nil
	end
	if SpeedCustomizer.JumpConn then
		SpeedCustomizer.JumpConn:Disconnect()
		SpeedCustomizer.JumpConn = nil
	end
end

local speedToggleBtn = Instance.new("TextButton")
speedToggleBtn.Size = UDim2.new(1, -30, 0, 38)
speedToggleBtn.Position = UDim2.new(0, 15, 0, 280)
speedToggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
speedToggleBtn.TextColor3 = Color3.new(1, 1, 1)
speedToggleBtn.Text = "SPEED OFF"
speedToggleBtn.Font = Enum.Font.GothamBold
speedToggleBtn.TextSize = isMobile and 12 or 14
speedToggleBtn.Parent = SettingsFrame
Instance.new("UICorner", speedToggleBtn).CornerRadius = UDim.new(0, 8)

speedToggleBtn.MouseButton1Click:Connect(function()
	if SpeedCustomizer.Enabled then
		disableSpeed()
		speedToggleBtn.Text = "SPEED OFF"
		speedToggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
	else
		enableSpeed()
		speedToggleBtn.Text = "SPEED ON"
		speedToggleBtn.BackgroundColor3 = Color3.fromRGB(0, 140, 255)
	end
end)

local settingsOpened = false
GearButton.MouseButton1Click:Connect(function()
	TweenService:Create(GearButton, TweenInfo.new(0.5, Enum.EasingStyle.Linear), {Rotation = GearButton.Rotation + 360}):Play()
	settingsOpened = not settingsOpened
	if settingsOpened then
		openSettingsGui()
	else
		closeSettingsGui()
	end
end)

local Container = Instance.new("Frame")
Container.Size = UDim2.new(1, -20, 1, -70)
Container.Position = UDim2.new(0, 10, 0, 60)
Container.BackgroundTransparency = 1
Container.Parent = MainFrame

local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Size = UDim2.new(1, 0, 1, 0)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.ScrollBarThickness = 3
ScrollFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 140, 255)
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 1)
ScrollFrame.Parent = Container

local GridLayout = Instance.new("UIGridLayout")
GridLayout.Parent = ScrollFrame
GridLayout.CellSize = isMobile and UDim2.new(0.33, 0, 0, 45) or UDim2.new(0.33, 0, 0, 50)
GridLayout.CellPadding = UDim2.new(0.05, 0, 0.05, 0)
GridLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

ScrollFrame.ChildAdded:Connect(function()
	local size = GridLayout.AbsoluteContentSize
	ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, size.Y)
end)

local FAST_SPEED = 58
local SLOW_SPEED = 29
local ARRIVE_THRESHOLD = 1
local MOVE_TIMEOUT = 8

local speedConnection = nil
local running = false

local rightPositions = {
	Vector3.new(-472.7, -7.3, 95.4),
	Vector3.new(-482.4, -5.4, 96.9),
	Vector3.new(-472.7, -7.3, 95.7),
	Vector3.new(-473.0, -7.3, 24.0),
}

local leftPositions = {
	Vector3.new(-475.0, -7.0, 27.4),
	Vector3.new(-483.2, -5.4, 22.4),
	Vector3.new(-473.5, -7.3, 24.5),
	Vector3.new(-471.2, -7.3, 97.3),
}

local TweenState = {
	DireitoEnabled = false,
	EsquerdoEnabled = false,
	usePosition3And4 = false
}

local function startSpeed(target, speed)
	if speedConnection then
		speedConnection:Disconnect()
	end

	speedConnection = RunService.RenderStepped:Connect(function()
		local char = player.Character
		if not char then return end
		local root = char:FindFirstChild("HumanoidRootPart")
		if not root then return end

		local direction = (target - root.Position)

		if direction.Magnitude > 0.5 then
			direction = direction.Unit
			root.AssemblyLinearVelocity = Vector3.new(
				direction.X * speed,
				root.AssemblyLinearVelocity.Y,
				direction.Z * speed
			)
		else
			root.AssemblyLinearVelocity = Vector3.zero
		end

		root.AssemblyAngularVelocity = Vector3.zero
	end)
end

local function stopSpeed()
	if speedConnection then
		speedConnection:Disconnect()
		speedConnection = nil
	end
end

local function moveTo(root, target)
	local startTime = tick()

	while tick() - startTime < MOVE_TIMEOUT do
		if not TweenState.DireitoEnabled and not TweenState.EsquerdoEnabled then
			stopSpeed()
			return false
		end
		
		local dist = (root.Position - target).Magnitude
		if dist <= ARRIVE_THRESHOLD then
			root.CFrame = CFrame.new(target)
			root.AssemblyLinearVelocity = Vector3.zero
			return true
		end
		task.wait(0.03)
	end
	return false
end

local function tweenDireito()
	if running then return end
	running = true
	TweenState.DireitoEnabled = true

	local char = player.Character
	if not char then 
		running = false
		TweenState.DireitoEnabled = false
		return 
	end
	local root = char:FindFirstChild("HumanoidRootPart")
	if not root then 
		running = false
		TweenState.DireitoEnabled = false
		return 
	end

	startSpeed(rightPositions[1], FAST_SPEED)
	if not moveTo(root, rightPositions[1]) then 
		stopSpeed() 
		running = false
		TweenState.DireitoEnabled = false
		return 
	end

	if not TweenState.DireitoEnabled then
		stopSpeed()
		running = false
		return
	end

	startSpeed(rightPositions[2], FAST_SPEED)
	if not moveTo(root, rightPositions[2]) then 
		stopSpeed() 
		running = false
		TweenState.DireitoEnabled = false
		return 
	end

	if not TweenState.DireitoEnabled then
		stopSpeed()
		running = false
		return
	end

	stopSpeed()
	task.wait(0.2)

	if TweenState.usePosition3And4 and TweenState.DireitoEnabled then
		startSpeed(rightPositions[3], SLOW_SPEED)
		if not moveTo(root, rightPositions[3]) then 
			stopSpeed() 
			running = false
			TweenState.DireitoEnabled = false
			return 
		end

		if not TweenState.DireitoEnabled then
			stopSpeed()
			running = false
			return
		end

		startSpeed(rightPositions[4], SLOW_SPEED)
		moveTo(root, rightPositions[4])
	end

	stopSpeed()
	running = false
	TweenState.DireitoEnabled = false
end

local function tweenEsquerdo()
	if running then return end
	running = true
	TweenState.EsquerdoEnabled = true

	local char = player.Character
	if not char then 
		running = false
		TweenState.EsquerdoEnabled = false
		return 
	end
	local root = char:FindFirstChild("HumanoidRootPart")
	if not root then 
		running = false
		TweenState.EsquerdoEnabled = false
		return 
	end

	startSpeed(leftPositions[1], FAST_SPEED)
	if not moveTo(root, leftPositions[1]) then 
		stopSpeed() 
		running = false
		TweenState.EsquerdoEnabled = false
		return 
	end

	if not TweenState.EsquerdoEnabled then
		stopSpeed()
		running = false
		return
	end

	startSpeed(leftPositions[2], FAST_SPEED)
	if not moveTo(root, leftPositions[2]) then 
		stopSpeed() 
		running = false
		TweenState.EsquerdoEnabled = false
		return 
	end

	if not TweenState.EsquerdoEnabled then
		stopSpeed()
		running = false
		return
	end

	stopSpeed()
	task.wait(0.2)

	if TweenState.usePosition3And4 and TweenState.EsquerdoEnabled then
		startSpeed(leftPositions[3], SLOW_SPEED)
		if not moveTo(root, leftPositions[3]) then 
			stopSpeed() 
			running = false
			TweenState.EsquerdoEnabled = false
			return 
		end

		if not TweenState.EsquerdoEnabled then
			stopSpeed()
			running = false
			return
		end

		startSpeed(leftPositions[4], SLOW_SPEED)
		moveTo(root, leftPositions[4])
	end

	stopSpeed()
	running = false
	TweenState.EsquerdoEnabled = false
end

local AntiRagdoll = {
	Enabled = false,
	connection = nil,
	keysPressed = {W=false,A=false,S=false,D=false,Space=false},
	lastSafePos = nil,
	lastSafeTime = 0
}

local function isBadState(state)
	return state == Enum.HumanoidStateType.Ragdoll
		or state == Enum.HumanoidStateType.Physics
		or state == Enum.HumanoidStateType.FallingDown
		or state == Enum.HumanoidStateType.PlatformStanding
end

local function startAntiRagdollSystem()
	if AntiRagdoll.connection then return end

	AntiRagdoll.connection = RunService.Heartbeat:Connect(function()
		if not AntiRagdoll.Enabled then return end

		local char = player.Character
		if not char then return end

		local humanoid = char:FindFirstChildOfClass("Humanoid")
		local root = char:FindFirstChild("HumanoidRootPart")
		if not humanoid or not root then return end

		local state = humanoid:GetState()
		local vel = root.AssemblyLinearVelocity

		if humanoid.FloorMaterial ~= Enum.Material.Air
			and not isBadState(state)
			and vel.Magnitude < 25
			and tick() - AntiRagdoll.lastSafeTime > 0.3 then
			AntiRagdoll.lastSafePos = root.Position
			AntiRagdoll.lastSafeTime = tick()
		end

		if isBadState(state) then
			humanoid.PlatformStand = false
			humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
			task.wait(0.1)

			for _, m in char:GetDescendants() do
				if m:IsA("Motor6D") and not m.Enabled then
					m.Enabled = true
				end
			end

			if AntiRagdoll.lastSafePos then
				root.CFrame = CFrame.new(AntiRagdoll.lastSafePos)
				root.AssemblyLinearVelocity = Vector3.new(0, 5, 0)
			end
		end

		humanoid.PlatformStand = false
		humanoid.Sit = false

		local move = Vector3.new()
		if AntiRagdoll.keysPressed.W then move += Vector3.new(0,0,-1) end
		if AntiRagdoll.keysPressed.S then move += Vector3.new(0,0,1) end
		if AntiRagdoll.keysPressed.A then move += Vector3.new(-1,0,0) end
		if AntiRagdoll.keysPressed.D then move += Vector3.new(1,0,0) end

		if move.Magnitude > 0 then
			humanoid:Move(move.Unit, true)
		end

		humanoid.Jump = AntiRagdoll.keysPressed.Space
	end)
end

local function stopAntiRagdollSystem()
	if AntiRagdoll.connection then
		AntiRagdoll.connection:Disconnect()
		AntiRagdoll.connection = nil
	end
end

local function enableAntiRagdoll()
	if AntiRagdoll.Enabled then return end
	AntiRagdoll.Enabled = true
	Config.AntiRagdollEnabled = true
	saveConfig(Config)
	startAntiRagdollSystem()
end

local function disableAntiRagdoll()
	if not AntiRagdoll.Enabled then return end
	AntiRagdoll.Enabled = false
	Config.AntiRagdollEnabled = false
	saveConfig(Config)
	stopAntiRagdollSystem()
end

UserInputService.InputBegan:Connect(function(input,gp)
	if gp then return end
	local kc = input.KeyCode
	if kc == Enum.KeyCode.W then AntiRagdoll.keysPressed.W = true
	elseif kc == Enum.KeyCode.A then AntiRagdoll.keysPressed.A = true
	elseif kc == Enum.KeyCode.S then AntiRagdoll.keysPressed.S = true
	elseif kc == Enum.KeyCode.D then AntiRagdoll.keysPressed.D = true
	elseif kc == Enum.KeyCode.Space then AntiRagdoll.keysPressed.Space = true
	end
end)

UserInputService.InputEnded:Connect(function(input)
	local kc = input.KeyCode
	if kc == Enum.KeyCode.W then AntiRagdoll.keysPressed.W = false
	elseif kc == Enum.KeyCode.A then AntiRagdoll.keysPressed.A = false
	elseif kc == Enum.KeyCode.S then AntiRagdoll.keysPressed.S = false
	elseif kc == Enum.KeyCode.D then AntiRagdoll.keysPressed.D = false
	elseif kc == Enum.KeyCode.Space then AntiRagdoll.keysPressed.Space = false
	end
end)

player.CharacterAdded:Connect(function()
	task.wait(0.6)
	if AntiRagdoll.Enabled then
		startAntiRagdollSystem()
	end
end)

local InfiniteJump = { Enabled = false, JumpConnection = nil }
local function enableInfJump()
	if InfiniteJump.Enabled then return end
	InfiniteJump.Enabled = true
	Config.InfiniteJumpEnabled = true
	saveConfig(Config)
	
	if InfiniteJump.JumpConnection then
		InfiniteJump.JumpConnection:Disconnect()
	end
	
	InfiniteJump.JumpConnection = UserInputService.JumpRequest:Connect(function()
		local char = player.Character
		if char and InfiniteJump.Enabled then
			local hrp = char:FindFirstChild("HumanoidRootPart")
			local hum = char:FindFirstChildOfClass("Humanoid")
			if hrp and hum then
				hrp.Velocity = Vector3.new(hrp.Velocity.X, 50, hrp.Velocity.Z)
			end
		end
	end)
end

local function disableInfJump()
	InfiniteJump.Enabled = false
	Config.InfiniteJumpEnabled = false
	saveConfig(Config)
	if InfiniteJump.JumpConnection then
		InfiniteJump.JumpConnection:Disconnect()
		InfiniteJump.JumpConnection = nil
	end
end

local AutoBat = { Enabled = false, Connection = nil }
local function getWeapon()
	return player.Backpack:FindFirstChild("Bat") or (player.Character and player.Character:FindFirstChild("Bat"))
end

local function enableAutoBat()
	if AutoBat.Enabled then return end
	AutoBat.Enabled = true
	Config.AutoBatEnabled = true
	saveConfig(Config)
	AutoBat.Connection = RunService.Heartbeat:Connect(function()
		if not AutoBat.Enabled then return end
		local char = player.Character
		local bat = getWeapon()
		if bat and char and char:FindFirstChild("Humanoid") then
			if bat.Parent ~= char then
				char.Humanoid:EquipTool(bat)
			end
			bat:Activate()
		end
	end)
end

local function disableAutoBat()
	AutoBat.Enabled = false
	Config.AutoBatEnabled = false
	saveConfig(Config)
	if AutoBat.Connection then
		AutoBat.Connection:Disconnect()
		AutoBat.Connection = nil
	end
end

local XrayBase = { Enabled = false, OriginalTransparency = {}, Connections = {} }
local function isPlayerBase(obj)
	if not (obj:IsA("BasePart") or obj:IsA("MeshPart") or obj:IsA("UnionOperation")) then return false end
	local n = obj.Name:lower()
	local p = obj.Parent and obj.Parent.Name:lower() or ""
	for _, keyword in ipairs({"base", "claim"}) do
		if n:find(keyword) or p:find(keyword) then return true end
	end
	return false
end

local function enableXrayBase()
	if XrayBase.Enabled then return end
	XrayBase.Enabled = true
	Config.XrayBaseEnabled = true
	saveConfig(Config)
	
	for _, conn in pairs(XrayBase.Connections) do conn:Disconnect() end
	XrayBase.Connections = {}
	XrayBase.OriginalTransparency = {}
	
	task.wait(0.1)
	
	for _, obj in ipairs(Workspace:GetDescendants()) do
		if isPlayerBase(obj) then
			XrayBase.OriginalTransparency[obj] = obj.LocalTransparencyModifier
			obj.LocalTransparencyModifier = 0.8
		end
	end
	
	XrayBase.Connections.descendantAdded = Workspace.DescendantAdded:Connect(function(obj)
		if XrayBase.Enabled and isPlayerBase(obj) and not XrayBase.OriginalTransparency[obj] then
			XrayBase.OriginalTransparency[obj] = obj.LocalTransparencyModifier
			obj.LocalTransparencyModifier = 0.8
		end
	end)
end

local function disableXrayBase()
	if not XrayBase.Enabled then return end
	XrayBase.Enabled = false
	Config.XrayBaseEnabled = false
	saveConfig(Config)
	for _, conn in pairs(XrayBase.Connections) do conn:Disconnect() end
	XrayBase.Connections = {}
	for obj, original in pairs(XrayBase.OriginalTransparency) do
		if obj and obj.Parent then obj.LocalTransparencyModifier = original end
	end
	XrayBase.OriginalTransparency = {}
end

local NoAnim = { Enabled = false, CharacterConnections = {} }
local function disableAnims(character)
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if not humanoid then return end
	for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
		track:Stop()
	end
	if NoAnim.CharacterConnections[character] then
		NoAnim.CharacterConnections[character]:Disconnect()
	end
	NoAnim.CharacterConnections[character] = humanoid.AnimationPlayed:Connect(function(track)
		if NoAnim.Enabled then
			track:Stop()
		end
	end)
end

local function enableNoAnim()
	if NoAnim.Enabled then return end
	NoAnim.Enabled = true
	Config.NoAnimEnabled = true
	saveConfig(Config)
	if player.Character then disableAnims(player.Character) end
	if NoAnim.CharacterConnections.characterAdded then
		NoAnim.CharacterConnections.characterAdded:Disconnect()
	end
	NoAnim.CharacterConnections.characterAdded = player.CharacterAdded:Connect(function(char)
		task.wait(0.1)
		if NoAnim.Enabled then
			disableAnims(char)
		end
	end)
end

local function disableNoAnim()
	if not NoAnim.Enabled then return end
	NoAnim.Enabled = false
	Config.NoAnimEnabled = false
	saveConfig(Config)
	if NoAnim.CharacterConnections.characterAdded then
		NoAnim.CharacterConnections.characterAdded:Disconnect()
		NoAnim.CharacterConnections.characterAdded = nil
	end
	for character, conn in pairs(NoAnim.CharacterConnections) do
		if character ~= "characterAdded" and conn then
			conn:Disconnect()
			NoAnim.CharacterConnections[character] = nil
		end
	end
end

local MeleeAimbot = { Enabled = false, Connection = nil, Attach = nil, Align = nil, Circle = nil }
local function enableMeleeAimbot()
	if MeleeAimbot.Enabled then return end
	MeleeAimbot.Enabled = true
	Config.MeleeAimbotEnabled = true
	saveConfig(Config)
	local char = player.Character or player.CharacterAdded:Wait()
	local hrp = char:WaitForChild("HumanoidRootPart")
	
	MeleeAimbot.Attach = Instance.new("Attachment", hrp)
	MeleeAimbot.Align = Instance.new("AlignOrientation", hrp)
	MeleeAimbot.Align.Attachment0 = MeleeAimbot.Attach
	MeleeAimbot.Align.Mode = Enum.OrientationAlignmentMode.OneAttachment
	MeleeAimbot.Align.RigidityEnabled = true
	
	MeleeAimbot.Circle = Instance.new("Part")
	MeleeAimbot.Circle.Shape = "Cylinder"
	MeleeAimbot.Circle.Material = "Neon"
	MeleeAimbot.Circle.Size = Vector3.new(0.05, 14.5, 14.5)
	MeleeAimbot.Circle.Color = Color3.new(1, 0, 0)
	MeleeAimbot.Circle.CanCollide = false
	MeleeAimbot.Circle.Massless = true
	MeleeAimbot.Circle.Parent = Workspace
	
	local weld = Instance.new("Weld")
	weld.Part0 = hrp
	weld.Part1 = MeleeAimbot.Circle
	weld.C0 = CFrame.new(0, -1, 0) * CFrame.Angles(0, 0, math.rad(90))
	weld.Parent = MeleeAimbot.Circle
	
	MeleeAimbot.Connection = RunService.RenderStepped:Connect(function()
		if not MeleeAimbot.Enabled or not player.Character then return end
		
		local target, dmin = nil, 7.25
		for _, p in ipairs(Players:GetPlayers()) do
			if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
				local d = (p.Character.HumanoidRootPart.Position - hrp.Position).Magnitude
				if d <= dmin then
					target = p.Character.HumanoidRootPart
					dmin = d
				end
			end
		end
		
		if target then
			player.Character.Humanoid.AutoRotate = false
			MeleeAimbot.Align.Enabled = true
			MeleeAimbot.Align.CFrame = CFrame.lookAt(hrp.Position, Vector3.new(target.Position.X, hrp.Position.Y, target.Position.Z))
			local bat = player.Character:FindFirstChild("Bat")
			if bat then bat:Activate() end
		else
			MeleeAimbot.Align.Enabled = false
			player.Character.Humanoid.AutoRotate = true
		end
	end)
end

local function disableMeleeAimbot()
	MeleeAimbot.Enabled = false
	Config.MeleeAimbotEnabled = false
	saveConfig(Config)
	if MeleeAimbot.Connection then MeleeAimbot.Connection:Disconnect() end
	if MeleeAimbot.Circle then MeleeAimbot.Circle:Destroy() end
	if MeleeAimbot.Align then MeleeAimbot.Align:Destroy() end
	if MeleeAimbot.Attach then MeleeAimbot.Attach:Destroy() end
	if player.Character and player.Character:FindFirstChild("Humanoid") then
		player.Character.Humanoid.AutoRotate = true
	end
end

local AutoSteal = { Enabled = false }
local AnimalsData = nil
pcall(function()
	AnimalsData = require(ReplicatedStorage:WaitForChild("Datas"):WaitForChild("Animals"))
end)

local allAnimalsCache = {}
local PromptMemoryCache = {}
local InternalStealCache = {}

local function getHRP()
	local char = player.Character
	if not char then return nil end
	return char:FindFirstChild("HumanoidRootPart")
end

local function isMyBase(plotName)
	local plot = Workspace:FindFirstChild("Plots") and Workspace.Plots:FindFirstChild(plotName)
	if not plot then return false end
	local sign = plot:FindFirstChild("PlotSign")
	if sign then
		local yourBase = sign:FindFirstChild("YourBase")
		if yourBase and yourBase:IsA("BillboardGui") then
			return yourBase.Enabled == true
		end
	end
	return false
end

local function scanSinglePlot(plot)
	if not plot or not plot:IsA("Model") then return end
	if isMyBase(plot.Name) then return end

	local podiums = plot:FindFirstChild("AnimalPodiums") or plot:FindFirstChild("Podiums")
	if not podiums then return end

	for _, podium in ipairs(podiums:GetChildren()) do
		if podium:IsA("Model") and podium:FindFirstChild("Base") then
			local animalName = "Unknown"
			local spawn = podium.Base:FindFirstChild("Spawn")
			if spawn then
				for _, child in ipairs(spawn:GetChildren()) do
					if child:IsA("Model") and child.Name ~= "PromptAttachment" then
						animalName = child.Name
						if AnimalsData and AnimalsData[animalName] and AnimalsData[animalName].DisplayName then
							animalName = AnimalsData[animalName].DisplayName
						end
						break
					end
				end
			end

			table.insert(allAnimalsCache, {
				name = animalName,
				plot = plot.Name,
				slot = podium.Name,
				worldPosition = (pcall(function() return podium:GetPivot().Position end) and podium:GetPivot().Position) or podium.Position or Vector3.new(),
				uid = plot.Name .. "_" .. podium.Name,
			})
		end
	end
end

local function initializeScanner()
	task.wait(0.5)
	local plots = Workspace:FindFirstChild("Plots")
	if not plots then return end

	allAnimalsCache = {}
	for _, plot in ipairs(plots:GetChildren()) do
		if plot:IsA("Model") then
			scanSinglePlot(plot)
		end
	end

	plots.ChildAdded:Connect(function(plot)
		if plot:IsA("Model") then
			task.wait(0.5)
			scanSinglePlot(plot)
		end
	end)

	task.spawn(function()
		while task.wait(4) do
			allAnimalsCache = {}
			for _, plot in ipairs(plots:GetChildren()) do
				if plot:IsA("Model") then
					scanSinglePlot(plot)
				end
			end
		end
	end)
end

local function findProximityPromptForAnimal(animalData)
	if not animalData then return nil end
	local cachedPrompt = PromptMemoryCache[animalData.uid]
	if cachedPrompt and cachedPrompt.Parent then return cachedPrompt end

	local plot = Workspace:FindFirstChild("Plots") and Workspace.Plots:FindFirstChild(animalData.plot)
	if not plot then return nil end

	local podiums = plot:FindFirstChild("AnimalPodiums") or plot:FindFirstChild("Podiums")
	if not podiums then return nil end

	local podium = podiums:FindFirstChild(animalData.slot)
	if not podium then return nil end

	local base = podium:FindFirstChild("Base")
	if not base then return nil end

	local spawn = base:FindFirstChild("Spawn")
	if not spawn then return nil end

	local attach = spawn:FindFirstChild("PromptAttachment")
	if not attach then return nil end

	for _, p in ipairs(attach:GetChildren()) do
		if p:IsA("ProximityPrompt") then
			PromptMemoryCache[animalData.uid] = p
			return p
		end
	end

	return nil
end

local function shouldSteal(animalData)
	if not animalData or not animalData.worldPosition then return false end
	local hrp = getHRP()
	if not hrp then return false end
	local currentDistance = (hrp.Position - animalData.worldPosition).Magnitude
	return currentDistance <= autoStealRange
end

local function buildStealCallbacks(prompt)
	if InternalStealCache[prompt] then return end
	local data = { holdCallbacks = {}, triggerCallbacks = {}, ready = true }

	local ok1, conns1 = pcall(getconnections, prompt.PromptButtonHoldBegan)
	if ok1 and type(conns1) == "table" then
		for _, conn in ipairs(conns1) do
			if type(conn.Function) == "function" then table.insert(data.holdCallbacks, conn.Function) end
		end
	end

	local ok2, conns2 = pcall(getconnections, prompt.Triggered)
	if ok2 and type(conns2) == "table" then
		for _, conn in ipairs(conns2) do
			if type(conn.Function) == "function" then table.insert(data.triggerCallbacks, conn.Function) end
		end
	end

	if (#data.holdCallbacks > 0) or (#data.triggerCallbacks > 0) then
		InternalStealCache[prompt] = data
	end
end

local function executeInternalStealAsync(prompt, animalData)
	local data = InternalStealCache[prompt]
	if not data or not data.ready then return false end

	data.ready = false
	IsStealing = true
	StealProgress = 0

	task.spawn(function()
		if #data.holdCallbacks > 0 then
			for _, fn in ipairs(data.holdCallbacks) do task.spawn(fn) end
		end

		local s = tick()
		local stealDuration = 0.5
		while tick() - s < stealDuration do
			StealProgress = (tick() - s) / stealDuration
			progressLabel.Text = math.floor(StealProgress * 100) .. "%"
			task.wait(0.015)
		end
		StealProgress = 1
		progressLabel.Text = "100%"

		if #data.triggerCallbacks > 0 then
			for _, fn in ipairs(data.triggerCallbacks) do task.spawn(fn) end
		end

		task.wait(0.05)
		data.ready = true

		task.wait(0.15)
		IsStealing = false
		StealProgress = 0
	end)

	return true
end

local function attemptSteal(prompt, animalData)
	if not prompt or not prompt.Parent then return false end
	buildStealCallbacks(prompt)
	if not InternalStealCache[prompt] then return false end
	return executeInternalStealAsync(prompt, animalData)
end

local function getNearestAnimal()
	local hrp = getHRP()
	if not hrp then return nil end
	local nearest = nil
	local minDist = math.huge
	for _, animalData in ipairs(allAnimalsCache) do
		if isMyBase(animalData.plot) then continue end
		if animalData.worldPosition then
			local dist = (hrp.Position - animalData.worldPosition).Magnitude
			if dist < minDist then
				minDist = dist
				nearest = animalData
			end
		end
	end
	return nearest
end

task.spawn(function()
	local progressTween = nil
	while task.wait(0.02) do
		if IsStealing then
			if progressTween then progressTween:Cancel() end
			progressTween = TweenService:Create(
				progressBarFill,
				TweenInfo.new(0.15, Enum.EasingStyle.Linear),
				{Size = UDim2.new(math.clamp(StealProgress, 0, 1), 0, 1, 0)}
			)
			progressTween:Play()
		else
			if progressTween then progressTween:Cancel() progressTween = nil end
			if progressBarFill.Size.X.Scale > 0 then
				progressBarFill.Size = UDim2.new(math.max(0, progressBarFill.Size.X.Scale - 0.05), 0, 1, 0)
				progressLabel.Text = "0%"
			end
		end
	end
end)

local last = 0
RunService.Heartbeat:Connect(function(dt)
	last = last + dt
	if last < 0.02 then return end
	last = 0

	if not AutoSteal.Enabled then return end
	if IsStealing then return end

	local target = getNearestAnimal()
	if not target then return end
	if not shouldSteal(target) then return end

	local prompt = PromptMemoryCache[target.uid]
	if not prompt or not prompt.Parent then
		prompt = findProximityPromptForAnimal(target)
	end

	if prompt then
		buildStealCallbacks(prompt)
		if not InternalStealCache[prompt] then
			for i = 1, 1 do
				task.wait(0.01)
				buildStealCallbacks(prompt)
				if InternalStealCache[prompt] then break end
			end
		end

		if InternalStealCache[prompt] and InternalStealCache[prompt].ready then
			attemptSteal(prompt, target)
		end
	end
end)

initializeScanner()

local function createToggle(text, enableFunc, disableFunc)
	local Holder = Instance.new("Frame")
	Holder.BackgroundTransparency = 1
	Holder.Size = UDim2.new(1, 0, 1, 0)
	
	local Label = Instance.new("TextLabel")
	Label.Size = UDim2.new(1, -10, 0.5, 0)
	Label.Position = UDim2.new(0, 5, 0, 5)
	Label.BackgroundTransparency = 1
	Label.Text = text
	Label.Font = Enum.Font.GothamBold
	Label.TextSize = isMobile and 10 or 12
	Label.TextColor3 = Color3.fromRGB(230, 230, 230)
	Label.TextXAlignment = Enum.TextXAlignment.Center
	Label.Parent = Holder
	
	local Toggle = Instance.new("Frame")
	Toggle.Size = UDim2.new(0, 40, 0, 18)
	Toggle.Position = UDim2.new(0.5, -20, 1, -22)
	Toggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
	Toggle.Parent = Holder
	Instance.new("UICorner", Toggle).CornerRadius = UDim.new(1, 0)
	
	local Circle = Instance.new("Frame")
	Circle.Size = UDim2.new(0, 14, 0, 14)
	Circle.Position = UDim2.new(0, 2, 0.5, -7)
	Circle.BackgroundColor3 = Color3.new(1, 1, 1)
	Circle.Parent = Toggle
	Instance.new("UICorner", Circle).CornerRadius = UDim.new(1, 0)
	
	local enabled = false
	
	local function animate(state)
		if state then
			TweenService:Create(Toggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(0, 140, 255)}):Play()
			TweenService:Create(Circle, TweenInfo.new(0.2), {Position = UDim2.new(1, -16, 0.5, -7)}):Play()
		else
			TweenService:Create(Toggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(60, 60, 60)}):Play()
			TweenService:Create(Circle, TweenInfo.new(0.2), {Position = UDim2.new(0, 2, 0.5, -7)}):Play()
		end
	end
	
	Toggle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			enabled = not enabled
			animate(enabled)
			if enabled then
				enableFunc()
			else
				disableFunc()
			end
		end
	end)
	
	return Holder
end

createToggle("INFINITE\nJUMP", enableInfJump, disableInfJump).Parent = ScrollFrame
createToggle("AUTO\nBAT", enableAutoBat, disableAutoBat).Parent = ScrollFrame
createToggle("XRAY\nBASE", enableXrayBase, disableXrayBase).Parent = ScrollFrame
createToggle("NO\nANIM", enableNoAnim, disableNoAnim).Parent = ScrollFrame
createToggle("MELEE\nAIMBOT", enableMeleeAimbot, disableMeleeAimbot).Parent = ScrollFrame
createToggle("TWEEN\nDIREITO", tweenDireito, function() TweenState.DireitoEnabled = false; stopSpeed() end).Parent = ScrollFrame
createToggle("TWEEN\nESQUERDO", tweenEsquerdo, function() TweenState.EsquerdoEnabled = false; stopSpeed() end).Parent = ScrollFrame
createToggle("POS\n3-4", function() TweenState.usePosition3And4 = true end, function() TweenState.usePosition3And4 = false end).Parent = ScrollFrame
createToggle("ANTI\nRAGDOLL", enableAntiRagdoll, disableAntiRagdoll).Parent = ScrollFrame
createToggle("AUTO\nSTEAL", function() AutoSteal.Enabled = true; Config.AutoStealEnabled = true; saveConfig(Config) end, function() AutoSteal.Enabled = false; Config.AutoStealEnabled = false; saveConfig(Config) end).Parent = ScrollFrame

local dragging = false
local dragStart, startPos

MainFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = MainFrame.Position
		TweenService:Create(UIScale, TweenInfo.new(0.15), {Scale = 0.97}):Play()
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging then
		local delta = input.Position - dragStart
		MainFrame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if dragging then
		dragging = false
		TweenService:Create(UIScale, TweenInfo.new(0.2, Enum.EasingStyle.Back), {Scale = 1}):Play()
	end
end)

local settingsDragging = false
local settingsDragStart, settingsStartPos

SettingsFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		settingsDragging = true
		settingsDragStart = input.Position
		settingsStartPos = SettingsFrame.Position
		TweenService:Create(SettingsUIScale, TweenInfo.new(0.15), {Scale = 0.97}):Play()
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if settingsDragging then
		local delta = input.Position - settingsDragStart
		SettingsFrame.Position = UDim2.new(
			settingsStartPos.X.Scale,
			settingsStartPos.X.Offset + delta.X,
			settingsStartPos.Y.Scale,
			settingsStartPos.Y.Offset + delta.Y
		)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if settingsDragging then
		settingsDragging = false
		TweenService:Create(SettingsUIScale, TweenInfo.new(0.2, Enum.EasingStyle.Back), {Scale = 1}):Play()
	end
end)

local Bubble = Instance.new("Frame")
Bubble.Size = isMobile and UDim2.new(0, 50, 0, 50) or UDim2.new(0, 55, 0, 55)
Bubble.Position = UDim2.new(0, 20, 0, 20)
Bubble.BackgroundColor3 = Color3.fromRGB(8, 20, 50)
Bubble.Parent = ScreenGui
Bubble.Active = true
Bubble.ClipsDescendants = true

Instance.new("UICorner", Bubble).CornerRadius = UDim.new(1, 0)

local BubbleStroke = Instance.new("UIStroke", Bubble)
BubbleStroke.Color = Color3.fromRGB(0, 140, 255)
BubbleStroke.Thickness = 2

local BubbleLabel = Instance.new("TextLabel")
BubbleLabel.Size = UDim2.new(1, 0, 1, 0)
BubbleLabel.BackgroundTransparency = 1
BubbleLabel.Text = "W"
BubbleLabel.Font = Enum.Font.GothamBold
BubbleLabel.TextSize = isMobile and 18 or 24
BubbleLabel.TextColor3 = Color3.fromRGB(0, 140, 255)
BubbleLabel.Parent = Bubble

local BubbleScale = Instance.new("UIScale", Bubble)

local bubbleDragging = false
local bubbleStart, bubblePos
local opened = false

Bubble.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		bubbleDragging = true
		bubbleStart = input.Position
		bubblePos = Bubble.Position
		TweenService:Create(BubbleScale, TweenInfo.new(0.15), {Scale = 0.9}):Play()
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if bubbleDragging then
		local delta = input.Position - bubbleStart
		Bubble.Position = UDim2.new(
			bubblePos.X.Scale,
			bubblePos.X.Offset + delta.X,
			bubblePos.Y.Scale,
			bubblePos.Y.Offset + delta.Y
		)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if bubbleDragging then
		bubbleDragging = false
		TweenService:Create(BubbleScale, TweenInfo.new(0.2, Enum.EasingStyle.Back), {Scale = 1}):Play()
		
		if not bubbleStart or (input.Position - bubbleStart).Magnitude < 5 then
			opened = not opened
			if opened then
				openGui()
			else
				closeGui()
			end
		end
	end
end)

print("✅ WAVE DUELS MOBILE - 100% COMPLETO E OTIMIZADO!")
print("📱 GUI Otimizada para Mobile")
print("💾 Sistema de Salvamento de Configurações")
print("🎯 Clique no botão 'SALVAR CONFIG' para persistir as mudanças")
print("🎮 Clique na bolinha W para abrir/fechar!")
print("⚡ Todas as configurações são salvas localmente!")
