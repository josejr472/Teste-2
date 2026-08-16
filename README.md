-- Davi Hub - Aim Assist
-- Coloque como LocalScript em StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local enabled = false
local guiOpen = true

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "DaviHub"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(230, 130)
main.Position = UDim2.new(0.5, -115, 0.5, -65)
main.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
main.BorderSizePixel = 0
main.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = main

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.Text = "Davi Hub"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 22
title.Font = Enum.Font.GothamBold
title.Parent = main

local toggle = Instance.new("TextButton")
toggle.Size = UDim2.new(1, -30, 0, 40)
toggle.Position = UDim2.fromOffset(15, 50)
toggle.Text = "Aim Assist: OFF"
toggle.TextSize = 16
toggle.Font = Enum.Font.GothamBold
toggle.TextColor3 = Color3.new(1, 1, 1)
toggle.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
toggle.Parent = main

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 8)
toggleCorner.Parent = toggle

-- Botão para abrir/fechar
local openButton = Instance.new("TextButton")
openButton.Size = UDim2.fromOffset(55, 55)
openButton.Position = UDim2.fromOffset(15, 200)
openButton.Text = "D"
openButton.TextSize = 24
openButton.Font = Enum.Font.GothamBold
openButton.TextColor3 = Color3.new(1, 1, 1)
openButton.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
openButton.Visible = false
openButton.Parent = gui

local openCorner = Instance.new("UICorner")
openCorner.CornerRadius = UDim.new(1, 0)
openCorner.Parent = openButton

-- Encontrar o jogador mais próximo do centro da câmera
local function getTarget()
	local closestPlayer = nil
	local closestDistance = math.huge

	local center = Vector2.new(
		camera.ViewportSize.X / 2,
		camera.ViewportSize.Y / 2
	)

	for _, otherPlayer in ipairs(Players:GetPlayers()) do
		if otherPlayer ~= player then
			local character = otherPlayer.Character
			local head = character and character:FindFirstChild("Head")
			local humanoid = character and character:FindFirstChildOfClass("Humanoid")

			if head and humanoid and humanoid.Health > 0 then
				local screenPosition, visible =
					camera:WorldToViewportPoint(head.Position)

				if visible and screenPosition.Z > 0 then
					local distance = (
						Vector2.new(screenPosition.X, screenPosition.Y)
						- center
					).Magnitude

					if distance < closestDistance then
						closestDistance = distance
						closestPlayer = otherPlayer
					end
				end
			end
		end
	end

	return closestPlayer
end

toggle.MouseButton1Click:Connect(function()
	enabled = not enabled

	if enabled then
		toggle.Text = "Aim Assist: ON"
	else
		toggle.Text = "Aim Assist: OFF"
	end
end)

openButton.MouseButton1Click:Connect(function()
	guiOpen = true
	main.Visible = true
	openButton.Visible = false
end)

-- Criar botão de fechar
local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.fromOffset(35, 30)
closeButton.Position = UDim2.new(1, -40, 0, 5)
closeButton.Text = "X"
closeButton.TextSize = 16
closeButton.Font = Enum.Font.GothamBold
closeButton.TextColor3 = Color3.new(1, 1, 1)
closeButton.BackgroundTransparency = 1
closeButton.Parent = main

closeButton.MouseButton1Click:Connect(function()
	guiOpen = false
	main.Visible = false
	openButton.Visible = true
end)

-- Aim Assist
RunService:BindToRenderStep(
	"DaviHubAimAssist",
	Enum.RenderPriority.Camera.Value + 1,
	function()
		if not enabled then
			return
		end

		local target = getTarget()

		if target and target.Character then
			local head = target.Character:FindFirstChild("Head")

			if head then
				local currentCFrame = camera.CFrame
				local targetCFrame = CFrame.lookAt(
					currentCFrame.Position,
					head.Position
				)

				camera.CFrame = currentCFrame:Lerp(
					targetCFrame,
					0.12
				)
			end
		end
	end
)
