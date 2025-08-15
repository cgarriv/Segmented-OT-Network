# Design and Implementation of a Segmented OT Network

Christian Garcia Rivero | August 10, 2025 | Cisco Packet Tracer | Purdue Model for ICS

### **_Overview_**

This document details the creation of a simulated Operational Technology network for a manufacturing environment. The primary objective is to build a secure, segmented, and functional network that demonstrates the foundational principles of industrial network engineering.

The project is built in phases, starting with a basic process control cell and scaling up to a multi-layered network with routing between segments. This document walks through the design process, implementation, and reasoning behind the creation of a simulated industrial control network.

### **_Phase 1_**

My first priority was to establish a stable and reliable foundation for the most critical part of the network: The Process Cell, representing Level 1 of the Purdue Model. This is where devices like PLCs and HMIs operate, so connectivity here must be robust and predictable.

To begin, I created a simple, flat Layer 2 network. My reasoning for this starting point was to simulate a common scenario in older or smaller industrial sites and build a functional base before adding complexity. Static IP addressing was implemented here for predictability. I designated the 192.168.10.0/24 subnet for this cell.

I implemented this by placing a PLC and a PC (acting as an HMI) onto the Packet Tracer workspace and connecting them to an industrial switch (IE-2000). I then configured their network interfaces: PLC was assigned 192.168.10.10 and the HMI was assigned 192.168.10.11

To validate this initial setup, I performed a ping from the HMI to the PLC. The successful replies confirmed that our Layer 2 foundation was solid and the core components could communicate as required.

```
Cisco Packet Tracer PC Command Line 1.0

C:\> ping 192.168.10.10

Pinging 192.168.10.10 with 32 bytes of data:

Reply from 192.168.10.10: bytes=32 time=2ms TTL=255
Reply from 192.168.10.10: bytes=32 time<1ms TTL=255
Reply from 192.168.10.10: bytes=32 time=9ms TTL=255
Reply from 192.168.10.10: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.10.10:`
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
Minimum = 0ms, Maximum = 9ms, Average = 2ms
C:\>
```
### **_Phase 2: Segmentation_**

While this flat network was functional, it presented a significant security risk. Any device on the network could directly communicate with the critical PLC. My next objective was to enhance security by implementing network segmentation, a core principle of the Purdue Model. The goal was to logically isolate the Process Cell (Level 1) from the Supervisory Network (Level 2), which would contain devices like an Engineering Workstation and a data-collecting Historian server.

To achieve this, I replaced the basic switch with a Cisco 3650 Layer 3 switch. This device was chosen specifically for its ability to create VLANs and route traffic between them, giving us more control.

My design called for two distinct VLANs:

VLAN 10, named PROCESS_CELL, for the PLC and HMI, using the 192.168.10.0/24 subnet.

VLAN 20, named SUPERVISORY, for the new devices, using separate 192.168.20.0/24 subnet.

This implementation began at the switch’s CLI. First, I created and named the two VLANs. With the logical containers in place, I then assigned the physical switch ports to them. The ports connected to the PLC and HMI were assigned to VLAN 10, while the ports for the Engineering Workstation and Historian were assigned to VLAN 20. This step effectively created two separate, isolated domains.

```
Switch# configure terminal
Switch(config)# vlan 10 
Switch(config-vlan)# name PROCESS_CELL
Switch(config-vlan)# vlan 20
Switch(config-vlan)# name SUPERVISORY
Switch(config-vlan)#exit 
Switch(config)# interface range GigabitEthernet1/0/1 - 2 
Switch(config-if-range)# switchport mode access 
Switch(config-if-range)# switchport access vlan 10 
Switch(config-if-range)# interface range GigabitEthernet1/0/3 - 4 
Switch(config-if-range)# switchport mode access 
Switch(config-if-range)# switchport access vlan 20 
Switch(config-if-range)# end

%SYS-5-CONFIG_I: Configured from console by console
Switch#show vlan brief

