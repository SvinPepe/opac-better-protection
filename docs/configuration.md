# Configuration

Configuration files are written by NeoForge to the server `config/` directory. File names correspond to module mod IDs:
- `config/customclaims_core-common.toml`
- `config/customclaims_protection-common.toml`

All keys, types, and defaults documented below are verified against the Java source classes `CoreConfig.java` and `ProtectionConfig.java`.

---

## `customclaims_core-common.toml`

Defined in `dev.customclaims.core.config.CoreConfig`.

| Config Key | Type | Default | Valid Range | Meaning |
| --- | --- | --- | --- | --- |
| `debug_logging` | Boolean | `false` | `true`, `false` | Enables verbose server-side debug logging for Custom Claims services. |
| `op_permission_level` | Integer | `2` | `0` .. `4` | Minimum operator level accepted for administrative commands and bypass checks. |
| `metrics.enabled` | Boolean | `true` | `true`, `false` | Enables anonymous heartbeat metrics for dedicated servers. |
| `metrics.endpoint` | String | `"https://customclaims-metrics.svinpepe.workers.dev/heartbeat"` | URL or empty string | HTTP(S) endpoint receiving anonymous metrics heartbeats. Set to `""` to prevent network requests. |
| `permissions.default_player_permissions` | List of Strings | see below | Permission nodes | Permission nodes automatically granted to every connected player without requiring OP or permission plugins. |

### Default Player Permissions

```toml
[permissions]
default_player_permissions = [
  "opac.warfare.start",
  "opac.warfare.status",
  "opac.protection.claimrules",
  "opac.protection.explosions.status",
  "opac.protection.explosions.toggle",
  "opac.protection.create.status",
  "opac.protection.create.toggle",
  "opac.protection.assembly.status",
  "opac.protection.assembly.toggle",
  "customclaims.war.start",
  "customclaims.war.status",
  "customclaims.claimrules",
  "customclaims.explosions.status",
  "customclaims.explosions.toggle",
  "customclaims.create.status",
  "customclaims.create.toggle",
  "customclaims.assembly.status",
  "customclaims.assembly.toggle"
]
```

---

## `customclaims_protection-common.toml`

Defined in `dev.customclaims.protection.config.ProtectionConfig`.

| Config Key | Type | Default | Valid Range | Meaning |
| --- | --- | --- | --- | --- |
| `foreign_interaction.block_break_limit` | Integer | `0` | `0` .. `100000` | Maximum number of blocks a foreign player can break in peaceful claims during a runtime session. `0` disables foreign breaking entirely. |
| `foreign_interaction.block_place_limit` | Integer | `0` | `0` .. `100000` | Maximum number of blocks a foreign player can place in peaceful claims during a runtime session. `0` disables foreign placing entirely. |
| `foreign_interaction.limit_reset_interval_seconds` | Integer | `3600` | `1` .. `604800` | Global interval in seconds between automatic resets of foreign player interaction counters. (Default: 3600 = 1 hour). |
| `claimrules.toggle_cooldown_seconds` | Integer | `600` | `0` .. `86400` | Cooldown period in seconds required between side-level `/claimrules` toggles (explosions, Create machines, contraption assembly). (Default: 600 = 10 minutes). |
| `explosions.custom_filter_enabled` | Boolean | `true` | `true`, `false` | Enables Custom Claims explosion filtering. When true, explosion block damage is cancelled in claimed chunks where side explosion protection is enabled. |
| `explosions.allow_in_war_chunks` | Boolean | `true` | `true`, `false` | When true, explosion block filtering is disabled in active contested war chunks (via `IWarfareBridge`), allowing explosions to breach fortifications during war. |
| `big_cannons.block_projectile_launch_from_protected_claims` | Boolean | `true` | `true`, `false` | When true, Create Big Cannons projectiles fired from inside a protected claimed chunk are cancelled before munitions are spent, and an actionbar alert is shown. |
| `big_cannons.log_blocked_projectiles` | Boolean | `true` | `true`, `false` | When true, logs Create Big Cannons projectile launches blocked by claim protection to the server console and action log. |
| `entity_interaction.contested_exceptions` | List of Strings | `["corpse:corpse"]` | List of `namespace:path` | Exact entity type identifiers whose right-click interactions bypass OPaC claim cancellation for participants in contested war chunks. |

### Contested Entity Exceptions

The `entity_interaction.contested_exceptions` key allows right-clicking specified entities inside contested war chunks without OPaC claim interference:
- By default, it contains `["corpse:corpse"]`, enabling players to retrieve their death corpses during an active siege.
- Accepts exact `namespace:path` IDs only (no wildcards or tags).
- If the entity mod (e.g. Corpse) is not installed, the entry is harmless and ignored.
- Set to `[]` to disable entity bypasses entirely.
- Changes take effect immediately on config reload.

Example:

```toml
[entity_interaction]
contested_exceptions = [
  "corpse:corpse",
  "gravestone:gravestone"
]
```

---

## Runtime State Files (`world/customclaims/`)

Certain toggle states and cooldown timers are persisted in the world folder under `world/customclaims/protection/`:

- `protection/explosion-protection.txt`: Persistent side-level explosion protection toggles (`sideId=true|false`).
- `protection/create-machines.txt`: Persistent side-level Create machine permissions (`sideId=true|false`).
- `protection/create-assemblies.txt`: Persistent side-level contraption assembly permissions (`sideId=true|false`).
- `protection/claimrule-toggle-cooldowns.txt`: Cooldown expiration timestamps per side and rule (`sideId:rule=epochSecond`).

---

## Obsolete & Removed Keys

In Better Protection 2.0:
- **`wither_rules.*`** (`disable_wither_summon_in_overworld`, etc.): Removed. Wither spawning and mob griefing are governed authoritatively by vanilla Minecraft game rules and OPaC claim settings.
- **`villager_protection.*`** (`enable_villager_protection`, etc.): Removed. Villager protection is governed by OPaC and standard protections.
- **`storage_rules.*`**: Removed. Container access and breaking are governed authoritatively by OPaC claim permissions.

If upgrading from 1.x, these obsolete sections may be safely deleted from your `.toml` configuration files.
