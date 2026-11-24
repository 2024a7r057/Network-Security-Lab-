# COM-511: Network Security Lab — Experiment 9
## Site-to-Site IPsec VPN in Tunnel Mode Using strongSwan (`swanctl`)

---

## 1. Objective

To configure and establish a Site-to-Site IPsec VPN using IKEv2 and strongSwan between two Linux virtual machines. The experiment demonstrates secure communication using ESP in Tunnel Mode, verifies the Security Associations (SAs), and analyzes encrypted network traffic using Wireshark.

## 2. Experiment Overview

In this experiment, I configured two Linux virtual machines as VPN gateways:

- **Gateway A:** Kali Linux (Initiator)
- **Gateway B:** Ubuntu Linux (Responder)

I used strongSwan with `charon-systemd` and `swanctl` to establish the VPN connection.

IKEv2 was used for authentication, key exchange and Security Association negotiation, while ESP (Encapsulating Security Payload) protected the actual network traffic.

The VPN protected communication between two simulated internal endpoints using loopback addresses:

- Gateway A: `10.10.1.1/32`
- Gateway B: `10.20.1.1/32`

The encrypted traffic travelled between the outer gateway addresses over the `192.168.159.0/24` network.

## 3. Network Topology and Addressing

```text
+-----------------------------------+                       +-----------------------------------+
|      GATEWAY A (Kali Linux)       |                       |     GATEWAY B (Ubuntu Linux)      |
|                                   |                       |                                   |
|  Inner Loopback (lo):             |                       |  Inner Loopback (lo):             |
|  10.10.1.1/32                     |                       |  10.20.1.1/32                     |
|         |                         |                       |         |                         |
|   [ Linux XFRM ]                  |                       |   [ Linux XFRM ]                  |
|         |                         |                       |         |                         |
|  Outer Interface (eth1):          |   ESP Tunnel (AES-GCM)|  Outer Interface (ens34):         |
|  192.168.159.140/24 ==============|=======================|============== 192.168.159.145/24  |
|  IKE ID: gw-a                     |   UDP 500/4500 & P:50 |  IKE ID: gw-b                     |
+-----------------------------------+                       +-----------------------------------+
```

### Addressing and Configuration Details

| Parameter | Gateway A (Initiator) | Gateway B (Responder) |
|---|---|---|
| Operating System | Kali Linux (`6.19.14+kali-amd64`) | Ubuntu Desktop |
| Outer Interface | `eth1` | `ens34` |
| Outer IP Address | `192.168.159.140/24` | `192.168.159.145/24` |
| Inner Protected Address | `10.10.1.1/32` | `10.20.1.1/32` |
| IKEv2 Identity | `gw-a` | `gw-b` |
| Authentication | Pre-Shared Key (PSK) | Pre-Shared Key (PSK) |
| IKE Proposal | `aes256gcm16-prfsha256-modp3072` | `aes256gcm16-prfsha256-modp3072` |
| ESP Proposal | `aes256gcm16` | `aes256gcm16` |
| VPN Mode | Tunnel Mode | Tunnel Mode |

### Cryptographic Parameters

| Component | Configuration |
|---|---|
| Key Exchange | IKEv2 |
| Encryption and Integrity | AES-256-GCM |
| Pseudorandom Function | PRF-HMAC-SHA-256 |
| Diffie-Hellman Group | MODP-3072 |
| Authentication | Pre-Shared Key |
| Data Protection Protocol | ESP |
| Encapsulation Mode | Tunnel Mode |

## 4. Implementation and Configuration

### Step 1: Installing strongSwan and Required Packages

I installed the required strongSwan components and `tcpdump` on the Linux systems.

```bash
sudo apt update
sudo apt install -y charon-systemd strongswan-swanctl tcpdump
```

I enabled and started the strongSwan service:

```bash
sudo systemctl enable --now strongswan
```

I checked the strongSwan daemon and its version:

```bash
sudo swanctl --version --daemon
```

The experiment used the modern VICI interface through `swanctl` instead of the older `ipsec.conf` configuration method.

