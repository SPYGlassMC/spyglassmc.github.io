---
layout: post
title: Parsing Entities and Score Holders
date: 2021-04-29 18:00:00 -0600
categories: develop
---

# Intro

[`minecraft:entity`](https://minecraft.fandom.com/wiki/Argument_types#minecraft:entity) and [`minecraft:score_holder`](https://minecraft.fandom.com/wiki/Argument_types#minecraft:score_holder)
are the two most complicated parsers used in *Minecraft: Java Edition* command system, at least in my opinion. This article
tries to document some notable things regarding parsing such arguments.

# UUID

Both `minecraft:entity` and `minecraft:score_holder` seem to accept UUIDs. However, the ways they parse UUIDs are not the
same - they're actually quite different. For `entity`, UUIDs can be contained in both unquoted strings and quoted strings,
and will be parsed by Java's built-in `UUID.fromString` method. For `score_holder`, however, UUIDs are just treated like
fake names, meaning that `01-1-1-1-1` and `1-1-1-1-1` are seen as two different UUIDs in your `/scoreboard` commands.

## `java.util.UUID.fromString`

Unlike what we've been told about UUID's format (`8-4-4-4-12`), the requirement of Java's `UUID.fromString` method is
quite loose\[1\]. In fact, any strings containing five segments separated by `-` where each segment is a hexadecimal
number representable in `Int64` are acceptable. SPYGlass uses something similar to the following TypeScript code to
achieve the same effect:

```typescript
const LongMax: bigint = 9223372036854775807n
declare const input: string

assert(input.match(/^[0-9a-f]+-[0-9a-f]+-[0-9a-f]+-[0-9a-f]+-[0-9a-f]+$/i))
const parts: bigint[] = input.split('-').map(p => BigInt(`0x${p}`))
assert(parts.every(p => p <= LongMax))
const mostSignificantBits: bigint = BigInt.asIntN(64, (parts[0] << 32n) | (parts[1] << 16n) | parts[2])
const leastSignificantBits: bigint = BigInt.asIntN(64, (parts[3] << 48n) | parts[4])
```

# Player Name

Both `entity` and `score_holder`, again, achieve parsing player names differently. For `entity`, the player name can be
either unquoted or quoted, and the length of the value must be between `1` and `16`. For `score_holder`, the parser
reads everything literally until it hits a space (` `) or the end of the command; although the parser itself doesn't have limitations at
the length of the value, it's still limited between `1` and `40` thanks to Brigadier and a check in
`net.minecraft.world.scores.Scoreboard#getOrCreatePlayerScore` (Mojang mappings, 21w15a).

# Entity Selector

Here comes the fun part. An entity selector must begin with `@`, followed by one of the selector variables (`p`, `a`, `r`, `s`, or `e`).

| Variable | Result        (Pseudocode)                                                                          |
| -------- | --------------------------------------------------------------------------------------------------- |
| `p`      | `limit = 1; playersOnly = true; sort = nearest; type = 'minecraft:player'`                   |
| `a`      | `limit = Integer.MAX_VALUE; playersOnly = true; sort = arbitrary; type = 'minecraft:player'` |
| `r`      | `limit = 1; playersOnly = true; sort = random; type = 'minecraft:player'`                    |
| `s`      | `limit = 1; playersOnly = false; currentEntity = true;`                                             |
| `e`      | `limit = Integer.MAX_VALUE; playersOnly = false; sort = arbitrary; addPredicate(e => e.isAlive())`  |

It then may have an optional arguments part, beginning with `[` and ending with `]`. Properties in the list are separated with commas (`,`),
and trailing commas are surprisingly valid. Keys can be either unquoted or quoted, while rules for values vary a lot. Here is a comprehensive
table showing rules and effects of all entity selector arguments:

| Argument        | Value                                                                                                                                                                                                                    | Applicable When...                      | Result (Pseudocode)                                                                                                                                                                      |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `advancements=` | A map from `minecraft:resource_location{category: 'advancement'}` to either Brigadier booleans or "maps from unquoted Brigadier strings to Brigadier booleans with trailing commas allowed" with trailing commas allowed | No existing `scores=`                   | `addPredicate(e => e.advancements justCheckIfItSomehowMagicallyMatches <value>)`                                                                                                         |
| `distance=`     | Non-negative `minecraft:float_range`                                                                                                                                                                                     | No existing `distance=`                 | `distance = <value>; dimensionLimited = true`                                                                                                                                            |
| `gamemode=`     | An unquoted Brigadier string of value `adventure`, `creative`, `spectator`, or `survival`                                                                                                                                | No existing `gamemode=` or `gamemode=!` | `addPredicate(e => e is Player && e.gamemode === <value>); playersOnly = true`                                                                                                           |
| `gamemode=!`    | An unquoted Brigadier string of value `adventure`, `creative`, `spectator`, or `survival`                                                                                                                                | No existing `gamemode=`                 | `addPredicate(e => e is Player && e.gamemode !== <value>); playersOnly = true`                                                                                                           |
| `level=`        | Non-negative `minecraft:int_range`                                                                                                                                                                                       | No existing `level=`                    | `addPredicate(e => e is Player && e.level in <value>); playersOnly = true`                                                                                                               |
| `limit=`        | A Brigadier integer greater than 0                                                                                                                                                                                       | No existing `limit=` and is not `@s`    | `limit = <value>`                                                                                                                                                                        |
| `name=`         | An Brigadier string (unquoted or quoted)                                                                                                                                                                                 | No existing `name=` or `name=!`         | `addPredicate(e => e.name === <value>)`                                                                                                                                                  |
| `name=!`        | An Brigadier string (unquoted or quoted)                                                                                                                                                                                 | No existing `name=`                     | `addPredicate(e => e.name !== <value>)`                                                                                                                                                  |
| `nbt=`          | `nbt:compound`                                                                                                                                                                                                           | Always                                  | `addPredicate(e => e.nbt matches <value>)`                                                                                                                                               |
| `nbt=!`         | `nbt:compound`                                                                                                                                                                                                           | Always                                  | `addPredicate(e => e.nbt !matches <value>)`                                                                                                                                              |
| `scores=`       | A map from unquoted Brigadier strings to `minecraft:int_range` with trailing commas allowed                                                                                                                              | No existing `scores=`                   | `addPredicate(e => <value>.every(([objective, range]) => e.scores[objective] in range))`                                                                                                 |
| `sort=`         | An unquoted Brigadier string of value `arbitrary`, `furthest`, `nearest`, or `random`                                                                                                                                    | No existing `sort=` and is not `@s`     | `sort = <value>`                                                                                                                                                                         |
| `team=`         | An unquoted Brigadier string                                                                                                                                                                                             | No existing `team=` or `team=!`         | `addPredicate(e => (e.team ?? '') === <value>)`                                                                                                                                          |
| `team=!`        | An unquoted Brigadier string                                                                                                                                                                                             | No existing `team=`                     | `addPredicate(e => (e.team ?? '') !== <value>)`                                                                                                                                          |
| `tag=`          | An unquoted Brigadier string                                                                                                                                                                                             | Always                                  | `addPredicate(e => <value> === '' ? e.tags.length === 0 : e.tags.includes(<value>))`                                                                                                     |
| `tag=!`         | An unquoted Brigadier string                                                                                                                                                                                             | Always                                  | `addPredicate(e => <value> === '' ? e.tags.length !== 0 : !e.tags.includes(<value>))`                                                                                                    |
| `type=`         | `minecraft:resource_location{category: 'entity_type', allowTag: true}`                                                                                                                                                   | No existing `type=` or `type=!`         | `addPredicate(e => <value> is Tag ? e.type in (<value> ?? []) : e.type === <value>); if (<value> === 'minecraft:player') playersOnly = true; if (<value> !is Tag) type = <value>` |
| `type=!`        | `minecraft:resource_location{category: 'entity_type', allowTag: true}`                                                                                                                                                   | No existing `type=`                     | `addPredicate(e => <value> is Tag ? e.type !in (<value> ?? []) : e.type !== <value>)`                                                                                                    |
| `x=`            | A Brigadier double                                                                                                                                                                                                       | No existing `x=`                        | `x = <value>; dimensionLimited = true`                                                                                                                                                   |
| `y=`            | A Brigadier double                                                                                                                                                                                                       | No existing `y=`                        | `y = <value>; dimensionLimited = true`                                                                                                                                                   |
| `z=`            | A Brigadier double                                                                                                                                                                                                       | No existing `z=`                        | `z = <value>; dimensionLimited = true`                                                                                                                                                   |
| `dx=`           | A Brigadier double                                                                                                                                                                                                       | No existing `dx=`                       | `dx = <value>; dimensionLimited = true`                                                                                                                                                  |
| `dy=`           | A Brigadier double                                                                                                                                                                                                       | No existing `dy=`                       | `dy = <value>; dimensionLimited = true`                                                                                                                                                  |
| `dz=`           | A Brigadier double                                                                                                                                                                                                       | No existing `dz=`                       | `dz = <value>; dimensionLimited = true`                                                                                                                                                  |
| `x_rotation=`   | Wrapped `minecraft:float_range`                                                                                                                                                                                          | No existing `x_rotation=`               | `addPredicate(e => e.x_rotation wrappedIn <value>)`                                                                                                                                      |
| `y_rotation=`   | Wrapped `minecraft:float_range`                                                                                                                                                                                          | No existing `y_rotation=`               | `addPredicate(e => e.y_rotation wrappedIn <value>)`                                                                                                                                      |

Note:
- Brigadier strings, no matter unquoted or quoted, are all emptyable, unless otherwise noticed.
- If `currentEntity` is set to `true`, the game will check if the current executor matches the predicate and returns it accordingly, making it one of the most efficient type of selectors.
- If `dimensionLimited` is set to `true`, the game will only search for entities in the `CommandSourceStack`'s dimension.
- If `playersOnly` is set to `true`, the game will only search in the player list instead of the entity list.
- If `type` is set, the game will only search for the specific entity types.
- If any of `distance=..max`, `x`, `y` ,`z`, `dx`, `dy`, or `dz` is set, the game will only search in the concerned sections instead of the whole dimension. This, however, will make it laggier
  if there are too many sections to check (e.g. `@e[distance=..10000000]`).

# Outro

I actually didn't plan to write such an article, but my notes kept getting more interesting as I was exploring the code.
Anyways, _Minecraft_'s command parsers are full of inconsistencies. @Mojang, please fix your game, this is virtually unplayable!!111!1

\[1\]: [view `src/share/classes/java/util/UUID.java`](http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/default/src/share/classes/java/util/UUID.java)
