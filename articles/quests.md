# Quests, Dialog, and Text

Core can add quests, conversations, and localized text from code.

A quest is one `Quests.Add` call. Core registers it, makes its text keys from the quest id, shows the NPC's speech bubble when the quest is on offer or ready to finish, runs the conversations, and pays the rewards. You write each line as plain text in the call.

```csharp
var wine = Quests.Add("timbn_larry_wine", TimbnGameIds.Npcs.Larry, "Larry's Thirst", quest => quest
    .Icon("quest_icon_larry_portal")
    .Description("Larry wants 3 bottles of wine. He insists it's medicinal.")
    .DoneDescription("Larry got his wine. Most of it ended up on the floor and he's having a lovely time.")
    .OfferWhen(() => TimbnQuests.StatusOf("6_intro_guards_burial") is QuestStatus.InProgress or QuestStatus.Completed)
    .Offer(talk => talk
        .Say("Hey, Keeper! I am feeling awfully dry recently.")
        .Say("Could you fetch us 3 bottles of wine?")
        .Accept("Fine, I'll find you some wine.", then => then.Say("Hurray! I will be looking forward to it."))
        .Decline("Not now.", then => then.Say("I might turn into dust at this point.")))
    .Cost(cost => cost.Item("grape_wine:1", 3))
    .Reward(reward => reward.Money(300).Item("grape_wine:2").Reputation(10))
    .Talk(talk => talk
        .Accept("Here's your wine.", then => then
            .Say("Feels great. Wine really is the best moisturizer.")
            .Say("Here is a bottle of the good stuff for your troubles."))));
```

`Quests.Add` takes the quest id, the NPC who gives it, and its name. Keep the id stable, since saves keep progress under it. You can leave out the NPC and call `Giver` instead, with an NPC id or a function that reads the run (see Repeating Quests), or leave out the name to keep all text in lang files (see Translations). When another mod already adds a quest with the same id, yours is skipped and the main menu's load error popup names both mods. The returned [`TimbnQuest`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnQuest) starts, readies, completes, resets, or cancels the quest from code.

## How a Quest Runs

| Step | What happens |
| --- | --- |
| Offer | The giver shows a bubble while the quest is on offer, which is once the quests named in `OfferAfter` are done and `OfferWhen` is true. Picking `Accept` starts it. |
| Start without an offer | `StartsOn(event)` starts the quest by itself when the player does something in the game. A quest with no offer and no event is started from code with `quest.Start()`, or right after the quest it `ContinuesFrom` is done. A quest with a `Setup` and none of these starts as soon as `NewRun()` rolls a run. |
| Ready | A stage with a `Talk` is ready as soon as it starts. `ReadyByCode()` also waits for `quest.MarkReady()`, and `ReadyOn(event)` also waits for a game event. Both are kept with the save. |
| Talk | The NPC shows a bubble once the stage's `Needs` are met and the player can pay its `Cost`. Picking `Accept` takes the cost, pays the rewards, plays the `then` lines and runs `OnDone`. |
| NPCs with a menu | On Larry, Herm, the vendors and other NPCs whose main menu has a Leave answer, the talk joins that menu as one answer just above Leave. The talk's bubble opens that menu too. A talk that is only an `Accept` shows the `Accept` text. Any other talk shows its first line if that line is a `PlayerSay`, and otherwise the quest's name. When the lines that follow finish, the menu opens again, as the game does after each answer. |
| Not payable yet | While the player does not meet the talk's `Needs` (or the `Cost` of an `Accept` only talk), the answer shows grayed out in the menu, which is the game's own reminder. |
| NPCs without a menu | The talk is a conversation of its own with `Accept` and `Decline`, or the game's leave answer. Core also does this for an NPC whose menu does not open with a Leave answer, and keeps doing it for that NPC until the player leaves the save. |
| Accept with no text | `Accept()` with no text completes the stage as soon as the lines are done, for something the NPC simply hands over. A stage with a `Cost` needs an answer the player picks. |
| No talk | A stage with no `Talk` moves on as soon as it is ready, and pays its rewards then. Its `DoneLines(then => then.PlayerSay(...))` play at that moment, such as the player thinking aloud on walking into a place. |
| Reset | `quest.Reset()` puts the quest back to not started in the loaded save, for playing it again while testing. Items and money already handed over stay spent. |

The talk is with the quest giver, or the NPC passed to `TalksWith`.

## Starting Another Quest

