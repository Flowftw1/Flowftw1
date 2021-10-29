--[[
    Script Name:        [on label] Buy mana fluids and uhs
    Description:        It's lua side where you will do actions on label buy potions.
    Author:             Ascer - example
]]

local config = {
    check = {label = "checkSupply", goto = "leave", mf = 30, uh = 200},       -- when label "check" then if amount of vials <= 5 or amount of uh <= 5 goto label "back" (minimal amount is 2 to verify message)
    refillmf = {label = "shopMF", mf = 3400},                    -- when label shop make refill (buy up to 30 vials and 20 uhs)
    refilluh = {label = "shopUH", uh = 1000},                    -- when label shop make refill (buy up to 30 vials and 20 uhs)
    dropEmptyVialsFromMainBp = false,                                -- throw empty vials from main backpack under ground
    uh = 3160                                                       -- id of uh rune
}


-- set params for start count
local currentVials, currentUhs = 351, 201

--> read label messages
function signal(label)
    if label == config.refillmf.label then
        buyFluidsUpTo(config.refillmf.mf)
	elseif label == config.refilluh.label then
        buyUhsUpTo(config.refilluh.uh)
    elseif label == config.check.label then
        if currentVials <= config.check.mf or currentUhs <= config.check.uh then
            Walker.Goto(config.check.goto)
        end    
    end
end

Walker.onLabel("signal")

--> read special messages in game
function proxyText(messages) 
    for i, msg in ipairs(messages) do 
        local vials = string.match(msg.message, "Using one of (.+) vials...")
        if vials ~= nil then
            currentVials = tonumber(vials)
        end
        local uhs = string.match(msg.message, "Using one of (.+) ultimate healing runes...")
        if uhs ~= nil then
            currentUhs = tonumber(uhs)
        end    
    end 
end 
Proxy.TextNew("proxyText")     

----------------------------------------------------------------------------------------------------------------------------------------------------------
--> Function:       buyFluidsUpTo(amount)
--> Description:    Buy up to current x mana fluids using green message and command instant.
--> Params:         
-->                 @amount number how many vials you want to have after visiting shop.
--> Return:         boolean true or nil
----------------------------------------------------------------------------------------------------------------------------------------------------------
function buyFluidsUpTo(amount)
    Self.UseItemWithMe(MANA_FLUID.id, 0)
    wait(500)
	
	for i =1, (amount-currentVials)/100 do
	Self.Say'instant buy 100 mana fluids'
	wait(1000)

    end    
end

----------------------------------------------------------------------------------------------------------------------------------------------------------
--> Function:       buyUhsUpTo(amount)
--> Description:    Buy up to current x ultimate healing runes using green message and command instant.
--> Params:         
-->                 @amount number how uhs you want to have after visiting shop.
--> Return:         boolean true or nil
----------------------------------------------------------------------------------------------------------------------------------------------------------
function buyUhsUpTo(amount)
    Self.UseItemWithMe(config.uh, 0)
    wait(500)
	
		for i =1, (amount-currentUhs)/100 do
	Self.Say'instant buy 100 ultimate healing runes'
	wait(1000)

    end    
end

----------------------------------------------------------------------------------------------------------------------------------------------------------
--> Function:       dropEmptyVials()
--> Description:    Drop empty vials from container index 0 and arrow slot to ground under your character
--> Params:         None
--> Return:         boolean true or false
----------------------------------------------------------------------------------------------------------------------------------------------------------
function dropEmptyVials()
    local items = Container.getItems(0)
    for i, item in pairs(items) do
        if item.id == MANA_FLUID.id and item.count == 0  then
            local pos = Self.Position()
            return Container.MoveItemToGround(0, i-1, pos.x, pos.y, pos.z, item.id, 1, 500) 
        end
    end
    local arrow = Self.Ammo()
    if arrow.id == MANA_FLUID.id and arrow.count == 0 then
        local pos = Self.Position()
        return Container.MoveItemFromEquipmentToGround(SLOT_AMMO, pos.x, pos.y, pos.z, arrow.id, 1, 500)
    end
    return false    
end    


--> Module for drop empty vials from main backpack and arrow slot.
Module.New("Drop empty vials from main backpack", function(mod)
    if config.dropEmptyVialsFromMainBp then
        dropEmptyVials()
    end    
    mod:Delay(300, 700)    
end)
