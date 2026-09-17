# Cisco_Packet_Tracer_Labs
Cisco Packet Tracer: DHCP Configuration Project
https://i.imgur.com/YourImageLinkHere.png
(Note: Replace the link above with the actual image link once you upload your screenshot to GitHub)

📌 Project Overview
This project demonstrates the configuration of Dynamic Host Configuration Protocol (DHCP) across a multi-router network topology. The goal is to automate IP address assignment for end devices (PCs and Server) in three different Local Area Networks (LANs) connected via Serial WAN links.

The central router (Router1) acts as the DHCP Server for all three networks, while Router0 and Router2 function as DHCP Relay Agents (using the ip helper-address command) to forward requests from their respective LANs to the central server.

🗺️ Network Topology & Addressing Scheme
Routers
Device	Interface	IP Address	Subnet Mask	Description
Router0	Fa0/0	192.168.10.1	255.255.255.0	Gateway for LAN 0
Se0/3/0	10.0.0.1	255.255.255.252	WAN Link to Router1
Router1	Fa0/0	192.168.11.1	255.255.255.0	Gateway for LAN 1
Se0/3/0	10.0.0.2	255.255.255.252	WAN Link to Router0
Se0/3/1	11.0.0.1	255.255.255.252	WAN Link to Router2
Router2	Fa0/0	192.168.12.1	255.255.255.0	Gateway for LAN 2
Se0/3/1	11.0.0.2	255.255.255.252	WAN Link to Router1
DHCP Pools (Configured on Router1)
Pool Name	Network Address	Default Gateway	DNS Server
POOL_LAN0	192.168.10.0/24	192.168.10.1	8.8.8.8
POOL_LAN1	192.168.11.0/24	192.168.11.1	8.8.8.8
POOL_LAN2	192.168.12.0/24	192.168.12.1	8.8.8.8
⚙️ Configuration Steps
1. Router0 Configuration (Relay Agent)
Router0 needs to forward DHCP requests from the 192.168.10.0/24 network to Router1.

bash
Router> enable
Router# configure terminal
Router(config)# hostname Router0
Router(config)# interface FastEthernet0/0
Router(config-if)# ip address 192.168.10.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit

! Configure Serial Link
Router(config)# interface Serial0/3/0
Router(config-if)# ip address 10.0.0.1 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit

! Configure DHCP Relay
Router(config)# interface FastEthernet0/0
Router(config-if)# ip helper-address 10.0.0.2
Router(config-if)# exit

! Configure Routing (OSPF or Static)
Router(config)# router ospf 1
Router(config-router)# network 192.168.10.0 0.0.0.255 area 0
Router(config-router)# network 10.0.0.0 0.0.0.3 area 0
2. Router1 Configuration (DHCP Server & Core)
Router1 holds the DHCP pools for all networks and routes traffic between them.

bash
Router> enable
Router# configure terminal
Router(config)# hostname Router1

! Configure Interfaces
Router(config)# interface FastEthernet0/0
Router(config-if)# ip address 192.168.11.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface Serial0/3/0
Router(config-if)# ip address 10.0.0.2 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface Serial0/3/1
Router(config-if)# ip address 11.0.0.1 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit

! Configure DHCP Pools
Router(config)# ip dhcp pool POOL_LAN0
Router(dhcp-config)# network 192.168.10.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.10.1
Router(dhcp-config)# dns-server 8.8.8.8
Router(dhcp-config)# exit

Router(config)# ip dhcp pool POOL_LAN1
Router(dhcp-config)# network 192.168.11.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.11.1
Router(dhcp-config)# dns-server 8.8.8.8
Router(dhcp-config)# exit

Router(config)# ip dhcp pool POOL_LAN2
Router(dhcp-config)# network 192.168.12.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.12.1
Router(dhcp-config)# dns-server 8.8.8.8
Router(dhcp-config)# exit

! Configure Routing (OSPF)
Router(config)# router ospf 1
Router(config-router)# network 192.168.11.0 0.0.0.255 area 0
Router(config-router)# network 10.0.0.0 0.0.0.3 area 0
Router(config-router)# network 11.0.0.0 0.0.0.3 area 0
3. Router2 Configuration (Relay Agent)
Router2 forwards requests from the 192.168.12.0/24 network.

bash
Router> enable
Router# configure terminal
Router(config)# hostname Router2

! Configure Interfaces
Router(config)# interface FastEthernet0/0
Router(config-if)# ip address 192.168.12.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface Serial0/3/1
Router(config-if)# ip address 11.0.0.2 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit

! Configure DHCP Relay
Router(config)# interface FastEthernet0/0
Router(config-if)# ip helper-address 11.0.0.1
Router(config-if)# exit

! Configure Routing (OSPF)
Router(config)# router ospf 1
Router(config-router)# network 192.168.12.0 0.0.0.255 area 0
Router(config-router)# network 11.0.0.0 0.0.0.3 area 0
✅ Verification
PC Configuration:

Go to PC0, PC1, PC2, PC3, PC4, PC5 and the Server.

Navigate to Desktop > IP Configuration.

Select DHCP.

Verify that the PCs receive IP addresses in their respective ranges (e.g., PC0 should get 192.168.10.x, PC4 should get 192.168.12.x).

Server Configuration:

Since Server0 is on the LAN 2 network, ensure it gets an IP from the 192.168.12.0/24 range or configure it statically as a DNS/Web server if required by the lab scope.

Router Verification:

Run show ip dhcp binding on Router1 to see the list of assigned IP addresses.

Run show ip route on all routers to ensure OSPF has populated the routing tables (look for O entries).

Ping from PC0 to PC4 to verify end-to-end connectivity.

🛠️ Tools Used
Cisco Packet Tracer

OSPF (Open Shortest Path First) for Routing

Cisco IOS CLI

📂 How to Use
Clone this repository.

Open the .pkt file in Cisco Packet Tracer.

Review the configurations or use the CLI commands provided above to rebuild the lab from scratch.