| Method | What it does |
| --- | --- |
| `OfferAfter(quest)` | Offers the quest only once another quest is completed, and draws it below that quest in the tree. Takes a `TimbnQuest` or the id of a game quest such as `58_port_supplies_order`. |
| `ContinuesFrom(quest)` | Makes the quest the next step of another of your quests, the way the game chains its own quests. It shares that quest's cell in the tree. With no `Offer` or `StartsOn` of its own it starts by itself shortly after that quest is completed. Only a mod quest can be continued. |
| `OfferWhen(condition)` | An extra condition for offering or starting, checked about once a second. |
| `OfferWhenExpression(expression)` | The same condition written as one of the game's script expressions, such as `IsTechUnlocked("wood_basic") && $money >= 500`. |

## Stages

A quest with several steps is a list of named stages in one quest. Whatever you call directly on the quest builder (a description, talk, cost and so on) is the first stage, and each `AddStage` adds the next. A quest with nothing of that kind, like the one below, takes its first `AddStage` as the first stage.

```csharp
var bats = Quests.Add("timbn_larry_bats", TimbnGameIds.Npcs.Larry, "Larry's Bats", quest => quest
    .OfferAfter(wine)
    .DoneDescription("The bats are gone.")
    .Offer(talk => talk
        .Say("There are bats in my grave.")
        .Accept("I'll help.", needs: need => need.OnWeekday(TimbnWeekday.Pride)))
    .AddStage("ask", "Ask Herm what to do about Larry's bats.", stage => stage
        .TalksWith(TimbnGameIds.Npcs.Herm)
        .Talk(talk => talk.Accept("Larry has bats.", then => then.Say("Sleep on it."))))
    .AddStage("sleep", "Sleep on it.", stage => stage.ReadyOn(TimbnQuestEvent.Slept()))
    .AddStage("candles", "Bring Herm 2 candles.", stage => stage
        .TalksWith(TimbnGameIds.Npcs.Herm)
        .Cost(cost => cost.Item("candle_basic", 2))
        .Talk(talk => talk.Accept("Here are the candles.")))
    .AddStage("smoke", "Smoke the bats out after dark.", stage => stage.ReadyByCode())
    .AddStage("tell", "Tell Larry the bats are gone.", stage => stage
        .Reward(reward => reward.Money(500).Reputation(10))
        .Talk(talk => talk
            .Accept("The bats are gone.", then => then
                .Say("Thank you, Keeper.")
                .SayAs(TimbnGameIds.Npcs.Herm, "Told you the smoke would work.")))));

bats.MarkReady("smoke");
```

Core makes each stage its own game quest, continuing the one before in the same spot in the quest tree, so the game's "Quest updated" popup shows at each change. `quest.MarkReady("smoke")` does nothing unless the player is on that stage, so it is safe to call whenever the event occurs. Stage names are saved and are part of the quest ids and text keys of every stage after the first, so keep them stable and add new stages at the end. A stage name can only hold lowercase letters, digits and underscores.

`quest.Stage` is the name of the stage the player is on, and `quest.Status` is in progress from the first stage starting until the last is done. `quest.Complete()` finishes the current stage immediately from code.

A stage takes the same calls as a quest, such as `Icon`, `Phase`, `TalksWith`, `Needs`, `Cost`, `Talk`, `Reward`, `ReadyByCode`, `ReadyOn`, `DoneLines`, `OnDone`, `OnStart`, `OnEnd` and the expression calls. It also has `Name(name)` for a name of its own in the quest tree and log, which otherwise uses the quest's name. The offer, `OfferWhen`, `StartsOn` and `OfferAfter` of the quest belong to its first stage.

## More Quest Calls

| Method | What it does |
| --- | --- |
| `Description(text)`, `DoneDescription(text)` | The quest's text in the log while it is in progress and once it is done. Left out, `DoneDescription` keeps the in progress text. `Description()` with no text reads it from the lang files (see Translations). |
| `Icon(id)`, `Icon(run => id)` | The quest's icon in the tree and log. A quest that continues another uses that quest's icon when left out. |
| `Phase(1 to 5)` | Which of the game's five rings the quest's node shows while in progress. Left out, a chain grows a ring with each step. |
| `OnStart(action)`, `OnEnd(action)` | Run when the quest starts (and again when a save loads with it in progress) and when it stops being in progress. Use them for effects the save does not keep, such as a marker. |
| `OnStartExpression(expr)`, `OnDoneExpression(expr)` | One of the game's script expressions, run by the game when the quest starts or is completed, such as `AddPerk("perk_x")` or `RevealTech("tech_x")`. Use them for changes the save keeps. A typo fails when the quest is added. |
| `OnDone(action)` | Runs once the stage is done, after the lines that follow `Accept`. |

