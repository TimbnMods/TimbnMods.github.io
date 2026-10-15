# TimbnInput

Signatures are in the [API reference](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnInput). The samples on this page keep keys as `ConfigEntry<KeyboardShortcut>` and use `GameKey`, which needs `using LazyBearTechnology;`.

## Keys and Control

### `TimbnInput.HotkeysEnabled`

True when a save is loaded, no game window is open (inventory, crafting, pause menu, a text field), and the game is reading input. It stays true when the game has taken the player's control, such as during a cutscene, so use it for a key that doesn't need the player to have control.

### `TimbnInput.PlayerHasControl`

True when `HotkeysEnabled` is true and the player has control. The game takes control away during dialog, cutscenes, sleep, teleports, fishing, building, ladders, work at a station, and attacks.

### `TimbnInput.GameHasInteraction`

True when the game has its own interaction ready for the Interaction key, such as a world object within reach or a big drop to pick up. A mod that uses the interaction key can stop listening during those moments.

### `IsDownInGame`

True if a `KeyboardShortcut` was pressed this frame while `HotkeysEnabled` is true. Call it from `OnUpdate` in place of `IsDown()`.

```csharp
if (PluginConfig.UnstuckKey.Value.IsDownInGame())
    Unstuck.Run(PluginConfig.UnstuckRange.Value);
```

### `IsDownWithControl`

True if a `KeyboardShortcut` was pressed this frame while `PlayerHasControl` is true, ensuring the key does nothing during dialog, cutscenes, or other times the player cannot act.

```csharp
if (PluginConfig.SpawnKey.Value.IsDownWithControl())
    Spawner.DropAtPlayer();
```

### `IsDownInWindow<TWindow>`

True if a `KeyboardShortcut` was pressed this frame while a game window of that type is open and active. Use it for a key that adds actions to one of the game's own windows.

```csharp
if (PluginConfig.TogglePin.Value.IsDownInWindow<UIBuildingWindow>())
    buildPins.TogglePinUnderCursor();
```

### `GameKey.IsDownInGame()` and `GameKey.IsHeldInGame()`

Checks the game's rebindable keys, such as Interaction. Returns true if the key was pressed this frame (for `IsDownInGame`) or is currently held (for `IsHeldInGame`), while `HotkeysEnabled` is true.

```csharp
if (_hint.IsVisible && GameKey.Interaction.IsDownInGame())
    PetTheCat();
```

### `IsDownIgnoringOtherKeys`

True if a `KeyboardShortcut` was pressed this frame, even while other keys are held. BepInEx's `IsDown` returns false whenever extra keys are down, so a key that needs to work while walking should use this method instead. It doesn't check whether the game is accepting input, so pair it with `HotkeysEnabled` or `PlayerHasControl`.

```csharp
if (TimbnInput.PlayerHasControl && PluginConfig.Sprint.Value.IsDownIgnoringOtherKeys())
    sprint.Start();
```

### `IsHeldIgnoringOtherKeys`

Similar to `IsDownIgnoringOtherKeys`, but true every frame the shortcut is held rather than just the frame it was pressed. Pair it with `HotkeysEnabled` or `PlayerHasControl` in the same way.
