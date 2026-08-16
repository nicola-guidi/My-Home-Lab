# Home Lab Infrastructure Report

> **Documentation Notice:** All IP addresses and network addresses shown in this document are fictional and are used exclusively for documentation purposes. They do not represent the actual network configuration.

---

# 1. Overview

This document describes the current architecture, configuration, and implementation of my home lab infrastructure.

The main objectives of the home lab are:

* Network security and segmentation
* Firewall and routing
* Intrusion detection and traffic monitoring
* Virtualization
* Network storage
* Local DNS
* Advertisement and tracker blocking
* VPN connectivity
* Infrastructure management
* Linux and system administration
* Testing and learning new technologies

The infrastructure is built around:

* A pfSense firewall appliance
* A network switch
* A Proxmox virtualization server
* A NAS
* A TP-Link wireless router configured in AP mode
* A Debian container running Pi-hole
* A desktop workstation
* Laptop clients

The desktop workstation is connected directly to the network switch using Ethernet.

The laptops connect wirelessly through the TP-Link Access Point.

---

# 2. Infrastructure Components

| Component | Role | Status |
| --- | --- | --- |
| pfSense Firewall Appliance | Firewall, router, and VPN gateway | Operational |
| Snort | Intrusion detection and traffic monitoring | Operational |
| Network Switch | Wired network connectivity | Operational |
| Proxmox Server | Virtualization platform | Operational |
| Debian Container | Container for Pi-hole | Operational |
| Pi-hole | Local DNS server and ad blocker | Operational |
| NAS | Network file server | Operational |
| Samba / SMB | Network file sharing | Operational |
| TP-Link Access Point | Wireless connectivity | Operational |
| Desktop Workstation | Administration / client | Operational |
| Laptops | Wireless client devices | Operational |

---

# 3. High-Level Architecture

The current physical and logical architecture is:

```text
                              Internet
                                  |
                           Home Router
                                  |
                              pfSense
                                  |
                               Switch
                 +----------------+----------------+----------------+
                 |                |                |                |
                 |                |                |                |
              Proxmox            NAS             Desktop          TP-Link 
               Server                          Workstation     Access Point
                 |                                                  |  
           Debian Container                                       Wi-Fi
                 |                                                  |
              Pi-hole                                        +------+------+
                                                             |             |
                                                           Laptop 1      Laptop 2                            

```

Pi-hole is a network service running inside the Debian container on Proxmox.

It provides DNS services to network clients but is not a physical network device and is not positioned as a network hop between clients and the switch.

The desktop workstation is physically connected directly to the switch.

The laptops connect to the TP-Link Access Point using Wi-Fi.

The TP-Link device operates in **Access Point mode** and does not perform the primary routing or firewall functions.

---

# 4. Network Architecture

The home lab uses separate networks for the upstream/home environment and the secure lab environment.

The addresses below are **fictional**.

## 4.1 Upstream / Home Network

```text
Network:       172.20.10.0/24
Gateway:       172.20.10.1
pfSense WAN:   172.20.10.254
```

This network represents the upstream network in front of the pfSense firewall.

The upstream home router provides Internet connectivity.

The pfSense WAN interface is connected to this network.

---

## 4.2 Secure Lab Network

```text
Network:       10.99.50.0/24
pfSense LAN:   10.99.50.1
```

This is the secure laboratory network behind pfSense.

The pfSense LAN interface acts as the default gateway for devices connected to the lab network.

The LAN interface does not have a separate default gateway because pfSense performs the routing between the LAN and WAN interfaces.

---

# 5. pfSense Firewall

## 5.1 Installation

The firewall appliance was installed using pfSense.

The installation process was:

1. Download the pfSense installation image.
2. Prepare the installation media.
3. Write the image to a USB drive using balenaEtcher.
4. Boot the firewall appliance from the USB drive.
5. Install pfSense on the hardware appliance.
6. Assign the network interfaces.
7. Configure the WAN and LAN networks.
8. Configure the default gateway.
9. Change the administrator password.
10. Verify network connectivity.

