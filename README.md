
<h1> Two Office Network-Configuration---CCNA-Recap</h1>
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
<align="left">This is the provided network topology. The devices are connected appropriately but not configured yet. The network connects to the internet via a dual-homed router at the top center of the image. Below the router, there are two Core Layer, multilayer switches, CSW1 and CSW2. The two core layer switches will be connected via a PaGP ether channel for redundancy and load balancing. Below the Core Switches, there are a total of 4 Distribution Layer, Layer 3 Switches: DSW-A1, DSW-A2, DSW-B1, and DSW-B2. The Access Layer consists of 6 Layer 2 switches: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, and ASW-B3. The Wireless LAN Controller (WLC1) is connected to ASW-A1. Lightweight Access Point (LWAP1) will be used for guest wireless access in Office A, and it is also connected to the access layer via ASW-A1. These Cisco C2960 switches have 24 Fast Ethernet ports, so each switch could have 24 end hosts connected. However, to demonstrate the knowledge of connecting an end host, each switch will only have one end host. In Office B, on the right, a second Lightweight Access Point (LWAP2) is used for guest Wi-Fi. LWAP2 is connected to the access layer via ASW-B1. There is one end host connected to the access layer via ASW-B2. SRV1, located in the bottom right of Office B, will be used as a Domain Name System (DNS) server, File Transfer Protocol (FTP) server, and a Syslog Log server for the network.
<br />
<p align="center">
<img src="https://i.imgur.com/3oSoI7g.png" height="80%" width="80%" alt="Diagram Layers"/>
</p><br />  
<br />
<h3>Part 1: Initial Setup</h3>
Step 1: Configure appropriate hostnames for each router and switch
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
<align="left">In packet tracer, you can access a network device's CLI by double-clicking the device and then clicking the CLI tab. The hostname is configured from the Global config mode. The 'enable' command allows the user to access privileged Exec mode indicated by #. The 'configure terminal' command allows the user to access Global Exec mode, indicated by (config#) 
<br />
<br />
Step 2: Configure the enable secret jeremysitlab on each router and switch. Use type 9 hashing if available; otherwise, use type 5.
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
<align="left">The 'enable secret' and 'enable algorithm-type script secret' commands each provide an added layer of security. These commands set a password for entering privileged EXEC mode, which is stored in a hashed format for security. The enable algorithm-type script secret command, or the type 9 hashing, provides the most secure hashing algorithm on current Cisco devices, but it's not supported on every device type. Type 5 hashing, used by the enable secret command, sets an MD5 hashed password on the device; it's considered less secure than level 9. Using hashing algorithms with passwords is always a good security practice. Even though MD5 hashing is no longer secure, it adds another layer of complexity for someone who may be attempting to tamper with the device. 
<br />
<br />
Step 3: Configure the user account cisco with secret ccna on each router and switch. Use type 9 hashing if available; otherwise, use type 5 <br/>

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
<align="left"> 
These commands create a local user account with a username 'cisco' and an encrypted password 'ccna'. Level 9 hashing of the password is preferred, but it is not supported on all devices. When the command 'username cisco algorithm-type script secret ccna' is not accepted on the device, the next preferred option for this lab is 'username cisco secret ccna'.
<br />
<br />
Step 4: Configure the console line to require login with a local user account. Set a 30-minute inactivity timeout. Enable synchronous logging.<br/>

  ```
line console 0
  login local
  exec-timeout 30
  logging synchronous
```
<p align="center">
<img src="https://i.imgur.com/KpT2OCm.png" height="80%" width="80%" alt="console line config"/>
</p>
<align="left"> 
The 'line console 0' command is used to enter the console line configuration mode on a Cisco device. This mode allows you to configure settings for the console line, which is the physical console port on the device. The 'login local' command allows usernames and passwords configured locally on the device can be used to log in. The 'exec-timeout 30' command signs out the signed-in user after 30 minutes of inactivity. The 'logging synchronous' command ensures that system messages do not interrupt your command input on the console, logging synchronous reprints your current command line after each log message. The step is repeated on each router and switch.
<br /> 
<br /> 

