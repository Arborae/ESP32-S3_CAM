# ESP32-S3 CAM → Gesture Server (MediaPipe + MQTT + Home Assistant)

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)
[![Docker build](https://img.shields.io/github/actions/workflow/status/arborae/ESP32-S3_CAM/docker.yml?label=Build%20%26%20Publish&logo=github)](https://github.com/arborae/ESP32-S3_CAM/actions/workflows/docker.yml)
[![GHCR](https://img.shields.io/badge/GHCR-images-blue?logo=docker)](https://github.com/arborae/ESP32-S3_CAM/pkgs/container/esp32-s3_cam)

> MJPEG stream from an ESP32-S3 + **hand gesture** recognition (MediaPipe) + **pinch distance** (thumb–index) + **MQTT** with autodiscovery for Home Assistant. Web UI with overlay **800×600 @ 25fps**.
>
> Stream MJPEG da ESP32-S3 + riconoscimento **gesture mani** (MediaPipe) + **pinch distance** (pollice–indice) + **MQTT** con autodiscovery per Home Assistant. UI web con overlay **800×600 @ 25fps**.

<p align="center">
  <img src="docs/images/demo_stream.gif" alt="Demo Stream" width="720"/>
</p>

<p align="center">🇬🇧 <b>English</b> · <a href="#-italiano">🇮🇹 Italiano</a></p>

---

# 🇬🇧 English

## 🔥 Main features

- ✅ **MJPEG stream** from ESP32-S3/OV2640 → Docker server
- ✋ **Hand gestures** optimized (victory V, OK, open hand, four corners)
- 🤏 **Pinch** (thumb–index): distance in **px** / **normalized** + **trend** (_opening / closing / steady_) and dedicated **left/right** modes (activated from the corners, auto-deactivation, binary sensors for Home Assistant)
- 📡 **MQTT** with **Home Assistant Discovery** (auto-created sensors)
- ⚙️ **Web UI** ready: `/`, `/stream`, `/status`, `/snapshot.jpg`, `/health`
- 🐳 **Docker Compose** and **.env** ready to go (no secrets in the code)
- 💾 **Persistent settings**: advanced preferences and gesture selection saved across restarts

## 🧰 Hardware

- **ESP32-S3** board with **OV2640** camera (ESP32-S3 CAM)
- 2.4 GHz Wi-Fi network
- A host with **Docker** (AMD64/ARM64) — e.g. mini-PC, NAS, RPi4
- (Optional) **Home Assistant** + **MQTT** broker

> Note: configure your board's pinout and prefer **SVGA 800×600** for the smoothest result.

## 🧱 Architecture

```text
ESP32-S3 (OV2640)
   │ MJPEG (HTTP)
   ▼
Gesture Server (Docker: Flask + OpenCV + MediaPipe Hands)
   │ Overlay + Feature extraction (gesture + pinch)
   ├── Web UI: / , /stream , /status , /snapshot.jpg , /health
   └── MQTT Publish (Home Assistant Discovery)
           │
           ▼
     Home Assistant (sensors + automations)
```

## 🚀 Quick start (Docker)

### With docker compose (recommended)
```bash
cd server
cp .env.example .env
# edit the placeholders in .env (do not commit `.env`)
docker compose up -d
# UI: http://<host>:12345/
```

### Main variables (.env)
```
SOURCE_URL=http://<ESP32-IP>/stream
MQTT_HOST=<MQTT-IP>
MQTT_PORT=1883
MQTT_USER=<user>
MQTT_PASSWORD=<password>
MQTT_BASE_TOPIC=gesture32
MQTT_DISCOVERY_PREFIX=homeassistant
TARGET_FPS=25
PINCH_DEADZONE_PX=8
PINCH_HISTORY=8
```

## 📦 Images on GHCR (GitHub Container Registry)

> Workflow included: on every push to `main` it builds and publishes to GHCR.

```bash
docker pull ghcr.io/arborae/esp32-s3_cam:latest
docker run --rm -p 12345:12345 \
  -e SOURCE_URL="http://<ESP32-IP>/stream" \
  -e MQTT_HOST="<MQTT-IP>" -e MQTT_PORT="1883" \
  -e MQTT_USER="<user>" -e MQTT_PASSWORD="<password>" \
  -e MQTT_BASE_TOPIC="gesture32" \
  ghcr.io/arborae/esp32-s3_cam:latest
```

Available tags: `:latest` (main), `:X.Y.Z` (releases), `:main-<sha>`.

## 📡 MQTT + Home Assistant

Sensors via discovery:
- `sensor.esp32_gesture` — current gesture label
- `sensor.esp32_gesture_confidence` — confidence (%)
- `sensor.esp32_hands_count` — detected hands
- `sensor.esp32_pinch_distance_px` — thumb–index distance (px)
- `sensor.esp32_pinch_distance_norm` — normalized distance (0..1)
- `sensor.esp32_pinch_state` — `opening` / `closing` / `steady`
- `binary_sensor.esp32_pinch_mode_left` — left pinch mode on/off
- `binary_sensor.esp32_pinch_mode_right` — right pinch mode on/off

<p align="center">
  <a href="docs/images/sensori_mqtt.jpg">
    <img src="docs/images/sensori_mqtt.jpg" alt="MQTT sensors in Home Assistant" width="360">
  </a>
</p>

Automation example:
```yaml
alias: Zoom with pinch
trigger:
  - platform: state
    entity_id: sensor.esp32_pinch_state
    to: 'opening'
  - platform: state
    entity_id: sensor.esp32_pinch_state
    to: 'closing'
action:
  - choose:
      - conditions: "{{ is_state('sensor.esp32_pinch_state','opening') }}"
        sequence:
          - service: script.zoom_in
      - conditions: "{{ is_state('sensor.esp32_pinch_state','closing') }}"
        sequence:
          - service: script.zoom_out
mode: restart
```

## 🧩 ESP32 firmware (Arduino IDE)

1. Install **ESP32** (Espressif) and select your **ESP32-S3** board.
2. Open `firmware/esp32s3-cam.ino` and fill in the placeholders (SSID/password, MQTT if used).
3. Upload and check the stream URL (e.g. `http://<ESP32-IP>/stream`).

Tips:
- If VLC won't open RTSP, use the **HTTP MJPEG** stream directly in the browser.
- For smoothness: JPEG quality ~80, 25 FPS, 800×600 resolution.

## 🛠️ Server endpoints

- `GET /` — UI (stream + values panel)
- `GET /stream` — MJPEG with overlay
- `GET /status` — gesture/pinch JSON
- `GET /snapshot.jpg` — single frame (800×600)
- `GET /health` — diagnostics

## 🧭 Settings panel

> Changes are applied automatically (auto-save ≈0.7 s); the **Save parameters** button stays available for a manual save. Each side-panel section can be collapsed/expanded with its **Hide/Show** button.

- **Status** — Video stream / MQTT / Last error indicators; **Enable debug** for detailed logging; **Log (debug)** panel.
- **Gesture MQTT** — minimum **confidence** before sending a gesture; **gesture list** to publish (excluded ones are not sent); **Save selection**.
- **Advanced Settings**
  - *A) Detection & sensitivity* — min confidence (0.5–0.95), motion sensitivity (px), temporal smoothing (averaged frames), gesture confirmation delay (ms).
  - *B) Pinch & complex interaction* — pinch threshold (normalized thumb–index distance), pinch stability (px) to switch steady↔opening/closing, pinch confirmation time (ms), corner area (%) for pointing gestures.
  - *C) Performance & filters* — processing frame rate (max FPS), frame size (320×240 / 640×480 / 800×600), brightness/contrast filter (±50%), auto-exposure compensation.
  - *D) MQTT & output* — MQTT update frequency (ms), float value precision, dynamic base topic, show MediaPipe landmarks, visual feedback.
