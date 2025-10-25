--// Services
local Players = game:GetService('Players')
local RunService = game:GetService('RunService')
local Workspace = game:GetService('Workspace')
local UserInputService = game:GetService('UserInputService')
local ReplicatedStorage = game:GetService('ReplicatedStorage')
local Lighting = game:GetService('Lighting')
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- Función alternativa para mousemoverel
local mousemoverel = mousemoverel or function(x, y)
    -- Alternativa para mover el mouse usando las funciones de Roblox
    local VirtualInputManager = game:GetService("VirtualInputManager")
    local currentX = UserInputService:GetMouseLocation().X
    local currentY = UserInputService:GetMouseLocation().Y
    
    VirtualInputManager:SendMouseMoveEvent(currentX + x, currentY + y)
end

--// Variables
local modifiedWeapons = {}
local weaponChamsConnection = nil
local weaponChamsEnabled = false
local weaponChamsColor = Color3.fromRGB(255, 255, 255)
local weaponChamsTransparency = 0.5

--// Variables para Body Chams automático
local bodyChamsEnabled = false
local bodyChamsConnection = nil
local bodyChamsParts = {}
local bodyChamsColor = Color3.fromRGB(255, 255, 255)
local bodyChamsTransparency = 0

--// Variables para Item ESP
-- Vehicle ESP
local vehicleESP = {
    Enabled = false,
    ShowDistance = false,
    Outline = false,
    Size = 14,
    Color = Color3.new(1, 1, 1),
    ActiveDrawings = {}
}

-- Grave ESP
local graveESP = {
    Enabled = false,
    ShowDistance = false,
    Outline = false,
    Size = 14,
    Color = Color3.fromRGB(255, 255, 255),
    ActiveDrawings = {}
}

-- Ammo ESP
local ammoESP = {
    Enabled = false,
    Outline = true,
    Size = 14,
    Color = Color3.fromRGB(255, 255, 255),
    ShowDistance = false,
    RenderDistance = 1000,
    ActiveDrawings = {}
}

-- Weapons ESP
local weaponsESP = {
    Enabled = false,
    Size = 14,
    Outline = false,
    Color = Color3.fromRGB(255, 255, 255),
    ExcludeMelee = false,
    ShowDistance = false,
    RenderRange = 500,
    ActiveDrawings = {}
}

-- Sistema de Skybox Changer
local skyboxes = {
    ["Standard"] = { "91458024", "91457980", "91458024", "91458024", "91458024", "91458002"},
    ["Blue Sky"] = { "591058823", "591059876", "591058104", "591057861", "591057625", "591059642" },
    ["Vaporwave"] = { "1417494030", "1417494146", "1417494253", "1417494402", "1417494499", "1417494643" },
    ["Redshift"] = { "401664839", "401664862", "401664960", "401664881", "401664901", "401664936" },
    ["Blaze"] = { "150939022", "150939038", "150939047", "150939056", "150939063", "150939082" },
    ["Among Us"] = { "5752463190", "5752463190", "5752463190", "5752463190", "5752463190", "5752463190" },
    ["Dark Night"] = { "6285719338", "6285721078", "6285722964", "6285724682", "6285726335", "6285730635" },
    ["Bright Pink"] = { "271042516", "271077243", "271042556", "271042310", "271042467", "271077958" },
    ["Purple Sky"] = { "570557514", "570557775", "570557559", "570557620", "570557672", "570557727" },
    ["Galaxy"] = { "15125283003", "15125281008", "15125277539", "15125279325", "15125274388", "15125275800" },
}

-- Sistema de sonidos de impacto
local hitsounds = {
    ["Default"] = "rbxassetid://5097463838",
    ["Gamesense"] = "rbxassetid://4817809188",
    ["CS:GO"] = "rbxassetid://6937353691",
    ["Neverlose"] = "rbxassetid://8726881116",
    ["TF2 Critical"] = "rbxassetid://296102734",
    ["Rust"] = "rbxassetid://1255040462",
    ["Call of Duty"] = "rbxassetid://5952120301",
    ["Minecraft"] = "rbxassetid://4018616850",
    ["Dink"]      = "rbxassetid://988593556",
    ["TF2"]       = "rbxassetid://8255306220",
    ["Bubble"]    = "rbxassetid://198598793",
    ["Quake"]     = "rbxassetid://1455817260",
    ["Among-Us"]  = "rbxassetid://7227567562",
    ["Ding"]      = "rbxassetid://2868331684",
    ["Minecraft_2"] = "rbxassetid://6361963422",
    ["Blackout"]  = "rbxassetid://3748776946",
    ["Osu"]       = "rbxassetid://7151989073"
};

local headsounds = {
    ["Default"] = "rbxassetid://5097463838",
    ["Gamesense"] = "rbxassetid://4817809188",
    ["CS:GO"] = "rbxassetid://6937353691",
    ["Neverlose"] = "rbxassetid://8726881116",
    ["TF2 Critical"] = "rbxassetid://296102734",
    ["Rust"] = "rbxassetid://1255040462",
    ["Call of Duty"] = "rbxassetid://5952120301",
    ["Minecraft"] = "rbxassetid://4018616850",
    ["Dink"]      = "rbxassetid://988593556",
    ["TF2"]       = "rbxassetid://8255306220",
    ["Bubble"]    = "rbxassetid://198598793",
    ["Quake"]     = "rbxassetid://1455817260",
    ["Among-Us"]  = "rbxassetid://7227567562",
    ["Ding"]      = "rbxassetid://2868331684",
    ["Minecraft_2"] = "rbxassetid://6361963422",
    ["Blackout"]  = "rbxassetid://3748776946",
    ["Osu"]       = "rbxassetid://7151989073"
};

-- Sistema de sonidos y efectos visuales
local effects_system = {
    settings = {
        hitSound = {
            enabled = false,
            soundId = hitsounds["Default"],
            volume = 2,
            pitch = 1,
            soundName = "Default"
        },
        headshotSound = {
            enabled = false,
            soundId = headsounds["Default"],
            volume = 2,
            pitch = 1,
            soundName = "Default"
        },
        hitMarker = {
            enabled = false,
            duration = 0.5
        }
    },
    sounds = {}
}

--// Variables para el sistema de hitsounds
local hitsoundEnabled = false
local headshotSoundEnabled = false
local currentHitsound = "rbxassetid://8726881116" -- Default Neverlose
local currentHeadshotSound = "rbxassetid://988593556" -- Default headshot

local hitsoundNames = {
    "Neverlose", "TF2", "Gamesense", "Rust", "Bubble", 
    "Quake", "Among-Us", "Ding", "Minecraft", "Blackout", "Osu"
}

local headshotSoundNames = {
    "Default", "TF2", "Gamesense", "Rust", "Bubble", 
    "Quake", "Among-Us", "Ding", "Minecraft", "Blackout", "Osu"
}

local hitsoundOptions = {
    ["Neverlose"] = "rbxassetid://8726881116",
    ["TF2"] = "rbxassetid://8255306220",
    ["Gamesense"] = "rbxassetid://4817809188",
    ["Rust"] = "rbxassetid://1255040462",
    ["Bubble"] = "rbxassetid://198598793",
    ["Quake"] = "rbxassetid://1455817260",
    ["Among-Us"] = "rbxassetid://7227567562",
    ["Ding"] = "rbxassetid://2868331684",
    ["Minecraft"] = "rbxassetid://6361963422",
    ["Blackout"] = "rbxassetid://3748776946",
    ["Osu"] = "rbxassetid://7151989073"
}

local headshotSoundOptions = {
    ["Default"] = "rbxassetid://988593556",
    ["TF2"] = "rbxassetid://8255306220",
    ["Gamesense"] = "rbxassetid://4817809188",
    ["Rust"] = "rbxassetid://1255040462",
    ["Bubble"] = "rbxassetid://198598793",
    ["Quake"] = "rbxassetid://1455817260",
    ["Among-Us"] = "rbxassetid://7227567562",
    ["Ding"] = "rbxassetid://2868331684",
    ["Minecraft"] = "rbxassetid://6361963422",
    ["Blackout"] = "rbxassetid://3748776946",
    ["Osu"] = "rbxassetid://7151989073"
}

function effects_system.playHitSound()
    if not effects_system.settings.hitSound.enabled then return end
    
    local sound = Instance.new("Sound")
    sound.SoundId = effects_system.settings.hitSound.soundId
    sound.Volume = effects_system.settings.hitSound.volume
    sound.Pitch = effects_system.settings.hitSound.pitch
    sound.Parent = game:GetService("SoundService")
    sound:Play()
    
    game:GetService("Debris"):AddItem(sound, 1)
end

function effects_system.playHeadshotSound()
    if not effects_system.settings.headshotSound.enabled then return end
    
    local sound = Instance.new("Sound")
    sound.SoundId = effects_system.settings.headshotSound.soundId
    sound.Volume = effects_system.settings.headshotSound.volume
    sound.Pitch = effects_system.settings.headshotSound.pitch
    sound.Parent = game:GetService("SoundService")
    sound:Play()
    
    game:GetService("Debris"):AddItem(sound, 1)
end

function effects_system.showHitMarker(position)
    if not effects_system.settings.hitMarker.enabled then return end
    
    local hitMarker = Instance.new("Frame")
    hitMarker.Size = UDim2.new(0, 10, 0, 10)
    hitMarker.Position = UDim2.new(0.5, -5, 0.5, -5)
    hitMarker.BackgroundColor3 = Color3.new(1, 1, 1)
    hitMarker.BorderSizePixel = 0
    hitMarker.Parent = game:GetService("Players").LocalPlayer.PlayerGui
    
    local corner1 = Instance.new("Frame", hitMarker)
    corner1.Size = UDim2.new(0, 20, 0, 2)
    corner1.Position = UDim2.new(0, -5, 0, 4)
    corner1.BackgroundColor3 = Color3.new(1, 0, 0)
    corner1.BorderSizePixel = 0
    
    local corner2 = Instance.new("Frame", hitMarker)
    corner2.Size = UDim2.new(0, 2, 0, 20)
    corner2.Position = UDim2.new(0, 4, 0, -5)
    corner2.BackgroundColor3 = Color3.new(1, 0, 0)
    corner2.BorderSizePixel = 0
    
    game:GetService("Debris"):AddItem(hitMarker, effects_system.settings.hitMarker.duration)
end

--// Funciones para aplicar hitsounds - CORREGIDAS
local function applyHitsound()
    if not hitsoundEnabled then return end
    
    -- Verificación segura de la jerarquía de objetos
    local hitmarker = game:GetService('ReplicatedStorage'):FindFirstChild('Assets')
    if hitmarker then
        local sounds = hitmarker:FindFirstChild('Sounds')
        if sounds then
            local defaultHitmarker = sounds:FindFirstChild('DefaultHitmarker')
            if defaultHitmarker then
                local hitmarkerSound = defaultHitmarker:FindFirstChild('Hitmarker')
                if hitmarkerSound then
                    hitmarkerSound.SoundId = currentHitsound
                end
            end
        end
    end
end

local function applyHeadshotSound()
    if not headshotSoundEnabled then return end
    
    -- Verificación segura de la jerarquía de objetos
    local headshot = game:GetService('ReplicatedStorage'):FindFirstChild('Assets')
    if headshot then
        local sounds = headshot:FindFirstChild('Sounds')
        if sounds then
            local defaultHitmarker = sounds:FindFirstChild('DefaultHitmarker')
            if defaultHitmarker then
                local headshotSound = defaultHitmarker:FindFirstChild('Headshot')
                if headshotSound then
                    headshotSound.SoundId = currentHeadshotSound
                end
            end
        end
    end
end

--// Bucle para mantener los hitsounds aplicados
task.spawn(function()
    while true do
        if hitsoundEnabled then applyHitsound() end
        if headshotSoundEnabled then applyHeadshotSound() end
        task.wait(1) 
    end
end)

--// Función para aplicar chams a un arma
function applyWeaponChams(weapon)
    if not weapon or not weaponChamsEnabled then
        return
    end

    -- Evitar modificar el mismo arma múltiples veces
    if modifiedWeapons[weapon] then
        return
    end
    modifiedWeapons[weapon] = true

    -- Buscar las partes del arma
    local parts = weapon:FindFirstChild('Weapon') or weapon
    if not parts then
        return
    end

    -- Aplicar chams a todas las partes visibles
    for _, part in pairs(parts:GetDescendants()) do
        if part:IsA('BasePart') and part.Transparency < 1 then
            -- Guardar propiedades originales
            if not part:FindFirstChild('OriginalProperties') then
                local origValues = Instance.new('Configuration')
                origValues.Name = 'OriginalProperties'

                local origColor = Instance.new('Color3Value')
                origColor.Name = 'Color'
                origColor.Value = part.Color
                origColor.Parent = origValues

                -- Cambio: guardar el valor numérico del material (IntValue en lugar de StringValue)
                local origMaterial = Instance.new('IntValue')
                origMaterial.Name = 'Material'
                origMaterial.Value = part.Material.Value -- Guardar el índice numérico
                origMaterial.Parent = origValues

                local origTransparency = Instance.new('NumberValue')
                origTransparency.Name = 'Transparency'
                origTransparency.Value = part.Transparency
                origTransparency.Parent = origValues

                origValues.Parent = part
            end

            -- Aplicar nuevas propiedades
            part.Material = Enum.Material.ForceField
            part.Color = weaponChamsColor
            part.Transparency = weaponChamsTransparency

            -- Eliminar SurfaceAppearance para que el material se vea correctamente
            if part:FindFirstChild('SurfaceAppearance') then
                part.SurfaceAppearance:Destroy()
            end
        end
    end
end

--// Función para restaurar el aspecto original de un arma
function restoreWeapon(weapon)
    if not weapon or not modifiedWeapons[weapon] then
        return
    end
    modifiedWeapons[weapon] = nil

    for _, part in pairs(weapon:GetDescendants()) do
        if part:IsA('BasePart') then
            local origProps = part:FindFirstChild('OriginalProperties')
            if origProps then
                -- Restaurar propiedades originales
                part.Color = origProps.Color.Value

                -- Cambio: restaurar el material usando el índice guardado
                local materialValue = origProps.Material.Value
                -- Verificar que el índice sea válido
                if materialValue and materialValue >= 0 and materialValue < #Enum.Material:GetEnumItems() then
                    part.Material = Enum.Material:GetEnumItem(materialValue)
                else
                    -- Si el índice no es válido, usar un material por defecto (Plastic)
                    part.Material = Enum.Material.Plastic
                end

                part.Transparency = origProps.Transparency.Value
                
                -- Eliminar el objeto de propiedades originales
                origProps:Destroy()
            end
        end
    end
end

LeftGroupBox:AddToggle('MyToggle', {
    Text = 'speed',
    Default = false, 
    Tooltip = 'This is a tooltip', 
    Callback = function(Value)
        
    end
})

--// Función para verificar y aplicar chams a todas las armas
function checkWeapons()
    -- Verificar el arma actual del jugador
    local character = LocalPlayer.Character
    if character then
        local tool = character:FindFirstChildOfClass('Tool')
        if tool then
            applyWeaponChams(tool)
        end
    end

    -- Verificar armas en la cámara (para juegos de primera persona)
    local currentWeapon = Workspace.Camera:FindFirstChild('CurrentWeapon')
    if currentWeapon then
        applyWeaponChams(currentWeapon)
    end
end

--// Función para alternar los chams (CORREGIDA)
function toggleWeaponChams()
    if weaponChamsEnabled then
        -- Conectar para verificar armas continuamente
        if not weaponChamsConnection then
            weaponChamsConnection = RunService.Heartbeat:Connect(checkWeapons)
        end
        -- Verificar armas inmediatamente
        checkWeapons()
    else
        -- Restaurar todas las armas modificadas
        for weapon, _ in pairs(modifiedWeapons) do
            restoreWeapon(weapon)
        end
        modifiedWeapons = {}
        
        -- Desconectar la conexión si existe
        if weaponChamsConnection then
            weaponChamsConnection:Disconnect()
            weaponChamsConnection = nil
        end
    end
end

--// Función para cambiar el color de los chams (CORREGIDA)
function setWeaponChamsColor(color)
    weaponChamsColor = color

    -- Volver a aplicar chams a todas las armas modificadas
    if weaponChamsEnabled then
        for weapon, _ in pairs(modifiedWeapons) do
            local parts = weapon:FindFirstChild('Weapon') or weapon
            if parts then
                for _, part in pairs(parts:GetDescendants()) do
                    if part:IsA('BasePart') and part:FindFirstChild('OriginalProperties') then
                        part.Color = weaponChamsColor
                    end
                end
            end
        end
    end
end

--// Función para cambiar la transparencia de los chams (CORREGIDA)
function setWeaponChamsTransparency(transparency)
    weaponChamsTransparency = transparency

    -- Volver a aplicar chams a todas las armas modificadas
    if weaponChamsEnabled then
        for weapon, _ in pairs(modifiedWeapons) do
            local parts = weapon:FindFirstChild('Weapon') or weapon
            if parts then
                for _, part in pairs(parts:GetDescendants()) do
                    if part:IsA('BasePart') and part:FindFirstChild('OriginalProperties') then
                        part.Transparency = weaponChamsTransparency
                    end
                end
            end
        end
    end
end

--// Detectar cambios de arma
LocalPlayer.CharacterAdded:Connect(function(character)
    character.ChildAdded:Connect(function(child)
        if child:IsA('Tool') and weaponChamsEnabled then
            applyWeaponChams(child)
        end
    end)
end)

--// Detectar armas en la cámara
Workspace.Camera.ChildAdded:Connect(function(child)
    if child.Name == 'CurrentWeapon' and weaponChamsEnabled then
        task.delay(0.1, function()
            applyWeaponChams(child)
        end)
    end
end)

-- Vehicle ESP Functions
local function createVehicleDrawing(chassis)
    local drawing = Drawing.new("Text")
    drawing.Visible = false
    drawing.Center = true
    drawing.Outline = vehicleESP.Outline
    drawing.Font = 2
    drawing.Color = vehicleESP.Color
    drawing.Size = vehicleESP.Size
    vehicleESP.ActiveDrawings[chassis] = drawing
    return drawing
