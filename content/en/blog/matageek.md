+++
title = "MATAGEEK"
tags = ['matageek']
date = 2021-06-25

# For description meta tag
description = "matageek"

# Comment next line and the default banner wil be used.
banner = 'img/matageek.png'

+++

![](img/matageek.png)

# What is MATAGEEK?
---
In a nutshell, it is a detection and notification system for hunting traps using BLE mesh network.

# Trap hunting is a tough job
---
Hunting is often associated with hunting rifles, but trap hunting is also an important method. However, in principle, the traps that have been set need to be patrolled every day.[^1] (To prevent non-hunting birds and animals from being caught, to prevent wounded animals from being created, etc.) Also, it is ideal to be able to take action as soon as possible when prey is caught.
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/96263081-cac6-4885-2b5d-d6f0faa54969.png)
So, if we can create something like the following image, it will be very useful.
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/8d2f2fb1-1e60-9222-44d6-5aff25b38ba4.png)
If it is something that sends a notification when the trap is activated, commercially available IoT modules such as SORACOM LTE-M Button[SORACOM LTE-M Button](https://soracom.jp/store/5208/)will be sufficient. However, in order to increase the capture rate of traps, it is common to place multiple traps in areas where animals are likely to pass, and placing an IoT module in each trap is a little difficult to say that it is possible for an individual in terms of cost. The cost of placing IoT modules in each trap is a little too high to be feasible for an individual.

# Mesh Network
---
In order to solve the above problem, we will use a mesh network using Bluetooth. There are IoT modules on the market that create a network with a parent device (that can use LTE lines) and child devices (that can communicate with the parent device over a short distance), but as far as I could find, the only ones that could be used for hunting are star-shaped networks. (If there are any, please let me know.)
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/2f2d1010-2abe-52d4-e1ff-b20e35fe1b86.png)
If you are familiar with Bluetooth, you may have thought `Bluetooth Mesh with Bluetooth 5.0`. However, if you are a little more familiar with Bluetooth, you must have felt uncomfortable with the title. The `Bluetooth Mesh` added in `Bluetooth 5.0` is based on `Bluetooth Low Energy (BLE)`, but **it is not a low power solution**.
In a nutshell, `Bluetooth Mesh` can't be turned off by specification to stay connected. And the radio is the main source of power consumption in Bluetooth devices. It is possible to run some nodes (one Bluetooth device in the mesh) in low power mode (Low Power mode), but the nodes that communicate with Low Power mode cannot be in Low Power mode. In other words, **it is impossible to operate all nodes in low power mode**.
In other words, it is impossible to run all nodes in low power mode. In fact, this mesh network achieves low power consumption by forming a mesh network using only the BLE function of `Bluetooth 4`.

# FruityMesh
---
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/19df096e-8859-64b8-defa-828207ac3a32.png)
There is a PJ called [FruityMesh](https://github.com/mwaylabs/fruitymesh) that creates mesh networks using only `BLE` of `Bluetooth 4`. I'd like to explain a lot about this PJ, but it would take up about three articles, so I'll skip the detailed explanation this time. In this PJ, we actually measured the power consumption and found that it was **less than 1mA** even during scanning (a function that searches for devices), which consumes a lot of power. Also, if it is possible to stop scanning when the number of nodes in the mesh reaches a certain level, it seems to be possible to run the system at approximately **150-250µA**, depending on the number of nodes. I'm going to make some changes to this so that I can get node information and control it from my phone. Thanks to our great predecessors🙏

# Composition
---

## Hunting traps module configuration
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/50e962de-9d79-3075-4b01-ccf83bd2ff61.png)

Form a mesh network with an arbitrary number of nodes, including at least one node in the mesh that has a gateway function (LTE line is available). The flow of setting up a node is as follows.

* Connect to a node from a smartphone, etc., and register the node to an arbitrary network (Enroll)
* Set the nodes in the mesh network to installation mode and place them in the traps.
* Put the nodes in the mesh network into detection mode

