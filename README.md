-- Venom domina - Menú de Movimiento para Roblox Studio (LocalScript)
-- Coloca este script dentro de StarterPlayer -> StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Configuración inicial
local REQUIRED_KEY = "Venom"
local DEFAULT_WALK_SPEED = 16
-- Pega aquí el ID de tu imagen de Roblox cuando la tengas (ejemplo: "rbxassetid://1234567890")
local MENU_IMAGE_ID = ""

local state = {
	unlocked = false,
	menuOpen = false,
	flyEnabled = false,
	flySpeed = 60,
	speedEnabled = false,
	customSpeed = 50,
	invisibleEnabled = false,
}

-- Limpiar instancia previa si existe
local existingGui = playerGui:FindFirstChild("VenomDominaGui")
if existingGui then
	existingGui:Destroy()
end

-- Crear ScreenGui principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "VenomDominaGui"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

-- Función auxiliar para hacer cualquier Frame/Button arrastrable (PC y móvil)
local function makeDraggable(dragHandle, targetFrame)
	local dragging = false
	local dragInput = nil
	local dragStart = nil
	local startPos = nil

	dragHandle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = targetFrame.Position

			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	dragHandle.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			dragInput = input
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragging then
			local delta = input.Position - dragStart
			targetFrame.Position = UDim2.new(
				startPos.X.Scale,
				startPos.X.Offset + delta.X,
				startPos.Y.Scale,
				startPos.Y.Offset + delta.Y
			)
		end
	end)
end

local function addCorner(parent, radius)
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, radius or 8)
	corner.Parent = parent
	return corner
end

-- ==========================================
-- 1. VENTANA DE KEY (Sistema de acceso)
-- ==========================================
local keyFrame = Instance.new("Frame")
keyFrame.Name = "KeyFrame"
keyFrame.Size = UDim2.new(0, 300, 0, 190)
keyFrame.Position = UDim2.new(0.5, -150, 0.5, -95)
keyFrame.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
keyFrame.BorderSizePixel = 0
keyFrame.Parent = screenGui
addCorner(keyFrame, 12)

local keyStroke = Instance.new("UIStroke")
keyStroke.Color = Color3.fromRGB(130, 50, 220)
keyStroke.Thickness = 2
keyStroke.Parent = keyFrame

local keyTitle = Instance.new("TextLabel")
keyTitle.Size = UDim2.new(1, 0, 0, 40)
keyTitle.BackgroundColor3 = Color3.fromRGB(32, 32, 42)
keyTitle.BorderSizePixel = 0
keyTitle.Text = "Venom domina - Key"
keyTitle.TextColor3 = Color3.fromRGB(240, 240, 255)
keyTitle.Font = Enum.Font.GothamBold
keyTitle.TextSize = 16
keyTitle.Parent = keyFrame
addCorner(keyTitle, 12)

local keyBox = Instance.new("TextBox")
keyBox.Size = UDim2.new(0.85, 0, 0, 38)
keyBox.Position = UDim2.new(0.075, 0, 0, 56)
keyBox.BackgroundColor3 = Color3.fromRGB(36, 36, 46)
keyBox.TextColor3 = Color3.fromRGB(255, 255, 255)
keyBox.PlaceholderText = "Escribe la key aquí..."
keyBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 170)
keyBox.Text = ""
keyBox.Font = Enum.Font.Gotham
keyBox.TextSize = 14
keyBox.ClearTextOnFocus = false
keyBox.Parent = keyFrame
addCorner(keyBox, 8)

local keyStatus = Instance.new("TextLabel")
keyStatus.Size = UDim2.new(0.85, 0, 0, 20)
keyStatus.Position = UDim2.new(0.075, 0, 0, 98)
keyStatus.BackgroundTransparency = 1
keyStatus.Text = ""
keyStatus.TextColor3 = Color3.fromRGB(255, 90, 90)
keyStatus.Font = Enum.Font.Gotham
keyStatus.TextSize = 12
keyStatus.Parent = keyFrame