end

local function updateVehicles()
    if not vehicleESP.Enabled then
        for chassis, drawing in pairs(vehicleESP.ActiveDrawings) do
            drawing.Visible = false
        end
        return
    end
    
    local camera = Workspace.CurrentCamera
    for chassis, drawing in pairs(vehicleESP.ActiveDrawings) do
        if chassis and chassis.Parent then
            local pos, onScreen = camera:WorldToViewportPoint(chassis.Position)
            if onScreen then
                drawing.Position = Vector2.new(pos.X, pos.Y)
                if vehicleESP.ShowDistance then
                    local distance = (chassis.Position - LocalPlayer.Character.PrimaryPart.Position).Magnitude
                    drawing.Text = string.format("Vehicle (%d)", distance)
                else
                    drawing.Text = "Vehicle"
                end
                drawing.Visible = true
            else
                drawing.Visible = false
            end
        else
            drawing:Remove()
            vehicleESP.ActiveDrawings[chassis] = nil
        end
    end
end

local function vehicleScanner()
    if not vehicleESP.Enabled then return end
    
    -- Buscar vehículos en el mundo
    for _, obj in pairs(Workspace:GetChildren()) do
        if obj:IsA("Model") and obj:FindFirstChild("Chassis") and not vehicleESP.ActiveDrawings[obj.Chassis] then
            createVehicleDrawing(obj.Chassis)
        end
    end
end

-- Grave ESP Functions
local function createGraveESP(model, meshPart)
    local text = Drawing.new("Text")
    text.Size = graveESP.Size
    text.Center = true
    text.Outline = graveESP.Outline
    text.Color = graveESP.Color
    text.Visible = graveESP.Enabled

    graveESP.ActiveDrawings[model] = { text = text, part = meshPart }
end

local function removeGraveESP(model)
    if graveESP.ActiveDrawings[model] then
        graveESP.ActiveDrawings[model].text:Remove()
        graveESP.ActiveDrawings[model] = nil
    end
end

local function updateGraveESP()
    local camera = Workspace.CurrentCamera
    
    -- Buscar tumbas nuevas
    for _, model in ipairs(Workspace:GetChildren()) do
        if model:IsA("Model") and model.Name == "Default" and not graveESP.ActiveDrawings[model] then
            local meshPart = model:FindFirstChildWhichIsA("MeshPart")
            if meshPart then
                createGraveESP(model, meshPart)
            end
        end
    end

    -- Actualizar tumbas existentes
    for model, data in pairs(graveESP.ActiveDrawings) do
        if not model:IsDescendantOf(Workspace) or not data.part then
            removeGraveESP(model)
        else
            local screenPos, onScreen = camera:WorldToViewportPoint(data.part.Position)
            if onScreen and graveESP.Enabled then
                local name = data.part:GetAttribute("DisplayName") or "Unknown"
                local searched = data.part:GetAttribute("Searched")
                local distanceStr = ""

                if typeof(searched) == "boolean" and searched == true then
                    name = "[Searched] " .. name
                end

                if graveESP.ShowDistance then
                    local dist = (camera.CFrame.Position - data.part.Position).Magnitude
                    distanceStr = string.format(" [%.1fm]", dist)
                end

                data.text.Text = name .. distanceStr
                data.text.Position = Vector2.new(screenPos.X, screenPos.Y)
                data.text.Size = graveESP.Size
                data.text.Color = graveESP.Color
                data.text.Outline = graveESP.Outline
                data.text.Visible = true
            else
                data.text.Visible = false
            end
        end
    end
end

-- Ammo ESP Functions
local function extract_id(str)
    if type(str) ~= "string" then
        return tostring(str)
    end
    local id = str:match("(%d+)")
    return id or str
end

local function apply_ammo_settings_to_drawing(txt)
    if not txt then return end
    txt.Outline = ammoESP.Outline
    txt.Size = ammoESP.Size
    txt.Color = ammoESP.Color
end

local function create_ammo_esp_on_source(source_part, ammo_label)
    if not ammoESP.Enabled then return end

    if ammoESP.ActiveDrawings[source_part] then return end

    local txt = Drawing.new("Text")
    txt.Center = true
    txt.Visible = true
    txt.Text = ammo_label
    apply_ammo_settings_to_drawing(txt)

    ammoESP.ActiveDrawings[source_part] = {
        drawing = txt,
        label = ammo_label,
    }
end

local function clear_all_ammo_esp()
    for source_part, data in pairs(ammoESP.ActiveDrawings) do
        if data.drawing and data.drawing.Remove then
            data.drawing:Remove()
        end
    end
    table.clear(ammoESP.ActiveDrawings)
end

local function scan_for_ammo_matches()
    if not ammoESP.Enabled then return end

    -- Acceso seguro a world_assets
    local worldAssets = Workspace:FindFirstChild("world_assets")
    if not worldAssets then return end
    
    local staticObjects = worldAssets:FindFirstChild("StaticObjects")
    if not staticObjects then return end
    
    local miscFolder = staticObjects:FindFirstChild("Misc")
    if not miscFolder then return end
    
    local ammoFolder = game:GetService("ReplicatedStorage"):WaitForChild("Assets", 5):WaitForChild("ViewModels", 5):WaitForChild("Combat", 5):WaitForChild("Ammo", 5)

    for _, misc_child in ipairs(miscFolder:GetChildren()) do
        if misc_child:IsA("Model") then
            local source = misc_child:FindFirstChild("Main")
            if source and source:IsA("MeshPart") then
                local src_mesh_id = extract_id(source.MeshId)
                local src_tex_id = extract_id(source.TextureID)
                
                for _, ammo_model in ipairs(ammoFolder:GetChildren()) do
                    if ammo_model:IsA("Model") and ammo_model.Name ~= "Model" then
                        local ammo_main = ammo_model:FindFirstChild("Main")
                        if ammo_main and ammo_main:IsA("MeshPart") then
                            local ammo_mesh_id = extract_id(ammo_main.MeshId)
                            local ammo_tex_id = extract_id(ammo_main.TextureID)
                            
                            if ammo_mesh_id == src_mesh_id and ammo_tex_id == src_tex_id then
                                create_ammo_esp_on_source(source, ammo_model.Name)
                            end
                        end
                    end
                end
            end
        end
    end
end

local function cleanup_removed_ammo()
    for source_part, data in pairs(ammoESP.ActiveDrawings) do
        local drawing_text = data.drawing
        if not source_part.Parent then
            drawing_text:Remove()
            ammoESP.ActiveDrawings[source_part] = nil
        end
    end
end

-- Weapons ESP Functions
local meleeBlacklist = {
    AxeFire = true, ChemLight = true, Cleaver = true, CombatKnife = true,
    Crowbar = true, HandSaw = true, Shovel = true, KitchenKnife = true,
    GrenadeSmokeMakeshift = true, GrenadeSmoke = true, AxeMakeshift = true,
    Cuffs = true, Fists = true, GenericItemThrow = true, Generic_consumable = true,
    GrenadeFrag = true, GrenadeGas = true, GrenadeMolotov = true,
    GrenadePipebomb = true, Snowball = true, ThrowableBottle = true,
    Wrench = true, SteakKnife = true,
}

local function getMeshPartsInfo(model)
    local info = {}
    for _, child in ipairs(model:GetChildren()) do
        if child:IsA("MeshPart") then
            local cNames = {}
            for _, c in ipairs(child:GetChildren()) do
                table.insert(cNames, c.Name)
            end
            table.sort(cNames)
            table.insert(info, { Name = child.Name, MeshId = child.MeshId, ChildrenNames = cNames })
        end
    end
    table.sort(info, function(a,b) return a.Name < b.Name end)
    return info
end

local function meshPartsMatch(a, b)
    if #a ~= #b then return false end
    for i = 1, #a do
        if a[i].Name ~= b[i].Name or a[i].MeshId ~= b[i].MeshId then
            return false
        end
        local c1, c2 = a[i].ChildrenNames, b[i].ChildrenNames
        if #c1 ~= #c2 then return false end
        for j = 1, #c1 do
            if c1[j] ~= c2[j] then return false end
        end
    end
    return true
end

local function getChildrenNames(model)
    local names = {}
    for _, c in ipairs(model:GetChildren()) do
        table.insert(names, c.Name)
    end
    table.sort(names)
    return names
end

local function childrenNamesMatch(a, b)
    if #a ~= #b then return false end
    for i = 1, #a do
        if a[i] ~= b[i] then return false end
    end
    return true
end

local function getDescendantsNames(model)
    local names = {}
    for _, d in ipairs(model:GetDescendants()) do
        table.insert(names, d.Name)
    end
    table.sort(names)
    return names
end

local function descendantsNamesMatch(a, b)
    if #a ~= #b then return false end
    for i = 1, #a do
        if a[i] ~= b[i] then return false end
    end
    return true
end

local gunDataMeshInfos = {}
local gunDataFolder = game:GetService("ReplicatedStorage"):WaitForChild("GunData", 5)
for _, folder in ipairs(gunDataFolder:GetChildren()) do
    local wm = folder:FindFirstChild("WorldModel")
    if wm then
        gunDataMeshInfos[folder.Name] = getMeshPartsInfo(wm)
    end
end

local function createWeaponESP(model, gunName)
    local handle = model:FindFirstChild("Handle")
    if not (handle and handle:IsA("BasePart")) then return end
    if weaponsESP.ActiveDrawings[model] then return end

    local text = Drawing.new("Text")
    text.Size = weaponsESP.Size
    text.Color = weaponsESP.Color
    text.Center = true
    text.Outline = weaponsESP.Outline
    text.OutlineColor = Color3.new(0, 0, 0)

    weaponsESP.ActiveDrawings[model] = {
        drawing = text,
        name = gunName,
    }
end

local function checkModelAgainstGunData(model)
    if not model:IsA("Model") then return nil end
    local meshInfo = getMeshPartsInfo(model)
    if #meshInfo > 0 then
        for name, refInfo in pairs(gunDataMeshInfos) do
            if meshPartsMatch(meshInfo, refInfo) then
                return name
            end
        end
    else
        local childNames = getChildrenNames(model)
        local candidates = {}
        for name, _ in pairs(gunDataMeshInfos) do
            local gd = gunDataFolder:FindFirstChild(name)
            if gd and gd:FindFirstChild("WorldModel") then
                if childrenNamesMatch(childNames, getChildrenNames(gd.WorldModel)) then
                    table.insert(candidates, name)
                end
            end
        end
        if #candidates == 1 then
            return candidates[1]
        elseif #candidates > 1 then
            local descNames = getDescendantsNames(model)
            for _, name in ipairs(candidates) do
                if descendantsNamesMatch(descNames, getDescendantsNames(gunDataFolder[name].WorldModel)) then
                    return name
                end
            end
        end
    end
    return nil
end

local function checkAndCreateWeaponESPForModel(model)
    local gunName = checkModelAgainstGunData(model)
    if gunName then
        if weaponsESP.ExcludeMelee and meleeBlacklist[gunName] then return end
        createWeaponESP(model, gunName)
    end
end

local function initialWeaponScan()
    -- Acceso seguro a world_assets
    local worldAssets = Workspace:FindFirstChild("world_assets")
    if not worldAssets then return end
    
    local staticObjects = worldAssets:FindFirstChild("StaticObjects")
    if not staticObjects then return end
    
    local miscFolder = staticObjects:FindFirstChild("Misc")
    if not miscFolder then return end
    
    for _, model in ipairs(miscFolder:GetChildren()) do
        checkAndCreateWeaponESPForModel(model)
    end
end

local function updateWeaponsESP()
    if not weaponsESP.Enabled then
        for model, data in pairs(weaponsESP.ActiveDrawings) do
            if data.drawing then
                data.drawing.Visible = false
            end
        end
        return
    end
    
    local camera = Workspace.CurrentCamera
    local camPos = camera.CFrame.Position
    
    for model, data in pairs(weaponsESP.ActiveDrawings) do
        local drawing = data.drawing
        local handle = model:FindFirstChild("Handle")
        
        if not handle or not model.Parent then
            drawing:Remove()
            weaponsESP.ActiveDrawings[model] = nil
        else
            local dist = (handle.Position - camPos).Magnitude
            local pos, onScreen = camera:WorldToViewportPoint(handle.Position)
            
            if onScreen and dist <= weaponsESP.RenderRange then
                if weaponsESP.ShowDistance then
                    drawing.Text = string.format("%s [%d]", data.name, math.floor(dist))
                else
                    drawing.Text = data.name
                end
                drawing.Position = Vector2.new(pos.X, pos.Y)
                drawing.Visible = true
            else
                drawing.Visible = false
            end
        end
    end
end

-- No Impact Damage Function
local function noImpactDamage()
    for _, v in pairs(game.Workspace:GetChildren()) do
        if v.Name == "WorldModel" and v:IsA("Model") then
            for _, k in pairs(v:GetDescendants()) do
                if k.Name == "Impact" and k:IsA("RemoteEvent") then
                    k:Destroy()
                end
            end
        end
    end
end

-- Body Chams Functions - CORREGIDAS
local function applyBodyChams(part)
    if part:IsA("BasePart") and not bodyChamsParts[part] then
        -- Guardar propiedades originales
        bodyChamsParts[part] = {
            Material = part.Material,
            Color = part.Color,
            Transparency = part.Transparency
        }
        
        -- Eliminar SurfaceAppearance si existe
        if part:FindFirstChild('SurfaceAppearance') then
            part.SurfaceAppearance:Destroy()
        end
        
        -- Aplicar nuevas propiedades
        part.Material = Enum.Material.ForceField
        part.Color = bodyChamsColor
        part.Transparency = bodyChamsTransparency
    end
end

local function restoreBodyChams(part)
    if bodyChamsParts[part] then
        local original = bodyChamsParts[part]
        part.Material = original.Material
        part.Color = original.Color
        part.Transparency = original.Transparency
        bodyChamsParts[part] = nil
    end
end

local function processCharacterForBodyChams(character)
    if character and character:IsA("Model") then
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                if bodyChamsEnabled then
                    applyBodyChams(part)
                else
                    restoreBodyChams(part)
                end
            end
        end
    end
end

local function applyToAllBodyChams()
    -- Aplicar a personajes existentes
    local characters = workspace:FindFirstChild("Characters")
    if characters then
        for _, character in pairs(characters:GetChildren()) do
            processCharacterForBodyChams(character)
        end
    end
    
    -- Aplicar a skins en la cámara
    local camera = workspace.CurrentCamera
    for _, skin in pairs(camera:GetDescendants()) do
        if skin.Name == "Skin" and skin:IsA("BasePart") then
            if bodyChamsEnabled then
                applyBodyChams(skin)
            else
                restoreBodyChams(skin)
            end
        end
    end
end

local function toggleBodyChams()
    if bodyChamsEnabled then
        -- Activar Body Chams
        if not bodyChamsConnection then
            bodyChamsConnection = RunService.Heartbeat:Connect(applyToAllBodyChams)
        end
        applyToAllBodyChams()
        print("Body Chams activados")
    else
        -- Desactivar Body Chams
        for part, _ in pairs(bodyChamsParts) do
            restoreBodyChams(part)
        end
        bodyChamsParts = {}
        
        if bodyChamsConnection then
            bodyChamsConnection:Disconnect()
            bodyChamsConnection = nil
        end
        print("Body Chams desactivados")
    end
end

-- Conexiones para Body Chams
workspace.DescendantAdded:Connect(function(descendant)
    if descendant.Name == "Characters" and descendant:IsA("Folder") then
        for _, character in pairs(descendant:GetChildren()) do
            if bodyChamsEnabled then
                processCharacterForBodyChams(character)
            end
        end
    end
end)

workspace.CurrentCamera.DescendantAdded:Connect(function(descendant)
    if descendant.Name == "Skin" and descendant:IsA("BasePart") and bodyChamsEnabled then
        applyBodyChams(descendant)
    end
end)

-- New example script written by wally
-- You can suggest changes with a pull request with a suggestion

local repo = 'https://raw.githubusercontent.com/violin-suzutsuki/LinoriaLib/main/'

local Library = loadstring(game:HttpGet(repo .. 'Library.lua'))()
local ThemeManager = loadstring(game:HttpGet(repo .. 'addons/ThemeManager.lua'))()
local SaveManager = loadstring(game:HttpGet(repo .. 'addons/SaveManager.lua'))()
print("DogStire Lite Version Bypassed");

local Window = Library:CreateWindow({
    Title = 'Lunar.Hook',
    Center = true,
    AutoShow = true,
    TabPadding = 8,
    MenuFadeTime = 0.2
})

local Tabs = {
    Main = Window:AddTab('Main'),
    Visuals = Window:AddTab('Visuals'),
    ItemESP = Window:AddTab('Item ESP'),
    World = Window:AddTab('World'),
    Character = Window:AddTab('Character'), -- Nueva pestaña Character
    Misc = Window:AddTab('Misc'),
    ['UI Settings'] = Window:AddTab('UI Settings'),
}

-- Variables del nuevo Aimbot
local aimbot = {
    enabled = false,
    showFOV = false,
    showSnapline = false,
    fovRadius = 200,
    fovThickness = 2,
    snaplineThickness = 2,
    smoothing = 0.5,
    fovColor = Color3.fromRGB(255, 255, 255),
    snaplineColor = Color3.fromRGB(255, 0, 0),
    fovRainbow = false,
    snaplineRainbow = false,
    keybind = Enum.UserInputType.MouseButton2,
    aiming = false,
    predictionXAdjust = 1,
    predictionYAdjust = 1,
    GRAVITY = 60
}

-- Variables del nuevo ESP
local espSettings = {
    enabled = false,
    showBoxes = false,
    filledBoxes = false,
    boxTransparency = 0.5,
    boxThickness = 2,
    boxColor = Color3.fromRGB(255, 255, 255),
    showNames = false,
    nameTextSize = 14,
    nameOutlines = false,
    nameColor = Color3.fromRGB(255, 255, 255),
    showDistance = false,
    distanceTextSize = 16,
    distanceOutlines = false,
    distanceColor = Color3.fromRGB(255, 255, 255),
    distanceUnits = "Studs",
    hideteammates = false,
    refreshRate = 0
}

