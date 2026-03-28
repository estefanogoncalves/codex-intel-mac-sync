# codex-intel-mac

Single-script workflow to rebuild Codex Desktop for Intel macOS from a source `Codex.dmg` or `Codex.app`.

It rebuilds the Electron runtime and native modules for `x86_64`, repackages the app bundle, and can emit a ready-to-distribute DMG.

## Automation

- Scheduled GitHub Action: [`.github/workflows/rebuild-intel-codex.yml`](.github/workflows/rebuild-intel-codex.yml)
- Workflow notes: [`docs/auto-release.md`](docs/auto-release.md)

## Script

- `./codex-intel.sh`

## Requirements

- An Intel Mac running macOS.
- Xcode Command Line Tools.
- `npm`/`npx` available in `PATH`.
- A source `Codex.dmg` or `Codex.app`.
- On Monterey only, you may also need `brew install llvm` for the first native rebuild.

## Behavior

- Running with no arguments prints usage + quick start instructions.
- If Xcode Command Line Tools are missing, the script asks to install them and triggers `xcode-select --install`.
- On Intel Macs running newer macOS releases, the script keeps using the system Apple toolchain when it already supports the required Electron 40 headers.
- On Monterey, if Apple clang cannot compile Electron 40 native modules, the script looks for Homebrew LLVM automatically and tells you to install it when missing.
- If no source app is found, the script tells you to provide `Codex.dmg`/`Codex.app` and shows a download page.

## What It Changes

- Rebuilds the Electron runtime and native modules for `x86_64`.
- Replaces the app runtime binaries with Intel builds.
- Injects rebuilt `better-sqlite3` and `node-pty` binaries into `app.asar.unpacked`.
- Optionally bundles an Intel `codex` CLI inside the app.
- Re-signs the rebuilt app ad hoc and can generate a DMG.

## Quick Start

1. Download `Codex.dmg` from:
   https://openai.com/codex/get-started/
2. Put it in this folder as `./Codex.dmg`, or pass `--source-dmg`.
3. Run:

```bash
./codex-intel.sh --dmg
```

Outputs:

- `./dist/Codex-Intel.dmg`

Output selection:

- `--dmg`: output DMG
- `--app`: output rebuilt app
- `--app --dmg`: output both
- If neither is provided, default is `--dmg` only.

## Typical Commands

Full flow (build + repackage + dmg):

```bash
./codex-intel.sh --source-dmg ./Codex.dmg --dmg
```

Output app only:

```bash
./codex-intel.sh --source-dmg ./Codex.dmg --app
```

Output both app + dmg:

```bash
./codex-intel.sh --source-dmg ./Codex.dmg --app --dmg
```

Use an installed app instead of dmg:

```bash
./codex-intel.sh --source-app /Applications/Codex.app --dmg
```

Monterey fallback with an explicit LLVM prefix:

```bash
./codex-intel.sh --source-app /Applications/Codex.app --dmg --llvm-prefix /usr/local/opt/llvm
```

Monterey note:

- The first native rebuild may require `brew install llvm` if the system Apple clang does not provide the C++20/libc++ headers required by Electron 40.

Build artifacts only:

```bash
./codex-intel.sh --build-only
```

Repackage only (reuse existing `.build`):

```bash
./codex-intel.sh --repackage-only --source-dmg ./Codex.dmg
```

DMG only:

```bash
./codex-intel.sh --dmg-only --dmg-source-app ./dist/Codex-Intel.app
```

## Troubleshooting

Monterey native rebuilds:

- If the system Apple clang does not provide the Electron 40 C++20/libc++ headers, install Homebrew LLVM with `brew install llvm`.
- If LLVM is installed in a non-default prefix, pass `--llvm-prefix /path/to/llvm`.

Missing source app:

- Pass `--source-dmg /path/to/Codex.dmg` or `--source-app /path/to/Codex.app`.
- If neither is passed, the script also checks `./Codex.dmg` and `/Applications/Codex.app`.

## DMG Window/Layout Options

Defaults:

- `--window-width 840`
- `--window-height 520`
- `--app-icon-x 240 --app-icon-y 260`
- `--apps-icon-x 650 --apps-icon-y 260`

Example:

```bash
./codex-intel.sh --dmg-only \
  --dmg-source-app ./dist/Codex-Intel.app \
  --output-dmg ./dist/Codex-Intel.dmg \
  --window-width 820 \
  --window-height 500
```

## Help

```bash
./codex-intel.sh --help
```

## Credits

- Original `codex-intel-mac` script and packaging flow by `ckvv`.
