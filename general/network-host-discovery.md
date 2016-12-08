##Using nmap to Learn About a Local Network##

This brief guide is meant as a simple reference for anyone comfortable in a Unix shell ("the command-line") to gain a basic understanding of a local network using [nmap](https://nmap.org/). It is not meant to be comprehensive, but rather as a starting point for users unfamiliar with such a task or with nmap.

nmap maintains a long list of options, and this short guide is by no mean exhaustive, nor even necessarily the best options.

The nmap book [page](https://nmap.org/book/man-host-discovery.html) on "Host Discovery" may be useful.

In these examples, the target network is listed as:

	192.168.1.0/24

...which translates to all the hosts on the 192.168.1.x network, which can mean up to 254 computers and devices.

This can be changed to any network IP address block, but can also be a single host, such as 192.168.1.1.

Shell commands are indented once, preceded by "$".

__List the hosts on a network__

To quickly show all the hosts on a network and their IP addresses, plus "latency" to the host:

	$ nmap -sn 192.168.1.0/24

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

__Getting MAC Addresses with the Same Command__

Running the above command with root privileges (or sudo) will also provide the network interface (wired or wireless) hard-coded MAC address. Additionally, it provides the suspected network interface manufacturer, based on the [organizationally unique identifier](http://standards-oui.ieee.org/oui.txt).

	$ sudo nmap -sn 192.168.1.0/24

Output will list each host, and provide a summary of the scanned hosts:

MAC Address: 00:0B:00:41:AB:4B (Fujian Start Computer Equipment)
Nmap scan report for 192.168.1.100
Host is up.
Nmap done: 256 IP addresses (7 hosts up) scanned in 1.34 seconds

Note that MAC addresses are only discoverable on a local network, and not over the internet.

__Do a more comprehensive scan of a particular host__

Outputs to the above commands produces a list that can be compared to existing inventory lists, or even a quick eyeball survey on smaller local networks. But what if a host can't be identified? A more in-depth scan might be warranted:

	$ nmap -A -T4 192.168.1.1

Output will include services listening on TCP ports 1000.

TCP ports often follow the standards for assigned services. For instance, an SMTP server normally resides on TCP port 25, while SSH is on TCP port 22. The complete list of officially assigned ports for TCP and other protocols is maintained by the standards organization the [Internet Assigned Numbers Authority](https://www.iana.org/). The [full list](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.txt) can be useful when figuring out what service a particular open port is providing.

To check the range of all 65535 ports, and not just 1-1000, use the "-p" option.

	$ nmap -A -T4 -p1-65535 192.168.1.1

Any range can be specified. "-p" can also be used to target individual ports to scan, such viewing TCP port 22, which is usually SSH.

	$ nmap -A -T4 -p22 192.168.1.1

The "-T4" option will speed up scans considerably.

To do the same, while also identifying the operating systems and their versions:

	$ nmap -sS -A 192.168.1.0/24

With every nmap command, adding "-v" will increase verbosity, which means additional details that may or may not be useful. If a particular command doesn't provide enough information, it may be worth running again with "-v".

To output the results to a text file, add "-oN output.txt" to the options. For those looking to diminish their vision, "-oX output.xml" will output to the XML format.

__Automating Daily nmap Scans and the ndiff Command__

On local networks where regularly monitoring of hosts is a necessarity, it may be worth automating regular scans such as daily or even hourly for unauthorized hosts.

nmap includes the ndiff tool, which makes comparing nmap output easier, allowing changes to be displayed once a local network's baseline hosts are established.

The [nmap Book](https://nmap.org/book/ndiff-man-periodic.html) provides a simple shell script to run out of the Unix cron daemon.

This is a customization of that shell script, and it should run on any Unix or Unix like system. The four variables can be edited as per particular needs or the system the script is running from.

_$targets_ refers to the nmap scan targets, and can be edited to scan individual hosts {192.168.1.1} or a network {192.168.1.0/24}.

_$options_ specifies the nmap options.

_$now_ refers to the time appended to the output file, reflecting the frequency that a scan is run. For daily scans, _$now_ might be set to "`date +%Y%m%d`" while hourly scans might use "`date +%Y%m%d-%H`" with %H indicating the hour.

_$nmap_ is just the location of the executible nmap application, which may vary per operating system.

The result to check is the diff-date file, which will include changes from the previous scan.

```
#!/bin/sh -x

# variables
targets="${targets:-{192.168.27.1,192.168.27.50}}"
options="${options:-"-v -T4 -F -sV"}"
now="${now:-"`date +%Y%m%d-%M`"}"
nmap="${nmap:-/usr/local/bin/nmap}"

# keep the scan outputs in your home directory, edit
cd /home/user/scans;

$nmap $options $targets -oN scan-$now.txt > /dev/null;

if [ -e scan-prev.txt ]; then
        ndiff scan-prev.txt scan-$now.txt > diff-$now
        echo "*** NDIFF RESULTS ***"
        cat diff-$now
        echo
fi

echo "*** NMAP RESULTS ***"

cat scan-$now.txt;

ln -sf scan-$now.txt scan-prev.txt
```

__Other Online nmap Resources__

Besides the [nmap book](https://nmap.org/book/), half of which is [free online](https://nmap.org/book/toc.html), there are a number of nmap "cheat sheets" available online which are easy enough to find.
