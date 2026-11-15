# Home Assistant + OpenThread Border Router + Matter Server (Docker)

Docker Compose stack for **Home Assistant**, **OpenThread Border Router (`otbr`)**, and **Python Matter Server (`matter-server`)** with a **SONOFF ZBDongle-E** (flashed with OpenThread RCP firmware) for controlling Matter-over-Thread smart home devices.

---

## 1. Repository Structure

- [`docker-compose.yml`](./docker-compose.yml) — Host-networked Docker Compose stack (`homeassistant`, `matter-server`, `otbr`).
- [`config/configuration.yaml`](./config/configuration.yaml) — Home Assistant configuration (`light.bedroom_bedside_lamps`, `input_select.bedroom_scene_rotation`, `input_select.bedroom_hold_mode`).
- [`config/scenes.yaml`](./config/scenes.yaml) — Bedroom lighting scenes (`Afternoon`, `Evening`, `Reading`, `Relax`, `Night Mode`).
- [`config/automations.yaml`](./config/automations.yaml) — IKEA BILRESA Dual-Button automations (single-click scenes, simultaneous `I+O` hold-mode toggle, stepless hardware brightness & color-temperature fades).
- [`topologia_systemu_smart_home.md`](./topologia_systemu_smart_home.md) — Smart home device and automation topology.

---

## 2. Deploying on a Linux Server

### Step 1: Clone the Repository & Configure `.env`
```bash
git clone <YOUR_GITHUB_REPO_URL> ~/homeassistant
cd ~/homeassistant
cp .env.example .env
```
Edit `.env` to set your server's network interface (`BACKBONE_IF`) and USB coordinator path (`ZBDONGLE_DEVICE`).

### Step 2: Copy Runtime State (`config/.storage` and `data/`)
To preserve existing Home Assistant accounts, Thread network keys, and paired Matter devices, copy the git-ignored folders to the target server:
```powershell
scp -r ./config/.storage <USER>@<SERVER_HOST>:~/homeassistant/config/
scp -r ./data <USER>@<SERVER_HOST>:~/homeassistant/
```
Then ensure file ownership on the server:
```bash
sudo chown -R $USER:$USER ~/homeassistant
```

### Step 3: Start the Stack
```bash
cd ~/homeassistant
docker compose up -d
```

---

## 3. Web Interfaces

- **Home Assistant UI**: `http://<SERVER_IP>:8123`
- **OTBR Web UI**: `http://<SERVER_IP>:80` (REST API on `:8081`)
- **Matter Server WebSocket**: `ws://127.0.0.1:5580/ws`
