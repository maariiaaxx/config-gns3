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
