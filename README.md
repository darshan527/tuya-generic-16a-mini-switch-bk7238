# Tuya Generic 16A Mini Switch (BK7238 / T1) — ESPHome Firmware & Conversion Guide

This repository contains the complete documentation, ESPHome configuration (`mini-switch.yaml`), and compiled firmware binaries for liberating the **Tuya Generic 16A Mini Smart Switch** based on the **Beken BK7238 (T1)** microcontroller over-the-air (without soldering or serial wires), restoring all original Tuya features and adding local Home Assistant controls.

---

## Device Information

* **Device Name**: Generic 16A Mini Switch
* **Chipset**: Beken BK7238 (Tuya T1 / T1-CHL / AU5101XA)
* **Flash**: 2MB (2048 KiB) / 256 KiB RAM
* **Original Tuya Module**: T1
* **Stock Tuya Firmware**: Main module: `v1.0.0(1.1.9)` / MCU module: `v1.1.9`
* **OpenBeken Device Profile**: [Tuya Generic 16A Mini Switch](https://openbekeniot.github.io/webapp/devices/Tuya_Generic_16A_Mini_Switch.html)
* **UnTuyaOS3 Tool**: [tuya-cloudcutter/UnTuyaOS3](https://github.com/tuya-cloudcutter/UnTuyaOS3)
* **Elektroda Forum Discussion**: [Topic 4161841](https://www.elektroda.com/rtvforum/topic4161841.html)

### Pin Mapping

| Pin | Function | Description |
| :--- | :--- | :--- |
| **P6** | External Switch Input | Wall/rocker toggle switch or push button input (`INPUT_PULLUP`) |
| **P15** | Onboard Button | Physical momentary button on the switch casing (`INPUT_PULLUP`, inverted) |
| **P17** | Status LED | Indicator LED (Active High) |
| **P26** | Relay | Main 16A power relay output |

---

## Features Implemented in this Firmware

1. **Native Home Assistant Integration**: Fast, local API control without cloud dependency.
2. **Indicator Light Mode (Select Dropdown)**:
   * `Follow Switch`: LED turns ON when the relay is ON, OFF when relay is OFF.
   * `Inverted Switch`: Acts as a night locator light (ON when relay is OFF).
   * `Always On`: LED remains continuously illuminated.
   * `Off Always`: LED stays completely off.
3. **External Switch Type (Select Dropdown)**:
   * `Rocker switch - flip to change state`: Standard 2-way/toggle behavior (toggles relay whenever the switch position changes).
   * `Rocker switch - state synchronous`: Switch ON = Relay ON; Switch OFF = Relay OFF.
   * `Button switch (Momentary)`: For momentary push buttons / bell press switches.
4. **Restart Status / Power-on Behavior (Select Dropdown)**:
   * `Restart Memory`: Remembers and restores the relay state prior to power loss.
   * `Power-off`: Always boots with the relay OFF.
   * `Power-on`: Always boots with the relay ON.
5. **Quick Power Cycle (Double-Flick Recovery)**:
   * If the wall switch/circuit breaker is toggled off and quickly turned back on within 5 seconds, it overrides the memory and forces the relay **ON**.
6. **Instant UI Execution**:
   * Changing any dropdown in Home Assistant or Web UI updates immediately without needing to toggle the relay.

---

## Initial Conversion Walkthrough (UnTuyaOS3)

The device was flashed over-the-air from factory Tuya firmware using [tuya-cloudcutter/UnTuyaOS3](https://github.com/tuya-cloudcutter/UnTuyaOS3):

```bash
git clone https://github.com/tuya-cloudcutter/UnTuyaOS3.git
cd UnTuyaOS3
```

### 1. Put Device into AP Pairing Mode
* Long-press the button on the switch for ~5 seconds until the LED starts blinking rapidly, then long-press again until it blinks slowly to activate the AP mode (`SmartLife-XXXX`).

### 2. Disconnect Wi-Fi Interface from NetworkManager & Run UnTuyaOS3
```bash
sudo nmcli device set <WIFI_INTERFACE> managed no
sudo ./UnTuyaOS3.sh -i <WIFI_INTERFACE>
```
* **Platform Selected**: `[1] T1`
* **Kickstart Firmware**: `esphome-kickstart-bk7238-tuya-2026-10-01.ota.ug.bin`
* UnTuyaOS3 connects to the open `SmartLife-XXXX` AP, injects the exploit, and uploads the kickstart image.

### 3. Connect to the Kickstart Web UI
* Once flashed, the device broadcasts a fallback AP (e.g. `kickstart-bk7238-tuya`).
* Connect to it, browse to `http://192.168.4.1`, and input your home Wi-Fi credentials.
* Note the IP address assigned to the switch on your local network (e.g., `http://192.168.20.137`).

### 4. Upload Final ESPHome Firmware
* Open `http://<DEVICE_IP>` (from kickstart) in your browser.
* Scroll down to **OTA Update**.
* Upload the compiled `firmware.uf2` (or `mini-switch.ota.ug.bin`).
* Click **Update**.

### 5. Wi-Fi Onboarding (Captive Portal)
* After updating, the switch boots into clean ESPHome and starts a Wi-Fi hotspot: `Mini-Switch-Fallback`.
* Connect to it from your phone or PC (open captive portal at `http://192.168.4.1`).
* Choose your home Wi-Fi and enter your password.
* The credentials will be saved directly into device flash memory. The switch will automatically connect to your network and pop up in Home Assistant!

---

## Compiling & Updating Firmware in Future

### Prerequisites

Create a Python virtual environment with ESPHome:
```bash
python3 -m venv venv
source venv/bin/activate
pip install esphome
```

### Compiling
```bash
esphome compile mini-switch.yaml
```

The output firmware files will be generated under:
* `.esphome/build/mini-switch/.pioenvs/mini-switch/firmware.uf2`
* `.esphome/build/mini-switch/.pioenvs/mini-switch/image_bk7238_app.ota.ug.bin`

### Flashing via OTA (Network)

You can flash over Wi-Fi directly using ESPHome:
```bash
esphome run mini-switch.yaml --device <DEVICE_IP>
```
Or upload `firmware.uf2` via the web browser at `http://<DEVICE_IP>`.

---

## Repository Files

* [`mini-switch.yaml`](mini-switch.yaml) - The full ESPHome configuration.
* [`firmware/firmware.uf2`](firmware/firmware.uf2) - Ready-to-flash full ESPHome firmware (UF2 format).
* [`firmware/mini-switch.ota.ug.bin`](firmware/mini-switch.ota.ug.bin) - Ready-to-flash full ESPHome OTA image (`.ota.ug.bin`).
* [`firmware/esphome-kickstart-bk7238-tuya-2026-10-01.ota.ug.bin`](firmware/esphome-kickstart-bk7238-tuya-2026-10-01.ota.ug.bin) - Kickstart firmware for virgin Tuya devices.

