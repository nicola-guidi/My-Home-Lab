Certo — ho sistemato la formattazione per GitHub, in particolare **tabelle, liste, code block, titoli, diagrammi Mermaid/ASCII e link**, mantenendo il contenuto tecnico e i placeholder per le informazioni sensibili.

# Home Lab Infrastructure Report

> **Documentation Notice:** All IP addresses and network addresses shown in this document are fictional and are used exclusively for documentation purposes. They do not represent the actual network configuration.

---

## 1. Overview

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
* Building a flexible environment for future services

The infrastructure is built around a **pfSense firewall**, a **Proxmox virtualization server**, an **FSRV NAS**, a network switch, a wireless access point, and several client devices.

---

## 2. Infrastructure Components

| Component                  | Role                                                  | Status      |
| -------------------------- | ----------------------------------------------------- | ----------- |
| pfSense Firewall Appliance | Firewall, router and VPN gateway                      | Operational |
| Snort                      | Intrusion detection and suspicious traffic monitoring | Operational |
| Network Switch             | Network connectivity                                  | Operational |
| Proxmox Server             | Virtualization platform                               | Operational |
| Debian Container           | Container for Pi-hole                                 | Operational |
| Pi-hole                    | Local DNS server and ad blocker                       | Operational |
| FSRV                       | NAS and network file server                           | Operational |
| Samba / SMB                | Network file sharing                                  | Operational |
| Wireless Access Point      | Wireless connectivity                                 | Operational |
| Desktop Workstation        | Administration / client                               | Operational |
| Laptops                    | Client devices                                        | Operational |

---

## 3. High-Level Architecture

```text
                              Internet
                                  |
                                  |
                           Home Router
                                  |
                                  |
                      +----------------------+
                      |       pfSense        |
                      |----------------------|
                      | Firewall             |
                      | Routing              |
                      | WireGuard VPN        |
                      | Snort IDS            |
                      +----------------------+
                                  |
                         Secure Lab Network
                                  |
                           +-------------+
                           |    Switch   |
                           +-------------+
                             |    |    |
              +--------------+    |    +----------------+
              |                   |                     |
           Proxmox              FSRV NAS           Access Point
              |                   |                     |
              |               Samba/SMB             Wi-Fi Clients
              |                   |
        +-----+------+         Shared
        |            |         Storage
  Debian Container  Future
        |           VMs/LXC
     Pi-hole
        |
   +----+----+
   |         |
Local DNS   Ad Blocking

              |
       Desktop / Laptops
```

---

# 4. Network Architecture

The home lab uses separate networks for the upstream/home environment and the secure lab environment.

The addresses below are **fictional**.

## 4.1 Home / DMZ Network

```text
Network:       172.20.10.0/24
Gateway:       172.20.10.1
pfSense WAN:   172.20.10.254
```

This network represents the upstream network in front of the pfSense firewall.

The pfSense WAN interface is connected to this network.

## 4.2 Secure Lab Network

```text
Network:       10.99.50.0/24
pfSense LAN:   10.99.50.1
```

This is the secure laboratory network behind pfSense.

The pfSense LAN interface acts as the default gateway for devices connected to the lab network.

The LAN interface does not require a separate default gateway.

---

# 5. pfSense Firewall

## 5.1 Installation

The firewall appliance was installed using pfSense.

The installation process was:

1. Download the pfSense ISO.
2. Extract the downloaded image where necessary.
3. Write the image to a USB drive using balenaEtcher.
4. Boot the firewall appliance from the USB drive.
5. Install pfSense on the hardware appliance.
6. Assign the network interfaces.
7. Configure the WAN and LAN networks.
8. Configure the default gateway.
9. Change the administrator password.
10. Verify connectivity.

---

## 5.2 Interface Assignment

The pfSense interfaces were configured as follows:

| Interface | Physical Interface | Network          | Fictional Address  | Purpose               |
| --------- | ------------------ | ---------------- | ------------------ | --------------------- |
| WAN       | `igc0`             | `172.20.10.0/24` | `172.20.10.254/24` | Upstream/home network |
| LAN       | `igc1`             | `10.99.50.0/24`  | `10.99.50.1/24`    | Secure lab network    |

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

After the installation, the following configuration was completed:

* WAN interface assignment
* LAN interface assignment
* WAN IP configuration
* LAN IP configuration
* Default gateway configuration
* Firewall configuration
* Administrator password change
* Password storage in a password manager
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

The lab network can reach the Internet through pfSense.

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

An example of a basic outbound rule is:

```text
Source:       10.99.50.0/24
Destination:  Any
Protocol:     Any
Action:       Allow
Purpose:      Allow lab hosts to access external networks
```

As the infrastructure grows, the firewall configuration should evolve towards more restrictive, service-specific rules.

Future firewall rules should document:

* Source
* Destination
* Protocol
* Destination port
* Action
* Purpose
* Security justification

---

# 7. Snort Intrusion Detection

The **Snort** package was installed and enabled on the pfSense firewall.

Snort is used as a network intrusion detection and traffic monitoring system to identify potentially suspicious or malicious network activity.

The purpose of deploying Snort in the home lab is to provide an additional layer of visibility on top of the pfSense firewall rules.

The security architecture is therefore:

```text
                         Internet
                            |
                            v
                      +-----------+
                      |  pfSense  |
                      |  Firewall |
                      +-----------+
                            |
                     +------+------+
                     |             |
                  Firewall       Snort
                   Rules       Monitoring
                     |             |
                     +------+------+
                            |
                            v
                     Secure Lab Network
```

Snort can monitor network traffic and generate alerts when traffic matches configured detection rules.

Potential events that can be identified include:

* Suspicious network traffic
* Known attack patterns
* Port scanning activity
* Network reconnaissance
* Exploit attempts
* Malicious traffic signatures
* Other traffic matching configured IDS rules

Snort provides additional visibility into network activity and can be used to investigate unusual events within the lab.

## 7.1 Snort Monitoring

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

Snort should be treated as a detection and monitoring layer rather than a replacement for firewall rules.

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

The actual configuration file should be stored securely and must not be committed to a public repository.

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