---

## 5.2 Interface Assignment

The pfSense interfaces were configured as follows:

| Interface | Physical Interface | Network | Fictional Address | Purpose |
| --- | --- | --- | --- | --- |
| WAN | `igc0` | `172.20.10.0/24` | `172.20.10.254/24` | Upstream/home network |
| LAN | `igc1` | `10.99.50.0/24` | `10.99.50.1/24` | Secure lab network |

### WAN Configuration

```text
Interface:        igc0
IP Address:       172.20.10.254/24
Default Gateway:  172.20.10.1
```

### LAN Configuration

```text
Interface:        igc1
IP Address:       10.99.50.1/24
Default Gateway:  None
```

The LAN interface does not have a directly configured default gateway because pfSense is responsible for routing traffic from the lab network towards the WAN.

---

## 5.3 Initial pfSense Configuration

After installation, the following configuration was completed:

* WAN interface assignment
* LAN interface assignment
* WAN IP configuration
* LAN IP configuration
* Default gateway configuration
* Firewall configuration
* Administrator password change
* Secure password storage
* Internet connectivity testing
* VPN connectivity testing

The administrator password was changed from the default value and stored securely in a password manager.

---

## 5.4 Connectivity Testing

The following functionality was verified:

* pfSense management access
* LAN connectivity
* WAN connectivity
* Internet access
* Routing between the lab network and upstream network
* VPN connectivity

The lab network can access the Internet through pfSense.

---

# 6. Firewall Rules

Firewall rules are used to control traffic between the secure lab network and the upstream network.

The general traffic flow is:

```text
Internet
   |
Home Router
   |
172.20.10.0/24
   |
pfSense
   |
10.99.50.0/24
   |
Secure Lab Network
```

A basic outbound rule allows hosts on the lab network to access external networks:

```text
Source:       10.99.50.0/24
Destination:  Any
Protocol:     Any
Action:       Allow
Purpose:      Allow lab hosts to access external networks
```

The pfSense firewall is responsible for enforcing network access policies between the lab network and the upstream network.

---

# 7. Snort Intrusion Detection

The **Snort** package was installed and enabled on the pfSense firewall.

Snort is used as a network intrusion detection and traffic monitoring system to identify potentially suspicious or malicious network activity.

The purpose of deploying Snort in the home lab is to provide additional visibility into network traffic.

The current security architecture is:

```text
                         Internet
                            |
                            v
                      +-----------+
                      |  pfSense  |
                      |  Firewall |
                      +-----------+
                            |
                     Firewall Rules
                            |
                            v
                          Snort
                     Traffic Monitoring
                            |
                            v
                     Secure Lab Network
```

Snort can monitor network traffic and generate alerts when traffic matches configured detection rules.

Potential events detected by configured rules can include:

* Suspicious network traffic
* Known attack patterns
* Port scanning activity
* Network reconnaissance
* Exploit attempts
* Malicious traffic signatures
* Other traffic matching configured IDS rules

Snort provides additional visibility into network activity and can be used to investigate unusual events within the lab.

---

## 7.1 Snort Monitoring Workflow

The monitoring workflow is:

```text
Network Traffic
      |
      v
   pfSense
      |
      v
    Snort
      |
      v
Detection Rules
      |
      v
   Alerts
      |
      v
Security Investigation
```

Snort is used as a detection and monitoring layer rather than a replacement for firewall rules.

The pfSense firewall remains responsible for enforcing network access policies, while Snort provides additional visibility into traffic patterns and potential threats.

---

# 8. WireGuard VPN

A VPN was configured on the pfSense firewall using WireGuard.

The purpose of the VPN configuration is to provide secure VPN connectivity through the firewall.

A WireGuard configuration file was generated and used during the setup.

The configuration contains sensitive information, including:

* Private keys
* Public keys
* VPN addresses
* VPN endpoints
* DNS configuration
* Allowed IPs

Sensitive information is intentionally excluded from this report.

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

The actual configuration file is stored securely and is not included in public documentation.

---

