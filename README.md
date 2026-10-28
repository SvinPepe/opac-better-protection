# Open Parties and Claims: Better Protection

**Open Parties and Claims: Better Protection** is an unofficial NeoForge addon for
[Open Parties and Claims](https://modrinth.com/mod/open-parties-and-claims).

It adds configurable claim protection rules for explosions, storage, Withers, villagers,
and foreign interactions, alongside deep integration for Create mining, contraption assembly,
Aeronautics / Offroad bore protection, Sable contraption protection, and Create Big Cannons
launch / impact rules. Open Parties and Claims remains the source of party membership and
claim ownership; this project builds advanced protection mechanics on top of OPaC territories.

## Requirements

- Minecraft `1.21.1` baseline; metadata allows experimental `[1.21.1,26.0)` probes (supports 1.21.x through 1.21.11)
- NeoForge `21.1.232` baseline; metadata allows experimental `[21.1.232,26.0)` probes
- Java `21`
- Open Parties and Claims `neoforge-1.21.1-0.27.5` or newer on the baseline stack

Optional integrations activate only when their target mods are installed:

- Create `mc1.21.1-6.0.9` or newer
- Create Aeronautics/Offroad `1.3.0` or newer for bore protection
- Sable `2.0.3` for optional contraption assembly protection
- Create Big Cannons `5.11.7` or newer

## What To Install

Server owners install the single distribution jar built by the `opac-better-protection` module:

```text
opac-better-protection/build/libs/opac-better-protection-<version>.jar
```

This single distribution jar bundles:
- `customclaims_core`
- `customclaims_protection`
- `customclaims_create`
- `customclaims_aeronautics`
- `customclaims_big_cannons`

## Quick Start

Build the artifact:

```powershell
.\gradlew.bat --no-daemon :opac-better-protection:jar test
```

On Linux/macOS:

```sh
./gradlew --no-daemon :opac-better-protection:jar test
```

Copy the resulting `opac-better-protection-*.jar` into the server `mods/` folder together
with Open Parties and Claims.

## Commands

```text
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
/claimrules gui
```

## Documentation

- [Server Admin Guide](docs/server-admin.md)
- [Compatibility](docs/compatibility.md)
- [Configuration](docs/configuration.md)
- [Architecture](docs/architecture.md)
- [Development](docs/development.md)
- [AGENTS.md](AGENTS.md)

## License

MIT. See [LICENSE](LICENSE).
