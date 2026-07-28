# config-gns3
gns3

Based on your topology, I'll provide complete configurations for all 5 switches. Based on the diagram, you have:

1. **Switch DMZ** - Connects DMZ servers
2. **Internal Switch** - Core switch with VLANs for internal zones
3. **Switch Backup** - For backup servers (Zone de sauvegarde)
4. **Switch Users** - For Zone Utilisateurs (if separate from internal switch)
5. **Switch Security** - For Zone Sécurité (if separate from internal switch)

However, looking at the diagram, the **Internal Switch** is the main switch that handles all VLANs (Users, Critical, Service, Security). The DMZ and Backup switches are separate. Let me provide configurations for all:

---

## 🔧 **Switch 1: Internal Switch** (Core Switch)

This is the main switch that handles VLANs for all internal zones and connects to the Router Interne via trunk.

### Network Interfaces
```
# GigabitEthernet0/0: Trunk to Router Interne (G0/1)
# GigabitEthernet0/1-4: Access ports for Zone Utilisateurs (VLAN 10)
# GigabitEthernet0/5-8: Access ports for Zone Critique (VLAN 20)
# GigabitEthernet0/9-12: Access ports for Zone Service (VLAN 30)
# GigabitEthernet0/13-16: Access ports for Zone Sécurité (VLAN 40)
# GigabitEthernet0/17-20: Trunk to other switches (optional)
```

### Complete Configuration
```bash
! Enter global configuration mode
configure terminal

! Set hostname
hostname Switch_Internal

! Disable DNS lookup
no ip domain-lookup

! Enable password encryption
service password-encryption

! Create banner
banner motd #
WARNING: Unauthorized access is prohibited.
#

! Set enable password
enable secret Cisco123

! ------------------------------------------------------------------
! Create VLANs
! ------------------------------------------------------------------
vlan 10
 name Zone_Utilisateurs
!
vlan 20
 name Zone_Critique
!
vlan 30
 name Zone_Service
!
vlan 40
 name Zone_Securite
!
vlan 50
 name Zone_Sauvegarde
!
vlan 999
 name Native_VLAN
!

! ------------------------------------------------------------------
! Configure VLAN interface for management
! ------------------------------------------------------------------
interface vlan 40
 description Management_VLAN
 ip address 172.16.40.252 255.255.255.0
 no shutdown
!

! ------------------------------------------------------------------
! Configure default gateway
! ------------------------------------------------------------------
ip default-gateway 172.16.40.254

! ------------------------------------------------------------------
! Configure Trunk Port to Router Interne
! ------------------------------------------------------------------
interface GigabitEthernet0/0
 description Trunk_to_Router_Interne
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30,40
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown
!

! ------------------------------------------------------------------
! Configure Access Ports - Zone Utilisateurs (VLAN 10)
! ------------------------------------------------------------------
interface range GigabitEthernet0/1 - 4
 description Zone_Utilisateurs_VLAN_10
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 spanning-tree portfast
 no shutdown
!

! ------------------------------------------------------------------
! Configure Access Ports - Zone Critique (VLAN 20)
! ------------------------------------------------------------------
interface range GigabitEthernet0/5 - 8
 description Zone_Critique_VLAN_20
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 spanning-tree portfast
 no shutdown
!

! ------------------------------------------------------------------
! Configure Access Ports - Zone Service (VLAN 30)
! ------------------------------------------------------------------
interface range GigabitEthernet0/9 - 12
 description Zone_Service_VLAN_30
 switchport mode access
 switchport access vlan 30
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 spanning-tree portfast
 no shutdown
!

! ------------------------------------------------------------------
! Configure Access Ports - Zone Sécurité (VLAN 40)
! ------------------------------------------------------------------
interface range GigabitEthernet0/13 - 16
 description Zone_Securite_VLAN_40
 switchport mode access
 switchport access vlan 40
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 spanning-tree portfast
 no shutdown
!

! ------------------------------------------------------------------
! Configure Trunk Ports to other switches (optional)
! ------------------------------------------------------------------
interface GigabitEthernet0/17
 description Trunk_to_Switch_DMZ
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 30,40,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown
!

interface GigabitEthernet0/18
 description Trunk_to_Switch_Sauvegarde
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 20,50,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown
!

interface GigabitEthernet0/19
 description Trunk_to_Switch_Users
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown
!

interface GigabitEthernet0/20
 description Trunk_to_Switch_Securite
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 40,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown
!

! ------------------------------------------------------------------
! Configure Spanning Tree (STP)
! ------------------------------------------------------------------
! Set as root bridge for all VLANs
spanning-tree vlan 1 root primary
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root primary
spanning-tree vlan 30 root primary
spanning-tree vlan 40 root primary
spanning-tree vlan 50 root primary
!
! Enable Rapid PVST+
spanning-tree mode rapid-pvst
!
! PortFast for access ports
spanning-tree portfast default

! ------------------------------------------------------------------
! Configure SNMP for monitoring (optional)
! ------------------------------------------------------------------
snmp-server community public RO
snmp-server community private RW

! ------------------------------------------------------------------
! Configure logging
! ------------------------------------------------------------------
logging buffered 4096
logging console notifications
!

! ------------------------------------------------------------------
! Configure SSH for management
! ------------------------------------------------------------------
! Generate RSA keys
crypto key generate rsa general-keys modulus 2048
!
! Configure SSH
ip ssh version 2
ip ssh authentication-retries 3
ip ssh time-out 60
!
! Configure VTY lines
line vty 0 15
 transport input ssh
 login local
 exec-timeout 5 0
!
! Create local user
username admin secret Cisco123

! ------------------------------------------------------------------
! Save configuration
! ------------------------------------------------------------------
write memory
```

---

## 🔧 **Switch 2: DMZ Switch**

This switch connects to the DMZ network and the Firewall Frontal.

### Network Interfaces
```
# GigabitEthernet0/0: Trunk to Firewall Frontal
# GigabitEthernet0/1-4: Access ports for DMZ servers
```

### Complete Configuration
```bash
! Enter global configuration mode
configure terminal

! Set hostname
hostname Switch_DMZ

! Disable DNS lookup
no ip domain-lookup

! Enable password encryption
service password-encryption

! Create banner
banner motd #
WARNING: Unauthorized access is prohibited.
#

! Set enable password
enable secret Cisco123

! ------------------------------------------------------------------
! Create VLANs (only DMZ needed)
! ------------------------------------------------------------------
vlan 50
 name DMZ
!
vlan 999
 name Native_VLAN
!

! ------------------------------------------------------------------
! Configure VLAN interface for management
! ------------------------------------------------------------------
interface vlan 50
 description DMZ_Management
 ip address 10.0.30.252 255.255.255.0
 no shutdown
!

! ------------------------------------------------------------------
! Configure default gateway
! ------------------------------------------------------------------
ip default-gateway 10.0.30.254

! ------------------------------------------------------------------
! Configure Trunk Port to Firewall Frontal
! ------------------------------------------------------------------
interface GigabitEthernet0/0
 description Trunk_to_Firewall_Frontal
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 50,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown
!

! ------------------------------------------------------------------
! Configure Access Ports for DMZ Servers (VLAN 50)
! ------------------------------------------------------------------
interface range GigabitEthernet0/1 - 4
 description DMZ_Server_VLAN_50
 switchport mode access
 switchport access vlan 50
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 spanning-tree portfast
 no shutdown
!

! ------------------------------------------------------------------
! Configure Trunk to Internal Switch (if needed)
! ------------------------------------------------------------------
interface GigabitEthernet0/5
 description Trunk_to_Internal_Switch
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 50,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown
!

! ------------------------------------------------------------------
! Configure Spanning Tree
! ------------------------------------------------------------------
spanning-tree mode rapid-pvst
spanning-tree portfast default

! ------------------------------------------------------------------
! Configure SSH
! ------------------------------------------------------------------
crypto key generate rsa general-keys modulus 2048
!
ip ssh version 2
line vty 0 15
 transport input ssh
 login local
 exec-timeout 5 0
!
username admin secret Cisco123

! ------------------------------------------------------------------
! Save configuration
! ------------------------------------------------------------------
write memory
```

---

## 🔧 **Switch 3: Backup Switch** (Zone de sauvegarde)

This switch handles backup servers (Primary and Secondary).

### Network Interfaces
```
# GigabitEthernet0/0: Trunk to Internal Switch
# GigabitEthernet0/1-2: Access ports for Backup servers
```

