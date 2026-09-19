local Players = game:GetService("Players")
local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

-- GUI principal
local gui = Instance.new("ScreenGui")
gui.Name = "UIInspector"
gui.ResetOnSpawn = false
gui.Parent = PlayerGui

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(420, 500)
main.Position = UDim2.new(0.5, -210, 0.5, -250)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true
main.Parent = gui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
title.Text = "UI Inspector - Elementos na tela"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 16
title.Parent = main

local scroll = Instance.new("ScrollingFrame")
scroll.Position = UDim2.fromOffset(5, 45)
scroll.Size = UDim2.new(1, -10, 1, -50)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel = 0
scroll.ScrollBarThickness = 6
scroll.Parent = main

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 4)
layout.Parent = scroll

local function addItem(obj)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, -5, 0, 55)
    button.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    button.BorderSizePixel = 0
    button.TextColor3 = Color3.new(1, 1, 1)
    button.TextSize = 12
    button.TextWrapped = true
    button.TextXAlignment = Enum.TextXAlignment.Left

    local pos = obj.AbsolutePosition
    local size = obj.AbsoluteSize

    button.Text =
        "Nome: " .. obj.Name ..
        "\nTipo: " .. obj.ClassName ..
        "\nID/Caminho: " .. obj:GetFullName() ..
        "\nPos: " .. math.floor(pos.X) .. ", " .. math.floor(pos.Y) ..
        " | Tam: " .. math.floor(size.X) .. "x" .. math.floor(size.Y)

    button.Parent = scroll

    -- Copia o caminho para o clipboard quando disponível
    button.MouseButton1Click:Connect(function()
        if setclipboard then
            setclipboard(obj:GetFullName())
        end

        print("Selecionado:", obj:GetFullName())
    end)
end

-- Procura elementos GUI visíveis
for _, obj in ipairs(PlayerGui:GetDescendants()) do
    if obj:IsA("GuiObject")
        and obj.Visible
        and obj.AbsoluteSize.X > 0
        and obj.AbsoluteSize.Y > 0 then

        addItem(obj)
    end
end

-- Atualiza o CanvasSize
layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    scroll.CanvasSize = UDim2.fromOffset(
        0,
        layout.AbsoluteContentSize.Y + 10
    )
end)

scroll.CanvasSize = UDim2.fromOffset(
    0,
    layout.AbsoluteContentSize.Y + 10
)
