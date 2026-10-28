# Server Admin Guide

This guide is for server owners and operators running **Open Parties and
Claims: Better Protection** on a NeoForge server.

## Install

Use the `opac-better-protection` distribution jar for normal server installs:

```text
opac-better-protection/build/libs/opac-better-protection-<version>.jar
```

For this release, that file is `opac-better-protection-2.0.0.jar`.

Required server mods:

- NeoForge for Minecraft `1.21.1` baseline, with experimental same-jar
  probes allowed through Minecraft `[1.21.1,26.0)` and NeoForge
  `[21.1.232,26.0)`
- Open Parties and Claims `0.27.5` or newer on the baseline stack
- Open Parties and Claims: Better Protection

The public jar is one modern artifact. `Minecraft 1.21.1 + NeoForge 21.1.232`
is the full tested baseline; newer 1.21.x releases are experimental
metadata-enabled support and must pass smoke tests before being called verified.
Forge, Fabric, Quilt, older `1.20.x`, and Minecraft `26.x` are not supported by
this jar - 26.x has incompatible APIs; see [Compatibility](compatibility.md).

Optional integrations:

- Create enables independent contraption-assembly and block-breaking protection.
- Create Aeronautics/Offroad enables Borehead Bearing and Rock Cutting Wheel
  protection through `/claimrules create`.
- Sable `2.0.3` enables optional contraption assembly protection through
  `/claimrules assembly`.
- Create Big Cannons enables cannon launch and terrain-damage protection.

The `opac-better-protection` jar declares multiple mod ids inside one jar. That is
expected: the jar contains core, protection, Create compat, Aeronautics/Offroad
compat, and Big Cannons compat modules.

## First Start Checklist

1. Back up the world.
2. Install Open Parties and Claims and `opac-better-protection-*.jar`.
3. Start the server once so NeoForge writes the common config files.
4. Review the generated config files documented in
   [Configuration](configuration.md).
5. Confirm `permissions.default_player_permissions` matches the command access
   you want for normal players.
6. Test `/claimrules explosions status`, `/claimrules create status`,
   and `/claimrules assembly status` with a non-operator player.

## Claim Rules Flow

`/claimrules` manages side-level territory protection toggles:

```text
/claimrules gui
/claimrules explosions status
/claimrules explosions enable
/claimrules explosions disable
/claimrules create status
/claimrules create enable
/claimrules create disable
/claimrules assembly status
/claimrules assembly enable
/claimrules assembly disable
/claimrules limits status
/claimrules limits reset
/claimrules limits resetall
```

`/claimrules explosions enable` enables explosion protection for the player's
current side. Players in a party manage nation rules; solo players manage
personal-claim rules.

`/claimrules create disable` blocks Create drills, saws, shared
`BlockBreakingMovementBehaviour` machines, and Aeronautics/Offroad Borehead
Bearing + Rock Cutting Wheel mining on the side's territory. Mining machines are
blocked by default until the side enables them.

`/claimrules assembly disable` independently blocks Create and Sable contraption
assembly. A structure entirely outside claims remains allowed. Once it touches a
claim, all eight corners of its source AABB must be peaceful claims of one side
with assembly enabled; mixed, foreign, and unclaimed corners are blocked. Active
contested-war chunks retain normal machine behavior.

`/claimrules gui` opens the optional client screen when the client also has the
protection module installed. Commands remain the full fallback. Client
installs also get an `Open Claim Rules` keybind in the `CustomClaims` controls
category (defaults to `K`).

Side-level toggles use `claimrules.toggle_cooldown_seconds`. Status commands
and `/claimrules gui` do not spend cooldown. Console and admin bypass cooldown.

## Permissions

Players can satisfy permissions through player tags, configured default-player
permissions, or the configured operator level. Console has admin-level access.

Permission nodes:

```text
opac.admin
opac.bypass
opac.protection.claimrules
opac.protection.explosions.status
opac.protection.explosions.toggle
opac.protection.create.status
opac.protection.create.toggle
opac.protection.assembly.status
opac.protection.assembly.toggle
opac.protection.limits.reset
opac.protection.bypass
opac.protection.admin
```

Legacy aliases (`customclaims.claimrules`, `customclaims.explosions.status`, etc.)
are supported for backward compatibility.

Default player permissions are configured in `customclaims_core-common.toml`.
By default, normal players can inspect and toggle explosion, Create-mining,
and assembly rules. Admin commands and limit-reset commands are not granted
by default.

## Protection Behavior

### Explosion Protection

Explosion protection removes protected claimed blocks from normal explosion
damage when a side has explosion protection enabled. If
`explosions.allow_in_war_chunks = true`, explosion block damage is not filtered
inside contested war chunks (detected via soft reflection bridge to Warfare).

### Create Big Cannons Protection

When `big_cannons.block_projectile_launch_from_protected_claims` is `true`,
Create Big Cannons projectiles fired from inside a protected claim are blocked
and an alert is displayed to the shooter.

### Corpse And Contested Entity Interactions

In `config/customclaims_protection-common.toml`,
`entity_interaction.contested_exceptions` defaults to `["corpse:corpse"]`. This
allows players to right-click a Corpse entity in contested chunks without OPaC
blocking the interaction. Corpse still decides who can retrieve its contents;
this setting does not override its ownership rules.

Add exact entity type IDs (`namespace:path`) to support other mods, or use `[]`
to disable the feature.

### Optional Mob Protection

Wither blocking and villager/trader damage protection are disabled by default.
At those defaults the handlers do not cancel spawns or damage, so vanilla and
OPaC mechanics remain unchanged. Both features remain available as explicit
server-config opt-ins.

## Runtime Data

Runtime data is stored under the world folder:

```text
world/customclaims/
```

Important files:

- `protection/explosion-protection.txt`: side explosion protection toggles.
- `protection/create-machines.txt`: side Create/Offroad mining allow/block toggles.
- `protection/create-assemblies.txt`: side Create/Sable assembly allow/block toggles.
- `protection/claimrule-toggle-cooldowns.txt`: last side toggle timestamps for cooldowns.
- `logs/actions.log`: general action log.
- `metrics/server-id.txt`: random anonymous id used by dedicated-server metrics.

Foreign interaction counters are runtime-only and are not persisted. They reset
globally on `foreign_interaction.limit_reset_interval_seconds` or through
`/claimrules limits resetall`.

## Operations

- Back up the world before installing, upgrading, or changing protection configs
  on production servers.
- Keep Open Parties and Claims installed and healthy; this addon relies on OPaC
  as the authority for parties and claims.
- If players report unexpected claim damage, inspect side `/claimrules` settings
  and protection config.
- Set `metrics.enabled = false` in `customclaims_core-common.toml` and restart
  to disable anonymous metrics.
