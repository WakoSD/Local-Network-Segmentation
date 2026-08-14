
# Local-Network-Segmentation
Implemented an isolated laboratory network to practice subnetting, DHCP configuration, NAT behavior, and network segmentation concepts.

## Objective: 
The objective of this laboratory was to create a dedicated wireless network using an Apple AirPort Extreme A1521, providing an independent environment for experimentation, testing, and future homelab deployments.
The laboratory network was designed to operate as a separate private network behind a secondary router, allowing the creation of an isolated addressing space with its own DHCP service and NAT configuration.

The main goals of this laboratory were:
- Create a separate IPv4 subnet for laboratory devices.
- Configure an independent DHCP scope.
- Understand routing behavior between multiple private networks.
- Observe how NAT affects communication between networks.
- Establish a dedicated environment for future homelab experiments.

The laboratory network was intended to function as a separate private environment, allowing experimental devices and services to operate independently from the main home network.

## 1. Architecture
<img width="827" height="600" alt="Diagrama sin título drawio (1)" src="https://github.com/user-attachments/assets/801d6311-10a9-4190-9e74-16c3ed392269" />

## 2. Used infraestructure
| Device | Function |
|--------|----------|
| Main Router | Provides Internet access and main network |
| Apple AirPort Extreme | Secondary router for the lab network |
| Lab PC1 | Client inside the test environment |
| Personal Desktop PC | Device used for the initial configuration |

## 3. Initial network state
Before creating the laboratory environment, the existing home network configuration was identified.
The following command was used:
```
ipconfig
```
Initial network information:
```
IPv4 Address:
192.168.100.9

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.100.1
```
The main home network was identified as:
```
192.168.100.0/24
```
Initial architecture:
```
Internet
   |
Home Router
192.168.100.1
   |
Home Network
192.168.100.0/24
```

## 4. Apple AirPort Extreme Configuration
### 4.1 Firmware Update
Because I had not used this device in a while, it had pending updates. The AirPort Extreme firmware was updated from version 7.7.9 to 7.9.1

### 4.2 Wireless Network Configuration
A new wireless network was created:

**SSID:**
WakoHomeLab

Wireless security configuration:

**Security:**
WPA2 Personal

A dedicated password was configured, different from the main home WiFi password.

### 4.3 WAN Configuration

The AirPort was connected to the main home router using Ethernet.
Physical connection:
**Home Router LAN Port ➔ AirPort WAN Port **

AirPort WAN configuration:
- IPv4 Configuration: Using DHCP
- The AirPort receives a private IP address from the main home network.

### 4.4 Lab DHCP Configuration
A new internal network was created using:
```
192.168.200.0/24
```
DHCP configuration:
```
DHCP Range:

Start:
192.168.200.10

End:
192.168.200.200
```
The AirPort acts as the default gateway for this network.

Laboratory network:
```
Gateway:
192.168.200.1

Clients:
192.168.200.10 - 192.168.200.200
```
Saved the first IPs for default settings and future servers. 

## 5. NAT configuration
The AirPort was configured using:
Share a public IP address

The AirPort performs NAT between the home network and the laboratory network.

Final architecture:
```
                 Internet
                    |
             Home Router
             192.168.100.1
                    |
                    |
              WAN AirPort
                    |
                   NAT
                    |
             WakoHomeLab
             192.168.200.0/24
```
## 6. Double NAT Detection
After configuration, the AirPort reported:

**Double NAT detected**

Reason:
The ISP router and the AirPort were both performing NAT.
Network flow:
```
Internet
    |
    |
Home Router
192.168.100.1
    |
    |
NAT #1
    |
    |
AirPort
    |
    |
NAT #2
    |
    |
192.168.200.0/24
```
The Double NAT configuration was intentionally maintained because the goal was to create a separate laboratory environment.

## 7. Connectivity Validation
### 7.1 DHCP Validation
A laptop was connected to: **WakoHomeLab**

The device successfully received an IP address:
**192.168.200.x**

This confirmed:
- DHCP was working correctly.
- The AirPort was successfully managing the laboratory subnet.

### 7.2 Internet Access Validation
The following devices successfully accessed the Internet through WakoHomeLab:

|Device	|Result |
|--------|----------|
|Laptop	|Successful |
|iPhone	|Successful |

This confirmed:
- DHCP functionality.
- NAT functionality.
- Internet connectivity.

## 8. Network Communication Testing
### 8.1 Laboratory Network ➔ Home Router
From the laboratory laptop in CMD:
```
ping 192.168.100.1
```
Result:

✅ **Successful response**

Interpretation:
The laboratory network can communicate with the upstream home router because the AirPort WAN interface is connected to the main network.

### 8.2 Laboratory Network ➔ Home Network
Testing communication from the laboratory laptop to devices inside the home network showed mixed results.

Example:
```
ping 192.168.100.9
```
⚠️ Results depended on the target device configuration.
Some devices did not respond because host-based firewalls can block this traffic.
**Therefore, ping alone was not considered a complete measure of network isolation.**

### 8.3 Home Network ➔ Laboratory Network
Testing connectivity from the main home network to the laboratory network:
```
ping 192.168.200.x
```
Result:

❌ **No response**

Explanation:
The laboratory network is located behind the AirPort router and uses a separate private subnet:
**192.168.200.0/24**

The main home router only has knowledge of the local home network: 192.168.100.0/24 and does not contain a routing entry for the laboratory subnet.
Because no route exists between:
192.168.100.0/24 and 192.168.200.0/24
devices in the main network cannot directly initiate communication with devices inside the laboratory network.

The AirPort acts as a boundary router performing NAT, allowing outbound communication from the laboratory network while preventing unsolicited inbound connections from the main network.

## 9. Network Summary
Testing showed that the AirPort Extreme created a separate private subnet and established a NAT boundary between the two networks.

The observed communication behavior was:

|Traffic Direction|Result|
|-----------------|------|
|Laboratory → Internet|	Allowed|
|Laboratory → Home Router|	Allowed|
|Laboratory → Some Home Devices|	Possible depending on host firewall configuration|
|Home Network → Laboratory Devices|	Restricted|

This behavior is expected from a secondary router performing NAT. The AirPort maintains connectivity between its WAN and LAN interfaces while preventing direct inbound access from the upstream network unless routing or forwarding rules are configured.

## 10. Conclusions
The laboratory successfully demonstrated the creation of a secondary private network using an Apple AirPort Extreme A1521.

The implementation provided:

- ✅ Separate IPv4 addressing space.
- ✅ Independent DHCP management.
- ✅ NAT-based network boundary.
- ✅ Dedicated wireless environment.
- ✅ Internet access for laboratory devices.

This laboratory established a foundation for exploring more advanced networking concepts in the future, including:

- Routing between private networks.
- Firewall-based segmentation.
- VLAN implementation.
- Network monitoring.
- Security controls.

Laboratory Status: ✅ Operational
Security Objective: ⚠️ Partially Achieved

Next step will be working on Firewall based segmentation adn VLAN implementation