# 9. WireGuard Package Installation Issue

During the initial WireGuard configuration, the WireGuard package could not be installed through the pfSense package manager.

The following error was encountered:

```text
Unable to retrieve package information
```

The issue was investigated through the pfSense CLI.

The repository information was checked using:

```bash
pkg-static info pfSense-repo
```

The pfSense repository package was then installed:

```bash
pkg-static install pfSense-repo
```

After resolving the repository issue, the WireGuard package became available through the pfSense package manager.

The WireGuard package was subsequently installed successfully.

---

# 10. WireGuard VPN Configuration

The WireGuard configuration was based on the Proton VPN pfSense WireGuard documentation.

The VPN configuration is operational.

The general architecture is:

```text
                    VPN Provider
                         |
                    WireGuard
                         |
                      pfSense
                         |
                  Secure Lab Network
                         |
                 +-------+-------+
                 |               |
              Servers          Clients
```

The VPN configuration was tested for:

* Tunnel establishment
* Connectivity
* DNS resolution
* Routing
* External IP address
* Firewall behavior

---

# 11. Proxmox Virtualization Server

A Proxmox server was installed and added to the infrastructure as the virtualization platform.

Proxmox is used to host virtual machines and containers for infrastructure services.

The current virtualization environment includes a Debian container used to host Pi-hole.

```text
                         Proxmox
                            |
                     Debian Container
                            |
                         Pi-hole
```

The Debian container provides an isolated environment for the Pi-hole service.

---

# 12. Proxmox Update Issue

The Proxmox server initially could not update because it was configured to use the Proxmox Enterprise repositories without an active subscription.

Running:

```bash
apt-get update
```

resulted in errors similar to:

```text
401 Unauthorized
```

The affected repositories included:

```text
https://enterprise.proxmox.com/debian/pve
https://enterprise.proxmox.com/debian/ceph-squid
```

The Enterprise repositories require a valid Proxmox subscription.

---

# 13. Proxmox No-Subscription Repository

Because this is a home lab rather than a production environment, the Enterprise repositories were disabled and the no-subscription repository was configured.

First, the available repository files were inspected:

```bash
ls -la /etc/apt/sources.list.d/
```

On Proxmox VE 9 / Debian 13 (Trixie), the configuration may contain:

```text
pve-enterprise.sources
ceph.sources
```

The Enterprise repository was disabled with:

```bash
mv /etc/apt/sources.list.d/pve-enterprise.sources \
   /etc/apt/sources.list.d/pve-enterprise.sources.disabled
```

If the Ceph Enterprise repository was present and not required, it was also disabled:

```bash
mv /etc/apt/sources.list.d/ceph.sources \
   /etc/apt/sources.list.d/ceph.sources.disabled
```

The Proxmox no-subscription repository was configured with:

```bash
cat > /etc/apt/sources.list.d/pve-no-subscription.sources <<'EOF'
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

The package database was then updated:

```bash
apt-get update
```

The Proxmox system was subsequently able to retrieve packages from the configured repository.

---

# 14. Pi-hole

Pi-hole has been successfully installed inside a **Debian container running on Proxmox**.

The Debian container provides an isolated and lightweight environment for the Pi-hole service.

Pi-hole is used for two primary purposes:

1. **Local DNS server**
2. **Advertisement and tracker blocking**

The current architecture is:

```text
                         Proxmox
                            |
                     Debian Container
                            |
                         Pi-hole
                       /         \
                      /           \
                 Local DNS     Ad Blocking
                      |
                 Lab Clients
```

---

## 14.1 Debian Container

The Pi-hole installation is hosted inside a Debian container rather than directly on the Proxmox host.

The container provides:

* Isolated environment
* Lightweight resource usage
* Independent operating system
* Service isolation
* Simple management through Proxmox

The container is dedicated to the Pi-hole service.

---

# 15. Pi-hole as Local DNS Server

Pi-hole is used as the local DNS server for the lab environment.

The DNS flow is:

```text
Client
   |
   | DNS Query
   v
