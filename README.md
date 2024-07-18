
<h1> Two Office Network-Configuration--CCNA-Recap</h1>
<h2>Description</h2>
  In this home lab, I am going to configure a three-tier LAN containing two offices using Cisco Packet Tracer for the virtual configuration. I will complete network configuration including IPv4 and IPv6, static routes, VLANs, spanning tree, OSPF, EtherChannel, DHCP, DNS, NAT, SSH, security features, and wireless configurations. Most configurations will be completed in the Cisco CLI, and wireless LAN configuration will be completed in the Cisco GUI. This project is a capstone project to demonstrate knowledge of all CCNA topics. The project was created by Jeremy of JeremysITLab.
  
<br />
<h2>Utilities Used</h2>

- <b>Cisco Packet Tracer</b>


<h2>Program walk-through:</h2>


<h3>Network Topology</h3>
<p align="center">
<img src="https://i.imgur.com/ZlyHjLe.png" height="80%" width="80%" alt="Topology"/>
</p><br />
<p align="left">This is the provided network topology. The devices are connected appropriately but not configured yet. The network connects to the internet via a dual-homed router at the top center of the image. Below the router, there are two Core Layer, multilayer switches, CSW1 and CSW2. The two core layer switches will be connected via a PaGP ether channel for redundancy and load balancing. Below the Core Switches, there are a total of 4 Distribution Layer, Layer 3 Switches: DSW-A1, DSW-A2, DSW-B1, and DSW-B2. The Access Layer consists of 6 Layer 2 switches: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, and ASW-B3. The Wireless LAN Controller (WLC1) is connected to ASW-A1. Lightweight Access Point (LWAP1) will be used for guest wireless access in Office A, and it is also connected to the access layer via ASW-A1. These Cisco C2960 switches have 24 Fast Ethernet ports, so each switch could have 24 end hosts connected. However, to demonstrate the knowledge of connecting an end host, each switch will only have one end host. In Office B, on the right, a second Lightweight Access Point (LWAP2) is used for guest Wi-Fi. LWAP2 is connected to the access layer via ASW-B1. There is one end host connected to the access layer via ASW-B2. SRV1, located in the bottom right of Office B, will be used as a Domain Name System (DNS) server, File Transfer Protocol (FTP) server, and a Syslog Log server for the network.
<br />
<p align="center">
<img src="https://i.imgur.com/3oSoI7g.png" height="80%" width="80%" alt="Diagram Layers"/>
</p><br />  
<br />
<h3>Part 1: Initial Setup</h3>
<h4>Step 1: Configure appropriate hostnames for each router and switch</h4>
<br/>
  
  ```
enable
configuration terminal
```
  ```
hostname <name>
exit
```
  
<p align="center">
<img src="https://i.imgur.com/XFh94DR.png" height="80%" width="80%" alt="hostname"/>
  <br />
  <img src="https://i.imgur.com/SDTysiV.png" height="80%" width="80%" alt="hostname"/>
</p><br />
<p align="left">In packet tracer, you can access a network device's CLI by double-clicking the device and then clicking the CLI tab. The hostname is configured from the Global config mode. The 'enable' command allows the user to access privileged Exec mode indicated by #. The 'configure terminal' command allows the user to access Global Exec mode, indicated by (config#) 
<br />
<br />
<h4>Step 2: Configure the enable secret jeremysitlab on each router and switch. Use type 9 hashing if available; otherwise, use type 5.</h4>
<br /> 

  ```
enable algorithm-type script secret jeremysitlab
```
or
  ```
enable secret jeremysitlab
```


<p align="center">
<img src="https://i.imgur.com/h7tupSf.png" height="80%" width="80%" alt="level 9 hashing"/>
</p>
<p align="center">
<img src="https://i.imgur.com/yeapZHJ.png" height="80%" width="80%" alt="level 5 hashing"/>
</p><br />
<p align="left">The 'enable secret' and 'enable algorithm-type script secret' commands each provide an added layer of security. These commands set a password for entering privileged EXEC mode, which is stored in a hashed format for security. The enable algorithm-type script secret command, or the type 9 hashing, provides the most secure hashing algorithm on current Cisco devices, but it's not supported on every device type. Type 5 hashing, used by the enable secret command, sets an MD5 hashed password on the device; it's considered less secure than level 9. Using hashing algorithms with passwords is always a good security practice. Even though MD5 hashing is no longer secure, it adds another layer of complexity for someone who may be attempting to tamper with the device. 
<br />
<br />
<h4>Step 3: Configure the user account cisco with secret ccna on each router and switch. Use type 9 hashing if available; otherwise, use type 5 </h4><br/>

   ```
username cisco algorithm-type script secret ccna

```
or
  ```
username cisco secret ccna
```

<p align="center">
<img src="https://i.imgur.com/q37zwUV.png" height="80%" width="80%" alt="u/p level 9"/>
</p>
<p align="center">
<img src="https://i.imgur.com/BQp8nD8.png" height="80%" width="80%" alt="u/p level 5"/>
</p><br />
<p align="left"> 
These commands create a local user account with a username 'cisco' and an encrypted password 'ccna'. Level 9 hashing of the password is preferred, but it is not supported on all devices. When the command 'username cisco algorithm-type script secret ccna' is not accepted on the device, the next preferred option for this lab is 'username cisco secret ccna'.
<br />
<br />
<h4>Step 4: Configure the console line to require login with a local user account. Set a 30-minute inactivity timeout. Enable synchronous logging.</h4>
<br/>
  
  ```
line console 0
  login local
  exec-timeout 30
  logging synchronous
```
<p align="center">
<img src="https://i.imgur.com/KpT2OCm.png" height="80%" width="80%" alt="console line config"/>
</p>
<p align="left"> 
The 'line console 0' command is used to enter the console line configuration mode on a Cisco device. This mode allows you to configure settings for the console line, which is the physical console port on the device. The 'login local' command allows usernames and passwords configured locally on the device can be used to log in. The 'exec-timeout 30' command signs out the signed-in user after 30 minutes of inactivity. The 'logging synchronous' command ensures that system messages do not interrupt your command input on the console, logging synchronous reprints your current command line after each log message. The step is repeated on each router and switch.
<br /> 
<br /> 