## Reading Quest State

| Member | What it gives |
| --- | --- |
| `quest.Id` | The quest id, which is also its first stage's id. |
| `quest.Status` | A `QuestStatus` for the loaded save, or null when no save is loaded. |
| `quest.Stage` | The name of the stage the player is on, or null. |
| `TimbnQuests.StatusOf(id)` | The status of any quest in the loaded save, the game's own included, or null. |
| `TimbnQuests.FireStoryTrigger(type, id)` | Fires one of the game's story triggers as if the player had just built that building or won that fight. It can finish a game quest whose step was done before the quest started. `Events.StoryTrigger` hears it too. |

```csharp
if (TimbnPlayer.HasBuilt("zmb_choir_unlock_s"))
    TimbnQuests.FireStoryTrigger(GlobalEventsSystem.Event.Type.BuildBuilding, "zmb_choir_unlock");
```

## Repeating Quests

A quest whose details change each time, such as which villager asks and for what, fills a [`TimbnQuestRun`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnQuestRun) in its `Setup`. Text refers to a run value by its name in braces, such as `{dish}`, and `Giver`, `TalksWith`, `Icon`, `Needs`, `Cost` and `Reward` have forms that read the run.

```csharp
var craving = Quests.Add("timbn_craving", "A Villager's Craving", quest => quest
    .Repeatable()
    .ExpiresAfter(3)
    .Setup(run =>
    {
        var villagers = TimbnNpcs.GroupMemberIds(TimbnGameIds.Groups.VillagersDayAll)
            .Where(npc => !TimbnQuests.HasQuestRun(npc)).ToList();
        if (villagers.Count == 0)
            return false;

        run.Npc("villager", villagers[UnityEngine.Random.Range(0, villagers.Count)]).Item("dish", "cook_muffin");
        return true;
    })
    .Giver(run => run.NpcId("villager"))
    .Description("Bring {villager} {dish}.")
    .Offer(talk => talk.Say("Could you bring me {dish}?").Accept("I'll bring it.").Decline("Not now."))
    .Cost((cost, run) => cost.Item(run.Item("dish")))
    .Reward((reward, run) => reward.Money(TimbnItems.SalePrice(run.Item("dish"))))
    .Talk(talk => talk.Accept("Here you go.", then => then.Say("Thank you!")).Decline("Not yet.")));

Events.NewDayStarted(_ =>
{
    if (UnityEngine.Random.Range(0, 7) == 0)
        craving.NewRun();
});
```

| Call | What it does |
| --- | --- |
| `Setup(run => bool)` | Fills a new run when `NewRun()` is called. Return false when nothing fits right now. Until a run is rolled the quest is never offered or started. |
| `Repeatable()` | Puts every stage back to not started once the last is done, and drops the quest from the tree and log until it starts again. No other quest can use `OfferAfter` or `ContinuesFrom` on it. |
| `ExpiresAfter(days)` | Cancels a run that goes that many game days unfinished, whether or not the player took the quest on. |
| `OnCancel(run => ...)` | Runs when a run is canceled, before the quest goes back to not started, such as to take back something the player was handed. |
| `quest.NewRun()` | Rolls a new run and returns true when one rolled. It does nothing while a run is going or until the offer rules allow it, and throws for a quest with no `Setup`. A quest with no offer or trigger starts as soon as the run rolls. |
| `quest.Cancel()` | Ends the current run early. |
| `quest.CurrentRun` | The run now going, or null. |
| `TimbnQuests.HasQuestRun(npcId)` | Whether any mod quest's current run names this NPC, so a `Setup` can skip NPCs who already have a quest. |

A run holds named NPCs (`Npc`), items (`Item`), numbers (`Number`) and text keys (`Text`), set in `Setup` and read with `Npc(name)` (or `NpcId(name)`), `Item`, `Number` and `Text`. A value name is lowercase letters, digits and underscores and starts with a letter. `run.Day` is the game day the run was rolled on. In text an NPC shows as their name, an item as its name, a text value as its line, and a number as itself. A run is kept with the save and ends when the quest is done or canceled.

## Needs and Costs

`Needs` is what must be true and is never taken. `Cost` is what the player pays and is taken when they pick the answer.

