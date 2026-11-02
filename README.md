# Game Data API — generic RPG/MMO data model

A game-agnostic, read-only API spec for publishing a game's static data (classes, items,
skills, creatures, NPCs, quests, zones, dungeons, crafting, progression).

- [openapi.yaml](openapi.yaml): the OpenAPI 3.0 spec (validated with Redocly)
- [reference.html](reference.html): rendered docs, open in a browser
- [redocly.yaml](redocly.yaml): lint config (`npx @redocly/cli lint openapi.yaml`)

## Design principles

1. **One access pattern for every collection.**
   `GET /{collection}` → all IDs · `GET /{collection}/{id}` → one record · `GET /{collection}/batch?ids=1,2,3` → several.
2. **Games define their own vocabulary.** Stats, elements, rarities, item categories,
   equipment slots and currencies are string keys served by `/lookups/*`, not enums baked
   into the spec. A sci-fi shooter and a fantasy MMO can use the same schema.
3. **A few shared building blocks do most of the work.**

   | Block | Used for |
   |---|---|
   | `Modifier` | Any stat change: gear bonuses, set bonuses, buffs, skill effects, enhancement levels, dungeon affixes |
   | `Requirement` | Any prerequisite: level, class, quest done, item owned, profession level, ... |
   | `Reward` | Quest, achievement and dungeon payouts (XP, currencies, items, choice items, unlocks) |
   | `ItemQuantity` / `LootEntry` | Costs, ingredients, box contents, drops (with chance and exclusive groups) |
   | `EntityRef` | A typed pointer to any record (`{type: "creature", id: 12}`) for cross-links |
   | `Location` / `Area` / `Spawn` | Where things are on a zone's map |
   | `LocalizedText` | `{ "en": "...", "fr": "..." }`; `?lang=en` trims to one language |

4. **Optional sub-objects instead of type-specific item schemas.** An `Item` has
   `equipment` when wearable, `use` when usable, `contents` when it's a container, so one
   schema covers weapons, armor, potions and loot boxes.
5. **Cache-friendly.** Data is static between patches; clients poll `GET /version` and
   refetch only when `dataVersion` changes.

## Collections

| Endpoint | Record | Notes |
|---|---|---|
| `/classes` | CharacterClass | Advancement tree via `parentId` + `tier`; base and per-level stats |
| `/items` | Item | Equipment, consumables, containers, materials |
| `/item-sets` | ItemSet | Bonuses unlocked at N pieces |
| `/skills` | Skill | Per-level table (`SkillLevel`): costs, cooldowns, damage scaling, modifiers |
| `/creatures` | Creature | Stats, resistances, attacks, scripted behaviors, loot, spawns |
| `/npcs` | Npc | Services, locations, shops (currency or barter prices) |
| `/quests` | Quest | Requirements, typed objectives, rewards, dialogue |
| `/zones` | Zone | Regions, points of interest, flags (pvp/safe/flying), map tiles |
| `/dungeons` | Dungeon | Entry requirements and cost, bosses, optional mutators |
| `/achievements` | Achievement | Tiered goals with rewards |
| `/professions` | Profession | Gathering nodes or crafting levels |
| `/recipes` | Recipe | Ingredients → outputs (with chance for random results) |
| `/progression/levels` | LevelEntry[] | XP curve and per-level grants |
| `/progression/enhancement` | EnhancementLevel[] | Upgrade chance, cost, failure result, bonuses per slot |
| `/lookups/*` | LookupEntry subtypes | stats, elements, rarities, item-categories, equipment-slots, currencies |
| `/assets/{kind}/{fileName}` | image | Records store only a file name in `icon` / `image` |

## Adapting it

- Rename the server URL in `openapi.yaml` and fill in `/lookups/*` with your game's keys first;
  everything else references them.
- Free-form string fields (`rank`, `type`, `services`, `tags`, ...) have examples but no enum,
  so you can use your own values.
- Generate a typed client or server stub:

```bash
npx @openapitools/openapi-generator-cli generate -i openapi.yaml -g typescript-fetch -o client
```