<h3>Part 2: VLANs, Layer-2 EtherChannel</h3>
<h4>Step 1: Configure a Layer-2 EtherChannel named PortChannel1 between DSW-A1 and DSW-A2 using a Cisco-proprietary protocol. Both switches should actively try to form an EtherChannel.</h4> 
<br/>

  ```
interface range g1/0/4-5
  channel-protocol pagp
  channel-group 1 mode desirable
```
<p align="center">
<img src="https://i.imgur.com/P6VctL5.png" height="80%" width="80%" alt="PAgP EtherChannel"/>
</p>
<p align="left"> 
This command configures a Layer 2 EtherChannel on a Cisco switch using the Port Aggregation Protocol (PAgP). PAgP is a Cisco proprietary protocol used to form an EtherChannel. EtherChannels bundle multiple physical links into a single logical link, providing redundancy, load balancing, and simplified network management, along with other benefits. The interface range command allows for grouping interfaces to be configured simultaneously, rather than configuring each interface individually. The 'channel-protocol pagp' line sets PAgP as the EtherChannel negotiation protocol for the group of interfaces; however, this line is not necessary. Finally, 'channel-group 1 mode desirable' adds the selected interfaces to EtherChannel group 1 in desirable mode, which actively tries to form an EtherChannel using PAgP. These commands need to be entered on both DSW-A1 and DSW-A2 only.
<br /> 
<br /> 
<h4>Step 2: Configure a Layer-2 EtherChannel named PortChannel1 between DSW-B1 and DSW-B2 using an open standard protocol. Both switches should actively try to form an EtherChannel.</h4><br />

  ```
interface range g1/0/4-5
  channel-protocol lagp
  channel-group 1 mode active
```
<p align="center">
<img src="https://i.imgur.com/DoKcwgb.png" height="80%" width="80%" alt="LACP EtherChannel"/>
</p>
<p align="left"> 
This command configures a Layer 2 EtherChannel on a switch using the Ling Aggregation Protocol (LACP). LACP is an open standard protocol used to form an EtherChannel, LACP is defined in the IEEE 802.3ad.  The 'channel-protocol lacp' line sets LACP as the EtherChannel negotiation protocol for the group of interfaces; it is not necessary. Finally, 'channel-group 1 mode active' adds the selected interfaces to EtherChannel group 1 in active mode, which actively tries to form an EtherChannel using LACP. These commands need to be entered on both DSW-B1 and DSW-B2 only.
<br /> 
<br /> 
<h4>Step 3: Configure all links between Access and Distribution switches, including the EtherChannels, as trunk links. Disable DTP on all ports. Set each trunk's native VLAN to VLAN 1000(unused). In Office A, allow VLANs 10, 20, 40, and 99 on all trunks.In Office B, allow VLANs 10, 20, 30, and 99 on all trunks.</h4>
<br />
  
Office A Access Layer:
  ```
interface range g0/1-2
  switchport mode trunk
  switchport nonegotiate
  switchport trunk native vlan 1000
  switchport trunk allowed vlan 10,20,40,99
```
<p align="center">
<img src="https://i.imgur.com/qQ3y9dq.png" height="80%" width="80%" alt="LACP EtherChannel"/>
</p>

Office B Access Layer:
  ```
interface range g0/1-2
  switchport mode trunk
  switchport nonegotiate
  switchport trunk native vlan 1000
  switchport trunk allowed vlan 10,20,30,99
```
<p align="center">
<img src="https://i.imgur.com/O1otgd2.png" height="80%" width="80%" alt="LACP EtherChannel"/>
</p>
<p align="left"> This configuration establishes two trunk ports that will not negotiate trunking dynamically, defines a native VLAN (VLAN 1000), and restricts the allowed VLANs for the trunk. A Virtual Local Area Network (VLAN) is a logical grouping of devices within a larger physical network, enabling network segmentation. A trunk is used when you need to carry multiple VLANs on an interface. Using the command 'switchport nonegotiate' ensures that the trunk will not attempt to negotiate with neighboring devices, providing more control and reducing potential misconfigurations. The command 'switchport trunk native vlan 1000' sets the native VLAN to an unused VLAN, ensuring that all packets are tagged with the source VLAN, preventing them from being inadvertently sent to an unintended destination. This reduces the risk of VLAN hopping and minimizes security risks. The command 'switchport trunk allowed vlan 10,20,40,99' used in Office A and 'switchport trunk allowed vlan 10,20,30,99' used in Office B specify which VLANs are allowed to be sent on the trunk. If a packet is not tagged with one of these VLANs for the respective offices, it is dropped. Office A commands are used on ASW-A1, ASW-A2, and ASW-A3. Office B commands are used on ASW-B1, ASW-B3, ASW-B3.
  
Office A Distribution Layer:
  ```
interface range g1/0/1-3
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99
```
  ```
interface po1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99
```

Office B Distribution Layer:
  ```
interface range g1/0/1-3
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99
```
  ```
interface po1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99
```

The Office A commands are used on switches DSW-A1 and DSW-A2, while the Office B commands are used on switches DSW-B1 and DSW-B2. They serve the same purpose as described above but operate in the opposite direction, from the distribution layer switches to the access layer switches. The interface po1 command is used to configure the EtherChannel, allowing traffic to be sent through it as well. During these configurations, you may receive several Syslog CDP warnings about native VLAN mismatches. This is normal while completing this step. Once all switches are configured appropriately, the Syslog warnings will stop.
<br /> 
<br /> 

<h4>Step 4: Configure one of each office’s Distribution switches as a VTPv2 server. Use the domain name JeremysITLab. Verify that other switches join the domain. Configure all Access switches as VTP clients.</h4>
Distribution Layer Switches:

  ```
vtp domain JeremysITLab
vtp version 2
```
<p align="center">
<img src="https://i.imgur.com/NQj2kwv.png" height="80%" width="80%" alt="VTP Server"/>
</p><br />
<p align="left"> The default VLAN Trunking Protocol (VTP) is a Cisco proprietary protocol used to manage VLAN configurations. VTP is enabled by default. Switches are in VTP server mode by default, which allows for the creation, modification, and deletion of VLANs within the VTP domain. The 'vtp domain JeremysITLab' command creates or joins other switches on the network into a group or domain to share VTP information. The 'vtp version 2' command ensures each switch utilizes the same version of VTP. These commands should be repeated on the distribution layer switches: DSW-A1, DSW-A2, DSW-B1, and DSW-B2.
<br />
Access Layer Switches:

  ```
vtp mode client
vtp domain JeremysITLab
vtp version 2
```
<p align="center">
<img src="https://i.imgur.com/Ltl2m4s.png" height="80%" width="80%" alt="VTP Client"/>
</p><br />
<p align="left"> The command 'vtp mode client' changes the VTP permissions for the switch. VTP client switches receive VLAN information from VTP servers and apply it to their configuration. They cannot create, modify, or delete VLANs. The command 'vtp domain JeremysITLab' joins the switch to a VTP domain named JeremysITLab, allowing it to share VTP information with other switches in the same domain. The command 'vtp version 2' ensures that each switch utilizes the same version of VTP. These commands should be repeated on the access layer switches: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, and ASW-B3.
<br />
<br />
<h4>Step 5: In Office A, create and name the following VLANs on one of the Distribution switches. Ensure that VTP propagates the change. VLAN 10: PCs. LAN 20: Phones. VLAN 40: Wi-Fi. VLAN 99: Management.</h4>
<br/>
Office A:
  
  ```
vlan 10
name PCs
vlan 20
name Phones
vlan 40
name wi-fi
vlan 99
name Management
```
<p align="center">
<img src="https://i.imgur.com/Z9oGuoy.png" height="80%" width="80%" alt="name vlans"/>
</p><br /> 
<p align="left">The command 'vlan 10' creates a VLAN with that number. Then the 'name PCs' command names that vlan. These commands need to be entered on one of Office A's VTP server switches, either DSW-A1 or DSW-A2. It will send VTP advertisements to other members of the same VTP domain. VTP advertisements are only shared via trunks, currently, the configurations between the distribution layer and core layers are access ports, so VTP information in Office A will not be shared with Office B and vice versa.
<br />
<br />
<h4>Step 6: In Office B, create and name the following VLANs on one of the Distribution switches. Ensure that VTP propagates the change. VLAN 10: PCs. LAN 20: Phones. VLAN 30: Servers. VLAN 99: Management.</h4><br/>
Office B:
  
  ```
vlan 10
name PCs
vlan 20
name Phones
vlan 30
name Servers
vlan 99
name Management
```