The WireGuard configuration was based on the [Proton VPN pfSense WireGuard documentation](https://protonvpn.com/support/pfsense-wireguard).

The VPN configuration is now operational.

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

The VPN should periodically be tested for:

* Tunnel establishment
* Connectivity
* DNS resolution
* Routing
* External IP address
* Firewall behavior
* VPN failure handling

---

# 11. Proxmox Virtualization Server

A Proxmox server was added to the infrastructure as the primary virtualization platform.

Proxmox is used to host virtual machines and containers for infrastructure services.

The virtualization layer currently includes a Debian container used to host Pi-hole.

```text
                         Proxmox
                            |
             +--------------+--------------+
             |                             |
       Debian Container                Future VMs/LXC
             |
          Pi-hole
```

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

The Enterprise repository can be disabled with:

```bash
mv /etc/apt/sources.list.d/pve-enterprise.sources \
   /etc/apt/sources.list.d/pve-enterprise.sources.disabled
```

If Ceph is actually used, its Enterprise repository can also be disabled:

```bash
mv /etc/apt/sources.list.d/ceph.sources \
   /etc/apt/sources.list.d/ceph.sources.disabled
```

The Proxmox no-subscription repository can then be configured:

```bash
cat > /etc/apt/sources.list.d/pve-no-subscription.sources <<'EOF'
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

If the system actually uses Ceph:

```bash
cat > /etc/apt/sources.list.d/ceph-no-subscription.sources <<'EOF'
Types: deb
URIs: http://download.proxmox.com/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

Finally:

```bash
apt-get update
```

> **Note:** A single-node home lab using local storage generally does not require Ceph. The Ceph repository should only be configured if Ceph is actually being used.

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

## 14.1 Debian Container

The Pi-hole installation is hosted inside a Debian container rather than directly on the Proxmox host.

The container provides:

* Isolated environment
* Lightweight resource usage
* Independent operating system
* Easy service management
* Simple backup and restore through Proxmox

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

The blocklists can be maintained and expanded over time according to the requirements of the lab.

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

This provides a centralized DNS service for the lab environment.

---

# 18. FSRV NAS

**FSRV** is the NAS used in the home lab and provides centralized network storage.

The NAS is connected to the network through the main network switch.

FSRV can be used for:

* Centralized file storage
* Shared files
* Backup storage
* Proxmox backups
* Application data
* ISO images
* Archives
* Other infrastructure data

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

# 20. FSRV NAS and Backup Strategy

FSRV can also act as a backup destination for the lab.

A possible architecture is:

```text
                     Proxmox
                        |
                 VM / LXC Backups
                        |
                        v
                     FSRV NAS
                        |
                  Backup Storage
```

Potential backup targets include:

* Proxmox virtual machines
* Proxmox containers
* Pi-hole container
* Configuration files
* Application data
* Important documents
* Infrastructure configuration

The NAS itself should not be considered a complete backup strategy.

Important data should ideally have multiple copies, including at least one copy separated from the primary infrastructure.

Future backup improvements may include:

* NAS snapshots
* Proxmox backups
* External backup storage
* Off-site backups
* Automated backup jobs
* Restore testing

---

# 21. Other Infrastructure Components

## 21.1 Network Switch

The network switch provides connectivity between the main infrastructure components.

It connects devices such as:

* pfSense
* Proxmox
* FSRV NAS
* Access Point
* Desktop workstation
* Laptops

A managed switch could later be used to implement VLAN-based segmentation.

## 21.2 Wireless Access Point

A wireless access point provides Wi-Fi connectivity for compatible devices.

Potential future improvements include:

* Separate guest Wi-Fi
* IoT network
* Secure management network
* VLAN-backed SSIDs
* Client isolation

## 21.3 Desktop Workstation

The desktop workstation is used as an administration and client device within the home lab.

Potential uses include:

* Infrastructure administration
* Proxmox management
* pfSense management
* Network testing
* Accessing the FSRV Samba share
* General workstation tasks

## 21.4 Laptops

Laptops are used as client devices within the lab environment.

They can be used to test:

* Network connectivity
* DNS resolution
* VPN connectivity
* Samba/SMB access
* Firewall rules
* General client access

---

# 22. Security Model

The infrastructure is designed around several basic security principles.

## 22.1 Network Segmentation

The secure lab network is separated from the upstream network using pfSense.

```text
Upstream Network
       |
    pfSense
       |
Secure Lab Network
```

## 22.2 Layered Security

Multiple security mechanisms are deployed:

```text
                    Network Traffic
                           |
                           v
                    +-------------+
                    |   pfSense   |
                    |   Firewall  |
                    +-------------+
                           |
                    Firewall Rules
                           |
                           v
                       Snort IDS
                           |
                    Suspicious Traffic
                       Detection
                           |
                           v
                    Secure Lab Network
                           |
                       Pi-hole
                           |
                  DNS / Ad Blocking
```

The combination of pfSense, firewall rules, Snort, WireGuard, and Pi-hole provides multiple security layers.

## 22.3 Least Privilege

Services should only be allowed to communicate with the networks and systems they actually require.

## 22.4 Credential Security

Passwords, private keys, API tokens, and other secrets should be stored securely.

The pfSense administrator password is stored in a password manager.

## 22.5 Management Access

Administrative interfaces should be accessible only from trusted management devices or networks.

## 22.6 Internet Exposure

Infrastructure services should not be exposed directly to the Internet unless there is a specific requirement and appropriate security controls are in place.

---

# 23. Sensitive Information Policy

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

Documentation-only network examples can use fictional private ranges such as:

```text
172.20.10.0/24
10.99.50.0/24
```

---

# 24. Current Infrastructure Status

| Component                        | Status      |
| -------------------------------- | ----------- |
| pfSense installation             | Complete    |
| WAN configuration                | Complete    |
| LAN configuration                | Complete    |
| Routing                          | Working     |
| Internet connectivity            | Working     |
| Firewall                         | Configured  |
| Snort installation               | Complete    |
| Snort monitoring                 | Operational |
| WireGuard package                | Installed   |
| WireGuard VPN                    | Working     |
| Proxmox installation             | Complete    |
| Proxmox repository configuration | Fixed       |
| Debian container                 | Complete    |
| Pi-hole installation             | Complete    |
| Pi-hole local DNS                | Working     |
| Pi-hole ad blocking              | Working     |
| FSRV NAS                         | Operational |
| FSRV network connectivity        | Working     |
| Samba / SMB share                | Configured  |
| VM / container deployment        | In progress |
| Proxmox backup target            | Planned     |
| Monitoring                       | Planned     |
| Centralized logging              | Planned     |
| VLAN segmentation                | Planned     |
| Off-site backup                  | Planned     |

---

# 25. Future Improvements

The next stages of the home lab will focus on improving security, reliability, automation, and observability.

## 25.1 Networking

Planned improvements include:

* Introduce VLANs
* Separate management traffic
* Separate server traffic
* Create a dedicated client network
* Create an IoT network
* Create a guest network
* Improve firewall rules

A possible future design is:

```text
                         pfSense
                            |
                      Managed Switch
                            |
          +-----------------+-----------------+
          |                 |                 |
      VLAN 10            VLAN 20            VLAN 30
    Management          Servers             Clients
          |                 |                 |
       Proxmox          FSRV / VMs        Workstations
                        Pi-hole
```

Additional VLANs could later be introduced for:

```text
VLAN 40 - IoT
VLAN 50 - Guest
VLAN 60 - Security / Monitoring
```

---

# 26. Monitoring

As the infrastructure grows, monitoring should be introduced.

Potential monitoring targets include:

* pfSense availability
* Proxmox resource usage
* CPU utilization
* Memory utilization
* Disk usage
* Disk health
* VM availability
* Container availability
* FSRV availability
* Samba availability
* DNS availability
* VPN tunnel status
* Snort alerts
* Pi-hole availability
* Network connectivity

The goal is to detect failures and suspicious activity before they affect services.

---

# 27. Centralized Logging

Centralized logging could be introduced in a future phase.

Potential log sources include:

* pfSense
* Proxmox
* Debian containers
* FSRV
* Samba
* WireGuard
* Pi-hole
* Snort
* DNS
* Authentication systems

A centralized logging solution would make troubleshooting and security analysis easier.

---

# 28. Backup Strategy

A complete backup strategy should eventually cover:

```text
                 Home Lab
                    |
          +---------+---------+
          |                   |
       Proxmox              FSRV NAS
          |                   |
      VM Backups          File Storage
          |                   |
          +---------+---------+
                    |
             External Backup
                    |
                Off-Site
```

Important considerations include:

* Automated backups
* Multiple backup copies
* Backup retention
* NAS snapshots
* Proxmox VM backups
* Container backups
* Off-site copies
* Encryption
* Restore testing
* Monitoring of failed backup jobs

A backup that has never been restored should not be assumed to be reliable.

---

# 29. Documentation Strategy

As the home lab becomes more complex, documentation should be maintained for:

* Network topology
* VLANs
* IP address allocation
* Firewall rules
* VPN configuration
* Snort configuration
* Snort alerts and monitoring
* Proxmox VMs
* Proxmox containers
* Pi-hole configuration
* DNS records
* FSRV configuration
* Samba shares
* Samba permissions
* Backup jobs
* Monitoring
* Service dependencies

Sensitive values should always be stored separately from the public documentation.

---

# 30. Final Architecture

The current home lab can be summarized into four primary infrastructure layers.

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

Responsible for:

* Firewall
* Routing
* Network segmentation
* VPN connectivity
* Suspicious traffic monitoring
* Intrusion detection
* Internet connectivity

## Layer 2 — Network Connectivity

```text
Network Switch
Access Point
```

Responsible for connecting infrastructure and client devices.

## Layer 3 — Compute and Services

```text
Proxmox
|
+-- Debian Container
|     |
|     +-- Pi-hole
|
+-- Future VMs / Containers
```

Responsible for running infrastructure services and workloads.

## Layer 4 — Storage

```text
FSRV NAS
|
+-- Samba / SMB
|
+-- Shared Storage
|
+-- Backup Storage
```

Responsible for centralized network storage, file sharing, and potentially backups.

---

# 31. Final High-Level Diagram

```text
                              Internet
                                  |
                                  |
                           Home Router
                                  |
                                  |
                      +----------------------+
                      |       pfSense        |
                      |----------------------|
                      | Firewall             |
                      | Routing              |
                      | WireGuard VPN        |
                      | Snort IDS            |
                      +----------------------+
                                  |
                         Secure Lab Network
                                  |
                           +-------------+
                           |    Switch   |
                           +-------------+
                             |    |    |
              +--------------+    |    +----------------+
              |                   |                     |
           Proxmox             FSRV NAS           Access Point
              |                   |                     |
              |               Samba/SMB             Wi-Fi Clients
              |                   |
        +-----+------+         Shared
        |            |         Storage
  Debian Container  Future
        |           VMs/LXC
     Pi-hole
        |
   +----+----+
   |         |
Local DNS   Ad Blocking

              |
       Desktop / Laptops
```

---

# 32. Security Architecture Summary

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
                       v
                    Pi-hole
                       |
                +------+------+
                |             |
            Local DNS      Ad Blocking
```

Each component has a distinct role:

| Technology       | Primary Function                                      |
| ---------------- | ----------------------------------------------------- |
| pfSense          | Firewall and routing                                  |
| Firewall Rules   | Access control                                        |
| WireGuard        | VPN connectivity                                      |
| Snort            | Intrusion detection and suspicious traffic monitoring |
| Pi-hole          | Local DNS and ad/tracker blocking                     |
| Proxmox          | Virtualization                                        |
| Debian Container | Isolated environment for Pi-hole                      |
| FSRV             | NAS and centralized storage                           |
| Samba            | Network file sharing                                  |

---

# 33. Conclusion

The home lab has evolved into a multi-purpose infrastructure environment combining networking, security, virtualization, DNS, storage, VPN, intrusion detection, and network file sharing.

The **pfSense firewall** provides the primary network security, routing, and VPN layer.

The **WireGuard VPN** is operational and provides secure VPN connectivity.

**Snort** has been installed and activated on pfSense to provide additional network visibility and monitor traffic for potentially suspicious or malicious activity.

The **Proxmox server** provides the virtualization platform for infrastructure services.

A dedicated **Debian container** has been created on Proxmox to host **Pi-hole**. Pi-hole is used as the local DNS server and as a DNS-based advertisement and tracker blocker for the lab environment.

**FSRV** is the NAS and provides centralized network storage. A **Samba/SMB network share** has been created on FSRV to provide network file sharing for authorized clients.

The current infrastructure therefore provides five major capabilities:

1. **Network and security** through pfSense
2. **Intrusion detection and traffic monitoring** through Snort
3. **Virtualization and services** through Proxmox and Debian containers
4. **Local DNS and ad blocking** through Pi-hole
5. **Centralized storage and file sharing** through FSRV and Samba

The next development phases will focus on:

* VLAN-based network segmentation
* More granular firewall policies
* Additional Proxmox services
* Backup automation
* Monitoring
* Centralized logging
* Off-site backups
* Infrastructure automation
* Improved Snort monitoring and alert management
* Improved documentation

The home lab provides a practical environment for developing skills in networking, virtualization, Linux administration, security, DNS, storage, VPNs, intrusion detection, and infrastructure management.
