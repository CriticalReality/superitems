# SuperItems User Manual

In-game custom items that run commands on left and right click. Permission: `superitems.admin`. Aliases: `/si`, `/sitems`.

## 1. Quick start

```
/superitems create fs --name frog stick --description big hopppps --on_right_click effect give @p minecraft:jump_boost 3 3 true --on_left_click playsound minecraft:entity.frog.ambient master @a ~ ~ ~
/superitems give fs
```

Creates item `fs`, then gives you one. Right click = jump boost. Left click = frog sound.

## 2. Commands

| Command | Does |
| --- | --- |
| `/si create <id> [flags]` | New item |
| `/si edit <id> [flags]` | Change flags only |
| `/si give <id> [player] [amount]` | Give copies (console must name a player; max 2304) |
| `/si info <id>` | Show all settings |
| `/si list` | List all ids |
| `/si delete <id>` | Remove item (copies in inventories go inert) |
| `/si reload` | Re-read `config.yml` and `items.yml` |

Ids: lowercase `a-z 0-9 _ -`, up to 32 characters.

## 3. Flags

A flag's value is everything up to the next `--flag`, so spaces and full commands are fine.

**`--name`**: display name.

```
--name Frog Stick
--name &aFrog &lStick
--name <gradient:green:yellow>Frog Stick</gradient>
```

`&` color codes or MiniMessage tags. Italic is off by default.

**`--description`** (alias `--lore`): lore lines.

```
--description big hopppps
--description line one | line two
--description line one --description line two
```

Use `|` or repeat the flag for more lines.

**`--material`**: base item. Default `stick` (change in `config.yml`).

```
--material blaze_rod
```

**`--model`**: item model, so a non-placeable base item looks like another item or block.

```
--material paper --model minecraft:dead_brain_coral
```

**`--glow`**: enchant shimmer.

```
--glow
--glow false
```

**`--on_right_click`** / **`--on_left_click`** (aliases `--right`, `--left`): commands to run. Repeat for several.

```
--on_right_click effect give @p minecraft:speed 10 1 true
--on_right_click title @p actionbar "Zoom"
--on_left_click particle minecraft:heart ~ ~1 ~ 0.3 0.3 0.3 0 10
--on_left_click playsound minecraft:entity.frog.ambient master @a ~ ~ ~
```

**`--consume none|right|left|both`**: remove one item after that click.

```
--consume right
```

**`--cooldown <seconds>`**: per player, per item. Action bar shows time left.

```
--cooldown 5
--cooldown 0.5
```

**`--cancel true|false`**: default `true`. Cancels the click so the base item does not place, break, attack or open chests. Set `false` to let normal behavior also happen.

```
--cancel false
```

## 4. How a command line runs

| Line | Runs as | Use for |
| --- | --- | --- |
| `effect give @p ...` | Console, as the player, at the player's position | Most things. `@p`, `@s`, `~ ~ ~` all work |
| `player:spawn` | The player, with their permissions | Plugin commands like `/spawn`, `/home` |
| `raw:say hi %player%` | Console, exactly as written | Commands that must not be wrapped |

`%player%` becomes the clicking player's name in every form. A leading `/` is optional.

```
--on_right_click player:warp mine
--on_right_click raw:give %player% minecraft:diamond 1
--on_right_click tp @s ~ ~10 ~
```

## 5. Edit

Only the flags you list change. Use `none` to clear one.

```
/si edit fs --name Super Frog Stick
/si edit fs --on_left_click none
/si edit fs --cooldown 3 --glow
/si edit fs --on_right_click say hi --on_right_click say bye
```

The `--on_*` flags replace the whole list, so list every command you want to keep.

Click commands change for existing copies immediately. Name, description, material and model only change on items you give afterwards.

## 6. Recipes

**Dead brain coral you use up:**

```
/si create coral --name &5Mystic Coral --material paper --model minecraft:dead_brain_coral --consume right --on_right_click effect give @p minecraft:regeneration 10 1 true
```

Paper base so it cannot be placed. The model makes it look like the coral.

**Teleport wand with cooldown:**

```
/si create wand --name &bBlink Wand --material blaze_rod --glow --cooldown 4 --on_right_click tp @s ^ ^ ^8 --on_right_click playsound minecraft:entity.enderman.teleport master @a ~ ~ ~
```

**Heal apple, stacks of 16:**

```
/si create heal --name &cHealing Apple --material apple --consume right --on_right_click effect give @p minecraft:instant_health 1 1 true
/si give heal Steve 16
```

**Left click only, chest still opens on right click:**

```
/si create zapper --material stick --on_left_click particle minecraft:electric_spark ~ ~1 ~ 0.2 0.2 0.2 0 20 --cancel false
```

**Run a plugin command (loadout redeem):**

```
/si create kit --name &6Loadout Token --material paper --consume right --on_right_click player:loadout redeem starter
```

Replace the command with whatever your other plugin uses.

## 7. Inspect

```
/si info fs
/si list
```

## 8. Saved file

`plugins/SuperItems/items.yml`. Edit by hand, then `/si reload`.

```yaml
items:
  fs:                       # id
    name: frog stick        # display name
    material: STICK         # base item
    model: minecraft:dead_brain_coral   # optional item model
    glow: false             # enchant shimmer
    description:            # lore lines
    - big hopppps
    on_right_click:         # commands on right click
    - effect give @p minecraft:jump_boost 3 3 true
    on_left_click:          # commands on left click
    - playsound minecraft:entity.frog.ambient master @a ~ ~ ~
    consume: none           # none | right | left | both
    cooldown: 0.0           # seconds
    cancel: true            # cancel the vanilla click
```

## 9. `config.yml`

```yaml
default-material: STICK                 # used when --material is omitted
cooldown-message: "&cOn cooldown: {seconds}s"   # action bar text
```

## 10. Troubleshooting

| Problem | Fix |
| --- | --- |
| `Unknown flag` | Typo. Valid: name, description, material, model, glow, on\_right\_click, on\_left\_click, consume, cooldown, cancel |
| `Expected the item id first` | Missing `--name` before text: `create fs --name frog stick`, not `create fs frog stick` |
| Item does nothing | Wrong id (deleted?), or no command for that click. Check `/si info <id>` |
| Block placed or chest opened | Keep `--cancel true`, or use a non-placeable `--material` |
| Plugin command fails | Prefix with `player:` or `raw:` |
| Old name still showing | Give a fresh copy; edits to looks only affect new items |
