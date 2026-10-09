# RetroHub v1

A native Swift/UIKit game-library launcher for iOS 14+, designed around the iPhone 11 Pro's portrait screen. It runs as an ordinary sandboxed app with no private entitlements, jailbreak hooks, embedded emulator cores, cloud backend or analytics. iPad layouts use safe areas and anchored popovers.

## Included

- Dark console-inspired library with system categories, search, favorites and recent emulator opens.
- Files document picker, folder imports, and incoming Open In file URLs. Imports are copied to app storage on a serial worker queue.
- Versioned atomic JSON local database, non-destructive scans, editable titles/system/developer/year/notes, and custom downsampled covers from Files.
- RetroArch, Delta and PPSSPP profiles, per-system defaults, per-game overrides, exact URL allowlist, friendly unavailable-handler messages and manual transfer instructions.
- Settings, About, metadata export and merge import. Existing entries are never deleted by metadata import or scan.
- Checked-in Xcode project, original procedural icons, Swift Package unit tests, and macOS CI.

## Build

Requires macOS with **Xcode 14.3 or later**, iOS SDK and its command line tools (tested compilation status below). Swift language mode is 5; deployment target is 14.0. CI uses `macos-14` and its selected Xcode. There are no third-party packages, CocoaPods or XcodeGen dependencies.

```sh
cd RetroHub
python3 scripts/validate_repository.py
swift test
open RetroHub.xcodeproj
```

Select the `RetroHub` scheme and iPhone simulator or device. For normal provisioned device installs, choose your own development team and a unique bundle identifier in Signing & Capabilities. No custom entitlements are needed. The shared Xcode scheme builds the app; core unit tests run separately through `swift test`.

For an IPA:

```sh
scripts/build-ipa.sh
```

The script runs unit tests, archives an arm64 device app, applies a public ad-hoc code signature and packages `build/RetroHub.ipa`. It requests no private entitlements. This package is intended for a compatible **existing TrollStore** installation; its signature is not sufficient for ordinary stock-iOS installation. Do not replace your jailbreak or installed apps. Use your existing installer to install the resulting IPA. **Installation and operation on your iPhone have not been tested.** Back up any existing RetroHub data before replacing or removing an installation.

The GitHub Actions workflow runs tests, archives the device app, compiles a simulator build and uploads the IPA and logs. Push this repository to GitHub and run **Build RetroHub**; a green job plus the artifact confirms CI compilation. The workflow has not been run from this environment. It needs no signing secrets or external services.

To regenerate the project after adding Swift files:

```sh
python3 scripts/generate_project.py
```

Commit the resulting project, plist and icons. Generation is deterministic. The bundle identifier is `org.retrohub.launcher` by default. The project supports device families 1 and 2; iPhone portrait is intentional for v1.

## Supported files

| System | Indexed extensions |
| --- | --- |
| NES | nes |
| SNES | sfc, smc |
| GB | gb |
| GBC | gbc |
| GBA | gba |
| N64 | n64, z64, v64 |
| Nintendo DS | nds |
| PS1 | cue, chd, pbp |
| PSP | iso, cso |
| Sega Genesis | md, gen, smd; bin with manual system choice |
| Arcade | zip, 7z with manual system choice |

Matching is case-insensitive and extension-based, not a ROM authenticity or emulator compatibility check. ZIP/7z/BIN/PBP/ISO are ambiguous; select their system during import. They can be assigned to any category and corrected in game details. ZIP/7z are kept intact, never extracted. ISO can represent other disc systems; choose the intended category. Some emulator builds do not support every listed container. Unsupported lone files are rejected. Folders copy all regular companion files, retaining structure; hidden files and symlinks are skipped. CUE-directory BINs are preserved as tracks and not separately indexed. Prefer separate folders per disc/game. M3U playlists and archive contents are not indexed in v1; scan is explicit, not continuous.

For multi-track games import **the folder**, not individual CUE/BIN files. Disc references are not repaired or validated. Supply your own lawful ROMs and, where required, BIOS files to the emulator. Nothing copyrighted is downloaded or bundled. Reimporting a ROM creates another independent copy; v1 deduplicates scans by stored relative path, not content hashes.

## Emulator limits

See [docs/INTEGRATIONS.md](docs/INTEGRATIONS.md) for upstream evidence. All three scheme names were verified in upstream iOS plist files. **App-only launches on your installed versions are unverified.** RetroHub does not attempt game-specific deep links. You transfer files through the share sheet/Files and select the game in the emulator. RetroArch core support and emulator compatibility with iOS 14 depend on your installation.

## Storage and backup

Files are in `Documents/RetroHub/Games/<import UUID>/…`, covers in `Documents/RetroHub/Covers`, database in `Documents/RetroHub/library.json`. Access through Files → On My iPhone → RetroHub → RetroHub, or Finder file sharing. All heavy import/scan/database work runs on one serial queue. Covers are downsampled to at most 800 pixels.

A metadata JSON export contains relative paths and metadata, **not ROMs or cover images**. Import validates paths/schema and merges by storage path, preserving existing games and local IDs. Entries with missing assets remain visible; restore files at the matching paths using a full backup. To back up everything, copy the whole inner `RetroHub` folder. Per-system emulator defaults live in UserDefaults and are not included in the metadata export.

Database writes are atomic and in-memory state changes only after persistence succeeds. Unknown newer schemas and corrupt databases show recovery instructions instead of being reset. There are no destructive migrations or delete-library controls. A failed individual file/folder import rolls back its newly created batch; earlier items in a multi-item import remain saved even if a later item fails. Interrupted imports may leave unindexed folders; use Rescan to recover. Files/Covers are not deleted by rescans. Uninstalling the app still removes its sandbox—back up first.

## Validation status

See [docs/TEST_RESULTS.md](docs/TEST_RESULTS.md). This environment is Linux with no Swift compiler or Xcode: structural checks ran, but Swift unit tests, iOS compilation, CI execution and on-device testing **have not run**. No IPA is supplied or claimed. The repository contains eight automated XCTest cases and a concrete macOS build workflow.

## Layout

`App/` — UIKit screens, app lifecycle, original assets and plist.  
`Sources/RetroHubCore/` — classification, models, versioned database, imports and URL policy.  
`Tests/RetroHubCoreTests/` — synthetic fixtures only, no game content.  
`scripts/` — reproducible project generation, structural checks and IPA packaging.  
`.github/workflows/` — macOS build/test automation.

Source and original procedural assets are MIT licensed. Apple SF Symbols are used under Apple's platform terms; they are system-provided, not redistributed game artwork. Imported game and cover rights remain with their owners.
