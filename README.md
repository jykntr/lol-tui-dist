# lol-tui-dist

Public release binaries for **lol-tui**, a terminal UI that shows real-time
League of Legends game intelligence — CS/min tracking, a kill/objective event
feed, and per-player rank, role, and pre-made party detection.

This repository contains **only build artifacts**. The source lives in a private
repository; releases here are mirrored automatically so the app's self-updater
has something public to read.

## Downloads

Grab the archive for your platform from the
[latest release](https://github.com/jykntr/lol-tui-dist/releases/latest):

| Platform | Asset | Notes |
|---|---|---|
| macOS, Apple Silicon (M1/M2/M3/M4) | `lol_tui-macos-aarch64.tar.gz` | Requires Homebrew Tesseract — see below |
| Windows 10/11, 64-bit | `lol_tui-windows-x86_64.zip` | Self-contained |

**Intel Macs and Linux are not supported.** No binaries are published for them;
running `--update` on those platforms reports that clearly rather than failing
oddly. Building from source is the only option there.

Each archive contains a single executable at its root — `lol_tui` on macOS,
`lol_tui.exe` on Windows.

---

## macOS

### 1. Install the Tesseract runtime

The macOS binary is dynamically linked against Homebrew's Tesseract and
Leptonica libraries, which power the CS scoreboard OCR. Without them the app
will not launch at all:

```bash
brew install tesseract
```

That one command is enough — Homebrew pulls in Leptonica and the rest of the
dependency chain automatically. If you don't have Homebrew, install it from
[brew.sh](https://brew.sh) first.

### 2. Download and extract

Downloading with `curl` from Terminal is the smoothest path, because Gatekeeper's
quarantine flag is applied by the *downloading application* — browsers set it,
`curl` does not. Do this and you skip the "unverified developer" dance entirely:

```bash
cd ~/Downloads
curl -L -o lol_tui.tar.gz \
  https://github.com/jykntr/lol-tui-dist/releases/latest/download/lol_tui-macos-aarch64.tar.gz
tar -xzf lol_tui.tar.gz
```

If you'd rather download through your browser, extract the archive from Terminal
with `tar -xzf` rather than double-clicking it. Finder's Archive Utility copies
the quarantine flag onto the extracted binary; `tar` does not.

### 3. Install it somewhere on your PATH

```bash
mkdir -p ~/.local/bin
mv lol_tui ~/.local/bin/
chmod +x ~/.local/bin/lol_tui
```

Add `~/.local/bin` to your PATH if it isn't already — put this in `~/.zshrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Prefer a directory you own (like `~/.local/bin`) over `/usr/local/bin`. The
self-updater replaces the binary in place, so a user-owned location means
`--update` never needs `sudo`.

### 4. If macOS blocks it anyway

If you downloaded through a browser or extracted with Finder, the first run will
fail with a dialog like *"lol_tui cannot be opened because the developer cannot
be verified"* or *"Apple could not verify lol_tui is free of malware."* This is
expected: the binary is unsigned and unnotarized, not broken.

First, check whether the quarantine flag is actually present:

```bash
xattr -l ~/.local/bin/lol_tui
```

If you see `com.apple.quarantine`, remove it:

```bash
xattr -d com.apple.quarantine ~/.local/bin/lol_tui
```

That's the reliable fix for a command-line binary. Alternatively, run the app
once, let it get blocked, then open **System Settings → Privacy & Security**,
scroll to the Security section, and click **Open Anyway** next to the message
about `lol_tui`. Confirm with Touch ID or your password, then run it again.

> Right-click → Open, the classic workaround, applies to `.app` bundles and no
> longer bypasses Gatekeeper on recent macOS versions. Use `xattr` or the
> Privacy & Security panel instead.

Once cleared, the approval sticks. It also does not come back after a
self-update: `--update` fetches the new binary over the network directly rather
than through a browser, so nothing re-applies the quarantine flag.

### 5. Run it

```bash
lol_tui
```

---

## Windows

### 1. Download and extract

Download `lol_tui-windows-x86_64.zip` from the
[latest release](https://github.com/jykntr/lol-tui-dist/releases/latest) and
extract it — for example to `C:\Users\<you>\bin`. The zip contains a single
`lol_tui.exe`. No extra runtime libraries are needed; the OCR dependencies are
compiled in.

### 2. Clear the Mark of the Web

Windows tags files downloaded from the internet, and Microsoft Defender
SmartScreen blocks unrecognized ones. Because this binary isn't code-signed with
a certificate Microsoft recognizes, you'll likely hit:

> **Windows protected your PC** — Microsoft Defender SmartScreen prevented an
> unrecognized app from starting.

Click **More info**, then **Run anyway**.

To clear the flag ahead of time instead, right-click the **zip file** before
extracting, choose **Properties**, tick **Unblock** at the bottom of the General
tab, and click OK. Unblocking the zip first means the extracted `.exe` comes out
clean. If you already extracted it, do the same on `lol_tui.exe` itself.

The PowerShell equivalent:

```powershell
Unblock-File -Path C:\Users\<you>\bin\lol_tui.exe
```

### 3. Add it to your PATH (optional)

```powershell
[Environment]::SetEnvironmentVariable(
  "Path",
  [Environment]::GetEnvironmentVariable("Path", "User") + ";C:\Users\<you>\bin",
  "User"
)
```

Open a new terminal afterward for the change to take effect.

### 4. Run it

```powershell
lol_tui.exe
```

Run it from Windows Terminal or PowerShell for correct colors and Unicode. The
legacy `cmd.exe` console renders the UI poorly.

---

## Self-update

lol-tui updates itself from this repository. No token or GitHub account needed —
that's the whole reason this public mirror exists.

### Updating

```bash
lol_tui --update
```

This checks for a newer release, downloads the asset for your platform, shows a
progress bar, and replaces the running executable in place. When it finishes:

```
Updated to v1.3.0. Restart lol-tui to use it.
```

Restart the app to pick up the new version. If you're already current, it says so
and exits without downloading anything.

Run `--update` from a normal shell, not while the TUI is open — it prints to
stdout and prompts for confirmation before replacing the binary.

### The startup notice

On launch, lol-tui quietly checks whether a newer release exists. If so, the
status bar at the bottom of the screen shows:

```
update available: v1.3.0 — run lol-tui --update
```

The check is network-only and never downloads anything on its own; updates are
always explicit. If the check fails — no network, GitHub unreachable — it's
silently ignored and the app starts normally.

To turn the check off for a single run:

```bash
lol_tui --no-update-check
```

To turn it off permanently, set this in your config file:

```toml
update_check = false
```

The config file lives at:

| Platform | Path |
|---|---|
| macOS | `~/Library/Application Support/lol-tui/config.toml` |
| Windows | `%APPDATA%\lol-tui\config.toml` |
| Linux | `~/.config/lol-tui/config.toml` |

### Versions before 1.3.0

Self-update landed in **v1.3.0**. Earlier builds have no `--update` flag, so
upgrading from one of those takes a single manual download — follow the install
steps above, overwriting the old binary. Every version after that can update
itself.

---

## Troubleshooting

**macOS: `dyld: Library not loaded: /opt/homebrew/opt/tesseract/lib/libtesseract.5.dylib`**
Tesseract isn't installed, or Homebrew has moved to a newer major version of the
library. Run `brew install tesseract`; if it's already installed, `brew upgrade
tesseract` and then update lol-tui so both sides line up.

**macOS: "cannot be opened because the developer cannot be verified"**
The quarantine flag. See [If macOS blocks it anyway](#4-if-macos-blocks-it-anyway).

**macOS: `zsh: permission denied: lol_tui`**
The executable bit was lost in extraction. Run `chmod +x lol_tui`.

**`--update` fails with "permission denied"**
The binary sits somewhere your user can't write, typically `/usr/local/bin` or
`C:\Program Files`. Move it to a directory you own (`~/.local/bin`,
`C:\Users\<you>\bin`) and re-run, or re-run with elevated privileges.

**`--update` reports "The latest release has no asset for this platform"**
A release was just tagged and its binaries are still building — the mirror
publishes assets a few minutes after the tag. Wait and try again.

**Windows: "VCRUNTIME140.dll was not found"**
Install the [Microsoft Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe).

**The app sits on "Waiting for game..."**
That's the idle state. lol-tui watches the League Live Client API on
`127.0.0.1:2999`, which only exists while a game is actually in progress —
champion select and the client lobby don't count. It picks up the game
automatically once loading starts.
