# Sol's MMNM Addon: how to use it

For **1.16.5 / MMNM 0.10.11+ / addon v0.27+** and **1.20.1 / MMNM 0.11.5+ / addon v0.28+**. Commands apply to both unless a version is marked.

Base MMNM already has protected sites, damage rules, and automatic block restoration. This guide covers the addon's changes to those tools, the settings it adds, and how to use them. You can start with one area and come back for spawning, music, or snapshots later.

**For server owners, admins, and players:** use this guide to set up protected areas, understand their rules, or find help with a specific feature. An ability can be disabled in one location, work without changing terrain in another, and behave normally outside those areas. Each site can also have its own entry messages, music, and weather.

Install the matching addon and MMNM on both the server and clients. Management commands require permission level **3**; players without that permission can use this guide to understand the settings that affect them. In single-player, enable cheats to use the setup commands.

## Find what you need

- [Start with one area](#start-with-one-area)
- [Commands that moved](#commands-that-moved)
- [Create, resize, and manage sites](#create-resize-and-manage-sites)
- [What each site setting does](#what-each-site-setting-does)
- [Block one ability or stop its terrain damage](#block-one-ability-or-stop-its-terrain-damage)
- [Control mobs and aggression](#control-mobs-and-aggression)
- [Set up custom spawning](#set-up-custom-spawning)
- [Entry messages, music, and weather](#entry-messages-music-and-weather)
- [Reuse settings with profiles](#reuse-settings-with-profiles)
- [Automatic restoration and containers](#automatic-restoration-and-containers)
- [Save or reset an area](#save-or-reset-an-area)
- [Config file reference](#config-file-reference)
- [When something doesn't work](#when-something-doesnt-work)

## Start with one area

Stand at the first corner of your build:

```text
/abprotect site new_cuboid pos1 arena
```

Move to the opposite corner, including the height you want, then run:

```text
/abprotect site new_cuboid pos2
/abprotect view protection true
/abprotect site info arena
```

This selects the blocks at your **feet**, not the block you're looking at. Both corners are included. The two positions must be in the same dimension.

For an arena where abilities can damage blocks and those blocks restore afterward:

```text
/abprotect props abilities_use arena true
/abprotect props block_destruction arena true
/abprotect props block_restoration arena true
```

Those commands only set ability use and terrain behavior. To allow combat damage as well:

```text
/abprotect props entity_damage arena true
/abprotect props player_damage arena true
```

Check `site info arena` before opening it to players. A new site's defaults are not a ready-made combat preset.

## Commands that moved

`/abprotect` is the short form. `/abilityprotection` uses the same reorganized command tree. When MMNM's master-command option is enabled, `/mmnm abprotect` is also available; MMNM places its canonical command under the master as configured.

| Base MMNM command | With the addon |
| --- | --- |
| `/abilityprotection new arena 20` | `/abprotect site new arena 20` |
| `/abilityprotection resize arena 30` | `/abprotect site resize arena 30` |
| `/abilityprotection rename arena arena2` | `/abprotect site rename arena arena2` |
| `/abilityprotection info arena` | `/abprotect site info arena` |
| `/abilityprotection list` | `/abprotect site list` |
| `/abilityprotection remove arena` | `/abprotect site remove arena` |
| `/abilityprotection view true` | `/abprotect view protection true` |
| `/abilityprotection props block_restoration arena true` | `/abprotect props block_restoration arena true` |
| `/abilityprotection props toggle arena` (MMNM 0.11.5) | `/abprotect site toggle arena` or `/abprotect props toggle arena` |

Use the new paths; the old top-level `new`, `info`, `list`, and similar paths are replaced.

Throughout this guide, `<site>`, `<ability>`, and `<entity>` mean values you supply, without angle brackets. `true` enables the named permission or option; `false` disables it. **20 ticks = about 1 second at normal server speed.**

Sites can be selected by their stored name or their **1-based number from `site list`**. Use names in saved instructions: deleting a site can shift later list numbers. Numeric names are interpreted as numbers first. Names are normalized by MMNM when created; use the name printed by `site list`. Quote arguments containing spaces, but simple names such as `arena` are easiest. Commands look in the executing dimension.

## Create, resize, and manage sites

| Command after `/abprotect` | What it does |
| --- | --- |
| `site new <name> <size>` | Creates the usual MMNM cube around your command position. Size is the distance from its center on each axis: `20` covers 41 block positions along each axis, within the world. Allowed size: 1–59,999,999. Large allowed values are not a recommendation. |
| `site new_cuboid pos1 <name>` | Stores your first corner. Running it again replaces your pending selection. |
| `site new_cuboid pos2` | Creates the exact 3D box between your positions. `pos2 exact` means the same thing. |
| `site new_cuboid pos2 full_height` | Uses the selected X/Z rectangle through the dimension's full build height. |
| `site resize <site> <size>` | Sets the ordinary MMNM size. **On a custom cuboid, this clears the custom corners and converts it to a centered cube.** |
| `site rename <site> <new_name>` | Renames the site and moves its addon settings/snapshot association with it. |
| `site info <site>` | Shows settings, shape/bounds, and restoration information. |
| `site list` | Shows the saved site order and selection numbers. |
| `site remove <site>` | Removes protection and its active addon association. It does not restore the build first. |
| `site toggle <site>` | **1.20.1 with MMNM 0.11.5:** switches the protection's enabled state without deleting it. Not present on 1.16.5. |
| `view protection true/false` | Shows/hides protection boundaries for you. |
| `view hud_text true/false` | Shows/hides your administrator HUD text identifying the current protected site. It does not disable public entry messages. |

Resize, rename, and remove are refused while that site's snapshot operation is active. Cuboid bounds are saved; you don't need to select them again after restarting.

Overlapping sites still depend on MMNM's site selection for most rules. Do not assume their rules combine or that the last-created site always wins. Entry/exit messages separately track each area crossed, including nested areas.

## What each site setting does

Use `/abprotect props <setting> <site> <value>`, for example:

```text
/abprotect props player_build arena false
```

The first table includes inherited MMNM settings because they still determine how the addon's features work.

| Setting | New-site default | Meaning |
| --- | --- | --- |
| `abilities_use` | `false` | Allows ability use when `true`. MMNM's own configured protection whitelist can provide exceptions; the addon's explicit ability blacklist is an additional restriction. |
| `block_destruction` | `false` | Allows terrain destruction handled by MMNM protection. Set `true` if you want destructive fights followed by restoration. |
| `block_restoration` | `false` | Enables MMNM's automatic restoration queue. This is separate from a manual saved-area snapshot. |
| `entity_damage` | `false` | MMNM's entity-damage permission. The addon also blocks entity-caused attacks involving protected participants and clears mob targets, so this affects aggression as well. |
| `player_damage` | `false` | MMNM's player-damage permission. Enabling this alone does not bypass `entity_damage false`. |
| `stat_loss` | `true` | Allows MMNM's stat-loss behavior. `false` disables it where MMNM applies the site's rule. |
| `death` | `true` | Allows MMNM death behavior. `false` uses MMNM's protected unconscious behavior instead. |
| `mob_spawns` | `true` | MMNM's general mob-spawn permission. The addon also has separate entity filters and managed spawn rules below. |
| `player_build` **(added)** | `true` | Controls player block breaking and placement; `false` also blocks bucket use at protected positions. It is not a general chest/door interaction lock. MMNM wanted-poster packages are exempt from its normal break restriction. |
| `mined_block_drops` **(added)** | `true` | Allows ordinary player-mined block drops in sites with both destruction and restoration enabled. `false` suppresses those drops. It neither permits mining through `player_build false` nor enables ability-generated drops. Normal drop rules still apply. |

| Timing setting | Default / accepted range | Meaning |
| --- | --- | --- |
| `unconscious_time` | 40 ticks / 0–1,200 | MMNM's unconscious duration when death is disabled. |
| `restoration_interval` | 20 ticks / 0–1,200 | Interval between automatic restoration passes. `0` removes the interval wait; it does not remove grace time or other eligibility checks. |
| `restoration_amount` | 15 / 1–500 | Number of blocks a restoration pass may process. Other limits can make the actual amount smaller. |
| `restoration_distance` | 10 blocks / 0–1,000 | Defers a block while a player is nearby. `0` disables this proximity check. **It is not the site's radius.** |

MMNM's global protection grace time also delays restoration after damage. Raising `restoration_amount` won't bypass that delay or make a block restore while its other conditions fail.

## Block one ability or stop its terrain damage

These are two different lists:

- **`blacklist`** prevents a player from using a listed ability while in the selected site. Equipped blocked abilities get a disabled indicator/message; the addon removes its own visual restriction after leaving.
- **`nogrief`** blocks a listed ability's supported terrain changes at protected block positions. It does not grant permission to cast, disable combat damage, or promise that every third-party ability uses a supported block-change path.

```text
/abprotect props ability nogrief add <ability> <site>
/abprotect props ability nogrief remove <ability> <site>
/abprotect props ability nogrief list <site>
/abprotect props ability nogrief clear <site>
```

Replace `nogrief` with `blacklist` for the same four operations. **Ability comes before site** for `add` and `remove`. Use Tab to select the registered ability accepted by your installed MMNM/addons; don't substitute a guessed display name.

To allow fighting with one terrain-heavy ability protected, enable `abilities_use`, set combat damage as wanted, and add that ability to `nogrief`. To prevent the ability being cast there at all, use `blacklist`. Clearing either list does not change the site's other permissions.

## Control mobs and aggression

```text
/abprotect props entity blacklist add arena minecraft:zombie
/abprotect props entity blacklist remove arena minecraft:zombie
/abprotect props entity blacklist list arena
/abprotect props entity blacklist clear arena
```

The same operations work with `whitelist`. These filters apply to **mobs joining/loading into the world in that site**, not every entity type and not an immediate purge of mobs already standing there.

**Whitelist entries are exceptions to the blacklist.** Adding one does not automatically block everything else. To admit only selected mobs:

```text
/abprotect props entity blacklist add arena all
/abprotect props entity whitelist add arena minecraft:cow
```

`blacklist remove arena all` removes the blanket rule; individually blacklisted entries remain. `blacklist clear arena` removes both. The whitelist overrides this addon's blacklist, but does not override every spawning restriction from MMNM or another mod.

For a peaceful area, use `props entity_damage arena false`. There is no separate `noaggro` command: the addon ties attack cancellation and clearing mob targets to the entity-damage setting. This is not a blanket cancellation of environmental damage such as falling.

## Set up custom spawning

Managed spawning makes periodic attempts near non-spectator players in or near the site. It uses loaded chunks and checks space and the site's entity filter. It creates mobs, not arbitrary projectiles or dropped items.

Example: try spawning one or two zombies every 30 seconds, with a 50% chance and a population cap of six:

```text
/abprotect props entity spawnrules defaults arena minecraft:zombie
/abprotect props entity spawnrules interval arena minecraft:zombie 600
/abprotect props entity spawnrules chance arena minecraft:zombie 50
/abprotect props entity spawnrules pack arena minecraft:zombie 1 2
/abprotect props entity spawnrules cap arena minecraft:zombie 6
```

Every row below starts with `/abprotect props entity spawnrules`.

| Rest of command | Default / what it does |
| --- | --- |
| `defaults <site> <entity>` | Creates a rule if missing. **Does not reset an existing rule.** Setting an individual option also creates a missing rule. |
| `enabled <site> <entity> true/false` | Default `true`. Pauses/resumes new attempts for this rule. |
| `interval <site> <entity> <ticks>` | Default 600; range 20–72,000. Delay between attempts, not a guaranteed spawn interval. |
| `chance <site> <entity> <percent>` | Default 35; range 0–100. Chance that an attempt proceeds. `0` means no successful attempts. |
| `pack <site> <entity> <min> <max>` | Default 1–3; each 1–64. Chosen group size, reduced by remaining cap/available positions. A max below min is raised to min. |
| `cap <site> <entity> <count>` | Default 8; range 0–512. Limits further spawning. Counts matching loaded mobs in the site and tracked managed mobs; lowering it doesn't delete existing mobs. `0` prevents further spawns. |
| `placement <site> <entity> <mode>` | Default `natural`. `natural` and `surface` currently use the same surface search. `water` searches water positions; `air` searches empty positions around player height. These are location modes, not a guarantee of vanilla biome/light spawning rules. |
| `persistent <site> <entity> true/false` | Default `true`. Protects managed mobs from normal despawning. Changing it is not a command to remove existing mobs. |
| `despawn_after <site> <entity> <ticks>` | Default 0; range 0–1,728,000. `0` means no addon expiry timer. An expired mob waits while it has a target or a non-spectator player is within 48 blocks. |
| `remove <site> <entity>` | Removes that spawn rule; does not remove mobs it already created. |
| `list <site>` | Prints configured rules and their values. |
| `clear <site>` | Removes all managed spawn rules on that site. |

A positive despawn timer also keeps newly managed mobs persistent until the addon's cleanup can remove them. Existing mobs keep the expiry time assigned when they spawned. Unloaded managed mobs can retain cap reservations, so leaving and returning does not necessarily refill the population immediately.

These rules are a separate spawning system. MMNM's `mob_spawns false` is not the switch used by the managed scheduler; use the rule's `enabled false`, `remove`, or `clear` to stop its attempts. Other event handlers can still reject spawned mobs.

## Entry messages, music, and weather

**Messages**

```text
/abprotect props entering_message set arena Arena | Fight here. Leave the town alone.
/abprotect props entering_message preview arena entering
/abprotect props entering_message preview arena leaving
```

The text before `|` is the title; the rest is the subtitle. The addon adds **Entering** or **Leaving** automatically. Despite the command name, the same title/subtitle is used for both directions; separate enter/leave text is not offered.

Other operations: `title <site> <text>`, `subtitle <site> <text>`, `info <site>`, and `clear <site>`, all after `/abprotect props entering_message`. Set a title before a subtitle. Title/subtitle limits are 160/220 characters. `set <site> "Title with spaces" subtitle text` is also accepted. Messages are shown to visiting players, not just operators.

**Music**

```text
/abprotect props music add arena minecraft:music_disc.cat
/abprotect props music shuffle arena true
/abprotect props music list arena
```

Use a Minecraft sound ID, not a file path or web link. Add up to **64 distinct tracks**. Without shuffle they play in list order; with shuffle the order is shuffled for each playlist cycle. The playlist repeats, with about five seconds between tracks, and uses the client's **Music** volume.

`music remove <site> <sound>` removes one track; `music clear <site>` removes the playlist; `music shuffle <site> false` returns to list order. Prefix each with `/abprotect props`.

Custom sounds must exist on each player's client/resource pack. The addon can also resolve `yourpack:arena_theme` directly from `assets/yourpack/sounds/arena_theme.ogg`. It doesn't send audio files to players. Tab suggestions can include client resource-pack sound events, but accepting an ID doesn't prove every player has its sound.

**Weather**

```text
/abprotect props weather_cycle arena no_weather
```

Modes: `default` follows world weather; `rain` shows rain; `thunderstorm` uses storm weather; `no_weather` clears it locally. Leaving returns to the applicable area's/world's weather. Server checks for rain at a position are adjusted too, while roof and biome conditions still apply. This does not change the dimension's global weather timer or create a separate lightning schedule.

## Reuse settings with profiles

```text
/abprotect profile create arena_rules
/abprotect profile copy arena arena_rules
/abprotect profile apply arena_rules arena2
```

`copy` goes **site → existing profile**. `apply` goes **profile → existing site**. It copies values once: later profile edits don't automatically update sites.

| Command after `/abprotect profile` | Use |
| --- | --- |
| `create <name>` | Creates a profile with size 10 and the defaults shown in the settings tables. |
| `list` | Lists profiles and their selection numbers. |
| `info <profile>` | Shows its values. `<profile> info` also works. |
| `modify <profile> <setting> <value>` | Edits size, the ten boolean settings in the table, or the four timing settings. Uses the same ranges. |
| `modify <profile> music add/remove <sound>` | Edits its playlist. |
| `modify <profile> music list/clear` | Shows/removes its playlist. |
| `modify <profile> music shuffle true/false` | Changes its shuffle setting. |
| `remove <profile>` | Deletes the profile, without undoing settings already applied to sites. |

Profiles include size, the base protection settings, building, mined drops, and music. They **do not copy** custom corner coordinates, ability lists, entity lists, spawn rules, weather, entry messages, snapshots, or the site's enabled/disabled state. Set those on each site.

Applying a profile changes a normal site's size. On an existing custom cuboid, its explicit corners remain in place; the profile isn't a way to clone or resize that shape.

## Automatic restoration and containers

With `block_destruction true` and `block_restoration true`, supported damage is queued for repair. The addon improves coordinate lookup and preserves the original block snapshot through overlapping changes. For ability-damaged containers, it captures inventory data and suppresses duplicate content drops while that restoration is pending.

This is **not continuous inventory backup**. Automatic restoration uses captured data from the damage path. A manual snapshot, below, restores the inventory saved when that snapshot was made.

The addon journals pending restoration entries, including block-entity data, to disk and recovers them after restart. This supplements MMNM's own saving: MMNM 0.11.5 already serializes restoration data, but the addon also retains payloads such as container data through its own journal. A sudden crash can still lose changes that hadn't reached disk.

Ability-generated block drops are suppressed in restorable areas to avoid dropping items and then recreating the same blocks. `mined_block_drops` is the separate exception for ordinary player mining.

## Save or reset an area

A snapshot is a saved block layout, including block-entity data such as container contents. Use it to reset an arena to a chosen state. It is not a backup of players, mobs, every entity, or the whole world.

```text
/abprotect snapshot savearea arena
/abprotect snapshot status arena
```

Wait for completion before relying on the save. If a completed snapshot already exists, replacing it requires:

```text
/abprotect snapshot savearea arena confirm
```

To reset the saved area:

```text
/abprotect snapshot restorearea arena
/abprotect snapshot restorearea arena confirm
```

The first command explains what will be overwritten. Confirmation replaces changed blocks and saved container contents within the **original saved bounds**, even if the site was resized later. It also clears that site's live restoration queue so older queued damage does not fight the reset.

Both saving and restoring temporarily lock interactions and relevant block simulation in the footprint. Block changes, container interaction, pistons, and related updates are restricted while the job runs. Admins see progress bars; visitors inside see an operation warning. Work is split over ticks, adapts to available time, and can pause under server load. Persisted jobs can resume after restarting; errors may require `retry` after their cause is resolved.

| Command after `/abprotect snapshot` | Use |
| --- | --- |
| `status <site>` | Shows the snapshot, saved bounds, job progress, load/error status, and storage location. |
| `cancel <site>` | Cancels an unfinished **save** and keeps the previous completed snapshot. It does **not** cancel an active restore. |
| `retry <site>` | Retries a job suspended after an error. Fix the reported disk/data problem first. |
| `fastmode <site> enable` | Displays the fast-mode warning. |
| `fastmode <site> enable confirm` | Gives one active job a larger budget. Can increase CPU, memory, disk work, and lag; use during maintenance. |
| `fastmode <site> disable` | Returns the job to its normal budget; it keeps running. |

Only one job can use fast mode at a time. Fast mode returns to normal after a restart, and can't accelerate the final durable-write stage. One site's save/restore cannot overlap another operation on that same site. Don't edit profiles/settings during an operation; wait for completion.

Storage is under `<world>/solsmmnmaddon/area_snapshots/`. The default keeps two completed generations, but commands restore the active generation; there is no in-game command to pick an older one. A replacement save only becomes active once completed. Back up this folder with the world if you need to retain recovery data.

## Config file reference

The file is `config/solsmmnmaddon-common.toml`. For a dedicated server, edit its copy while stopped and restart. These are global settings, unlike the per-site commands. Leave snapshot budgets at their defaults unless you have a reason to tune them.

The table uses `section.key` notation; TOML groups keys under sections such as `[gura]` and `[snapshots]`.

| Key | Default; allowed range | Meaning |
| --- | --- | --- |
| `gura.disableGekishinLaunchedDestroyedBlocks` | `true` | Stops Gekishin's launched block debris. `false` permits it again. |
| `gura.disableShimaYurashiLaunchedDestroyedBlocks` | `true` | Same option for Shima Yurashi, independently. |
| `snapshots.blocksPerTick` | 4,096; 128–65,536 | Base ceiling for detailed snapshot block work per tick. Empty sections can be saved in bulk; adaptive/fast budgets can scale work. |
| `snapshots.tickBudgetMillis` | 4; 1–20 | Base main-thread time budget. Live-queue capture stays within this base budget. |
| `snapshots.adaptiveSaveBudgetMillis` | 12; 1–40 | Maximum save budget when the server has spare tick time. |
| `snapshots.adaptiveRestoreBudgetMillis` | 12; 1–40 | Equivalent maximum for restoration. |
| `snapshots.fastModeTickBudgetMillis` | 30; 5–45 | Maximum budget for confirmed fast mode; values above 40 are discouraged by the config. |
| `snapshots.fastModeBlocksPerTick` | 131,072; 4,096–1,048,576 | Fast-mode block-work ceiling. The time limit can stop work sooner. |
| `snapshots.minimumFreeDiskMb` | 1,024; 128–1,048,576 | Refuses a new snapshot below this free-space threshold. It isn't an estimate of the space that save will need. |
| `snapshots.retainGenerations` | 2; 1–10 | Completed snapshot generations retained per site. |
| `restorationPersistence.flushIntervalTicks` | 10; 1–200 | Opportunities to journal dirty restoration chunks. Large backlogs can take longer; clean shutdown also flushes changes. |

## When something doesn't work

| Problem | Check |
| --- | --- |
| Old command says “incorrect argument” | Use `site new/info/list/...` or `view protection`, not the old top-level path. Management needs permission level 3. |
| Site can't be found | Run `site list` in the correct dimension. Use the printed stored name or current number. |
| Ability still can't be used after adding no-grief | No-grief doesn't enable casting. Check `abilities_use`, the ability blacklist, and MMNM's normal ability requirements. |
| An ability fired outside still reaches the area | Blacklisting checks the player's casting location. Use appropriate no-grief and damage rules for effects reaching protected positions. |
| Players can't place or break | Check `player_build`, other protection rules, and whether a snapshot save/restore is locking the footprint. |
| Blocks won't restore | Check destruction/restoration settings, the queue, MMNM grace time, player proximity, loaded chunks, and any active snapshot job. On 1.20.1, missing support can also postpone a block. |
| My whitelist didn't remove other mobs | It only overrides blacklists. Use `blacklist ... all` for an allow-only setup; existing mobs aren't purged. |
| Custom mobs aren't spawning | Check enabled/chance/cap, entity filters, nearby players, loaded space, and placement. Managed rules only create mobs. |
| `defaults` didn't reset my spawn rule | Remove the rule, then create it again. `defaults` leaves existing values alone. |
| A timed mob hasn't despawned | Expiry waits while a player is within 48 blocks or the mob has a target; unloaded mobs aren't processed until loaded. |
| Music is silent | Check Music volume and the sound ID/resource pack on that client. The server doesn't distribute the audio through this command. |
| Profile missed some settings | Ability/entity lists, spawn rules, weather, messages, snapshots, and cuboid corners are not profile fields. |
| Snapshot appears stuck | Use `snapshot status`. A load pause resumes with available capacity; a suspended error needs its cause resolved before `retry`. |
| A save is too large | Check whether you selected `full_height`. An exact cuboid can avoid saving unnecessary vertical space. |

For a bug report, include Minecraft, MMNM, addon and Forge versions; the command you ran; `site info`; and the ability/entity involved. For snapshot problems, add `snapshot status` and the relevant log error.
