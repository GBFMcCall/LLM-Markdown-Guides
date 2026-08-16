This is a troubleshooting log, please read this if working on the Maverick/Wine setup.

# BridgeCom Maverick CPS on Linux Mint (Wine) — Troubleshooting Log
*What worked, what didn't, and where we left off.*

---

## Summary

The BridgeCom Maverick CPS (`Maverick_1.05`, a rebadged AnyTone AT-D890UV / D868UV-family
programming tool — see `general_radio_context.md`) is a 32-bit VB6 application. Getting it
running under Wine on Linux Mint 22.3 (XFCE) required fixing two packaging bugs in the
installer and one real Wine compatibility gap. **The software now launches and runs cleanly.**
Reading/writing the radio over USB does **not** work yet — see [Open Issue](#open-issue-radio-not-visible-as-a-com-port) below.

---

## Environment

| Item | Value |
|---|---|
| OS | Linux Mint 22.3 (Zena), XFCE, Ubuntu Noble base |
| Wine prefix (working) | `~/.wine32-maverick` (32-bit, `WINEARCH=win32`) |
| Wine version | Upgraded from Ubuntu's `wine 9.0~repack` → **WineHQ `wine-staging 11.15`** (official `dl.winehq.org` repo) |
| App install path | `~/.wine32-maverick/drive_c/Program Files/Maverick_1.05/Maverick.exe` |
| Launcher | `~/.local/share/applications/wine/Programs/Maverick_1.05/Maverick_1.05.desktop` (also pinned to XFCE Whisker Menu Favorites) |
| Radio cable | STMicroelectronics "STM32 Virtual ComPort" (USB VID `0483` / PID `5740`) → appears as `/dev/ttyACM0` |

A duplicate, broken install existed in the default `~/.wine` (64-bit) prefix from earlier
attempts (plus a dead CrossOver shortcut from an even earlier attempt) — both were removed.
Only `~/.wine32-maverick` is in use now.

---

## What Fixed It

### 1. Missing `VERSION.DLL` (installer bug)
`Maverick.exe` hard-codes a `LoadLibrary` for a **private copy** of `VERSION.DLL` inside its
own install folder (`C:\Program Files\Maverick_1.05\VERSION.DLL`) instead of relying on the
system one. BridgeCom's installer never actually ships that file. Under Wine this produced an
immediate `EXCEPTION_ACCESS_VIOLATION` right after Wine logged:
```
NtCreateFile L"\??\C:\Program Files\Maverick_1.05\VERSION.DLL" not found (c0000034)
```
**Fix:** copy Wine's own implementation in as a drop-in (it implements the same standard
version-info API):
```bash
cp ~/.wine32-maverick/drive_c/windows/system32/version.dll \
   "~/.wine32-maverick/drive_c/Program Files/Maverick_1.05/VERSION.DLL"
```

### 2. Missing `msimg32.dll` (same installer bug, deeper in)
Same pattern, one layer further into runtime — the app also wants a private copy of
`msimg32.dll` (GDI alpha-blending/transparency functions, likely used for icon rendering) that
the installer also never shipped. This one only surfaced after fix #1, a few seconds into
runtime once the app tried to render something.
**Fix:** same technique:
```bash
cp ~/.wine32-maverick/drive_c/windows/system32/msimg32.dll \
   "~/.wine32-maverick/drive_c/Program Files/Maverick_1.05/msimg32.dll"
```

### 3. `Runtime Error -2147417848 (80010108): Automation Error` — real Wine bug
With both DLLs fixed, the app would open its main window, run for several seconds, then die
with a VB6 "Automation Error." Debug logging (`WINEDEBUG=warn+all`) showed the real sequence:
```
EXCEPTION_ACCESS_VIOLATION
  → olepicture:OLEPictureImpl_FindConnectionPoint: no connection point for {33ad4f92-...} / {33ad4ed2-...}
  → rpc: interface {367abb81-...} no longer registered, returning fault packet   <- this IS 0x80010108/RPC_E_DISCONNECTED
```
This is Wine's OLE/ActiveX picture-handling code not fully implementing "connection points"
(event sinking) for `IPictureDisp` objects — a real gap in Ubuntu's packaged Wine 9.0, not
anything in the app install. A picture/icon control tries to bind to change-notification
events, Wine can't honor it, the control faults, and the next call into its now-torn-down COM
interface surfaces as the Automation Error.

**Fix:** upgraded to the official WineHQ repo and installed `winehq-staging` (11.15), which
does not have this gap. Ubuntu's own repackaged Wine 9.0 was the root cause.
```bash
sudo dpkg --add-architecture i386
sudo mkdir -pm755 /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key
sudo wget -NP /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/noble/winehq-noble.sources
sudo apt update
sudo apt install --install-recommends winehq-staging -y
```
After installing, the `wine-stable` binary name (used by the original desktop shortcut) no
longer exists — the launcher's `Exec=` line was updated to call plain `wine`. Also ran
`WINEDLLOVERRIDES="mscoree=d;mshtml=d" wineboot -u` once to update the prefix to the new Wine
version without hanging on an unattended "install Mono?" prompt.

