# Lynx Android ARM64 Termux test

Download and extract the `Lynx-Android-ARM64-NativeAOT-NEON` GitHub Actions artifact. Copy both `Lynx` and `appsettings.json` into:

`/storage/emulated/0/DroidFish/uci/Lynx/`

Then run this complete command in Termux:

```bash
pkg update -y
pkg install -y coreutils binutils file
termux-setup-storage

set -euo pipefail
SRC_DIR="/storage/emulated/0/DroidFish/uci/Lynx"
ROOT="$HOME/lynx-android-test"
BIN="$ROOT/Lynx"
LOG="$ROOT/uci-smoke-test.log"
mkdir -p "$ROOT"

if [ ! -s "$SRC_DIR/Lynx" ]; then
    echo "ERROR: Missing Android executable: $SRC_DIR/Lynx"
    exit 1
fi
if [ ! -s "$SRC_DIR/appsettings.json" ]; then
    echo "ERROR: Missing required sidecar: $SRC_DIR/appsettings.json"
    exit 1
fi

cp "$SRC_DIR/Lynx" "$BIN"
cp "$SRC_DIR/appsettings.json" "$ROOT/appsettings.json"
chmod 700 "$BIN"
cd "$ROOT"

file "$BIN"
sha256sum "$BIN"
readelf -h "$BIN" | grep -E 'Class:|Machine:'

{
    echo uci
    sleep 1
    echo isready
    sleep 1
    echo runtimeconfig
    sleep 1
    echo "position startpos"
    echo "go depth 10"
    sleep 30
    echo quit
} | timeout 40s "$BIN" 2>&1 | tee "$LOG"

grep -F 'uciok' "$LOG" >/dev/null
grep -F 'readyok' "$LOG" >/dev/null
grep -F 'ARM AdvSimd (NEON) = True' "$LOG" >/dev/null || {
    echo "ERROR: ARM AdvSimd/NEON was not reported. Inspect $LOG"
    exit 1
}
grep -E '^bestmove [a-h][1-8][a-h][1-8][qrbn]?' "$LOG" >/dev/null || {
    echo "ERROR: No legal-looking bestmove line found. Inspect $LOG"
    exit 1
}

echo "PASS: Lynx Android ARM64 UCI smoke test. Log: $LOG"
```

This is a runtime smoke test, not a strength test. Keep `Threads`, Hash, time control, and opening conditions matched when comparing the Android binary against other engines. Online tablebase access is disabled by default in `appsettings.json`.
