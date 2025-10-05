-- โหลดบริการ
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Debris = game:GetService("Debris")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer

local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- GUI หลัก
local screenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
screenGui.Name = "kaogui"
screenGui.ResetOnSpawn = false

local openButton = Instance.new("TextButton", screenGui)
openButton.Size = UDim2.new(0, 120, 0, 40)
openButton.Position = UDim2.new(1, -130, 0, 10)
openButton.Text = "⚡ เปิด kaogui"
openButton.BackgroundColor3 = Color3.fromRGB(0, 0, 40)
openButton.TextColor3 = Color3.fromRGB(0, 255, 255)
openButton.Font = Enum.Font.SciFi
openButton.TextSize = 16

local guiFrame = Instance.new("Frame", screenGui)
guiFrame.Size = UDim2.new(0, 300, 0, 220)
guiFrame.Position = UDim2.new(0.5, -150, 0.5, -110)
guiFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 20)
guiFrame.BorderColor3 = Color3.fromRGB(0, 255, 255)
guiFrame.Visible = false

local title = Instance.new("TextLabel", guiFrame)
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "👁 kaogui เท่จัด"
title.TextColor3 = Color3.fromRGB(0, 255, 255)
title.BackgroundTransparency = 1
title.Font = Enum.Font.FredokaOne
title.TextSize = 24

local closeButton = Instance.new("TextButton", guiFrame)
closeButton.Size = UDim2.new(0, 40, 0, 30)
closeButton.Position = UDim2.new(1, -45, 0, 5)
closeButton.Text = "X"
closeButton.Font = Enum.Font.Code
closeButton.TextColor3 = Color3.fromRGB(255, 0, 0)
closeButton.BackgroundColor3 = Color3.fromRGB(20, 0, 0)

local function createButton(text, posX, posY)
	local btn = Instance.new("TextButton", guiFrame)
	btn.Size = UDim2.new(0, 120, 0, 40)
	btn.Position = UDim2.new(0, posX, 0, posY)
	btn.Text = text .. ": OFF"
	btn.Font = Enum.Font.Arcade
	btn.TextSize = 14
	btn.BackgroundColor3 = Color3.fromRGB(0, 0, 40)
	btn.TextColor3 = Color3.fromRGB(255, 0, 0)
	return btn
end

local espToggle = createButton("ESP", 20, 60)
local speedToggle = createButton("Speed", 160, 60)
local flyToggle = createButton("Fly", 20, 110)
local fullbrightToggle = createButton("Fullbright", 160, 110)

-- ตัวแปรสถานะ
local espEnabled, speedEnabled, flyEnabled = false, false, false
local fullbrightEnabled = false
local speedValue, defaultSpeed = 150, humanoid.WalkSpeed

local originalLightingSettings = {
	ClockTime = Lighting.ClockTime,
	Brightness = Lighting.Brightness,
	GlobalShadows = Lighting.GlobalShadows,
	Ambient = Lighting.Ambient,
	OutdoorAmbient = Lighting.OutdoorAmbient
}

local function enableFullbright()
	Lighting.ClockTime = 14
	Lighting.Brightness = 2
	Lighting.GlobalShadows = false
	Lighting.Ambient = Color3.new(1, 1, 1)
	Lighting.OutdoorAmbient = Color3.new(1, 1, 1)
end

local function disableFullbright()
	for k, v in pairs(originalLightingSettings) do
		Lighting[k] = v
	end
end