### Register (Enroll) a node to an arbitrary network
Nodes need to create mesh networks only with specific nodes (otherwise they will connect to each other if strangers place them nearby). For this reason, a node can belong to a specific network by providing the following two pieces of information. Therefore, a node can be made to belong to a specific network by adding the following two information (this is called "Enroll"), which is equivalent to "Provisioning" in `Bluetooth Mesh`.（Reference：[Provisioning a Bluetooth Mesh Network](https://www.bluetooth.com/blog/provisioning-a-bluetooth-mesh-network-part-1/)）
* Network ID
* Network Key

The network ID is the identification ID of the network to which you belong, and the network key is the common key to connect to the target network, which consists of 16 bytes. The network ID is the ID of the network to which you belong, and the network key is the common key to connect to the target network. This function can be realized by the standard function implemented in FruityMesh, but in the current implementation, it is necessary to enrol each node individually by wire (UART, etc.), so we have added the following function so that the process can be done only once.

* Nodes initially start with the default network ID and network key, and mesh networks are created between nodes that are also in the initial state.
* Connect to the node in the initial state with a smartphone and send the network ID and network key to the nodes in the mesh network.

However, since the initial state node forms a mesh network indiscriminately with other initial state nodes, the registration process must be performed in an environment where there are no other nodes other than the node in your possession.

### Installation mode and detection mode
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/fffb7bb2-e6a5-edfe-c20d-eb38d22af4a4.png)
The ability to switch modes is one of the features we added as a hunting module. The difference between these modes is mainly in the frequency of Bluetooth scanning, which in turn is a difference in power consumption. Placement mode scans more frequently and consumes more power, but makes it easier to find other nodes. On the other hand, a node in detection mode can consume much less power while maintaining its current connection and not searching for other nodes.
When setting up a trap in a hunting ground, the system is set to installation mode, and if a mesh network is formed as expected after the trap is set, the system is switched to detection mode for extended operation.

## Infrastructure
It sends a notification to the user when any of the nodes in the mesh is activated. In the diagram, there is a description of how the information can be viewed in a browser, but I won't explain that in this article, or rather I haven't done it yet 😂.

![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/46123292-871e-7002-a38d-8c093b3c0d78.png)

AWS IoT is used to issue certificates and private keys for using SORACOM BEAM encryption, which will be described later. This allows for individual encryption for each SIM, so if anyone wants to use this mechanism, security can be guaranteed for each.

# encryption
---
Encryption is not easy, but it is unavoidable. In this case, encryption is required for each of the two types of communication.

* Encryption of trap actuation information sent to AWS IoT
* Encryption of communications within a mesh network

Since communication in a mesh network is a local network, you need to go to the location where the traps are set up to sniff them. So, to be honest, I don't think we need to worry about it that much, but since FruityMesh already has the encryption feature, I'm grateful for it.

## Encryption of trap actuation information

The MQTT protocol, which is mainly used in IoT, is used to send the operating status of the trap, but since nothing is encrypted as it is, we would like to use MQTTS. However, using MQTTS is a bit difficult with the limited resources of a typical IoT device (in this case, we will use 64Mhz and 256KB SRAM). That's where [SORACOM BEAM](https://soracom.jp/services/beam/) comes in, a service that allows you to offload high-load processing such as encryption and connection settings to the cloud for IoT devices.
The device side simply sends unencrypted packets from the carrier's closed network to the Beam endpoint, as shown in the figure below. The Beam side then encrypts the received packets using the specified encryption method and forwards the packets to the specified destination.
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/db602a56-c17b-2387-4ac7-0f00ae952274.png)

## Encryption of communications within a mesh network