| Needs method | What the answer shows |
| --- | --- |
| `Item(id, count)` | A lock until the player carries the items. |
| `Resource(id, amount)`, `Money(amount)` | A lock until the player has the resource. |
| `Reputation(amount)` | A lock until the player has that much favor with the NPC they are talking to. Herm and the village head share one reputation. |
| `Gratitude(amount)` | The gratitude a town vendor asks for. |
| `OnWeekday(TimbnWeekday.Pride)` | Only answerable on that day of the week. |
| `OrderCompleted("quest_order_iron")` | Waits for a vendor order to be finished this week. |

| Cost method | What the answer shows |
| --- | --- |
| `Item(id, count)` | Items the player hands over, with a `0/3` style count. |
| `Resource(id, amount)`, `Money(amount)` | A resource the player pays, such as `money`. |

A quest's `Needs` and `Cost` belong to its `Talk`, and its `Accept` answer shows them. An offer's `Accept` has its own `needs` and `cost` parameters, so an offer can be locked to a day or cost money to take. The cost is taken when the answer is picked and handed back if the quest can't start.

```csharp
.Offer(talk => talk
    .Say("The ferryman wants to be paid upfront.")
    .Accept("Here's the fare.", then => then.Say("Hop in."), cost: cost => cost.Money(200))
    .Decline("Maybe later."))
```

## Rewards

| Method | What the player gets |
| --- | --- |
| `Money(amount)` | Money in the game's raw units, where 100 is one silver coin. |
| `Item(id, count)` | An item. What doesn't fit drops at the player's feet. |
| `Resource(id, amount)` | Any other resource. |
| `Reputation(amount)` | Reputation with the NPC of the talk, or the giver for a stage with no talk. The game's own quests give 10. |
| `ShowItem(id, count)`, `ShowResource(id, amount)` | The same reward picture without giving it, for a reward your own step hands out. |

Rewards show on the `Accept` answer. Money and reputation also show on the quest window's reward line.

## Game Events

