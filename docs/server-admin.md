# Server Admin Guide

This guide is for server owners and operators running **Open Parties and Claims: Better Protection** on a NeoForge server.

## Installation

Install the single distribution jar built by the `opac-better-protection` module:

```text
opac-better-protection/build/libs/opac-better-protection-<version>.jar
```

Example filename: `opac-better-protection-2.0.0.jar`.

### Requirements

- **Minecraft:** `1.21.1` baseline (experimental metadata range `[1.21.1,26.0)`, tested through 1.21.11)
- **NeoForge:** `21.1.232` baseline (experimental metadata range `[21.1.232,26.0)`)
- **Java:** `21`
- **Open Parties and Claims:** `neoforge-1.21.1-0.27.5` or newer

### Optional Supported Integrations

- **Open Parties and Claims: Warfare (`2.0.0`+):** Companion warfare mod. When present, enables dynamic soft warfare bridging in contested chunks.
- **Create (`mc1.21.1-6.0.9`+):** Mechanical contraption, drill, and deployer claim protection.
- **Create Big Cannons (`5.11.7`+):** Cannon launch protection from protected claims and impact logging.
- **Create Aeronautics / Offroad (`1.3.0`+):** Borehead Bearing and Rock Cutting Wheel bore mining protection.
- **Sable (`2.0.3`+):** 8-corner bounding box contraption assembly protection.

Note: The `opac-better-protection` jar contains multiple mod IDs (`customclaims_core` via JarJar, `customclaims_protection`, `customclaims_create`, `customclaims_aeronautics`, `customclaims_big_cannons`, `opac_better_protection`). This is normal and intentional.

## Commands Reference

The root command `/claimrules` manages territory protection toggles and foreign interaction limits:

```text
/claimrules
/claimrules gui
/claimrules limits me
/claimrules limits reset <player>
/claimrules limits resetall
/claimrules explosions status
/claimrules explosions enable
/claimrules explosions disable
/claimrules create status
/claimrules create enable
/claimrules create disable
/claimrules assembly status
/claimrules assembly enable
/claimrules assembly disable
```

### Command Details

| Command | Permission Required | Description |
| --- | --- | --- |
| `/claimrules` or `/claimrules gui` | `opac.protection.claimrules` | Opens the Claim Rules GUI on clients with the mod installed. Players without the client mod receive an instructional message directing them to use text commands. |
| `/claimrules limits me` | *(None / player only)* | Displays the caller's personal foreign interaction counter: blocks broken, blocks placed, and configured session limits. |
| `/claimrules limits reset <player>` | `opac.protection.limits.reset` | Resets the foreign interaction block break and place counters for the specified target player. |
| `/claimrules limits resetall` | `opac.protection.limits.reset` | Resets foreign interaction counters for all players across the server. |
| `/claimrules explosions status` | `opac.protection.explosions.status` | Queries whether explosion protection is enabled or disabled for the player's side (party or personal claim). |
| `/claimrules explosions enable` | `opac.protection.explosions.toggle` | Enables explosion protection for the side. Claimed blocks will be shielded from explosion damage. |
| `/claimrules explosions disable` | `opac.protection.explosions.toggle` | Disables explosion protection for the side. Claimed blocks can be destroyed by explosions. |
| `/claimrules create status` | `opac.protection.create.status` | Queries whether foreign Create machines/drills and Aeronautics bores are allowed on the side's territory. |
| `/claimrules create enable` | `opac.protection.create.toggle` | Allows foreign Create machines, drills, saws, and Aeronautics bores to operate on the side's claims. |
| `/claimrules create disable` | `opac.protection.create.toggle` | Blocks foreign Create machines, drills, saws, and Aeronautics bores from affecting blocks on the side's claims. |
| `/claimrules assembly status` | `opac.protection.assembly.status` | Queries whether contraption assembly (Create / Sable) is allowed on the side's claims. |
| `/claimrules assembly enable` | `opac.protection.assembly.toggle` | Allows contraption assembly on the side's claims. |
| `/claimrules assembly disable` | `opac.protection.assembly.toggle` | Blocks contraption assembly across the side's claim borders. |

