# Data Stores 101
## What are DataStores?
DataStores are, unsurprisingly, used to store data. One DataStore can hold data for tens of thousands of different players, and can even be used to store non-player data! Storing data is essential for just about any game, so it's important to learn early on so you don't struggle with adding it when you're deep into a game's development.

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
## Section 2: Storing Multiple Values
Okay, so storing our Coins value works all fine and dandy. But what if we want to store both our Coins value and our Gems value? Well, you might be tempted to just create another DataStore and use several SetAsync/GetAsync calls, like this:
```lua
local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local CoinsDataStore = DataStoreService:GetDataStore("Coins")
local GemsDataStore = DataStoreService:GetDataStore("Gems")

Players.PlayerAdded:Connect(function(player)
	-- ...
	local storedCoinAmnt = 100
	local storedGemsAmnt = 20
	local success, err = pcall(function()
		local dataCoins = CoinsDataStore:GetAsync(player.UserId)
		local dataGems = GemsDataStore:GetAsync(player.UserId)
		if dataCoins ~= nil then
			storedCoinAmnt = dataCoins
			storedGemsAmnt = dataGems
		end
	end)
	-- ...
end)

Players.PlayerRemoving:Connect(function(player)
	local success, err = pcall(function()
		CoinsDataStore:SetAsync(player.UserId, player.Stats.Coins.Value)
		GemsDataStore:SetAsync(player.UserId, player.Stats.Gems.Value)
	end)
	-- ...
end)
```
However, not only is an approach like this unwieldy and hard to maintain, it's also impossible to use when you're storing lots of data! DataStores are rate limited, meaning that they can only handle a certain number of requests before they start to slow down or stop working entirely, and those rate limits are harsh. Consecutive DataStore requests like this cause saving to take ages and will destroy your rate limits.

Instead, we can turn to something that you might have heard of before—a table! Tables, in short, allow you to store many values in one container. If a single variable is like a box that can store one thing, then think of a table like a moving truck, which can store many boxes. Here's a bit of table terminology that might be useful:
* Index/Key - The value used to "access" another value in the table, i.e. `table[index]` or `table.index`. An index can be of any type, including numbers, strings, Instances, and even functions or other tables!
* 'Indexed by' - Used to say that a table's indices follow a certain pattern. For example, a list of parts might be indexed by number, and a table that stores people's heights might be indexed by the person's name.
* Array - A table indexed by ordered integers, starting at 1. 
* Dictionary - A table indexed by anything that isn't an ordered integer.
* Mixed table - A table which is both indexed by ordered integers AND unordered/non-integers.

So, let's try using a table to store both our Coins value and our Gems value.
```lua
local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")
local PlayerDataStore = DataStoreService:GetDataStore("PlayerData") --[[We're not just storing Coins anymore,
	so it's important to have a descriptive name!]]
