# Architecture

This document describes the architectural design of **Open Parties and Claims: Better Protection**, its component modules, service containers, persistence structure, and integrations with Open Parties and Claims (OPaC), Create, Create Big Cannons, Create Aeronautics/Offroad, and the companion mod Open Parties and Claims: Warfare.

## Overview

The project uses Minecraft `1.21.1` and NeoForge `21.1.232` as its compile baseline, with experimental metadata for Minecraft `[1.21.1,26.0)` on NeoForge `[21.1.232,26.0)`.

Open Parties and Claims (OPaC) is the single authority for party memberships and land claim boundaries. Better Protection acts as an advanced territorial defense and mechanical moderation layer, enforcing side-level toggles, explosion filtering, machine rules, and foreign interaction limits on top of OPaC territories.

## Modules

| Gradle project | Mod id | Distribution Role |
| --- | --- | --- |
| `customclaims-core` | `customclaims_core` | Shared foundation: OPaC claim/party adapters, territory status, permissions, storage, action logging, anonymous metrics, and `IWarfareBridge` contract. |
| `customclaims-protection` | `customclaims_protection` | Core protection rules: explosion filtering, `/claimrules` commands/GUI, foreign interaction rate limiting, cooldowns, contested entity bypasses, and soft warfare bridge lookup. |
| `customclaims-create` | `customclaims_create` | Optional Create integration: contraption movement behaviour, mechanical drills/saws, and deployer claim boundary protection. |
| `customclaims-aeronautics` | `customclaims_aeronautics` | Optional Aeronautics/Offroad integration: Borehead Bearing & Rock Cutting Wheel bore stalling, plus Sable assembly checks. |
| `customclaims-big-cannons` | `customclaims_big_cannons` | Optional Create Big Cannons (CBC) integration: cancels cannon launches inside protected claims and logs impact damage. |
| `opac-better-protection` | multiple mod ids | Production single-jar bundle: embeds `customclaims-core` via JarJar and packages protection + all compat modules. |

## Dependency Structure

```text
customclaims-core (base)
  ▲
  │ (implements adapters, IWarfareBridge, permissions)
  │
customclaims-protection
  ▲
  ├──────────────────────────┬──────────────────────────┐
  │                          │                          │
customclaims-create    customclaims-aeronautics   customclaims-big-cannons
  │                          │                          │
  └──────────────────────────┼──────────────────────────┘
                             │
                  opac-better-protection
            (single-jar distribution bundle)
```

## The Soft Warfare Bridge (`IWarfareBridge`)

Better Protection and its companion mod, **Open Parties and Claims: Warfare**, are completely decoupled. Neither repository depends on the other at compile time.

Instead, they communicate through a dynamic, zero-dependency soft bridge pattern:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                          customclaims-core                             │
│                                                                        │
│   public interface IWarfareBridge {                                    │
│       boolean isContestedWarChunk(ServerLevel level, ChunkPos pos);    │
│   }                                                                    │
│   public final class WarfareBridgeHolder {                             │
│       private static volatile IWarfareBridge bridge = (l, c) -> false;│
│       public static void register(IWarfareBridge newBridge) { ... }    │
│       public static boolean isContestedWarChunk(...) { ... }           │
│   }                                                                    │
└────────────────────────────────────────────────────────────────────────┘
                    ▲                                  ▲
                    │ (queries via reflection)         │ (registers handler)
                    │                                  │
┌───────────────────┴───────────────┐     ┌────────────┴─────────────────┐
│     customclaims-protection       │     │        opac-warfare          │
│                                   │     │   (companion mod, optional)  │
│ WarfareBridgeHolder (compat)      │     │                              │
│ - checks if Warfare is loaded     │     │ WarManager.java:             │
│ - falls back to false if missing  │     │ WarfareBridgeHolder.register(│
│ - invokes isContestedWarChunk     │     │     this::isContestedWarChunk│
└───────────────────────────────────┘     │ );                           │
                    │                     └──────────────────────────────┘
                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                   Contested War Chunk Protections                      │
