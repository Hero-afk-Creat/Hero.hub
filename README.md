--[[
Script de Teste Interno - YBA ESP + Auto Farm + GUI
Desenvolvido para fins de detecção e testes sob autorização.
]]

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- GUI Base
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "YBATestGUI"

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 300, 0, 400)
MainFrame.Position = UDim2.new(0, 50, 0.5, -200)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Name = "MainFrame"

-- Título
local Title = Instance.new("TextLabel", MainFrame)
Title.Text = "YBA Test Interface"
Title.Size = UDim2.new(1, 0, 0, 40)
Title.TextColor3 = Color3.new(1,1,1)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.SourceSansBold
Title.TextScaled = true

-- Botões
function createButton(text, yPosition, callback)
	local button = Instance.new("TextButton", MainFrame)
	button.Text = text
	button.Size = UDim2.new(1, -20, 0, 40)
	button.Position = UDim2.new(0, 10, 0, yPosition)
	button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	button.TextColor3 = Color3.new(1, 1, 1)
	button.Font = Enum.Font.SourceSans
	button.TextScaled = true
	button.MouseButton1Click:Connect(callback)
end

-- ESP Function
local espActive = false
function toggleESP()
	espActive = not espActive
	if espActive then
		for _, player in pairs(Players:GetPlayers()) do
			if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
				local billboard = Instance.new("BillboardGui", player.Character)
				billboard.Name = "ESP"
				billboard.Size = UDim2.new(0, 100, 0, 20)
				billboard.Adornee = player.Character.HumanoidRootPart
				billboard.AlwaysOnTop = true
				local nameLabel = Instance.new("TextLabel", billboard)
				nameLabel.Size = UDim2.new(1, 0, 1, 0)
				nameLabel.Text = player.Name
				nameLabel.BackgroundTransparency = 1
				nameLabel.TextColor3 = Color3.new(1, 0, 0)
			end
		end
	else
		for _, player in pairs(Players:GetPlayers()) do
			if player.Character and player.Character:FindFirstChild("ESP") then
				player.Character.ESP:Destroy()
			end
		end
	end
end

-- Auto Farm
local autoFarmActive = false
function toggleAutoFarm()
	autoFarmActive = not autoFarmActive
	if autoFarmActive then
		spawn(function()
			while autoFarmActive do
				local nearestMob = nil
				local shortestDistance = math.huge
				for _, v in pairs(workspace:GetDescendants()) do
					if v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Name ~= LocalPlayer.Name then
						local dist = (v.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
						if dist < shortestDistance then
							shortestDistance = dist
							nearestMob = v
						end
					end
				end
				if nearestMob then
					LocalPlayer.Character.HumanoidRootPart.CFrame = nearestMob.HumanoidRootPart.CFrame * CFrame.new(0, 0, -3)
					wait(0.2)
					-- Simula ataque básico
					local args = {[1] = "LightAttack"}
					game.ReplicatedStorage:WaitForChild("CombatRemote"):FireServer(unpack(args))
				end
				wait(0.5)
			end
		end)
	end
end

-- Botões da Interface
createButton("Ativar/Desativar ESP", 50, toggleESP)
createButton("Ativar/Desativar AutoFarm", 100, toggleAutoFarm)

-- Drag da Interface
local UIS = game:GetService("UserInputService")
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = MainFrame.Position
	end
end)
MainFrame.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement then
		dragInput = input
	end
end)
UIS.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		local delta = input.Position - dragStart
		MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
			startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)
UIS.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)