local unlockBtn = Instance.new("TextButton")
unlockBtn.Size = UDim2.new(0.85, 0, 0, 38)
unlockBtn.Position = UDim2.new(0.075, 0, 0, 130)
unlockBtn.BackgroundColor3 = Color3.fromRGB(130, 50, 220)
unlockBtn.Text = "Entrar"
unlockBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
unlockBtn.Font = Enum.Font.GothamBold
unlockBtn.TextSize = 15
unlockBtn.Parent = keyFrame
addCorner(unlockBtn, 8)

makeDraggable(keyTitle, keyFrame)

-- ==========================================
-- 2. BOTÓN FLOTANTE CON FOTO PARA ABRIR EL MENÚ
-- ==========================================
local openBtn = Instance.new("ImageButton")
openBtn.Name = "OpenMenuButton"
openBtn.Size = UDim2.new(0, 62, 0, 62)
openBtn.Position = UDim2.new(0, 20, 0.5, -31)
openBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 32)
openBtn.Image = MENU_IMAGE_ID
openBtn.ScaleType = Enum.ScaleType.Crop
openBtn.Visible = false
openBtn.Parent = screenGui
addCorner(openBtn, 14)

local openStroke = Instance.new("UIStroke")
openStroke.Color = Color3.fromRGB(130, 50, 220)
openStroke.Thickness = 2
openStroke.Parent = openBtn

local openFallbackLabel = Instance.new("TextLabel")
openFallbackLabel.Size = UDim2.new(1, -6, 1, -6)
openFallbackLabel.Position = UDim2.new(0, 3, 0, 3)
openFallbackLabel.BackgroundTransparency = 1
openFallbackLabel.Text = "Venom"
openFallbackLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
openFallbackLabel.Font = Enum.Font.GothamBold
openFallbackLabel.TextSize = 13
openFallbackLabel.TextWrapped = true
openFallbackLabel.Visible = (MENU_IMAGE_ID == "")
openFallbackLabel.Parent = openBtn

makeDraggable(openBtn, openBtn)

-- ==========================================
-- 3. MENÚ PRINCIPAL (Venom domina)
-- ==========================================
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 330, 0, 470)
mainFrame.Position = UDim2.new(0.5, -165, 0.5, -235)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 26)
mainFrame.BorderSizePixel = 0
mainFrame.Visible = false
mainFrame.Parent = screenGui
addCorner(mainFrame, 12)

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(130, 50, 220)
mainStroke.Thickness = 2
mainStroke.Parent = mainFrame

-- Barra superior con fotito y título
local topBar = Instance.new("Frame")
topBar.Size = UDim2.new(1, 0, 0, 46)
topBar.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
topBar.BorderSizePixel = 0
topBar.Parent = mainFrame
addCorner(topBar, 12)

local menuIcon = Instance.new("ImageLabel")
menuIcon.Name = "MenuIcon"
menuIcon.Size = UDim2.new(0, 32, 0, 32)
menuIcon.Position = UDim2.new(0, 10, 0.5, -16)
menuIcon.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
menuIcon.Image = MENU_IMAGE_ID
menuIcon.ScaleType = Enum.ScaleType.Crop
menuIcon.Parent = topBar
addCorner(menuIcon, 8)

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -96, 1, 0)
titleLabel.Position = UDim2.new(0, 50, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Venom domina"
titleLabel.TextColor3 = Color3.fromRGB(240, 240, 255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 16
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = topBar

-- Botón X para cerrar el menú
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -38, 0.5, -15)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 55, 65)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 15
closeBtn.Parent = topBar
addCorner(closeBtn, 8)

makeDraggable(topBar, mainFrame)

-- Contenedor de opciones
local content = Instance.new("Frame")
content.Size = UDim2.new(1, -24, 1, -58)
content.Position = UDim2.new(0, 12, 0, 52)
content.BackgroundTransparency = 1
content.Parent = mainFrame

-- ==========================================
-- SECCIÓN: VOLAR (3D por todo el cielo + velocidad)
-- ==========================================
local flyToggleBtn = Instance.new("TextButton")
flyToggleBtn.Size = UDim2.new(1, 0, 0, 42)
flyToggleBtn.Position = UDim2.new(0, 0, 0, 4)
flyToggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
flyToggleBtn.Text = "Volar: OFF"
flyToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
flyToggleBtn.Font = Enum.Font.GothamBold
flyToggleBtn.TextSize = 15
flyToggleBtn.Parent = content
addCorner(flyToggleBtn, 8)

