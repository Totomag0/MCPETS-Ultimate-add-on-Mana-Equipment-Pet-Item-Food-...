# MCPetsUltimate

Expands [MCPets](https://mcpets.gitbook.io/mcpets) with physical & magic combat stats, a mana system, pet equipment, custom foods, tradeable pet items, a pet storage box, a skills menu with a keybind, a health / mana bar and new MythicMobs mechanics, plus fixes for MCPets.

**Version 1.0.0** · MCPets 4.1.6 and 4.2.1 · Paper 1.21.11 and 26.x · Java 21 · Author: Totomag0 · Contributor: N0TDREAMS

[Wiki](https://github.com/Totomag0/MCPETS-Ultimate-add-on-Mana-Equipment-Pet-Item-Food-...) · [Discord](https://discord.com/invite/QWB6Nqgwye)

## Features

- **Combat stats**: separate physical and magic damage and resistances for pets, also usable on MythicMobs mobs.
- **Mana**: a mana pool per pet, with the MythicMobs mechanic and condition `petMana`.
- **Equipment**: weapon, armor and jewel slots, made in an in-game editor, with random stat ranges, MythicMobs drops and resource pack looks (item model, CustomModelData, tooltip). Players cannot wear or use pet equipment themselves.
- **Pet foods**: health, mana, XP and timed buffs for every stat, made in an in-game editor (`/mcpetsultimate food`).
- **Tradeable pets**: convert a pet into an item with all its data, and keep duplicates in `/petsbox`. The pet skin goes back to default, so a skin unlocked by permission is never given away.
- **Skills menu**: `/petskills` lists the skills of the summoned pet; a button sends its signal. Open it with a key: Advancements (L), Swap hands (F) or Shift + F.
- **Health and mana bar**: a line under the pet name with bars and numbers, styles and symbols set in the config. Follows the name of ModelEngine models.
- **Poison and wither**: percent of max health per second, magic or raw.
- **MythicMobs**: `petAttack`, `petMana`, `petequipment` drop and `<pet.*>` placeholders.
- **Comfort**: colored pet names, `Lv:` after the name (text set in the language file), owner names in pet death messages, no vanilla action on a right click (a wolf no longer sits), auto pickup.
- **7 languages**: English, French, German, Spanish, Russian, Chinese and Japanese.
- **MCPets fixes**: resistance division, other players' pet menus, blank lines in icons, health after `/mm reload`, death loop of MCPets 4.2.1 on Paper 26, respawn timers after a restart, pets named "None", duplicate death messages.

## Requirements

| Plugin | Status |
| --- | --- |
| MCPets 4.1.6 or 4.2.1 | Required |
| MythicMobs 5.x | Required |
| ModelEngine | Optional |
| MythicLib | Optional (elements, damage types) |
| LuckPerms | Optional (pet to item conversion) |
| packetevents | Optional (Advancements key as the skills keybind) |

## Installation

1. Drop `MCPetsUltimate-1.0.0.jar` into `plugins/`.
2. Restart the server.
3. Pick your language with `language:` in `plugins/MCPetsUltimate/config.yml`, then `/mcpetsultimate reload`.

## Commands

| Command | Effect |
| --- | --- |
| `/mcpetsultimate reload` | Reloads the config, language, pet stats and foods |
| `/mcpetsultimate equipment [give <id> [player]]` | Equipment editor, or gives an equipment |
| `/mcpetsultimate food [give <id> [player] [amount]]` | Food editor, or gives a food |
| `/mcpetsultimate pet give <pet> [player] [level] [amount]` | Gives a tradeable pet item |
| `/mcpetsultimate pet signal <signal> [player]` | Sends a signal to the summoned pet |
| `/petskills [player]` | Skills menu of the summoned pet |
| `/petsbox` | Box of duplicate pets |

`/mcpetsultimate` also answers to `/mcpu`. Permissions, stats, formulas, placeholders and every config key: see the wiki.

## Documentation

Everything is in the **[wiki](https://github.com/Totomag0/MCPETS-Ultimate-add-on-Mana-Equipment-Pet-Item-Food-...)**, also in this repository: [docs/WIKI.md](docs/WIKI.md). Questions and help: [Discord](https://discord.com/invite/QWB6Nqgwye).

## Building

MCPets, ModelEngine, MythicLib and packetevents are not on a Maven repository: put `MCPets 4.1.6 (1.21.+).jar`, `ModelEngine-4.1.1.jar`, `MythicLib-1.7.jar` and `packetevents-spigot-2.13.0.jar` next to `build.gradle`, then run:

```bash
./gradlew jar
```

The jar is written to `build/libs/MCPetsUltimate-1.0.0.jar`. The version is set in `build.gradle`. Compiled against MCPets 4.1.6; MCPets 4.2.1 is supported at runtime.