-- Variables del Skeleton ESP
local skeleton = {
    enabled = false,
    color = Color3.new(1, 1, 1),
    thickness = 1.8
}

-- Variables para el Inventory Viewer
local inventoryViewerSettings = {
    enabled = false
}

-- Variables para el sistema de escuadrón
local squadMembers = {}

-- Variables para World
local worldSettings = {
    noGrass = false,
    noRain = false,
    noFoliage = false,
    noFog = false,
    noGreenGas = false,
    foliageTransparency = 0.5,
    timeOfDay = false,
    timeValue = 12,
    skybox = "Default"
}

-- Variables para Combat adicionales
local combatSettings = {
    instantAim = false,
    noRecoil = false,
    autoReload = false,  -- Nueva variable para Auto Reload
    antiAim = false,
    antiAimPitch = 90,
    antiAimSpeed = 90,
    fovChanger = false,
    fovValue = 90,
    gunOffset = false,
    offsetX = 2,
    offsetY = 0,
    offsetZ = -2,
    noImpactDamage = false  -- Nueva variable para No Impact Damage
}

-- Variables para almacenar funciones originales y conexiones
local originalFunctions = {}
local connections = {}

-- Variables para guardar valores originales
local originalTimeOfDay = 12
local originalFoliageTransparency = {}
local originalAtmosphereDensity = 0.3
local originalRainEnabled = true -- Para guardar estado original de la lluvia

-- Variables para el FOV Changer (CORREGIDAS)
local fovchanger = false
local currentFovValue = 90 -- Valor por defecto del slider
local defaultFov = 90 -- FOV predeterminado del juego

-- Variables para el Gun Offset (NUEVA IMPLEMENTACIÓN)
local offsetEnabled = false
local offsetX, offsetY, offsetZ = 2, 0, -2
local heartbeatConn = nil
local originalOffsets = {}

-- Variables para el Anti Aim (NUEVA IMPLEMENTACIÓN)
local antiAimEnabled = false
local fakePitchMax = math.rad(90)
local fakePitchMin = math.rad(-90)
local fakeYawSpeed = math.rad(20)
local currentYaw = 0
local pitchSpeed = 2

-- Variables para No Foliage (NUEVAS)
local foliageCache = {} -- Para guardar los valores originales
local foliageConnections = {} -- Para guardar las conexiones de las partes

-- Variable global para No Grass
getgenv().grassdisabled = false

-- Variables para Auto Reload
local autoReloadConnection = nil

-- Optimización: Variables de caché
local lastSquadUpdate = 0
local lastWorldCheck = 0
local lastEspUpdate = 0
local SQUAD_UPDATE_INTERVAL = 2
local WORLD_CHECK_INTERVAL = 0.5
local ESP_UPDATE_INTERVAL = 0.1

-- Guardar skybox original
local originalSkybox = {}
local function saveOriginalSkybox()
    local sky = Lighting:FindFirstChild("Sky")
    if sky then
        originalSkybox = {
            SkyboxBk = sky.SkyboxBk,
            SkyboxDn = sky.SkyboxDn,
            SkyboxFt = sky.SkyboxFt,
            SkyboxLf = sky.SkyboxLf,
            SkyboxRt = sky.SkyboxRt,
            SkyboxUp = sky.SkyboxUp
        }
    else
        originalSkybox = nil
    end
end

saveOriginalSkybox()

-- Guardar valores originales al inicio
originalTimeOfDay = Lighting.ClockTime
if Lighting.Atmosphere then
    originalAtmosphereDensity = Lighting.Atmosphere.Density
end

-- Guardar transparencia original del follaje
local function saveOriginalFoliageTransparency()
    for _, part in ipairs(workspace:GetDescendants()) do
        if part:IsA("BasePart") and (part.Name == "Leaves" or part.Material == Enum.Material.LeafyGrass) then
            originalFoliageTransparency[part] = part.Transparency
        end
    end
end

saveOriginalFoliageTransparency()

-- Guardar estado original de la lluvia
local function saveOriginalRainState()
    originalRainEnabled = Workspace.Terrain:FindFirstChild("__RainSplash") ~= nil
end

saveOriginalRainState()

-- Crear círculo FOV
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = aimbot.fovRadius
fovCircle.Thickness = aimbot.fovThickness
fovCircle.NumSides = 64
fovCircle.Color = aimbot.fovColor
fovCircle.Transparency = 1
fovCircle.Filled = false
fovCircle.Visible = aimbot.showFOV

-- Optimización: Reutilizar objetos Drawing
local espObjects = {}
local skeletonLines = {}
local CharactersFolder = workspace:WaitForChild("Characters") or workspace

-- Configuración del Inventory Viewer
local boxWidth = 250
local lineSpacing = 18
local boxPadding = 3
local currentPos = Vector2.new(Camera.ViewportSize.X - boxWidth - 20, 20)
local targetPos = currentPos
local shadowOffset = 5
local maxLines = 15

-- Crear objetos del Inventory Viewer (inicialmente ocultos)
local inventoryBox = Drawing.new("Square")
inventoryBox.Filled = true
inventoryBox.Color = Color3.fromRGB(0, 0, 0)
inventoryBox.Transparency = 1.85
inventoryBox.Visible = false
inventoryBox.ZIndex = 1

local inventoryBorder = Drawing.new("Square")
inventoryBorder.Filled = false
inventoryBorder.Color = Color3.fromRGB(0, 0, 0)
inventoryBorder.Thickness = 2
inventoryBorder.Visible = false
inventoryBorder.ZIndex = 2

local inventoryTitle = Drawing.new("Text")
inventoryTitle.Text = "Inventory Viewer"
inventoryTitle.Size = 16
inventoryTitle.Color = Color3.fromRGB(138, 43, 226)
inventoryTitle.Font = 4
inventoryTitle.Outline = true
inventoryTitle.OutlineColor = Color3.fromRGB(0, 0, 0)
inventoryTitle.Visible = false
inventoryTitle.ZIndex = 3

local inventoryTextLines = {}
for i = 1, maxLines do
    local t = Drawing.new("Text")
    t.Size = 13
    t.Color = Color3.fromRGB(138, 43, 226)
    t.Font = 4
    t.Outline = true
    t.OutlineColor = Color3.fromRGB(0, 0, 0)
    t.Visible = false
    t.ZIndex = 4
    table.insert(inventoryTextLines, t)
end

-- Sistema de arrastre para el Inventory Viewer
local dragging = false
local dragStartPos = nil
local mouseStartPos = nil

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local mousePos = Vector2.new(Mouse.X, Mouse.Y)
        if mousePos.X >= inventoryBox.Position.X and mousePos.X <= inventoryBox.Position.X + inventoryBox.Size.X and
           mousePos.Y >= inventoryBox.Position.Y and mousePos.Y <= inventoryBox.Position.Y + inventoryBox.Size.Y and
           inventoryViewerSettings.enabled then
            dragging = true
            dragStartPos = currentPos
            mouseStartPos = mousePos
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        targetPos = Vector2.new(Mouse.X, Mouse.Y) - (mouseStartPos - dragStartPos)
    end
end)

