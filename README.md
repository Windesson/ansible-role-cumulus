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

    auto bridge.499
    iface bridge.499
      alias Network Management
      address 192.168.9.100/24
      gateway   192.168.9.1
      
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


Router-Router Interface (YAML).

    peer_interfaces:
      - interface: swp47
        description: uplink to AT&T
        ipv4: 192.168.7.1/30
        ospfv2: 1 area 0
        speed: 1000
        duplex: full

      - interface: swp48
        description: uplink to AT&T	
        ipv4: 192.168.8.1/30
        ospfv2: 1 area 0
        speed: 1000
        duplex: full

Router-Router Interface (Ouput).

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


Portchannel_ Interface (YAML).

    portchannel_interfaces:
      - portchannel: 1
        description: Downlink to n0-access-a, swp49
        interface: swp1
        clag_id: 1

      - portchannel: 2
        description: Downlink to n0-access-b, swp50
        interface: swp2
        clag_id: 2

      - portchannel: 3
        interface: swp7-8
        description: cross-link to n0-dist-c
        mlag_crosslink: true
        clagd_backup_ip: 172.28.17.3

Portchannel_ Interface (Output).

    auto swp1
    iface swp1
      alias Downlink to n0-access-a, swp49

    auto bond1
    iface bond1
       bond-slaves swp1
       bond-mode 802.3ad
       bond-miimon 100
       bond-use-carrier 1
       bond-xmit-hash-policy layer3+4
       bond-lacp-rate 1
       bond-min-links 1
       clag-id 1
       clagd-priority 1024

    auto swp2
    iface swp2
      alias Downlink to n0-access-b, swp50

    auto bond2
    iface bond2
       bond-slaves swp2
       bond-mode 802.3ad
       bond-miimon 100
       bond-use-carrier 1
       bond-xmit-hash-policy layer3+4
       bond-lacp-rate 1
       bond-min-links 1
       clag-id 2
       clagd-priority 1024

    auto swp7
    iface swp7
      alias cross-link to n0-dist-c

    auto swp8
    iface swp8
      alias cross-link to n0-dist-c

    auto bond3
    iface bond3
       bond-slaves swp7 swp8
       bond-mode 802.3ad
       bond-miimon 100
       bond-use-carrier 1
       bond-xmit-hash-policy layer3+4
       bond-lacp-rate 1
       bond-min-links 1

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