VLAN Name Status Ports
---- -------------------------------- --------- ------------------------------
1 default active Gig1/0/5, Gig1/0/6, Gig1/0/7, Gig1/0/8
Gig1/0/9, Gig1/0/10, Gig1/0/11, Gig1/0/12
Gig1/0/13, Gig1/0/14, Gig1/0/15, Gig1/0/16
Gig1/0/17, Gig1/0/18, Gig1/0/19, Gig1/0/20
Gig1/0/21, Gig1/0/22, Gig1/0/23, Gig1/0/24
Gig1/1/1, Gig1/1/2, Gig1/1/3, Gig1/1/4
10 PROCESS_CELL active Gig1/0/1, Gig1/0/2
20 SUPERVISORY active Gig1/0/3, Gig1/0/4
1002 fddi-default active
1003 token-ring-default active
1004 fddinet-default active
1005 trnet-default active
Switch#
```

**_Step 2: Configure All End Devices_**

With the network segmented into two VLANs, the next logical step was to configure the end devices to reside in their respective subnets. This involved assigning each device a static IP address, a subnet mask, and a Default Gateway.

For the Supervisory Network (VLAN 20):  
I configured the Engineering Workstation with the IP 192.168.20.20 and the Historian server with 192.168.20.21. Both were assigned 192.168.20.1 as their Default Gateway.

Then, I revisited the devices in the Process Cell (VLAN 10). They needed to be updated to include the new gateway. I updated both the PLC and HMI to use 192.168.10.1 as their Default Gateway.

**_Step 3: Switch Layer 3 and Routing Configuration_**

With the end devices configured, the final step was to make the switch the gateway for both VLANs. I created a Switch Virtual Interface (SVI) for each VLAN and assigned it the corresponding gateway IP address. These SVIs act as a virtual router interface. Finally, I enabled the switch’s global routing capability.

```
Switch# configure terminal
Switch(config)# interface Vlan10
Switch(config-if)# ip address 192.168.10.1 255.255.255.0
Switch(config-if)# no shutdown 
Switch(config-if)# interface Vlan20 
Switch(config-if)# ip address 192.168.20.1 255.255.255.0 
Switch(config-if)# no shutdown 
Switch(config-if)# exit 
Switch(config)# ip routing
Switch(config)# end
```

This configuration completes the implementation of a segmented, multi-VLAN network. To verify functionality, I initiated a ping from the HMI in VLAN 10 to the Engineering Workstation in VLAN 20. The successful replies confirmed that traffic was being routed by the Layer 3 switch between the two isolated networks.

Current Device IP Schema:

| Device Name     | Device Type | VLAN | IP Address      | Subnet Mask     | Default Gateway |
| --------------- | ----------- | ---- | --------------- | --------------- | --------------- |
| **PLC**         | PLC         | 10   | `192.168.10.10` | `255.255.255.0` | `192.168.10.1`  |
| **HMI**         | PC          | 10   | `192.168.10.11` | `255.255.255.0` | `192.168.10.1`  |
| **Eng-Station** | PC          | 20   | `192.168.20.20` | `255.255.255.0` | `192.168.20.1`  |
| **Historian**   | Server      | 20   | `192.168.20.21` | `255.255.255.0` | `192.168.20.1`  |

### ***Phase 3: Overcoming Challenges by Pivoting to Router-on-a-Stick*** 

During the security implementation phase, I encountered a significant issue where the simulator would not correctly apply an Access Control List (ACL) to the switch’s virtual interface (SVI). After extensive troubleshooting, I concluded that this was a bug in the Open Beta 9.0.0 software.  

This challenge presented a valuable opportunity to adapt the design. Rather than reverting to a different software version, I pivoted to a classic and robust topology known as “Router-on-a-Stick (ROAS). The goal remained the same, but the implementation changed: 

- The 3650 Switch was reconfigured to handle only Layer 2 operations (VLANs). All Layer 3 SVIs and the ip routing command were cleaned.

- A 4331 Router was added to the network to handle all Layer 3 operations; routing and security. 

The physical connection from the switch to the router (GigabitEthernet1/0/5) was configured as a trunk port. This allows the single link to carry traffic for both VLAN 10 and VLAN 20, using 802.1Q tagging to keep them separate. On the router, I created two virtual sub-interfaces (GigabitEthernet0/0/0.10 and GigabitEthernet0/0/0.20), assigning them the gateway IP addresses for their respective VLANs.

### ***Phase 4: Implementing and Refining ACL Security*** 

With the ROAS design fully functional, the final step was to implement the security policy. I created an extended named ACL called VLAN10_SEC on the router. The initial policy was to allow the Historian and Engineer Workstations to initiate contact with the Process Cell devices.  

However, testing revealed another critical learning opportunity. While the initial ping request from the workstation to the PLC was permitted, the PLC’s reply was blocked. The simulation mode in Packet Tracer showed the return packet being dropped at the router with the message: The packet does not match the criteria of any statement in the access-list.
## Packet Tracer Simulation Log

This log details the step-by-step journey of a single ICMP packet from the Engineering Workstation (PC2) to the PLC (PLC0) and its attempted return. The simulation was critical in diagnosing the stateless ACL issue. 

```
**============================================================  
PART 1: ICMP ECHO (PING) REQUEST [PC2 -> PLC0]  
============================================================  
  
