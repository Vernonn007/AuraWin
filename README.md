# AuraWin

An ambient edge-light for Windows: it reads whatever's currently playing system-wide,
pulls a vivid color palette out of the album art, and animates a customizable glowing
band around your screen's edges — corner to corner, in sync with the bass and vocals.

Inspired by [Lumn](https://www.trylumn.xyz/) for macOS, rebuilt for Windows with more
customization: independent bass/mid-high reactivity, multiple color modes, adjustable
waveform shape/thickness/count, and a frame-rate control for GPU usage.

## Features

- **Always-on, click-through overlay** — sits on top of every app without blocking
  clicks or keyboard input, and never shows up in the taskbar or Alt+Tab.
- **Color synced to what's playing**
  - **Album Art (Gradient)** — a smooth gradient across the most vivid colors
    pulled from the current track's cover art.
  - **Album Art (Solid)** — the single most vivid color from the cover art.
  - **Rainbow** — a true full-spectrum rainbow gradient.
  - **Single Color** — pick one exact color yourself from a color picker.
  - Falls back to a deterministic color derived from the track title/artist for
    sources that don't expose real album art to Windows (common with some
    browser-based/web players).
- **Independent audio reactivity**
  - **Bass** drives the glow's height (amplitude) and brightness punch.
  - **Mid/high** (vocals, hats, cymbals) drives how fast the light flows around
    the loop.
  - Each has its own 0–100% sensitivity slider, plus a master "Reactivity" dial.
- **Customizable waveform** — shape (Sine / Triangle / Pulse / SmoothBlob),
  thickness, how many ripples loop around the screen (1–3, with real gaps between
  them at 2–3), and amplitude.
- **Adjustable glow, brightness, and saturation** — including a contrast halo so
  the light stays visible even against a similarly-colored background.
- **Display modes** — always visible, or only while something's actually playing.
- **Frame rate control (10–240 fps)** — with a warning above 60fps, since this
  directly trades off against GPU usage.
- **System tray** — quick access to Settings, a debug/diagnostics overlay toggle,
  and Quit.

## Screenshots

*(Add a screenshot or short clip here once you've got one — a dark desktop with
the glow active around the edges shows it off best.)*

## Download
Download `AuraWinSetup.exe`, running it installs AuraWin into Program
Files with a Start Menu entry and a proper uninstaller, like any normal Windows app.

## Requirements

- Windows 10 (build 19041+) or Windows 11

On first launch, Windows will ask permission for AuraWin to read media session info
— this is what lets it see the current track and album art. It never accesses your
microphone; audio levels are read via WASAPI **loopback** capture (the system output),
not any input device.

## Using it

AuraWin runs from the **system tray** (look for its icon near the clock, including
inside the hidden-icons overflow arrow). Right-click it for:
- **Settings** — opens the customization window (also opens on double-click)
- **Show debug overlay** — toggles an on-screen readout of live audio
  levels/capture status, useful if something looks wrong
- **Quit**

### Settings reference

**Color**
| Control | What it does |
|---|---|
| Color mode | AlbumArtGradient / AlbumArtSolid / Rainbow / SingleColor |
| Sample points from album art | 3 or 4 — how many colors to pull for gradient mode |
| Pick color... | Only shown in SingleColor mode — opens a color picker |
| Saturation | 0–250%; also raises the minimum saturation floor so muted album art still looks vivid |
| Glow / brightness | Bloom size and baseline brightness |

**Waveform**
| Control | What it does |
|---|---|
| Shape | Sine / Triangle / Pulse / SmoothBlob |
| Thickness | Base width of the glowing band |
| Wave count | 1 = one continuous wave around the loop; 2-3 = that many, with gaps between them |
| Amplitude | Max reach of the bass-driven height punch |
| Overall reactivity | Master multiplier over both sensitivities below |
| Bass sensitivity | How much bass drives height + brightness |
| Mid/high sensitivity | How much vocals/hats/cymbals drive flow speed |

**Behavior**
| Control | What it does |
|---|---|
| Display mode | Always on, or only while media is playing |
| Launch at login | *(UI present; not yet wired to the Windows registry — see Roadmap)* |

**Performance**
| Control | What it does |
|---|---|
| Frame rate | 10–240 fps; a warning appears above 60fps since higher costs more GPU |

Settings are saved automatically to `%AppData%\AuraWin\settings.json`.

## How it works (brief)

- **Now-playing + album art**: `Services/MediaSessionService.cs` uses Windows'
  `GlobalSystemMediaTransportControlsSessionManager` (SMTC) — the same system that
  powers the media overlay on your lock screen / volume flyout.
- **Audio reactivity**: `Services/AudioLevelService.cs` captures system output via
  WASAPI loopback and splits it into bass/mid-high bands using simple IIR filters,
  each with its own fast-attack/slow-decay envelope follower.
- **Color selection**: `Services/ColorExtractor.cs` downsamples the album art,
  clusters it into candidate colors, and scores each by saturation + a
  mid-lightness preference (similar to Android's "Vibrant" palette extraction) —
  not just whichever color covers the most pixels.
- **Rendering**: `Overlay/EdgeGlowCanvas.cs` draws one continuous filled shape per
  frame — a rounded-rectangle "picture frame" whose inner edge undulates with the
  audio-reactive wave — rather than many small strokes, so it reads as one smooth
  band of light.
- **The overlay window itself**: `Overlay/OverlayWindow.xaml(.cs)` spans the full
  virtual desktop, borderless and transparent; `Overlay/NativeMethods.cs` applies
  the Win32 extended styles (`WS_EX_TRANSPARENT | WS_EX_LAYERED | WS_EX_TOOLWINDOW`)
  that make it click-through and hidden from the taskbar/Alt+Tab.

## Project layout

```
AuraWin/
  AuraWin.sln
  AuraWin/
    App.xaml(.cs)                     tray icon + wires everything together
    Models/AuraSettings.cs            settings model + JSON persistence
    Services/
      MediaSessionService.cs          SMTC now-playing + album art
      AudioLevelService.cs            WASAPI loopback capture, bass/mid-high bands
      ColorExtractor.cs               album art -> vivid color palette
    Overlay/
      OverlayWindow.xaml(.cs)         the click-through, full-desktop host window
      EdgeGlowCanvas.cs               the actual rendered glow
      NativeMethods.cs                click-through / topmost Win32 interop
    Settings/SettingsWindow.xaml(.cs) tray-accessible customization panel
    Properties/PublishProfiles/       self-contained single-file publish config
  installer/AuraWin.iss               Inno Setup script for a real installer
```

## Roadmap / known limitations

- "Launch at login" is a working UI toggle but doesn't yet write the Windows
  registry Run key
- Per-monitor targeting isn't implemented yet — the overlay always spans the
  full virtual desktop (all monitors)
- No auto-update mechanism
- Sources that don't expose real album art to SMTC (some browser/web players)
  use a text-derived fallback palette rather than the actual cover art

## License

*(Add whichever license you'd like — MIT is a common choice for a small utility
like this. Create a `LICENSE` file in the repo root and reference it here.)*

## Credits

Inspired by [Lumn](https://www.trylumn.xyz/) for macOS.
