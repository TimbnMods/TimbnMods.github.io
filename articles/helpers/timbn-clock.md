# TimbnClock

Everything here reads as zero, null or false at the main menu, and `TimeText` as `00:00`. Signatures are in the [API reference](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnClock).

## Time and Days

### `Day`

The day of the save, counting up from 1. This matches the number passed to `NewDayStarted` handlers.

### `WeekdayNumber`

The day's place in the game's six-day week, from 1 to 6. `Events.NewWeekdayStarted` passes its handlers a `TimbnWeekday` instead.

### `Weekday`

Today's day of the week as a `TimbnWeekday?` enum value (such as `TimbnWeekday.Gluttony`), or null at the main menu.

```csharp
if (TimbnClock.Weekday == TimbnWeekday.Pride)
    Logger.LogInfo("Town orders can be accepted today.");
```

### `TimeOfDay`

The time of day as a decimal running from 0 to 1: 0 at midnight, 0.25 at dawn, 0.5 at noon, and 0.75 at dusk, returning to 1 at the next midnight.

### `TimeText`

The time of day as a 24-hour clock, such as `06:30`.

### `IsPaused`

True when the game's clock is stopped. The game pauses during some cutscenes or while a mod holds `Clock.Freeze`. To freeze the clock or run it faster, see [Controlling Time](../changing-the-game.md#controlling-time).

### `DayFractionBetween(from, to)`

How far the clock moved from one time of day to another, as a fraction of a day. It counts across midnight, so 0.9 to 0.1 is 0.2. Useful with `TimeOfDayChanged` to work out how much game time passed since the last tick. Does not calculate time across multiple days.

### `Minutes`

The game clock in minutes since the save began, counting whole days and the part of today. Use it for waits that follow game time, so they stop while the game is paused and speed up with it.

```csharp
_leaveAt = TimbnClock.Minutes + 30;
// later
if (TimbnClock.Minutes >= _leaveAt)
    herm.WalkTo("npc_sim_village_right_side_road_34");
```

`Minutes` is a `double`, so keep `_leaveAt` as a `double` too.

### `SetTime(timeOfDay)`

Sets the time of day, the way the game does when a cutscene jumps the clock. The lighting and everything tied to the clock update immediately. The day doesn't change, so setting a time earlier than now turns the clock back within the same day. Values outside 0 to 1 wrap around. Returns false at the main menu.

```csharp
TimbnClock.SetTime(0.25f);
```
