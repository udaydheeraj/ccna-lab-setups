Objective: Block HTTP (TCP port 80) from Sales subnet to the Web Server. Allow everything else.

Topology (simple)
Sales (PC1) 192.168.10.10 --- SW --- R1 Fa0/0 --- Fa0/1 --- Server 172.16.1.100
Other (PC2) 192.168.20.10 ---/

IP Plan

Sales: 192.168.10.0/24 (PC1 = .10)

Other: 192.168.20.0/24 (PC2 = .10)

Router R1:

Fa0/0 (LAN) = 192.168.10.1 (if single LAN, we assume R1 routes both VLANs via switch; for simplicity use two routers or subnets)

Fa0/1 (Server) = 172.16.1.1

Server = 172.16.1.100 (HTTP running on Server web service)

For Packet Tracer you can place two PCs and one Server on a switch with router connected via two interfaces or use separate switches—keep cabling consistent.

Steps

Build and IP configure devices

PC1: IP 192.168.10.10/24, GW 192.168.10.1

PC2: IP 192.168.20.10/24, GW 192.168.20.1

Server: IP 172.16.1.100/24, GW 172.16.1.1

R1:

interface GigabitEthernet0/0
  ip address 192.168.10.1 255.255.255.0
  no shutdown
interface GigabitEthernet0/1
  ip address 172.16.1.1 255.255.255.0
  no shutdown


(If PC2 is on different subnet, R1 needs a subinterface or second interface. For simplicity you can use a single LAN and different hosts—adjust accordingly.)

Create extended ACL (numbered 101) — deny Sales → WebServer on TCP 80, permit everything else:

R1(config)# access-list 101 deny tcp 192.168.10.0 0.0.0.255 host 172.16.1.100 eq 80
R1(config)# access-list 101 permit ip any any


Explanation: first line denies TCP from Sales subnet to host 172.16.1.100 port 80; second allows everything else.

Apply ACL — close to the source (apply inbound on interface connecting Sales network). If Sales and Other share same interface, apply on that interface in:

R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip access-group 101 in


Test

From PC1 (Sales): open Web Browser → http://172.16.1.100 → should be blocked.

From PC2: same request → should succeed.

Verify

R1# show access-lists 101
Extended IP access list 101
    deny tcp 192.168.10.0 0.0.0.255 host 172.16.1.100 eq www (hitcount=5)
    permit ip any any (hitcount=20)
R1# show ip interface GigabitEthernet0/0
  (look for "ip access-group 101 in")


Troubleshoot

If both blocked: check the source wildcard mask and interface where applied.

If none blocked: ensure ACL applied on correct interface/direction.

Use debug ip packet detail carefully in lab to see matches (can be noisy).
