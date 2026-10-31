-- LocalScript em StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local FOV_MIN = 60
local FOV_MAX = 300
local FOV_INICIAL = 160
local ESPESSURA = 3
local SUAVIDADE = 0.2
local DISTANCIA_CAMERA = 12
local ALTURA_CAMERA = 2

local CORES = {
	Color3.fromRGB(255, 0, 0),     -- vermelho
	Color3.fromRGB(0, 255, 0),     -- verde
	Color3.fromRGB(0, 150, 255),   -- azul
	Color3.fromRGB(255, 255, 0),   -- amarelo
	Color3.fromRGB(255, 255, 255), -- branco
}

local LARGURA_PAINEL = 240
local ALTURA_PAINEL = 280
local ALTURA_TITULO = 35

local ativo = false          -- liga/desliga o aimbot
local mostrarFOV = true      -- mostra/esconde o círculo
local wallcheck = false      -- OFF = mira através da parede, ON = precisa de linha de visão
local alvo = nil
local mirando = false
local arrastandoSlider = false
local arrastandoPainel = false
local minimizado = false
local corSelecionada = 1

-- ===== GUI principal =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SimpleAimbot"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.fromOffset(LARGURA_PAINEL, ALTURA_PAINEL)
frame.Position = UDim2.fromOffset(20, 200)
frame.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
frame.BorderSizePixel = 0
frame.ClipsDescendants = false -- precisa ser false para a label ficar fora do painel
frame.Parent = screenGui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

-- Label "Made By" embaixo do painel (acompanha o painel quando ele é movido)
local creditos = Instance.new("TextLabel")
creditos.Size = UDim2.new(1, 0, 0, 18)
creditos.Position = UDim2.new(0, 0, 1, 4)
creditos.BackgroundTransparency = 1
creditos.Text = "Made By yanyxz"
creditos.TextColor3 = Color3.fromRGB(150, 150, 150)
creditos.Font = Enum.Font.Gotham
creditos.TextSize = 13
creditos.Parent = frame

local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1, -40, 0, ALTURA_TITULO)
titulo.BackgroundTransparency = 1
titulo.Text = "Simple Aimbot"
titulo.TextColor3 = Color3.new(1, 1, 1)
titulo.Font = Enum.Font.GothamBold
titulo.TextSize = 18
titulo.Parent = frame

local btnMin = Instance.new("TextButton")
btnMin.Size = UDim2.fromOffset(28, 24)
btnMin.Position = UDim2.new(1, -34, 0, 5)
btnMin.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
btnMin.BorderSizePixel = 0
btnMin.TextColor3 = Color3.new(1, 1, 1)
btnMin.Font = Enum.Font.GothamBold
btnMin.TextSize = 18
btnMin.Text = "-"
btnMin.Parent = frame
Instance.new("UICorner", btnMin).CornerRadius = UDim.new(0, 6)

local conteudo = Instance.new("Frame")
conteudo.Size = UDim2.new(1, 0, 0, ALTURA_PAINEL - ALTURA_TITULO)
conteudo.Position = UDim2.fromOffset(0, ALTURA_TITULO)
conteudo.BackgroundTransparency = 1
conteudo.Parent = frame

-- Botão Aimbot (liga/desliga)
local botaoAimbot = Instance.new("TextButton")
botaoAimbot.Size = UDim2.new(0, 200, 0, 36)
botaoAimbot.Position = UDim2.new(0.5, -100, 0, 0)
botaoAimbot.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
botaoAimbot.BorderSizePixel = 0
botaoAimbot.TextColor3 = Color3.new(1, 1, 1)
botaoAimbot.Font = Enum.Font.GothamSemibold
botaoAimbot.TextSize = 16
botaoAimbot.Text = "Aimbot: OFF"
botaoAimbot.Parent = conteudo
Instance.new("UICorner", botaoAimbot).CornerRadius = UDim.new(0, 8)

-- Botão Show FOV
local botaoMostrar = Instance.new("TextButton")
botaoMostrar.Size = UDim2.new(0, 200, 0, 28)
botaoMostrar.Position = UDim2.new(0.5, -100, 0, 42)
botaoMostrar.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
botaoMostrar.BorderSizePixel = 0
botaoMostrar.TextColor3 = Color3.new(1, 1, 1)
botaoMostrar.Font = Enum.Font.GothamSemibold
botaoMostrar.TextSize = 14
botaoMostrar.Text = "Show FOV: ON"
botaoMostrar.Parent = conteudo
Instance.new("UICorner", botaoMostrar).CornerRadius = UDim.new(0, 8)

-- Slider de FOV
local textoFOV = Instance.new("TextLabel")
textoFOV.Size = UDim2.new(1, 0, 0, 20)
textoFOV.Position = UDim2.fromOffset(0, 80)
textoFOV.BackgroundTransparency = 1
textoFOV.TextColor3 = Color3.new(1, 1, 1)
textoFOV.Font = Enum.Font.Gotham
textoFOV.TextSize = 14
textoFOV.Parent = conteudo

