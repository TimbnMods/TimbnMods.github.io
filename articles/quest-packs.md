# Quest Packs

A quest pack is a set of quests and conversations written as JSON files, with no code. The Timbn Quest Packs mod reads them and calls the same quest builder a C# mod calls (see [Quests, Dialog, and Text](quests.md)), so a pack can do nearly everything the builder does except run code of its own. Small sets of conditions, actions and values stand in for that code.

> [!NOTE]
> Quest Packs is its own mod and needs Timbn Core 2.0.0. Players install it to use any pack, and a mod that ships a pack should tell them so.

A pack cannot make repeatable quests or quest runs yet (`Repeatable`, `Setup`, `NewRun` and `{name}` values), since those need code to fill each run.

## Where Packs Live

There are two kinds of pack. They use the same files and differ in who owns them.

### A standalone pack

A folder anywhere under `BepInEx/plugins` (at any depth) that has a `pack.json` and a `quests` folder and no DLL. That covers a Nexus or Vortex install, r2modman's `plugins/<Team-Package>/` folders and the Steam Workshop folders WorkshopSync sets up, with no change to any of them.

```text
BepInEx/plugins/LarrysWine/
  pack.json
  quests/       any number of .json files
  lang/         en.txt, de.txt and so on, the same format as a mod's lang folder
  sprites/      PNGs, each added under the pack id plus its file name
```

The Quest Packs mod owns these. It reads them when the game starts and again each time the main menu opens, but only if a pack was added or removed or a file in one changed (it compares file names, sizes, and timestamps). To try an edit, change the file, go back to the main menu and load the save again. A folder with a DLL in it is never a standalone pack.

### A mod's pack

A `quests` folder next to a Timbn mod's DLL. The mod owns it, so it loads when the mod starts (right after the mod's own `lang` folder) and goes away when the mod unloads. That means F6 reloads it for a mod in `scripts`.

Its id is the mod's GUID in lowercase, with anything other than letters, digits and underscores turned into an underscore, so `Timbn.GraveyardKeeperTwo.Examples` becomes `timbn_graveyardkeepertwo_examples`. A `pack.json` next to the DLL can name a shorter id and a bubble color. Without one, the bubble color is the mod's `Dialog.BubbleColor`. The mod's own `lang` folder already translates the pack. A `sprites` folder next to the DLL works the same as in a standalone pack.

The mod's players need the Quest Packs mod installed, since Core itself does not read packs.

## A Small Pack

This pack is a `pack.json` and one quest file. Larry asks for a wine, then Herm does, and Larry mentions it once the first is done.

```json
{ "id": "larrys_wine", "name": "Larry's Wine", "bubbleColor": "#9ad0ff" }
```

```json
{
  "$schema": "https://raw.githubusercontent.com/TimbnMods/Timbn.GraveyardKeeperTwo/main/src/Timbn.GraveyardKeeperTwo.QuestPacks/quests.schema.json",
  "quests": [
    {
      "id": "ask",
      "giver": "npc_larry",
      "name": "Wine for Larry",
      "icon": "quest_icon_larry_portal",
      "description": "Bring Larry a bottle of wine.",
      "done": "Larry got his wine.",
      "offerWhen": "questStarted:6_intro_guards_burial",
      "offer": {
        "lines": [ "Can you spare a bottle of wine?" ],
        "accept": "Sure.",
        "decline": "Not now."
      },
      "cost": { "items": { "grape_wine:1": 1 } },
      "reward": { "money": 100, "reputation": 5 },
      "talk": {
        "accept": { "text": "Here you go.", "then": [ "Lovely. Herm might want one too." ] }
      }
    },
    {
      "id": "herm",
      "giver": "npc_herm",
      "name": "Wine for Herm",
      "description": "Bring Herm a bottle of wine.",
      "after": "ask",
      "offer": {
        "lines": [ "Larry says you have good wine. Can I have a bottle?" ],
        "accept": "Sure.",
        "decline": "No."
      },
      "cost": { "items": { "grape_wine:1": 1 } },
      "reward": { "items": { "nails_copper": 3 } },
      "talk": {
        "accept": {
          "text": "Here.",
          "then": [
            {
              "when": "carries:grape_wine:1",
              "then": [ { "say": "Thanks. You still have %1 left.", "values": [ "itemCount:grape_wine:1" ] } ],
              "otherwise": [ "Thanks." ]
            }
          ]
        }
      }
    }
  ],
  "talks": [
    {
      "id": "tip",
      "npc": "npc_larry",
      "when": { "all": [ "questCompleted:ask", "questNotStarted:herm" ] },
      "once": true,
      "lines": [ "Herm likes wine too. Go see him." ]
    }
  ]
}
```

