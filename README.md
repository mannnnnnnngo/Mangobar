<div align="center">

<img src="Mangobar.png" width="140" alt="MangoBar icon">

# 🥭 MangoBar

**A Windows-style taskbar for macOS — every open app in a row along the edge of the screen, and it can exclude the one pretending to be Java.**

Made by Mingyu 🧑‍💻

<br>

![macOS](https://img.shields.io/badge/macOS-13%2B-202020?style=for-the-badge&logo=apple&logoColor=white)
![Python](https://img.shields.io/badge/Python-PyObjC-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Version](https://img.shields.io/badge/version-1.2.0-7C5CFF?style=for-the-badge)
![Price](https://img.shields.io/badge/price-free-2EA043?style=for-the-badge)
![Permissions](https://img.shields.io/badge/permissions%20to%20run-none-0EA5E9?style=for-the-badge)

</div>

---

> [!NOTE]
> **MangoBar is completely free.** 🆓 It is macOS only — it is PyObjC talking to AppKit, which
> doesn't exist anywhere else. **No Xcode required**, and no compile step either: the app is a
> thin shim around the Python in this folder, so editing the source takes effect on the next
> launch. 🐍

---

## 📖 Contents

| | | |
| --- | --- | --- |
| [🧐 Why this exists](#-why-this-exists) | [📥 Install](#-install) | [👀 Using it](#-using-it) |
| [🚫 Excluding apps](#-excluding-apps) | [🧩 Layout](#-layout) | [🔐 Permissions](#-permissions) |
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

```bash
python3 -m venv .venv
.venv/bin/pip install pyobjc-framework-Cocoa pyobjc-framework-Quartz pyobjc-framework-ApplicationServices
./build_app.sh
```

Installs **MangoBar.app** into `/Applications` so it shows up in Finder like any other app. The
source stays in this folder — the app is a thin shim, so editing Python takes effect on the next
launch with no rebuild. Only re-run `build_app.sh` if `bundle/launcher.c`, the `Info.plist` or
`VERSION` changes.

> [!IMPORTANT]
> The launcher **embeds** the Python interpreter rather than exec-ing it. That is what makes
> macOS attribute Screen Recording and Accessibility grants to **MangoBar** — not to Python, and
> not to your editor. 🔐

For development without installing:

```bash
./run.sh
```

First run writes `~/.config/mangobar/config.json`.

---

## 👀 Using it

Right-click the bar → **Preferences…**. Eight tabs:

| Tab | Contents |
|---|---|
| ⚙️ General | Position, size, margin, corner radius, display, login, hover behaviour |
| 🎨 Theme | Preset, four colour wells, opacity, blur, icon/font sizing |
| 🧩 Areas | Show Menu / Desktop / Battery / Trash / Separators / Clock |
| 📜 Menu | Which folders, utilities and power actions appear in the menu button |
| 📱 Apps | Included / Excluded lists |
| 🕒 Date & Time | Clock format, calendar |
| ⌨️ Shortcuts | Not implemented yet |
| 🔬 Advanced | Click behaviour, background apps, app order, debug logging |

Everything writes to the same JSON file, which you can also edit directly — changes apply within
about two seconds, no restart. ✨

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

## 🚢 The macOS Dock

Two taskbars along the same edge is the one arrangement nobody wants: they overlap, they fight
for the same pointer, and the one underneath wins the clicks. So MangoBar moves the Dock out of
its way — **to the right, hidden until the pointer goes looking for it** — the first time it runs.

| Setting | Default | |
| --- | --- | --- |
| `dock.move_out_of_the_way` | `true` | Off, MangoBar leaves the Dock exactly where it is |
| `dock.position` | `right` | `right`, `left` or `bottom` |
| `dock.autohide` | `true` | The Dock slides away until you push the pointer at that edge |

Preferences → **Advanced** → *The macOS Dock* has the same three.

Nothing here is hidden or one-way. It is what you would type yourself:

```bash
defaults write com.apple.dock orientation -string right
defaults write com.apple.dock autohide -bool true
killall Dock
```

**What the Dock was doing before is written to `~/.config/mangobar/dock-before.json`** the first
time MangoBar changes it, and put back when you switch the setting off or quit MangoBar —
because quitting takes the bar away, and a Mac with no bar *and* no Dock is a Mac you cannot use.
If that file ever goes missing, System Settings → Desktop & Dock has the same two controls.

---

## 🔐 Permissions

The bar works with **zero permissions**. Two optional features need one:

| Feature | Permission | Without it |
|---|---|---|
| 🖼️ Live window previews on hover | Screen Recording | Hover shows the name only |
| 🪟 Window titles, per-window switching, Show Desktop | Accessibility | Falls back to app-level clicks |

Nothing is requested until you use a feature that needs it.

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
           widgets/  base · registry · app_buttons · clock · battery
                     trash · desktop · menu_button · separator
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