-- Función para actualizar el Inventory Viewer
local function updateInventoryViewer()
    currentPos = currentPos:Lerp(targetPos, 0.2)
    
    -- Actualizar tamaño y posición
    local height = (#inventoryTextLines + 2) * lineSpacing + boxPadding * 2
    inventoryBox.Size = Vector2.new(boxWidth, height)
    inventoryBox.Position = currentPos
    
    inventoryBorder.Size = Vector2.new(boxWidth + 5, height + 5)
    inventoryBorder.Position = Vector2.new(currentPos.X - 2.5, currentPos.Y - 2.5)
    
    inventoryTitle.Position = Vector2.new(currentPos.X + boxPadding, currentPos.Y + boxPadding)
    
    for i, t in ipairs(inventoryTextLines) do
        if t.Visible then
            t.Position = Vector2.new(currentPos.X + boxPadding, currentPos.Y + boxPadding + 20 + (i-1) * lineSpacing)
        end
    end
    
    -- Obtener información del objetivo
    local target = nil
    local minDist = math.huge
    local center = Camera.ViewportSize / 2
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local pos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            if onScreen then
                local dist = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                if dist < minDist then
                    minDist = dist
                    target = player
                end
            end
        end
    end
    
    -- Actualizar texto
    for _, t in ipairs(inventoryTextLines) do
        t.Visible = false
    end
    
    if target and target:FindFirstChild("GunInventory") then
        local lines = {target.Name}
        for _, gun in ipairs(target.GunInventory:GetChildren()) do
            if gun:IsA("ObjectValue") and gun.Value then
                local name = tostring(gun.Value)
                local scope = gun:FindFirstChild("AttachmentReticle") and tostring(gun.AttachmentReticle.Value) .. "x" or "No scope"
                table.insert(lines, name .. " - " .. scope)
                
                local mag = gun:FindFirstChild("BulletsInMagazine")
                local reserve = gun:FindFirstChild("BulletsInReserve")
                if mag and reserve then
                    table.insert(lines, tostring(mag.Value) .. " / " .. tostring(reserve.Value))
                end
            end
        end
        
        for i, line in ipairs(lines) do
            if i <= maxLines then
                inventoryTextLines[i].Text = line
                inventoryTextLines[i].Visible = true
            end
        end
    else
        inventoryTextLines[1].Text = "sem alvo"
        inventoryTextLines[1].Visible = true
    end
end

-- Función para mostrar/ocultar el Inventory Viewer
local function toggleInventoryViewer(state)
    inventoryViewerSettings.enabled = state
    
    inventoryBox.Visible = state
    inventoryBorder.Visible = state
    inventoryTitle.Visible = state
    
    for _, t in ipairs(inventoryTextLines) do
        t.Visible = false
    end
    
    if state then
        inventoryTextLines[1].Text = "No target"
        inventoryTextLines[1].Visible = true
    end
end

-- Función para aplicar No Impact Damage
local function applyNoImpactDamage()
    if combatSettings.noImpactDamage then
        noImpactDamage()
        -- Conectar evento para detectar nuevos WorldModels
        if not connections.noImpactDamage then
            connections.noImpactDamage = workspace.ChildAdded:Connect(function(child)
                if child.Name == "WorldModel" and child:IsA("Model") then
                    for _, k in pairs(child:GetDescendants()) do
                        if k.Name == "Impact" and k:IsA("RemoteEvent") then
                            k:Destroy()
                        end
                    end
                end
            end)
        end
    else
        if connections.noImpactDamage then
            connections.noImpactDamage:Disconnect()
            connections.noImpactDamage = nil
        end
    end
end

-- Optimización: Función para actualizar miembros del escuadrón con caché
local function updateSquadMembers()
    local currentTime = tick()
    if currentTime - lastSquadUpdate < SQUAD_UPDATE_INTERVAL then return end
    lastSquadUpdate = currentTime
    
    squadMembers = {}
    local wrapper = LocalPlayer:FindFirstChild("PlayerGui")
    if wrapper then
        local gameUI = wrapper:FindFirstChild("GameUI")
        if gameUI then
            local menu = gameUI:FindFirstChild("Menu")
            if menu then
                local squad = menu:FindFirstChild("Squad")
                if squad then
                    local squadPage = squad:FindFirstChild("SquadPage")
                    if squadPage then
                        local squadWrapper = squadPage:FindFirstChild("Squad")
                        if squadWrapper then
                            local wrapper = squadWrapper:FindFirstChild("Wrapper")
                            if wrapper then
                                for _, v in pairs(wrapper:GetDescendants()) do
                                    if v:IsA("TextLabel") and v.Name == "Label" then
                                        if v.Text ~= "DISBAND" and v.Text ~= "LEAVE" then
                                            squadMembers[v.Text] = true
                                        end
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end
    end
end

-- Funciones auxiliares
local function worldToScreen(pos)
    local pt, onScreen = Camera:WorldToViewportPoint(pos)
    return Vector2.new(pt.X, pt.Y), onScreen
end

-- Optimización: Caché de velocidad de bala
local lastBulletSpeedCheck = 0
local cachedBulletSpeed = 2200
local BULLET_SPEED_CACHE_INTERVAL = 1

local function getEquippedBulletSpeed()
    local currentTime = tick()
    if currentTime - lastBulletSpeedCheck < BULLET_SPEED_CACHE_INTERVAL then
        return cachedBulletSpeed
    end
    lastBulletSpeedCheck = currentTime
    
    local sel = LocalPlayer.CurrentSelectedObject and LocalPlayer.CurrentSelectedObject.Value
    if typeof(sel) ~= "Instance" then 
        cachedBulletSpeed = 2200
        return 2200 
    end

    local slot = LocalPlayer.GunInventory:FindFirstChild(sel.Name)
    if not (slot and slot.Value) then 
        cachedBulletSpeed = 2200
        return 2200 
    end

    local gunName
    if typeof(slot.Value) == "string" then
        gunName = slot.Value
    elseif typeof(slot.Value) == "Instance" then
        gunName = tostring(slot.Value)
    else
        cachedBulletSpeed = 2250
        return 2250
    end

    local data = ReplicatedStorage.GunData:FindFirstChild(gunName)
    if not data or not data:FindFirstChild("Stats") or not data.Stats:FindFirstChild("BulletSettings") then 
        cachedBulletSpeed = 2200
        return 2200 
    end

    local spd = data.Stats.BulletSettings:FindFirstChild("BulletSpeed")
    cachedBulletSpeed = (spd and spd:IsA("IntValue")) and spd.Value or 2200
    return cachedBulletSpeed
end

-- Función para calcular la caída de la bala
local function calculateDrop(distance, speed)
    if not speed or speed <= 0 then return 0 end
    local t = distance / speed
    return 0.5 * aimbot.GRAVITY * t * t
end

-- Función para convertir distancia según unidades
local function convertDistance(studs, unit)
    local conversions = {
        ["Studs"] = studs,
        ["Meters"] = studs * 0.28,
        ["Feet"] = studs * 0.9186,
        ["Yards"] = studs * 0.3062,
        ["Inches"] = studs * 11.0236,
        ["Kilometers"] = studs * 0.00028
    }
    return conversions[unit] or studs
end

-- Funciones del Skeleton ESP
local function createLine()
    local line = Drawing.new("Line")
    line.Visible = false
    line.Color = skeleton.color
    line.Thickness = skeleton.thickness
    line.Transparency = 1
    return line
end

local R15Bones = {
    {"Head", "UpperTorso"},
    {"UpperTorso", "LowerTorso"},
    {"LowerTorso", "LeftUpperLeg"},
    {"LeftUpperLeg", "LeftLowerLeg"},
    {"LeftLowerLeg", "LeftFoot"},
    {"LowerTorso", "RightUpperLeg"},
    {"RightUpperLeg", "RightLowerLeg"},
    {"RightLowerLeg", "RightFoot"},
    {"UpperTorso", "LeftUpperArm"},
    {"LeftUpperArm", "LeftLowerArm"},
    {"LeftLowerArm", "LeftHand"},
    {"UpperTorso", "RightUpperArm"},
    {"RightUpperArm", "RightLowerArm"},
    {"RightLowerArm", "RightHand"},
}

local function createSkeleton(character)
    local lines = {}
    for _, bone in ipairs(R15Bones) do
        local part0 = character:FindFirstChild(bone[1])
        local part1 = character:FindFirstChild(bone[2])
        if part0 and part1 then
            local line = createLine()
            table.insert(lines, {line = line, part0 = part0, part1 = part1})
        end
    end
    return lines
end

local function updateSkeleton(character, lines)
    local root = character:FindFirstChild("HumanoidRootPart")
    if not root then return end

    -- Verificar si el personaje pertenece al jugador local
    local player = Players:GetPlayerFromCharacter(character)
    if player and player == LocalPlayer then
        -- Ocultar todas las líneas si es el jugador local
        for _, data in pairs(lines) do
            data.line.Visible = false
        end
        return
    end

    -- Solo mostrar si tanto el ESP general como el Skeleton están activados
    if not skeleton.enabled or not espSettings.enabled then
        for _, data in pairs(lines) do
            data.line.Visible = false
        end
        return
    end

    for _, data in pairs(lines) do
        local pos0, onScreen0 = Camera:WorldToViewportPoint(data.part0.Position)
        local pos1, onScreen1 = Camera:WorldToViewportPoint(data.part1.Position)
        if onScreen0 and onScreen1 then
            data.line.From = Vector2.new(pos0.X, pos0.Y)
            data.line.To = Vector2.new(pos1.X, pos1.Y)
            data.line.Visible = true
        else
            data.line.Visible = false
        end
    end
end

local function clearSkeleton(lines)
    for _, data in pairs(lines) do
        data.line.Visible = false
        data.line:Remove()
    end
end

-- Funciones del nuevo ESP
local function createESP()
    return {
        box = Drawing.new("Square"),
        name = Drawing.new("Text"),
        distance = Drawing.new("Text")
    }
end

local function clearESP(esp)
    for _, obj in pairs(esp) do
        if obj and obj.Remove then
            obj:Remove()
        end
    end
end

local function hideESP(esp)
    for _, obj in pairs(esp) do
        obj.Visible = false
    end
end

local function setupPlayerESP(player)
    if player == LocalPlayer then return end
    espObjects[player] = createESP()

    player.CharacterAdded:Connect(function()
        -- No necesitamos hacer nada aquí, el ESP se actualizará en el bucle principal
    end)

    player.CharacterRemoving:Connect(function()
        hideESP(espObjects[player])
    end)
end

-- Funciones de Combat adicionales

-- Instant Aim
local function applyInstantAim()
    if combatSettings.instantAim then
        for _, v in next, getgc(true) do
            if type(v) == 'table' then
                for i, val in next, v do
                    if type(i) == 'string' and string.find(i, 'GunAim') then
                        if type(val) == 'number' then
                            v[i] = 100000000
                        elseif type(val) == 'function' and not originalFunctions[i] then
                            originalFunctions[i] = val
                            hookfunction(val, function(...)
                                return 100000000
                            end)
                        end
                    end
                end
            end
        end
    else
        -- Restaurar funciones originales
        for i, originalFunc in pairs(originalFunctions) do
            if type(i) == 'string' and string.find(i, 'GunAim') then
                for _, v in next, getgc(true) do
                    if type(v) == 'table' and v[i] then
                        v[i] = originalFunc
                    end
                end
            end
        end
        originalFunctions = {}
    end
end

-- No Recoil
local function applyNoRecoil()
    if combatSettings.noRecoil then
        local spring = require(game:GetService("Players").LocalPlayer:WaitForChild("PlayerScripts"):WaitForChild("GunController"):WaitForChild("Data"):WaitForChild("Spring"))
        local velocity = spring._positionVelocity

        if not originalFunctions["velocity"] then
            originalFunctions["velocity"] = velocity
        end
        
        hookfunction(velocity, newcclosure(function(self, now)
            return self._target or Vector3.zero, Vector3.zero
        end))
    else
        -- Restaurar función original
        if originalFunctions["velocity"] then
            local spring = require(game:GetService("Players").LocalPlayer:WaitForChild("PlayerScripts"):WaitForChild("GunController"):WaitForChild("Data"):WaitForChild("Spring"))
            hookfunction(spring._positionVelocity, originalFunctions["velocity"])
            originalFunctions["velocity"] = nil
        end
    end
end

-- Auto Reload
local function toggleAutoReload()
    if combatSettings.autoReload then
        if not autoReloadConnection then
            autoReloadConnection = game:GetService("RunService").Heartbeat:Connect(function()
                local ReplicatedStorage = game:GetService("ReplicatedStorage")
                local gunEventFolder = ReplicatedStorage:FindFirstChild("GunEvent")
                if gunEventFolder then
                    local gunReloadStart = gunEventFolder:FindFirstChild("GunReloadStart")
                    if gunReloadStart then
                        gunReloadStart:FireServer()
                    end
                end
            end)
        end
    else
        if autoReloadConnection then
            autoReloadConnection:Disconnect()
            autoReloadConnection = nil
        end
    end
end

-- Función applyFovChanger (CORREGIDA DEFINITIVAMENTE)
local function applyFovChanger()
    if combatSettings.fovChanger then
        fovchanger = true
        pcall(function()
            -- Obtener el controlador cada vez para evitar problemas de inicialización
            local controller = require(LocalPlayer.PlayerScripts:WaitForChild("CameraFovController"))
            -- Aplicamos el FOV actual del slider
            controller.ClientFov = currentFovValue
            controller:Enable()
        end)
    else
        fovchanger = false
        -- Usamos delay para asegurar que se aplique el cambio
        delay(0.1, function()
            pcall(function()
                -- Obtener el controlador cada vez
                local controller = require(LocalPlayer.PlayerScripts:WaitForChild("CameraFovController"))
                -- Restauramos el FOV predeterminado (90)
                controller.ClientFov = defaultFov
                controller:Enable()
            end)
        end)
    end
end

-- Gun Offset - NUEVA IMPLEMENTACIÓN
-- Función para aplicar el offset
local function applyOffset()
    local player = Players.LocalPlayer
    local selObj = player.CurrentSelectedObject and player.CurrentSelectedObject.Value
    if not selObj then return end

    local gunName = selObj.Name
    local invItem = player.GunInventory:FindFirstChild(gunName)
    if not (invItem and invItem.Value) then return end

    local dataFolder = ReplicatedStorage.GunData:FindFirstChild(invItem.Value.Name)
    if not dataFolder then return end

    local offsetValue = dataFolder.Stats:FindFirstChild("Offset")
    if not (offsetValue and offsetValue:IsA("Vector3Value")) then return end

    if not originalOffsets[invItem.Value.Name] then
        originalOffsets[invItem.Value.Name] = offsetValue.Value
    end

    offsetValue.Value = Vector3.new(offsetX, offsetY, offsetZ)
end

-- Función para restaurar offsets originales
local function restoreOffsets()
    for gunName, origValue in pairs(originalOffsets) do
        local dataFolder = ReplicatedStorage.GunData:FindFirstChild(gunName)
        if dataFolder then
            local offsetValue = dataFolder.Stats:FindFirstChild("Offset")
            if offsetValue and offsetValue:IsA("Vector3Value") then
                offsetValue.Value = origValue
            end
        end
    end
    table.clear(originalOffsets)
end

local function applyGunOffset()
    offsetEnabled = combatSettings.gunOffset

    if offsetEnabled then
        if not heartbeatConn then
            heartbeatConn = RunService.Heartbeat:Connect(applyOffset)
        end
    else
        if heartbeatConn then
            heartbeatConn:Disconnect()
            heartbeatConn = nil
        end
        restoreOffsets()
    end
end

-- Anti Aim - NUEVA IMPLEMENTACIÓN
local function applyAntiAim()
    antiAimEnabled = combatSettings.antiAim
    -- No necesitamos crear una conexión aquí ya que usamos BindToRenderStep
end

-- Funciones de World

-- No Grass - Nueva implementación
local function applyNoGrass()
    if worldSettings.noGrass then
        -- Si ya está deshabilitado, no hacemos nada
        if getgenv().grassdisabled then return end
        
        local terrain = Workspace:WaitForChild("Terrain", 10)
        local CLEAR_RADIUS = 512
        local VERTICAL_EXT  = 128
        local INTERVAL     = 5
        local lastTime     = 0

        local function clearGrassAt(position)
            local minV = position - Vector3.new(CLEAR_RADIUS, VERTICAL_EXT, CLEAR_RADIUS)
            local maxV = position + Vector3.new(CLEAR_RADIUS, VERTICAL_EXT, CLEAR_RADIUS)
            local region = Region3.new(minV, maxV):ExpandToGrid(4)
            terrain:ReplaceMaterial(region, 4, Enum.Material.Grass, Enum.Material.LeafyGrass)
        end

        if Workspace.CurrentCamera then
            clearGrassAt(Workspace.CurrentCamera.CFrame.Position)
        end

        connections.noGrass = RunService.Heartbeat:Connect(function(dt)
            lastTime = lastTime + dt
            if lastTime < INTERVAL then return end
            lastTime = 0

            local cam = Workspace.CurrentCamera
            if cam then
                clearGrassAt(cam.CFrame.Position)
            end
        end)
        getgenv().grassdisabled = true
    else
        if connections.noGrass then
            connections.noGrass:Disconnect()
            connections.noGrass = nil
            getgenv().grassdisabled = false
            
            -- Restaurar césped original
            local terrain = Workspace:FindFirstChild("Terrain")
            if terrain then
                local CLEAR_RADIUS = 512
                local VERTICAL_EXT = 128
                
                local function restoreGrassAt(position)
                    local minV = position - Vector3.new(CLEAR_RADIUS, VERTICAL_EXT, CLEAR_RADIUS)
                    local maxV = position + Vector3.new(CLEAR_RADIUS, VERTICAL_EXT, CLEAR_RADIUS)
                    local region = Region3.new(minV, maxV):ExpandToGrid(4)
                    terrain:ReplaceMaterial(region, 4, Enum.Material.LeafyGrass, Enum.Material.Grass)
                end
                
                if Workspace.CurrentCamera then
                    restoreGrassAt(Workspace.CurrentCamera.CFrame.Position)
                end
            end
        end
    end
end

-- No Rain
local function applyNoRain()
    if worldSettings.noRain then
        if not connections.noRain then
            connections.noRain = RunService.RenderStepped:Connect(function()
                for _, obj in ipairs(workspace.Terrain:GetChildren()) do
                    if obj.Name == "__RainSplash" then
                        obj:Destroy()
                    end
                end
            end)
        end
    else
        if connections.noRain then
            connections.noRain:Disconnect()
            connections.noRain = nil
            
            -- Restaurar lluvia original si estaba habilitada
            if originalRainEnabled then
                -- Forzar la regeneración de la lluvia
                local rainEnabled = Workspace:FindFirstChild("RainEnabled")
                if rainEnabled then
                    rainEnabled.Value = true
                end
            end
        end
    end
end

-- No Foliage - IMPLEMENTACIÓN CORREGIDA
local function applyNoFoliage()
    if worldSettings.noFoliage then
        -- Activar No Foliage
        -- Buscar todas las partes de follaje en el mundo
        for _, part in ipairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") and (part.Name == "Leaves" or part.Material == Enum.Material.LeafyGrass) then
                -- Guardar el valor original si no está en caché
                if not foliageCache[part] then
                    foliageCache[part] = part.Transparency
                    -- Conectar un evento para restaurar cuando se desactive
                    local connection
                    connection = part:GetPropertyChangedSignal("Transparency"):Connect(function()
                        if worldSettings.noFoliage then
                            -- Si está activado, forzar la transparencia
                            part.Transparency = worldSettings.foliageTransparency
                        end
                    end)
                    foliageConnections[part] = connection
                end
                -- Aplicar la transparencia configurada
                part.Transparency = worldSettings.foliageTransparency
            end
        end
        
        -- También conectar un evento para nuevas partes que aparezcan
        if not connections.foliageAdded then
            connections.foliageAdded = workspace.DescendantAdded:Connect(function(descendant)
                if descendant:IsA("BasePart") and (descendant.Name == "Leaves" or descendant.Material == Enum.Material.LeafyGrass) then
                    if not foliageCache[descendant] then
                        foliageCache[descendant] = descendant.Transparency
                        local connection
                        connection = descendant:GetPropertyChangedSignal("Transparency"):Connect(function()
                            if worldSettings.noFoliage then
                                descendant.Transparency = worldSettings.foliageTransparency
                            end
                        end)
                        foliageConnections[descendant] = connection
                    end
                    descendant.Transparency = worldSettings.foliageTransparency
                end
            end)
        end
    else
        -- Desactivar No Foliage: restaurar valores originales
        for part, originalTransparency in pairs(foliageCache) do
            if part and part.Parent then
                part.Transparency = originalTransparency
                -- Desconectar el evento
                if foliageConnections[part] then
                    foliageConnections[part]:Disconnect()
                    foliageConnections[part] = nil
                end
            end
        end
        
        -- Limpiar caché y conexiones
        foliageCache = {}
        for _, connection in pairs(foliageConnections) do
            connection:Disconnect()
        end
        foliageConnections = {}
        
        -- Desconectar el evento de añadido
        if connections.foliageAdded then
            connections.foliageAdded:Disconnect()
            connections.foliageAdded = nil
        end
    end
end

-- No Fog
local function applyNoFog()
    if worldSettings.noFog then
        if not connections.noFog then
            connections.noFog = RunService.RenderStepped:Connect(function()
                if Lighting.Atmosphere then
                    Lighting.Atmosphere.Density = 0
                end
            end)
        end
    else
        if connections.noFog then
            connections.noFog:Disconnect()
            connections.noFog = nil
            
            -- Restaurar densidad original
            if Lighting.Atmosphere then
                Lighting.Atmosphere.Density = originalAtmosphereDensity
            end
        end
    end
end

-- No Green Gas
local function applyNoGreenGas()
    if worldSettings.noGreenGas then
        if game.Workspace:FindFirstChild("world_assets") then
            if game.Workspace:FindFirstChild("world_assets").StaticObjects.Misc:FindFirstChild("TOSIC") then
                if game.Workspace:FindFirstChild("world_assets").StaticObjects.Misc.TOSIC:FindFirstChild("Model") then
                    -- Guardar referencia al gas verde
                    local gasModel = game.Workspace:FindFirstChild("world_assets").StaticObjects.Misc.TOSIC.Model
                    local gasChildren = {}
                    for i,v in pairs(game.Workspace:FindFirstChild("world_assets").StaticObjects.Misc.TOSIC:GetChildren()) do
                        table.insert(gasChildren, v)
                    end
                    
                    -- Mover a StarterPack
                    gasModel.Parent = game.StarterPack
                    for i,v in pairs(gasChildren) do
                        v.Parent = game.StarterPack
                    end
                    
                    -- Guardar referencias para restaurar después
                    connections.gasModel = gasModel
                    connections.gasChildren = gasChildren
                end
            end
        end
    else
        if connections.gasModel and connections.gasChildren then
            -- Restaurar gas verde a su ubicación original
            if game.Workspace:FindFirstChild("world_assets") then
                connections.gasModel.Parent = game.Workspace:FindFirstChild("world_assets").StaticObjects.Misc.TOSIC
                for i,v in pairs(connections.gasChildren) do
                    v.Parent = game.Workspace:FindFirstChild("world_assets").StaticObjects.Misc.TOSIC
                end
            end
            
            -- Limpiar referencias
            connections.gasModel = nil
            connections.gasChildren = nil
        end
    end
end

-- Time Of Day - CORREGIDO
local function applyTimeOfDay()
    if worldSettings.timeOfDay then
        -- Si ya hay una conexión, la desconectamos
        if connections.timeOfDay then
            connections.timeOfDay:Disconnect()
            connections.timeOfDay = nil
        end
        -- Establecemos el tiempo actual
        Lighting.ClockTime = worldSettings.timeValue
        -- Creamos una nueva conexión para mantener el tiempo
        connections.timeOfDay = RunService.RenderStepped:Connect(function()
            Lighting.ClockTime = worldSettings.timeValue
        end)
    else
        if connections.timeOfDay then
            connections.timeOfDay:Disconnect()
            connections.timeOfDay = nil
        end
        -- Restaurar tiempo original inmediatamente
        Lighting.ClockTime = originalTimeOfDay
    end
end

-- Skybox
local function applySkybox()
    local sky = Lighting:FindFirstChild("Sky")
    
    if worldSettings.skybox == "Default" then
        -- Restaurar skybox original
        if sky then
            sky:Destroy()
        end
        
        if originalSkybox then
            local newSky = Instance.new("Sky")
            newSky.Parent = Lighting
            newSky.SkyboxBk = originalSkybox.SkyboxBk
            newSky.SkyboxDn = originalSkybox.SkyboxDn
            newSky.SkyboxFt = originalSkybox.SkyboxFt
            newSky.SkyboxLf = originalSkybox.SkyboxLf
            newSky.SkyboxRt = originalSkybox.SkyboxRt
            newSky.SkyboxUp = originalSkybox.SkyboxUp
        end
    elseif skyboxes[worldSettings.skybox] then
        -- Usar skyboxes de la tabla skyboxes
        if not sky then
            sky = Instance.new("Sky")
            sky.Parent = Lighting
        end
        
        local skyboxData = skyboxes[worldSettings.skybox]
        sky.SkyboxBk = "rbxassetid://" .. skyboxData[1]
        sky.SkyboxDn = "rbxassetid://" .. skyboxData[2]
        sky.SkyboxFt = "rbxassetid://" .. skyboxData[3]
        sky.SkyboxLf = "rbxassetid://" .. skyboxData[4]
        sky.SkyboxRt = "rbxassetid://" .. skyboxData[5]
        sky.SkyboxUp = "rbxassetid://" .. skyboxData[6]
    else
        -- Usar skyboxes existentes
        if not sky then
            sky = Instance.new("Sky")
            sky.Parent = Lighting
        end

        if worldSettings.skybox == "Galaxy" then
            sky.SkyboxBk = "rbxassetid://1138550863"
            sky.SkyboxDn = "rbxassetid://1138551165"
            sky.SkyboxFt = "rbxassetid://1138552163"
            sky.SkyboxLf = "rbxassetid://1138551555"
            sky.SkyboxRt = "rbxassetid://1138552890"
            sky.SkyboxUp = "rbxassetid://153520294"
        elseif worldSettings.skybox == "Island" then
            sky.SkyboxBk = "http://www.roblox.com/asset/?id=14753804949"
            sky.SkyboxDn = "http://www.roblox.com/asset/?id=14753795573"
            sky.SkyboxFt = "http://www.roblox.com/asset/?id=14753807625"
            sky.SkyboxLf = "http://www.roblox.com/asset/?id=14753797417"
            sky.SkyboxRt = "http://www.roblox.com/asset/?id=14753799966"
            sky.SkyboxUp = "http://www.roblox.com/asset/?id=14753810287"
        elseif worldSettings.skybox == "Purple Sky" then
            sky.SkyboxBk = "http://www.roblox.com/asset/?id=16553658937"
            sky.SkyboxDn = "http://www.roblox.com/asset/?id=16553660713"
            sky.SkyboxFt = "http://www.roblox.com/asset/?id=16553662144"
            sky.SkyboxLf = "http://www.roblox.com/asset/?id=16553664042"
            sky.SkyboxRt = "http://www.roblox.com/asset/?id=16553665766"
            sky.SkyboxUp = "http://www.roblox.com/asset/?id=16553667750"
        elseif worldSettings.skybox == "Forest" then
            sky.SkyboxBk = "rbxassetid://17428978603"
            sky.SkyboxDn = "rbxassetid://17428977445"
            sky.SkyboxFt = "rbxassetid://17428977114"
            sky.SkyboxLf = "rbxassetid://17428978399"
            sky.SkyboxRt = "rbxassetid://17428976828"
            sky.SkyboxUp = "rbxassetid://17428976669"
        elseif worldSettings.skybox == "City" then
            sky.SkyboxBk = "http://www.roblox.com/asset/?id=10345426"
            sky.SkyboxDn = "http://www.roblox.com/asset/?id=10345444"
            sky.SkyboxFt = "http://www.roblox.com/asset/?id=10345426"
            sky.SkyboxLf = "http://www.roblox.com/asset/?id=10345426"
            sky.SkyboxRt = "http://www.roblox.com/asset/?id=10345426"
            sky.SkyboxUp = "http://www.roblox.com/asset/?id=10345487"
        elseif worldSettings.skybox == "Minecraft" then
            sky.SkyboxBk = "http://www.roblox.com/asset/?id=3754796725"
            sky.SkyboxDn = "http://www.roblox.com/asset/?id=3754833439"
            sky.SkyboxFt = "http://www.roblox.com/asset/?id=3754795891"
            sky.SkyboxLf = "http://www.roblox.com/asset/?id=3754798649"
            sky.SkyboxRt = "http://www.roblox.com/asset/?id=3754799327"
            sky.SkyboxUp = "http://www.roblox.com/asset/?id=3754888841"
        end
    end
end

-- Elementos de la UI - Aimbot
local LeftGroupBox = Tabs.Main:AddLeftGroupbox('Aimbot settings')

LeftGroupBox:AddToggle('AimbotToggle', {
    Text = 'Enable Aimbot',
    Default = false,
    Tooltip = 'Activa o desactiva el aimbot',
    
    Callback = function(Value)
        aimbot.enabled = Value
    end
})

LeftGroupBox:AddToggle('FovAimbot', {
    Text = 'Show FOV Circle',
    Default = false,
    Tooltip = 'Muestra u oculta el círculo FOV',
    
    Callback = function(Value)
        aimbot.showFOV = Value
        fovCircle.Visible = Value
    end
})

LeftGroupBox:AddToggle('SnaplineToggle', {
    Text = 'Show Snapline',
    Default = false,
    Tooltip = 'Muestra u oculta la línea de mira',
    
    Callback = function(Value)
        aimbot.showSnapline = Value
    end
})

LeftGroupBox:AddDivider()

LeftGroupBox:AddSlider('FovCircle',{
    Text = 'FOV Radius',
    Default = 200,
    Min = 0,
    Max = 500,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        aimbot.fovRadius = Value
        fovCircle.Radius = Value
    end
})

LeftGroupBox:AddSlider('FovThickness',{
    Text = 'FOV Thickness',
    Default = 2,
    Min = 1,
    Max = 10,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        aimbot.fovThickness = Value
        fovCircle.Thickness = Value
    end
})

LeftGroupBox:AddSlider('SnaplineThickness',{
    Text = 'Snapline Thickness',
    Default = 2,
    Min = 1,
    Max = 10,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        aimbot.snaplineThickness = Value
    end
})

LeftGroupBox:AddSlider('Smoothing',{
    Text = 'Aimbot Smoothness',
    Default = 50,
    Min = 0,
    Max = 100,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        aimbot.smoothing = Value / 100
    end
})

LeftGroupBox:AddSlider('PredictionX',{
    Text = 'Prediction X Adjustment',
    Default = 100,
    Min = 0,
    Max = 200,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        aimbot.predictionXAdjust = Value / 100
    end
})

LeftGroupBox:AddSlider('PredictionY',{
    Text = 'Prediction Y Adjustment',
    Default = 100,
    Min = 0,
    Max = 200,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        aimbot.predictionYAdjust = Value / 100
    end
})

LeftGroupBox:AddToggle('RainbowFOV', {
    Text = 'Rainbow FOV',
    Default = false,
    Tooltip = 'Activa el modo arcoíris para el FOV',
    
    Callback = function(Value)
        aimbot.fovRainbow = Value
    end
})

LeftGroupBox:AddToggle('RainbowSnapline', {
    Text = 'Rainbow Snapline',
    Default = false,
    Tooltip = 'Activa el modo arcoíris para la línea de mira',
    
    Callback = function(Value)
        aimbot.snaplineRainbow = Value
    end
})

LeftGroupBox:AddLabel('FOV Color'):AddColorPicker('FovColor', {
    Default = Color3.new(1, 1, 1),
    Title = 'Color del FOV',
    Transparency = 0,
    
    Callback = function(Value)
        aimbot.fovColor = Value
        fovCircle.Color = aimbot.fovColor
    end
})

LeftGroupBox:AddLabel('Snapline Color'):AddColorPicker('SnaplineColor', {
    Default = Color3.new(1, 0, 0),
    Title = 'Color de la línea de mira',
    Transparency = 0,
    
    Callback = function(Value)
        aimbot.snaplineColor = Value
    end
})

LeftGroupBox:AddLabel('Aimbot Key'):AddKeyPicker('AimbotKeybind', {
    Default = 'MB2',
    SyncToggleState = false,
    Mode = 'Hold',
    Text = 'Aimbot',
    NoUI = false,
    Callback = function(Value)
        aimbot.aiming = Value
    end,
    ChangedCallback = function(New)
        aimbot.keybind = New
    end
})

-- Elementos de la UI - Combat adicionales
local CombatGroupBox = Tabs.Main:AddRightGroupbox('Combat Settings')

CombatGroupBox:AddToggle('InstantAim', {
    Text = 'Instant Aim',
    Default = false,
    Tooltip = 'Activa el apuntado instantáneo',
    
    Callback = function(Value)
        combatSettings.instantAim = Value
        applyInstantAim()
    end
})

CombatGroupBox:AddToggle('NoRecoil', {
    Text = 'No Recoil',
    Default = false,
    Tooltip = 'Elimina el retroceso de las armas',
    
    Callback = function(Value)
        combatSettings.noRecoil = Value
        applyNoRecoil()
    end
})

CombatGroupBox:AddToggle('AutoReload', {
    Text = 'Auto Reload',
    Default = false,
    Tooltip = 'Recarga automática del arma',
    
    Callback = function(Value)
        combatSettings.autoReload = Value
        toggleAutoReload()
    end
})

-- Elementos de la UI - Sistema de Efectos (movido a Misc)
-- Se moverá más adelante a la sección Misc

-- Elementos de la UI - ESP
local ESPGroupBox = Tabs.Visuals:AddLeftGroupbox('ESP Settings')

ESPGroupBox:AddToggle('ESPEnabled', {
    Text = 'Enable ESP',
    Default = false,
    Tooltip = 'Activa o desactiva el ESP',
    
    Callback = function(Value)
        espSettings.enabled = Value
        
        if Value then
            -- Activar todas las opciones visuales
            Toggles.BoxesEnabled:SetValue(true)
            Toggles.NamesEnabled:SetValue(true)
            Toggles.DistanceEnabled:SetValue(true)
            Toggles.SkeletonEnabled:SetValue(true)
        else
            -- Desactivar todas las opciones visuales
            Toggles.BoxesEnabled:SetValue(false)
            Toggles.NamesEnabled:SetValue(false)
            Toggles.DistanceEnabled:SetValue(false)
            Toggles.SkeletonEnabled:SetValue(false)
            
            -- Ocultar todos los elementos ESP
            for player, esp in pairs(espObjects) do
                hideESP(esp)
            end
            
            -- Ocultar todas las líneas del esqueleto
            for character, lines in pairs(skeletonLines) do
                for _, data in pairs(lines) do
                    data.line.Visible = false
                end
            end
        end
    end
})

ESPGroupBox:AddToggle('HideTeammates', {
    Text = 'Ignore Teammates',
    Default = false,
    Tooltip = 'Oculta a los miembros de tu escuadrón',
    
    Callback = function(Value)
        espSettings.hideteammates = Value
    end
})

ESPGroupBox:AddDivider()

ESPGroupBox:AddToggle('BoxesEnabled', {
    Text = 'ESP Boxes',
    Default = false,
    Tooltip = 'Muestra las cajas alrededor de los jugadores',
    
    Callback = function(Value)
        espSettings.showBoxes = Value
        -- Si el ESP general está desactivado, no mostrar las cajas
        if not espSettings.enabled then
            for player, esp in pairs(espObjects) do
                esp.box.Visible = false
            end
        end
    end
})

ESPGroupBox:AddToggle('FilledBoxes', {
    Text = 'Filled Boxes',
    Default = false,
    Tooltip = 'Rellena las cajas con color',
    
    Callback = function(Value)
        espSettings.filledBoxes = Value
    end
})

ESPGroupBox:AddSlider('BoxTransparency',{
    Text = 'Box Transparency',
    Default = 50,
    Min = 0,
    Max = 100,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        espSettings.boxTransparency = Value / 100
    end
})

ESPGroupBox:AddSlider('BoxThickness',{
    Text = 'Box Thickness',
    Default = 2,
    Min = 1,
    Max = 10,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        espSettings.boxThickness = Value
    end
})

ESPGroupBox:AddLabel('Box Color'):AddColorPicker('BoxColor', {
    Default = Color3.new(1, 1, 1),
    Title = 'Color de las cajas',
    Transparency = 0,
    
    Callback = function(Value)
        espSettings.boxColor = Value
    end
})

ESPGroupBox:AddDivider()

ESPGroupBox:AddToggle('NamesEnabled', {
    Text = 'Player Names',
    Default = false,
    Tooltip = 'Muestra los nombres de los jugadores',
    
    Callback = function(Value)
        espSettings.showNames = Value
        -- Si el ESP general está desactivado, no mostrar los nombres
        if not espSettings.enabled then
            for player, esp in pairs(espObjects) do
                esp.name.Visible = false
            end
        end
    end
})

ESPGroupBox:AddSlider('NameTextSize',{
    Text = 'Name Text Size',
    Default = 14,
    Min = 8,
    Max = 24,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        espSettings.nameTextSize = Value
    end
})

ESPGroupBox:AddToggle('NameOutlines', {
    Text = 'Name Outlines',
    Default = false,
    Tooltip = 'Muestra el contorno de los nombres',
    
    Callback = function(Value)
        espSettings.nameOutlines = Value
    end
})

ESPGroupBox:AddLabel('Name Color'):AddColorPicker('NameColor', {
    Default = Color3.new(1, 1, 1),
    Title = 'Color de los nombres',
    Transparency = 0,
    
    Callback = function(Value)
        espSettings.nameColor = Value
    end
})

ESPGroupBox:AddDivider()

ESPGroupBox:AddToggle('DistanceEnabled', {
    Text = 'Distance Display',
    Default = false,
    Tooltip = 'Muestra la distancia a los jugadores',
    
    Callback = function(Value)
        espSettings.showDistance = Value
        -- Si el ESP general está desactivado, no mostrar la distancia
        if not espSettings.enabled then
            for player, esp in pairs(espObjects) do
                esp.distance.Visible = false
            end
        end
    end
})

ESPGroupBox:AddSlider('DistanceTextSize',{
    Text = 'Distance Text Size',
    Default = 16,
    Min = 8,
    Max = 24,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        espSettings.distanceTextSize = Value
    end
})

ESPGroupBox:AddToggle('DistanceOutlines', {
    Text = 'Distance Outlines',
    Default = false,
    Tooltip = 'Muestra el contorno de la distancia',
    
    Callback = function(Value)
        espSettings.distanceOutlines = Value
    end
})

ESPGroupBox:AddLabel('Distance Color'):AddColorPicker('DistanceColor', {
    Default = Color3.new(1, 1, 1),
    Title = 'Color de la distancia',
    Transparency = 0,
    
    Callback = function(Value)
        espSettings.distanceColor = Value
    end
})

ESPGroupBox:AddDropdown('DistanceUnits', {
    Text = 'Distance Units',
    Default = 'Studs',
    Values = {'Meters', 'Studs', 'Feet', 'Yards', 'Inches', 'Kilometers'},
    
    Callback = function(Value)
        espSettings.distanceUnits = Value
    end
})

ESPGroupBox:AddDivider()

ESPGroupBox:AddToggle('SkeletonEnabled', {
    Text = 'Enable Skeleton',
    Default = false,
    Tooltip = 'Activa o desactiva el esqueleto ESP',
    
    Callback = function(Value)
        skeleton.enabled = Value
        -- Si el ESP general está desactivado o el Skeleton está desactivado, ocultar las líneas
        if not skeleton.enabled or not espSettings.enabled then
            for character, lines in pairs(skeletonLines) do
                for _, data in pairs(lines) do
                    data.line.Visible = false
                end
            end
        end
    end
})

ESPGroupBox:AddSlider('SkeletonThickness',{
    Text = 'Outline Thickness',
    Default = 1.8,
    Min = 0.5,
    Max = 5,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        skeleton.thickness = Value
        -- Actualizar todas las líneas existentes
        for character, lines in pairs(skeletonLines) do
            for _, data in pairs(lines) do
                data.line.Thickness = Value
            end
        end
    end
})

ESPGroupBox:AddLabel('Skeleton Color'):AddColorPicker('SkeletonColor', {
    Default = Color3.new(1, 1, 1),
    Title = 'Color del esqueleto',
    Transparency = 0,
    
    Callback = function(Value)
        skeleton.color = Value
        -- Actualizar todas las líneas existentes
        for character, lines in pairs(skeletonLines) do
            for _, data in pairs(lines) do
                data.line.Color = Value
            end
        end
    end
})

-- ESP Refresh Rate
ESPGroupBox:AddDivider()

ESPGroupBox:AddSlider('RefreshRate',{
    Text = 'ESP Refresh Rate',
    Default = 0,
    Min = 0,
    Max = 1000,
    Rounding = 1,
    Compact = false,
    Tooltip = '0 = Tiempo real | Valores más altos = Actualización más lenta',
    
    Callback = function(Value)
        espSettings.refreshRate = Value
    end
})

-- Elementos de la UI - Inventory Viewer
local InventoryViewerGroupBox = Tabs.Visuals:AddRightGroupbox('Inventory Viewer')

InventoryViewerGroupBox:AddToggle('InventoryViewerEnabled', {
    Text = 'Enable Inventory Viewer',
    Default = false,
    Tooltip = 'Muestra el inventario del jugador más cercano',
    
    Callback = function(Value)
        inventoryViewerSettings.enabled = Value
        toggleInventoryViewer(Value)
    end
})

-- Elementos de la UI - Item ESP
local VehicleESPGroupBox = Tabs.ItemESP:AddLeftGroupbox('Vehicle ESP')

VehicleESPGroupBox:AddToggle('VehicleESPEnabled', {
    Text = 'Enable Vehicle ESP',
    Default = false,
    Tooltip = 'Muestra los vehículos en el mapa',
    
    Callback = function(Value)
        vehicleESP.Enabled = Value
    end
})

VehicleESPGroupBox:AddToggle('VehicleESPShowDistance', {
    Text = 'Show Distance',
    Default = false,
    Tooltip = 'Muestra la distancia a los vehículos',
    
    Callback = function(Value)
        vehicleESP.ShowDistance = Value
    end
})

VehicleESPGroupBox:AddToggle('VehicleESPOutline', {
    Text = 'Outline',
    Default = false,
    Tooltip = 'Muestra el contorno del texto',
    
    Callback = function(Value)
        vehicleESP.Outline = Value
        for _, drawing in pairs(vehicleESP.ActiveDrawings) do
            drawing.Outline = Value
        end
    end
})

VehicleESPGroupBox:AddSlider('VehicleESPSize',{
    Text = 'Text Size',
    Default = 14,
    Min = 8,
    Max = 24,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        vehicleESP.Size = Value
        for _, drawing in pairs(vehicleESP.ActiveDrawings) do
            drawing.Size = Value
        end
    end
})

VehicleESPGroupBox:AddLabel('Vehicle ESP Color'):AddColorPicker('VehicleESPColor', {
    Default = Color3.new(1, 1, 1),
    Title = 'Color del texto',
    Transparency = 0,
    
    Callback = function(Value)
        vehicleESP.Color = Value
        for _, drawing in pairs(vehicleESP.ActiveDrawings) do
            drawing.Color = Value
        end
    end
})

-- Grave ESP
local GraveESPGroupBox = Tabs.ItemESP:AddRightGroupbox('Grave ESP')

GraveESPGroupBox:AddToggle('GraveESPEnabled', {
    Text = 'Enable Grave ESP',
    Default = false,
    Tooltip = 'Muestra las tumbas en el mapa',
    
    Callback = function(Value)
        graveESP.Enabled = Value
    end
})

GraveESPGroupBox:AddToggle('GraveESPShowDistance', {
    Text = 'Show Distance',
    Default = false,
    Tooltip = 'Muestra la distancia a las tumbas',
    
    Callback = function(Value)
        graveESP.ShowDistance = Value
    end
})

GraveESPGroupBox:AddToggle('GraveESPOutline', {
    Text = 'Outline',
    Default = false,
    Tooltip = 'Muestra el contorno del texto',
    
    Callback = function(Value)
        graveESP.Outline = Value
        for _, data in pairs(graveESP.ActiveDrawings) do
            if data.text then
                data.text.Outline = Value
            end
        end
    end
})

GraveESPGroupBox:AddSlider('GraveESPSize',{
    Text = 'Text Size',
    Default = 14,
    Min = 8,
    Max = 24,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        graveESP.Size = Value
        for _, data in pairs(graveESP.ActiveDrawings) do
            if data.text then
                data.text.Size = Value
            end
        end
    end
})

GraveESPGroupBox:AddLabel('Grave ESP Color'):AddColorPicker('GraveESPColor', {
    Default = Color3.fromRGB(255, 255, 255),
    Title = 'Color del texto',
    Transparency = 0,
    
    Callback = function(Value)
        graveESP.Color = Value
        for _, data in pairs(graveESP.ActiveDrawings) do
            if data.text then
                data.text.Color = Value
            end
        end
    end
})

-- Ammo ESP
local AmmoESPGroupBox = Tabs.ItemESP:AddLeftGroupbox('Ammo ESP')

AmmoESPGroupBox:AddToggle('AmmoESPEnabled', {
    Text = 'Enable Ammo ESP',
    Default = false,
    Tooltip = 'Muestra la munición en el mapa',
    
    Callback = function(Value)
        ammoESP.Enabled = Value
        if Value then
            scan_for_ammo_matches()
        else
            clear_all_ammo_esp()
        end
    end
})

AmmoESPGroupBox:AddToggle('AmmoESPShowDistance', {
    Text = 'Show Distance',
    Default = false,
    Tooltip = 'Muestra la distancia a la munición',
    
    Callback = function(Value)
        ammoESP.ShowDistance = Value
    end
})

AmmoESPGroupBox:AddToggle('AmmoESPOutline', {
    Text = 'Outline',
    Default = true,
    Tooltip = 'Muestra el contorno del texto',
    
    Callback = function(Value)
        ammoESP.Outline = Value
        for _, data in pairs(ammoESP.ActiveDrawings) do
            if data.drawing then
                data.drawing.Outline = Value
            end
        end
    end
})

AmmoESPGroupBox:AddSlider('AmmoESPSize',{
    Text = 'Text Size',
    Default = 14,
    Min = 8,
    Max = 24,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        ammoESP.Size = Value
        for _, data in pairs(ammoESP.ActiveDrawings) do
            if data.drawing then
                data.drawing.Size = Value
            end
        end
    end
})

AmmoESPGroupBox:AddLabel('Ammo ESP Color'):AddColorPicker('AmmoESPColor', {
    Default = Color3.fromRGB(255, 255, 255),
    Title = 'Color del texto',
    Transparency = 0,
    
    Callback = function(Value)
        ammoESP.Color = Value
        for _, data in pairs(ammoESP.ActiveDrawings) do
            if data.drawing then
                data.drawing.Color = Value
            end
        end
    end
})

AmmoESPGroupBox:AddSlider('AmmoESPRenderDistance',{
    Text = 'Render Distance',
    Default = 1000,
    Min = 100,
    Max = 2000,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        ammoESP.RenderDistance = Value
    end
})

-- Weapons ESP
local WeaponsESPGroupBox = Tabs.ItemESP:AddRightGroupbox('Weapons ESP')

WeaponsESPGroupBox:AddToggle('WeaponsESPEnabled', {
    Text = 'Enable Weapons ESP',
    Default = false,
    Tooltip = 'Muestra las armas en el mapa',
    
    Callback = function(Value)
        weaponsESP.Enabled = Value
        if Value then
            initialWeaponScan()
        end
    end
})

WeaponsESPGroupBox:AddToggle('WeaponsESPShowDistance', {
    Text = 'Show Distance',
    Default = false,
    Tooltip = 'Muestra la distancia a las armas',
    
    Callback = function(Value)
        weaponsESP.ShowDistance = Value
    end
})

WeaponsESPGroupBox:AddToggle('WeaponsESPOutline', {
    Text = 'Outline',
    Default = false,
    Tooltip = 'Muestra el contorno del texto',
    
    Callback = function(Value)
        weaponsESP.Outline = Value
        for _, data in pairs(weaponsESP.ActiveDrawings) do
            if data.drawing then
                data.drawing.Outline = Value
            end
        end
    end
})

WeaponsESPGroupBox:AddSlider('WeaponsESPSize',{
    Text = 'Text Size',
    Default = 14,
    Min = 8,
    Max = 24,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        weaponsESP.Size = Value
        for _, data in pairs(weaponsESP.ActiveDrawings) do
            if data.drawing then
                data.drawing.Size = Value
            end
        end
    end
})

WeaponsESPGroupBox:AddLabel('Weapons ESP Color'):AddColorPicker('WeaponsESPColor', {
    Default = Color3.fromRGB(255, 255, 255),
    Title = 'Color del texto',
    Transparency = 0,
    
    Callback = function(Value)
        weaponsESP.Color = Value
        for _, data in pairs(weaponsESP.ActiveDrawings) do
            if data.drawing then
                data.drawing.Color = Value
            end
        end
    end
})

WeaponsESPGroupBox:AddToggle('WeaponsESPExcludeMelee', {
    Text = 'Exclude Melee',
    Default = false,
    Tooltip = 'Excluye las armas cuerpo a cuerpo',
    
    Callback = function(Value)
        weaponsESP.ExcludeMelee = Value
    end
})

WeaponsESPGroupBox:AddSlider('WeaponsESPRenderRange',{
    Text = 'Render Range',
    Default = 500,
    Min = 100,
    Max = 1000,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        weaponsESP.RenderRange = Value
    end
})

-- Elementos de la UI - World
local WorldGroupBox = Tabs.World:AddLeftGroupbox('World')

WorldGroupBox:AddToggle('NoGrass', {
    Text = 'No Grass',
    Default = false,
    Tooltip = 'Elimina el césped del juego',
    
    Callback = function(Value)
        worldSettings.noGrass = Value
        applyNoGrass()
    end
})

WorldGroupBox:AddToggle('NoRain', {
    Text = 'No Rain',
    Default = false,
    Tooltip = 'Elimina la lluvia del juego',
    
    Callback = function(Value)
        worldSettings.noRain = Value
        applyNoRain()
    end
})

WorldGroupBox:AddToggle('NoFoliage', {
    Text = 'No Foliage',
    Default = false,
    Tooltip = 'Elimina el follaje del juego',
    
    Callback = function(Value)
        worldSettings.noFoliage = Value
        applyNoFoliage()
    end
})

WorldGroupBox:AddSlider('FoliageTransparency',{
    Text = 'Foliage Transparency',
    Default = 50,
    Min = 0,
    Max = 100,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        worldSettings.foliageTransparency = Value / 100
        if worldSettings.noFoliage then
            applyNoFoliage()
        end
    end
})

WorldGroupBox:AddToggle('NoFog', {
    Text = 'No Fog',
    Default = false,
    Tooltip = 'Elimina la niebla del juego',
    
    Callback = function(Value)
        worldSettings.noFog = Value
        applyNoFog()
    end
})

WorldGroupBox:AddToggle('NoGreenGas', {
    Text = 'No Green Gas',
    Default = false,
    Tooltip = 'Elimina el gas verde del juego',
    
    Callback = function(Value)
        worldSettings.noGreenGas = Value
        applyNoGreenGas()
    end
})

WorldGroupBox:AddToggle('TimeOfDay', {
    Text = 'Time Of Day',
    Default = false,
    Tooltip = 'Cambia la hora del día',
    
    Callback = function(Value)
        worldSettings.timeOfDay = Value
        applyTimeOfDay()
    end
})

WorldGroupBox:AddSlider('TimeValue',{
    Text = 'Time Value',
    Default = 12,
    Min = 0,
    Max = 24,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        worldSettings.timeValue = Value
        applyTimeOfDay()
    end
})

WorldGroupBox:AddDropdown('Skybox', {
    Text = 'Skybox',
    Default = 'Default',
    Values = {'Default', 'Standard', 'Blue Sky', 'Vaporwave', 'Redshift', 'Blaze', 'Among Us', 'Dark Night', 'Bright Pink', 'Purple Sky', 'Galaxy', 'Island', 'Forest', 'City', 'Minecraft'},
    
    Callback = function(Value)
        worldSettings.skybox = Value
        applySkybox()
    end
})

WorldGroupBox:AddLabel('Environment Tint'):AddColorPicker('EnvironmentTint', {
    Default = Color3.new(1, 1, 1),
    Title = 'Tinte del entorno',
    Transparency = 0,
    
    Callback = function(Value)
        game:GetService("Lighting").ColorCorrection.TintColor = Value
    end
})

-- Elementos de la UI - Character (Nueva pestaña)
local CharacterGroupBox = Tabs.Character:AddLeftGroupbox('Character Settings')

CharacterGroupBox:AddToggle('NoImpactDamage', {
    Text = 'No Impact Damage (in Car)',
    Default = false,
    Tooltip = 'Elimina el daño por impacto cuando estás en un vehículo',
    
    Callback = function(Value)
        combatSettings.noImpactDamage = Value
        applyNoImpactDamage()
    end
})

LeftGroupBox:AddToggle('MyToggle', {
    Text = 'Fly',
    Default = false, 
    Tooltip = 'This is a tooltip', 
    Callback = function(Value)
        -- Plataforma que segue o player, toggle com Y, agora com velocidade ajustável
local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")

local platform = Instance.new("Part")
platform.Size = Vector3.new(7, 1, 7)
platform.Anchored = true
platform.CanCollide = true
platform.Color = Color3.fromRGB(150, 255, 150)
platform.Name = "FollowPlatform"
platform.Parent = workspace
platform.Transparency = 0.1

local enabled = false
local runConn

local followSpeed = 0.35 -- 0.1 = devagar, 1 = instantâneo, 0.5 = rápido

function enablePlatform()
    if runConn then runConn:Disconnect() end
    runConn = game:GetService("RunService").RenderStepped:Connect(function(dt)
        if hrp and hrp.Parent then
            local targetPos = hrp.Position - Vector3.new(0, hrp.Size.Y/2 + platform.Size.Y/2 + 0.1, 0)
            platform.Position = platform.Position:Lerp(targetPos, math.clamp(followSpeed, 0, 1))
            platform.Transparency = 0.1
        end
    end)
    platform.Transparency = 0.1
end

function disablePlatform()
    if runConn then runConn:Disconnect() end
    platform.Transparency = 1
end

game:GetService("UserInputService").InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.Y then
        enabled = not enabled
        if enabled then
            enablePlatform()
        else
            disablePlatform()
        end
    end
end)

disablePlatform()
    end
})

