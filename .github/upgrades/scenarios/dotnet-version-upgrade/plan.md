# .NET Version Upgrade Plan

## Overview

**Target**: Upgrade all projects from their current TFMs (netstandard2.0/net48/net8.0) to net10.0 only.
**Scope**: 4 projects — 1 class library (Tier 0), 3 dependents: 1 test project + 2 samples (Tier 1).

### Selected Strategy
**Bottom-Up (Dependency-First)** — Upgrade from leaf nodes to root applications, tier by tier.
**Rationale**: 4 projects with 2-tier dependency graph. Library (`Hangfire.Storage.SQLite`) currently targets `netstandard2.0;net48` — the .NET Framework TFM requires bottom-up ordering.

```
Tier 1: [ConsoleSample]  [WebSample]  [Hangfire.Storage.SQLite.Test]
                  ↓             ↓                    ↓
Tier 0:      [Hangfire.Storage.SQLite]
```

**Tier 0 completion criteria**: Library builds on net10.0, all multi-targeting removed.
**Tier 1 completion criteria**: All dependents build and tests pass on net10.0.

## Tasks

### 01-prerequisites: Verify SDK and toolchain

Confirm the .NET 10 SDK is installed and available for the upgrade. Check whether any `global.json` files exist in the repository that might pin an older SDK version and would need updating before projects can target net10.0. Verify the solution loads cleanly before making any changes.

**Done when**: `dotnet --version` reports a .NET 10 SDK; any `global.json` SDK version constraint is compatible with net10.0; solution restores and loads without errors.

---

### 02-sqlite-library: Upgrade Hangfire.Storage.SQLite library to net10.0

Upgrade `src/main/Hangfire.Storage.SQLite/Hangfire.Storage.SQLite.csproj` from `netstandard2.0;net48` (with OS-conditional net48) to a single `net10.0` TFM. Per user intent: no cross-platform multi-targeting, no netstandard2.0 compatibility shim, no net48 support — net10.0 only.

Remove the OS-conditional `<PropertyGroup>` blocks in the csproj and replace with a single unconditional `<TargetFramework>net10.0</TargetFramework>`. Remove `NETStandard.Library` package reference (not needed for net10.0). Remove `SQLitePCLRaw.bundle_e_sqlite3` and evaluate whether `SQLitePCLRaw.bundle_green` or a simpler provider is the right choice for net10.0 (the current project had `bundle_e_sqlite3` — keep it unless it causes issues). Update any source files flagged by the assessment: 18 `TimeSpan.FromXxx(double)` occurrences across `SQLiteDistributedLock.cs`, `SQLiteStorageOptions.cs`, `SQLiteStorage.cs`, `ExpirationManager.cs`, and `HangfireSQLiteConnection.cs`. These methods still exist in .NET 10 but new integer-based overloads were added — resolve any compilation errors or warnings by using the correct overload (e.g., `TimeSpan.FromSeconds(30)` → `TimeSpan.FromSeconds(30)` is still valid; only fix if there's an actual build error). Build the library and confirm it compiles cleanly on net10.0. Between-tier validation: ConsoleSample, WebSample, and the test project will be broken at this point (expected — Tier 1 is next).

**Done when**: `Hangfire.Storage.SQLite.csproj` targets `net10.0` only; no `netstandard2.0` or `net48` TFMs remain; OS-conditional PropertyGroups removed; library builds without errors; NuGet restore succeeds.

---

### 03-dependents: Upgrade samples and test project to net10.0

Upgrade all three Tier 1 projects from `net8.0` to `net10.0`:
- `src/samples/ConsoleSample/ConsoleSample.csproj`
- `src/samples/WebSample/WebSample.csproj`
- `src/test/Hangfire.Storage.SQLite.Test/Hangfire.Storage.SQLite.Test.csproj`

For each project, change `<TargetFramework>net8.0</TargetFramework>` to `<TargetFramework>net10.0</TargetFramework>`. The assessment shows 1 mandatory issue (Project.0002 — TFM change) and 40 potential Api.0002 issues in the test project — all `TimeSpan.FromXxx(double)` calls, same pattern as the library. Fix any compilation errors. The WebSample targets ASP.NET Core 8 — ensure the ASP.NET Core framework reference picks up the net10.0 version automatically (no explicit package version pinned). Run the full test suite after all three are updated.

**Done when**: All three projects target `net10.0`; full solution builds without errors; all tests pass.

---

### 04-final-validation: Full solution validation and cleanup

Perform a clean restore and full solution build to confirm no project is left behind. Run the complete test suite. Verify the NuGet package output from `Hangfire.Storage.SQLite` correctly reports `net10.0` as the only TFM. Clean up any leftover artifacts from the multi-targeting era (e.g., bin/obj folders with net48/netstandard2.0 outputs). Document post-migration recommendations: Central Package Management can now be added cleanly since all projects are SDK-style on a single TFM.

**Done when**: `dotnet build` succeeds for the full solution with zero errors; all tests pass; the generated NuGet package contains only a `net10.0` target; no stale multi-targeting artifacts remain.
