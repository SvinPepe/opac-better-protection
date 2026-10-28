# Open Parties and Claims: Better Protection

[![GitHub](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/SvinPepe/opac-better-protection)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**Open Parties and Claims: Better Protection** is an unofficial NeoForge addon for
[Open Parties and Claims](https://modrinth.com/mod/open-parties-and-claims) (OPaC).
Source code repository: [https://github.com/SvinPepe/opac-better-protection](https://github.com/SvinPepe/opac-better-protection).

It provides advanced claim protection mechanics, configurable side-level rules for explosions and technical mods, rate-limiting on foreign player interactions, and deep integrations for Create, Create Big Cannons, and Create Aeronautics/Offroad. Open Parties and Claims remains the authoritative source for party membership and claim ownership; this project builds advanced protection layers on top of OPaC territories.

## Companion Mod: Open Parties and Claims: Warfare

[Open Parties and Claims: Warfare](https://github.com/SvinPepe/opac-warfare) is the official companion warfare addon.

- **Autonomous Operation:** Both mods function 100% independently. Better Protection does not require Warfare to run, and Warfare does not require Better Protection.
- **Dynamic Soft Bridge:** When both mods are installed on the same server, a zero-dependency reflection bridge (`IWarfareBridge`) activates automatically.
- **War Chunk Overrides:** Inside actively contested war chunks, protection restrictions are dynamically lifted for war participants:
  - **Explosions:** Block damage filtering is disabled (`explosions.allow_in_war_chunks = true`), permitting military breaches.
  - **Create Big Cannons:** Cannon projectile launches are permitted from contested war territory.
  - **Create Drills & Saws:** Breaker contraptions and drills automatically operate in contested war chunks for participants.
  - **Aeronautics / Offroad Bores:** Borehead bearings and rock cutting wheels are permitted in contested territory.
  - **Grave & Entity Access:** Contested entity interaction exceptions (such as `corpse:corpse`) allow war participants to retrieve their items without territory protection interference.

## Key Features

- **Explosion Protection:**
  - Independent side-level toggles (`/claimrules explosions enable/disable`) with direct OPaC claim synchronization.
  - Protects claims against TNT, creepers, fireballs, and modded explosions.
  - Controlled by configurable cooldowns to prevent combat abuse.
- **Create Contraption & Machine Rules:**
  - Granular control over foreign mechanical drills, saws, deployers, and shared `BlockBreakingMovementBehaviour` machines.
  - Toggled per side via `/claimrules create enable/disable`.
  - Claim owners and side members can always freely operate their machines within their own territory.
- **Create Big Cannons (CBC) Protection:**
  - Prevents firing CBC cannons from within protected claimed territory (`big_cannons.block_projectile_launch_from_protected_claims`), stopping safe-zone artillery bombardment.
  - Actionbar warnings notifying the gunner and nearby players when a shot is cancelled.
  - Dedicated impact and launch logging (`big_cannons.log_blocked_projectiles`).
- **Aeronautics / Offroad Bore Protection:**
  - Detects Borehead Bearings and Rock Cutting Wheels from Create Aeronautics/Offroad.
  - Automatically respects the side's `/claimrules create` setting, safely stalling drills that attempt to bore into protected claims without permission.
- **Contraption Assembly Protection (Create & Sable):**
  - Independent toggle `/claimrules assembly enable/disable` protecting against unauthorized contraption assembly across claim borders.
  - Validates all 8 corners of the contraption's bounding box: once an assembly touches a claim, all corners must belong to peaceful claims of the assembling side.
- **Foreign Interaction Rate Limiting:**
  - Limits foreign (non-member) block breaks and placements in peaceful claims (`foreign_interaction.block_break_limit`, `foreign_interaction.block_place_limit`).
  - Runtime session counters automatically reset on a configurable interval or via operator commands.
- **Entity, Villager & Wither Protection:**
  - Integrates natively with Open Parties and Claims to ensure villagers, passive mobs, and containers inside claimed land are fully protected from outsider harm and hostile destruction.
  - Mob griefing and Wither destruction inside peaceful claims are governed cleanly through OPaC's authoritative claim flags and vanilla mechanics, eliminating duplicate conflicting event cancellations.
  - Contested entity interaction bypasses (e.g. `corpse:corpse`) ensure war participants can recover items and access player corpses during active sieges without claim lockouts.
- **Claim Rules Management (/claimrules & GUI):**
  - Intuitive `/claimrules` command suite for inspecting status, toggling rules, and managing limits.
  - Optional client GUI accessible via `/claimrules gui` or the `Open Claim Rules` keybind (defaults to `K` in the `CustomClaims` category).
  - Anti-spam toggle cooldowns per side.

## Requirements

- Minecraft `1.21.1` baseline; metadata allows experimental `[1.21.1,26.0)` probes (supports 1.21.x through 1.21.11)
- NeoForge `21.1.232` baseline; metadata allows experimental `[21.1.232,26.0)` probes
- Java `21`
- Open Parties and Claims `neoforge-1.21.1-0.27.5` or newer on the baseline stack

Optional integrations activate only when their target mods are installed:
- Open Parties and Claims: Warfare `2.0.0` or newer (companion warfare mod)
- Create `mc1.21.1-6.0.9` or newer
- Create Big Cannons `5.11.7` or newer
- Create Aeronautics / Offroad `1.3.0` or newer for bore protection
- Sable `2.0.3` for contraption assembly protection

## What To Install

Server owners install the single distribution jar built by the `opac-better-protection` module:

```text
opac-better-protection/build/libs/opac-better-protection-<version>.jar
```

This single distribution jar bundles:
- `customclaims_core` (embedded via JarJar)
- `customclaims_protection`
- `customclaims_create`
- `customclaims_aeronautics`
- `customclaims_big_cannons`

## Building from Source

To build the release jar and run tests:

```powershell
.\gradlew.bat --no-daemon :opac-better-protection:jar test
```

On Linux / macOS:

```sh
./gradlew --no-daemon :opac-better-protection:jar test
```

Copy the resulting `opac-better-protection-*.jar` into the server `mods/` folder together with Open Parties and Claims.

## Commands

```text
/claimrules                              # Opens the Claim Rules GUI (player only)
/claimrules gui                          # Opens the Claim Rules GUI (player only)
/claimrules limits me                    # Inspects personal foreign interaction limit counters
/claimrules limits reset <player>        # Resets foreign interaction limits for a player (admin)
/claimrules limits resetall              # Resets foreign interaction limits for all players (admin)
/claimrules explosions status            # Checks explosion protection status for current side
/claimrules explosions enable            # Enables explosion protection for current side
/claimrules explosions disable           # Disables explosion protection for current side
/claimrules create status                # Checks Create machine protection status
/claimrules create enable                # Allows foreign Create machines on territory
/claimrules create disable               # Blocks foreign Create machines on territory
/claimrules assembly status              # Checks contraption assembly protection status
/claimrules assembly enable              # Allows contraption assembly on territory
/claimrules assembly disable             # Blocks contraption assembly on territory
```

## Documentation

- [Server Admin Guide](docs/server-admin.md)
- [Compatibility Policy](docs/compatibility.md)
- [Configuration](docs/configuration.md)
- [Architecture](docs/architecture.md)
- [Development Guide](docs/development.md)
- [AGENTS.md](AGENTS.md)

## License

MIT. See [LICENSE](LICENSE).
