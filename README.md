# Secure-Enterprise-Network-Architecture

## Objective

Design and implement a secure, scalable enterprise network that demonstrates core networking principles, including VLAN segmentation, inter-VLAN routing, dynamic routing protocols, redundancy, and foundational network security controls in a production-style environment.

### Skills Learned

- Designed and implemented a secure enterprise campus network using layered network architecture principles
- Configured VLAN segmentation to isolate departments and reduce broadcast domains
- Implemented inter-VLAN routing to enable controlled communication between network segments
- Deployed dynamic routing protocols to support scalable and efficient network traffic flow
- Applied network security best practices, including segmentation and access control concepts
- Configured redundant network paths to improve fault tolerance and availability
- Troubleshot routing, switching, and connectivity issues using systematic diagnostic techniques
- Interpreted network topology diagrams and translated designs into functional configurations
- Validated network functionality through testing, verification, and documentation
- Applied defense-in-depth principles through VLAN segmentation and controlled routing
- Reduced attack surface by isolating network segments and limiting lateral movement
- Implemented secure routing and switching configurations aligned with enterprise standards


### Tools Used

- Cisco Packet Tracer – Simulated and validated enterprise network topology and configurations
- Cisco IOS – Configured routing, switching, VLANs, and inter-VLAN communication
- Layer 2 Switching – Implemented VLANs, trunking, and access port configurations
- Layer 3 Routing – Configured inter-VLAN routing and dynamic routing protocols
- Dynamic Routing Protocols – Enabled scalable routing for multi-network environments
- Network Topology Diagrams – Designed and interpreted logical and physical network layouts
- Command-Line Interface (CLI) – Performed device configuration, verification, and troubleshooting
- Network Segmentation Tools – VLAN-based isolation to enforce access boundaries
- Access Control Concepts – Implemented controlled traffic flow between network segments
- Network Testing & Verification – Validated secure connectivity and routing behavior

## Steps
drag & drop screenshots here or use imgur and reference them using imgsrc

Every screenshot should have some text explaining what the screenshot is about.

Example below.

*Ref 1: Network Diagram*

#### HOSTNAMES, CONSOLE LINE, ENABLE SECRET
1. To begin the lab we will start by changing the hostname of applicable devices to ensure that it matches with the label provided to them. Let’s start by clicking the device “R1” in Cisco Packet Tracer. Click on the CLI tab and go from User EXEC mode to Privileged EXEC mode then finally to Global Configuration mode by inputting these commands.
    * Router> enable
    * Router# Configure Terminal 
    * Router(config)# hostname R1
    * R1(config)# exit
2. Next we will configure the enable secret ‘networklab’ on each router/switch. We will use type 9 hashing where available and type 5 hashing for the rest. The devices using type 5 hashing is the routers and access switches in this network topology. Starting with R1, we can only use MD5 hashing for this model of device. Starting from global configuration mode in the CLI, these are the commands needed to add an enable secret in MD5 hashing.
    * R1(config)# enable secret networklab
    * R1(config)# do sh run | include secret (to verify that the enable secret has been set in place)
3. Now we must also add the enable secret ‘networklab’ for the rest of the devices using type 9 hashing. Starting from global configuration mode on CSW1, these are the commands to add the secret in type 9 encryption.
    * CSW1(config)# enable algorithm-type scrypt secret networklab 
    * CSW1(config)# do sh run | include secret (to verify that changes were made and the secret was added)
4. Next, we will configure the user account ‘admin’ with secret ‘lab’ on each router/switch and just like in the previous step, we will apply type 9 hashing where available, and the rest will be type 5 hashing 
    * The type 5 devices CLI should use this command: R1(config)# username admin secret lab
    * Type 9 devices will use this command in CLI: R1(config)# username admin algorithm-type script secret lab
5. We will now configure the console line to require login with a local user account, set a 30-minute inactivity timeout, and enable synchronous logging
    * To configure the console line: R1(config)# line console 0
    * To require login using a local user account (just like the one we set in the previous step): R1(config-line)# login local
    * To set the exec timeout after 30 minutes in CLI: R1(config-line)# exec-timeout 30 
    * To enable synchronous logging on the device: R1(config-line)# logging synchronous (makes it easier to read syslog messages that are coming in