<p align="center">
<img src="https://i.imgur.com/JIikWct.png" height="80%" width="80%" alt="create/name vlans"/>
</p><br /> 
<p align="left">The above commands create and name each VLAN in Office B. The above command only needs to be entered on one of the VTP servers. VTP advertisements will be sent to each of the other switches within the VTP domain in Office B.
<br />
<br />
<h4>Step 7: Configure each Access switch’s access port. LWAPs will not use FlexConnect. PCs in VLAN 10, Phones in VLAN 20. SRV1 in VLAN 30. Manually configure access mode and explicitly disable DTP.</h4>
<br/>

 ASW-A1 and ASW-B1 
  ```
interface f0/1
switchport mode access
switchport nonegotiate
switchport access vlan 99
```
<br />
<p align="left"> Office B does not have a Wi-Fi Vlan because LWAPs only need access to the Management VLAN. They will tunnel information to the WLC in office A, then the traffic will be assigned to VLAN 40. So the ports to the LWAPs can be configured as access ports, not trunks. The 'interface f0/1' command enters interface configuration mode. 'switchport mode access' makes this port an access port. 'switchport nonegotiate' disables Dynamic Trunking Protocol (DTP), although it's redundant because when an interface becomes an access port no more DTP messages will be sent from the interface. Enter the above commands on both ASW-A1 in Office A and ASW-B1 in Office B

ASW-A2, ASW-A3 and ASW-B2 
  ```
interface f0/1
switchport mode access
switchport nonegotiate
switchport access vlan 10
switchport voice vlan 20
```

<p align="center">
<img src="https://i.imgur.com/mSqu8QO.png" height="80%" width="80%" alt="Access VLANs"/>
</p><br />
<p align="left"> The above command makes int f0/1 an access port, join VLAN 10 and VLAN 20, and disables DTP.
<br />
ASW-B3
  
  ```
interface f0/1
switchport mode access
switchport nonegotiate
switchport access vlan 30
```
<p align="center">
<img src="https://i.imgur.com/gm7U1fX.png" height="80%" width="80%" alt="Access VLAN"/>
</p><br />
<p align="left"> This image shows the output of 'show vlan brief' with interface fa0/1 in Server VLAN.
<br />
<br />
<h4>Step 8: Configure ASW-A1’s connection to WLC1.  It must support the Wi-Fi and Management VLANs. The Management VLAN should be untagged. Disable DTP.</h4>
<br />

 Office A ASW-A1
  ```
interface f0/2
switchport mode trunk
switchport trunk allowed vlan 40,99
switchport trunk native vlan 99
switchport nonegotiate
```
<align="left"> These commands have already been explained in previous steps. This command is only entered on ASW-A1, in Office A, which connects to the WLC. Interface f0/2 is connected to WLC1 and it needs to support multiple VLANS so it is configured as a trunk. It needs access to Wi-Fi VLAN and the management VLAN so both are allowed over the trunk with the 'switchport trunk allowed vlan 40,99'. We are asked to have the management VLAN traffic be untagged, so the native VLAN for the trunk is assigned to VLAN 99, the Management VLAN. And we are asked to disable DTP, so the 'switchport nonegotiate' command is issued.
<br />
<br />
<h4>Step 9:  Administratively disable all unused ports on the Access and Distribution switches.</h4>
<br />
Access and Distribution Layer Switches:

  ```
show interfaces status
```

  ```
interface range g1/0/6-24,g1/1/3-4
shutdown
```
<p align="center">
<img src="https://i.imgur.com/AsosK2s.png" height="80%" width="80%" alt="Shutdown interfaces"/>
</p><br />
<p align="left"> Shutting down all unused ports is a good security practice. Switches will automatically allow new physical connections to the switch to connect to the network. Administratively shutting down the interface, will not allow access to the network. This step needs to be repeated on each switch in Office A and Office B: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, ASW-B3, DSW-A2, DSW-B1 and DSW-B2. The 'show interfaces status' command should be used on each switch to determine which interfaces to use with the 'interface range' command.
<br />
<br />
<h3>Part 3: IP Addresses, Layer-3 EtherChannel, HSRP </h3>
<h4>Step 1: Configure the following IP addresses on R1’s interfaces and enable them:</h4>
<br />

- <b>G0/0/0: DHCP client</b>
- <b>G0/1/0: DHCP client</b>
- <b>G0/0: 10.0.0.33/30</b>
- <b>G0/1: 10.0.0.37/30</b>
- <b>Loopback0: 10.0.0.76/32</b>

 ```
interface g0/0/0
 ip address dhcp
 no shutdown

interface g0/1/0
 ip address dhcp
 no shutdown

interface g0/0
 ip address 10.0.0.33 255.255.255.252
 no shutdown

interface g0/1
 ip address 10.0.0.37 255.255.255.252
 no shutdown

interface l0
 ip address 10.0.0.76 255.255.255.255

```

<p align="left">At this step, we are beginning to assign static IPv4 addresses. Generally, in your network, it's good practice to set static IP addresses on network interfaces so that you can have set addresses to diagnose network errors. The external-facing interfaces, G0/0/0 and G0/1/0, use Dynamic Host Configuration Protocol (DHCP) to have their interfaces assigned an address from the Internet Service Provider that they are connected to. Later, Network Address Translation (NAT) will be applied so that internal devices will be able to reach the Internet.

From Global Configuration mode, enter the command 'interface `<interface #>`' to enter interface configuration mode (config-if)#, followed by the command 'ip address `<ip address>` `<netmask>`' to assign a static IP address. The command 'no shutdown' needs to be used on each interface because router interfaces are down by default, unlike switches. The 'no shutdown' command is not needed on the loopback interface l0 because the command int l0 creates a virtual loopback interface, which is active by default. Now, the output of the command 'do show ip interface brief' from Global configuration mode should look like this:</p>
<p align="center">
<img src="https://i.imgur.com/SAq9sCL.png" height="80%" width="80%" alt="R1 ipv4 interfaces"/>
</p>
<br />
<br />
<h4>Step 2: Enable IPv4 routing on all Core and Distribution switches</h4>
<br />

 ```
ip routing
```

<p align="center">
<img src="https://i.imgur.com/jdaNlgB.png" height="80%" width="80%" alt="ip routing"/>
</p>
<p align="left">This command needs to be entered on all core and distribution switches: CSW1, CSW2, DSW-A1, DSW-A2, DSW-B1, and DSW-B2. Switches, by default, utilize Layer 2 addressing (MAC addresses). They do not use or need IP addresses or have an IP routing table. These core and distribution switches are all Layer 3 or multilayer switches. The command IP routing allows each of these switches to utilize Layer 3 addressing (IP protocols) while maintaining their Layer 2 functions. Now they can build IP routing tables.
<br />
<br />
<h4>Step 3: Create a Layer-3 EtherChannel between CSW1 and CSW2 using a Cisco-proprietary protocol. Both switches should actively try to form an EtherChannel. Configure the following IP addresses:</h4>

- <b>CSW1 PortChannel1: 10.0.0.41/30</b>
- <b>CSW2 PortChannel1: 10.0.0.42/30</b>

CSW1:

   ```
interface range g1/0/2-3
no switchport
channel-group 1 mode desirable
interface po1
ip address 10.0.0.41 255.255.255.252
```
CSW2:

  ```
interface range g1/0/2-3
no switchport
channel-group 1 mode desirable
interface po1
ip address 10.0.0.42 255.255.255.252
```
<p align="center">
<img src="https://i.imgur.com/44CGceb.png" height="80%" width="80%" alt="ip routing"/>
</p><br />
<p align="left">The above commands create a Layer 3 EtherChannel using PAgP. The first line of the command groups the interfaces in interface configuration mode. The command 'no switchport' disables Layer 2 addressing on the ports, turning them into Layer 3 ports. The command 'channel-group 1 mode desirable' creates an EtherChannel port using PAgP mode desirable, which actively tries to create an EtherChannel. The final two lines, starting with 'interface po1', enter configuration mode for the PortChannel 1 interface and assign an IP address so the EtherChannel can communicate with other Layer 3 network devices.
<br />
<br />
</h4>Step 4: Configure the following IP addresses on CSW1:</h4>
  