### Complete Configuration
```bash
! Enter global configuration mode
configure terminal

! Set hostname
hostname Switch_Backup

! Disable DNS lookup
no ip domain-lookup

! Enable password encryption
service password-encryption

! Create banner
banner motd #
WARNING: Unauthorized access is prohibited.
#

! Set enable password
enable secret Cisco123

! ------------------------------------------------------------------
! Create VLANs
! ------------------------------------------------------------------
vlan 50
 name Backup_Network
!
vlan 999
 name Native_VLAN
!

! ------------------------------------------------------------------
! Configure VLAN interface for management
! ------------------------------------------------------------------
interface vlan 50
 description Backup_Management
 ip address 192.168.100.252 255.255.255.0
 no shutdown
!

! ------------------------------------------------------------------
! Configure default gateway
! ------------------------------------------------------------------
ip default-gateway 192.168.100.254

! ------------------------------------------------------------------
! Configure Trunk to Internal Switch
! ------------------------------------------------------------------
interface GigabitEthernet0/0
 description Trunk_to_Internal_Switch
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 50,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown
!

! ------------------------------------------------------------------
! Configure Access Ports for Backup Servers (VLAN 50)
! ------------------------------------------------------------------
interface GigabitEthernet0/1
 description Backup_Primaire
 switchport mode access
 switchport access vlan 50
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 spanning-tree portfast
 no shutdown
!

interface GigabitEthernet0/2
 description Backup_Secondaire
 switchport mode access
 switchport access vlan 50
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 spanning-tree portfast
 no shutdown
!

! ------------------------------------------------------------------
! Configure Spanning Tree
! ------------------------------------------------------------------
spanning-tree mode rapid-pvst
spanning-tree portfast default

! ------------------------------------------------------------------
! Configure SSH
! ------------------------------------------------------------------
crypto key generate rsa general-keys modulus 2048
!
ip ssh version 2
line vty 0 15
 transport input ssh
 login local
 exec-timeout 5 0
!
username admin secret Cisco123

! ------------------------------------------------------------------
! Save configuration
! ------------------------------------------------------------------
write memory
```

---

## 🔧 **Switch 4: Users Switch** (Zone Utilisateurs - Optional)

If you want to physically separate the user zone from the core switch, use this switch.

### Network Interfaces
```
# GigabitEthernet0/0: Trunk to Internal Switch
# GigabitEthernet0/1-4: Access ports for Users
```

### Complete Configuration
```bash
! Enter global configuration mode
configure terminal

! Set hostname
hostname Switch_Users

! Disable DNS lookup
no ip domain-lookup

! Enable password encryption
service password-encryption

! Create banner
banner motd #
WARNING: Unauthorized access is prohibited.
#

! Set enable password
enable secret Cisco123

! ------------------------------------------------------------------
! Create VLANs
! ------------------------------------------------------------------
vlan 10
 name Zone_Utilisateurs
!
vlan 999
 name Native_VLAN
!

! ------------------------------------------------------------------
! Configure VLAN interface for management
! ------------------------------------------------------------------
interface vlan 10
 description Users_Management
 ip address 172.16.10.252 255.255.255.0
 no shutdown
!

! ------------------------------------------------------------------
! Configure default gateway
! ------------------------------------------------------------------
ip default-gateway 172.16.10.254

! ------------------------------------------------------------------
! Configure Trunk to Internal Switch
! ------------------------------------------------------------------
interface GigabitEthernet0/0
 description Trunk_to_Internal_Switch
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown
!

! ------------------------------------------------------------------
! Configure Access Ports for Users (VLAN 10)
! ------------------------------------------------------------------
interface range GigabitEthernet0/1 - 4
 description Zone_Utilisateurs_VLAN_10
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 spanning-tree portfast
 no shutdown
!

! ------------------------------------------------------------------
! Configure Spanning Tree
! ------------------------------------------------------------------
spanning-tree mode rapid-pvst
spanning-tree portfast default

! ------------------------------------------------------------------
! Configure SSH
! ------------------------------------------------------------------
crypto key generate rsa general-keys modulus 2048
!
ip ssh version 2
line vty 0 15
 transport input ssh
 login local
 exec-timeout 5 0
!
username admin secret Cisco123

! ------------------------------------------------------------------
! Save configuration
! ------------------------------------------------------------------
write memory
```

---

## 🔧 **Switch 5: Security Switch** (Zone Sécurité - Optional)

If you want to physically separate the security zone, use this switch.

### Network Interfaces
```
# GigabitEthernet0/0: Trunk to Internal Switch
# GigabitEthernet0/1-4: Access ports for Security devices
```

### Complete Configuration
```bash
! Enter global configuration mode
configure terminal

! Set hostname
hostname Switch_Security

! Disable DNS lookup
no ip domain-lookup

! Enable password encryption
service password-encryption

! Create banner
banner motd #
WARNING: Unauthorized access is prohibited.
#

! Set enable password
enable secret Cisco123

! ------------------------------------------------------------------
! Create VLANs
! ------------------------------------------------------------------
vlan 40
 name Zone_Securite
!
vlan 999
 name Native_VLAN
!

! ------------------------------------------------------------------
! Configure VLAN interface for management
! ------------------------------------------------------------------
interface vlan 40
 description Security_Management
 ip address 172.16.40.253 255.255.255.0
 no shutdown
!

! ------------------------------------------------------------------
! Configure default gateway
! ------------------------------------------------------------------
ip default-gateway 172.16.40.254

! ------------------------------------------------------------------
! Configure Trunk to Internal Switch
! ------------------------------------------------------------------
interface GigabitEthernet0/0
 description Trunk_to_Internal_Switch
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 40,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown
!

! ------------------------------------------------------------------
! Configure Access Ports for Security Devices (VLAN 40)
! ------------------------------------------------------------------
interface range GigabitEthernet0/1 - 4
 description Zone_Securite_VLAN_40
 switchport mode access
 switchport access vlan 40
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 spanning-tree portfast
 no shutdown
!

! ------------------------------------------------------------------
! Configure Spanning Tree
! ------------------------------------------------------------------
spanning-tree mode rapid-pvst
spanning-tree portfast default

! ------------------------------------------------------------------
! Configure SSH
! ------------------------------------------------------------------
crypto key generate rsa general-keys modulus 2048
!
ip ssh version 2
line vty 0 15
 transport input ssh
 login local
 exec-timeout 5 0
!
username admin secret Cisco123

! ------------------------------------------------------------------
! Save configuration
! ------------------------------------------------------------------
write memory
```

---

## 📊 **Complete VLAN and IP Addressing Summary**

| Device | VLAN | Network | Interface | IP Address | Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Internal Switch** | - | - | VLAN 40 | 172.16.40.252 | 172.16.40.254 |
| **DMZ Switch** | 50 | 10.0.30.0/24 | VLAN 50 | 10.0.30.252 | 10.0.30.254 |
| **Backup Switch** | 50 | 192.168.100.0/24 | VLAN 50 | 192.168.100.252 | 192.168.100.254 |
| **Users Switch** | 10 | 172.16.10.0/24 | VLAN 10 | 172.16.10.252 | 172.16.10.254 |
| **Security Switch** | 40 | 172.16.40.0/24 | VLAN 40 | 172.16.40.253 | 172.16.40.254 |

---

## 🔗 **Inter-Switch Connections**

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  [Router Frontal] ──────┐                                                │
│                         │                                                │
│                    [DMZ Switch] ──────┐                                  │
│                         │             │                                  │
│  [Router Interne] ── [Internal Switch] ── [Backup Switch]              │
│                         │             │                                  │
│                    [Users Switch]     [Security Switch]                  │
│                         │             │                                  │
│                    [Users PCs]      [Security Devices]                   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 🔍 **Verification Commands**

### On Any Switch:
```bash
# Show VLANs
show vlan brief

# Show trunk ports
show interfaces trunk

# Show interface status
show interfaces status

# Show running configuration
show running-config

# Show MAC address table
show mac address-table

# Show spanning tree
show spanning-tree

# Show port security
show port-security

# Test connectivity
ping 172.16.40.254
ping 172.16.40.252  ! Ping internal switch
ping 10.0.30.254
```

### Testing from a User PC (VLAN 10):
```bash
ipconfig /all
ping 172.16.10.254       ! Gateway (Router Interne)
ping 172.16.10.252       ! Users Switch
ping 172.16.20.10        ! Domain Controller
ping 10.0.30.100         ! Web Server in DMZ
ping 8.8.8.8             ! Internet
tracert 8.8.8.8
```

---

## ⚠️ **Important Notes**

1. **Switch Images**: For GNS3, use Cisco IOU Layer 2 images like `i86bi-linux-l2-adventerprise-15.1b.bin` for full Layer 2 switching capabilities.

2. **Trunking**: Ensure trunk ports use `dot1q` encapsulation (Cisco switches use this by default).