### Step 2: Configuring the Inner Loopback Addresses

Since the experiment simulated communication between two protected internal networks, I assigned separate loopback addresses to both gateways.

**On Gateway A (Kali Linux):**

```bash
sudo ip address add 10.10.1.1/32 dev lo
ip -br address show dev lo
```

**On Gateway B (Ubuntu Linux):**

```bash
sudo ip address add 10.20.1.1/32 dev lo
ip -br address show dev lo
```

These addresses represented the internal endpoints that would communicate through the VPN tunnel.

### Step 3: Configuring the IPsec Connection

I configured the VPN connection using the `swanctl.conf` file located at:

```text
/etc/swanctl/swanctl.conf
```

A randomly generated Pre-Shared Key was created using:

```bash
openssl rand -base64 33
```

The same PSK was configured on both gateways. The `0s` prefix was used to indicate that the secret was Base64 encoded.

**Gateway A — Kali Linux configuration:**

```hcl
connections {
  lab-vpn {
    version = 2
    local_addrs = 192.168.159.140
    remote_addrs = 192.168.159.145
    proposals = aes256gcm16-prfsha256-modp3072

    local {
      auth = psk
      id = gw-a
    }

    remote {
      auth = psk
      id = gw-b
    }

    children {
      lab-net {
        local_ts = 10.10.1.1/32
        remote_ts = 10.20.1.1/32
        esp_proposals = aes256gcm16
      }
    }
  }
}

secrets {
  ike-lab {
    secret = 0s[REDACTED_BASE64_PSK]
  }
}
```

On Gateway B, I configured the corresponding connection with the local and remote gateway addresses, identities and traffic selectors reversed.

The configuration ensured that both gateways used matching IKEv2 and ESP proposals and the same authentication secret.

### Step 4: Loading the Configuration

After configuring the connection, I secured the configuration file permissions:

```bash
sudo chmod 600 /etc/swanctl/swanctl.conf
```

I loaded the connection and authentication credentials:

```bash
sudo swanctl --load-all
```

I verified that the VPN connection was available:

```bash
sudo swanctl --list-conns
```

### Step 5: Initiating the VPN Connection

Before establishing the tunnel, I started capturing packets on Gateway A's outer network interface.

```bash
sudo tcpdump -ni eth1 -w exp9-ipsec.pcap 'udp port 500 or udp port 4500 or proto 50 or proto 51'
```

I kept the capture running and used another terminal to initiate the CHILD Security Association:

```bash
sudo swanctl --initiate --child lab-net
```

The VPN connection was successfully established between the two gateways.

## 5. Verification and Results

After initiating the VPN, I performed several checks to verify that the tunnel was established and protected traffic was successfully exchanged.

### A. Security Association Verification

I used the following command to inspect the established Security Associations:

```bash
sudo swanctl --list-sas
```

The output showed the following results:

**IKE Security Association (lab-vpn):**
- Status: `ESTABLISHED`
- Protocol: IKEv2
- Encryption: `AES_GCM_16-256`
- PRF: `PRF_HMAC_SHA2_256`
- Diffie-Hellman Group: `MODP_3072`
- Gateway A: `192.168.159.140 (gw-a)`
- Gateway B: `192.168.159.145 (gw-b)`

**CHILD Security Association (lab-net):**
- Status: `INSTALLED`
- Mode: Tunnel Mode
- Protocol: ESP
- Encryption: `AES_GCM_16-256`

The observed SPI values were:

| Direction | SPI | Traffic Statistics |
|---|---|---|
| Inbound | `0xc6ec928a` | 672 bytes, 8 packets |
| Outbound | `0xc807675e` | 672 bytes, 8 packets |

These results confirmed that the IKE and CHILD Security Associations were successfully established and installed.

### B. Linux XFRM State and Policy Verification

I used the Linux XFRM commands to check the kernel-level IPsec configuration.

**To view the Security Association states:**

```bash
sudo ip xfrm state
```

**To view the installed security policies:**

```bash
sudo ip xfrm policy
```

The output showed that the Linux kernel had installed the required ESP states and traffic-selection policies.

