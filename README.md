# Home Lab Infrastructure

> **Documentation Notice:** All IP addresses, hostnames, and network information shown in this document are fictional and used for documentation purposes only.

## Overview

This home lab provides a segmented environment for networking, cybersecurity, virtualization, Linux administration, DNS, VPNs, and network storage.

The infrastructure is built around:

- **pfSense** — firewall, router, and WireGuard VPN gateway
- **Snort** — intrusion detection and traffic monitoring
- **Proxmox** — virtualization platform
- **Debian Container** — Pi-hole host
- **Pi-hole** — local DNS and ad/tracker blocking
- **FSRV NAS** — network storage and SMB file sharing
- **Network Switch** — wired connectivity
- **TP-Link Access Point** — wireless connectivity
- **Desktop and laptops** — administration and client devices

---

## Network Architecture

The lab network is separated from the upstream/home network by pfSense.

### Upstream Network

```text
Network:       172.20.10.0/24
Gateway:       172.20.10.1
pfSense WAN:   172.20.10.254/24
```

### Lab Network

```text
Network:       10.99.50.0/24
pfSense LAN:   10.99.50.1/24
```

pfSense acts as the default gateway for the lab network and routes traffic between the LAN and the upstream network.

### Physical Topology

```text
                         Internet
                            |
                       Home Router
                            |
                         pfSense
                            |
                         Switch
            +---------------+---------------+
            |               |               |
         Proxmox         FSRV NAS         Desktop
            |
      Debian Container
            |
         Pi-hole

                         Switch
                            |
                    TP-Link AP (AP Mode)
                            |
                          Wi-Fi
                       +----+----+
                       |         |
                    Laptop 1  Laptop 2
```

The desktop is connected directly to the switch using Ethernet.

The laptops connect through the TP-Link Access Point configured in AP mode.

---

# pfSense

pfSense provides the primary firewall, routing, and VPN functionality.

## Interface Configuration

| Interface | Network | Address | Purpose |
|---|---|---|---|
| WAN | `172.20.10.0/24` | `172.20.10.254/24` | Upstream network |
| LAN | `10.99.50.0/24` | `10.99.50.1/24` | Lab network |

### WAN

```text
Interface:        igc0
IP Address:       172.20.10.254/24
Default Gateway:  172.20.10.1
```

### LAN

```text
Interface:        igc1
IP Address:       10.99.50.1/24
Default Gateway:  None
```

The LAN interface does not require a separate default gateway because pfSense performs the routing between the LAN and WAN interfaces.

---

## Firewall Rules

The lab network is allowed to initiate outbound connections:

```text
Source:       10.99.50.0/24
Destination:  Any
Protocol:     Any
Action:       Allow
```

Inbound traffic from the WAN to the LAN is explicitly denied:

```text
Source:       WAN / Any
Destination:  LAN / Any
Protocol:     Any
Action:       Deny
```

This prevents unsolicited inbound connections from the upstream network from directly accessing hosts on the lab LAN.

The firewall is stateful, so return traffic for connections initiated from the LAN is allowed according to the firewall state.

### Traffic Policy

```text
                    WAN
                     |
                     v
                 +--------+
                 | pfSense|
                 +--------+
                     |
          +----------+----------+
          |                     |
       Outbound              Inbound
       LAN -> WAN             WAN -> LAN
          |                     |
        ALLOW                  DENY
```

---

# Snort IDS

