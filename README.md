-- LocalScript (StarterGui)

local UIS = game:GetService("UserInputService")
local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local humanoidRoot = char:WaitForChild("HumanoidRootPart")

-- Atualiza refs após respawn
player.CharacterAdded:Connect(function(newChar)
	char = newChar
	humanoidRoot = char:WaitForChild("HumanoidRootPart")
end)

-- ===== GUI =====
local screenGui = Instance.new("ScreenGui")
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Container arrastável
local container = Instance.new("Frame")
container.Size = UDim2.new(0, 220, 0, 200)
container.Position = UDim2.new(0.5, -110, 0.6, 0)
container.BackgroundTransparency = 1
container.Parent = screenGui

local function criarBotao(nome, texto, pos)
	local botao = Instance.new("TextButton")
	botao.Name = nome
	botao.Size = UDim2.new(0, 200, 0, 50)
	botao.Position = pos
	botao.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	botao.Text = ""
	botao.Font = Enum.Font.FredokaOne
	botao.TextScaled = true
	botao.Parent = container

	-- Fundo preto semitransparente atrás do texto
	local textoLabel = Instance.new("TextLabel")
	textoLabel.Size = UDim2.new(1, -10, 1, -10)
	textoLabel.Position = UDim2.new(0, 5, 0, 5)
	textoLabel.BackgroundColor3 = Color3.fromRGB(0,0,0)
	textoLabel.BackgroundTransparency = 0.5
	textoLabel.Text = texto
	textoLabel.TextColor3 = Color3.fromRGB(255,255,255)
	textoLabel.Font = Enum.Font.FredokaOne
	textoLabel.TextScaled = true
	textoLabel.Parent = botao

	return botao, textoLabel
end

local botaoToggle, labelToggle = criarBotao("BotaoToggle", "Ativar Chão", UDim2.new(0, 10, 0, 10))
local botaoParar,  labelParar  = criarBotao("BotaoParar",  "Parar Chão",  UDim2.new(0, 10, 0, 70))
local botaoESP,    labelESP    = criarBotao("BotaoESP",    "Esp Player",  UDim2.new(0, 10, 0, 130))

local labelAutor = Instance.new("TextLabel")
labelAutor.Size = UDim2.new(0, 200, 0, 30)
labelAutor.Position = UDim2.new(0, 10, 0, 180)
labelAutor.BackgroundColor3 = Color3.fromRGB(0,0,0)
labelAutor.BackgroundTransparency = 0.5
labelAutor.Text = "by Darkzin"
labelAutor.Font = Enum.Font.FredokaOne
labelAutor.TextScaled = true
labelAutor.TextColor3 = Color3.fromRGB(255,255,255)
labelAutor.Parent = container

-- Botão redondo para abrir/fechar container
local menuButton = Instance.new("TextButton")
menuButton.Size = UDim2.new(0, 60, 0, 60)
menuButton.Position = UDim2.new(0.05, 0, 0.8, 0)
menuButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
menuButton.Text = ""
menuButton.TextScaled = true
menuButton.Font = Enum.Font.FredokaOne
menuButton.TextColor3 = Color3.fromRGB(255,255,255)
menuButton.Parent = screenGui

local uicorner = Instance.new("UICorner")
uicorner.CornerRadius = UDim.new(1,0)
uicorner.Parent = menuButton

task.spawn(function()
	while menuButton.Parent do
		menuButton.BackgroundColor3 = Color3.fromRGB(math.random(0,255), math.random(0,255), math.random(0,255))
		task.wait(0.5)
	end
end)

local containerVisivel = true
menuButton.MouseButton1Click:Connect(function()
	containerVisivel = not containerVisivel
	container.Visible = containerVisivel
end)

-- ===== Drag custom: mover container
local function makeDragHandle(handle, target)
	local dragging = false
	local dragStart, startPos

	handle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = target.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	UIS.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			local delta = input.Position - dragStart
			target.Position = UDim2.new(
				startPos.X.Scale, startPos.X.Offset + delta.X,
				startPos.Y.Scale, startPos.Y.Offset + delta.Y
			)
		end
	end)
end

makeDragHandle(container, container)
makeDragHandle(botaoToggle, container)
makeDragHandle(botaoParar, container)
makeDragHandle(botaoESP, container)
makeDragHandle(labelAutor, container)

-- ===== Chão invisível =====
local velocidade = 10
local limiteSubida = 500
local chao = nil
local subindo = false
local baseY = nil

