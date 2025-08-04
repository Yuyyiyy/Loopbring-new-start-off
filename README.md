-- SERVICES
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local Debris = game:GetService("Debris")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer

-- COLORS
local darkBlue = Color3.fromRGB(10, 20, 40)
local darkGold = Color3.fromRGB(180, 140, 30)
local defaultBtnColor = Color3.fromRGB(25, 25, 45)

-- STATES
local toggleOpen = false
local loopList = {}
local playerButtons = {}
local speedBoostOn = false
local antiLagOn = false
local frontLoopMode = false
local loopbringAllActive = false

-- GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "PsychoLoopbring"
gui.ResetOnSpawn = false

local toggle = Instance.new("TextButton", gui)
toggle.Size = UDim2.new(0, 120, 0, 25)
toggle.Position = UDim2.new(0, 10, 0, 10)
toggle.BackgroundColor3 = darkBlue
toggle.TextColor3 = darkGold
toggle.Text = "Psycho Loopbring"
toggle.Font = Enum.Font.GothamBold
toggle.TextSize = 14
toggle.BorderSizePixel = 0

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 220, 0, 260)
frame.Position = UDim2.new(0, 10, 0, 45)
frame.BackgroundColor3 = darkBlue
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
speedButton.BackgroundColor3 = Color3.fromRGB(30, 30, 60)
speedButton.TextColor3 = Color3.fromRGB(255, 255, 255)
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
				tweenColor(speedButton, Color3.fromRGB(30, 30, 60))
			end
		end
	end
end

speedButton.MouseButton1Click:Connect(function()
	speedBoostOn = not speedBoostOn
	applySpeedBoost(speedBoostOn)
end)

-- Reapply speed boost automatically when character respawns
LocalPlayer.CharacterAdded:Connect(function(char)
	if speedBoostOn then
		char:WaitForChild("Humanoid", 5)
		applySpeedBoost(true)
	end
end)

-- ANTI-LAG
local antiLagButton = Instance.new("TextButton", scroll)
antiLagButton.Size = UDim2.new(1, 0, 0, 28)
antiLagButton.BackgroundColor3 = Color3.fromRGB(30, 30, 60)
antiLagButton.TextColor3 = Color3.fromRGB(255, 255, 255)
antiLagButton.Font = Enum.Font.GothamBold
antiLagButton.TextSize = 13
antiLagButton.Text = "Anti-Lag Shield"
antiLagButton.LayoutOrder = 1

local function activateAntiLag()
	-- Destroy or simplify effects but KEEP avatar colors and lighting intact
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

	-- No changes to Lighting to keep screen colors and avatar color unchanged
end

antiLagButton.MouseButton1Click:Connect(function()
	antiLagOn = not antiLagOn
	tweenColor(antiLagButton, antiLagOn and darkGold or Color3.fromRGB(30, 30, 60))
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
loopbringAllButton.BackgroundColor3 = Color3.fromRGB(30, 30, 60)
loopbringAllButton.TextColor3 = Color3.fromRGB(255, 255, 255)
loopbringAllButton.Font = Enum.Font.GothamBold
loopbringAllButton.TextSize = 13
loopbringAllButton.Text = "Loopbring All"
loopbringAllButton.LayoutOrder = 2

loopbringAllButton.MouseButton1Click:Connect(function()
	local state = not loopbringAllActive
	loopbringAllActive = state
	tweenColor(loopbringAllButton, state and darkGold or Color3.fromRGB(30, 30, 60))
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
loopbringToggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 60)
loopbringToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
loopbringToggleButton.Font = Enum.Font.GothamBold
loopbringToggleButton.TextSize = 13
loopbringToggleButton.Text = "Toggle Loopbring Position"
loopbringToggleButton.LayoutOrder = 3

loopbringToggleButton.MouseButton1Click:Connect(function()
	frontLoopMode = not frontLoopMode
	tweenColor(loopbringToggleButton, frontLoopMode and darkGold or Color3.fromRGB(30, 30, 60))
end)

-- ADD PLAYER BUTTON
local function addPlayerButton(targetPlayer)
	if targetPlayer == LocalPlayer then return end

	local button = Instance.new("TextButton")
	button.Size = UDim2.new(1, 0, 0, 28)
	button.BackgroundColor3 = defaultBtnColor
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Font = Enum.Font.GothamBold
	button.TextSize = 13
	button.Text = targetPlayer.Name
	button.LayoutOrder = 100
	button.Parent = scroll

	playerButtons[targetPlayer.Name] = button

	button.MouseButton1Click:Connect(function()
		local name = targetPlayer.Name
		loopList[name] = not loopList[name]
		tweenColor(button, loopList[name] and darkGold or defaultBtnColor)
	end)
end

for _, p in ipairs(Players:GetPlayers()) do addPlayerButton(p) end
Players.PlayerAdded:Connect(addPlayerButton)

Players.PlayerRemoving:Connect(function(player)
	-- Do nothing to keep loopList data intact
end)

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
							hrp.Velocity = Vector3.zero
							hrp.RotVelocity = Vector3.zero

							local offset
							if frontLoopMode then
								offset = myHRP.CFrame.LookVector * 3
							else
								offset = myHRP.CFrame.RightVector * 3 + myHRP.CFrame.LookVector
							end

							hrp.CFrame = myHRP.CFrame + offset + Vector3.new(0, 1, 0)
						end
					end
				end
			end
		end
		task.wait(0.08)
	end
end)
