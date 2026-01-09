# Network Infrastructure Troubleshooting: pfSense & Virtual Labs

## Project Overview
This project documents a hands-on troubleshooting session within a virtualized home lab environment. The goal was to diagnose and resolve connectivity failures between a **pfSense Firewall**, **Kali Linux**, and **Windows** clients.

## Lab Architecture
* **Hypervisor:** VMware Workstation
* **Gateway/Firewall:** pfSense (acting as the router and DHCP server)
* **Attacker/Client:** Kali Linux
* **Victim/Client:** Windows 10
* **Network Segments:** LAN (Internal) vs. WAN (External/Internet)

## The Incident (Symptoms)
After deploying the lab, the following connectivity issues were observed:
1.  **Internal Isolation:** Windows machines could not ping each other.
2.  **Gateway Failure:** Kali Linux could not ping the pfSense default gateway (`192.168.100.1`).
3.  **Partial Connectivity:** Internet access was functional, but internal routing (LAN-to-LAN) was broken.

## Troubleshooting Methodology (The OSI Model Approach)

### Phase 1: Layer 3 Configuration Check
I verified the IP configuration on all endpoints to ensure they were on the correct subnet.
* **Windows:** Verified via `ipconfig`.
* **Linux:** Verified via `ip route` and `ifconfig`.
* **Finding:** All IPs, Subnet Masks, and Default Gateways were configured correctly.

### Phase 2: Gateway Analysis (pfSense)
Since Kali could not access the pfSense Web UI, I accessed it via the Windows machine (which had partial access).
* **Diagnostics:** Used the **pfSense Packet Capture** tool on the LAN interface.
* **Test:** Initiated a `ping` from Kali to pfSense.
* **Observation:** The capture showed ICMP "Echo Requests" arriving at the firewall, but **zero** "Echo Replies" being sent back.

![Uploading packet capture image.png…]()


### Phase 3: Root Cause Analysis
The packet capture confirmed that traffic was reaching the firewall but being dropped.
* **Cause:** pfSense operates on a **"Default Deny"** basis. There was no explicit firewall rule allowing ICMP (Ping) traffic on the LAN interface for these specific hosts.
* **Resolution:** Created a new Firewall Rule on the LAN interface to allow traffic from `LAN Net` to `Any`.






## Key Takeaways
* **Don't Assume:** Just because the Internet works doesn't mean the LAN is configured correctly.
* **Tools Matter:** When the terminal fails, use the Firewall's internal diagnostic tools (Packet Capture).
* **Default Behavior:** Enterprise firewalls like pfSense block everything by default, unlike consumer routers which allow everything.