3. **Native VLAN**: I use VLAN 999 as Native VLAN to avoid VLAN 1 security issues. The Native VLAN should match on both ends of a trunk.

4. **Port Security**: I've configured port security on access ports to prevent MAC flooding attacks. Adjust the MAC count based on your needs.

5. **Spanning Tree**: The Internal Switch is configured as the root bridge for all VLANs to ensure a loop-free topology.

6. **Management**: All switches are accessible via SSH from the Security VLAN (VLAN 40) for secure management.

7. **Resource Usage**: If you're limited on resources, you can:
   - Combine Users and Security switches into the Internal Switch
   - Use fewer VLANs initially and scale up
   - Use Layer 3 switches for routing if needed

8. **VLAN Creation Order**: Some switches require VLANs to be created in global config mode before assigning them to interfaces. The configuration above does this correctly.

# CONFIG ROUTER 

Based on your topology, I'll provide complete configurations for both routers. In your diagram, you have two router-like devices: the **Router Frontal** (which connects to the Internet and DMZ) and the **Router Interne** (which handles internal routing between VLANs). I'll provide Cisco IOS configurations for both.

---

## 🚦 **Router Frontal Configuration** (Edge Router)

This router sits at the edge, connecting to the Internet, DMZ, and the internal router.

### Network Interfaces
```
# Interface assignments
# GigabitEthernet0/0: Exterieur (Internet) - 10.0.1.254/24
# GigabitEthernet0/1: DMZ - 10.0.30.254/24
# GigabitEthernet0/2: To Router Interne - 192.168.0.1/30
```

### Complete Router Frontal Configuration
```bash
! Enter global configuration mode
configure terminal

! Set hostname
hostname Router_Frontal

! Disable DNS lookup (to avoid delays)
no ip domain-lookup

! Enable password encryption
service password-encryption

! Create banner
banner motd #
WARNING: Unauthorized access is prohibited.
#

! Set enable password
enable secret Cisco123

! ------------------------------------------------------------------
! Configure interfaces
! ------------------------------------------------------------------

! Interface to Exterieur (Internet)
interface GigabitEthernet0/0
 description Connection_to_Internet
 ip address 10.0.1.254 255.255.255.0
 ip nat outside
 ip virtual-reassembly
 no shutdown
 duplex auto
 speed auto
!

! Interface to DMZ
interface GigabitEthernet0/1
 description Connection_to_DMZ
 ip address 10.0.30.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
 no shutdown
 duplex auto
 speed auto
!

! Interface to Router Interne
interface GigabitEthernet0/2
 description Connection_to_Router_Interne
 ip address 192.168.0.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly
 no shutdown
 duplex auto
 speed auto
!

! ------------------------------------------------------------------
! Configure default route to Internet
! ------------------------------------------------------------------
ip route 0.0.0.0 0.0.0.0 10.0.1.1

! ------------------------------------------------------------------
! Configure static routes to internal networks
! ------------------------------------------------------------------
ip route 172.16.10.0 255.255.255.0 192.168.0.2
ip route 172.16.20.0 255.255.255.0 192.168.0.2
ip route 172.16.30.0 255.255.255.0 192.168.0.2
ip route 172.16.40.0 255.255.255.0 192.168.0.2

! ------------------------------------------------------------------
! Configure NAT (Network Address Translation)
! ------------------------------------------------------------------

! Define ACL for NAT (internal networks that can access Internet)
access-list 1 permit 10.0.30.0 0.0.0.255
access-list 1 permit 192.168.0.0 0.0.0.3
access-list 1 permit 172.16.10.0 0.0.0.255
access-list 1 permit 172.16.20.0 0.0.0.255
access-list 1 permit 172.16.30.0 0.0.0.255
access-list 1 permit 172.16.40.0 0.0.0.255

! Configure PAT (Port Address Translation)
ip nat inside source list 1 interface GigabitEthernet0/0 overload

! Static NAT for Web Server in DMZ (port forwarding)
ip nat inside source static tcp 10.0.30.100 80 interface GigabitEthernet0/0 80
ip nat inside source static tcp 10.0.30.100 443 interface GigabitEthernet0/0 443

! Static NAT for SSH to Web Server (optional)
! ip nat inside source static tcp 10.0.30.100 22 interface GigabitEthernet0/0 2222

! ------------------------------------------------------------------
! Configure Firewall (ACLs)
! ------------------------------------------------------------------

! Create ACL to protect the router itself
access-list 100 deny ip any host 10.0.1.254 log
access-list 100 deny ip any host 10.0.30.254 log
access-list 100 deny ip any host 192.168.0.1 log
access-list 100 permit ip any any

! Apply ACL to all interfaces (inbound)
interface GigabitEthernet0/0
 ip access-group 100 in
!
interface GigabitEthernet0/1
 ip access-group 100 in
!
interface GigabitEthernet0/2
 ip access-group 100 in
!

! Create ACL for traffic filtering (forwarding)
! Allow HTTP/HTTPS from Internet to Web Server
access-list 101 permit tcp any host 10.0.30.100 eq 80
access-list 101 permit tcp any host 10.0.30.100 eq 443

! Allow ICMP for testing (optional - be careful)
access-list 101 permit icmp any host 10.0.30.100 echo
access-list 101 permit icmp any host 10.0.30.100 echo-reply

! Allow SSH from Internal networks to DMZ (for management)
access-list 101 permit tcp 192.168.0.0 0.0.0.3 host 10.0.30.100 eq 22
access-list 101 permit tcp 172.16.10.0 0.0.0.255 host 10.0.30.100 eq 22
access-list 101 permit tcp 172.16.40.0 0.0.0.255 host 10.0.30.100 eq 22

! Deny everything else (implicit)
access-list 101 deny ip any any log

! Apply ACL to DMZ interface (inbound)
interface GigabitEthernet0/1
 ip access-group 101 in
!

! Allow traffic from Router Interne to Internet and DMZ
access-list 102 permit ip 192.168.0.0 0.0.0.3 any
access-list 102 permit ip 172.16.0.0 0.0.255.255 any

! Apply ACL to Router Interne interface (inbound)
interface GigabitEthernet0/2
 ip access-group 102 in
!

! ------------------------------------------------------------------
! Configure SSH for secure management
! ------------------------------------------------------------------
! Generate RSA keys
crypto key generate rsa general-keys modulus 2048
!
! Configure SSH
ip ssh version 2
ip ssh authentication-retries 3
ip ssh time-out 60
!
! Configure VTY lines for SSH
line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
!

! Create local user for SSH
username admin secret Cisco123

! ------------------------------------------------------------------
! Enable logging
! ------------------------------------------------------------------
logging buffered 4096
logging console notifications
!

! ------------------------------------------------------------------
! Save configuration
! ------------------------------------------------------------------
write memory
```

---

## 🚦 **Router Interne Configuration** (Internal Router)

This router handles all internal routing between VLANs and connects to the edge router. It uses **router-on-a-stick** with VLAN sub-interfaces.

### Network Interfaces
```
# Interface assignments
# GigabitEthernet0/0: To Router Frontal - 192.168.0.2/30
# GigabitEthernet0/1.10: VLAN 10 - Zone Utilisateurs - 172.16.10.254/24
# GigabitEthernet0/1.20: VLAN 20 - Zone Critique - 172.16.20.254/24
# GigabitEthernet0/1.30: VLAN 30 - Zone Service - 172.16.30.254/24
# GigabitEthernet0/1.40: VLAN 40 - Zone Sécurité - 172.16.40.254/24
```