Pi-hole
   |
   +----> Local DNS Resolution
   |
   +----> Upstream DNS
```

Pi-hole provides local DNS functionality for internal hosts and services.

This makes it possible to resolve local infrastructure services using hostnames rather than relying exclusively on IP addresses.

Example fictional hostnames:

```text
<FICTIONAL_HOSTNAME>
<FICTIONAL_SERVER>
<FICTIONAL_NAS>
```

Actual internal hostnames and addresses are intentionally omitted from this report.

---

# 16. Pi-hole Ad Blocking

Pi-hole is also configured as an ad blocker.

DNS-based filtering allows unwanted domains to be blocked before clients establish connections to them.

The general process is:

```text
Client
   |
   | DNS Request
   v
Pi-hole
   |
   +---- Domain allowed ----> Upstream DNS
   |
   +---- Domain blocked ----> Blocked
```

Pi-hole can therefore reduce:

* Advertisements
* Tracking domains
* Telemetry domains
* Known unwanted domains

---

# 17. Pi-hole DNS Architecture

The complete DNS architecture is:

```text
                    Lab Clients
                         |
                    DNS Requests
                         |
                         v
                    +---------+
                    | Pi-hole |
                    +---------+
                         |
              +----------+----------+
              |                     |
        Local DNS Records      Upstream DNS
              |                     |
              |                  Internet
              |
       Internal Services
```

Pi-hole acts as the centralized DNS service for clients configured to use it.

The physical network path remains:

```text
Client
   |
Switch / Access Point
   |
Secure Lab Network
```

DNS requests are logically sent to the Pi-hole service:

```text
Client
   |
   | DNS
   v
Pi-hole
```

---

# 18. FSRV NAS

**FSRV** is the NAS used in the home lab and provides centralized network storage.

The NAS is connected to the network through the main network switch.

FSRV provides network storage for authorized clients and infrastructure services.

The general architecture is:

```text
                 Network Switch
                       |
                    FSRV NAS
                       |
                Network Storage
                       |
             +---------+---------+
             |                   |
          Proxmox             Clients
```

The NAS provides a dedicated storage layer independent from the compute resources provided by Proxmox.

FSRV is used for:

* Centralized file storage
* Shared files
* Network file sharing
* Storage accessed by authorized clients

---

# 19. Samba / SMB Network Share

A network share was created on **FSRV** using **Samba**.

Samba provides SMB-compatible network file sharing and allows authorized clients to access shared directories over the network.

The current architecture is:

```text
Client
   |
   | SMB
   |
   v
+---------+
|  FSRV   |
|   NAS   |
+---------+
   |
   +-- Samba / SMB Share
          |
          +-- Shared Files
```

The share is intended to provide centralized file access to authorized devices.

---

## 19.1 Share Configuration

The share configuration can be documented using fictional values:

```text
Server:         <FICTIONAL_NAS_HOSTNAME>
Share Name:     <FICTIONAL_SHARE_NAME>
Protocol:       SMB
Authentication: User-based
Access:         Restricted
```

Real hostnames, usernames, IP addresses, and passwords should not be included in publicly shared documentation.

---

## 19.2 Linux Client Access

A Linux client can access the share using `smbclient`:

```bash
smbclient //<FICTIONAL_NAS_HOSTNAME>/<FICTIONAL_SHARE_NAME> -U <USERNAME>
```

The share can also be mounted locally:

```bash
sudo mount -t cifs //<FICTIONAL_NAS_HOSTNAME>/<FICTIONAL_SHARE_NAME> /mnt/shared \
  -o username=<USERNAME>
```

Credentials should not be exposed directly in shell history or public documentation.

For persistent mounts, a protected credentials file can be used instead of placing the password directly in `/etc/fstab`.

---

# 20. Network Switch

The network switch provides wired connectivity between the main infrastructure components.

The switch connects:

* pfSense
* Proxmox
* FSRV NAS
* TP-Link Access Point
* Desktop workstation

The physical connections are:

```text
                         Network Switch
                  +----------+----------+----------+
                  |          |          |          |
                  |          |          |          |
               Proxmox     FSRV      Desktop    TP-Link
               Server       NAS      Workstation    AP
                                                    |
                                                  Wi-Fi
                                                    |
                                             +------+------+
                                             |             |
                                          Laptop 1      Laptop 2