### Client GUI & Keybind

When a player connects with Better Protection installed on their client:
- The command `/claimrules` (or `/claimrules gui`) opens an interactive screen showing current side rules, cooldown timers, and toggle buttons.
- A default keybind `Open Claim Rules` (default key: `K`) is registered under the `CustomClaims` controls category.

## Permissions & Roles

Permissions are registered with NeoForge's `PermissionAPI` and integrate seamlessly with LuckPerms or other permission managers.

### Node List

| Permission Node | Default | Description |
| --- | --- | --- |
| `opac.protection.claimrules` | Granted to all players (`true`) | Access to `/claimrules` and the client GUI screen. |
| `opac.protection.explosions.status` | Granted to all players (`true`) | View side explosion protection status. |
| `opac.protection.explosions.toggle` | Granted to all players (`true`) | Toggle side explosion protection (`enable`/`disable`). |
| `opac.protection.create.status` | Granted to all players (`true`) | View side Create machine rule status. |
| `opac.protection.create.toggle` | Granted to all players (`true`) | Toggle side Create machine rule (`enable`/`disable`). |
| `opac.protection.assembly.status` | Granted to all players (`true`) | View side contraption assembly rule status. |
| `opac.protection.assembly.toggle` | Granted to all players (`true`) | Toggle side contraption assembly rule (`enable`/`disable`). |
| `opac.protection.limits.reset` | Requires OP level 2 (`false`) | Reset player interaction limits (`reset`, `resetall`). |
| `opac.protection.bypass` | Requires OP level 2 (`false`) | Bypass foreign interaction limits. |
| `opac.protection.admin` | Requires OP level 2 (`false`) | Protection administration: bypass rule toggle cooldowns and restrictions. |
| `opac.admin` | Requires OP level 2 (`false`) | Global administrator access across OPaC and Custom Claims modules. |
| `opac.bypass` | Requires OP level 2 (`false`) | Global bypass for all claim restrictions and limits. |

### Legacy Aliases

For backward compatibility with older configurations and permission files, all nodes have equivalent `customclaims.*` aliases automatically mapped:
- `customclaims.claimrules` -> `opac.protection.claimrules`
- `customclaims.explosions.status` -> `opac.protection.explosions.status`
- `customclaims.explosions.toggle` -> `opac.protection.explosions.toggle`
- `customclaims.create.status` -> `opac.protection.create.status`
- `customclaims.create.toggle` -> `opac.protection.create.toggle`
- `customclaims.assembly.status` -> `opac.protection.assembly.status`
- `customclaims.assembly.toggle` -> `opac.protection.assembly.toggle`
- `customclaims.limits.reset` -> `opac.protection.limits.reset`
- `customclaims.admin` -> `opac.admin`
- `customclaims.bypass` -> `opac.bypass`

### Default Player Permissions

Configured in `config/customclaims_core-common.toml` under `permissions.default_player_permissions`. Server operators who prefer to restrict rule toggling to party leaders or privileged ranks can remove the `.toggle` nodes from this list or manage them via LuckPerms groups.

## Foreign Interaction Limits

Foreign interaction limits control how many blocks an outsider (a player who is not the claim owner and not a member of the owning party) can break or place in peaceful claims during a runtime session.

### Configuration

In `config/customclaims_protection-common.toml`:

```toml
[foreign_interaction]
# Limit on blocks an outsider may break in peaceful claims per session (0 = none allowed)
block_break_limit = 0

# Limit on blocks an outsider may place in peaceful claims per session (0 = none allowed)
block_place_limit = 0

# Interval in seconds between automatic global resets of session counters (default: 3600 = 1 hour)
limit_reset_interval_seconds = 3600
```

### Operational Rules

