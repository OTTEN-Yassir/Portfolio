---
title: Conception and deployement of an network infrastructure
publishDate: 2024-02-17 23:29:30
img: /assets/logicalTopologie.png
img_alt: OTTEN Yassir
description: |
  The network infrastructure project involves designing and implementing a comprehensive enterprise network, integrating routing, switching, wireless, and various hosted services across multiple sites in  3 Days.
tags:
  - Network
  - Architecture and Services
  - Working group
  - 3 Days 
---

# Table of Contents

1. [Introduction](#introduction)
2. [Planned Topology](#planned-topology)
3. [Routing and Switching](#routing-and-switching)
    - [Firewall and Router 1](#firewall-and-router-1)
    - [Basic Router Configuration](#basic-router-configuration)
    - [Internet Access via NAT](#internet-access-via-nat)
    - [Switches](#switches)
    - [BGP Link](#bgp-link)
    - [OSPF Propagation](#ospf-propagation)
4. [WiFi](#wifi)
5. [Service Hosting](#service-hosting)
    - [Proxmox Configuration](#proxmox-configuration)
    - [DNS Server](#dns-server)
    - [WEB Server](#web-server)
    - [DHCP Server](#dhcp-server)
    - [RADIUS Server](#radius-server)
    - [FTP Server](#ftp-server)
    - [DFS Server](#dfs-server)
    - [Etherpad](#etherpad)
    - [Nextcloud](#nextcloud)
    - [OpenVPN](#openvpn)
6. [Conclusion](#conclusion)


## Introduction

The network cross-project course brings together a student group for three full days to design and implement a complete network infrastructure. This report presents our activity within this project. Our goal was to establish an enterprise network across two sites (Brussels and Nivelles), equivalent to an Autonomous System (AS64514), integrating multiple security zones managed by a firewall, connection to a second Autonomous System (AS64515), and the Internet. Additionally, providing users with various services: WiFi access, FTP server, VPN connection using OpenVPN, shared EtherPad file, Docker Swarm, Nextcloud, all hosted on Windows or Linux virtual machines integrated into Proxmox servers.

## Planned Topology

Logical:   

![logical topologie](/assets/logicalTopologie.png)

Physical :  

![physical topologie](/assets/physicalTopologie.png)


*VLAN and security zone*:  
Each Vlan corresponds to one security zone.  
VLAN 10: DMZ, 172.30.10.0 /24  
VLAN 20: STAFF, 172.30.20.0 /24    
VLAN 30: GUEST, 172.30.30.0 /24  
VLAN 40: ADMIN, 172.30.40.0 /24  
VLAN 50: MANAGEMENT, 172.30.0.0 /24  
VLAN 100: WAN, 172.30.100.0 /24  
VLAN 200: BGP,  172.30.255.252 /30  

## Routing and Switching

Following the envisioned topology, we configured the various machines. Various difficulties encountered during the configuration of switches and routers as well as during the deployment of services led us to progressively adapt this topology.  

#### Firewall and Router 1
  
**Base**  
To remove the remaining configuration from a previous use, the Palo Alto  firewall was reset to its factory settings. Once this was done, a connection via the serial port and default credentials were sufficient for us to configure a management interface accessible via the network at the address 172.30.0.21.  
The firewall initially served as the default router to the Internet, following a router-on-a-stick topology. For this purpose, a single physical interface was used, divided into multiple sub-interfaces, one per VLAN. This approach routes all traffic to and from the Internet through the Palo Alto and allows it to filter all packets.   Therefore, uninspected Internet traffic must be separated from the rest of the network: it was isolated in VLAN 100, declared only on the firewall.  
A virtual router, named Router 1, is declared and linked to each sub-interface to perform routing to the Internet and inter-VLAN routing.   A NAT rule is added to allow packets to transit to the local network 600 with compatible addresses.  
For simplicity, the firewall's first rule allows all traffic, and all used sub-interfaces allow pings.  

**Cutting off the local network**  
After experiencing several internet outages in the local network and inspecting the traffic passing through the virtual router, we realized that this approach was likely causing issues.  
Therefore, the decision was made to use router 4 as the gateway to the internet, with a separate physical interface for traffic in 172.16.0.0/24.  
The firewall remained a privileged gateway for packet filtering between security zones.    

**Removal of the firewall**  
Various internet connectivity tests and packet inspections using WireShark revealed that the firewall was blocking some traffic.  
It appears that the firewall considers all domain name redirections as threats and blocks them, despite our security rule allowing all traffic in all directions.  
To allow other services to function correctly, the configuration of the different routers was adjusted so that they could route inter-zone traffic themselves, i.e., inter-VLAN traffic.  
The default gateway of the various machines was changed to bypass the firewall, and router 4 became the default route to the internet without passing through the firewall.  
These modifications effectively isolate the firewall virtually, as it no longer serves as a gateway and thus no longer blocks traffic.

#### Basic Router Configuration  
The creation of six sub-interfaces on each router for the implementation of interVLAN routing.  
These sub-interfaces (.10, .20, .30, .40, .50, .100) were deployed to optimize data routing and ensure effective network segmentation.  
We tested router connectivity with pings and encountered no difficulties in this regard.  
One of the four routers, initially R1 and then R4, was configured to accept WAN traffic, namely VLAN100.  
SSH connection was successfully configured on each router with the username "Admin" and password "cisco".  

#### Internet Access via NAT  
We implemented NAT (Network Address Translation) on router 4, but soon realized that it was a bad idea to exit the private network with a different public IP address for each machine.  
With this implementation, we didn't have enough available IP addresses locally for all our machines.   
Therefore, we opted for PAT (Port Address Translation), which allows us to use a single public IP address for all machines on our network.  
To achieve this, we simply overloaded the output interface.  

#### Switches  
Configuring VLANs on the appropriate ports of the switches allowed for the establishment of logical connections between network devices.  
We configured three TRUNK ports per switch, one for switch-switch link and two for router-switch links (SW_BXL with R2 and R3, SW_NVL with R1 and R4).  
Ports were configured in ACCESS VLAN 50 mode for Proxmox machines, the WiFi controller, and the access point.  
Other ports were configured based on requirements.  
Ports directed towards the Proxmox servers were eventually declared in TRUNK mode during the readdressing of virtual machines to integrate them into their final VLANs.  
SSH access was properly configured on each switch using the Admin account and Cisco passwords.  

#### BGP Link  
Our network forms AS 64514 and is connected to AS 64512 through a direct link from Router 3, and to AS 64515 via 64512.  
R3 declares its peer as a BGP neighbor and shares with it the network address 172.30.0.0/16, which encompasses the entire AS 64514.  
This sharing is only possible if R3 has this network in its routing table, for which a route 172.30.0.0/16 is declared on loopback0.  
Despite various configurations attempted and the addition of neighboring networks to its routing table, AS 64512 never propagated the network addresses of AS 64514 and 64515 to each other.  
Consequently, these two ASs were unable to exchange traffic.  

#### OSPF Propagation  
Each router in our network is in area 0, and they all share the subnets to which they are directly connected in OSPF 1.  
Router 3, which is our border router, also shares the routes learned in BGP via the neighboring AS (64512) using the command "redistribute BGP 64514 subnets".  
On the other routers, by executing the command "show ip route", we can see that the route to the neighboring AS (64512) has indeed been learned.  

## WiFi

The setup of a WIFI network allows users to have wireless access to the internet. To achieve this, we created three distinct WLANs:
- **GUEST** - which did not contain any security measures.  
- **lab604** - using WPA2-PSK.  
- **ENTERPRISE** - with the RADIUS server mentioned in the services section.  

We decided to work with a Cisco access point in lightweight mode, which allows it to be configured and managed by a controller.  
Therefore, using the controller, we easily configured the 3 different WLANs.  
The goal was to isolate the traffic of each WLAN in a different VLAN.     Eventually, we routed everything through VLAN 50, which is not optimal in terms of security, due to hardware limitations: the access point does not support multiple sub-interfaces for interVLAN, so we were forced to route all traffic through a single VLAN.  

## Hosting of Services  
#### Proxmox Configuration  
Firstly, we began by installing the Proxmox hypervisor on two machines representing the two sites (Nivelles and Brussels). We chose Proxmox because it offers centralized management of virtual machines and containers.  
  


We encountered a storage issue due to improper initial configuration of Proxmox (insufficient allocation of space to pve-root to support ISO transfers in swap). This was resolved by using fdisk to extend the /dev/sda3 partition to make room for swap. We also created a VMs partition and an ISOs directory to store the virtual machine disks and ISOs, respectively.  
  


**TRUNK**:  
After deploying the various services, we configured trunking on the Proxmox machines. To do this, we modified the vmbr0 interface (main virtual bridge) to be "VLAN aware" (able to interact directly with VLANs and tag packets). We then had to create a sub-interface (vmbr0.xx) for each VLAN to connect the different virtual machines to these interfaces.  
  


After completing the environment setup, we began installing the Windows machines, including one machine containing the domain controller, two member servers, and two client machines at each site.  
  


Additionally, we installed Active Directory and added all machines to the gra.esi domain. Initially, we placed all machines in the same network before changing the configuration to place each machine in its respective VLAN. We created various organizational units and added some users to each of them.  
  


**Docker Swarm**:  
Secondly, we attempted to implement the OpenVPN, Etherpad, and Nextcloud services. For this approach, we used Docker Swarm running on two CentOS 9 machines on the Proxmox 2 platform in Nivelles.  
  


Docker Swarm provides us with the concept of a cluster and enables better management of these services with one machine acting as the "Manager" and all others as "Workers". The services are automatically launched using a docker-compose.yml file from the Manager machine, which selects which Worker will run which service based on resources and policy used.  

#### Serveur DNS  

On DC1-BXL (the machine where AD was installed), we installed the DNS service by creating a forward lookup zone and a reverse lookup zone. We were unable to configure replication due to communication issues with the inter-VLAN. We verified the service using the `nslookup + IP address` command on each machine.  

#### Web Server  

On SRV1-BXL and SRV1-NVL, we installed the web server. After installation, when entering the domain name and port number in the browser, we encountered an error (we cannot find your web page). We checked the file location and found that the file extension was incorrect. We tried renaming the file, but it still had .htm which caused the issue. We attempted to set up FTP sharing via the web server. We created a directory containing a text file. When typing the address \\IP\\directoryName in the browser, we could see the text file.  


**Issues encountered**:  
- On the web server, even after changing the file extension, we received a browser error stating that the file could not be found.  
- One of the member servers refused to join the domain because we did not set the DC's IP address as the preferred DNS server.  

#### DHCP Server  

On SRV2-BXL and SRV2-NVL, we installed the DHCP service. However, we were unable to configure this service because we had not yet assigned the correct addresses to our various machines.  

#### RADIUS Server  

On the SRV2-BXL server, we installed and configured a RADIUS server that will allow guests to connect to the Wi-Fi networks at different sites.    

#### FTP Server  

On SRV2-NVL, we installed the FTP service to share a folder that all organizational units will have access to. We verified on different machines that the share was present and all users have access to it.  

#### DFS Server  

On SRV1-NVL, we installed the DFS service but unfortunately, due to lack of time, we were not able to configure it.  

#### Etherpad  

On the Manager machine, we configured the launch of the Etherpad service running on the Worker machine.  

Initially, after pulling the correct image, we launched Etherpad using the `Docker service create ...` command. This approach worked correctly as the service was accessible, but we wanted to automate the process.  

Subsequently, we configured a docker-compose.yml file containing all the necessary information for the successful launch of the Etherpad service.  

Finally, we only had to run this file using the `docker stack deploy -c docker-compose.yml` (docker up -d) command, and the service ran correctly. The `docker service etherpad ps` command shows that the service is active and the "nodes" involved (the Worker machine).  

#### Nextcloud  

On the Manager machine, we pulled the Nextcloud image. Then, we configured the .yml file in the same way as for Etherpad to automate the launch on the Worker.  

Unfortunately, due to time constraints, the testing of this service could not be carried out, but theoretically, it should work without any issues.  

#### OpenVPN  

This service could not be set up in time.  

## Conclusion  

Following the provided instructions, our group designed, implemented, and configured a functional computer network across two sites, integrating internet access for wired and WiFi-connected devices. Difficulties encountered with the firewall forced us to set it aside to prioritize connectivity.  

With the exception of a few partially functional or unimplemented services due to time constraints, the various services were deployed on their respective virtual machines and made accessible to network users. These virtual machines are distributed across different security zones, facilitated by the readdressing and vlan-aware configuration of the proxmox machines hosting them.  

Our group members worked cohesively, regularly updating each other on mutual progress in an agile-inspired approach. This allowed us to account for the necessary time for each task, adapt our work in case of difficulty, and synchronize actions as needed.  
