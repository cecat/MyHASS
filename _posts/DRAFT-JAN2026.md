---
layout: default
title: "Setting up Permanend Outdoor LED Strings in Home Assistant using WLED"
date: 2026-01-03
categories: integrations lighting
---
## Setting up LED Strips in Home Assistant using WLED

*January 3, 2026*

Rather than spend hundreds of dollars on smart outdoor permanent holiday lights,
I decided to use the same strategy that I have used for 
<a href="https://blog.deepblueberry.com/integrations/lighting/2025/01/31/wled/" target="_blank" rel="noopener noreferrer">indoor smart LED strings</a> 
leveraging the
<a href="https://kno.wled.ge/" target="_blank" rel="noopener noreferrer">WLED Project</a>. 

The WLED project has very nice
<a href="https://kno.wled.ge/basics/getting-started/" target="_blank" rel="noopener noreferrer">getting started</a>
instructions, including wiring examples, and tips to avoid
<a href="https://kno.wled.ge/basics/top5_mistakes/" target="_blank" rel="noopener noreferrer">common mistakes</a>.

Going from small indoor strings to larger, more power-hungry, outdoor LED strings
introduced a bit more complexity regarding power, and this post documents two paths
I took.  The first was a DIY approach, trying to build on the simpler indoor / small
string solution.  Besides extra complexity for power, I ended up using switching from
the D1 mini to an  ESP32 (finding that the D1 mini is really
under-powered for this job).  The second approach was to evaluate several packaged systems,
one of which I found to be ideal for my use case (3-4 strings of 50 LEDs).  This was not
as fun as DIY but much more practical and of course a system with much better engineering
than my DIY system.


---

# The DIY Solution

## Parts

* <a href="https://www.amazon.com/dp/B081PX9YFV?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1" target="_blank" rel="noopener noreferrer">D1 Mini</a>
($3) NOTE: Make sure you are ordering units with 4MB memory. There are 1MB versions and these cannot run WLED.
* <a href="https://www.amazon.com/dp/B09MKSVV5H" target="_blank" rel="noopener noreferrer">WS2812B/SMD5050 individually addressable LED Strip</a>
($11)
* <a href="https://www.amazon.com/dp/B08KT6BG5F" target="_blank" rel="noopener noreferrer">Self-adhesive LED strip mounting clips</a>
($7 for 80)
* A 330 ohm resistor. You could buy 100 for $7-8 or you may as well get virtually a life-time supply of different sizes such as in a
<a href="https://www.amazon.com/BOJACK-Values-Resistor-Resistors-Assortment/dp/B08FHPJ5G8/ref=sr_1_7_sspa?th=1" target="_blank" rel="noopener noreferrer">kit like this</a> .
* <a href="https://www.amazon.com/dp/B082QZGY9V?th=1" target="_blank" rel="noopener noreferrer">Low-profile USB power adapter</a>
* <a href="https://www.amazon.com/dp/B0982S1FHY" target="_blank" rel="noopener noreferrer">1' USB-A to micro-USB cables</a>


## Assembly

(see <a href="https://kno.wled.ge/basics/top5_mistakes/" target="_blank" rel="noopener noreferrer">WLED common mistakes</a>).


<img src="/media/wled/breadboard.png" width="200px"/>
<img src="/media/wled/schematic.png" width="200px"/>
<img src="/media/wled/pcb.png" width="200px"/>

*Above: How to wire this up.  (L to R: Physical view; schematic; pcb layout).
For completeness I show a barrel connector for power in case you wanted to go
with an external, beefy power supply.  One could readily hard-wire the power
supply but a connector offers better flexibility (e.g., you may wish to swap
out the D1 Mini or the power supply later).*
(<a href="/projects/ESP-to-LED-Strip.fzz" target="_blank" rel="noopener noreferrer">Download the Fritzing file</a>)


# Install WLED on ESP32 / ESP32-S3 (Web Installer) + Wi-Fi Setup

This guide uses **ESP32-S3-WROOM-1-N16R8** as an example, but the same flow should
work for most **ESP32-family** boards supported by WLED.

---

## What you need

- A **data-capable** USB cable
- **Chrome** or **Microsoft Edge** (Web Serial is required for the installer)
- Your Wi-Fi SSID + password

---

## 1) Put the board into flashing mode (important for some ESP32-S3 boards)

Many ESP32 boards auto-enter download/flash mode, but some **ESP32-S3 dev boards** require a manual step.

### ESP32-S3 special steps (example: ESP32-S3-WROOM-1-N16R8)

1. **Unplug** the board from USB.
2. **Press and hold the second button** on the board (often labeled **BOOT**; **not** the reset/EN button).
3. While holding that button, **plug in USB**.
4. **Release** the button after it’s plugged in.