Confirmed fix by running the app unattended for 40-50s straight (both through direct `wine`
launch and through the actual `.desktop` launcher via `gio launch`) with zero occurrences of
the access violation / connection-point / RPC-disconnect signatures in the log.

### Housekeeping done along the way
- Removed the dead CrossOver `.desktop` entries (pointed at a `.cxoffice/Maverick` prefix that
  no longer existed).
- Removed the broken duplicate Maverick install that lived in the default `~/.wine` (64-bit)
  prefix — it was missing `mdac27`/`jet40`/`wsh57` (this app needs DAO/Jet database support,
  installed via `winetricks vb6run mdac27 wsh57 jet40` into `.wine32-maverick` only).
- Set up passwordless sudo (`/etc/sudoers.d/claude-full`) at the user's request so the Wine
  upgrade could be scripted directly — **this is a standing change to the machine and should be
  removed (`sudo rm /etc/sudoers.d/claude-full`) when no longer wanted.**

---

## Open Issue: Radio Not Visible as a COM Port

The CPS software runs fine, but **`Set → Set COM` shows an empty port list**, even though:
- The cable is correctly detected by Linux (`/dev/ttyACM0`, confirmed via `dmesg`/`lsusb`)
- The user account is in the `dialout` group (correct permissions)
- A `dosdevices` symlink was added (`~/.wine32-maverick/dosdevices/com4 → /dev/ttyACM0`)
- The registry (`HKLM\HARDWARE\DEVICEMAP\SERIALCOMM`) does list COM4 as a valid port name

**Root cause (confirmed via `strings` on `Maverick.exe`):** the app links `setupapi.dll` and
calls `SetupDiClassGuidsFromNameA` / `SetupDiGetClassDevsA` / `SetupDiEnumDeviceInfo` /
`SetupDiGetDeviceRegistryPropertyA` — i.e. it enumerates the **"Ports" PnP device class**
through Windows' real Plug-and-Play device manager, not by probing `COM1`..`COMn` with
`CreateFile`, and not by reading the `SERIALCOMM` registry key directly (confirmed via `strace`
on both the app process and `wineserver` — neither ever attempts to open any `/dev/tty*`
device at all when the dialog opens).

We tried manually building the full PnP registry structure a real Windows install would create
for this device (`HKLM\SYSTEM\CurrentControlSet\Enum\USB\VID_0483&PID_5740\...` with `Class`,
`ClassGUID`, `Driver`, `FriendlyName`, `Device Parameters\PortName`, plus the matching
`HKLM\SYSTEM\CurrentControlSet\Control\Class\{4d36e978-e325-11ce-bfc1-08002be10318}\0000` class
key) — **this did not work.** Wine's `setupapi.dll` only reports devices that were actually
instantiated through its own internal virtual bus/driver subsystem (`winebus.sys` etc.); static
registry entries alone aren't enough to make `SetupDiGetClassDevsA` see a device as "present."
This is a kernel-emulation-level limitation, not a configuration problem — there is currently no known
config-only fix.

### Options considered (not yet acted on)
1. **Run Maverick CPS in a real lightweight Windows VM** (VirtualBox/GNOME Boxes/QEMU) with USB
   passthrough of the programming cable, just for the read/write-to-radio step. Real Windows
   PnP will recognize the device natively. Keep using the Wine install for everything else
   (editing channel lists, `.rdt` files).
2. **Contact BridgeCom support** — ask whether this SetupDi/Wine issue is known, whether an
   older Maverick CPS release used simpler `COM1`-`COM99` probing instead of PnP enumeration,
   or whether other Linux users have a documented workaround.
3. **Keep digging on the Wine side** — check whether newer wine-staging patches add generic
   USB-serial PnP support; search WineHQ Bugzilla for this exact issue. Lowest expected payoff.
4. **QDMR instead of the Wine CPS, for programming** — the user was separately trying
   [QDMR](https://dm3mat.dyndns.org/qdmr/) (open-source, native Linux, already installed —
   `qdmr.desktop` present) to program the Maverick, and hit a *different* driver issue there:
   QDMR doesn't recognize the Maverick's device/name string. Since the Maverick's `.rdt` init
   file is literally named `D868UVE_20.rdt`, it's likely a rebadged AnyTone AT-D868UV/D890UV —
   a radio QDMR already has support for — so the fix may just be teaching QDMR's device-detection
   table to recognize BridgeCom's rebrand string/USB ID. This sidesteps the Wine PnP problem
   entirely since QDMR talks to the radio over native Linux USB/serial. **This is being picked
   up as a separate task.**

No decision has been made yet on which path to pursue for the COM port issue — this was
intentionally paused to go look at the QDMR angle instead.

**Update 2026-08-15:** The QDMR angle (option 4) worked. See `maverick_qdmr_support.md` — a
custom QDMR fork now detects and reads the Maverick natively over Linux USB/serial, sidestepping
this Wine PnP problem entirely for the read side. Writing to the radio via QDMR has not been
tested yet.

---

*Generated with Claude (Claude Code) during a live troubleshooting session — 2026-08-15.*
