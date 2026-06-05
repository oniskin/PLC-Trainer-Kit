# Bill of Materials (BOM)

## Hardware
Hardware used in the build. Optional items are marked with an * and aren't required for core functionality - they either expand funtionality or add quality of life features.

I am based in the UK, so all prices and links reflect this.

The more expensive components (PLC, Switch, PSU, Server) were all bought second hand, so your milage may vary RE cost and condition. (If you need help buying your S7-1200, check out my [buyers guide!](https://cdino.net/blog/2026/s7-1200-plc-cpu-buyers-guide/))

| **Item**                  | **Link**               | **Notes**                                                                                      |
|---------------------------|------------------------|------------------------------------------------------------------------------------------------|
|Siemens S7-1200 CPU || I got the DC/DC/DC model with 4.* firmware. IMO it's the best variant for hobbiests.|
|24VDC PSU||I picked up a SITOP PSU100L for a good deal on eBay (although it's probably overkill for this project) but any PSU that will power 24VDC componenets should be fine. You will also need a way of powering it. In my build, I stripped a kettle lead and powered it off mains.|
|DIN Rail|[RS Components (500mm)](https://uk.rs-online.com/web/p/din-rails/0467406)|I used a 250mm rail on the top for power, and a 500mm rail on the bottom for the other componenets.|
|Piece of wood||I used a 12x500x500mm piece of ply that I got cut at my local B&Q for a fiver.|
|Green LED Push Button|[eBay](https://www.ebay.co.uk/itm/297362516760?var=594872514861)|Works as an input and an output. I wired it as NC.|
|Red LED Push Button|[eBay](https://www.ebay.co.uk/itm/297362516760?var=594872514866)|Works as an input and an output. I wired it as NO.|
|24VDC Fan|[Amazon](https://www.amazon.co.uk/dp/B0DPQKPLF9?ref=ppx_yo2ov_dt_b_fed_asin_title)|Could be replaced with a PWM fan, it your PLC supports it.|
|Terminal Blocks|[RS Components](https://uk.rs-online.com/web/p/din-rail-terminal-blocks/2624161)|Used for connecting power and signals around the board. I bought 20.|
|Jumper Bars|[RS Components](https://uk.rs-online.com/web/p/din-rail-terminal-accessories/0102374)|Used to jump power across terminals. I cut them down to size as required.|
|MCB|[RS Components](https://uk.rs-online.com/web/p/mcbs/2650204)|An extra level of saftey and acts as an on/off switch|
|Eather Terminal Blocks|[RS Components](https://uk.rs-online.com/web/p/din-rail-terminal-blocks/8725628)|For earthing the PLC, PSU, and DIN rails.|
|Red Wire|[Amazon](https://www.amazon.co.uk/dp/B0FQP5QHZ7?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)|For +24VDC. I bought 10M.|
|Black Wire|[Amazon](https://www.amazon.co.uk/dp/B0FQP6H7Z7?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)|For 0VDC. I bought 10M.|
|Ground Wire|[Amazon](https://www.amazon.co.uk/dp/B01MYBMAQ5?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)|For grounding|
|Bootlace Ferrules*|||
|Button Box*|[Amazon](https://www.amazon.co.uk/dp/B07WH9VJZ5?ref=ppx_yo2ov_dt_b_fed_asin_title)|Used to house and mount buttons|
|Wire Duct*|[CEF](https://www.cef.co.uk/catalogue/products/4349444-25mm-x-25mm-open-slotted-trunking-grey-2m-length)|Keep those wires neat and tidy!|
|Mini PC*||Run SCADA/OPC-UA/UNS systems.|
|Network Switch*||Connect the PLC, server, and laptop. I picked up a “Weidmuller IE SW BL05 STX” unmanaged switch on a whim, however, a managed switch would be a lot more fun (and money)!|
|Stand*|[Amazon](https://www.amazon.co.uk/dp/B0B6VKS7DJ?ref=ppx_yo2ov_dt_b_fed_asin_title)|To hold up the rig.|

## Tools

| **Item**                  |  **Notes**                                                                                      |
|---------------------------|------------------------------------------------------------------------------------------------|
|Flathead Screw Driver|For opening terminals|
|Ferrule Crimper|For crimping ferrules|
|Wire Stripper|For stripping wire|
|Snips|For cutting wire|
|Junior Hack Saw|For cutting wire ducts|


## Software
TIA Portal does require a license for use, although there is a trial available. I would not condone downloading cracked or pirated versions of the software, but I can't stop you.


| **Software** | **Notes**|
|--------------|----------|
|Windows 11|Running TIA Portal|
|TIA Portal|To program the PLC|
|[Ubuntu Server](https://ubuntu.com/download/server)|To run SCADA/UNS/MQTT Broker, etc|
|[Kali](https://www.kali.org/)|To emulate the attacker|
|[opnsense](https://opnsense.org/)|Firewall|