- <b>G1/0/1: 10.0.0.34/30</b>  
- <b>G1/1/1: 10.0.0.45/30</b>
- <b>G1/1/2: 10.0.0.49/30</b>
- <b>G1/1/3: 10.0.0.53/30</b>  
- <b>G1/1/4: 10.0.0.57/30 </b>  
- <b>Loopback0: 10.0.0.77/32</b>
  
Disable all unused interfaces.

  ```
interface g1/0/1
ip address 10.0.0.34 255.255.255.252
interface g1/1/1
ip address 10.0.0.45 255.255.255.252
interface g1/1/2 
ip address 10.0.0.49 255.255.255.252
interface g1/1/3
ip address 10.0.0.53 255.255.255.252
interface g1/1/4
ip address 10.0.0.57 255.255.255.252
interface L0
ip address 10.0.0.77 255.255.255.255
interface range g1/0/4-24
shutdown
```


<p><p align="left">These commands assign IP addresses to individual interfaces. The command 'show ip int br | include unassigned' was used to identify which interfaces are not being used before entering the interface range. Interfaces g1/0/2-3 were excluded from the shutdown command as they are used for the EtherChannel. It is good security practice to administratively shut down switch interfaces that are not in use. Additionally, moving these switches to an unused VLAN, enabling port security, and disabling CDP or LLDP are also good security measures for unused ports. The IP Interface brief should now look like this :
<p align="center">
<img src="https://i.imgur.com/KKH5Ieq.png" height="80%" width="80%" alt="ip routing"/>
</p><br />
<br />
<h4>Step 5: Configure the following IP addresses on CSW2:</h4>
  
- <b>G1/0/1: 10.0.0.38/30</b>  
- <b>G1/1/1: 10.0.0.61/30</b>
- <b>G1/1/2: 10.0.0.65/30</b>
- <b>G1/1/3: 10.0.0.69/30</b>  
- <b>G1/1/4: 10.0.0.73/30 </b>  
- <b>Loopback0: 10.0.0.78/32</b>
  
Disable all unused interfaces.

  ```
interface g1/0/1
ip address 10.0.0.38 255.255.255.252
interface g1/1/1
ip address 10.0.0.61 255.255.255.252
interface g1/1/2 
ip address 10.0.0.65 255.255.255.252
interface g1/1/3
ip address 10.0.0.69 255.255.255.252
int g1/1/4
ip address 10.0.0.73 255.255.252
interface L0
ip address 10.0.0.78 255.255.255.255
interface range g1/0/4-24
shutdown
```


<p align="left">Same commands for CSW1, but the IP addresses have been updated. It's a good security practice to administratively shut down switch interfaces that are not being used. Additionally, moving these switches to an unused VLAN, enabling port security, and disabling CDP or LLDP are also good security measures for unused ports. The IP Interface brief should now look like this :
<p align="center">
<img src="https://i.imgur.com/kucSUUK.png" height="80%" width="80%" alt="ip routing"/>
</p><br />
<br />
<h4>Step 6: Configure the following IP addresses on DSW-A1:</h4>
  
- <b>G1/1/1: 10.0.0.46/30</b>
- <b>G1/1/2: 10.0.0.62/30/<b> 
- <b>Loopback0: 10.0.0.7/32</b>

 ```
interface g1/1/1
ip address 10.0.0.46 255.255.255.252
int g1/1/2
ip address 10.0.0.62 255.255.255.252
interface L0
ip address 10.0.0.79 255.255.255.255
```


<p align="left">These are the IP interface configurations for DSW-A1. The IP Interface brief should now look like this :
<p align="center">
<img src="https://i.imgur.com/a7fl291.png" height="80%" width="80%" alt="dsw-A1"/>
</p><br />
<h4>Step 7: Configure the following IP addresses on DSW-A2: </h4>
  
- <b>G1/1/1: 10.0.0.50/30</b>
- <b>G1/1/2: 10.0.0.66/30</b>
- <b>Loopback0: 10.0.0.80/32</b>
  

```
interface g1/1/1
ip address 10.0.0.50 255.255.255.252
int g1/1/2
ip address 10.0.0.66 255.255.255.252
interface L0
ip address 10.0.0.80 255.255.255.255
```


<p align="left">Entering these IP addresses is very tedious, but they must be all entered correctly along with correct netmasks. The IP Interface brief should now look like this :
<p align="center">
<img src="https://i.imgur.com/xdU65i8.png" height="80%" width="80%" alt="dsw-a2"/>
</p><br />
<br />
<h4>Step 8: Configure the following IP addresses on DSW-B1:</h4>
  
- <b>G1/1/1: 10.0.0.54/30</b>
- <b>G1/1/2: 10.0.0.70/30</b>
- <b>Loopback0: 10.0.0.81/32</b>
  

```
interface g1/1/1
ip address 10.0.0.54 255.255.255.252
int g1/1/2
ip address 10.0.0.70 255.255.255.252
interface L0
ip address 10.0.0.81 255.255.255.255
```

<p align="left"> The IP Interface brief on DSW-B1 should now look like this :
<p align="center">
<img src="https://i.imgur.com/nJ4M81r.png" height="80%" width="80%" alt="dsw-b1"/>
</p><br />
<br />
<h4>Step 9: Configure the following IP addresses on DSW-B2: </h4>
<br />
  
- <b>G1/1/1: 10.0.0.58/30</b>
- <b>G1/1/2: 10.0.0.74/30</b>
- <b>Loopback0: 10.0.0.82/32</b>
  

```
interface g1/1/1
ip address 10.0.0.58 255.255.255.252
int g1/1/2
ip address 10.0.0.74 255.255.255.252
interface L0
ip address 10.0.0.82 255.255.255.255
```


I made typos while entering some of these interface configurations, so it's good to check your work by using the command 'do show ip interface brief' from global config mode or 'show ip interface brief' from privileged exec mode. If any typos were made, use 'interface `<# of interface with error>`' then 'no ip address `<incorrect address>` `<netmask>`' commands to delete the entry. Then reenter the correct IP address. It is also good to use the 'write' command from privileged exec mode or 'do write' from global config mode to save your work. These commands write the running configuration to the startup configuration, saving your work.
<br /> 
<p align="left">The IP Interface brief on DSW-B2 should now look like this :
<p align="center">
<img src="https://i.imgur.com/7ut7p1V.png" height="80%" width="80%" alt="dsw-b2"/>
</p><br />
<br />
<h4>Step 10: Manually configure SRV1’s IP settings:</h4> 
  
- <b>Default Gateway: 10.5.0.1</b>
- <b>IPv4 Address: 10.5.0.4</b>
- <b>Subnet Mask: 255.255.255.0</b>
  
<p align="left">To enter the configuration setting on end hosts, in packet tracer, you must use the GUI to manually set the configuration.
<br />
<p align="center">
<img src="https://i.imgur.com/RII6kW3.png" height="80%" width="80%" alt="SRV1"/>
<img src="https://i.imgur.com/CDEaAJB.png" height="80%" width="80%" alt="SRV1"/>
</p><br />
<br />
<h4>Step 11: Configure the following management IP addresses on the Access switches (interface VLAN 99):</h4>

