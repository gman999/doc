Soekris 4801: Continuing Relevance After All These Years

The venerable [Soekris 4801](https://soekris.com) with its 128 megabyte of RAM and slow 233 MHz processor can still be of utility. Thousands reside quietly in drawers and closets around the globe, imagined to be far past their expiration date.

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

Welcome to the OpenBSD/i386 5.8 installation program.
(I)nstall, (U)pgrade, (A)utoinstall or (S)hell?

Type I to enter the install screen.

Plug in the CF card adapter and note the device listed to standard output, normally sd(4) with a respective number, such as sd2.

The OpenBSD installation program is graphics-free, yet clear and simple to follow.

Choose your keyboard layout ('?' or 'L' for list) [default]
hit "enter"

System hostname? (short form, e.g. 'foo')
choose a hostname

Next the list of interfaces will be listed, both wired and the recognized wireless devices. This can be configured for DHCP, as it will only be used to pull the relevant install sets from an OpenBSD mirror. Note that the interfaces only apply temporarily to the bootstrap computer, and will need to be adjusted manually on the actual Soekris device which uses sis(4) for the network interfaces.

Password for root account? (will not echo)

Start sshd(8) by default [yes]?  
sshd will likely be a necessary function

Do you want the X Window System to be started by xdm(1)? [no]  
For the functions listed previously, X Windows and xdm(1) are unnecessary.

Setup a user? (enter a lower-case loginname, or 'no') [no]  
A non-privileged user is an important step in running a secure system.

Allow root ssh login? (yes, no, prohibit-password) [no]
sshd(8) access should only be done with non-privileged users, and not the root user.

What timezone are you in? ('?' for list) [(local timezone guess)]

Next is determining the install disk.

Available disks are: sd0 sd1 sd2.

In this case, standard output noted that the CF card is located on sd2. An incorrect choice here, such as selecting the bootstrap computer's disk, will mean overwriting the wrong disk.

Which disk is the root disk? ('?' for details) [sd0] ?

Select ? to verify the correct disk.



The default install will cease booting with this line unless console access is enabled.

entry point at 0x200120


In /etc/ttys, change this line:

	tty00	"/usr/libexec/getty std.9600"	unknown	off

to this:

	tty00	"/usr/libexec/getty std.9600"	vt220	on secure

Create /etc/boot.conf, and add the following line:

	set tty com0

###Final Configuration###

Setup the non-privileged user with doas(1) permissions in /etc/doas.conf(5):

	permit nopass keepenv $user as root

Create /etc/rc.conf.local for minimizing unnecessary daemons, such as sndiod(1) and smtpd(8):

	sndiod_flags=NO
	smtpd_flags=NO

###Booting Up###


