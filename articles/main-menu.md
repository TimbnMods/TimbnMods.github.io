# Main Menu

Every loaded Timbn mod appears on the main menu with its name and version above the game's credits, so players can see what is loaded at a glance. You don't need to add that line yourself. Both calls below are for anything beyond it, and both go away when the mod unloads.

## Adding a Line

### `MainMenu.AddLine(label, value)`

Adds another line to that list, drawn like the game's own "published by" credit. The `label` is the plain part and the `value` is the highlighted part. Lines are listed in the order they were added and show only on the main menu, never in-game. Returns a handle that removes the line early when disposed.

```csharp
MainMenu.AddLine("Pet Pack: ", "3 pets");
```

## Showing a Popup

### `MainMenu.AddPopup(header, text)`

Shows a message in the game's dialog window with an OK button the next time the main menu is on screen, or right away if it already is. Each popup shows once, and multiple popups will queue up and display in sequence. Pass a `header`, since the game logs an error when it is empty. Returns a handle that cancels the popup if it hasn't shown yet.

```csharp
protected override void OnAwake()
{
    MainMenu.AddPopup("Vanilla Tweaks", "The tech point cap is now 9999.");
}
```

Core uses this popup to warn when the game's version or build differs from the one the mods were tested on (once for each build, see [Game Versions](game-versions.md)), and to list any mod that failed to load. That includes mods skipped by BepInEx due to missing or incompatible dependencies, plus Timbn mods (or individual features) that were disabled because a game update modified a required method or field.
