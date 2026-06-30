# 01-prerequisites: Verify SDK and toolchain

Confirm the .NET 10 SDK is installed and available for the upgrade. Check whether any `global.json` files exist in the repository that might pin an older SDK version and would need updating before projects can target net10.0. Verify the solution loads cleanly before making any changes.

**Done when**: `dotnet --version` reports a .NET 10 SDK; any `global.json` SDK version constraint is compatible with net10.0; solution restores and loads without errors.