- <b>ASW-A1: 10.0.0.4/28</b>
- <b>ASW-A2: 10.0.0.5/28</b>
- <b>ASW-A3: 10.0.0.6/28</b>
- <b>ASW-B1: 10.0.0.20/28</b>
- <b>ASW-B2: 10.0.0.21/28</b>
- <b>ASW-B3: 10.0.0.22/28</b>

And configure the appropriate subnet’s first usable address as the default gateway.<br />

```
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.4 255.255.255.240
```
ASW-A2:

  ```
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.5 255.255.255.240
```
ASW-A3:

  ```
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.6 255.255.255.240
```
<p align="center">
<img src="https://i.imgur.com/V1VATBg.png" height="80%" width="80%" alt="asw-a2"/>
</p><br />
ASW-B1:

  ```
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.20 255.255.255.240
```
ASW-B2:

  ```
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.21 255.255.255.240
```

ASW-B3:

   ```
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.22 255.255.255.240

```
<p align="center">
<img src="https://i.imgur.com/n4MbmL8.png" height="80%" width="80%" alt="asw-b2"/>
</p><br />
<p align="left">These interface configurations create virtual interfaces that allow the management VLAN to communicate with these access layer, layer 2 switches. The 'default-gateway' command is added to the configuration because layer 2 switches do not have routing tables, they do not know where to send ip traffic. The 'default-gateway' command tells the switch where to forward IP traffic. Switches in Office A and Office B have different default gateway addresses because each office has a netmask of /28 meaning there are 16 total addresses. For office A, 10.0.0.0 is the network address and 10.0.0.15 is the broadcast address making 10.0.0.1 the first usable address. For Office B, 10.0.0.16 is the network address and 10.0.0.31 is the broadcast address making 10.0.0.17 the first usable address.
<br />
<br />
<h4>Step 12: Configure HSRPv2 group 1 for Office A’s Management subnet (VLAN 99). Make DSW-A1 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-A1.</h4>
  
  - <b>Subnet: 10.0.0.0/28</b> 
  - <b>VIP: 10.0.0.1</b> 
  - <b>DSW-A1: 10.0.0.2</b> 
  - <b>DSW-A2: 10.0.0.3</b>
  
 DSW-A1:
   ```
interface vlan 99
ip address 10.0.0.2 255.255.255.240
standby version 2
standby 1 ip 10.0.0.1
standby 1 priority 105
standby 1 preempt
```
<p align="center">
<img src="https://i.imgur.com/ghXifzc.png" height="80%" width="80%" alt="asw-b2"/>
</p><br />

DSW-A2:
   ```
interface vlan 99
ip address 10.0.0.3 255.255.255.240
standby version 2
standby 1 ip 10.0.0.1
```
<p align="center">
<img src="https://i.imgur.com/IpQEeju.png" height="80%" width="80%" alt="asw-b2"/>
</p><br />
These configurations access VLAN 99, assign an IP address, set Hot Standby Redundancy Protocol version 2 (HSRPv2), and assign the virtual IP address the switches will share. HSRP versions 1 and two are not compatible, so make sure both switches are using standby version 2 with the command 'standby version 2'. A virtual IP(VIp) address differs from a standard IP address because both switches are going to share the virtual address, in this case, the VIp is set with the 'standby1 ip 10.0.0.1' command in HSRP group 1. When configuring HSRP each switch has a priority of 100. The directions ask us to make DSW-A1 the active router by increasing the priority by 5 above default, the command 'standby 1 priority 105' does this. First-hop redundancy protocols(FHRP) are a way of load balancing (if configured), and providing redundancy in the network by having a primary and second router. If the primary router fails or goes down, the secondary router will become the primary router to ensure traffic can still traverse the network. HSRP is a Cisco proprietary FHRP, the active router is elected by the router with the highest priority, in this case DSW-A1 priority of 105, or the router with the highest IP address. In this case, DSW-A2 is the standby router, which will take over forwarding traffic if DSW-A1 fails. The command 'standby preempt' will force an Active election if it comes back online. If this line was not added, and when down, DSW-A2 would become the active route and stay the active router, until it went down or another router joined the standby group with preemption enabled.
<br />
These switches can act as routers because they are multilayer switches. They can act as both switches and routers.
<br />
<br />
<h4>Step 13. Configure HSRPv2 group 2 for Office A’s PCs subnet (VLAN 10). Make DSW-A1 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-A1.</h4> 

- <b>Subnet 10.1.0.0/24</b>
- <b>VIP: 10.1.0.1</b>
- <b>DSW-A1: 10.1.0.2</b>
- <b>DSW-A2: 10.1.0.3</b>

DSW-A1:
   ```
interface vlan 10
ip address 10.1.0.2 255.255.255.0
standby version 2
standby 2 ip 10.1.0.1
standby 2 priority 105
standby 2 preempt
```
<p align="center">
<img src="https://i.imgur.com/WLTTP9R.png" height="80%" width="80%" alt="v10a1"/>
</p><br />

DSW-A2:
   ```
interface vlan 10
ip address 10.1.0.3 255.255.255.0
standby version 2
standby 2 ip 10.1.0.1
```
<p align="center">
<img src="https://i.imgur.com/hlxsoWA.png" height="80%" width="80%" alt="v10a2"/>
</p><br />
These commands do the same thing as step 12 but for the PCs subnet. The standby group is changed to standby group 2 to allow VLAN separation, redundancy, and failover. This also allows for a separate VIP to be assigned just for this specific VLAN. Separating each VLAN into its own standby groups allows network traffic to still flow if there is a misconfiguration or failure in one VLAN because each VLAN will have its traffic sent separately from each other VLAN.
<br />
<br />
<h4>Step 14. Configure HSRPv2 group 3 for Office A’s Phones subnet (VLAN 20). Make DSW-A2 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-A2.</h4>

- <b>Subnet: 10.2.0.0/24</b> 
- <b>VIP: 10.2.0.1</b>
- <b>DSW-A1: 10.2.0.2</b>
- <b>DSW-A2: 10.2.0.3</b>

DSW-A1:
   ```
interface vlan 20
ip address 10.2.0.2 255.255.255.0
standby version 2
standby 3 ip 10.2.0.1

```
<p align="center">
<img src="https://i.imgur.com/ufOBlry.png" height="80%" width="80%" alt="v20a1"/>
</p><br />

DSW-A2:
   ```
interface vlan 20
ip address 10.2.0.3 255.255.255.0
standby version 2
standby 3 ip 10.2.0.1
standby 3 priority 105
standby 3 preempt
```
<p align="center">
<img src="https://i.imgur.com/ImSobvT.png" height="80%" width="80%" alt="v20a1"/>
</p><br />
These configurations perform the same function but with updated IP addresses making DSW-A2 the Active router and DSW-A1 the standby router. This is the first example of how HSRPv2 can provide load balancing. The Phones subnet will prioritize sending traffic via DSW-A2 instead of DSW-A1 like the management and pcs subnet.
<br />
<br />
<h4>Step 15. Configure HSRPv2 group 4 for Office A’s Wi-Fi subnet (VLAN 40). Make DSW-A2 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-A2.</h4>

- <b>Subnet: 10.6.0.0/24</b>
- <b>VIP: 10.6.0.1  </b>
- <b>DSW-A1: 10.6.0.2</b>
- <b>DSW-A2: 10.6.0.3</b>

DSW-A1:
   ```
interface vlan 40
ip address 10.6.0.2 255.255.255.0
standby version 2
standby 4 ip 10.6.0.1

```
<p align="center">
<img src="https://i.imgur.com/G6O3f7F.png" height="80%" width="80%" alt="v20a1"/>
</p><br />

