# AGENTS.md

Guidance for AI coding agents working in this repository. Humans should read
[`README.md`](README.md) and [`CONTRIBUTING.md`](CONTRIBUTING.md) first; this
file points agents at the right places and the rules they must follow.

> Claude Code reads `CLAUDE.md`, which is a symlink to this file, so both see
> the same guidance.

## Project overview

The **Simple Active Belt Tensioner (SABT)** is a haptic device for sim racing
that dynamically tensions a racing harness in response to game telemetry. It is
aimed at people without a software or electronics background — the build
requires no soldering or programming. The project is **pre-release** and under
active development. It has two distinct halves:

- **Hardware** — 3D-printable mechanical parts, designed in FreeCAD and
  distributed as STEP files.
- **Software** — a SimHub plugin (C# / WPF) that reads telemetry and drives the
  motors.

See [`README.md`](README.md) for the full overview, cost breakdown and design
goals.

## Repository layout

| Path | Contents |
|------|----------|
| `Printables/` | Ready-to-print STEP files (exported output — **do not hand-edit**; change the FreeCAD source instead). |
| `Sources/Printables/` | FreeCAD `.FCStd` source files for the printed parts. |
| `Sources/Plugin/` | C# SimHub plugin source. See [`Sources/Plugin/AGENTS.md`](Sources/Plugin/AGENTS.md) for build and architecture details. |
| `Software/` | Pre-built plugin distribution (`SABT SimHub Plugin.zip`). |

Key docs at the root: [`README.md`](README.md),
[`INSTRUCTIONS.md`](INSTRUCTIONS.md) (build & assembly), [`SAFETY.md`](SAFETY.md),
[`FAQ.md`](FAQ.md), [`CONTRIBUTING.md`](CONTRIBUTING.md),
[`LICENSE.md`](LICENSE.md).

## Where the code lives

Almost all software work happens in `Sources/Plugin/`. Read the
[plugin AGENTS.md](Sources/Plugin/AGENTS.md) before changing any C# code.

## Contribution rules (must follow)

These come from [`CONTRIBUTING.md`](CONTRIBUTING.md) — read it in full before
opening anything:

- **One issue per feature, enhancement or bug fix.** Keep scope tight; use
  concise Title Case issue names.
- **One PR per issue.** Name it `Issue 123` and link `#123` at the top of the
  description.
- **Code must build and function before you open a (non-draft) PR.** Use a
  Draft PR for work in progress.
- **No superfluous files** — don't commit OS-specific metadata (`.DS_Store`),
  temporary files, or build artifacts.

### AI-generated content

Using coding assistants is fine, **but submissions that are entirely or mostly
agent-generated will not be accepted.** Keep changes minimal, focused, reviewed
and human-owned — a maintainer should not have to do more work cleaning up an
agent's output than they would writing it themselves.

## Licensing

Dual-licensed: **MIT** for the software (the SimHub plugin) and **CERN-OHL-P**
for the hardware (CAD / 3D parts). Do **not** reuse the "SABT" or "GW"
branding/logos in derivative products, and do not imply author endorsement. See
[`LICENSE.md`](LICENSE.md).
