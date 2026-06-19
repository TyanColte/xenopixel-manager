# Xenopixel v3 Font Manager

A browser-based SD card manager for **Xenopixel v3** lightsaber controllers. Edit font names and blade colors, preview and replace sound files, manage system sounds, tune hardware settings with sliders, and back up or restore your entire SD card — all from your browser, with no software to install.

```
┌─────────────────────────────────────────────────────────────────────┐
│  ⚔  XENOPIXEL MANAGER  v3                          [ Open SD Card ]│
├────────────┬────────────┬────────────┬────────────────────────────┤
│  Fonts     │Sys. Sounds │  Settings  │           Backup           │
└────────────┴────────────┴────────────┴────────────────────────────┘
```

---

## Requirements

| Requirement | Details |
|---|---|
| **Browser** | Chrome 86+ or Edge 86+ (desktop only) — requires the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API) |
| **Secure context** | `localhost` **or** a real HTTPS domain — `file://` and plain-HTTP LAN addresses are blocked by the browser |
| **SD card** | Xenopixel v3, mounted as a readable/writable drive |

> Firefox and Safari do not support the File System Access API and will not work.

---

## Installation

### Option 1 — Docker (quickest)

```sh
docker run -d \
  --name xenopixel-manager \
  -p 8080:80 \
  --restart unless-stopped \
  ghcr.io/tyancolte/xenopixel-manager:latest
```

Open **http://localhost:8080** in Chrome or Edge.

Multi-arch image (linux/amd64 + linux/arm64) — runs on x86 desktops and ARM servers like Raspberry Pi.

---

### Option 2 — Docker Compose

Save this as `docker-compose.yml`:

```yaml
services:
  xenopixel-manager:
    image: ghcr.io/tyancolte/xenopixel-manager:latest
    container_name: xenopixel-manager
    ports:
      - "8080:80"
    restart: unless-stopped
```

```sh
docker compose up -d
```

---

### Option 3 — Serve the HTML file directly

The entire app is a single `index.html`. Any local HTTP server on `localhost` works — no build step needed.

**Python 3** (built-in, zero dependencies):
```sh
# from the directory containing index.html
python3 -m http.server 8080
```

**Node.js / npx:**
```sh
npx serve -l 8080
```

Open **http://localhost:8080** in Chrome or Edge.

> `localhost` counts as a secure context so the File System Access API works. Plain `file://` does not.

---

### Option 4 — Behind a reverse proxy (LAN / remote access)

The Docker container serves plain HTTP on port 80. Wrap it with a TLS-terminating proxy to use it from another device on your network (e.g. a laptop plugged into the SD card reader on a different machine).

**Caddy** — automatic HTTPS, zero certificate management:
```caddyfile
xenopixel.example.com {
    reverse_proxy localhost:8080
}
```

**nginx:**
```nginx
server {
    listen 443 ssl;
    server_name xenopixel.example.com;
    # ... your SSL certificate config ...

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
    }
}
```

> The container already sets `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: require-corp` headers — they pass through Caddy and nginx unchanged.

---

## Connecting to your SD card

1. Plug your Xenopixel v3 SD card into a card reader connected to the computer running the browser
2. Click **Open SD Card**
3. Select the **root of the SD card** — the folder that directly contains the numbered font folders (`1`, `2`, … `34`) and the `setting/` directory
4. Grant read/write permission when prompted

The app reads and writes the card in place — no files are staged or copied to a temp location.

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                         [ ⚔ ]                                      │
│                                                                     │
│                  Xenopixel v3 Font Manager                         │
│          Manage fonts, sounds and settings for your                 │
│          Xenopixel v3 lightsaber directly in your browser.         │
│                                                                     │
│                     [ Open SD Card ]                               │
│                                                                     │
│           Requires Chrome or Edge  ·  HTTPS or localhost           │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Fonts

All 34 font slots are displayed as a card grid. Each card shows the slot number, font name, blade color accent (derived from the `fontconfig.ini` RGB value), and file count.

