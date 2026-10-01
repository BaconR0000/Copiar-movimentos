local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

local selectedPlayer = nil
local followConnection = nil
local animationConnection = nil
local copiedTracks = {}

-- =========================================
-- CONFIGURAÇÕES
-- =========================================

local positionMode = "Behind"
local distance = 3

local MIN_DISTANCE = 0
local MAX_DISTANCE = 20

local positions = {
	Behind = Vector3.new(0, 0, 1),
	Front = Vector3.new(0, 0, -1),
	Left = Vector3.new(-1, 0, 0),
	Right = Vector3.new(1, 0, 0),
	Inside = Vector3.new(0, 0, 0)
}

-- =========================================
-- GUI
-- =========================================

local gui = Instance.new("ScreenGui")
gui.Name = "CopyMovementGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 250, 0, 390)
frame.Position = UDim2.new(0, 15, 0.5, -195)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Parent = gui

-- =========================================
-- TÍTULO
-- =========================================

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 35)
title.Text = "Copiar Movimentos"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
title.TextScaled = true
title.Active = true
title.Parent = frame

-- =========================================
-- ARRASTAR
-- =========================================

local dragging = false
local dragStart
local startPosition

local function updateDrag(input)

	local delta = input.Position - dragStart

	frame.Position = UDim2.new(
		startPosition.X.Scale,
		startPosition.X.Offset + delta.X,
		startPosition.Y.Scale,
		startPosition.Y.Offset + delta.Y
	)
end

title.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		dragging = true
		dragStart = input.Position
		startPosition = frame.Position

		input.Changed:Connect(function()

			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end

		end)
	end
end)

UserInputService.InputChanged:Connect(function(input)

	if dragging then

		if input.UserInputType == Enum.UserInputType.MouseMovement
			or input.UserInputType == Enum.UserInputType.Touch then

			updateDrag(input)

		end
	end
end)

-- =========================================
-- TÍTULO POSIÇÃO
-- =========================================

local positionTitle = Instance.new("TextLabel")
positionTitle.Size = UDim2.new(1, -20, 0, 22)
positionTitle.Position = UDim2.new(0, 10, 0, 40)
positionTitle.Text = "Posição:"
positionTitle.TextColor3 = Color3.new(1, 1, 1)
positionTitle.BackgroundTransparency = 1
positionTitle.TextScaled = true
positionTitle.Parent = frame

-- =========================================
-- BOTÕES DE POSIÇÃO
-- =========================================

local positionFrame = Instance.new("Frame")
positionFrame.Size = UDim2.new(1, -20, 0, 62)
positionFrame.Position = UDim2.new(0, 10, 0, 64)
positionFrame.BackgroundTransparency = 1
positionFrame.Parent = frame

local positionLayout = Instance.new("UIGridLayout")
positionLayout.CellSize = UDim2.new(0.31, 0, 0, 27)
positionLayout.CellPadding = UDim2.new(0.025, 0, 0, 4)
positionLayout.Parent = positionFrame

local positionButtons = {}

local function createPositionButton(text, mode)

	local button = Instance.new("TextButton")

	button.Text = text
	button.TextColor3 = Color3.new(1, 1, 1)
	button.TextScaled = true
	button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
	button.BorderSizePixel = 0
	button.Parent = positionFrame

	button.MouseButton1Click:Connect(function()

		positionMode = mode

		for currentMode, currentButton in pairs(positionButtons) do

			if currentMode == positionMode then
				currentButton.BackgroundColor3 =
					Color3.fromRGB(40, 140, 80)
			else
				currentButton.BackgroundColor3 =
					Color3.fromRGB(60, 60, 60)
			end

		end
	end)

	positionButtons[mode] = button
end

createPositionButton("Atrás", "Behind")
createPositionButton("Frente", "Front")
createPositionButton("Esquerda", "Left")
createPositionButton("Direita", "Right")
createPositionButton("Dentro", "Inside")

positionButtons["Behind"].BackgroundColor3 =
	Color3.fromRGB(40, 140, 80)

-- =========================================
-- DISTÂNCIA
-- =========================================

local distanceTitle = Instance.new("TextLabel")
distanceTitle.Size = UDim2.new(1, -20, 0, 22)
distanceTitle.Position = UDim2.new(0, 10, 0, 130)
distanceTitle.Text = "Distância:"
distanceTitle.TextColor3 = Color3.new(1, 1, 1)
distanceTitle.BackgroundTransparency = 1
distanceTitle.TextScaled = true
distanceTitle.Parent = frame

local distanceFrame = Instance.new("Frame")
distanceFrame.Size = UDim2.new(1, -40, 0, 32)
distanceFrame.Position = UDim2.new(0, 20, 0, 153)
distanceFrame.BackgroundTransparency = 1
distanceFrame.Parent = frame