│                                                                        │
│  - Explosions: bypass filtering if explosions.allow_in_war_chunks=true │
│  - CBC Cannons: allow firing from contested war chunks                 │
│  - Create Drills: allow block breaking for war participants           │
│  - Aeronautics Bores: allow bore mining in war territory              │
│  - Entity Interactions: grant OPaC bypass for contested exceptions    │
│    (e.g. corpse:corpse for recovering player graves during siege)     │
└────────────────────────────────────────────────────────────────────────┘
```

### Bridge Execution Details

1. If `opac-warfare` is installed:
   When `WarManager` starts up, it registers its active siege query: `WarfareBridgeHolder.register(this::isContestedWarChunk)`.
   When an explosion, cannon launch, or Create machine operates, Better Protection queries the bridge. If the chunk is under active siege, normal peaceful claim protections are relaxed to allow military actions.
2. If `opac-warfare` is absent:
   The reflection lookup gracefully detects that Warfare is not present or uses the default lambda `(level, chunkPos) -> false`. No `ClassNotFoundException` or `NoClassDefFoundError` is thrown, and all chunks are governed by standard peaceful claim rules.

## Service Architecture

Modules use small, focused service containers injected at startup:

### `CoreServices` (`customclaims-core`)

- `ClaimService`: Interacts with OPaC claim registries.
- `PartyService`: Resolves player party affiliations, roles, and membership.
- `TerritoryService`: Maps chunk coordinates and worlds to `TerritoryStatus` (`UNCLAIMED`, `PEACEFUL_CLAIMED`, `WAR_CONTESTED`, `FOREIGN_LIMITED_INTERACTION`).
- `TerritoryStateService`: Manages runtime overrides (temporary contested states, snapshots).
- `PermissionService`: Centralized permission checking integrating NeoForge PermissionAPI and LuckPerms.
- `DataStorageService`: Handles disk I/O under `world/customclaims/`.
- `MetricsService`: Delivers anonymous heartbeat metrics on dedicated servers.
- `RollbackService`: Interface for rollback operations.
- `ActionLogService`: Logs significant territorial events.

### `ProtectionServices` (`customclaims-protection`)

- `ExplosionProtectionService`: Intercepts NeoForge `ExplosionEvent.Detonate`, determines affected claims, and removes protected blocks from destruction lists while keeping OPaC state synchronized.
- `CreateMachinesProtectionService`: Evaluates whether mechanical contraptions, drills, saws, or assemblies are permitted at target coordinates based on claim ownership and `/claimrules`.
- `ClaimRulesCooldownService`: Enforces configurable toggle cooldowns (`claimrules.toggle_cooldown_seconds`) per side to stop combat toggling.
- `ClaimRulesService`: Business logic for toggling and querying rules (`explosions`, `create`, `assembly`) by side ID.
- `ForeignInteractionLimitService`: Tracks runtime session counts of foreign block breaks and placements in peaceful claims.
- `OpenPartiesProtectionBypassService`: Temporarily relaxes OPaC's default block/entity cancel hooks when an allowed foreign interaction or contested entity interaction occurs.
- `EntityInteractionProtectionService`: Verifies whether an entity right-click event matches `entity_interaction.contested_exceptions` (e.g. `corpse:corpse`) within contested chunks.

## Compat Integrations Breakdown

### 1. Open Parties and Claims (OPaC)
- Core adapter: `OpenPartiesClaimAdapter.java`.
- Reads chunk ownership from `ClaimsManager.INSTANCE`.
- Respects party memberships from `PartyManager.INSTANCE`.
- Coordinates claim-level explosion permissions directly with OPaC's internal claim flags.

### 2. Create
- `BlockBreakingMovementBehaviourMixin`: Hooks `tick` to check `CreateContraptionProtectionHooks.shouldBlockBreaker`. If blocked, the block destruction progress is cancelled and progress data is cleared.
- `ContraptionAssemblyMixin`: Hooks assembly start to ensure contraption blocks do not violate claim protection rules.
- Deployers and mechanical arms: Verified via `BlockInteractionHandler` and `CreateMachinesProtectionService`.

### 3. Create Big Cannons (CBC)
- `MountedBigCannonContraptionMixin`: Intercepts cannon firing sequences.
- `BigCannonLaunchProtectionHooks`: Calculates global muzzle position from cannon contraption state. If the muzzle is inside a protected claim where explosions are blocked, the launch is aborted before propellant/ammunition is consumed, an actionbar error is displayed to the gunner, and an audit message is logged.

### 4. Create Aeronautics / Offroad
- `MultiMiningServerManagerMixin` & `MultiMiningBlockBreakingDataMixin`: Intercepts multi-block bore mining operations.
- `AeronauticsBoreProtectionHooks`: Detects Borehead Bearings and Rock Cutting Wheels. If the target block is protected under `/claimrules create`, the bearing's `setStalled(true)` is invoked, preventing bore damage into claimed land.

### 5. Sable
- `SableAssemblyMixin` & `SableAssemblyProtectionHooks`: Evaluates the complete bounding box of the proposed contraption. If any part of the structure touches a claim, all 8 corners must reside in peaceful claims of the same side with `/claimrules assembly` enabled.

## Persistence Layout

All runtime files are placed in the server's world directory under `world/customclaims/`:

```text
world/customclaims/
├── protection/
│   ├── explosion-protection.txt         # Side toggles for explosion protection (sideId=boolean)
│   ├── create-machines.txt              # Side toggles for foreign Create machines (sideId=boolean)
│   ├── create-assemblies.txt            # Side toggles for contraption assembly (sideId=boolean)
│   └── claimrule-toggle-cooldowns.txt   # Side cooldown expiration timestamps (sideId:rule=epochSecond)
├── logs/
│   └── actions.log                      # Action audit log
└── metrics/
    └── server-id.txt                    # Anonymous persistent server UUID
```

Note: Foreign interaction break/place counters are strictly in-memory runtime records and reset upon server restart, scheduled interval tick, or administrative command.

## Invariants

1. **Authority:** OPaC is the sole authoritative registry for claims and parties. Better Protection never creates ghost claims or mutates ownership records without OPaC.
2. **Decoupled Warfare:** No hard compile-time or runtime dependency on `opac-warfare`. All war chunk checks must pass safely through `WarfareBridgeHolder`.
3. **Graceful Degradation:** If Create, CBC, Aeronautics, or Sable are not installed, their respective mixins and hooks remain dormant without throwing errors.
4. **Client-Side Safety:** All client packets, screen classes, and key registrations must remain guarded behind `FMLEnvironment.dist == Dist.CLIENT`.