```
┌─────────────────────────────────────────────────────────────────────┐
│  Fonts                                [ + Import ]  [ ↕ Sort ]     │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │░░░░░░░░░░░░░░░░░░│  │░░░░░░░░░░░░░░░░░░│  │                  │  │
│  │  SLOT 01         │  │  SLOT 02         │  │  SLOT 03         │  │
│  │  Proffie Gold    │  │  Dark Templar    │  │  (empty)         │  │
│  │  [18] [●]        │  │  [22] [●]        │  │                  │  │
│  │              01  │  │              02  │  │              03  │  │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
  ↑ radial glow tinted to blade color        ↑ ghost slot watermark
```

### Drag to swap slots

Drag any card onto another slot to swap their positions — all sound files, `fontconfig.ini`, and effect parameters move with the font.

### Font editor panel

Click a card to open the editor:

```
┌─────────────────────────────────────────────────────────────────────┐
│  ←  ████  PROFFIE GOLD  ·  Slot 1                  [ Save ] [ ✕ ] │
├─────────────────────────────────────────────────────────────────────┤
│  Font Name       [ Proffie Gold__________________________ ]         │
│  Blade Color     R [ 58 ]  G [ 114 ]  B [ 232 ]   ██████████       │
│                                                                     │
│  ─── Sound Categories ──────────────────────────────────────────── │
│                                                                     │
│  Hum  (1)                          [▶] [↑ Replace]  [↺ All]       │
│    hum (1).wav                                                      │
│                                                                     │
│  Ignition  (3)               [+ Add] [▶] [↑ Replace]  [↺ All]    │
│    in (1).wav   in (2).wav   in (3).wav                            │
│                                                                     │
│  Clash  (8)                  [+ Add] [▶] [↑ Replace]  [↺ All]    │
│    clash (1).wav … clash (8).wav                                   │
│                                                                     │
│  … Swing · Retraction · Blaster · Lockup · Drag · Melt · …        │
└─────────────────────────────────────────────────────────────────────┘
```

| Action | What it does |
|---|---|
| **▶ Play** | Streams the WAV file through your browser audio |
| **↑ Replace** | Pick a single `.wav` to overwrite that file |
| **+ Add** | Add one or more `.wav` files to the category |
| **↺ All** | Replace the entire category in one go — pick multiple files, they are sorted naturally and written as `category (1).wav`, `category (2).wav`, … |

Sound categories present in a Xenopixel v3 font:

| Key | Label | Key | Label |
|---|---|---|---|
| `hum` | Hum | `swing` | Swing |
| `in` | Ignition | `swingh` | Swing Heavy |
| `out` | Retraction | `swingl` | Swing Light |
| `clash` | Clash | `blaster` | Blaster Deflect |
| `force` | Force | `lock` | Lockup Loop |
| `beginlock` | Lockup Start | `endlock` | Lockup End |
| `drag` | Drag Loop | `begindrag` | Drag Start |
| `enddrag` | Drag End | `melt` | Melt Loop |
| `beginmelt` | Melt Start | `endmelt` | Melt End |
| `stab` | Stab | `spin` | Spin |
| `font` | Font Announce | `track` | Music Track |

---

## System Sounds

Manages the sounds in the `setting/` folder — the controller's own UI sounds, independent of any font.

```
┌─────────────────────────────────────────────────────────────────────┐
│  System Sounds                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Volume Sounds  (1–100)                   [ ↺ Replace All ]        │
│    volume (1).wav  [▶] [↑]                                         │
│    volume (2).wav  [▶] [↑]                                         │
│    …                                                                │
│                                                                     │
│  Power Sounds  (1–100)                    [ ↺ Replace All ]        │
│  Blade Sounds  (1–100)                    [ ↺ Replace All ]        │
│  Light Effect Sounds  (1–100)             [ ↺ Replace All ]        │
│  Saber Sounds  (1–7)                      [ ↺ Replace All ]        │
│                                                                     │
│  ─── Single Sounds ──────────────────────────────────────────────  │
│    poweron.wav  [▶] [↑]       poweroff.wav  [▶] [↑]               │
│    lowbattery.wav  [▶] [↑]    ready.wav  [▶] [↑]                  │
│    lock.wav  [▶] [↑]          force.wav  [▶] [↑]                  │
│    … (28 single-file system sounds)                                 │
└─────────────────────────────────────────────────────────────────────┘
```

