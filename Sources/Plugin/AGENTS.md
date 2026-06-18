# AGENTS.md — SimHub Plugin

Guidance for AI agents working on the SABT SimHub plugin. See the
[repo-level AGENTS.md](../../AGENTS.md) for project-wide rules (especially the
contribution and AI-content rules). `CLAUDE.md` in this directory is a symlink
to this file.

## Stack

- **C# on .NET Framework 4.8**, WPF, MVVM.
- Builds as a SimHub plugin library: `User.ActiveBeltTensioner.dll`.
- Root namespace: `User.ActiveBeltTensioner`.

## Build

Windows + Visual Studio 2015 or newer is required.

1. Install SimHub and set the `SIMHUB_INSTALL_PATH` environment variable to its
   install directory. Every SimHub / third-party reference
   (`SimHub.Plugins`, `GameReaderCommon`, `MahApps.Metro`, `OxyPlot`, `log4net`,
   `Newtonsoft.Json`, `Wotever*`, …) is resolved from that path — see
   `User.ActiveBeltTensioner.csproj`. Without it the build fails.
2. Open `User.ActiveBeltTensioner.sln` and build (Debug or Release).
3. A post-build step `XCOPY`s the plugin DLL/PDB and the language `.resx` files
   into `%SIMHUB_INSTALL_PATH%`. Restart SimHub to load the new build.

There is **no automated unit-test suite**. Verification is: it compiles, and it
behaves correctly when loaded and exercised manually in SimHub.

## Architecture / key files

- `DevicePlugin.cs` — plugin entry point; implements SimHub's `IPlugin`,
  `IDataPlugin` and `IWPFSettingsV2`. Runs the ~60 Hz telemetry control loop and
  builds a `TelemetrySnapshot` (Surge / Sway / Heave G-forces + Speed +
  IsActive) that drives motor torque.
- `MotorController.cs` — RS485 serial communication to the Waveshare DDSM driver
  board. Contains the nested `Motor` state (torque smoothing, failure handling).
  Shared motor/telemetry state is guarded by locks — keep access thread-safe.
- `DeviceControl.xaml` / `DeviceControl.xaml.cs` + `DeviceViewModel.cs` — the
  WPF settings UI (MVVM).
- `DeviceSettings.cs` — profile-based settings persistence via SimHub.
- `Languages/*.resx` — localization (EN / DE / FR / IT). When adding a
  user-facing string, add it to **all** locale files; English is the fallback.

## Conventions

- Stay within the `User.ActiveBeltTensioner` namespace.
- Keep the MVVM separation (View / ViewModel / settings).
- Put user-facing strings in the locale `.resx` files, not inline literals.
- Preserve thread-safe access to shared motor and telemetry state.

## Don't commit

Build artifacts: `bin/`, `obj/`, etc. (already covered by `.gitignore`).
