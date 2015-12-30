Soekris 4801: Continuing Relevance After All These Years

The venerable [Soekris 4801](https://soekris.com) with its 128 megabyte of RAM and slow 233 MHz processor can still maintain a useful role. Thousands reside quietly in drawers and closets around the globe, imagined to be far past their expiration date.

OpenBSD is an ideal candidate operating system for the Soekris, since it's built lean by default, yet an array of functions are available in the base system.

Such a device could be a serial console server, a recursive/caching DNS, a lightweight monitoring server, etc., without installing any third-party software.

###The Install###

Needs:

* bootable media with OpenBSD/i386 stable version on a USB stick
(using snapshots is not ideal in this scenario)

* compact flash (CF) card with adapter

* bootstrap i386/amd64 computer with any operating system

* a serial console cable with a null modem

With the bootable media attached via USB, launch the bootstrap computer and enter the OpenBSD installation program.

Type i to enter the install system.

	Welcome to the OpenBSD/i386 5.8 installation program.
	(I)nstall, (U)pgrade, (A)utoinstall or (S)hell?

Plug in the CF card adapter and note the device listed to standard output, normally sd(4) with a respective number, such as sd2.

The OpenBSD installation program is graphics-free, yet clear and simple to follow.

Hit "enter" for [default]

	Choose your keyboard layout ('?' or 'L' for list) [default]

Designate the system's hostname

	System hostname? (short form, e.g. 'foo')

Next the list of interfaces will be listed, both wired and the recognized wireless devices. This can be configured for DHCP, as it will only be used to pull the relevant install sets from an OpenBSD mirror. Note that the interfaces only apply temporarily to the bootstrap computer, and will need to be adjusted manually on the actual Soekris device which uses sis(4) for the network interfaces.

	Password for root account? (will not echo)

sshd(8) will likely be necessary

	Start sshd(8) by default [yes]?  

For the functions listed previously, X Windows and xdm(1) are unnecessary.

	Do you want the X Window System to be started by xdm(1)? [no]  

A non-privileged user is an important step in running a secure system.

	Setup a user? (enter a lower-case loginname, or 'no') [no]  

	Allow root ssh login? (yes, no, prohibit-password) [no]

	What timezone are you in? ('?' for list) [(local timezone guess)]

Next is determining the install disk.

	Available disks are: sd0 sd1 sd2.

In this case, standard output noted that the CF card is located on sd2. An incorrect choice here, such as selecting the bootstrap computer's disk, will mean overwriting the wrong disk.

Select ? to verify the correct disk.

	Which disk is the root disk? ('?' for details) [sd0] ?
....

###Pre-First Boot Configuration###

There is an important set of changes to make to successfully boot the Soekris board with OpenBSD as there is no VGA output, and the default output is to console.

The default install will cease booting at this line unless console access is enabled.

	entry point at 0x200120

The easiest way to make these changes is to mount(8) the CF card on an OpenBSD computer and manually make the changes.

	mount /dev/sd2a /mnt

In /etc/ttys, change this line:

	tty00	"/usr/libexec/getty std.9600"	unknown	off

to this:

	tty00	"/usr/libexec/getty std.9600"	vt220	on secure

Create /etc/boot.conf, and add the following line:

	set tty com0

###Final Configuration###

Setup the non-privileged user with doas(1) permissions in /etc/doas.conf(5):

	permit nopass keepenv {user} as root

Create /etc/rc.conf.local for minimizing unnecessary daemons, such as sndiod(1) and smtpd(8):

	sndiod_flags=NO
	smtpd_flags=NO

Install the CF card into the Soekris, plug in an ethernet cable to the first port on the right and plug in the serial console cable.

###Booting Up###

Setup the bootstrap computer with the serial console cable and some terminal emulation application.

By default, the Soekris 4801 used 19200 as the console speed, yet we set the speed to 9600 in the /etc/boot.conf and /etc/ttys files. Therefore, set the Soekris console speed to 9600 in the BIOS by hitting control-p during the Soekris' initial boot stage.

To show the current BIOS settings:

	comBIOS monitor.   Press ? for help.

	> show

Set the console speed to 9600.

	> set conspeed=9600

Begin booting OpenBSD with the new settings.

	> boot

All done!
