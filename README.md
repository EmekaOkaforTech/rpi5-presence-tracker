# rpi5-presence-tracker

Real-time room presence tracking with moving floor plan icon and face-based identity — built for Raspberry Pi 5 + Frigate + Home Assistant.

Part of the [YouTube series](https://www.youtube.com/@YourChannel).

## What This Does

- Tracks **where** in the room a detected person is — the floor plan icon moves in real time
- Identifies **who** is in the room using local face recognition (no cloud)
- Runs entirely on the Raspberry Pi 5 (ARM64 native — no x86 hardware required)

## Prerequisites

- Raspberry Pi 5 with Frigate running (Video 3 of the series)
- Home Assistant with MQTT (Mosquitto) configured
- HACS installed in Home Assistant
- `custom:button-card` installed via HACS

## Stack

| Component | Purpose |
|-----------|---------|
| `codeproject/ai-server:arm64` | Face recognition engine (ARM64 native) |
| `skrashevich/double-take` | Frigate integration + face training UI |

## Setup

### 1. Clone and configure

```bash
git clone https://github.com/YOUR_REPO/rpi5-presence-tracker.git
cd rpi5-presence-tracker
cp .env.example .env
nano .env
```

Fill in your MQTT host, MQTT credentials, and Frigate URL in `.env`.

### 2. Configure Double Take

```bash
nano data/double-take/config/config.yml
```

Replace `YOUR_MQTT_HOST`, `YOUR_MQTT_USER`, `YOUR_MQTT_PASSWORD`, and `YOUR_FRIGATE_IP` with your real values.

### 3. Deploy

```bash
docker compose up -d
```

Wait 2–3 minutes for CodeProject.AI to download and install its face recognition module on first run.

### 4. Train face recognition

Open the Double Take UI at `http://pilab3.local:3000`.

For each person in your household:
1. Click **Train**
2. Create a new subject with the person's name
3. Upload 5–10 clear photos of their face (different lighting, angles)
4. Click **Save**

### 5. Add Home Assistant entities

Copy the contents of `homeassistant/packages/presence_entities.yaml` into your HA configuration.

If you use HA packages, place the file at `/config/packages/presence_entities.yaml` and ensure your `configuration.yaml` includes:
```yaml
homeassistant:
  packages: !include_dir_named packages
```

Restart Home Assistant.

### 6. Add automations

Copy the automations from `homeassistant/packages/presence_automations.yaml` into Home Assistant via:

**Settings → Automations → Create Automation → Edit as YAML**

Add each automation block individually.

### 7. Add the dashboard card

In your Home Assistant floor plan dashboard:

1. Edit the dashboard
2. Add card → Manual
3. Paste the contents of `homeassistant/dashboard.yaml`
4. Save

## Calibration

The position mapping is a direct linear transform from camera pixel coordinates to floor plan percentage. If the icon position doesn't accurately match where the person is on your floor plan, adjust the mapping in the automation:

```yaml
# Default: direct 1:1 mapping
{{ (cx / 640 * 100) | round(1) }}   # x
{{ (cy / 480 * 100) | round(1) }}   # y

# To shift or scale: apply offset and scale factors
{{ ((cx / 640 * 100) * 0.8 + 10) | round(1) }}   # example: scale 80%, offset 10%
```

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Icon not moving | Check automations are active; subscribe to `frigate/events` in MQTT Explorer to confirm events are flowing |
| Identity shows "Unknown" | Check Double Take UI — confirm face training completed; check `double-take/matches/+` MQTT topic for published matches |
| CodeProject.AI slow on first start | Normal — it downloads the face module on first run. Wait 3–5 minutes |
| Double Take can't reach Frigate | Confirm `FRIGATE_URL` uses port 5000 (HTTP), not 8971 (HTTPS) |