### Complete Router Interne Configuration
```bash
! Enter global configuration mode
configure terminal

! Set hostname
hostname Router_Interne

! Disable DNS lookup
no ip domain-lookup

! Enable password encryption
service password-encryption

! Create banner
banner motd #
WARNING: Unauthorized access is prohibited.
#

! Set enable password
enable secret Cisco123

! ------------------------------------------------------------------
! Configure interfaces
! ------------------------------------------------------------------

! Interface to Router Frontal
interface GigabitEthernet0/0
 description Connection_to_Router_Frontal
 ip address 192.168.0.2 255.255.255.252
 ip nat inside
 no shutdown
 duplex auto
 speed auto
!

! Main trunk interface (no IP address)
interface GigabitEthernet0/1
 description Trunk_to_Internal_Switch
 no ip address
 no shutdown
 duplex auto
 speed auto
!

! Sub-interface for Zone Utilisateurs (VLAN 10)
interface GigabitEthernet0/1.10
 description VLAN_10_Zone_Utilisateurs
 encapsulation dot1Q 10
 ip address 172.16.10.254 255.255.255.0
 ip helper-address 172.16.20.10
 no shutdown
!

! Sub-interface for Zone Critique (VLAN 20)
interface GigabitEthernet0/1.20
 description VLAN_20_Zone_Critique
 encapsulation dot1Q 20
 ip address 172.16.20.254 255.255.255.0
 no shutdown
!

! Sub-interface for Zone Service (VLAN 30)
interface GigabitEthernet0/1.30
 description VLAN_30_Zone_Service
 encapsulation dot1Q 30
 ip address 172.16.30.254 255.255.255.0
 no shutdown
!

! Sub-interface for Zone Sécurité (VLAN 40)
interface GigabitEthernet0/1.40
 description VLAN_40_Zone_Securite
 encapsulation dot1Q 40
 ip address 172.16.40.254 255.255.255.0
 no shutdown
!

! ------------------------------------------------------------------
! Configure default route to Router Frontal
! ------------------------------------------------------------------
ip route 0.0.0.0 0.0.0.0 192.168.0.1

! ------------------------------------------------------------------
! Configure static routes for networks (connected routes)
! All VLANs are directly connected, so no static routes needed.
! Only need to reach Router Frontal and Internet.
! ------------------------------------------------------------------

! ------------------------------------------------------------------
! Configure DHCP Relay (IP Helper) for VLAN 10 (Users)
! ------------------------------------------------------------------
! The ip helper-address is already configured on G0/1.10

! ------------------------------------------------------------------
! Configure Access Control Lists (Internal Firewall)
! ------------------------------------------------------------------

! Create ACLs for Zone-to-Zone traffic control

! 1. Allow Zone Utilisateurs to Zone Service (Web & Mail)
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.30.100 eq 80
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.30.100 eq 443
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.30.101 eq 25
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.30.101 eq 587
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.30.101 eq 110
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.30.101 eq 143
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.30.101 eq 993

! 2. Allow Zone Utilisateurs to Zone Critique (AD Authentication)
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.20.10 eq 389
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.20.10 eq 636
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.20.10 eq 88
access-list 110 permit udp 172.16.10.0 0.0.0.255 host 172.16.20.10 eq 88
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.20.10 eq 53
access-list 110 permit udp 172.16.10.0 0.0.0.255 host 172.16.20.10 eq 53
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.20.11 eq 389
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.20.11 eq 636
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.20.11 eq 88
access-list 110 permit udp 172.16.10.0 0.0.0.255 host 172.16.20.11 eq 88
access-list 110 permit tcp 172.16.10.0 0.0.0.255 host 172.16.20.11 eq 53
access-list 110 permit udp 172.16.10.0 0.0.0.255 host 172.16.20.11 eq 53

! 3. Allow Zone Service to Zone Critique (All traffic)
access-list 110 permit ip 172.16.30.0 0.0.0.255 172.16.20.0 0.0.0.255

! 4. Allow Zone Sécurité to all zones (Monitoring)
access-list 110 permit ip 172.16.40.0 0.0.0.255 172.16.10.0 0.0.0.255
access-list 110 permit ip 172.16.40.0 0.0.0.255 172.16.20.0 0.0.0.255
access-list 110 permit ip 172.16.40.0 0.0.0.255 172.16.30.0 0.0.0.255

! 5. Allow Zone Critique to Zone Service (Mail relay)
access-list 110 permit tcp 172.16.20.10 0.0.0.0 host 172.16.30.101 eq 25
access-list 110 permit tcp 172.16.20.11 0.0.0.0 host 172.16.30.101 eq 25

! 6. Allow ICMP for testing (from any zone)
access-list 110 permit icmp 172.16.10.0 0.0.0.255 any echo
access-list 110 permit icmp 172.16.10.0 0.0.0.255 any echo-reply
access-list 110 permit icmp 172.16.20.0 0.0.0.255 any echo
access-list 110 permit icmp 172.16.20.0 0.0.0.255 any echo-reply
access-list 110 permit icmp 172.16.30.0 0.0.0.255 any echo
access-list 110 permit icmp 172.16.30.0 0.0.0.255 any echo-reply
access-list 110 permit icmp 172.16.40.0 0.0.0.255 any echo
access-list 110 permit icmp 172.16.40.0 0.0.0.255 any echo-reply

! 7. Deny User to Security zone
access-list 110 deny ip 172.16.10.0 0.0.0.255 172.16.40.0 0.0.0.255 log

! 8. Deny User to Critical zone (except allowed authentication)
access-list 110 deny ip 172.16.10.0 0.0.0.255 172.16.20.0 0.0.0.255 log

! 9. Allow all other traffic to Router Frontal (for Internet access)
access-list 110 permit ip any 192.168.0.0 0.0.0.3

! 10. Deny everything else (implicitly)
access-list 110 deny ip any any log

! Apply ACL to internal VLAN interfaces (inbound)
interface GigabitEthernet0/1.10
 ip access-group 110 in
!
interface GigabitEthernet0/1.20
 ip access-group 110 in
!
interface GigabitEthernet0/1.30
 ip access-group 110 in
!
interface GigabitEthernet0/1.40
 ip access-group 110 in
!

! ------------------------------------------------------------------
! Protect the router itself (Input ACLs)
! ------------------------------------------------------------------

! Allow SSH, ICMP, and established connections to the router
access-list 120 permit tcp 172.16.40.0 0.0.0.255 any eq 22
access-list 120 permit icmp any any echo
access-list 120 permit icmp any any echo-reply
access-list 120 permit tcp any any established
access-list 120 permit udp any any established
access-list 120 deny ip any any log

! Apply to all interfaces (inbound)
interface GigabitEthernet0/0
 ip access-group 120 in
!
interface GigabitEthernet0/1.10
 ip access-group 120 in
!
interface GigabitEthernet0/1.20
 ip access-group 120 in
!
interface GigabitEthernet0/1.30
 ip access-group 120 in
!
interface GigabitEthernet0/1.40
 ip access-group 120 in
!

! ------------------------------------------------------------------
! Configure SSH for secure management
! ------------------------------------------------------------------
crypto key generate rsa general-keys modulus 2048
!
ip ssh version 2
ip ssh authentication-retries 3
ip ssh time-out 60
!
! Configure VTY lines for SSH
line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
!
username admin secret Cisco123

! ------------------------------------------------------------------
! Enable logging
! ------------------------------------------------------------------
logging buffered 4096
logging console notifications
!

! ------------------------------------------------------------------
! Save configuration
! ------------------------------------------------------------------
write memory
```

---

## 📊 **Network Diagram Summary**

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                Internet                                │
│                              (10.0.1.0/24)                            │
│                                  │                                     │
│                           [10.0.1.1]                                  │
│                                  │                                     │
│                    ┌─────────────┴─────────────┐                      │
│                    │    Router Frontal         │                      │
│                    │  10.0.1.254 (G0/0)       │                      │
│                    │  10.0.30.254 (G0/1)      │                      │
│                    │  192.168.0.1 (G0/2)      │                      │
│                    └─────────────┬─────────────┘                      │
│                         ┌────────┴────────┐                          │
│                         │                 │                          │
│                    [DMZ]              [Lien Interne]                  │
│               10.0.30.0/24         192.168.0.0/30                    │
│                    │                 │                               │
│               [Web Server]      [192.168.0.2]                        │
│              10.0.30.100            │                                │
│                         ┌───────────┴───────────┐                    │
│                         │    Router Interne     │                    │
│                         │  192.168.0.2 (G0/0)  │                    │
│                         │  G0/1.10 (VLAN 10)   │                    │
│                         │  G0/1.20 (VLAN 20)   │                    │
│                         │  G0/1.30 (VLAN 30)   │                    │
│                         │  G0/1.40 (VLAN 40)   │                    │
│                         └───────────┬───────────┘                    │
│                                     │                                │
│                          ┌──────────┴──────────┐                     │
│                          │  Internal Switch    │                     │
│                          │  (Trunk - VLANs)    │                     │
│                          └──────────┬──────────┘                     │
│           ┌───────┬───────┬───────┼───────┬───────┬───────┐        │
│           │       │       │       │       │       │       │        │
│      ┌────┴──┐┌───┴───┐┌──┴───┐┌─┴───┐┌──┴───┐┌──┴───┐┌──┴───┐  │
│      │VLAN 10││VLAN 20││VLAN 30││VLAN 40││VLAN 10││VLAN 20││VLAN 30│  │
│      │Users  ││Critique││Service││Securité││Users  ││Critique││Service│  │
│      └───────┘└───────┘└───────┘└───────┘└───────┘└───────┘└───────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔍 **Verification Commands**

