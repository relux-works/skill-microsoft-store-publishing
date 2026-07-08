# Windows GUI build gotchas (Go)

Lessons from shipping a pure-syscall Go Win32 GUI app (CGO_ENABLED=0). These
bite on the first real-hardware run; cert will flag some of them.

## A console window pops up — and closing it kills the GUI

A Go program defaults to the **console subsystem**: launching the GUI opens a
terminal alongside it, and closing that terminal terminates the whole process.
Build with the **windows-gui subsystem**:

```bash
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 \
  go build -trimpath -ldflags="-s -w -buildid= -H windowsgui" -o app.exe .
```

Do the same for any bundled child process (e.g. a helper daemon) so it doesn't
pop its own console.

## No console → log to a file

With `-H windowsgui` there is **no stderr**. `slog`/`log` output vanishes. Log
to a file so you can diagnose:

```go
f, _ := os.OpenFile(filepath.Join(cfgDir, "app.log"),
    os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0o644)
w := io.MultiWriter(os.Stderr, f) // stderr too, for CLI runs
log := slog.New(slog.NewTextHandler(w, &slog.HandlerOptions{Level: slog.LevelDebug}))
```

`%APPDATA%\<App>\app.log` is the natural place. This log is what you ask a
tester to send you.

## Raw Win32 looks like Windows 3.1 — set Segoe UI

Controls created with `CreateWindowExW` default to an **ancient bitmap font**.
The single biggest modernization is setting **Segoe UI** (ClearType) on every
control:

```go
// CreateFontW(height<0 for point-ish, 0,0,0, weight, italic,underline,strike,
//   DEFAULT_CHARSET, 0,0, CLEARTYPE_QUALITY(5), 0, "Segoe UI")
font := createFont(-19, 400) // ~14px, FW_NORMAL
SendMessageW(control, WM_SETFONT /*0x30*/, font, 1)
```

For clean secondary text, handle `WM_CTLCOLORSTATIC` (0x0138): `SetBkMode`
TRANSPARENT + `SetTextColor` gray, return the window's brush.

## Text clips — size generously and DPI-aware

`STATIC` controls silently clip wrapped Cyrillic (or any long text). Give them
generous heights and a large-enough window; don't hand-tune to the pixel. Add
a **Per-Monitor-V2 DPI-aware manifest** so the window isn't tiny on hi-DPI.

## Icon + manifest via go-winres (keeps CGO=0)

Embed the app icon (multi-size `.ico`) and the DPI manifest as a Windows
resource with **go-winres**, generated in CI (don't commit the binary `.syso`):

```bash
go install github.com/tc-hib/go-winres@v0.3.3
go-winres make --in winres/winres.json --arch amd64 --product-version "$TAG"
# produces rsrc_windows_amd64.syso, auto-linked by `go build`
```

Load the embedded icon at runtime with `LoadIconW(hInst, MAKEINTRESOURCE / name)`
and set it on the window class and the tray (`NIF_ICON`).
