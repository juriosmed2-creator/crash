-- CrashServer.lua
-- GUI de exemplo com geração, verificação e remoção de keys.
-- Comentários em português explicam cada parte.

-- Tabela em memória com keys válidas (poderia ser substituída por DataStore se quiser persistência)
local keys = {
	-- Algumas keys iniciais para teste (opcionais)
	"TEST-KEY-0001",
	"ABCDEF-123456-XYZ",
}

-- Função utilitária para gerar uma key aleatória
local function generateKey(length)
	local charset = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
	local result = {}
	for i = 1, length do
		local idx = math.random(1, #charset)
		result[i] = charset:sub(idx, idx)
	end
	-- formata com hífens a cada 4 caracteres para melhor leitura
	local s = table.concat(result)
	return s:sub(1,4) .. "-" .. s:sub(5,8) .. "-" .. s:sub(9,12) .. "-" .. s:sub(13,16)
end

-- Função que será chamada quando uma key for verificada com sucesso
local function onKeyVerified(key)
	-- Aqui você coloca as funções/features que a key deve desbloquear.
	-- Importante: remover uma key (Remove Key) não deve remover a lógica desta função.
	-- Coloque a sua lógica real aqui.
	warn("Key verificada:", key)
end

-- Criação da interface (mantive a aparência original com algumas correções)
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local Sub = Instance.new("TextLabel")
local KeyBox = Instance.new("TextBox")
local GetKey = Instance.new("TextButton")
local Verify = Instance.new("TextButton")
local RemoveKey = Instance.new("TextButton")
local StatusLabel = Instance.new("TextLabel")
local UICornerFrame = Instance.new("UICorner")
local UICornerButton1 = Instance.new("UICorner")
local UICornerButton2 = Instance.new("UICorner")
local UICornerButton3 = Instance.new("UICorner")
local UICornerBox = Instance.new("UICorner")

-- Parent do ScreenGui (evita problemas caso CoreGui esteja restrito)
-- Se estiver num LocalScript normal dentro do PlayerGui, troque ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.Name = "CrashServerGUI"
ScreenGui.Parent = game.CoreGui -- note: dependendo do ambiente, isso pode ser restrito; use PlayerGui se preferir

-- Frame principal
Frame.Parent = ScreenGui
Frame.BackgroundColor3 = Color3.fromRGB(150, 200, 255)
Frame.BorderSizePixel = 0
Frame.Size = UDim2.new(0, 320, 0, 240)
Frame.Position = UDim2.new(0.5, -160, 0.5, -120)
UICornerFrame.Parent = Frame
UICornerFrame.CornerRadius = UDim.new(0, 16)

-- Title
Title.Parent = Frame
Title.Text = "Crash Server"
Title.TextColor3 = Color3.new(0,0,0)
Title.BackgroundTransparency = 1
Title.Size = UDim2.new(1, 0, 0, 44)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 28
Title.Position = UDim2.new(0, 0, 0, 4)

-- Subtitle
Sub.Parent = Frame
Sub.Text = "Key custa 100 Robux"
Sub.TextColor3 = Color3.new(0,0,0)
Sub.BackgroundTransparency = 1
Sub.Size = UDim2.new(1, 0, 0, 20)
Sub.Position = UDim2.new(0, 0, 0, 44)
Sub.Font = Enum.Font.SourceSans
Sub.TextSize = 16

-- KeyBox
KeyBox.Parent = Frame
KeyBox.PlaceholderText = "Enter Key"
KeyBox.BackgroundColor3 = Color3.fromRGB(230, 240, 255)
KeyBox.TextColor3 = Color3.new(0,0,0)
KeyBox.Size = UDim2.new(1, -20, 0, 34)
KeyBox.Position = UDim2.new(0, 10, 0, 76)
KeyBox.ClearTextOnFocus = false
UICornerBox.Parent = KeyBox
UICornerBox.CornerRadius = UDim.new(0, 10)

-- GetKey button
GetKey.Parent = Frame
GetKey.Text = "Get Key"
GetKey.BackgroundColor3 = Color3.fromRGB(200, 220, 255)
GetKey.TextColor3 = Color3.new(0,0,0)
GetKey.Size = UDim2.new(0.45, -10, 0, 36)
GetKey.Position = UDim2.new(0, 10, 0, 124)
UICornerButton1.Parent = GetKey
UICornerButton1.CornerRadius = UDim.new(0, 10)

-- Verify button
Verify.Parent = Frame
Verify.Text = "Verify Key"
Verify.BackgroundColor3 = Color3.fromRGB(200, 220, 255)
Verify.TextColor3 = Color3.new(0,0,0)
Verify.Size = UDim2.new(0.45, -10, 0, 36)
Verify.Position = UDim2.new(0.55, 0, 0, 124)
UICornerButton2.Parent = Verify
UICornerButton2.CornerRadius = UDim.new(0, 10)

-- RemoveKey button (remove a key da tabela sem tocar nas funções)
RemoveKey.Parent = Frame
RemoveKey.Text = "Remove Key"
RemoveKey.BackgroundColor3 = Color3.fromRGB(240, 200, 200)
RemoveKey.TextColor3 = Color3.new(0,0,0)
RemoveKey.Size = UDim2.new(1, -20, 0, 34)
RemoveKey.Position = UDim2.new(0, 10, 0, 170)
UICornerButton3.Parent = RemoveKey
UICornerButton3.CornerRadius = UDim.new(0, 10)

-- StatusLabel para feedback ao usuário
StatusLabel.Parent = Frame
StatusLabel.Text = ""
StatusLabel.TextColor3 = Color3.new(0,0,0)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Size = UDim2.new(1, 0, 0, 20)
StatusLabel.Position = UDim2.new(0, 0, 0, 210)
StatusLabel.Font = Enum.Font.SourceSans
StatusLabel.TextSize = 14

-- Função para mostrar status temporário
local function setStatus(text, seconds)
	StatusLabel.Text = text
	if seconds and seconds > 0 then
		delay(seconds, function()
			-- limpa só se não tiver sido sobrescrito
			if StatusLabel.Text == text then
				StatusLabel.Text = ""
			end
		end)
	end
end

-- Botão GetKey: gera uma key e coloca na caixa de texto (e tenta copiar para clipboard)
GetKey.MouseButton1Click:Connect(function()
	local newKey = generateKey(16)
	table.insert(keys, newKey)
	KeyBox.Text = newKey
	-- tenta copiar para clipboard (nem sempre disponível)
	local ok, err = pcall(function()
		if setclipboard then
			setclipboard(newKey)
		end
	end)
	if ok then
		setStatus("Key gerada e copiada para o clipboard.", 4)
	else
		setStatus("Key gerada. (clipboard não disponível)", 4)
	end
end)

-- Botão Verify: verifica se a key está na tabela
Verify.MouseButton1Click:Connect(function()
	local input = KeyBox.Text and KeyBox.Text:upper() or ""
	if input == "" then
		setStatus("Digite uma key para verificar.", 3)
		return
	end
	local found = false
	for _, k in ipairs(keys) do
		if k == input then
			found = true
			break
		end
	end
	if found then
		setStatus("Key válida! Desbloqueando funções...", 4)
		-- chama a função que ativa as features
		onKeyVerified(input)
	else
		setStatus("Key inválida.", 3)
	end
end)

-- Botão RemoveKey: remove a key da tabela sem alterar outras funções
RemoveKey.MouseButton1Click:Connect(function()
	local input = KeyBox.Text and KeyBox.Text:upper() or ""
	if input == "" then
		setStatus("Digite a key a remover.", 3)
		return
	end
	local removed = false
	for i = #keys, 1, -1 do
		if keys[i] == input then
			table.remove(keys, i)
			removed = true
		end
	end
	if removed then
		setStatus("Key removida da lista. (Não removi nenhuma função)", 4)
	else
		setStatus("Key não encontrada na lista.", 3)
	end
end)

-- Segurança simples: normaliza keys na tabela para uppercase
for i, k in ipairs(keys) do
	keys[i] = tostring(k):upper()
end

-- Fim do script
