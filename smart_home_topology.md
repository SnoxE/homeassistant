# Smart Home System Architecture and Topology (Bedroom)

Technical, network, and logical documentation for the bedroom lighting system based on **Matter over Thread** controlled by **Home Assistant**.

---

## 1. Component List and System Roles

| Device | Model / Specification | Power Supply | Thread Network Role | Location / Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **Server / Host** | Linux Host Server (x86_64, SSD, battery UPS) | 230V AC (built-in battery UPS) | **Application Host** (Home Assistant, Matter Server, OTBR) | Desk / Technical cabinet |
| **Radio Adapter** | SONOFF ZBDongle-E (Silicon Labs EFR32MG21) | USB (shielded USB 2.0 extension cable) | **Thread Border Router (RCP)** | Positioned away from host chassis |
| **Bulb 1** | IKEA KAJPLATS E27 (`light.kajplats_e27_ws_globe_1055lm`) | 230V AC (continuous mains power) | **Thread Router (FTD - Mesh Repeater)** | Bedside lamp 1 (ÅRSTID) |
| **Bulb 2** | IKEA KAJPLATS E27 (`light.kajplats_e27_ws_globe_1055lm_2`) | 230V AC (continuous mains power) | **Thread Router (FTD - Mesh Repeater)** | Bedside lamp 2 (ÅRSTID) |
| **Bulb 3** | IKEA KAJPLATS E27 RGB (`light.sypialnia_floor_lamp_corner`) | 230V AC (continuous mains power) | **Thread Router (FTD - Mesh Repeater)** | Corner floor standing lamp |
| **Remote 1** | IKEA BILRESA (Scroll wheel) | CR2032 coin-cell battery | **Sleepy End Device (SED)** | Nightstand / Bedside |
| **Remote 2** | IKEA BILRESA (Dual button: I / O) | CR2032 coin-cell battery | **Sleepy End Device (SED)** | **Mobile / Free-roaming** (bed, desk, couch) |

---

## 2. Layered Architecture Diagram

```text
+--------------------------------------------------------------------------+
|                                HOST LAYER                                |
|  Linux Host Server (SSD + built-in battery UPS)                          |
|  - OS: Ubuntu Server / Docker Compose                                    |
|  - Services: Home Assistant Core, Python Matter Server, OpenThread (OTBR)|
+------------------------------------+-------------------------------------+
                                     |
                                     | Dedicated USB 2.0 port
                                     | + Shielded USB 2.0 extension cable (1 m)
                                     v
+--------------------------------------------------------------------------+
|                           BORDER ROUTER LAYER                            |
|  SONOFF ZBDongle-E (Silicon Labs EFR32MG21)                              |
|  - Firmware: OpenThread RCP (Radio Co-Processor)                         |
|  - Band: 2.4 GHz IEEE 802.15.4 (Thread channel: 15 / 20 / 25)            |
+------------------------------------+-------------------------------------+
                                     |
               ~~~~~~~~ IPv6 / Thread Mesh (Radio Network) ~~~~~~~~
               |                                                  |
               v                                                  v
   +-----------------------+                          +-----------------------+
   |   KAJPLATS Bulb 1     |                          |   KAJPLATS Bulb 2     |
   |  (Left Bedside Lamp)  |                          |  (Right Bedside Lamp) |
   |   [Thread Router FTD] |                          |   [Thread Router FTD] |
   +-----------------------+                          +-----------------------+
               ^                                                  ^
               |                                                  |
               +------------------------+-------------------------+
                                        :
               ....... Wireless Thread Communication (SED) .......
                                        :
               +------------------------+-------------------------+
               |                                                  |
               v                                                  v
+-------------------------------+              +-------------------------------+
|  IKEA BILRESA (Scroll Wheel)  |              |   IKEA BILRESA (Dual Button)  |
|  - Battery End Device (SED)   |              |  - Battery End Device (SED)   |
|  - Bedside zone               |              |  - Mobile / Free-roaming      |
+-------------------------------+              +-------------------------------+
```

---

## 3. Thread Mesh Operating Principles

1. **Self-Healing Mesh (Full Thread Devices - FTD):**
   * The KAJPLATS bulbs are connected to continuous 230V mains power and act as **Thread Routers (Repeaters)**.
   * They form the backbone of the mesh—if a device has a weaker direct signal to the server, packets automatically hop through the nearest bulb.
