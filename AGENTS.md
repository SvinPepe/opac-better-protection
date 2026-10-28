# Codex Working Notes

This file is the short operational memory for Codex and other coding agents working on **Open Parties and Claims: Better Protection**.
For full context, read `docs/architecture.md`, `docs/server-admin.md`, `docs/configuration.md`, and `docs/development.md`.

## Project Snapshot

- Gradle multi-module NeoForge project with Minecraft `1.21.1` compile baseline and Java `21`.
- Product name: **Open Parties and Claims: Better Protection**.
- Repository: `https://github.com/SvinPepe/opac-better-protection`.
- Companion mod: **Open Parties and Claims: Warfare** (`https://github.com/SvinPepe/opac-warfare`).
- OPaC is the authority for parties and claims; this mod adds extended claim rules,
  explosion protection, foreign interaction limits, `/claimrules` commands/GUI, and deep
  compat integrations for Create, Create Big Cannons, and Create Aeronautics/Offroad.
- Distribution server artifact:
  - `opac-better-protection/build/libs/opac-better-protection-*.jar`
  (Single jar bundling: Core via JarJar, Protection, Create, Aeronautics, and Big Cannons).
- Compatibility anchor: baseline `Minecraft 1.21.1 + NeoForge 21.1.232`;
  metadata allows experimental Minecraft `[1.21.1,26.0)` and NeoForge `[21.1.232,26.0)` probes.
  Minecraft 26.x is incompatible (ResourceLocation renamed, FMLEnvironment.dist removed, Java 25 required).

## Module Layout & Ownership

- `customclaims-core`: OPaC adapters (`OpenPartiesClaimAdapter`), territory services (`TerritoryService`, `TerritoryStateService`),
  permission service (NeoForge PermissionAPI + LuckPerms), storage (`DataStorageService`), action logs, message service,
  metrics, rollback interface, and warfare bridge interface (`IWarfareBridge`, `dev.customclaims.api.war.WarfareBridgeHolder`).
- `customclaims-protection`: Claim rules (`ClaimRulesService`), direct OPaC API explosion filtering (`ExplosionProtectionService`),
  foreign interaction rate limiting (`ForeignInteractionLimitService`), contested entity interaction bypasses (`EntityInteractionProtectionService`),
  network payloads, `/claimrules` commands and client GUI screen.
- `customclaims-create`: Optional Create hooks for contraption movement, deployers, and block-breaking protection (`BlockBreakingMovementBehaviourMixin`, `ContraptionAssemblyMixin`, `CreateContraptionProtectionHooks`).
- `customclaims-aeronautics`: Optional Aeronautics/Offroad hooks for Borehead Bearing & Rock Cutting Wheel protection (`AeronauticsBoreProtectionHooks`), and Sable assembly protection (`SableAssemblyProtectionHooks`).
- `customclaims-big-cannons`: Optional Create Big Cannons hooks for cannon projectile launch cancellation from protected claims (`BigCannonLaunchProtectionHooks`, `MountedBigCannonContraptionMixin`) and impact logging.
- `opac-better-protection`: Bundles `customclaims-protection`, `customclaims-create`, `customclaims-aeronautics`, and `customclaims-big-cannons`, while packaging `customclaims-core` into `META-INF/jarjar/`.

## Soft Warfare Bridge Rule

- **Decoupled Architecture:** Better Protection and Warfare are completely independent projects.
- **No Compile Dependency:** `customclaims-protection` (and all other Better Protection modules) MUST NOT depend on `opac-warfare` or any Warfare classes at compile time.
- **Dynamic Bridge Mechanism:**
  - `customclaims-core` defines `dev.customclaims.api.war.IWarfareBridge` and `dev.customclaims.api.war.WarfareBridgeHolder`.
  - `customclaims-protection/compat/WarfareBridgeHolder.java` dynamically checks for Warfare via reflection (`isContestedWarChunk(level, chunkPos)`).
  - If `opac-warfare` is present, Warfare registers its contested chunk lookup handler (`WarfareBridgeHolder.register(this::isContestedWarChunk)`).
  - If `opac-warfare` is absent, the bridge gracefully falls back to returning `false` without throwing `ClassNotFoundException` or failing startup.