local flyControlRow = Instance.new("Frame")
flyControlRow.Size = UDim2.new(1, 0, 0, 38)
flyControlRow.Position = UDim2.new(0, 0, 0, 52)
flyControlRow.BackgroundTransparency = 1
flyControlRow.Parent = content

local flyMinusBtn = Instance.new("TextButton")
flyMinusBtn.Size = UDim2.new(0, 50, 1, 0)
flyMinusBtn.Position = UDim2.new(0, 0, 0, 0)
flyMinusBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 58)
flyMinusBtn.Text = "-10"
flyMinusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
flyMinusBtn.Font = Enum.Font.GothamBold
flyMinusBtn.TextSize = 14
flyMinusBtn.Parent = flyControlRow
addCorner(flyMinusBtn, 8)

local flySpeedInput = Instance.new("TextBox")
flySpeedInput.Size = UDim2.new(1, -112, 1, 0)
flySpeedInput.Position = UDim2.new(0, 56, 0, 0)
flySpeedInput.BackgroundColor3 = Color3.fromRGB(34, 34, 44)
flySpeedInput.Text = tostring(state.flySpeed)
flySpeedInput.PlaceholderText = "Vel. vuelo (ej. 60)"
flySpeedInput.TextColor3 = Color3.fromRGB(255, 255, 255)
flySpeedInput.Font = Enum.Font.GothamBold
flySpeedInput.TextSize = 14
flySpeedInput.Parent = flyControlRow
addCorner(flySpeedInput, 8)

local flyPlusBtn = Instance.new("TextButton")
flyPlusBtn.Size = UDim2.new(0, 50, 1, 0)
flyPlusBtn.Position = UDim2.new(1, -50, 0, 0)
flyPlusBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 58)
flyPlusBtn.Text = "+10"
flyPlusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
flyPlusBtn.Font = Enum.Font.GothamBold
flyPlusBtn.TextSize = 14
flyPlusBtn.Parent = flyControlRow
addCorner(flyPlusBtn, 8)

local flyHint = Instance.new("TextLabel")
flyHint.Size = UDim2.new(1, 0, 0, 28)
flyHint.Position = UDim2.new(0, 0, 0, 94)
flyHint.BackgroundTransparency = 1
flyHint.Text = "Apunta la cámara hacia el cielo para subir o usa Espacio / Shift"
flyHint.TextColor3 = Color3.fromRGB(165, 165, 185)
flyHint.Font = Enum.Font.Gotham
flyHint.TextSize = 11
flyHint.TextWrapped = true
flyHint.Parent = content

-- ==========================================
-- SECCIÓN: CORRER (Velocidad en tierra)
-- ==========================================
local speedToggleBtn = Instance.new("TextButton")
speedToggleBtn.Size = UDim2.new(1, 0, 0, 42)
speedToggleBtn.Position = UDim2.new(0, 0, 0, 134)
speedToggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
speedToggleBtn.Text = "Velocidad: OFF"
speedToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
speedToggleBtn.Font = Enum.Font.GothamBold
speedToggleBtn.TextSize = 15
speedToggleBtn.Parent = content
addCorner(speedToggleBtn, 8)

local speedControlRow = Instance.new("Frame")
speedControlRow.Size = UDim2.new(1, 0, 0, 38)
speedControlRow.Position = UDim2.new(0, 0, 0, 182)
speedControlRow.BackgroundTransparency = 1
speedControlRow.Parent = content

local minusBtn = Instance.new("TextButton")
minusBtn.Size = UDim2.new(0, 50, 1, 0)
minusBtn.Position = UDim2.new(0, 0, 0, 0)
minusBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 58)
minusBtn.Text = "-5"
minusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minusBtn.Font = Enum.Font.GothamBold
minusBtn.TextSize = 14
minusBtn.Parent = speedControlRow
addCorner(minusBtn, 8)

local speedInput = Instance.new("TextBox")
speedInput.Size = UDim2.new(1, -112, 1, 0)
speedInput.Position = UDim2.new(0, 56, 0, 0)
speedInput.BackgroundColor3 = Color3.fromRGB(34, 34, 44)
speedInput.Text = tostring(state.customSpeed)
speedInput.PlaceholderText = "Velocidad (ej. 50)"
speedInput.TextColor3 = Color3.fromRGB(255, 255, 255)
speedInput.Font = Enum.Font.GothamBold
speedInput.TextSize = 14
speedInput.Parent = speedControlRow
addCorner(speedInput, 8)