## pack.json

Only a standalone pack needs one. A mod's pack uses it to override its id or bubble color.

| Field | What it does |
| --- | --- |
| `id` | Required. Lowercase letters, digits and underscores. Never change it once players have the pack, since saves keep progress under the quest ids made from it. |
| `name` | A readable name for the pack. Optional, and it defaults to the id. |
| `bubbleColor` | The color of the pack's speech bubbles, as `#RRGGBB` or `#RRGGBBAA`. Core's default color when left out. |

Any other field is an error and the pack is skipped. When two standalone packs share an id, only the first loads (folders are read in path order) and the log says so.

## Ids and Text Keys

Every quest, talk and sprite id a pack writes gets the pack id in front, so `"id": "ask"` in the `larrys_wine` pack is `larrys_wine_ask` in the game, in saves and in text keys. Ids hold lowercase letters, digits and underscores.

Wherever a quest id is expected (`after`, `continues`, the quest conditions and `startQuest`), a short id that names a quest in the same pack means that quest. Anything else is used as written, so it can be a game quest such as `6_intro_guards_burial` or another pack's full id. For a quest with `stages`, `after`, `continues` and `questCompleted` mean its last step, so they wait for the whole quest, while `questStarted`, `questNotStarted` and `startQuest` mean its first. `continues` only takes a quest from the same pack.

An `icon` that matches a PNG in the pack's `sprites` folder (without `.png`) means that sprite. Anything else is a game sprite id such as `quest_icon_larry_portal`.

Text keys follow the quest builder's rules, using the full ids. For the quest above they are the following.

| Text | Key |
| --- | --- |
| Name | `larrys_wine_ask` |
| Offer lines | `larrys_wine_ask_offer_1`, `_2` and so on |
| Offer answers | `larrys_wine_ask_accept`, `larrys_wine_ask_decline` |
| Lines after an offer answer | `larrys_wine_ask_accept_1` and so on |
| Talk lines | `larrys_wine_ask_talk_1`, `_2` and so on |
| Talk answers | `larrys_wine_ask_talk_accept`, `_talk_decline`, `_talk_resume` |
| Lines after the talk's accept | `larrys_wine_ask_talk_accept_1` and so on |

A later stage uses its own id in place of `ask`, such as `larrys_wine_ask_deliver_talk_accept`. A talk's lines are `larrys_wine_tip_1`, `_2` and so on in the order written. The full list of quest keys is under Notes in [Quests, Dialog, and Text](quests.md#notes).

Lines are numbered in the order written, so adding a line in the middle of a talk that has shipped moves the numbered keys after it, and a translation then lands on the wrong line. A `say`, `playerSay`, `sayAs` or answer can carry `"key": "ask"` to be `larrys_wine_tip_ask` instead. A keyed line takes no number, so it never moves the others. A key holds lowercase letters, digits and underscores and starts with a letter.

### Translations