local barra = Instance.new("Frame")
barra.Size = UDim2.new(0, 200, 0, 8)
barra.Position = UDim2.new(0.5, -100, 0, 108)
barra.BackgroundColor3 = Color3.fromRGB(90, 90, 90)
barra.BorderSizePixel = 0
barra.Parent = conteudo
Instance.new("UICorner", barra).CornerRadius = UDim.new(1, 0)

local bolinhaSlider = Instance.new("Frame")
bolinhaSlider.Size = UDim2.fromOffset(16, 16)
bolinhaSlider.AnchorPoint = Vector2.new(0.5, 0.5)
bolinhaSlider.Position = UDim2.new(0, 0, 0.5, 0)
bolinhaSlider.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
bolinhaSlider.BorderSizePixel = 0
bolinhaSlider.Parent = barra
Instance.new("UICorner", bolinhaSlider).CornerRadius = UDim.new(1, 0)

-- Cor do FOV
local textoCor = Instance.new("TextLabel")
textoCor.Size = UDim2.new(1, 0, 0, 20)
textoCor.Position = UDim2.fromOffset(0, 130)
textoCor.BackgroundTransparency = 1
textoCor.Text = "FOV colour"
textoCor.TextColor3 = Color3.new(1, 1, 1)
textoCor.Font = Enum.Font.Gotham
textoCor.TextSize = 14
textoCor.Parent = conteudo

local swatches = {}
local TAM_SWATCH = 24
local ESPACO = 10
local larguraTotal = #CORES * TAM_SWATCH + (#CORES - 1) * ESPACO
local inicioX = (LARGURA_PAINEL - larguraTotal) / 2

-- Botão Wallcheck
local botaoWall = Instance.new("TextButton")
botaoWall.Size = UDim2.new(0, 200, 0, 32)
botaoWall.Position = UDim2.new(0.5, -100, 0, 190)
botaoWall.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
botaoWall.BorderSizePixel = 0
botaoWall.TextColor3 = Color3.new(1, 1, 1)
botaoWall.Font = Enum.Font.GothamSemibold
botaoWall.TextSize = 14
botaoWall.Text = "Wallcheck: OFF"
botaoWall.Parent = conteudo
Instance.new("UICorner", botaoWall).CornerRadius = UDim.new(0, 8)

-- ===== FOV (círculo) =====
local tamanhoAtual = FOV_INICIAL

local bola = Instance.new("Frame")
bola.Name = "FOV"
bola.Size = UDim2.fromOffset(tamanhoAtual, tamanhoAtual)
bola.AnchorPoint = Vector2.new(0.5, 0.5)
bola.Position = UDim2.new(0.5, 0, 0.5, 0)
bola.BackgroundTransparency = 1
bola.BorderSizePixel = 0
bola.ZIndex = 5
bola.Visible = false
bola.Parent = screenGui
Instance.new("UICorner", bola).CornerRadius = UDim.new(0.5, 0)

local contorno = Instance.new("UIStroke")
contorno.Color = CORES[corSelecionada]
contorno.Thickness = ESPESSURA
contorno.Parent = bola

local function atualizarVisibilidadeFOV()
	bola.Visible = ativo and mostrarFOV
end

local function selecionarCor(indice)
	corSelecionada = indice
	contorno.Color = CORES[indice]
	for i, sw in ipairs(swatches) do
		sw.BorderSizePixel = (i == indice) and 2 or 0
	end
end

for i, cor in ipairs(CORES) do
	local sw = Instance.new("TextButton")
	sw.Size = UDim2.fromOffset(TAM_SWATCH, TAM_SWATCH)
	sw.Position = UDim2.fromOffset(inicioX + (i - 1) * (TAM_SWATCH + ESPACO), 155)
	sw.BackgroundColor3 = cor
	sw.BorderColor3 = Color3.fromRGB(255, 255, 255)
	sw.BorderSizePixel = 0
	sw.Text = ""
	sw.AutoButtonColor = true
	sw.Parent = conteudo
	Instance.new("UICorner", sw).CornerRadius = UDim.new(1, 0)
	swatches[i] = sw

	sw.MouseButton1Click:Connect(function()
		selecionarCor(i)
	end)
end

selecionarCor(corSelecionada)

-- ===== Tamanho do FOV =====
local function atualizarTamanho(pct)
	pct = math.clamp(pct, 0, 1)
	tamanhoAtual = math.floor(FOV_MIN + (FOV_MAX - FOV_MIN) * pct)
	bola.Size = UDim2.fromOffset(tamanhoAtual, tamanhoAtual)
	bolinhaSlider.Position = UDim2.new(pct, 0, 0.5, 0)
	textoFOV.Text = "Fov: " .. tamanhoAtual
end

atualizarTamanho((FOV_INICIAL - FOV_MIN) / (FOV_MAX - FOV_MIN))

local function pctDoMouse(x)
	return (x - barra.AbsolutePosition.X) / barra.AbsoluteSize.X
end

barra.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		arrastandoSlider = true
		atualizarTamanho(pctDoMouse(input.Position.X))
	end
end)

bolinhaSlider.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		arrastandoSlider = true
	end
end)

