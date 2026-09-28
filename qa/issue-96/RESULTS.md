# Issue #96 - results

Running record for the sitting in [CHECKLIST.md](CHECKLIST.md). Usernames and
local paths are stripped before anything is written here.

Codes: `pass` dark and legible · `light` leaks light (says where) · `broken`
does not work, clips, or is unreadable · `skip` deliberately not themed ·
`n/a` does not exist here · `-` not checked yet.

## Build and gateway

| | |
|---|---|
| Module | `feature/maker-edition-support` @ `c406553` (upstream `main` `2d97196` + #155), version `0.0.1-SNAPSHOT` |
| Signing | - |
| `.modl` sha256 | - |
| Gateway | Ignition 8.3.9 Maker Edition |
| Perspective | - |

## Runs

| Run | Date (UTC) | OS | Display | Scale | Session | Java | Scope |
|---|---|---|---|---|---|---|---|
| W-100 | - | Windows 11 | Dell P2425H 1920x1080 | 100 % | - | 17.0.19 (Azul, launcher) | full walk |
| W-125 | - | Windows 11 | Dell | 125 % | - | | key rows |
| W-150 | - | Windows 11 | Dell | 150 % | - | | key rows |
| W-250 | - | Windows 11 | Laptop panel 3840x2400 | 250 % | - | | key rows |
| L-100 | - | Ubuntu 26.04.1 LTS | Dell | 100 % | - | | full walk |
| L-125 | - | Ubuntu 26.04.1 LTS | Dell | 125 % | - | | key rows |
| L-150 | - | Ubuntu 26.04.1 LTS | Dell | 150 % | - | | key rows |

## Results

Blank cells are rows that are not part of that run.

| # | Row | W-100 | W-125 | W-150 | W-250 | L-100 | L-125 | L-150 |
|---|---|---|---|---|---|---|---|---|
| 1 | Switch | - | | | | - | | |
| 2 | `env:` block | - | - | - | - | - | - | - |
| 3 | In-window menu bar | - | - | - | - | - | - | - |
| 4 | Menu popups: hover, accelerators, checkmarks | - | - | - | - | - | - | - |
| 5 | Mnemonics | - | | | | - | | |
| 6 | Status bar | - | - | - | - | - | - | - |
| 7 | Section headers | - | - | - | - | - | - | - |
| 8 | Selected dock tab | - | - | - | - | - | - | - |
| 9 | Side-pane buttons | - | - | - | - | - | - | - |
| 10 | Workspace tab strip | - | - | - | - | - | - | - |
| 11 | Property editor on view switch | - | - | - | - | - | - | - |
| 12 | Swing file chooser | - | | | | - | | |
| 13 | Native title bar and frame | - | | | | - | | |
| 14 | Surfaces opened while dark | - | | | | - | | |
| 15 | Toggle off | - | - | - | - | - | - | - |
| 16 | Relaunch (started dark) | - | | | | - | | |
| 17 | Size: dark vs stock (#76) | - | - | - | - | - | - | - |

## `env:` blocks

One per run, with the display the Designer was on and the `before switch` /
`after switch` font lines.

_None yet._

## Findings

Each finding: what was seen, run and display, the relaunched-stock
comparison, and the inspector chain if light. Status is one of `open`
(comparison not done yet), `not ours` (stock looks the same), `known` (on the
deliberately-light list or already filed), or `bug` (issue draft below).

_None yet._

## Not tested

- Vision, Reporting, the §L popup sources beyond row 4, the Tag Editor override
  icons (#135/#136), an 8.1 gateway.