<h3>Part 2: VLANs, Layer-2 EtherChannel</h3>
Step 1: Configure a Layer-2 EtherChannel named PortChannel1 between DSW-A1 and DSW-A2 using a Cisco-proprietary protocol. Both switches should actively try to form an EtherChannel. <br/>

  ```
interface range g1/0/4-5
  channel-protocol pagp
  channel-group 1 mode desirable
```
<p align="center">
<img src="https://i.imgur.com/P6VctL5.png" height="80%" width="80%" alt="PAgP EtherChannel"/>
</p>
<align="left"> 
This command configures a Layer 2 EtherChannel on a Cisco switch using the Port Aggregation Protocol (PAgP). PAgP is a Cisco proprietary protocol used to form an EtherChannel. EtherChannels bundle multiple physical links into a single logical link, providing redundancy, load balancing, and simplified network management, along with other benefits. The interface range command allows for grouping interfaces to be configured simultaneously, rather than configuring each interface individually. The 'channel-protocol pagp' line sets PAgP as the EtherChannel negotiation protocol for the group of interfaces; however, this line is not necessary. Finally, 'channel-group 1 mode desirable' adds the selected interfaces to EtherChannel group 1 in desirable mode, which actively tries to form an EtherChannel using PAgP. These commands need to be entered on both DSW-A1 and DSW-A2 only.
<br /> 
<br /> 
Step 2: Configure a Layer-2 EtherChannel named PortChannel1 between DSW-B1 and DSW-B2 using an open standard protocol. Both switches should actively try to form an EtherChannel.<br />

  ```
interface range g1/0/4-5
  channel-protocol lagp
  channel-group 1 mode active
```
<p align="center">
<img src="https://i.imgur.com/DoKcwgb.png" height="80%" width="80%" alt="LACP EtherChannel"/>
</p>
<align="left"> 
This command configures a Layer 2 EtherChannel on a switch using the Ling Aggregation Protocol (LACP). LACP is an open standard protocol used to form an EtherChannel, LACP is defined in the IEEE 802.3ad.  The 'channel-protocol lacp' line sets LACP as the EtherChannel negotiation protocol for the group of interfaces; it is not necessary. Finally, 'channel-group 1 mode active' adds the selected interfaces to EtherChannel group 1 in active mode, which actively tries to form an EtherChannel using LACP. These commands need to be entered on both DSW-B1 and DSW-B2 only.
<br /> 
<br /> 
Step 3: Configure all links between Access and Distribution switches, including the EtherChannels, as trunk links. Disable DTP on all ports. Set each trunk's native VLAN to VLAN 1000(unused). In Office A, allow VLANs 10, 20, 40, and 99 on all trunks.In Office B, allow VLANs 10, 20, 30, and 99 on all trunks.<br />
  
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
<align="left"> This configuration establishes two trunk ports that will not negotiate trunking dynamically, defines a native VLAN (VLAN 1000), and restricts the allowed VLANs for the trunk. A Virtual Local Area Network (VLAN) is a logical grouping of devices within a larger physical network, enabling network segmentation. A trunk is used when you need to carry multiple VLANs on an interface. Using the command 'switchport nonegotiate' ensures that the trunk will not attempt to negotiate with neighboring devices, providing more control and reducing potential misconfigurations. The command 'switchport trunk native vlan 1000' sets the native VLAN to an unused VLAN, ensuring that all packets are tagged with the source VLAN, preventing them from being inadvertently sent to an unintended destination. This reduces the risk of VLAN hopping and minimizes security risks. The command 'switchport trunk allowed vlan 10,20,40,99' used in Office A and 'switchport trunk allowed vlan 10,20,30,99' used in Office B specify which VLANs are allowed to be sent on the trunk. If a packet is not tagged with one of these VLANs for the respective offices, it is dropped. Office A commands are used on ASW-A1, ASW-A2, and ASW-A3. Office B commands are used on ASW-B1, ASW-B3, ASW-B3.
  
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

