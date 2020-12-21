# Data Stores 101
## What are Data Stores?
Data Stores are, unsurprisingly, used to store data. One Data Store can hold data for tens of thousands of different players, and can even be used to store non-player data! Storing data is essential for just about any game, so let's start with storing a single value.

For the sake of this tutorial, I've prepared a simple place where there are four things that we should be able to store by the end—two different currencies, a color, and some items that the player can pick up. I won't go into detail on how to make this place, since that's not the focus of the article, but you can download it [here](https://mega.nz/file/lKhSFajA#CdQj8-1WcL0YSN2TpYBKogmuzon-mZCD6XEkbsrUHKI). If you download it, make sure to publish the place to Roblox and enable "Studio Access to API services" in the game settings.
## Storing a Single Value
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
