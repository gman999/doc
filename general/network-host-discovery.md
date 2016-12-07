This brief guide is meant as a simple reference for anyone comfortable in a Unix shell ("the command-line") to gain a basic understanding of a local network using [nmap](https://nmap.org/). It is not meant to be comprehensive, but rather as a starting point for users unfamiliar with such a task or with nmap.

The nmap book page on it

https://nmap.org/book/man-host-discovery.html

In these examples, the target network is listed as:

	192.168.1.0/24

which translates to all the hosts on the 192.168.1.x network, which can mean up to 256 computers and devices.

This can be changed to any network IP address block, but can also be a single host, such as 192.168.1.1.

__List the hosts on a network__

To quickly show all the hosts on a network and their IP addresses, plus "latency" to the host:

	$ nmap -sP -A 192.168.1.0/24

Output for one host might look like this:

Nmap scan report for firewall (192.168.1.1)
Host is up (0.0020s latency).

Nmap scan report for 192.168.1.50
Host is up (0.00060s latency).

Nmap scan report for 192.168.1.100
Host is up (0.0021s latency).

Nmap scan report for 192.168.1.101
Host is up (0.0036s latency).

Higher latency numbers might indicate network issues, including misconfiguration of network interfaces or network congestion.

__Do a more comprehensive scan of a particular host__

	$ nmap -A -T4 192.168.1.1

Output will include services listening on TCP ports 1000.

MAC addresses can only be discovered on a local network, not remotely over the internet. This command must be run with root privileges, such as with sudo. To find all the MAC addresses on a local network:

	$ nmap -sP 192.168.1.0/24

The output will include not just the MAC and IP addresses, but also the suspected network interface manufacturer, based on the [organizationally unique identifier](http://standards-oui.ieee.org/oui.txt).

To do the same, while also identifying the operating systems and their versions:

	$ nmap -sS -n -A 192.168.1.0/24

With every nmap command, adding -v will increase verbosity, which means additional details that may or may not be useful.