LeftGroupBox:AddToggle('MyToggle', {
    Text = 'Speed',
    Default = false, 
    Tooltip = 'This is a tooltip', 
    Callback = function(Value)
        -- Parede vertical colada atrás do player, toggle Y, velocidade ajustável
local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")

local wall = Instance.new("Part")
wall.Size = Vector3.new(4, 10, 0.6) -- largura, altura, profundidade
wall.Anchored = true
wall.CanCollide = true
wall.Color = Color3.fromRGB(255, 100, 100)
wall.Name = "VerticalWall"
wall.Parent = workspace
wall.Transparency = 0.2

local enabled = false
local runConn

local followSpeed = 0.55 -- 0.1 = devagar, 1 = instantâneo

function enableWall()
    if runConn then runConn:Disconnect() end
    runConn = game:GetService("RunService").RenderStepped:Connect(function(dt)
        if hrp and hrp.Parent then
            -- Calcula posição imediatamente atrás do player, alinhado à rotação
            local backOffset = hrp.CFrame.LookVector * -1.5 -- distância colada atrás
            local targetPos = hrp.Position + backOffset
            local targetCFrame = CFrame.new(targetPos, targetPos + hrp.CFrame.LookVector) * CFrame.Angles(0, math.pi, 0)
            wall.CFrame = wall.CFrame:Lerp(targetCFrame, math.clamp(followSpeed, 0, 1))
            wall.Transparency = 0.2
        end
    end)
    wall.Transparency = 0.2
end

function disableWall()
    if runConn then runConn:Disconnect() end
    wall.Transparency = 1
end

game:GetService("UserInputService").InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.B then
        enabled = not enabled
        if enabled then
            enableWall()
        else
            disableWall()
        end
    end
end)

disableWall()
    end
})




