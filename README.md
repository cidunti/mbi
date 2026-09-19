local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local GuiService = game:GetService("GuiService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Remove versão anterior
local old = PlayerGui:FindFirstChild("BlockInspector")
if old then
    old:Destroy()
end

--------------------------------------------------
-- GUI
--------------------------------------------------

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BlockInspector"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.DisplayOrder = 999999
ScreenGui.Parent = PlayerGui

local Main = Instance.new("Frame")
Main.Name = "Main"
Main.Size = UDim2.fromOffset(470, 430)
Main.Position = UDim2.new(0.5, -235, 0.5, -215)
Main.BackgroundColor3 = Color3.fromRGB(17, 17, 17)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Main.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 8)
Corner.Parent = Main

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 38)
Title.BackgroundColor3 = Color3.fromRGB(27, 27, 27)
Title.BorderSizePixel = 0
Title.Text = "UI INSPECTOR"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.Parent = Main

local Toggle = Instance.new("TextButton")
Toggle.Size = UDim2.fromOffset(190, 36)
Toggle.Position = UDim2.fromOffset(140, 48)
Toggle.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
Toggle.BorderSizePixel = 0
Toggle.Text = "INSPEÇÃO: OFF"
Toggle.TextColor3 = Color3.new(1, 1, 1)
Toggle.Font = Enum.Font.GothamBold
Toggle.TextSize = 13
Toggle.Parent = Main

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 6)
ToggleCorner.Parent = Toggle

local Info = Instance.new("ScrollingFrame")
Info.Name = "Info"
Info.Position = UDim2.fromOffset(10, 94)
Info.Size = UDim2.new(1, -20, 1, -104)
Info.BackgroundColor3 = Color3.fromRGB(23, 23, 23)
Info.BorderSizePixel = 0
Info.ScrollBarThickness = 6
Info.CanvasSize = UDim2.new(0, 0, 0, 0)
Info.Parent = Main

local InfoCorner = Instance.new("UICorner")
InfoCorner.CornerRadius = UDim.new(0, 6)
InfoCorner.Parent = Info

local Text = Instance.new("TextLabel")
Text.Size = UDim2.new(1, -15, 0, 500)
Text.Position = UDim2.fromOffset(7, 5)
Text.BackgroundTransparency = 1
Text.TextColor3 = Color3.fromRGB(235, 235, 235)
Text.Font = Enum.Font.Code
Text.TextSize = 13
Text.TextXAlignment = Enum.TextXAlignment.Left
Text.TextYAlignment = Enum.TextYAlignment.Top
Text.TextWrapped = false
Text.Text = "Clique em INSPEÇÃO e depois clique diretamente no botão que deseja analisar."
Text.Parent = Info

--------------------------------------------------
-- FUNÇÕES
--------------------------------------------------

local Inspecting = false

local function isInsideInspector(x, y)
    local p = Main.AbsolutePosition
    local s = Main.AbsoluteSize

    return x >= p.X
        and x <= p.X + s.X
        and y >= p.Y
        and y <= p.Y + s.Y
end

local function safeClipboard(value)
    if typeof(setclipboard) == "function" then
        pcall(function()
            setclipboard(value)
        end)
    end
end

local function getImageId(obj)
    if obj:IsA("ImageButton") or obj:IsA("ImageLabel") then
        local image = obj.Image

        if image and image ~= "" then
            local id = string.match(image, "%d+")

            if id then
                return id
            end

            return image
        end
    end

    return nil
end