The outbound policy was:

```text
src 10.10.1.1/32
dst 10.20.1.1/32
dir out
```

It directed matching traffic through the ESP tunnel using the outer gateway addresses:

```text
src 192.168.159.140
dst 192.168.159.145
proto esp
spi 0xc807675e
mode tunnel
```

The inbound policy used the reverse addresses to identify and process incoming protected traffic.

This confirmed that the Linux XFRM subsystem was handling the IPsec packet processing at the kernel level.

### C. Protected Traffic Connectivity Test

To verify communication between the two protected endpoints, I used the following command on Gateway A:

```bash
ping -I 10.10.1.1 -c 4 10.20.1.1
```

**Observed Result:**

```text
4 packets transmitted
4 packets received
0% packet loss
```

All four ICMP packets were successfully received.

I used `-I 10.10.1.1` to specify the source address so that the packets matched the configured IPsec traffic selector.

This confirmed successful communication between the two internal endpoints through the VPN tunnel.

### D. Wireshark Packet Capture Analysis

I opened the captured file `exp9-ipsec.pcap` in Wireshark to examine the packets exchanged during VPN establishment and data transmission.

I used the following display filter:

```text
isakmp || esp || ah
```

The captured traffic showed three main stages:

**1. IKE_SA_INIT (Frames 1–2)**

- The gateways exchanged IKEv2 negotiation messages over UDP port 500.
- Cryptographic proposals and nonces were exchanged.
- Diffie-Hellman key exchange using MODP-3072 was performed.
- The initial keying material was established.

**2. IKE_AUTH (Frames 3–4)**

- The gateways authenticated each other using the configured Pre-Shared Key.
- Their IKE identities (`gw-a` and `gw-b`) were authenticated.
- The CHILD Security Association for protected traffic was negotiated.
- The authentication exchange was encrypted.

**3. ESP Data Transmission (Frames 5–20)**

- A total of 16 ESP data frames were observed.
- The packets travelled between outer IP addresses `192.168.159.140` and `192.168.159.145`.
- Each observed frame was 154 bytes.
- ESP used IP Protocol 50.
- The SPI values and sequence numbers remained visible.
- The original inner IP addresses (`10.10.1.1` and `10.20.1.1`) and ICMP payload were not readable in the encrypted packet capture.

**AH Filter Observation:**

I also checked the following display filter:

```text
ah
```

No AH packets were observed because the experiment used ESP rather than AH. AES-GCM provided both encryption and integrity protection for the ESP traffic.

### E. Final Verification Summary

| Verification | Observed Result |
|---|---|
| strongSwan service | Running |
| Configuration loading | Successful |
| IKEv2 negotiation | Established |
| CHILD Security Association | Installed |
| ESP Tunnel Mode | Active |
| XFRM states and policies | Installed |
| ICMP connectivity | Successful |
| Packet loss | 0% |
| ESP traffic in Wireshark | Observed |
| AH traffic | Not observed |

## 6. Environment Cleanup

After completing the experiment and collecting the required results, I terminated the CHILD Security Association and removed the temporary loopback address.

**On Gateway A:**

```bash
sudo swanctl --terminate --child lab-net
sudo ip address del 10.10.1.1/32 dev lo
```

**On Gateway B:**

```bash
sudo ip address del 10.20.1.1/32 dev lo
```

This removed the temporary network configuration used for the experiment.

## 7. Conclusion

I successfully configured and established a Site-to-Site IPsec VPN in Tunnel Mode between Kali Linux and Ubuntu using strongSwan and IKEv2. The gateways authenticated each other using a Pre-Shared Key and negotiated the required IKE and ESP Security Associations.

The successful ICMP connectivity test confirmed that the protected endpoints could communicate through the VPN tunnel. I also verified the installed XFRM states and policies and analyzed the captured packets in Wireshark. The packet capture demonstrated the IKEv2 negotiation process and the use of ESP to protect the actual data traffic.

Through this experiment, I understood how IPsec provides secure communication between two networks using authentication, key exchange, encryption and integrity protection.
