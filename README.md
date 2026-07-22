-- LocalScript для телефона/планшета (Скрытая рука, меню кнопок и радужные искры при взрыве)
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local localPlayer = Players.LocalPlayer

local bombCooldown = 1.5
local canDropBomb = true

----------------------------------------------------------------
-- 1. УВЕЛИЧЕННАЯ ЗОЛОТАЯ КНОПКА В БАРЕ
----------------------------------------------------------------
local playerGui = localPlayer:WaitForChild("PlayerGui")
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "C4ControlSettingsGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local function createGoldGradient(parent)
	local gradient = Instance.new("UIGradient")
	gradient.Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 215, 0)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 140, 0)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 215, 0))
	})
	gradient.Parent = parent
	
	task.spawn(function()
		while true do
			for offset = -1, 1, 0.03 do
				gradient.Offset = Vector2.new(offset, 0)
				task.wait(0.02)
			end
		end
	end)
end

local openButton = Instance.new("TextButton")
openButton.Name = "TopBarMenuButton"
openButton.Size = UDim2.new(0, 42, 0, 42)
openButton.Position = UDim2.new(0, 100, 0, 3) 
openButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
openButton.TextColor3 = Color3.fromRGB(255, 255, 255)
openButton.TextSize = 22
openButton.Font = Enum.Font.SourceSansBold
openButton.Text = "💣"
openButton.Parent = screenGui

local uicornerBtn = Instance.new("UICorner")
uicornerBtn.CornerRadius = UDim.new(1, 0)
uicornerBtn.Parent = openButton

local strokeBtn = Instance.new("UIStroke")
strokeBtn.Thickness = 2
strokeBtn.Color = Color3.fromRGB(130, 65, 0)
strokeBtn.Parent = openButton

createGoldGradient(openButton)

----------------------------------------------------------------
-- 2. ГЛАВНАЯ ПАНЕЛЬ С СИСТЕМОЙ ПЕРЕТАСКИВАНИЯ
----------------------------------------------------------------
local mainPanel = Instance.new("Frame")
mainPanel.Name = "MainPanel"
mainPanel.Size = UDim2.new(0, 0, 0, 0)
mainPanel.Position = UDim2.new(0.5, 0, 0.4, 0)
mainPanel.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
mainPanel.Visible = false
mainPanel.ClipsDescendants = true
mainPanel.Active = true 
mainPanel.Parent = screenGui

local uicornerPanel = Instance.new("UICorner")
uicornerPanel.CornerRadius = UDim.new(0, 16)
uicornerPanel.Parent = mainPanel

local strokePanel = Instance.new("UIStroke")
strokePanel.Thickness = 3
strokePanel.Color = Color3.fromRGB(150, 75, 0)
strokePanel.Parent = mainPanel

createGoldGradient(mainPanel)

-- Система перетаскивания (Drag & Drop)
local dragging, dragInput, dragStart, startPos
local function update(input)
	local delta = input.Position - dragStart
	mainPanel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

mainPanel.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = mainPanel.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then dragging = false end
		end)
	end
end)

mainPanel.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then update(input) end
end)

-- Заголовок
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 35)
title.BackgroundTransparency = 1
title.Text = "Настройки C4 Бомб"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 18
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Parent = mainPanel

local strokeTitle = Instance.new("UIStroke")
strokeTitle.Thickness = 1.5
strokeTitle.Color = Color3.fromRGB(0, 0, 0)
strokeTitle.Parent = title

-- Поле ввода времени
local cooldownInput = Instance.new("TextBox")
cooldownInput.Size = UDim2.new(0, 260, 0, 35)
cooldownInput.Position = UDim2.new(0.5, -130, 0, 40)
cooldownInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
cooldownInput.TextColor3 = Color3.fromRGB(255, 215, 0)
cooldownInput.Text = tostring(bombCooldown)
cooldownInput.TextSize = 15
cooldownInput.Font = Enum.Font.SourceSansBold
cooldownInput.PlaceholderText = "Время жизни (сек)"
cooldownInput.Parent = mainPanel

