
# Tor-Gapping Gateway – Linksys WRT3200ACM

## Overview

The **Tor-Gapping Gateway** is a network security project designed to enforce anonymity at the network edge by routing all outbound traffic through the [Tor network](https://www.torproject.org/), combined with **real-time traffic monitoring, security analytics, and advanced firewall enforcement**.

It is both:
- A **functional privacy tool** for everyday use.
- A **cybersecurity learning platform** to explore threat modeling, traffic analysis, protocol security, and intrusion detection.

The project begins with a dedicated OpenWrt-based Tor gateway on a **Linksys WRT3200ACM**, and expands into a **comprehensive monitoring and analysis dashboard** that inspects all network traffic for leaks, anomalies, and threats.

---

## Project Goals

- **Enforce Anonymity by Default** – All devices connected to the network automatically use Tor for supported traffic.
- **Prevent Data Leaks** – Block unprotected protocols (UDP, QUIC) or reroute them securely.
- **Gain Visibility** – Collect, analyze, and visualize all inbound and outbound traffic.
- **Hands-On Cybersecurity Learning** – Build, test, and refine security policies with real-world data.
- **Document and Demonstrate Skills** – Produce a portfolio-ready project that shows both implementation and analysis.

---

## Features

### 1. Tor Gateway Core
- Full-network Tor routing for all TCP and DNS traffic.
- DNS leak protection via Tor's DNSPort.
- Custom firewall rules to intercept and control traffic.
- Optional fail-safe “kill switch” mode if Tor fails.

### 2. Security Enforcement
- Block UDP/QUIC and other unprotected traffic.
- VLAN/SSID segmentation for Tor, VPN, and direct internet.
- SSH key-only router administration with restricted management VLAN.

### 3. Traffic Monitoring & Analysis
- **Data Collection**: `tcpdump`/`tshark` packet captures on WAN/LAN interfaces.
- **Protocol Classification**: Identify and categorize all observed traffic.
- **Leak Detection**: Automated scans for non-Tor outbound connections.
- **Anomaly Detection**: Highlight unusual patterns for further investigation.

### 4. Visualization & Dashboard
- **Web-based dashboard** (planned) to display:
  - Real-time traffic charts (bandwidth, protocols, destinations).
  - Tor circuit information.
  - Devices connected and their activity.
  - Security alerts (blocked traffic, policy violations).
- Integration with **Grafana + Prometheus** or **ELK stack** for advanced analytics.

### 5. Security Testing & Experimentation
- Simulate misconfigured clients and confirm traffic isolation.
- Test DNS leak prevention with tools like `dnsleaktest.com`.
- Measure performance impact of Tor routing.
- Document attack vectors and mitigations.

---

## Architecture
```text
[ Client Devices ]
   |  (LAN/VLAN: Tor, VPN, Direct)
   v
[ WRT3200ACM Tor-Gapping Gateway ]
  ├─ OpenWrt
  ├─ Tor (TransPort: 9040, DNSPort: 9053)
  ├─ Firewall (nftables) + Kill-Switch
  └─ Monitoring (pcap/tshark, exporters)
        |
        v
[ ISP Router / Modem ]
        |
        v
[ Internet ]
   └─[ Tor Exit Nodes ]
```


- **Traffic Path**: All TCP/DNS → Tor → Internet.
- **Security Layers**: Firewall + Anonymity Network + Monitoring + Analytics.
- **Future Capability**: On-gateway ML-driven anomaly detection.

---

## Technology Stack

- **Router Firmware**: OpenWrt 23.x
- **Anonymity Network**: Tor (v3)
- **Traffic Analysis**: tcpdump, tshark, Suricata/Snort
- **Visualization**: Grafana, Prometheus, ELK stack (planned)
- **Automation**: Shell scripts, OpenWrt `uci` firewall config
- **Monitoring**: collectd / node_exporter

---

## Roadmap

| Phase | Description |
|-------|-------------|
| **1** | Build and configure Tor gateway with full-network routing and DNS leak protection. |
| **2** | Implement firewall rules for UDP/QUIC blocking and VLAN segmentation. |
| **3** | Add traffic collection and leak detection scripts. |
| **4** | Develop real-time web dashboard for monitoring and analytics. |
| **5** | Integrate intrusion detection and automated alerts. |
| **6** | Conduct and document security testing scenarios. |

---

## Security Considerations

- **Not a complete anonymity guarantee** – Browser fingerprinting, malware, and endpoint compromise remain risks.
- **Performance trade-offs** – Tor routing adds latency and reduces throughput.
- **Ongoing Maintenance** – Tor node changes, protocol updates, and firewall tuning required.

---

## Educational Value

This project demonstrates:
- Secure router configuration.
- Network protocol analysis.
- Firewall and packet filtering.
- SIEM/log analysis workflows.
- End-to-end network security design.

---

## Disclaimer

For **educational and research purposes only**. Users are responsible for understanding and complying with all applicable laws and regulations in their jurisdiction.