DSW-A2:
   ```
interface vlan 40
ip address 10.6.0.3 255.255.255.0
standby version 2
standby 4 ip 10.6.0.1
standby 4 priority 105
standby 4 preempt
```
<p align="center">
<img src="https://i.imgur.com/iCK2dFA.png" height="80%" width="80%" alt="v20a1"/>
</p><br />
Wi-Fi traffic (VLAN 40) will utilize DSW-A2 as the primary router, and DSW-A1 as the standby router. We can also see the Wi-Fi subnet will allow 244 hosts because of the /24 subnet mask. While in the previous configurations, the VLANs would only allow 14 hosts per subnet because of the /28 subnet mask. I hope this shows how HSRP provides redundancy, fail-over, and load balancing which are all good things to build into your network. Networks are expected to run 24/7, and building redundancy into the network is how to do so. We are done configuring HSRP for Office A.
<br />
<br />
<h4>Step 16: Configure HSRPv2 group 1 for Office B’s Management subnet (VLAN 99). Make DSW-B1 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-B1.</h4>

- <b>Subnet: 10.0.0.16/28</b>
- <b>VIP: 10.0.0.17</b>
- <b>DSW-B1: 10.0.0.18</b>
- <b>DSW-B2: 10.0.0.19</b>

DSW-B1:

   ```
interface vlan 99
ip address 10.0.0.18 255.255.255.240
standby version 2
standby 1  ip 10.0.0.17
standby 1 priority 105
standby 1 preempt
```
<p align="center">
<img src="https://i.imgur.com/B28PcDG.png" height="80%" width="80%" alt="v20a1"/>
</p><br />

DSW-B2:
  ```
interface vlan 99
ip address 10.0.0.19 255.255.255.240
standby version 2
standby 1 ip 10.0.0.17
```
<p align="center">
<img src="https://i.imgur.com/xtX7W5x.png" height="80%" width="80%" alt="v20a1"/>
</p><br />
These configurations are similar to step 11 but in the Office B network. DSW-B1 Active, DSW-B2 standby for the Management VLAN
<br />
<br />
<h4>Step 17. Configure HSRPv2 group 2 for Office B’s PCs subnet (VLAN 10). Make DSW-B1 the active router by increasing its priority to 5 above the default, and enable preemption on DSW-B1.</h4>

- <b>Subnet: 10.3.0.0/24</b>
- <b>VIP: 10.3.0.1</b>
- <b>DSW-B1: 10.3.0.2</b>
- <b>DSW-B2: 10.3.0.3</b>

DSW-B1:

   ```
interface vlan 10
ip address 10.3.0.2 255.255.255.0
standby version 2
standby 2  ip 10.3.0.1
standby 2 priority 105
standby 2 preempt
```
<p align="center">
<img src="https://i.imgur.com/spcqYvd.png" height="80%" width="80%" alt="v20a1"/>
</p><br />

DSW-B2:
  ```
interface vlan 10
ip address 10.3.0.3 255.255.255.0
standby version 2
standby 2 ip 10.3.0.1
```
These are the same configurations as step 13, but in the 10.3.0.0 subnet, allowing 244 hosts in Office B. DSW-B1 is the active router and DSW-B2 is standby router in the PCs subnet.
<br />
<br />
<h4>Step 18. Configure HSRPv2 group 3 for Office B’s Phones subnet (VLAN 20). Make DSW-B2 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-B2.</h4>

- <b>Subnet: 10.4.0.0/24</b>
- <b>VIP: 10.4.0.1</b>
- <b>DSW-B1: 10.4.0.2</b>
- <b>DSW-B2: 10.4.0.3</b>

DSW-B1:

   ```
interface vlan 20
ip address 10.4.0.2 255.255.255.0
standby version 2
standby 3  ip 10.4.0.1
```
<p align="center">
<img src="https://i.imgur.com/vAbl9pt.png" height="80%" width="80%" alt="v20a1"/>
</p><br />

DSW-B2:
   ```
interface vlan 20
ip address 10.4.0.3 255.255.255.0
standby version 2
standby 3 ip 10.4.0.1
standby 3 priority 105
standby 3 preempt
```
<p align="center">
<img src="https://i.imgur.com/ubUXDkr.png" height="80%" width="80%" alt="v20a1"/>
</p><br />
Another example of load balancing. The Phones subnet will use DSW-B2 as the active router, and DSW-B1 as the standby router.
<br />
<br />
<h4>Step 19. Configure HSRPv2 group 4 for Office B’s Servers subnet (VLAN 30). Make DSW-B2 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-B2.</h4>

- <b>Subnet: 10.5.0.0/24</b>
- <b>VIP: 10.5.0.1</b>
- <b>DSW-B1 10.5.0.2</b>
- <b>DSW-B2: 10.5.0.3</b>

DSW-B1:

   ```
interface vlan 30
ip address 10.5.0.2 255.255.255.0
standby version 2
standby 4  ip 10.5.0.1
```
<p align="center">
<img src="https://i.imgur.com/e57pk7R.png" height="80%" width="80%" alt="v20a1"/>
</p><br />

DSW-B2:
  ```
interface vlan 30
ip address 10.5.0.3 255.255.255.0
standby version 2
standby 4 ip 10.5.0.1
standby 4 priority 105
standby 4 preempt
```
<p align="center">
<img src="https://i.imgur.com/DTmnJui.png" height="80%" width="80%" alt="v20a1"/>
</p><br />
That's it for configuring HSRPv2 in Office B. The command 'logging synchronous' command entered earlier is very helpful for this stage. If you mix up the VLAN ip address and the VIp you will get tons of Syslog message indicating misconfigurations. If you get those syslog errors, hit the up arrow to repeat the last command but put 'no' at the beginning of the line to undo the last command. Typos will certainly create Syslog messages during HSRP configurations. Configure the HSRP Standby Router for each VLAN with an STP priority one increment above the lowest priority.
<br />
<br />
<h3>Part 4 - Rapid Spanning Tree Protocol</h3>
<h4>Step 1: Configure Rapid PVST+ on all Access and Distribution switches. Configure Rapid PVST+ on all Access and Distribution switches. Configure Rapid PVST+ on all Access and Distribution switches. Ensure that the Root Bridge for each VLAN aligns with the HSRP Active router by configuring the lowest possible STP priority.</h4>

   ```
spanning-tree mode rapid-pvst
```
<p align="center">
<img src="https://i.imgur.com/ETI9pf2.png" height="80%" width="80%" alt="v20a1"/>
</p><br />
This command needs to be entered on each Access and Distribution Switch: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, ASW-B3, DSW-A1, DSW-A2, DSW-B1 and DSW-B2. This enables Rapid Per VLAN Spanning Treet (RPVST+). Rapid PVST+ is a Cisco improvement over Rapid Spanning Tree. Spanning tree protocols intend to prevent Layer 2 Broadcast storms by blocking redundancy ports. Spanning Tree reduces network traffic, to keep networks up and running by eliminating unneeded network traffic. Rapid PVST+ improves on RSTP, by creating individual spanning trees for each VLAN, allowing for more granular control and allows for traffic load balancing by utilizing different paths for different VLANs.

DSW-A1
   ```
spanning-tree vlan 10,99 priority 0
spanning-tree vlan 20,40 priority 4096
```
<p align="center">
<img src="https://i.imgur.com/3eVnSSu.png" height="80%" width="80%" alt="v20a1"/>
</p><br />

