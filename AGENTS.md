# Codex Working Notes

This file is the short operational memory for Codex and other coding agents.
For full context, read `docs/architecture.md` and `docs/development.md`.

## Project Snapshot

- Gradle multi-module NeoForge project with Minecraft `1.21.1` compile baseline and Java `21`.
- Product name: **Open Parties and Claims: Better Protection**.
- OPaC is the authority for parties and claims; this mod adds extended claim rules,
  explosion protection, storage protection, Withers, villagers, foreign interactions,
  and Create / Aeronautics / Big Cannons compat integrations.
- Server artifact:
  - `opac-better-protection/build/libs/opac-better-protection-*.jar` (Better Protection: Core, Protection, Create, Aeronautics, Big Cannons)
- Compatibility anchor: baseline `Minecraft 1.21.1 + NeoForge 21.1.232`;
  metadata allows experimental Minecraft `[1.21.1,26.0)` and NeoForge `[21.1.232,26.0)` probes.
  Do not promise failed candidates, Forge, Fabric, or Quilt from this artifact.
  Minecraft 26.x is maintained on a separate port branch.

## Read First

Before changing behavior, inspect the relevant files:

- Modules and metadata: `settings.gradle`, `build.gradle`, `gradle.properties`.
- Core services: `customclaims-core/src/main/java/dev/customclaims/core/CoreServices.java`.
- Protection services: `customclaims-protection/src/main/java/dev/customclaims/protection/ProtectionServices.java`.
- Commands: `ClaimRulesCommand.java`.
- Configs: `CoreConfig.java`, `ProtectionConfig.java`.

## Safe Checks

CI parity:

```powershell
.\gradlew.bat --no-daemon :opac-better-protection:jar test
```

Focused compile:

```powershell
.\gradlew.bat --no-daemon "-Dorg.gradle.workers.max=1" :customclaims-core:compileJava :customclaims-protection:compileJava :customclaims-create:compileJava :customclaims-aeronautics:compileJava :customclaims-big-cannons:compileJava -q
```

Docs-only changes do not require a full Gradle build, but command/config docs
must be checked against Java source.

## Module Ownership

- `customclaims-core`: OPaC adapters, side identity, territory status,
  permissions (NeoForge PermissionAPI + LuckPerms), storage, logs, common messages, rollback interface, `IWarfareBridge`.
- `customclaims-protection`: claim rules, direct OPAC API explosion protection,
  foreign interaction limits, claim-rule GUI payloads, `/claimrules`.
- `customclaims-create`: optional Create hooks.
- `customclaims-aeronautics`: optional Aeronautics/Offroad bore mining hooks.
- `customclaims-big-cannons`: optional Create Big Cannons hooks.
- `opac-better-protection`: bundles `customclaims-core`, `customclaims-protection`, `customclaims-create`, `customclaims-aeronautics`, `customclaims-big-cannons`.

## Do Not Break

- OPaC remains the source of truth for party membership and claim ownership.
- Contested chunks compatibility is supported via dynamic bridge (`WarfareBridgeHolder`) if Warfare is present.
- Optional Create, Aeronautics/Offroad, and CBC integrations must stay safe when
  target mods are not loaded.
- Do not add multi-version Gradle targets or mark
  experimental candidate versions as verified without the compatibility
  probe in `docs/compatibility.md`.
- Client-only code must stay behind client-side checks.
- Runtime data stays under `world/customclaims/`.

## Docs Maintenance

- Update `README.md` when the public entrypoint or install story changes.
- Update `docs/server-admin.md` for operator-visible behavior changes.
- Update `docs/configuration.md` for config key/default changes. Keep the
  defaults table exact; do not leave stale values after code changes.
- Update `docs/architecture.md` for module, service, persistence, or compat
  boundary changes.
- Update `docs/development.md` for build, CI, or workflow changes.
