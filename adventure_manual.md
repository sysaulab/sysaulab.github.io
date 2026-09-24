
# pydventure

A single-file, turn-based browser roguelike. Open `adventure.html` in any
modern browser — no build step, no server, no dependencies.

---

## The goal

Collect **seven crystals**, one from each of the seven dungeons, and place
them on the altar in the shrine.

```
overworld  ──►  level1 … level7   (one crystal each, guarded by a dragon)
                    │
                    ▼
                level8 (shrine)  ← requires crystal ×7
                    │
                    ▼
                 victory
```

Entering the shrine **consumes all seven crystals**. Inside waits a red
dragon. Claim the victory item on the far side to end the game.

If your HP reaches zero you collapse, wake at the entrance of the current
map with full health, and lose your retreat target. Progress is saved to
`localStorage` after every action.

---

## How to play

Every action is a turn. You either move, use something, or craft — and if
enemies are in the room, anything you do other than fight or heal has to get
past them first.

### Movement

| Input | Action |
|---|---|
| `N` `S` `E` `W` (keys or on-screen buttons) | move one room |
| `U` `D` | take stairs / teleport between maps |
| Click a door button | same as the key |

There is no "explore" command — the room description names every exit and
any locked door tells you what it wants.

### Commands

The UI is click-driven, but every action has a text form:

```
move n              move north
move u              go up / take stairs up
use fire            throw a fire at the first enemy
use shelter         rest and heal to full
use meal            eat a meal (+2 HP)
use heart           permanently raise max HP by 1
use sword           (sword is a level, not an item — see Crafting)
take wood           pick up an item from the floor
fight               attack the enemies in this room
retreat             go back to the room you came from
combine rock rock   craft (see Crafting)
```

`use <item>` on a locked door opens it if the item matches. If only one
door needs that item you can omit the direction; otherwise specify
(`use bomb n`).

### Combat

Combat is a single opposed roll. You never grind down an enemy's HP with
your sword — either you win the room or you don't.

```
your pool  = 1 + fight skill + (20 × sword level)
enemy pool = sum of enemy HP in the room

roll       = rand(0, your pool)
enemyRoll  = rand(0, enemy pool)

win  → room cleared, +1 fight skill, roll drops
lose → -1 HP, enemies remain
```

**Fire** bypasses the roll entirely: 10–20 damage to the front enemy.
A dragon (250 HP) takes about 15–25 fires to bring down.

### Avoidance and retreat

**Avoidance** is only used when you try to leave a room that still has
enemies in it.

```
your pool  = 1 + avoidance skill
enemy pool = sum of enemy HP in the room
```

Win → you slip through. Lose → -1 HP and you stay put.

**Retreat** is the exception. If it is the very first thing you do after
entering a room, it is free — you're still standing in the doorway, you
just step back out. Any other action (including a failed retreat) uses up
your free peek, and after that retreating costs an avoidance roll like any
other move.

### Inventory

You can carry **at most 10 of any one item**. The status bar shows each
item as `name:count/10`. Picking up more than fits leaves the excess on the
floor. You cannot carry a crafting result that would overflow a slot — use
something first.

To use an item from the status bar: **double-click** it on desktop, or
**long-press** (~½ second) on touch.

---

## The world

| Map | Kind | Notes |
|---|---|---|
| `overworld` | overworld, 2 floors | surface + one underground cave level, connected by two stair pairs |
| `level1` … `level7` | dungeon, 1 floor each | difficulty scales; each holds one crystal, guarded by a dragon |
| `level8` | shrine, 1 floor | needs `crystal ×7` to enter; holds the victory item |

Dungeons and the shrine are entered via stairs down from the overworld
(they appear as `D` doors on dead-end and normal overworld rooms). Each
dungeon's `U` exit returns you to the overworld spot you came from.

Enemies respawn. After **90 seconds** away from a room, cleared enemies
have a chance to come back:

| Map kind | Respawn chance |
|---|---|
| overworld | 15% |
| dungeon / shrine | 35% |

---

## Crafting

Open the ⚗ panel (bottom-left) or type `combine <a> <b>`. All recipes are
exactly two inputs.

