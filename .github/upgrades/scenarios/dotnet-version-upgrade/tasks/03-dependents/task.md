# 03-dependents: Upgrade samples and test project to net10.0

Upgrade all three Tier 1 projects from `net8.0` to `net10.0`:
- `src/samples/ConsoleSample/ConsoleSample.csproj`
- `src/samples/WebSample/WebSample.csproj`
- `src/test/Hangfire.Storage.SQLite.Test/Hangfire.Storage.SQLite.Test.csproj`

For each project, change `<TargetFramework>net8.0</TargetFramework>` to `<TargetFramework>net10.0</TargetFramework>`. The assessment shows 1 mandatory issue (Project.0002 — TFM change) and 40 potential Api.0002 issues in the test project — all `TimeSpan.FromXxx(double)` calls, same pattern as the library. Fix any compilation errors. The WebSample targets ASP.NET Core 8 — ensure the ASP.NET Core framework reference picks up the net10.0 version automatically (no explicit package version pinned). Run the full test suite after all three are updated.

**Done when**: All three projects target `net10.0`; full solution builds without errors; all tests pass.
