# Introduction

Suricata was cool. It's a great tool to rapidly alert on known-bad network behavior.

To be honest, it's great to have, but the bad-guys will get past that pretty quickly. They can practice their attacks against Snort / Suricata publicly available rules (which get used by all our favorite vendors, too!)

We need something deeper to complement Suricata. Something that tracks and correlates all this network data. Something that can help spot complex attacks and strange behavior over a longer time period.

Sound like fun?

**Let's do MORE network detection! With Zeek!**

*Let's Go!*

# First, Let's get Zeek running

### Install Suricata.

First, you'll need an Ubuntu virtual machine.
I'm using the same virtual machine from the [Suricata install](Kit-Build-Guides/7-Suricata-Network-Detection/README.md) we just did.

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

Like in the [Suricata install](Kit-Build-Guides/7-Suricata-Network-Detection/README.md), we will configure Zeek to listen to the proper interface.

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

Then go to the ```af-packet``` section, changing the default value to the name of the network interface Suricata should be listening to.

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

If that's hard to read, we can only select certain columns with the `zeekcut` utility. Critical!

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

Great job! You DID IT!

# References

- Industrial Control System Security, by Pascal Ackerman
- Suricata Documentation
- ChatGPT!