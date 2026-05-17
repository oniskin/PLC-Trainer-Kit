# Introduction

You have now built your kit and performed your first Modbus attack!

**Awesome.**

In the last section, we discussed how network security monitoring is ideally suited to detect these attacks, and if you can detect them early, you stand a better chance of stopping them before they cause real damage.

In this section, we're going to setup *Suricata*, a tool used to detect threats and alert on suspicious packet flows.

We'll build the following key detections that will alert us to Rogue Modbus Commands.

- Medium: Unauthorized Modbus client.
- Critical: Unauthorized Rogue Modbus **write** command!

Suricata is going to look inside the packets as they travel from the HMI to the PLC and back. If the packets match the above criteria, we'll get an alert.

*Let's Go!*

# First, Let's get Suricata Set Up

### Install Suricata.

First, you'll need an Ubuntu virtual machine.
I created a new virtual machine just to keep things separated.

Next, install Suricata following the Suricata Documentation [Quickstart Guide](https://docs.suricata.io/en/latest/quickstart.html).

Follow the steps carefully top to bottom.

At the time of writing, the commands were as below (look at documentation. They could've changed.)

```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install suricata jq
```

```jq``` is a tool to parse ```json``` formated data.

### Configure to listen to network traffic

Now that it is installed, you'll need to edit the config file to make sure the Suricata network sensor is listening on the right interface.

Suricata will ingest network traffic as it flows through the host and trigger alerts for any packets that match pre-defined rules.

First, you need to find out what your network interface is called. Mine was called ```ens37```

```
analyst@nsm:~$ ip a
<--Snip-->
3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
    altname enp2s5
    inet 192.168.0.254/24 brd 192.168.0.255 scope global noprefixroute ens37
<--Snip-->
```

 open the config file.
```
sudo vim /etc/suricata/suricata.yaml
```

Then go to the ```af-packet``` section, changing the default value to the name of the network interface Suricata should be listening to.

```
af-packet:
    - interface: enp1s0  # <-- changes to whatever yours is called.
      cluster-id: 99
      cluster-type: cluster_flow
      defrag: yes
      tpacket-v3: yes
```

### Download Signatures

```
sudo suricata-update
```

This will download the signatures needed to fire detections.

### Running Suricata

```
sudo systemctl restart suricata
sudo systemctl status suricata
```

Status check should say it's running.

```
sudo tail -f /var/log/suricata/fast.log
```
*In a different Terminal, run below*
```
curl http://testmynids.org/uid/index.html
```

# Rogue Modbus Detections!

There is only limited built-in Modbus detections in Suricata.

The primary detection we will employ is any attempt to read or write to the PLC from a device other than the Ignition Gateway.

Following from this, we need to define the **Modbus clients and servers**.

### Defining Modbus Clients and Servers in Suricata

Open the config file and edit the ```MODBUS_CLIENT``` and ```MODBUS_SERVER``` variables.

```
    MODBUS_CLIENT: "[192.168.0.3/32]"     # Address of the Ignition Gateway
    MODBUS_SERVER: "[192.168.0.10/32]"    # address of the PLC
```

Enable modbus protocol:
```
    modbus:
      # How many unanswered Modbus requests are considered a flood.
      # If the limit is reached, the app-layer-event:modbus.flooded; will match.
      #request-flood: 500

      enabled: yes         # <--- This was no by default
      detection-ports:
        dp: 502
      # According to MODBUS Messaging on TCP/IP Implementation Guide V1.0b, it
      # is recommended to keep the TCP connection opened with a remote device
      # and not to open and close it for each MODBUS/TCP transaction. In that
      # case, it is important to set the depth of the stream reassembling as
      # unlimited (stream.reassembly.depth: 0)

      # Stream reassembly size for modbus. By default track it completely.
      stream-depth: 0
```

### Now add Suricata rules to detect Rogue Modbus Commands!

While the config file is still open, add ```local.rules``` to the list of rules that are loaded when Suricata starts.

```
default-rule-path: /var/lib/suricata/rules
    
rule-files:
  - suricata.rules
  - local.rules
```

Next, open ```/var/lib/suricata/rules/local.rules``` with a text editor.

```
sudo vim /var/lib/suricata/rules/local.rules
```
Read about Suricata rules and how they work in the [Suricata Documentation](https://docs.suricata.io/en/latest/rules/intro.html).

Add the following 5 rules to ```local.rules```:

```
alert tcp !$MODBUS_CLIENT any -> $MODBUS_SERVER 502 (msg:"OT-LAB: Rogue Modbus Client Detected!"; flow:to_server,established; app-layer-protocol:modbus; sid:1000001; rev:1;)
alert tcp !$MODBUS_CLIENT any -> $MODBUS_SERVER 502 (msg:"OT-LAB: Rogue Modbus *WRITE Single Coil* Detected!"; flow:to_server,established; modbus:function 5; sid:1000002; rev:1;)
alert tcp !$MODBUS_CLIENT any -> $MODBUS_SERVER 502 (msg:"OT-LAB: Rogue Modbus *WRITE Single Register* Detected!"; flow:to_server,established; modbus:function 6; sid:1000003; rev:1;)
alert tcp !$MODBUS_CLIENT any -> $MODBUS_SERVER 502 (msg:"OT-LAB: Rogue Modbus *WRITE Multiple Coils* Detected!"; flow:to_server,established; modbus:function 15; sid:1000004; rev:1;)
alert tcp !$MODBUS_CLIENT any -> $MODBUS_SERVER 502 (msg:"OT-LAB: Rogue Modbus *WRITE Multiple Registers* Detected!"; flow:to_server,established; modbus:function 16; sid:1000005; rev:1;)
```

All these rules do is generate an alert when a device that is not within the definition of ```MODBUS_CLIENT``` tries to read or write values to a critical Modbus device we have defined.

### Testing our detection rule

First, monitor the ```fast.log``` file, where we are expecting to see detections.

```
sudo tail -f /var/log/suricata/fast.log
```

Now, reexecute the attack from the last section from your Kali attack machine!

```
┌──(david㉿kali)-[~/Scripts/modbus-in-the-middle]
└─$ modbus read 192.168.0.10 %M8192 8                                  
%M8192     0
%M8193     1
%M8194     0
%M8195     0
%M8196     0
%M8197     0
%M8198     0
%M8199     0
                                                                                                                    
┌──(david㉿kali)-[~/Scripts/modbus-in-the-middle]
└─$ modbus write 192.168.0.10 %M8192 1                               
                                                                                                                    
┌──(david㉿kali)-[~/Scripts/modbus-in-the-middle]
└─$ modbus write 192.168.0.10 %M8192 0                                               
```

Voila! you should see the following alert in the ```fast.log``` file.

```
05/17/2026-00:28:51.995694  [**] [1:1000004:1] OT-LAB: Rogue Modbus *WRITE Multiple Coils* Detected! [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.0.137:59980 -> 192.168.0.
10:502
05/17/2026-00:28:51.996051  [**] [1:1000001:1] OT-LAB: Rogue Modbus Client Detected! [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.0.137:59980 -> 192.168.0.10:502
05/17/2026-00:28:51.996577  [**] [1:1000001:1] OT-LAB: Rogue Modbus Client Detected! [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.0.137:59980 -> 192.168.0.10:502
```

Great job! You DID IT!

# References

- Industrial Control System Security, by Pascal Ackerman
- Suricata Documentation
- ChatGPT!