```

The desktop workstation is connected directly to the switch using Ethernet.

---

# 21. TP-Link Access Point

A TP-Link device is used as the wireless access point for the home lab.

The TP-Link is configured in **Access Point (AP) mode**.

In AP mode, the device provides wireless connectivity while the primary routing and firewall functions remain on pfSense.

The Access Point bridges wireless clients to the wired lab network.

The connectivity path is:

```text
                 Secure Lab Network
                         |
                       Switch
                         |
                 TP-Link Access Point
                      (AP Mode)
                         |
                       Wi-Fi
                         |
                +--------+--------+
                |                 |
             Laptop 1          Laptop 2
```

The TP-Link Access Point is not the primary router or firewall for the lab network.

---

# 22. Desktop Workstation

The desktop workstation is used as an administration and client device within the home lab.

The desktop is connected directly to the network switch via Ethernet.

```text
Desktop Workstation
        |
     Ethernet
        |
      Switch
        |
Secure Lab Network
```

The desktop is used for:

* Infrastructure administration
* Proxmox management
* pfSense management
* Network testing
* Accessing the FSRV Samba share
* General workstation tasks

The desktop is not physically connected to Pi-hole.

Pi-hole is a network service running inside the Debian container on Proxmox.

If the desktop is configured to use Pi-hole as its DNS server, DNS requests are sent over the network to the Pi-hole service.

---

# 23. Laptop Clients

The laptops are wireless client devices connected to the lab network through the TP-Link Access Point.

The connectivity path is:

```text
Laptop
   |
 Wi-Fi
   |
TP-Link AP
 (AP Mode)
   |
Ethernet
   |
Switch
   |
Secure Lab Network
```

The laptops are used to test:

* Network connectivity
* DNS resolution
* VPN connectivity
* Samba/SMB access
* Firewall rules
* General client access

---

# 24. Current Network Topology

The current physical network topology is:

```text
                              Internet
                                  |
                           Home Router
                                  |
                              pfSense
                                  |
                               Switch
                    +-------------+-------------+
                    |             |             |
                    |             |             |
                 Proxmox       FSRV NAS      Desktop
                    |                         Workstation
             Debian Container
                    |
                 Pi-hole

                               Switch
                                  |
                         TP-Link Access Point
                              (AP Mode)
                                  |
                                Wi-Fi
                                  |
                           +------+------+
                           |             |
                        Laptop 1      Laptop 2
```

The desktop workstation has a direct Ethernet connection to the switch.

The laptops connect to the switch through the TP-Link Access Point.

Pi-hole runs as a service inside the Debian container on Proxmox and is accessed over the network by clients configured to use it as their DNS server.

---

# 25. Security Model

The infrastructure is designed around several security principles currently implemented in the environment.

## 25.1 Network Separation

The lab network is separated from the upstream network using pfSense.

```text
Upstream Network
       |
    pfSense
       |
Secure Lab Network
```

pfSense controls the traffic between the upstream network and the lab network.

---

## 25.2 Firewall

The pfSense firewall controls network traffic entering and leaving the lab network.

The firewall provides:

* Network routing
* Traffic filtering
* Access control
* WAN/LAN separation

---

## 25.3 Intrusion Detection

Snort provides additional visibility into network traffic.

```text
Network Traffic
      |
   pfSense
      |
    Snort
      |
   Alerts
```

Snort operates alongside the pfSense firewall and provides intrusion detection capabilities.

---

## 25.4 VPN

WireGuard provides VPN connectivity through pfSense.

Sensitive WireGuard credentials and private keys are not included in this documentation.

---

## 25.5 DNS Filtering

Pi-hole provides DNS-based filtering for clients configured to use it.

```text
Client
   |
 DNS Query
   |
   v
