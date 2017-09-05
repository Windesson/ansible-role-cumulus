# Cumulus OS - Intent Based Configurator
This PlayBook will automatically configure the LINUX Interface and Quagga file based on the YAML variables.

Interface Syntax:
--------------
interface: swp45-46,swp47,swp48
interface: swp41-43,swp4
interface: swp1-40

To run it: 
------------
ansible-playbook provison.yml

Check ouput:
-------------
cd /files

# Playbook Configuration Syntax

Management Interface (YAML).

    mgmt_interface:
      description: vrf mgmt network
      ipv4: 192.168.100.10/24
      gateway: 192.168.100.1
      mac: 8c:ea:1b:20:cb:c0

Management Interface (Output)

    # The primary network interface
    auto eth0
    iface eth0 inet static
        vrf mgmt
        address 192.168.100.10/24
        gateway 192.168.100.1
        alias vrf mgmt network 
        
VLAN Interface (YAML).

    - vlan: 499
      description: Network Management
      ipv4: 192.168.9.100/24
      gateway: 192.168.9.1

VLAN Interface (Output)

    vlan_interfaces:
      - vlan: 499
        description: Network Management
        ipv4: 192.168.9.100/24
        gateway: 192.168.9.1
      
ACCESS Interface (YAML).

    access_interfaces:
     - interface: swp1-40
       description: Network DATA AND VOICE

     - interface: swp41-43,swp44
       description: Network DATA-ONLY
       vlans: 100,300

     - interface: swp45-46,swp47,swp48
       description: Network VOICE-ONLY
       vlans: 200

ACCESS Interface (Output)

    % for i in range(45, 46):
    auto swp${i}
    iface swp${i}
      alias Network VOICE-ONLY
      bridge-vids 200
      bridge-pvid 200
      mstpctl-bpduguard yes
      mstpctl-portadminedge yes
    % endfor

    auto swp47
    iface swp47
      alias Network VOICE-ONLY
      bridge-vids 200
      bridge-pvid 200
      mstpctl-bpduguard yes
      mstpctl-portadminedge yes

    auto swp48
    iface swp48
      alias Network VOICE-ONLY
      bridge-vids 200
      bridge-pvid 200
      mstpctl-bpduguard yes
      mstpctl-portadminedge yes


Quagga - Router (YAML)

    router:
       ospfv2:
          - pid: 1
            rid: 192.168.10.1
            no_passives:
               - swp47
               - swp48
               - vlan300
 
 Quagga - Router (OUTPUT)
 
     router ospf 1
      ospf router-id 192.168.10.1
      log-adjacency-changes
      auto-cost reference-bandwidth 1000000
      passive-interface default
      no passive-interface swp47
      no passive-interface swp48
      no passive-interface bridge.300

Quagga - Interfaces  (YAML)

    # The peers network interface
    auto swp47
    iface swp47 inet static
      alias uplink to AT&T
      address   192.168.7.1
      netmask   255.255.255.252
      link-speed 1000
      link-duplex full
    auto swp48
    iface swp48 inet static
      alias uplink to AT&T
      address   192.168.8.1
      netmask   255.255.255.252
      link-speed 1000
      link-duplex full
    # The vlan network interface

    auto bridge.300
    iface bridge.300
      alias cross-link to n0-dist-b
      address 192.168.3.1/30

    auto bridge.499
    iface bridge.499
      alias Network-Management
      address 192.168.9.200/24
      address-virtual 00:00:5e:00:01:01 192.168.9.1/24

    auto bridge.200
    iface bridge.200
      alias Network-Voice
      address 192.168.2.100/24
      address-virtual 00:00:5e:00:01:02 192.168.2.1/24

    auto bridge.100
    iface bridge.100
      alias Network-Data
      address 192.168.1.100/24
      address-virtual 00:00:5e:00:01:03 192.168.1.1/24

Quagga - Interfaces  (OUTPUT)

        !
        interface swp47
         ip pim sm
         ip ospf network point-to-point
         ip ospf 1 area 0
         link-detect
        !
        interface swp48
         ip pim sm
         ip ospf network point-to-point
         ip ospf 1 area 0
         link-detect
        !
        !
        interface bridge.300
         ip pim sm
         ip ospf network point-to-point
         ip ospf 1 area 0
         link-detect
        !
        interface bridge.499
         ip pim sm
         ip ospf 1 area 0
         link-detect
        !
        interface bridge.200
         ip pim sm
         ip ospf 1 area 0
         link-detect
        !
        interface bridge.100
         ip pim sm
         ip ospf 1 area 0
         link-detect
        !
        !