local plusBtn = Instance.new("TextButton")
plusBtn.Size = UDim2.new(0, 50, 1, 0)
plusBtn.Position = UDim2.new(1, -50, 0, 0)
plusBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 58)
plusBtn.Text = "+5"
plusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
plusBtn.Font = Enum.Font.GothamBold
plusBtn.TextSize = 14
plusBtn.Parent = speedControlRow
addCorner(plusBtn, 8)

local presetLabel = Instance.new("TextLabel")
presetLabel.Size = UDim2.new(1, 0, 0, 20)
presetLabel.Position = UDim2.new(0, 0, 0, 230)
presetLabel.BackgroundTransparency = 1
presetLabel.Text = "Ajustes rápidos de correr:"
presetLabel.TextColor3 = Color3.fromRGB(190, 190, 210)
presetLabel.Font = Enum.Font.Gotham
presetLabel.TextSize = 12
presetLabel.TextXAlignment = Enum.TextXAlignment.Left
presetLabel.Parent = content

local presets = {
	{ name = "Despacio", value = 8 },
	{ name = "Normal", value = 16 },
	{ name = "Rápido", value = 50 },
	{ name = "Turbo", value = 100 },
}

local presetContainer = Instance.new("Frame")
presetContainer.Size = UDim2.new(1, 0, 0, 36)
presetContainer.Position = UDim2.new(0, 0, 0, 254)
presetContainer.BackgroundTransparency = 1
presetContainer.Parent = content

local presetLayout = Instance.new("UIListLayout")
presetLayout.FillDirection = Enum.FillDirection.Horizontal
presetLayout.Padding = UDim.new(0, 6)
presetLayout.Parent = presetContainer

-- ==========================================
-- SECCIÓN: INVISIBLE (Solo visual para ti)
-- ==========================================
local invisibleToggleBtn = Instance.new("TextButton")
invisibleToggleBtn.Size = UDim2.new(1, 0, 0, 42)
invisibleToggleBtn.Position = UDim2.new(0, 0, 0, 304)
invisibleToggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
invisibleToggleBtn.Text = "Invisible: OFF"
invisibleToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
invisibleToggleBtn.Font = Enum.Font.GothamBold
invisibleToggleBtn.TextSize = 15
invisibleToggleBtn.Parent = content
addCorner(invisibleToggleBtn, 8)

-- ==========================================
-- 4. LÓGICA DE VELOCIDAD, VUELO 3D E INVISIBLE (Sin errores)
-- ==========================================
local flyAttachment = nil
local flyLinearVelocity = nil
local flyAlignOrientation = nil
local flyConnection = nil
local savedDecalTransparencies = {}

local function getCharacterParts()
	local char = player.Character
	if not char then
		return nil, nil, nil
	end
	local humanoid = char:FindFirstChildOfClass("Humanoid")
	local rootPart = char:FindFirstChild("HumanoidRootPart")
	return char, humanoid, rootPart
end

local function applyInvisibility()
	local char = player.Character
	if not char then
		return
	end

	for _, obj in ipairs(char:GetDescendants()) do
		if obj:IsA("BasePart") then
			obj.LocalTransparencyModifier = state.invisibleEnabled and 1 or 0
		elseif obj:IsA("Decal") or obj:IsA("Texture") then
			if state.invisibleEnabled then
				if savedDecalTransparencies[obj] == nil then
					savedDecalTransparencies[obj] = obj.Transparency
				end
				obj.Transparency = 1
			else
				if savedDecalTransparencies[obj] ~= nil then
					obj.Transparency = savedDecalTransparencies[obj]
					savedDecalTransparencies[obj] = nil
				end
			end
		end
	end
end

local function updateInvisibleUI()
	if state.invisibleEnabled then
		invisibleToggleBtn.Text = "Invisible: ON"
		invisibleToggleBtn.BackgroundColor3 = Color3.fromRGB(45, 165, 95)
	else
		invisibleToggleBtn.Text = "Invisible: OFF"
		invisibleToggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
	end
