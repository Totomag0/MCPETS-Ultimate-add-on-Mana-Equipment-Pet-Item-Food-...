# MCPetsUltimate

Expands [MCPets](https://mcpets.gitbook.io/mcpets) with physical & magic combat stats, a mana system, pet equipment, custom foods, tradeable pet items, a pet storage box and new MythicMobs mechanics, plus fixes for MCPets' damage handling.

**Version 1.0.0** · Paper 1.21.11 · Java 21 · Author: Totomag0 · Contributor: N0TDREAMS

## Features

- **Combat stats**: separate physical and magic damage and resistances for pets, also usable on MythicMobs mobs.
- **Mana**: a mana pool per pet, with MythicMobs mechanic and condition `petMana`.
- **Equipment**: weapon, armor and jewel slots, made in an in-game editor, with random stat ranges and MythicMobs drops.
- **Pet foods**: health, mana, XP and timed buffs for every stat.
- **Tradeable pets**: convert a pet into an item with all its data, and keep duplicates in `/petsbox`.
- **Poison and wither**: percent of max health per second, magic or raw.
- **MythicMobs**: `petAttack`, `petMana`, `petequipment` drop and `<pet.*>` placeholders.
- **7 languages**: English, French, German, Spanish, Russian, Chinese and Japanese.
- **MCPets fixes**: resistance division, other players' pet menus, blank lines in icons, health after `/mm reload`.

## Requirements

| Plugin | Status |
| --- | --- |
| MCPets 4.1.6 | Required |
| MythicMobs 5.x | Required |
| ModelEngine | Optional |
| MythicLib | Optional |
| LuckPerms | Optional (pet to item conversion) |

## Installation

1. Drop `MCPetsUltimate-1.0.0.jar` into `plugins/`.
2. Restart the server.
3. Pick your language with `language:` in `plugins/MCPetsUltimate/config.yml`.

## Documentation

Everything (commands, permissions, stats, formulas, placeholders, languages, config) is in the **[wiki](docs/WIKI.md)**.

## Building

MCPets, ModelEngine and MythicLib are not on a Maven repository: put `MCPets 4.1.6 (1.21.+).jar`, `ModelEngine-4.1.1.jar` and `MythicLib-1.7.jar` next to `build.gradle`, then run:

```bash
./gradlew jar
```

The jar is written to `build/libs/`.
