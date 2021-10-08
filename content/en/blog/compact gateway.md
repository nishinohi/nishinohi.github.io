+++
title = "Creating a Compact Gateway"
tags = ['matageek', 'hw']
date = 2020-06-25

# For description meta tag
description = "Creating a Compact Gateway"

# Comment next line and the default banner wil be used.
banner = 'https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/0b931b39-7010-718a-dd5f-1441b42e4556.png'

+++

# What I made
---
Using the [3G module created previously](/en/blog/compact-3g-module/) as a reference, I have created a compact Gateway that combines a microcontroller and an LTE module.

# OSH
---

## Products for reference

First, get a schematic from OSH. In this case, I will refer to the [circuit diagram of Wio LTE](https://wiki.seeedstudio.com/Wio_LTE_Cat.1/#resource), which is also a `SORACOM` certified device and provided by `Seeed`. However, if you order PCBA from Seeed, it is cheaper and faster to use the parts list provided by Seeed. However, if you order PCBA from Seeed, it is better to use the parts list provided by Seeed.

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/be013c2a-44ea-5685-80b6-ce512f362d3e.png)

## Read the schematic

As before, remove unnecessary circuits except for communication. The `Wio LTE` has a microcontroller called `STM32F4`, but this time I wanted to add WiFi function, so I replaced the microcontroller with [ESP32](https://www.espressif.com/en/products/socs/esp32).Also, the Wio LTE had a three-layer implementation, but I thought I could fit it in two layers by reducing the number of parts, so I decided to implement it in two layers.

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/f465b3cb-c4f8-cee5-9b40-156178dd0e87.png)

# Remodeling operations
---
## Remodeling content

The following is the circuit diagram and wiring diagram for this modification. Since only the communication function is needed, the number of parts has been reduced considerably from the above schematic. I have already replaced the microcontroller with an ESP32, and I have placed the WiFi antenna so that it is outside the board to avoid interference, but I don't know what kind of placement is best for the antenna since I have no knowledge of radio waves.

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/6c3f39db-0c74-3454-c8de-9f69e457b944.png)

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/39b15ca3-394e-8e31-a0e0-4e27ce5c8f31.png)

## Finished product

Let's take a look at the finished product.There are some wires sticking out that I was forced to add. This was added because I needed to tweak the PIN to activate the LTE module a bit.

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/bb3ded8d-9348-0f1b-ef78-6bdc3db6c787.png)

## Trouble

The SIM card holder provided by Seeed for Eagle had a flaw in the library, and if I wired it as is, it would not read the SIM correctly. So I had to melt the solder and forcibly pull off the SIM card holder and rewire it. It didn't look good, but I was relieved to see that it worked correctly. By the way, Seeed sent me a coupon to use in Seeed with their apology.

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/0b931b39-7010-718a-dd5f-1441b42e4556.png)

# EC25-J is a special acquisition channel
---

In fact, the `EC25-J`, which is implemented as a module for LTE this time, was originally printed with a technical compliance mark, which **cannot be ordered by individuals**. I talked to an engineer and marketing manager of Seeed whom I met when I was speaking at a Seeed-sponsored workshop, and he said he would arrange for `EC25-J` to be implemented, and this module was completed. I would like to take this opportunity to thank him very much.