end

local function applyWalkSpeed()
	local _, humanoid = getCharacterParts()
	if not humanoid then
		return
	end
	if state.speedEnabled then
		humanoid.WalkSpeed = state.customSpeed
	else
		humanoid.WalkSpeed = DEFAULT_WALK_SPEED
	end
end

local function updateSpeedUI()
	speedInput.Text = tostring(state.customSpeed)
	if state.speedEnabled then
		speedToggleBtn.Text = "Velocidad: ON (" .. tostring(state.customSpeed) .. ")"
		speedToggleBtn.BackgroundColor3 = Color3.fromRGB(45, 165, 95)
	else
		speedToggleBtn.Text = "Velocidad: OFF"
		speedToggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
	end
end

local function setCustomSpeed(newSpeed)
	local clamped = math.clamp(math.floor(newSpeed), 1, 500)
	state.customSpeed = clamped
	updateSpeedUI()
	applyWalkSpeed()
end

local function updateFlyUI()
	flySpeedInput.Text = tostring(state.flySpeed)
	if state.flyEnabled then
		flyToggleBtn.Text = "Volar: ON (" .. tostring(state.flySpeed) .. ")"
		flyToggleBtn.BackgroundColor3 = Color3.fromRGB(45, 165, 95)
	else
		flyToggleBtn.Text = "Volar: OFF"
		flyToggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 65)
	end
end

local function setFlySpeed(newSpeed)
	local clamped = math.clamp(math.floor(newSpeed), 5, 500)
	state.flySpeed = clamped
	updateFlyUI()
end

local function stopFly()
	if flyConnection then
		flyConnection:Disconnect()
		flyConnection = nil
	end
	if flyLinearVelocity then
		flyLinearVelocity:Destroy()
		flyLinearVelocity = nil
	end
	if flyAlignOrientation then
		flyAlignOrientation:Destroy()
		flyAlignOrientation = nil
	end
	if flyAttachment then
		flyAttachment:Destroy()
		flyAttachment = nil
	end
	local _, humanoid = getCharacterParts()
	if humanoid then
		humanoid.PlatformStand = false
	end
end

local function startFly()
	stopFly()
	local _, humanoid, rootPart = getCharacterParts()
	if not humanoid or not rootPart then
		return
	end

	humanoid.PlatformStand = true

	flyAttachment = Instance.new("Attachment")
	flyAttachment.Name = "VenomFlyAttachment"
	flyAttachment.Parent = rootPart

	flyLinearVelocity = Instance.new("LinearVelocity")
	flyLinearVelocity.Name = "VenomFlyVelocity"
	flyLinearVelocity.Attachment0 = flyAttachment
	flyLinearVelocity.RelativeTo = Enum.ActuatorRelativeTo.World
	flyLinearVelocity.MaxForce = math.huge
	flyLinearVelocity.VectorVelocity = Vector3.zero
	flyLinearVelocity.Parent = rootPart

	flyAlignOrientation = Instance.new("AlignOrientation")
	flyAlignOrientation.Name = "VenomFlyAlign"
	flyAlignOrientation.Mode = Enum.OrientationAlignmentMode.OneAttachment
	flyAlignOrientation.Attachment0 = flyAttachment
	flyAlignOrientation.MaxTorque = math.huge
	flyAlignOrientation.Responsiveness = 50
	flyAlignOrientation.CFrame = rootPart.CFrame
	flyAlignOrientation.Parent = rootPart

	flyConnection = RunService.RenderStepped:Connect(function()
		local _, hum, root = getCharacterParts()
		local camera = workspace.CurrentCamera
		if not hum or not root or not camera or not flyLinearVelocity or not flyAlignOrientation then
			return
		end

		local camCF = camera.CFrame
		local camLookFlat = Vector3.new(camCF.LookVector.X, 0, camCF.LookVector.Z)
		local camRightFlat = Vector3.new(camCF.RightVector.X, 0, camCF.RightVector.Z)

		if camLookFlat.Magnitude > 0.001 then
			camLookFlat = camLookFlat.Unit
		end
		if camRightFlat.Magnitude > 0.001 then
			camRightFlat = camRightFlat.Unit
		end

		local moveDir = hum.MoveDirection
		local forwardAmount = moveDir:Dot(camLookFlat)
		local rightAmount = moveDir:Dot(camRightFlat)

		local vertical = 0
		if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
			vertical += 1
		end
		if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) or UserInputService:IsKeyDown(Enum.KeyCode.C) then
			vertical -= 1
		end

		local velocity = (camCF.LookVector * forwardAmount)
			+ (camCF.RightVector * rightAmount)
			+ Vector3.new(0, vertical, 0)

		if velocity.Magnitude > 0.001 then
			velocity = velocity.Unit * state.flySpeed
		else
			velocity = Vector3.zero
		end

		flyLinearVelocity.VectorVelocity = velocity
		flyAlignOrientation.CFrame = CFrame.lookAt(root.Position, root.Position + camCF.LookVector)
	end)