- **Live preview** — Confidence bar/value and Pinch distance bar/value for the current frame.

## 🧯 Troubleshooting

- **UI shows but no gestures** → check the container logs, verify the stream and MediaPipe.
- **No sensors in HA** → confirm broker/credentials, `discovery_prefix`, and that HA uses the same broker.
- **Frame drops** → lower `TARGET_FPS`, reduce JPEG quality on the ESP32 side, check host CPU.

## 🤝 Contributing

- See **CONTRIBUTING.md**
- Templates for **Bug/Feature/PR** in `.github/ISSUE_TEMPLATE/` and `.github/pull_request_template.md`

## 📝 License

MIT — see [`LICENSE`](./LICENSE).

<br>

---
---

<a id="-italiano"></a>

# 🇮🇹 Italiano

## 🔥 Caratteristiche principali

- ✅ **Stream MJPEG** da ESP32-S3/OV2640 → server Docker
- ✋ **Gesture hands** ottimizzate (V di vittoria, OK, mano aperta, quattro angoli)
- 🤏 **Pinch** (pollice–indice): distanza **px** / **normalizzata** + **trend** (_opening / closing / steady_) e modalità dedicate **sinistra/destra** (attivazione dagli angoli, disattivazione automatica, sensori binari per Home Assistant)
- 📡 **MQTT** con **Home Assistant Discovery** (sensori auto-creati)
- ⚙️ **UI Web** pronta: `/`, `/stream`, `/status`, `/snapshot.jpg`, `/health`
- 🐳 **Docker Compose** e **.env** già pronti (no segreti nel codice)
- 💾 **Impostazioni persistenti**: preferenze avanzate e selezione gesti salvate tra i riavvii

