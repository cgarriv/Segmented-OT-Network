# Design and Implementation of a Segmented OT Network

Christian Garcia Rivero | August 10, 2025 | Cisco Packet Tracer | Purdue Model for ICS

### **_Overview_**

This document details the creation of a simulated Operational Technology network designed for a manufacturing environment. The primary objective is to build a secure, segmented, and functional network that demonstrates the foundational principles of industrial network engineering.

The project is built in phases, starting with a basic process control cell and scaling up to a multi-layered, secure network with a DMZ, inter-VLAN routing, and operational hardening. This document walks through the design process, implementation, and reasoning behind the creation of a simulated industrial control network.

### **_Methodologies and Principles:_**

- **Purdue Model for ICS:** The network architecture is based on the Purdue Model, establishing a hierarchical framework to separate critical industrial control systems from other network zones.

- **Network Segmentation:** The core security strategy involves logically isolating the Process Cell (Level 1) from the Supervisory Network (Level 2) and a DMZ (Level 3.5) to control traffic flow and limit the blast radius of potential security incidents.

- **Router-on-a-Stick (ROAS):** This topology was adopted to handle all Layer 3 operations, using a single router to manage routing and security policies for multiple VLANs via a trunk link.

- **Defense-in-Depth:** Security is implemented in layers, starting with VLAN segmentation and progressively adding Access Control Lists (ACLs) and Layer 2 hardening features to create a more resilient and secure environment.

- **Phased Implementation:** The project was built iteratively, starting with a simple flat network and deliberately adding complexity in phases, which allowed for methodical testing and validation at each stage. 

### **_Technologies and Tools_**:

- **Simulation Software:**
	- Cisco Packet Tracer

- **Hardware:**
	- Cisco ISR 4331 Router 
	- Cisco 3650-24RS Multilayer Switch
	- Level 0 Devices: Sensors and Actuators
	- Level 1 Device: PLC
	- Data Historian Server
	- Servers: Patch Server, Relay
	- PCs: Engineering Workstation, HMI, JumpHost

- **Security Mechanisms:**
	- **Virtual LANs (VLANs):** Process Cell (VLAN 10), Supervisory network (VLAN 20), DMZ (VLAN 30), and a BLACKHOLE VLAN (VLAN 999).
	- **Extended Access Control Lists (ACLs)**
	- **Layer 2 Hardening**: PortFast, BPDU Guard, unused ports shutdown.

- **Operational Tools:**
	- **Syslog**
	- **Network Time Protocol (NTP)**
 
### **_Phase 1_**

My first priority was to establish a stable and reliable foundation for the most critical part of the network: The Process Cell, representing Level 1 of the Purdue Model. This is where devices like PLCs and DCS controllers operate, so connectivity here must be robust and predictable.

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

While this flat network was functional, it presented a significant security risk. Any device on the network could directly communicate with the critical PLC. My next objective was to enhance security by implementing network segmentation, a core principle of the Purdue Model. The goal was to logically isolate the Process Cell (Level 1) from the Supervisory Network (Level 2), which would contain devices like a HMI, an Engineering Workstation and a data-collecting Historian server.

To achieve this, I replaced the basic switch with a Cisco 3650 Layer 3 switch. This device was chosen specifically for its ability to create VLANs and route traffic between them, giving us more control.

My design called for two distinct VLANs:

VLAN 10, named PROCESS_CELL, for the PLC using the 192.168.10.0/24 subnet.

VLAN 20, named SUPERVISORY, for the HMI and new devices, using a separate 192.168.20.0/24 subnet.

This implementation began at the switch’s CLI. First, I created and named the two VLANs. With the logical containers in place, I then assigned the physical switch ports to them. The ports connected to the PLC were assigned to VLAN 10, while the ports for the HMI, Engineering Workstation, and Historian were assigned to VLAN 20. This step effectively created two separate, isolated domains.

