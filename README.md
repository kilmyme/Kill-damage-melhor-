local settings = {
	repeatamount = 2,
	exceptions = {
		"SayMessageRequest", "MeleeUpdateEvent", "NinjaBombEvent", "BulletUpdateEvent",
		"CharacterSoundEvent", "PlayAnimation", "Animate", "MoveUpdate",
		"Idle", "Run", "Jump", "Sit", "Climb"
	}
}

local enabled = false
local player = game:GetService("Players").LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Forçar PlayerGui carregar antes de criar GUI
local playerGui
repeat
	task.wait()
	playerGui = player:FindFirstChildOfClass("PlayerGui")
until playerGui

-- Criar GUI principal
local gui = Instance.new("ScreenGui")
gui.Name = "SpamGui"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = false
gui.Enabled = true
gui.Parent = playerGui

-- Botão ☠️ principal
local button = Instance.new("TextButton", gui)
button.Size = UDim2.new(0, 45, 0, 45)
button.Position = UDim2.new(0, 30, 0.8, 0)
button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
button.TextColor3 = Color3.new(1, 1, 1)
button.Font = Enum.Font.GothamBold
button.TextSize = 25
Instance.new("UICorner", button).CornerRadius = UDim.new(1, 0)

local function updateButtonText()
	if enabled then
		button.Text = "☠️\n" .. tostring(settings.repeatamount)
	else
		button.Text = "☠️"
	end
end
updateButtonText()

-- Botão +
local buttonPlus = Instance.new("TextButton", gui)
buttonPlus.Size = UDim2.new(0, 30, 0, 30)
buttonPlus.Position = UDim2.new(0, 80, 0.8, 0)
buttonPlus.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
buttonPlus.Text = "+"
buttonPlus.TextColor3 = Color3.new(1,1,1)
buttonPlus.Font = Enum.Font.GothamBold
buttonPlus.TextSize = 25
Instance.new("UICorner", buttonPlus).CornerRadius = UDim.new(0.4, 0)

-- Botão -
local buttonMinus = Instance.new("TextButton", gui)
buttonMinus.Size = UDim2.new(0, 30, 0, 30)
buttonMinus.Position = UDim2.new(0, 0, 0.8, 0)
buttonMinus.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
buttonMinus.Text = "-"
buttonMinus.TextColor3 = Color3.new(1,1,1)
buttonMinus.Font = Enum.Font.GothamBold
buttonMinus.TextSize = 25
Instance.new("UICorner", buttonMinus).CornerRadius = UDim.new(0.4, 0)

-- Botão ⚡ especial
local buttonSpecial = Instance.new("TextButton", gui)
buttonSpecial.Size = UDim2.new(0, 40, 0, 40)
buttonSpecial.Position = UDim2.new(0, 100, 0.8, -60)
buttonSpecial.BackgroundColor3 = Color3.fromRGB(255, 170, 0)
buttonSpecial.Text = "⚡"
buttonSpecial.TextColor3 = Color3.new(1,1,1)
buttonSpecial.Font = Enum.Font.GothamBlack
buttonSpecial.TextSize = 30
Instance.new("UICorner", buttonSpecial).CornerRadius = UDim.new(0.5, 0)

-- Drag do botão especial ⚡
local draggingSpecial, dragInputSpecial, dragStartSpecial, startPosSpecial

buttonSpecial.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
		draggingSpecial = true
		dragStartSpecial = input.Position
		startPosSpecial = buttonSpecial.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				draggingSpecial = false
			end
		end)
	end
end)

buttonSpecial.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement then
		dragInputSpecial = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInputSpecial and draggingSpecial then
		local delta = input.Position - dragStartSpecial
		local newX = startPosSpecial.X.Offset + delta.X
		local newY = startPosSpecial.Y.Offset + delta.Y
		buttonSpecial.Position = UDim2.new(startPosSpecial.X.Scale, newX, startPosSpecial.Y.Scale, newY)
	end
end)

-- Botões + e -
buttonPlus.MouseButton1Click:Connect(function()
	settings.repeatamount += 1
	updateButtonText()
end)