Step 4: Configure one of each office’s Distribution switches as a VTPv2 server. Use the domain name JeremysITLab. Verify that other switches join the domain. Configure all Access switches as VTP clients.
Distribution Layer Switches:
  ```
vtp domain JeremysITLab
vtp version 2
```
<p align="center">
<img src="https://i.imgur.com/NQj2kwv.png" height="80%" width="80%" alt="VTP Server"/>
</p><br />
<align="left"> The default VLAN Trunking Protocol (VTP) is a Cisco proprietary protocol used to manage VLAN configurations. VTP is enabled by default. Switches are in VTP server mode by default, which allows for the creation, modification, and deletion of VLANs within the VTP domain. The 'vtp domain JeremysITLab' command creates or joins other switches on the network into a group or domain to share VTP information. The 'vtp version 2' command ensures each switch utilizes the same version of VTP. These commands should be repeated on the distribution layer switches: DSW-A1, DSW-A2, DSW-B1, and DSW-B2.
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
<align="left"> The command 'vtp mode client' changes the VTP permissions for the switch. VTP client switches receive VLAN information from VTP servers and apply it to their configuration. They cannot create, modify, or delete VLANs. The command 'vtp domain JeremysITLab' joins the switch to a VTP domain named JeremysITLab, allowing it to share VTP information with other switches in the same domain. The command 'vtp version 2' ensures that each switch utilizes the same version of VTP. These commands should be repeated on the access layer switches: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, and ASW-B3.
<br />
<br />
Step 5: In Office A, create and name the following VLANs on one of the Distribution switches. Ensure that VTP propagates the change. VLAN 10: PCs. LAN 20: Phones. VLAN 40: Wi-Fi. VLAN 99: Management.<br/>
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
<align="left">The command 'vlan 10' creates a VLAN with that number. Then the 'name PCs' command names that vlan. These commands need to be entered on one of Office A's VTP server switches, either DSW-A1 or DSW-A2. It will send VTP advertisements to other members of the same VTP domain. VTP advertisements are only shared via trunks, currently, the configurations between the distribution layer and core layers are access ports, so VTP information in Office A will not be shared with Office B and vice versa.
<br />
<br />
Step 6: In Office B, create and name the following VLANs on one of the Distribution switches. Ensure that VTP propagates the change. VLAN 10: PCs. LAN 20: Phones. VLAN 30: Servers. VLAN 99: Management.<br/>
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
<align="left">The above commands create and name each VLAN in Office B. The above command only needs to be entered on one of the VTP servers. VTP advertisements will be sent to each of the other switches within the VTP domain in Office B.
<br />
<br />
Step 7: Configure each Access switch’s access port. LWAPs will not use FlexConnect. PCs in VLAN 10, Phones in VLAN 20. SRV1 in VLAN 30. Manually configure access mode and explicitly disable DTP.<br/>

 ASW-A1 and ASW-B1 
  ```
interface f0/1
switchport mode access
switchport nonegotiate
switchport access vlan 99
```
<br />
<align="left"> Office B does not have a Wi-Fi Vlan because LWAPs only need access to the Management VLAN. They will tunnel information to the WLC in office A, then the traffic will be assigned to VLAN 40. So the ports to the LWAPs can be configured as access ports, not trunks. The 'interface f0/1' command enters interface configuration mode. 'switchport mode access' makes this port an access port. 'switchport nonegotiate' disables Dynamic Trunking Protocol (DTP), although it's redundant because when an interface becomes an access port no more DTP messages will be sent from the interface. Enter the above commands on both ASW-A1 in Office A and ASW-B1 in Office B

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
<align="left"> The above command makes int f0/1 an access port, join VLAN 10 and VLAN 20, and disables DTP.
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
<align="left"> This image shows the output of 'show vlan brief' with interface fa0/1 in Server VLAN.
<br />
<br />
Step 8: Configure ASW-A1’s connection to WLC1.  It must support the Wi-Fi and Management VLANs. The Management VLAN should be untagged. Disable DTP.

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
Step 9:  Administratively disable all unused ports on the Access and Distribution switches.
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
<align="left"> Shutting down all unused ports is a good security practice. Switches will automatically allow new physical connections to the switch to connect to the network. Administratively shutting down the interface, will not allow access to the network. This step needs to be repeated on each switch in Office A and Office B: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, ASW-B3, DSW-A2, DSW-B1 and DSW-B2. The 'show interfaces status' command should be used on each switch to determine which interfaces to use with the 'interface range' command.


  
<h3>Part 3: IP Addresses, Layer-3 EtherChannel, HSRP </h3>
Step 1: Configure the following IP addresses on R1’s interfaces and enable them:<br />

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

