-- Aimbot avançado por FOV com círculo, mira suave e ESP
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- CONFIGURAÇÕES
local AimPart = "Head" -- ou "HumanoidRootPart"
local AimFov = 120 -- raio de ativação do aimbot
local AimSmoothness = 0.15 -- 0 = snap instantâneo, 1 = sem mover
local AimKey = Enum.UserInputType.MouseButton2

-- VARIÁVEIS
local isAiming = false
local espBoxes = {}
local espNames = {}

-- CÍRCULO DO FOV
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = AimFov
fovCircle.Thickness = 2
fovCircle.Transparency = 1
fovCircle.Color = Color3.fromRGB(255, 0, 0)
fovCircle.Filled = false
fovCircle.Visible = true

-- FUNÇÃO PARA PEGAR O JOGADOR MAIS PRÓXIMO DENTRO DO FOV
local function getClosestPlayer()
	local closest = nil
	local shortestDist = AimFov
	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(AimPart) then
			local part = player.Character[AimPart]
			local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
			if onScreen then
				local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)).Magnitude
				if dist < shortestDist then
					shortestDist = dist
					closest = part
				end
			end
		end
	end
	return closest
end

-- FUNÇÃO PARA ATUALIZAR OS ESPs
local function updateESP()
	for _, box in pairs(espBoxes) do box.Visible = false end
	for _, name in pairs(espNames) do name.Visible = false end

	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local hrp = player.Character.HumanoidRootPart
			local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
			if onScreen then
				local distance = (Camera.CFrame.Position - hrp.Position).Magnitude
				local scale = 1 / distance * 100
				local width = 40 * scale
				local height = 60 * scale

				-- Caixa
				local box = espBoxes[player]
				if not box then
					box = Drawing.new("Square")
					box.Thickness = 1
					box.Color = Color3.fromRGB(0, 255, 0)
					box.Transparency = 1
					box.Filled = false
					espBoxes[player] = box
				end
				box.Size = Vector2.new(width, height)
				box.Position = Vector2.new(pos.X - width / 2, pos.Y - height / 2)
				box.Visible = true

				-- Nome
				local nameText = espNames[player]
				if not nameText then
					nameText = Drawing.new("Text")
					nameText.Color = Color3.fromRGB(0, 255, 0)
					nameText.Size = 14
					nameText.Center = true
					nameText.Outline = true
					nameText.Font = 2
					espNames[player] = nameText
				end
				nameText.Text = player.Name
				nameText.Position = Vector2.new(pos.X, pos.Y - height / 2 - 15)
				nameText.Visible = true
			end
		end
	end
end

-- DETECTAR TECLA PARA ATIVAR AIMBOT
UIS.InputBegan:Connect(function(input)
	if input.UserInputType == AimKey then
		isAiming = true
	end
end)

UIS.InputEnded:Connect(function(input)
	if input.UserInputType == AimKey then
		isAiming = false
	end
end)

-- LOOP PRINCIPAL
RunService.RenderStepped:Connect(function()
	fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

	updateESP()

	if isAiming then
		local targetPart = getClosestPlayer()
		if targetPart then
			local camPos = Camera.CFrame.Position
			local newCFrame = CFrame.new(camPos, targetPart.Position)
			Camera.CFrame = Camera.CFrame:Lerp(newCFrame, AimSmoothness)
		end
	end
end)
