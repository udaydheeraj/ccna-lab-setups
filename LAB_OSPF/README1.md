Single-Area OSPF (Area 0)
Goal

Build a small network with 3 routers and 2 PCs. Run OSPF in Area 0 everywhere and verify end-to-end pings.

Topology
PC1 — SW1 — R1 — R2 — R3 — SW2 — PC2


(Use 2960 switches; routers like 1941/2901/2911. Use “Auto Connection” or Copper Straight-Through for PC–SW/ SW–Rtr, and Copper Cross-Over for router–router if needed.)

IP Addressing
Node	Interface	IP/Mask	Note
PC1	NIC	192.168.1.10 /24	GW 192.168.1.1
PC2	NIC	192.168.3.10 /24	GW 192.168.3.1
R1	G0/0	192.168.1.1 /24	to SW1/PC1
R1	G0/1	10.0.12.1 /30	to R2
R2	G0/0	10.0.12.2 /30	to R1
R2	G0/1	10.0.23.1 /30	to R3
R3	G0/0	10.0.23.2 /30	to R2
R3	G0/1	192.168.3.1 /24	to SW2/PC2
Step-by-Step
1) Build the topology

Place 2 PCs, 2 switches, 3 routers.

Cable up as shown.

2) Configure PC IPs

PC1: IP 192.168.1.10, Mask 255.255.255.0, Gateway 192.168.1.1

PC2: IP 192.168.3.10, Mask 255.255.255.0, Gateway 192.168.3.1

3) Configure router interfaces

R1

enable
conf t
hostname R1
no ip domain-lookup
interface g0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
interface g0/1
 ip address 10.0.12.1 255.255.255.252
 no shutdown


R2

enable
conf t
hostname R2
no ip domain-lookup
interface g0/0
 ip address 10.0.12.2 255.255.255.252
 no shutdown
!
interface g0/1
 ip address 10.0.23.1 255.255.255.252
 no shutdown


R3

enable
conf t
hostname R3
no ip domain-lookup
interface g0/0
 ip address 10.0.23.2 255.255.255.252
 no shutdown
!
interface g0/1
 ip address 192.168.3.1 255.255.255.0
 no shutdown

4) Enable OSPF (Area 0 everywhere)

R1

router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0


R2

router ospf 1
 router-id 2.2.2.2
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.23.0 0.0.0.3 area 0


R3

router ospf 1
 router-id 3.3.3.3
 network 10.0.23.0 0.0.0.3 area 0
 network 192.168.3.0 0.0.0.255 area 0


(Optional hardening: avoid OSPF hellos toward PCs)

R1(config)# router ospf 1
R1(config-router)# passive-interface g0/0
R3(config)# router ospf 1
R3(config-router)# passive-interface g0/1

5) Verify neighbors and routes

Neighbors:
show ip ospf neighbor

Interfaces/timers/DR-BDR:
show ip ospf interface g0/1 (on R1/R2) and show ip ospf interface g0/0 (on R2/R3)

Routes:
show ip route ospf

You should see OSPF routes for 192.168.1.0/24 on R3 and 192.168.3.0/24 on R1.

6) Test end-to-end

From PC1, ping 192.168.3.10 (PC2).

From R1, traceroute 192.168.3.10 — path should be R1→R2→R3.

7) (Optional) Cost demo

Force OSPF to prefer a different path by changing interface cost:

R2(config)# interface g0/1
R2(config-if)# ip ospf cost 100


Re-check show ip route or traceroute to see changes.
