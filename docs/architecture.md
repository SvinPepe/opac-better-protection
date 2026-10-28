# Architecture

This repository is a Gradle multi-module NeoForge project with a Minecraft
`1.21.1` compile baseline and experimental metadata for Minecraft
`[1.21.1,26.0)` on NeoForge `[21.1.232,26.0)`. Open Parties and
Claims (OPaC) is the source of party membership and claim
ownership. This project adds advanced protection rules, interaction limits,
explosions filtering, and mod compatibility.

## Modules

| Gradle project | Mod id | Role |
| --- | --- | --- |
| `customclaims-core` | `customclaims_core` | Shared OPaC adapters, territory services, permissions, storage, messages, logs, and rollback interface. |
| `customclaims-protection` | `customclaims_protection` | Claim rules, foreign interaction limits, explosions filtering, opt-in Wither and villager/trader protection, GUI payloads, and `/claimrules`. |
| `customclaims-create` | `customclaims_create` | Optional Create contraption movement and block-breaking hooks. |
| `customclaims-aeronautics` | `customclaims_aeronautics` | Optional Aeronautics/Offroad bore-mining and Sable assembly hooks. |
| `customclaims-big-cannons` | `customclaims_big_cannons` | Optional Create Big Cannons launch and terrain-damage hooks. |
| `opac-better-protection` | multiple mod ids | Single-jar distribution that bundles core, protection, and compat modules. |

## Dependency Shape

Project dependencies:

```text
customclaims-core
  <- customclaims-protection
       <- customclaims-create
       <- customclaims-aeronautics
       <- customclaims-big-cannons
opac-better-protection bundles all modules
```

External dependencies:

- `customclaims-core` requires Open Parties and Claims.
- `customclaims-create` has optional Create integration guarded by
  `ModList.get().isLoaded("create")`.
- `customclaims-aeronautics` has optional Aeronautics/Offroad integration guarded
  by `ModList.get().isLoaded("offroad")` and optional mixins.
- `customclaims-big-cannons` has optional Create Big Cannons integration guarded
  by `ModList.get().isLoaded("createbigcannons")`.
- Optional warfare integration uses a soft reflection bridge (`WarfareBridgeHolder`)
  to check whether a chunk is in an active war without requiring `opac-warfare` at compile or run time.

## Service Records

The code uses small service containers created by module entrypoints:

- `CoreServices` creates OPaC adapters, party/claim/territory services,
  permission service, data storage, logs, message service, anonymous dedicated-server
  metrics, and rollback service.
- `ProtectionServices` builds foreign interaction limits, OPaC bypass service,
  contested entity interaction exceptions, explosion protection, Create mining/assembly rules,
  claim-rule cooldowns, `/claimrules` orchestration, opt-in Wither rules, and opt-in
  villager/trader protection.

When adding behavior, prefer placing domain logic in a service and keeping event
handlers and command classes thin.

## Entrypoints And Events

Core:

- `CustomClaimsCoreMod` registers `CoreConfig` and initializes
  `CoreServices`. Its metrics service listens for server start, tick, and stop
  events; HTTP delivery runs asynchronously off the server thread.

Protection:

- `CustomClaimsProtectionMod` registers `ProtectionConfig`, claim-rule payloads,
  and client-only GUI/keybind registration when running on the client.
- Registers `/claimrules`.
- Handles block interaction, break/place, explosions, opt-in Wither spawn
  blocking, opt-in villager/trader damage protection, and foreign interaction
  reset ticks.
- Handles server-side `EntityInteract` and `EntityInteractSpecific` at highest
  priority. `EntityInteractionProtectionService` checks configured entity type
  exceptions and player interaction status.

Compat:

- `CustomClaimsCreateMod` activates Create hooks only when Create is loaded.
- `CustomClaimsAeronauticsMod` activates Offroad bore-mining hooks and optional Sable assembly checks only when their target mods are loaded.
- `CustomClaimsBigCannonsMod` activates CBC hooks only when Create Big Cannons
  is loaded.

## Persistence

All project-owned runtime files are resolved under:

```text
world/customclaims/
```

Current file names:

- `protection/explosion-protection.txt`: side explosion rule state.
- `protection/create-machines.txt`: side Create/Offroad mining rule state.
- `protection/create-assemblies.txt`: side Create/Sable assembly rule state.
- `protection/claimrule-toggle-cooldowns.txt`: side toggle cooldown timestamps.
- `logs/actions.log`: general action log.
- `metrics/server-id.txt`: randomly generated anonymous dedicated-server id.

Foreign interaction counters are runtime-only and are not persisted.

## Important Invariants

- OPaC remains the authority for party membership and claim ownership.
- Claim rule changes and cooldowns are tracked per side (`ClaimSideId`).
- Optional compat code must stay safe when the target mod is not installed, including Aeronautics/Offroad nested modules.
- Client-only registrations must stay behind `FMLEnvironment.dist == Dist.CLIENT`
  checks.
