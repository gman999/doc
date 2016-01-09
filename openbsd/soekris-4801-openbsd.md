##Soekris 4801: Relevant After All These Years##

The venerable [Soekris 4801](https://soekris.com) with 128 megabytes of RAM and slow 233 MHz processor can still maintain a useful role. Thousands reside quietly in drawers and closets around the globe, forgotten as expired goods past their relevance.

Even an ancient Soekris box running OpenBSD is more than adequate for use as a serial console server, a recursive/caching DNS, a lightweight monitoring server, etc.

OpenBSD is an ideal candidate operating system for the Soekris, since it's built lean by default, yet an array of functions are available in the base system.

One might also consider custom OpenBSD systems such as [flashrd](http://www.nmedia.net/flashrd) or its derivation [resflash](http://stable.rcesoftware.com/resflash/). Both are aimed at small systems, and minimize disk writes which is ideal for flash media-based systems. However, flashrd hasn't been updated since OpenBSD 5.7 and resflash is in its early stages of development.

In terms of formatting with this piece:

* __bold__ indicates user input

* indented and shadowed text indicates standard output as it's displayed

* a number after a command, such as sshd(8), indicates an OpenBSD manual page

* and { } represents custom user input, such as a user name

###The Install###

Needs:

* bootable media with OpenBSD/i386 [stable](http://www.openbsd.org/faq/faq5.html#Flavors) on a [USB stick](http://www.openbsd.org/faq/faq4.html#Flash). Using current (snapshots) is not ideal in this scenario, as we're looking for a low-maintenance system.

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

	System hostname? (short form, e.g. 'foo') {hostname}

Next the list of interfaces is displayed, both wired and wireless devices. This can be configured for DHCP if available, as it will only be used to pull the relevant install sets from a remote OpenBSD mirror. Note that the interfaces only apply temporarily to the bootstrap computer, and will need to be adjusted manually on the actual Soekris device which uses sis(4) for the network interfaces.

	Password for root account? (will not echo)

sshd(8) will likely be necessary.

	Start sshd(8) by default [yes]? yes

Since we are looking for a small, tight system, X Windows and xdm(1) are unnecessary.

	Do you want the X Window System to be started by xdm(1)? [no] no

Configuring a non-privileged user is an important step in running a secure system.

	Setup a user? (enter a lower-case loginname, or 'no') [no] {user name}

And set the password.

	Allow root ssh login? (yes, no, prohibit-password) [no] no

	Available disks are: sd0 sd1 sd2.

An incorrect choice here, such as selecting the bootstrap computer's disk, will mean overwriting the wrong disk.

Type __?__ to verify the correct disk.

	Which disk is the root disk? ('?' for details) [sd0] ?

Enter the appropriate disk, in this case it's sd2.

Next the fdisk(8) utility runs displaying the disk's slices.

Choose __w__ to utilize the entire disk.

	Use (W)hole disk, use the (O)penBSD area, or (E)dit the MBR? [OpenBSD] w

Next the autoconfigured partition layout will be displayed. How the disk is configured depends on a few factors, including the CF card size and the function of this system. For an 8Gb CF card, it might make sense to create a 1G /, 300M swap, plus /usr and /home partitions. While swap should generally be avoided on media such as CF cards, it could be vital for a system with only 128Mb of RAM. Note that a /tmp partition should be configured, but in this case a tmpfs mount will be added after the initial boot.

	Use (A)uto layout, (E)dit auto layout, or create (C)ustom layout [a]? c

At this point, the installer will ask for the next disk to configure. Type __done__ as our disk setup is complete.

	Which disk do you wish to initialize? (or 'done') [done] done

The install sets now need to be accessed, either on a local disk or via HTTP. __http__ is our choice, as the install media didn't contain the files, and the install sets need to be pulled from a remote OpenBSD mirror.

	Let's install the sets!

	Location of sets? (disk http or 'done') [http] http

If you're using a proxy, enter it here.

	HTTP proxy URL? (e.g. 'http://proxy:8080', or 'none') [none]

Now enter an OpenBSD mirror. Typing __?__ will display a list of the current mirrors.

	HTTP Server? (hostname, list#, 'done' or '?') {your-mirror-selection}

The OpenBSD project insists on maintaining identical directory hierarchies on their mirrors. The next prompt illustrates why, since you just need to hit "enter."

	Server directory? [pub/OpenBSD/5.8/i386]

The full list of sets is then displayed. In our case, the Soekris install should be painfully small, and not contain any unnecessary parts of the base system.

	Set name(s)? (or 'abort' or 'done') [done]

To remove install sets, type __-__ before the ones you want to remove, including the SMP kernel, since Soekris has an ancient CPU. Useless uses of __*__ are acceptable.

	-bsd.mp

	-comp*

	-man*

	-game*

	-x*

At this point there should only be three install components remaining: the kernel bsd, the RAM kernel bsd.rd and the vital base58.tgz.

	Location of sets? (disk http or 'done') [http] {done}

We are sure the bsd.mp kernel is unnecessary, so type __yes__.

	Are you *SURE* your install is complete without 'bsd.mp'? [no] yes

The install process will complete and return to a prompt from which we'll reboot.

	# reboot

As the system reboots, disconnect both the USB disk with the install media and the CF card adapter.

Ideally, the bootstrap computer will boot normally, and you didn't do a fresh OpenBSD install to the wrong disk. Measure twice, cut once!

###Pre-First Boot Configuration###

There is an important set of changes to make to boot the Soekris board with OpenBSD as there is no VGA output, and the default output is to console.

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

OpenBSD [removed sudo](http://www.openbsd.org/faq/faq10.html#doas) from its base, and replaced it with doas(1). Configuration is simple. Setup the non-privileged user with doas(1) permissions in /etc/doas.conf(5):

	permit nopass keepenv {user name} as root

Create /etc/rc.conf.local for minimizing unnecessary daemons, such as sndiod(1) and smtpd(8), and forcing ntpd(8) to synchronize on boot:

	sndiod_flags=NO
	smtpd_flags=NO
	ntpd_flags="-s"

Install the CF card into the Soekris, plug in an ethernet cable to the first port on the right (sis0) and plug in the serial console cable.

Depending on the CF card model, there might be various errors connected to mounting wd(4), which refers to the CF card, due to DMA mode errors. The system may or may not still boot, but there will at least be a delay as the kernel adjusts.

This is a known issue with legacy hardware and UDMA. The first step for a (one-time) successful boot is to disable UDMA and use PIO mode.

At the boot prompt:

	boot> boot hd0a:/bsd -c

Which delivers you into the UKC(8), the User Kernel Config, in which we'll just type __change wd__, set the __0x0ff0__ for wd(4) devices and __quit__:

	UKC> change wd
	 53 wd* at wdc0|wdc1|wdc*|wdc*|pciide*|pciide* channel -1 flags 0x0
	change (y/n) ?
	channel [-1] ? 
	flags [0] ? 0x0ff0
	 53 wd* changed
	 53 wd* at wdc0|wdc1|wdc*|wdc*|pciide*|pciide* channel -1 flags 0xff0
	UKC> quit

The [OpenBSD FAQ](http://www.openbsd.org/faq/faq14.html#pciideErr) documents this issue with legacy hardware. The change to wd(4) can be made permanent with config(8) in multi-user mode.

###Booting Up###

By default, the Soekris 4801 uses 19200 as the console speed, yet we set the speed to 9600 in the /etc/boot.conf and /etc/ttys files. In this case, we'll set the Soekris console speed to 9600 in the BIOS.

From an OpenBSD shell and the serial console cable connected, use cu(1) to access the Soekris system.

	cu -l /dev/ttyU0 -s 19200

Setting systems to 9600 makes sense since it's the default for programs such as cu(1), and it's rarely necessary to have speeds exceeding 9600 for console access.

Plug in the Soekris power, and hit control-p when prompted, and display the current BIOS settings.

	comBIOS monitor.   Press ? for help.

	> show

Set the console speed to 9600.

	> set conspeed=9600

Type __show__ to confirm the new setting.

	> show

Begin booting OpenBSD.

	> boot

For more details about Soekris' BIOS, the [Soekris Wiki](http://wiki.soekris.info/What_do_all_those_BIOS_settings_do%3F) provides some insight.

At this point, close the current cu(1) session with __~ .__ and reconnnect with:

	cu -l /dev/ttyU0

Finally, it's necessary to configure the sis(4) network interface as directed by hostname.if(5).

	vi /etc/hostname.sis0

Either set a static address with:

	inet {IP} {netmask}

And add the appropriate gateway in /etc/mygate and DNS in /etc/resolv.conf.

Or set to dhcp:

	dhcp

As noted earlier, the /tmp partition will be mounted as a tmpfs device, and not use the CF card. In this case, a small 10Mb /tmp partition should be sufficient.

	echo "swap /tmp tmpfs rw,-s10m,nodev,nosuid,noexec 0 0" >>/etc/fstab

Reboot and the Soekris is ready for the particular function you've chosen.

To confirm the disk layout, use mount(8):

$ mount

	/dev/wd0a on / type ffs (local)
	/dev/wd0e on /home type ffs (local, nodev, nosuid)
	/dev/wd0d on /usr type ffs (local, nodev)
	tmpfs on /tmp type tmpfs (local, nodev, noexec, nosuid)

###Finally###

The Soekris running OpenBSD should be ready for any number of simple functions at this point, with the obvious restraints one would expect from a device from a low-powered legacy device.
