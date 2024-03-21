
"ADD THIS INTO A CLIENT SCRIPT SUCH AS QB-SMALLRESOURCES"

function Lerp(a, b, t)
    return a + (b - a) * t
end

local recoilValues = {
    ["rifle"] = {
        ["hipfire"] = {
            up = 0.005,
            min = 0.05,
            max = 0.15,
            recoveryRate = 0.001,
            recoilIncrease = 0.01,
        },
        ["aimed"] = {
            up = 0.005,
            min = 0.05,
            max = 0.15,
            recoveryRate = 0.001,
            recoilIncrease = 0.0015,
        }
    },
}

local currentRecoil = 0.0
local isShooting = false
local aiming = false
local wasShooting = false

function HandleShooting()
    if IsControlJustPressed(0, 24) then
        isShooting = true
    elseif IsControlReleased(0, 24) then
        isShooting = false
        currentRecoil = 0.0
    end

    if isShooting then
        local recoilConfig = recoilValues[GetCurrentWeaponType()][aiming and "aimed" or "hipfire"]
        if recoilConfig.recoilIncrease then
            currentRecoil = math.min(currentRecoil + recoilConfig.recoilIncrease, recoilConfig.max)
        end
    end
end

function RecoilRecovery()
    if not isShooting and currentRecoil > 0 then
        local recoilConfig = recoilValues[GetCurrentWeaponType()][aiming and "aimed" or "hipfire"]
        currentRecoil = math.max(currentRecoil - recoilConfig.recoveryRate, 0)
    end

    if wasShooting and not isShooting then
        local pitch = GetGameplayCamRelativePitch()
        SetGameplayCamRelativePitch(0.0, 1.0)
    end

    wasShooting = isShooting
end

function ApplyRecoilEffect()
    local recoilConfig = recoilValues[GetCurrentWeaponType()][aiming and "aimed" or "hipfire"]
    
    local reducedUpRecoil = recoilConfig.up * 0.5
    local multiplier = Lerp(recoilConfig.min, recoilConfig.max, currentRecoil) * 0.5
    
    local recoilAmount = reducedUpRecoil * multiplier
    
    local pitch = GetGameplayCamRelativePitch()
    
    local newPitch = pitch + recoilAmount
    if newPitch > pitch then
        newPitch = pitch
    end
    
    SetGameplayCamRelativePitch(newPitch, 1.0)
end

Citizen.CreateThread(function()
    while true do
        Citizen.Wait(0)
        
        currentWeaponType = GetCurrentWeaponType()

        HandleShooting()

        aiming = IsAimModeActive()

        RecoilRecovery()

        ApplyRecoilEffect()
    end
end)

function IsAimModeActive()
    local playerPed = PlayerPedId()
    
    if IsPlayerFreeAiming(playerPed) then
        local currentWeapon = GetSelectedPedWeapon(playerPed)
        local weaponGroup = GetWeapontypeGroup(currentWeapon)
        
        if weaponGroup ~= 0 and not IsPedShooting(playerPed) then
            local isAimingDownSights = IsAimCamActive() or IsFirstPersonAimCamActive()
            return isAimingDownSights
        end
    end
    
    return false
end

function GetCurrentWeaponType()
    local playerPed = PlayerPedId()
    local weaponHash = GetSelectedPedWeapon(playerPed)
    local weaponType = GetWeapontypeGroup(weaponHash)
    
    if weaponType == GetHashKey("GROUP_PISTOL") then
        return "pistol"
    elseif weaponType == GetHashKey("GROUP_SMG") then
        return "smg"
    elseif weaponType == GetHashKey("GROUP_RIFLE") then
        return "rifle"
    elseif weaponType == GetHashKey("GROUP_SHOTGUN") then
        return "shotgun"
    elseif weaponType == GetHashKey("GROUP_SNIPER") then
        return "sniper"
    elseif weaponType == GetHashKey("GROUP_HEAVY") then
        return "heavy"
    elseif weaponType == GetHashKey("GROUP_THROWN") then
        return "thrown"
    elseif weaponType == GetHashKey("GROUP_MELEE") then
        return "melee"
    else
        return "unknown"
    end
end

print("Zylo's recoil is active")
