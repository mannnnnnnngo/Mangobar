<div align="center">

<img src="Mangobar.png" width="140" alt="MangoBar icon">

# 🥭 MangoBar

**A Windows-style taskbar for macOS — every open app in a row along the edge of the screen, and it can exclude the one pretending to be Java.**

Made by Mingyu 🧑‍💻

<br>

![macOS](https://img.shields.io/badge/macOS-13%2B-202020?style=for-the-badge&logo=apple&logoColor=white)
![Python](https://img.shields.io/badge/Python-PyObjC-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Version](https://img.shields.io/badge/version-1.1.0-7C5CFF?style=for-the-badge)
![Price](https://img.shields.io/badge/price-free-2EA043?style=for-the-badge)
![Permissions](https://img.shields.io/badge/screen%20recording-required-F09614?style=for-the-badge)
![Data](https://img.shields.io/badge/data%20sent%20anywhere-none-2EA043?style=for-the-badge)

</div>

---

> [!NOTE]
> **MangoBar is completely free.** 🆓 It is macOS only — it is PyObjC talking to AppKit, which
> doesn't exist anywhere else. The installed app is **self-contained**: it carries its own copy
> of Python, so there is nothing to install alongside it. 🐍

> [!IMPORTANT]
> **1.1.0 fixes MangoBar not opening.** Every release before this one shipped an app bundle that
> pointed at the *build machine's* source folder — `/Users/runner/work/Mangobar` on the GitHub
> runner that built it. That path exists on nobody else's Mac, so the app launched, failed to
> import itself, and died without drawing anything. It now carries its source, its packages and a
> whole Python interpreter inside the bundle, and refers to nothing outside it.

---

## 📖 Contents

| | | |
| --- | --- | --- |
| [🧐 Why this exists](#-why-this-exists) | [📥 Install](#-install) | [👀 Using it](#-using-it) |
| [🚫 Excluding apps](#-excluding-apps) | [🧩 Layout](#-layout) | [🔐 Permissions](#-permissions) |
| [🕶️ Privacy](#️-privacy) | [🆕 What's new in 1.1.0](#-whats-new-in-110) | |
| [⚙️ Configuration](#-configuration) | [🪫 Memory](#-memory) | [🚧 Known limitations](#-known-limitations) |
| [🗂️ Where things live](#-where-things-live) | [🧱 Source layout](#-source-layout) | [🗑️ Uninstall](#-uninstall) |

---

## 🧐 Why this exists

macOS has no taskbar. The Dock shows what you pinned, not what is open, and there is no row of
window buttons along the edge of the screen. MangoBar draws that row: every running app, click one
to switch to it.

A taskbar that identifies apps by bundle id and display name alone gets some of them wrong — that is
why Minecraft shows up as **"Java"** and cannot be excluded. MangoBar matches on bundle id,
executable name, executable path, or a regex, and lets you rename and re-icon anything it gets wrong.

```json
{"apps": {"overrides": [
  {"match": {"exec_name": "java", "mode": "exact"}, "name": "Minecraft", "exclude": true}
]}}
```

To be a hundred percent honest, the other reason is wanting my own version. 🥭

---

## 📥 Install

Download **`mangobar-1.1.0.dmg`** from the
[latest release](https://github.com/mannnnnnnngo/Mangobar/releases/latest), drag the mango onto
Applications, then **right-click MangoBar → Open** the first time. That last step matters — the
app isn't signed with a paid Apple developer account, and right-click → Open is Apple's own way
past the warning. You only do it once.

Nothing else needs installing. The bundle carries its own Python and its own copies of PyObjC.

### Building it yourself

```bash
python3 -m venv .venv
.venv/bin/pip install pyobjc-framework-Cocoa pyobjc-framework-Quartz pyobjc-framework-ApplicationServices
./build_app.sh          # builds and installs to /Applications
./package.sh            # builds and wraps it in dist/mangobar-<version>.dmg
```

The venv's Python is the one that gets **copied into the bundle**, so it has to be a framework or
`--enable-shared` build — python.org, Homebrew and `actions/setup-python` all are. `build_app.sh`
says so plainly rather than producing a bundle that cannot start.

> [!IMPORTANT]
> The launcher **embeds** the Python interpreter rather than exec-ing it. That is what makes macOS
> attribute Screen Recording and Accessibility grants to **MangoBar** — not to Python, and not to
> your editor. It `dlopen`s it rather than linking it, so a broken interpreter is a dialog you can
> read instead of a crash inside dyld that says nothing. 🔐

Because the source is copied in, editing `mangobar/*.py` needs a rebuild to show up in the
installed app. While working, run it straight out of this folder instead:

```bash
./run.sh
```

First run writes `~/.config/mangobar/config.json` and shows the seven-step tour.

---

## 👀 Using it

MangoBar is a dock. Hide Apple's from **System Settings → Desktop & Dock → "Automatically hide and
show the Dock"** and MangoBar takes over from there.

| Do this | Get this |
|---|---|
| 🖱️ Click an app | Go to it. Click the one you're already in and it hides. |
| 👁️ Point at an app | Its name, and a live picture of its window — even on another desktop. |
| 🖱️ **Click that picture** | Straight to the app. New in 1.1.0. |
| 🖱️ Right-click an app | Pin it, so it stays on the bar whether it's running or not. |
| 🖱️ Right-click the bar | Preferences, the tutorial, permissions, and Quit. |

Clicking an app **always** brings it back now, including when its windows are minimised or you
closed the last one with ⌘W. Activating an app does not pull a minimised window out of the Dock
on its own, so MangoBar asks for that explicitly before it activates.

### Preferences

Right-click the bar → **Preferences…**. Eleven panes, in a sidebar grouped three ways:

| Group | Panes |
|---|---|
| **Bar** | ⚙️ General · 🎨 Theme · 🧩 Areas |
| **Contents** | 📜 Menu · 📱 Apps · 🕒 Date & Time |
| **App** | ⌨️ Shortcuts · 🔒 Permissions · 🔽 Updates · ❓ Tutorial |

Everything writes to the same JSON file, which you can also edit directly — changes apply within
about two seconds, no restart. ✨

**Updates** shows which version you are on, checks for a newer one, and has switches for checking,
downloading and installing automatically. **Tutorial** replays the seven-step tour. **General**
has *Open on Login*.

---

## 🚫 Excluding apps

Preferences → **Apps** lists every running app, including background helpers. Select one and
press **→** to exclude it.

Exclusion is absolute: an excluded entry never appears in the bar, whatever launched it. Entries
are keyed by bundle id when there is one and by executable name when there isn't, so
terminal-launched processes and Java apps can be excluded too, which matching on bundle id alone
cannot do.

To catch something with no bundle id, add a rule by hand:

```json
"apps": {"exclude": ["java", "node", {"exec_name": "python3", "mode": "exact"}]}
```

> [!TIP]
> Turn on **Advanced → Show background / terminal-launched apps** to make them visible in the bar
> first, so you can see what you are excluding. 👁️

---

## 🧩 Layout

The bar is three areas, each an ordered list of widgets. Rearranging it is a config edit:

```json
"areas": {
  "left":   ["menu", "desktop", "pinned", "tasks"],
  "center": [],
  "right":  ["battery", "trash", "clock"]
}
```

Available widgets: `menu` `desktop` `pinned` `tasks` `battery` `trash` `clock` `separator`

- **pinned** — compact squares, always visible whether the app is running or not
- **tasks** — wider labelled buttons for running apps that aren't pinned

---

## 🔐 Permissions

| Feature | Permission | Without it |
|---|---|---|
| 🖼️ Live window previews on hover | **Screen Recording** — required | The hover card shows only the app's name |
| 🪟 Window titles, per-window switching, restoring minimised windows, Show Desktop | Accessibility — recommended | Falls back to app-level clicks |

Both are asked for the first time MangoBar opens, and both live in **System Settings → Privacy &
Security**. Preferences → **Permissions** shows what macOS currently thinks and has a button for
each.

> [!NOTE]
> Screen Recording is **required** from 1.1.0. It was described as optional before, which was
> generous in two directions: the previews are most of the reason to point at a button at all,
> and the code that drew them referred to a module it never imported, so they had in fact never
> worked in any released version. Both halves of that are fixed here.
>
> macOS calls the permission Screen Recording because it is the same one a screen recorder uses.
> MangoBar never records anything — see below.

---

## 🕶️ Privacy

**Nothing leaves your Mac.** There is no account, no analytics, and no server.

- Window previews are captured here, held in memory while the card is on screen, and thrown away.
  They are never written to disk and never sent anywhere.
- The list of apps you have open never leaves the process that drew it.
- Your settings are one JSON file in your own home folder.

The only network request MangoBar ever makes is reading one small text file on GitHub to find out
whether a newer version exists. It sends nothing about you or this Mac, and Preferences → Updates
switches even that off.

---

## 🆕 What's new in 1.1.0

| | |
|---|---|
| 🛠️ **It opens** | The bundle is self-contained. Releases before this pointed at the build machine's file paths and died silently on every other Mac. |
| 🖼️ **Previews work** | `hover_card.py` used `cg.` throughout and never imported it, so every capture raised `NameError` inside a timer and previews had never once appeared. |
| 🖱️ **Previews are clickable** | Click the picture of a window to go to it. The card stays up while your pointer is on it. |
| 🪟 **Minimised and ⌘W'd apps come back** | Clicking an app now un-minimises it, or asks it for a new window, instead of appearing to do nothing. |
| 🗂️ **Sidebar preferences** | Eleven panes grouped into Bar / Contents / App, each with a line saying what it is for. |
| ❓ **A tutorial** | Seven steps, shown on first launch and replayable from the menu. |
| 🔒 **A Permissions pane** | Live state, a button each, and the plain statement that nothing leaves this Mac. |
| 💿 **A proper installer** | The disk image now opens the same drag-to-Applications window the other Mango apps use. |

---

## ⚙️ Configuration

Everything lives in `~/.config/mangobar/config.json`. The highlights:

| Key | Meaning |
|---|---|
| `bar.position` | `bottom` `top` `left` `right` |
| `bar.thickness` | Bar height in points (44 is the default) |
| `apps.pinned` | Bundle ids, always shown, in this order |
| `apps.include` / `apps.exclude` | Match rules; empty include = allow all |
| `apps.overrides` | Rename / re-icon / force-exclude misidentified apps |
| `appearance.icon_size` | Icon size (22 default) |
| `appearance.group_gap` | Space between pinned squares and task rectangles |
| `theme.background_opacity` | 0.0–1.0 |
| `behavior.clicking_active_hides` | Click the active app to hide it |

---

## 🪫 Memory

Roughly **82 MB** resident with the working set warm, falling to around 60 MB once the bar has
been idle for a few minutes and macOS reclaims the pages it isn't touching. It does not climb
with uptime. The breakdown of the warm figure:

| | |
|---|---|
| Python 3.9 interpreter | ~10 MB |
| `import AppKit` (PyObjC bridge metadata) | ~25 MB |
| `NSApplication` + window server connection | ~15 MB |
| MangoBar's own modules, panels, icons | ~30 MB |

Three things keep it there, and all three are easy to undo by accident.

> [!WARNING]
> **Don't `import Quartz` or `import ApplicationServices`.** They cost ~18 MB *each*, because
> PyObjC builds bridge metadata for entire umbrella frameworks. MangoBar uses about ten
> CoreGraphics functions and a dozen Accessibility ones, so it binds those directly instead —
> `macos/cg.py` for CoreGraphics, and plain `HIServices` (~0.8 MB) for the AX calls. `cg.py`
> documents the two things that have to be right: `already_cfretained` on every Create/Copy
> function, and `registerCFSignature` for the opaque CG types. Without either one, captures and
> window lists leak instead of being freed.

**Don't read window dictionaries on a timer.** Every value pulled out of a
`CGWindowListCopyWindowInfo` dictionary costs ~100 bytes that PyObjC 11.1 never frees — this is
a bug in the bridge, not in MangoBar, and it is not fixable from here (11.1 is the newest release
for Python 3.9). Scanning window bounds once a second leaked ~18 MB an hour on its own.
`macos/windows.py` therefore uses the on-screen window *count* — which needs nothing bridged out
of the dictionaries — as a change detector, and only pays for a real scan when the window set
actually moves.

**Don't repaint something that hasn't changed.** The same ~100 bytes per bridged value applies to
drawing, so a widget that repaints on a timer leaks whether or not a pixel moved. The clock polls
faster than the smallest unit it displays, and the trash icon has only two states, so both
compare first and only call `setNeedsDisplay_` when the result actually changed.

What remains is proportional to activity rather than to uptime: rebuilding the bar costs ~18 KB
and happens when an app launches, quits, or is switched to. Left alone, the bar does not grow.
The permanent fix for the underlying bug is PyObjC 12 or newer, which needs Python 3.10+ — this
repo runs on the 3.9 that ships with Apple's Command Line Tools.

---

## 🚧 Known limitations

- **`bar.position: "left"` / `"right"`** lay out horizontally; vertical layout is not implemented
  yet.
- **Attention flashing** — an app's button blinking when it wants your attention — has no public API,
  and is not implemented.
- **Per-window grouping, drag-to-reorder and auto-hide** are partly there. `checklist.json` is the
  source of truth for what is and isn't done, item by item.

---

## 🗂️ Where things live

```
~/.config/mangobar/config.json
```

One file, yours, written on first run and re-read within about two seconds of any change. It is
not in this repository and never should be — a bar layout belongs to one machine, not to
everyone who clones this. 📂

---

## 🧱 Source layout

Layered so any one piece can be replaced without touching the rest:

```
core/      events (pub/sub) · models · identity ← the app-matching engine
config/    defaults · store (JSON, live-reloaded)
macos/     workspace · screens · windows · cg   ← all AppKit isolated here
sources/   running_apps · filters · windows_ax  ← where content comes from
actions/   app_actions · window_actions         ← what clicks do
ui/        panel · theme · bar_view · menus · hover_card
           preferences · prefs_fields · prefs_apps · prefs_updates
           prefs_privacy · prefs_tutorial
           widgets/  base · registry · app_buttons · clock · battery
                     trash · desktop · menu_button · separator
bundle/    launcher.c · make_icon.swift · the .icns
installer/ background.swift · make_dmg.sh    ← shared by all five mango apps
```

| 📄 File | Purpose |
| --- | --- |
| `mangobar/core/identity.py` | How an app is recognised — bundle id, exec name, path, regex. The reason Minecraft can be called Minecraft |
| `mangobar/core/events.py` | The pub/sub bus every layer talks over, so nothing below `ui/` knows a bar exists |
| `mangobar/macos/cg.py` | The ten CoreGraphics functions, bound by hand instead of importing Quartz |
| `mangobar/macos/windows.py` | Window enumeration, with the count-first change detector that keeps the bridge leak off the timer |
| `mangobar/sources/filters.py` | Include / exclude / override rules applied to the running-app list |
| `mangobar/ui/widgets/registry.py` | One line per widget — the entire cost of adding one |
| `mangobar/ui/theme.py` | Every colour and metric. Restyling touches this file only |
| `mangobar/ui/prefs_fields.py` | Every setting, one line each, plus which sidebar section its tab belongs to |
| `mangobar/ui/prefs_tutorial.py` | The seven steps, shown both as a pane and as the first-run window |
| `bundle/launcher.c` | Finds the Python inside the bundle, `dlopen`s it, and runs `mangobar` — with no path from the machine that built it |
| `checklist.json` | What is done, what isn't, and what was deliberately cut |

Adding a widget: write the module, add one line to `ui/widgets/registry.py`, name it in `areas`.
If it takes more than that, the registry is wrong.

---

## 🗑️ Uninstall

```bash
pkill -f mangobar
rm -rf /Applications/MangoBar.app ~/.config/mangobar
```

Then delete this folder. Nothing is installed system-wide; the Dock is never modified.

---

## ⚖️ Licence

MangoBar is **free to use** but **not open source**. The source is published here to be read, not
reused: it may not be redistributed, resold, built upon, or presented as anyone else's work. The
full terms are in [`LICENSE`](LICENSE).

Copyright © 2026 Mingyu. All rights reserved.

---

<div align="center">

**Made with 🥭 by Mingyu**

🆓 Free forever · 🔒 Zero permissions to run · 🐍 No Xcode, no compile

Part of [🥭 MangoApps](https://github.com/mannnnnnnngo/MangoApps)

</div>
