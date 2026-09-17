# Niri + Noctalia v5 Dotfiles

A scrollable-tiling [Niri](https://niri.wm/) setup paired with the [Noctalia v5](https://noctalia.dev/) desktop shell (bar, panels, launcher, control center, notifications, wallpaper, lock screen, desktop widgets).

<p align="center">
  <img src="Screenshots/Screenshot.png" width="48%" />
  <img src="Screenshots/Screenshot2.png" width="48%" />
</p>

---

## Requirements

### Core

| Package | Purpose |
|---|---|
| `niri` (recent git / 26.04+) | Wayland compositor. Needs a recent build for `background-effect` blur support and `custom-shader` animations. |
| `noctalia` (v5) | The desktop shell. Provides bar, launcher, panels, dock, control center, notifications, wallpaper, lock screen and session actions. |
| `niri-float-sticky` | [Daemon](https://github.com/probeldev/niri-float-sticky) that makes floating windows "sticky" across workspaces (niri doesn't do this natively). Exposes `-ipc toggle_sticky` for the `Mod+Shift+G` bind. |

### Clipboard

| Package | Purpose |
|---|---|
| `wl-clipboard` (`wl-copy`/`wl-paste`) | Wayland clipboard, used by the cliphist watchers. |
| `cliphist` | Clipboard history store; the watchers pipe `wl-paste` into it in the background at startup. |
| `wl-clip-persist` | Keeps clipboard content alive after an app exits. |

### System / media

| Package | Purpose |
|---|---|
| `libnotify` (`notify-send`) | Notifications (lid open/close events, recording toasts). |
| `wf-recorder` | Fallback recording engine (`Mod+Print`). |
| `playerctl` | Media playback control (Play/Stop/Prev/Next binds). |
| `wireplumber` (`wpctl`) | Default audio sink mute toggle (`Mod+Shift+M`). |
| `gsettings` + `adw-gtk3-dark` | GTK theme applied at startup. |
| `Colloid-Dark` icon theme + `Qogirr` cursor theme | Desktop icon/cursor theming. |
| `evtest` | Required by the Show Keys plugin (`/dev/input/event*` access). |

### Referenced apps

| App | Used for |
|---|---|
| `kitty` | Default terminal (`Mod+Return`). |
| `nautilus` | File manager (`Super+E`). |
| `zen-browser` | Browser (`Super+B`). |
| `zed` | Zed projects provider in the launcher (`/zed`, `Mod+Z`). |
| `gh` | GitHub Kanban plugin (authenticated via `gh auth login`). |

> **Outdated:** Noctalia v4 (Quickshell, `qs -c noctalia-shell …`) is no longer used. All binds are pure Noctalia v5 (`noctalia msg …`) — the v4 fallback shims were removed from `config.d/binds.kdl`.

---

## File structure

```
~/.config/niri/
├── config.kdl                      # Main entry point
├── noctalia.kdl                    # Carbon-blue accent theme for layout focus/border
├── config.d/
│   ├── input-and-cursor.kdl        # Keyboard, touchpad, mouse, trackpoint, gestures, cursor theme
│   ├── layout-and-overview.kdl     # Gaps, columns, shadows, overview, lid switch events
│   ├── window-rules.kdl            # Per-app window rules (Noctalia, foot/kitty, keyboards, PiP…)
│   ├── environment.kdl             # Wayland env vars (GDK/QT/SDL/XDG…)
│   ├── startup.kdl                 # Autostarted daemons, clipboard, GTK theme
│   ├── animations.kdl              # Springs + custom GLSL open/close sweep shaders
│   ├── binds.kdl                   # All keybindings
│   ├── layer-rules.kdl             # Noctalia/waybar layer surface blur & shadow rules
│   ├── liquid-glass.kdl            # (disabled) iOS-style liquid-glass effect rules
│   └── user-extra.kdl              # Scratch space for personal overrides
└── Screenshots/                    # Preview images
```

`config.kdl` includes everything from `config.d/` plus `noctalia.kdl`. `liquid-glass.kdl` is optionally included (`//include optional=true`) and is currently disabled.

### What each file does

- **config.kdl** — `prefer-no-csd`, hotkey overlay (skipped at startup), screenshot path (`~/Pictures/Screenshots/…`), includes, and the `eDP-1` output at scale 1.
- **noctalia.kdl** — accent theme: `#33b1ff` active / `#161616` inactive / `#ee5396` urgent for focus-ring, border, tab-indicator, insert-hint, and recent-windows highlight.
- **input-and-cursor.kdl** — US+Arabic (`us,ara`) xkb layout on `pc105` with `grp:alt_shift_toggle`, per-window layout tracking, touchpad tap + natural scroll, `Super` mod key, `Qogirr` cursor at 32px, hot corners off.
- **layout-and-overview.kdl** — zero gaps, transparent background, preset column widths (1/3, 1/2, 2/3), soft window shadow, workspace overview zoom, and lid-close/open notifications.
- **window-rules.kdl** — default border/shadow/opacity for all windows; Noctalia settings window floats at 1080×920; foot/kitty/Alacritty open at half width; PiP windows float bottom-right at 560×315; the on-screen `Keyboard` gets blur + rounded corners; `pavucontrol` and `nm-connection-editor` float.
- **environment.kdl** — forces Wayland backends (`GDK_BACKEND`, `QT_QPA_PLATFORM=wayland;xcb`, `SDL_VIDEODRIVER`, `CLUTTER_BACKEND`), `XDG_CURRENT_DESKTOP=niri`, `qt6ct` platform theme, no CSD on Qt, Mozilla Wayland, Electron ozone auto.
- **startup.kdl** — autostarts `noctalia`, `niri-float-sticky`, cliphist watchers (`wl-paste --watch cliphist store`), `wl-clip-persist`, then applies GTK/icon themes.
- **animations.kdl** — animations `off` globally with `slowdown 0.25`, runtime springs for workspace/window movement, and two 1400 ms GLSL shaders: a corner-to-corner sweep reveal on `window-open`, and a bob-then-slide-down on `window-close`.
- **binds.kdl** — full Noctalia v5 keybinding set (below). Everything is `noctalia msg …` IPC.
- **layer-rules.kdl** — disables blur/xray on dock + desktop widgets, adds shadows to `waybar` and `noctalia-bar-default`, blurs behind the window switcher, and places wallpaper layers within the backdrop.

### `liquid-glass.kdl` (disabled)

A saved "iOS glass" style setup: frosted blur + custom `liquid-glass` refraction/edge-lighting parameters for windows and Noctalia panels. Not currently included — re-enable with `include optional=true "config.d/liquid-glass.kdl"`.

---

## Noctalia plugins in use

### Installed plugins (`~/.config/noctalia/plugins.json`, source: `noctalia-dev/noctalia-plugins`)

| Plugin | Version | What it adds | Extra requirements |
|---|---|---|---|
| `kde-connect` | 1.2.1 | Mobile device integration (Phone Display, sync, file browsing) from bar/control center | `kdeconnectd` running; `sshfs` + `libfuse` for file browsing |
| `keep-awake-plus` | 0.3.0 | Session keep-awake with partial/full inhibit modes + persistent last choice | — |
| `keybind-cheatsheet` | 3.7.3 | Searchable keybind viewer that auto-detects Niri (see also `blackbartblues/keymap` below) | — |
| `notes-scratchpad` | 1.1.6 | Quick scratchpad for throwaway notes | — |
| `pomodoro` | 1.2.0 | Pomodoro timer (bar widget + panel + alarm sound) | — |
| `screen-toolkit` | 1.3.3 | Color picker, annotate, record, pin, OCR, QR scan, palette, measure, webcam mirror (bound at `Mod+F2`) | `grim`, `slurp`, `hyprpicker`, `tesseract`, `ffmpeg`, `wl-screenrec`/`wf-recorder`, `translate-shell`, `imagemagick`, `zbar`, `curl`, `jq` |
| `show-keys` | 1.0.2 | Real-time keypress OSD via `evtest` | `evtest` + read access to `/dev/input/event*` |
| `tamagotchi` | 1.1.2 | Desktop Tamagotchi pet living on the taskbar | — |

### Plugins referenced by keybinds (Noctalia v5 plugin store: `noctalia-dev/community-plugins` + official)

| Bind | Plugin ID | What it opens | Extra requirements |
|---|---|---|---|
| `Mod+M` | `aabidk20/yt-music:panel` | YouTube Music client (panel + miniplayer) | `yt-dlp`, `mpv`, `mpv-mpris`, `jq`, `curl`, `nc` |
| `Mod+G` | `shangshui0302/github-kanban:kanban` | GitHub dashboard (PRs, issues, notifications, heatmap) | `gh` authenticated, `xdg-open` |
| `Mod+Shift+C` | `oldirtty/color_picker:panel` | Screen color picker panel | `hyprpicker` |
| `Mod+Alt+C` | `yuuto/calculator:panel` | Calculator panel | — |
| `Mod+Shift+W` | `ashur-d/wallpaper-widget:hub` | Wallpaper carousel/switcher | optional `magick`/`convert`/`ffmpeg` for thumbnails |
| `` Mod+grave `` | `nightwatch75/todo:panel` | Prioritized to-do list | — |
| `Mod+F3` | `noctalia/screen_recorder:service` | GPU screen recording + replay buffer | `gpu-screen-recorder` |
| `Mod+Shift+Slash` | `blackbartblues/keymap:panel` | Searchable keymap cheatsheet/editor | `niri` CLI on `PATH` |

### Built-in Noctalia features driven by these binds

- **Launcher views** — `Mod+Space` (main), `Mod+Tab` (windows list `/win`), `Mod+Z` (Zed `/zed`), `Mod+Slash` (emoji `/emo`).
- **Control center** — `Mod+S` (main), `Mod+N` (notifications), `Mod+Escape`-style panel toggles.
- **Session** — `Mod+Shift+E` (session menu), `Mod+Alt+L` (lock screen), `XF86PowerOff` (session panel).
- **Media/volume/brightness OSDs** — via `noctalia msg volume-*`, `brightness-*`, etc.
- **Screenshots** — `Print` region, `Ctrl+Print` fullscreen, `Alt+Print` window (via `noctalia msg screenshot-*`).
- **Desktop widgets** — `Mod+F1` enters edit mode; `Mod+Shift+U` sets a random wallpaper.

---

## Keybindings

`Mod` = `Super`. Full listing is in `config.d/binds.kdl` (the **Keymap/cheatsheet** plugins render it; `Mod+Shift+Slash` shows it).

### Core window management
| Keys | Action |
|---|---|
| `Mod+Q` | Close window |
| `Mod+F` | Fullscreen |
| `Mod+A` | Toggle floating |
| `Mod+D` | Maximize column |
| `Mod+C` / `Mod+Ctrl+C` | Center column / center visible columns |
| `Mod+Shift+G` | Toggle sticky floating window (`niri-float-sticky -ipc toggle_sticky`) |
| `Mod+R` / `Mod+Shift+R` / `Mod+Ctrl+R` | Cycle width / cycle height / reset height |
| `Mod+Minus` / `Mod+Equal` | Column width −/+ 10% |
| `Mod+Shift+Minus` / `Mod+Shift+Equal` | Window height −/+ 10% |
| `Mod+[` / `Mod+]` | Consume/expel window left/right |

### Focus & movement (Vim-style)
| Keys | Action |
|---|---|
| `Mod+H` / `Mod+L` | Focus column left/right |
| `Mod+J` / `Mod+K` | Focus window/workspace down/up |
| `Mod+Shift+H` / `Mod+Shift+L` | Move column left/right |
| `Mod+Shift+J` / `Mod+Shift+K` | Move window down/up (or across workspaces) |
| `Mod+1…9` | Focus workspace |
| `Mod+Shift+1…9` | Move column to workspace |

### Noctalia / launcher / media
| Keys | Action |
|---|---|
| `Mod+O` | Toggle overview |
| `Mod+Space` | Launcher |
| `Mod+Tab` | Windows list |
| `Mod+S` | Control center |
| `Mod+Comma` | Settings |
| `Mod+V` | Clipboard history |
| `Mod+N` | Notifications |
| `Mod+M` | YouTube Music |
| `Mod+G` | GitHub dashboard |
| `Mod+Shift+E` | Session menu |
| `Mod+Alt+L` | Lock screen |
| `Mod+Shift+M` | Mute sink |
| `Mod+Shift+P` / `Mod+Shift+N` / `Mod+Shift+B` | Play/pause / next / previous (locked) |
| `XF86Audio*` / `XF86MonBrightness*` | Volume & brightness OSDs (locked) |
| `Mod+Print` | Record with `wf-recorder` (`~/Videos/…`) |
| `Print` / `Ctrl+Print` / `Alt+Print` | Noctalia region / fullscreen / window screenshots |

---

## Resources

- Niri: https://niri.wm/
- Noctalia v5 docs: https://docs.noctalia.dev/v5/
- Niri + Noctalia integration guide: https://docs.noctalia.dev/v5/compositor-settings/niri
- Plugins: https://noctalia.dev/plugins
- niri-float-sticky: https://github.com/probeldev/niri-float-sticky