DSW-A2
   ```
spanning-tree vlan 10,99 priority 0
spanning-tree vlan 20,40 priority 4096
```


DSW-B1
   ```
spanning-tree vlan 10,99 priority 0
spanning-tree vlan 20,30 priority 4096
```

DSW-B2
  ```
spanning-tree vlan 10,99 priority 4096
spanning-tree vlan 20,30 priority 0
```
The root bridges(aka switches) for each VLAN receive a priority of 0. In Spanning Tree, the root bridge is elected by the bridge that has the lowest priority, if two or more bridges have the same priority, then the root bridge is determined by the bridge with the lowest Bridge ID(mac address). Since DSW-A1 and DSW-B1 are the active routers in HSRP for VLANs 10,99 they both receive a priority of 0(lowest priority level), with the command 'spanning-tree vlan 10,99 priority 0'. Bridge priorities are adjusted in increments of 4096 (2^12) so one increment above the lowest priority is 4096, hence the command 'spanning-tree vlan 20,40 priority 4096'. In Office B VLAN 30 is substituted for vlan 40 in the commands.
<br />
<br />
<h4>Step 2: Enable PortFast and BPDU Guard on all ports connected to end hosts (including WLC1). Perform the configurations in interface config mode.</h4>

  ```
interface f0/1
spanning-tree portfast
spanning-tree bpduguard enable
```
<p align="center">
<img src="https://i.imgur.com/E32PaFH.png" height="80%" width="80%" alt="v20a1"/>
</p><br />
This command is repeated on each Access Layer Switch: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, and ASW-B3
<br />
<br />
<h3>Part 5 – Static and Dynamic Routing</h3>
<h4>Step 1: Configure OSPF on R1 (LAN-facing interfaces) and all Core and Distribution switches (all Layer-3 interfaces)</h4>
a. Use process ID 1 and Area 0 
b. Manually configure each device’s RID to match the loopback interface IP.
c. On switches, use the network command to match the exact IP address of each interface.
d. On R1, enable OSPF in interface config mode.
e. Make sure OSPF is enabled on all loopback interfaces, too. Loopback interfaces should be passive.
f. Each Distribution switch’s SVIs (except the Management VLAN SVI) should be passive, too.
g. Configure all physical connections between OSPF neighbors to use a network type that doesn’t elect a DR/BDR. NOTE: This doesn’t work on the Layer-3 PortChannel interfaces between CSW1/CSW2. Leave them as the default network type.

R1:

  ```
router ospf 1
router-id 10.0.0.76
interface l0
ip ospf 1 area 0
passive-interface l0
interface range g0/0-2
ip ospf 1 area 0
ip ospf network point-to-point
```
<p align="center">
<img src="https://i.imgur.com/rgEGqjF.png" height="80%" width="80%" alt="v20a1"/>
</p><br />
<p align="left">I used the command 'do show ip interface brief' to figure out the loopback interface's IP address. In the screenshot, you can see I forgot to add the loopback interface when originally completing this lab, so I added the interface here. It's not necessary to do this over again, In the R1 configurations, Part 3 Step 1, the loopback interface was already added.
<br/>
OSPF(Open Shortest Path First) is a Link-State dynamic IP routing protocol, where all routers in an area (autonomous system) share a map until all routers have the same map of IP routes. To enable OSPF on a router, use the command 'router osfp (process id)', it will also put you in router configuration mode. The command 'router-id (IP address)' is used to manually set the router-id. To enable OSPF on a single interface, enter interface configuration with the command 'interface (interface #)', then enter the command 'ip ospf (same process id) area (area #)' The command 'passive-interface (interface #)' to conserve router resources. This will prevent the interface from sending OSPF Hello messages. It's used on this loopback interface because we know it cannot create an OSPF neighbor relation, since it's a virtual interface. Lastly, we use the command 'ip ospf network point-to-point', since there are only 2 routers in the network between R1 and CSW1, and R1 and CSW2. The point-to-point network typer creates full adjacencies and eliminates the DR/ BDR election process. 
<br/>

CSW1:

   ```
router osfp 1
router-id 10.0.0.77
network 10.0.0.77 0.0.0.0 area 0
passive-interface l0
network 10.0.0.41 0.0.0.0 area 0
network 10.0.0.34 0.0.0.0 area 0
network 10.0.0.45 0.0.0.0 area 0
network 10.0.0.49 0.0.0.0 area 0
network 10.0.0.53 0.0.0.0 area 0
network 10.0.0.57 0.0.0.0 area 0
interface range g1/0/1, g1/1/1-4
ip ospf network point-to-point
```
<p align="center">
<img src="https://i.imgur.com/RWOgqUd.png" height="80%" width="80%" alt="ospf core"/>
</p><br />
CSW2:

   ```
router osfp 1
router-id 10.0.0.78
network 10.0.0.78 0.0.0.0 area 0
passive-interface l0
network 10.0.0.61 0.0.0.0 area 0
network 10.0.0.65 0.0.0.0 area 0
network 10.0.0.69 0.0.0.0 area 0
network 10.0.0.73 0.0.0.0 area 0
network 10.0.0.78 0.0.0.0 area 0
network 10.0.0.38 0.0.0.0 area 0
interface range g1/0/1, g1/1/1-4
ip ospf network point-to-point
```

These commands do the same as above, but instead of enabling OSPF on the individual interface, the 'network' command allows you to enable OSFP on an entire network, wild card masks are used instead of netmasks.

DSW-A1:

  ```
router osfp 1
router-id 10.0.0.79
passive-interface l0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 40
passive-interface vlan 99
network 10.0.0.46 0.0.0.0 area 0
network 10.0.0.62 0.0.0.0 area 0
network 10.0.0.79 0.0.0.0 area 0
network 10.1.0.2 0.0.0.0 area 0
network 10.2.0.2 0.0.0.0 area 0
network 10.6.0.2 0.0.0.0 area 0
network 10.0.0.2 0.0.0.0 area 0
int range g1/1/1-2
ip ospf network point-point
```
DSW-A2:

  ```
router osfp 1
router-id 10.0.0.80
passive-interface l0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 40
passive-interface vlan 99
network 10.0.0.50 0.0.0.0 area 0
network 10.0.0.66 0.0.0.0 area 0
network 10.0.0.80 0.0.0.0 area 0
network 10.1.0.3 0.0.0.0 area 0
network 10.2.0.3 0.0.0.0 area 0
network 10.6.0.3 0.0.0.0 area 0
network 10.0.0.3 0.0.0.0 area 0
int range g1/1/1-2
ip ospf network point-point
```