<p><align="left">At this step, we are beginning to assign static IPv4 addresses. Generally, in your network, it's good practice to set static IP addresses on network interfaces so that you can have set addresses to diagnose network errors. The external-facing interfaces, G0/0/0 and G0/1/0, use Dynamic Host Configuration Protocol (DHCP) to have their interfaces assigned an address from the Internet Service Provider that they are connected to. Later, Network Address Translation (NAT) will be applied so that internal devices will be able to reach the Internet.

From Global Configuration mode, enter the command 'interface `<interface #>`' to enter interface configuration mode (config-if)#, followed by the command 'ip address `<ip address>` `<netmask>`' to assign a static IP address. The command 'no shutdown' needs to be used on each interface because router interfaces are down by default, unlike switches. The 'no shutdown' command is not needed on the loopback interface l0 because the command int l0 creates a virtual loopback interface, which is active by default. Now, the output of the command 'do show ip interface brief' from Global configuration mode should look like this:</p>
<p align="center">
<img src="https://i.imgur.com/SAq9sCL.png" height="80%" width="80%" alt="R1 ipv4 interfaces"/>
</p>
</br>
</br>
Step 2: Enable IPv4 routing on all Core and Distribution switches<br />

 ```
ip routing
```

<p align="center">
<img src="https://i.imgur.com/jdaNlgB.png" height="80%" width="80%" alt="ip routing"/>
</p>
<p><align="left">This command needs to be entered on all core and distribution switches: CSW1, CSW2, DSW-A1, DSW-A2, DSW-B1, and DSW-B2. Switches, by default, utilize Layer 2 addressing (MAC addresses). They do not use or need IP addresses or have an IP routing table. These core and distribution switches are all Layer 3 or multilayer switches. The command ip routing allows each of these switches to utilize Layer 3 addressing (IP protocols) while maintaining their Layer 2 functions. Now they can build IP routing tables.
</br>
</br>
Step 3: Create a Layer-3 EtherChannel between CSW1 and CSW2 using a Cisco-proprietary protocol. Both switches should actively try to form an EtherChannel. Configure the following IP addresses:<br />

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
</p></br>
<align="left">The above commands create a Layer 3 EtherChannel using PAgP. The first line of the command groups the interfaces in interface configuration mode. The command 'no switchport' disables Layer 2 addressing on the ports, turning them into Layer 3 ports. The command 'channel-group 1 mode desirable' creates an EtherChannel port using PAgP mode desirable, which actively tries to create an EtherChannel. The final two lines, starting with 'interface po1', enter configuration mode for the PortChannel 1 interface and assign an IP address so the EtherChannel can communicate with other Layer 3 network devices.
</br>
</br>
Step 4: Configure the following IP addresses on CSW1: </br>
  
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


<p><align="left">These commands assign IP addresses to individual interfaces. The command 'show ip int br | include unassigned' was used to identify which interfaces are not being used before entering the interface range. Interfaces g1/0/2-3 were excluded from the shutdown command as they are used for the EtherChannel. It is good security practice to administratively shut down switch interfaces that are not in use. Additionally, moving these switches to an unused VLAN, enabling port security, and disabling CDP or LLDP are also good security measures for unused ports. The IP Interface brief should now look like this :
<p align="center">
<img src="https://i.imgur.com/KKH5Ieq.png" height="80%" width="80%" alt="ip routing"/>
</p></br>
</br>
Step 5: Configure the following IP addresses on CSW2: </br>
  
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


<p><align="left">Same commands for CSW1, but the IP addresses have been updated. It's a good security practice to administratively shut down switch interfaces that are not being used. Additionally, moving these switches to an unused VLAN, enabling port security, and disabling CDP or LLDP are also good security measures for unused ports. The IP Interface brief should now look like this :
<p align="center">
<img src="https://i.imgur.com/kucSUUK.png" height="80%" width="80%" alt="ip routing"/>
</p></br>
</br>
Step 6: Configure the following IP addresses on DSW-A1: </br>
  
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