Pi-hole
   |
   +---- Allowed ----> Upstream DNS
   |
   +---- Blocked ----> Blocked
```

This provides an additional security and privacy layer at the DNS level.

---

## 25.6 Credential Security

Passwords, private keys, API tokens, and other secrets are treated as sensitive information.

The pfSense administrator password is stored securely in a password manager.

Sensitive credentials are not included in this documentation.

---

# 26. Sensitive Information Policy

The public version of this documentation must not contain:

* Real internal IP addresses
* Real public IP addresses
* MAC addresses
* Wi-Fi passwords
* VPN private keys
* API tokens
* Authentication credentials
* Proxmox tokens
* Samba passwords
* NAS credentials
* Device serial numbers
* Hardware asset identifiers
* Sensitive hostnames
* VPN configuration files

All sensitive values should be replaced with placeholders.

Examples:

```text
<REAL_IP_ADDRESS>
<REAL_HOSTNAME>
<USERNAME>
<REDACTED>
<FICTIONAL_NAS_HOSTNAME>
<FICTIONAL_SHARE_NAME>
<FICTIONAL_VPN_ADDRESS>
```

Documentation-only network examples use fictional private ranges such as:

```text
172.20.10.0/24
10.99.50.0/24
```

---

# 27. Current Infrastructure Status

| Component | Status |
| --- | --- |
| pfSense installation | Complete |
| WAN configuration | Complete |
| LAN configuration | Complete |
| Routing | Working |
| Internet connectivity | Working |
| Firewall | Configured |
| Snort installation | Complete |
| Snort monitoring | Operational |
| WireGuard package | Installed |
| WireGuard VPN | Working |
| Proxmox installation | Complete |
| Proxmox repository configuration | Fixed |
| Debian container | Complete |
| Pi-hole installation | Complete |
| Pi-hole local DNS | Working |
| Pi-hole ad blocking | Working |
| FSRV NAS | Operational |
| FSRV network connectivity | Working |
| Samba / SMB share | Configured |
| TP-Link Access Point | Operational |
| Desktop network connectivity | Working |
| Laptop Wi-Fi connectivity | Working |

---

# 28. Implemented Services

The current home lab provides the following implemented services.

## pfSense

* Firewall
* Router
* WAN/LAN routing
* Network access control
* WireGuard VPN
* Snort integration

## Snort

* Intrusion detection
* Suspicious traffic monitoring
* Network traffic alerts

## Proxmox

* Virtualization platform
* Container hosting

## Debian Container

* Isolated operating system environment
* Pi-hole hosting

## Pi-hole

* Local DNS
* DNS-based advertisement blocking
* Tracker blocking
* Internal DNS resolution

## FSRV

* Network attached storage
* Centralized file storage

## Samba

* SMB network file sharing
* Authenticated access to shared storage

## TP-Link Access Point

* Wireless network connectivity
* AP mode bridging

---

# 29. Complete Infrastructure Architecture

The current infrastructure can be divided into four functional layers.

## Layer 1 — Network and Security

```text
pfSense
|
+-- Firewall
|
+-- Routing
|
+-- WireGuard VPN
|
+-- Snort IDS
```

This layer provides network routing, firewalling, VPN connectivity, and intrusion detection.

---

## Layer 2 — Network Connectivity

```text
Network Switch
|
+-- Proxmox
|
+-- FSRV NAS
|
+-- Desktop Workstation
|
+-- TP-Link Access Point
      |
      +-- Laptops
```

This layer provides the physical network connectivity between the infrastructure and client devices.

---

## Layer 3 — Compute and Services

```text
Proxmox
|
+-- Debian Container
      |
      +-- Pi-hole