-- Elementos de la UI - Misc (Nuevo tab)
-- Camera & Movement
local CameraMovementGroupBox = Tabs.Misc:AddLeftGroupbox('Camera & Movement')

CameraMovementGroupBox:AddToggle('FovChanger', {
    Text = 'FOV Changer',
    Default = false,
    Tooltip = 'Cambia el campo de visión',
    
    Callback = function(Value)
        combatSettings.fovChanger = Value
        applyFovChanger()
    end
})

CameraMovementGroupBox:AddSlider('FovValue',{
    Text = 'FOV Value',
    Default = 90,
    Min = 1,
    Max = 120,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        currentFovValue = Value
        if fovchanger then
            pcall(function()
                local controller = require(LocalPlayer.PlayerScripts:WaitForChild("CameraFovController"))
                controller.ClientFov = currentFovValue
                controller:Enable()
            end)
        end
    end
})

CameraMovementGroupBox:AddToggle('GunOffset', {
    Text = 'Gun Offset',
    Default = false,
    Tooltip = 'Modifica la posición de las armas',
    
    Callback = function(Value)
        combatSettings.gunOffset = Value
        applyGunOffset()
    end
})

CameraMovementGroupBox:AddSlider('OffsetX',{
    Text = 'X Value',
    Default = 2,
    Min = -20,
    Max = 20,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        offsetX = Value
    end
})

CameraMovementGroupBox:AddSlider('OffsetY',{
    Text = 'Y Value',
    Default = 0,
    Min = -20,
    Max = 20,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        offsetY = Value
    end
})

CameraMovementGroupBox:AddSlider('OffsetZ',{
    Text = 'Z Value',
    Default = -2,
    Min = -20,
    Max = 20,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        offsetZ = Value
    end
})

CameraMovementGroupBox:AddToggle('AntiAim', {
    Text = 'Anti Aim',
    Default = false,
    Tooltip = 'Activa el anti-aim',
    
    Callback = function(Value)
        combatSettings.antiAim = Value
        applyAntiAim()
    end
})

CameraMovementGroupBox:AddSlider('AntiAimPitch',{
    Text = 'Pitch Angle Range',
    Default = 90,
    Min = 0,
    Max = 180,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        local radVal = math.rad(Value)
        fakePitchMax = radVal
        fakePitchMin = -radVal
    end
})

CameraMovementGroupBox:AddSlider('AntiAimSpeed',{
    Text = 'Anti-Aim Speed',
    Default = 90,
    Min = 0,
    Max = 180,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        pitchSpeed = Value
    end
})

-- Weapon Chams
local WeaponChamsGroupBox = Tabs.Misc:AddRightGroupbox('Weapon Chams')

WeaponChamsGroupBox:AddToggle('WeaponChams', {
    Text = 'Weapon Chams',
    Default = false,
    Tooltip = 'Aplica chams a las armas',
    
    Callback = function(Value)
        weaponChamsEnabled = Value
        toggleWeaponChams()
    end
})

WeaponChamsGroupBox:AddLabel('Weapon Chams Color'):AddColorPicker('WeaponChamsColor', {
    Default = Color3.new(1, 1, 1),
    Title = 'Color de los chams de arma',
    Transparency = 0,
    
    Callback = function(Value)
        weaponChamsColor = Value
        setWeaponChamsColor(Value)
    end
})

WeaponChamsGroupBox:AddSlider('WeaponChamsTransparency',{
    Text = 'Transparency',
    Default = 50,
    Min = 0,
    Max = 100,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        weaponChamsTransparency = Value / 100
        setWeaponChamsTransparency(weaponChamsTransparency)
    end
})

-- Body Chams (movido a la derecha, debajo de Weapon Chams)
local BodyChamsGroupBox = Tabs.Misc:AddRightGroupbox('Body Chams')

BodyChamsGroupBox:AddToggle('BodyChams', {
    Text = 'Body Chams',
    Default = false,
    Tooltip = 'Aplica chams a los jugadores (ForceField blanco)',
    
    Callback = function(Value)
        bodyChamsEnabled = Value
        toggleBodyChams()
    end
})

-- Controles de Body Chams
BodyChamsGroupBox:AddLabel('Body Chams Color'):AddColorPicker('BodyChamsColor', {
    Default = Color3.new(1, 1, 1),
    Title = 'Color de los chams',
    Transparency = 0,
    
    Callback = function(Value)
        bodyChamsColor = Value
        if bodyChamsEnabled then
            for part, _ in pairs(bodyChamsParts) do
                part.Color = Value
            end
        end
    end
})

BodyChamsGroupBox:AddSlider('BodyChamsTransparency',{
    Text = 'Transparency',
    Default = 0,
    Min = 0,
    Max = 100,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        bodyChamsTransparency = Value / 100
        if bodyChamsEnabled then
            for part, _ in pairs(bodyChamsParts) do
                part.Transparency = bodyChamsTransparency
            end
        end
    end
})

-- Elementos de la UI - Sistema de Efectos (movido desde Main a Misc)
local EffectsGroupBox = Tabs.Misc:AddRightGroupbox('Effects System')

-- Elementos para Hitsounds
EffectsGroupBox:AddToggle('HitsoundEnabled', {
    Text = 'Enable Hitsounds',
    Default = false,
    Tooltip = 'Activa los sonidos de impacto',
    
    Callback = function(Value)
        hitsoundEnabled = Value
        applyHitsound()
    end
})

EffectsGroupBox:AddDropdown('HitsoundName', {
    Text = 'Hitsound',
    Default = 'Neverlose',
    Values = hitsoundNames,
    
    Callback = function(Value)
        currentHitsound = hitsoundOptions[Value]
        applyHitsound()
    end
})

-- Elementos para Headshot Sounds
EffectsGroupBox:AddToggle('HeadshotSoundEnabled', {
    Text = 'Enable Headshot Sound',
    Default = false,
    Tooltip = 'Activa los sonidos de impacto en la cabeza',
    
    Callback = function(Value)
        headshotSoundEnabled = Value
        applyHeadshotSound()
    end
})

EffectsGroupBox:AddDropdown('HeadshotSoundName', {
    Text = 'Headshot Sound',
    Default = 'Default',
    Values = headshotSoundNames,
    
    Callback = function(Value)
        currentHeadshotSound = headshotSoundOptions[Value]
        applyHeadshotSound()
    end
})

-- Elementos para Hit Marker (visual)
EffectsGroupBox:AddToggle('HitMarkerEnabled', {
    Text = 'Enable Hit Marker',
    Default = false,
    Tooltip = 'Muestra un marcador visual al impactar',
    
    Callback = function(Value)
        effects_system.settings.hitMarker.enabled = Value
    end
})

EffectsGroupBox:AddSlider('HitMarkerDuration',{
    Text = 'Hit Marker Duration',
    Default = 0.5,
    Min = 0.1,
    Max = 2,
    Rounding = 1,
    Compact = false,
    
    Callback = function(Value)
        effects_system.settings.hitMarker.duration = Value
    end
})

-- UI Settings
local MenuGroup = Tabs['UI Settings']:AddLeftGroupbox('Menu')
MenuGroup:AddButton('Unload', function() Library:Unload() end)
MenuGroup:AddLabel('Menu bind'):AddKeyPicker('MenuKeybind', { Default = 'End', NoUI = true, Text = 'Menu keybind' })

Library.ToggleKeybind = Options.MenuKeybind

-- Addons
ThemeManager:SetLibrary(Library)
SaveManager:SetLibrary(Library)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({ 'MenuKeybind', 'AimbotKeybind' })

ThemeManager:SetFolder('MyScriptHub')
SaveManager:SetFolder('MyScriptHub/specific-game')

SaveManager:BuildConfigSection(Tabs['UI Settings'])
ThemeManager:ApplyToTab(Tabs['UI Settings'])
SaveManager:LoadAutoloadConfig()

-- Conectar eventos después de crear los elementos
Toggles.AimbotToggle:OnChanged(function()
    aimbot.enabled = Toggles.AimbotToggle.Value
end)

Toggles.FovAimbot:OnChanged(function()
    aimbot.showFOV = Toggles.FovAimbot.Value
    fovCircle.Visible = aimbot.showFOV
end)

