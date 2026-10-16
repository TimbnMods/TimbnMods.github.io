# Changing the Game

## Changing GameBalance

### `Balance.Edit<T>(id, apply, revert)`

Changes one of the game's own definitions, such as a perk, an item, or a craft, and puts it back when the mod unloads. Apply runs right away if the balance has loaded and again on the fresh copy each time it loads, so the change survives a reload. Capture any state needed by `revert` from within `apply`, and pair it with `Settings.While` to follow a toggle.

```csharp
var ticks = 0;
Settings.While(PluginConfig.PerkTalentBonus, () =>
    Balance.Edit<PerkDef>(
        "perk_green_thumb",
        perk => { ticks = perk.craftStartTicks; perk.craftMasteryBonus = ticks; perk.craftStartTicks = 0; },
        perk => { perk.craftStartTicks = ticks; perk.craftMasteryBonus = 0; }
    ));
```

### `Balance.Add`

Registers a custom definition under an unused ID (if a collision occurs, the game's definition is retained and the conflict is logged as an error). It goes in straight away if the balance has loaded and again each time it loads, and comes out when the mod unloads. Adding a `CraftDef` or `ItemDef` also rebuilds the game's craft lookups, and a `BuildingDef`, `WGODef` or `QuestDef` refreshes the lookups it needs too. Pass a second argument to run code on the loaded balance right before each insert, for a definition that needs something from it. Returns a handle that removes the definition early when disposed.

### `Balance.AddCopy<T>(templateId, id, edit)`

Adds a copy of one of the game's definitions under a new id, such as an `ItemDef`, `CraftDef`, `WGODef` or `BuildingDef`. Every field is copied from the game's definition each time the balance loads, then `edit` runs on the copy, so you only change what differs. `edit` is optional.

```csharp
Balance.AddCopy<ItemDef>("cook_baked_fish", "timbn_smoked_fish", fish => TimbnBalance.SetIcon(fish, "timbn_smoked_fish"));
```

### `TimbnBalance.Loaded`

The loaded `GameBalance`, or null if the game hasn't loaded it yet. Unlike `GameBalance.Me` it never forces a load.

### `TimbnBalance.SetIcon(item, iconId)`

Changes an item's icon in both places the game keeps it.

### `TimbnBalance.GetIcon(item)`

Reads the icon id an item shows, which is the one `SetIcon` set if there is one.

### `TimbnBalance.Expression(formula)`

Builds a game expression for an expression field, such as an item's `onUseExpressions`, the way the balance parses its own. Unlike `new LazyExpression(formula)`, a `#perk` or `$resource` in it works.

```csharp
item.onUseExpressions = [TimbnBalance.Expression("AddPerk(\"buff_beer\")")];
```

### `TimbnBalance.TextExpression(value)`

Wraps plain text as an expression, for a field the game reads as an expression but only shows, such as a craft's `customCraftResultIcon`.

To add a new item, station or craft rather than change the game's, see [Stations and Crafts](stations-and-crafts.md), where `Balance.AddItem` is covered too.

## Holds

Before modifying player state or time speed, understand how holds operate.

A hold changes something the game or another mod may also change. It ends when its handle is disposed, the mod unloads, or the player returns to the main menu. Several mods can hold the same thing at once, and it only ends when the last hold is released.

```csharp
_fastTime = Clock.MultiplyGameSpeed(7f);
_playerControl = Player.TakeControl();
// later
_fastTime?.Dispose();
_playerControl?.Dispose();
```

So if three mods take control of the player, the player only gets control back once all three release it. One mod giving control back never breaks another mod that is taking it for a custom cutscene.

For speed holds, the newest `Player.MultiplyWalkSpeed` or `Clock.MultiplyGameSpeed` is the one that applies. When it is released, the next newest takes over again.

## Controlling the Player

### `Player.TakeControl()`

Takes the player's control away the way the game's cutscenes do, so they cannot walk, interact, attack, use the hotbar or open menus, and cannot be pushed. Each call adds its own reason to the game's list of reasons the player is held, and the game only gives control back once every reason has let go, so a game cutscene ending in the meantime never frees the player early. Core's conversations use it too. Read the game's own keys with `GameKey.Interaction.IsDownInGame()` (see [TimbnInput](helpers/timbn-input.md)) while it is held, and use `TimbnPlayer.GameHasControl(...)` to detect when the game itself takes control.

### `Player.MultiplyWalkSpeed(multiplier)`

Multiplies how fast the player walks, on top of the game's own slowdowns such as aiming a bow. Returns a handle that ends the change when disposed.

### `Player.WalkTo(position, arrived)`

Walks the player to a spot along the navmesh as if they held the movement keys, and calls `arrived` when they get there. Without control held, any movement input from the player ends the walk. Take control first for a walk a scene needs. Dispose the handle to cancel it.

```csharp
_control = Player.TakeControl();
_walk = Player.WalkTo(bench, SitDown);
```

### `Player.SlideTo(position, arrived)`

Moves the player along the NPC navmesh with the game's own walker, the way a story scene does. Physics collisions are disabled along the way so they never get stuck, but note that they will pass through gaps and across ledges. Movement keys don't stop it, so dispose the handle to cancel it.

### `Player.Carry(item, putDown, rule)`

Puts an item over the player's head the way they carry a log. The player puts it down with the interaction key, and `putDown` runs once they do. The item never reaches a save. The carry rule defines how the item is held and dropped; each successive rule adds restrictions over the previous one.

| Rule | What it does |
| --- | --- |
| `TimbnCarryRule.Vanilla` | As the game carries a log. The key puts it down, the game's own drops take it, and other things stack on top while the player has carry slots to spare. The default. |
| `TimbnCarryRule.Exclusive` | The only thing in their hands. What they held before drops at their feet, nothing else can be picked up while they hold it, and another `Carry` is refused until it is gone. The key still puts it down. |
| `TimbnCarryRule.OnlyByCode` | Exclusive, and nothing but disposing the handle drops it. Useful for items (such as a mug) that the player should only hand over at a custom mod interaction point. |

```csharp
_mug = Player.Carry(new Item("my_mod_mug"), Spilled, TimbnCarryRule.OnlyByCode);
// once it is handed over
_mug.Dispose();
```

Check `TimbnPlayer.HandsFree` first when a mod would rather refuse than drop what the player holds.

## Controlling Time

### `Clock.MultiplyGameSpeed(multiplier)`

Sets how fast the game's time runs. Returns a handle, and disposing it gives back the speed from before unless the game changed it in the meantime.

### `Clock.Freeze()`

Stops the game's clock while the world keeps moving, and returns a handle. The clock runs again when the last hold is released, unless the game paused it itself in the meantime, for a fight or a story scene, in which case the release leaves it to the game.
