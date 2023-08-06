**Addon logs QBCore**

Description - More Logs are gonna be added soon, if you have some please do a PR 😄

1) Impound Logs For qb-policejob
2) Character Creation log for qb-multicharacter

----------------------------------------------

# Impound Logs For qb-policejob

find event "police:client:ImpoundVehicle" in qb-policejob > client > job.lua replace it with 
```lua
RegisterNetEvent('police:client:ImpoundVehicle', function(fullImpound, price)
    local vehicle = QBCore.Functions.GetClosestVehicle()
    local bodyDamage = math.ceil(GetVehicleBodyHealth(vehicle))
    local engineDamage = math.ceil(GetVehicleEngineHealth(vehicle))
    local totalFuel = exports['cdn-fuel']:GetFuel(vehicle)
    if vehicle ~= 0 and vehicle then
        local ped = PlayerPedId()
        local pos = GetEntityCoords(ped)
        local vehpos = GetEntityCoords(vehicle)
        if #(pos - vehpos) < 5.0 and not IsPedInAnyVehicle(ped) then
           QBCore.Functions.Progressbar('impound', Lang:t('progressbar.impound'), 5000, false, true, {
                disableMovement = true,
                disableCarMovement = true,
                disableMouse = false,
                disableCombat = true,
            }, {
                animDict = 'missheistdockssetup1clipboard@base',
                anim = 'base',
                flags = 1,
            }, {
                model = 'prop_notepad_01',
                bone = 18905,
                coords = { x = 0.1, y = 0.02, z = 0.05 },
                rotation = { x = 10.0, y = 0.0, z = 0.0 },
            },{
                model = 'prop_pencil_01',
                bone = 58866,
                coords = { x = 0.11, y = -0.02, z = 0.001 },
                rotation = { x = -120.0, y = 0.0, z = 0.0 },
            }, function() -- Play When Done
                local plate = QBCore.Functions.GetPlate(vehicle)
                local modelHash = GetEntityModel(vehicle)
                local vehicleName = GetDisplayNameFromVehicleModel(modelHash)
                TriggerServerEvent("police:server:Impound", plate, fullImpound, price, bodyDamage, engineDamage, totalFuel, vehicleName)
                while NetworkGetEntityOwner(vehicle) ~= 128 do  -- Ensure we have entity ownership to prevent inconsistent vehicle deletion
                    NetworkRequestControlOfEntity(vehicle)
                    Wait(100)
                end
                QBCore.Functions.DeleteVehicle(vehicle)
                TriggerEvent('QBCore:Notify', Lang:t('success.impounded'), 'success')
                ClearPedTasks(ped)
            end, function() -- Play When Cancel
                ClearPedTasks(ped)
                TriggerEvent('QBCore:Notify', Lang:t('error.canceled'), 'error')
            end)
        end
    end
end)
```

find event "police:server:Impound" in qb-policejob > server > main.lua repalce it with 
```lua
RegisterNetEvent('police:server:Impound', function(plate, fullImpound, price, body, engine, fuel, vehname)
    local src = source
    local Player = QBCore.Functions.GetPlayer(src)
    local firsname = Player.PlayerData.charinfo.firstname
    local lastname = Player.PlayerData.charinfo.lastname
    local gradename = Player.PlayerData.job.grade.name
    local name = firsname .. " " .. lastname
    price = price and price or 0
    if IsVehicleOwned(plate) then
        if not fullImpound then
            MySQL.query(
                'UPDATE player_vehicles SET state = ?, depotprice = ?, body = ?, engine = ?, fuel = ? WHERE plate = ?',
                {0, price, body, engine, fuel, plate})
            TriggerClientEvent('QBCore:Notify', src, Lang:t("info.vehicle_taken_depot", {price = price}))
            TriggerEvent('qb-log:server:CreateLog', 'seizelogs', 'Vehicle Impounded', 'red', "**[" .. GetPlayerName(src) .. "]** **" .. name .. " Impounded a Car**\nRank: **".. gradename .." ** \nModel: **" .. vehname .. "**\nPlate: **[" .. plate .. "]**\nPrice: **$" .. price .. "**\nBody Health: **" .. body .. "**\nEngine Health: **" .. engine .. "**\nFuel: **" .. fuel .. "**", false)
        else
            MySQL.query(
                'UPDATE player_vehicles SET state = ?, body = ?, engine = ?, fuel = ? WHERE plate = ?',
                {2, body, engine, fuel, plate})
            TriggerClientEvent('QBCore:Notify', src, Lang:t("info.vehicle_seized"))
            TriggerEvent('qb-log:server:CreateLog', 'seizelogs', 'Vehicle Confiscated', 'red', "**[" .. GetPlayerName(src) .. "]** **" .. name .. " Confiscated a Car**\nRank: **".. gradename .." ** \nModel: **" .. vehname .. "**\nPlate: **[" .. plate .. "]**\nBody Health: **" .. body .. "**\nEngine Health: **" .. engine .. "**\nFuel: **" .. fuel .. "**", false)
        end
    end
end)
```

Result for **Impound Logs For qb-policejob**

![image](https://github.com/OmiJod/addonlogs-qbcore/assets/69292814/d1fabeb9-67a3-479a-9531-4e41619e7407)

----------------------------------------------

# Character Creation log for qb-multicharacter

find "qb-multicharacter:server:createCharacter" in qb-multicharacter > server > main.lua and add the below line in the correct place 
```lua
            TriggerEvent('qb-log:server:CreateLog', 'charlogs', 'Character Created', 'red', '**'..GetPlayerName(src)..'** Created a Character: First Name:** '.. data.firstname .. " ** Last Name:** ".. data.lastname.. "**", false)
```
for example 
![image](https://github.com/OmiJod/addonlogs-qbcore/assets/69292814/bd4a8096-c58b-4f0e-a5e7-41e394ddc3b1)

Result for **Character Creation log for qb-multicharacter**

![image](https://github.com/OmiJod/addonlogs-qbcore/assets/69292814/1d946127-8157-4d8c-8a7c-b7e59b3dca03)

