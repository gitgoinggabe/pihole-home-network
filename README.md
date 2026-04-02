# Pi-hole Home Network Ad Blocker

A Raspberry Pi configured as a DNS sinkhole to block ads, trackers, and malicious domains across every device on the home network. No browser extensions or per-device configuration required.

---

## Results

| Metric | Value |
|---|---|
| Total DNS queries processed | 1,933+ |
| Queries blocked | 498 |
| Block rate | 25.7% |
| Domains on blocklist | 91,978 |
| Active clients protected | 3+ |

---

## Hardware and Software

| Component | Details |
|---|---|
| Device | Raspberry Pi |
| Operating System | Raspberry Pi OS Lite |
| Ad Blocker | Pi-hole |
| Router | Xfinity Gateway |
| Network Range | 10.0.0.x |

---

## Network Architecture

```
Internet
    |
    v
Xfinity Gateway (10.0.0.1)
    |
    v
Raspberry Pi -- Pi-hole DNS + DHCP Server (10.0.0.x)
    |
    |-- Phones
    |-- Laptops
    |-- Smart TVs
    |-- All other connected devices
```

All devices on the network receive their IP address and DNS server assignment from Pi-hole via DHCP. DNS queries for known ad and tracking domains are blocked before they reach the internet.

---

## Setup

### Step 1 -- Flash Raspberry Pi OS

Raspberry Pi Imager was used to flash the operating system onto an micro SD card. WiFi credentials and a custom hostname were pre-configured using the Imager advanced settings panel before the first boot.

### Step 2 -- Install Pi-hole

After booting the Pi and connecting via Putty to SSH, Pi-hole was installed using the official script:

```bash
curl -sSL https://install.pi-hole.net | bash
```

The installer walks through upstream DNS selection, network interface configuration, and admin dashboard setup.

### Step 3 -- Reserve a Static IP

Inside the Xfinity Gateway dashboard at `http://10.0.0.1`, the Raspberry Pi was located under Connected Devices. Its IP address was reserved so the assignment does not change between reboots or lease renewals.

### Step 4 -- Configure DHCP

The Xfinity gateway does not expose a DNS configuration field in its interface. To work around this, the following approach was used:

- The Xfinity DHCP range was narrowed to `10.0.0.200 -- 10.0.0.253`
- Pi-hole's built-in DHCP server was enabled with a range of `10.0.0.10 -- 10.0.0.199`

Pi-hole now distributes IP addresses to all devices on the network and automatically sets itself as their DNS server in the process.

### Step 5 -- Add Blocklists

The following community-maintained blocklists were added through the Pi-hole admin dashboard under Lists:

```
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
https://adaway.org/hosts.txt
https://blocklistproject.github.io/Lists/ads.txt
https://blocklistproject.github.io/Lists/tracking.txt
```

After adding the lists, Gravity was updated via Tools to apply the new domains.

---

## How It Works

When a device on the network makes a DNS request, the query goes to Pi-hole first. Pi-hole checks the requested domain against its blocklist. If the domain is on the list, Pi-hole returns a blank response and the request goes no further. If the domain is not on the list, Pi-hole forwards the query to an upstream DNS resolver such as Cloudflare or Google. The device never contacts the ad server directly.

---

## Blocklists

| List | Purpose |
|---|---|
| StevenBlack Hosts | Unified ad and malware domains |
| AdAway | Mobile advertising domains |
| BlocklistProject Ads | Broad ad domain coverage |
| BlocklistProject Tracking | Tracking and telemetry domains |

---

## Limitations

Pi-hole blocks DNS queries. It does not inspect or intercept traffic. This creates some gaps in coverage.

YouTube ads are served from the same domain as video content. Blocking the domain would break YouTube entirely, so these ads pass through.

Many mobile games use ad SDKs that either hardcode DNS servers like `8.8.8.8` or serve ads from first-party domains. These also bypass Pi-hole.

If the Raspberry Pi goes offline, DNS resolution fails for the whole network. A fallback upstream DNS entry in Pi-hole settings reduces this risk.


## Skills Demonstrated

- Raspberry Pi setup and Linux administration
- DNS server configuration and management
- DHCP server setup and network segmentation
- IPv4 networking fundamentals
- Home network architecture and troubleshooting
- Pi-hole deployment and blocklist management

---

## References

- Pi-hole Documentation: https://docs.pi-hole.net/
- Raspberry Pi Imager: https://www.raspberrypi.com/software/
- StevenBlack Blocklists: https://github.com/StevenBlack/hosts
- Pi-hole Community: https://discourse.pi-hole.net/
