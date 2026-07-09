-- Безопасное ожидание загрузки игры
if not game:IsLoaded() then
    game.Loaded:Wait()
end

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui", 10)
local Camera = workspace.CurrentCamera

-- Безопасное изменение FOV
if Camera then
    Camera.FieldOfView = 115
end

-- Функция растягивания с защитой от вылетов
local function safeStretch(guiObject)
    pcall(function()
        if guiObject:IsA("Frame") or guiObject:IsA("ImageLabel") or guiObject:IsA("TextLabel") then
            if not guiObject:GetAttribute("Stretched") then
                guiObject:SetAttribute("Stretched", true)
                local s = guiObject.Size
                guiObject.Size = UDim2.new(s.X.Scale * 1.3, s.X.Offset * 1.3, s.Y.Scale, s.Y.Offset)
            end
        end
    end)
end

-- Безопасный обход элементов с паузами (чтобы не крашило)
local function scanGui(gui)
    local descendants = gui:GetDescendants()
    for i, child in ipairs(descendants) do
        safeStretch(child)
        -- Каждые 50 элементов делаем микро-паузу, чтобы разгрузить процессор
        if i % 50 == 0 then 
            task.wait() 
        end
    end
    
    -- Вместо тяжелого DescendantAdded используем легкую задержку для новых элементов
    gui.ChildAdded:Connect(function(newChild)
        task.wait(0.2)
        safeStretch(newChild)
        for _, subChild in ipairs(newChild:GetDescendants()) do
            safeStretch(subChild)
        end
    end)
end

-- Запуск
if PlayerGui then
    for _, gui in ipairs(PlayerGui:GetChildren()) do
        if gui:IsA("ScreenGui") then
            task.spawn(scanGui, gui) -- Запуск в отдельном потоке
        end
    end

    PlayerGui.ChildAdded:Connect(function(child)
        if child:IsA("ScreenGui") then
            task.wait(0.5)
            task.spawn(scanGui, child)
        end
    end)
end

print("[Xeno] Безопасный режим растяга активирован.")
