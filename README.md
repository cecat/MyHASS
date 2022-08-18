# A Primer on Home Assistant (HASS)
Charlie Catlett (August 2022)

(note- I am referencing specific products but I have no relationship with any of the companies, and will not make any money if you buy them.  All it means is that they work for me, but there are likely other options in many cases as well.]

## If you are anxious to get started...

You could skip the stuff below and head straight to the Statup Guide, which is fabulous.  You may also want to check out some of the getting started tips (selecting hardware, mistakes to avoid, etc.) at my favorite source of help for Home ssistant - [Everything Smart Home](https://www.youtube.com/c/EverythingSmartHome).

## Why am I using HASS?

There are lots of smart/automated home products, many of which interoperate, all of which have their own apps, many can be aggregated through Amazon, Apple, or Google home devices, and pretty much *all* of them use cloud services, making your home rely on working Internet and spreading your data around the world.  You can also do pretty powerful things with services like IFTTT, but these are very simple things (one trigger, one action) so you can't even do simple things like "turn on the light for 10 minutes" (turn on light; wait 10 min; turn off light - 3 actions, 2 too many for IFTTT) much less more sophisticated things such as actions that have conditions (only do X from 12am to 5am).

I started with DIY systems I built myself in about 1999, the first system being an old fashioned thermostat wired to a Radio Shack auto-dialer (and a big (big) 12v battery in case of power outages), so if the power or heat went out in winter I'd get a call (I got several calls).  Multiple generations of DIY later I found myself accumulating many very nice (but independent) home automation systems.  Philips Hue lights (and a hub), Samsung Smartthings (and a hub), Amazon Blink cameras (and a hub), Aqara door sensors (and a hub), various smart outlets (no hub, or rather the hub is in China...), etc.  All of those hubs rely on Internet connectivity to commercial clouds. 

I adopted HASS to (a) bring all of these systems together, and (b) move as much as possible away from reliance on Internet/cloud for my automations and data.

## What am I doing with HASS?

I have a few important functions that I implement with HASS, which I would say fall into several categories.

1. Avoiding Unpleasant Events (or at least getting an alert when they happen).  I have Samsung Smartthings moisture sensors located near the sump pump well, next to the water heater, and under the kitchen sink.  I am immediately notified (lights turning on, text message) if moisture is detected.  I have a (DIY) sensor to keep track of how often the sump pump runs (how worried I should be if it were to stop), how often the water heater runs, and when the HVAC system is running.  This gives me a feel for normal behavior, and on several occasions that has helped me anticipate problems and prevent them.

2. Safety.  In some spaces, such as the garage, or back deck, I have lights automatically turn on when doors open or movement is detected (no fumbling with wall switches).  I use Aqara door sensors to detect when any of our exterior doors open, so I can set an alert if that happens when no one is home or in the dark of night.  I use Blink cameras just to be able to check on things (for instance, one is pointed at the sump pump so if it is running a lot I can check in on it (from anywhere).  I use several types of cameras along with a package called Frigate to detect when a human is seen in the front or back of the house (and depending on the time of day or whether we are at home) the system saves a 30s clip and I can have the system do nothing or trigger a sequence of actions.  

3. Remote Control.  From anywhere on the Internet I can check on things, turn lights on or off, make sure doors are closed, open or close the garage doors, etc. I can also control the lawn irrigation system.  All of these things are possible with individual apps from the companies that make those systems, but in HASS I have my own dashboard for the home.

4. Fun Stuff.  I use Life360 with a family circle that HASS uses to keep track of how many of us are home (allowing me to have certain automated actions related to security when no one is home).  I have another circle of a half dozen co-workers and keep track of how many of us are at our lab.  At 4:30pm on weekdays HASS checks to see if 4 or more of us are there, and if so all of us get a text (and slack) message suggesting we go to the pub.

## What are a couple of things to get rolling?

My suggestion is to start with the systems you have (if any), and integrate them with HASS.  If you don't have any systems, you might start with something like moisture detection, smart outlets for things like holiday lights or outdoor lights (e.g., turn them on at dusk and off at midnight).  If you are adventurous, you might skip right to Frigate and have the lights turn on when a human shows up after dark (yes, you can do this with many products via motion detection, but they can't tell the difference between a human and a strong wind hitting tree branches).  From there you could try some of the systems I use or other things you find in the plentiful online documentation, community forums, and YouTube channels from the HASS community.

## How do I begin?

### Hardware.

1. Your server.  You will want at minimum a Raspberry Pi 4, but you might also consider more powerful platforms such as a Nuc. The HASS installation pages show other platforms that are supported.

2. Storage.  If you have a very simple system in mind you can get by with a small (e.g, 32GB microSSD) on a Pi4 as suggested on the installation pages linked below.  But chances are you will want to do more ambitious things like Frigate or better database options, in which case you should buy an SSD (go for 1 TB at least).

3. Other.  If you plan to use Frigate, you will need to either go with one of the beefier servers or get a Coral TPU (USB device that plugs into your server).  A Pi3 is probably fine if all you are doing is controlling lights and monitoring sensors, but Frigate will bring it to its knees (it will be unresponsive).  A Pi4 can barely keep up with 1 camera, using 80% of its CPU to do so and giving you sluggish performance interacting with the system.

NOTE: Unfortunately as of August 2022 it is nearly impossible to find either a Pi4 or a Coral TPU (escept from opportunitists at insane prices). You could get started with a Pi3 but don't try to use Frigate as it will make the system more or less unresponsive, using 99% of the CPU.

### Software and Installation

Here you should dive into the fabulous online documentation that will walk you through.  I would advise that you go with **Home Assistant Operating System** (the first option on the "Installation" page.  The other options are more advanced, but also more complicated.  For instance, you can install Home Assistant Container as a docker image if you have a beefy server (e.g., you can run Docker and there is a version of Home Assistant Container that runs on Synology NAS systems like mine). But an important component of Home Assistant called Supervisor must then be installed separately in order to get functions that you will need.  So unless you are good with Linux and Docker, and have something specific in mind you want to do that requires this approach, go for Home Assistant OS.  (you can migrate easily later, as HASS has nice backup/restore functions)

### Follow the **[Getting Started guide](https://www.home-assistant.io/getting-started/)**!


## Where can I find more help and ideas?

Googling helps, but there is also an active [Home Assistant Community](https://www.home-assistant.io/getting-started/join-the-community/) you can join, which has lots of resources and tutorials.  I myself have found [Everything Smart Home](https://www.youtube.com/c/EverythingSmartHome) to be excellent for both beginning and advanced things.  For instance, tutorials on [installing your first system](https://www.youtube.com/watch?v=SHg6fa0x7OA), one on [mistakes to avoid](https://www.youtube.com/watch?v=i1083cCR2CI), and another on things to do once you are up and running. You might start with the *Home Assistant Guide* [playlist](https://www.youtube.com/c/EverythingSmartHome/playlists) there.