Toggles.SnaplineToggle:OnChanged(function()
    aimbot.showSnapline = Toggles.SnaplineToggle.Value
end)

Options.FovCircle:OnChanged(function()
    aimbot.fovRadius = Options.FovCircle.Value
    fovCircle.Radius = aimbot.fovRadius
end)

Options.FovThickness:OnChanged(function()
    aimbot.fovThickness = Options.FovThickness.Value
    fovCircle.Thickness = aimbot.fovThickness
end)

Options.SnaplineThickness:OnChanged(function()
    aimbot.snaplineThickness = Options.SnaplineThickness.Value
end)

Options.Smoothing:OnChanged(function()
    aimbot.smoothing = Options.Smoothing.Value / 100
end)

Options.PredictionX:OnChanged(function()
    aimbot.predictionXAdjust = Options.PredictionX.Value / 100
end)

Options.PredictionY:OnChanged(function()
    aimbot.predictionYAdjust = Options.PredictionY.Value / 100
end)

Toggles.RainbowFOV:OnChanged(function()
    aimbot.fovRainbow = Toggles.RainbowFOV.Value
end)

Toggles.RainbowSnapline:OnChanged(function()
    aimbot.snaplineRainbow = Toggles.RainbowSnapline.Value
end)

Options.FovColor:OnChanged(function()
    aimbot.fovColor = Options.FovColor.Value
    fovCircle.Color = aimbot.fovColor
end)

Options.SnaplineColor:OnChanged(function()
    aimbot.snaplineColor = Options.SnaplineColor.Value
end)

-- Eventos Combat adicionales
Toggles.InstantAim:OnChanged(function()
    combatSettings.instantAim = Toggles.InstantAim.Value
    applyInstantAim()
end)

Toggles.NoRecoil:OnChanged(function()
    combatSettings.noRecoil = Toggles.NoRecoil.Value
    applyNoRecoil()
end)

Toggles.AutoReload:OnChanged(function()
    combatSettings.autoReload = Toggles.AutoReload.Value
    toggleAutoReload()
end)

-- Eventos Misc (nuevo tab)
Toggles.FovChanger:OnChanged(function()
    combatSettings.fovChanger = Toggles.FovChanger.Value
    applyFovChanger()
end)

Options.FovValue:OnChanged(function()
    currentFovValue = Options.FovValue.Value
    if fovchanger then
        pcall(function()
            local controller = require(LocalPlayer.PlayerScripts:WaitForChild("CameraFovController"))
            controller.ClientFov = currentFovValue
            controller:Enable()
        end)
    end
end)

Toggles.GunOffset:OnChanged(function()
    combatSettings.gunOffset = Toggles.GunOffset.Value
    applyGunOffset()
end)

Options.OffsetX:OnChanged(function()
    offsetX = Options.OffsetX.Value
end)

Options.OffsetY:OnChanged(function()
    offsetY = Options.OffsetY.Value
end)

Options.OffsetZ:OnChanged(function()
    offsetZ = Options.OffsetZ.Value
end)

Toggles.AntiAim:OnChanged(function()
    combatSettings.antiAim = Toggles.AntiAim.Value
    applyAntiAim()
end)

Options.AntiAimPitch:OnChanged(function()
    local radVal = math.rad(Options.AntiAimPitch.Value)
    fakePitchMax = radVal
    fakePitchMin = -radVal
end)

Options.AntiAimSpeed:OnChanged(function()
    pitchSpeed = Options.AntiAimSpeed.Value
end)

-- Eventos Weapon Chams
Toggles.WeaponChams:OnChanged(function()
    weaponChamsEnabled = Toggles.WeaponChams.Value
    toggleWeaponChams()
end)

Options.WeaponChamsColor:OnChanged(function()
    weaponChamsColor = Options.WeaponChamsColor.Value
    setWeaponChamsColor(Options.WeaponChamsColor.Value)
end)

Options.WeaponChamsTransparency:OnChanged(function()
    weaponChamsTransparency = Options.WeaponChamsTransparency.Value / 100
    setWeaponChamsTransparency(weaponChamsTransparency)
end)

-- Eventos Body Chams
Toggles.BodyChams:OnChanged(function()
    bodyChamsEnabled = Toggles.BodyChams.Value
    toggleBodyChams()
end)

-- Eventos del Sistema de Efectos
Toggles.HitsoundEnabled:OnChanged(function()
    hitsoundEnabled = Toggles.HitsoundEnabled.Value
    applyHitsound()
end)

Options.HitsoundName:OnChanged(function(Value)
    currentHitsound = hitsoundOptions[Value]
    applyHitsound()
end)

Toggles.HeadshotSoundEnabled:OnChanged(function()
    headshotSoundEnabled = Toggles.HeadshotSoundEnabled.Value
    applyHeadshotSound()
end)

Options.HeadshotSoundName:OnChanged(function(Value)
    currentHeadshotSound = headshotSoundOptions[Value]
    applyHeadshotSound()
end)

Toggles.HitMarkerEnabled:OnChanged(function()
    effects_system.settings.hitMarker.enabled = Toggles.HitMarkerEnabled.Value
end)

Options.HitMarkerDuration:OnChanged(function(Value)
    effects_system.settings.hitMarker.duration = Value
end)

-- Eventos ESP
Toggles.ESPEnabled:OnChanged(function()
    espSettings.enabled = Toggles.ESPEnabled.Value
    
    if espSettings.enabled then
        -- Activar todas las opciones visuales
        Toggles.BoxesEnabled:SetValue(true)
        Toggles.NamesEnabled:SetValue(true)
        Toggles.DistanceEnabled:SetValue(true)
        Toggles.SkeletonEnabled:SetValue(true)
    else
        -- Desactivar todas las opciones visuales
        Toggles.BoxesEnabled:SetValue(false)
        Toggles.NamesEnabled:SetValue(false)
        Toggles.DistanceEnabled:SetValue(false)
        Toggles.SkeletonEnabled:SetValue(false)
        
        -- Ocultar todos los elementos ESP
        for player, esp in pairs(espObjects) do
            hideESP(esp)
        end
        
        -- Ocultar todas las líneas del esqueleto
        for character, lines in pairs(skeletonLines) do
            for _, data in pairs(lines) do
                data.line.Visible = false
            end
        end
    end
end)

Toggles.HideTeammates:OnChanged(function()
    espSettings.hideteammates = Toggles.HideTeammates.Value
end)

Toggles.BoxesEnabled:OnChanged(function()
    espSettings.showBoxes = Toggles.BoxesEnabled.Value
    -- Si el ESP general está desactivado, no mostrar las cajas
    if not espSettings.enabled then
        for player, esp in pairs(espObjects) do
            esp.box.Visible = false
        end
    end
end)

Toggles.FilledBoxes:OnChanged(function()
    espSettings.filledBoxes = Toggles.FilledBoxes.Value
end)

Options.BoxTransparency:OnChanged(function()
    espSettings.boxTransparency = Options.BoxTransparency.Value / 100
end)

Options.BoxThickness:OnChanged(function()
    espSettings.boxThickness = Options.BoxThickness.Value
end)

Options.BoxColor:OnChanged(function()
    espSettings.boxColor = Options.BoxColor.Value
end)

Toggles.NamesEnabled:OnChanged(function()
    espSettings.showNames = Toggles.NamesEnabled.Value
    -- Si el ESP general está desactivado, no mostrar los nombres
    if not espSettings.enabled then
        for player, esp in pairs(espObjects) do
            esp.name.Visible = false
        end
    end
end)

Options.NameTextSize:OnChanged(function()
    espSettings.nameTextSize = Options.NameTextSize.Value
end)

Toggles.NameOutlines:OnChanged(function()
    espSettings.nameOutlines = Toggles.NameOutlines.Value
end)

Options.NameColor:OnChanged(function()
    espSettings.nameColor = Options.NameColor.Value
end)

Toggles.DistanceEnabled:OnChanged(function()
    espSettings.showDistance = Toggles.DistanceEnabled.Value
    -- Si el ESP general está desactivado, no mostrar la distancia
    if not espSettings.enabled then
        for player, esp in pairs(espObjects) do
            esp.distance.Visible = false
        end
    end
end)

Options.DistanceTextSize:OnChanged(function()
    espSettings.distanceTextSize = Options.DistanceTextSize.Value
end)

Toggles.DistanceOutlines:OnChanged(function()
    espSettings.distanceOutlines = Toggles.DistanceOutlines.Value
end)

Options.DistanceColor:OnChanged(function()
    espSettings.distanceColor = Options.DistanceColor.Value
end)

Options.DistanceUnits:OnChanged(function()
    espSettings.distanceUnits = Options.DistanceUnits.Value
end)

-- Eventos del Skeleton ESP
Toggles.SkeletonEnabled:OnChanged(function()
    skeleton.enabled = Toggles.SkeletonEnabled.Value
    -- Si el ESP general está desactivado o el Skeleton está desactivado, ocultar las líneas
    if not skeleton.enabled or not espSettings.enabled then
        for character, lines in pairs(skeletonLines) do
            for _, data in pairs(lines) do
                data.line.Visible = false
            end
        end
    end
end)

Options.SkeletonThickness:OnChanged(function()
    skeleton.thickness = Options.SkeletonThickness.Value
    -- Actualizar todas las líneas existentes
    for character, lines in pairs(skeletonLines) do
        for _, data in pairs(lines) do
            data.line.Thickness = skeleton.thickness
        end
    end
end)

Options.SkeletonColor:OnChanged(function()
    skeleton.color = Options.SkeletonColor.Value
    -- Actualizar todas las líneas existentes
    for character, lines in pairs(skeletonLines) do
        for _, data in pairs(lines) do
            data.line.Color = skeleton.color
        end
    end
end)

-- Evento para el Refresh Rate
Options.RefreshRate:OnChanged(function()
    espSettings.refreshRate = Options.RefreshRate.Value
end)

-- Eventos Item ESP - Vehicle ESP
Toggles.VehicleESPEnabled:OnChanged(function()
    vehicleESP.Enabled = Toggles.VehicleESPEnabled.Value
end)

Toggles.VehicleESPShowDistance:OnChanged(function()
    vehicleESP.ShowDistance = Toggles.VehicleESPShowDistance.Value
end)

Toggles.VehicleESPOutline:OnChanged(function()
    vehicleESP.Outline = Toggles.VehicleESPOutline.Value
    for _, drawing in pairs(vehicleESP.ActiveDrawings) do
        drawing.Outline = vehicleESP.Outline
    end
end)

Options.VehicleESPSize:OnChanged(function()
    vehicleESP.Size = Options.VehicleESPSize.Value
    for _, drawing in pairs(vehicleESP.ActiveDrawings) do
        drawing.Size = vehicleESP.Size
    end
end)

Options.VehicleESPColor:OnChanged(function()
    vehicleESP.Color = Options.VehicleESPColor.Value
    for _, drawing in pairs(vehicleESP.ActiveDrawings) do
        drawing.Color = vehicleESP.Color
    end
end)

-- Eventos Item ESP - Grave ESP
Toggles.GraveESPEnabled:OnChanged(function()
    graveESP.Enabled = Toggles.GraveESPEnabled.Value
end)

Toggles.GraveESPShowDistance:OnChanged(function()
    graveESP.ShowDistance = Toggles.GraveESPShowDistance.Value
end)

Toggles.GraveESPOutline:OnChanged(function()
    graveESP.Outline = Toggles.GraveESPOutline.Value
    for _, data in pairs(graveESP.ActiveDrawings) do
        if data.text then
            data.text.Outline = graveESP.Outline
        end
    end
end)

Options.GraveESPSize:OnChanged(function()
    graveESP.Size = Options.GraveESPSize.Value
    for _, data in pairs(graveESP.ActiveDrawings) do
        if data.text then
            data.text.Size = graveESP.Size
        end
    end
end)

Options.GraveESPColor:OnChanged(function()
    graveESP.Color = Options.GraveESPColor.Value
    for _, data in pairs(graveESP.ActiveDrawings) do
        if data.text then
            data.text.Color = graveESP.Color
        end
    end
end)

-- Eventos Item ESP - Ammo ESP
Toggles.AmmoESPEnabled:OnChanged(function()
    ammoESP.Enabled = Toggles.AmmoESPEnabled.Value
    if ammoESP.Enabled then
        scan_for_ammo_matches()
    else
        clear_all_ammo_esp()
    end
end)

Toggles.AmmoESPShowDistance:OnChanged(function()
    ammoESP.ShowDistance = Toggles.AmmoESPShowDistance.Value
end)

Toggles.AmmoESPOutline:OnChanged(function()
    ammoESP.Outline = Toggles.AmmoESPOutline.Value
    for _, data in pairs(ammoESP.ActiveDrawings) do
        if data.drawing then
            data.drawing.Outline = ammoESP.Outline
        end
    end
end)

Options.AmmoESPSize:OnChanged(function()
    ammoESP.Size = Options.AmmoESPSize.Value
    for _, data in pairs(ammoESP.ActiveDrawings) do
        if data.drawing then
            data.drawing.Size = ammoESP.Size
        end
    end
end)

Options.AmmoESPColor:OnChanged(function()
    ammoESP.Color = Options.AmmoESPColor.Value
    for _, data in pairs(ammoESP.ActiveDrawings) do
        if data.drawing then
            data.drawing.Color = ammoESP.Color
        end
    end
end)

Options.AmmoESPRenderDistance:OnChanged(function()
    ammoESP.RenderDistance = Options.AmmoESPRenderDistance.Value
end)

-- Eventos Item ESP - Weapons ESP
Toggles.WeaponsESPEnabled:OnChanged(function()
    weaponsESP.Enabled = Toggles.WeaponsESPEnabled.Value
    if weaponsESP.Enabled then
        initialWeaponScan()
    end
end)

Toggles.WeaponsESPShowDistance:OnChanged(function()
    weaponsESP.ShowDistance = Toggles.WeaponsESPShowDistance.Value
end)

Toggles.WeaponsESPOutline:OnChanged(function()
    weaponsESP.Outline = Toggles.WeaponsESPOutline.Value
    for _, data in pairs(weaponsESP.ActiveDrawings) do
        if data.drawing then
            data.drawing.Outline = weaponsESP.Outline
        end
    end
end)

Options.WeaponsESPSize:OnChanged(function()
    weaponsESP.Size = Options.WeaponsESPSize.Value
    for _, data in pairs(weaponsESP.ActiveDrawings) do
        if data.drawing then
            data.drawing.Size = weaponsESP.Size
        end
    end
end)

Options.WeaponsESPColor:OnChanged(function()
    weaponsESP.Color = Options.WeaponsESPColor.Value
    for _, data in pairs(weaponsESP.ActiveDrawings) do
        if data.drawing then
            data.drawing.Color = weaponsESP.Color
        end
    end
end)

Toggles.WeaponsESPExcludeMelee:OnChanged(function()
    weaponsESP.ExcludeMelee = Toggles.WeaponsESPExcludeMelee.Value
end)

Options.WeaponsESPRenderRange:OnChanged(function()
    weaponsESP.RenderRange = Options.WeaponsESPRenderRange.Value
end)

-- Eventos Inventory Viewer
Toggles.InventoryViewerEnabled:OnChanged(function()
    inventoryViewerSettings.enabled = Toggles.InventoryViewerEnabled.Value
    toggleInventoryViewer(Toggles.InventoryViewerEnabled.Value)
end)

-- Eventos World
Toggles.NoGrass:OnChanged(function()
    worldSettings.noGrass = Toggles.NoGrass.Value
    applyNoGrass()
end)

Toggles.NoRain:OnChanged(function()
    worldSettings.noRain = Toggles.NoRain.Value
    applyNoRain()
end)

Toggles.NoFoliage:OnChanged(function()
    worldSettings.noFoliage = Toggles.NoFoliage.Value
    applyNoFoliage()
end)

Options.FoliageTransparency:OnChanged(function()
    worldSettings.foliageTransparency = Options.FoliageTransparency.Value / 100
    if worldSettings.noFoliage then
        applyNoFoliage()
    end
end)

Toggles.NoFog:OnChanged(function()
    worldSettings.noFog = Toggles.NoFog.Value
    applyNoFog()
end)

Toggles.NoGreenGas:OnChanged(function()
    worldSettings.noGreenGas = Toggles.NoGreenGas.Value
    applyNoGreenGas()
end)

Toggles.TimeOfDay:OnChanged(function()
    worldSettings.timeOfDay = Toggles.TimeOfDay.Value
    applyTimeOfDay()
end)

Options.TimeValue:OnChanged(function()
    worldSettings.timeValue = Options.TimeValue.Value
    applyTimeOfDay()
end)

Options.Skybox:OnChanged(function()
    worldSettings.skybox = Options.Skybox.Value
    applySkybox()
end)

Options.EnvironmentTint:OnChanged(function()
    game:GetService("Lighting").ColorCorrection.TintColor = Options.EnvironmentTint.Value
end)

-- Inicializar ESP para jugadores existentes
for _, player in ipairs(Players:GetPlayers()) do
    setupPlayerESP(player)
end

-- Conectar eventos de jugadores
Players.PlayerAdded:Connect(setupPlayerESP)

Players.PlayerRemoving:Connect(function(player)
    if espObjects[player] then
        clearESP(espObjects[player])
        espObjects[player] = nil
    end
end)

-- Eventos de entrada para el aimbot
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    
    if input.UserInputType == aimbot.keybind then
        aimbot.aiming = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == aimbot.keybind then
        aimbot.aiming = false
    end
end)

-- Función principal del anti-aim
RunService:BindToRenderStep("CameraAngleSpoof", Enum.RenderPriority.Character.Value - 1, function(dt)
    if not antiAimEnabled then return end

    local fakePitch = fakePitchMin + math.sin(tick() * pitchSpeed) * (fakePitchMax - fakePitchMin)

    currentYaw = currentYaw + fakeYawSpeed * dt
    if currentYaw > math.pi * 2 then
        currentYaw = currentYaw - math.pi * 2
    end

    local CameraAngleEvent = ReplicatedStorage:WaitForChild("CameraAngleEvent")
    CameraAngleEvent:FireServer(fakePitch, currentYaw)
end)

