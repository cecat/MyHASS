---
layout: default
title: "Finite State Machines and Autonomy (of the simplest kind)"
date: 2024-01-10
categories: integrations hvac
---

One of my motivations for doing home automation has been
to keep the pipes from freezing at a remote property. In winter we reduce
the heat at 63F to be economical (not wasteful) but high enough that we
don't get mold everywhere (which seems to happen below about 60F).  The
crawlspace is pretty well sealed and unless the outside temperature
dips below perhaps -10F for a long period of time it will stay above 
freezing if the house is reasonably warm (70-75F).  I should probably
put a space heater with thermostat in the crawlspace, but for now I'm
using waste heat from the house to keep the crawlspace above freezing,
and this is only needed a few days a year.

Several years ago I installed
a [Particle Electron](https://docs.particle.io/electron/), which is
an Arduino-compatible, cellular connected 
(3G, very inexpensive service) device. I outfitted this with a 
[DS18B20 temperature sensors](https://www.amazon.com/gp/product/B012C597T0/ref=ppx_yo_dt_b_asin_title_o04_s00?ie=UTF8&psc=1), 
drilled a small hole in the floor (under the fridge), and dangled
the temperature sensor in the crawlspace.  Cellular coverage is not 
great at this location, so I upgraded the
[Taoglas antenna](https://docs.particle.io/assets/datasheets/PC104.07.0165C.pdf)
 that came with the Electron with a slightly larger one. Still the
connectivity is spotty, but I have this report (using MQTT) to my Home
Assistant (HA) server every 5 minutes though I have seen some long (>1h) gaps
so may need to revisit this. So far, at least when the temperatures are
low and it matters, this has not had an impact.  The Electron is plugged
into a wall outlet,
but also has a LiPo battery, so I check where it's getting it's power
as well, and if it's drawing from the battery that tells me the power
is out at the property, and I won't have Internet thus no way to
adjust the temperature (this also triggers a MQTT message with
a topic associated with a boolean on HA, so I track the power and 
get automated alerts if it goes out.
Fortunately that is an edge case that has only
materialized once or twice in 25 years so far.

In addition to reporting crawl space temperature, the Electron will send 
a warning message (using the MQTT topic mapped in HA to a boolean) when
the crawl space drops to 37F, and a similar warning message when the 
temperature hits 35F.  I had automations to send me texts, turn on 
lights, etc. when these were received so that I could manually increae
the heat via remote access to my Nest thermostat.

While we were experiencing the deep freeze around here in January 2024
I decided to automate these adjustments.  (Having said this, *of course*
it makes much more sense to put a heater with a thermostat in the
crawlspace! But for now , partly because we almost never reach any
danger levels, I'm using waste heat from the house as an interim hack.)

The code is
[github.com/cecat/Crawlspacer/](https://github.com/cecat/Crawlspacer/) 
and also has some internal logic to send warning messages if the
temperature hits 35F and again if it hits 32F.  (more on this below)

The Electron gives me crawlspace temperature,
and for outdoor temperature I use a 
HA weather plug in ([Meteorologisk Institutt](Met.no)),
though there are several 
like this. In this case, based on the Lat/Lon you configure it finds the
closest National Weather Service station.

I implemented a rules-based control system using a finite state machine
approach with four states, each a 'danger level,' 
zero through 3.  State changes  are each triggered by 
outdoor temperature (OT) and crawlspace temperature (CT) thresholds.
When moving into a given state the
thermostat setting is increased or decreased:

|Danger Level | Outside Temperature | Crawlspace Temperature | Set HVAC to: |
| :---        | :---                | :---                   | :---         |
| Level-0     |       T >= 20F       | n/a                   | 63F          |
| Level-1     | 10F < T <  20F       | n/a                   | 68F          |
| Level-2     |       T <= 10F       | >  39F                | 70F          |
| Level-3     |       T <= 10F       | <= 39F                | 73F          |
| | | | |
| ALERT 1     |       n/a           | <=35F                  | <SMS msg>    |
| ALERT 2     |       n/a           | <=32F                  | <SMS msg>    |
 

I kept the original alert messages for 37F and 35F, which are shown
as the extra alert levels at the bottom of the table.
As before, these trigger more agressive automations than just SMS messages,
including turning on a warning light in our bedroom, since these indicate
that human intervention may be needed.  It's a 2 hour drive (if I'm home!)
so if we hit these temperature levels I decide whether
to go drain the pipes.  If I decide correctly then I never get the 
warning message that the temperature dropped to 32F!

I also explored (along with a buddy named GPT-4) other methods such as
ML models or PID, but even with
2+ years of data the subset that is useful (when the temperatures
are very low) is quite tiny from a model training point of view. I did 
do some analysis, for the periods where the outdoor temperature was below
40F, of the correlations between outdoor temperature and
crawlspace temperature (pulling the latter down) as well as between
the crawlspace temperature and indoor temperature (pulling it up).
In very round numbers, the crawlspace will drop 1 degree for every 4 degree
drop outside. And the crawlspace temperature will increase one degree
for every 3 degree increase in indoor temperature.

