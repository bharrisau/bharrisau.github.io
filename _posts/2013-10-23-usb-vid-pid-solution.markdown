---
layout: post
title:  "USB PID/VID: A mitigating solution"
categories: usb oshw pid vid
description: "An attempt to mitigate the issue of finding a VID/PID by providing a USB communication spec that covers the majority of OSHW devices."
---

There has been [recent][arachnid] [coverage][hackaday] of the difficulty small projects have in getting a PID/VID for their USB devices. Even if they are implementing devices that only use class interfaces (and don't need a driver), the device still needs a VID/PID to enumerate with. One solution is to try and get USB-IF to sell smaller blocks of numbers, but they are hesitant to do that because the VID/PID isn't a 32-bit address space they can apportion how they want (this is a political, not technical distinction).

Another solution is to get one VID/PID that all these simple projects can use, and specify a minimal protocol that they need to implement to be compatible with a generic driver. I'll take the first stab at this, hopefully it gets some traction.

<!--excerpt-->

**Note:** This has all been done in a couple of hours, based on the features I've previously needed in my devices. I'm not claiming it is complete or anything close.

**Note:** Class interfaces (mass storage, HID) are easy to implement. Just add them to the descriptor and the host will start up a driver automatically.

**Note:** Many devices need a bulk endpoint to transfer data (e.g. JTAG adapters). While not explicitly supported, nothing in this specification prevents additional endpoints. Vendor specific code is used to support these additional endpoints. The memory map can be used to change the device mode (or a custom vendor request can be implemented).

### 1. Introduction

### 2. Device descriptor
This specification defines a standard format for some fields of the device descriptor.

#### 2.1 idVendor
The idVendor to be used for this specification is 0x1D50. *(hopefully)*

#### 2.2 idProduct
The idProduct to be used for this specification is *0x????*.

#### 2.3 bcdDevice
bcdDevice is used to indicate which version of the specification is implemented on the device. The value to be used for this version is 0x0010.

#### 2.4 iManufacturer and iProduct
iManufacturer and iProduct are used by the driver to provide device specific customisation. For example, custom vendor requests. Implementers not wanting to provide customisations for the driver may omit these strings, or may use any string that does not include the reserved characters '[' or ']'. Implementers wanting to specify the manufacturer and product according to this specification shall use strings of the form:

*The following is not completed. How should identifiers be done? URL or a UUID?*
{% highlight js %}
/[^[]*\[[a-zA-Z0-9 .:/]\]/
{% endhighlight %}  

An example string is "Small red widget [product identifier]"  

### 3. Control Endpoint
#### 3.1 Vendor-Device Requests
The standard communication interface for a simple device is through vendor requests. The format of the setup packet for these requests is described below:

**bmRequestType:** is 0xC0 for a device->host transfer, or 0x40 for any other transfer.  
**bRequest:** specifies the command. Values less than 0x80 are reserved by this spec.  
**wValue, wIndex and wLength:** are specific to each command. 

##### 3.1.1 Get Status (0x00)
Get status is a device->host message. The host makes the request with wValue=0, wIndex=0, wLength=2, and the device responds with an 2 byte data phase. The format of the 2 bytes is:  
1 byte status  
1 byte return value from last operation

The status is 0xFF for programming mode. It is recommended that 0x00 be used for run mode, but the device can use any value other than 0xFF for custom modes. 
Th standard format for return values is 0x00 for success, 0x01 for any command still running, and 0xFF for error.

##### 3.1.2 Reset (0x01)
Reset is a host->device message. wIndex and wLength are not used and are set to 0x00. wValue specifies the mode to reset into: two reset modes are specified.

###### 3.1.2.1 Standard reset mode (0x00)
Standard reset is the same as power cycling. The device will reset and start executing user code. The device should wait until the control transfer is completed to execute the reset. No return value is set for this command.

###### 3.1.2.2 Programming reset mode (0x01)
Programming reset is used to reprogram the flash. If supported, the device will stop executing user code and will ready itself to receive programming instructions. The return value will be 0x00 if the device is ready, 0x01 if it is still getting ready, or 0xFF if the command is unsupported.

##### 3.1.3 Features (0x02)
Features is a device->host message. The host makes the request with wValue=0, wIndex=0, wLength=8, and the device responds with an 8 byte data phase.

The format of the 8 bytes is:  
2 byte memory map length  
1 byte options flag  
5 bytes - reserved (0x00)

The bits in the options flag are as such:  
bit 7 - programming mode supported  
bit 6 - partial erase supported  
bits 5-0 - reserved (default to 0)


##### 3.1.4 Read (0x03)
After issuing the Features command, the driver may request a read of any address in the memory map. Read is a device->host message. The host sets wIndex to the start of the target memory range, and sets wLength to the length of the target memory range. wValue is not used and defaults to 0x00. The maximum length for wLength is 8 bytes.

The device responds with wLength bytes of data. If any of the targeted memory address is outside the supported area, the device uses 0x00 for those bytes.

##### 3.1.5 Write (0x04)
After issuing the Features command, the driver may request a write of any address in the memory map. Write is a host->device message. The host sets wIndex to the start of the target memory range, and sets wLength to the length of the target memory range. wValue is not used and defaults to 0x00. The maximum length for wLength is 8 bytes.

The device ignores any data outside of the supported memory range. The return value is set to 0x00 if the write is finished, 0x01 if it is still processing, or 0xFF if there was an error.

##### 3.1.6 Mass Erase (0x10)
Mass erase is a host->device message. wValue, wIndex, and wLength are not used and default to 0. This command is only valid in programming mode. The return value is set to 0x00 when erase is finished, 0x01 if it is still in progress, or 0xFF if the device is not in programming mode. It is not expected that mass erase can fail.

##### 3.1.7 Partial Erase (0x11)
Partial erase is a host->device message. wValue, wIndex, and wLength are not used and default to 0. Partial erase only deletes the user code, leaving any flash reserved for non-volatile storage in tact. This command is only valid in programming mode. The return value is set to 0x00 when erase is finished, 0x01 if it is still in progress, 0xFF if the device is not in programming mode, or oxEE if an unknown error occurred. If the return value is 0xEE the device may be in a non-bootable state, the driver should reattempt partial erasure, or inform the user and request a mass erase.

##### 3.1.8 Program (0x18)
Program is a host->device message. wValue is the first flash address, and wIndex is the last (inclusive). wLength is set to the length of the data, or wIndex-wValue+1. The maximum length is 32 bytes. The return value is 0x00 if flash writing is done, 0x01 if it is still in progress, or 0xFF if an error occurred. 

##### 3.1.9 Checksum (0x1C)
Checksum is a host->device message. Checksum confirms the checksum of the flash. The checksum is calculated using CRC16 on the specified flash areas. wValue is set to the target checksum. The return value is 0x00 if the checksums match, 0x01 if the command is still processing, or oxFF if the checksums do not match.

wIndex is used to specify the data included in the checksum, the acceptable values are:  
0x00 to include all flash
0x01 to exclude the user non-volatile storage

### 4. Misc
#### 4.1 Flash layout
Devices implementing partial erase must reserve a range of flash for non-volatile storage. If it is desired for the host to read the non-volatile storage, it should be made accessible through the memory map (and hence the read command).

#### 4.2 Minimum specification
For a device that does not provide programming mode, the only required commands are Get Status, Reset, Features, Read and Write.

If a device only uses class interfaces and does not need a device driver, it may set bcdDevice to 0x0000. Devices electing to use this minimum specification must not implement any vendor requests on the control endpoint, nor may they use any interface with bInterfaceClass=0xFF. That is, they are only permitted to implement interfaces that can use a class driver on the host computer. The device driver will always select the first configuration for these devices.

[arachnid]: http://www.arachnidlabs.com/blog/2013/10/18/usb-if-no-vid-for-open-source/
[hackaday]: http://hackaday.com/2013/10/22/usb-implementers-forum-says-no-to-open-source/