DEVICE: PC2 (Source)  
------------------------------------------------------------  
LAYER 3:  
  - Ping process creates an ICMP Echo Request.  
  - Destination 192.168.10.10 is on a different subnet.  
  - Next-hop is set to the default gateway (192.168.20.1).  
LAYER 2:  
  - ARP table lookup for the gateway's MAC address is successful.  
  - PDU is encapsulated into an Ethernet frame.  
LAYER 1:  
  - FastEthernet0 sends the frame to the switch.  
  
DEVICE: Multilayer Switch 1 (First Pass)  
------------------------------------------------------------  
LAYER 1 (Inbound):  
  - GigabitEthernet1/0/4 receives the frame.  
LAYER 2:  
  - Switch looks up the destination MAC (the router) in its MAC table.  
  - Outgoing port (Gi1/0/5) is a trunk; the frame is tagged for VLAN 20.  
LAYER 1 (Outbound):  
  - GigabitEthernet1/0/5 sends the tagged frame to the router.  
  
DEVICE: Router0 (Forwarding)  
------------------------------------------------------------  
LAYER 1 (Inbound):  
  - GigabitEthernet0/0/0 receives the frame.  
LAYER 2 (Inbound):  
  - Sub-interface Gi0/0/0.20 accepts the frame based on the VLAN 20 tag.  
  - The 802.1Q tag is removed (de-encapsulated).  
LAYER 3:  
  - Destination IP (192.168.10.10) is looked up in the routing table.  
  - A route is found via sub-interface Gi0/0/0.10.  
  - TTL is decremented.  
LAYER 2 (Outbound):  
  - The PDU is re-encapsulated with a new 802.1Q tag for VLAN 10.  
LAYER 1 (Outbound):  
  - GigabitEthernet0/0/0 sends the newly tagged frame back to the switch.  
  
DEVICE: Multilayer Switch 1 (Second Pass)  
------------------------------------------------------------  
LAYER 1 (Inbound):  
  - GigabitEthernet1/0/5 receives the frame.  
LAYER 2:  
  - Switch reads the VLAN 10 tag.  
  - Destination MAC (the PLC) is looked up in the MAC table.  
  - Outgoing port (Gi1/0/1) is an access port in VLAN 10.  
LAYER 1 (Outbound):  
  - GigabitEthernet1/0/1 sends the untagged frame to the PLC.  
  
DEVICE: PLC0 (Destination)  
------------------------------------------------------------  
LAYER 1 (Inbound):  
  - GigabitEthernet0 receives the frame.  
LAYER 3 (Inbound):  
  - The packet's destination IP matches the device's IP.  
  - The ICMP process receives the Echo Request.  
  
============================================================  
PART 2: ICMP ECHO REPLY [PLC0 -> PC2] (and Failure Point)  
============================================================  
  
DEVICE: PLC0 (Replying)  
------------------------------------------------------------  
LAYER 3 (Outbound):  
  - The ICMP process creates an Echo Reply.  
  - Destination 192.168.20.20 is on a different subnet.  
  - Next-hop is set to the default gateway (192.168.10.1).  
