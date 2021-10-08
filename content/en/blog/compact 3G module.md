+++
title = "Creating a compact 3G module"
tags = ['matageek', 'hw']
date = 2018-05-16

# For description meta tag
description = "Creating a compact 3G module"

# Comment next line and the default banner wil be used.
banner = 'https://qiita-image-store.s3.amazonaws.com/0/167138/14906106-a395-36c1-ab82-4246a866a06d.png'

+++

# What I made
---
This is a 3G communication module with GPS function, and since I don't have any knowledge about HW design, I didn't design HW from scratch, but rather redesigned and manufactured a compact module by modifying a multifunctional 3G module provided by OSH.

# Why I made it
---
At the time I built this (2016 or so), there were no relatively cheap modules that could use the mobile radio network. At the time, the most common choice for a cheap hobbyist environment with microcontroller-based 3G communication was `Raspberry Pi ZERO + 3G dongle + SORACOM SIM`. However, the USB dongle was too big, the Raspberry Pi ZERO was too big, and the power supply resources were too tight, so I decided to make my own module that would work best for me.

# OSH (Open Source Hardware)
## Meet Adafruit
While wandering the net looking for a cheap 3G module, I came across a relatively inexpensive ($79.95) module called [Adafruit FONA 3G Cellular](https://learn.adafruit.com/adafruit-fona-3g-cellular-gps-breakout/) from Adafruit Industries, famous for their OSH. They also provide a library to use it with Arduino. It's all very well. However, it is not available for use in Japan due to the TELEC（TELEC certification is required for many wireless devices used in Japan）.
![](https://cdn-shop.adafruit.com/640x480/2687-01.jpg)

## Replace with a module that complies with the TELEC
Fona uses `SIM5320` as a module for 3G communication, but there is a module for Japan called `SIM5320J`, which is TELEC certified with a specific antenna. There is a module for Japan, called `SIM5320J`, which has an A rating with a specific antenna, so I can replace the module with it and use it in Japan. All Adafruit products are OSH, and all schematics are available in Eagle data format.

## Remodeling content
The Adafruit module has a lot of parts implemented to take advantage of the various functions of the SIM5320, but I thought I could reduce the number of parts if I only wanted to communicate with it, so I decided to modify the circuit as follows.

- Deleted speaker circuit for phone function
- Remove earphone jack
- Removed charge/discharge management circuit for lithium batteries (to drive AAA batteries)

# Analyze the schematic

## My level of knowledge of electrical circuits
I got all my knowledge about electronics from the Internet and YouTubers, but I still understood it well enough. I think you don't need to know anything about electric circuits to understand what I'm about to say.

## OSH
Thanks to Adafruit, I can clone the repository below.

- [FONA schematic（Eagle）](https://github.com/adafruit/Adafruit-FONA-SIMCOM-3G-Breakout-PCB)
- [FONA library](https://github.com/adafruit/Adafruit_FONA)

## Read the schematic.
If you don't have [Eagle](https://www.autodesk.co.jp/products/eagle/free-download)installed, install it first. You will find that it is not as complicated as you think. The schematics are described separately for each function, so they are easy to read. The wiring diagrams may look complicated at first glance, but the actual wiring can be generated automatically by Eagle's functions to some extent from the schematics.

![](https://cdn-learn.adafruit.com/assets/assets/000/027/324/original/adafruit_products_schem.png)
![](https://cdn-learn.adafruit.com/assets/assets/000/027/325/medium800/adafruit_products_fabprint.png)

## Delete unnecessary circuits
As mentioned in the first part, if I don't need any functions other than 3G communication, the following circuits are unnecessary and will be removed this time.

- Speaker for phone function
- Earphone jack
- Charge/Discharge Management Circuit for Lithium Battery

Also, since `FONA` itself is designed to be used with `Arduino`, the 3.3V signal from `SIM5320` is converted to 5V signal with a level shifter (the triangular part in the "LEVEL SHIFTING" section of the circuit diagram). However, I am planning to use the `ESP8266`, and since the 3.3V signal can be used as it is, the level shifter is unnecessary. Therefore, the following items in the circuit diagram are unnecessary.

- 3.5MM HEADPHONE/MIC
- LEVEL SHIFTING
- LIPO CHARGER
- AUDIO FILTERING

The schematic and wiring diagram after deleting them is as follows. It has become much simpler and easier. I wanted to make the antenna connector smaller, so I changed the connector for the 3G communication antenna from SMA to UFL, but it doesn't matter either way (there are conversion connectors).

![](https://qiita-image-store.s3.amazonaws.com/0/167138/2665f1e4-c606-0bfc-8fd9-628e0ebe85bf.png)
![](https://qiita-image-store.s3.amazonaws.com/0/167138/14906106-a395-36c1-ab82-4246a866a06d.png)

Please refer to the following link for the details of the modified schematic.

* [Adafruit-FONA-SIMCOM-3G-Breakout-PCB](https://github.com/nishinohi/Adafruit-FONA-SIMCOM-3G-Breakout-PCB)

## Place an order
Once the design of the base is complete, all that is left is to place the order. I ordered from [Seeed](https://www.seeedstudio.com/), a service called [Fusion PCB](https://www.seeedstudio.com/fusion_pcb.html), which not only manufactures the board, but also mounts the components for a reasonable price.
This time, the price was about `$100` for one surface mount and five boards only, which is about `$20` more than FONA, but it is amazing that they were able to keep the price so low for a single order.


The following article is very helpful for the detailed process of ordering. Also, Seeed's [twitter](https://twitter.com/SeeedFusion?lang=ja) page is available in Japanese and they can answer your questions.

* [Fusion PCBAでハードウェアを「コンパイル」する](https://qiita.com/mayfair/items/0206bd437c4302be5500#fnref1)

## Wait for it to arrive
Apparently, the availability of the SIM5320J was not good and it took about a month for delivery. It took about a month for delivery. There are some mysterious wires sticking out, but they are traces of a design error that forced me to repair the regulator wiring.

![](https://qiita-image-store.s3.amazonaws.com/0/167138/8ee1a381-8110-9020-147b-83811be9ffe6.jpeg)

# Try it

## Fixing the library
The library provided by Adafruit is not compatible with the new firmware of the `SIM5320` and I had to fix it by looking at the documentation on the AT commands of the SIM5320. The Adafruit website says that the library is currently undergoing a major overhaul, but it has been abandoned for about 9 months.

## The completed figure
Here is the module that is actually ready to run.
It looks like this, but it recognizes the SIM and publishes various things to AWS IoT via MQTT.

![](https://qiita-image-store.s3.amazonaws.com/0/167138/058dd852-7efb-5e7f-cd9c-ad9973689649.jpeg)

# Next step
This time, I created it as a module only, but I would like to redesign it as a simple Gateway by incorporating a microcontroller.