local uicornerInput = Instance.new("UICorner")
uicornerInput.CornerRadius = UDim.new(0, 6)
uicornerInput.Parent = cooldownInput

-- Функция создания красивых кнопок
local function createMenuButton(text, color)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0, 125, 0, 35)
	btn.BackgroundColor3 = color
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.Font = Enum.Font.SourceSansBold
	btn.TextSize = 14
	btn.Text = text
	btn.Parent = mainPanel
	
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 6)
	corner.Parent = btn
	
	return btn
end

-- Кнопки для Золотой бомбы
local addGoldBtn = createMenuButton("Выдать Золотую", Color3.fromRGB(215, 145, 20))
addGoldBtn.Position = UDim2.new(0, 10, 0, 85)

local remGoldBtn = createMenuButton("Убрать Золотую", Color3.fromRGB(180, 40, 40))
remGoldBtn.Position = UDim2.new(0, 145, 0, 85)

-- Кнопки для Синей бомбы
local addBlueBtn = createMenuButton("Выдать Синюю", Color3.fromRGB(20, 110, 215))
addBlueBtn.Position = UDim2.new(0, 10, 0, 130)

local remBlueBtn = createMenuButton("Убрать Синюю", Color3.fromRGB(180, 40, 40))
remBlueBtn.Position = UDim2.new(0, 145, 0, 130)

-- Открытие/Закрытие
local menuOpen = false
openButton.MouseButton1Down:Connect(function()
	menuOpen = not menuOpen
	if menuOpen then
		mainPanel.Visible = true
		mainPanel:TweenSizeAndPosition(
			UDim2.new(0, 280, 0, 180), 
			UDim2.new(0.5, -140, 0.4, -90), 
			Enum.EasingDirection.Out,
			Enum.EasingStyle.Back, 
			0.3,
			true
		)
	else
		mainPanel:TweenSizeAndPosition(
			UDim2.new(0, 0, 0, 0),
			UDim2.new(0.5, mainPanel.Position.X.Offset, 0.4, mainPanel.Position.Y.Offset), 
			Enum.EasingDirection.In,
			Enum.EasingStyle.Quad,
			0.2,
			true,
			function() mainPanel.Visible = false end
		)
	end
end)

cooldownInput.FocusLost:Connect(function()
	local value = tonumber(cooldownInput.Text)
	if value and value >= 0 then bombCooldown = value else cooldownInput.Text = tostring(bombCooldown) end
end)

----------------------------------------------------------------
-- 3. МЕХАНИКА СКРЫТИЯ РУКИ И РАДУЖНОГО ВЗРЫВА
----------------------------------------------------------------
local function createC4Block(color, isStatic)
	local block = Instance.new("Part")
	block.Shape = Enum.PartType.Block
	block.Size = Vector3.new(1.5, 0.6, 1.2) 
	block.Color = color
	block.Material = Enum.Material.SmoothPlastic
	block.Reflectance = 0.12

	if isStatic then
		block.Anchored = false
		block.CanCollide = true
		
		local bodyAngularVelocity = Instance.new("BodyAngularVelocity")
		bodyAngularVelocity.MaxTorque = Vector3.new(math.huge, math.huge, math.huge) 
		bodyAngularVelocity.AngularVelocity = Vector3.new(0, 0, 0)
		bodyAngularVelocity.Parent = block
	else
		block.Massless = true
	end
	return block
end

