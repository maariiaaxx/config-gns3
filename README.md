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
