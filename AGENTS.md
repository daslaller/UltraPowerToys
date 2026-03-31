---
description: 'Top-level AI contributor guidance for developing PowerToys - a collection of Windows productivity utilities'
applyTo: '**'
---

# PowerToys – AI Contributor Guide

This is the top-level guidance for AI contributions to PowerToys. Keep changes atomic, follow existing patterns, and cite exact paths in PRs.

## Overview

PowerToys is a set of utilities for power users to tune and streamline their Windows experience.

| Area | Location | Description |
|------|----------|-------------|
| Runner | `src/runner/` | Main executable, tray icon, module loader, hotkey management |
| Settings UI | `src/settings-ui/` | WinUI/WPF configuration app communicating via named pipes |
| Modules | `src/modules/` | Individual PowerToys utilities (each in its own subfolder) |
| Common Libraries | `src/common/` | Shared code: logging, IPC, settings, DPI, telemetry, utilities |
| Build Tools | `tools/build/` | Build scripts and automation |
| Documentation | `doc/devdocs/` | Developer documentation |
| Installer | `installer/` | WiX-based installer projects |

For architecture details and module types, see [Architecture Overview](doc/devdocs/core/architecture.md).

## Conventions

For detailed coding conventions, see:
- [Coding Guidelines](doc/devdocs/development/guidelines.md) – Dependencies, testing, PR management
- [Coding Style](doc/devdocs/development/style.md) – Formatting, C++/C#/XAML style rules
- [Logging](doc/devdocs/development/logging.md) – C++ spdlog and C# Logger usage

### Component-Specific Instructions

These instruction files are automatically applied when working in their respective areas:
- [Runner & Settings UI](.github/instructions/runner-settings-ui.instructions.md) – IPC contracts, schema migrations
- [Common Libraries](.github/instructions/common-libraries.instructions.md) – ABI stability, shared code guidelines

## Build

### Prerequisites

- Visual Studio 2022 17.4+ or Visual Studio 2026
- Windows 10 1803+ (April 2018 Update or newer)
- Initialize submodules once: `git submodule update --init --recursive`

### Build Commands

| Task | Command |
|------|---------|
| First build / NuGet restore | `tools\build\build-essentials.cmd` |
| Build current folder | `tools\build\build.cmd` |
| Build with options | `build.ps1 -Platform x64 -Configuration Release` |

### Build Discipline

1. One terminal per operation (build → test). Do not switch or open new ones mid-flow
2. After making changes, `cd` to the project folder that changed (`.csproj`/`.vcxproj`)
3. Use scripts to build: `tools/build/build.ps1` or `tools/build/build.cmd`
4. For first build or missing NuGet packages, run `build-essentials.cmd` first
5. **Exit code 0 = success; non-zero = failure** – treat this as absolute
6. On failure, read the errors log: `build.<config>.<platform>.errors.log`
7. Do not start tests or launch Runner until the build succeeds

### Build Logs

Located next to the solution/project being built:
- `build.<configuration>.<platform>.errors.log` – errors only (check this first)
- `build.<configuration>.<platform>.all.log` – full log
- `build.<configuration>.<platform>.trace.binlog` – for MSBuild Structured Log Viewer

For complete details, see [Build Guidelines](tools/build/BUILD-GUIDELINES.md).

## Tests

### Test Discovery

- Find test projects by product code prefix (e.g., `FancyZones`, `AdvancedPaste`)
- Look for sibling folders or 1-2 levels up named `<Product>*UnitTests` or `<Product>*UITests`

### Running Tests

1. **Build the test project first**, wait for exit code 0
2. Run via VS Test Explorer (`Ctrl+E, T`) or `vstest.console.exe` with filters
3. **Avoid `dotnet test`** in this repo – use VS Test Explorer or vstest.console.exe

### Test Types