local function setupBombLogic(tool, color)
	tool.RequiresHandle = true
	local handle = createC4Block(color, false)
	handle.Name = "Handle"
	handle.Parent = tool
	
	tool.Grip = CFrame.new(0, 0, 0) * CFrame.Angles(0, 0, 0)

	tool.Activated:Connect(function()
		local character = localPlayer.Character
		if not character or not canDropBomb then return end

		local humanoid = character:FindFirstChildOfClass("Humanoid")
		local rootPart = character:FindFirstChild("HumanoidRootPart")
		
		if humanoid and rootPart and humanoid.Health > 0 then
			canDropBomb = false
			humanoid.Jump = true 
			task.wait(0.12)
			
			local droppedBlock = createC4Block(color, true)
			droppedBlock.Name = "DroppedVisualC4"
			
			local currentPos = rootPart.Position - Vector3.new(0, 2, 0)
			droppedBlock.CFrame = CFrame.new(currentPos) 
			droppedBlock.Parent = game.Workspace
			
			droppedBlock.Velocity = Vector3.new(0, -12, 0)

			-- Таймер до создания эффекта
			task.spawn(function()
				-- Создаем искры за 0.5 сек до конца жизни бомбы
				local waitTime = math.max(0, bombCooldown - 0.5)
				task.wait(waitTime)
				
				if droppedBlock then
					local sparkles = Instance.new("Sparkles")
					sparkles.Enabled = true
					sparkles.Parent = droppedBlock
					
					-- Цикл создания радужного перелива (цвета постоянно меняются)
					task.spawn(function()
						local hue = 0
						while droppedBlock and sparkles and sparkles.Parent do
							sparkles.SparkleColor = Color3.fromHSV(hue, 1, 1) -- Создаем радужный цвет
							hue = hue + 0.05
							if hue > 1 then hue = 0 end
							task.wait(0.02) -- Скорость перелива радуги
						end
					end)
					
					-- Сама бомба форму не меняет, просто ждем финальные 0.5 сек и взрываем её
					task.wait(0.5)
					
					if droppedBlock then
						-- Быстрое исчезновение
						for transparency = 0, 1, 0.2 do
							if droppedBlock then droppedBlock.Transparency = transparency end
							task.wait(0.02)
						end
						droppedBlock:Destroy()
					end
				end
			end)
			
			task.wait(0.2) 
			canDropBomb = true
		end
	end)
end

local function giveGold()
	local backpack = localPlayer:WaitForChild("Backpack")
	if not backpack:FindFirstChild("Gold C4 Block") and not (localPlayer.Character and localPlayer.Character:FindFirstChild("Gold C4 Block")) then
		local goldTool = Instance.new("Tool")
		goldTool.Name = "Gold C4 Block"
		setupBombLogic(goldTool, Color3.fromRGB(255, 185, 35))
		goldTool.Parent = backpack
	end
end

local function giveBlue()
	local backpack = localPlayer:WaitForChild("Backpack")
	if not backpack:FindFirstChild("Blue C4 Block") and not (localPlayer.Character and localPlayer.Character:FindFirstChild("Blue C4 Block")) then
		local blueTool = Instance.new("Tool")
		blueTool.Name = "Blue C4 Block"
		setupBombLogic(blueTool, Color3.fromRGB(0, 120, 255))
		blueTool.Parent = backpack
	end
end

local function removeBomb(name)
	local backpack = localPlayer:FindFirstChild("Backpack")
	if backpack then local t = backpack:FindFirstChild(name) if t then t:Destroy() end end
	local character = localPlayer.Character
	if character then local t = character:FindFirstChild(name) if t then t:Destroy() end end
end

addGoldBtn.MouseButton1Down:Connect(giveGold)
remGoldBtn.MouseButton1Down:Connect(function() removeBomb("Gold C4 Block") end)
addBlueBtn.MouseButton1Down:Connect(giveBlue)
remBlueBtn.MouseButton1Down:Connect(function() removeBomb("Blue C4 Block") end)

local function giveAll() giveGold() giveBlue() end
localPlayer.CharacterAdded:Connect(function() task.wait(0.8) giveAll() end)
if localPlayer.Character then giveAll() end
