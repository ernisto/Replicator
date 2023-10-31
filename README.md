# Replicator
Generated by [Rojo](https://github.com/rojo-rbx/rojo) 7.4.0-rc3.

## Wally Installation
To install with wally, insert it above wally.toml [dependecies]
```toml
Replicator = "ernisto/Replicator@0.1.0"
```

## Usage (Server)
```lua
local function Inventory(owner: Player)
    
    local container = Instance.new("Folder", owner)
    container.Name = "Inventory"
    
    local client = Replicator.get(owner)
    local self = wrapper(container)
    
    --// Remote Events  ## create RemoteEvents
    self.itemEquipped = client:_signal("itemEquipped")
    self.itemAdded = client:_signal("itemAdded")
    
    --// Remote Methods  ## create RemoteFunctions
    function client:equipItem(item: Tool)
        
        self.itemEquipped:_emitOn({ owner }, item, item.privateData)   -- whitelist :FireOnClient()
        self.itemEquipped:_emitOff({ owner }, item)                    -- blacklist :FireOnClient()
        
        return self:equipItem(item)
    end
    
    --// Methods
    function self:addItem(item)
        
        self.itemAdded:_emit(item)    -- :FireAllClients
    end
    
    --// End
    return self
end
```

## Usage (Client)
```lua
local function Inventory(container: Folder)
    
    local server = Replicator.get(container)
    local self = wrapper(container)
    
    self.itemEquipped:connect(function(item: Tool)
        
        print("item equipped", item)
    end)
    self.itemAdded:connect(function(item: Tool)
        
        print("item added", item)
        if container.owner == localPlayer then
            
            local equipButton = Button{ text = "EQUIP" }
            equipButton.onClick:connect(function()
                
                self:invokeEquipItemAsync(item) -- wrapped by evaera/Promise
                    :andThenCall(equipButton.Destroy, equipButton)
            end)
        end
    end)
    
    --// End
    return self
end
```