end

-- ==========================================
-- 5. CONEXIONES DE BOTONES Y EVENTOS
-- ==========================================
local function setMenuVisible(visible)
	state.menuOpen = visible
	mainFrame.Visible = visible
	openBtn.Visible = state.unlocked and not visible
end

local function tryUnlock()
	if keyBox.Text == REQUIRED_KEY then
		state.unlocked = true
		keyFrame.Visible = false
		setMenuVisible(true)
	else
		keyStatus.Text = "Key incorrecta. Intenta de nuevo."
	end
end

unlockBtn.MouseButton1Click:Connect(tryUnlock)
keyBox.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		tryUnlock()
	end
end)

closeBtn.MouseButton1Click:Connect(function()
	setMenuVisible(false)
end)

openBtn.MouseButton1Click:Connect(function()
	setMenuVisible(true)
end)

flyToggleBtn.MouseButton1Click:Connect(function()
	state.flyEnabled = not state.flyEnabled
	updateFlyUI()
	if state.flyEnabled then
		startFly()
	else
		stopFly()
	end
end)

flyMinusBtn.MouseButton1Click:Connect(function()
	setFlySpeed(state.flySpeed - 10)
end)

flyPlusBtn.MouseButton1Click:Connect(function()
	setFlySpeed(state.flySpeed + 10)
end)

flySpeedInput.FocusLost:Connect(function()
	local num = tonumber(flySpeedInput.Text)
	if num then
		setFlySpeed(num)
	else
		flySpeedInput.Text = tostring(state.flySpeed)
	end
end)

speedToggleBtn.MouseButton1Click:Connect(function()
	state.speedEnabled = not state.speedEnabled
	updateSpeedUI()
	applyWalkSpeed()
end)

minusBtn.MouseButton1Click:Connect(function()
	setCustomSpeed(state.customSpeed - 5)
end)

plusBtn.MouseButton1Click:Connect(function()
	setCustomSpeed(state.customSpeed + 5)
end)

speedInput.FocusLost:Connect(function()
	local num = tonumber(speedInput.Text)
	if num then
		setCustomSpeed(num)
	else
		speedInput.Text = tostring(state.customSpeed)
	end
end)

for _, preset in ipairs(presets) do
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0.25, -4.5, 1, 0)
	btn.BackgroundColor3 = Color3.fromRGB(40, 40, 54)
	btn.Text = preset.name
	btn.TextColor3 = Color3.fromRGB(235, 235, 250)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 12
	btn.Parent = presetContainer
	addCorner(btn, 6)

	btn.MouseButton1Click:Connect(function()
		state.speedEnabled = true
		setCustomSpeed(preset.value)
	end)
end

invisibleToggleBtn.MouseButton1Click:Connect(function()
	state.invisibleEnabled = not state.invisibleEnabled
	updateInvisibleUI()
	applyInvisibility()
end)

-- Mantener estado limpio al reaparecer el personaje
player.CharacterAdded:Connect(function(character)
	savedDecalTransparencies = {}
	character:WaitForChild("Humanoid")
	character:WaitForChild("HumanoidRootPart")
	task.wait(0.1)
	applyWalkSpeed()
	if state.flyEnabled then
		startFly()
	end
	if state.invisibleEnabled then
		applyInvisibility()
	end
end)

updateFlyUI()
updateSpeedUI()
updateInvisibleUI()
