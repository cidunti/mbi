local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- Remove versão anterior
local old = PlayerGui:FindFirstChild("BlockInspector")
if old then
    old:Destroy()
end

local Gui = Instance.new("ScreenGui")
Gui.Name = "BlockInspector"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true
Gui.Parent = PlayerGui

-- Janela
local Main = Instance.new("Frame")
Main.Size = UDim2.fromOffset(430, 330)
Main.Position = UDim2.new(0.5, -215, 0.5, -165)
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Main.Parent = Gui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 8)
Corner.Parent = Main

-- Título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundColor3 = Color3.fromRGB(28, 28, 28)
Title.BorderSizePixel = 0
Title.Text = "Block Inspector"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.TextSize = 17
Title.Font = Enum.Font.GothamBold
Title.Parent = Main

-- Botão de inspeção
local InspectButton = Instance.new("TextButton")
InspectButton.Size = UDim2.fromOffset(180, 36)
InspectButton.Position = UDim2.new(0.5, -90, 0, 52)
InspectButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
InspectButton.BorderSizePixel = 0
InspectButton.Text = "ATIVAR INSPEÇÃO"
InspectButton.TextColor3 = Color3.new(1, 1, 1)
InspectButton.TextSize = 13
InspectButton.Font = Enum.Font.GothamBold
InspectButton.Parent = Main

local InspectCorner = Instance.new("UICorner")
InspectCorner.CornerRadius = UDim.new(0, 6)
InspectCorner.Parent = InspectButton

-- Área de informações
local Info = Instance.new("TextLabel")
Info.Position = UDim2.fromOffset(12, 100)
Info.Size = UDim2.new(1, -24, 1, -112)
Info.BackgroundColor3 = Color3.fromRGB(24, 24, 24)
Info.BorderSizePixel = 0
Info.TextColor3 = Color3.fromRGB(235, 235, 235)
Info.TextSize = 13
Info.Font = Enum.Font.Code
Info.TextXAlignment = Enum.TextXAlignment.Left
Info.TextYAlignment = Enum.TextYAlignment.Top
Info.TextWrapped = true
Info.Text = "Ative a inspeção e clique no botão que deseja identificar."
Info.Parent = Main

local InfoCorner = Instance.new("UICorner")
InfoCorner.CornerRadius = UDim.new(0, 6)
InfoCorner.Parent = Info

local Inspecting = false

local function getValue(value)
    if typeof(value) == "Vector2" then
        return string.format("(%.1f, %.1f)", value.X, value.Y)
    elseif typeof(value) == "UDim2" then
        return string.format(
            "X{%d, %.2f} Y{%d, %.2f}",
            value.X.Scale,
            value.X.Offset,
            value.Y.Scale,
            value.Y.Offset
        )
    elseif typeof(value) == "Color3" then
        return string.format(
            "(%.0f, %.0f, %.0f)",
            value.R * 255,
            value.G * 255,
            value.B * 255
        )
    else
        return tostring(value)
    end
end

local function inspectObject(obj)
    local lines = {}

    table.insert(lines, "===== OBJETO =====")
    table.insert(lines, "Nome: " .. obj.Name)
    table.insert(lines, "Classe: " .. obj.ClassName)
    table.insert(lines, "")
    table.insert(lines, "===== CAMINHO =====")
    table.insert(lines, obj:GetFullName())
    table.insert(lines, "")

    table.insert(lines, "===== TELA =====")
    table.insert(lines, "Posição: " .. getValue(obj.AbsolutePosition))
    table.insert(lines, "Tamanho: " .. getValue(obj.AbsoluteSize))
    table.insert(lines, "Visível: " .. tostring(obj.Visible))
    table.insert(lines, "")

    if obj:IsA("GuiButton") then
        table.insert(lines, "===== BOTÃO =====")
        table.insert(lines, "Active: " .. tostring(obj.Active))
        table.insert(lines, "AutoButtonColor: " .. tostring(obj.AutoButtonColor))
        table.insert(lines, "Selectable: " .. tostring(obj.Selectable))
        table.insert(lines, "")
    end

    if obj:IsA("TextButton") then
        table.insert(lines, "===== TEXTO =====")
        table.insert(lines, "Text: " .. tostring(obj.Text))
        table.insert(lines, "TextSize: " .. tostring(obj.TextSize))
        table.insert(lines, "")
    end

    if obj:IsA("ImageButton") or obj:IsA("ImageLabel") then
        table.insert(lines, "===== IMAGEM =====")
        table.insert(lines, "Image: " .. tostring(obj.Image))
        table.insert(lines, "ImageRectOffset: " .. getValue(obj.ImageRectOffset))
        table.insert(lines, "ImageRectSize: " .. getValue(obj.ImageRectSize))
        table.insert(lines, "")
    end

    if obj:IsA("GuiObject") then
        table.insert(lines, "===== HIERARQUIA =====")
        table.insert(lines, "Pai: " .. (obj.Parent and obj.Parent:GetFullName() or "nil"))
        table.insert(lines, "ZIndex: " .. tostring(obj.ZIndex))
    end

    local result = table.concat(lines, "\n")

    -- Mostra na janela
    Info.Text = result

    -- Copia o caminho, se disponível
    if setclipboard then
        setclipboard(obj:GetFullName())
    end

    print("===== BLOCK INSPECTOR =====")
    print(result)
end

InspectButton.MouseButton1Click:Connect(function()
    Inspecting = not Inspecting

    if Inspecting then
        InspectButton.Text = "CLIQUE NO BOTÃO"
        InspectButton.BackgroundColor3 = Color3.fromRGB(70, 110, 70)
        Info.Text = "Modo inspeção ativado.\n\nClique diretamente no botão Block que você quer identificar."
    else
        InspectButton.Text = "ATIVAR INSPEÇÃO"
        InspectButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
        Info.Text = "Inspeção desativada."
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if not Inspecting then
        return
    end

    if input.UserInputType ~= Enum.UserInputType.MouseButton1
        and input.UserInputType ~= Enum.UserInputType.Touch then
        return
    end

    -- Não inspeciona o próprio menu
    local position

    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        position = UserInputService:GetMouseLocation()
    else
        position = input.Position
    end

    local objects = PlayerGui:GetGuiObjectsAtPosition(position.X, position.Y)

    for _, obj in ipairs(objects) do
        if obj ~= Main
            and not obj:IsDescendantOf(Main)
            and obj.Visible
            and obj:IsA("GuiObject") then

            inspectObject(obj)

            -- Desliga automaticamente depois da seleção
            Inspecting = false
            InspectButton.Text = "ATIVAR INSPEÇÃO"
            InspectButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)

            break
        end
    end
end)