1. **Default Strict Protection:** By default, both limits are set to `0`. This means foreign visitors cannot break or place any blocks inside protected peaceful claims.
2. **Session Quota:** If a server operator sets `block_break_limit = 5`, each non-member player can break up to 5 blocks in foreign claims per reset interval.
3. **Tracking & Reset:** Counters are maintained in server memory. They reset automatically every `limit_reset_interval_seconds` (default: 1 hour), on server restarts, or manually via `/claimrules limits reset <player>` and `/claimrules limits resetall`.
4. **Player Query:** Players can check their current quota consumption anytime using `/claimrules limits me`.

## Rule Toggle Cooldowns

To prevent combat exploitation (such as disabling explosion protection right before TNT impacts or quickly toggling machine rules back and forth), rule changes are rate-limited per side.

- **Config Key:** `claimrules.toggle_cooldown_seconds` in `customclaims_protection-common.toml` (default: `600` seconds / 10 minutes).
- **Scope:** Cooldowns apply per side (`ClaimSideId`) and per rule (`explosions`, `create`, `assembly`).
- **Persistence:** Cooldown expiration timestamps are saved in `world/customclaims/protection/claimrule-toggle-cooldowns.txt` so restarts do not clear active cooldowns.
- **Bypass:** The server console and players with `opac.protection.admin`, `opac.admin`, or operator status bypass all cooldowns.
- **Informational Commands:** Running status checks (`/claimrules explosions status`, etc.) and opening the GUI never consume cooldown.

## Soft Warfare Integration (`opac-warfare`)

When the optional companion mod **Open Parties and Claims: Warfare** (`opac-warfare`) is installed alongside Better Protection:

1. **Peaceful Claims:** Outside of wars, all standard Better Protection rules, explosion filters, and machine restrictions operate normally.
2. **Active Contested War Chunks:** Inside chunks where an active war or siege is declared:
   - **Explosions Allowed:** When `explosions.allow_in_war_chunks = true` (default), explosions cause normal block damage in the contested chunk, allowing attackers and defenders to breach fortifications.
   - **Cannon Fire:** CBC cannons may be fired from inside contested war chunks without being blocked by launch protection.
   - **Create & Aeronautics Bores:** Mechanical drills, saws, and Aeronautics borehead bearings are permitted to operate in contested war chunks for war participants.
   - **Grave & Corpse Recovery:** The config setting `entity_interaction.contested_exceptions` (default: `["corpse:corpse"]`) grants participants an immediate OPaC bypass to right-click corpses/graves in contested war chunks to reclaim their inventory without claim access violations.

## Entity, Villager, and Storage Mechanics in 2.0

In Better Protection 2.0:
- **Villagers & Withering:** Hardcoded Wither summoning bans and intrusive villager damage hooks were removed. Entity protection and mob griefing rules are handled natively by vanilla Minecraft and Open Parties and Claims claim settings.
- **Storage Access:** Storage protection is governed authoritatively by OPaC's native claim access lists and vanilla permissions, preventing duplicate or desynchronized lock checks.

## Troubleshooting & Operations

1. **Players cannot toggle rules:**
   - Check if the side is on cooldown. The error message indicates remaining seconds.
   - Verify the player has `opac.protection.<rule>.toggle` permission.
2. **Explosions are breaking claimed blocks:**
   - Check if `/claimrules explosions status` is `ENABLED` for that claim's side.
   - Check if `explosions.custom_filter_enabled = true` in `customclaims_protection-common.toml`.
   - Check if the chunk is currently in an active war (see `explosions.allow_in_war_chunks`).
3. **CBC Cannons are not firing:**
   - If cannons fail to fire with a red actionbar message, the cannon muzzle is inside a protected claim where explosion protection is enabled. Fire from unclaimed land or an active war chunk.
4. **Create drills stall on claim borders:**
   - This is intended behavior when `/claimrules create` is disabled for that claim. The owning side can enable foreign machines via `/claimrules create enable`.
