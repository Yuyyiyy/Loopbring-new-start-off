-- SERVICES
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local Debris = game:GetService("Debris")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer

-- COLORS
local darkNavyBlue = Color3.fromRGB(10, 20, 40)
local darkGold = Color3.fromRGB(180, 140, 30)
local defaultBtnColor = Color3.fromRGB(25, 25, 45)
local standardBtnColor = Color3.fromRGB(30, 30, 60)
local whiteText = Color3.fromRGB(255, 255, 255)

-- STATES
local toggleOpen = false
local loopList = {}
local playerButtons = {}
local speedBoostOn = false
local antiLagOn = false
local frontLoopMode = false
local loopbringAllActive = false
local autoUseToolsOn = false

-- GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "PsychoLoopbring"
gui.ResetOnSpawn = false

local toggle = Instance.new("TextButton", gui)
toggle.Size = UDim2.new(0, 120, 0, 25)
toggle.Position = UDim2.new(0, 10, 0, 10)
toggle.BackgroundColor3 = darkNavyBlue
toggle.TextColor3 = darkGold
toggle.Text = "Psycho Loopbring"
toggle.Font = Enum.Font.GothamBold
toggle.TextSize = 14
toggle.BorderSizePixel = 0

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 220, 0, 300)
frame.Position = UDim2.new(0, 10, 0, 45)
frame.BackgroundColor3 = darkNavyBlue
frame.BorderSizePixel = 0
frame.Visible = false
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, -20, 0, 30)
title.Position = UDim2.new(0, 10, 0, 5)
title.BackgroundTransparency = 1
title.Text = "Psycho Loopbring"
title.TextColor3 = darkGold
title.TextScaled = true
title.Font = Enum.Font.FredokaOne
title.TextXAlignment = Enum.TextXAlignment.Left

local scroll = Instance.new("ScrollingFrame", frame)
scroll.Position = UDim2.new(0, 10, 0, 40)
scroll.Size = UDim2.new(1, -20, 1, -50)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel = 0
scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
scroll.ScrollBarThickness = 5
scroll.ClipsDescendants = true

local layout = Instance.new("UIListLayout", scroll)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0, 3)

-- UTILITIES
local function tweenColor(object, color, duration)
	TweenService:Create(object, TweenInfo.new(duration or 0.25), {BackgroundColor3 = color}):Play()
end

local function setNoClip(character, state)
	for _, part in pairs(character:GetDescendants()) do
		if part:IsA("BasePart") then
			part.CanCollide = not state
		end
	end
end

-- TOGGLE UI
toggle.MouseButton1Click:Connect(function()
	toggleOpen = not toggleOpen
	frame.Visible = toggleOpen
end)

-- SPEED BOOST
local speedButton = Instance.new("TextButton", scroll)
speedButton.Size = UDim2.new(1, 0, 0, 28)
speedButton.BackgroundColor3 = standardBtnColor
speedButton.TextColor3 = whiteText
speedButton.Font = Enum.Font.GothamBold
speedButton.TextSize = 13
speedButton.Text = "Speed Boost"
speedButton.LayoutOrder = 0

local function applySpeedBoost(state)
	local char = LocalPlayer.Character
	if char then
		local humanoid = char:FindFirstChildOfClass("Humanoid")
		if humanoid then
			if state then
				humanoid.WalkSpeed = 120
				humanoid.JumpPower = 120
				tweenColor(speedButton, darkGold)
			else
				humanoid.WalkSpeed = 16
				humanoid.JumpPower = 50
				tweenColor(speedButton, standardBtnColor)
			end
		end
	end
end

speedButton.MouseButton1Click:Connect(function()
	speedBoostOn = not speedBoostOn
	applySpeedBoost(speedBoostOn)
end)

LocalPlayer.CharacterAdded:Connect(function(char)
	if speedBoostOn then
		local humanoid = char:WaitForChild("Humanoid", 5)
		if humanoid then
			applySpeedBoost(true)
		end
	end
end)

-- ANTI-LAG SHIELD
local antiLagButton = Instance.new("TextButton", scroll)
antiLagButton.Size = UDim2.new(1, 0, 0, 28)
antiLagButton.BackgroundColor3 = standardBtnColor
antiLagButton.TextColor3 = whiteText
antiLagButton.Font = Enum.Font.GothamBold
antiLagButton.TextSize = 13
antiLagButton.Text = "Anti-Lag Shield"
antiLagButton.LayoutOrder = 1

local function activateAntiLag()
	for _, obj in pairs(LocalPlayer.Character:GetDescendants()) do
		if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Sound") or obj:IsA("Decal") or obj:IsA("Texture") then
			obj:Destroy()
		elseif obj:IsA("MeshPart") then
			obj.TextureID = ""
			obj.Material = Enum.Material.SmoothPlastic
		end
	end
	for _, v in pairs(workspace:GetDescendants()) do
		if v:IsA("Explosion") or v:IsA("Fire") or v:IsA("Smoke") or v:IsA("Sparkles") then
			v:Destroy()
		end
	end
end

antiLagButton.MouseButton1Click:Connect(function()
	antiLagOn = not antiLagOn
	tweenColor(antiLagButton, antiLagOn and darkGold or standardBtnColor)
	if antiLagOn then
		task.spawn(function()
			while antiLagOn do
				activateAntiLag()
				task.wait(1.5)
			end
		end)
	end
end)

-- LOOPBRING ALL
local loopbringAllButton = Instance.new("TextButton", scroll)
loopbringAllButton.Size = UDim2.new(1, 0, 0, 28)
loopbringAllButton.BackgroundColor3 = standardBtnColor
loopbringAllButton.TextColor3 = whiteText
loopbringAllButton.Font = Enum.Font.GothamBold
loopbringAllButton.TextSize = 13
loopbringAllButton.Text = "Loopbring All"
loopbringAllButton.LayoutOrder = 2