local minusButton = Instance.new("TextButton")
minusButton.Size = UDim2.new(0, 45, 1, 0)
minusButton.Text = "−"
minusButton.TextColor3 = Color3.new(1, 1, 1)
minusButton.TextScaled = true
minusButton.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
minusButton.BorderSizePixel = 0
minusButton.Parent = distanceFrame

local distanceLabel = Instance.new("TextLabel")
distanceLabel.Size = UDim2.new(1, -100, 1, 0)
distanceLabel.Position = UDim2.new(0, 50, 0, 0)
distanceLabel.Text = tostring(distance)
distanceLabel.TextColor3 = Color3.new(1, 1, 1)
distanceLabel.TextScaled = true
distanceLabel.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
distanceLabel.BorderSizePixel = 0
distanceLabel.Parent = distanceFrame

local plusButton = Instance.new("TextButton")
plusButton.Size = UDim2.new(0, 45, 1, 0)
plusButton.Position = UDim2.new(1, -45, 0, 0)
plusButton.Text = "+"
plusButton.TextColor3 = Color3.new(1, 1, 1)
plusButton.TextScaled = true
plusButton.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
plusButton.BorderSizePixel = 0
plusButton.Parent = distanceFrame

local function updateDistance()

	distanceLabel.Text = tostring(distance)

end

minusButton.MouseButton1Click:Connect(function()

	distance = math.clamp(
		distance - 1,
		MIN_DISTANCE,
		MAX_DISTANCE
	)

	updateDistance()

end)

plusButton.MouseButton1Click:Connect(function()

	distance = math.clamp(
		distance + 1,
		MIN_DISTANCE,
		MAX_DISTANCE
	)

	updateDistance()

end)

-- =========================================
-- JOGADORES
-- =========================================

local listTitle = Instance.new("TextLabel")
listTitle.Size = UDim2.new(1, -20, 0, 22)
listTitle.Position = UDim2.new(0, 10, 0, 190)
listTitle.Text = "Escolha o jogador:"
listTitle.TextColor3 = Color3.new(1, 1, 1)
listTitle.BackgroundTransparency = 1
listTitle.TextScaled = true
listTitle.Parent = frame

local list = Instance.new("ScrollingFrame")
list.Size = UDim2.new(1, -20, 0, 105)
list.Position = UDim2.new(0, 10, 0, 215)
list.CanvasSize = UDim2.new(0, 0, 0, 0)
list.ScrollBarThickness = 5
list.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
list.BorderSizePixel = 0
list.Parent = frame

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 4)
layout.Parent = list

-- =========================================
-- PARAR
-- =========================================

local stopButton = Instance.new("TextButton")
stopButton.Size = UDim2.new(1, -20, 0, 42)
stopButton.Position = UDim2.new(0, 10, 1, -52)

stopButton.Text = "⛔ PARAR DE COPIAR"
stopButton.TextColor3 = Color3.new(1, 1, 1)
stopButton.TextScaled = true
stopButton.BackgroundColor3 = Color3.fromRGB(170, 40, 40)
stopButton.BorderSizePixel = 0
stopButton.Parent = frame

-- =========================================
-- PARAR ANIMAÇÕES
-- =========================================

local function stopCopiedAnimations()

	for animationId, track in pairs(copiedTracks) do

		pcall(function()
			track:Stop(0.1)
			track:Destroy()
		end)

		copiedTracks[animationId] = nil
	end

	copiedTracks = {}

end

-- =========================================
-- PARAR TUDO
-- =========================================

local function stopCopying()

	selectedPlayer = nil

	if followConnection then
		followConnection:Disconnect()
		followConnection = nil
	end

	if animationConnection then
		animationConnection:Disconnect()
		animationConnection = nil
	end

	stopCopiedAnimations()

	stopButton.Text = "⛔ PARADO"

	task.delay(1, function()

		if not selectedPlayer then
			stopButton.Text = "⛔ PARAR DE COPIAR"
		end

	end)
end

-- =========================================
-- COPIAR ANIMAÇÕES
-- =========================================