#### VLANs, Layer-2 Etherchannel
1. In Office A, we will configure a Layer 2 EtherChannel named “PortChannel1” between DSW-A1 and DSW-A2 using the Cisco proprietary protocol, PAgP
    * Do this by entering the CLI of DSW-A1 then in global config mode, use the command DSW-A1(config)# do show cdp neighbors to find the ports that DSW-A2 is connected to. This shows us that the correct ports to use is Gig 1/0/4 and Gig 1/0/5.
    * Use DSW-A1(config)# int range 1/0/4-5 to select the range of ports to configure
    * DSW-A1(config-if-range)# channel-group 1 mode desirable
    * In order to verify use the command DSW-A1(config-if-range)# do show etherchannel summary
    * Do the same steps for “DSW-A2”
2. Next, we will configure a Layer 2 EtherChannel named “PortChannel1” between DSW-B1 and DSW-B2 using the open standard protocol LACP
    * Go into the CLI of DSW-B1 and configure the correct ports to have EtherChannel on
    * DSW-B1(config)# int range g1/0/4-5
    * DSW-B1(config-if-range)# channel-group 1 mode active
    * We will then go over to the CLI go DSW-B2 and input the same commands 
    * DSW-B2(config)# int range g1/0/4-5 
    * DSW-B2(config-if-range) # channel-group 1 mode active
    * Then we will verify that EtherChannel has been configured by using the # do show etherchannel summary command
3. Now we will configure all links between Access and Distribution switches, including the EtherChannels, as trunk links. The main objectives that we will achieve is explicitly disabling DTP on all ports, setting each trunk’s native VLAN to VLAN 1000 (unused), and allowing VLANs 10, 20, 40, and 99 on all trunks for Office A while allowing VLANs 10, 20, 30, and 99 on all trunks in Office B
    * First we will go into DSW-A1
4. Next we will configure one of each office’s Distribution switches as a VTPv2 server. We will use the domain name networklabtest
    * We will also verify that other switches join the domain and web will configure all access switches as VTP clients
    * Starting with Office A:
        * VLAN 10: PCs
        * VLAN 20: Phones
        * VLAN 40: Wi-Fi
        * VLAN 99: Management
    * Next we will move on to Office B:
        * VLAN 10: PCs
        * VLAN 20: Phones
        * VLAN 30: Servers
        * VLAN 99: Management
    * VTP will push these configuration changes to the switches so we will configure it first on DSW-A1
    * We will create each VLAN on DSW-A1 and ensuring that we are naming it properly. We can then verify changes have been made by using the command ‘# show vlan brief’
    * Next we will make the same changes to Office B, on switch DSW-B1. Make sure to add VLAN 30 rather than VLAN 40 for this office
5. Now we will configure each Access switch’s access port and meet the following conditions
    * LWAPs will not use FlexConnect
    * PCs in VLAN 10, Phones in VLAN 20
    * SRV1 in VLAN 30
    * Manually configure access mode and explicitly disable DTP
    * Each access switch’s FastEthernet0/1 interface are connected to the end hosts
    * On ASW-A1:
        * ASW-A1(config)# interface f0/1
        * ASW-A1(config-if)# Switchport mode access
        * ASW-A1(config-if)# switchport nonegotiate
        * ASW-A1(config-if)# switchport access vlan 99 
    * Perform the same commands above on ASW-B1
    * Now we will assign the F0/1 interface to the phone VLAN(10) and the voice VLAN(20) on the access switches connected to the phones. These switches are ASW-A2, ASW-A3, and ASW-B2
        * Interface f0/1
        * Switchport mode access
        * Switchport nonegotiate 
        * Switchport access vlan 10
        * Switchport voice vlan 20
    * Lastly, we will configure ASW-B3 to connect to SRV1
        * Interface f0/1
        * Switchport mode access
        * Switchport nonegotiate
        * Switchport access vlan 30 
6. For this next section, we will configure ASW-A1’s connection to WLC1 meeting these conditions:
    * It must support the Wi-Fi and Management VLANs
    * The Management VLAN should be untagged
    * Disable DTP
        * WLC1 is connected to interface F0/2 so to configure we will use these commands
        * Interface f0/2
        * Switchport mode trunk 
        * Switchport trunk allowed vlan 40,99
        * Switchport trunk native vlan 99 (making the Management VLAN the native VLAN)
        * Switchport nonegotiate (disabling DTP)
7. Lastly, and for security purposes, we will disable all of the unused ports on the access and distribution switches
    * Use the command ‘# do show interfaces status’
    * All interfaces with the status “not connect” can be safely disabled as they are not in use
    * For DSW-A1, DSW-A2, DSW-B1, DSW-B2
        * Interface range g1/0/6-24,g1/1/3-4
        * Shutdown
    * For ASW-A2, ASW,A3, ASW-B1, ASW-B2, ASW-B3
        * Interface range f0/2-24
        * Shutdown
#### IP Addresses, Layer-3 Etherchannel, HSR