```
Switch# configure terminal
Switch(config)# vlan 10 
Switch(config-vlan)# name PROCESS_CELL
Switch(config-vlan)# vlan 20
Switch(config-vlan)# name SUPERVISORY
Switch(config-vlan)#exit 
Switch(config)# interface GigabitEthernet1/0/1 
Switch(config-if-range)# switchport mode access 
Switch(config-if-range)# switchport access vlan 10 
Switch(config-if-range)# interface range GigabitEthernet1/0/2 - 4 
Switch(config-if-range)# switchport mode access 
Switch(config-if-range)# switchport access vlan 20 
Switch(config-if-range)# end

%SYS-5-CONFIG_I: Configured from console by console
Switch#show vlan brief

VLAN Name Status Ports
---- -------------------------------- --------- ------------------------------
1 default active Gig1/0/9, Gig1/0/10, Gig1/0/11, Gig1/0/12
Gig1/0/13, Gig1/0/14, Gig1/0/15, Gig1/0/16
Gig1/0/17, Gig1/0/18, Gig1/0/19, Gig1/0/20
Gig1/0/21, Gig1/0/22, Gig1/0/23, Gig1/0/24
Gig1/1/1, Gig1/1/2, Gig1/1/3, Gig1/1/4
10 PROCESS_CELL active Gig1/0/1
20 SUPERVISORY active Gig1/0/2, Gig1/0/3, Gig1/0/4
1002 fddi-default active
1003 token-ring-default active
1004 fddinet-default active
1005 trnet-default active
Switch#
```

**_Step 2: Configure All End Devices_**

With the network segmented into two VLANs, the next logical step was to configure the end devices to reside in their respective subnets. This involved assigning each device a static IP address, a subnet mask, and a Default Gateway.

For the Supervisory Network (VLAN 20):  
I configured the Engineering Workstation with the IP 192.168.20.20, The HMI with IP: 192.168.20.21 and the Historian server with 192.168.20.22. All were assigned 192.168.20.1 as their Default Gateway.

Then, I revisited the device in the Process Cell (VLAN 10). It needed to be updated to include the new gateway. I updated the PLC to use 192.168.10.1 as its Default Gateway.

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

This configuration completes the implementation of a segmented, multi-VLAN network. To verify functionality, I initiated a ping from the HMI in VLAN 20 to the PLC in VLAN 10. The successful replies confirmed that traffic was being routed by the Layer 3 switch between the two isolated networks.

Current Device IP Schema:

| Device Name     | Device Type | VLAN | IP Address      | Subnet Mask     | Default Gateway |
| --------------- | ----------- | ---- | --------------- | --------------- | --------------- |
| **PLC**         | PLC         | 10   | `192.168.10.10` | `255.255.255.0` | `192.168.10.1`  |
| **HMI**         | PC          | 20   | `192.168.20.21` | `255.255.255.0` | `192.168.20.1`  |
| **Eng-Station** | PC          | 20   | `192.168.20.20` | `255.255.255.0` | `192.168.20.1`  |
| **Historian**   | Server      | 20   | `192.168.20.22` | `255.255.255.0` | `192.168.20.1`  |

### ***Phase 3: Overcoming Challenges by Pivoting to Router-on-a-Stick*** 

During the security implementation phase, I encountered a significant issue where the simulator would not correctly apply an Access Control List (ACL) to the switch’s virtual interface (SVI). After extensive troubleshooting, I concluded that this was a bug in the Open Beta 9.0.0 software.  

This challenge presented a valuable opportunity to adapt the design. Rather than reverting to a different software version, I pivoted to a classic and robust topology known as “Router-on-a-Stick (ROAS). The goal remained the same, but the implementation changed: 

- The 3650 Switch was reconfigured to handle only Layer 2 operations (VLANs). All Layer 3 SVIs and the ip routing command were cleaned.

- A 4331 Router was added to the network to handle all Layer 3 operations; routing and security. 

The physical connection from the switch to the router (GigabitEthernet1/0/5) was configured as a trunk port. This allows the single link to carry traffic for both VLAN 10 and VLAN 20, using 802.1Q tagging to keep them separate. On the router, I created two virtual sub-interfaces (GigabitEthernet0/0/0.10 and GigabitEthernet0/0/0.20), assigning them the gateway IP addresses for their respective VLANs.

### ***Phase 4: Implementing and Refining ACL Security*** 

With the ROAS design fully functional, the final step was to implement the security policy. I created an extended named ACL called VLAN10_SEC on the router. The initial policy was to allow the HMI, Historian and Engineer Workstations to initiate contact with the Process Cell devices.  

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

This demonstrated the stateless nature of standard ACLs. A rule permitting traffic in one direction does not automatically permit the reply. To resolve this, I refined the ACL by adding explicit rules to allow the return traffic from the Process Cell device back to the Supervisory devices.

