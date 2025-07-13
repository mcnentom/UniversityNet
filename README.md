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
* STP & BPDU Guard
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
    * Differnt VLAN for Unsuded Ports
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







