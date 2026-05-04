# TRYHACKME WRITEUP TEMPLATE

---

## ROOM NAME: Nmap

## DATE: 4 May 2026

## PLATFORM: TryHackMe

---

## 1. TOPIC OVERVIEW

This TryHackMe room explores the base concept of Nmap (Network Mapper) in depth. Showcasing practical use cases over Basic Nmap Scanning and Enumeration for Reconnaissance of a Network.

---

## 2. KEY CONCEPTS

* Concept 1: Basic Nmap Scans (TCP,UDP,ICMP and Stealth Scanning using Nmap)
* Concept 2: Introduction and Walkthrough - Nmap Scripting Engine (NSE)
* Concept 3: Firewall/IDS/IPS Evasion

---

## 3. IMPORTANT COMMANDS

### Command:

```bash
nmap -sn <target>
```

(ICMP Network Scanning)

**Explanation:**

* sn --> Ping Scan (-sP in past)
  
Standard, fast Nmap Scan without involving ports.
Can be used with Both CIDR(0/24) [Includes .0 and .255] and Octet Range Addressing(1-254) [Skips .0 and .255]
  In this scan, nmap pings every host with ICMP / ARP(if locally on network) / (TCP/ACK) packet.

**Example Output:**
<img width="537" height="144" alt="image" src="https://github.com/user-attachments/assets/a3d86a10-681e-4ab7-8ae2-ef5da95f37b6" />


---

### Command:

```bash
nmap -sT <target>
```

**Explanation:**
The standard TCP connect Scan. If the port's open, it'll respond with SYN-ACK. After the 3 Way Handshake is done, the Target must send RST packet to end the Communication. But it's worth it to note that an RST packet will be sent if the port is closed before handshake is completed.

**Example Output:**
<img width="522" height="198" alt="image" src="https://github.com/user-attachments/assets/c4faf467-d042-47b6-8ee6-71a09f8fe2c4" />

---

### Command:

```bash
nmap -sS <target>
```

**Explanation:**
Used for TCP SYN Scan (Half Open / Stealth Scan). Used for Scanning network in stealth mode. (SYN,SYN-ACK,RST)
Though modern firewalls can prevent these scans and send back RST flag.
If no response is received, port is filtered.

**Example Output:**
<img width="525" height="189" alt="image" src="https://github.com/user-attachments/assets/bf3893c0-4300-4e6c-832d-bf999d6194c5" />


---

### Command:

```bash
nmap -sU <target>
```

**Explanation:**
The standard UDP scanning. This type of scan sends packets blindly on the target and waits for a response. If no response is received, Open|Filtered
If a UDP response is received(very unusual), then the port is surely open. If not, another request's sent, for a double check. A closed port will respond with ICMP packet.

Very Slow Scan. Hence, only top ports are scanned using --top-ports <number>

**Example Output:**
<img width="528" height="189" alt="image" src="https://github.com/user-attachments/assets/61534d57-ffae-4f83-aa4a-64ef5d2e2c4f" />


---

### Command:

```bash
nmap -p 21 --script=ftp-anon.nse <target>
```

**Explanation:**
This command scans the port number 21 of the target to check if the service allows anonymous FTP logins. This is done using a script provided by the Nmap Scripting Engine (NSE) named ftp-anon.nse
This command shows scripting in Nmap using NSE.

**Example Output:**
<img width="543" height="201" alt="image" src="https://github.com/user-attachments/assets/fe3b9b88-64db-41dc-b250-d9b0c254ebb1" />


---

## 4. COMMONLY USED SWITCHES

* -sC → scans using default scripts

* -sV → Attempts to detect version of the service

* -O → Attempts to detect Operating system the target is running on. (smb-os- discovery.nse can also be used for this)

* -v/ -vv/ -vvv → Increases verbosity of the scan as nmap often doesn't provide enough information for pentesting

* -A → Aggressive mode (Loud). Provides faster, better results but at cost of being detected.

* -T0/1/2/3/4/5 → Time based control switch. We can set Parallelism, Delay of packets transmission using these switches.

---

## 5. PRACTICAL TASKS

**What you did:**

1. Pinged Target with an ICMP + ARP(if localhost) + (TCP/ACK) packets. This step was very important to know if host is up. 1 Host was Up

Attach Screenshot Of the Scan

2. An Xmas Scan (using -sX) on the first 999 ports. All the ports were open|filtered