```bash
# On Router Frontal
show ip interface brief
show ip route
show ip nat translations
show ip nat statistics
show access-lists
show running-config
ping 10.0.1.1
ping 192.168.0.2
traceroute 8.8.8.8

# On Router Interne
show ip interface brief
show ip route
show ip arp
show vlan
show access-lists
show running-config
ping 192.168.0.1
ping 10.0.1.254
ping 10.0.30.100
traceroute 8.8.8.8

# Test from a client in VLAN 10
ipconfig /all
ping 172.16.10.254
ping 172.16.20.10
ping 10.0.30.100
ping 8.8.8.8
tracert 8.8.8.8
```

---

## 📝 **Key Configuration Summary**

| Feature | Router Frontal | Router Interne |
| :--- | :--- | :--- |
| **WAN Interface** | G0/0: 10.0.1.254/24 | G0/0: 192.168.0.2/30 |
| **DMZ Interface** | G0/1: 10.0.30.254/24 | - |
| **Internal Trunk** | - | G0/1: VLAN sub-interfaces |
| **Default Route** | 0.0.0.0/0 → 10.0.1.1 | 0.0.0.0/0 → 192.168.0.1 |
| **Static Routes** | Internal networks via 192.168.0.2 | Connected routes (no static needed) |
| **NAT** | PAT (overload) + Static NAT for web server | No NAT (handled by Router Frontal) |
| **Firewall** | ACL 101, 102 on interfaces | ACL 110, 120 on interfaces |
| **DHCP Relay** | - | IP helper-address on VLAN 10 |

---

## ⚠️ **Important Notes**

1. **Router Images**: Make sure you have the correct Cisco IOS image loaded in GNS3. Recommended: `c7200-adventerprisek9-mz.152-4.S6.bin` or `c3725-adventerprisek9-mz.124-15.T14.image`.

2. **Switch Configuration**: The internal switch must be configured with VLANs 10, 20, 30, 40 and a trunk port to the Router Interne.

3. **DHCP**: The `ip helper-address` on VLAN 10 forwards DHCP requests to the Domain Controller (172.16.20.10).

4. **NAT**: Router Frontal handles all NAT. Router Interne does not need NAT.

5. **Security**: Both routers have ACLs that act as a stateful firewall. The default policy is to deny everything except explicitly allowed traffic.

6. **Management**: SSH is enabled on both routers. Access is restricted to the Zone Sécurité network (VLAN 40) for security.

7. **Logging**: Both routers log denied packets. Monitor these logs regularly for security incidents.

# 📋 **Complete GNS3 Topology Creation Steps - Line by Line**

Based on your network diagram, here's a comprehensive step-by-step guide to build the entire topology in GNS3 from scratch.

---

## PHASE 1: GNS3 Setup & Image Import

### Step 1: Install GNS3
```bash
# Download GNS3 from official website
# Install GNS3 (Windows/Mac/Linux)
# During installation, ensure VMware/VirtualBox integration is enabled
```

### Step 2: Import Cisco IOS Images

```bash
# For Routers - Import Cisco 7200 or 3725 images
1. Open GNS3 → Edit → Preferences
2. Click on "IOS Routers" → New
3. Browse to your router image: c7200-adventerprisek9-mz.152-4.S6.bin
4. Follow wizard:
   - Name: "Cisco 7200"
   - Default RAM: 256 MB
   - Default IOS: Select the image
   - Platform: c7200
5. Click Finish

# For Switches - Import IOU Layer 2 images
1. Go to Preferences → IOU Devices
2. Click New → Browse to: i86bi-linux-l2-adventerprise-15.1b.bin
3. Follow wizard:
   - Name: "IOU L2 Switch"
   - RAM: 256 MB
   - Ensure "Enable L2 switching" is checked
4. Click Finish

# For Linux VMs
1. Download Ubuntu Server ISO
2. In GNS3: Edit → Preferences → QEMU VMs
3. Click New → Import appliance from file
4. Select the Ubuntu appliance or create custom
5. Configure:
   - Name: "Ubuntu Server"
   - RAM: 1024 MB
   - Network adapters: 4
```

### Step 3: Import Windows VMs

```bash
# Windows Server 2022/2019
1. Download Windows Server ISO from Microsoft
2. In GNS3: Edit → Preferences → QEMU VMs
3. Click New → Create a new QEMU VM
4. Configure:
   - Name: "Windows Server 2022"
   - RAM: 2048 MB (minimum 4096 MB recommended)
   - Network adapters: 2
   - Disk size: 40 GB
   - OS Type: Windows
5. Install Windows Server on the VM
6. Set "Linked Base VM" option for cloning

# Windows 10/11 Client
1. In GNS3: Edit → Preferences → QEMU VMs
2. Click New → Create a new QEMU VM
3. Configure:
   - Name: "Windows 10 Client"
   - RAM: 2048 MB
   - Network adapters: 1
   - Disk size: 30 GB
   - OS Type: Windows
4. Install Windows 10/11 on the VM
5. Set "Linked Base VM" option for cloning
```

---

## PHASE 2: Build the Topology - Network Devices Placement

### Step 4: Place Network Devices

```bash
# From the "Nodes" panel on the left side:

1. Drag "Ethernet Switch" onto canvas (for Exterieur)
   - Name: "Switch_Exterieur"

2. Drag "IOU L2 Switch" onto canvas (Internal Core Switch)
   - Name: "Switch_Internal"

3. Drag "IOU L2 Switch" onto canvas (DMZ Switch)
   - Name: "Switch_DMZ"

4. Drag "IOU L2 Switch" onto canvas (Backup Switch)
   - Name: "Switch_Backup"

5. Drag "Cisco 7200" onto canvas (Router Frontal)
   - Name: "Router_Frontal"

6. Drag "Cisco 7200" onto canvas (Router Interne)
   - Name: "Router_Interne"

7. Drag "Ubuntu Server" onto canvas (Firewall Frontal)
   - Name: "Firewall_Frontal"

8. Drag "Ubuntu Server" onto canvas (Firewall Interne)
   - Name: "Firewall_Interne"

9. Drag "Cloud" onto canvas (Internet)
   - Name: "Internet_Cloud"
   - Configure with NAT or local connection

10. Drag "Ethernet Switch" onto canvas (Users Switch - optional)
    - Name: "Switch_Users"

11. Drag "Ethernet Switch" onto canvas (Security Switch - optional)
    - Name: "Switch_Securite"
```

### Step 5: Place Server VMs

```bash
# DMZ Zone
12. Drag "Ubuntu Server" onto canvas
    - Name: "Web_Server_DMZ"
    - IP: 10.0.30.100

# Zone Critique
13. Drag "Windows Server 2022" onto canvas
    - Name: "DC_Primaire"
    - IP: 172.16.20.10

14. Drag "Windows Server 2022" onto canvas
    - Name: "DC_Secondaire"
    - IP: 172.16.20.11

15. Drag "Ubuntu Server" onto canvas
    - Name: "Backup_Primaire"
    - IP: 192.168.100.10

16. Drag "Ubuntu Server" onto canvas
    - Name: "Backup_Secondaire"
    - IP: 192.168.100.11

# Zone Service
17. Drag "Ubuntu Server" onto canvas
    - Name: "Serveur_WER"
    - IP: 172.16.30.100

18. Drag "Ubuntu Server" onto canvas
    - Name: "Serveur_MAIL"
    - IP: 172.16.30.101

19. Drag "Ubuntu Server" onto canvas
    - Name: "Base_Donnees"
    - IP: 172.16.30.102

# Zone Sécurité
20. Drag "Ubuntu Server" onto canvas
    - Name: "SIEM"
    - IP: 172.16.40.10

21. Drag "Ubuntu Server" onto canvas
    - Name: "SOAR"
    - IP: 172.16.40.11

22. Drag "Ubuntu Server" onto canvas
    - Name: "EDR"
    - IP: 172.16.40.12

23. Drag "Ubuntu Server" onto canvas
    - Name: "CTI"
    - IP: 172.16.40.13

24. Drag "Ubuntu Server" onto canvas
    - Name: "Analyse_Securite"
    - IP: 172.16.40.20

# Zone Utilisateurs
25. Drag "Windows 10 Client" onto canvas
    - Name: "Utilisateur_1"
    - IP: 172.16.10.10

26. Drag "Windows 10 Client" onto canvas
    - Name: "Utilisateur_2"
    - IP: 172.16.10.11
```

---

## PHASE 3: Physical Connections

### Step 6: Connect Switch_Exterieur

```bash
# Connect Switch_Exterieur to Internet Cloud and Router Frontale
27. Internet_Cloud (eth0) → Switch_Exterieur (F0/0)
28. Router_Frontal (G0/0) → Switch_Exterieur (F0/1)
29. Firewall_Frontal (eth0) → Switch_Exterieur (F0/2)  # If using Linux firewall
```

### Step 7: Connect Router_Frontal

