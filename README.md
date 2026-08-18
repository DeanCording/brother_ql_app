# Brother QL Printer App

[![Stars](https://img.shields.io/github/stars/Dodoooh/brother_ql_app)](https://github.com/Dodoooh/brother_ql_app/stargazers)
[![CI](https://img.shields.io/github/actions/workflow/status/Dodoooh/brother_ql_app/ci.yml?branch=main&label=CI)](https://github.com/Dodoooh/brother_ql_app/actions/workflows/ci.yml)
[![Docker Pulls](https://img.shields.io/docker/pulls/dodoooh/brother_ql_app)](https://hub.docker.com/r/dodoooh/brother_ql_app)
[![Image size](https://img.shields.io/docker/image-size/dodoooh/brother_ql_app/dev?label=image)](https://hub.docker.com/r/dodoooh/brother_ql_app/tags)
[![Release](https://img.shields.io/github/v/release/dodoooh/brother_ql_app)](https://github.com/dodoooh/brother_ql_app/releases)
[![Issues](https://img.shields.io/github/issues/Dodoooh/brother_ql_app)](https://github.com/Dodoooh/brother_ql_app/issues)
[![Licence](https://img.shields.io/badge/licence-CC%20BY--NC--SA%204.0-lightgrey)](LICENSE)

A web interface and REST API for Brother QL label printers. Compose a label in
the browser or send it from a script. The app renders it, queues it and prints
it over the network, over USB, or through a Linux device node.

[![Demo](https://img.shields.io/badge/DEMO-open%20in%20your%20browser-22d3ee?style=for-the-badge)](https://dodoooh.github.io/brother_ql_app/)

[![The compose view, light theme on the left and dark on the right](doc/images/compose-themes.png)](https://dodoooh.github.io/brother_ql_app/)

- **🖼 Image Printing**: Upload and print images effortlessly to create visually appealing labels.

- **📱 QR Code Generation**: Create and print QR codes for URLs, contact information, or any text data.

- **🔀 Combined Layouts**: Print text and QR codes together — or text and an uploaded image side by side — with customizable positioning.

- **📄 PDF Printing**: Upload a PDF and print selected pages, with per-page preview and fit/fill scaling.

- **👁️ Live Preview**: See how your labels will look before printing for all label types, with an instant client-side preview backed by a true-to-print server render.

- **🔤 Font Selection**: Choose a typeface and style (regular / bold / italic / bold italic) per label, or set a default for all of them. Ships with DejaVu and Liberation (metric-compatible with Arial, Times New Roman and Courier New); drop your own `.ttf` or `.otf` into the font directory and it appears in the picker within seconds — no restart, no rebuild.

- **⚙️ Custom Settings**: Fine-tune typeface, font size, label size, alignment, rotation, threshold, dithering and red printing. Copies (1–100) and cut mode are available directly in every compose section.

- **🗂 Print Queue**: Submit multiple jobs and have them printed sequentially. Pause/resume the queue, emergency-stop (cancel all waiting jobs), delete individual jobs, reprint with the same settings, or re-open a job's parameters — all from the Queue panel.

- **🛡 Large-batch Confirmation**: Printing 10 or more copies requires an explicit confirmation in the UI and an explicit flag in the API, so a big run is never started by accident.

- **🔗 API Support**: Seamlessly integrate with external systems like Home Assistant ❤️ via a comprehensive, documented API.

- **🖨 Multiple Printer Support**: Control multiple printers simultaneously via the API, enabling the use of different label sizes and configurations for various tasks.

- **🔄 Printer Keep Alive**: Prevent your printer from shutting down with the keep alive feature — keep it awake continuously, or only for a configurable window after each print. A toggle in the top bar makes it accessible from anywhere.

- **📱 Responsive Design**: Enjoy a smooth user experience on desktop, tablet, and smartphone devices.

- **🌙 Dark Mode**: Modern interface with automatic dark mode support based on system preferences.

- **📚 Swagger Documentation**: Explore and test the API using the built-in Swagger UI documentation.

- **🔄 Error Handling**: Robust error handling with informative messages and toast notifications.

## 📸 Screenshots

### Desktop
![Desktop UI](doc/images/screenshot_desktop.png)

### Dark Mode
![Dark mode UI](doc/images/screenshot_dark.png)

### QR Code
![QR Code label](doc/images/screenshot_qr.png)

### Mobile
![Mobile UI](doc/images/screenshot_mobile.png)


## Features

- Text, images, QR codes and PDFs, plus combined layouts (text with a QR code or
  an image beside it)
- A live preview rendered by the same code that drives the printer
- A print queue with per-job status, reprint and cancel
- Media detection: the app can read which roll is loaded and follow it
- Print alignment calibration per label type, with an optional bleed
- Round die-cut labels, and text that runs lengthwise along continuous tape
- Optional mains power control through a relay
- Keep-alive for network printers that fall asleep
- A documented REST API with a Swagger UI, for Home Assistant, Apple Shortcuts
  or a shell script

## Quick start

```yaml
# docker-compose.yml
services:
  brother_ql_app:
    image: dodoooh/brother_ql_app:dev
    container_name: brother_ql_app
    ports:
      - "5000:5000"
    volumes:
      - ./data:/app/data
    restart: unless-stopped
    environment:
      - PYTHONPATH=/app
      - FLASK_ENV=production
```

```bash
docker compose up -d
```

The interface is then at `http://<host>:5000` and the API documentation at
`http://<host>:5000/api/v1/ui/`. Set your printer address under Settings or with
`PUT /api/v1/settings`.

The image is published for `linux/amd64` and `linux/arm64`, which covers a
64-bit Raspberry Pi, a NAS and an ordinary server. There is no 32-bit build:
Pillow ships no armv7 wheel, so it would have to be compiled, and this image
deliberately carries no compiler. On a 32-bit system, build it yourself with
`docker build`.

The API is open to whoever can reach the port. That matches what it drives: a
label printer accepts a raster on port 9100 from anyone on the same network, with
or without this app in front of it. Give it a network you trust, or a reverse
proxy, and see `API_KEY` below if you run it without the interface.


## Printing from a script

```bash
curl -X POST http://<host>:5000/api/v1/text/print \
  -H 'Content-Type: application/json' \
  -d '{
        "text": "Shelf A-12",
        "settings": { "label_size": "62", "font_size": 50 }
      }'
```

The call returns a job id straight away. Poll `GET /api/v1/jobs/{id}` if you need
the result. Add `-H 'X-API-Key: ...'` if you set that variable. Image and PDF
upload, QR codes, combined layouts, printer status and queue control work the
same way and are documented in the Swagger UI, which is generated from the
specification the server routes on. The demo carries the same reference as a
[static page](https://dodoooh.github.io/brother_ql_app/api/), for reading it
without installing anything.

## Print queue

Jobs are printed one at a time, because the printer accepts one connection at a
time. The queue shows what each job is doing, including the phases a job passes
through while it waits for a printer to become available.

![The print queue](doc/images/queue-phases.png)

## Mains power control

A Brother QL starts up on its own as soon as mains power returns, at least the
QL-820NWB this was built against. That makes a relay in front of it a remote
power switch: cutting the supply switches the printer off, restoring it switches
the printer on, and nobody has to walk over and press the button.


The relay is anything that can be driven by a webhook: a Shelly, a Tasmota plug,
an ESPHome switch, or a Node-RED flow in front of one. The app closes it when a
job arrives at a printer that is not answering, and opens it again once the
configured window has passed.

![The mains power section in Settings](doc/images/settings-relay.png)

A printer that has just been switched on needs time. The app waits for it,
holding the job in the queue instead of failing it, and retries the print a few
times while the device settles. The waiting times come from watching a QL-820NWB
boot, so they are one printer's numbers for now.

The feature is off by default, and switching the printer *off* again is a
separate setting. `printer_auto_power_off_minutes` should match what the
printer's own menu shows, since the app subtracts it from the keep-alive window
and has no way to read it from the device.


## Configuration

Settings live in `data/settings.json`. Edit them in the interface or with
`PUT /api/v1/settings`. Any of them can also be sent with a single print request
under `settings`, where they override the stored value for that job only.

**Printer**

| Setting | Default | What it does |
|---|---|---|
| `printer_uri` | `tcp://192.168.1.100` | Also `usb://0x04f9:0x209b` or `file:///dev/usb/lp0` |
| `printer_model` | `QL-800` | For example `QL-820NWB` |
| `label_size` | `62` | The loaded media, for example `62`, `62red` or `d24` |
| `ipp_port` | `631` | Port used for status, media detection and the printer clock |
| `printers` | one entry | A list of printers to choose from. The fields above are the default one |

**Layout and rendering**

| Setting | Default | What it does |
|---|---|---|
| `font_size` | `50` | Starting size. Shrinks further if the text does not fit |
| `font_family` | `` | The default typeface, as a family name from `GET /api/v1/fonts` (e.g. `Liberation Sans`). Empty means the built-in default, `DejaVu Sans`. A family that is not installed falls back to the default at render time rather than failing the print, so a config naming a font you have not copied over yet still works.|
| `font_style` | `bold` | The default style within that family — `regular`, `bold`, `italic` or `bold_italic`. Defaults to `bold`. A style the family does not provide degrades to the closest one it has.|
| `alignment` | `left` | `left`, `center` or `right` |
| `vertical_alignment` | `middle` | `top`, `middle` or `bottom`, where the medium leaves room |
| `orientation` | `across` | `lengthwise` runs the text along continuous tape |
| `text_markup` | `false` | Honour `**bold**` and `*italic*`. Sets the base text lighter |
| `rotate` | `0` | Rotates the finished label by `0`, `90`, `180` or `270` |
| `threshold` | `70.0` | Where grey turns into black, in percent |
| `dither` | `false` | Dither instead of thresholding. Better for photographs |
| `red` | `false` | Two-colour printing on red media such as `62red` |
| `dpi_600`, `hq` | `false`, `true` | Higher resolution and quality mode, where the model supports it |
| `compress` | `false` | Compresses the raster on the way to the printer |
| `copies`, `cut_mode` | `1`, `each` | Number of labels and when to cut: `each`, `end` or `none` |
| `calibration` | `{}` | Per label type: offset in millimetres and a size correction |
| `bleed_mm` | `{}` | Per label type: prints into the unprintable margin. Experimental |

**Media detection**

| Setting | Default | What it does |
|---|---|---|
| `media_auto_switch` | `false` | Adopt the roll the printer reports as loaded |
| `owned_media` | `[]` | The media you actually have. Narrows an ambiguous roll to one |
| `media_preference` | `{}` | Which variant wins where a roll stays ambiguous |
| `media_memory` | `{}` | What was last used on each medium. Maintained by the app |

**Keep-alive and mains power**

| Setting | Default | What it does |
|---|---|---|
| `keep_alive_enabled` | `false` | Keeps a network printer from falling asleep |
| `keep_alive_interval` | `60` | Seconds between heartbeats |
| `keep_alive_mode` | `forever` | `timed` stops after the window below |
| `keep_alive_duration_seconds` | `7200` | Length of that window, measured from the last print |
| `relay_webhook_enabled` | `false` | Mains power control, see above |
| `relay_webhook_turn_on_url` | empty | POSTed to switch the printer on |
| `relay_webhook_turn_off_url` | empty | Empty means the turn-on URL is reused |
| `relay_webhook_turn_off_enabled` | `false` | Opt in separately to switching off |
| `relay_webhook_turn_off_delay_minutes` | `5` | Grace period after the keep-alive window closes |
| `printer_auto_power_off_minutes` | `10` | The printer's own timer. Subtracted from the window above |

Environment variables:

| Variable | Default | Purpose |
|---|---|---|
| `API_KEY` | unset | Requires `X-API-Key` on `/api/v1`. The bundled interface does not send it and stops working, so this suits API-only installations |
| `CORS_ORIGINS` | same-origin | Comma-separated list of allowed origins |
| `UPLOAD_FOLDER` | `src/uploads` | Where rendered labels and uploads are staged |
| `MAX_UPLOAD_IMAGE_PIXELS` | `50000000` | Larger images are refused before they are decoded |
| `MAX_PDF_PAGES` | `20` | Larger page selections are refused before rendering |
| `JOB_FILE_TTL_SECONDS` | `86400` | How long job files are kept |
| `RELAY_WEBHOOK_AUTHORIZATION` | unset | Sent as the relay webhook's `Authorization` header |
| `LOG_LEVEL` | `INFO` | Python logging level |

The relay credential is an environment variable and not a setting, because
`GET /settings` returns the configuration to any client that can read it.

## Tested printers and media

What has actually been printed, rather than what the label catalogue says is
possible. The app knows about far more media than this table lists; these are
the combinations somebody has held in their hand.

| Printer | Media | Tested by | Date | Version |
|---|---|---|---|---|
| QL-820NWB | 62 mm continuous (`62`) | Dodoooh | 2026-08-04 | 4.0.0-rc.1 |
| QL-820NWB | 12 mm continuous (`12`) | Dodoooh | 2026-08-04 | 4.0.0-rc.1 |
| QL-820NWB | 24 mm round die-cut (`d24`) | Dodoooh | 2026-08-04 | 4.0.0-rc.1 |

If you print on something that is not in the table, a line here is welcome:
printer, media, and whether anything needed calibration.


### Fonts

Text is rendered server-side with the typefaces installed in the container. The
image bundles two families' worth:

| Family | Styles | Notes |
|---|---|---|
| DejaVu Sans / Serif / Sans Mono | regular, bold, italic, bold italic | The default. `DejaVu Sans` + `bold` is what this app used before fonts were selectable, so existing labels are unchanged. |
| Liberation Sans / Serif / Mono | regular, bold, italic, bold italic | Metric-compatible with Arial, Times New Roman and Courier New. |

`GET /api/v1/fonts` lists what is actually available, grouped into families with
the styles each one provides:

```json
{
  "fonts": [
    { "family": "Liberation Mono", "styles": ["regular", "bold", "italic", "bold_italic"], "user_supplied": false }
  ],
  "default_family": "DejaVu Sans",
  "user_font_dir": "/app/data/fonts"
}
```

#### Adding your own

Copy a `.ttf`, `.otf` or `.ttc` into **`/app/data/fonts`** — `./data/fonts` on
the host with the stock `docker-compose.yml`. It shows up in the picker within a
few seconds; no restart and no image rebuild. The directory is created on
startup if it is missing, and is scanned recursively.

```bash
cp Inter-SemiBold.ttf ./data/fonts/
```

A face dropped in there **shadows a bundled one of the same name**, so you can
substitute your own cut of a family. Set `FONTS_DIR` to scan somewhere else.

Family names come from the font's own name table rather than its filename, which
is what keeps `DejaVuSans.ttf` and `DejaVuSansCondensed.ttf` as two separate
entries instead of one overwriting the other.

#### Choosing a font per label

Every compose form has a typeface and style pair, and the API takes them
alongside the existing size options:

```jsonc
// POST /api/v1/text/print
{ "text": "Widget XYZ", "settings": { "font_family": "Liberation Mono", "font_style": "bold" } }

// POST /api/v1/qrcode/print — the caption has its own font
{ "qr": { "data": "..." },
  "text": { "content": "Shelf B4", "font_family": "DejaVu Serif", "font_style": "italic" } }
```

Omit either field (or send an empty string) to inherit the saved default. An
unknown family or a style the family lacks falls back to the closest available
face rather than failing the print — a label in the wrong typeface beats no
label at all.

### Multiple Printers

To control more than one printer, define a `printers` array. Each entry describes a printer that can be selected via the API; the top-level fields above act as the default printer.

```json
{
    "printer_uri": "tcp://192.168.1.100",
    "printer_model": "QL-820NWB",
    "label_size": "62",
    "font_size": 50,
    "alignment": "left",
    "rotate": 0,
    "threshold": 70.0,
    "dither": false,
    "compress": false,
    "red": false,
    "keep_alive_enabled": true,
    "keep_alive_interval": 30,
    "ipp_port": 631,
    "printers": [
        {
            "id": "default",
            "name": "QL-820NWB (Office)",
            "printer_uri": "tcp://192.168.1.100",
            "printer_model": "QL-820NWB",
            "label_size": "62"
        },
        {
            "id": "label-station",
            "name": "QL-800 (Warehouse)",
            "printer_uri": "tcp://192.168.1.101",
            "printer_model": "QL-800",
            "label_size": "29"
        }
    ]
}
```

Each printer entry supports:

- `id`: A unique identifier used to reference the printer
- `name`: A human-readable label shown in the UI
- `printer_uri`: The URI of this printer (network or USB)
- `printer_model`: The model of this printer
- `label_size`: The default label size for this printer

### Environment Variables

The following environment variables can be set (e.g. in `docker-compose.yml` or via `docker run -e`):

- `API_KEY`: Opt-in API authentication. When set, every request to `/api/v1/*` must include the header `X-API-Key: <your-key>`. The bundled UI, static assets, Swagger UI and the health probes remain unauthenticated. When unset, the API is open.
- `CORS_ORIGINS`: Comma-separated list of allowed cross-origin origins (e.g. `https://home.example.com,https://hass.example.com`). When unset, only same-origin requests are allowed.
- `SECRET_KEY`: The Flask secret key. Set a stable value in production; if unset, an ephemeral random key is generated on each start.
- `ENABLE_SWAGGER_UI`: Set to `true` or `false` to force the Swagger UI on or off. When unset, the UI defaults to on unless `FLASK_ENV=production`.
- `UPLOAD_FOLDER`: Directory used to persist uploaded image/PDF files (job files and shared files). Defaults to an app-relative `uploads/` folder.
- `JOB_FILE_TTL_SECONDS`: How long persisted image/PDF job files are kept so they can be reprinted or re-opened (default `86400`, i.e. 24 hours).
- `SHARE_TTL_SECONDS`: How long files staged via the `/share` endpoint are kept before cleanup (default `3600`, i.e. 1 hour).

## 📔 API Documentation

The API is fully documented using OpenAPI/Swagger. You can access the interactive documentation at [http://localhost:5000/api/v1/ui/](http://localhost:5000/api/v1/ui/) when the application is running.

### Available Endpoints

**Settings & printers**
- **GET /api/v1/settings**: Get current settings
- **PUT /api/v1/settings**: Update settings
- **GET /api/v1/printers**: Get available printers
- **POST /api/v1/printers/status**: Check printer status
- **GET /api/v1/printers/keep-alive**: Get keep-alive status
- **PUT /api/v1/printers/keep-alive**: Start/stop the keep-alive feature

**Printing**
- **POST /api/v1/text/print**: Print text on a label
- **POST /api/v1/image/print**: Print image on a label
- **POST /api/v1/qrcode/print**: Print QR code on a label
- **POST /api/v1/label/text-qrcode**: Print combined text and QR code on a label
- **POST /api/v1/label/text-image**: Print an uploaded image and a text block side by side on a label
- **POST /api/v1/pdf/print**: Print selected pages of an uploaded PDF
- **POST /api/v1/share**: Generic share hand-off endpoint for mobile share shortcuts (image/PDF)

**Live preview** (true-to-print PNG render)
- **POST /api/v1/text/preview**, **/qrcode/preview**, **/label/preview**, **/image/preview**, **/pdf/preview**

**Print queue**
- **GET /api/v1/jobs**: List recent jobs (newest first)
- **GET /api/v1/jobs/{id}**: Get a single job's status
- **POST /api/v1/jobs/{id}/cancel**: Cancel a queued job
- **POST /api/v1/jobs/{id}/reprint**: Re-queue a job with the same settings
- **POST /api/v1/jobs/{id}/delete**: Delete a waiting or finished job
- **GET /api/v1/jobs/{id}/file**: Download a job's persisted image/PDF
- **POST /api/v1/jobs/clear**: Remove finished jobs
- **POST /api/v1/jobs/clear-all**: Remove all jobs
- **GET /api/v1/jobs/queue**: Queue control status (paused + counts)
- **POST /api/v1/jobs/pause** · **/jobs/resume** · **/jobs/stop**: Control queue processing

**Health**
- **GET /health**: Liveness probe reporting that the web application process is up
- **GET /health/printer**: Readiness probe reporting whether the configured printer is reachable

> **Large batches:** Any print request that would print 10 or more copies must include an explicit `confirm_large_batch` flag (boolean `true` for JSON endpoints, the string `"true"` for multipart endpoints). Without it the request is rejected with HTTP 400 and the error code `CONFIRMATION_REQUIRED`.

> **Optional `settings`:** On every print and preview endpoint, `settings` (and any field within it) is optional — anything omitted is taken from the app's saved configuration, with request fields overriding. So printing on the configured printer needs no `settings` at all.

> **Raw PNG previews:** The preview endpoints (`/text/preview`, `/qrcode/preview`, `/label/preview`, `/image/preview`) return the JSON wrapper `{"image": "data:image/png;base64,…"}` by default. Send `Accept: image/png` to instead receive the raw PNG bytes (with `X-Label-Width-Px` / `X-Label-Height-Px` headers) — handy for piping a preview straight to an `<img>`.

> **Automatic wrapping:** Long text is wrapped at word boundaries to fit the label (over-long words are hard-broken) instead of being truncated — for plain text, text+QR and QR captions, in both print and preview. It is on by default; disable per request with `text.wrap: false` (or `settings.text_wrap: false`).

> **Dry run:** Add `dry_run: true` to any print request to validate it end-to-end (render + printer reachability) **without** printing or queueing — ideal for endless (62 mm) media and CI. The response is `{ "ok": true, "dry_run": true, "printer_reachable": …, "would_print": { "label_size", "copies", "width_px", "height_px" } }`.

## 📤 Example API Usage

### **Text Printing**

Use the `/api/v1/text/print` endpoint to print formatted text.

```python
import requests

url = 'http://localhost:5000/api/v1/text/print'
payload = {
    "text": {
        "content": "Hello World!\nThis is a test print.",
        "font_size": 40,
        "alignment": "center"
    },
    "settings": {
        "printer_uri": "tcp://192.168.1.100",
        "printer_model": "QL-800",
        "label_size": "62",
        "rotate": 90,
        "threshold": 70.0,
        "dither": True,
        "compress": True,
        "red": False
    }
}

response = requests.post(url, json=payload)

if response.status_code == 200:
    print("Success:", response.json())
else:
    print("Error:", response.status_code, response.json())
```

### **Image Printing**

Use the `/api/v1/image/print` endpoint to upload and print an image.

```python
import requests
import json

url = 'http://localhost:5000/api/v1/image/print'
image_path = '/path/to/image.jpg'  # Replace with the path to your image
settings = {
    "settings": {
        "printer_uri": "tcp://192.168.1.100",
        "printer_model": "QL-800",
        "label_size": "62",
        "rotate": 180,
        "threshold": 70.0,
        "dither": False,
        "compress": False,
        "red": True
    }
}

with open(image_path, 'rb') as img_file:
    files = {
        'image': img_file
    }
    data = {
        'settings': json.dumps(settings["settings"])
    }
    response = requests.post(url, files=files, data=data)

if response.status_code == 200:
    print("Success:", response.json())
else:
    print("Error:", response.status_code, response.json())
```

### **QR Code Printing**

Use the `/api/v1/qrcode/print` endpoint to generate and print a QR code.

```python
import requests

url = 'http://localhost:5000/api/v1/qrcode/print'
payload = {
    "qr": {
        "data": "https://github.com/dodoooh/brother_ql_app",
        "box_size": 10,
        "border": 4,
        "error_correction": "M",
        "version": 1,
        "size": 400
    },
    "settings": {
        "printer_uri": "tcp://192.168.1.100",
        "printer_model": "QL-800",
        "label_size": "62",
        "rotate": 0,
        "threshold": 70.0,
        "dither": False,
        "compress": False,
        "red": False
    }
}

response = requests.post(url, json=payload)

if response.status_code == 200:
    print("Success:", response.json())
else:
    print("Error:", response.status_code, response.json())
```

### **Combined Text and QR Code**

Use the `/api/v1/label/text-qrcode` endpoint to print text and QR code together.

```python
import requests

url = 'http://localhost:5000/api/v1/label/text-qrcode'
payload = {
    "text": {
        "content": "Product: Widget XYZ\nSKU: 12345\nPrice: $19.99",
        "font_size": 30,
        "alignment": "left"
    },
    "qr": {
        "data": "https://github.com/dodoooh/brother_ql_app",
        "box_size": 10,
        "border": 4,
        "error_correction": "M",
        "position": "right",
        "size": 400,
        "version": 1
    },
    "settings": {
        "printer_uri": "tcp://192.168.1.100",
        "printer_model": "QL-800",
        "label_size": "62",
        "rotate": 0,
        "threshold": 70.0,
        "dither": False,
        "compress": False,
        "red": False
    }
}

response = requests.post(url, json=payload)

if response.status_code == 200:
    print("Success:", response.json())
else:
    print("Error:", response.status_code, response.json())
```

### **Printer Keep Alive**

Use the `/api/v1/printers/keep-alive` endpoint to control the printer keep alive feature.

```python
import requests

url = 'http://localhost:5000/api/v1/printers/keep-alive'

# Enable keep alive feature
payload = {
    "settings": {
        "printer_uri": "tcp://192.168.1.100",
        "printer_model": "QL-800"
    },
    "keep_alive": {
        "enabled": True,
        "interval": 300  # Interval in seconds (5 minutes)
    }
}

response = requests.post(url, json=payload)

if response.status_code == 200:
    print("Success:", response.json())
else:
    print("Error:", response.status_code, response.json())

# Disable keep alive feature
payload = {
    "settings": {
        "printer_uri": "tcp://192.168.1.100",
        "printer_model": "QL-800"
    },
    "keep_alive": {
        "enabled": False
    }
}

response = requests.post(url, json=payload)

if response.status_code == 200:
    print("Success:", response.json())
else:
    print("Error:", response.status_code, response.json())
```

## 🔄 Keep Alive

Network Brother QL printers tend to power themselves off automatically after a period of inactivity. The keep-alive feature periodically writes to the printer's raw port (`9100`) at the configured `keep_alive_interval` to keep the connection warm and reduce the chance of an unexpected power-off.

> **Note:** Keep-alive is a best-effort mitigation. The only guaranteed way to prevent the printer from shutting down is to disable the device's own power-saving setting (set **Auto Power Off = Off** in the printer's menu).

Keep-alive can run in two modes (configurable in the settings):

- **Forever**: keeps the printer awake continuously while enabled.
- **Timed**: keeps the printer awake only for a configurable window after each print (e.g. 2 hours), then lets it sleep until the next print.

A keep-alive toggle is always available in the top bar so it can be turned on or off from any view.

## 🗂 Print Queue

All print requests are queued and processed sequentially by a single background worker, so the printer (which accepts only one connection at a time) is never driven concurrently. The **Queue** panel lists recent jobs with their status (`queued`, `printing`, `done`, `failed`, `cancelled`) and lets you:

- **Pause/Resume** processing — a job that is already printing finishes; the next ones wait.
- **Stop** — an emergency stop that pauses the queue and cancels all waiting jobs (a printing job still finishes).
- **Reprint** a job with the same settings, or **Open** its parameters back into the form.
- **Delete** a single waiting or finished job, **Clear finished**, or **Clear all**.

The same controls are available via the `/api/v1/jobs/*` endpoints listed above. Image and PDF jobs keep their uploaded file for a configurable time (`JOB_FILE_TTL_SECONDS`, default 24 h) so they can be reprinted or re-opened.

## 📲 Share from your Phone

The generic `/share` endpoint lets you send a PDF or image straight from your phone into the print form using **Apple Shortcuts** (iOS) or **HTTP Shortcuts** (Android) — no PWA required.

Send the file as a `multipart/form-data` POST with a `file` field to `POST /api/v1/share`. The server detects whether it is a PDF or an image (via magic bytes, content type or extension), stages it under a random token, and responds with a `302` redirect to `/?share=<token>&type=<pdf|image>`. Opening that URL loads the file into the matching tab (PDF or Image), ready to print with a single tap.


## Development

```bash
python3 -m venv venv && source venv/bin/activate
pip install -r requirements-dev.txt
python src/app.py
```

Python 3.11 is required. The test suite needs a TrueType font installed
(`fonts-dejavu`); without one, about a third of it skips.

```bash
pytest
```

`src/api/openapi.yaml` is the source of truth for routing. An endpoint changes
there and in its controller, or it does not change.

## AI assistance

Yes, this project is built with AI assistance. It belongs to the toolchain like
anything else here, and what comes out of it is reviewed and tested by a human
before it lands. Keeping a project this size moving, mostly alone and in spare
time, would not otherwise be realistic.

## Contributing

Contributions are welcome. Open an issue or send a pull request. Reports from
printers and media that are not in the table above are useful too.

## Acknowledgments

- **DL6ER** for support for additional printer models and label types, and for
  USB printing
- **[MSanteler](https://github.com/MSanteler)** for finding and diagnosing the
  hardcoded label width, which meant every roll narrower than 62 mm printed at
  the wrong scale

## Changelog

[changelog.md](changelog.md).

## Licence

CC BY-NC-SA 4.0. See [LICENSE](LICENSE).
