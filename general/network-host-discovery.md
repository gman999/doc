
The nmap book page on it

https://nmap.org/book/man-host-discovery.html

In these examples, the target network is listed as:

	192.168.1.0/24

which translates to all the hosts on the 192.168.1.x network, which can mean up to 256 computers and devices.

This can be changed to any network IP address block, but can also be a single host, such as 192.168.1.1.

To quickly show all the hosts on a network and their IP addresses:

	$ nmap -Sp -A 192.168.1.0/24

MAC addresses can only be discovered on a local network, not remotely over the internet. To find all the MAC addresses on a local network:

	$ nmap -sP 192.168.1.0/24


To do the same, while also identifying the operating systems and their versions:

	$ nmap -sP -n -A 192.168.1.0/24
