# project Overview

This covers the design of a university/Enterprize network which can be scaled to meet user requirements. The projeect implements the following Configurations:

* Basic configurations:
    * Hostname
    * Console password for accessing console 
    * Login Password
    * Secure Shell
    * Secure vty
    * Password encryption
* Vlan and inter-vlan routing
* Subnetting and ip addressing
* Wireless network Configurations
* Telephony service
* Access list and NAT
* FTP,Email,DHCP,DNS
* EtherChannel
* HSRP
* Portfast & BPDU Guard
* Firewall Basic configs
* Trunking and Access ports configs
* OSPF

### Security Concerns Addressed

* DHCP Attacks
    * Starvation
        * Mitigation Technique: Port Security
    * Spoofing
        * Mitigation Technique: DHCP Snooping

* ARP Attacks
    * Mitigation Technique:Dynamic ARP INSPECTIOn(DAI)
* STP Attacks
    * Portfast and BPDU Guard
* VLAN Attacks
    * Shutdown Unused Ports
    * Differnt VLAN for Unused Ports
    * Enabling nonegotiate and different native vlan for trunk Ports

## Objectives

1. Ensure inter-department communication using routing protocols.
2. Secure the network using ASA firewall.
3. Enabling telephony services for voice communication.
4. Ensure secure login to the intermediary devices using SSH.
5. Enable efficient load balancing and redundancy using HSRP
6. Configure etherchannel for high bandwidth transmision in the network.

## Tools used

* Cisco packet tracer.
* Routers,Switches,WLC,LAP,Servers.
* End devices such as PC,tablets,ip Phones.
* Firewall.

## Network Design

## Basic Settings

On the layer 2 switches,these are the configurations commands:

* _hostname Admin-sw_
* _line console 0_
* _password mcnen123_
* _logging synchronous_
* _login_
* _exit_

Vlan Config
* _vlan 10_
* _name MGT_
* _exit_
* _vlan 20_
* _name LAN_
* _exit_
* _vlan 30_
* _name Voice_
* _exit_
* _vlan 40_
* _name WLAN_
* _exit_
* _vlan 50_
* _name Servers_
* _exit_
* _vlan 60_
* _name DeadPorts_
* _exit_
* _vlan 1000_
* _name Ports_
* _exit_

Password encryption

* _enable password mcnen123_
* _service password-encryption_
* _no ip domain-lookup_

SSH config
* _ip domain-name mcnen.com_
* _username mcnen password mcnen123_
* _crypto key generate rsa general-keys modulus 1024_
* _line vty 0 15_
* _login local_
* _transport input ssh_
* _exit_

ACLs to define the subnet that can SSH the switch
* _access-list 1 permit 192.168.0.0 0.0.0.255_
* _access-list 1 deny any_
* _line vty 0 15_
* _access-class 1 in_
* _exit_

## Vlan and inter-vlan routing

Vlans were already configured on the l2-switch.
For L3-switch the vlans need to be defined first same way as in l2_switch.

### Inter-Vlan Routing Config and HSRP
* _ip routing_  **Note:** This enables routing on the switch.

* _int vlan 10_
* _ip address 192.168.0.2 255.255.255.0_
**Note:** SVI ip address of the int facing thw switch
* _standby 10 ip 192.168.0.1_
**Note:** Defines the virtual IP shared with the two L# switch
* _standby 10 priority 110_
* _standby 10 preempt_
* _no shut_
* _exit_

* _int vlan 20_
* _ip address 172.20.10.2 255.255.255.0_
* _standby 20 ip 172.20.10.1_
* _standby 20 priority 110_
* _standby 20 preempt_
* _no shut_
* _exit_

* _int vlan 30_
* _ip address 172.20.11.2 255.255.255.0_
* _standby 30 ip 172.20.11.1_
* _standby 30 priority 110_
* _standby 30 preempt_
* _no shut_
* _exit_

* _int vlan 40_
* _ip address 10.20.0.2 255.255.0.0_
* _standby 40 ip 10.20.0.1_
* _standby 40 priority 100_
* _standby 40 preempt_
* _no shut_
* _exit_

* _int vlan 50_
* _ip address 172.20.15.2 255.255.255.0_
* _standby 50 ip 172.20.15.1_
* _standby 50 priority 100_
* _standby 50 preempt_
* _no shut_
* _exit_


## Telephony service

* _int fa0/0.30_
* _encapsulation dot1Q 30_
* _ip address 172.20.11.1 255.255.255.0_
* _no shut_
* _exit_
* _service dhcp_
* _ip dhcp pool voice_
* _network 172.20.11.0 255.255.255.0_
* _default-router 172.20.11.1_
* _option 150 ip 172.20.11.1_
* _exit_

* _telephony-service_
* _max-dn 15_
* _max-ephones 15_
* _ip source-address 172.20.11.1 port 2000_
* _auto assign 1 to 15_
* _exit_