loopbringAllButton.MouseButton1Click:Connect(function()
	local state = not loopbringAllActive
	loopbringAllActive = state
	tweenColor(loopbringAllButton, state and darkGold or standardBtnColor)
	for name, btn in pairs(playerButtons) do
		if Players:FindFirstChild(name) and name ~= LocalPlayer.Name then
			loopList[name] = state or nil
			tweenColor(btn, state and darkGold or defaultBtnColor)
		end
	end
end)

-- TOGGLE LOOPBRING POSITION
local loopbringToggleButton = Instance.new("TextButton", scroll)
loopbringToggleButton.Size = UDim2.new(1, 0, 0, 28)
loopbringToggleButton.BackgroundColor3 = standardBtnColor
loopbringToggleButton.TextColor3 = whiteText
loopbringToggleButton.Font = Enum.Font.GothamBold
loopbringToggleButton.TextSize = 13
loopbringToggleButton.Text = "Toggle Loopbring Position"
loopbringToggleButton.LayoutOrder = 3

loopbringToggleButton.MouseButton1Click:Connect(function()
	frontLoopMode = not frontLoopMode
	tweenColor(loopbringToggleButton, frontLoopMode and darkGold or standardBtnColor)
end)

-- AUTO USE TOOLS BUTTON (MATCHING STYLE)
local autoUseButton = Instance.new("TextButton", scroll)
autoUseButton.Size = UDim2.new(1, 0, 0, 28)
autoUseButton.BackgroundColor3 = standardBtnColor
autoUseButton.TextColor3 = whiteText
autoUseButton.Font = Enum.Font.GothamBold
autoUseButton.TextSize = 13
autoUseButton.Text = "Auto Use Tools: OFF"
autoUseButton.LayoutOrder = 4

local function autoUseTools()
	if not autoUseToolsOn then return end
	task.spawn(function()
		for _, tool in ipairs(LocalPlayer.Backpack:GetChildren()) do
			if tool:IsA("Tool") then
				tool.Parent = LocalPlayer.Character
			end
		end
		for _, tool in ipairs(LocalPlayer.Character:GetChildren()) do
			if tool:IsA("Tool") then
				pcall(function() tool:Activate() end)
			end
		end
	end)
end

autoUseButton.MouseButton1Click:Connect(function()
	autoUseToolsOn = not autoUseToolsOn
	autoUseButton.Text = autoUseToolsOn and "Auto Use Tools: ON" or "Auto Use Tools: OFF"
	tweenColor(autoUseButton, autoUseToolsOn and darkGold or standardBtnColor)
end)

-- ADD PLAYER BUTTONS WITH KILL TRACKER
local function addPlayerButton(targetPlayer)
	if targetPlayer == LocalPlayer then return end

	local button = Instance.new("TextButton")
	button.Size = UDim2.new(1, 0, 0, 28)
	button.BackgroundColor3 = defaultBtnColor
	button.TextColor3 = whiteText
	button.Font = Enum.Font.GothamBold
	button.TextSize = 13
	button.Text = targetPlayer.Name
	button.LayoutOrder = 100
	button.Parent = scroll

	playerButtons[targetPlayer.Name] = button

	-- Kill tracker label
	local killLabel = Instance.new("TextLabel", button)
	killLabel.Size = UDim2.new(0, 60, 1, 0)
	killLabel.Position = UDim2.new(1, -65, 0, 0)
	killLabel.BackgroundTransparency = 1
	killLabel.TextColor3 = Color3.new(1, 0.2, 0.2)
	killLabel.Font = Enum.Font.GothamBold
	killLabel.TextSize = 12
	killLabel.TextXAlignment = Enum.TextXAlignment.Right
	killLabel.Text = "Kills: 0"

	local kills = 0

	-- Track kills on target player's character death
	local function onCharacterAdded(char)
		local humanoid = char:WaitForChild("Humanoid", 10)
		if humanoid then
			humanoid.Died:Connect(function()
				-- Check if LocalPlayer currently has a tool (as a proxy for kill)
				local killerTool = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Tool")
				if killerTool then
					kills = kills + 1
					killLabel.Text = "Kills: " .. kills
				end
			end)
		end
	end

	if targetPlayer.Character then
		onCharacterAdded(targetPlayer.Character)
	end
	targetPlayer.CharacterAdded:Connect(onCharacterAdded)

	button.MouseButton1Click:Connect(function()
		local name = targetPlayer.Name
		loopList[name] = not loopList[name]
		tweenColor(button, loopList[name] and darkGold or defaultBtnColor)
	end)
end

for _, p in ipairs(Players:GetPlayers()) do
	addPlayerButton(p)
end
Players.PlayerAdded:Connect(addPlayerButton)

-- LOOPBRING EXECUTION
task.spawn(function()
	while true do
		local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
		if myHRP then
			for name, active in pairs(loopList) do
				if active then
					local target = Players:FindFirstChild(name)
					if target and target.Character then
						local hrp = target.Character:FindFirstChild("HumanoidRootPart")
						if hrp then
							setNoClip(target.Character, true)
							hrp.Velocity = Vector3.new(0, 0, 0)
							hrp.RotVelocity = Vector3.new(0, 0, 0)
							local offset
							if frontLoopMode then
								offset = myHRP.CFrame.LookVector * 3 + Vector3.new(0, 1, 0)
							else
								offset = myHRP.CFrame.RightVector * 3 + myHRP.CFrame.LookVector + Vector3.new(0, 1, 0)
							end
							hrp.CFrame = myHRP.CFrame + offset
						end
					end
				end
			end
		end
		autoUseTools()
		task.wait(0)
	end
end)
