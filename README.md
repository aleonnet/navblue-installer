# NavBlue — Site oficial

Site estático do **NavBlue** (HUD de navegação para motociclistas), hospedado no GitHub Pages. Três páginas, zero build:

| Página | URL | Propósito |
|---|---|---|
| `index.html` | `/` | **Landing page** do produto: parallax "night ride", mock animado do device, telas reais do app, links para as lojas, demo e instalador |
| `install.html` | `/install.html` | **Instalador web de firmware** para o device Waveshare ESP32-S3 AMOLED 1.75C, via Web Serial ([esptool-js](https://github.com/espressif/esptool-js)) |
| `demo.html` | `/demo.html` | **Demo cinematográfica** da engine de navegação (replay interativo gerado por `navblue_flutter_app/tools/simulation/guidance-sim_cinematic.py`) |

**Firmware publicado:** `1.4.1` (`firmware/navblue_waveshare_v1.4.1.bin`)

## Companion mobile app (Navblue)

Firmware flashed with this installer powers the **Navblue** HUD hardware. The **[Navblue]** mobile app works **together with the Navblue device**: it connects over **Bluetooth** to the external display and provides **routes and turn-by-turn navigation** for motorcyclists, so you can follow directions on the device without mounting the phone on the handlebar.

| Store | Link |
|--------|------|
| **Google Play** (Android) | [Navblue on Google Play](https://play.google.com/store/apps/details?id=com.energuide.navblue) |
| **App Store** (iOS) | [Navblue on the App Store](https://apps.apple.com/app/navblue/id6651858865) |

> The web installer only updates device firmware; routing and map features require the Navblue app on a phone paired to the board.

## Requirements

- **Browser**: Google Chrome or Microsoft Edge (desktop or Android)
- **Connection**: USB-C cable to the Waveshare board
- **Protocol**: Page must be served over HTTPS (GitHub Pages works out of the box)

> Safari and iOS are **not supported** (Web Serial API unavailable).

## Project Structure

```
navblue-device-web-installer/
  index.html            # Landing page do produto (self-contained)
  install.html          # Instalador web de firmware (self-contained, sem CDN)
  demo.html             # Demo cinematográfica da engine (artefato gerado; usa CDNs MapLibre/Fonts)
  index-v2.html         # esptool-js via CDN (referência histórica)
  index-v1.html         # versão ESP Web Tools (referência histórica)
  manifest.json         # Manifesto do firmware (versão, chip, caminho do binário)
  assets/               # Logo, favicon, badges, telas do app, recorte do device, ícones de manobra
  lib/
    esptool-bundle.js   # bundle local do esptool-js 0.5.7
  fonts/
    orbitron-latin.woff2
    dm-sans-latin.woff2
    dm-sans-latin-ext.woff2
  firmware/
    navblue_waveshare_v*.bin   # Binário mesclado (gerado pelo build script)
  .gitignore
  README.md
```

## Building the Firmware

The build script lives in the firmware project and outputs the merged binary here:

```bash
cd /path/to/navblue_esp32s3_waveshare-amoled-175C
./build-web-installer.sh
```

This will:
1. Compile the firmware via PlatformIO
2. Merge bootloader + partitions + app into a single binary via `esptool.py`
3. Copy the binary to `../navblue-device-web-installer/firmware/`
4. Update `manifest.json` with the current version

### Prerequisites

- PlatformIO CLI (`pio`)
- `esptool.py` (installed with PlatformIO or `pip install esptool`)
- Python 3 (for manifest update)

## Local Testing

**Do not** open `install.html` via `file://` in the browser. The installer loads `manifest.json` and firmware with `fetch()`; `file://` origins break that flow. Always serve this directory over HTTP(S). The landing (`/`) and demo (`/demo.html`) also work best served over HTTP.

Recommended local flow:

```bash
cd navblue-device-web-installer
python3 -m http.server 8080
```

Then open **Chrome or Edge** at:

```
http://localhost:8080/
```

(Use another free port if `8080` is taken — e.g. `python3 -m http.server 8080` → `http://localhost:8080/`.)

**Alternative:** `npx serve .` (may serve over HTTPS depending on the tool) — still use `localhost` or HTTPS so Web Serial is allowed.

> Web Serial requires **HTTPS** or **`http://localhost`**. For full testing, use a local server as above or deploy to GitHub Pages.

## Deploy to GitHub Pages

### Step 1 — Create the repository

```bash
cd navblue-device-web-installer

git init
git add .
git commit -m "Initial commit — navblue device web installer"

# Create repo on GitHub (requires gh CLI)
gh repo create navblue-installer --public --source=. --push
```

Or manually:
1. Go to https://github.com/new
2. Create repository named `navblue-installer`
3. Push:
   ```bash
   git remote add origin https://github.com/aleonnet/navblue-installer.git
   git branch -M main
   git push -u origin main
   ```

### Step 2 — Include the firmware binary

By default `.gitignore` excludes `firmware/*.bin`. To distribute the firmware via GitHub Pages, temporarily override:

```bash
git add -f firmware/navblue_waveshare_v1.4.1.bin
git commit -m "Add firmware v1.4.1 binary for web installer"
git push
```

### Step 3 — Enable GitHub Pages

1. Go to **Settings > Pages** in your repository
2. Under **Source**, select **Deploy from a branch**
3. Select **main** branch, **/ (root)** folder
4. Click **Save**

### Step 4 — Set the entry page

The `index.html` is already the main installer. The page will be live at:

```
https://aleonnet.github.io/navblue-installer/
```

### Updating firmware

When you release a new firmware version:

```bash
# 1. Build the new firmware
cd /path/to/navblue_esp32s3_waveshare-amoled-175C
./build-web-installer.sh

# 2. Commit and push the new binary
cd /path/to/navblue-device-web-installer
git add -f firmware/navblue_waveshare_v*.bin manifest.json
git commit -m "Update firmware to vX.Y.Z"
git push
```

GitHub Pages will automatically redeploy within minutes.

## How It Works

1. User opens the page in Chrome/Edge
2. Clicks **Connect Device**
3. Browser prompts to select the USB serial port (the Waveshare board)
4. esptool-js connects, detects the chip, and validates compatibility (ESP32-S3)
5. Page shows detected chip info and **Install Firmware** button
6. User clicks **Install Firmware** — firmware is downloaded and flashed with progress
7. Device is automatically reset after flashing

If the device isn't detected, the user may need to enter **Download Mode**:
- Hold **BOOT** button
- Press **RESET** button (while holding BOOT)
- Release **BOOT**

## Technology

- **esptool-js** 0.5.7 — Web Serial flashing (local bundle, no CDN dependency)
- **Canvas 2D** — Interactive particle background with network connections
- **Orbitron + DM Sans** — Typography (self-hosted woff2)
- Single HTML file, zero build step, fully self-contained

## Version History

| File | Description |
|---|---|
| `index.html` | **Current** — esptool-js self-contained (all assets local) |
| `index-v2.html` | esptool-js via unpkg CDN (reference) |
| `index-v1.html` | ESP Web Tools with external modals (reference) |