**↺ Replace All** — pick a folder of `.wav` files at once; they are sorted naturally and written as `category (1).wav`, `category (2).wav`, …

---

## Saber Settings

Visual editor for `setting/config.ini`. All settings use human-readable labels with practical descriptions. Numeric settings with natural ranges show a slider alongside the value; settings requiring precise numbers use a text field.

```
┌─────────────────────────────────────────────────────────────────────┐
│  Saber Settings                                  [ Save Settings ] │
├─────────────────────────────────────────────────────────────────────┤
│  Audio                                                              │
│  ┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄  │
│  Max Volume                                                         │
│  Hard output ceiling — keep ≤100 to prevent distortion             │
│                                          ──────●──── 80            │
│  Current Volume                                                     │
│  Active playback level                   ────────●── 95            │
│  Blade Brightness                                                   │
│  LED output intensity                    ──────●──── 75%           │
│                                                                     │
│  Blade                                                              │
│  ┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄  │
│  Main Blade Pixel Count                                  [ 128 ]   │
│  A standard 92 cm neopixel blade has 128 pixels                    │
│  Side / Crossguard Pixel Count                           [   0 ]   │
│  Side Blade Ignition Delay                                          │
│  Delay before side blade lights          ●─────────── 0 ms         │
│                                                                     │
│  Behavior                                                           │
│  ┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄  │
│  Clash Sensitivity                                                  │
│  Lower = triggers on lighter contact     ──●──────── 5             │
│  Ignition Duration (ms)                              [ 1200 ]      │
│  Retraction Duration (ms)                            [  800 ]      │
│  Auto Power-Off (ms)                             [ 3600000 ]       │
│  Flash on Clash                                       [ On  ]      │
│                                                                     │
│  Motion Control  ·  Blade Modes  ·  Pre-On Timing  ·  Hardware     │
└─────────────────────────────────────────────────────────────────────┘
```

Settings are written to `setting/config.ini` on the SD card when you click **Save Settings**. The raw INI key names are preserved exactly; only the displayed labels change.

### Settings reference

| Section | Setting | Type | Range |
|---|---|---|---|
| Audio | Max Volume | slider | 0–100 |
| Audio | Current Volume | slider | 0–100 |
| Audio | Blade Brightness | slider | 0–100% |
| Blade | Main Blade Pixel Count | number | 1–500 |
| Blade | Side/Crossguard Pixel Count | number | 0–500 |
| Blade | Side Blade Ignition Delay | slider | 0–3000 ms |
| Behavior | Clash Sensitivity | slider | 1–20 |
| Behavior | Ignition Duration | number | ms |
| Behavior | Retraction Duration | number | ms |
| Behavior | Auto Power-Off | number | ms |
| Behavior | Flash on Clash | toggle | on/off |
| Motion | Enable Motion Control | toggle | on/off |
| Motion | Twist to Ignite / Retract | toggle | on/off |
| Motion | Twist Threshold | number | 50–1000 |
| Motion | Pull/Push to Ignite/Retract | toggle | on/off |
| Motion | Swing to Ignite | toggle | on/off |
| Motion | Swing Threshold | slider | 5–150 |
| Blade Modes | Torch / Velocity / Ghost / … | toggle | on/off |
| Pre-On Timing | Style 1–7 Pre-On Duration | number | 0–10000 ms |
| Hardware | Bluetooth | toggle | on/off |
| Hardware | Button Count | number | 1–2 |
| Hardware | Crystal Chamber Motor Speed | slider | 0–100% |