```
Router# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)# ip access-list extended VLAN10_SEC
Router(config-ext-nacl)# permit ip host 192.168.10.10 host 192.168.20.20
Router(config-ext-nacl)# permit ip host 192.168.10.10 host 192.168.20.21
Router(config-ext-nacl)# permit ip host 192.168.10.10 host 192.168.20.22
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
| **HMI**         | PC          | 20   | `192.168.20.21`                                                                  | `255.255.255.0` | `192.168.20.1`  |
| **Eng-Station** | PC          | 20   | `192.168.20.20`                                                                  | `255.255.255.0` | `192.168.20.1`  |
| **Historian**   | Server      | 20   | `192.168.20.22`                                                                  | `255.255.255.0` | `192.168.20.1`  |
| **Router0**     | Router      | N/A  | Sub-Interface VLAN 10: `192.168.10.1` <br> Sub-Interface VLAN 20: `192.168.20.1` | `255.255.255.0` | N/A             |

### Current Network Snapshot:

<img width="561" height="544" alt="Screenshot 2025-08-16 at 2 55 00 PM" src="https://github.com/user-attachments/assets/429050b5-646b-4631-ad4b-e61e29090efc" />

<img width="611" height="607" alt="Screenshot 2025-08-15 at 5 32 50 PM" src="https://github.com/user-attachments/assets/942e8ca1-a252-4451-a81f-7a32e75e84d8" />

### ***Phase 5: ICS DMZ (Level 3.5)*** 

I reached a point where the L1/L2 segmentation and initial ACL policy were solid. The next logical step was to formalize a broker zone between Supervisory (Level 2) and anything “northbound”. I decided to stand up a dedicated ICS DMZ (VLAN 30) with three hosts: A JumpHost for controlled administrative access, a Patch Server for updates and centralized logging, and a Historian Relay to stage data flow out of L2. The design goal was very clear: L2 can initiate toward DMZ for specific services; DMZ cannot initiate towards L2 or L1. This preserves blast-radius control and matches common ICS reference architecture. 

### 5.1 Addressing and topology snapshot 

I created 192.168.30.0/24 for the DMZ and added a third router subinterface on the ROAS link: 
- JumpHost (PC3): 192.168.30.10/24 
- Patch Server: 192.168.30.11/24 
- Historian Relay: 192.168.30.12/24 
- Router gateway on VLAN 30: 192.168.30.1/24 on G0/0/0.30 

This keeps the mental model simple when I’m reading ACLs later: .10 = JumpHost, .11 = Patch, etc.. The switch keeps doing pure Layer 2 functions; the router enforces all Layer 3 policies. 

#### Router: Create the DMZ subinterface + gateway:
```
Router#configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Router(config)#interface GigabitEthernet0/0/0.30
Router(config-subif)#
%LINK-3-UPDOWN: Interface GigabitEthernet0/0/0.30, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0.30, changed state to up

Router(config-subif)#encapsulation dot1q 30
Router(config-subif)#ip address 192.168.30.1 255.255.255.0
Router(config-subif)#no shutdown
Router(config-subif)#exit
Router(config)#end
Router#
%SYS-5-CONFIG_I: Configured from console by console

Router#show ip interface brief | include 0/0/0.
GigabitEthernet0/0/0 unassigned YES unset up up
GigabitEthernet0/0/0.10192.168.10.1 YES manual up up
GigabitEthernet0/0/0.20192.168.20.1 YES manual up up
GigabitEthernet0/0/0.30192.168.30.1 YES manual up up
Router#
```

#### Map DMZ Access Ports:
```
Switch#configure terminal
Enter configuration commands, one per line. End with CNTL/Z.

Switch(config)#interface GigabitEthernet1/0/6
Switch(config-if)#description DMZ_JumpHost
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 30
Switch(config-if)#spanning-tree portfast

%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on GigabitEthernet1/0/6 but will only
have effect when the interface is in a non-trunking mode.
Switch(config-if)#end
Switch#
%SYS-5-CONFIG_I: Configured from console by console
```

I repeated the same pattern for the Patch Server and Historian Relay. This mirrors how I built VLAN 10/20. 

### 5.2 Policy Enforcement:

I wrote the rules as sentences first, then translated them to ACLs. L2 is allowed to initiate very specific sessions into the DMZ; the DMZ is reply-only back to L2, and completely blocked from initiating to L1. For the PLC path, I intentionally kept Modbus/TCP, DNP3, and EtherNet/IP open from L2 to L1, just for practice purposes.
### Router: Update VLAN 20 ACL
```
Router#configure terminal

