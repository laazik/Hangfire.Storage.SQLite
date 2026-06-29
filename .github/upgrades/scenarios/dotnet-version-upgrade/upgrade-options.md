# Upgrade Options — Hangfire.Storage.SQLite

Assessment: 4 projects (1 library net48/netstandard2.0, 3 modern net8.0); all SDK-style; 0 incompatible packages; 60 source-compat warnings (TimeSpan.FromXxx double overloads); user intent: net10.0 only, no cross-platform multi-targeting.

## Strategy

### Upgrade Strategy
Solution contains a .NET Framework project (net48) alongside modern .NET projects — Bottom-Up is required to handle the Framework→modern boundary correctly.

| Value | Description |
|-------|-------------|
| **Bottom-Up** (selected) | Fixed — .NET Framework present with 4 projects. Upgrade library (Tier 0) first, then dependents (Tier 1). Between-tier validation after each tier. |

## Project Structure

### Project Approach
Hangfire.Storage.SQLite targets net48/netstandard2.0. User explicitly wants net10.0 only — no multi-targeting, no cross-platform dependencies.

| Value | Description |
|-------|-------------|
| **In-place** (selected) | Replace TFM directly to net10.0. All multi-targeting (netstandard2.0, net48) removed. Cleanest result; user confirmed no need to support Framework consumers. |
| Multi-targeting | Would add net10.0 alongside net48 — explicitly not wanted by user. |

### Package Management
Upgrade crosses Framework→modern boundary; deferring CPM avoids VersionOverride friction during migration.

| Value | Description |
|-------|-------------|
| **Per-Project (defer CPM to post-migration)** (selected) | Each project keeps its own package versions during migration. CPM can be added cleanly after all projects are on net10.0. |
| Central Package Management (CPM) | Would require VersionOverride during migration — better after stabilization. |

## Compatibility

### Unsupported API Handling
60 Api.0002 occurrences flagged — all are TimeSpan.FromXxx(double) calls. These methods still exist in .NET 10 (new integer overloads were added in .NET 7, but double overloads remain). Simple, mechanical fixes where needed.

| Value | Description |
|-------|-------------|
| **Fix Inline** (selected) | Resolve all TimeSpan.FromXxx warnings in the same task. Changes are mechanical and few — no deferred stubs needed. |
| Defer Complex Changes | Not warranted — none of the 60 flagged items require architectural decisions. |

## Modernization

### Nullable Reference Types
Nullable not currently enabled; target is net10.0. 4 projects, crossing Framework boundary — leave disabled during migration to avoid compounding effort.

| Value | Description |
|-------|-------------|
| **Leave Disabled** (selected) | Do not enable nullable during this upgrade. Can be enabled as a separate effort after migration stabilizes. |
| Enable Nullable Reference Types | Would add warnings during migration — better deferred. |