* _ephone-dn 1_
* _number 401_
* _exit_

* _ephone-dn 2_
* _number 402_
* _exit_

* _ephone-dn 3_
* _number 403_
* _exit_

* _ephone-dn 4_
* _number 404_
* _exit_

* _ephone-dn 5_
* _number 405_
* _exit_

## EtherChannel

The load capacity of the networkis expectedly high during operation calling for the need to deploy a laod balancing technique.
To enhance more laod carrying capacity etherchannelis configured in which physically the intermediary devices have mulitple connection between them in which logically the intermediry device views as one link overcoming the issue of STP.
To configure etherChannel the link Aggregation control protocol(LACP) was used as shown:

* _int range gig1/0/7-10_
* _channel-group 1 mode active_
* _no shut_
* _exit_

* _int port-channel 1_
* switchport mode trunk_
* _switchport allow vlan 10,20,30,40,50,1000_
* _exit_

## Open Shortest Path First(OSPF)
This a link-state routing protocol that concept of areas helping to control routing update traffic.
Each intermediary device in the network connects to a network which may or may not be known by the adjascent device. For the device not familiar with a certain network it learns that network from another device that advertises the network.
OSPF was configured as follows:
* _router ospf 50_
* _router-id 1.1.1.1_
* _network 192.168.0.0 0.0.0.255 area 0_
* _network 172.20.10.0 0.0.0.255 area 0_
* _network 172.20.11.0 0.0.0.255 area 0_
* _network 10.20.0.0 0.0.255.255 area 0_
* _network 10.10.20.0 0.0.0.3 area 0_
* _network 10.10.21.0 0.0.0.3 area 0_
* _exit_

## Portfast & BPDU Guard & STP Attack mitigation
Portfast brings a link to an end device from an intermediary devices to forwarding state as the intermediary device reboots or is in learning & listening state.
When enables it allows user to access resources without the wait for intermediary devices to go to forwarding state.
Portfast is enabled on ports connected to intermediary devices which brings up the need to estabish a way to prevent this port from receiving a bpdu.
BPDU Guard is a network security feature designed to protect the spanning-tree network from potential rogue switches (physical or virtual) by placing interfaces into an error-disabled state when receiving Bridge Protocol Data Units (BPDUs)

To enable Portfast:
For all Access ports
* _int range fa0/1-24,gig0/1-2_
* _spanning-tree portfast_
* _exit_

Then on all portfast enabled ports enable BPDU Guard:
* _spanning-tree portfast bpduguard enable_

## Firewall Basic configs
Basic firewall configs were as follows:

* _hostname Uni-Firewall_
* _enable password mcnen123_
* _username mcnen password mcnen123_

* _int gig1/1_
* _no shut_
* _ip address 10.10.20.2 255.255.255.252_
* _nameif INSIDE_
* _security-level 100_
* _exit_

* _int gig1/2_
* _no shut_
* _ip address 10.10.21.2 255.255.255.252_
* _nameif INSIDE2_
* _security-level 100_
* _exit_

* _int gig1/3_
* _no shut_
* _ip address 10.10.10.1 255.255.255.0_
* _nameif DMZ_
* _security-level 50_
* _exit_

* _int gig1/4_
* _no shut_
* _ip address 198.100.0.2 255.255.255.252_
* _nameif OUTSIDE_
* _security-level 0_
* _exit_

* _aaa authentication ssh console local_
* _crypto key generate rsa modulus 1024_
* _ssh 192.168.0.0 255.255.255.0 INSIDE_
* _ssh timeout 3_

## Access list and NAT

To configure Network Address translation:

    object network IN-OUT
    subnet 10.20.0.0 255.255.0.0
    subnet 192.168.0.0 255.255.255.0
    subnet 172.20.10.0 255.255.255.0
    nat (INSIDE, OUTSIDE) dynamic interface
    nat (INSIDE2, OUTSIDE) dynamic interface
    exit