## 🧰 Hardware

- Board **ESP32‑S3** con camera **OV2640** (ESP32‑S3 CAM)
- Rete Wi‑Fi 2.4 GHz
- Un host con **Docker** (AMD64/ARM64) — es. mini‑PC, NAS, RPi4
- (Opzionale) **Home Assistant** + broker **MQTT**

> Nota: configura il pinout della tua board e preferisci **SVGA 800×600** per la migliore fluidità.

## 🧱 Architettura

```text
ESP32-S3 (OV2640)
   │ MJPEG (HTTP)
   ▼
Gesture Server (Docker: Flask + OpenCV + MediaPipe Hands)
   │ Overlay + Feature extraction (gesture + pinch)
   ├── UI Web: / , /stream , /status , /snapshot.jpg , /health
   └── MQTT Publish (Home Assistant Discovery)
           │
           ▼
     Home Assistant (sensori + automazioni)
```

## 🚀 Prova rapida (Docker)

### Con docker compose (consigliato)
```bash
cd server
cp .env.example .env
# modifica i placeholder nel file .env (non committare `.env`)
docker compose up -d
# UI: http://<host>:12345/
```

### Variabili principali (.env)
```
SOURCE_URL=http://<IP-ESP32>/stream
MQTT_HOST=<IP-MQTT>
MQTT_PORT=1883
MQTT_USER=<user>
MQTT_PASSWORD=<password>
MQTT_BASE_TOPIC=gesture32
MQTT_DISCOVERY_PREFIX=homeassistant
TARGET_FPS=25
PINCH_DEADZONE_PX=8
PINCH_HISTORY=8
```

## 📦 Immagini su GHCR (GitHub Container Registry)

> Workflow incluso: alla push su `main` builda e pubblica su GHCR.

```bash
docker pull ghcr.io/arborae/esp32-s3_cam:latest
docker run --rm -p 12345:12345 \
  -e SOURCE_URL="http://<IP-ESP32>/stream" \
  -e MQTT_HOST="<IP-MQTT>" -e MQTT_PORT="1883" \
  -e MQTT_USER="<user>" -e MQTT_PASSWORD="<password>" \
  -e MQTT_BASE_TOPIC="gesture32" \
  ghcr.io/arborae/esp32-s3_cam:latest
```

Tag disponibili: `:latest` (main), `:X.Y.Z` (release), `:main-<sha>`.

## 📡 MQTT + Home Assistant

Sensori via discovery:
- `sensor.esp32_gesture` — label gesto corrente
- `sensor.esp32_gesture_confidence` — confidenza (%)
- `sensor.esp32_hands_count` — mani rilevate
- `sensor.esp32_pinch_distance_px` — distanza pollice‑indice (px)
- `sensor.esp32_pinch_distance_norm` — distanza normalizzata (0..1)
- `sensor.esp32_pinch_state` — `opening` / `closing` / `steady`
- `binary_sensor.esp32_pinch_mode_left` — modalità pinch sinistra attiva/disattiva
- `binary_sensor.esp32_pinch_mode_right` — modalità pinch destra attiva/disattiva

