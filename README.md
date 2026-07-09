-- [[ Скрипт на псевдо-растягивание экрана и интерфейса ]] --

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = workspace.CurrentCamera

-- 1. Изменяем FOV (Поле зрения), чтобы раздвинуть камеру по бокам
Camera.FieldOfView = 110 -- Стандартно 70, больше значение = шире обзор

-- 2. Функция, которая насильно растягивает элементы интерфейса (UI) по горизонтали
local function stretchUI(guiObject)
    if guiObject:IsA("Frame") or guiObject:IsA("ImageLabel") or guiObject:IsA("TextLabel") then
        -- Берём текущий размер
        local currentSize = guiObject.Size
        -- Умножаем ширину на 1.3 (растягиваем по оси X)
        guiObject.Size = UDim2.new(
            currentSize.X.Scale * 1.3, 
            currentSize.X.Offset * 1.3, 
            currentSize.Y.Scale, 
            currentSize.Y.Offset
        )
    end
end

-- Применяем растяг ко всему интерфейсу, который уже есть и который появится
local function scanGui(gui)
    for _, child in ipairs(gui:GetDescendants()) do
        stretchUI(child)
    end
    gui.DescendantAdded:Connect(stretchUI)
end

-- Запуск сканирования UI
for _, gui in ipairs(PlayerGui:GetChildren()) do
    if gui:IsA("ScreenGui") then
        scanGui(gui)
    end
end

PlayerGui.ChildAdded:Connect(function(child)
    if child:IsA("ScreenGui") then
        scanGui(child)
    end
end)

print("Lua-растяг (FOV + UI) успешно активирован!")
