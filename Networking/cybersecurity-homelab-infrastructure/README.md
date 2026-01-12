# Cybersecurity Home Lab: Infrastructure Design & Network Troubleshooting

## 

## Project Overview

This project documents the end-to-end design, deployment, and operational troubleshooting of a virtualized cybersecurity laboratory. The goal was to create a safe, isolated environment for ****Red Team**** (Attack) and ****Blue Team**** (Defense) simulations using enterprise-grade routing.

After the initial build, I conducted a live troubleshooting session to resolve a critical connectivity failure between network segments, demonstrating practical network diagnostic skills.

---

## Part 1: Infrastructure & Architecture (The Build)

### Lab Architecture

* ****Hypervisor:**** VMware Workstation Pro

* ****Gateway/Router:**** pfSense (Virtual Appliance)

* ****Network Segments:****

    * ****WAN:**** Bridged to Physical Network (Internet Access).

    * ****LAN:**** Host-Only/Internal Network (`192.168.1.x`) - **Strictly Isolated**.

### Component Configuration

1.  ****The Gateway
(pfSense):**** Acts as the edge firewall, DHCP server, and DNS forwarder.

2.  ****The Attacker
(Kali Linux):**** Penetration testing node located on the LAN.

3.  ****The Victim
(Windows 10):**** Corporate workstation simulation.

4.  ****The Server
(Metasploitable 2):**** Intentionally vulnerable target for exploit
development.


<img width="960" height="423" alt="VMware Library" src="https://github.com/user-attachments/assets/e9544291-dc9f-4a1e-9f23-6fd207086ad3" />


### Network Isolation Strategy

To prevent malware from escaping into my physical home network, I utilized ****LAN Segments**** within VMware. This ensures that traffic between Kali and the Victim flows strictly through the pfSense virtual router, mimicking a real corporate DMZ/Intranet.


<img width="960" height="495" alt="pfSense dashboard" src="https://github.com/user-attachments/assets/83ecce39-a9bc-43fb-8830-71eebb32b402" />


---

## Part 2: The Incident & Troubleshooting (The Fix)

### The Incident (Symptoms)

Post-deployment, I encountered a major routing failure:

* ****Symptom A:**** Windows clients could not communicate with each other.

* ****Symptom B:**** The Kali Linux machine could not ping the default gateway (`192.168.1.1`).

* ****Symptom C:**** Internet access was functional, but internal LAN-to-LAN routing was dead.

### Troubleshooting Methodology (OSI Model)

#### Phase 1: Layer 3 Configuration Verification

I first verified the IP configuration on all endpoints.

* ****Action:**** Checked `ip route` on Kali and `ipconfig` on Windows.

* ****Result:**** All IPs, Subnet Masks, and Default Gateways were correct. The issue was not a misconfiguration on the endpoints.

#### Phase 2: Gateway Analysis (Packet Capture)

Since the endpoints were configured correctly, I suspected the firewall was dropping traffic. So for the windows i off the firewall to be able to detect the problem, then i created a new rule for windows allowing ICMP connection on each device then i accessed the ****pfSense Diagnostics**** tab and ran a ****Packet Capture**** on the LAN interface while attempting to ping the gateway.

* ****Observation:**** The capture showed ICMP "Echo Requests" reaching the firewall.

* ****Error:**** There were ****zero**** "Echo
  Replies" sent back. The traffic was hitting a wall.

<img width="960" height="503" alt="window firwall rule on" src="https://github.com/user-attachments/assets/a446c803-47a6-4840-94ea-928bf9129be8" />

<img width="952" height="418" alt="Packet capture fail" src="https://github.com/user-attachments/assets/d57f793b-6cd5-4db1-866a-1c9ea057f5bb" />

<img width="960" height="465" alt="packet capture image" src="https://github.com/user-attachments/assets/7622af04-7bb7-4464-b63e-b409028b4710" />


#### Phase 3: Root Cause Analysis & Resolution

The packet capture confirmed that pfSense was receiving the
request but refusing to answer.

* ****Root Cause:**** pfSense operates on a ****"Default Deny"****  basis. Unlike consumer routers, it blocks all traffic on internal interfaces unless an explicit "Pass" rule exists.

* ****The Fix:**** I navigated to ****Firewall >
  Rules > LAN**** and created a new rule to allow traffic from `LAN Net` to
  `Any`.

<img width="960" height="505" alt="kali firewall rule" src="https://github.com/user-attachments/assets/2f126b1f-736c-46f1-b12d-915ccf860e7d" />

<img width="960" height="466" alt="Firewall rule for all connections" src="https://github.com/user-attachments/assets/7daf19d2-2451-443d-9e00-02dfdde45a13" />


### Final Verification

After applying the rule, I re-ran the connectivity tests from the Kali terminal.

* ****Result:**** Successful ICMP replies from the gateway. Network connectivity was fully restored.

<img width="959" height="505" alt="Windows 10 can now ping" src="https://github.com/user-attachments/assets/db826e7a-003c-469a-878c-dffb32498346" />

<img width="960" height="504" alt="Windows server 2022 can now ping" src="https://github.com/user-attachments/assets/1ded2d7c-642f-4fe9-97d0-8e1610ff8c21" />

<img width="958" height="433" alt="Kali can ping" src="https://github.com/user-attachments/assets/0c805c63-80ac-41e9-8587-a8d5d8633cc9" />


---

## Key Takeaways

1.  ****Visibility is Power:**** When the terminal says "Destination Unreachable," use the firewall's internal packet capture to see **where** the packet dies.

2.  ****Default Deny:**** Enterprise security tools assume everything is a threat until told otherwise. This was a valuable lesson in ACL (Access Control List) management.

3.  ****Isolation:**** Building this lab taught me how to architect secure, segmented networks that allow for safe malware testing.
