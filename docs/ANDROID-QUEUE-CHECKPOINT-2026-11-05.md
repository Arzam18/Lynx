# Android ARM64 chess-engine queue checkpoint

Date: 2026-11-05

## Current position

- Sapling — user confirmed Android NativeAOT build successful.
- Lynx — current engine; Android NativeAOT workflow and minimal runtime diagnostic port prepared from `Lynx-main.zip`.
- Booot — next after Lynx build and phone verification.

## Lynx acceptance gates

1. Run `.github/workflows/android-arm64.yml` with `workflow_dispatch`.
2. Confirm restore resolves `Microsoft.DotNet.ILCompiler` 10.0.14 and `Microsoft.NETCore.App.Runtime.NativeAOT.linux-bionic-arm64` 10.0.14.
3. Confirm publish produces `out/Lynx` and `out/appsettings.json`.
4. Confirm ELF machine is AArch64.
5. On the phone, run `runtimeconfig`; expected `ARM AdvSimd (NEON) = True`, with x86 BMI/SSE/AVX flags false.
6. Confirm `uci`, `isready`, and `go depth 10` produce `uciok`, `readyok`, and a legal `bestmove`.
7. Run deeper searches and at least one longer game before marking complete.

Do not call the engine successful based on an Actions compile alone; phone-side UCI and stability checks remain required.