A standalone pack's `lang` folder holds one `<language id>.txt` per language, one `key = text` per line, using the keys above. Lines starting with `#` are comments and `\n` is a line break. The text in the JSON is the default, and a lang file overrides it for its language. Ship an `en.txt` too so translators have something to copy, since the log warns about any key that a lang folder has no English line for. The language ids and markup are the same as a mod's (see Translations in [Quests, Dialog, and Text](quests.md#translations)).

```text
# lang/de.txt
larrys_wine_ask = Wein für Larry
larrys_wine_ask_offer_1 = Kannst du eine Flasche Wein entbehren?
larrys_wine_ask_accept = Klar.
```

## A Quest File

Each file in `quests` is one JSON object with two optional lists. Files are read in name order, and a quest or talk can sit in any of them.

```json
{
  "$schema": "...",
  "quests": [ ],
  "talks": [ ]
}
```

JSON comments (`//` and `/* */`) are allowed, so authors can leave notes. Any other top level field is reported and ignored. Ids must be unique across all of a pack's files, and a second quest or talk with the same id is skipped.

### Schema

`quests.schema.json` and `pack.schema.json` sit next to the Quest Packs code, and each release carries the schema that matches it. A file points at one with a `$schema` line, which the mod ignores. VS Code and most editors then give autocomplete, hover text and red underlines for misspelled fields, wrong types and unknown conditions, events and actions.

- `https://raw.githubusercontent.com/TimbnMods/Timbn.GraveyardKeeperTwo/main/src/Timbn.GraveyardKeeperTwo.QuestPacks/quests.schema.json`
- `https://raw.githubusercontent.com/TimbnMods/Timbn.GraveyardKeeperTwo/main/src/Timbn.GraveyardKeeperTwo.QuestPacks/pack.schema.json`

## Quests

A quest is an object in the `quests` list. `id`, `giver` and `name` are required. Everything else is optional, but a quest has to be able to finish, so give it a `talk`, `readyOn` or `readyWhen`.

| Field | What it does | C# call |
| --- | --- | --- |
| `id` | The quest's id, before the pack id is added. | `Quests.Add` |
| `giver` | The NPC who gives it, such as `npc_larry`. | `Quests.Add` |
| `name` | Its name in the quest tree. | `Quests.Add` |
| `icon` | The quest's icon, a game sprite id or a PNG from the pack. A `continues` step with none uses the icon of the step it continues. | `Icon` |
| `phase` | The ring in the quest tree, 1 to 5. | `Phase` |
| `description` | The quest window text while in progress. | `Description` |
| `done` | The quest window text once done. With stages it goes to the last step. | `DoneDescription` |
| `after` | One quest id or a list. The quest is offered once they are all done. | `OfferAfter` |
| `continues` | A quest in the same pack. This quest becomes its next step and starts by itself when that one is done. | `ContinuesFrom` |
| `offerWhen` | A [condition](#conditions) that has to be true to offer or start the quest. | `OfferWhen` |
| `offer` | The conversation that offers the quest (see [Offer](#offer)). | `Offer` |
| `startsOn` | An [event](#events) that starts the quest. | `StartsOn` |
| `npc` | The NPC the step's `talk` is with. Defaults to the giver. | `TalksWith` |
| `needs` | What has to be true before the NPC offers the step's `talk`. It is checked and never taken. | `Needs` |
| `cost` | What the player pays to finish the step, taken when they pick the talk's `accept` answer. | `Cost` |
| `talk` | The conversation that finishes the step (see [Quest talk](#quest-talk)). | `Talk` |
| `reward` | What the player gets, paid when the talk's `accept` is picked or, with no talk, when the step is done. | `Reward` |
| `readyOn` | An [event](#events) that readies the step. | `ReadyOn` |
| `readyWhen` | A [condition](#conditions) checked about once a second while the quest is in progress. When it turns true the step is ready. | `ReadyByCode` |
| `then` | [Lines](#lines) that play when a step with no `talk` is done. | `DoneLines` |
| `onDone` | One [action](#actions) or a list, run when the step is done. | `OnDone` |
| `runOnStart`, `runOnDone` | One game script expression or a list, run by the game itself when the quest starts or completes. Use them for changes the save keeps. | `OnStartExpression`, `OnDoneExpression` |
| `stages` | A list of the next steps (see [Stages](#stages)). | `AddStage` |

The quest builder's own rules apply and are reported the same way. A step needs a `talk`, `readyOn` or `readyWhen`, and cannot have both `readyOn` and `readyWhen`. An offer and a talk each need an `accept`. `needs` and `cost` need a `talk`. A step with a `talk` keeps its last lines in the talk instead of `then`.

### Needs and cost

`needs` and `cost` are objects. Money is in the game's raw units, where 100 is one silver coin. An item id can hold a colon for its quality, so `grape_wine:1` is a wine of quality 1.

| Field | In | What it is |
| --- | --- | --- |
| `items` | `needs`, `cost` | Items and counts, written as `{ "grape_wine:1": 3 }`. |
| `resources` | `needs`, `cost` | Resource ids and amounts, such as `{ "tech_red": 5 }`. |
| `money` | `needs`, `cost` | A whole number. |
| `reputation` | `needs` | Reputation with the NPC the player is talking to. |
| `happiness` | `needs` | The gratitude a town vendor asks for. |
| `onDay` | `needs` | One of Gluttony, Sloth, Lust, Envy, Pride, or Wrath. |
| `orderCompleted` | `needs` | A town vendor's order finished this week, such as `quest_order_iron`. |

The player keeps what `needs` checks. `cost` is taken when the answer is picked, shows on the answer and grays it out until the player can pay. See Needs and Costs in [Quests, Dialog, and Text](quests.md#needs-and-costs).

### Reward

| Field | What the player gets |
| --- | --- |
| `money` | A whole number of the game's raw units. |
| `items` | Items and counts. What does not fit drops at the player's feet. |
| `resources` | Resource ids and amounts. |
| `reputation` | Reputation with the NPC of the talk, or the giver for a step with no talk. |
| `shows`, `showsResources` | Items or resources drawn on the answer but not handed out, for a reward a `give` line hands out itself. |

A talk's `accept` shows the reward and the game pays it when the answer is picked.

### Offer

```json
"offer": {
  "lines": [ "Can you spare a bottle of wine?" ],
  "accept": "Sure.",
  "decline": "Not now."
}
```

`lines` are the [lines](#lines) the NPC says first. `accept` is required. `accept` and `decline` are either the answer text alone or `{ "text": "...", "then": [lines] }`, where `then` plays after the answer. The offer's `accept` can also hold `needs` and `cost`, so an offer can be locked to a day or cost money to take. The cost is given back if the quest cannot start.

### Quest talk

The conversation that finishes a step. It is with the step's `npc`, or the giver. Its `lines` play, then the player picks `accept`, which takes the step's `cost`, gives its `reward` and completes the step, or `decline` to come back later.

| Field | What it does |
| --- | --- |
| `lines` | [Lines](#lines) the NPC says first. |
| `accept` | Required. The answer text alone, `{ "text", "then" }`, or `true`. |
| `decline` | The answer text alone or `{ "text", "then" }`. |
| `resume` | A line the NPC says in place of `lines` when the player comes back after hearing them once. An opening `playerSay` still plays first. |

`"accept": true` has no answer to pick and completes the step as soon as the `lines` are done, for something the NPC simply hands over. That talk needs `lines`, cannot have `decline` or `resume`, and its step cannot have a `cost`. The cost sits on the step, so a `cost` inside `accept` is reported as not belonging there.

When the NPC has a menu of its own, as Herm and Larry do, the talk joins it as one answer.

```json
"npc": "npc_herm",
"cost": { "money": 200 },
"talk": {
  "lines": [ { "playerSay": "Larry wants a bottle of quality wine." }, "That'll cost you 2 silver." ],
  "accept": { "text": "A little steep, but ok.", "then": [ "Come back on Pride." ] },
  "decline": "I don't have the money right now.",
  "resume": "Here to pay the 2 silver?"
}
```

## Stages

A quest of several steps can be written as one quest with `stages`, the same as the builder's `AddStage`. Each stage needs an `id` and a `description`, and takes any of `name`, `icon`, `phase`, `done`, `npc`, `needs`, `cost`, `talk`, `reward`, `readyOn`, `readyWhen`, `then`, `onDone`, `runOnStart` and `runOnDone`. An `offer`, `offerWhen`, `startsOn`, `after` or `continues` on a stage is reported and ignored, since those belong to the first step. Stage ids hold lowercase letters, digits and underscores.

Core makes each step its own game quest continuing the one before. When the quest has step fields of its own (`description`, `phase`, `npc`, `needs`, `cost`, `talk`, `reward`, the ready fields, `then`, `onDone` or the run expressions), it is the first step and keeps the quest's id, and each stage is a later step whose id is the quest id plus the stage id, such as `larrys_wine_ask_deliver`. Otherwise the first stage is the first step and keeps the quest's id, and the quest's `offer`, `offerWhen`, `startsOn`, `after` and `icon` belong to it. The quest's `done` goes to the last step either way.

```json
{
  "id": "ask", "giver": "npc_larry", "name": "Gold Wine",
  "offer": { "lines": [ "Could you find me gold wine?" ], "accept": "Sure." },
  "description": "Pay Herm for the gold wine.",
  "npc": "npc_herm",
  "cost": { "money": 200 },
  "talk": {
    "lines": [ { "playerSay": "Larry wants gold wine." }, "That'll be 2 silver." ],
    "accept": "Here's the silver.",
    "decline": "Not now."
  },
  "done": "Larry got his gold wine.",
  "stages": [
    {
      "id": "deliver",
      "description": "Bring Larry the gold wine.",
      "cost": { "items": { "grape_wine:3": 1 } },
      "talk": { "accept": "Here's your wine." }
    }
  ]
}
```

A stage after the first is saved under its id, so keep stage ids stable and add new stages at the end.

## Lines

Lines are a list of steps. A plain string is a line the NPC says, so the simplest conversation is `["Hello.", "Bring me wine."]`. Lines go before the answers, and what follows an answer goes in that answer's `then`.

| Step | What it does |
| --- | --- |
| `"Some text"` or `{ "say": "...", "values": [...], "key": "..." }` | The NPC says it. |
| `{ "playerSay": "...", "values": [...], "key": "..." }` | The player says it. |
| `{ "sayAs": "npc_herm", "text": "...", "key": "..." }` | Another NPC says it, skipped when they are not around. |
| `{ "give": "flitch", "count": 2 }` | Gives items. `count` defaults to 1. What does not fit drops at the player's feet. |
| `{ "take": "flitch", "count": 2 }` | Takes items. `count` defaults to 1. |
| `{ "wait": 1.5 }` | Pauses, in real seconds. |
| `{ "cinematic": true }` | The game's black story bars until the conversation ends. |
| `{ "when": <condition>, "then": [...], "otherwise": [...] }` | Picks lines by the game's state. `otherwise` is optional. |
| `{ "ask": [answers] }` | Answer choices. Each is a string or `{ "text", "then", "needs", "cost", "reward", "key" }`. |
| `{ "do": <action or list> }` | Runs [actions](#actions). |
| `{ "run": "AddRep(\"npc_larry\", 5)" }` | Runs a game script expression against the NPC being talked to. |

A step object can only hold the fields listed for it, and anything else is reported and ignored.

### Values

`values` fill a line's `%1`, `%2` and so on, worked out when the line shows.

| Value | What it shows |
| --- | --- |
| `itemCount:<item>` | How many of the item the player carries. |
| `ownedCount:<item>` | How many they have anywhere, chests included. |
| `resource:<id>` | How much of the resource they have. |

Any other string or a number shows as written.

## Conditions

A condition is a string `name:argument`, an object such as `{ "carries": "grape_wine:1", "count": 3 }` when it needs a count, or `{ "all": [...] }`, `{ "any": [...] }` and `{ "not": ... }` around others. An item id can hold a colon for its quality, which is why a count goes in the object form. For `resource` and `reputation`, the number can be written as `amount` in the object form.

| Condition | True when |
| --- | --- |
| `questStarted:<id>` | The quest is in progress or done. |
| `questCompleted:<id>` | The quest is done. |
| `questNotStarted:<id>` | The quest is neither in progress nor done. |
| `carries:<item>` | The player carries at least one (or `count`). |
| `owns:<item>` | The player has at least one (or `count`) anywhere, chests included. |
| `hasTech:<id>` | The tech is researched. |
| `resource:<id>:<amount>` | The player has at least that much of the resource, such as `resource:money:500`. |
| `reputation:<npc>:<amount>` | The player has at least that much reputation with the NPC. |
| `day:<day>` | It is that day of the week, one of Gluttony, Sloth, Lust, Envy, Pride, or Wrath. |
| `time:<from>-<to>` | The game hour is in the range, such as `time:18-24`. A range like `time:22-4` wraps past midnight. |
| `expr:<expression>` | One of the game's script expressions is true, such as `expr:$money >= 500` or `expr:IsTechUnlocked("wood_basic")`. |

## Actions

An action is a string `name:argument` or an object with a `count`. A list of actions runs in order.

| Action | What it does |
| --- | --- |
| `startQuest:<id>` | Starts a mod quest that can start now. The log notes when nothing started. |
| `give:<item>`, `take:<item>` | Gives or takes items, with `count` in the object form. |
| `addResource:<id>:<amount>` | Adds to a resource, such as money. |
| `reputation:<npc>:<amount>` | Reputation with the NPC. |
| `run:<expression>` | Runs one of the game's script expressions, such as `run:AddPerk("perk_x")`. |

## Events

Events are strings. Only `slept` takes no argument.

| Event | Fires when |
| --- | --- |
| `slept` | The player wakes up after sleeping. |
| `crafted:<craft>` | The player finishes the craft. |
| `built:<object>` | The player finishes building the object. |
| `enteredZone:<tag>` | The player walks into one of the game's marked areas. |
| `walkedInto:<world zone>` | The player walks into a world zone, such as the graveyard. |
| `wonFight:<fight>` | The player wins the fight. |
| `usedItem:<item>` | The player uses the item. |

These are the builder's [`TimbnQuestEvent`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnQuestEvent) factories, and `walkedInto` is `TimbnQuestEvent.EnteredWorldZone` in C#.

## Game Expressions

The game's own quests, items and conversations run formula strings, such as `AddPerk("perk_x")`, `$money += 50` or `IsQuestCompleted("x")`. A pack reaches them through `runOnStart` and `runOnDone` (the game runs them at its own exact moment), the `run` line and action, and the `expr` condition. Core compiles each one when the pack loads, so a typo is reported like any other mistake instead of failing in the middle of a quest. See [Game Expressions](expressions.md) for the functions.

## Talks

A talk is a conversation that is not a quest, the same as `Dialog.AddTalk`. It is an object in the `talks` list. `id`, `npc` and `lines` are required.

| Field | What it does |
| --- | --- |
| `id` | The talk's id, before the pack id is added. |
| `npc` | The NPC who offers it. |
| `lines` | The [lines](#lines) of the talk. |
| `when` | A [condition](#conditions) for when the NPC offers it. Always when left out. |
| `once` | True to stop offering it once the player starts it in that save. False when left out. |
| `bubbleColor` | A color for this talk's speech bubble. The pack's color when left out. |

```json
{ "id": "tip", "npc": "npc_larry", "when": "hasTech:wood_basic", "once": true, "bubbleColor": "#ffcc66", "lines": [ "Herm sells nails." ] }
```

## Mistakes

A pack is checked as it loads, and one mistake never stops the rest.

- A file that is not valid JSON is skipped entirely.
- A quest or talk with a mistake is skipped and the rest of the pack loads. That covers a missing field, a wrong type, a number too big for the game, an unknown condition, action or event, an NPC id the game does not have, and any of the quest builder's own errors.
- A field that does not belong is reported and ignored, so a typo like `descriptoin` does not go unnoticed.
- An NPC id (`giver`, a talk's `npc`, a quest or stage `npc`, `sayAs`, and the `reputation` condition and action) is checked against [`TimbnGameIds.Npcs`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnGameIds.Npcs), so a typo like `npc_lary` is reported instead of the quest never showing up. See [Game Ids](game-ids.md).
- A quest id that another mod or pack already adds is reported and the second one skipped.
- A `pack.json` that is not valid skips a standalone pack. For a mod's pack it falls back to the mod's own id.
- An unhandled exception while loading a pack rolls back what that pack added and reports it, so one broken pack never stops the other packs.
- Core's save start checks also apply to pack quests, so an `after` id that matches no quest, and a loop of `after`, both get a warning.

Every message names the pack folder, the file, the quest or talk and the line, and is written to `BepInEx/LogOutput.log` as a warning.

```text
LarrysWine/quests/wine.json, quest 'ask': line 12, "phase" should be a whole number. It was skipped.
```

Each pack that loads also logs a line with how many quests and talks it added and how many problems it had. The Quest Packs setting `General / ShowProblemsOnMainMenu` is off by default. When on, it also lists the problems in a main menu popup, the first six and a count of the rest (see [Main Menu](main-menu.md)).

## Using C# Instead

Everything in a pack maps onto the C# quest builder, as the Quests table shows, so a pack can grow into a mod when it needs code. See [Quests, Dialog, and Text](quests.md) for the builder, including the pieces a pack does not have, such as repeatable quests and runs.