local function createESP(char, isNPC)
	if not char:FindFirstChild("Head") or char:FindFirstChild("ESP") then return end

	local highlight = Instance.new("Highlight", char)
	highlight.Name = "ESP"
	highlight.Adornee = char
	highlight.FillColor = isNPC and Color3.fromRGB(255, 255, 0) or Color3.fromRGB(255, 0, 0)
	highlight.OutlineColor = Color3.fromRGB(255, 255, 255)

	local billboard = Instance.new("BillboardGui", char.Head)
	billboard.Name = "HPDisplay"
	billboard.Size = UDim2.new(0, 100, 0, 30)
	billboard.StudsOffset = Vector3.new(0, 4, 0)
	billboard.AlwaysOnTop = true

	local textLabel = Instance.new("TextLabel", billboard)
	textLabel.Name = "HPText"
	textLabel.Size = UDim2.new(1, 0, 1, 0)
	textLabel.BackgroundTransparency = 1
	textLabel.TextColor3 = Color3.new(1, 1, 1)
	textLabel.TextStrokeTransparency = 0
	textLabel.TextScaled = true
	textLabel.Font = Enum.Font.SourceSansBold

	-- เอฟเฟกต์ออร่า NPC
	if isNPC and char:FindFirstChild("HumanoidRootPart") and not char:FindFirstChild("AuraFX") then
		local effect = Instance.new("ParticleEmitter")
		effect.Name = "AuraFX"
		effect.Color = ColorSequence.new(Color3.fromRGB(0, 255, 255))
		effect.LightEmission = 1
		effect.Size = NumberSequence.new(1.5)
		effect.Texture = "rbxassetid://301174754"
		effect.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.2), NumberSequenceKeypoint.new(1, 1)})
		effect.Lifetime = NumberRange.new(1)
		effect.Rate = 5
		effect.Rotation = NumberRange.new(0, 360)
		effect.RotSpeed = NumberRange.new(20)
		effect.Speed = NumberRange.new(0)
		effect.SpreadAngle = Vector2.new(360, 360)
		effect.Parent = char.HumanoidRootPart
	end
end

RunService.RenderStepped:Connect(function()
	if espEnabled then
		for _, npc in ipairs(workspace:GetDescendants()) do
			if npc:IsA("Model") and not Players:GetPlayerFromCharacter(npc) and npc ~= LocalPlayer.Character then
				local humanoid = npc:FindFirstChildOfClass("Humanoid")
				local hrp = npc:FindFirstChild("HumanoidRootPart")
				if humanoid and hrp and humanoid.Health > 0 and not npc:FindFirstChild("ESP") then
					if hrp.Velocity.Magnitude > 0.5 then
						createESP(npc, true)
					end
				end
				local head = npc:FindFirstChild("Head")
				if head and head:FindFirstChild("HPDisplay") then
					head.HPDisplay.HPText.Text = "HP: " .. math.floor(humanoid.Health)
				end
			end
		end
	end

	if speedEnabled and LocalPlayer.Character then
		local h = LocalPlayer.Character:FindFirstChild("Humanoid")
		if h and h.WalkSpeed < speedValue then
			h.WalkSpeed = speedValue
		end
	end
end)

-- ปุ่มกด
espToggle.MouseButton1Click:Connect(function()
	espEnabled = not espEnabled
	espToggle.Text = espEnabled and "ESP: ON" or "ESP: OFF"
	espToggle.TextColor3 = espEnabled and Color3.fromRGB(0, 255, 255) or Color3.fromRGB(255, 0, 0)
end)

speedToggle.MouseButton1Click:Connect(function()
	speedEnabled = not speedEnabled
	speedToggle.Text = speedEnabled and "Speed: ON" or "Speed: OFF"
	speedToggle.TextColor3 = speedEnabled and Color3.fromRGB(0, 255, 255) or Color3.fromRGB(255, 0, 0)
end)

flyToggle.MouseButton1Click:Connect(function()
	flyEnabled = not flyEnabled
	flyToggle.Text = flyEnabled and "Fly: ON" or "Fly: OFF"
	flyToggle.TextColor3 = flyEnabled and Color3.fromRGB(0, 255, 255) or Color3.fromRGB(255, 0, 0)
	if flyEnabled then
		loadstring(game:HttpGet("https://rawscripts.net/raw/Universal-Script-Gui-Fly-v3-37111"))()
	end
end)

fullbrightToggle.MouseButton1Click:Connect(function()
	fullbrightEnabled = not fullbrightEnabled
	fullbrightToggle.Text = fullbrightEnabled and "Fullbright: ON" or "Fullbright: OFF"
	fullbrightToggle.TextColor3 = fullbrightEnabled and Color3.fromRGB(0, 255, 255) or Color3.fromRGB(255, 0, 0)
	if fullbrightEnabled then enableFullbright() else disableFullbright() end
end)

openButton.MouseButton1Click:Connect(function()
	guiFrame.Visible = true
	openButton.Visible = false
end)

closeButton.MouseButton1Click:Connect(function()
	guiFrame.Visible = false
	openButton.Visible = true
end)
