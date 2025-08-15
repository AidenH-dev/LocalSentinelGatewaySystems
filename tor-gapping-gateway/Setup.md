# Setup and configuration of base configuration for tor routing

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

   Install the required packages. Configure Tor client.
   ```bash
   opkg update
   opkg install tor
   ```
   Configure Tor client
   ```bash
   cat << EOF > /etc/tor/custom
   AutomapHostsOnResolve 1
   AutomapHostsSuffixes .
   VirtualAddrNetworkIPv4 172.16.0.0/12
   VirtualAddrNetworkIPv6 [fc00::]/8
   DNSPort 0.0.0.0:9053
   DNSPort [::]:9053
   TransPort 0.0.0.0:9040
   TransPort [::]:9040
   EOF
   cat << EOF >> /etc/sysupgrade.conf
   /etc/tor
   EOF
   uci del_list tor.conf.tail_include="/etc/tor/custom"
   uci add_list tor.conf.tail_include="/etc/tor/custom"
   uci commit tor
   service tor restart
   ```
   Disable IPv6 GUA prefix and announce IPv6 default route.

5. **DNS over Tor**
   Configure firewall to intercept DNS traffic.
   
   ```bash
   # Intercept DNS traffic
   uci -q del firewall.dns_int
   uci set firewall.dns_int="redirect"
   uci set firewall.dns_int.name="Intercept-DNS"
   uci set firewall.dns_int.family="any"
   uci set firewall.dns_int.proto="tcp udp"
   uci set firewall.dns_int.src="lan"
   uci set firewall.dns_int.src_dport="53"
   uci set firewall.dns_int.dest_port="53"
   uci set firewall.dns_int.target="DNAT"
   uci commit firewall
   service firewall restart
   Redirect DNS traffic to Tor and prevent DNS leaks.

   # Enable DNS over Tor
   service dnsmasq stop
   uci set dhcp.@dnsmasq[0].localuse="0"
   uci set dhcp.@dnsmasq[0].noresolv="1"
   uci set dhcp.@dnsmasq[0].rebind_protection="0"
   uci -q delete dhcp.@dnsmasq[0].server
   uci add_list dhcp.@dnsmasq[0].server="127.0.0.1#9053"
   uci add_list dhcp.@dnsmasq[0].server="::1#9053"
   uci commit dhcp
   service dnsmasq start
   ```
   
7. Firewall

   Configure firewall to intercept LAN traffic. Disable LAN to WAN forwarding to prevent traffic leaks.

   ```bash
   # Intercept TCP traffic
   cat << "EOF" > /etc/nftables.d/tor.sh
   TOR_CHAIN="dstnat_$(uci -q get firewall.tcp_int.src)"
   TOR_RULE="$(nft -a list chain inet fw4 ${TOR_CHAIN} \
   | sed -n -e "/Intercept-TCP/p")"
   nft replace rule inet fw4 ${TOR_CHAIN} \
   handle ${TOR_RULE##* } \
   fib daddr type != { local, broadcast } ${TOR_RULE}
   EOF
   uci -q delete firewall.tor_nft
   uci set firewall.tor_nft="include"
   uci set firewall.tor_nft.path="/etc/nftables.d/tor.sh"
   uci -q delete firewall.tcp_int
   uci set firewall.tcp_int="redirect"
   uci set firewall.tcp_int.name="Intercept-TCP"
   uci set firewall.tcp_int.src="lan"
   uci set firewall.tcp_int.src_dport="0-65535"
   uci set firewall.tcp_int.dest_port="9040"
   uci set firewall.tcp_int.proto="tcp"
   uci set firewall.tcp_int.family="any"
   uci set firewall.tcp_int.target="DNAT"
   ```
   ```bash
   # Disable LAN to WAN forwarding
   uci -q delete firewall.@forwarding[0]
   uci commit firewall
   service firewall restart
   ```
   
8. Test

   From any device connected to the WRT3200ACM, visit:
   https://check.torproject.org

### Considerations

While all DNS lookups, TCP connections and Accidental Direct Connections are now tor protected all UDP or QUIC are not protected. This leads us to the next problem to solve covering the privacy and security of that type of traffic. (LINK TO SELF HOSTED VPN PART OF PROJECT) More work has been done blocking and disabling UDP and QUIC in the router firewall to disable any user from accidentally exposing themself when connected to the network. 

**Acknowledgements**

Installing and configuring guidelines steps 4-8 were provided by this OpenWRT user guide: https://openwrt.org/docs/guide-user/services/tor/client