---

## Backup & Restore

```
┌──────────────────────────────┐   ┌──────────────────────────────┐
│  Backup                      │   │  Restore                     │
│                              │   │                              │
│  ☑  01 · Proffie Gold        │   │  Import a ZIP or folder      │
│  ☑  02 · Dark Templar        │   │  previously exported from    │
│  ☑  03 · Rebel Legacy        │   │  this app to restore fonts   │
│  ☐  04 · (empty)             │   │  onto the current SD card.   │
│  …                           │   │                              │
│  [ Select All / None ]       │   │  [ Restore from ZIP ]        │
│  [ Download ZIP ]            │   │  [ Restore from Folder ]     │
│  [ Save to Folder ]          │   │                              │
└──────────────────────────────┘   └──────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│  Import Fonts                                                    │
│                                                                  │
│  Add new fonts from a ZIP or folder into empty slots.           │
│  Existing fonts are not overwritten.                            │
│                                                                  │
│  [ Import from ZIP ]   [ Import from Folder ]                   │
└──────────────────────────────────────────────────────────────────┘
```

| Action | What it does |
|---|---|
| **Download ZIP** | Packages selected font folders into a `.zip` — download to your computer |
| **Save to Folder** | Writes selected fonts directly to a folder on your computer (uses File System Access API) |
| **Restore from ZIP** | Reads a `.zip` backup and writes each font into the matching slot on the SD card |
| **Restore from Folder** | Reads a backup folder and restores fonts to the SD card |
| **Import from ZIP/Folder** | Adds fonts from a pack into the first available empty slots without touching existing fonts |

---

## SD Card Structure

The app expects the standard Xenopixel v3 layout:

```
SD card root/
├── 1/                        ← Font slot 1
│   ├── fontconfig.ini        ← Font name, blade color, effect parameters
│   ├── hum (1).wav
│   ├── in (1).wav            ← Ignition
│   ├── in (2).wav
│   ├── out (1).wav           ← Retraction
│   ├── clash (1).wav
│   ├── clash (2).wav
│   └── …                     ← Other sound categories
├── 2/
│   └── …
├── …
├── 34/                       ← Maximum 34 font slots
└── setting/
    ├── config.ini            ← Hardware configuration (edited in Settings tab)
    ├── volume (1).wav        ← Volume change tones (up to 100)
    ├── power (1).wav         ← Power button sounds (up to 100)
    ├── blade (1).wav         ← Blade effect sounds (up to 100)
    ├── lighteffect (1).wav   ← Light effect sounds (up to 100)
    ├── saber (1).wav         ← Saber bank sounds (up to 7)
    ├── poweron.wav           ┐
    ├── poweroff.wav          │
    ├── lowbattery.wav        ├─ Single system sounds (28 files)
    ├── ready.wav             │
    ├── lock.wav              │
    └── …                     ┘
```

### `fontconfig.ini` format

```
FontName=(R,G,B),A,B,C,D,E,F,G,H
```

- `FontName` — the display name shown on the saber (and in the app)
- `(R,G,B)` — blade color as 0–255 RGB values; this drives the blade accent color throughout the UI
- `A–H` — eight effect parameter integers controlling hum style, blaster behavior, force effect, lockup type, and more

Example:
```
Proffie Gold=(255,165,0),1,0,1,0,0,0,200,500
```

---

## Building from source

```sh
git clone https://github.com/tyancolte/xenopixel-manager.git
cd xenopixel-manager
docker build -t xenopixel-manager .
docker run -d -p 8080:80 xenopixel-manager
```

Multi-arch builds (linux/amd64 + linux/arm64) are produced automatically by GitHub Actions on every push to `main` and on every tagged release (`v*.*.*`). Images are published to `ghcr.io/tyancolte/xenopixel-manager`.

---

## License

MIT
