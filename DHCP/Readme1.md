Topology (simple)
PC1 --- SW1 --- R1 (Router)
PC2 ---/

IP addressing / plan

LAN network: 192.168.10.0/24

R1 G0/0 (gateway) = 192.168.10.1/24

DHCP pool: .100 – .200

Internet not required for this lab

Devices to place in Packet Tracer

1 Router (e.g., 1941) → R1

1 Switch (2960) → SW1

2 PCs → PC1, PC2

Step-by-step

Cable & power on

Connect PC1 Fa0 to SW1 Fa0/1, PC2 Fa0 to SW1 Fa0/2. SW1 Fa0/24 → R1 G0/0 (use straight-through). Turn devices on.

Configure router interface

Open R1 CLI and paste:

enable
configure terminal
hostname R1
no ip domain-lookup
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit


Exclude addresses you don’t want distributed (gateway, servers, reserved):

R1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.99


(That reserves .1–.99 for static assignments; DHCP will hand out .100–.254 unless pool restricted.)

Create DHCP pool on router

R1(config)# ip dhcp pool OFFICE
R1(dhcp-config)# network 192.168.10.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# dns-server 8.8.8.8
R1(dhcp-config)# lease 7
R1(dhcp-config)# exit


Configure PCs to use DHCP

On PC1: Desktop → IP Configuration → select DHCP (clear any static IP).

On PC2: same.

Watch DORA (Simulation mode)

Switch Packet Tracer to Simulation (bottom-right).

Click Event Filters → Enable DHCP / BOOTP (or UDP) and ICMP optional.

On PC1, in Desktop → Command Prompt → ipconfig /renew (or simply enable DHCP and reboot PC).

In Simulation you’ll see: DHCPDISCOVER (client broadcast) → DHCPOFFER (router) → DHCPREQUEST → DHCPACK. Inspect each packet (inspect icon) to view DHCP options (offered IP, default gateway, DNS).

Verify on router

R1# show ip dhcp binding
IP address      Client-ID/Lease expiration        Type
192.168.10.100  0100.50aa.bbcc.0102               automatic
R1# show ip dhcp pool
Pool OFFICE :
 Utilization/Size: 2/101
 Subnet: 192.168.10.0/255.255.255.0


Verify on PC

Desktop → IP Configuration → confirm assigned IP (e.g., 192.168.10.100) and gateway 192.168.10.1.

ping 192.168.10.1 should succeed.

Troubleshooting checklist

If PC gets APIPA (169.254.x.x) or no IP: check router interface up show ip interface brief.

If router shows no bindings: ensure the router has DHCP pool matching the client’s subnet.

If DHCPOFFER not seen: check switch port mode (should be access), cable, and Simulation filters.

debug ip dhcp server events (use carefully in lab) shows DHCP activity