LAYER 2 (Outbound):  
  - ARP lookup for the gateway's MAC is successful.  
  - The PDU is encapsulated into an Ethernet frame.  
LAYER 1 (Outbound):  
  - GigabitEthernet0 sends the frame to the switch.  
  
DEVICE: Multilayer Switch 1 (Third Pass)  
------------------------------------------------------------  
LAYER 1 (Inbound):  
  - GigabitEthernet1/0/1 receives the frame.  
LAYER 2:  
  - The frame is from VLAN 10. The destination MAC is the router.  
  - Outgoing port (Gi1/0/5) is a trunk; the frame is tagged for VLAN 10.  
LAYER 1 (Outbound):  
  - GigabitEthernet1/0/5 sends the tagged frame to the router.  
  
DEVICE: Router0 (Failure Point)  
------------------------------------------------------------  
LAYER 1 (Inbound):  
  - GigabitEthernet0/0/0 receives the frame.  
LAYER 2 (Inbound):  
  - Sub-interface Gi0/0/0.10 accepts the frame based on the VLAN 10 tag.  
LAYER 3 (Inbound):  
  - The interface has an inbound ACL: VLAN10_SEC.  
  - The packet (Source: 192.168.10.10, Dest: 192.168.20.20) is checked.  
  - >> FAILURE: The packet does not match any 'permit' statement.  
  - >> The packet is denied and dropped by the ACL's implicit deny rule.**
```

This demonstrated the stateless nature of standard ACLs. A rule permitting traffic in one direction does not automatically permit the reply. To resolve this, I refined the ACL by adding explicit rules to allow the return traffic from the Process Cell devices back to the Supervisory devices.

```
Router# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)# ip access-list extended VLAN10_SEC
Router(config-ext-nacl)# permit ip host 192.168.10.10 host 192.168.20.21
Router(config-ext-nacl)# permit ip host 192.168.10.10 host 192.168.20.20
Router(config-ext-nacl)# permit ip host 192.168.10.11 host 192.168.20.20
Router(config-ext-nacl)# end
```
And finally we had successful ping with 0% packet loss:

```
C:\>ping 192.168.10.10  

Pinging 192.168.10.10 with 32 bytes of data:

Reply from 192.168.10.10: bytes=32 time<1ms TTL=254
Reply from 192.168.10.10: bytes=32 time<1ms TTL=254
Reply from 192.168.10.10: bytes=32 time<1ms TTL=254
Reply from 192.168.10.10: bytes=32 time<1ms TTL=254

Ping statistics for 192.168.10.10:
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
Minimum = 0ms, Maximum = 0ms, Average = 0ms
C:\>
```
### Updated Device IP Schema:

| Device Name     | Device Type | VLAN | IP Address                                                                       | Subnet Mask     | Default Gateway |
| --------------- | ----------- | ---- | -------------------------------------------------------------------------------- | --------------- | --------------- |
| **PLC**         | PLC         | 10   | `192.168.10.10`                                                                  | `255.255.255.0` | `192.168.10.1`  |
| **HMI**         | PC          | 10   | `192.168.10.11`                                                                  | `255.255.255.0` | `192.168.10.1`  |
| **Eng-Station** | PC          | 20   | `192.168.20.20`                                                                  | `255.255.255.0` | `192.168.20.1`  |
| **Historian**   | Server      | 20   | `192.168.20.21`                                                                  | `255.255.255.0` | `192.168.20.1`  |
| **Router0**     | Router      | N/A  | Sub-Interface VLAN 10: `192.168.10.1` <br> Sub-Interface VLAN 20: `192.168.20.1` | `255.255.255.0` | N/A             |

### Current Network Snapshot:

<img width="561" height="545" alt="Screenshot 2025-08-15 at 5 31 59 PM" src="https://github.com/user-attachments/assets/d6c7267d-3724-41fd-a008-a68437bad00d" />

<img width="611" height="607" alt="Screenshot 2025-08-15 at 5 32 50 PM" src="https://github.com/user-attachments/assets/942e8ca1-a252-4451-a81f-7a32e75e84d8" />