```

This layer provides the virtualization environment and infrastructure services.

---

## Layer 4 — Storage

```text
FSRV NAS
|
+-- Samba / SMB
|
+-- Shared Storage
```

This layer provides centralized network storage and file sharing.

---

# 30. Final Architecture Diagram

```text
                              Internet
                                  |
                           Home Router
                                  |
                              pfSense
                                  |
                               Switch
              +-------------------+-------------------+
              |                   |                   |
              |                   |                   |
           Proxmox             FSRV NAS            Desktop
              |                                     Workstation
       Debian Container
              |
           Pi-hole

                               Switch
                                  |
                         TP-Link Access Point
                              (AP Mode)
                                  |
                                Wi-Fi
                                  |
                           +------+------+
                           |             |
                        Laptop 1      Laptop 2
```

The logical services provided by the infrastructure are:

```text
pfSense
|
+-- Firewall
+-- Routing
+-- WireGuard VPN
+-- Snort IDS

Proxmox
|
+-- Debian Container
      |
      +-- Pi-hole
            |
            +-- Local DNS
            +-- Ad Blocking

FSRV
|
+-- Samba / SMB
+-- Network Storage
```

---

# 31. Security Architecture Summary

The current security architecture can be summarized as:

```text
                         Internet
                            |
                            v
                     +-------------+
                     |   pfSense   |
                     +-------------+
                       |    |    |
                       |    |    +---- WireGuard VPN
                       |    |
                       |    +--------- Snort IDS
                       |
                  Firewall Rules
                       |
                       v
                 Secure Lab Network
                       |
             +---------+---------+
             |                   |
          Network              Services
          Clients                 |
             |                Pi-hole
             |                   |
       +-----+-----+        +----+----+
       |           |        |         |
    Desktop     Laptops   Local DNS  Ad Blocking
       |           |
     Wired       Wi-Fi
       |           |
     Switch    TP-Link AP
```

Each component has a distinct role:

| Technology | Primary Function |
| --- | --- |
| pfSense | Firewall and routing |
| Firewall Rules | Network access control |
| WireGuard | VPN connectivity |
| Snort | Intrusion detection and traffic monitoring |
| Pi-hole | Local DNS and ad/tracker blocking |
| Proxmox | Virtualization |
| Debian Container | Isolated environment for Pi-hole |
| FSRV | NAS and centralized storage |
| Samba | Network file sharing |
| Network Switch | Wired network connectivity |
| TP-Link AP | Wireless network connectivity in AP mode |

---

# 32. Conclusion

The home lab is a multi-purpose infrastructure environment combining networking, security, virtualization, DNS, storage, VPN, intrusion detection, and network file sharing.

The **pfSense firewall** provides the primary network security, routing, and VPN layer.

The **WireGuard VPN** is operational and provides VPN connectivity through pfSense.

**Snort** is installed and active on pfSense, providing additional network visibility and intrusion detection capabilities.

The **Proxmox server** provides the virtualization platform used to host infrastructure services.

A dedicated **Debian container** runs on Proxmox and hosts **Pi-hole**.

Pi-hole provides **local DNS resolution** and **DNS-based advertisement and tracker blocking** for clients configured to use it.

**FSRV** provides centralized network storage.

A **Samba/SMB network share** is configured on FSRV to provide authenticated network file sharing to authorized clients.

The **network switch** provides the primary wired connectivity for the infrastructure.

The **desktop workstation is connected directly to the switch via Ethernet**.

The **TP-Link Access Point is configured in AP mode** and provides wireless connectivity for the laptops.

The current infrastructure provides the following implemented capabilities:

1. **Firewall and routing** through pfSense
2. **Intrusion detection and traffic monitoring** through Snort
3. **VPN connectivity** through WireGuard
4. **Virtualization** through Proxmox
5. **Local DNS and DNS-based filtering** through Pi-hole
6. **Centralized network storage** through FSRV
7. **Network file sharing** through Samba/SMB
8. **Wired network connectivity** through the network switch
9. **Wireless network connectivity** through the TP-Link Access Point

The home lab provides a practical environment for developing and demonstrating skills in:

* Networking
* Firewall configuration
* Routing
* VPNs
* Intrusion detection
* Linux administration
* Virtualization
* DNS
* Network storage
* SMB file sharing
* Network troubleshooting
* Infrastructure management
* Security monitoring