```bash
# Router_Frontal interfaces
30. Router_Frontal (G0/0) → Switch_Exterieur (F0/1)  [Already done]
31. Router_Frontal (G0/1) → Switch_DMZ (F0/0)
32. Router_Frontal (G0/2) → Router_Interne (G0/0)
```

### Step 8: Connect Switch_DMZ

```bash
# DMZ Switch connections
33. Switch_DMZ (F0/1) → Web_Server_DMZ (eth0)
34. Switch_DMZ (F0/2) → Firewall_Frontal (eth1)  # If using Linux firewall
35. Switch_DMZ (F0/3) → Router_Frontal (G0/1)  # Already done
36. Switch_DMZ (F0/4) → Switch_Internal (G0/17)  # Trunk to core
```

### Step 9: Connect Router_Interne

```bash
# Router_Interne interfaces
37. Router_Interne (G0/0) → Router_Frontal (G0/2)  # Already done
38. Router_Interne (G0/1) → Switch_Internal (G0/0)  # Trunk
39. Router_Interne (G0/2) → Firewall_Interne (eth0)  # If using Linux firewall
```

### Step 10: Connect Switch_Internal (Core Switch)

```bash
# Internal Core Switch connections - Access Ports (VLAN 10 - Zone Utilisateurs)
40. Switch_Internal (G0/1) → Utilisateur_1 (eth0)
41. Switch_Internal (G0/2) → Utilisateur_2 (eth0)
42. Switch_Internal (G0/3) → Switch_Users (G0/0)  # Trunk to Users Switch

# Access Ports (VLAN 20 - Zone Critique)
43. Switch_Internal (G0/5) → DC_Primaire (eth0)
44. Switch_Internal (G0/6) → DC_Secondaire (eth0)
45. Switch_Internal (G0/7) → Switch_Backup (G0/0)  # Trunk to Backup

# Access Ports (VLAN 30 - Zone Service)
46. Switch_Internal (G0/9) → Serveur_WER (eth0)
47. Switch_Internal (G0/10) → Serveur_MAIL (eth0)
48. Switch_Internal (G0/11) → Base_Donnees (eth0)

# Access Ports (VLAN 40 - Zone Sécurité)
49. Switch_Internal (G0/13) → SIEM (eth0)
50. Switch_Internal (G0/14) → SOAR (eth0)
51. Switch_Internal (G0/15) → EDR (eth0)
52. Switch_Internal (G0/16) → CTI (eth0)
53. Switch_Internal (G0/17) → Analyse_Securite (eth0)
54. Switch_Internal (G0/18) → Switch_Securite (G0/0)  # Trunk to Security

# Trunk Ports
55. Switch_Internal (G0/0) → Router_Interne (G0/1)  # Already done
56. Switch_Internal (G0/17) → Switch_DMZ (F0/4)  # Already done
57. Switch_Internal (G0/19) → Firewall_Interne (eth1)  # If using Linux
58. Switch_Internal (G0/20) → Firewall_Interne (eth2)  # If using Linux
```

### Step 11: Connect Firewall_Frontal (Linux)

```bash
# Firewall_Frontal interfaces
59. Firewall_Frontal (eth0) → Switch_Exterieur (F0/2)
60. Firewall_Frontal (eth1) → Switch_DMZ (F0/2)
61. Firewall_Frontal (eth2) → Firewall_Interne (eth0)
```

### Step 12: Connect Firewall_Interne (Linux)

```bash
# Firewall_Interne interfaces
62. Firewall_Interne (eth0) → Firewall_Frontal (eth2)
63. Firewall_Interne (eth1) → Switch_Internal (G0/19)  # Trunk VLANs
64. Firewall_Interne (eth2) → Switch_Internal (G0/20)  # Trunk VLANs
```

### Step 13: Connect Switch_Backup

```bash
# Backup Switch connections
65. Switch_Backup (G0/1) → Backup_Primaire (eth0)
66. Switch_Backup (G0/2) → Backup_Secondaire (eth0)
67. Switch_Backup (G0/0) → Switch_Internal (G0/7)  # Already done
```

### Step 14: Connect Switch_Users (Optional)

```bash
# Users Switch connections
68. Switch_Users (G0/1) → Utilisateur_1 (eth0)
69. Switch_Users (G0/2) → Utilisateur_2 (eth0)
70. Switch_Users (G0/0) → Switch_Internal (G0/3)  # Already done
```

### Step 15: Connect Switch_Securite (Optional)

```bash
# Security Switch connections
71. Switch_Securite (G0/1) → SIEM (eth0)
72. Switch_Securite (G0/2) → SOAR (eth0)
73. Switch_Securite (G0/3) → EDR (eth0)
74. Switch_Securite (G0/4) → CTI (eth0)
75. Switch_Securite (G0/5) → Analyse_Securite (eth0)
76. Switch_Securite (G0/0) → Switch_Internal (G0/18)  # Already done
```

---

## PHASE 4: Configure Network Devices

### Step 16: Configure Router_Frontal

```bash
# Connect to Router_Frontal console
# Enter privileged EXEC mode
enable
configure terminal

# Hostname
hostname Router_Frontal

# Configure interfaces
interface GigabitEthernet0/0
 description Internet_Connection
 ip address 10.0.1.254 255.255.255.0
 ip nat outside
 no shutdown

interface GigabitEthernet0/1
 description DMZ_Connection
 ip address 10.0.30.254 255.255.255.0
 ip nat inside
 no shutdown

interface GigabitEthernet0/2
 description To_Router_Interne
 ip address 192.168.0.1 255.255.255.252
 ip nat inside
 no shutdown

# Default route
ip route 0.0.0.0 0.0.0.0 10.0.1.1

# Static routes to internal networks
ip route 172.16.10.0 255.255.255.0 192.168.0.2
ip route 172.16.20.0 255.255.255.0 192.168.0.2
ip route 172.16.30.0 255.255.255.0 192.168.0.2
ip route 172.16.40.0 255.255.255.0 192.168.0.2

# NAT Configuration
access-list 1 permit 10.0.30.0 0.0.0.255
access-list 1 permit 192.168.0.0 0.0.0.3
access-list 1 permit 172.16.10.0 0.0.0.255
access-list 1 permit 172.16.20.0 0.0.0.255
access-list 1 permit 172.16.30.0 0.0.0.255
access-list 1 permit 172.16.40.0 0.0.0.255

ip nat inside source list 1 interface GigabitEthernet0/0 overload

# SSH Configuration
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
username admin secret Cisco123
line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0

# Save configuration
write memory
```

### Step 17: Configure Router_Interne

```bash
# Connect to Router_Interne console
enable
configure terminal

# Hostname
hostname Router_Interne

# Configure interfaces
interface GigabitEthernet0/0
 description To_Router_Frontal
 ip address 192.168.0.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 description Trunk_to_Switch
 no ip address
 no shutdown

# Sub-interfaces for VLANs
interface GigabitEthernet0/1.10
 description VLAN_10_Zone_Utilisateurs
 encapsulation dot1Q 10
 ip address 172.16.10.254 255.255.255.0
 ip helper-address 172.16.20.10
 no shutdown

interface GigabitEthernet0/1.20
 description VLAN_20_Zone_Critique
 encapsulation dot1Q 20
 ip address 172.16.20.254 255.255.255.0
 no shutdown

interface GigabitEthernet0/1.30
 description VLAN_30_Zone_Service
 encapsulation dot1Q 30
 ip address 172.16.30.254 255.255.255.0
 no shutdown

interface GigabitEthernet0/1.40
 description VLAN_40_Zone_Securite
 encapsulation dot1Q 40
 ip address 172.16.40.254 255.255.255.0
 no shutdown

# Default route
ip route 0.0.0.0 0.0.0.0 192.168.0.1

# ACL Configuration
access-list 110 permit tcp 172.16.10.0 0.0.0.255 172.16.30.0 0.0.0.255 eq 80
access-list 110 permit tcp 172.16.10.0 0.0.0.255 172.16.30.0 0.0.0.255 eq 443
access-list 110 permit tcp 172.16.10.0 0.0.0.255 172.16.20.0 0.0.0.255 eq 389
access-list 110 permit udp 172.16.10.0 0.0.0.255 172.16.20.0 0.0.0.255 eq 53
access-list 110 permit ip 172.16.40.0 0.0.0.255 any
access-list 110 deny ip 172.16.10.0 0.0.0.255 172.16.40.0 0.0.0.255 log
access-list 110 deny ip any any log

# Apply ACL to sub-interfaces
interface GigabitEthernet0/1.10
 ip access-group 110 in

interface GigabitEthernet0/1.20
 ip access-group 110 in

interface GigabitEthernet0/1.30
 ip access-group 110 in

interface GigabitEthernet0/1.40
 ip access-group 110 in

# SSH Configuration
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
username admin secret Cisco123
line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0

# Save configuration
write memory
```