| Result | Inputs | Effect |
|---|---|---|
| **fire** | wood ×2 | throwable — 10–20 dmg, one enemy |
| **tool** | rock ×2 | intermediate |
| **plank** | tool ×1 + wood ×1 | intermediate |
| **shelter** | plank ×2 | full heal |
| **coal** | fire ×1 + wood ×1 | intermediate |
| **grill** | coal ×2 | intermediate |
| **meal** | grill ×1 + food ×1 | +2 HP |
| **steel** | coal ×1 + ore ×1 | intermediate |
| **sword** | tool ×1 + steel ×1 | **+1 sword level** (permanent) |

### Cost in raw materials

Every intermediate is consumed, so a full chain costs:

| Item | Raw cost |
|---|---|
| fire | 2 wood |
| tool | 2 rock |
| plank | 2 rock + 1 wood |
| shelter | 4 rock + 2 wood |
| coal | 3 wood |
| grill | 6 wood |
| meal | 6 wood + 1 food |
| steel | 3 wood + 1 ore |
| **sword** | **2 rock + 3 wood + 1 ore** |

### Where things come from

- **wood, rock, food** — enemy drops (food only from overworld enemies) and random floor resources
- **bomb** — enemy drops (mid/high tier), plus a loose stash of 3 on the overworld's top floor
- **ore** — 1 per dungeon floor, mined from the floor
- **heart** — guaranteed drop from a dragon

Resource pools per terrain (weighted):

| Terrain | Drops |
|---|---|
| forest | wood ×4, rock ×1 |
| plains | wood ×1, rock ×1 |
| desert | rock ×2 |
| dead plain | rock ×3, wood ×1 |
| valley | rock ×3, wood ×1 |
| cave | rock ×2, wood ×1 |
| stone hall | coal ×2 |
| crypt | coal ×3 |
| armory | coal ×2 |
| library | coal ×4, rock ×1 |
| sanctum | rock ×3, coal ×3 |
| shrine hall | rock ×2, coal ×2 |

---

## Enemies

Enemy HP is also its contribution to the room's "enemy power" — the number
you have to beat to fight or slip past.

| Enemy | HP | Found in | Drops |
|---|---|---|---|
| octorok_blue | 1 | plains, cave | wood, food |
| bat_blue | 2 | forest, cave | wood, food |
| spider_blue | 3 | forest | wood, food |
| fan | 5 | desert | wood, food |
| rhinos | 6 | valley | wood, rock, food |
| knight_blue | 7 | dead plain | wood, rock, food |
| octorok_red | 11 | plains | wood, rock, food |
| bat_red | 12 | forest, cave, stone hall, library | wood, food |
| spider_red | 13 | forest, crypt | wood, rock, food |
| fan_red | 15 | desert | wood, rock, bomb, food |
| rhinos_red | 16 | valley | wood, rock, bomb, food |
| knight_red | 17 | dead plain, armory | wood, rock, bomb, food |
| ghosts | 19 | cemetery | wood, rock, bomb, food |
| stalfos | 20 | stone hall, crypt | wood, rock, bomb |
| darknut | 24 | armory, sanctum, shrine hall | wood, rock, bomb |
| wizzrobe | 27 | library, sanctum, shrine hall | wood, rock, bomb |
| **dragon** | **250** | each dungeon's crystal room | wood, rock, bomb, **heart** |
| **dragon_red** | **500** | the shrine | wood, rock, bomb, **heart** |

Drops are rolled individually per enemy, 0–1 of each listed item.

**Bomb walls** appear in every dungeon (two per floor). They are ordinary
doors that require and consume one **bomb** to pass.

---

## Tips

- **Craft a sword early.** Each sword level adds 20 to your fight roll —
  one sword is worth more than 20 levels of fight skill. The first
  dungeon's ore plus a few trees is enough.
- **Retreat first, fight later.** The free retreat on entry is your
  scouting tool. If a room looks too strong, back out before you commit.
- **Fire is the boss killer.** A dragon's 250 EP is brutal to roll against,
  but wood is everywhere and each fire chip is 10–20. Stockpile wood on the
  overworld before descending.
- **Food comes from the surface.** Dungeon and shrine enemies never drop
  it. If you plan to live off meals, farm the overworld.
- **Don't overfill.** The 10-per-item cap will silently refuse crafting
  output. Burn a fire or eat a meal before crafting the eleventh.
- **Hearts stack.** Every dragon gives one, and each one is a permanent
  max-HP increase. Killing all eight bosses roughly doubles your health
  bar.
- **The shrine eats your crystals.** You arrive with 7, you enter with 0.
  There is no going back to a dungeon to "re-farm" one afterward.

Ok, now you can [play the game](adventure.html).