| Type | Requirements | Setup |
|------|--------------|-------|
| Unit Tests | Standard dev environment | None |
| UI Tests | WinAppDriver v1.2.1, Developer Mode | Install from [WinAppDriver releases](https://github.com/microsoft/WinAppDriver/releases/tag/v1.2.1) |
| Fuzz Tests | OneFuzz, .NET 8 | See [Fuzzing Tests](doc/devdocs/tools/fuzzingtesting.md) |

### Test Discipline

1. Add or adjust tests when changing behavior
2. If tests skipped, state why (e.g., comment-only change, string rename)
3. New modules handling file I/O or user input **must** implement fuzzing tests

### Special Requirements

- **Mouse Without Borders**: Requires 2+ physical computers (not VMs)
- **Multi-monitor utilities**: Test with 2+ monitors, different DPI settings

For UI test setup details, see [UI Tests](doc/devdocs/development/ui-tests.md).

## Boundaries

### Ask for Clarification When

- Ambiguous spec after scanning relevant docs
- Cross-module impact (shared enum/struct) is unclear
- Security, elevation, or installer changes involved
- GPO or policy handling modifications needed

### Areas Requiring Extra Care

| Area | Concern | Reference |
|------|---------|-----------|
| `src/common/` | ABI breaks | [Common Libraries Instructions](.github/instructions/common-libraries.instructions.md) |
| `src/runner/`, `src/settings-ui/` | IPC contracts, schema | [Runner & Settings UI Instructions](.github/instructions/runner-settings-ui.instructions.md) |
| Installer files | Release impact | Careful review required |
| Elevation/GPO logic | Security | Confirm no regression in policy handling |

### What NOT to Do

- Don't merge incomplete features into main (use feature branches)
- Don't break IPC/JSON contracts without updating both runner and settings-ui
- Don't add noisy logs in hot paths
- Don't introduce third-party deps without PM approval and `NOTICE.md` update

## Validation Checklist

Before finishing, verify:

- [ ] Build clean with exit code 0
- [ ] Tests updated and passing locally
- [ ] No unintended ABI breaks or schema changes
- [ ] IPC contracts consistent between runner and settings-ui
- [ ] New dependencies added to `NOTICE.md`
- [ ] PR is atomic (one logical change), with issue linked

## Documentation Index

### Core Architecture
- [Architecture Overview](doc/devdocs/core/architecture.md)
- [Runner](doc/devdocs/core/runner.md)
- [Settings System](doc/devdocs/core/settings/readme.md)
- [Module Interface](doc/devdocs/modules/interface.md)

### Development
- [Coding Guidelines](doc/devdocs/development/guidelines.md)
- [Coding Style](doc/devdocs/development/style.md)
- [Logging](doc/devdocs/development/logging.md)
- [UI Tests](doc/devdocs/development/ui-tests.md)
- [Fuzzing Tests](doc/devdocs/tools/fuzzingtesting.md)

### Build & Tools
- [Build Guidelines](tools/build/BUILD-GUIDELINES.md)
- [Tools Overview](doc/devdocs/tools/readme.md)

### Instructions (Auto-Applied)
- [Runner & Settings UI](.github/instructions/runner-settings-ui.instructions.md)
- [Common Libraries](.github/instructions/common-libraries.instructions.md)

## Cursor Cloud specific instructions

### Platform constraint

PowerToys is a **Windows-only** desktop application. All C++/C# projects target `net9.0-windows10.0.26100.0` and MSVC with Windows SDK. The full solution (`PowerToys.slnx`) **cannot be built or tested on Linux**. `dotnet restore` on the full solution fails due to missing C++ toolchain and case-sensitivity issues (e.g. `Common/` vs `common/` in paths).

### What works on Linux (Cloud Agent VMs)

| Capability | Command | Notes |
|---|---|---|
| Git submodules | `git submodule update --init --recursive` | Required before any build; deps: spdlog, expected-lite |
| .NET tool restore | `dotnet tool restore` | Restores `xstyler` (XAML linter) and `dotnet-consolidate` |
| XAML lint check | `dotnet xstyler --passive --file <path>` or `--directory <dir>` | Verifies XAML formatting without modifying files |
| Package consolidation check | `dotnet dotnet-consolidate -s PowerToys.slnx` | Checks NuGet package version consistency (may emit warnings about missing projects) |
| MCP dev server | `cd tools/mcp/github-artifacts && npm install && npm start` | Node.js MCP server for GitHub issue image/attachment fetching; the only runnable service |
| MCP server tests | `cd tools/mcp/github-artifacts && npm test` | Requires `GITHUB_TOKEN` env var for API calls |
| Code review / static analysis | Read/grep C#, C++, XAML source | No compilation needed for code reviews |

### .NET SDK setup

The update script installs .NET 9.0 SDK and .NET 8.0 runtime (needed by `dotnet-consolidate`) to `$HOME/.dotnet`. The PATH is configured in `~/.bashrc`:
```
export PATH="$HOME/.dotnet:$HOME/.dotnet/tools:$PATH"
export DOTNET_ROOT="$HOME/.dotnet"
```

### Key limitations for Cloud Agents

- **No MSBuild/compilation**: Cannot build `.csproj`, `.vcxproj`, or `.slnx` projects. Build validation requires Windows + Visual Studio 2022/2026.
- **No `dotnet test`**: The repo advises against `dotnet test` even on Windows; use VS Test Explorer or `vstest.console.exe`.
- **No UI testing**: WinAppDriver + Developer Mode on Windows required.
- **Case sensitivity**: Linux filesystem is case-sensitive; some solution/project references use different casing than actual directory names (e.g. `Common/` vs `common/`). This breaks `dotnet restore` for the full solution.

### Practical workflow for Cloud Agents

1. Make code changes (C#, C++, XAML, docs, scripts).
2. Run `dotnet xstyler --passive --file <changed.xaml>` for XAML formatting checks.
3. For code review tasks, rely on static analysis (reading code, grepping patterns).
4. For the MCP dev tool (`tools/mcp/github-artifacts/`), run `npm test` to validate changes.
5. Build/test validation must be deferred to Windows CI or a Windows dev environment.