| Event | When it fires |
| --- | --- |
| `TimbnQuestEvent.Crafted(craftId)` | The player finishes a craft. |
| `TimbnQuestEvent.Built(objectId)` | The player finishes building something. |
| `TimbnQuestEvent.EnteredZone(zoneTag)` | The player walks into one of the game's marked areas. |
| `TimbnQuestEvent.EnteredWorldZone(worldZoneId)` | The player walks into a world zone, such as the graveyard. |
| `TimbnQuestEvent.WonFight(fightId)` | The player wins a fight. |
| `TimbnQuestEvent.Slept()` | The player wakes up after sleeping. It counts the moment the sleep ends, just before the game writes its wake up save, so a reload keeps it. Any day change during the sleep has already happened by then. Loading a save does not count (the game's own wake up event fires on every load, so Core ignores it), and neither does a sleep the game refuses because the player has full energy. |
| `TimbnQuestEvent.UsedItem(itemId)` | The player uses an item. |

## Conversations

A quest's conversations and a plain talk share the same steps. A plain talk is added with `Dialog.AddTalk` and takes the text itself. `Dialog.AddDynamicTalk` builds the talk fresh each time the player talks, so ordinary C# can choose and fill in the lines from the game's state.

```csharp
Dialog.AddTalk(TimbnGameIds.Npcs.Larry, "my_larry_trade", () => true, talk => talk
    .Cinematic()
    .Say("Got any boards?")
    .SayAs(TimbnGameIds.Npcs.Herm, "Don't sell him any.")
    .Ask(ask => ask
        .Answer("Here, take two.", then => then.WaitRealSeconds(1f).Say("Thanks!"), cost: cost => cost.Item("flitch", 2))
        .Answer("Not today.")));
```

| Step | What it does |
| --- | --- |
| `Say(line)` | A line in the NPC's speech bubble. |
| `SayAs(npc, line)` | A line from another NPC, skipped with a warning when that NPC isn't in the scene. |
| `PlayerSay(line)` | A line in the player's speech bubble. |
| `Line(name)` | A line with its text only in the mod's lang files (see Translations). |
| `Named(name)` | Names the line before it, so its key is the talk id plus the name instead of a number. |
| `Ask(ask => ask.Answer(...))` | Shows answer choices and waits for one. The picked answer's lines play, then the talk carries on. |
| `When(condition, then, otherwise)` | Branches on a `Func<bool>` when the conversation reaches it. |
| `When(expression, then, otherwise)` | The same branch, picked by one of the game's script expressions such as `$money >= 500`. |
| `RunCode(action)` | Runs your code. |
| `RunExpression(expression)` | Runs one of the game's script expressions against the NPC being talked to. |
| `WaitRealSeconds(seconds)` | Pauses, in real seconds. |
| `Cinematic()` | Shows the game's black story bars until the talk ends. |
| `Give(item, count)` | Gives the player items. What doesn't fit drops at their feet. |
| `Take(item, count)` | Takes items from the player without asking, as many as they have up to the count. |

Lines go before the answers. Put what follows an answer in that answer's `then`. A quest's offer takes `Accept` and `Decline`, and a quest's talk takes `Accept`, `Decline` and `Resume` (what the NPC says on a return visit). Every mod talk turns the NPC toward the player, stops their walking, and restores both when the talk ends.

`AddTalk` writes its lines once, when you call it, so a C# `if` inside it only runs then. Pick between lines with `When`, or use `AddDynamicTalk`, which writes them again each time. Its numbered keys can land on a different line from one talk to the next, so give any line a translator needs a name with `Named`. Both return a handle that removes the talk early, and the talk goes away when the mod unloads anyway.

A line can carry values. `Say("You have %1 boards.", boards)` fills the game's `%1` and `%2` placeholders, and a `Func<object>` value is worked out when the line is shown.

A line whose key has no text in the current language is logged once as a warning, since the game would show the raw key.

### Answer Extras

An answer is `Answer(text, then, needs, cost, reward, key)`. Needs, cost and reward use the same builders as quests. The answer is grayed out until the player can meet them. When every answer carries a price, a lock or a reward, Core adds a leave answer so a player who can't pay is never stuck.

```csharp
talk.Ask(ask => ask
    .Answer("Trade sticks for boards.", cost: cost => cost.Item("stick"), reward: reward => reward.Item("flitch", 2))
    .Answer("No thanks."));
```

### Menu Answers

`Dialog.AddMenuAnswer(targetId, answerId, isOffered, text, then, needs, cost, buttonColor)` adds an answer to the menu an NPC or object opens when the player interacts, next to the game's own answers and above Leave. `targetId` is an NPC id or the id of an object with a menu, such as `royal_mailbox`. `answerId` must be unique within your mod and is also the answer's text key, so the lines that follow are `answerId_1`, `answerId_2` and so on. Two mods can use the same answer id on one NPC without getting in each other's way. Picking the answer takes its cost, plays the lines that follow, and shows the menu again. `isOffered`, `needs` and `cost` are read each time the menu opens, so a price can follow a setting. Answers only join the main menu, the one the NPC opens when the player talks to it and that has a Leave answer. If the target has no such menu, nothing is added there. The text can color any part of itself with the game's highlight in square brackets or a color tag, and `buttonColor` gives the answer's button a color of its own.

```csharp
Dialog.AddMenuAnswer(TimbnGameIds.Npcs.TavernOwner, "my_mod_hire", () => !_booked, "Play at my [tavern] tonight.",
    then => then.Say("I'll bring my violin.").RunCode(() => _booked = true),
    cost: cost => cost.Money(500));

Dialog.AddMenuAnswer("royal_mailbox", "my_mod_burn", () => true, "<color=#ff6050>Burn the letters</color>",
    buttonColor: new Color(0.45f, 0.12f, 0.1f));
```

### One-Shot Talks

`AddTalk(..., new TimbnTalkOptions { Once = true })` stops the NPC from offering the talk once the player starts it in that save. `AddDynamicTalk` takes the same options. The talk is marked when it starts and written when the game next saves, so a player who quits without saving sees it again. `Dialog.HasSeenTalk("id")` checks whether the player has seen it, and `Dialog.ResetTalk("id")` offers it again. [`TimbnTalkOptions`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnTalkOptions) also takes a `BubbleColor` for that one talk. A talk id must be unique across NPCs, and reusing one on another NPC throws.

## Translations

Text added with `Text.Add` appears in every language. To let players translate a mod, put its text in files instead. Core loads them when the mod starts, so there is no call to make. It reads every `lang/<language id>.txt` next to the mod's DLL (`en`, `de`, `fr`, `pt-br`, `es`, `ru`, `pl`, `ja`, `zh_cn`, `ko`, or `tr`), one `key = text` per line. Text from a lang file always overrides text added in code for the same key, so a translation can replace any quest line.

```text
# lang/de.txt
timbn_larry_wine_talk_accept = Hier ist dein Wein.
timbn_larry_wine_talk_accept_1 = Fühlt sich großartig an.
```

Lines starting with `#` are comments and `\n` is a line break. The game's own markup works too. `(*slv*)` draws the named icon (`slv`, `gld` and `brz` are the silver, gold and bronze coins), `#(other_key)` pastes another key's text, and `@(rule)` fills in one of the game's replacement words. The game shows the file for its current language and falls back to `en.txt` for any key a translation lacks. A translator copies `en.txt` to `de.txt` and translates it without needing a rebuild. Ship the files with the mod from its csproj.

```xml
<ItemGroup>
  <None Include="lang\*.txt" CopyToOutputDirectory="PreserveNewest" />
</ItemGroup>
```

In code, [`TimbnMarkup`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnMarkup) writes the markup for you, so you don't have to remember it. `TimbnMarkup.Icon("slv")` gives `(*slv*)`, `TimbnMarkup.Weekday(TimbnWeekday.Pride)` gives the day icon `(*day_pride*)`, and `TimbnMarkup.Localized("npc_herm")` gives `#(npc_herm)`, Herm's name in the player's language. Use them inside an interpolated line, such as `$"Come back on {TimbnMarkup.Weekday(TimbnWeekday.Pride)}."`.

To keep the text out of code entirely, write the quest with names instead of text. `Quests.Add` without a name reads the name from the quest id's key, `Description()` reads `quest_open_<id>_d`, `Line("ask")` is a line read from the conversation's key plus `_ask`, and `Accept` and `Decline` without text read their usual keys. A key missing from `en.txt` fails when the quest is added.

```csharp
Quests.Add("my_craving", quest => quest
    .Giver(run => run.NpcId("villager"))
    .Description()
    .Offer(talk => talk
        .Line("ask")
        .Accept(then => then.Line("thanks"))
        .Decline())
    ...);
```

```text
# lang/en.txt
my_craving = A Neighbor's Craving
quest_open_my_craving_d = Cook {dish} for the NPC who asked for it.
my_craving_offer_ask = I've a craving for {dish}. Could you cook me some?
my_craving_accept = I'll cook it.
my_craving_accept_thanks = Bless you, I'll be waiting.
my_craving_decline = Not now.
```

## Notes

> [!NOTE]
> **Quest keys.** A quest's lines get keys named after its id, numbered in the order they are written. A stage after the first uses the quest id plus its name as its id.

| Text | Key |
| --- | --- |
| Name | `<id>` |
| Quest description | `quest_open_<id>_d` |
| Quest description once done | `quest_closed_<id>_d` |
| Offer lines | `<id>_offer_1`, `<id>_offer_2` |
| Accept and its lines | `<id>_accept`, `<id>_accept_1` |
| Decline and its lines | `<id>_decline`, `<id>_decline_1` |
| Talk lines | `<id>_talk_1`, `<id>_talk_2` |
| Talk Accept and its lines | `<id>_talk_accept`, `<id>_talk_accept_1` |
| Talk Decline and Resume | `<id>_talk_decline`, `<id>_talk_resume` |
| Done lines | `<id>_done_1` |

> [!NOTE]
> **Game talks come first.** An NPC keeps waiting dialog in a queue, and only the front dialog plays when spoken to. Mod talks always wait behind the game's own, including ones the game adds later, and step aside while the NPC has a game quest ready to hand in, so they never delay the story. A mod talk shows a tinted copy of the game's speech bubble over the NPC, so players can tell it apart. It is gold by default, and each mod can pick its own color with `Dialog.BubbleColor = new Color(0.6f, 0.8f, 1f);`.
>
> **Placement is automatic.** Started mod quests that don't follow a game quest are packed into a band of rows at the top of the tree, and every game quest moves down by the band's height plus one spacer row. A group of mod quests that follows a game quest goes into the nearest free spot below it instead, linked to every quest in that parent's cell (the game stacks several quests in one cell and shows only the latest). Positions are never saved, so the layout is simply redone whenever a quest registers, unregisters or starts. A mod quest that hasn't started yet isn't in the tree.
>
> **Chains share a cell.** A quest that `ContinuesFrom` another mod quest goes in that quest's cell, the way the game stacks its own chains, and the cell shows the newest step. Only the first quest chained to a given quest shares its cell, so a second one gets its own node below with a line to it. A quest that only comes `OfferAfter` another always gets its own node below.