3. TCP SYN Scan on the first 5000 Ports (using -sS). 5 Ports were meant to be open. But in this scenario, the attack machine shows all filtered.

4. Scanned Port 21 using ftp-anon.nse in nmap while closely monitoring the traffic using Wireshark. The communication with Port 21 failed repeatedly even though the anonymous login should've been possible as the room suggests.

---

### Tools Used:

* Nmap - For Scanning the network
* Burp Suite - For monitoring network traffic during Nmap Scans.

---

## 6. ATTACK FLOW (IF APPLICABLE)

**Step-by-step:**

1. Scan target - The target was scanned for open ports using nmap, nmap's ftp-anon.nse script

2. Identify service - The service was identified on Port 21 and it did allow anonymous logins

3. Exploit vulnerability - Not applicable (Recon phase only)

---

## 7. MISTAKES / CONFUSIONS

* There was confusion between TCP Syn Scan and ICMP Scan as both their switches were similar (-sS and -sn) even though both of the scans are very different.

* The Target machine was not showing Open ports as specified in the documentation given my TryHackMe, but the conclusion was it was showing Open|Filtered.

* Initially, my mindset revolved around Nmap --> Attack Tool. The room cleared the use case of Nmap is not attack, it's Reconnaissance.

* Got confused between --badsum and --data-length.

---

## 8. KEY LEARNINGS

* Learnt the fundamental use of Nmap, NSE(Nmap Scripting Engine), fundamentals of firewall/IDS/IPS evasion. This room builds confidence with keyboard over Nmap.

* Nmap scans are very important for gathering intel over the target before performing any attacks. This will increase the chances of success. The scans reveal a lot of information such as Open Ports, Active Services on the target.

* For an instance, the attacker might need to assess if the firewall configured over the network is Time based. They can use -T0/1/2/3/4/5 switch to assess the results. A time based firewall would act differently based on time.

---

## 8. QUICK NOTES (REVISION)

* Nmap is a Reconnaissance Tool

* Fundamental Scans include TCP Scans, UDP Scans, ICMP Scans

* TCP Scans build up a connection between the target and attacker using a handshake, UDP sends packet to the target without establishing the connection and waits for a response.

* Types Of TCP Scans - TCP CONNECT SCANS (-sT), SYN STEALTH SCANS(-sS)

* More Stealth Scans - NULL[-sN], FIN[-sF], Xmas Scan[-sX] (Shows Xmas tree in Wireshark)

* ICMP Network Scanning - -sn, works on both CIDR notation and Octet Range Addressing

* NSE(Nmap Scripting Engine) - Powerful addition to Nmap, All scripts written in lua programming language.

* Searching of scripts should be done in "script.db" stored in /usr/share/nmap/scripts/script.db

* Searching of scripts can be done using - grep and ls -l

* "--badsum" adds invalid checksum after each data packet sent by Nmap. Pre-configured firewalls would automatically respond to them. But any real TCP/IP Stack would drop the packet instead of responding. Detects presence of firewall configuration over the network.

* Nmap Switch allowing to append arbitrary length of random data to the end of packets is --data-length. It can be used to bypass firewalls which have default filters to bypass  packets sent by Nmap by sending RST for each one of them. Using this might let us bypass the firewall restrictions and get our results.

---

## 9. REAL WORLD INSIGHT

* Why SYN scan is stealthy? - Where TCP scans perform a full three-way handshake with the Target, SYN scans send back RST TCP packet after receiving SYN-ACK from the target. Similarly, a NULL scan sends TCP packet with NO flag, FIN scan sends packet with FIN flag, Xmas Scan Sends malformed, mix of different types of flags containing TCP Packet.

* UDP Scans Are Unreliable - If there is no response from the sent UDP packet, the port is either filtered by firewall, OR open. This information is highly unreliable for further attacks. UDP packets are highly unlikely to give response for the confirmation of an Open Port.

* Usage of Time Switch (T) is underrated:
  -T0 - Paranoid - Very-Slow - IDS-Invasion
  -T1 - Sneaky - Slow - IDS-Invasion
  -T2 - Polite - Slightly Slow - For unstable, fragile systems
  -T3 - Normal - Default Settings.
  -T4 - Aggressive - Fast - Only for reliable, strong networks. Slow, unstable networks will lead to unreliable results and missed scanning on ports.
  -T5 - Insane - Very Fast - Only for very fast networks and when speed matters more than accuracy. This scan is not reliable and must be used responsibly.

---
