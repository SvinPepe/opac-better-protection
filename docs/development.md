# Development Guide

This repository contains the source code for **Open Parties and Claims: Better Protection**, an advanced claim moderation and technical mod integration addon for NeoForge.

---

## Prerequisites

- **Java Development Kit (JDK):** Version `21` (Eclipse Temurin, Microsoft, or Oracle JDK 21).
- **Git**
- **Gradle:** Use the included wrapper (`gradlew` / `gradlew.bat`).

---

## Project Structure & Architecture

The project is structured as a multi-module Gradle project:

```text
opac-better-protection/
├── customclaims-core/            # Foundational OPaC adapters, territory, permissions, storage, IWarfareBridge
├── customclaims-protection/      # Protection services, explosions, /claimrules, GUI, foreign limits
├── customclaims-create/          # Create contraption and breaker movement hooks
├── customclaims-aeronautics/     # Aeronautics/Offroad borehead and Sable assembly hooks
├── customclaims-big-cannons/     # Create Big Cannons launch hooks and impact logging
└── opac-better-protection/       # Release distribution module (bundles and jarjars dependencies)
```

### Packaging Mechanism (JarJar & Fat-Jar)

In NeoForge / Java 21:
- Modules sharing package namespaces (`dev.customclaims.*`) cannot be placed loosely on the module path due to JPMS split-package restrictions.
- The `opac-better-protection` module solves this by embedding `customclaims-core` into `META-INF/jarjar/` and merging class outputs of the other submodules into the single distribution jar:
  ```text
  opac-better-protection/build/libs/opac-better-protection-<version>.jar
  ```

---

## Build & Test Commands

All commands should be executed from the repository root (`c:\Users\svinpepe\opac-better-protection`).

### 1. Full Build & Test (CI Parity)

Executes the full compilation, test suite, and packages the production distribution jar:

```powershell
.\gradlew.bat --no-daemon :opac-better-protection:jar test
```

On Linux or macOS:

```sh
./gradlew --no-daemon :opac-better-protection:jar test
```

### 2. Fast Focused Compilation

Quickly compiles Java sources across all functional modules without packaging or running long tasks:

```powershell
.\gradlew.bat --no-daemon "-Dorg.gradle.workers.max=1" :customclaims-core:compileJava :customclaims-protection:compileJava :customclaims-create:compileJava :customclaims-aeronautics:compileJava :customclaims-big-cannons:compileJava -q
```

### 3. Build Distribution Jar Only

```powershell
.\gradlew.bat --no-daemon :opac-better-protection:jar
```

The resulting jar will be located at:
`opac-better-protection/build/libs/opac-better-protection-<version>.jar`

### 4. Run Unit Tests Only

```powershell
.\gradlew.bat --no-daemon test
```

### 5. Stopping Gradle Daemons

If locks or worker daemons conflict after running tasks:

```powershell
.\gradlew.bat --stop
```

---

## Running Development Environments

NeoForge userdev provides pre-configured run configurations:

### Dedicated Server Run

```powershell
.\gradlew.bat :opac-better-protection:runServer
```

Launches a headless NeoForge dedicated server in the `run/server/` working directory.

### Client Run

```powershell
.\gradlew.bat :opac-better-protection:runClient
```

Launches the Minecraft client in `run/client/` with debug logging enabled.

---

## Soft Bridge Development Rules

1. **Decoupled Warfare:** Never add a Gradle dependency on `opac-warfare` or import Warfare classes directly into `customclaims-protection`. Use `IWarfareBridge` and `dev.customclaims.protection.compat.WarfareBridgeHolder`.
2. **Mod-Loaded Guards:** Any code interacting with Create, Create Big Cannons, or Aeronautics must be guarded by `ModList.get().isLoaded("<modid>")` checks and separated into compat classes to prevent classloader errors when the target mod is absent.
3. **Client Isolation:** UI code (such as `ClaimRulesScreen`), key mappings, and client packet receivers must be isolated behind `FMLEnvironment.dist == Dist.CLIENT` checks.
4. **Service Container Pattern:** Keep event listeners and command classes thin. Place domain logic in service classes (`ExplosionProtectionService`, `CreateMachinesProtectionService`, etc.).

---

## Pre-Commit Checklist

Before submitting changes:
1. Run `.\gradlew.bat --no-daemon :opac-better-protection:jar test` to verify build and test success.
2. If adding or modifying config keys, update `ProtectionConfig.java` / `CoreConfig.java` AND `docs/configuration.md`.
3. If modifying commands or permissions, update `docs/server-admin.md` and `AGENTS.md`.
4. Ensure all documentation links are valid relative markdown links.