-- ===== Minimizar / restaurar =====
btnMin.MouseButton1Click:Connect(function()
	minimizado = not minimizado
	conteudo.Visible = not minimizado
	btnMin.Text = minimizado and "+" or "-"
	frame.Size = minimizado
		and UDim2.fromOffset(LARGURA_PAINEL, ALTURA_TITULO)
		or UDim2.fromOffset(LARGURA_PAINEL, ALTURA_PAINEL)
end)

-- ===== Arrastar o painel pela barra de título =====
local offsetArraste = Vector2.new(0, 0)

titulo.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		arrastandoPainel = true
		offsetArraste = Vector2.new(input.Position.X, input.Position.Y) - frame.AbsolutePosition
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input.UserInputType ~= Enum.UserInputType.MouseMovement then return end

	if arrastandoSlider then
		atualizarTamanho(pctDoMouse(input.Position.X))
	end

	if arrastandoPainel then
		local vp = camera.ViewportSize
		local x = math.clamp(input.Position.X - offsetArraste.X, 0, vp.X - frame.AbsoluteSize.X)
		local y = math.clamp(input.Position.Y - offsetArraste.Y, 0, vp.Y - frame.AbsoluteSize.Y - 22)
		frame.Position = UDim2.fromOffset(x, y)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		arrastandoSlider = false
		arrastandoPainel = false
	end
end)

-- ===== Wallcheck: linha de visão =====
local function temLinhaDeVisao(p, minhaCabeca, cabeca)
	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Exclude
	params.FilterDescendantsInstances = { player.Character, p.Character }

	local direcao = cabeca.Position - minhaCabeca.Position
	local resultado = workspace:Raycast(minhaCabeca.Position, direcao, params)
	return resultado == nil
end

-- Retorna true se o alvo pode ser mirado com o wallcheck atual
local function alvoVisivel(p)
	if not wallcheck then return true end -- OFF: mira através da parede

	local minhaCabeca = player.Character and player.Character:FindFirstChild("Head")
	local cabeca = p.Character and p.Character:FindFirstChild("Head")
	if not minhaCabeca or not cabeca then return false end

	return temLinhaDeVisao(p, minhaCabeca, cabeca)
end

-- ===== Detecção: jogador dentro do círculo =====
local function jogadorNoFOV(p)
	local char = p.Character
	if not char then return false end

	local hum = char:FindFirstChildOfClass("Humanoid")
	local cabeca = char:FindFirstChild("Head")
	if not hum or hum.Health <= 0 or not cabeca then return false end

	local pos, naFrente = camera:WorldToViewportPoint(cabeca.Position)
	if not naFrente then return false end

	local centro = bola.AbsolutePosition + bola.AbsoluteSize / 2
	local raio = bola.AbsoluteSize.X / 2
	if (Vector2.new(pos.X, pos.Y) - centro).Magnitude > raio then return false end

	return alvoVisivel(p)
end

local function atualizarAlvo()
	if alvo and jogadorNoFOV(alvo) then return end

	alvo = nil
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= player and jogadorNoFOV(p) then
			alvo = p
			return
		end
	end
end

-- ===== Botões =====
botaoAimbot.MouseButton1Click:Connect(function()
	ativo = not ativo
	botaoAimbot.Text = ativo and "Aimbot: ON" or "Aimbot: OFF"
	atualizarVisibilidadeFOV()

	if not ativo then
		alvo = nil
		mirando = false
		if camera.CameraType == Enum.CameraType.Scriptable then
			camera.CameraType = Enum.CameraType.Custom
		end
	end
end)

botaoMostrar.MouseButton1Click:Connect(function()
	mostrarFOV = not mostrarFOV
	botaoMostrar.Text = mostrarFOV and "Show FOV: ON" or "Show FOV: OFF"
	atualizarVisibilidadeFOV()
end)

botaoWall.MouseButton1Click:Connect(function()
	wallcheck = not wallcheck
	botaoWall.Text = wallcheck and "Wallcheck: ON" or "Wallcheck: OFF"
	alvo = nil
end)

-- ===== Botão direito do mouse =====
UserInputService.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		mirando = true
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		mirando = false
	end
end)

-- ===== Mira em terceira pessoa =====
RunService.RenderStepped:Connect(function()
	if not ativo then return end

	atualizarAlvo()

	local char = player.Character
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	local cabeca = alvo and alvo.Character and alvo.Character:FindFirstChild("Head")

	if mirando and cabeca and hrp then
		camera.CameraType = Enum.CameraType.Scriptable

		local foco = hrp.Position + Vector3.new(0, ALTURA_CAMERA, 0)
		local horizontal = Vector3.new(cabeca.Position.X - hrp.Position.X, 0, cabeca.Position.Z - hrp.Position.Z)
		local direcao = horizontal.Magnitude > 0.1 and horizontal.Unit or hrp.CFrame.LookVector

		local posCamera = foco - direcao * DISTANCIA_CAMERA
		local destino = CFrame.lookAt(posCamera, cabeca.Position)
		camera.CFrame = camera.CFrame:Lerp(destino, SUAVIDADE)
	elseif camera.CameraType == Enum.CameraType.Scriptable then
		camera.CameraType = Enum.CameraType.Custom
	end
end)
