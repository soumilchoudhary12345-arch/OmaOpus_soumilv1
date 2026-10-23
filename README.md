# OmaOpus_soumilv1 󱑽 — The Universe of Music. In Your Bar. Now with Offline Downloads.

> **Fork of [OmaOpus](https://github.com/MaenExists/omaopus) v1.1.0 by [MaenExists](https://github.com/MaenExists)**
> Extended with a full offline download manager — stream from YouTube, save as MP3, play without a network.
> All credit for the original player, visualizer, and bar-widget architecture goes to MaenExists. MIT license preserved.

<p align="center">
  <img src="preview.png" alt="OmaOpus Showcase" width="100%">
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-emerald?style=flat-square" alt="License"></a>
  <img src="https://img.shields.io/badge/Fork-soumilv1-orange?style=flat-square" alt="Fork">
  <img src="https://img.shields.io/badge/Memory_Footprint-<15MB_RAM-cyan?style=flat-square" alt="Memory">
  <img src="https://img.shields.io/badge/Stream_Latency-<0.5s-violet?style=flat-square" alt="Latency">
  <img src="https://img.shields.io/badge/Idle_CPU-0.0%-green?style=flat-square" alt="CPU">
</p>

---

## 🆕 What's New in soumilv1 — Offline Download Manager

Everything below is on top of upstream OmaOpus v1.1.0. The audio engine, search, queue, favorites, and 32-band visualizer are unchanged.

### Backend (`oma-player`, +312 lines)

| Feature | How it works |
| :--- | :--- |
| **Serial download worker** | Detached via `setsid` so downloads survive panel/shell reloads. Single worker guarded by `flock` (`download.lock`) — no parallel yt-dlp fights. |
| **Download queue** | FIFO queue in `download_queue.json`, read-modify-write serialized with a second lock (`enqueue.lock`). Duplicate URLs are detected and deduped. |
| **MP3 conversion** | `yt-dlp -x --audio-format mp3 --audio-quality 192K` with embedded metadata + thumbnail. Files land in `~/Music/OmaOpus` as `Title [videoId].mp3`. |
| **Live progress** | A poll loop parses yt-dlp's log and streams percent updates into `download.json`; the UI's `FileView` reacts in real time. |
| **Crash self-healing** | If the worker dies mid-download, `downloads-list` detects the stale "downloading" state (no lock holder), requeues the track at the FRONT, and respawns the worker. |
| **Dedupe** | Video ID checked against `downloaded.json` — already-saved tracks short-circuit with "already in library". |
| **Offline playback** | `play-local` loads any library file straight into mpv — no network, no URL resolution. |
| **Safe delete** | `delete-download` refuses paths outside `~/Music/OmaOpus`, stops playback if the deleted file is current, and cleans `downloaded.json`. |
| **Library scan** | `downloads-list` scans with `ffprobe` (title/artist/duration), caches results in `library.json` keyed by a name/size/mtime signature — repeat calls are instant. |

New CLI surface:

```bash
oma-player download [url] [title]     # enqueue current or given track
oma-player download-cancel <url>      # drop a queued (not active) download
oma-player downloads-list             # library + pending queue + saved IDs (JSON)
oma-player play-local <path> [title] [artist]
oma-player delete-download <path>
```

New state files (all under `~/.local/state/omaopus/`): `download.json`, `downloaded.json`, `download_queue.json`, `download.log`, `library.json`, plus the two flock files.

### UI (`BarWidget.qml`, +546 lines)

- **4th tab: 󰈂 Downloads** — active-download card with live percent + progress bar, clickable pending queue (cancel with one click), and the full offline library list with file sizes.
- **Download buttons everywhere** — search results, queue rows, favorites rows, and the now-playing header all get a state-aware 󰈂 button: idle → download, queued → show position, downloading → spinner with percent, saved → checkmark (click opens the Downloads tab).
- **Offline playback** — click any row in Downloads to play it locally through the same mpv daemon.
- **Delete from disk** — 󰅖 on any library row (guarded to the library directory only).
- **Bigger panel** — 390×560 → 540×900 to fit the new tab.
- **IPC additions** — `quickshell ipc call maen.omaopus download` and `... tab downloads` for keybinding integration (SUPER+ALT+D hint in the footer).

---

> *"Every once in a while, a tool comes along that changes how you interact with your desktop forever. Music shouldn't require a dedicated window, an extra workspace, or half a gigabyte of RAM. It should simply be there — woven into the cosmos of your bar."*

### Why OmaOpus?

There are plenty of music players in Linux, but **none like this for Omarchy OS**.

Opening an entire browser window or a heavy Electron app just to play background music fractures your focus, clutters your Hyprland workspaces, and burns 400MB–600MB of memory. 

**OmaOpus is the first and only full-featured, zero-footprint music workstation living natively in the Omarchy top bar.**

It runs as a silent, featherweight background daemon powered by C-speed `mpv` IPC. One click on your bar reveals a studio-grade interface with millisecond YouTube search, gapless queue autoplay, persistent favorites, and a hardware-accelerated 32-band audio visualizer. When closed, it consumes **0.0% CPU and less than 15MB of RAM**.

---

## ✨ Features That Set It Apart

### 🪶 1. Native In-Bar Daemon — No Windows to Juggle
- **Zero Desktop Clutter**: Sits unobtrusively in your status bar as an animated live mini-equalizer pill when playing, and a tranquil icon when idle.
- **Headless Background Audio**: Music continues playing uninterrupted in the background through a low-latency Unix socket daemon.
- **Microscopic Footprint**: Strict 16MB demuxer ring buffers keep RAM usage locked **under 15MB**.

### 📊 2. Real-Time 32-Band FFT Spectrum Analyzer
- **Native GPU SceneGraph Rendering**: 32 hardware-accelerated equalizer columns running at 40 FPS with zero frame drops.
- **Authentic Hi-Fi Ballistics**: Instant transient attack with smooth logarithmic analog meter decay, simulated gravity peak-hold caps, and glossy acrylic baseline reflections.
- **Studio Frequency Axis**: Calibrated frequency markers (`20Hz` · `100Hz` · `500Hz` · `1kHz` · `4kHz` · `10kHz` · `20kHz`).
- **4 Cosmic Palettes** (interactive click-to-cycle):
  - **★ Cyber Neon**: Neon Cyan -> Electric Blue -> Violet -> Hot Pink peaks.
  - **☀ Solar Amber**: Warm McIntosh vintage gold and molten amber glow.
  - **✦ Matrix Emerald**: Cyberpunk electric mint, jade, and deep emerald.
  - **🌌 Electric Violet**: Midnight indigo to radiant lavender and orchid.
- **True Idle Silence**: Guaranteed 0.0 resting baseline when paused or stopped. No phantom jitter.

### 🎛️ 3. Smart Queue & Gapless Autoplay
- **Predictive Pre-Buffering**: While the current track is playing, OmaOpus resolves and pre-buffers the next track in the background.
- **Continuous Flow**: Automatically transitions to the next track with zero awkward silence.
- **Queue Management**: One-click reordering, track removal, and instant queue clearing.

### 🔍 4. Instant Search & One-Click Favorites
- **Sub-Second Search**: Fast flat-playlist extraction delivers YouTube search results in milliseconds.
- **Disk Caching**: Frequently searched queries load from disk in **under 50ms**.
- **Persistent Favorites**: Star your favorite tracks with one click (`󰋔`) to build your permanent local music library.

### ⌨️ 5. Keyboard-First & Quickshell IPC
- Full keyboard navigation: Type to search, navigate with `↑ / ↓`, press `Enter` to play, and tap `Esc` to dismiss.
- Scriptable CLI & global keybinding support via QuickShell IPC:
  ```bash
  quickshell ipc call maen.omaopus toggle  # Toggle dropdown player
  quickshell ipc call maen.omaopus play    # Toggle play / pause
  quickshell ipc call maen.omaopus next    # Skip to next track
  quickshell ipc call maen.omaopus prev    # Previous track
  ```

---

## 📊 The Difference: Web Browser vs. OmaOpus

| Metric | Web Browser (YouTube / Web Player) | OmaOpus |
| :--- | :---: | :---: |
| **RAM Footprint** | ~400 MB – 650 MB | **< 15 MB** *(97% memory reduction)* |
| **Idle CPU Load** | 2.0% – 6.0% | **~0.0%** |
| **Window Clutter** | Requires browser tab / window | **100% In-Bar Popup** *(0 windows)* |
| **Audio Latency** | Script initialization overhead | **< 0.5s** *(direct audio stream)* |
| **Audio Spectrum** | None / basic web canvas | **32-Band Hardware SceneGraph FFT** |
| **Desktop Integration** | Detached web app | **Native Omarchy & Quickshell UI** |

---

## 🚀 Instant Installation

Install in one command through the official Omarchy plugin manager:

```bash
omarchy plugin add https://github.com/MaenExists/omaopus.git --enable
```

*The widget will immediately appear in your top bar between your status widgets.*

---

## 🎮 Controls & Shortcuts

| Action | How to Trigger |
| :--- | :--- |
| **Open / Close Player** | Left-Click Top Bar Icon or `Super+Shift+M` |
| **Play / Pause** | Center control button or IPC `play` |
| **Next / Previous Track** | `󰒭` / `󰒮` buttons or IPC `next` / `prev` |
| **Cycle Cosmic Palette** | Click theme capsule (`CYBER NEON` / `SOLAR AMBER` / `MATRIX EMERALD` / `ELECTRIC VIOLET`) |
| **Toggle Visualizer** | Click `󰺢` / `󰺠` in player header |
| **Autoplay Toggle** | Click `󰈑` / `󰈒` in player header |
| **Add / Remove Favorite**| Click `󰋔` / `󰋑` on any track |
| **Queue Track** | Click `󰐍` on search results |
| **Dismiss Panel** | `Esc` or click outside |

---

## 🛠️ Built Purely for Speed

- **Quickshell & QML 2.0**: Native hardware-accelerated desktop UI rendered directly via Wayland / OpenGL.
- **Headless mpv Daemon**: High-fidelity audio playback over low-latency PipeWire / PulseAudio.
- **Python FFT Engine**: Log-spaced discrete 32-band FFT with dynamic Auto-Gain Control and transient onset detection.
- **yt-dlp**: High-speed audio-only stream extraction (`251/140/ba`).

---

## 📄 License

MIT © [MaenExists](https://github.com/MaenExists)
