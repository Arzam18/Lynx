# Lynx Android build reference

The build is based on the proven Sapling workflow pattern, adapted to this repository's .NET 10 CLI project:

- `src/Lynx.Cli/Lynx.Cli.csproj` is the publish target.
- SDK is pinned to patched `10.0.114`; NativeAOT compiler/runtime pack is `10.0.14`.
- The Actions-only PackageDownload request uses `Microsoft.NETCore.App.Runtime.NativeAOT.linux-bionic-arm64`.
- NDK r27c provides the cross-linker/toolchain; output RID is `linux-bionic-arm64`.
- No forced scalar property is used. Lynx has no `SaplingBackend`-like compile-time selector.
- The source modifications are limited to the ARM capability diagnostic, SDK pin, workflow, and Android port/test documentation.
