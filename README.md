# avm-plugin-java

avm's native OpenJDK provider: version install/switching and automatic
`JAVA_HOME`. Works with [avm](https://github.com/PrajaNova/avm) via avm's
plugin marketplace — install with:

```bash
avm plugin add java
```

That fetches this repo's latest compiled release for your platform from
GitHub — no Rust toolchain or network access needed beyond the one
download. See [Release process](#release-process) below if you're
building/publishing this repo itself.

## Features

- **Live version index, OpenJDK only** — `avm java versions` reads the
  [foojay Disco API](https://api.foojay.io), filtered to `temurin`
  (Eclipse Adoptium's build of OpenJDK — the reference "no vendor games"
  distribution, and unlike some vendors it keeps its full historical patch
  archive instead of pruning old releases). One request returns the whole
  patch history already sorted newest-first.
- **Version install & switching** — per-project (local) and machine-wide
  (global) pins, same model as jenv/asdf-java/sdkman.
- **Automatic `JAVA_HOME`** — computed directly from the selected version,
  in-process, every time you run a command — no stale/incorrectly-cached
  value possible (see the main repo's
  [`docs/plugins/CREATING_A_PLUGIN.md`](https://github.com/PrajaNova/avm/blob/main/docs/plugins/CREATING_A_PLUGIN.md)
  for why that mattered enough to be the reason this provider exists).

## Commands

Once installed (`avm plugin add java`), everything is under `avm java`:

| Command | What it does |
| --- | --- |
| `avm java` | Interactive menu (list / browse versions / install latest / uninstall / help) |
| `avm java list` | Show the selected and installed versions |
| `avm java versions` | Browse the 10 most recent releases |
| `avm java <major> versions` | e.g. `avm java 17 versions` — every patch release on that major line |
| `avm java latest versions` | Just the newest release |
| `avm java use <version> [-g\|--global]` | Select an installed version, locally (default) or globally |
| `avm java set <version> [-g\|--global]` | Alias for `use` |
| `avm java install <version\|latest\|N>` | Install (if missing) + auto-pin: local, and global too if nothing's pinned globally yet |
| `avm java install <version> --global` | Install + pin globally only |
| `avm java install <version> --no-pin` | Install without touching any pin |
| `avm java uninstall <version>` | Remove a managed version |

`<version>` accepts an exact Temurin version (`17.0.13+11`), a bare major
(`17` — resolves to that major's latest), or `latest`. Versions install
with an `openjdk-` prefix on disk (`~/.avm/tools/java/openjdk-17.0.13+11`)
so an existing manually-pinned `openjdk-*` version string keeps working.

## Environment

`JAVA_HOME` is exported automatically to the selected version's install
root whenever it's the active pin (local overriding global, same as every
other avm tool) — nothing extra to configure.

## Release process

Tag-triggered (`vX.Y.Z`) GitHub Actions workflow builds
`avm-plugin-java_<os>_<arch>.tar.gz` for `linux_amd64`, `linux_arm64`, and
`darwin_arm64`, and publishes them as a GitHub Release — that's what `avm
plugin add java` downloads. See
[`avm-marketplace`](https://github.com/PrajaNova/avm-marketplace) for the
registry entry that points at this repo, and the main
[avm repo](https://github.com/PrajaNova/avm)'s
`docs/plugins/CREATING_A_PLUGIN.md` for the full wire protocol this
executable speaks (`manifest`, `versions`, `is-installed`,
`installed-versions`, `executable-path`, `env-vars`, `install`,
`uninstall`).

```bash
cargo build --release
# binary at target/release/avm-plugin-java
```