local function buildInfo(objects, clickX, clickY)

    local output = {}

    table.insert(output, "========== CLIQUE ==========")
    table.insert(output, "X: " .. math.floor(clickX))
    table.insert(output, "Y: " .. math.floor(clickY))
    table.insert(output, "")
    table.insert(output, "Objetos encontrados: " .. tostring(#objects))
    table.insert(output, "")

    for index, obj in ipairs(objects) do

        if obj
            and obj:IsA("GuiObject")
            and obj:IsDescendantOf(PlayerGui)
            and not obj:IsDescendantOf(ScreenGui) then

            table.insert(output, "============================")
            table.insert(output, "OBJETO #" .. tostring(index))
            table.insert(output, "============================")

            table.insert(output, "Nome: " .. obj.Name)
            table.insert(output, "Classe: " .. obj.ClassName)
            table.insert(output, "Caminho:")
            table.insert(output, obj:GetFullName())
            table.insert(output, "")

            table.insert(output, "Posição:")
            table.insert(
                output,
                "X=" .. math.floor(obj.AbsolutePosition.X)
                .. " | Y=" .. math.floor(obj.AbsolutePosition.Y)
            )

            table.insert(output, "Tamanho:")
            table.insert(
                output,
                "W=" .. math.floor(obj.AbsoluteSize.X)
                .. " | H=" .. math.floor(obj.AbsoluteSize.Y)
            )

            table.insert(output, "")
            table.insert(output, "Visible: " .. tostring(obj.Visible))
            table.insert(output, "ZIndex: " .. tostring(obj.ZIndex))

            if obj.Parent then
                table.insert(output, "")
                table.insert(output, "Pai:")
                table.insert(output, obj.Parent:GetFullName())
            end

            local imageId = getImageId(obj)

            if imageId then
                table.insert(output, "")
                table.insert(output, "IMAGE / ASSET ID:")
                table.insert(output, tostring(imageId))
            end

            if obj:IsA("TextButton") then
                table.insert(output, "")
                table.insert(output, "Texto:")
                table.insert(output, tostring(obj.Text))
            end

            if obj:IsA("GuiButton") then
                table.insert(output, "")
                table.insert(output, "É botão: SIM")
                table.insert(output, "Active: " .. tostring(obj.Active))
                table.insert(
                    output,
                    "AutoButtonColor: "
                    .. tostring(obj.AutoButtonColor)
                )
            end

            -- Hierarquia acima do objeto
            table.insert(output, "")
            table.insert(output, "HIERARQUIA:")

            local current = obj
            local level = 0

            while current and current ~= PlayerGui and level < 15 do

                table.insert(
                    output,
                    string.rep("  ", level)
                    .. "└─ "
                    .. current.Name
                    .. " ["
                    .. current.ClassName
                    .. "]"
                )

                current = current.Parent
                level += 1
            end

            table.insert(output, "")
        end
    end

    return table.concat(output, "\n")
end

--------------------------------------------------
-- PROCURA OBJETOS NO PONTO CLICADO
--------------------------------------------------

local function getObjectsAtPoint(x, y)

    local found = {}
    local already = {}

    local function addObjects(list)

        if not list then
            return
        end

        for _, obj in ipairs(list) do

            if obj
                and obj:IsA("GuiObject")
                and obj:IsDescendantOf(PlayerGui)
                and not obj:IsDescendantOf(ScreenGui)
                and obj.Visible then

                if not already[obj] then
                    already[obj] = true
                    table.insert(found, obj)
                end
            end
        end
    end

    -- Coordenada normal
    pcall(function()
        addObjects(
            PlayerGui:GetGuiObjectsAtPosition(x, y)
        )
    end)

    -- Alternativa pelo GuiService
    pcall(function()
        addObjects(
            GuiService:GetGuiObjectsAtPosition(x, y)
        )
    end)

    -- Tenta compensar o GuiInset
    local insetTopLeft = GuiService:GetGuiInset()

    pcall(function()
        addObjects(
            PlayerGui:GetGuiObjectsAtPosition(
                x - insetTopLeft.X,
                y - insetTopLeft.Y
            )
        )
    end)

    pcall(function()
        addObjects(
            PlayerGui:GetGuiObjectsAtPosition(
                x + insetTopLeft.X,
                y + insetTopLeft.Y
            )
        )
    end)

    -- Coloca objetos mais "profundos" primeiro
    table.sort(found, function(a, b)
        local depthA = 0
        local depthB = 0

        local ca = a
        local cb = b

        while ca and ca ~= PlayerGui do
            depthA += 1
            ca = ca.Parent
        end

        while cb and cb ~= PlayerGui do
            depthB += 1
            cb = cb.Parent
        end

        return depthA > depthB
    end)

    return found
end

--------------------------------------------------
-- BOTÃO DE INSPEÇÃO
--------------------------------------------------

Toggle.MouseButton1Click:Connect(function()

    Inspecting = not Inspecting

    if Inspecting then

        Toggle.Text = "INSPEÇÃO: ON"
        Toggle.BackgroundColor3 = Color3.fromRGB(55, 100, 55)

        Text.Text =
            "INSPEÇÃO ATIVADA\n\n"
            .. "Clique diretamente no botão/elemento que deseja analisar.\n\n"
            .. "O resultado aparecerá aqui."

    else

        Toggle.Text = "INSPEÇÃO: OFF"
        Toggle.BackgroundColor3 = Color3.fromRGB(45, 45, 45)

        Text.Text = "Inspeção desativada."
    end
end)

--------------------------------------------------
-- CLIQUE NA TELA
--------------------------------------------------

UserInputService.InputBegan:Connect(function(input)

    if not Inspecting then
        return
    end

    if input.UserInputType ~= Enum.UserInputType.MouseButton1 then
        return
    end

    local mousePosition = UserInputService:GetMouseLocation()

    local x = mousePosition.X
    local y = mousePosition.Y

    -- Não inspeciona o próprio menu
    if isInsideInspector(x, y) then
        return
    end

    local objects = getObjectsAtPoint(x, y)

    if #objects == 0 then

        Text.Text =
            "NENHUM OBJETO ENCONTRADO\n\n"
            .. "Coordenadas:\n"
            .. "X = " .. math.floor(x) .. "\n"
            .. "Y = " .. math.floor(y)

        return
    end

    local result = buildInfo(objects, x, y)

    Text.Text = result

    -- Copia o caminho do primeiro objeto encontrado
    local first = objects[1]

    if first then
        safeClipboard(first:GetFullName())
    end

    print(result)
end)

--------------------------------------------------
-- ATUALIZA SCROLL
--------------------------------------------------

Text:GetPropertyChangedSignal("TextBounds"):Connect(function()

    Info.CanvasSize = UDim2.fromOffset(
        0,
        Text.TextBounds.Y + 15
    )
end)

task.spawn(function()
    while ScreenGui.Parent do
        task.wait(0.2)

        Info.CanvasSize = UDim2.fromOffset(
            0,
            Text.TextBounds.Y + 15
        )
    end
end)