Enter configuration commands, one per line. End with CNTL/Z.

Router(config)#
Router(config)#ip access-list extended VLAN20_SEC
Router(config-ext-nacl)#remark Allow Supervisory hosts to ping their gateway
Router(config-ext-nacl)#permit icmp 192.168.20.0 0.0.0.255 host 192.168.20.1 echo
Router(config-ext-nacl)#remark Allow ICMP echo from Supervisory to DMZ (testing)
Router(config-ext-nacl)#permit icmp 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255 echo
Router(config-ext-nacl)#remark Eng WS -> DMZ JumpHost (RDP)
Router(config-ext-nacl)#permit tcp host 192.168.20.20 host 192.168.30.10 eq 3389
Router(config-ext-nacl)#remark Eng WS -> DMZ PatchSrv (HTTPS)
Router(config-ext-nacl)#permit tcp host 192.168.20.20 host 192.168.30.11 eq 443
Router(config-ext-nacl)#remark L2 Histortian -> DMZ Historian Relay
Router(config-ext-nacl)#permit tcp host 192.168.20.22 host 192.168.30.12 eq 443
Router(config-ext-nacl)#exit
Router(config)#
Router(config)#end
Router#
%SYS-5-CONFIG_I: Configured from console by console
```

I validated each flow interactively (RDP to .10, HTTPS to .11/.12) and kept the "443 placeholder" until the real vendor ports are defined.
### Router: DMZ ACLs

```
Router#configure terminal
Enter configuration commands, one per line. End with CNTL/Z.

Router(config)#ip access-list extended VLAN30_SEC
Router(config-ext-nacl)#remark Allow return TCP to Supervisory
Router(config-ext-nacl)#permit tcp 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255 established
Router(config-ext-nacl)#remark Allow ICMP echo-replies back to Supervisory
Router(config-ext-nacl)#permit icmp 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255 echo-reply
Router(config-ext-nacl)#remark Block DMZ from initiating to Process Cell
Router(config-ext-nacl)#deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
Router(config-ext-nacl)#remark Log any other DMZ-initiated traffic
Router(config-ext-nacl)#deny ip any any
Router(config-ext-nacl)#exit
Router(config)#interface GigabitEthernet0/0/0.30
Router(config-subif)#ip access-group VLAN30_SEC in
Router(config-subif)#end
Router#
%SYS-5-CONFIG_I: Configured from console by console
```

The single deny protects the cell boundary regardless of which DMZ host is compromised. The established and echo-reply lines preserve usability and my ability to test.

I documented all stacks to show options and gain practice on all of them. 
#### Modbus/TCP (502) - HMI + Eng WS to PLC
```
Router#configure terminal
Enter configuration commands, one per line. End with CNTL/Z.

Router(config)#ip access-list extended VLAN10_SEC
Router(config-ext-nacl)#remark Modbus/TCP from L2 to PLC
Router(config-ext-nacl)#permit tcp host 192.168.20.21 host 192.168.10.10 eq 502
Router(config-ext-nacl)#permit tcp host 192.168.20.20 host 192.168.10.10 eq 502
Router(config-ext-nacl)#exit
Router(config)#end
Router#
%SYS-5-CONFIG_I: Configured from console by console
```

#### EtherNet/IP CIP (TCP 44818, UDP 2222) - HMI + Eng WS to PLC
```
Router#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.

Router(config)#ip access-list extended VLAN10_SEC
Router(config-ext-nacl)#remark EtherNet/IP CIP from L2 to PLC
Router(config-ext-nacl)#permit tcp host 192.168.20.21 host 192.168.10.10 eq 44818
Router(config-ext-nacl)#permit tcp host 192.168.20.20 host 192.168.10.10 eq 44818
Router(config-ext-nacl)#permit udp host 192.168.20.20 host 192.168.10.10 eq 2222
Router(config-ext-nacl)#permit udp host 192.168.20.21 host 192.168.10.10 eq 2222
Router(config-ext-nacl)#exit
Router(config)#end
Router#
%SYS-5-CONFIG_I: Configured from console by console
```

#### DNP3 (TCP 20000) 
```
Router#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.

