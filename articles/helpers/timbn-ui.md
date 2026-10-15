# TimbnUI

Signatures are in the [API reference](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnUI).

## Windows, Notices and Drawing

### `ActiveWindow`

The game window on top, such as the inventory, a chest, or the pause menu, or null when the player is in the world with nothing open.

### `IsScreenFading`

True while a screen fade to or from black is showing. You can wait until the fade finishes before running logic.

### `IsWindowOpen<TWindow>()`

Whether the game window on top is of the given type, such as the building window.

```csharp
if (TimbnUI.IsWindowOpen<UIBuildingWindow>())
    BuildPins.Draw();
```

### `FadeThrough(action)`

Fades the screen to black, runs your code while it is black, and fades back in, the way the game hides a change such as the player sitting down or a teleport. When the game has no fade to show, or one is already showing, the code runs right away. Nothing owns the code, so it still runs if the mod unloads during the fade. Use `UI.FadeThrough` below for code that must not outlive the mod.

```csharp
TimbnUI.FadeThrough(() => TimbnPlayer.TeleportTo(campfire));
```

### `TimbnUI.Notify(key)`, `TimbnUI.Notify(key, iconId)`

Shows a line of text in the game's own notification popup, the one it uses for things like a closed shop, with an icon beside it if you pass one. The text is looked up by its localization key, so add it with `Text.Add` or a lang file. It can name an item or an NPC with the game's `#(id)` markup. Does nothing at the main menu.

```csharp
TimbnUI.Notify("my_mod_bar_open", "i_cup_beer");
```

### `TimbnUI.NotifyOnce(key)`

The same popup, but only when that text isn't on screen already, for a warning that would otherwise stack up every frame.

The remaining helpers live on the mod's `UI` property, and everything created through it is cleaned up when the mod unloads.

### `UI.FadeThrough(action)`

Works like `TimbnUI.FadeThrough`, but returns a handle that can cancel the action before the screen fades to black, such as when the player interrupts the action.

```csharp
_fade?.Dispose();
_fade = UI.FadeThrough(SitDown);
```

### `UI.CreateHint()`

Makes the game's "press a key to do something" hint for any spot in the world. While it shows, the game ignores a press of its key, the way it does for its own hints, so the press only reaches your mod.

The prompt adjusts automatically for language and whether the player uses a keyboard or controller. Move it with `Show(position, text)` as often as you like and dismiss it with `Hide()`. `IsVisible` shows whether it is on screen. The text has to be in the player's language already, so pass a key through `LLBase.L`. The key is `GameKey.Interaction` unless you pass another one after the text, and a second text and key show a second row, the way a cargo lift shows Open and Put. It is hidden when the mod unloads and when the player leaves the save. Dispose a hint the mod is done with, such as one made per visitor, to release its resources. One kept for the mod's whole life does not need to be disposed.

```csharp
_hint = UI.CreateHint();
// every frame the player stands by the fire
_hint.Show(fire + Vector3.up * 1.2f, LLBase.L("my_mod_meditate"));
// by a keg, with two keys
_hint.Show(keg, LLBase.L("my_mod_open"), GameKey.Interaction, LLBase.L("my_mod_pour"), GameKey.Action);
// once they walk away
_hint.Hide();
```

### `UI.CreateBubble()`

Makes an icon bubble to show over any spot in the world, such as a beer over a customer's head. Move it with `Show(position, iconId, crossedOut)` as often as you like (`crossedOut` is optional and draws a cross over the icon) and dismiss it with `Hide()`. `IsVisible` shows whether it is on screen. `Framed`, `Scale` and `Tint` change how it looks. It is hidden when the mod unloads and when the player leaves the save, and a bubble made per customer should be disposed once that customer is gone to release its resources.

```csharp
_wants = UI.CreateBubble();
_wants.Show(customer.Position + Vector3.up * 2f, "i_cup_beer");
```

### `UI.OpenStorage(contents, titleKey, slots, itemIds)`

Opens the game's chest window on a storage only your mod knows about, such as a keg, with the player's inventory beside it. `contents` is a dictionary of item id to count that you keep, usually in your save data, and every move the player makes is written back into it. `titleKey` is a text key added with `Text.Add`, shown over the storage, and `slots` is how many slots it has. `itemIds` limits what may be put in, and leaving it out takes anything. The window closes when the mod unloads.

```csharp
UI.OpenStorage(Data.Current.Keg, "my_mod_keg", 10, "beer");
```

### `UI.AddObjectAction(action)`

Adds an interaction to an object that lacks native game interactions, such as petting a cat on a box. Standing by the object shows a hint, and pressing Interaction walks the player to a spot beside it where they face one way and play an animation for a while. Describe it with a `TimbnObjectAction`, which takes the name the object's GameObject starts with, the spot the player stands at (its height is ignored) and a text key for the hint. Set `Facing` (up by default), `Animation` (working with their hands by default) and `GameMinutes` (10 by default) to change how it plays out. Three optional callbacks hook it up to your mod. `CanPerform` is asked before the hint shows, so returning false hides the action until a condition is met. `OnStarted` runs when the player reaches the spot and the animation begins. `OnFinished` runs only when the full minutes pass, not when the game takes the controls back first.

```csharp
UI.AddObjectAction(new TimbnObjectAction("common_box_08", new Vector3(-104.36f, 0f, -338.20f), "my_mod_pet"));
```

### `UI.DrawGui(draw)`

Draws IMGUI in place of an `OnGUI` method, on the main menu and in-game while the mod is enabled. A handler that throws is logged once and stopped.

### `UI.CreateDebugWindow(title, draw, toggle)`

Draws a debug or cheat menu with its box, title, close key, and "load a save" note done for you. `toggle` is an optional config entry holding the key that opens it. Leave it out to open the window from code with `IsVisible`. Use `TimbnGui.Header(text)` to draw a bold heading inside it. The returned window has `IsVisible` to open or close it from code, `Area` for its size and place, `AnchorRight` to pin it to the right edge, and `InGameOnly` (true by default) to hide it at the main menu.
