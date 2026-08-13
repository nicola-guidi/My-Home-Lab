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
* An FSRV NAS
* A TP-Link wireless access point configured in AP mode
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
| FSRV | NAS and network file server | Operational |
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
                             |    |    +----------------------+
                             |    |                           |
                             |    |                  TP-Link Access Point
                             |    |                       (AP Mode)
                             |    |                           |
                             |    |                         Wi-Fi
                             |    |                           |
                             |    |                    +------+------+
                             |    |                    |             |
                             |    |                 Laptop 1      Laptop 2
                             |    |
                             |    +---------------- Desktop Workstation
                             |
                             +-------------------------- FSRV NAS
                             |
                         Proxmox Server
                             |
                     +-------+--------+
                     |
              Debian Container
                     |
                  Pi-hole