DSW-B1:

  ```
router osfp 1
router-id 10.0.0.81
passive-interface l0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 30
passive-interface vlan 99
network 10.0.0.54 0.0.0.0 area 0
network 10.0.0.70 0.0.0.0 area 0
network 10.0.0.81 0.0.0.0 area 0
network 10.3.0.2 0.0.0.0 area 0
network 10.4.0.2 0.0.0.0 area 0
network 10.5.0.2 0.0.0.0 area 0
network 10.0.0.18 0.0.0.0 area 0
int range g1/1/1-2
ip ospf network point-point
```
DSW-B2:

  ```
router osfp 1
router-id 10.0.0.82
passive-interface l0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 30
passive-interface vlan 99
network 10.0.0.58 0.0.0.0 area 0
network 10.0.0.74 0.0.0.0 area 0
network 10.0.0.81 0.0.0.0 area 0
network 10.3.0.3 0.0.0.0 area 0
network 10.4.0.3 0.0.0.0 area 0
network 10.5.0.3 0.0.0.0 area 0
network 10.0.0.18 0.0.0.0 area 0
int range g1/1/1-2
ip ospf network point-point
```
<p align="center">
<img src="https://i.imgur.com/wj9D1dT.png" height="80%" width="80%" alt="ospf core"/>
</p><br />
<p align='left'> These commands enable OSPF on the router, manually set the router ID, set passive-interfaces, activate OSPF on the network, and make the connection between routers point-to-point networks. OSPF is a layer 3 protocol, so it is not used on the Access Layer Switches.
<br />
<br />
<h4>2. Configure one static default route for each of R1’s Internet connections. They should be recursive routes.</h4>
a. Make the route via G0/1/0 a floating static route by configuring an AD value 1 greater than the default.
b. R1 should function as an OSPF ASBR, advertising its default route to other routers in the OSPF domain.

  ```
ip route 0.0.0.0 0.0.0.0 203.0.113.1
ip route 0.0.0.0 0.0.0.0 203.0.113.5 2
default-information originate
```
<p align="center">
<img src="https://i.imgur.com/C8vaicz.png" height="80%" width="80%" alt="ospf default"/>
</p><br />
<p align='left'>I started by using the 'do show ip interface g0/1/0' command to get information about the interface. I can see that the interfaces are part of the /30 subnet, meaning 4 total addresses in the subnet. The network address is 203.0.113.4, broadcast address is 203.0.113.7. int g0/1/0 is 203.0.113.6 so I assumed the ISP's address is 203.0.113.5, I confirmed by pinging the address. A standard static route has an Administrative Distance (AD) of 1, to create a floating static route, it must have an AD higher than 1, so the 2 is added to the end of the ip route command to make it 1 AD higher than the default static route ad. The same methodology was used to determine the IP address for the other ISP. The command 'default-information originate' advertises the default route out of the network to other routers. It also makes R1 an Autonomous System Boundary Router (ASBR) or a router connected to an external network, in this case, the internet.
<br />
<br />
<h3>Part 6 – Network Services: DHCP, DNS, NTP, SNMP, Syslog, FTP, SSH, NAT</h3>
<h4>Step 1. Configure the following DHCP pools on R1 to make it serve as the DHCP server for hosts in Offices A and B. Exclude the first ten usable host addresses of each pool; they must not be leased to DHCP clients.</h4>
a. Pool: A-Mgmt
i. Subnet: 10.0.0.0/28
ii. Default gateway: 10.0.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
v. WLC: 10.0.0.7
b. Pool: A-PC
i. Subnet: 10.1.0.0/24
ii. Default gateway: 10.1.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
c. Pool: A-Phone
i. Subnet: 10.2.0.0/24
ii. Default gateway: 10.2.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
d. Pool: B-Mgmt
i. Subnet: 10.0.0.16/28
ii. Default gateway: 10.0.0.17
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
v. WLC: 10.0.0.7
e. Pool: B-PC
i. Subnet: 10.3.0.0/24
ii. Default gateway: 10.3.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
f. Pool: B-Phone
i. Subnet: 10.4.0.0/24
ii. Default gateway: 10.4.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
g. Pool: Wi-Fi
i. Subnet: 10.6.0.0/24
ii. Default gateway: 10.6.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)

R1:
   ```
ip dhcp excluded-address 10.0.0.1 10.0.0.10
ip dhcp excluded-address 10.1.0.1 10.1.0.10
ip dhcp excluded-address 10.2.0.1 10.2.0.10
ip dhcp excluded-address 10.0.0.17 10.0.0.26
ip dhcp excluded-address 10.3.0.1 10.3.0.10
ip dhcp excluded-address 10.4.0.1 10.4.0.10
ip dhcp excluded-address 10.6.0.1 10.6.0.10
```

<p align='left'>These are the first 10 addresses from each of the subnets. The DHCP-excluded addresses are excluded independent of a DHCP pool, so this is entered directly in Global Config mode. The 'ip dhcp excluded address' command is followed by the first ip address in the range, followed by the last address in the range so R1 will not assign the address to clients.</p>

R1:
   ```
ip dhcp pool A-Mgmt
network 10.0.0.0 255.255.255.240
default-router 10.0.0.1
dns-server 10.5.0.4
domain-name jeremeysitlab.com
option 43 ip 10.0.0.7

ip dhcp pool A-PC
network 10.1.0.0 255.255.255.0
default-router 10.1.0.1
dns-server 10.5.0.4
domain-name jeremeysitlab.com

ip dhcp pool A-Phone
network 10.2.0.0 255.255.255.0
default-router 10.2.0.1
dns-server 10.5.0.4
domain-name jeremeysitlab.com
```

<p align="center">
<img src="https://i.imgur.com/cviR7Vv.png" height="80%" width="80%" alt="dhcp office a"/>
</p><br />
<p align='left'> The command 'ip DHCP pool (pool name)' creates a DHCP pool. The network command tells the pool which addresses are eligible for that specific DHCP pool. The 'default-router (ip address)' command sets the default-router, or the router to send traffic to. The 'DNS-server (ip address)' command sets the Domain Name System (DNS) server's IP address for the DHCP pool. For this network, SRV1 is the DNS server for the entire network. So whenever human-readable addresses need to be translated to ipv4 addresses the end hosts will reach out to SRV1 to get the correct translated address. The 'domain-name (domain-name)' command sets the domain name for each subnet. The 'option 43 ip 10.0.0.7' command sets the IP address for the WLC.</p>

R1:
   ```
ip dhcp pool B-Mgmt
network 10.0.0.16 255.255.255.240
default-router 10.0.0.17
dns-server 10.5.0.4
domain-name jeremeysitlab.com
option 43 ip 10.0.0.7

ip dhcp pool B-PC
network 10.3.0.0 255.255.255.0
default-router 10.3.0.1
dns-server 10.5.0.4
domain-name jeremeysitlab.com

ip dhcp pool B-Phone
network 10.4.0.0 255.255.255.0
default-router 10.4.0.1
dns-server 10.5.0.4
domain-name jeremeysitlab.com

ip dhcp pool WiFi
network 10.6.0.0 255.255.255.0
default-router 10.6.0.1
dns-server 10.5.0.4
domain-name jeremeysitlab.com
```
<p align="center">
<img src="https://i.imgur.com/nAPLY0c.png" height="80%" width="80%" alt="dhcp office b"/>
</p><br />
<br />
<br />
<h4>Step Configure the Distribution switches to relay wired DHCP clients’ broadcast messages to R1’s Loopback0 IP address.</h4>

DSW-A1 and DSW-A2
  ```
interface vlan 10
ip helper-address 10.0.0.76
interface vlan 20
ip helper-address 10.0.0.76
interface vlan 40
ip helper-address 10.0.0.76
interface vlan 99
ip helper-address 10.0.0.76
```
<p align="center">
<img src="https://i.imgur.com/Ov6Mw2I.png" height="80%" width="80%" alt="dhcp office b"/>
</p><br />

DSW-B1 and DSW-B2
   ```
interface vlan 10
ip helper-address 10.0.0.76
interface vlan 20
ip helper-address 10.0.0.76
interface vlan 30
ip helper-address 10.0.0.76
interface vlan 99
ip helper-address 10.0.0.76
```
<p align="center">
<img src="https://i.imgur.com/beG2Sdq.png" height="80%" width="80%" alt="dhcp office b"/>
</p><br />
<p align='left'> These commands set a DHCP helper relay to the loopback interface on R1. So Whenever the SVI in each VLAN receives a DHCP message, it will forward the message to R1.
