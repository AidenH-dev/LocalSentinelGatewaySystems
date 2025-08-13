#  Tor-Gapping Gateway — Linksys WRT3200ACM Edition

**_“Tor Gapping”_** — My term for adding a **dedicated router** in front of a default network ISP router that forces **all devices** to route their traffic through the **Tor network**.

This build uses the **Linksys WRT3200ACM** running **OpenWrt** to act as a transparent Tor gateway for **all connected devices**.

---

##  What is Tor Gapping?

> **Tor Gapping** = Tor + Network Gap  
> Placing an *additional* router between your ISP (in my case, T-Mobile 5G) and your devices, configured to transparently route **all TCP and DNS traffic** through Tor.

The goal:
- Hide the public IP of **every device** on the network.
- Make anonymity the *default* network state.
- Provide a cybersecurity-focused portfolio piece that demonstrates **network security engineering**.

---

##  Features

- **Full-network Tor routing** — all devices connect through Tor automatically.
- **Transparent proxy** — no Tor setup on client devices required.
- **DNS leak protection** — all DNS queries resolved over Tor.
- **Dual-band Wi-Fi** — AC3200 speeds, can dedicate one SSID for Tor traffic.
- **OpenWrt-powered** — custom firewall rules, reproducible builds, community support.

---

##  Hardware & Software

| Component             | Choice                         | Purpose |
|-----------------------|--------------------------------|---------|
| **Router**            | Linksys WRT3200ACM             | Dedicated Tor gateway hardware |
| **Firmware**          | [OpenWrt 23.x](https://openwrt.org/toh/linksys/wrt3200acm) | Custom firmware with Tor support |
| **Anonymity Network** | Tor (v3)                       | Hides source IP, supports onion services |
| **Firewall**          | `iptables` / `nftables`        | Forces all TCP/DNS through Tor |
| **ISP**               | T-Mobile 5G Gateway            | Internet uplink |

---

##  Installation Overview

1. **Flash OpenWrt** to the Linksys WRT3200ACM.
2. **SSH into the router** and install Tor:
   ```bash
   opkg update
   opkg install tor tor-geoip