To configure ACLs

    access-list OUT-IN extended permit icmp 10.40.0.0 255.255.255.0 172.20.10.0 255.255.255.0
    access-list OUT-IN extended permit icmp 10.40.0.0 255.255.255.0 192.168.0.0 255.255.255.0
    access-list OUT-IN extended permit icmp 10.40.0.0 255.255.255.0 10.20.0.0 255.255.0.0
    access-list OUT-IN extended permit icmp 200.100.20.0 255.255.255.252  172.20.10.0 255.255.255.0
    access-list OUT-IN extended permit icmp 200.100.20.0 255.255.255.252  192.168.0.0 255.255.255.0
    access-list OUT-IN extended permit icmp 200.100.20.0 255.255.255.252  10.20.0.0 255.255.0.0
    access-list OUT-IN extended permit icmp 198.100.0.0 255.255.255.252  172.20.10.0 255.255.255.0
    access-list OUT-IN extended permit icmp 198.100.0.0 255.255.255.252  192.168.0.0 255.255.255.0
    access-list OUT-IN extended permit icmp 198.100.0.0 255.255.255.252  10.20.0.0 255.255.0.0
    access-group OUT-IN in int OUTSIDE

    access-list DMZ-ACCESS extended permit icmp any 10.10.10.0 255.255.255.0
    access-list DMZ-ACCESS extended permit tcp any host 10.10.10.10 eq 80
    access-list DMZ-ACCESS extended permit tcp any host 10.10.10.10 eq 8080
    access-group DMZ-ACCESS in int DMZ

    access-list DMZ-OUT extended permit icmp 10.10.10.0 255.255.255.0 any
    access-list DMZ-OUT extended permit tcp host 10.10.10.10 any eq 80
    access-list DMZ-OUT extended permit tcp host 10.10.10.10 any eq 8080
    access-group DMZ-OUT in int DMZ

    access-list OUT-DMZ extended permit icmp any 10.10.10.0 255.255.255.0
    access-list OUT-DMZ extended permit tcp any host 10.10.10.10 eq 80
    access-list OUT-DMZ extended permit tcp any host 10.10.10.10 eq 8080
    access-group OUT-DMZ in int OUTSIDE

    access-list DMZ-IN extended permit icmp 10.10.10.0 255.255.255.0 any
    access-list DMZ-IN extended permit tcp host 10.10.10.10 any eq 80
    access-list DMZ-IN extended permit tcp host 10.10.10.10 any eq 8080
    access-group DMZ-IN in int INSIDE
    access-group DMZ-IN in int INSIDE2

    access-list IN-DMZ extended permit icmp any 10.10.10.0 255.255.255.0 
    access-list IN-DMZ extended permit tcp any host 10.10.10.10 eq 80
    access-list IN-DMZ extended permit tcp any host 10.10.10.10 eq 8080
    access-group IN-DMZ in int INSIDE2
    access-group IN-DMZ in int INSIDE


## DHCP Attacks
### DHCp Starvation
A DHCP starvation attack is a type of denial of service (DoS) attack where an attacker floods a DHCP server with a large number of forged DHCP requests, typically using spoofed MAC addresses, to exhaust the available IP address pool.
The attack works by sending numerous DHCP DISCOVER packets, each with a different source MAC address, causing the DHCP server to allocate IP addresses to these fake requests preventing legitimate clients from obtaining IP Addresses.
To prevent this attack the switch is configured with port-security to limit the  number of mac addresses read from a particular port.
To configure port security:

On all access ports:
* _int range fa0/3-24,gig0/1-2_
* _switchport mode access_
* _switchport port-security maximum 2_
* _switchport port-security sticky_
* _switchport port-security violation shutdown_
* _exit_

### DHCp Spoofing
Type of cyber attack where an attacker sets up a rogue DHCP server on a network to respond to DHCP requests from clients, providing malicious configurations.
This allows the attacker to manipulate network traffic, often redirecting it through their own machine.
The primary goal of DHCP spoofing is to intercept and control the communication between devices and the legitimate DHCP server, which can lead to various malicious activities such as
intercepting sensitive data or disrupting network services.

To configure DHCP snooping:
* _int range fa0/1-2_
**Note:** For a trusted ports
* _ip dhcp snooping trust_
* _exit_
* _int range fa0/3-24,gig0/1-2_
* _ip dhcp snooping limit rate 4_
* _exit_
* _ip dhcp snooping vlan 10,20,40_

## ARP Attacks
Cyberattacks where a malicious actor sends fake ARP (Address Resolution Protocol) messages within a local network to associate their MAC address with the IP address of a legitimate
system.
There are two main types of ARP attacks: ARP spoofing and ARP poisoning. ARP spoofing involves sending fake ARP packets to link an attacker's MAC address with an IP of a computer
already on the LAN. ARP poisoning occurs after a successful ARP spoofing, where the attacker changes the company's ARP table to contain falsified MAC maps.

Prevention methods include using static ARP entries, enabling Dynamic ARP Inspection (DAI) on managed switches, implementing packet filtering, using encrypted traffic (HTTPS, SSH, or
VPNs) and deploying intrusion detection systems.

To configure ARP attacks:

Upon enabling DHCP snooping
* _ip dhcp snooping_
* _ip dhcp snooping vlan 10,20,40_
* _ip arp inspection vlan 10,20,40_
* _int range fa0/1-2_
* _ip dhcp snooping trust_
* _ip arp inspection trust_
* _ip arp inspection validate src-mac dst-mac ip_

## VLAN ATTACKS
On trunk ports no negotiate  and change native vlan is enabled.

* _int range fa0/1-2_
* _switchport nonegotiate_
* _switchport trunk native vlan 999_
* _exit_

All unused ports  are adminstratively shutdown.

