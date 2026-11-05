# Lynx Android ARM64 port audit

## Input

- Archive: `Lynx-main.zip`
- Source version: 1.11.0
- Target project: `src/Lynx.Cli/Lynx.Cli.csproj`
- Core library: `src/Lynx/Lynx.csproj`

## Change set

The Android port is additive and intentionally small:

1. `global.json` — pins the patched .NET 10 SDK 10.0.114.
2. `.github/workflows/android-arm64.yml` — adds a dispatch-only NativeAOT build for `linux-bionic-arm64` using NDK r27c and the patched 10.0.14 runtime/compiler packs.
3. `src/Lynx/UCIHandler.cs` — adds `ARM AdvSimd (NEON)` to the existing `runtimeconfig` hardware diagnostic.
4. `docs/ANDROID-ARM64-PORT.md` — documents target, CPU paths, and constraints.
5. `docs/ANDROID-TERMUX-TEST.md` — gives the device-side UCI smoke test.
6. `docs/ANDROID-BUILD-REFERENCE.md` — records the Sapling NativeAOT adaptation.
7. `docs/ANDROID-PORT-AUDIT.md` — this audit.
8. `docs/ANDROID-SHA256SUMS.txt` — checksums for the modified files.

No original engine search, evaluation, move generation, tuning values, thread defaults, time management, or hash defaults were changed. The `appsettings.json` sidecar is included unchanged in the final publish output.

## CPU behavior observed in source

- `Attacks.cs`: BMI2 PEXT/PDEP tables are initialized and used only when BMI2 is supported; otherwise magic-bitboard attack generation is used.
- `Model/Bitboard.cs`: BMI1 `ExtractLowestSetBit` is guarded; arithmetic extraction is the fallback.
- `Model/TranspositionTable.cs`: SSE prefetch is guarded; TT index calculation has a non-BMI2 `UInt128` fallback.
- `UCIHandler.cs`: the existing x86 flags remain; ARM AdvSimd is now reported as an additional capability.

The engine contains no external `.nnue`, `.weights`, or `.net` model assets in the supplied archive. No external model files are required by this port. The added AdvSimd flag verifies the runtime capability only; Lynx's supplied source does not implement a dedicated NEON NNUE/search kernel. Dotprod is not claimed or forced.

## Build validation status

Static source checks, YAML parsing, Bash syntax checks, and the temporary PackageDownload injection/XML check were completed in this workspace. This environment does not contain the .NET SDK or NativeAOT compiler, so no native executable was built here. GitHub Actions must pass the restore, publish, AArch64 ELF, and artifact steps; the Android UCI smoke test must then pass on-device before Lynx is marked successful.
