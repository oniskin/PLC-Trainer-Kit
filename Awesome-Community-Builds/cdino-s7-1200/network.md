# Network

![network diagram](media/network.png)

## Zones

The network is split into three zones, enforced by an OPNsense firewall VM running on Proxmox:

| Zone | Subnet | What lives here |
|------|--------|-----------------|
| WAN | Home router uplink | OPNsense WAN interface |
| DMZ  | 172.16.1.0/24 | Factory stack - NeuronEX, HiveMQ, Ignition, Caddy |
| OT / Factory floor | 10.0.0.0/24 | S7-1200 PLC, network switch |

There's also a separate IT zone for the historian and dashboards, covered below.

The idea is to mirror how a real industrial site is structured - the PLC sits in an isolated OT segment, the SCADA and gateway sit in a DMZ, and nothing in the OT zone can initiate outbound connections. Data flows one way: NeuronEX polls the PLC and pushes to the broker. Everything downstream reads from the broker.

## Factory Stack

I've included the [docker-compose.yml](factory-network/docker-compose.yml) and [Caddyfile](factory-network/Caddyfile) so you should be able to spin up the same setup without too much faff.

The stack runs on a single Linux machine (Ubuntu Server) in the DMZ. All services run in Docker and are proxied behind Caddy at `*.factory.home.lab`. DNS is handled by OPNsense Unbound host overrides - no extra DNS server needed.

| Service | Role | URL |
|---------|------|-----|
| NeuronEX | OPC-UA client → Sparkplug B / JSON publisher | `neuronex.factory.home.lab` |
| HiveMQ CE | MQTT broker - the UNS spine | No web UI in CE |
| Ignition | SCADA, HMI, historian | `ignition.factory.home.lab` |
| Caddy | Reverse proxy, TLS termination | - |

**Grab a Maker Edition licence for Ignition** - it's free for personal use and is significantly better than the two-hour trial reset. Get it at [inductiveautomation.com/ignition/maker-edition](https://inductiveautomation.com/ignition/maker-edition).

## IT Stack

A second Ubuntu Server VM runs the IT zone historian and dashboard stack, sitting at `*.home.lab`. This zone has read-only access to HiveMQ - it subscribes to MQTT topics but has no write path back to the factory zone. OPNsense enforces this.

| Service | Role | URL |
|---------|------|-----|
| Telegraf | MQTT subscriber, InfluxDB writer | - |
| InfluxDB | Time-series historian | `influxdb.home.lab` |
| Grafana | Dashboards and visualisation | `grafana.home.lab` |
| Caddy | Reverse proxy, TLS termination | - |

## Data Flow

```
S7-1200 (OT zone)
    ↑ OPC-UA poll (port 4840)
NeuronEX (DMZ)
    ↓ Sparkplug B → HiveMQ (UNS)
    ↓ JSON → HiveMQ (IT historian feed)
HiveMQ CE (DMZ)
    ↓ Sparkplug B subscribe        ↓ JSON subscribe
Ignition (DMZ)                Telegraf → InfluxDB → Grafana (IT zone)
SCADA / HMI
```

Write-back from Ignition to the PLC flows via OPC-UA direct, or via Sparkplug B DCMD through HiveMQ → NeuronEX → OPC-UA write.

## ⚠️ Proxmox USB NIC - Promiscuous Mode

If you're using a USB-to-Ethernet adapter as an extra NIC on your Proxmox host (e.g. for the OT network interface), you **must** enable promiscuous mode on it or bridged VM traffic will silently fail. I spent several hours troubleshooting this before finding the fix - hopefully this saves you the same pain.

Enable it manually (doesn't survive reboot):

```bash
ip link set [nic] promisc on
```

Make it permanent by adding this to the relevant bridge block in `/etc/network/interfaces`:

```
post-up ip link set [nic] promisc on
```

Replace `[nic]` with your actual interface name (e.g. `enxc8a3628e182e`). Run `ip link show` to find it.

The bridge will show as active and the VM interfaces will look fine - the only symptom is that traffic from VMs on that bridge never reaches the physical network. ARP requests go nowhere and you'll see no traffic on `tcpdump` on the physical NIC. Promiscuous mode is required for a Linux bridge to pass traffic with foreign MAC addresses, and USB NICs don't always enable it automatically when added to a bridge.
