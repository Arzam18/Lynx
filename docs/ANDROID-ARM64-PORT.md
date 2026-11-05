# Lynx Android ARM64 NativeAOT port

## Source and output

- Source revision supplied for this port: Lynx 1.11.0 (`Directory.Build.props`).
- UCI executable project: `src/Lynx.Cli/Lynx.Cli.csproj`.
- Core engine project: `src/Lynx/Lynx.csproj`.
- Build target: `linux-bionic-arm64`, a .NET NativeAOT Bionic target for Android ARM64.
- The workflow builds the CLI only. It does not build tests, benchmarks, developer tools, a GUI, or an APK.
- `appsettings.json` is copied alongside the executable and is required by this port's packaging.
- No `.nnue`, `.weights`, or `.net` resource files are present in the supplied source archive. Lynx uses its existing source evaluation, PSQT, and search implementation; no network file was added or replaced.

## NativeAOT toolchain

The workflow follows the working Sapling Android NativeAOT path:

- Ubuntu 24.04 x86_64 build runner.
- .NET SDK 10.0.114 and patched .NET 10.0.14 compiler/runtime packs.
- Android NDK r27c, with its LLVM toolchain placed first on `PATH`.
- RID `linux-bionic-arm64`.
- `PublishAot=true`, `PublishAotUsingRuntimePack=true`, `SelfContained=true`, and `DisableUnsupportedError=true`.
- The workflow injects `Microsoft.NETCore.App.Runtime.NativeAOT.linux-bionic-arm64` version 10.0.14 into the temporary Actions checkout for restore. It does not permanently add an Android package dependency to Lynx's project files.

.NET's Android Bionic NativeAOT path produces a command-line executable, not an Android APK or managed Android application. The Android runtime identifier is `linux-bionic-arm64`, not `android-arm64`.

## CPU and optimization audit

The supplied Lynx source contains guarded x86-specific optimizations:

- BMI2/PEXT slider attacks are used only when `Bmi2.X64.IsSupported` (or `Bmi2.IsSupported`) is true. Otherwise the existing magic-bitboard attack functions are used.
- BMI1 `ExtractLowestSetBit` is used only when supported. Otherwise the existing arithmetic least-significant-bit extraction remains active.
- SSE prefetch is used only when `Sse.IsSupported`; otherwise TT prefetch is skipped.
- SSE4.2, AVX2, and AVX-512 are reported as CPU flags but are not forced at compile time.

The port does not add an x86 ISA flag, a CPU-specific ARM target, Dotprod, or a new SIMD search/evaluation implementation. `runtimeconfig` now also reports `ARM AdvSimd (NEON) = True/False`, which lets the phone report the .NET runtime's ARM SIMD capability. This flag is a capability diagnostic, not proof that the current Lynx search contains a handwritten NEON kernel. The x86 PEXT path remains unavailable on ARM and falls back to magic bitboards. .NET exposes `AdvSimd.IsSupported` as the runtime feature check for ARM Advanced SIMD. [Microsoft API reference](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.intrinsics.arm.advsimd.issupported?view=net-10.0)

No search heuristics, evaluation values, thread counts, or engine options were changed. The existing default `Threads` setting in `appsettings.json` is 1.

## Device smoke test

On the Samsung Galaxy A16 5G, the expected diagnostic is `ARM AdvSimd (NEON) = True`; x86 BMI/SSE/AVX flags should report false. Send `uci`, `isready`, `runtimeconfig`, `position startpos`, then `go depth 10`. Require `uciok`, `readyok`, a `bestmove`, and stable execution. The supplied `docs/ANDROID-TERMUX-TEST.md` provides a single Termux test command.

This workspace has no .NET SDK/NativeAOT compiler installed, so the actual publish must be completed by the GitHub Actions workflow. The workflow's AArch64 ELF validation and the phone-side UCI test are separate acceptance gates.
