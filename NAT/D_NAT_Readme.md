Topology (Corrected for Packet Tracer)
   Inside LAN (Private)                   Router R1 (NAT)             Outside (Public Internet)
 ┌─────────┐   ┌─────────┐
 │ PC1     │   │ PC2     │     ┌───────────┐       ┌─────────────┐       ┌────────────┐
 │ 192.168.1.10 │ 192.168.1.20 │---│ Fa0/0    R1 │ Fa0/1 203.0.0.1 │---│ DNS Server  │ 203.0.0.8
 │ GW:1.1   │   │ GW:1.1   │     │ 192.168.1.1│       │             │       │ Web Server  │ 203.0.0.200
 └─────────┘   └─────────┘     └───────────┘       └─────────────┘       └────────────┘
            │ PC3: 192.168.1.30


⚠️ Note: DNS server is now 203.0.0.8 (same subnet as R1’s Fa0/1). This fixes the earlier issue.

🔹 Step-by-Step Configuration
Step 1 — Assign IPs

PCs:

PC1: 192.168.1.10 /24, GW: 192.168.1.1, DNS: 203.0.0.8

PC2: 192.168.1.20 /24, GW: 192.168.1.1, DNS: 203.0.0.8

PC3: 192.168.1.30 /24, GW: 192.168.1.1, DNS: 203.0.0.8

R1 Interfaces:

R1> enable
R1# configure terminal

! Inside LAN
R1(config)# interface fa0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# ip nat inside
R1(config-if)# exit

! Outside WAN
R1(config)# interface fa0/1
R1(config-if)# ip address 203.0.0.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# ip nat outside
R1(config-if)# exit


DNS Server:

IP: 203.0.0.8 /24

GW: 203.0.0.1

Add A-record → www.example.com = 203.0.0.200

Web Server:

IP: 203.0.0.200 /24

GW: 203.0.0.1

Enable HTTP service

Step 2 — Configure Dynamic NAT

Create a pool of public IPs:
(e.g., 203.0.0.10 – 203.0.0.20)

R1(config)# ip nat pool MYPOOL 203.0.0.10 203.0.0.20 netmask 255.255.255.0


Match inside private IPs with an ACL:

R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255


Bind NAT to pool:

R1(config)# ip nat inside source list 1 pool MYPOOL

Step 3 — Verification

From PC1 → ping Web Server:

PC1> ping 203.0.0.200


✅ Should work.

From PC2 → resolve domain:

PC2> ping www.example.com


✅ DNS resolves (to 203.0.0.200) and ping succeeds.

From PC3 → browse:

Open browser → http://www.example.com
✅ Loads web page.

On R1 → Check NAT translations:

R1# show ip nat translations
Pro Inside global     Inside local     Outside local     Outside global
icmp 203.0.0.10       192.168.1.10     203.0.0.200       203.0.0.200
icmp 203.0.0.11       192.168.1.20     203.0.0.200       203.0.0.200
icmp 203.0.0.12       192.168.1.30     203.0.0.200       203.0.0.200