<p><align="left">These are the IP interface configurations for DSW-A1. The IP Interface brief should now look like this :
<p align="center">
<img src="https://i.imgur.com/a7fl291.png" height="80%" width="80%" alt="dsw-A1"/>
</p></br>
Step 7: Configure the following IP addresses on DSW-A2: </br>
  
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


<p><align="left">Entering these IP addresses is very tedious, but they must be all entered correctly along with correct netmasks. The IP Interface brief should now look like this :
<p align="center">
<img src="https://i.imgur.com/xdU65i8.png" height="80%" width="80%" alt="dsw-a2"/>
</p></br>
</br>
Step 8: Configure the following IP addresses on DSW-B1: </br>
  
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


<p><align="left"> The IP Interface brief on DSW-B1 should now look like this :
<p align="center">
<img src="https://i.imgur.com/nJ4M81r.png" height="80%" width="80%" alt="dsw-b1"/>
</p></br>
</br>
Step 9: Configure the following IP addresses on DSW-B2: </br>
  
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


I made typos while entering some of these interface configurations, so it's good to check your work by using the command 'do show ip interface brief' from global config mode or 'show ip interface brief' from privileged exec mode. If any typos were made, use 'interface `<# of interface with error>`' and 'no ip address `<incorrect address>` `<netmask>`' commands to delete the entry. Then reenter the correct IP address. It is also good to use the 'write' command from privileged exec mode or 'do write' from global config mode to save your work. These commands write the running configuration to the startup configuration.
  
The IP Interface brief on DSW-B2 should now look like this :
<p align="center">
<img src="https://i.imgur.com/7ut7p1V.png" height="80%" width="80%" alt="dsw-b2"/>
</p></br>
</br>
Step 10: Manually configure SRV1’s IP settings: </br>
  
- <b>Default Gateway: 10.5.0.1</b>
- <b>IPv4 Address: 10.5.0.4</b>
- <b>Subnet Mask: 255.255.255.0</b>
  
To enter the configuration setting on end hosts, in packet tracer, you must use the GUI to manually set the configuration.
</br>
<p align="center">
<img src="https://i.imgur.com/RII6kW3.png" height="80%" width="80%" alt="SRV1"/>
<img src="https://i.imgur.com/CDEaAJB.png" height="80%" width="80%" alt="SRV1"/>
</p></br>
</br>




























<br /> 
<p align="center">
<img src="https://i.imgur.com/1LyLUHR.png" height="80%" width="80%" alt="Webhook"/>
</p>
<br /> First I connect a webhook to my Wazuh named Wazuh-alerts by editing my Wazuh-manager's ossec file.
<br /> 
<br />
Step 2: Extract File Hash from Alert 
<p align="center">
<img src="https://i.imgur.com/m7yA65E.png" height="80%" width="80%" alt="Regex"/>
</p>
<br /> I connected the Webhook to a Shuffle tool to parse the Wazuh alert for the SHA256 hash of the alert.
<br /> 
<br />
Step 3: Check Reputation Score w/ VirusTotal 
<p align="center">
<img src="https://i.imgur.com/0FLswQz.png" height="80%" width="80%" alt="VT App"/>
</p>
<br /> I connected the Suffle tool to a VirusTotal app to check the reputation score of the parsed SHA256 from the alert
<br /> 
<br />
Step 4: Send Details to TheHive to Create an Alert
<p align="center">
<img src="https://i.imgur.com/CMlfKXN.png" height="80%" width="80%" alt="TheHive App"/>
</p>
<br /> I then connected TheHive to my VirusTotal app inside of Shuffle so I could have cases created inside of TheHive once Wazuh generates an alert.
<br /> 
<br />
Step 5: Send an E-mail to the SOC Analyst to Begin the Investigation 
<p align="center">
<img src="https://i.imgur.com/So95PGA.png" height="80%" width="80%" alt="e-mail app"/>
</p>
<br /> I then connected an E-mail app so Shuffle will email me/ the analyst every time an alert is generated to take action.
<br /> 
<br />
<p align="center">
<img src="https://i.imgur.com/vtKH3Kl.png" height="80%" width="80%" alt="Done"/>
</p>
<br /> The last thing I do is connect the Wazuh manager to workflow to have Wazuh actively respond and block the source IP when a Mimikatz alert is generated. Now we have fully functioning SIEM and SOAR!