Router(config)#ip access-list extended VLAN10_SEC
Router(config-ext-nacl)#remark DNP3 from L2 to PLC
Router(config-ext-nacl)#permit tcp host 192.168.20.21 host 192.168.10.10 eq 20000
Router(config-ext-nacl)#permit tcp host 192.168.20.20 host 192.168.10.10 eq 20000
Router(config-ext-nacl)#exit
Router(config)#end
Router#
%SYS-5-CONFIG_I: Configured from console by console
```

This preserves the original intent from earlier phases of explicit host/port tuples and reply allowances, now extended to cover OT protocols.

### 5.3 Validation

I validated each allowed service from L2 and intentionally attempted disallowed initiations from DMZ to verify the denies. Packet Tracer makes these quick to demonstrate: from PC2 I connected to the JumpHost via RDP and the Patch Server and Historian Relay over HTTPS; from PC3 I tried to ping the PLC and the Engineering Workstation to show hard failure with the router answering “unreachable” (a good sign the ACLs are doing their job at the gateway).

#### PC2 (192.168.20.20) should succeed
```
C:\>ping 192.168.30.10

Pinging 192.168.30.10 with 32 bytes of data:

Reply from 192.168.30.10: bytes=32 time=1ms TTL=127
Reply from 192.168.30.10: bytes=32 time<1ms TTL=127
Reply from 192.168.30.10: bytes=32 time<1ms TTL=127
Reply from 192.168.30.10: bytes=32 time=2ms TTL=127

Ping statistics for 192.168.30.10:
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
Minimum = 0ms, Maximum = 2ms, Average = 0ms
C:\>
```

#### DMZ JumpHost (192.168.30.10) to Supervisory should fail cannot initiate to VLAN 20

```
C:\>ping 192.168.20.20

Pinging 192.168.20.20 with 32 bytes of data:

Reply from 192.168.30.1: Destination host unreachable.
Reply from 192.168.30.1: Destination host unreachable.
Reply from 192.168.30.1: Destination host unreachable.
Reply from 192.168.30.1: Destination host unreachable.

Ping statistics for 192.168.20.20:
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
C:\>
```
#### DMZ JumpHost (192.168.30.10) to Process Cell should fail cannot initiate to VLAN 10

```
C:\>ping 192.168.10.10

Pinging 192.168.10.10 with 32 bytes of data:

Reply from 192.168.30.1: Destination host unreachable.
Reply from 192.168.30.1: Destination host unreachable.
Reply from 192.168.30.1: Destination host unreachable.
Reply from 192.168.30.1: Destination host unreachable.

Ping statistics for 192.168.10.10:
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
C:\>
```

### ***Phase 6: Visibility and Operational Hardening***

At this point I wanted two operational guarantees: I can see what's happening (central logs, correct timestamps), and the L2 fabric is tidy and predictable.

#### 6.1 Central Syslog and Time:

For the lab, I used the router as the authoritative time source (NTP master, stratum 4) and pointed network devices at the Patch Server (192.168.30.11) for syslog. In production I would replace the NTP master line with real upstream servers.
#### Central time + Logs (DMZ PatchSrv as NTP/Syslog)

```
Router#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.

Router(config)#service timestamps log datetime msec
Router(config)#logging host 192.168.30.11
Router(config)#logging trap
Router(config)#logging buffered 8192
Router(config)#
*Mar 01, 08:21:57.2121: SYS-5-LOG_CONFIG_CHANGE: Buffer logging: level debugging, xml disabled, filtering disabled, size (8192) 
Router(config)#end
Router#
*Mar 01, 08:22:03.2222: SYS-5-CONFIG_I: Configured from console by console
-------------------------------------------------------------------------------------------------------------
Switch(config)#service timestamps log datetime msec
Switch(config)#logging host 192.168.30.11
Switch(config)#logging trap
Switch(config)#end
Switch#
*Mar 01, 08:56:19.5656: SYS-5-CONFIG_I: Configured from console by console
```

#### NTP Demo (ROUTER):

```

Router#show ntp status
Clock is synchronized, stratum 4, reference is 127.127.1.1
nominal freq is 250.0000 Hz, actual freq is 249.9990 Hz, precision is 2**24
reference time is FFFFFFFFEC1F4362.00000172 (8:50:42.370 UTC Sat Aug 16 2025)
clock offset is 0.00 msec, root delay is 0.00  msec
root dispersion is 0.00 msec, peer dispersion is 0.24 msec.
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is - 0.000001193 s/s system poll interval is 5, last update was 18 sec ago.
Router#show ntp associations