### Step 18: Configure Switch_Internal

```bash
# Connect to Switch_Internal console
enable
configure terminal

# Hostname
hostname Switch_Internal

# Create VLANs
vlan 10
 name Zone_Utilisateurs
vlan 20
 name Zone_Critique
vlan 30
 name Zone_Service
vlan 40
 name Zone_Securite
vlan 50
 name Zone_Sauvegarde
vlan 999
 name Native_VLAN
exit

# Management VLAN
interface vlan 40
 description Management
 ip address 172.16.40.252 255.255.255.0
 no shutdown

# Default gateway
ip default-gateway 172.16.40.254

# Trunk to Router_Interne
interface GigabitEthernet0/0
 description Trunk_to_Router_Interne
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30,40
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown

# Access ports for Zone Utilisateurs (VLAN 10)
interface range GigabitEthernet0/1 - 4
 description Zone_Utilisateurs_VLAN_10
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 spanning-tree portfast
 no shutdown

# Access ports for Zone Critique (VLAN 20)
interface range GigabitEthernet0/5 - 8
 description Zone_Critique_VLAN_20
 switchport mode access
 switchport access vlan 20
 switchport port-security maximum 2
 spanning-tree portfast
 no shutdown

# Access ports for Zone Service (VLAN 30)
interface range GigabitEthernet0/9 - 12
 description Zone_Service_VLAN_30
 switchport mode access
 switchport access vlan 30
 switchport port-security maximum 2
 spanning-tree portfast
 no shutdown

# Access ports for Zone Sécurité (VLAN 40)
interface range GigabitEthernet0/13 - 16
 description Zone_Securite_VLAN_40
 switchport mode access
 switchport access vlan 40
 switchport port-security maximum 2
 spanning-tree portfast
 no shutdown

# Trunk ports to other switches
interface GigabitEthernet0/17
 description Trunk_to_DMZ
 switchport trunk allowed vlan 30,40,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown

interface GigabitEthernet0/18
 description Trunk_to_Backup
 switchport trunk allowed vlan 20,50,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown

# Spanning Tree
spanning-tree mode rapid-pvst
spanning-tree vlan 1 root primary
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root primary
spanning-tree vlan 30 root primary
spanning-tree vlan 40 root primary
spanning-tree portfast default

# SSH Configuration
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
username admin secret Cisco123
line vty 0 15
 transport input ssh
 login local
 exec-timeout 5 0

# Save configuration
write memory
```

### Step 19: Configure Switch_DMZ

```bash
enable
configure terminal
hostname Switch_DMZ

vlan 50
 name DMZ
vlan 999
 name Native_VLAN
exit

interface vlan 50
 ip address 10.0.30.252 255.255.255.0
 no shutdown

ip default-gateway 10.0.30.254

interface GigabitEthernet0/0
 description Trunk_to_Router_Frontal
 switchport trunk allowed vlan 50,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown

interface range GigabitEthernet0/1 - 4
 description DMZ_Access_Ports
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
 no shutdown

interface GigabitEthernet0/5
 description Trunk_to_Internal
 switchport trunk allowed vlan 50,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown

spanning-tree mode rapid-pvst
spanning-tree portfast default

crypto key generate rsa general-keys modulus 2048
ip ssh version 2
username admin secret Cisco123
line vty 0 15
 transport input ssh
 login local

write memory
```

### Step 20: Configure Switch_Backup

```bash
enable
configure terminal
hostname Switch_Backup

vlan 50
 name Backup_Network
vlan 999
 name Native_VLAN
exit

interface vlan 50
 ip address 192.168.100.252 255.255.255.0
 no shutdown

ip default-gateway 192.168.100.254

interface GigabitEthernet0/0
 description Trunk_to_Internal
 switchport trunk allowed vlan 50,999
 switchport trunk native vlan 999
 switchport mode trunk
 no shutdown

interface GigabitEthernet0/1
 description Backup_Primaire
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
 no shutdown

interface GigabitEthernet0/2
 description Backup_Secondaire
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
 no shutdown

spanning-tree mode rapid-pvst
spanning-tree portfast default

crypto key generate rsa general-keys modulus 2048
ip ssh version 2
username admin secret Cisco123
line vty 0 15
 transport input ssh
 login local

write memory
```

---

## PHASE 5: Configure Linux Firewalls

### Step 21: Configure Firewall_Frontal (Ubuntu Linux)

```bash
# Boot the Firewall_Frontal VM
# Login and configure network interfaces

# Edit network configuration
sudo nano /etc/netplan/01-netcfg.yaml

# Add the following:
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 10.0.1.254/24
      gateway4: 10.0.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
    eth1:
      addresses:
        - 10.0.30.254/24
    eth2:
      addresses:
        - 192.168.0.1/30

# Apply configuration
sudo netplan apply

# Enable IP forwarding
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Install iptables-persistent
sudo apt update
sudo apt install -y iptables-persistent

# Create firewall script
sudo nano /root/firewall-frontal.sh

# Add the firewall rules (from previous firewall configuration)
# Make it executable
sudo chmod +x /root/firewall-frontal.sh
sudo /root/firewall-frontal.sh

# Save iptables rules
sudo netfilter-persistent save

# Enable SSH for management
sudo systemctl enable ssh
sudo systemctl start ssh
```

### Step 22: Configure Firewall_Interne (Ubuntu Linux)

```bash
# Boot the Firewall_Interne VM
# Login and configure network interfaces

# Install VLAN support
sudo apt update
sudo apt install -y vlan
sudo modprobe 8021q
echo "8021q" | sudo tee -a /etc/modules

# Edit network configuration
sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.0.2/30
      gateway4: 192.168.0.1
    eth1:
      dhcp4: false
  vlans:
    vlan.10:
      id: 10
      link: eth1
      addresses:
        - 172.16.10.254/24
    vlan.20:
      id: 20
      link: eth1
      addresses:
        - 172.16.20.254/24
    vlan.30:
      id: 30
      link: eth1
      addresses:
        - 172.16.30.254/24
    vlan.40:
      id: 40
      link: eth1
      addresses:
        - 172.16.40.254/24

# Apply configuration
sudo netplan apply

# Enable IP forwarding
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Install iptables-persistent
sudo apt install -y iptables-persistent

# Create firewall script
sudo nano /root/firewall-interne.sh

# Add the firewall rules (from previous configuration)
sudo chmod +x /root/firewall-interne.sh
sudo /root/firewall-interne.sh

# Save rules
sudo netfilter-persistent save
```

---

## PHASE 6: Configure Windows Domain Controllers

### Step 23: Configure DC_Primaire

```bash
# Boot DC_Primaire VM

# Configure Static IP
Network Settings:
- IP Address: 172.16.20.10
- Subnet Mask: 255.255.255.0
- Default Gateway: 172.16.20.254
- Preferred DNS: 127.0.0.1
- Alternate DNS: 172.16.20.11

# Install Active Directory Domain Services
1. Open Server Manager
2. Click "Add roles and features"
3. Select "Active Directory Domain Services"
4. Click Next through wizard
5. Install

# Promote to Domain Controller
1. After installation, click "Promote this server to a domain controller"
2. Choose "Add a new forest"
3. Root domain name: projet.local
4. Set DSRM password: Cisco123!
5. Click Next through wizard
6. Install

# Create AD Users
1. Open Active Directory Users and Computers
2. Create Organizational Units:
   - Zones
   - Users
   - Servers
3. Create users:
   - Username: user1
   - Password: Cisco123!
   - Username: user2
   - Password: Cisco123!
```

### Step 24: Configure DC_Secondaire

```bash
# Boot DC_Secondaire VM

# Configure Static IP
Network Settings:
- IP Address: 172.16.20.11
- Subnet Mask: 255.255.255.0
- Default Gateway: 172.16.20.254
- Preferred DNS: 172.16.20.10
- Alternate DNS: 127.0.0.1

# Join as Secondary Domain Controller
1. Open Server Manager
2. Click "Add roles and features"
3. Select "Active Directory Domain Services"
4. Install
5. Promote to Domain Controller
6. Choose "Add a domain controller to an existing domain"
7. Domain: projet.local
8. Enter administrator credentials
9. Complete wizard

# Configure DNS replication
1. Open DNS Manager
2. Right-click on server
3. Properties
4. Ensure zone transfers are allowed
```

---

## PHASE 7: Configure Servers

### Step 25: Configure Web Server (DMZ)

