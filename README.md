# Network Troubleshooting in Cisco Packet Tracer

## Overview

This project is dedicated to network troubleshooting in **Cisco Packet Tracer** 

---

## Initial Setup

Two organizations operate in the same office space and rent part of the premises from a single provider.

### Organization 1
- Network: `172.16.10.0/24`
- IP addresses: `172.16.10.10–12`

### Organization 2
- Network: `172.16.20.0/24`
- IP addresses: `172.16.20.10–11`

The IP addressing **must not be changed**.

All networking equipment is shared and provided by the landlord.  
The system administrator’s task is to ensure that computers from different organizations **cannot see each other**.

The administrator started configuring the network but did not complete the work and was reassigned to another site.

---

## Information Left by the Previous Administrator

- All computers are connected to a single switch.
- Which computer is connected to which switch port is unknown.
- Switch port `GigabitEthernet0/1` is connected to router port `FastEthernet1/0`.

The following configuration was applied on the router:

    Router> enable
    Router# configure terminal

    Router(config)# interface FastEthernet1/0.100
    Router(config-subif)# encapsulation dot1Q 100
    Router(config-subif)# ip address 172.16.10.1 255.255.255.0
    Router(config-subif)# exit

    Router(config)# interface FastEthernet1/0.200
    Router(config-subif)# encapsulation dot1Q 200
    Router(config-subif)# ip address 172.16.20.1 255.255.255.0
    Router(config-subif)# exit

    Router(config)# access-list 101 deny ip 172.16.10.0 0.0.0.255 172.16.20.0 0.0.0.255
    Router(config)# access-list 101 permit ip any any

    Router(config)# access-list 102 deny ip 172.16.20.0 0.0.0.255 172.16.10.0 0.0.0.255
    Router(config)# access-list 102 permit ip any any

    Router(config)# interface FastEthernet1/0.100
    Router(config-subif)# ip access-group 101 in
    Router(config-subif)# ip access-group 102 out
    Router(config-subif)# exit

    Router(config)# interface FastEthernet1/0.200
    Router(config-subif)# ip access-group 101 out
    Router(config-subif)# ip access-group 102 in
    Router(config-subif)# exit

    Router# write
    Building configuration...
    [OK]

---

## Configuration Breakdown

- `enable` switches the router from user EXEC mode to privileged EXEC mode.
- `configure terminal` enters global configuration mode.
- Subinterface `FastEthernet1/0.100` is created for **VLAN 100**.
- Subinterface `FastEthernet1/0.200` is created for **VLAN 200**.
- IEEE **802.1Q (dot1Q)** encapsulation is used for VLAN tagging.
- IP address `172.16.10.1` is assigned as the default gateway for VLAN 100.
- IP address `172.16.20.1` is assigned as the default gateway for VLAN 200.
- **ACL 101** blocks traffic from `172.16.10.0/24` to `172.16.20.0/24`.
- **ACL 102** blocks traffic from `172.16.20.0/24` to `172.16.10.0/24`.
- Each ACL includes a `permit ip any any` rule to allow all other traffic.
- ACLs are applied to inbound and outbound directions on the corresponding subinterfaces.

---

## Summary

On the router interface connected to the switch, two subinterfaces are configured:

- **VLAN 100** → `172.16.10.0/24`
- **VLAN 200** → `172.16.20.0/24`

### Default Gateway Addresses
- `172.16.10.1`
- `172.16.20.1`

Inter-VLAN traffic is restricted using **Access Control Lists (ACLs)**.

---

## Task 1 — Switch Configuration

Configure the switch so that computers from each organization can reach the default gateway of their own subnet:

- `172.16.10.1`
- `172.16.20.1`

After completing the configuration, run the `write` command to ensure the settings are saved and persist after a reboot.

---

## Task 2 — Server Deployment and Network Isolation

Both organizations decided to place their servers at the same provider.

### Infrastructure
- Router
- Switch
- Two servers

### Switch Configuration
- **VLAN 10** — first organization
- **VLAN 20** — second organization
- A **trunk port** connects the switch to the router

### Router Configuration

**VLAN 10 (Organization 1)**
- Subnet: `192.168.10.0/24`
- Router IP: `192.168.10.1`
- Server IP: `192.168.10.10`

**VLAN 20 (Organization 2)**
- Subnet: `192.168.20.0/24`
- Router IP: `192.168.20.1`
- Server IP: `192.168.20.10`

Routers **R0** and **R1** are connected via unconfigured `FastEthernet0/0` interfaces.

### Objective
- Restrict access between `192.168.10.0/24` and `192.168.20.0/24`
- Configure connectivity between the two sites
- Ensure that each organization can access **only its own server**

---

## Task 3 — NAT Configuration

In the external provider network, three servers are available:
- `210.0.11.50–52`

From subnet `210.0.11.0/24`, the provider assigned two public IP addresses:
- `210.0.11.10`
- `210.0.11.20`

The address `210.0.11.10` is configured on interface `FastEthernet0/1` of router **R1**.

### Objective
Configure **Network Address Translation (NAT)** to provide access to these external servers from both sites.

---

## Technologies Used

- Cisco Packet Tracer
- VLAN (802.1Q)
- Router-on-a-Stick
- Access Control Lists (ACL)
- Inter-VLAN Routing
- Static Routing
- Network Address Translation (NAT)