local function criarChao()
	if not chao then
		baseY = (humanoidRoot and humanoidRoot.Position.Y or 0) - 10
		chao = Instance.new("Part")
		chao.Size = Vector3.new(2048, 2, 2048)
		chao.Anchored = true
		chao.Position = Vector3.new(0, baseY, 0)
		chao.Color = Color3.fromRGB(34,139,34)
		chao.Transparency = 1
		chao.CanCollide = true
		chao.Name = "ChaoGigante"
		chao.Parent = workspace
	end
end

task.spawn(function()
	while true do
		local dt = task.wait()
		if chao and subindo and baseY then
			if chao.Position.Y < baseY + limiteSubida then
				local deslocamento = velocidade * dt
				chao.CFrame = chao.CFrame * CFrame.new(0, deslocamento, 0)
			end

			-- Mantém o personagem em cima do chão sem impedir movimento
			if humanoidRoot then
				local distancia = chao.Position.Y + 5 - humanoidRoot.Position.Y
				if distancia > 0 then
					humanoidRoot.CFrame = CFrame.new(humanoidRoot.Position.X, humanoidRoot.Position.Y + distancia, humanoidRoot.Position.Z)
				end
			end
		end
	end
end)

-- Botão Ativar / Tirar Chão
botaoToggle.MouseButton1Click:Connect(function()
	if not chao then
		criarChao()
		subindo = true
		labelToggle.Text = "Tirar Chão"
	else
		chao:Destroy()
		chao = nil
		subindo = false
		baseY = nil
		labelToggle.Text = "Ativar Chão"
	end
end)

-- Botão Parar apenas pausa o chão
botaoParar.MouseButton1Click:Connect(function()
	if chao then
		subindo = false
	end
end)

-- ===== Botão ESP =====
local espAtivo = false
local highlights = {}
local nameTags = {}

botaoESP.MouseButton1Click:Connect(function()
	espAtivo = not espAtivo
	if espAtivo then
		for _, plr in pairs(game.Players:GetPlayers()) do
			if plr ~= player and plr.Character then
				-- Highlight
				local highlight = Instance.new("Highlight")
				highlight.Adornee = plr.Character
				highlight.FillColor = Color3.fromRGB(255,0,0)
				highlight.FillTransparency = 0.7
				highlight.OutlineColor = Color3.fromRGB(255,0,0)
				highlight.OutlineTransparency = 0
				highlight.Parent = workspace
				highlights[plr] = highlight

				-- Nome em cima
				local billboard = Instance.new("BillboardGui")
				billboard.Name = "ESPName"
				billboard.Adornee = plr.Character:FindFirstChild("HumanoidRootPart")
				billboard.Size = UDim2.new(0,200,0,50)
				billboard.AlwaysOnTop = true
				billboard.Parent = plr.Character

				local textLabel = Instance.new("TextLabel")
				textLabel.Size = UDim2.new(1,0,1,0)
				textLabel.BackgroundTransparency = 1
				textLabel.Text = plr.Name
				textLabel.TextColor3 = Color3.fromRGB(255,0,0)
				textLabel.Font = Enum.Font.FredokaOne
				textLabel.TextScaled = true
				textLabel.Parent = billboard

				nameTags[plr] = billboard
			end
		end
	else
		for plr, highlight in pairs(highlights) do
			if highlight then highlight:Destroy() end
		end
		for plr, billboard in pairs(nameTags) do
			if billboard then billboard:Destroy() end
		end
		highlights = {}
		nameTags = {}
	end
end)

-- Atualiza ESP para novos jogadores
game.Players.PlayerAdded:Connect(function(plr)
	plr.CharacterAdded:Connect(function(char)
		if espAtivo then
			local highlight = Instance.new("Highlight")
			highlight.Adornee = char
			highlight.FillColor = Color3.fromRGB(255,0,0)
			highlight.FillTransparency = 0.7
			highlight.OutlineColor = Color3.fromRGB(255,0,0)
			highlight.OutlineTransparency = 0
			highlight.Parent = workspace
			highlights[plr] = highlight

			local billboard = Instance.new("BillboardGui")
			billboard.Name = "ESPName"
			billboard.Adornee = char:WaitForChild("HumanoidRootPart")
			billboard.Size = UDim2.new(0,200,0,50)
			billboard.AlwaysOnTop = true
			billboard.Parent = char

			local textLabel = Instance.new("TextLabel")
			textLabel.Size = UDim2.new(1,0,1,0)
			textLabel.BackgroundTransparency = 1
			textLabel.Text = plr.Name
			textLabel.TextColor3 = Color3.fromRGB(255,0,0)
			textLabel.Font = Enum.Font.FredokaOne
			textLabel.TextScaled = true
			textLabel.Parent = billboard

			nameTags[plr] = billboard
		end
	end)
end)