-- Conexiones para Item ESP
-- Vehicle ESP
RunService.RenderStepped:Connect(updateVehicles)
RunService.Heartbeat:Connect(vehicleScanner)
Workspace.ChildAdded:Connect(function(child)
    if child:IsA("Model") and child:FindFirstChild("Chassis") then
        createVehicleDrawing(child.Chassis)
    end
end)

-- Grave ESP
RunService.RenderStepped:Connect(updateGraveESP)

-- Ammo ESP
RunService.RenderStepped:Connect(function()
    local camera = Workspace.CurrentCamera
    if not ammoESP.Enabled or not camera then return end

    local camPos = camera.CFrame.Position

    for source_part, data in pairs(ammoESP.ActiveDrawings) do
        local drawing_text = data.drawing

        if not source_part.Parent then
            drawing_text:Remove()
            ammoESP.ActiveDrawings[source_part] = nil
        else
            local dist = (source_part.Position - camPos).Magnitude

            if dist > ammoESP.RenderDistance then
                drawing_text.Visible = false
            else
                apply_ammo_settings_to_drawing(drawing_text)

                if ammoESP.ShowDistance then
                    drawing_text.Text = data.label .. " [" .. tostring(math.floor(dist)) .. "m]"
                else
                    drawing_text.Text = data.label
                end

                local screen_pos, on_screen = camera:WorldToViewportPoint(source_part.Position)
                if on_screen then
                    drawing_text.Position = Vector2.new(screen_pos.X, screen_pos.Y)
                    drawing_text.Visible = true
                else
                    drawing_text.Visible = false
                end
            end
        end
    end
end)

-- Conexiones seguras para Ammo ESP
local function setupAmmoESPConnections()
    -- Acceso seguro a world_assets
    local worldAssets = Workspace:FindFirstChild("world_assets")
    if worldAssets and worldAssets:FindFirstChild("StaticObjects") then
        local staticObjects = worldAssets.StaticObjects
        if staticObjects:FindFirstChild("Misc") then
            staticObjects.Misc.ChildAdded:Connect(function()
                task.defer(scan_for_ammo_matches)
            end)
        end
    end

    local ammoFolder = game:GetService("ReplicatedStorage"):FindFirstChild("Assets")
    if ammoFolder and ammoFolder:FindFirstChild("ViewModels") then
        local viewModels = ammoFolder.ViewModels
        if viewModels:FindFirstChild("Combat") then
            viewModels.Combat.ChildAdded:Connect(function()
                task.defer(scan_for_ammo_matches)
            end)
        end
    end
end

setupAmmoESPConnections()
RunService.Heartbeat:Connect(cleanup_removed_ammo)

-- Weapons ESP
RunService.RenderStepped:Connect(updateWeaponsESP)

-- Conexión segura para Weapons ESP
local function setupWeaponsESPConnections()
    -- Acceso seguro a world_assets
    local worldAssets = Workspace:FindFirstChild("world_assets")
    if worldAssets and worldAssets:FindFirstChild("StaticObjects") then
        local staticObjects = worldAssets.StaticObjects
        if staticObjects:FindFirstChild("Misc") then
            staticObjects.Misc.ChildAdded:Connect(function(newModel)
                wait(0.1)
                checkAndCreateWeaponESPForModel(newModel)
            end)
        end
    end
end

setupWeaponsESPConnections()

-- Nuevo Aimbot con corrección para mousemoverel
RunService.RenderStepped:Connect(function()
    if Library.Unloaded then return end
    
    local t = tick()
    local center2d = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    
    -- Actualizar círculo FOV
    fovCircle.Position = center2d
    fovCircle.Visible = aimbot.showFOV
    fovCircle.Color = aimbot.fovRainbow and Color3.fromHSV(t % 5 / 5, 1, 1) or aimbot.fovColor
    
    -- Obtener cabezas de jugadores
    local heads = {}
    for _, mdl in ipairs(Workspace.Characters:GetChildren()) do
        if mdl:IsA("Model") and mdl:FindFirstChild("Head") then
            table.insert(heads, mdl.Head)
        end
    end
    
    -- Excluir cabeza del jugador local
    local myHead = LocalPlayer.Character and (LocalPlayer.Character:FindFirstChild("ServerColliderHead") or LocalPlayer.Character:FindFirstChild("Head"))
    local ignoreIdx, ignoreDist = nil, math.huge
    if myHead then
        for i, head in ipairs(heads) do
            local d = (head.Position - myHead.Position).Magnitude
            if d < ignoreDist then
                ignoreDist = d
                ignoreIdx = i
            end
        end
    end
    
    -- Encontrar objetivo más cercano dentro del FOV
    local targetHead, target2d, minDist = nil, nil, math.huge
    for i, head in ipairs(heads) do
        if i ~= ignoreIdx then
            local pos2d, onScreen = worldToScreen(head.Position)
            if onScreen then
                local dist = (pos2d - center2d).Magnitude
                if dist <= aimbot.fovRadius and dist < minDist then
                    minDist = dist
                    targetHead = head
                    target2d = pos2d
                end
            end
        end
    end
    
    -- Actualizar snapline
    if aimbot.snapline then
        aimbot.snapline:Remove()
        aimbot.snapline = nil
    end
    
    if aimbot.showSnapline and targetHead then
        aimbot.snapline = Drawing.new("Line")
        aimbot.snapline.From = center2d
        aimbot.snapline.To = target2d
        aimbot.snapline.Color = aimbot.snaplineRainbow and Color3.fromHSV(t % 5 / 5, 1, 1) or aimbot.snaplineColor
        aimbot.snapline.Thickness = aimbot.snaplineThickness
        aimbot.snapline.Transparency = 1
        aimbot.snapline.Visible = true
    end
    
    -- Lógica de apuntado con corrección para mousemoverel
    if aimbot.aiming and aimbot.enabled and targetHead then
        local camPos = Camera.CFrame.Position
        local headPos = targetHead.Position
        local horizDist = (Vector3.new(headPos.X, camPos.Y, headPos.Z) - camPos).Magnitude
        local speed = getEquippedBulletSpeed()
        local drop = calculateDrop(horizDist, speed) * 1.4 * aimbot.predictionYAdjust
        
        -- Calcular predicción de movimiento
        local predictedOffset = Vector3.zero
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("ServerColliderHead") then
                local velocity = plr.Character.ServerColliderHead.Velocity
                local predictedTime = horizDist / speed
                local prediction = velocity * predictedTime * aimbot.predictionXAdjust
                if (plr.Character.ServerColliderHead.Position - headPos).Magnitude < 10 then
                    predictedOffset = prediction
                    break
                end
            end
        end
        
        -- Calcular posición compensada
        local compensatedPos = headPos + Vector3.new(0, drop * 1.5, 0) + predictedOffset
        local comp2d, onS = worldToScreen(compensatedPos)
        
        -- Mover mouse hacia el objetivo usando la función alternativa
        if onS then
            local delta = comp2d - center2d
            -- Usar la función mousemoverel (ya sea la original o nuestra alternativa)
            mousemoverel(delta.X * aimbot.smoothing, delta.Y * aimbot.smoothing)
        end
    end
    
    -- Actualizar Inventory Viewer si está habilitado
    if inventoryViewerSettings.enabled then
        updateInventoryViewer()
    end
    
    -- Actualizar ESP si está habilitado
    if espSettings.enabled then
        local currentTime = tick()
        
        -- Si refreshRate es 0, actualizar cada frame
        -- Si refreshRate > 0, actualizar solo si ha pasado el tiempo suficiente
        local shouldUpdate = espSettings.refreshRate == 0 or (currentTime - lastEspUpdate > espSettings.refreshRate / 1000)
        
        if shouldUpdate then
            lastEspUpdate = currentTime
            
            for player, esp in pairs(espObjects) do
                if not player or not player.Character or not player:IsDescendantOf(Players) then
                    hideESP(esp)
                    continue
                end

                -- Verificar si es compañero de equipo y está activada la opción de ocultar compañeros
                if espSettings.hideteammates and squadMembers[player.Name] then
                    hideESP(esp)
                    continue
                end

                local character = player.Character
                -- Intentar encontrar ServerColliderHead primero, si no existe, usar Head normal
                local head = character:FindFirstChild("ServerColliderHead") or character:FindFirstChild("Head")
                local hrp = character:FindFirstChild("HumanoidRootPart")

                if not head or not hrp then
                    hideESP(esp)
                    continue
                end

                local headPos, onScreen1 = Camera:WorldToViewportPoint(head.Position)
                local footPos, onScreen2 = Camera:WorldToViewportPoint(hrp.Position)

                if not (onScreen1 and onScreen2) then
                    hideESP(esp)
                    continue
                end

                -- Calcular dimensiones de la caja
                local height = math.abs(footPos.Y - headPos.Y)
                local width = height / 2
                local x = footPos.X - width / 2
                local y = headPos.Y

                -- Actualizar caja
                esp.box.Size = Vector2.new(width, height)
                esp.box.Position = Vector2.new(x, y)
                esp.box.Color = espSettings.boxColor
                esp.box.Filled = espSettings.filledBoxes
                esp.box.Thickness = espSettings.boxThickness
                esp.box.Transparency = espSettings.boxTransparency
                esp.box.Visible = espSettings.showBoxes

                -- Actualizar nombre
                esp.name.Text = player.Name
                esp.name.Position = Vector2.new(x + width / 2, y - 16)
                esp.name.Size = espSettings.nameTextSize
                esp.name.Center = true
                esp.name.Outline = espSettings.nameOutlines
                esp.name.Color = espSettings.nameColor
                esp.name.Visible = espSettings.showNames

                -- Actualizar distancia
                local dist = (Camera.CFrame.Position - hrp.Position).Magnitude
                local converted = convertDistance(dist, espSettings.distanceUnits)
                esp.distance.Text = string.format("%.1f %s", converted, espSettings.distanceUnits)
                esp.distance.Position = Vector2.new(x + width / 2, y + height + 2)
                esp.distance.Size = espSettings.distanceTextSize
                esp.distance.Center = true
                esp.distance.Outline = espSettings.distanceOutlines
                esp.distance.Color = espSettings.distanceColor
                esp.distance.Visible = espSettings.showDistance
            end
        end
        
        -- Actualizar Skeleton ESP (siempre se actualiza con el mismo refresh rate)
        for character, lines in pairs(skeletonLines) do
            if not character:IsDescendantOf(CharactersFolder) then
                clearSkeleton(lines)
                skeletonLines[character] = nil
            else
                updateSkeleton(character, lines)
            end
        end

        for _, character in ipairs(CharactersFolder:GetChildren()) do
            if not skeletonLines[character] then
                skeletonLines[character] = createSkeleton(character)
            end
        end
    end
    
    -- Actualizar miembros del escuadrón (con caché)
    updateSquadMembers()
end)

-- Conectar a eventos de impacto
local hitMarkerRemote = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("HitMarker")
hitMarkerRemote.OnClientEvent:Connect(function(isHeadshot)
    -- Solo mostrar el hitmarker visual, no reproducir sonidos
    effects_system.showHitMarker()
end)

-- bypass
    if not LPH_OBFUSCATED then
        function LPH_NO_VIRTUALIZE(f)
            return f;
        end;
        function LPH_JIT(...)
            return ...
        end;
        function LPH_JIT_MAX(...)
            return ...
        end;
        function LPH_NO_UPVALUES(f)
            return function(...)
                return f(...);
            end;
        end;
        function LPH_ENCSTR(...)
            return ...
        end;
        function LPH_ENCNUM(...)
            return ...
        end;
        function LPH_CRASH()
            return print(debug.traceback());
        end;
    end;
    --
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local Module = require(ReplicatedStorage:WaitForChild("EmberSharedLibrary"):WaitForChild("EmberShared"):WaitForChild("Datastructures"):WaitForChild("Encryption"):WaitForChild("AES"))

    if not isfunctionhooked(Module.util.padByteString) then
        local old;
        
        old = hookfunction(Module.util.padByteString, LPH_NO_UPVALUES(function(val, ...)
            local vals = tostring(val):split("\n")
            
            if #vals > 2 then
                local spoofed = vals[#vals]
                return old("\n"..spoofed, ...)
            end

            return old(val, ...)
        end))

        --warn("Successfully bypassed.")
    else
        warn("Already DogStire Lite")
    end

-- Watermark
Library:SetWatermarkVisibility(true)
local FrameTimer = tick()
local FrameCounter = 0;
local FPS = 60;

local WatermarkConnection = game:GetService('RunService').RenderStepped:Connect(function()
    FrameCounter += 1;
    if (tick() - FrameTimer) >= 1 then
        FPS = FrameCounter;
        FrameTimer = tick();
        FrameCounter = 0;
    end;
    Library:SetWatermark(('Lunar.Hook | %s fps | %s ms'):format(
        math.floor(FPS),
        math.floor(game:GetService('Stats').Network.ServerStatsItem['Data Ping']:GetValue())
    ));
end);

Library:OnUnload(function()
    -- Desconectar todas las conexiones al descargar
    for _, connection in pairs(connections) do
        if connection then
            connection:Disconnect()
        end
    end
    connections = {}
    
    -- Limpiar conexiones del follaje
    for _, connection in pairs(foliageConnections) do
        connection:Disconnect()
    end
    foliageConnections = {}
    
    -- Desconectar el evento de añadido de follaje
    if connections.foliageAdded then
        connections.foliageAdded:Disconnect()
        connections.foliageAdded = nil
    end
    
    -- Desconectar conexión de recarga automática
    if autoReloadConnection then
        autoReloadConnection:Disconnect()
        autoReloadConnection = nil
    end
    
    -- Restaurar funciones originales
    for i, originalFunc in pairs(originalFunctions) do
        if type(i) == 'string' and string.find(i, 'GunAim') then
            for _, v in next, getgc(true) do
                if type(v) == 'table' and v[i] then
                    v[i] = originalFunc
                end
            end
        end
    end
    originalFunctions = {}
    
    -- Restaurar valores originales para World
    if worldSettings.noFoliage then
        for part, origTransparency in pairs(originalFoliageTransparency) do
            if part and part.Parent then
                part.Transparency = origTransparency
            end
        end
    end
    
    if worldSettings.noFog then
        if Lighting.Atmosphere then
            Lighting.Atmosphere.Density = originalAtmosphereDensity
        end
    end
    
    if worldSettings.skybox ~= "Default" then
        applySkybox()
    end
    
    -- Restaurar tiempo original
    Lighting.ClockTime = originalTimeOfDay
    
    -- Restaurar FOV predeterminado
    pcall(function()
        local controller = require(LocalPlayer.PlayerScripts:WaitForChild("CameraFovController"))
        controller.ClientFov = defaultFov
        controller:Enable()
    end)
    
    -- Restaurar offsets originales
    restoreOffsets()
    
    -- Restaurar césped original
    local terrain = Workspace:FindFirstChild("Terrain")
    if terrain then
        local CLEAR_RADIUS = 512
        local VERTICAL_EXT = 128
        
        local function restoreGrassAt(position)
            local minV = position - Vector3.new(CLEAR_RADIUS, VERTICAL_EXT, CLEAR_RADIUS)
            local maxV = position + Vector3.new(CLEAR_RADIUS, VERTICAL_EXT, CLEAR_RADIUS)
            local region = Region3.new(minV, maxV):ExpandToGrid(4)
            terrain:ReplaceMaterial(region, 4, Enum.Material.LeafyGrass, Enum.Material.Grass)
        end
        
        if Workspace.CurrentCamera then
            restoreGrassAt(Workspace.CurrentCamera.CFrame.Position)
        end
    end
    
    -- Restaurar lluvia original si estaba habilitada
    if originalRainEnabled then
        local rainEnabled = Workspace:FindFirstChild("RainEnabled")
        if rainEnabled then
            rainEnabled.Value = true
        end
    end
    
    -- Restaurar gas verde si estaba oculto
    if connections.gasModel and connections.gasChildren then
        if game.Workspace:FindFirstChild("world_assets") then
            connections.gasModel.Parent = game.Workspace:FindFirstChild("world_assets").StaticObjects.Misc.TOSIC
            for i,v in pairs(connections.gasChildren) do
                v.Parent = game.Workspace:FindFirstChild("world_assets").StaticObjects.Misc.TOSIC
            end
        end
    end
    
    -- Limpiar objetos del Inventory Viewer
    inventoryBox:Remove()
    inventoryBorder:Remove()
    inventoryTitle:Remove()
    for _, t in ipairs(inventoryTextLines) do
        t:Remove()
    end
    
    -- Restaurar armas originales completamente
    for weapon, _ in pairs(modifiedWeapons) do
        restoreWeapon(weapon)
    end
    modifiedWeapons = {}
    
    -- Desconectar conexión de Weapon Chams
    if weaponChamsConnection then
        weaponChamsConnection:Disconnect()
        weaponChamsConnection = nil
    end
    
    -- Restaurar Body Chams
    for part, _ in pairs(bodyChamsParts) do
        restoreBodyChams(part)
    end
    bodyChamsParts = {}
    
    -- Desconectar conexión de Body Chams
    if bodyChamsConnection then
        bodyChamsConnection:Disconnect()
        bodyChamsConnection = nil
    end
    
    -- Limpiar objetos de Item ESP
    for _, drawing in pairs(vehicleESP.ActiveDrawings) do
        drawing:Remove()
    end
    vehicleESP.ActiveDrawings = {}
    
    for _, data in pairs(graveESP.ActiveDrawings) do
        if data.text then
            data.text:Remove()
        end
    end
    graveESP.ActiveDrawings = {}
    
    for _, data in pairs(ammoESP.ActiveDrawings) do
        if data.drawing then
            data.drawing:Remove()
        end
    end
    ammoESP.ActiveDrawings = {}
    
    for _, data in pairs(weaponsESP.ActiveDrawings) do
        if data.drawing then
            data.drawing:Remove()
        end
    end
    weaponsESP.ActiveDrawings = {}
    
    WatermarkConnection:Disconnect()
    Library.Unloaded = true
end)