```bash
# Boot Web_Server_DMZ VM

# Configure static IP
sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 10.0.30.100/24
      gateway4: 10.0.30.254
      nameservers:
        addresses: [172.16.20.10, 172.16.20.11]

sudo netplan apply

# Install Apache
sudo apt update
sudo apt install -y apache2 openssh-server

# Create test webpage
echo "<h1>Welcome to DMZ Web Server</h1>" | sudo tee /var/www/html/index.html

# Enable SSH
sudo systemctl enable ssh
sudo systemctl start ssh
```

### Step 26: Configure Serveur_WER

```bash
# Boot Serveur_WER VM

# Configure static IP
sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 172.16.30.100/24
      gateway4: 172.16.30.254
      nameservers:
        addresses: [172.16.20.10, 172.16.20.11]

sudo netplan apply

# Join domain (if Linux)
# Install Samba and join domain
sudo apt update
sudo apt install -y realmd sssd sssd-tools

# Join domain
sudo realm join --user=administrator projet.local

# Install application (example: Node.js)
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

### Step 27: Configure Serveur_MAIL

```bash
# Boot Serveur_MAIL VM

# Configure static IP
sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 172.16.30.101/24
      gateway4: 172.16.30.254
      nameservers:
        addresses: [172.16.20.10, 172.16.20.11]

sudo netplan apply

# Install Postfix mail server
sudo apt update
sudo apt install -y postfix mailutils

# Configure Postfix
sudo dpkg-reconfigure postfix
# Select: Internet Site
# System mail name: mail.projet.local

# Install Dovecot for IMAP
sudo apt install -y dovecot-core dovecot-imapd

# Configure basic mail accounts
sudo useradd -m -s /bin/bash user1
echo "user1:Cisco123!" | sudo chpasswd
sudo useradd -m -s /bin/bash user2
echo "user2:Cisco123!" | sudo chpasswd
```

### Step 28: Configure Base_Donnees

```bash
# Boot Base_Donnees VM

# Configure static IP
sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 172.16.30.102/24
      gateway4: 172.16.30.254
      nameservers:
        addresses: [172.16.20.10, 172.16.20.11]

sudo netplan apply

# Install MariaDB/MySQL
sudo apt update
sudo apt install -y mariadb-server

# Secure installation
sudo mysql_secure_installation

# Create database and user
sudo mysql -e "CREATE DATABASE projet_db;"
sudo mysql -e "CREATE USER 'webapp'@'%' IDENTIFIED BY 'Cisco123!';"
sudo mysql -e "GRANT ALL PRIVILEGES ON projet_db.* TO 'webapp'@'%';"
sudo mysql -e "FLUSH PRIVILEGES;"

# Allow remote access
sudo sed -i 's/bind-address.*/bind-address = 0.0.0.0/' /etc/mysql/mariadb.conf.d/50-server.cnf
sudo systemctl restart mariadb
```

---

## PHASE 8: Configure Security Zone

### Step 29: Configure SIEM (Wazuh)

```bash
# Boot SIEM VM

# Configure static IP
sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 172.16.40.10/24
      gateway4: 172.16.40.254
      nameservers:
        addresses: [172.16.20.10, 172.16.20.11]

sudo netplan apply

# Install Docker
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y docker-ce docker-compose

# Install Wazuh
curl -sO https://packages.wazuh.com/4.3/wazuh-install.sh
sudo bash wazuh-install.sh --generate-config-files

# Generate Wazuh configuration
sudo bash wazuh-install.sh --wazuh-indexer node-1 \
  --wazuh-server wazuh-1 \
  --wazuh-dashboard dashboard \
  --start-cluster

# Access Wazuh dashboard
# https://172.16.40.10
# Default credentials: admin/admin
```

### Step 30: Configure SOAR (Shuffle)

```bash
# Boot SOAR VM

# Configure static IP
sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 172.16.40.11/24
      gateway4: 172.16.40.254
      nameservers:
        addresses: [172.16.20.10, 172.16.20.11]

sudo netplan apply

# Install Docker (if not already installed)
sudo apt update
sudo apt install -y docker.io docker-compose

# Install Shuffle
git clone https://github.com/shuffle/shuffle.git
cd shuffle
sudo docker-compose up -d

# Access Shuffle
# http://172.16.40.11:3001
```

### Step 31: Configure EDR (Wazuh Agent)

```bash
# Install Wazuh Agent on all Windows and Linux machines

# For Linux servers:
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo apt install -y wazuh-agent

# Configure agent
sudo nano /var/ossec/etc/ossec.conf
# Set MANAGER_IP to 172.16.40.10

sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent

# For Windows:
# Download Wazuh agent installer
# Install and set manager IP: 172.16.40.10
```

---

## PHASE 9: Configure Users

### Step 32: Configure Utilisateur_1 & Utilisateur_2

```bash
# Boot Windows 10/11 VM

# Configure Static IP
Network Settings:
- IP Address: 172.16.10.10 (Utilisateur_1) / 172.16.10.11 (Utilisateur_2)
- Subnet Mask: 255.255.255.0
- Default Gateway: 172.16.10.254
- Preferred DNS: 172.16.20.10
- Alternate DNS: 172.16.20.11

# Join Domain
1. Open System Properties
2. Click "Change settings" under Computer name
3. Click "Change..."
4. Select Domain: projet.local
5. Enter domain admin credentials: administrator / Cisco123!
6. Restart when prompted

# Test Domain Login
1. Logout
2. Login as: projet\user1
3. Password: Cisco123!
```

---

## PHASE 10: Testing

### Step 33: Connectivity Tests

```bash
# From Router_Frontal
ping 10.0.1.1
ping 192.168.0.2
ping 172.16.10.254
ping 172.16.20.254
ping 172.16.30.254
ping 172.16.40.254

# From Utilisateur_1
ping 172.16.10.254
ping 172.16.20.10
ping 172.16.30.100
ping 10.0.30.100
ping 8.8.8.8
tracert 8.8.8.8

# Browse to Web Server
# Open browser: http://10.0.30.100
# Should see: "Welcome to DMZ Web Server"

# Check AD Authentication
# On Utilisateur_1, login as user1
# Should authenticate successfully

# Test Mail
# Use mail client to test SMTP/IMAP
```

---

## PHASE 11: Save and Backup

### Step 34: Save GNS3 Project

```bash
1. In GNS3, click File → Save Project
2. Name: "Enterprise_Lab_Project"
3. Click Save

# Backup configurations
# For routers and switches
show running-config
# Copy and save to a text file

# For Linux servers
sudo tar -czf /root/backup.tar.gz /etc/netplan /root/firewall*.sh

# For Windows servers
# Use Windows Backup utility
```

---

## 📊 **Final Topology Verification Checklist**

| Device | Status | IP Address | Verified |
| :--- | :--- | :--- | :--- |
| Router_Frontal | Running | 10.0.1.254, 10.0.30.254, 192.168.0.1 | ☐ |
| Router_Interne | Running | 192.168.0.2, 172.16.x.254 | ☐ |
| Switch_Internal | Running | 172.16.40.252 | ☐ |
| Switch_DMZ | Running | 10.0.30.252 | ☐ |
| Switch_Backup | Running | 192.168.100.252 | ☐ |
| Firewall_Frontal | Running | 10.0.1.254, 10.0.30.254, 192.168.0.1 | ☐ |
| Firewall_Interne | Running | 192.168.0.2, 172.16.x.254 | ☐ |
| Web_Server_DMZ | Running | 10.0.30.100 | ☐ |
| DC_Primaire | Running | 172.16.20.10 | ☐ |
| DC_Secondaire | Running | 172.16.20.11 | ☐ |
| Backup_Primaire | Running | 192.168.100.10 | ☐ |
| Serveur_WER | Running | 172.16.30.100 | ☐ |
| Serveur_MAIL | Running | 172.16.30.101 | ☐ |
| Base_Donnees | Running | 172.16.30.102 | ☐ |
| SIEM | Running | 172.16.40.10 | ☐ |
| SOAR | Running | 172.16.40.11 | ☐ |
| Utilisateur_1 | Running | 172.16.10.10 | ☐ |
| Utilisateur_2 | Running | 172.16.10.11 | ☐ |

---

## 🚀 **Start All Devices**

```bash
1. In GNS3, click "Start all devices" button
2. Wait for all devices to boot (3-5 minutes)
3. Test connectivity as shown in Phase 10
4. Your enterprise lab is now ready!
```

This completes the entire topology creation process! You now have a fully functional enterprise network lab with:
- ✅ 2 Routers
- ✅ 3-5 Switches  
- ✅ 2 Linux Firewalls
- ✅ 2 Domain Controllers
- ✅ 1 Web Server
- ✅ 2 Application Servers
- ✅ 1 Database Server
- ✅ 4 Security Devices
- ✅ 2 Client Workstations
