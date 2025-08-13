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
2. **Wire Linksys WRT to 5G Gateway**
   ```javascript
   Physical Wiring
   
   1: Take an Ethernet cable and plug one end into the LAN port of the T-Mobile 5G Gateway (often labeled LAN or numbered).
   2: Plug the other end into the WAN port (sometimes blue) on the WRT3200ACM.
   3: Connect the computer to one of the LAN ports (yellow) on the WRT3200ACM or join its Wi-Fi network.
   ```
   ```md
   WAN Configuration on OpenWrt
   
   1. Log in to OpenWrt’s web interface (LuCI):
   http://192.168.1.1
   
   2. Go to Network → Interfaces.
   
   3. Edit the WAN interface:
      - Protocol: DHCP client (most T-Mobile gateways hand out IPs automatically).
      - Device: Set to the physical WAN port (likely eth1 on WRT3200ACM).
      - Save & Apply.
   
   4. Make sure LAN and WAN are separate networks:
      - LAN: 192.168.1.x (the internal network)
      - WAN: will get a different IP from the T-Mobile gateway (often 192.168.12.x or 192.168.0.x).
   ```
3. **Test Connection**
   Test Internet on OpenWrt
   SSH into the WRT3200ACM:
   
   ```bash
   ssh root@192.168.1.1
   ping -c 4 openwrt.org
   ```
   If replies, it’s online. If not:
   
   Double-check WAN is set to DHCP client.
   
   Reboot both the T-Mobile gateway and the WRT3200ACM.
4. **SSH into the router** and install Tor:
   ```bash
   ssh root@192.168.1.1
   ```