[Snort](https://www.snort.org/) is installed on pfSense and provides additional traffic monitoring and intrusion detection.

Snort can generate alerts for traffic matching configured detection rules, including:

- Port scanning
- Network reconnaissance
- Exploit attempts
- Known malicious traffic patterns
- Other suspicious activity

Snort complements the firewall but does not replace firewall rules.

```text
Network Traffic
      |
   pfSense
      |
    Snort
      |
   Alerts
```

---

# WireGuard VPN

WireGuard is configured on pfSense and provides VPN connectivity.

The configuration was tested for:

- Tunnel establishment
- Routing
- DNS resolution
- Connectivity
- External IP address
- Firewall behavior

Sensitive information such as private keys, endpoints, and authentication data is excluded from this documentation.

Example sanitized configuration:

```ini
[Interface]
PrivateKey = <REDACTED>
Address = <FICTIONAL_VPN_ADDRESS>

[Peer]
PublicKey = <REDACTED>
Endpoint = <REDACTED>
AllowedIPs = 0.0.0.0/0
```

## WireGuard Installation Issue

During the initial configuration, the WireGuard package was unavailable through the pfSense package manager because of a repository issue.

The repository information was checked using:

```bash
pkg-static info pfSense-repo
```

The repository package was then installed:

```bash
pkg-static install pfSense-repo
```

After resolving the repository issue, the WireGuard package became available and was installed successfully.

---

# Proxmox

Proxmox is used as the virtualization platform for infrastructure services.

The current environment contains a Debian container dedicated to Pi-hole.

```text
Proxmox
   |
Debian Container
   |
Pi-hole
```

## Proxmox Repository Issue

The Proxmox server initially attempted to use the Enterprise repositories without an active subscription.

Running:

```bash
apt-get update
```

resulted in errors similar to:

```text
401 Unauthorized
```

The Enterprise repositories were disabled and the no-subscription repository was configured for the home lab.

Example configuration:

```text
http://download.proxmox.com/debian/pve
Suite: trixie
Component: pve-no-subscription
```

After the repository configuration was changed:

```bash
apt-get update
```

completed successfully.

---

# Pi-hole

Pi-hole runs inside a dedicated Debian container on Proxmox.

It provides:

- Local DNS
- Internal hostname resolution
- Advertisement blocking
- Tracker blocking
- DNS-based filtering

## DNS Architecture

```text
Client
   |
   | DNS Query
   v
Pi-hole
   |
   +---- Local DNS
   |
   +---- Upstream DNS
```

Clients configured to use Pi-hole send their DNS queries to the service over the lab network.

Pi-hole provides centralized DNS filtering without requiring a separate DNS service on each client.

---

# FSRV NAS

**FSRV** provides centralized network storage for the home lab.

The NAS is connected to the main network switch and provides storage independently from the compute resources hosted by Proxmox.

```text
                 Network Switch
                       |
                    FSRV NAS
                       |
                Network Storage
```

FSRV is used for:

- Centralized file storage
- Shared files
- Network file sharing
- Storage for authorized clients

---

# Samba / SMB

Samba is configured on FSRV to provide authenticated SMB network file sharing.

```text
Client
   |
   | SMB
   v
FSRV NAS
   |
Samba / SMB Share
```

Example sanitized configuration:

```text
Server:         <FICTIONAL_NAS_HOSTNAME>
Share:          <FICTIONAL_SHARE_NAME>
Protocol:       SMB
Authentication: User-based
Access:         Restricted
```

A Linux client can access the share using `smbclient`:

```bash
smbclient //<FICTIONAL_NAS_HOSTNAME>/<FICTIONAL_SHARE_NAME> -U <USERNAME>
```

The share can also be mounted using CIFS:

```bash
sudo mount -t cifs //<FICTIONAL_NAS_HOSTNAME>/<FICTIONAL_SHARE_NAME> /mnt/shared \
  -o username=<USERNAME>
```

Credentials are not included in public documentation.

---

# Network Connectivity

The network switch provides wired connectivity for the main infrastructure components.

```text
Switch
├── pfSense
├── Proxmox
├── FSRV NAS
├── Desktop
└── TP-Link Access Point
      └── Wi-Fi Clients
```

The TP-Link device operates in **Access Point mode**.

Routing and firewall functions remain on pfSense.

---

# Security Architecture

The infrastructure uses multiple security layers.

| Component | Function |
|---|---|
| pfSense | Firewall and routing |
| WAN → LAN deny rule | Blocks unsolicited inbound traffic |
| Firewall Rules | Network access control |
| Snort | Intrusion detection |
| WireGuard | VPN connectivity |
| Pi-hole | DNS filtering and ad/tracker blocking |
| Proxmox | Virtualization and service isolation |
| Samba Authentication | Controlled file access |

### Security Flow

```text
                         Internet
                            |
                       Home Router
                            |
                         pfSense
                            |
                     Firewall Rules
                            |
                  +---------+---------+
                  |                   |
                Snort                 LAN
                                      |
                    +-----------------+----------------+
                    |                 |                |
                 Proxmox            FSRV            Clients
                    |                                  |
              Debian Container                     Desktop
                    |                                  |
                 Pi-hole                           Laptops
```

---

# Current Infrastructure Status

| Component | Status |
|---|---|
| pfSense | Operational |
| WAN/LAN Routing | Operational |
| Firewall | Configured |
| WAN → LAN Deny Rule | Configured |
| Snort | Operational |
| WireGuard | Operational |
| Proxmox | Operational |
| Debian Container | Operational |
| Pi-hole DNS | Operational |
| Pi-hole Filtering | Operational |
| FSRV NAS | Operational |
| Samba / SMB | Operational |
| Network Switch | Operational |
| TP-Link Access Point | Operational |
| Desktop Connectivity | Operational |
| Laptop Wi-Fi Connectivity | Operational |

---

# Sensitive Information

The following information is intentionally excluded from this documentation:

- Real internal IP addresses
- Real public IP addresses
- MAC addresses
- Wi-Fi passwords
- VPN private keys
- API tokens
- User passwords
- NAS credentials
- Proxmox tokens
- Sensitive hostnames
- Device serial numbers
- Authentication files

Documentation uses fictional values such as:

```text
172.20.10.0/24
10.99.50.0/24
<FICTIONAL_HOSTNAME>
<FICTIONAL_NAS_HOSTNAME>
<FICTIONAL_SHARE_NAME>
<FICTIONAL_VPN_ADDRESS>
<REDACTED>
```

---

# Skills Demonstrated

This home lab provides hands-on experience with:

- Network segmentation
- Firewall configuration
- Routing
- WAN/LAN security
- VPN configuration
- Intrusion detection
- Linux administration
- Virtualization
- DNS
- DNS filtering
- Network storage
- SMB/Samba
- Network troubleshooting
- Infrastructure management
- Security monitoring

---

# Summary

The home lab is a segmented infrastructure environment built around pfSense.

**pfSense** provides firewalling, routing, and WireGuard VPN connectivity. An explicit **deny-all WAN-to-LAN rule** prevents unsolicited inbound traffic from reaching the lab network.

**Snort** provides additional intrusion detection and network traffic monitoring.

**Proxmox** provides the virtualization layer and hosts a dedicated Debian container running **Pi-hole**.

**Pi-hole** provides local DNS resolution and DNS-based advertisement and tracker blocking.

**FSRV** provides centralized network storage through **Samba/SMB**.

The network switch provides wired connectivity, while the **TP-Link Access Point** provides wireless connectivity for laptop clients.

The resulting environment provides a practical platform for learning and demonstrating networking, cybersecurity, virtualization, Linux administration, DNS, VPNs, storage, and infrastructure management.
