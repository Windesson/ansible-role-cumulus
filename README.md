# Cumulus OS - Intent Based Configurator
This PlayBook will automatically configure interface and quagga file based on the YAML variables.

Interface syntax:
--------------
interface: swp45-46,swp47,swp48
interface: swp41-43,swp4
interface: swp1-40

To run it: 
------------
ansible-playbook provison.yml

Check OUTPUT
-------------
cd /files

# Plaubook Configuration

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