address         ref clock       st   when     poll    reach  delay          offset            disp
*~127.127.1.1   .LOCL.          3    28       64      377    0.00           0.00              0.24
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
 * 
Router#show clock
8:51:13.543 UTC Sat Aug 16 2025


```
#### Syslog Demo:

```
Router#show access-lists VLAN30_SEC
Extended IP access list VLAN30_SEC
    permit tcp 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255 established
    permit icmp 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255 echo-reply (21 match(es))
    deny ip any any (81 match(es))
    permit icmp 192.168.30.0 0.0.0.255 host 192.168.30.1 echo-reply
    permit udp host 192.168.30.11 eq 123 host 192.168.30.1
    deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
    permit udp host 192.168.30.11 eq 123 host 192.168.30.1 range 1024 65535
    permit udp host 192.168.30.11 eq 123 host 192.168.30.1 eq 123

Router#show logging
Syslog logging: enabled (0 messages dropped, 0 messages rate-limited,
          0 flushes, 0 overruns, xml disabled, filtering disabled)

No Active Message Discriminator.


No Inactive Message Discriminator.


    Console logging: level debugging, 32 messages logged, xml disabled,
          filtering disabled
    Monitor logging: level debugging, 32 messages logged, xml disabled,
          filtering disabled
    Buffer logging:  level debugging, 0 messages logged, xml disabled,
          filtering disabled

    Logging Exception size (8192 bytes) 
    Count and timestamp logging messages: disabled
    Persistent logging: disabled

No active filter modules.

ESM: 0 messages dropped
    Trap logging: level informational, 32 message lines logged
        Logging to 192.168.30.11  (udp port 514,  audit disabled,
             authentication disabled, encryption disabled, link up),
             14 message lines logged,
             0 message lines rate-limited,
             0 message lines dropped-by-MD,
             xml disabled, sequence number disabled
             filtering disabled

Log Buffer (8192 bytes):

```

#### Syslog Demo 2:
<img width="802" height="329" alt="Screenshot 2025-08-16 at 9 01 49 PM" src="https://github.com/user-attachments/assets/64084144-79c6-4e13-a60f-15fbb8dff454" />

### 6.2 L2 Hygiene: Trunk/Native, Blackhole VLAN, and Edge Guardrails:

I locked down the trunk to the router, moved the native to a BLACKHOLE VLAN, and parked/disabled unused access ports there. On edge ports I enabled PortFast and BPDU Guard so a stray loop can't form from a mis-patch.

#### Trunk + Port Hygiene
```
Switch>enable
Switch#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.

Switch(config)#vlan 999
Switch(config-vlan)#name BLACKHOLE
Switch(config-vlan)#exit
Switch(config)#interface GigabitEthernet1/0/5
Switch(config-if)#switchport trunk allowed vlan 10,20,30,999
Switch(config-if)#switchport trunk native vlan 999
Switch(config-if)#end
Switch#
%SYS-5-CONFIG_I: Configured from console by console

Switch(config)#interface range GigabitEthernet1/0/9 - 24
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport access vlan 999
Switch(config-if-range)#shutdown

%LINK-5-CHANGED: Interface GigabitEthernet1/0/9, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/10, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/11, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/12, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/13, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/14, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/15, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/16, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/17, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/18, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/19, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/20, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/21, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/22, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/23, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet1/0/24, changed state to administratively down
Switch(config-if-range)#end
Switch#
%SYS-5-CONFIG_I: Configured from console by console


