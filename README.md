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

*Ref 1: Network Diagram*
<img width="2556" height="1411" alt="NetworkLabStart" src="https://github.com/user-attachments/assets/b129eb5d-e46a-4215-bebc-8a29d6a6062a" />

#### HOSTNAMES, CONSOLE LINE, ENABLE SECRET
1. To begin the lab we will start by changing the hostname of applicable devices to ensure that it matches with the label provided to them. Let’s start by clicking the device “R1” in Cisco Packet Tracer. Click on the CLI tab and go from User EXEC mode to Privileged EXEC mode then finally to Global Configuration mode by inputting these commands.
    * Router> enable
    * Router# Configure Terminal 
    * Router(config)# hostname R1
    * R1(config)# exit
  <img width="700" height="710" alt="HostnameCLI" src="https://github.com/user-attachments/assets/b6d89473-1738-4783-95d7-c9dfe734fad9" />
  
2. Next we will configure the enable secret ‘networklab’ on each router/switch. We will use type 9 hashing where available and type 5 hashing for the rest. The devices using type 5 hashing is the routers and access switches in this network topology. Starting with R1, we can only use MD5 hashing for this model of device. Starting from global configuration mode in the CLI, these are the commands needed to add an enable secret in MD5 hashing.
    * R1(config)# enable secret networklab
    * R1(config)# do sh run | include secret (to verify that the enable secret has been set in place)
<img width="699" height="706" alt="enableSecretCLI" src="https://github.com/user-attachments/assets/cbc1017c-bf58-4e38-8ce1-06a700c8238a" />

3. Now we must also add the enable secret ‘networklab’ for the rest of the devices using type 9 hashing. Starting from global configuration mode on CSW1, these are the commands to add the secret in type 9 encryption.
    * CSW1(config)# enable algorithm-type scrypt secret networklab 
    * CSW1(config)# do sh run | include secret (to verify that changes were made and the secret was added)
<img width="698" height="706" alt="enableSecret9CLI" src="https://github.com/user-attachments/assets/26cbae55-4c36-4d70-a6fd-714040333be8" />

4. Next, we will configure the user account ‘admin’ with secret ‘lab’ on each router/switch and just like in the previous step, we will apply type 9 hashing where available, and the rest will be type 5 hashing 
    * The type 5 devices CLI should use this command: R1(config)# username admin secret lab
    * Type 9 devices will use this command in CLI: R1(config)# username admin algorithm-type script secret lab
<img width="698" height="706" alt="usernameSecret5" src="https://github.com/user-attachments/assets/57a8be74-03be-401f-9e88-7d3969684558" />
<img width="677" height="155" alt="usernameSecret9" src="https://github.com/user-attachments/assets/c37be9ec-bb89-452c-ac17-19e2bfb5cc02" />

5. We will now configure the console line to require login with a local user account, set a 30-minute inactivity timeout, and enable synchronous logging
    * To configure the console line: R1(config)# line console 0
    * To require login using a local user account (just like the one we set in the previous step): R1(config-line)# login local
    * To set the exec timeout after 30 minutes in CLI: R1(config-line)# exec-timeout 30 
    * To enable synchronous logging on the device: R1(config-line)# logging synchronous (makes it easier to read syslog messages that are coming in
<img width="651" height="110" alt="ConsoleLineConfiguration" src="https://github.com/user-attachments/assets/ef14234c-78c3-489d-803e-6473812b10d1" />

#### VLANs, Layer-2 Etherchannel
1. In Office A, we will configure a Layer 2 EtherChannel named “PortChannel1” between DSW-A1 and DSW-A2 using the Cisco proprietary protocol, PAgP
    * Do this by entering the CLI of DSW-A1 then in global config mode, use the command DSW-A1(config)# do show cdp neighbors to find the ports that DSW-A2 is connected to. This shows us that the correct ports to use is Gig 1/0/4 and Gig 1/0/5.
    * Use DSW-A1(config)# int range 1/0/4-5 to select the range of ports to configure
    * DSW-A1(config-if-range)# channel-group 1 mode desirable
    * In order to verify use the command DSW-A1(config-if-range)# do show etherchannel summary
    * Do the same steps for “DSW-A2”
<img width="660" height="264" alt="EtherchannelPAgP" src="https://github.com/user-attachments/assets/af266cd3-5f9b-47c3-9930-18d4b064aab1" />
<img width="660" height="370" alt="EtherChannelPAgP2DSW" src="https://github.com/user-attachments/assets/21d44064-cad5-4e91-b0a9-e97d066878d7" />

2. Next, we will configure a Layer 2 EtherChannel named “PortChannel1” between DSW-B1 and DSW-B2 using the open standard protocol LACP
    * Go into the CLI of DSW-B1 and configure the correct ports to have EtherChannel on
    * DSW-B1(config)# int range g1/0/4-5
    * DSW-B1(config-if-range)# channel-group 1 mode active
    * We will then go over to the CLI go DSW-B2 and input the same commands 
    * DSW-B2(config)# int range g1/0/4-5 
    * DSW-B2(config-if-range) # channel-group 1 mode active
    * Then we will verify that EtherChannel has been configured by using the # do show etherchannel summary command