<p align="center">
  <a href="docs/images/sensori_mqtt.jpg">
    <img src="docs/images/sensori_mqtt.jpg" alt="Sensori MQTT in Home Assistant" width="360">
  </a>
</p>

Esempio automazione:
```yaml
alias: Zoom con pinch
trigger:
  - platform: state
    entity_id: sensor.esp32_pinch_state
    to: 'opening'
  - platform: state
    entity_id: sensor.esp32_pinch_state
    to: 'closing'
action:
  - choose:
      - conditions: "{{ is_state('sensor.esp32_pinch_state','opening') }}"
        sequence:
          - service: script.zoom_in
      - conditions: "{{ is_state('sensor.esp32_pinch_state','closing') }}"
        sequence:
          - service: script.zoom_out
mode: restart
```

## 🧩 Firmware ESP32 (Arduino IDE)

1. Installa **ESP32** (Espressif) e scegli la tua board **ESP32-S3**.
2. Apri `firmware/esp32s3-cam.ino` e compila i placeholder (SSID/password, eventuale MQTT).
3. Carica e verifica l’URL stream (es. `http://<ESP32-IP>/stream`).

Tips:
- Se VLC non apre l’RTSP, usa direttamente lo stream **HTTP MJPEG** nel browser.
- Per fluidità: qualità JPEG ~80, FPS 25, risoluzione 800×600.

## 🛠️ Endpoint del server

- `GET /` — UI (stream + pannello valori)
- `GET /stream` — MJPEG con overlay
- `GET /status` — JSON gesto/pinch
- `GET /snapshot.jpg` — frame singolo (800×600)
- `GET /health` — diagnostica

## 🧭 Pannello impostazioni

> Le modifiche vengono applicate automaticamente (auto-save ≈0,7 s) mentre il pulsante **Salva parametri** resta disponibile per un salvataggio manuale. Ogni sezione del pannello laterale può essere compressa/espansa con il pulsante **Nascondi/Mostra** dedicato.

- **Stato** — indicatori Video stream / MQTT / Ultimo errore; **Attiva debug** per il logging dettagliato; riquadro **Log (debug)**.
- **Gesture MQTT** — **confidence** minima prima di inviare una gesture; **lista gesture** da pubblicare (le escluse non vengono inviate); **Salva selezione**.
- **Advanced Settings**
  - *A) Rilevamento e sensibilità* — confidence minima (0,5–0,95), sensibilità movimento (px), smoothing temporale (frame mediati), ritardo conferma gesture (ms).
  - *B) Pinch e interazione complessa* — soglia di pinch (distanza normalizzata pollice–indice), stabilità pinch (px) per passare da steady a opening/closing, tempo conferma pinch (ms), area angolo (%) per le gesture di puntamento.
  - *C) Prestazioni e filtri* — frame rate elaborazione (FPS max), dimensione frame (320×240 / 640×480 / 800×600), filtro luminosità/contrasto (±50%), auto-exposure compensation.
  - *D) MQTT e output* — frequenza aggiornamento MQTT (ms), precisione valori float, topic base dinamico, mostra landmark MediaPipe, feedback visivo.
- **Preview dal vivo** — barra/valore di Confidence e Pinch distance del frame corrente.

## 🧯 Troubleshooting

- **UI si vede ma niente gesture** → controlla i logs del container, verifica stream e MediaPipe.
- **No sensori in HA** → conferma broker/credenziali, `discovery_prefix`, e che HA usi lo stesso broker.
- **Frame drop** → riduci `TARGET_FPS`, abbassa qualità JPEG lato ESP32, verifica CPU host.

## 🤝 Contribuire

- Vedi **CONTRIBUTING.md**
- Template per **Bug/Feature/PR** in `.github/ISSUE_TEMPLATE/` e `.github/pull_request_template.md`

## 📝 Licenza

MIT — vedi [`LICENSE`](./LICENSE).
