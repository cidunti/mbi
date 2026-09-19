local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

for _, obj in ipairs(playerGui:GetDescendants()) do
    if obj:IsA("TextButton") or obj:IsA("ImageButton") then
        print(
            "Nome:", obj.Name,
            "| Posição:", obj.AbsolutePosition,
            "| Tamanho:", obj.AbsoluteSize
        )
    end
end
