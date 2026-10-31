# SuperItems 2.0: Quick User Manual

A Paper plugin (Java 25, Paper 26.x) for creating custom **items, armor, armor sets, mobs/minibosses/bosses, loot tables and spawners** in game. No coding required.

## 1. Install
1. Build with `./gradlew build` (or `./gradlew jar`). The jar lands in `build/libs/SuperItems-2.0.0.jar`.
2. Drop it in your server's `plugins/` folder, start the server, and you are done.
3. Permission: `superitems.admin` (ops by default). Aliases: `/superitems`, `/sitems`, `/si`.

## 2. The one rule that covers everything
Every command is: `/superitems [group] <action> <id> --flag value --flag value`

* A value runs until the next `--flag`, so it can contain spaces and whole commands.
* Flags can repeat (`--effect ... --effect ...`).
* On `edit`, use `none` to clear a flag: `--on_hit none`.
* Text accepts `&` color codes or MiniMessage (`<green>`, `<#ff8800>`).
* A command with a mistake changes nothing, and it tells you why.
* Press **Tab** for flag and value suggestions. Type a group name alone (`/superitems mob`) to list its commands.

| Group | Actions |
| --- | --- |
| *(items)* | `create edit delete give list info reload` |
| `armor` | `create edit delete give list info` |
| `set` | `create edit delete list info` |
| `mob` | `create edit delete list info spawn kill active` |
| `loot` | `create edit delete list info roll drop test` |
| `spawner` | `create edit delete list info` |

**Build in this order**, because commands check that referenced ids exist: items/armor, then sets and loot, then mobs, then spawners.

## 3. Items
```
/superitems create frog_stick --name &aFrog Stick --description &7big hopppps --material stick --glow
  --on_right_click effect give @p minecraft:jump_boost 3 3 true
  --on_left_click playsound minecraft:entity.frog.ambient master @p
  --cooldown 5 --consume right
/superitems give frog_stick [player] [amount]
```
* **Click flags:** `--on_right_click`, `--on_left_click`, `--cooldown s`, `--consume none|right|left|both`, `--cancel true|false`.
* **Look:** `--name --description --material --model --glow --color #rrggbb` (leather dye).
* **Combat:** `--damage 400-600` rolls a random hit in that range (scaled by attack charge), `--on_hit`, `--on_damaged`, `--trigger_cooldown s`.
* **Stats:** `--attribute "name amount [add|mult|mult_total] [slot]"`, `--unbreakable`. Setting any attribute replaces the base item's defaults, so list every stat you want.
* **Worn/held:** `--on_equip`, `--on_unequip`, `--while_worn <cmd> --interval s` (default 2 s).
* **Placeholders:** `%target%`, `%targetname%` (on_hit), `%attacker%` (on_damaged), `%player%`.

## 4. Armor and sets
```
/superitems armor create crown --piece helmet --tier netherite --name &6King's Crown --armor 4 --toughness 3 --health 6 --unbreakable
/superitems set create kings --pieces crown,plate --bonus 2 effect give @s minecraft:resistance 3 0 true --on_activate 2 title @s title "Set!"
```
* Armor: `--piece helmet|chestplate|leggings|boots`, `--tier leather|chainmail|iron|golden|diamond|netherite|turtle`, stats `--armor --toughness --health --speed --kb_resist`. For percent bonuses use `--attribute "speed 0.15 mult"`.
* Sets: `--bonus "<n> <command>"` repeats while at least *n* pieces are worn (effects should outlast `--interval`). `--on_activate` / `--on_deactivate` fire once when the count is reached or lost.

## 5. Mobs, minibosses and bosses
```
/superitems mob create king --tier boss --type wither_skeleton --name &4&lSkeleton King
  --health 800 --damage 14 --speed 1.2x --scale 1.5 --mainhand blade --helmet crown
  --timer 12 summon minecraft:skeleton ~ ~ ~   --phase 50 effect give @s minecraft:speed infinite 2 true
  --loot king_loot --xp 300-500 --vanilla_drops false --announce server
/superitems mob spawn king [amount] [world x y z]
```
* **Tiers:** `enemy`, `miniboss`, `boss`. Tier only sets defaults (bosses get a boss bar and never despawn); any default can be overridden.
* **Stats:** a trailing `x` multiplies the vanilla value (`1.5x`).
* **Gear:** item/armor ids or vanilla materials; `--equipment_drop percent`.
* **Abilities:** `--timer "seconds cmd"` (while a player is within `--active_range`, default 64), `--phase "percent cmd"` (once, when health falls to that percent), `--on_spawn --on_hit --on_damaged --on_death --on_kill`.
* **Boss bar:** `--bar_color --bar_style --bar_range --bar_title`.
* **Natural spawns:** `--replace zombie 10` swaps 10% of zombie spawns for this mob; `--worlds a,b` limits it.
* **Mob commands run as the mob**, so `@s` is the mob. Prefix `target:` to run as the involved player, `player:` for that player's permissions, `raw:` for console. `on_death` always runs from the console. Extras: `%mob% %mobid% %x% %y% %z% %world% %killer%`.
* `mob kill [id]` removes mobs without drops; `mob active` counts live ones.

## 6. Loot tables
```
/superitems loot create king_loot --rolls 1-2 --chance 100 --deliver drop
  --entry 50 vanilla:bone 5-12   --entry 10 item:blade   --entry 5 item:crown chance=50
  --entry 1 table:rare_gems      --entry 30 nothing      --always 1 vanilla:experience_bottle 3-6
```
* Entry = `weight kind:ref [amount] [chance=%]`. Kinds: `item:`, `vanilla:`, `table:` (nest tables), `command:`, `nothing`.
* `--always` entries roll every time. `--deliver inventory` sends mob loot straight to the killer.
* On `edit`, `--entry` **adds**; use `--entry none --entry ...` to replace.
* Tune with `loot test <table> [times]` (simulates, gives nothing), `loot roll <table> [player]`, `loot drop <table> [world x y z]`.

## 7. Spawners
```
/superitems spawner create crypt --mob king --here --interval 600 --max 1 --range 48
```
`--here` or `--location x y z [world]` (console must name the world), `--interval` seconds (default 30), `--amount` per wave (1-50), `--max` alive at once (1-500), `--radius` scatter, `--range` activation distance (0 = always), `--enabled true|false`.

## 8. Files and config
`plugins/SuperItems/`: `items.yml`, `sets.yml`, `mobs.yml`, `loot.yml`, `spawners.yml`, `config.yml`. Edit by hand, then `/superitems reload`.
`config.yml` keys: `default-material`, `cooldown-message`, `damage-indicators`, `damage-indicator-format`, `natural-replacement`.

## 9. Troubleshooting
* **"No mob called ..."**: create the mob before the spawner/loot that references it.
* **Nothing happens on edit**: a flag was invalid; read the chat error.
* **Spawner idle**: no player within `--range`, or `--max` already alive.
* **Mob edits not showing**: behavior edits apply to live mobs, but name/gear/stats only apply to newly spawned ones.
