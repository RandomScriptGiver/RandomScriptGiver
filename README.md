-- Mobile-Friendly Object Picker GUI

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local pickedObject = nil
local bodyPosition = nil
local bodyGyro = nil
local followDistance = 8

-- Tunggu PlayerGui
repeat wait() until player:FindFirstChild("PlayerGui")

-- Buat GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ObjectControlGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Crosshair "+"
local cross = Instance.new("TextLabel")
cross.Size = UDim2.new(0, 20, 0, 20)
cross.Position = UDim2.new(0.5, -10, 0.5, -10)
cross.Text = "+"
cross.BackgroundTransparency = 1
cross.TextColor3 = Color3.new(1, 1, 1)
cross.TextScaled = true
cross.Font = Enum.Font.SourceSansBold
cross.Parent = screenGui

-- Tombol builder
local function createButton(name, order, callback)
	local button = Instance.new("TextButton")
	button.Size = UDim2.new(0.4, 0, 0.06, 0)
	button.Position = UDim2.new(0.3, 0, 0.65 + (order * 0.07), 0)
	button.AnchorPoint = Vector2.new(0, 0)
	button.Text = name
	button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	button.TextColor3 = Color3.new(1, 1, 1)
	button.Font = Enum.Font.SourceSansBold
	button.TextScaled = true
	button.Parent = screenGui
	button.MouseButton1Click:Connect(callback)
end

-- Tombol fungsi
createButton("Take", 0, function()
	if pickedObject then return end

	local ray = workspace:Raycast(camera.CFrame.Position, camera.CFrame.LookVector * 100, RaycastParams.new())
	if ray and ray.Instance and ray.Instance:IsA("BasePart") and not ray.Instance.Anchored then
		pickedObject = ray.Instance

		bodyPosition = Instance.new("BodyPosition")
		bodyPosition.MaxForce = Vector3.new(1e9, 1e9, 1e9)
		bodyPosition.D = 1000
		bodyPosition.P = 20000
		bodyPosition.Position = pickedObject.Position
		bodyPosition.Parent = pickedObject

		bodyGyro = Instance.new("BodyGyro")
		bodyGyro.MaxTorque = Vector3.new(1e9, 1e9, 1e9)
		bodyGyro.D = 500
		bodyGyro.P = 10000
		bodyGyro.CFrame = camera.CFrame
		bodyGyro.Parent = pickedObject
	end
end)

createButton("Get Closer", 1, function()
	if pickedObject then
		followDistance = math.max(2, followDistance - 5)
	end
end)

createButton("Keep Away", 2, function()
	if pickedObject then
		followDistance = math.min(100, followDistance + 5)
	end
end)

createButton("Release", 3, function()
	if pickedObject then
		if bodyPosition then bodyPosition:Destroy() end
		if bodyGyro then bodyGyro:Destroy() end
		pickedObject = nil
		bodyPosition = nil
		bodyGyro = nil
	end
end)

-- Update posisi object tiap frame
RunService.RenderStepped:Connect(function()
	if pickedObject and bodyPosition and bodyGyro then
		local camCF = camera.CFrame
		local targetPos = camCF.Position + camCF.LookVector * followDistance
		local distance = (pickedObject.Position - targetPos).Magnitude

		bodyPosition.Position = targetPos
		bodyGyro.CFrame = CFrame.new(Vector3.zero, camCF.LookVector)

		-- Naikkan gaya dorong jika terlalu jauh
		if distance > 100 then
			bodyPosition.P = 100000
		else
			bodyPosition.P = 20000
		end
	end
end)