local function syncAnimations(targetHumanoid, myHumanoid)

	local targetAnimator =
		targetHumanoid:FindFirstChildOfClass("Animator")

	local myAnimator =
		myHumanoid:FindFirstChildOfClass("Animator")

	if not targetAnimator or not myAnimator then
		return
	end

	local activeAnimations = {}

	for _, targetTrack in ipairs(
		targetAnimator:GetPlayingAnimationTracks()
	) do

		if targetTrack.IsPlaying
			and targetTrack.Animation
			and targetTrack.Animation.AnimationId ~= "" then

			local animationId =
				targetTrack.Animation.AnimationId

			activeAnimations[animationId] = true

			local myTrack =
				copiedTracks[animationId]

			if not myTrack then

				local animation =
					Instance.new("Animation")

				animation.AnimationId =
					animationId

				local success, result =
					pcall(function()

						return myAnimator:LoadAnimation(
							animation
						)

					end)

				animation:Destroy()

				if success and result then

					myTrack = result
					copiedTracks[animationId] = myTrack

					pcall(function()
						myTrack.Priority =
							targetTrack.Priority
					end)

					pcall(function()
						myTrack.Looped =
							targetTrack.Looped
					end)

					myTrack:Play(
						0.05,
						targetTrack.WeightCurrent,
						targetTrack.Speed
					)
				end
			end

			if myTrack then

				pcall(function()

					myTrack:AdjustSpeed(
						targetTrack.Speed
					)

					myTrack:AdjustWeight(
						targetTrack.WeightCurrent,
						0.05
					)

					myTrack.Looped =
						targetTrack.Looped

					local difference =
						math.abs(
							myTrack.TimePosition -
							targetTrack.TimePosition
						)

					if difference > 0.15 then

						myTrack.TimePosition =
							targetTrack.TimePosition

					end

				end)
			end
		end
	end

	for animationId, myTrack in pairs(copiedTracks) do

		if not activeAnimations[animationId] then

			pcall(function()
				myTrack:Stop(0.1)
				myTrack:Destroy()
			end)

			copiedTracks[animationId] = nil
		end
	end
end

-- =========================================
-- COPIAR MOVIMENTO
-- =========================================

local function copyMovement(targetPlayer)

	stopCopying()

	selectedPlayer = targetPlayer

	-- Movimento

	followConnection =
		RunService.RenderStepped:Connect(function()

			if not selectedPlayer then
				return
			end

			local myCharacter = player.Character
			local targetCharacter = targetPlayer.Character

			if not myCharacter or not targetCharacter then
				return
			end

			local myRoot =
				myCharacter:FindFirstChild(
					"HumanoidRootPart"
				)

			local targetRoot =
				targetCharacter:FindFirstChild(
					"HumanoidRootPart"
				)

			if myRoot and targetRoot then

				local basePosition =
					positions[positionMode]

				local offset

				if positionMode == "Inside" then

					offset = Vector3.new(0, 0, 0)

				else

					offset =
						basePosition.Unit * distance

				end

				myRoot.CFrame =
					targetRoot.CFrame *
					CFrame.new(offset)

			end
		end)

	-- Animações

	animationConnection =
		RunService.RenderStepped:Connect(function()

			if not selectedPlayer then
				return
			end

			local myCharacter = player.Character
			local targetCharacter = targetPlayer.Character

			if not myCharacter or not targetCharacter then
				return
			end

			local myHumanoid =
				myCharacter:FindFirstChildOfClass(
					"Humanoid"
				)

			local targetHumanoid =
				targetCharacter:FindFirstChildOfClass(
					"Humanoid"
				)

			if myHumanoid and targetHumanoid then

				syncAnimations(
					targetHumanoid,
					myHumanoid
				)

			end
		end)
end

-- =========================================
-- ATUALIZAR LISTA
-- =========================================

local function updatePlayerList()

	for _, child in ipairs(list:GetChildren()) do

		if child:IsA("TextButton") then
			child:Destroy()
		end
	end

	for _, otherPlayer in ipairs(
		Players:GetPlayers()
	) do

		if otherPlayer ~= player then

			local button = Instance.new("TextButton")

			button.Size =
				UDim2.new(1, -8, 0, 30)

			button.Text =
				otherPlayer.DisplayName
				.. " (@"
				.. otherPlayer.Name
				.. ")"

			button.TextColor3 =
				Color3.new(1, 1, 1)

			button.TextScaled = true

			button.BackgroundColor3 =
				Color3.fromRGB(60, 60, 60)

			button.BorderSizePixel = 0
			button.Parent = list

			button.MouseButton1Click:Connect(function()

				copyMovement(otherPlayer)

			end)
		end
	end

	task.wait()

	list.CanvasSize =
		UDim2.new(
			0,
			0,
			0,
			layout.AbsoluteContentSize.Y + 5
		)
end

-- =========================================
-- EVENTOS
-- =========================================

stopButton.MouseButton1Click:Connect(
	stopCopying
)

Players.PlayerAdded:Connect(
	updatePlayerList
)

Players.PlayerRemoving:Connect(function(
	leavingPlayer
)

	if selectedPlayer == leavingPlayer then
		stopCopying()
	end

	updatePlayerList()

end)

updatePlayerList()# Copiar-movimentos
Copia os movimentos do jogador selecionado 
