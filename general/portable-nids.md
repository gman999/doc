##Building a Portable Network Intrusion Detection System (NIDS)##
_An ALIX board running OpenBSD with Snort_

Put on an APU or a BB, plus in and wait for alerts

Find malware, local network attacks, suspicious c/b traffic

__Hardware Options__

The two most common choices for small embedded systems are x86, such as PCEngines' [APU](http://pcengines.ch/apu2.htm) or [ALIX](http://pcengines.ch/alix.htm) boards or more recently popular ARM hardware, such as the Raspberry Pi and BeagleBone.

While ARM support is improving, x86 hardware remains the best supported, particularly with network device drives.

__Operating System__

While these directions are adaptable for any Unix-like system, in this case the option is [OpenBSD](https://www.openbsd.org/), an open source operating system noted for its focus on security and following [POSIX](https://www.opengroup.org/austin/) standards. Both security and standards are critical attributes for a NIDS device. Security is obvious enough, since an operating system that requires prolific knob-turning is not by default secure. Standards are equally important, as following accepted network protocols and communications allows interoperability for the device on any network, regardless of the array of devices.

__Building__

The device's operating system install should be small and light. In this case, an ALIX board is used with [OpenBSD's -stable branch](https://www.openbsd.org/stable.html).

The ALIX board uses a compact-flash card for storage.

Configuring

First, install the snort 

Using

http://www.instructables.com/id/Raspberry-Pi-Firewall-and-Intrusion-Detection-Syst/

https://www.sans.org/reading-room/whitepapers/forensics/portable-system-network-forensics-data-collection-analysis-37100