The board will have a red LED htat lights solid.  If the green LED (or other) is
blinking this means it's running and not in boot loader mode (try the above again,
holding BOOT for a few seconds after plugging in the board).

---

## 2) Install WLED using the web installer (recommended)

1. Open the web installer:  
   **https://wled-install.github.io/**

2. Click **Install** (or similar) and select the USB serial device when prompted.
   (look for a device named something like /dev/cu.usbmodem1234 (or some other number)).

3. Choose the correct target:
   - Pick **ESP32-S3** if you have an ESP32-S3 board (like the N16R8 example).
   - For other boards, pick the matching ESP32 variant shown in the installer list.

4. If offered, choose **Erase / Clean install** (recommended for fresh boards or if you’ve tried flashing before).

5. Start the install and wait for it to finish.

### ESP32-S3 note after installation
- **Manually press Reset/EN** (or unplug/replug USB) after the installer completes.
- Some ESP32-S3 boards need that manual reset to start the new firmware.

---

## 3) Confirm WLED Access Point (AP) appears

After the reset, WLED should create its own AP (usually `WLED-AP`) when it is not yet configured for your Wi-Fi.

**Important timing notes (observed on ESP32-S3 boards):**
- It can take **1–2 minutes** before `WLED-AP` shows up in a Wi-Fi scan.
- After joining the AP, the browser UI at **http://4.3.2.1** can take **15–20 seconds** to respond.

### Join the AP
1. On your phone/laptop, connect to the Wi-Fi network:
   - **SSID:** `WLED-AP`
   - **Password:** `wled1234` (if prompted)

2. In a browser, open (note **http**, not https):
   - **http://4.3.2.1**

> If you try `https://4.3.2.1`, it will fail. Use **http**.


---

## 4) Wiring & LED Configuration

In WLED UI:

- Go to **Config → LED Preferences**
- Set:
  - **LED Type**: WS2811
  - **LED Count**: (e.g., 50, 144)
  - **GPIO**: Use the actual data pin (e.g., **GPIO5** if you wired it that way)
- Save and reboot

---

## 5) Integrate with Home Assistant

1. Open Home Assistant
2. Go to **Settings → Devices & Services**
3. Click **+ Add Integration**
4. Search for **WLED**
5. Enter the IP address of the WLED device (find via router or mDNS)
6. Device will be discovered and added automatically

You can now control segments, colors, effects, brightness, and more directly from Home Assistant.

---


## 🧪 Optional: Flash via `esptool.py` CLI (macOS/Linux/Windows)

```bash
# Erase flash (optional but recommended)
esptool.py --chip esp32s3 --port /dev/cu.usbmodem1101 erase_flash

# Write WLED firmware
esptool.py --chip esp32s3 --port /dev/cu.usbmodem1101 --baud 460800   write_flash -z 0x0 WLED_0.15.3_ESP32-S3_8MB_opi.bin
```

Then press **RST** and wait ~10s for `WLED-AP` to appear.

---

## This DIY system has Verified with:

- ESP32-S3-WROOM-1-N16R8 Dev Board
- macOS with `esptool.py v4.8.1`
- WLED 0.15.3 (December 2025)
- NovaBright P12 WS2811 IC 12V Digital ADDRESSABLE RGB Full Color Christmas Bullet Pixel LED String Lights
  (18AWG 4 Inch Spacing IP68 Outdoor Light 50PX Set)


---

# Commercial Off The Shelf (COTS) Solution

I did a bunch of Internet searching, getting advice from AI chatbots, and product reviews to order 
a packaged system that comes with WLED pre-flashed and would, I thought, allow me to plug a power
supply in one and and one of my 50-LED strings at the other end - no DIY or fiddling needed.  It turns
out that this device probably works fine for smaller strings, but could not light my 50-LED string.
The good news was that my 50-LED string did not burn it up, as it has nice internal protections (it would
briefly light the string, then the internal proections would audibly click to turn them off).  The bad 
news was that I would need to separately power the LED strings.  This would still be easier than DIY,
but I had hoped for a more turnkey solution.

**Speculating below as the unit has not yet arrived... to be updated!**

This led me to the second packaged unit I tried, which I concluded was the way to go (modulo not as
fun as DIY) because of its capacity (4 strings) and excellent engineering with fuzes on each of the
four channels and of with the simplicity of power in on one end and LED strings out on the other.

With either of these devices, integration with HASS is the same as any WLED device, so you can look 
at the instructions above or better yet at the HASS WLED Integration pages.

https://www.home-assistant.io/integrations/wled/