buttonMinus.MouseButton1Click:Connect(function()
	if settings.repeatamount > 1 then
		settings.repeatamount -= 1
		updateButtonText()
	end
end)

-- ⚡ Botão especial ativa fogo azul e turbo
buttonSpecial.MouseButton1Click:Connect(function()
	local oldAmount = settings.repeatamount
	local wasEnabled = enabled

	settings.repeatamount = 20
	enabled = true
	button.BackgroundColor3 = Color3.fromRGB(255, 85, 0)
	updateButtonText()

	local character = player.Character
	local fireEffects = {}

	if character then
		for _, part in ipairs(character:GetDescendants()) do
			if part:IsA("BasePart") and not part:FindFirstChild("TurboFire") then
				local fire = Instance.new("Fire")
				fire.Name = "TurboFire"
				fire.Heat = 10
				fire.Size = 6
				fire.Color = Color3.fromRGB(0, 170, 255)        -- Azul forte
				fire.SecondaryColor = Color3.fromRGB(100, 200, 255) -- Azul claro
				fire.Parent = part
				table.insert(fireEffects, fire)
			end
		end
	end

	task.wait(5)

	settings.repeatamount = oldAmount
	updateButtonText()

	if not wasEnabled then
		enabled = false
		button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
	else
		button.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
	end

	for _, fire in ipairs(fireEffects) do
		if fire and fire.Parent then
			fire:Destroy()
		end
	end
end)

-- ☠️ Botão principal ativa aura e toggle
local dragging, dragInput, dragStart, startPos = false
local auraLight, auraParticles

button.MouseButton1Click:Connect(function()
	if not dragging then
		enabled = not enabled
		button.BackgroundColor3 = enabled and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(255, 0, 0)
		updateButtonText()

		local character = player.Character
		if character then
			local hrp = character:FindFirstChild("HumanoidRootPart")
			if hrp then
				if enabled then
					if not auraLight then
						auraLight = Instance.new("PointLight")
						auraLight.Name = "GreenAuraLight"
						auraLight.Color = Color3.fromRGB(0, 255, 0)
						auraLight.Range = 8
						auraLight.Brightness = 1
						auraLight.Shadows = false
						auraLight.Parent = hrp
					end
					if not auraParticles then
						auraParticles = Instance.new("ParticleEmitter")
						auraParticles.Name = "GreenAuraParticles"
						auraParticles.Color = ColorSequence.new(Color3.fromRGB(0, 255, 0))
						auraParticles.LightEmission = 0.3
						auraParticles.Rate = 15
						auraParticles.Lifetime = NumberRange.new(0.5, 1)
						auraParticles.Speed = NumberRange.new(0, 0.5)
						auraParticles.Size = NumberSequence.new(1)
						auraParticles.Texture = ""
						auraParticles.Parent = hrp
					end
				else
					if auraLight then auraLight:Destroy(); auraLight = nil end
					if auraParticles then auraParticles:Destroy(); auraParticles = nil end
				end
			end
		end
	end
end)

-- Drag do botão ☠️
button.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = button.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then dragging = false end
		end)
	end
end)

button.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		local delta = input.Position - dragStart
		local newX = startPos.X.Offset + delta.X
		local newY = startPos.Y.Offset + delta.Y
		button.Position = UDim2.new(startPos.X.Scale, newX, startPos.Y.Scale, newY)
		buttonPlus.Position = UDim2.new(0, newX + 50, 0.8, 0)
		buttonMinus.Position = UDim2.new(0, newX - 30, 0.8, 0)
	end
end)

-- Hook seguro para repetir chamadas
local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt, false)

mt.__namecall = newcclosure(function(self, ...)
	local args = {...}
	local method = getnamecallmethod()
	for _, name in ipairs(settings.exceptions) do
		if self.Name == name then
			return old(self, ...)
		end
	end
	if enabled and (method == "FireServer" or method == "InvokeServer") then
		for i = 1, settings.repeatamount do
			old(self, unpack(args))
		end
		return
	end
	return old(self, ...)
end)

setreadonly(mt, true)
