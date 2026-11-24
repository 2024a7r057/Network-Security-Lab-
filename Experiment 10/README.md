# COM-511: Network Security Lab — Experiment 10
## Firewall and Intrusion Detection using pfSense & Snort

### 1. Objective
To configure pfSense as a network firewall and Snort as an Intrusion Detection System (IDS). The experiment tests firewall filtering and detection of port scans, FTP brute-force attempts, and unusual ICMP traffic.

### 2. Lab Environment

The experiment uses an isolated virtual network (`192.168.1.0/24`) with three virtual machines.

| System | Interface / Role | IP Address |
|---|---|---|
| pfSense 2.8.1 | WAN (`em0`) | `10.0.2.15` |
| pfSense 2.8.1 | LAN (`em1`) | `192.168.1.1/24` |
| Kali Linux | Attacker | `192.168.1.10/24` |
| Metasploitable 2 | Target | `192.168.1.20/24` |

**Network flow:** Kali Linux → pfSense + Snort → Metasploitable 2

### 3. Configuration

**A. pfSense Setup**
1. Assigned `em0` as WAN (NAT) and `em1` as LAN.
2. Configured the LAN IP as `192.168.1.1/24`.
3. Enabled DHCP with the range `192.168.1.100–192.168.1.200`.
4. Accessed the web interface at `https://192.168.1.1`.

**B. Firewall Rules**

Configured the rules under **Firewall → Rules → LAN**.

| Action | Protocol | Source | Destination Port | Purpose |
|---|---|---|---|---|
| Pass | Any | Any | 80, 443 | Anti-lockout rule |
| Pass | TCP | LAN net | 80 | Allow HTTP |
| Pass | TCP | LAN net | 443 | Allow HTTPS |
| Pass | UDP | LAN net | 53 | Allow DNS |
| Block | Any | LAN net | Any | Block remaining traffic |

**C. Snort IDS Setup**
1. Installed Snort through **System → Package Manager → Available Packages**.
2. Added the LAN (`em1`) interface under **Services → Snort**.
3. Enabled Emerging Threats / Community rulesets.
4. Started the Snort detection service.

### 4. Testing and Results

**A. Firewall Testing**

Allowed HTTP connection:

```bash
curl -I http://192.168.1.20
```

**Result:** `HTTP/1.1 200 OK` — HTTP access was successful.

Blocked Telnet connection:

```bash
telnet 192.168.1.20 23
```

**Result:** Connection timed out. The firewall log was checked under **Status → System Logs → Firewall**.

**B. Snort IDS Testing**

The following tests were performed from Kali Linux against Metasploitable 2.

| Test | Command | Observation |
|---|---|---|
| Port scanning | `nmap -p 1-1000 192.168.1.20` | Snort generated `[ET SCAN]` alerts. |
| FTP brute force | `hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ftp://192.168.1.20` | Snort generated `[ET POLICY]` FTP brute-force alerts. |
| ICMP flood | `ping -f 192.168.1.20` | Snort flagged unusually high ICMP traffic. |

Alerts were checked under **Services → Snort → Alerts**, where the source and destination IP addresses, timestamps, and rule information could be observed.

### 5. Conclusion
In this experiment, I configured pfSense firewall rules to control network traffic and installed Snort to detect suspicious activities. HTTP access was successful, while the Telnet connection was restricted. Snort generated alerts for port scanning, FTP brute-force attempts, and high-rate ICMP traffic. This demonstrated how a firewall and IDS provide different but complementary layers of network security.
