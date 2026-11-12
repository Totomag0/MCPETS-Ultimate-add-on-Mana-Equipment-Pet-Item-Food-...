# MCPetsUltimate Wiki

MCPetsUltimate adds to MCPets 4.1.6 new pet stats, a real physical / magic combat system, mana, equipment, pet foods, tradeable pet items, a 7-language system and several MCPets fixes. Target server: Paper 1.21.11.

Version 1.0.0. Author: Totomag0. Contributor: N0TDREAMS.

## Contents

- [Installation](#installation)
- [New pet stats](#new-pet-stats)
- [Combat: damage and resistances](#combat-damage-and-resistances)
- [Poison and wither](#poison-and-wither)
- [Mana](#mana)
- [Equipment](#equipment)
- [Pet foods](#pet-foods)
- [Tradeable pets and /petsbox](#tradeable-pets-and-petsbox)
- [MythicMobs](#mythicmobs)
- [Lore and menu placeholders](#lore-and-menu-placeholders)
- [Languages](#languages)
- [Comfort and protection](#comfort-and-protection)
- [What the plugin changes in MCPets](#what-the-plugin-changes-in-mcpets)
- [config.yml reference](#configyml-reference)

## Installation

Drop `MCPetsUltimate-1.0.x.jar` into `plugins/` and restart. The plugin creates `config.yml`, `petfoods.yml`, `equipment.yml` and the `lang/` folder in `plugins/MCPetsUltimate/`.

| Plugin | Status | Role |
| --- | --- | --- |
| MCPets 4.1.6 | Required | The pets |
| MythicMobs 5.x | Required | Mobs, mechanics, drops |
| ModelEngine | Optional | Models, dismount on death |
| MythicLib | Optional | Elements, `mmodamage`, damage types |
| LuckPerms | Optional | Required to convert a pet into an item |

### Commands

| Command | Permission | Effect |
| --- | --- | --- |
| `/mcpetsultimate reload` | `mcpetsultimate.admin` | Reloads the config, language, pet stats and foods |
| `/mcpetsultimate equipment` | `mcpetsultimate.admin` | Opens the equipment editor |
| `/mcpetsultimate equipment give <id> [player]` | `mcpetsultimate.admin` | Gives an equipment (stats rolled at random) |
| `/mcpetsultimate food give <id> [player] [amount]` | `mcpetsultimate.admin` | Gives a pet food |
| `/mcpetsultimate pet give <pet> [player] [level] [amount]` | `mcpetsultimate.admin` | Gives a tradeable pet item at the chosen level, full health and mana |
| `/petsbox` | `mcpetsultimate.box` | Opens the box of duplicate pets |
| `/mcpets reload` or `/pets reload` | (MCPets) | Also reloads the plugin right after MCPets |

`/mcpetsultimate` also answers to `/mcpu`.

### Permissions

| Permission | Default | Effect |
| --- | --- | --- |
| `mcpetsultimate.admin` | OP | Admin commands |
| `mcpetsultimate.box` | Everyone | Use `/petsbox` |
| `mcpetsultimate.box.bypasscooldown` | OP | No delay between two box swaps |
| `mcpetsultimate.name.color` | Everyone | Colors in pet names |
| `mcpetsultimate.pets.others` | Nobody | Open the menu of other players' pets |

## New pet stats

These keys go into the MCPets pet files (`plugins/MCPets/Pets/*.yml`), at the root or in each `Levels.<level>` to change them per level, like `MaxHealth`. Equipment and foods add on top.

| Key | Example | Effect |
| --- | --- | --- |
| `MoveSpeedModifier` | 1.5 | Movement speed (x1.5) |
| `MaxMana` | 100 | Size of the mana pool |
| `ManaRegen` | 2 | Mana regenerated per second |
| `PhysicalResistance` | 5 | Removed from each physical hit taken, x (ResistanceModifier + PhysicalResistancePercent/100) |
| `PhysicalResistancePercent` | 50 | +0.5 on the physical resistance modifier |
| `MagicResistance` | 5 | Removed from each magic hit taken, x (1 + MagicResistancePercent/100) |
| `MagicResistancePercent` | 20 | x1.2 on the magic resistance |
| `PhysicalDamage` | 3 | x DamageModifier = damage of each physical hit dealt |
| `MagicDamage` | 3 | x Power = damage of each magic hit dealt |
| `AutoPickupItem` | true | The pet picks up the items around it |
| `AutoPickupRadius` | 5 | Pickup radius in blocks (5 by default) |

Modifiers work like in MCPets: 1 = +0%, 2 = +100%. Example: MagicDamage 50 with Power 2 = 100 magic damage per hit.

```yaml
Levels:
  level2:
    MaxHealth: 300
    ResistanceModifier: 2
    PhysicalResistance: 5      # 5 x 2 = 10 removed from each physical hit
    MagicResistance: 50
    MagicResistancePercent: 200
    PhysicalDamage: 30
    MagicDamage: 30
    MaxMana: 100
    ManaRegen: 2
```

## Combat: damage and resistances

Every hit is either physical or magic. The matching resistance is removed as a flat value, and a hit never goes below 0.

### Formulas

| Case | Calculation |
| --- | --- |
| Physical hit taken | damage - PhysicalResistance x (ResistanceModifier + PhysicalResistancePercent / 100) |
| Magic hit taken | damage - MagicResistance x (1 + MagicResistancePercent / 100) |
| Physical hit of the pet | PhysicalDamage x DamageModifier (replaces the base damage) |
| Magic hit of the pet | MagicDamage x Power (replaces the base damage) |

Tested in game: a 40 melee hit on a pet with PhysicalResistance 5 and ResistanceModifier 2 removes 10, the pet loses 30 health. A 40 magic spell on a pet with 25 total magic resistance takes 15 health.

### What counts as magic

- MythicMobs `damage{a=40;cause=MAGIC}`, and the causes POISON, WITHER, DRAGON_BREATH, SONIC_BOOM, THORNS.
- `mmodamage{a=40;type=MAGIC}` (MythicLib) and `petAttack{type=MAGIC}`.
- Vanilla sources: harming potions, evoker fangs, guardian beam, Warden sonic boom, Thorns, dragon breath.
- Everything else is physical: melee, projectiles, `damage{a=40}` without a cause.

### MythicMobs mob resistances

The 4 keys `PhysicalResistance`, `PhysicalResistancePercent`, `MagicResistance` and `MagicResistancePercent` at the root of a mob file give it the same resistances (modifier base 1). `/mm reload` applies the changes.

### Fully absorbed hits

When the resistance exceeds the hit, the pet loses nothing (0.000001 damage, invisible). With MythicLib, the plugin avoids an exact 0: MythicLib treats a 0 damage hit as fake and would keep its attack data stuck on the pet.

### Debug

`debug-damage: true` logs every hit taken by a pet in the console: raw damage, plugin calculation, final damage and health one tick later.

## Poison and wither

On a pet, poison and wither no longer deal their vanilla damage: they remove a percent of the max health every second while the effect lasts. Both can kill the pet. Players and other mobs keep the vanilla behavior.

| Effect | Lvl I | Lvl II | Lvl III | Type |
| --- | --- | --- | --- | --- |
| Poison | 2% | 3% | 4% | Magic: the magic resistance is removed every second |
| Wither | 2% | 3% | 4% | Raw: ignores every resistance |

Example: a pet with 300 max health under Wither I loses 6 health per second. Under Poison I it would lose 6 minus its magic resistance, so nothing with 6 resistance or more.

```yaml
pet-damage-over-time:
  enabled: true          # false = vanilla poison and wither
  poison:
    base-percent: 2      # level I
    percent-per-level: 1 # + per level above
  wither:
    base-percent: 2
    percent-per-level: 1
```

## Mana

Each pet has a mana pool (`MaxMana`) that refills by `ManaRegen` per second. The mana is saved with the player data and survives restarts. It also follows the pet when it becomes an item or goes to the box.

- Spend or give back mana in a skill: `petMana{amount=-50} @self`.
- Cast a skill only when there is enough mana: condition `petMana{amount=50}`.
- Give mana with a food: type `MANA` in `petfoods.yml`.
- Regeneration interval: `mana-regen-interval` (20 ticks = 1 second).

```yaml
Archangemana:
  Cooldown: 1
  Conditions:
  - petMana{amount=50}
  Skills:
  - petMana{amount=-50} @self
  - petAttack{type=MAGIC;element=FIRE} @target
```

## Equipment

Each pet wears three pieces of equipment: a weapon, an armor and a jewel. Their stats add to the pet's while it is summoned.

1. An operator makes the equipment with `/mcpetsultimate equipment`: a menu to choose the id, category, base item, name, lore and stats (text typed in the chat). Everything is saved in `equipment.yml`.
2. Each stat is a fixed value or a range (`1-5`). The range is rolled with 2 decimals when the item is made. Editing the template later does not change the items already given.
3. Players get the items with `/mcpetsultimate equipment give <id> [player]` or a MythicMobs drop: `- petequipment{id=fire_sword} 1 0.1`.
4. The owner equips the pet from the Equipment button (iron chestplate) of the MCPets pet menu. Only an equipment of the right category fits each slot, and it exists in one copy only (no duplication).

The equipment follows the pet when it becomes an item or goes to the box.

| Stat | Unit | Effect |
| --- | --- | --- |
| MaxHealth | flat | Extra max health |
| Regeneration | flat | Extra health per second |
| MaxMana / ManaRegen | flat | Extra max mana / mana per second |
| PhysicalResistance / MagicResistance | flat | Extra resistances |
| PhysicalDamage / MagicDamage | flat | Extra flat damage |
| PhysicalResistancePercent / MagicResistancePercent | % | Resistance modifier bonuses |
| Damage / Resistance / Power | % | Added to the MCPets modifiers (+10 = +10%) |
| MoveSpeed | % | Movement speed (+10 = +0.1) |

Category and stat names and the lore lines are in the language file (`equipment:`). The equipment menu layout (slots, materials) is in `equipment-menu:` of `config.yml`, its texts in the language file.

## Pet foods

Pet foods are defined in `plugins/MCPetsUltimate/petfoods.yml`, apart from the MCPets one. The owner right-clicks their summoned pet with the item, and one item is used (not in creative). Do not put these foods in `plugins/MCPets/petfoods.yml` too: MCPets reads an unknown type as HEALTH.

| Type | Effect |
| --- | --- |
| `HEALTH` | Gives Power health at once (not used when health is full) |
| `MANA` | Gives Power mana at once (not used when mana is full) |
| `EXP` | Gives Power XP (not used at the max level) |
| `BUFF_DAMAGE` / `BUFF_RESISTANCE` / `BUFF_POWER` | MCPets modifier, flat (0.2 = 1 -> 1.2) |
| `BUFF_MOVE_SPEED` | MoveSpeedModifier, flat (0.5 = 1 -> 1.5) |
| `BUFF_MAX_HEALTH` / `BUFF_REGENERATION` | Max health / regeneration per second |
| `BUFF_MAX_MANA` / `BUFF_MANA_REGEN` | Max mana / mana per second |
| `BUFF_PHYSICAL_DAMAGE_FLAT` / `BUFF_MAGIC_DAMAGE_FLAT` | PhysicalDamage / MagicDamage |
| `BUFF_PHYSICAL_RESISTANCE_FLAT` / `_PERCENT` | PhysicalResistance / PhysicalResistancePercent |
| `BUFF_MAGIC_RESISTANCE_FLAT` / `_PERCENT` | MagicResistance / MagicResistancePercent |

| Key | Role |
| --- | --- |
| `ItemId` | Material (`CARROT`) or MythicMobs item id |
| `CustomModelData` | Optional |
| `Name` / `Lore` | Optional, colors allowed. Let several foods share the same item |
| `Type` / `Power` | Effect and amount |
| `Operator` | `ADD` (+Power) or `MULTIPLY` (+Power% of the current value). HEALTH / MANA: % of the max. EXP: % of the next level |
| `Duration` | Buff duration in ticks. The same food given again replaces its buff; different foods stack |
| `Cooldown` | Seconds before this food can be given again to the same pet |
| `Pets` | Ids of the pets that can eat it (all when empty) |
| `Permission` | Permission needed to give it |

```yaml
SpeedBoost:
  ItemId: SUGAR
  Name: '&fSwift sugar'
  Lore:
  - '&7Speed &a+0.5 &7for &e30s&7.'
  Type: BUFF_MOVE_SPEED
  Power: 0.5
  Operator: ADD
  Duration: 600
  Cooldown: 30
```

Buffs are not saved across restarts and end when the pet is revoked or dies. The `food-*` messages are in the `messages:` section of the language file.

## Tradeable pets and /petsbox

### Convert a pet into an item

A barrier button (Convert to item, slot 8) is added to the MCPets pet menu. It turns the pet into an item with all its data: level, XP, health, mana, name and equipment. The item copies the MCPets pet icon; its name and lore are `pet-item:` in the language file. Admins can also make one with `/mcpetsultimate pet give`.

- Only the owner can convert their pet, and LuckPerms is required.
- The pet must be unlocked by a personal global permission of the player (not a group or a context), or it could be duplicated.
- Right click on the item: the player gets the pet back. When they already own it, the copy goes to their box.
- Left click on the item: the pet goes straight to the box.
- Each item can only be used once (anti-duplication).

### The box (/petsbox)

MCPets keeps one pet per id. The extra copies wait in the box.

- Left click: swap the copy with the current pet (delay `swap-cooldown`, 5 s by default, bypassed by `mcpetsultimate.box.bypasscooldown`).
- Right click: turn the copy into an item.
- Capacity: `max-pets` (45 max). Box full: the item is not used and the player is told.
- With `use-item-profile: true`, each entry shows the name and lore of the pet item, without the lines listed in `hide-item-lines` (language file).

## MythicMobs

### Mechanics and conditions

| Syntax | Type | Effect |
| --- | --- | --- |
| `petAttack{type=MAGIC;element=FIRE;multiplier=1} @target` | Mechanic | Pet hit: (MagicDamage x Power) or (PhysicalDamage x DamageModifier) x multiplier. The target's resistance applies |
| `petMana{amount=-50} @self` | Mechanic | Adds or spends mana, between 0 and MaxMana |
| `petMana{amount=50}` | Condition | True when the pet has at least this much mana |
| `- petequipment{id=fire_sword} 1 0.1` | Drop | Drops an equipment with randomly rolled stats |

`type` is `MAGIC` or `PHYSICAL`. `element` takes the MythicLib elements (FIRE, ICE, EARTH, WIND, THUNDER, WATER, DARKNESS, LIGHTNESS) and only works with MythicLib. An unknown element is reported in the console.

### Skill placeholders

For skills cast by the pet (caster = the pet).

| Placeholder | Value |
| --- | --- |
| `<pet.mana>` / `<pet.maxmana>` | Current / max mana |
| `<pet.manaregen>` | Mana per second |
| `<pet.movespeedmodifier>` | Speed modifier |
| `<pet.damage>` | PhysicalDamage x DamageModifier (real physical hit) |
| `<pet.power>` | MagicDamage x Power (real magic hit). Replaces the MCPets `<pet.power>` |
| `<pet.powermodifier>` | The MCPets Power modifier |

### Magic damage from a mob

- `damage{a=40;cause=MAGIC} @target` works with or without MythicLib.
- `mmodamage{a=40;type=MAGIC} @target` needs MythicLib.
- `damage{a=40} @target` without a cause is physical.

## Lore and menu placeholders

They work in `pet-item` and `box` of the language file. Those marked "menus" also work in the pet icons of the MCPets menus (`language.yml`). Stats include equipment and foods.

| Placeholder | Value |
| --- | --- |
| `%pet%` `%pet_id%` `%name%` | Pet name, id, name given with /pets name (colors included) |
| `%level%` `%levelname%` | Level |
| `%experience%` `%threshold%` `%progress%` `%progressbar%` | XP, XP of the next level (`max-level-text` at the max), progress |
| `%health%` `%maxhealth%` `%regeneration%` | Health, max health, regeneration |
| `%damagemodifier%` `%resistancemodifier%` `%power%` | MCPets modifiers |
| `%mana%` `%maxmana%` `%manaregen%` `%manaregenmodifier%` | Mana (menus) |
| `%movespeedmodifier%` | Speed (menus) |
| `%health_percent%` `%mana_percent%` `%exp_percent%` | Percents with one decimal, colored by `percent-colors` (menus) |
| `%physical_resistance%` `%magic_resistance%` | Flat resistances (menus) |
| `%physical_resistance_percent%` `%magic_resistance_percent%` | Resistance modifier bonuses (menus) |
| `%physical_damage%` `%magic_damage%` | Flat damage (menus) |
| `%physical_damage_total%` `%magic_damage_total%` | Flat x modifier: what a hit really deals (menus) |
| `%physical_resistance_total%` `%magic_resistance_total%` | Flat x modifier: what a hit really loses (menus) |
| `%weapon%` `%armor%` `%jewel%` | Names of the worn equipment (`none` when empty) |

Colors allowed everywhere: `&4`, `&#RRGGBB`, `§x§R§R§G§G§B§B` and MiniMessage (`<red>`, `<gradient:red:gold>`).

## Languages

Two files set the language: the plugin one (messages, items, box, plugin menus, editor, commands) and the MCPets one (`/pets` menus). Change both.

### Plugin language

`language:` in `plugins/MCPetsUltimate/config.yml` picks the file `plugins/MCPetsUltimate/lang/<language>.yml`. English is the default. `/mcpetsultimate reload` applies the change.

| Code | Language |
| --- | --- |
| `en_US` | English (default) |
| `fr_FR` | French |
| `de_DE` | German |
| `es_ES` | Spanish |
| `ru_RU` | Russian |
| `zh_CN` | Simplified Chinese |
| `ja_JP` | Japanese |

A key missing from the file falls back to the bundled one, then to English. Another language: copy `en_US.yml` to `lang/xx_XX.yml`, translate it and set `language: xx_XX`. A pet item keeps the language it was made in.

### MCPets menus (/pets)

MCPets has no language option: its menus come from **`plugins/MCPets/language.yml`**. To show the plugin stats there, replace the keys below in that file with the block of your language, then run `/mcpets reload`.

| `language.yml` key | Role |
| --- | --- |
| `pet_stats` | The whole stats sheet under the pet icon (replace the full value, on all its lines) |
| `nickname` | Nickname line |
| `nickname_item_lore` | "Click here to revoke" line |
| `pet_status_revoked` / `pet_status_dead` | Status of a revoked or dead pet |
| `pet_stats_max_level` | XP bar text at the max level |

The keys go at the root of the file (no space in front), in place of the old ones. The `pet_stats: |-` block spans several lines, each indented by 2 spaces. The `%mana%`, `%health_percent%`, `%physical_damage%`, `%magic_resistance%`, `*_total` and other plugin placeholders are filled by the plugin. Empty lines are removed by the plugin while `mcpets-menu-hide-blank-lines: true`.

#### English (en_US)

```yaml
nickname: '§9Nickname: §r%nickname%'
nickname_item_lore: §c§lClick here to revoke your pet
pet_status_revoked: §c§l✖ §r§7(%timeleft%s left)
pet_status_dead: §cDead §7(%timeleft%s left)
pet_stats_max_level: §7Max level reached.
pet_stats: |-
  §7➤ Status: %status%
  §6✦ Level §7- §6%levelname%
  §a§l⬆ §a%experience%/%threshold% xp
   %progressbar% [%exp_percent%%]
  §c❤ §fHealth: %health%§7/§f%maxhealth% [%health_percent%%§f]
  §9★ §fMana: %mana%§7/§f%maxmana% [%mana_percent%%§f]
  §c❤ §fHealth regeneration: %regeneration%/s
  §9★ §fMana regeneration: %manaregen%/s
  §6🗡 §fDamage modifier: §f%damagemodifier%%
  §6🗡 §fDamage: §f%physical_damage% §7[∑%physical_damage_total%]
  §9🔮 §fPower modifier: §f%power%%
  §9🔮 §fPower: §f%magic_damage% §7[∑%magic_damage_total%]
  §7🛡 §fResistance modifier: §f%resistancemodifier%%
  §7🛡 §fResistance: §f%physical_resistance% §7[∑%physical_resistance_total%]
  §d⛨ §fMagic resistance modifier: §f%magic_resistance_percent%%
  §d⛨ §fMagic resistance: §f%magic_resistance% §7[∑%magic_resistance_total%]
  §x§A§0§5§2§2§D§l🥾 §fMovement speed: %movespeedmodifier%
```

#### French (fr_FR)

```yaml
nickname: '§9Surnom : §r%nickname%'
nickname_item_lore: §c§lCliquez ici pour révoquer votre familier
pet_status_revoked: §c§l✖ §r§7(%timeleft%s restantes)
pet_status_dead: §cMort §7(%timeleft%s restantes)
pet_stats_max_level: §7Niveau maximum atteint.
pet_stats: |-
  §7➤ Statut : %status%
  §6✦ Niveau §7- §6%levelname%
  §a§l⬆ §a%experience%/%threshold% xp
   %progressbar% [%exp_percent%%]
  §c❤ §fPoints de Vie : %health%§7/§f%maxhealth% [%health_percent%%§f]
  §9★ §fPoints de Mana : %mana%§7/§f%maxmana% [%mana_percent%%§f]
  §c❤ §fRégénération de Vie : %regeneration%/s
  §9★ §fRégénération de Mana : %manaregen%/s
  §6🗡 §fModificateur de Dégâts : §f%damagemodifier%%
  §6🗡 §fDégâts : §f%physical_damage% §7[∑%physical_damage_total%]
  §9🔮 §fModificateur de Puissance : §f%power%%
  §9🔮 §fPuissance : §f%magic_damage% §7[∑%magic_damage_total%]
  §7🛡 §fModificateur de résistance : §f%resistancemodifier%%
  §7🛡 §fRésistance : §f%physical_resistance% §7[∑%physical_resistance_total%]
  §d⛨ §fModificateur de résistance Magique : §f%magic_resistance_percent%%
  §d⛨ §fRésistance Magique : §f%magic_resistance% §7[∑%magic_resistance_total%]
  §x§A§0§5§2§2§D§l🥾 §fVitesse de Marche : %movespeedmodifier%
```

#### Deutsch (de_DE)

```yaml
nickname: '§9Spitzname: §r%nickname%'
nickname_item_lore: §c§lHier klicken, um dein Haustier zurückzurufen
pet_status_revoked: §c§l✖ §r§7(noch %timeleft%s)
pet_status_dead: §cTot §7(noch %timeleft%s)
pet_stats_max_level: §7Höchste Stufe erreicht.
pet_stats: |-
  §7➤ Status: %status%
  §6✦ Stufe §7- §6%levelname%
  §a§l⬆ §a%experience%/%threshold% EP
   %progressbar% [%exp_percent%%]
  §c❤ §fLeben: %health%§7/§f%maxhealth% [%health_percent%%§f]
  §9★ §fMana: %mana%§7/§f%maxmana% [%mana_percent%%§f]
  §c❤ §fLebensregeneration: %regeneration%/s
  §9★ §fManaregeneration: %manaregen%/s
  §6🗡 §fSchadensmodifikator: §f%damagemodifier%%
  §6🗡 §fSchaden: §f%physical_damage% §7[∑%physical_damage_total%]
  §9🔮 §fKraftmodifikator: §f%power%%
  §9🔮 §fKraft: §f%magic_damage% §7[∑%magic_damage_total%]
  §7🛡 §fWiderstandsmodifikator: §f%resistancemodifier%%
  §7🛡 §fWiderstand: §f%physical_resistance% §7[∑%physical_resistance_total%]
  §d⛨ §fMagieresistenz-Modifikator: §f%magic_resistance_percent%%
  §d⛨ §fMagieresistenz: §f%magic_resistance% §7[∑%magic_resistance_total%]
  §x§A§0§5§2§2§D§l🥾 §fLaufgeschwindigkeit: %movespeedmodifier%
```

#### Español (es_ES)

```yaml
nickname: '§9Apodo: §r%nickname%'
nickname_item_lore: §c§lHaz clic aquí para retirar tu mascota
pet_status_revoked: §c§l✖ §r§7(quedan %timeleft%s)
pet_status_dead: §cMuerta §7(quedan %timeleft%s)
pet_stats_max_level: §7Nivel máximo alcanzado.
pet_stats: |-
  §7➤ Estado: %status%
  §6✦ Nivel §7- §6%levelname%
  §a§l⬆ §a%experience%/%threshold% xp
   %progressbar% [%exp_percent%%]
  §c❤ §fVida: %health%§7/§f%maxhealth% [%health_percent%%§f]
  §9★ §fManá: %mana%§7/§f%maxmana% [%mana_percent%%§f]
  §c❤ §fRegeneración de vida: %regeneration%/s
  §9★ §fRegeneración de maná: %manaregen%/s
  §6🗡 §fModificador de daño: §f%damagemodifier%%
  §6🗡 §fDaño: §f%physical_damage% §7[∑%physical_damage_total%]
  §9🔮 §fModificador de poder: §f%power%%
  §9🔮 §fPoder: §f%magic_damage% §7[∑%magic_damage_total%]
  §7🛡 §fModificador de resistencia: §f%resistancemodifier%%
  §7🛡 §fResistencia: §f%physical_resistance% §7[∑%physical_resistance_total%]
  §d⛨ §fModificador de resistencia mágica: §f%magic_resistance_percent%%
  §d⛨ §fResistencia mágica: §f%magic_resistance% §7[∑%magic_resistance_total%]
  §x§A§0§5§2§2§D§l🥾 §fVelocidad de movimiento: %movespeedmodifier%
```

#### Русский (ru_RU)

```yaml
nickname: '§9Прозвище: §r%nickname%'
nickname_item_lore: §c§lНажмите, чтобы отозвать питомца
pet_status_revoked: §c§l✖ §r§7(осталось %timeleft%с)
pet_status_dead: §cМёртв §7(осталось %timeleft%с)
pet_stats_max_level: §7Достигнут максимальный уровень.
pet_stats: |-
  §7➤ Статус: %status%
  §6✦ Уровень §7- §6%levelname%
  §a§l⬆ §a%experience%/%threshold% опыта
   %progressbar% [%exp_percent%%]
  §c❤ §fЗдоровье: %health%§7/§f%maxhealth% [%health_percent%%§f]
  §9★ §fМана: %mana%§7/§f%maxmana% [%mana_percent%%§f]
  §c❤ §fРегенерация здоровья: %regeneration%/с
  §9★ §fРегенерация маны: %manaregen%/с
  §6🗡 §fМодификатор урона: §f%damagemodifier%%
  §6🗡 §fУрон: §f%physical_damage% §7[∑%physical_damage_total%]
  §9🔮 §fМодификатор силы: §f%power%%
  §9🔮 §fСила: §f%magic_damage% §7[∑%magic_damage_total%]
  §7🛡 §fМодификатор защиты: §f%resistancemodifier%%
  §7🛡 §fЗащита: §f%physical_resistance% §7[∑%physical_resistance_total%]
  §d⛨ §fМодификатор маг. защиты: §f%magic_resistance_percent%%
  §d⛨ §fМагическая защита: §f%magic_resistance% §7[∑%magic_resistance_total%]
  §x§A§0§5§2§2§D§l🥾 §fСкорость передвижения: %movespeedmodifier%
```

#### 简体中文 (zh_CN)

```yaml
nickname: '§9昵称: §r%nickname%'
nickname_item_lore: §c§l点击此处召回你的宠物
pet_status_revoked: §c§l✖ §r§7(剩余 %timeleft%秒)
pet_status_dead: §c已死亡 §7(剩余 %timeleft%秒)
pet_stats_max_level: §7已达到最高等级。
pet_stats: |-
  §7➤ 状态: %status%
  §6✦ 等级 §7- §6%levelname%
  §a§l⬆ §a%experience%/%threshold% 经验
   %progressbar% [%exp_percent%%]
  §c❤ §f生命值: %health%§7/§f%maxhealth% [%health_percent%%§f]
  §9★ §f法力值: %mana%§7/§f%maxmana% [%mana_percent%%§f]
  §c❤ §f生命恢复: %regeneration%/秒
  §9★ §f法力恢复: %manaregen%/秒
  §6🗡 §f伤害加成: §f%damagemodifier%%
  §6🗡 §f伤害: §f%physical_damage% §7[∑%physical_damage_total%]
  §9🔮 §f力量加成: §f%power%%
  §9🔮 §f力量: §f%magic_damage% §7[∑%magic_damage_total%]
  §7🛡 §f抗性加成: §f%resistancemodifier%%
  §7🛡 §f抗性: §f%physical_resistance% §7[∑%physical_resistance_total%]
  §d⛨ §f魔法抗性加成: §f%magic_resistance_percent%%
  §d⛨ §f魔法抗性: §f%magic_resistance% §7[∑%magic_resistance_total%]
  §x§A§0§5§2§2§D§l🥾 §f移动速度: %movespeedmodifier%
```

#### 日本語 (ja_JP)

```yaml
nickname: '§9ニックネーム: §r%nickname%'
nickname_item_lore: §c§lクリックしてペットを戻す
pet_status_revoked: §c§l✖ §r§7(残り %timeleft%秒)
pet_status_dead: §c死亡 §7(残り %timeleft%秒)
pet_stats_max_level: §7最大レベルに到達しました。
pet_stats: |-
  §7➤ 状態: %status%
  §6✦ レベル §7- §6%levelname%
  §a§l⬆ §a%experience%/%threshold% XP
   %progressbar% [%exp_percent%%]
  §c❤ §f体力: %health%§7/§f%maxhealth% [%health_percent%%§f]
  §9★ §fマナ: %mana%§7/§f%maxmana% [%mana_percent%%§f]
  §c❤ §f体力回復: %regeneration%/秒
  §9★ §fマナ回復: %manaregen%/秒
  §6🗡 §fダメージ補正: §f%damagemodifier%%
  §6🗡 §fダメージ: §f%physical_damage% §7[∑%physical_damage_total%]
  §9🔮 §fパワー補正: §f%power%%
  §9🔮 §fパワー: §f%magic_damage% §7[∑%magic_damage_total%]
  §7🛡 §f耐性補正: §f%resistancemodifier%%
  §7🛡 §f耐性: §f%physical_resistance% §7[∑%physical_resistance_total%]
  §d⛨ §f魔法耐性補正: §f%magic_resistance_percent%%
  §d⛨ §f魔法耐性: §f%magic_resistance% §7[∑%magic_resistance_total%]
  §x§A§0§5§2§2§D§l🥾 §f移動速度: %movespeedmodifier%
```

The German, Spanish, Russian, Chinese and Japanese translations should be checked by a native speaker.

## Comfort and protection

| Feature | How it works | Setting |
| --- | --- | --- |
| Colored names | `/pets name` and the rename button keep `&4`, hex and MiniMessage | Permission `mcpetsultimate.name.color` (everyone) |
| Level above the head | Adds `Lv:<level>` after the name, updated within a second after a level up | `name-tag-suffix` in the language file (`''` = off) |
| Auto pickup | The pet picks up the items around it: its inventory first, then the player, the rest stays on the ground. Respects items thrown by the owner and items of another player | `AutoPickupItem` / `AutoPickupRadius` in the pet file, `auto-pickup-interval` |
| Dismount on death | Riders get off on the killing hit, without riding the death animation | `dismount-on-death`, `debug-dismount` |
| Other players' pets | A right or left click no longer opens their menu (even for an OP) and shows "This pet belongs to &lt;player&gt;." | Message `pet-not-yours`, permission `mcpetsultimate.pets.others` |
| Clean MCPets icons | Removes the empty lines MCPets hardcodes in pet icons (around the nickname, before the stats) | `mcpets-menu-hide-blank-lines` |
| Health after `/mm reload` | The pet health and max health are restored when MythicMobs resets them to the mob file values | Automatic |
| Reload | `/mcpets reload` also reloads the plugin; stats are re-read at each level up | `reload-stats-on-level-up` |

## What the plugin changes in MCPets

| MCPets behavior | With MCPetsUltimate |
| --- | --- |
| Every hit is divided by the ResistanceModifier, twice for melee, never for skills | Division undone: the ResistanceModifier only multiplies PhysicalResistance (`mcpets-resistance-divide: true` to keep it) |
| No physical / magic split | Separate physical and magic resistances and damage |
| Vanilla poison and wither | % of max health per second (`pet-damage-over-time`) |
| An OP opens the menu of any pet (right or left click) | Blocked, except with the `mcpetsultimate.pets.others` permission |
| Name colors only with `mcpets.color` (OP), only `#RRGGBB` | Every format, with `mcpetsultimate.name.color` |
| `<pet.power>` = Power modifier | `<pet.power>` = real magic damage, `<pet.powermodifier>` = the old one |
| One copy per pet | Extra copies in `/petsbox`, pets tradeable as items |
| The pet menu opens after the last food item | Blocked while feeding |
| Hardcoded empty lines in pet icons | Removed (`mcpets-menu-hide-blank-lines`) |
| No plugin reload with MCPets | `/mcpets reload` also reloads the plugin |

Buttons added to the MCPets pet menu: Equipment (slot 1) and Convert to item (slot 8).

## config.yml reference

The config only holds settings. Every text is in `lang/<language>.yml`.

| Key | Default | Role |
| --- | --- | --- |
| `language` | en_US | Language file to use |
| `mana-regen-interval` | 20 | Ticks between two mana regenerations |
| `auto-pickup-interval` | 10 | Ticks between two auto pickups |
| `reload-stats-on-level-up` | true | Re-reads the pet file stats at each level |
| `convert-button.slot` | 8 | Slot of the Convert to item button in the MCPets menu |
| `mcpets-menu-hide-blank-lines` | true | Removes the empty lines of the MCPets pet icons |
| `percent-colors` | 15/30/60/100 | Colors of the percents |
| `dismount-on-death` | true | Riders get off when the pet dies |
| `debug-dismount` / `debug-damage` | false | Debug logs in the console |
| `mcpets-resistance-divide` | false | Keep the MCPets division by the ResistanceModifier |
| `pet-damage-over-time` | on, 2% + 1%/lvl | Poison and wither as % of max health |
| `equipment-menu` | - | Equipment menu layout: button, slots, materials |
| `box` | 45 pets, 5 s | Capacity, swap delay, display mode |

Language file sections: `convert-button`, `pet-item`, `max-level-text`, `name-tag-suffix`, `equipment`, `equipment-menu`, `box`, `messages`, `commands`, `editor`. Other files: `petfoods.yml` (foods) and `equipment.yml` (equipment, written by the editor). After a change: `/mcpetsultimate reload`.