Although [pairing and bonding](https://www.bluetooth.com/blog/bluetooth-pairing-part-1-pairing-feature-exchange/#:~:text=Bonding%20is%20the%20exchange%20of,that%20allows%20bonding%20to%20occur.) encryption is defined in the `Bluetooth 4` specification, FruityMesh uses its own encryption. It uses its own encryption at the application layer for unencrypted communication of `GAP` (which is like a basic convention for Bluetooth communication and encryption is also specified here). (This specification seems to be intended for interoperability among different devices.) The encryption method is simply called symmetric key cryptography. When a connection is established between nodes, they authenticate each other by exchanging packets encrypted with a random number using a symmetric key within a certain timeout.

# Devices
---

## Devices for Mesh Networks
I'm talking about what kind of equipment to actually use. This time, we will use the [Nordic nRF52840](https://www.nordicsemi.com/products/nrf52840/), which is capable of communicating over a relatively long distance (about 30m in sight distance by actual measurement). There are not so many products that use this SoC and can be used in Japan (telec mark is required in Japan), and from the ease of purchase, I think the following two are the best. Adafruit has a SWD connector for writing programs via J-LINK, and SparkFun also has a SWD connector, but it is physically difficult to use.

* [SparkFun Pro nRF52840 Mini](https://www.sparkfun.com/products/15025)
* [Adafruit Feather nRF52840 Express](https://www.adafruit.com/product/4062)
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/adbbe057-a68f-213d-6836-94bcd5ff0405.png)

It seems that [SEGGER Embedded Studio](https://www.segger.com/products/development-tools/embedded-studio/) is often used for development of Nordic products, but it is also possible to develop with VSCode, and I have been developing with the latter. I have written a simple guide for development, which I would like to introduce here.

* [VSCode + ARM GNU ToolsでNordicの開発環境を構築する](https://qiita.com/nishinohi/items/545521df7b06e66149c9)

FruityMesh also introduces how to build the environment, so please refer to [here](https://www.bluerange.io/docs/fruitymesh/Quick-Start.html) when you build FruityMesh yourself.Since we are using CMake, it is a little different from the way Nordic development environment is built.

## Gateway
If you want to use a SORACOM BEAM, you should use a SORACOM SIM. If you use SORACOM BEAM, you should use SORACOM SIM. For your information, devices that have been confirmed to work with various SORACOM services are available [here](https://soracom.jp/support_partners/certified_device/). However, since the actual function you are looking for in a gateway is just to send packets using a carrier network, I can't deny that commercial gateways have too many functions. In the past, I made a small module just for using the carrier network, and I'd like to introduce it here for your reference.

* [Create a 3G module with WiFi and GPS functions](en/tags/hw/)

# Connecting Mesh Networks and Smartphones
---
FruityMesh is equipped with a function to accept commands like a terminal via wire (UART, etc.), and various settings and registration processes can be performed using it, but I would like to execute such operations from my smartphone anyway.
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/2ec10a07-89e3-d2fd-afeb-535368093ad2.png)

FruityMesh has a function to connect to a smartphone. However, there is no smartphone application that can connect to the FruityMesh mesh network, so we developed our own. I'm not going to post a link to it because it's currently only available for Android and the way to save the security key is completely out of the question, but if you're interested, I've published it on my [Github](https://github.com/nishinohi/FruityMeshAppUart) account.

The operation screen of the app looks like this

## Scanning a node
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/c4170459-f24e-a306-a29f-21ae613f65b3.png)
Only the advertisement packets (packets to have the Bluetooth device discovered) that contain the Service ID of FruityMesh will be displayed. Also, the initial state nodes described in the node enrolment will be displayed with gray icons. Note that `MataGeek` is the name of this PJ. (Matagi (refers to those who hunt in groups in Japan) + Geek)

## Node enrollment
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/2dcc4a33-11de-a26c-6556-8f6dd305fe6c.png)
Currently, pressing the ACTIVATE button will send the network ID and network key already stored in the terminal (smartphone). Enrolled nodes will be rebooted with the configured settings.

## Mode change
![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/167138/34d0af56-e54d-7d00-20d7-c5430520a21a.png)
Press START DETECT to change the mode to detection mode. Conversely, if you are in detection mode, you can change to installation mode. Basic information about the node can also be found here.

# Battery problem
---

With the above configuration, I think I have achieved the desired function. However, I'm not sure what I should do with the power supply since I'm completely self-taught in electrical circuits and don't have any proper knowledge. I don't know the correct answer to the following questions.

* Is there a small, high-dose battery that is non-flammable?
* I don't know how to design a power supply circuit for low power consumption.

For the battery, I would like to use a lithium-ion battery if possible, but I don't want to use anything flammable as long as the worst case scenario is that I might lose the node in the mountains.
I have always used a three-terminal regulator to power the electrical circuits I try. I know that this is not suitable for low power products, but I honestly don't know what kind of products are suitable. If anyone knows, please let me know.

[^1]: 第Ⅱ章 捕獲に関する基礎知識 - 農林水産省