2. **Sleepy End Devices (SED):**
   * The BILRESA remotes spend most of their time in deep sleep to preserve battery life.
   * They wake up immediately upon a physical button press or wheel rotation, transmit the Matter event frame to the nearest bulb router, and return to sleep.
3. **Mobile Remote Roaming:**
   * The dual-button remote is not fixed to a single wall location. As it moves around the room, it automatically communicates through the bulb router with the highest Link Quality Indicator (LQI).

---

## 4. Logical Entity and Automation Mapping in Home Assistant

### Light Groups in Home Assistant
* `light.bedroom_bedside_lamps` – Grouped ÅRSTID bedside lamps (left + right).

---

### Remote 1: BILRESA with Scroll Wheel (Bedside Zone)
Dedicated to precise, relaxing brightness and color-temperature adjustment directly from the bed.

| Remote Event | Target in Home Assistant | Action |
| :--- | :--- | :--- |
| **Single wheel click** | `light.bedroom_bedside_lamps` | Toggle (On / Off) bedside lamps |
| **Double wheel click** | `light.bedroom_bedside_lamps` | Night mode: bedside lamps set to 3% warm light (2200K) |
| **Long wheel press** | Wheel Mode Toggle | Switch wheel mode between **Brightness** and **Color Temperature (2200K–4000K)** |
| **Rotate clockwise (right)** | `light.bedroom_bedside_lamps` | Smoothly brighten (`+8%`, `transition: 0.2s`) or cooler color temp (`+150K`) |
| **Rotate counter-clockwise (left)** | `light.bedroom_bedside_lamps` | Smoothly dim (`-8%`, `transition: 0.2s`) or warmer color temp (`-150K`) |

---

### Remote 2: BILRESA Dual Button (Mobile / Free-Roaming)
Dedicated to quick control and switching lighting scenes on the ÅRSTID bedside lamps from anywhere in the room.

| Remote Event | Matter Event Type | Target in Home Assistant | Action |
| :--- | :--- | :--- | :--- |
| **Button "I" (Single click)** | `multi_press_1` | `light.bedroom_bedside_lamps` | Activate **Afternoon** scene (`80%` brightness, `3300K`) |
| **Buttons "I" + "O" (Click together)** | `multi_press_1` (both within 350ms) | `input_select.bedroom_hold_mode` | Toggle **Hold Mode** between **Brightness** and **Color Temperature (`2202K–4500K`)** (with brief visual pulse confirmation) |
| **Button "I" (Double click)** | `multi_press_2` | `light.bedroom_bedside_lamps` | Scene rotation: **Evening** (`2700K`, `50%`) $\rightarrow$ **Reading** (`3500K`, `80%`) $\rightarrow$ **Relax** (`2200K`, `20%`) |
| **Button "I" (Hold)** | `long_press` $\rightarrow$ `long_release` | `light.bedroom_bedside_lamps` | Smooth hardware ramp up: **Brighten to `100%`** (in Brightness mode) OR **Cooler white to `4500K`** (in Color Temp mode); freezes on release |
| **Button "O" (Single click)** | `multi_press_1` | `light.bedroom_bedside_lamps` / `all` | If bedside lamps are ON: turn off bedside lamps (`0.8s`). If bedside lamps are already OFF: turn off **ALL lights in the entire apartment** (`0.8s`) |
| **Button "O" (Double click)** | `multi_press_2` | `light.bedroom_bedside_lamps` + `light.sypialnia_floor_lamp_corner` | Night mode: bedside lamps (`3%`, `2200K`) + corner floor lamp (`20%`, keep color) |
| **Button "O" (Hold)** | `long_press` $\rightarrow$ `long_release` | `light.bedroom_bedside_lamps` | Smooth hardware ramp down: **Dim to `2%`** (in Brightness mode) OR **Warmer amber to `2202K`** (in Color Temp mode); freezes on release |

---

## 5. Best Practices and Deployment Rules

1. **Continuous Mains Power (230V):**
   Physical cord switches on the ÅRSTID lamps must remain switched ON at all times. Cutting power with the physical switch turns off the Thread router node.
2. **Antenna Electromagnetic Isolation:**
   Keep the SONOFF ZBDongle-E away from USB 3.0 ports using a shielded USB 2.0 extension cable (0.5–1 m) plugged into a USB 2.0 port.
3. **Pairing Order:**
   Always pair mains-powered KAJPLATS bulbs first (to establish the Thread router mesh), and then pair the battery-powered BILRESA remotes.