<img width="657" height="137" alt="EtherChannelLACPDSWB1" src="https://github.com/user-attachments/assets/8c244336-c61d-4dc1-94cd-c57ff2af3b35" />
<img width="662" height="378" alt="EtherChannelLACPDSWB2" src="https://github.com/user-attachments/assets/090a538c-900f-408b-8ede-de803c69ddba" />

3. Now we will configure all links between Access and Distribution switches, including the EtherChannels, as trunk links. The main objectives that we will achieve is explicitly disabling DTP on all ports, setting each trunk’s native VLAN to VLAN 1000 (unused), and allowing VLANs 10, 20, 40, and 99 on all trunks for Office A while allowing VLANs 10, 20, 30, and 99 on all trunks in Office B
    * First we will go into DSW-A1
<img width="662" height="502" alt="TrunkLinkConfiguration" src="https://github.com/user-attachments/assets/7e06ab34-5a65-4701-9162-32e18eff3fd7" />
<img width="658" height="284" alt="TrunkLinkConfigurationDSW-A2" src="https://github.com/user-attachments/assets/a397f93c-ba2b-487a-804e-4bcb7ac86de0" />
<img width="660" height="198" alt="ASW-A1TrunkLinkConfiguration" src="https://github.com/user-attachments/assets/ec497f3d-917d-42c2-988b-cd58a992b5aa" />
<img width="656" height="293" alt="TrunkLinkConfigurtionDSW-B1" src="https://github.com/user-attachments/assets/f2c50b09-54f9-4b00-8ec7-6bc352bb0527" />
<img width="662" height="174" alt="ASW-B1TrunkConfiguration" src="https://github.com/user-attachments/assets/93602605-1437-4f3a-8476-bca394be9c6c" />

4. Next we will configure one of each office’s Distribution switches as a VTPv2 server. We will use the domain name networklabtest
<img width="659" height="45" alt="DomainSetupDSW-B1" src="https://github.com/user-attachments/assets/e2eeaa89-2e21-40ff-9e81-d1091628a768" />

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
<img width="658" height="222" alt="VLANSetupDSW-A1" src="https://github.com/user-attachments/assets/80b253d1-3c15-46ba-86ef-6b4dae0abdde" />

 * Next we will make the same changes to Office B, on switch DSW-B1. Make sure to add VLAN 30 rather than VLAN 40 for this office
<img width="660" height="261" alt="VLANSetupDSW-B1" src="https://github.com/user-attachments/assets/b2c9c527-ffd0-48da-9b16-e48a802f4d86" />

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
<img width="659" height="65" alt="ASW-A1AccessPortConfiguration" src="https://github.com/user-attachments/assets/1f51a545-3d0f-4515-b978-03f2303304bd" />

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
<img width="661" height="635" alt="Disabling Unused Ports DSW-A1" src="https://github.com/user-attachments/assets/bc4ea26a-0fb9-4a9d-ab80-e4abc49cd39e" />

#### IP Addresses, Layer-3 Etherchannel, HSR
1. Now we will assign IP addressing to R1’s interfaces and enable them. We will aim to meet the following conditions
    * G0/0/0: DHCP client
    * G0/1/0: DHCP client
    * G0/0: 10.0.0.33/30
    * G0/1: 10.0.0.76/32
        * First, we will configure g0/0/0 and g0/1/0 on R1 to enable DHCP then configure each interface 
            * R1(config)# int range g0/0/0, g0/1/0
            * R1(config-if-range)# ip add dhcp
            * R1(config-if-range)# no shutdown (enables the interfaces)
   <img width="316" height="54" alt="EnablingDHCP on R1 Interfaces" src="https://github.com/user-attachments/assets/3f8d396e-311c-4f29-8ef0-09a146e37874" />

        * Next, go into the other two interfaces to configure them starting with g0/0
            * # int g0/0
            * # ip address 10.0.0.33 255.255.255.252
            * # no shutdown
        * Now on g0/1
            * # int g0/1
            * # ip address 10.0.0.37 255.255.255.252
            * # no shutdown
   <img width="466" height="135" alt="StaticIPs on R1 g0:0 g0:1" src="https://github.com/user-attachments/assets/0f6bcb67-2a2f-4c94-b9be-f46b79b2dada" />

        * Lastly we will configure the loopback interface 
            * # int l0
            * # ip address 10.0.0.76 255.255.255.255
        * Verify all interface configurations with the command ‘# show ip interface brief’
   <img width="484" height="194" alt="LoopbackInterfaceIPconfiguration and Verifying Interfaces" src="https://github.com/user-attachments/assets/ff321f2b-dd96-4606-a686-3e1a70dd8a29" />

3. Next, we will enable IPv4 routing on all Core and Distribution switches
