# SoftRF on the SenseCap T1000-E (Card Edition)

![SoftRF](https://img.shields.io/badge/SoftRF-MB208-blue)
![Status](https://img.shields.io/badge/Status-Development-orange)
![License](https://img.shields.io/badge/License-GPL--3.0-green)

**SoftRF** is a DIY, multifunctional, compatible, sub-1 GHz ISM band radio-based proximity awareness system for general aviation.

This repository documents the use of Moshe Braner's SoftRF MB208 firmware on the **Seeed SenseCAP T1000-E Card Edition**. It provides T1000-E specific installation instructions, bootloader migration procedures, firmware binaries, configuration guidance and operational notes, bringing together information from multiple SoftRF sources into a single device-focused guide.

## Overview

- **Dual protocol support**: FLARM-compatible FSK and FANET LoRa operation using time-sliced scheduling on the LR1110 radio
- **GPS PPS synchronization**: precise RF slot timing for dual-protocol operation; a valid GNSS fix and PPS signal are required
- **Fast protocol switching**: typically about 5–6 ms radio reconfiguration between FSK and LoRa modes
- **LR1110 optimized**: fast standby mode transitions and rapid FSK/LoRa reconfiguration
- **Traffic awareness**: reception and processing of compatible traffic broadcasts, including collision alerts
- **Multiple output formats**: NMEA, GDL90, D1090, JSON and MAVLink, depending on the firmware configuration and available T1000-E interfaces
- **SenseCAP T1000-E Card**: compact, sealed tracker with an LR1110 radio, GNSS, BLE and SPI flash storage for flight logs

<img width="391" height="317" alt="SenseCap_T1000E" src="https://github.com/user-attachments/assets/3e919131-ca08-4444-bbb6-4e658738db44" />

When SoftRF is configured for a FLARM-compatible protocol supported by the installed firmware, it can:

* transmit compatible traffic information to nearby FLARM and SoftRF devices
* make the aircraft visible to receiving OGN ground stations, subject to protocol coverage and tracking settings <https://glidertracker.org/>
* receive compatible traffic for output to connected navigation devices
* generate collision warnings based on received traffic and the selected alarm settings, stand alone or in conjunction with other devices

## The T1000-E at a glance

| | |
|---|---|
| CPU / Radio | nRF52840 with Semtech LR1110 |
| Edition | "Card Edition" (T-Beam = Prime Mark II, T-Echo = Badge, M1 = Handheld, M3 = Pocket) |
| Enclosure | sealed — internal battery not accessible, magnetic USB connector |
| Connectivity | **BLE only mode**, USB — no WiFi, no UDP/TCP, no secondary serial |
| NMEA output | USB (CDC) and/or Bluetooth LE |
| Display | none — configuration via Configurator, `settings.txt` or NMEA |
| Flight logging | yes (IGC or compressed IGZ files stored in flash) |

> ⚠️ **Important:** A new T1000-E ships with a bootloader that is **not compatible**
> with SoftRF. The bootloader and SoftDevice must be downgraded (to version 6.1.1)
> **before** SoftRF can be installed — see Step 2 below.

---

## Installation

### Step 1 — Download the required files

All files are in the [`Binaries`](https://github.com/gh4chris/SoftRF-T1000-E/tree/main/Binaries) folder:

| File | Purpose |
|------|---------|
| [`bootloader_flasher_v3.exe`](https://raw.githubusercontent.com/gh4chris/SoftRF-T1000-E/main/Binaries/bootloader_flasher_v3.exe) | Bootloader / SoftDevice downgrade tool (Windows) |
| [`SoftRF-firmware-Card_T1000E-1.7-a827d2-VB007-prd.uf2`](https://raw.githubusercontent.com/gh4chris/SoftRF-T1000-E/main/Binaries/SoftRF-firmware-Card_T1000E-1.7-a827d2-VB007-prd.uf2) | SoftRF bootloader for the T1000-E (UF2) |
| [`SoftRF.MB208.nRF52.uf2.zip`](https://raw.githubusercontent.com/gh4chris/SoftRF-T1000-E/main/Binaries/SoftRF.MB208.nRF52.uf2.zip) | SoftRF MB208 firmware, nRF52 (UF2, zipped) |

> ⚠️ The Bootloader Flasher currently runs on **Windows** PCs only.

### Step 2 — Flash the bootloader

1. Connect the T1000-E to your computer with a USB cable.
2. Double-click `bootloader_flasher_v3.exe`.

![Bootloader Flasher Utility](https://raw.githubusercontent.com/gh4chris/SoftRF-T1000-E/master/images/BootloaderFlasher_1.png)

3. Select your device's **COM port** from the dropdown menu.
4. Click **"Flash Device"** and wait for the flashing process to complete.

![Successful flash](https://raw.githubusercontent.com/gh4chris/SoftRF-T1000-E/master/images/BootloaderFlasher_2.png)

> ✅ **Success!** You should see a confirmation message when the bootloader is flashed successfully.

### Step 3 — Install the SoftRF firmware

After the bootloader flash completes successfully, a **File Explorer window pops up
automatically** showing the `T1000-E` drive.

1. Simply **drag and drop** the SoftRF firmware `.uf2` file into the `T1000-E` drive window.

![Firmware drag and drop](https://raw.githubusercontent.com/gh4chris/SoftRF-T1000-E/master/images/t1000e-drive.png)

2. The device will automatically install the firmware, reboot, and be ready to use with SoftRF.

> 💡 **Tip:** If the File Explorer window doesn't appear automatically, you can manually
> enter DFU mode:
> 1. Power off the device
> 2. Press & hold the button
> 3. Quickly connect the USB cable **TWICE** (double-tap connection)
> 4. The `T1000-E` drive should appear

### Updating the firmware

Once the bootloader has been downgraded, the T1000-E updates the same way as the
T-Echo, M1 and M3 (see the
[nRF52840 instructions](https://github.com/lyusupov/SoftRF/blob/master/software/firmware/binaries/README.md#nrf52840)):

1. Download the firmware archive, for example `SoftRF.MB208.nRF52.uf2.zip`, and extract the `.uf2` file.
2. Connect the device via USB — or enter DFU mode by double-tapping the USB cable while
   holding the button, or by sending the NMEA command `$PSRFC,DFU*2F`.
3. Drag & drop the `.uf2` file onto the `T1000-E` drive. The device installs the firmware
   and reboots automatically.

User settings are preserved when upgrading between compatible firmware versions.
(There is no WiFi on this device, so firmware updates via WiFi — a T-Beam feature —
are not available.)

### Compiling it yourself

The firmware source code is maintained in Moshe Braner's SoftRF repository; this repository primarily provides T1000-E-specific documentation and prebuilt binaries.

The nRF52 binaries (for T-Echo, M1, M3 and T1000-E) are built with the
**Adafruit nRF52 board support package version 1.2.0**. Details in the
[under-the-hood document](https://raw.githubusercontent.com/moshe-braner/SoftRF/refs/heads/master/software/firmware/documentation/SoftRF_MB_under_the_hood.txt).

---

## Everyday operation

### Power on / off

- **On:** click the button. **Off:** press and hold the button. The LED shows **red**
  while booting and shutting down, otherwise green or blue.
- **USB power:** when connected to (or disconnected from) USB power, the device starts up
  briefly but then goes back to sleep and just charges the battery. To boot normally while
  attached to USB power, either first let it fully boot on battery power and *then* connect
  USB, or enable the `power_ext` setting.
- **Beeps:** an early beep (and red LED) while booting; when booting is complete a single
  beep followed by **1–5 short beeps** reporting the battery state of charge (one low-pitched
  beep = empty, five short high-pitched = full). Later, **two shorter rising beeps** signal
  that a GNSS fix has been achieved.

### GNSS fix

SoftRF only transmits its position and interprets other traffic once the GNSS has a fix.
On the T1000-E the **green LED blinks rapidly** while waiting for a fix and **stays lit**
once it is attained (slow blinking = low battery). A cold start may take up to 30 minutes.
Until the leap-second information is received from the satellites (up to 12 more minutes),
the exact time may be off; SoftRF then assumes 18 leap seconds.

### Battery care

After several weeks in the apparently "off" state the battery may be depleted.
**Always recharge the battery before attempting to use the device for a long flight
following a storage period of more than a couple of weeks.**

---

## Configuration

The T1000-E has no Wi-Fi and does not host its own web interface. It can nevertheless be configured through a browser-based external Configurator using Web Bluetooth or Web Serial. Three ways to configure:

1. **Vlad's Configurator** (recommended): open <https://skysignals.app/mysoftrf/> in a
   Web-BLE capable browser (Chrome / Edge) on a phone or laptop, tap **BLE** (or **USB**)
   and select the SoftRF device in the list. Do **not** pair via the system Bluetooth menu.
   It can also be downloaded and used offline.
2. **`settings.txt`:** connect the device via USB, open it as a drive and edit the
   `settings.txt` file (one `label,value` per line — comments in the file explain the
   values). Save and reboot for the changes to take effect.
3. **NMEA commands** via a terminal program on USB or Bluetooth — see below.

> **Note:** For compatibility with the Configurator (and direct configuration via NMEA),
> do not include the characters `$`, `#` and `!` in any settings, such as `fanet_name`.

### Basic settings

| Setting | Recommendation |
|---|---|
| Mode | `Normal` |
| Device ID | fixed per device; current T1000-E generated IDs normally start with `8`; cannot be changed |
| Aircraft ID | your ICAO hex ID if the aircraft is registered; register the ID at <http://ddb.glidernet.org/> so OGN viewers show your registration / contest ID |
| ID type | `ICAO` or `Device` — the transmitted ID depends on **both** the ID and the ID type! |
| Protocol | `Latest` = compatible with the new (post-2024) FLARM protocol — recommended. OGNTP and ADS-L are visible to OGN stations but invisible to FLARMs. FANET and PAW are also available; multi-protocol modes are possible (see below) |
| Region (band) | defaults to *automatic* and is determined after the first GNSS fix; the detected region is only retained if the settings are saved. ⚠️ Verify the selected region, particularly in countries such as CN, RU or KR, and set it manually if required (EU = 868 MHz, US = 915 MHz) |
| Aircraft type | glider, powered plane, etc. — important for collision avoidance and OGN tracking |
| Alarm trigger | `Latest` (recommended) — predicts the near-future paths of circling aircraft; automatically falls back to `Vector` (straight lines) or `Distance` when appropriate |
| Volume | volume of the audible collision warnings |
| Bluetooth | BLE — the only option on the T1000-E, turned on by default |
| NMEA output | `USB` and/or `Bluetooth` (UDP/TCP are not available on this device) |
| NMEA sentences | GNSS, Sensors, Traffic; individual subtypes selectable in the advanced settings |
| Serial port baud rate | default 38400 |
| Stealth | on = you are not shown on other aircraft's displays (and vice versa, except at close range / collision danger) |
| No track | tells OGN ground stations not to report your position |
| Flight logging | Off / Always / Airborne / Traffic — see below |
| IGC encryption key | only relevant when using the OGNTP protocol |

### Advanced settings (`settings.txt` reference)

Full details in the
[under-the-hood document](https://raw.githubusercontent.com/moshe-braner/SoftRF/refs/heads/master/software/firmware/documentation/SoftRF_MB_under_the_hood.txt).

| Setting | Values / meaning |
|---|---|
| `id_method` | address type: 1 = ICAO ID, 2 = device ID, 5 = FANET-style ID (vendor 87), 7 = override, 0 = random, 3 = anonymous |
| `ignore_id` | aircraft ID to exclude from traffic data and warnings (`000000` = off) |
| `follow_id` | aircraft ID to prioritize in traffic reports (`000000` = off) |
| `band` | region; auto by default, overwritten when the location is known |
| `tx_power` | file-only setting: 2 = full power (default), 1 = low (2 mW), 0 = receive-only |
| `hrange` / `vrange` | reporting range in km / hundreds of meters |
| `relay` | 0 = off, 1 = relay landed-out, 2 = relay all traffic, 3 = relay-only |
| `pflaa_cs` | 1 (default) include callsign after hex ID in PFLAA sentences |
| `expire` | seconds to keep reporting traffic not heard from (1–30, default 5) |
| `altprotocol` | enables periodic transmissions using an additional protocol (1 = OGNTP, 6 = Legacy, 7 = Latest, 8 = ADS-L), depending on the selected operating mode |
| `flr_adsl` | 1 = simultaneous Latest+ADS-L reception and occasional ADS-L transmissions — recommended in the EU |
| `nmea_out` / `nmea_out2` | 0 = off, 1 = serial, 2 = UDP, 3 = TCP, 4 = USB, 5 = Bluetooth, 6 = secondary serial — on the T1000-E only **4 (USB)** and **5 (Bluetooth)** are available |
| `nmea_g` / `nmea_s` / `nmea_t` / `nmea_e` (and `nmea2_*`) | hex bitfields for sentence subtypes: 1 = basic (GGA+RMC / PGRMZ / PFLAA), 2 = GSA / LK8EX1 / PFLAJ, 4 = GST / PFLAM, 8 = GSV / FNNGB, F = all |
| `debug_flags` | 32-bit bitfield: 01 WIND, 02 PROJECTION, 04 ALARM, 08 LEGACY (raw packet dump), 10 DEEPER (RF slot timing, LR1110 detail), 20 DEEPER2 |
| `baud_rate` | 1=4800, 2=9600, 3=19200, 4=38400, 5=57600, 6=115200; 0/default = 38400 |
| `gdl90` / `d1090` | alternative output formats (GDL90 / Dump1090), usually off |
| `logflight` | IGC flight log: 0 = off, 1 = always, 2 = when airborne, 3 = + near traffic, 4 = + far/non-airborne traffic |
| `loginterval` | seconds between flight log position records (1–255) |
| `compflash` | T1000-E: 0 = uncompressed IGC files, 1 = compressed IGZ files |
| `alarmlog` | 1 = log alarms, takeoffs and landings to `alarmlog.txt` in flash |
| `gn_to_gp` | 1 = convert `$GN/$GA/$GL` sentences to `$GP` in the NMEA output |
| `power_save` | 1 = turn Bluetooth off after 10 minutes if not connected |
| `power_ext` | automatically shut down when external power is disconnected (after ≥1 h runtime, not airborne, battery < 3.9 V); also prevents "charging mode" instead of a normal boot when USB and battery power are both present |
| `fanet_name` | FANET pilot name (no `$ # !`) |
| `auto_sos` | FANET SOS: 0 = off, 1 = manual, 2 = auto |

### NMEA configuration (handy without a web UI)

```
$PSRFS,?*57                 list all settings and their current values
$PSRFS,0,<label>,?          query one setting
$PSRFS,0,<label>,<value>    change one setting (then $PSRFC,SAV*3C to save & reboot)
$PSRFS,1,<label>,<value>    change one setting + save + reboot immediately
$PSRFC,SAV*3C               save settings and reboot
$PSRFC,RST*2D               reset
$PSRFC,OFF*37               shut down
$PSRFC,EEP*28               back up the settings to EEPROM
$PSRFC,DFU*2F               reboot in DFU mode for UF2 firmware update
$PSRFT,1*5E / $PSRFT,0*5F   test mode on / off
$PSRFT,!*4E                 toggle test mode
$PSRFT,?*50                 query test mode, debug flags, and chip ID
```

Each sentence must end with `*` plus a valid NMEA checksum — you can send `*xx` and
SoftRF will reply showing the correct checksum. If reading/writing `settings.txt` fails,
SoftRF automatically falls back to EEPROM. If the flash file system is unreliable, EEPROM
storage can be forced with `$PSRFS,1,mode,16*5D` (undo: `$PSRFS,1,mode,0*6A`).

A convenient browser-based alternative is
[Vlad's Configurator](https://skysignals.app/mysoftrf/), which works through the same
`$PSRF*` mechanism via BLE or USB.

### Multi-protocol operation

Single protocols:

* **ADS-L** (MDR, on the same frequencies as FLARM)
* **Legacy** — compatible with the old (pre-2024) FLARM protocol, not recommended
* **Latest** — compatible with the new (post-2024) FLARM protocol, recommended
* **OGNTP**
* **FANET**
* **PAW** (Pilot Aware, P3I)

Time-slicing dual-protocol modes (examples):

* **Latest + FANET** — *the default dual-protocol configuration of the MB208 build provided here:*
  slot 0: TX/RX in Latest, slot 1: TX/RX in FANET (RX in Latest every 4 seconds)
* **Latest + PAW**, **FANET + OGNTP**, **FANET + ADS-L**, **PAW + ADS-L**, **PAW + OGNTP** —
  one protocol per time slot
* **FANET + PAW** — alternating seconds: RX in PAW 25% / FANET 75% of the time

With `flr_adsl=1`, simultaneous Latest+ADS-L dual-mode reception is added, and in the
FANET+PAW modes SoftRF can even transmit and receive in **four** protocols at once
(set `expire` to 10+ seconds in that case). The configured aircraft identity is used to derive the corresponding identifier for each active protocol. The complete table of all combinations is in the
[under-the-hood document](https://raw.githubusercontent.com/moshe-braner/SoftRF/refs/heads/master/software/firmware/documentation/SoftRF_MB_under_the_hood.txt).

### FANET messaging

MB208 supports FANET messaging: pilot name (`fanet_name`), callsign, and ad-hoc
broadcast and unicast text messages, exchanged with connected apps over BLE or USB.
The `auto_sos` setting controls the SOS function (0 = off, 1 = manual, 2 = auto).
Use a messaging-capable app (or Vlad's Configurator) to send and receive messages.

### Flight and alarm logging

Flight logging is supported on the T1000-E (it has a file system in SPI flash).
`logflight` selects whether to log (airborne = automatic start on takeoff, stop after
landing), `loginterval` the seconds between fixes. Log files are named
`MMDDhhmm.igc` (date/time of takeoff). With `compflash=1` compressed **IGZ** files are
written instead of IGC — decompress them with the `igcdecoder.py` script from
[Moshe's repository](https://github.com/moshe-braner/SoftRF).
With `alarmlog=1` every traffic alarm, takeoff and landing (with maximum altitude) is
appended to `alarmlog.txt`. Log files and `settings.txt` can be read and edited by
connecting the device to a PC via USB and opening it as a drive.

### Air relay

Optional relaying by airborne aircraft of radio packets from other aircraft; relayed
packets are not relayed a second time. `relay=1` relays packets from gliders that
"landed out" (aircraft type set to zero), `relay=2` also relays FLARM traffic relayed in
ADS-L (and ADS-B traffic, if an ADS-B receiver is attached), `relay=3` is "relay only"
(8 mW, no own position transmissions). Relaying happens at most once every 5 seconds in
total, and only FLARM traffic farther than 10 km away (or closer if lower) is relayed.

### Test mode

In test mode, air-relay happens even when not airborne, and ADS-L transmissions on the
ground occur once per second as if airborne. Toggle with `$PSRFT,1*5E` / `$PSRFT,0*5F`
/ `$PSRFT,!*4E`, or query with `$PSRFT,?*50`.

### Monitoring the RSSI of received signals

SoftRF stores the current RSSI of each tracked aircraft; the current highest RSSI of all
tracked aircraft is sent at the end of the `$PSRFH` "heartbeat" NMEA sentence every
10 seconds via USB or Bluetooth. Checking it via a Bluetooth terminal has the advantage
that one can be some distance away from the device.

### Data bridging

On nRF52 devices like the T1000-E, USB-serial data can be forwarded to Bluetooth and
vice versa, eliminating the need for an IOIO box or Bluetooth dongle. GNSS and FLARM
messages are not forwarded; other complete NMEA sentences are.

### Troubleshooting

* **No fix / slow fix:** the stock GNSS antenna needs a clear outdoor view. A cold start
  can take up to 30 minutes.
* **Device does not boot properly:** send `$PSRFC,RST*2D`, or double-tap the USB cable
  to re-enter DFU mode and re-flash the firmware.
* **Settings lost after upgrade:** upgrading across incompatible versions resets settings
  to defaults — check them after a major upgrade.
* **BLE connection problems:** do not pair via the system Bluetooth menu; connect from
  within the app / Configurator instead. Reboot restores Bluetooth after `power_save`.

---

## Resources

* [SoftRF MB user guide](https://raw.githubusercontent.com/moshe-braner/SoftRF/refs/heads/master/software/firmware/documentation/SoftRF_MB_user_guide.txt) (Moshe Braner)
* [SoftRF MB "under the hood"](https://raw.githubusercontent.com/moshe-braner/SoftRF/refs/heads/master/software/firmware/documentation/SoftRF_MB_under_the_hood.txt) (advanced settings, protocol table)
* [SoftRF-PG — original T1000-E setup pages](https://slash-bit.github.io/SoftRF-PG/) (Vlad Belayev)
* [SenseCap T1000E Card Operating Instructions](https://github.com/slash-bit/SoftRF-PG/blob/master/documents/Documentation/SenseCap%20T1000E%20Card%20Operating%20Instructions.md) — device features, beep codes, usage tips
* [Card Edition Quick Start](https://github.com/lyusupov/SoftRF/wiki/Card-Edition.-Quick-start) (Linar Yusupov, manual method)
* [nRF52840 flashing instructions](https://github.com/lyusupov/SoftRF/blob/master/software/firmware/binaries/README.md#nrf52840)
* [Vlad's Configurator](https://skysignals.app/mysoftrf/)
* [Moshe Braner's SoftRF fork](https://github.com/moshe-braner/SoftRF) · [SoftRF mainline](https://github.com/lyusupov/SoftRF)
* [OGN device registration](http://ddb.glidernet.org/) · [glidertracker.org](https://glidertracker.org/)
* [SoftRF Community](https://groups.google.com/g/softrf_community)

## Credits

* Linar Yusupov — original SoftRF
* Moshe Braner — SoftRF fork, user guide & "under the hood" documentation
* Vlad Belayev — bootloader downgrade tool and Configurator
* Seeed / SenseCap — T1000-E hardware

License: GPL-3.0
