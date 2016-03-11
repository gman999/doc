##Tor SSHD(8) Hidden Service as a Remedy for Dynamic IPs##

Free dynamic IP services have evolved into a sloppy system requiring regular attention while inconsistently delivering the respective service.

I access a number of hosts utilizing dynamic IPs, and have fought through a number of services over the years. Some have evolved into a fee-based model, while others are now just inoperable. Even worse, I suspect that many now consider themselves purely data mining services, not just for the hosts they provide services for, but even for logging in. Well, maybe not on purpose, but once I see domains such as [these](http://queair.net/files/blocklist.txt) appearing as a login page loads in the browser, the hair raises on the back of my neck.

One solution is to run sshd(8) as a Tor hidden service. A Tor hidden service providees a number of benefits, but in this case it allows for a static hostname regardless of the host's IP address.

A Tor hidden service hostname is a long, somewhat ugly hash that can only be accessed over Tor. It may fail the [Zooko's Triangle](https://en.wikipedia.org/wiki/Zooko%27s_triangle) "human meaningful" criteria since the long hashes as hostnames are difficult to remember as opposed to normal FQDNs or even IPv4 addresses, but you can't have everything. It requires Tor to be running on both the source host that is using ssh(1) and the destination host with the dynamic IP. torsocks is also required to use ssh(1) over Tor on the source host.

source-->(Tor network)-->destination with dynamic IP

The "source" is using ssh(1) over torsocks(1) 
The "destination" is sshd(8) listening over the Tor network as a hidden service

Other potential problems arise with the possibility that Tor crashes on the destination host, say, on small embedded boxes. Some boxes I access have only 256M or even just 128M of RAM, and Tor isn't meant to be stable on that latter. It is unimaginable to have a phone with even 128M of RAM today, so asking a host with less to run sshd(8) and Tor is a bit presumptuous.

On the other hand, setup is easy.  In some cases, I do occasionally access those hosts on the local network, so sshd(8) needs to listen on both the local RFC1918 IPv4 network, in addition to on the public internet as a Tor hidden service.

Here's the simple setup.

On the public internet, sshd(8) will listen only on the Tor network with port 22/tcp. On the local network, Tor will listen on port 2222/tcp, as a normal daemon.

In the section of the torrc file, subtitled with:

############### This section is just for location-hidden services ###

Add the following lines to have a hidden service listening the localhost's tcp/22 on FreeBSD:

	HiddenServiceDir /var/db/tor/sshd

	HiddenServicePort 22 127.0.0.1:22

For OpenBSD, the Tor data directory is located in /var/tor, so the first line would be:

	HiddenServiceDir /var/tor/sshd

The first line is the Tor data directory, which is the location of hidden service's private key, in addition to its hashed Tor hostname. We created an __sshd__ directory for this particular service, but any standard Unix directory name is fine.

Next, we'll need to edit the appropriate /etc/ssh/sshd_config(5) lines. The first entry will allow sshd(8) to listen normally on the local network:

	ListenAddress 192.168.0.2:2222

That means sshd(8) will listen on tcp/2222 on the local network for SSH connections.

After that line, add another for sshd(8) as a Tor hidden service:

	ListenAddress localhost:22

Now restart Tor, then restart sshd(8) in whichever way your operating system requires.

	$ /etc/rc.d/sshd restart

Remotely accessing sshd(8) as a hidden service, as noted, requires both Tor and torsocks.

For instance, one might run:

	$ torsocks ssh ugly-long-hash.onion

Note we're relying on either/both the host and network based firewalls to keep access to the 192.168 address solely on the local network. Although without some type of forwarding on the NAT device, it's not accessible anyway.
