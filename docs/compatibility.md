# Compatibility Policy

This document defines the supported platforms, dependencies, mod compatibility matrix, and verification procedures for **Open Parties and Claims: Better Protection**.

## Official Distribution Artifact

This project publishes a single distribution jar for NeoForge servers:

```text
opac-better-protection/build/libs/opac-better-protection-<version>.jar
```

Example: `opac-better-protection-2.0.0.jar`.

---

## Verified Dependency Matrix

The baseline compile and verified test stack:

| Component | Target Version | Supported Range | Status |
| --- | --- | --- | --- |
| **Minecraft** | `1.21.1` | `[1.21.1,26.0)` | Baseline verified (probes supported through 1.21.11) |
| **NeoForge** | `21.1.232` | `[21.1.232,26.0)` | Baseline verified |
| **Java Toolchain** | `21` | `21` | Required |
| **Open Parties and Claims (OPaC)** | `neoforge-1.21.1-0.27.5` | `[0.27.5,)` | Required dependency |
| **Open Parties and Claims: Warfare** | `2.0.0` | `[2.0.0,)` | Optional companion mod (via dynamic soft bridge) |
| **Create** | `mc1.21.1-6.0.9` | `[6.0.9,)` | Optional integration |
| **Create Big Cannons (CBC)** | `5.11.7` | `[5.11.7,)` | Optional integration |
| **Resourceful Packet Library (RPL)** | `2.1.2` | `[2.1.2,)` | Required by CBC |
| **Create Aeronautics / Offroad** | `1.3.0` | `[1.3.0,)` | Optional integration |
| **Sable** | `2.0.3` | `[2.0.3,)` | Optional integration |

---

## Experimental Range & Metadata

The jar metadata in `neoforge.mods.toml` specifies:
- `minecraft_version_range = "[1.21.1,26.0)"`
- `neo_version_range = "[21.1.232,26.0)"`

This intentionally permits operators to test the jar on newer 1.21.x maintenance releases (such as 1.21.2 through 1.21.11) without loader rejections. However, only `1.21.1` is considered fully verified until the candidate passes the Compatibility Probe below.

---

## Incompatible Platforms & Versions

### Minecraft 26.x
**Minecraft 26.x is strictly incompatible with this build.**
Attempting to run this jar on Minecraft 26.1+ will fail due to breaking API and runtime changes:
1. `net.minecraft.resources.ResourceLocation` was renamed to `net.minecraft.resources.Identifier`.
2. `net.neoforged.fml.loading.FMLEnvironment.dist` (static field) was removed in favor of `FMLEnvironment.getDist()`.
3. Java 25 is required for 26.x, whereas this jar is compiled for Java 21.

A dedicated 26.x port will be released as a separate artifact.

### Other Mod Loaders
- **Fabric / Quilt:** Not supported by this jar.
- **Legacy Forge:** Not supported.
- **Minecraft 1.20.x and older:** Not supported.

---

## Companion Mod: Open Parties and Claims: Warfare

[Open Parties and Claims: Warfare](https://github.com/SvinPepe/opac-warfare) is designed to run alongside Better Protection:
- **No Hard Dependency:** Neither mod requires the other at compile time or at runtime.
- **Automatic Soft Integration:** When both jars are installed on a NeoForge server, Better Protection queries the warfare state via `IWarfareBridge`.
- **War Chunk Behavior:** In active contested chunks, explosion filtering is bypassed, CBC cannons are allowed to fire, and Create/Aeronautics drills operate normally for war combatants.

---

## Compatibility Probe Procedure

When verifying candidate updates for Minecraft 1.21.x:

1. **Build Artifact:**
   ```powershell
   .\gradlew.bat --no-daemon :opac-better-protection:jar test
   ```

2. **Deploy on Test Server:**
   Install the built jar on a test server running the candidate Minecraft/NeoForge build along with OPaC and optional compat mods.

3. **Verify Server Boot:**
   Confirm that the server boots cleanly without `ClassNotFoundException`, `NoSuchMethodError`, mixin apply failures, or packet registry errors.

4. **Execute Functional Checks:**
   - Execute `/claimrules explosions status`, `/claimrules create status`, and `/claimrules assembly status`.
   - Test explosion protection against TNT in a claimed chunk.
   - Verify that Create drills/saws are blocked in foreign claims with `/claimrules create disable`.
   - If CBC is installed, verify that firing a cannon from a protected claim displays the blocked alert.
   - If Aeronautics is installed, verify that borehead bearings stall at claim borders.
   - If Warfare is installed, verify that active contested war chunks allow explosions and participant machinery.

Document the candidate version as verified only after all checks pass.
