# Sprites, Text, and Audio

## Sprites

### `Sprites.AddPng(name, pngBytes)`

Adds a PNG as a sprite the game can show by name, such as for an item's or a buff's `IconId`, a bubble, or a quest `Icon`. PNGs smaller than the game's 48 by 48 icon canvas are centered on one, and a larger PNG is squashed to fit with a warning in the log. A PNG the game can't read logs a warning and adds nothing. The sprite is removed when the mod unloads, or early when the returned handle is disposed.

```csharp
Sprites.AddPng("my_mod_mug", mugPng);
```

### `Sprites.AddEmbeddedPng(name)`

The same for a PNG embedded in the mod's DLL as `<name>.png`, added under that name. It throws if the mod has no such embedded resource.

```csharp
Sprites.AddEmbeddedPng("timbn_stick_box");
```

```xml
<ItemGroup>
  <EmbeddedResource Include="Icons\*.png" LogicalName="%(Filename)%(Extension)" />
</ItemGroup>
```

## Text

### `Text.Add(key, text)` and `Text.Add(texts)`

Adds localization text under a key, shown in every language. Both forms return a handle that removes the text early. `Text.Add(texts)` takes a dictionary of text by key. Use the key anywhere the game takes one, such as a notification or a hint. Text for a quest or a talk is added for you. Text added here is removed when the mod unloads, and any previously overridden text is restored. To let players translate a mod, put its text in `lang/<language>.txt` files instead (see [Translations](quests.md#translations)). Text from a lang file takes precedence over text added in code for the same key.

```csharp
Text.Add("my_mod_pet", "Pet");
TimbnUI.Notify("my_mod_parcel_lost");
```

[`TimbnMarkup`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnMarkup) formats the game's text markup so you don't need to format the raw syntax manually.

| Member | What it gives |
| --- | --- |
| `TimbnMarkup.Icon("slv")` | `(*slv*)`, which draws the named icon. |
| `TimbnMarkup.Weekday(TimbnWeekday.Pride)` | `(*day_pride*)`, the icon of a day of the week. |
| `TimbnMarkup.Localized("npc_herm")` | `#(npc_herm)`, the text of another key, such as Herm's name in the player's language. |

## Audio

### `Audio.PlayMusic(trackId)`

Plays one of the game's music tracks on a loop, such as `violin` or `char_theme_linda` (see [Game Ids](game-ids.md)), fading out whatever was playing. When the handle is disposed, the track fades out and previous music returns. A newer track from any mod takes over, and previous music resumes when that track ends. It also stops when the mod unloads and on the way to the main menu. Nothing plays at the main menu, and a track ID the game does not recognize logs a warning and plays nothing.

```csharp
_song = Audio.PlayMusic("violin");
// once the performer stops
_song.Dispose();
```

### `Audio.PlaySound(soundId)`

Plays one of the game's sounds once, the same wherever the player stands. It ends on its own, so there is nothing to undo, and nothing plays at the main menu. `TimbnGameIds.Sounds` lists the ids (see [Game Ids](game-ids.md)). They also appear as the `onUseSound` of the game's items, such as `item_use_food`, `item_use_drink`, or `item_use_paper`, and `coins_sound` is the clink of money. An unknown id logs a warning and plays the game's default sound.

```csharp
// a customer takes their beer
Audio.PlaySound("item_use_drink");
```
