##Soekris 4801: Continuing Relevance After All These Years##

The venerable [Soekris 4801](https://soekris.com) with its 128 megabytes of RAM and slow 233 MHz processor can still maintain a useful role. Thousands reside quietly in drawers and closets around the globe, imagined to be far past their expiration date.

OpenBSD is an ideal candidate operating system for the Soekris, since it's built lean by default, yet an array of functions are available in the base system.

Such a device could be a serial console server, a recursive/caching DNS, a lightweight monitoring server, etc., without installing any third-party software.

###The Install###

Needs:

* bootable media with OpenBSD/i386 stable version on a [USB stick](http://www.openbsd.org/faq/faq4.html#Flash). Using snapshots is not ideal in this scenario.

* compact flash (CF) card with adapter

* bootstrap i386/amd64 computer with any operating system

* a serial console cable with a null modem

With the bootable media attached via USB, launch the bootstrap computer and enter the OpenBSD installation program.

The OpenBSD installer is graphics-free, yet clear and simple to navigate.

Type __i__ to enter the install system.

	Welcome to the OpenBSD/i386 5.8 installation program.
	(I)nstall, (U)pgrade, (A)utoinstall or (S)hell? i

Plug in the CF card adapter and note the device listed to standard output, normally sd(4) with a respective number, such as sd2.

Hit "enter" for [default]

	Choose your keyboard layout ('?' or 'L' for list) [default]

Designate the system's hostname

	System hostname? (short form, e.g. 'foo')

Next the list of interfaces is displayed, both wired and wireless devices. This can be configured for DHCP if available, as it will only be used to pull the relevant install sets from a remote OpenBSD mirror. Note that the interfaces only apply temporarily to the bootstrap computer, and will need to be adjusted manually on the actual Soekris device which uses sis(4) for the network interfaces.

	Password for root account? (will not echo)

sshd(8) will likely be necessary.

	Start sshd(8) by default [yes]? yes

For the functions listed previously, X Windows and xdm(1) are unnecessary.

	Do you want the X Window System to be started by xdm(1)? [no] no

Configuring a non-privileged user is an important step in running a secure system.

	Setup a user? (enter a lower-case loginname, or 'no') [no] user-name

And set the password.

	Allow root ssh login? (yes, no, prohibit-password) [no] no

	What timezone are you in? ('?' for list) [(local timezone guess)]

Next determine the disk to which OpenBSD is installed.

	Available disks are: sd0 sd1 sd2.

In this case, standard output noted that the CF card is located on sd2. An incorrect choice here, such as selecting the bootstrap computer's disk, will mean overwriting the wrong disk.

Type __?__ to verify the correct disk.

	Which disk is the root disk? ('?' for details) [sd0] ?

Enter the appropriate disk, in this case it's sd2.

Next the fdisk(8) utility runs displaying the disk's slices.

Choose __w__ to utilize the entire disk.

	Use (W)hole disk, use the (O)penBSD area, or (E)dit the MBR? [OpenBSD] w

Next the autoconfigured partition layout will be displayed. Unless there are particular needs, choose __a__ for auto.

	Use (A)uto layout, (E)dit auto layout, or create (C)ustom layout [a]? a

At this point, the installer will ask for the next disk to configure. Type __done__ as our disk setup is complete.

	Which disk do you wish to initialize? (or 'done') [done] done

The install sets now need to be accessed, either on a local disk or via HTTP. __http__ is our choice, as the install media didn't contain the files, and the install sets need to be pulled from a remote OpenBSD mirror.

	Let's install the sets!

	Location of sets? (disk http or 'done') [http] http

If you're using a proxy, enter it here.

	HTTP proxy URL? (e.g. 'http://proxy:8080', or 'none') [none]

Now enter an OpenBSD mirror. Typing __?__ will display a list of the current mirrors.

	HTTP Server? (hostname, list#, 'done' or '?') your-mirror-selection

The OpenBSD project insists on maintaining identical directory hierarchies on their mirrors. The next prompt should illustrate why, since you just need to hit "enter."

	Server directory? [pub/OpenBSD/5.8/i386]

Now the full list of sets is displayed. In our case, the Soekris install should be painfully small, and not contain any unnecessary parts of the base system.

	Set name(s)? (or 'abort' or 'done') [done]

To remove install sets, type __-__ before the ones you want to remove, including the SMP kernel, since Soekris has an ancient CPU. Useless uses of __*__ are acceptable.

	-bsd.mp

	-comp*

	-man*

	-game*

	-x*

At this point there should only be three install components remaining: the kernel bsd, the RAM kernel bsd.rd and the vital base58.tgz.

	Location of sets? (disk http or 'done') [http] done

We are quite sure the bsd.mp kernel is unnecessary, so type __yes__.

	Are you *SURE* your install is complete without 'bsd.mp'? [no] yes

The install process will complete and return to a prompt, from which we'll reboot.

	# reboot

As the system reboots remove both the USB disk with the install media and the CF card adapter.

Ideally, the bootstrap computer will boot normally, and you didn't do a fresh OpenBSD install to the wrong disk. Measure twice, cut once!

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

Install the CF card into the Soekris, plug in an ethernet cable to the first port on the right (sis0) and plug in the serial console cable.

###Booting Up###

Setup the bootstrap computer with the serial console cable and some terminal emulation application.

By default, the Soekris 4801 uses 19200 as the console speed, yet we set the speed to 9600 in the /etc/boot.conf and /etc/ttys files. In this case, we'll set the Soekris console speed to 9600 in the BIOS.

From an OpenBSD shell and the serial console cable connected, use cu(1) to access the Soekris system.

	cu -l /dev/ttyU0 -s 19200

<<<<<<< HEAD
I recommend setting every system to 9600 since it's the default for programs such as cu(1), it's unnecessary to have speeds exceeding 9600 for any console access. Don't confuse console speed with broadband connectivity.

To show the current BIOS settings:
=======
Plug in the Soekris power, and hit control-p when prompted, and display the current BIOS settings.
>>>>>>> afa4da533e188b4b580d6e265f8e53433c7fc72f

	comBIOS monitor.   Press ? for help.

	> show

Set the console speed to 9600.

	> set conspeed=9600

Type __show__ to confirm the new setting.

	> show

Begin booting OpenBSD.

	> boot

Finally, it's necessary to configure the sis(4) network interface as directed by hostname.if(5).

	vi /etc/hostname.sis0

Either set a static address with:

	inet {IP} netmask {netmask}

And also the appropriate gateway in /etc/mygate and DNS in /etc/resolv.conf.

Or set to dhcp:

	dhcp

Reboot and the Soekris is ready for the particular function you've chosen.