- **War Chunk Behavior:** In active contested war chunks, explosion filtering is skipped (if `explosions.allow_in_war_chunks = true`), Create drills/contraptions and CBC cannons are allowed for war participants, and contested entity exceptions (like `corpse:corpse`) bypass territory locks.

## Permissions & Commands

Permission tree root: `opac.protection.*`

| Node | Default | Description |
| --- | --- | --- |
| `opac.protection.claimrules` | `true` | Allows opening the `/claimrules` GUI |
| `opac.protection.explosions.status` | `true` | Check explosion protection status |
| `opac.protection.explosions.toggle` | `true` | Toggle explosion protection for player's side |
| `opac.protection.create.status` | `true` | Check Create machine protection status |
| `opac.protection.create.toggle` | `true` | Toggle Create machine rule for player's side |
| `opac.protection.assembly.status` | `true` | Check contraption assembly rule status |
| `opac.protection.assembly.toggle` | `true` | Toggle contraption assembly rule for player's side |
| `opac.protection.limits.reset` | `false` | Reset foreign interaction limits (`/claimrules limits reset`) |
| `opac.protection.bypass` | `false` | Bypass foreign interaction limits |
| `opac.protection.admin` | `false` | Full protection administration and cooldown bypass |
| `opac.admin` | `false` | Global OPaC / Custom Claims administration |
| `opac.bypass` | `false` | Global OPaC / Custom Claims bypass |

Legacy `customclaims.*` aliases are registered and mapped automatically for backward compatibility.

`/claimrules` command structure:
- `/claimrules` / `/claimrules gui`
- `/claimrules limits me`
- `/claimrules limits reset <player>`
- `/claimrules limits resetall`
- `/claimrules explosions [status|enable|disable]`
- `/claimrules create [status|enable|disable]`
- `/claimrules assembly [status|enable|disable]`

## Safe Checks

CI parity (full jar build and tests):

```powershell
.\gradlew.bat --no-daemon :opac-better-protection:jar test
```

Focused fast compile:

```powershell
.\gradlew.bat --no-daemon "-Dorg.gradle.workers.max=1" :customclaims-core:compileJava :customclaims-protection:compileJava :customclaims-create:compileJava :customclaims-aeronautics:compileJava :customclaims-big-cannons:compileJava -q
```

Note: If running from an external terminal or subagent session, ensure the working directory is set to `c:\Users\svinpepe\opac-better-protection`.

## Do Not Break

- OPaC remains the authority for party membership and claim ownership.
- Contested chunks compatibility must remain strictly soft/dynamic via `WarfareBridgeHolder`.
- Optional Create, Aeronautics/Offroad, Sable, and CBC integrations must remain safe when target mods are absent (`ModList.get().isLoaded(...)`).
- Client-only code (GUI, key mappings, client packets) must stay behind `FMLEnvironment.dist == Dist.CLIENT` guards.
- Runtime data must stay under `world/customclaims/`.
- Config keys in `ProtectionConfig.java` and `CoreConfig.java` must stay strictly synchronized with documentation.

## Docs Maintenance

- Update `README.md` when entrypoints, public links, or core capabilities change.
- Update `docs/server-admin.md` for operator-visible commands, permissions, or mechanics.
- Update `docs/configuration.md` for config key, type, or default changes. Keep the table exact.
- Update `docs/architecture.md` for module, service, bridge, or persistence changes.
- Update `docs/development.md` for build, test, or workflow changes.
- Update `docs/compatibility.md` for Minecraft, NeoForge, or mod version matrix adjustments.
