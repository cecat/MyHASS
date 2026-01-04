---
layout: default
title: "Setting up Permanend Outdoor LED Strings in Home Assistant using WLED"
date: 2026-01-03
categories: integrations lighting
---
## Setting up LED Strips in Home Assistant using WLED

*January 3, 2026*

Rather than spend hundreds of dollars on smart outdoor permanent holiday lights,
I decided to try to use the same strategy that I used for 
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

I am by no means an LED or lighting expert - I am just reporting my experience and would welcome
advice.

---

# The DIY Solution

I started with a D1 mini and learned two things.  First, there are multiple D1 mini configs, and I
accidentally ordered some with 1MB rather than 4MB memory.  WLED will install just fine on the 1MB
version but it won't run.  Second, the D1 mini is a bit under-powered for anything but a small 
string of LEDs, and an ESP32 is much more beefy, while only costing $2 more (so why compromise?).

I first ordered some outdoor rated LED strings, specifically some
<a href="https://www.hollywoodleds.com/products/novabright-ws2811-ic-12v-digital-addressable-rgb-full-color-christmas-bullet-pixel-led-string-lights-18awg-4-inch-spacing-ip68-outdoor-light-50px-set" target="_blank" rel="noopener noreferrer">NovaBright WS2811</a>
strings from
<a href="https://www.hollywoodleds.com" target="_blank" rel="noopener noreferrer">Hollygood LEDs</a>
(I am sure many places sell them, but these were on sale and seem very solid/durable).

To drive these I need 12v power and the data line is 5v.  This means I need to boost the data signal
coming out of the ESP32 from 3.3v to 5v, so it's a bit more complex than the simple D1 mini driving a tiny
LED string.


## Parts

Besides the LED string(s) linked above, I used the following:

* <a href="https://www.amazon.com/dp/B08Q2TWYRJ"
   target="_blank" rel="noopener noreferrer">10A Power Supply</a>.
  (for a single string you can probably lie with a
   <a href="https://www.amazon.com/dp/B01GEA8PQA"
   target="_blank" rel="noopener noreferrer">5A Power Supply</a>.
* <a href="https://www.amazon.com/dp/B0F5QCK6X5"
   target="_blank" rel="noopener noreferrer">ESP32 controller</a>.
* <a href="https://www.amazon.com/dp/B0D9M1KRYL"
   target="_blank" rel="noopener noreferrer">12v-to-5v Buck Converter</a>.
   This is a bit bulky but it's weatherproof and more robust - can handle up to 3A whereas
   lower power / smaller form vactor alternatives such as
      <a href="https://www.amazon.com/dp/B0B779ZYN1"
      target="_blank" rel="noopener noreferrer">this</a> are only rated for 1.8A (still not bad!).
* <a href="https://www.amazon.com/dp/B08R6BCSYC"
   target="_blank" rel="noopener noreferrer">Logic level shifter (to boost 3.3v to 5v).</a>.
* <a href="https://www.amazon.com/dp/B09CDQWTFG"
   target="_blank" rel="noopener noreferrer">3-pin waterproof LED (pixel) connectors.</a>.
* <a href="https://www.amazon.com/dp/B0CFJV2M3B"
   target="_blank" rel="noopener noreferrer">18 Gauge 3-conductor cable.</a>.
* <a href="https://www.amazon.com/dp/B08R6BCSYC"
   target="_blank" rel="noopener noreferrer">Logic level shifter (to boost 3.3v to 5v)</a>.

And dipped into my inventory for:
* A 330 ohm resistor for each string directly connected to the board (I designed for two).
* Screw terminal block connectors (three 2-terminal and two 3-terminal).
* A 25v 1k uF electrolytic capacitor
* A proto PCB board with power rails. Something like
  <a href="https://www.amazon.com/ElectroCookie-Solderable-Breadboard-Electronics-Gold-Plated/dp/B07ZYNWJ1S"
   target="_blank" rel="noopener noreferrer">this</a>.
  should work - I could not find where I bought mine, but you don't want to mess with a
  straight-up stripboard (having to cut channels to place ICs).

## Assembly


<img src="/media/outdoor/breadboard.png" width="200px"/>
<img src="/media/outdoor/schematic.png" width="200px"/>
<img src="/media/outdoor/IRL.png" width="200px"/>

*Above: How to wire this up.  (L to R: Physical view; schematic; IRL).
The IRL was my third layout, and I had run out of 2-port screw terminal
blocks so used a 3-port for input (main) power at top left.  You can also
see the board includes a header for a D1 mini, which is not used in favor of
the ESP32.  Only one header was needed since we only used pins on one side
(same with the ESP32).
(<a href="/projects/AddressibleLED-Controller.fzz" target="_blank" rel="noopener noreferrer">Download the Fritzing file</a>)

# Commercial Controllers

My exploratory DIY approach above works for my particular setup but lacks the robustness of fuses
or other protections for making mistakes in wiring, overloading the power, etc.
I tried one off-the-shelf WLED controller  with capacity for a single string, and then ordered
a second one that has capacity for up to four strings (much more heavy duty).  I am ver happy 
with the first one and the second one arrives next week, so this post will be updated soon.

## IoTorero Addressable Strip Controller (w/ build-in microphone)  (WIP)

I ordered the
<a href="https://www.athom.tech/blank-1/ethernet-wled-esp32-addressable-dmx-led-strip-controller" target="_blank" rel="noopener noreferrer">IoTorero Controller</a>
from 
<a href="https://www.athom.tech" target="_blank" rel="noopener noreferrer">Athom Tech</a>x
I have ordered from them in the past as they have lots of nice HASS-friendly devices including Tasmota and 
WLED. They are in China but shipping is cheap and pretty quick (about a week).

The unit is pretty much turnkey and a nice solution for a single LED strip. It is also designed to control
many types of LED strip, including dumb ones. 

**Todo: daisy-chain experiment to see if it is powerful enough to drive two strings.**

<img src="/media/outdoor/Athom-IOTorero.jpeg" width="200px"/>
<img src="/media/outdoor/Athom-wiring.jpeg" width="200px"/>

*Above: Athom WLED server and wiring for WS2811 LED strings.*

## Dig-Quad  (WIP)

I'm eager for this one to arrive as it looks like a very sturdy design and is ideal for my intended
use with 3-4 50-LED strings.  I ordered a 
(<a href="https://quinled.info/quinled-dig-quad/" target="_blank" rel="noopener noreferrer">QuinLED Dig-Quad</a>)
from 
(<a href="https://www.drzzs.com/shop/" target="_blank" rel="noopener noreferrer">Dr. Zzs</a>), 
which is an interesting online shop with gadgets related to Home Assistant, sensing, and LED
power and control.

This controller can drive four strings, and is about twice the cost of the IoTorero so for
my four strings it's 50% lower cost than four single-string controllers. It is, however, a single
point of failure FWIW.

The unit will arrive in a few days, at which time I will test and update this post. 

<img src="/media/outdoor/QuinLED-Dig-Quad.png" width="200px"/>

*Above: QuinLED Dig-Quad.*

# Install WLED on ESP32 / ESP32-S3 (Web Installer) + Wi-Fi Setup [WIP needs updating]

Once you have your WLED controller set up you will likely want to integrate it with Home Assistant.
This guide uses **ESP32-S3-WROOM-1-N16R8** as an example, but the same flow should
work for most **ESP32-family** boards supported by WLED.

If you are integrating one of the commercial units you don't need to install WLED, so can skip to 
Step 3.


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
