# A Primer on Home Assistant (HASS)
Charlie Catlett (August 2022)

*[Note- I am referencing specific products but I have no relationship with any
of the companies, and will not make any money if you buy them.  All it means is
that they work for me, but there are likely other options in many cases as
well.]*

---

## If you are anxious to get started...

You could skip the stuff below and head straight to the Home Assistant
[Getting Started pages](https://www.home-assistant.io/getting-started/),
which are fabulous.  You may also want to check out some of the getting started
videos (selecting hardware, mistakes to avoid, etc.) at my favorite source of
help for Home Assistant:
[Everything Smart Home](https://www.youtube.com/c/EverythingSmartHome).

---

## Why am I using HASS?

There are lots of smart/automated home products, many of which interoperate, all
of which have their own apps. Many can be aggregated through Amazon, Apple, or
Google home devices, and pretty much *all* of them use cloud services, making
your home rely on working Internet and spreading your data around the world.
You can also do pretty powerful things with services like IFTTT, but these are
very simple things (one trigger, one action) so you can't do less simple (but
still not complicated) things like "turn on the light for 10 minutes" (turn on
light; wait 10 min; turn off light - 3 actions, 2 too many for IFTTT) much less
more sophisticated things such as actions that have conditions (only do X from
12am to 5am). (I love IFTTT and used it a lot, but with HASS I don't need it)

<img src="/media/Legacy1999.jpg" align="left" width="200px"/>
I started with DIY systems I built myself in about 1999 to avoid pipes
freezing in a house far from my home in case the power (thus heat) or
just the heat were to fail in winter. My first system used an old fashioned
thermostat wired to a Radio Shack auto-dialer (and a couple of (big) 6v
batteries in case of power outages), so if the power and/or heat went out in
winter (dropping the temperature, triggering the thermostat), I'd get a call
(this saved my pipes multiple times).  Multiple generations of DIY later I found
myself accumulating many very nice (but independent) home automation systems.
Philips Hue lights (and a hub), Samsung Smartthings (and a hub), Amazon Blink
cameras (and a hub), Aqara door sensors (and a hub), various smart outlets (no
hub, or rather the hub is in China...), etc.  All of those hubs rely on Internet
connectivity to commercial clouds.  I adopted HASS to (a) bring all of these
systems together, and (b) move as much as possible away from reliance on
Internet/cloud for my automations and data. 
*(Photo shows the original install, not sure what happened to that thermostat.
The install date on the schematic is Jan 7, 1999.)*

<br clear="left"/>

## What am I doing with HASS?

I have a few important functions that I implement with HASS, which I would say
fall into several categories.

1. **Avoiding Unpleasant Events** (or at least getting an alert when they
happen, or might happen).  I have moisture sensors located near the sump
pump well, next to the water heater, and under the kitchen sink.  I am
immediately notified (lights turning on, text message) if moisture is detected.
I built a
[sensor package](https://github.com/cecat/UtilityWatchMQTT)
that keeps track of how often the sump pump runs (how worried
I should be if it were to stop), how often the water heater runs, and when the
HVAC system is running.  This gives me a feel for normal behavior, and on
several occasions that has helped me anticipate problems and prevent them.
Similarly, for a house that is 2h away I built a
[sensor package](https://github.com/cecat/Lake-Watch)
that every 5 minutes
sends the temperature of the crawlspace and whether the power is on.
(it has a battery and cellular connectivity so can operate for a dozen
hours if the power goes out). This gives me ample warning about power or
heat outages before pipes start to freeze.
Both of these devices talk to Home Assistant via MQTT.
I also have sensors on the garage doors, so I get notified if a garage door
is open for >10 min, because it's heated and I don't want to heat the front
yard in winter. (these talk to Home Assistant via Zigbee)

2. **Safety**.  In some spaces, such as the garage, or back deck, I have lights
automatically turn on when doors open or movement is detected (no fumbling with
wall switches).  I use sensors to detect when any of our exterior
doors open, so I can get notified if that happens when no one is home or in the
dark of night. Or I can set actions such as turning on lights when those
doors open.  I use cameras just to be able to check on things whenever of from
wherever I am.  One is pointed at the sump pump so if it is running a lot (see 1)
I can check in on it (from anywhere).  I use several types of cameras along
with a package ("add on") called [Frigate](https://docs.frigate.video/) to
detect when a human is seen in the front or back of the house
Frigate saves a 30s clip and I can have the system do nothing or trigger a
sequence of actions.  (depending on the time of day or whether anyone is at home) 

3. **Remote Control**.  From anywhere on the Internet I can check on things,
turn lights on or off, make sure doors are closed, open or close the garage
doors, etc. I can also control the lawn irrigation system.  All of these things
are possible with individual apps from the companies that make those systems,
but in HASS I have my own dashboard for the home.

4. **Fun Stuff**.  I use Life360 with a family circle that HASS uses to keep
track of how many of us are home (allowing me to have certain automated actions
related to security when no one is home).  I have another circle of a half dozen
friends and keep track of how many of us are at our lab at any given time.
At 4:30pm on weekdays HASS checks to see if 4 or more of us are there, and
if so we all get a text (and slack) message suggesting we go to the pub.

## What are a couple of things to get rolling?

My suggestion is to start with the systems you have (if any), and integrate them
with HASS.  Doing so in the simplest fashion (without ditching their hubs) won't
affect other things you already might do such as through your
Amazon/Apple/Google assistants, IFTTT, or the associated smartphone apps.  But
to get the best value from HASS you'll want to get rid of the hubs (where
possible) and have them
talk directly to HASS (depending on the system, this may be a simple reset or
may require new firmware, see my systems description below). You can check out
[what "integrations" are available](https://www.home-assistant.io/integrations/)
and it's likely that your systems are supported with integrations.

If you don't have any systems, you might start with something like moisture
detection, smart outlets for things like holiday lights or outdoor lights
(e.g., turn them on at dusk and off at midnight).  If you are adventurous, you
might skip right to Frigate and have the lights turn on when a human shows up
after dark (yes, you can do this with many products via motion detection, but
they can't tell the difference between a human and a strong wind hitting tree
branches).  From there you could try some of the systems I use or other things
you find in the plentiful online documentation, community forums, and YouTube
channels from the HASS community.

If you do go with Frigate you will want a Coral TPU but a Raspberry Pi4 (see
Hardware discussion below) can keep up with a single camera, albeit Frigate will
use 70-80% of the CPU. Don't even think about doing Frigate with a Pi3.
(I tried) Frigate will take a Pi3 to its knees .  I really like the Amcrest wifi
cameras (see my inventory of devices below) - inexpensive and easy to config in
Frigate.  

## Begin

### Hardware.

1. **Your server**.  You will want at minimum a Raspberry Pi4, but you might
also consider more powerful platforms such as a Nuc. The HASS installation pages
show other platforms that are supported.

2. **Storage**.  If you have a very simple system in mind you can get by with a
small (e.g, 32GB microSSD) on a Pi4 as suggested on the installation pages
linked below.  But chances are you will want to do more ambitious things like
[Frigate](https://docs.frigate.video/) or better database options, in which case
you should buy an SSD (go for 1 TB at least).

3. **Other**.  If you plan to use Frigate, you will need to either go with one
of the beefier servers or get a Coral TPU (USB device that plugs into your
server).  A Pi3 is probably fine if all you are doing is controlling lights and
monitoring sensors, but Frigate will bring it to its knees (it will be
unresponsive).  A Pi4 can barely keep up with 1 camera, using 70-80% of its CPU to
do so and giving you sluggish performance interacting with the system.

*NOTE: Unfortunately as of August 2022 it is nearly impossible to find either a
Pi4 or a Coral TPU (except from opportunitists at insane prices). 

### Software and Installation

Here you should dive into the fabulous online documentation that will walk you
through.  I would advise that you go with **Home Assistant Operating System**
(the first option on the
["Installation" page](https://www.home-assistant.io/installation/).
The other options are more advanced, but also more complicated.  For instance,
you can install Home Assistant Container as a docker image if you have a beefy
server (e.g., you can run Docker and there is a version of Home Assistant
Container that runs on Synology NAS systems like mine). But an important
component of Home Assistant called Supervisor must then be installed separately
in order to get functions that you will need.  So unless you are good with Linux
and Docker, and have something specific in mind you want to do that requires
this approach, go for Home Assistant OS.  (you can migrate easily later, as HASS
has nice backup/restore functions)

### Follow the **[Getting Started guide](https://www.home-assistant.io/getting-started/)**!


## More Help and Ideas

Googling helps, and the [Home Assistant site](https://www.home-assistant.io/)
has extensive documentation on all aspects, integrations, add-ons, etc. of Home
Assistant.  There is also an active
[Home Assistant Community]
(https://www.home-assistant.io/getting-started/join-the-community/)
you can join, which has lots of resources and tutorials.  I myself have found
[Everything Smart Home](https://www.youtube.com/c/EverythingSmartHome) to be
excellent for both beginning and advanced things.  For instance, tutorials on
[installing your first system](https://www.youtube.com/watch?v=SHg6fa0x7OA),
one on [mistakes to avoid](https://www.youtube.com/watch?v=i1083cCR2CI), and
another on things to do once you are up and running. You might start with the
*Home Assistant Guide*
[playlist](https://www.youtube.com/c/EverythingSmartHome/playlists)
there.

## What exactly do I have (in case you are curious)

1. **Hardware**:  Raspberry Pi4 with a 1TB SSD. I also have a
[Coral TPU](https://coral.ai/products/accelerator), a
[Zigbee USB stick](https://www.amazon.com/dresden-elektronik-ConBee-Universal-Gateway/dp/B07PZ7ZHG5)
(combined gateway and radio, controlled via a HASS add-on and integration), and
a [Z-Wave USB stick](https://www.amazon.com/Z-Wave-Stick-Assistant-HomeSeer-Software/dp/B07GNZ56BK/ref=sr_1_4?crid=2DD46HYRSA669&keywords=z-wave&qid=1660804098&s=electronics&sprefix=z-wave%2Celectronics%2C55&sr=1-4)
(combined Gateway/radio, controlled via HASS integration).
I ran on a 32GB microSD card for a few years but switched to an SSD because (a) Frigate filled it up
while I wasn't looking and (b) I wanted to move to InfluxDB and Grafana for fancy graphing, etc.
and this will have a lot more storage activity, shortening the life of the microSD card.
Update- I recently switched from the Pi4 to a mini-PC (Intel Celeron) for both performance and
because I was also having trouble with the Pi4 being able to detect the SSD.  Moreover, the SSD
connects to the mini-PC has direct M.2 and SATA SSD connectors whereas the connection to the Pi4
is via USB.

2. **HA Install**: I run
[Home Assistant Operating System](https://www.home-assistant.io/installation/generic-x86-64).

3. **Commercial Systems**:  
- [Hue lights](https://www.philips-hue.com/en-us) 
- Samsung Smartthings ([moisture sensors](https://www.amazon.com/Aeotec-SmartThings-Battery-Powered-Compatible/dp/B095TR9NYR/ref=sr_1_4?crid=2GGGMCCWZAQNT&keywords=smartthings+water+leak+sensor&qid=1660831204&sprefix=smartthings+%2Caps%2C97&sr=8-4)).
I was able to turn off the hub, reset these units, and add them to my HASS system via the Zigbee integration.
- [Orbit B-Hive irrigation system](https://www.amazon.com/Orbit-57950-12-Station-Controller-Compatible/dp/B01D15HOTU/ref=sr_1_5?crid=GI5GJOT02AR6&keywords=b-hyve+orbit&qid=1660832013&sprefix=b-hive+orbit%2Caps%2C96&sr=8-5)
- [TP-Link smart plugs](https://www.amazon.com/gp/product/B01KBFWW0O/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)
- [MyQ garage door controller](https://www.amazon.com/Smart-Garage-Opener-Chamberlain-myQ-G0401/dp/B08GD3D9YJ/ref=sr_1_1?crid=2B01WH1IK0Z5&keywords=myq&qid=1660832189&s=hi&sprefix=myq%2Ctools%2C113&sr=1-1) (works with most modern but *not all* openers) (Relies on MyQ cloud).
- [Aqara door sensors](https://www.amazon.com/Aqara-MCCGQ11LM-Window-Sensor-White/dp/B07D37VDM3/ref=sr_1_4?crid=Y9IKHD916DYA&keywords=aqara&qid=1660832246&s=hi&sprefix=aqara%2Ctools%2C90&sr=1-4) (all exterior). I was able to turn off the hub, reset these units, and add them to my HASS system via the Zigbee integration.
- [Lutron Caseta switches](https://www.amazon.com/s?k=lutron+caseta&i=tools&crid=3BAI2YG29DFSV&sprefix=lutron%2Ctools%2C79&ref=nb_sb_ss_ts-doa-p_2_6)
(I have not explored whether I can ditch the hub) Make sure you have relatively
modern wiring with ground in switch boxes).
- Aoycocr smart plugs that I
[hacked](https://www.youtube.com/watch?v=O5GYh470m5k&t=8s) to run
[Tasmota](https://tasmota.github.io/docs/). Sadly, Aoycocr updated their firmware
about 2 years ago so that this process no longer works.

4. **DIY sensors**: I used a
[Particle Photon](https://store.particle.io/products/photon) to create a
[utilities monitor](https://github.com/cecat/UtilityWatchMQTT) to track the duty
cycles and activities of my sump pump, water heater, and HVAC system.  This
system checks that state of these utilities every few seconds and reports to
HASS using MQTT. For the home 2h away I used a
[Particle Electron](https://docs.particle.io/electron/) (celular with
a backup LiPo battery, so does not
rely on power/Internet and thus can report such outages) to create a
[crawlspace temperature monitor](https://github.com/cecat/Lake-Watch) so that I
am alerted whenever the crawspace drops near to freezing temperatures.  

5. **HASS Add-ons**: InfluxDB database;, Grafana for fancy time series graphs,
Frigate, Studio Code Server, Terminal & SSH, Z-Wave JS, and MQTT (which Frigate
uses to communicate with HASS).

6. **Cameras**:  For Frigate I have three cameras. One is a
[Ubiquiti Unifi Protect G4-Bullet](https://www.amazon.com/gp/product/B08JCTVQ88/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)
PoE camera.  It's a really nice, solid, high-quality camera but pretty
expensive.  Moreover it requires a proprietary (and expensive) Ubiquiti Network
Video Recorder (NVR). Fortunately there is a third party compatible NVR package
that runs on my Synology NAS. This requires fiddling with Docker installs of
third party packages on the NAS system which is extra complexity and steps.  I
recently bought two much less expensive
[Amcrest Smarthome 1080p Wifi Outdoor Security Cameras](https://www.amazon.com/gp/product/B07WK8FH3X/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1).
These don't require a separate NVR so are a snap to get running. 

7. **About Networks**: You'll want to make sure your home wifi router can do DNS
reservations so that your server and cameras always get the same IP address.
This is important for configuring the cameras (via their RTSP URLs) as well as
for MQTT so devices can find the server.  If you plan to access HASS remotely
(it has a very nice smartphone app) you'll also want to forward port 8123 for
HASS and 1883 for MQTT.  For $5/mo though you can subscribe to
[Nabucasa](https://nabucasa.com/), a company run by Home Assistant founders that
provides remote access without opening ports on your router.

## Current Projects

I'm working on a couple of things at the moment.  Having set up Frigate to take
actions when certain objects (e.g., humans) are detected during certain time
periods (including sending a picture of the person on my Apple watch), I 
am working on a way to gently suggest to geese that they not congregate
on our lawn, where they leave nasty messes behind.
I have a Z-Wave connected 
[outdoor smart outlet](https://www.getzooz.com/zooz-zen15-power-switch/)
to power the well pump that we use for irrigation (using lake water).
This was easy to connect (no hub required) to HASS via the Z-Wave integration
(requires a Z-Wave USB gateway/radio, see my hardware setup above)).
I installed one of the Amcrest wifi cameras and set
Frigate to detect "bird" objects.  To avoid low (but non-zero) probability events that
might damage the well pump, I connected a
[pressure sensor](https://www.amazon.com/gp/product/B0748BHLQL/ref=ppx_yo_dt_b_asin_title_o04_s00?ie=UTF8&psc=1)
(that will screw into one of the ports of the well pump) to a D1_mini ESP device
to report the water pressure at the pump every 60s, letting
HASS decide whether it's operating safely (so it can automatically turn it off
if not).  In normal circumstances the pump runs at about 40psi, but if it loses
its prime it will sputter down to 20psi or so, and if someone were to turn off
all of the spigots it would shoot up to 90psi (though I have a relief valve, it
might still be hard on the pump).
I'm happy to say that the system detects geese reasonably well and turning on the
sprinkler causes the geese to immediately decamp (at around 7s into
[this clip;](https://www.dropbox.com/s/szf8qru47ypq84e/clip_goosecam_1663267722.24819-1663267741.046731.mp4?dl=0)
the sprinkler is on the dock and not easy to see but the effect on the geese
is obvious).  But the camera is still pretty
far from the lakefront so it does not detect them until they move up closer to
the house (and thus have been there for a while).  Next step will be to
move the camera down to the dock.

I'm also now using [ESPHome](https://esphome.io/) which is a very nice
way to create simple, very cheap, wifi-connected sensors for HASS.  For
instance, a [Particle Photon](https://store.particle.io/products/photon) is
about $20 and one can get ESP devices with GPIO pins (e.g., the
[D1_mini](https://www.amazon.com/gp/product/B081PX9YFV/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)
for about $3 each).  I built a set of these with
[DS18B20 temperature sensors](https://www.amazon.com/gp/product/B012C597T0/ref=ppx_yo_dt_b_asin_title_o04_s00?ie=UTF8&psc=1)
to evaluate the performance of our HVAC system throughout the home. The nice
waterproof sensors on 6' cables are about $2-3 each.  You'd be tempted, though,
to just solder one of the tiny (and even cheaper) 
[IC form factor DS18B20 sensors](https://www.amazon.com/Diymore-DS18B20-Digital-Thermometer-Temperature/dp/B01IVN3X6K/ref=sr_1_3?crid=3MF0PRS9NQ69X&keywords=ds18b20&qid=1662260376&sprefix=ds18b20%2Caps%2C119&sr=8-3)
to the ESP board.  This would be
very elegant (no wires, etc.) but you'd be measuring the heat coming off of the
ESP board rather than the ambient room temperature, so go for the ones on 
cables.  Below you
can see a grafana dashboard showing the temps in 8 rooms and a shaded outside location.
The relatively flat green
line is the temperature reading of the thermostat) and outside temperature 
is the purple line. The sensors report every ~60s and I'm graphing the time simple moving
average (built-in HASS filter) with a 10min window.

<img src="/media/temps.png" align="center">
