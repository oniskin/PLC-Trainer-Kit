# Introduction

Suricata was cool. It's a great tool to rapidly alert on known-bad network behavior.

To be honest, it's great to have, but the bad-guys will get past that pretty quickly. They can practice their attacks against Snort / Suricata publicly available rules (which get used by all our favorite vendors, too!)

We need something deeper to complement Suricata. Something that tracks and correlates all this network data. Something that can help spot complex attacks and strange behavior over a longer time period.

Sound like fun?

**Let's do MORE network detection! With Zeek!**

*Let's Go!*

# First, Let's get Zeek running

### Install Zeek.

First, you'll need an Ubuntu virtual machine.
I'm using the same virtual machine from the [Suricata install](../7-Suricata-Network-Detection/README.md) we just did.

Next, install Zeek  following the Documentation [Installing Zeek](https://docs.zeek.org/en/lts/install.html).

This tutorial needed a *little* finessing.

At the time of writing, the commands were as below (look at documentation. They could've changed.)

```
echo 'deb https://download.opensuse.org/repositories/security:/zeek/xUbuntu_24.04/ /' | sudo tee /etc/apt/sources.list.d/security:zeek.list
curl -fsSL https://download.opensuse.org/repositories/security:zeek/xUbuntu_24.04/Release.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/security_zeek.gpg > /dev/null
sudo apt update
sudo apt install zeek
```

This will add the zeek packages and encryption keys to our Ubuntu system and install the latest version available.

Next, you'll probably want to add Zeek to your PATH so you can just type `zeek` instead of `really/long/directory/zeek` every time.

```
echo 'export PATH="$PATH:/opt/zeek/bin"' >> ~/.bashrc
source ~/.bashrc
```

Since you usually run zeek with `sudo`, you may also want to add zeek to your sudo PATH.

```
sudo visudo
```
this will open up Nano text editer that will safely edit the `/etc/sudoers` file. Whatever that means! :D

find the Defaults `secure_path=` line and append `:/opt/zeek/bin`

### Testing Zeek with a pcap file

Instructions and pcap file for download are in the [Quick Start Guide](https://docs.zeek.org/en/lts/quickstart.html).

Make a directory in your home directory to store the logs.

```
mkdir zeek-test
mv quickstart.pcap zeek-test/
cd zeek-test
zeek -r quickstart.pcap LogAscii::use_json=T
ls
```

You should see a bunch of logs. Follow the quick start guide.

# Configure Zeek to listen to LIVE network traffic

Like in the [Suricata install](../7-Suricata-Network-Detection/README.md), we will configure Zeek to listen to the proper interface.

```
analyst@nsm:~$ ip a
<--Snip-->
3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
    altname enp2s5
    inet 192.168.0.254/24 brd 192.168.0.255 scope global noprefixroute ens37
<--Snip-->
```

Open the config file.
```
sudo vim /opt/zeek/etc/node.cfg
```

Then go to and change the default value to the name of the network interface Zeek should be listening to.

```
# Example ZeekControl node configuration.
#
# This example has a standalone node ready to go except for possibly changing
# the sniffing interface.

# This is a complete standalone configuration.  Most likely you will
# only need to change the interface.
[zeek]
type=standalone
host=localhost
interface=eth0    # <- Change to your interface name (mine is ens37)
```

### Running Zeek!

Start `ZeekControl` and run `deploy` to start the zeek service.
```
analyst@nsm:~/zeek$ sudo zeekctl
Warning: zeekctl option UseWebSocket is set, but websockets non-functional: Failed to import websockets module (ModuleNotFoundError("No module named 'websockets'"))
Warning: The print and netstats commands will be non-functional, please install the Python websockets package version 11.0 or later.

Welcome to ZeekControl 2.6.0-56

Type "help" for help.

[ZeekControl] > deploy
checking configurations ...
installing ...
removing old policies in /opt/zeek/spool/installed-scripts-do-not-touch/site ...
removing old policies in /opt/zeek/spool/installed-scripts-do-not-touch/auto ...
creating policy directories ...
installing site policies ...
generating standalone-layout.zeek ...
generating local-networks.zeek ...
generating zeekctl-config.zeek ...
generating zeekctl-config.sh ...
stopping ...
stopping zeek ...
starting ...
starting zeek ...

# start Ignition.
# start and stop your fan over the network a few times.
# Do some network activity!
# when you've done some test actions, stop zeek here

[ZeekControl] > stop
stopping zeek ...
[ZeekControl] > 

```

### Reviewing the Zeek logs

If Zeek is still running, your logs will be in `/opt/zeek/logs/current`.

If you've stopped Zeek, your logs will be in `/opt/zeek/logs/<date-string>/`.

You can read the logs with a gzip utility called `zcat` as shown in the Quick Start Guide.

Check out the Modbus logs!! Pretty awesome, right?

```
analyst@nsm:~/zeek$ sudo zcat /opt/zeek/logs/2026-05-23/modbus.11:23:23-11:24:29.log.gz
#separator \x09
#set_separator	,
#empty_field	(empty)
#unset_field	-
#path	modbus
#open	2026-05-23-11-23-23
#fields	ts	uid	id.orig_h	id.orig_p	id.resp_h	id.resp_p	tid	unit	func	pdu_type	exception
#types	time	string	addr	port	addr	port	count	count	string	string	string
1779553403.588504	CbZ28x2qJnDfXoiHL5	192.168.0.3	49339	192.168.0.10	502	909	0	READ_COILS	REQ	-
1779553403.589262	CbZ28x2qJnDfXoiHL5	192.168.0.3	49339	192.168.0.10	502	909	0	READ_COILS	RESP	-
1779553403.590238	CbZ28x2qJnDfXoiHL5	192.168.0.3	49339	192.168.0.10	502	910	0	READ_DISCRETE_INPUTS	REQ	-
1779553403.590952	CbZ28x2qJnDfXoiHL5	192.168.0.3	49339	192.168.0.10	502	910	0	READ_DISCRETE_INPUTS	RESP	-
1779553404.588987	CbZ28x2qJnDfXoiHL5	192.168.0.3	49339	192.168.0.10	502	911	0	READ_COILS	REQ	-
1779553404.589517	CbZ28x2qJnDfXoiHL5	192.168.0.3	49339	192.168.0.10	502	911	0	READ_COILS	RESP	-
1779553404.590709	CbZ28x2qJnDfXoiHL5	192.168.0.3	49339	192.168.0.10	502	912	0	READ_DISCRETE_INPUTS	REQ	-
1779553404.591362	CbZ28x2qJnDfXoiHL5	192.168.0.3	49339	192.168.0.10	502	912	0	READ_DISCRETE_INPUTS	RESP	-
1779553405.588188	CbZ28x2qJnDfXoiHL5	192.168.0.3	49339	192.168.0.10	502	913	0	READ_DISCRETE_INPUTS	REQ	-
1779553405.588462	CbZ28x2qJnDfXoiHL5	192.168.0.3	49339	192.168.0.10	502	913	0	READ_DISCRETE_INPUTS	RESP	-
```

If that's hard to read, we can only select certain columns with the `zeekcut` utility. Critical to keep your sanity!

In this example, I used `zeekcut` to only show origin IP, destination IP, and the Modbus function code.

```
analyst@nsm:~/zeek$ sudo zcat /opt/zeek/logs/2026-05-23/modbus.11:23:23-11:24:29.log.gz | zeek-cut id.orig_h id.resp_h func
192.168.0.3	192.168.0.10	READ_COILS
192.168.0.3	192.168.0.10	READ_COILS
192.168.0.3	192.168.0.10	READ_DISCRETE_INPUTS
192.168.0.3	192.168.0.10	READ_DISCRETE_INPUTS
192.168.0.3	192.168.0.10	READ_COILS
192.168.0.3	192.168.0.10	READ_COILS
192.168.0.3	192.168.0.10	READ_DISCRETE_INPUTS
192.168.0.3	192.168.0.10	READ_DISCRETE_INPUTS
192.168.0.3	192.168.0.10	READ_DISCRETE_INPUTS
192.168.0.3	192.168.0.10	READ_DISCRETE_INPUTS
192.168.0.3	192.168.0.10	READ_COILS
192.168.0.3	192.168.0.10	READ_COILS
```

# Detecting ARP Spoofing with Zeek!

We were able to detect Modbus anomalies with Suricata. But ARP spoofing was more difficult.

Zeek takes a broader and more flexible view of network traffic. In this section, we'll implement ARP Spoof detection in Zeek.

*Let's GO!*

### Detecting ARP events with Zeek

By default, Zeek does not have an `arp.log` file like it does for `http` and `dns`. But it does dissect ARP and has events that we can hook into with Zeek scripts.

Let's first detect ARP requests and replies. This will also serve as a Zeek script primer.

Don't get bogged down in Zeek script syntax.
1. You can do alot with Claude or ChatGPT.
2. You can learn [Zeek scripting](https://docs.zeek.org/en/current/reference/zeekscript/index.html) as a followup to this... and you SHOULD! 😄

Open your text editor, creating a file called `arp.zeek` in the directory you created above and insert the following contents.

```
event arp_request(mac_src: string, mac_dst: string, SPA: addr, SHA: string, TPA: addr, THA: string)
    {
    print fmt("ARP request: %s/%s asks for %s", SPA, SHA, TPA);
    }

event arp_reply(mac_src: string, mac_dst: string, SPA: addr, SHA: string, TPA: addr, THA: string)
    {
    print fmt("ARP reply: %s is-at %s", SPA, SHA);
    }
```

Now run Zeek. You should see the following.

```
analyst@nsm:~/zeek$ sudo zeek -i ens37 arp.zeek 
listening on ens37

ARP request: 192.168.0.137/00:0c:29:CC:DD:b5 asks for 192.168.0.10
ARP reply: 192.168.0.10 is-at 00:d0:7c:AA:BB:8c
ARP request: 192.168.0.10/00:d0:7c:AA:BB:8c asks for 192.168.0.3
ARP reply: 192.168.0.3 is-at 00:0c:29:EE:FF:5d
```

Cool. We can see ARP traffic!

### Adding a custom Zeek Script to detect ARP Spoofing!

Now let's implement the ARP Spoof detection. I got this script by simply asking ChatGPT for a zeek script that would detect ARP spoofing. Crazy, right?!

Create a file called `/opt/zeek/share/zeek/site/arp-spoof-detect.zeek` with the following content:

```
@load base/frameworks/notice

module ARPWatch;

export {
    redef enum Notice::Type += {
        Possible_ARP_Spoofing
    };
}

# Tracks the MAC address Zeek has seen for each IP.
global ip_to_mac: table[addr] of string;

# Optional: ignore these IPs if needed.
# Example:
# const ignored_ips: set[addr] = { 0.0.0.0, 255.255.255.255 } &redef;
const ignored_ips: set[addr] = { 0.0.0.0 } &redef;

function observe_arp(spa: addr, sha: string, source: string)
    {
    if ( spa in ignored_ips )
        return;

    if ( spa in ip_to_mac )
        {
        if ( ip_to_mac[spa] != sha )
            {
            NOTICE([
                $note=Possible_ARP_Spoofing,
                $msg=fmt("Possible ARP spoofing: IP %s changed MAC from %s to %s via %s",
                         spa, ip_to_mac[spa], sha, source),
                $identifier=fmt("%s", spa)
            ]);
            }
        }
    else
        {
        ip_to_mac[spa] = sha;
        }
    }

event arp_request(mac_src: string, mac_dst: string, SPA: addr, SHA: string, TPA: addr, THA: string)
    {
    observe_arp(SPA, SHA, "arp_request");
    }

event arp_reply(mac_src: string, mac_dst: string, SPA: addr, SHA: string, TPA: addr, THA: string)
    {
    observe_arp(SPA, SHA, "arp_reply");
    }
```

Now, add the zeek script to the zeek config, `sudo vim /opt/zeek/share/zeek/site/local.zeek`, and add the following to the end:

```
@load ./arp-spoof-detect
```

Now, let's check to make sure there are no errors with our config and script.

```
analyst@nsm:~/zeek$ sudo zeekctl check
Warning: zeekctl option UseWebSocket is set, but websockets non-functional: Failed to import websockets module (ModuleNotFoundError("No module named 'websockets'"))
Warning: The print and netstats commands will be non-functional, please install the Python websockets package version 11.0 or later.
zeek scripts are ok.
```

Next, let's *give it a go!* Let's run Zeek with our new script.

```
analyst@nsm:~/zeek$ sudo zeekctl deploy
Warning: zeekctl option UseWebSocket is set, but websockets non-functional: Failed to import websockets module (ModuleNotFoundError("No module named 'websockets'"))
Warning: The print and netstats commands will be non-functional, please install the Python websockets package version 11.0 or later.
checking configurations ...
installing ...
removing old policies in /opt/zeek/spool/installed-scripts-do-not-touch/site ...
removing old policies in /opt/zeek/spool/installed-scripts-do-not-touch/auto ...
creating policy directories ...
installing site policies ...
generating standalone-layout.zeek ...
generating local-networks.zeek ...
generating zeekctl-config.zeek ...
generating zeekctl-config.sh ...
stopping ...
stopping zeek ...
starting ...
starting zeek ...
analyst@nsm:~/zeek$ 
```

OK, Zeek is running with our script. Now it is time to detect ARP spoofing!

### Detecting ARP Spoofing with Zeek

Let's give our custom Zeek script a try.

1. Start Zeek with your custom script. `sudo zeekctl deploy`.
2. Monitor the Zeek `notice.log`.

```
analyst@nsm:~/zeek$ sudo zeekctl deploy
analyst@nsm:~/zeek$ sudo tail -F /opt/zeek/logs/current/notice.log
```

`tail -F` will monitor a file, showing changes as they happen. So, you'll see new alerts as they come in.

3. Startup your Kali attack box (the one we used in our [Modbus Attack](../5-Modbus-Deep-Dive-and-first-ATTACK/README.md).
4. Initiate an ARP Spoof Man-In-The-Middle attack.

```
sudo ettercap -T -i eth0 -M arp:remote /192.168.0.3// /192.168.0.10//
```

You are now performing an ARP spoof attack. Let's see if Zeek caught it!

On your Zeek machine, you should see the following in the `tail -F` output:

```
analyst@nsm:~/zeek$ sudo tail -F /opt/zeek/logs/current/notice.log
tail: cannot open '/opt/zeek/logs/current/notice.log' for reading: No such file or directory
tail: '/opt/zeek/logs/current/notice.log' has appeared;  following new file
#separator \x09
#set_separator	,
#empty_field	(empty)
#unset_field	-
#path	notice
#open	2026-05-23-17-01-35
#fields	ts	uid	id.orig_h	id.orig_p	id.resp_h	id.resp_p	fuid	file_mime_type	file_desc	proto	note	msg	sub	src	dst	p	n	peer_descr	actions	email_dest	suppress_for	remote_location.country_code	remote_location.region	remote_location.city	remote_location.latitude	remote_location.longitude
#types	time	string	addr	port	addr	port	string	string	string	enum	enum	string	string	addr	addr	port	count	string	set[enum]	set[string]	interval	string	string	string	double	double
1779573695.231390	-	-	-	-	-	-	-	-	-	ARPWatch::Possible_ARP_Spoofing	Possible ARP spoofing: IP 192.168.0.10 changed MAC from 00:d0:7c:AA:BB:8c to 00:0c:29:CC:DD:b5 via arp_reply	-	-	-	-	-	-	Notice::ACTION_LOG	(empty)	3600.000000	-	-	-	-	-
1779573695.231409	-	-	-	-	-	-	-	-	-	ARPWatch::Possible_ARP_Spoofing	Possible ARP spoofing: IP 192.168.0.3 changed MAC from 00:0c:29:EE:FF:5d to 00:0c:29:CC:DD:b5 via arp_reply	-	-	-	-	-	-	Notice::ACTION_LOG	(empty)	3600.000000	-	-	-	-	-
```

There you go! Do you see it? Both the PLC and the Ignition Gateway (HMI) are now mapped to `00:0c:29:CC:DD:b5`.

Great job! You DID IT!

# References

- [Industrial Control System Security, Volume 2](https://www.packtpub.com/en-us/product/industrial-cybersecurity-9781800202092), by Pascal Ackerman
- [Zeek Documentation](https://docs.zeek.org/en/current/)
- ChatGPT!
