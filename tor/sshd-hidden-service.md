##Tor SSHD Hidden Service as a Remedy for Dynamic IPs##

Free dynamic IP services have evolved into a sloppy system requiring regular attention while inconsistently delivering their commodity.

I access a number of hosts utilizing dynamic IPs, and have fought through a number of services over the years. Some have evolved into a fee-based model, while others are now just inoperable. Even worse, I suspect that many now consider themselves purely data mining services, not just for the hosts they provide services for, but even for logging in. Well, maybe not on purpose, but once I see domains such as [these](http://queair.net/files/blocklist.html) showing up on a login page, the hair raises on the back of my neck.

One solution is SSHD as a Tor hidden service.

A Tor hidden service hostname is a long, somewhat ugly hash that can only be accessed over Tor. It may fail the [Zooko's Triangle](https://en.wikipedia.org/wiki/Zooko%27s_triangle) "human meaningful" criteria since the long hashes as hostnames are difficult to remember as opposed to normal FQDNs or even IPv4 addresses, but you can't have everything. It requires Tor to be running on both the local host, for torsocks, and on the destination host.

Other problems arise with the possibility that Tor crashes, say, on small embedded boxes. Some boxes I access have only 256M or even just 128M of RAM, and Tor isn't meant to be stable on that latter. It is unimaginable to have a phone with even 128M of RAM today, so asking a host with less to run SSHD and Tor is a bit presumptuous.

On the other hand, setup is easy.  In some cases, I do occasionally access those hosts on the local network, so SSHD needs to listen on both the local IPv4 network, in addition to as a Tor hidden service.

Here's the simple setup.

In the section of the torrc file, subtitled with:

############### This section is just for location-hidden services ###

Add the following lines to have a hidden service listening the localhost's tcp/22:

HiddenServiceDir /var/db/tor/sshd
HiddenServicePort 22 127.0.0.1:22

The first line is the Tor data directory, which is the location of hidden service's private key, in addition to its hostname. We created an __sshd__ directory for this particular service, but any standard Unix directory name is fine.

Next, we'll need to edit the appropriate /etc/ssh/sshd_config lines. The first entry will allow SSHD to listen normally on the local network:

ListenAddress 192.168.0.2:2222

That means SSHD will listen on tcp/2222 on the local network for SSH connections.

After that line, add another for SSHD as a Tor hidden service:

ListenAddress localhost:22

Now restart Tor, then restart SSHD in whichever way your operating system requires.

	/etc/rc.d/sshd restart

Remotely accessing SSHD as a hidden service, as noted, requires Tor and its step-child torsocks.

For instance, one might run:

$ torsocks ssh ugly-long-hash.onion

Note we're relying on either/both the host and network based firewalls to keep access to the 192.168 address solely on the local network. Although without some type of forwarding on the NAT device, it's not accessible anyway.
