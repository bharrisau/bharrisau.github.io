---
layout: post
title:  "USB PID/VID: A mitigating solution"
categories: usb oshw pid vid
description: "An attempt to mitigate the issue of finding a VID/PID by providing a USB communication spec that covers the majority of OSHW devices."
---

There has been [recent][arachnid] [coverage][hackaday] of the difficulty small projects have in getting a PID/VID for their USB devices. Even if they are implementing devices that only use class interfaces (and don't need a driver), the device still needs a VID/PID to enumerate with. One solution is to try and get USB-IF to sell smaller blocks of numbers, but they are hesitant to do that because the VID/PID isn't a 32-bit address space they can apportion how they want (this is a political, not technical distinction).

Another solution is to get one VID/PID that all these simple projects can use, and specify a minimal protocol that they need to implement to be compatible with a generic driver. I'll take the first stab at this, hopefully it gets some traction.

<!--excerpt-->

I previously had some stuff written up here, but it was brought to my attention that DFU and CDC-ACM are sufficiently lightweight and tools already exist for them. So here is the drastically simplified proposal.

#### Devices with host<->device communication
The VID/PID pair of 0x????/0x???? is available to support these devices. The device is expected to provide a CDC-ACM interface to emulate a serial port. The device MUST comply with the following to use this VID/PID pair:

1. The device MUST implement CDC-ACM. There are no requirements on the capabilities provided by the interface (i.e. bmCapabilities may be any value allowed by the spec).
2. The device descriptor MUST set bDeviceClass, bDeviceSubClass, bDeviceProtocol, and bcdDevice to be 0x02, 0x00, 0x00, and 0x0010, respectively.
3. Device descriptor MUST specify manufacturer and product descriptions. The product description MUST be available at least in language 0x0409 (English/US).
4. The product description MUST include a URL to a project page for the device. The product page MUST provide documentation of the protocol implemented on the serial port.
5. The device MAY provide a DFU interface that complies with the DFU specification.

#### Devices in DFU mode
The VID/PID pair of 0x????/0x???? is available to support these devices. The device MUST comply with the following to use this VID/PID pair:

1. The device MUST implement the DFU specification.
2. The device descriptor MUST set bDeviceClass, bDeviceSubClass, bDeviceProtocol, and bcdDevice to be 0x00, 0x00, 0x00, and 0x0010, respectively.
3. Device descriptor MUST specify manufacturer and product descriptions. The product description MUST be available at least in language 0x0409 (English/US).
4. The product description MUST include a URL to a project page for the device. The product page MUST provide documentation of the format used for the firmware file.
5. The DFU interface descriptor MUST provide a string descriptor. This descriptor MUST be available at least in language 0x0409 (English/US).
6. The device MAY provide alternate settings for the DFU interface. If alternative settings are used, all interface string descriptors MUST be unique.

#### Devices only providing a DFU interface
The same VID/PID pair used for 'devices in DFU mode' may be used for these devices. The device is only permitted to provide a DFU interface. When in run mode, the device MUST comply with the following to use this VID/PID pair:

1. The device MUST implement the DFU specification for run-mode with the exception that the device descriptor and configuration descriptor MUST be implemented from the DFU mode specification.
2. The device descriptor MUST set bDeviceClass, bDeviceSubClass, bDeviceProtocol, and bcdDevice to be 0x00, 0x00, 0x00, and 0x0010, respectively.

#### Definitions
DFU: [USB Device Firmware Upgrade Specification](http://www.usb.org/developers/devclass_docs/DFU_1.1.pdf).  
CDC: [USD Communications Device Class](http://www.usb.org/developers/devclass_docs/CDC1.2_WMC1.1_012011.zip) (see CDC120.pdf).  
ACM: [Abstract Control Model](http://www.usb.org/developers/devclass_docs/CDC1.2_WMC1.1_012011.zip) (see PSTN120.pdf).

[arachnid]: http://www.arachnidlabs.com/blog/2013/10/18/usb-if-no-vid-for-open-source/
[hackaday]: http://hackaday.com/2013/10/22/usb-implementers-forum-says-no-to-open-source/