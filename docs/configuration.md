# Configuration

NeoForge writes common config files under the server `config/` directory. The
file names are based on each module mod id, for example
`customclaims_protection-common.toml`.

Values below are the defaults from the current Java config classes.

## `customclaims_core-common.toml`

| Key | Default | Meaning |
| --- | --- | --- |
| `debug_logging` | `false` | Enables additional server-side Custom Claims debug logging. |
| `op_permission_level` | `2` | Operator level accepted by the permission service. |
| `metrics.enabled` | `true` | Enables anonymous heartbeat metrics on dedicated servers. |
| `metrics.endpoint` | `""` | HTTP(S) heartbeat endpoint; empty means no request is sent. |
| `permissions.default_player_permissions` | see below | Permission nodes granted to every player without requiring OP or tags. |

Default player permissions:

```toml
permissions.default_player_permissions = [
  "opac.protection.claimrules",
  "opac.protection.explosions.status",
  "opac.protection.explosions.toggle",
  "opac.protection.create.status",
  "opac.protection.create.toggle",
  "opac.protection.assembly.status",
  "opac.protection.assembly.toggle",
  "customclaims.claimrules",
  "customclaims.explosions.status",
  "customclaims.explosions.toggle",
  "customclaims.create.status",
  "customclaims.create.toggle",
  "customclaims.assembly.status",
  "customclaims.assembly.toggle"
]
```

## `customclaims_protection-common.toml`

| Key | Default | Meaning |
| --- | --- | --- |
| `foreign_interaction.block_break_limit` | `0` | Runtime-session block break limit for foreign peaceful claims. |
| `foreign_interaction.block_place_limit` | `0` | Runtime-session block place limit for foreign peaceful claims. |
| `foreign_interaction.limit_reset_interval_seconds` | `3600` | Global runtime reset interval for foreign claim counters. |
| `entity_interaction.contested_exceptions` | `["corpse:corpse"]` | Exact entity type IDs whose right-click interactions grant an OPaC bypass to attacking/defending sides in contested chunks. |
| `claimrules.toggle_cooldown_seconds` | `600` | Cooldown for side-level `/claimrules` toggles. |
| `explosions.custom_filter_enabled` | `true` | Filters block damage from explosions in protected claimed chunks via direct OPAC API sync. |
| `explosions.allow_in_war_chunks` | `true` | Does not filter explosion block damage in contested war chunks (via IWarfareBridge soft hook). |
| `big_cannons.block_projectile_launch_from_protected_claims` | `true` | Cancels Create Big Cannons projectiles spawned from protected claimed chunks and displays an actionbar alert. |
| `big_cannons.log_blocked_projectiles` | `true` | Logs Create Big Cannons projectiles blocked by explosion protection. |

The runtime `/claimrules create` side toggle is persisted in `protection/create-machines.txt`. It controls foreign Create machines (drills, saws, deployers) across borders while allowing claim owners/members to freely use their machines inside their claims.

`entity_interaction.contested_exceptions` accepts exact `namespace:path` IDs, not
wildcards or tags. An empty list (`[]`) disables these exceptions. IDs for mods
that are not installed are harmless; Corpse is not a required dependency. The
server reads the current list on each interaction, including after a config reload.
For example:

```toml
[entity_interaction]
contested_exceptions = ["corpse:corpse", "minecraft:chest_minecart"]
```

The independent `/claimrules assembly` side toggle is persisted in `protection/create-assemblies.txt`. It controls Create and optional Sable contraption assembly. On the first upgrade that creates this file, existing `create` values and their cooldowns are copied to `assembly`.

Wither blocking and villager/trader damage protection are opt-in. With the defaults above, CustomClaims neither cancels Wither spawns nor villager/trader damage, leaving vanilla and OPaC mechanics unchanged.

NeoForge preserves values already present in server config files. Existing servers that want the new passive behavior must set all three `wither_rules.disable_wither_summon_in_*` values and `villager_protection.enable_villager_protection` to `false`, or regenerate the protection config.

The removed `storage_rules.*` keys are obsolete and can be deleted from existing config files. Storage access and breaking are now governed by vanilla and OPaC only.

## Operational Defaults To Review

- Tighten `permissions.default_player_permissions` if normal players should not
  toggle side rules.
- `foreign_interaction.block_break_limit = 0` and
  `foreign_interaction.block_place_limit = 0` mean foreign peaceful block
  break/place actions are not allowed by the limit service.
