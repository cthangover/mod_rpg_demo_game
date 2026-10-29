# RPG Demo — a small tile-map world

RPG Demo is a short, complete tile-map adventure: a village by the Grey Forest, the elder's house and the forest with a wolf living at the cave. You walk with the keyboard, talk to people, open chests, gather things in the forest, sell them to the merchant and fight. It is made only of data files and scenarios — no code — so it doubles as a working example of how a tile-map game is put together.

The demo has no game card of its own: it is the world of the **RPG Plus** game, which includes this mod and builds on it. Start it from **Play → RPG Plus → New game**: after the short crossroads prologue of RPG Plus you arrive in this village. The screenshots below come from RPG Plus, so some people and goods you see there are added by that game.

![The village by the Grey Forest at night: houses, a cobbled square with lamps, villagers, a pond and the hero on the path.](assets/village.jpg)

## How to play

- **Walk** with WASD or the arrow keys, **run** with Shift.
- **Talk, open, read** with E or F while facing a person or an object.
- Walking onto a doorway or the edge of a path takes you to the next map.

The story: the elder asks you to drive off the wolf at the cave east of the village. Go past the pond, along the forest path, deal with the slime blocking the way and face the wolf. When it is beaten, the elder thanks you — in his own way.

## What is in the world

- **Three maps.** The village (32×18 tiles), the elder's house and the Grey Forest, all drawn with the village tileset of Village Pack.
- **People.** The elder gives and finishes the quest, villagers chat, a sign names the village; one door is locked.
- **Chests.** A chest in the elder's house and one in the forest; once opened they stay open.
- **Gathering in the forest.** Brushwood, stones and forest mushrooms lie along the paths; each spot can be picked once. A mushroom restores 5 health.
- **The merchant.** Sells the Elixir (restores 20 health) for 10 coins and buys brushwood for 1 coin and stones for 2.
- **Battles.** The slime on the path and the wolf at the cave are visible on the map and start a fight when you talk to them. In the forest, random encounters with one or two slimes happen about every 48 steps.

![The Grey Forest: the path to the cave with the wolf, a chest, stones, mushrooms and the slime on the path.](assets/forest.jpg)

![The merchant's shop: goods on the left, the selected Elixir with its price and the Buy button, your bag on the right. RPG Plus adds more goods to this shop.](assets/shop.jpg)

![A battle against a slime: the hero at the bottom left, the slime at the top, the battle log on the right.](assets/battle.jpg)

## For authors: how it is built

```filestree
rpg_demo_game/
├── maps/                     rpg_village.json, rpg_elder_house.json, rpg_forest.json
├── map_events/<map>/events.json   people, doors, chests, pickups, monsters
├── scenes/                   one scene per map
├── scenarios/rpg_demo/       intro, map entries and events/*.scenario
├── characters/               the hero, the slime and the wolf
├── items/rpg_demo/           brushwood, stones, mushroom, elixir and their icons
├── shops/village_merchant.json
└── locale/                   names and scenario translations in seven languages
```

A **map** is a grid of tile numbers in layers over a tileset, with its own name, size and zoom. The forest map also lists its random encounters:

```json
"Encounters": {
  "Steps": 48,
  "Groups": ["rpg_slime", "rpg_slime,rpg_slime"]
}
```

An **event** is something placed on a tile. A door with `"Trigger": "Touch"` teleports; a monster runs a scenario and disappears once its flag is set:

```json
{
  "Id": "wolf_boss",
  "X": 25,
  "Y": 8,
  "Sprite": "village/wolf",
  "Dir": "Left",
  "Scenario": "rpg_demo/events/wolf_boss",
  "Condition": "rpg_wolf_dead != 1"
}
```

The **scenario** says the lines and starts the fight; winning sets the flag:

```scenario
ptext "A wolf sits by the cave. It looks you over the way it looks at the elder's goats." key=rpg_demo/wolf/line1
text "— Grrr." key=rpg_demo/wolf/line2
action rpg.battle enemies=rpg_wolf,rpg_slime win_flag=rpg_wolf_dead
end
```

In the mod's scenario files these lines are written in Russian; the listing shows them in English. Every line has a `key=` with its translation in `locale/scenarios_<lang>.properties`, so the game shows them in your interface language:

```properties
rpg_demo/wolf/line1 = A wolf sits by the cave. It looks you over the way it looks at the elder's goats.
```

Other pieces work the same way: a chest runs `action item.add`, sets a flag and is replaced by an open-chest event with the opposite `Condition`; the merchant runs `action trade.open shop=rpg_demo/village_merchant`; the first visit to the village runs the intro scenario, which puts the hero in the party with `action party.add id=rpg_hero`.

To change the maps and events visually, use the map editor of the RPG editor tools.

## If something goes wrong

1. **A door does nothing.** A `Touch` door works when you step on it; an `Action` one needs E or F while facing it.
2. **A monster or chest came back.** Its `Condition` must test the same flag the scenario sets.
3. **The shop is empty.** The trading mod must be in the game, and the shop id in the scenario must match `shops/village_merchant.json`.
