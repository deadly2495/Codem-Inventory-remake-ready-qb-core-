**mInventory on QB-Core 1.3.0 Useable Items:**
For people in new qb-core 1.3.0 Useable items don't work by default with our inventory, those changes are needed:

-------------**qb-core\server\functions.lua**

---Create a usable item
---@param item string
---@param data function|table
function QBCore.Functions.CreateUseableItem(item, data)
    local rawFunc = nil

    if type(data) == 'table' then
        if rawget(data, '__cfx_functionReference') then
            rawFunc = data
        elseif data.cb and rawget(data.cb, '__cfx_functionReference') then
            rawFunc = data.cb
        elseif data.callback and rawget(data.callback, '__cfx_functionReference') then
            rawFunc = data.callback
        end
    end

    if not rawFunc and type(data) == 'function' then
        rawFunc = Citizen.CreateFunctionReference(data)
    end

    if rawFunc then
        QBCore.UsableItems[item] = {
            func = rawFunc,
            resource = GetInvokingResource()
        }
        -- Optinal for debugging:
        -- print(("✅ Registered usable item: ^3%s^0"):format(item))
    else
        -- Optinal for debugging:
        -- print(("^1[QBCore] Failed to register usable item: %s - Invalid handler^0"):format(item))
    end
end




------------**codem-inventory\editable\serverexport.lua**

function UseItem(itemname, ...)

    local itemData = Core.Functions.CanUseItem(itemname)
    if not itemData then return end

    local callback = nil

    if type(itemData) == 'table' then
        callback = itemData.func or itemData.cb or itemData.callback
    elseif type(itemData) == 'function' then
        callback = itemData
    end

    if not callback then return end

    TriggerEvent('codem-inventory:usedItem', itemname, ...)
    callback(...)
end