Switch(config)#! HMI/servers/DMZ ports
Switch(config)#interface range GigabitEthernet1/0/1 - 4, GigabitEthernet1/0/6 - 8
Switch(config-if-range)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on GigabitEthernet1/0/1 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on GigabitEthernet1/0/2 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on GigabitEthernet1/0/3 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on GigabitEthernet1/0/4 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on GigabitEthernet1/0/6 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on GigabitEthernet1/0/7 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on GigabitEthernet1/0/8 but will only
have effect when the interface is in a non-trunking mode.
Switch(config-if-range)#spanning-tree bpduguard enable
Switch(config-if-range)#storm-control broadcast level 5.00	
Switch(config-if-range)#end
Switch#
%SYS-5-CONFIG_I: Configured from console by console
```

### Updated Device IP Schema (Post-DMZ):

| Device                 | Type       | VLAN | IP Address                                                                | Subnet Mask   | Default Gateway |
| ---------------------- | ---------- | ---- | ------------------------------------------------------------------------- | ------------- | --------------- |
| PLC0                   | PLC        | 10   | 192.168.10.10                                                             | 255.255.255.0 | 192.168.10.1    |
| Sensors and Actuators  | L0 Devices | 10   | —                                                                         | —             | —               |
| PC2 (Engineering WS)   | PC         | 20   | 192.168.20.20                                                             | 255.255.255.0 | 192.168.20.1    |
| PC1 (HMI)              | PC         | 20   | 192.168.20.21                                                             | 255.255.255.0 | 192.168.20.1    |
| Server0 (Historian)    | Server     | 20   | 192.168.20.22                                                             | 255.255.255.0 | 192.168.20.1    |
| PC3 (JumpHost)         | PC         | 30   | 192.168.30.10                                                             | 255.255.255.0 | 192.168.30.1    |
| Patch Server           | Server     | 30   | 192.168.30.11                                                             | 255.255.255.0 | 192.168.30.1    |
| Data Historian (Relay) | Server     | 30   | 192.168.30.12                                                             | 255.255.255.0 | 192.168.30.1    |
| Router0 (ISR4331)      | ROAS       | —    | Gi0/0/0.10=192.168.10.1, Gi0/0/0.20=192.168.20.1, Gi0/0/0.30=192.168.30.1 | 255.255.255.0 | —               |

### Test Matrix

| Test                  | Source → Dest                    | Protocol/Port                               | Expected result                  |
| --------------------- | -------------------------------- | ------------------------------------------- | -------------------------------- |
| Eng WS → JumpHost     | 192.168.20.20 → 192.168.30.10    | TCP 3389                                    | RDP connects (allowed)           |
| Eng WS → Patch Server | 192.168.20.20 → 192.168.30.11    | TCP 443                                     | HTTPS loads (allowed)            |
| Historian → Relay     | 192.168.20.22 → 192.168.30.12    | TCP 443                                     | HTTPS loads (allowed)            |
| JumpHost → Eng WS     | 192.168.30.10 → 192.168.20.20    | any                                         | Fails; ACL deny increments       |
| JumpHost → PLC        | 192.168.30.10 → 192.168.10.10    | any                                         | Fails; ACL deny increments       |
| HMI/WS → PLC          | 192.168.20.21/20 → 192.168.10.10 | TCP 502 / TCP 44818 (+ UDP 2222 if modeled) | PLC reachable (allowed)          |
| Syslog delivery       | Router/Switch → 192.168.30.11    | UDP 514                                     | Events land with timestamps      |
| Time                  | Router (master)                  | NTP                                         | `show ntp status` = synchronized |

### ***Phase 7: Future Work***

#### Formalizing Security with a Zone-Based Firewall:

The next phase of this project will formalize the network’s security policy by migrating from interface-based ACLs to a more powerful Zone-Based Firewall. A ZBF is stateful by nature, which will automatically handle return traffic and simplify rule sets.
#### Expanding the Edge with Wireless/IIoT Connectivity:

To support modern industrial devices, the network will be expanded to include a secure wireless segment for IIoT sensors and mobile operator stations. This will be accomplished by developing a new, dedicated VLAN and WLAN secured with WPA2/3 and 802.1x authentication to prevent unauthorized access.
#### Introducing Redundancy and High Availability:

To protect against costly downtime, the final implementation will introduce redundancy to eliminate single point failure. We will achieve gateway redundancy by adding a second router and configuring the Hot Standby Router Protocol (HSRP), which will provide seamless failover. Concurrently, a second multilayer switch will be deployed to provide switch-level redundancy, with critical devices like the PLC and servers connected to both switches. Spanning Tree Protocol will be optimized to manage these redundant paths and prevent network loops. This resilient design will ensure the network can tolerate a critical device or link failure without impacting manufacturing operations.

### Current Network Snapshot:
<img width="823" height="635" alt="Screenshot 2025-08-17 at 12 13 35 AM" src="https://github.com/user-attachments/assets/c0e6451c-b90c-4f15-bf40-90954ebae93a" />

<img width="401" height="696" alt="Screenshot 2025-08-16 at 11 48 35 PM" src="https://github.com/user-attachments/assets/8da7c63d-0bc3-4162-bcf1-bd6814b0c40d" />



