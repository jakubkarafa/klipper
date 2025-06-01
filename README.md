# jakubkarafa/klipper – Klipper Fork with Eddy USB + DGUS Display Support

This is a custom fork of [Klipper](https://github.com/Klipper3d/klipper) that merges the official upstream with [Desuuuu's fork](https://github.com/Desuuuu/klipper).  

> ⚙️ This fork is actively used on a **Raspberry Pi 4** running **Fluidd** as the web interface.
> Access the printer from any browser on your local network to monitor and control it via Fluidd.

The goal is to bring the latest Klipper improvements while retaining compatibility with:

- ✅ **Creality Ender 6 stock DGUS touchscreen** (via Desuuuu’s DGUS support)
- ✅ **BigTreeTech Eddy USB** probe for Z-offset calibration
- ✅ **BIQU H2 V2S** printhead (included in config)

---

## 🚀 Why This Fork?

The official Klipper repository does not support the Ender 6’s stock DGUS display or the Eddy USB probe out of the box.  
This fork merges both official features and community patches to support these components while staying as close to upstream Klipper as possible.

---

## 📦 How to Install via KIAUH

We recommend using [KIAUH](https://github.com/th33xitus/kiauh) (Klipper Installation And Update Helper) to install Klipper, Moonraker, and Fluidd/Mainsail easily.

🛠️ **Why use KIAUH?**
- It simplifies Klipper setup and updates
- Automatically installs Moonraker and your web UI (like Fluidd or Mainsail)
- Manages multiple Klipper instances or MCU firmwares if needed

### 🔧 Installing KIAUH

If you don't already have KIAUH installed, you can install it with:

```bash
cd ~
git clone https://github.com/th33xitus/kiauh.git
cd kiauh
./kiauh.sh
```

From the menu, choose to install:
1. Klipper
2. Moonraker
3. Fluidd or Mainsail (your preferred web UI)

📌 Note: KIAUH doesn’t support selecting a custom Klipper fork directly through the UI.  
You can safely swap to this fork without breaking KIAUH integration:

1. Use KIAUH to install Klipper (official) as normal:
   ```
   ./kiauh/kiauh.sh
   ```
   > Select "1" to install Klipper, then proceed through Moonraker/Fluidd/Mainsail if needed.

2. After installation, **replace the Klipper folder with this fork**:
   ```bash
   cd ~/klipper
   git remote set-url origin https://github.com/jakubkarafa/klipper
   git fetch origin
   git checkout dgus-reloaded
   git reset --hard origin/dgus-reloaded
   ```

3. Rebuild the firmware if needed:
   ```bash
   make menuconfig
   make
   ```

This keeps all your KIAUH integrations working while using the latest features from this fork.

## 🖥️ Setting up DGUS Display (Ender 6)

Your Ender 6’s stock touchscreen will only function properly if flashed with DGUS-compatible firmware.

👉 **Flashing guide**:  
[DGUS-reloaded-Klipper Wiki – Flashing the firmware](https://github.com/Desuuuu/DGUS-reloaded-Klipper/wiki/Flashing-the-firmware)

### 📌 Summary:
1. Download DGUS firmware from [Desuuuu’s Releases](https://github.com/Desuuuu/DGUS-reloaded-Klipper/releases)
2. Extract to a FAT32 SD card
3. Insert into display and power on – it will auto-flash
4. Remove SD and reboot

More details: https://github.com/Desuuuu/DGUS-reloaded-Klipper

---

## 🔌 Flashing BigTreeTech Eddy USB Firmware

The Eddy USB probe must be flashed with firmware before use.

### 🛠️ Step-by-step Instructions:

#### 1. Enter Bootloader Mode
- **Hold the BOOT button** while plugging Eddy into your Pi/PC via USB
- A drive called `RPI-RP2` will appear

🖼️ **Where is the BOOT button?**  
![Eddy Boot Button](https://bttwiki.com/img/Eddy/Eddy_System5.png)  
**⚠️ Don’t disassemble your Eddy!**  
The button is fully accessible without disassembly. The exploded image is for visibility only.

#### 2. Confirm Device via `lsusb`

```bash
lsusb
```

You should see something like:
```
Bus 001 Device 007: ID 2e8a:0003 Raspberry Pi RP2 Boot
```

#### 3. Build Klipper Firmware

```bash
cd ~/klipper
make menuconfig
```

Select the following options:


![RP2040 USB Configuration Menu](https://bttwiki.com/img/rp2040_usb_menuconfig.png)

```
[*] Enable extra low-level configuration options

Micro-controller Architecture: Raspberry Pi RP2040/RP235x
Processor model: rp2040
Bootloader offset: No bootloader
Flash chip: GENERIC_03H with CLKDIV 4
USB communication: USBSERIAL
```

Then build:
```bash
make
```

#### 4. Flash the Eddy USB

Replace `2e8a:0003` with your actual device ID found in step 2.

```bash
make flash FLASH_DEVICE=2e8a:0003
```

#### 5. Get Serial Path

```bash
ls /dev/serial/by-id/
```

Example result:
```
/dev/serial/by-id/usb-Klipper_rp2040_Eddy_USB_Probe_if00
```

Use this in your config under `[mcu eddy]` and `[edddystream]`.

---

## 🧾 Example Configuration for Ender 6 + Eddy USB + BIQU H2 V2S

This fork includes a tested Klipper `printer.cfg` designed for:

- 🖨️ Ender 6 with stock board
- 📐 BigTreeTech Eddy USB probe
- 🔩 BIQU H2 V2S direct-drive extruder
- 🖥️ DGUS display using `[t5uid1]` (via Desuuuu firmware)

📄 [View the configuration file → `config/creality-ender6-biquh2v2s-eddyusb-dgus.cfg`](https://github.com/jakubkarafa/klipper/blob/dgus-reloaded/config/creality-ender6-biquh2v2s-eddyusb-dgus.cfg)

---

### 🧠 Config Highlights

- Pre-configured `[mcu eddy]`, `[probe_eddy_current]`, and `[edddystream]`
- Macros for auto-probing and safe homing
- Tuned PID values for BIQU H2 V2S
- Mesh boundaries optimized for Ender 6 + Eddy USB
- `[t5uid1]` section enabled for DGUS UI

---

### ⚙️ Calibration and Setup Notes

- You **must calibrate** the Eddy probe using the correct commands — not just `PROBE_CALIBRATE`
- Follow BTT’s official instructions here:  
  👉 https://bttwiki.com/Eddy.html#calibration

---

### 📚 Full Firmware and Setup Reference (Required Reading)

🛠️ Eddy USB requires proper firmware and configuration. Refer to these official guides from BTT:

- 🔧 [Compiling Firmware](https://bttwiki.com/Eddy.html#compiling-firmware)
- 📤 [Update Firmware](https://bttwiki.com/Eddy.html#update-firmware)
- ⚙️ [Klipper Configuration](https://bttwiki.com/Eddy.html#klipper-eddy-configuration)
- 📏 [Calibration](https://bttwiki.com/Eddy.html#calibration)

📁 All of these steps are critical to getting your Eddy USB working correctly.

---

## ❓ Questions or Feedback

> If you run into issues, have suggestions, or want to contribute improvements, feel free to [open an issue](https://github.com/jakubkarafa/klipper/issues) or submit a pull request.  
> Contributions that improve support for Ender 6, Eddy USB, or DGUS displays are especially welcome!

---

## 🙏 Credits

- [Klipper](https://github.com/Klipper3d/klipper)
- [Desuuuu/DGUS-reloaded-Klipper](https://github.com/Desuuuu/DGUS-reloaded-Klipper)
- [BTT Eddy Probe](https://github.com/bigtreetech/BIGTREETECH-Eddy)
- [BTT Eddy Wiki](https://bttwiki.com/Eddy.html)
