# Issue #96 - Windows and Linux Designer sitting

Working checklist for one sitting per OS. Built from issue #96,
`docs/QA-CHECKLIST.md` (the *(Windows, Linux)* rows, "What is still
unchecked" and "Compare against a relaunched Designer before calling something
a bug") and the README's "Known limitations". Results go in
[RESULTS.md](RESULTS.md).

## 1. Build under test

`feature/maker-edition-support` on the fork: `c406553` = upstream `main`
`2d97196` plus the Maker Edition commit (PR #155). Without that commit the
module does not load on an 8.3.9 Maker gateway.

### Build (Ubuntu build machine, JDK 17 Temurin)

```bash
java -version                        # must report 17
git clone https://github.com/aott33/ignition-designer-dark-mode-module.git ddm-qa
cd ddm-qa
git checkout feature/maker-edition-support
git log -1 --format='%h %s'          # expect: c406553 Declare Ignition Maker Edition compatibility
```

If more than one JDK is installed, point Gradle at 17 first:
`export JAVA_HOME=/usr/lib/jvm/temurin-17-jdk-amd64` (adjust to your path).

**Option A - signed with a throwaway certificate (recommended).** The gateway
asks you to accept the certificate once, the same flow as a release. No
gateway setting to change.

```bash
mkdir -p ~/ddm-qa-signing
keytool -genkeypair -alias ddm-qa -keyalg RSA -keysize 2048 -validity 365 \
  -dname "CN=Designer Dark Mode QA build" \
  -keystore ~/ddm-qa-signing/qa.p12 -storetype PKCS12 -storepass qa-pass
keytool -exportcert -alias ddm-qa -keystore ~/ddm-qa-signing/qa.p12 \
  -storetype PKCS12 -storepass qa-pass -rfc -file ~/ddm-qa-signing/qa.pem

./gradlew clean build \
  -Pignition.signing.keystoreFile=$HOME/ddm-qa-signing/qa.p12 \
  -Pignition.signing.keystorePassword=qa-pass \
  -Pignition.signing.certFile=$HOME/ddm-qa-signing/qa.pem \
  -Pignition.signing.certAlias=ddm-qa \
  -Pignition.signing.certPassword=qa-pass

ls -l build/designer-dark-mode.modl
sha256sum build/designer-dark-mode.modl   # record it in RESULTS.md
```

**Option B - unsigned.** `./gradlew clean build` gives
`build/designer-dark-mode.unsigned.modl`. Use it only if your gateway already
accepts unsigned modules (as it did if you installed your #155 test build
unsigned).

These are the same Gradle properties `ops/lib.sh` passes for the dev
gateway.

### Install on the 8.3.9 Maker gateway

1. Gateway web UI → **Config → Modules** → **Install or Upgrade a Module** →
   pick the `.modl`.
2. Accept the certificate (`CN=Designer Dark Mode QA build`) and the licence.
   If a released build is installed, this replaces it and asks again for the
   new certificate.
3. The module list must show **Designer Dark Mode** running, version
   `0.0.1-SNAPSHOT`. It must not show "Not eligible for use with Ignition
   Maker Edition".
4. Afterwards, install the next release over it to go back.

Fallback if the Maker gateway is not available: a Standard dev gateway on the
build machine with `ops/setup.sh` (8.3.6 in Docker, port 8088). It builds
whatever is checked out. Record which gateway you used.

## 2. One-time setup, both OSes

- **Designer Launcher** → select the gateway → **Edit** → *Additional JVM
  Arguments*: `-Ddesignerdarkmode.debug=true`. It persists across launches.
- **Project** with at least **two Perspective views**, each with one
  component (a Label is enough), so the property editor has rows and there
  is something to switch between.
- **Only the display under test switched on** (Windows: `Win+P` → *PC screen
  only* or *Second screen only*). The `env:` block reads the **primary**
  screen, not the one the Designer is on. With both screens on, a Designer on
  the Dell still logs the panel's 2.5.
- **Relaunch the Designer after every scale or display change.** The `env:`
  block is written once per Designer session, at its first theme switch.
- **Screenshots:** PNG crops at native size, never resized or JPEG. I read
  colours from the pixels. Windows: `Win+Shift+S`, or Snipping Tool → Record
  for the flash in row 11. Ubuntu: `PrtSc`.
- **Leave Dark Mode off when you quit** at the end of each run, so the next
  launch starts stock and gives a stock baseline.

### Pulling the log

Windows (PowerShell):

```powershell
$log = "$env:USERPROFILE\.ignition\designer-dark-mode.log"
(Select-String $log -Pattern 'env: ' | Select-Object -Last 8).Line
(Select-String $log -Pattern 'switching to|before switch|after switch|failing|failed' | Select-Object -Last 10).Line
Get-Content $log -Tail 40      # right after an inspector dump
```

Linux:

```bash
log=~/.ignition/designer-dark-mode.log
grep 'env: ' "$log" | tail -8
grep -E 'switching to|before switch|after switch|failing|failed' "$log" | tail -10
tail -40 "$log"                # right after an inspector dump
```

Linux session facts (once per run):

```bash
echo "session=$XDG_SESSION_TYPE desktop=$XDG_CURRENT_DESKTOP"
gsettings get org.gnome.mutter experimental-features
gsettings get org.gnome.desktop.interface text-scaling-factor
xrdb -query 2>/dev/null | grep -i dpi
```

Ubuntu 25.10 and later ship GNOME without an Xorg session, so expect
`wayland`. The Designer's Java 17 runs through XWayland either way.

Paste raw output. I strip usernames and paths before anything goes in
RESULTS.md.

## 3. Deliberately light - do not report

- **Native title bar and window frame** (Windows, Linux). Light by decision
  (`skip`).
- **Perspective view canvas**. It follows the session's theme.
- **Perspective view editor rulers and surround**. Known and undecided.
  Note it, do not file it.
- **Symbol Factory thumbnails**. They are the symbol artwork.
- **Colour value swatches** (a colour property's swatch). They are values,
  not chrome.
- **The Designer Launcher, the login window, and the brief stock Designer
  at launch** before the theme applies. By design.
- **OS file-type icons inside a Swing file chooser**. They are the
  platform's own icons.
- **Two `NullPointerException`s in the Output Console on each switch back to
  light** (`TreeCollapsedIconPainter` / `TreeExpandedIconPainter`). Known,
  #61, nothing on screen.

## 4. Before calling anything a bug

This follows the "Compare against a relaunched Designer" section of the QA
checklist.

1. Is it on the list above? Then stop.
2. Compare with the **stock baseline** from step A (same display, same
   scale). If the stock Designer looks the same, it is Ignition's own
   styling, not the module's.
3. **Quit and relaunch** with Dark Mode off. Turn it on **once** and go
   straight to the surface. Still wrong? Hover it, press **Ctrl+Shift+I**,
   and send the log tail. If it only appears after several toggles, say how
   many.
4. Only then is it a finding. It gets an F-number in RESULTS.md, and an
   issue draft if it holds up.

For something that looks off **after switching back to light**, do the same
comparison against a relaunched stock Designer. If they are identical, there
is nothing to fix.

## 5. Full walk: 100 %, Dell only

Same rows on both OSes. **Windows first.** About 40 minutes.

**A. Stock baseline** (Designer launched with Dark Mode off). Take crops of
rows 3-10 and 12 while light. These are the relaunched-stock reference for
every comparison later.

**B. Switch on.** Tools → Dark Mode.

| # | Row | Where / how | Pass looks like |
|---|---|---|---|
| 1 | Switch | Tools → Dark Mode | Whole Designer dark in about a second. "Applying dark mode…" flashes in the status bar, and no "N of M steps failing" appears |
| 2 | `env:` block | Log commands above | `java.desktop access: 8 of 8 required packages reachable`, no `MISSING`. `screen.transform` matches the display under test (1.0 here). The `before switch: font` and `after switch: font` lines match. This answers #96's open question about the launcher's `--add-opens` |
| 3 | In-window menu bar | File … Help across the top of the main frame | Dark bar. Titles legible. Hovering a title highlights it dark. The open menu's title highlighted dark, not light |
| 4 | Menu popups | Open each menu, hover down it. Also one right-click menu (Project Browser tree) | Hover highlight is dark (#93: it was the light Windows blue `#ABDAFF`). Accelerator text (e.g. Ctrl+S) legible and not clipped. Checkmarks visible (Tools → Dark Mode, View → Panels). Disabled items grey but readable |
| 5 | Mnemonics | Press and release **Alt**, then Alt+F | Underlines appear under the menu letters and in the open popup, legible on dark. Alt+F opens File. If underlines only show while Alt is used and the stock baseline shows them always, that is FlatLaf's convention: a note, not a bug |
| 6 | Status bar | Bottom edge of the main frame | Dark, text legible (#93: was `#F0F0F0`) |
| 7 | Section headers | Property editor. With nothing selected: SESSION PROPS etc. With a component selected: PROPS / POSITION / CUSTOM / META | Header strips dark, labels legible, collapse arrows visible (#93) |
| 8 | Selected dock tab | Any dock with two panels tabbed together (drag Tag Browser onto Project Browser if needed). The tabs at the dock's bottom edge | Selected tab dark, text legible (#93: was light) |
| 9 | Side-pane buttons | Auto-hide a dock panel (pin button, or right-click its title → Auto Hide). The button on the frame edge; hover it; open it | Button dark, text legible, hover and open states dark (#93: were light). Un-hide it after |
| 10 | Workspace tab strip | Open both views. The tabs along the **bottom edge** of the workspace | Selected tab dark with legible text (#81: Windows painted it `#E3E3E3` with light text. Fixed headlessly, never seen by eye on Windows) |
| 11 | Property editor on view switch | Select a component, then switch between the two views with the row-10 tabs, 5 times. Also open a view for the first time this session from the Project Browser | Rows never paint light, not even for a frame. If they flash: video it (Snipping Tool → Record) and say how long (a blink, or about half a second), whether it happens every switch or only the first, and which display. Seen on Windows before the latest watcher changes; this re-checks it on current main |
| 12 | Swing file chooser | File → Import… (cancel). Image Management → Upload. Tag Browser menu → Export | Dark panel and file list, legible names, "Look in" and "Files of type" dropdowns dark when opened, buttons dark. If the dialog is native (Windows Explorer, or a GTK/portal dialog on Linux): record `skip (native)`. Test: Ctrl+Shift+I over it logs nothing for a native dialog |
| 13 | Native title bar and frame | Around every window | Light on Windows. Linux: whatever GNOME draws. `skip` either way, recorded not filed |
| 14 | Surfaces opened while dark | View → Panels → Tag Browser off, then on again. View → Panels → OPC Browser, expand one level | Both come up dark. Log: `TreeIconRecolorer: wrapped N tree renderer(s)` with a rising N |
| 15 | Toggle off | Tools → Dark Mode off | Rows 3-10 match the step A baseline, including Windows' own status bar grey and blue menu hover. Tag Browser and OPC Browser from row 14 light, with their icons and row heights |
| 16 | Relaunch | Turn Dark Mode on, quit, relaunch | Comes up dark (`switching to dark` at startup). Repeat row 11 in this started-dark session. Then turn Dark Mode off and quit |
| 17 | Size (#76) | Same crop of the Project Browser and a menu in stock (step A) and dark (step B) | Same text size, row height and text width in both. The `before switch` and `after switch` font lines agree |

## 6. Scaled runs (key rows only)

Each run: set the scale → relaunch the Designer (starts stock) → crop the
Project Browser and a menu → Dark Mode on → rows **2, 3-4, 6-10, 11, 17** →
row **15** → quit with Dark Mode off. About 10 minutes each.

| Run | OS | Display | Scale | Notes |
|---|---|---|---|---|
| W-125 | Windows | Dell | 125 % | Settings → System → Display → Scale |
| W-150 | Windows | Dell | 150 % | |
| W-250 | Windows | Laptop panel only | 250 % | `Win+P` → PC screen only. Your earlier 2.5 block came from this panel; this run ties one to the test build |
| L-125 | Linux | Dell | 125 % | Settings → Displays → Scale (turn on Fractional Scaling if 125 % is missing). Run the session-facts commands |
| L-150 | Linux | Dell | 150 % | Same |

What to look for at a scaled setting:

- **Windows:** expect `screen.transform` 1.25 / 1.5 / 2.5.
- **Linux (XWayland):** Java 17 does not follow GNOME's fractional scale. Expect
  `screen.transform=1.0`, and either a blurry, compositor-upscaled Designer or
  a small, unscaled one. Neither is the module's doing.
- **The #76 question in both cases:** is dark ever smaller than stock at the
  same setting? That is row 17.

Optional, Windows, if time: with both screens on, drag the Designer from one
display to the other while dark. Report anything that breaks. The `env:`
block will not describe this case.

## 7. Not in this sitting

Vision (§F, §N), Reporting (§H), the §L popup sources beyond row 4, the
Tag Editor override icons (#135/#136), and an 8.1 gateway. The write-up will
say so.
