_G.autoDayCare = true
local DAYCARE_PET = "Pastel Goat" -- name of the pet you want to fuse
local PET_AMOUNT = "10" -- amount to put into daycare
-- put "max" to put max amount of pets in daycare
local IS_SHINY = false
local PET_TYPE = 0 -- change to 2 for Rainbow, 1 for Golden, or 0 for Normal
local START_DELAY = 0 -- delay in seconds before starting (set to 0 for no delay)





repeat
    task.wait()
until game:IsLoaded()

task.wait(START_DELAY)


local Library = game.ReplicatedStorage:WaitForChild('Library')
local LocalPlayer = game:GetService("Players").LocalPlayer
local Workspace = game:GetService("Workspace")
local pets = require(Library).Save.Get().Inventory.Pet

local daycareModule = require(Library.Client.DaycareCmds)

if PET_AMOUNT == "max" then
    PET_AMOUNT = daycareModule.GetMaxSlots()
end

if PET_TYPE == 0 then
    PET_TYPE = nil
end

local function trim(string)
    if not string then
        return false
    end
    return string:match("^%s*(.-)%s*$")
end

local function split(input, separator)
    if separator == nil then
        separator = "%s"
    end
    local parts = {}
    for str in string.gmatch(input, "([^" .. separator .. "]+)") do
        table.insert(parts, str)
    end
    return parts
end

local function teleportToDaycare()

    local zonePath
    local teleported = false
    while not teleported do
        for _, v in pairs(Workspace.Map:GetChildren()) do
            local zoneName = trim(split(v.Name, "|")[2])
            if zoneName and zoneName == "Beach" then
                zonePath = game:GetService("Workspace").Map[v.Name]
                LocalPlayer.Character.HumanoidRootPart.CFrame = zonePath.PERSISTENT.Teleport.CFrame
                teleported = true
                break
            end
        end
        task.wait()
    end

    if not zonePath:FindFirstChild("INTERACT") then
        local loaded = false
        local detectLoad = zonePath.ChildAdded:Connect(function(child)
            if child.Name == "INTERACT" then
                loaded = true
            end
        end)

        repeat
            task.wait()
        until loaded

        detectLoad:Disconnect()
    end

    LocalPlayer.Character.HumanoidRootPart.CFrame = zonePath.INTERACT.Machines.DaycareMachine.PadGlow.CFrame
end

local petId
for id, petData in pairs(pets) do
    if petData["id"] == DAYCARE_PET then
        if tonumber(petData["pt"]) == PET_TYPE then
            if IS_SHINY then
                if petData["sh"] then
                    petId = id
                    break
                end
            else
                if not petData["sh"] then
                    petId = id
                    break
                end
            end
        end
    end
end

if not petId then
    print("Pet not found")
else
    print("Found pet: " .. petId)
end


local function getActivePet()
    for i, _ in pairs(daycareModule.GetActive()) do
        return i
    end
end

local activePetId = getActivePet()

local daycareAvailable
if not activePetId then
    daycareAvailable = true
else
    daycareAvailable = false
end

while _G.autoDayCare do
    if daycareAvailable then
        print("Daycare is available")

        teleportToDaycare()

        task.wait()

        local args = {
            [1] = {
                [petId] = PET_AMOUNT
            }
        }

        game:GetService("ReplicatedStorage").Network:FindFirstChild("Daycare: Enroll"):InvokeServer(unpack(args))

        print("put pet into daycare")

        task.wait(2.5)

        daycareAvailable = false
    else
        print("Daycare is not available, waiting for pets to be ready")

        activePetId = getActivePet()

        print("Waiting for current daycare pet: " .. tostring(activePetId))

        while daycareModule.ComputeRemainingTime(activePetId) > 0 and _G.autoDayCare do
            task.wait(1)
        end

        if not _G.autoDayCare then
            break
        end

        print("Daycare pet is ready")

        teleportToDaycare()

        task.wait()

        game:GetService("ReplicatedStorage").Network:FindFirstChild("Daycare: Claim"):InvokeServer()

        print("Claimed pet from daycare")

        daycareAvailable = true
    end
end
