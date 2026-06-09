# Bill of Materials (BOM)

## Hardware

Optional items are marked with * - they're not required for core functionality but either expand what the build can do or make life easier.

I'm based in the UK so all links and prices reflect that. The more expensive components (PLC, switch, PSU, server) were all bought second hand, so your mileage may vary on cost and condition. If you need help buying your S7-1200, check out my [buyer's guide](https://cdino.net/blog/2026/s7-1200-plc-cpu-buyers-guide/).

| **Item** | **Link** | **Notes** |
|----------|----------|-----------|
| Siemens S7-1200 CPU | | I got the DC/DC/DC variant with 4.x firmware - IMO the best option for hobbyists. The DC/DC/DC model runs entirely on 24VDC which keeps the wiring simpler and safer. |
| 24VDC PSU | | I picked up a Siemens SITOP PSU100L on eBay - massively overspecced for this project, but I got it cheap and it's built like a tank. Any PSU that can supply 24VDC at a couple of amps will do the job. You'll also need a way to power it from mains - I stripped a kettle lead and wired it straight in. |
| DIN Rail | [RS Components (500mm)](https://uk.rs-online.com/web/p/din-rails/0467406) | I used a 250mm rail on the top for power components and a 500mm rail on the bottom for everything else. |
| Piece of wood | | A 12×500×500mm sheet of ply that I had cut at my local B&Q for a fiver. Does the job. |
| Green LED Push Button | [eBay](https://www.ebay.co.uk/itm/297362516760?var=594872514861) | Works as both input and output - the LED is wired to a PLC output, the button to an input. I wired it as NC (normally closed). |
| Red LED Push Button | [eBay](https://www.ebay.co.uk/itm/297362516760?var=594872514866) | Same as above. Wired as NO (normally open). |
| 24VDC Fan | [Amazon](https://www.amazon.co.uk/dp/B0DPQKPLF9?ref=ppx_yo2ov_dt_b_fed_asin_title) | Simple on/off fan as a physical output to demonstrate control. Could be swapped for a PWM fan if your PLC supports it. |
| Terminal Blocks | [RS Components](https://uk.rs-online.com/web/p/din-rail-terminal-blocks/2624161) | Used for distributing power and routing signals around the panel. I bought 20 and used most of them. |
| Jumper Bars | [RS Components](https://uk.rs-online.com/web/p/din-rail-terminal-accessories/0102374) | For bridging power across adjacent terminals. Cut them down to the length you need. |
| MCB | [RS Components](https://uk.rs-online.com/web/p/mcbs/2650204) | Adds overcurrent protection and doubles as a clean on/off switch for the whole panel. Worth having. |
| Earth Terminal Blocks | [RS Components](https://uk.rs-online.com/web/p/din-rail-terminal-blocks/8725628) | For earthing the PLC, PSU, and DIN rails. Don't skip this. |
| Red Wire | [Amazon](https://www.amazon.co.uk/dp/B0FQP5QHZ7?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1) | +24VDC. I bought 10m. |
| Black Wire | [Amazon](https://www.amazon.co.uk/dp/B0FQP6H7Z7?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1) | 0VDC. I bought 10m. |
| Ground Wire | [Amazon](https://www.amazon.co.uk/dp/B01MYBMAQ5?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1) | Earth/ground connections. |
| Bootlace Ferrules* | | Highly recommended - they make a much cleaner and more reliable connection in spring terminals than bare wire ends. Get a crimper to go with them. |
| Button Box* | [Amazon](https://www.amazon.co.uk/dp/B07WH9VJZ5?ref=ppx_yo2ov_dt_b_fed_asin_title) | Plastic enclosure for mounting the buttons neatly on the panel. |
| Wire Duct* | [CEF](https://www.cef.co.uk/catalogue/products/4349444-25mm-x-25mm-open-slotted-trunking-grey-2m-length) | Slotted trunking to keep wiring tidy and routed cleanly. Makes the build look the part. |
| Mini PC / Server* | | For running the SCADA, gateway, MQTT broker, and firewall. I used a Proxmox server with VMs, but any Linux machine will do. |
| Network Switch* | | To connect the PLC, server, and laptop on the factory network. I grabbed a Weidmüller IE SW BL05 STX unmanaged switch - it was a random eBay find. A managed switch would give you a lot more to play with from a security perspective, but costs accordingly. |
| Stand* | [Amazon](https://www.amazon.co.uk/dp/B0B6VKS7DJ?ref=ppx_yo2ov_dt_b_fed_asin_title) | A monitor stand to prop the board up at a sensible angle. |

## Tools

| **Item** | **Notes** |
|----------|-----------|
| Flathead screwdriver | For opening spring terminals - get a proper thin-bladed one, not a chunky household one. |
| Ferrule crimper | For crimping bootlace ferrules onto wire ends. Non-negotiable if you're using ferrules. |
| Wire stripper | Self-explanatory. An adjustable one that handles multiple wire gauges is worth the few extra quid. |
| Snips | For cutting wire to length. |
| Junior hacksaw | For cutting wire duct to length. |

## Software

TIA Portal requires a licence - a trial is available from Siemens directly. I would not condone downloading cracked or pirated versions of the software, but I can't stop you.

| **Software** | **Link** | **Notes** |
|--------------|----------|-----------|
| Windows 11 | | Required to run TIA Portal. A VM works fine. |
| TIA Portal | | For programming and configuring the S7-1200. The trial version is sufficient to get started. |
| Ubuntu Server | [ubuntu.com](https://ubuntu.com/download/server) | Runs the factory stack (SCADA, gateway, MQTT broker) and IT stack (historian, dashboards) in Docker. |
| OPNsense | [opnsense.org](https://opnsense.org/) | Firewall and network segmentation between WAN, DMZ, and OT zones. Runs as a VM on Proxmox. |
| Kali Linux | [kali.org](https://www.kali.org/) | For the attacker simulation scenarios. Run as a VM on the OT or DMZ network depending on what you're demonstrating. |
