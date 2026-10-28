# Development

This repository uses Gradle, NeoForge userdev, Java `21`, and a `1.21.1`
compile baseline for **Open Parties and Claims: Better Protection**.

## Setup

Required locally:

- JDK `21`
- Git
- The Gradle wrapper included in this repository

The build resolves Minecraft/NeoForge and mod dependencies through configured
Maven repositories, including Modrinth Maven for Open Parties and Claims,
Create, Create Big Cannons, and related compat dependencies.
Aeronautics/Offroad compat uses the local bundled jar in `libs/` for
development runtime coverage.

This repository compiles against `Minecraft 1.21.1 + NeoForge 21.1.232`, while
metadata allows experimental same-jar probes on Minecraft `[1.21.1,26.0)` and
NeoForge `[21.1.232,26.0)`.

Important version properties live in `gradle.properties`:

```properties
minecraft_version=1.21.1
minecraft_version_range=[1.21.1,26.0)
neo_version=21.1.232
neo_version_range=[21.1.232,26.0)
opc_version=neoforge-1.21.1-0.27.5
create_version=mc1.21.1-6.0.9
cbc_version=5.11.7
rpl_version=2.1.2
mod_version=2.0.0
```

## CI-Parity Build

GitHub Actions runs:

```sh
./gradlew --no-daemon build :opac-better-protection:jar test
```

On Windows, use:

```powershell
.\gradlew.bat --no-daemon build :opac-better-protection:jar test
```

The uploaded release artifact comes from:

```text
opac-better-protection/build/libs/opac-better-protection-*.jar
```

## Release Checklist

Before release, run:

```powershell
.\gradlew.bat --no-daemon :opac-better-protection:jar test
```

Smoke-test the built jar on the baseline `Minecraft 1.21.1 + NeoForge 21.1.232` with OPaC:

- server boots without mixin, classloading, packet, or config errors;
- `/claimrules gui`, `/claimrules explosions status`, `/claimrules create status`,
  and `/claimrules assembly status` work;
- protection blocks foreign peaceful interactions when limits are reached;
- Create machine protection works when Create is installed;
- Aeronautics/Offroad bore protection works when Aeronautics/Offroad is installed;
- Create Big Cannons launch protection works when CBC is installed.

## Focused Commands

Compile all functional protection modules:

```powershell
.\gradlew.bat --no-daemon "-Dorg.gradle.workers.max=1" :customclaims-core:compileJava :customclaims-protection:compileJava :customclaims-create:compileJava :customclaims-aeronautics:compileJava :customclaims-big-cannons:compileJava -q
```

Build the distribution jar:

```powershell
.\gradlew.bat --no-daemon :opac-better-protection:jar
```

Run test suite:

```powershell
.\gradlew.bat --no-daemon test
```

If Gradle has just downloaded plugins or a worker daemon is stuck, stop daemons:

```powershell
.\gradlew.bat --stop
```

## Source Layout

Each functional module keeps normal Java sources under:

```text
<module>/src/main/java/dev/customclaims/<area>/
```

Resources live under:

```text
<module>/src/main/resources/
```

Generated resources are ignored by git through `**/src/generated/`.

## Working Rules

- Prefer existing services over adding logic directly to event handlers.
- Keep command classes responsible for parsing, permission checks, and messages;
  move behavior into services.
- Keep config keys stable unless a migration or release note is planned.
- Do not add hard dependencies from optional compat modules to absent target
  mods without a mod-load guard; Aeronautics/Offroad and Create hooks must remain optional.
- Keep client-only UI and keybind code behind client-side checks.
- Update `docs/configuration.md` whenever a config key, default, or meaning changes.
- Update `docs/server-admin.md` whenever a command, permission, runtime file, or operator-visible behavior changes.
- Update `docs/architecture.md` when module boundaries, service ownership, or persistence changes.
