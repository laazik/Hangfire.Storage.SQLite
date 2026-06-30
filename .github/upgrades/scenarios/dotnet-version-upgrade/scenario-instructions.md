# .NET Version Upgrade

## Preferences
- **Flow Mode**: Automatic
- **Target Framework**: net10.0

## Source Control
- **Source Branch**: master
- **Working Branch**: upgrade-dotnet-10
- **Commit Strategy**: After Each Task
- **Branch Sync**: Auto (Merge)

## Upgrade Options
**Source**: .github/upgrades/scenarios/dotnet-version-upgrade/upgrade-options.md

### Strategy
- Upgrade Strategy: Bottom-Up (fixed — .NET Framework project present, 4 projects)

### Project Structure
- Project Approach: In-place (user intent: net10.0 only, remove all multi-targeting)
- Package Management: Per-Project (defer CPM to post-migration)

### Compatibility
- Unsupported API Handling: Fix Inline

### Modernization
- Nullable Reference Types: Leave Disabled

## Strategy
**Selected**: Bottom-Up (Dependency-First)
**Rationale**: Library targets net48 alongside netstandard2.0 — Framework→modern boundary requires bottom-up ordering. 4 projects, 2-tier dependency graph.

### Execution Constraints
- Strict tier ordering: Tier 0 (library) must complete and validate before Tier 1 (dependents) begins
- Between-tier validation: after upgrading the library, confirm it builds cleanly on net10.0 before touching dependents
- In-place TFM replacement: no multi-targeting, remove all OS-conditional PropertyGroups, single net10.0 TFM
- Fix all Api.0002 (TimeSpan.FromXxx) compilation errors inline — do not stub or defer
- Full test suite must pass before final validation task is complete
