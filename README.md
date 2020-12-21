# Data Stores 101
## What are Data Stores?
Data Stores are, unsurprisingly, used to store data. One Data Store can hold data for tens of thousands of different players, and can even be used to store non-player data! Storing data is essential for just about any game, so it's important to learn early on so you don't struggle with adding it when you're deep into a game's development.

For the sake of this tutorial, I've prepared a simple place where there are four things that we should be able to store by the end—two different currencies, a color, and some items that the player can pick up. I won't go into detail on how to make this place, since that's not the focus of the article, but you can download it [here](https://mega.nz/file/lKhSFajA#CdQj8-1WcL0YSN2TpYBKogmuzon-mZCD6XEkbsrUHKI). If you download it, make sure to publish the place to Roblox and enable "Studio Access to API services" in the game settings.
## Section 1: Storing a Single Value
Right now, I have a very simple script which creates some values for the player.
```lua
local Players = game:GetService("Players")

Players.PlayerAdded:Connect(function(player)
	
	local stats = Instance.new("Folder")
	stats.Name = "Stats"
	
	local coins = Instance.new("IntValue")
	coins.Name = "Coins"
	coins.Value = 100
	coins.Parent = stats
	
	local gems = Instance.new("IntValue")
	gems.Name = "Gems"
	gems.Value = 20
	gems.Parent = stats
	
	local color = Instance.new("Color3Value")
	color.Name = "Color"
	color.Value = Color3.fromRGB(255, 0, 0)
	color.Parent = stats
	
	stats.Parent = player
	
end)
```
Let's start off by just trying to save and load our Coins value.

First, we need to get the DataStoreService, and then get a DataStore using it.
```lua
local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local CoinsDataStore = DataStoreService:GetDataStore("Coins")
```
Using GetDataStore gives you the ability to read data from and write data to a unique DataStore. A game can have any number of DataStores, but for most practical purposes, you aren't going to need more than one or two.

Now, we need to make it so that the Coins value saves when the player leaves. Let's add a PlayerRemoving connection, like this:
```lua
Players.PlayerRemoving:Connect(function(player)

end)
```
To save a value, you use the SetAsync method of a DataStore, which takes a key and a value. The key is used to retrieve the value with the GetAsync method. When saving player data, you should always use the player's UserId as the key, since if you used the player's name, that player would lose all their data if they changed their username. Some people like to attach a prefix or suffix to the UserId, like "PlayerData_XXXXXXX", but it isn't necessary.
```lua
Players.PlayerRemoving:Connect(function(player)
	CoinsDataStore:SetAsync(player.UserId, player.Stats.Coins.Value)
end)
```
Now, this works, but you should always wrap methods that make web requests in a pcall. These types of methods have a chance to error—not necessarily because the developer wrote code that errors, but because something happened on Roblox's servers. Usually, code that has errored would stop running, but a pcall "catches" errors and allows code that has errored to still continue running. Generally, if the method has a little marking that says `[yields]` next to it on the [API reference](https://developer.roblox.com/en-us/api-reference), then it should be wrapped in a pcall.
```lua
Players.PlayerRemoving:Connect(function(player)
	local success, err = pcall(function()
		CoinsDataStore:SetAsync(player.UserId, player.Stats.Coins.Value)
	end)
	if success then
		print("Data saved successfully.")
	else
		warn("Data failed to save. Error:", err)
	end
end)
```
Now that we've saved the data, we can load it in PlayerAdded. Loading data is very similar to saving it. The only big difference is that now, we're not saving a value, but setting a variable to the value <i>returned</i> by GetAsync.
```lua
local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local CoinsDataStore = DataStoreService:GetDataStore("Coins")

Players.PlayerAdded:Connect(function(player)
	
	local stats = Instance.new("Folder")
	stats.Name = "Stats"
	
	local storedCoinAmnt = 100
	local success, err = pcall(function()
		-- Make sure to use GetAsync and not SetAsync when loading a value.
		local data = CoinsDataStore:GetAsync(player.UserId)
		if data ~= nil then --[[We need to make sure that the player does actually have a value stored
		before setting the variable to that value, since you can't set an IntValue's value to nil.]]
			storedCoinAmnt = data
		end
	end)
	if success then
		print("Data loaded successfully.")
	else
		warn("Data failed to load. Error:", err)
	end
	local coins = Instance.new("IntValue")
	coins.Name = "Coins"
	coins.Value = storedCoinAmnt
	coins.Parent = stats
	
	local gems = Instance.new("IntValue")
	gems.Name = "Gems"
	gems.Value = 20
	gems.Parent = stats
	
	local color = Instance.new("Color3Value")
	color.Name = "Color"
	color.Value = Color3.fromRGB(255, 0, 0)
	color.Parent = stats
	
	stats.Parent = player
	
end)
```
There! Now once a player leaves the game, the amount of coins that they had should save when they re-join. Your final code should look like this:
```lua
local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local CoinsDataStore = DataStoreService:GetDataStore("Coins")

Players.PlayerAdded:Connect(function(player)
	
	local stats = Instance.new("Folder")
	stats.Name = "Stats"
	
	local storedCoinAmnt = 100
	local success, err = pcall(function()
		local data = CoinsDataStore:GetAsync(player.UserId)
		if data ~= nil then
			storedCoinAmnt = data
		end
	end)
	if success then
		print("Data loaded successfully.")
	else
		warn("Data failed to load. Error:", err)
	end
	local coins = Instance.new("IntValue")
	coins.Name = "Coins"
	coins.Value = storedCoinAmnt
	coins.Parent = stats
	
	local gems = Instance.new("IntValue")
	gems.Name = "Gems"
	gems.Value = 20
	gems.Parent = stats
	
	local color = Instance.new("Color3Value")
	color.Name = "Color"
	color.Value = Color3.fromRGB(255, 0, 0)
	color.Parent = stats
	
	stats.Parent = player
	
end)

Players.PlayerRemoving:Connect(function(player)
	local success, err = pcall(function()
		CoinsDataStore:SetAsync(player.UserId, player.Stats.Coins.Value)
	end)
	if success then
		print("Data saved successfully.")
	else
		warn("Data failed to save. Error:", err)
	end
end)
```
