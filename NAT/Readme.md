Lab Goal

Configure Static NAT on a router so that an inside PC (private IP) can communicate with an outside server (public IP) using a mapped public address.

Verify with ping and web access.

🔹 Topology
        INSIDE LAN (Private)                OUTSIDE (Public Internet)
 ┌─────────┐       ┌──────────┐
 │ PC0     │-------│  R1      │-------[ Server0 ]
 │ 192.168.1.10    │          │        203.0.0.100
 │ GW:192.168.1.1  │          │
 └─────────┘       └──────────┘
                   Fa0/0   Fa0/1
                 192.168.1.1  203.0.0.1

🔹 Step-by-Step Configuration
Step 1 — Place Devices

1 Router (R1)

1 PC (PC0)

1 Server (Server0)

Step 2 — IP Addressing
On PC0:

IP Address: 192.168.1.10

Subnet Mask: 255.255.255.0

Default Gateway: 192.168.1.1

On Server0:

IP Address: 203.0.0.100

Subnet Mask: 255.255.255.0

Gateway: 203.0.0.1

On Router R1:
R1> enable
R1# configure terminal

! Inside interface
R1(config)# interface fa0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

! Outside interface
R1(config)# interface fa0/1
R1(config-if)# ip address 203.0.0.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

Step 3 — Define Inside & Outside NAT Interfaces
R1(config)# interface fa0/0
R1(config-if)# ip nat inside
R1(config-if)# exit

R1(config)# interface fa0/1
R1(config-if)# ip nat outside
R1(config-if)# exit

Step 4 — Configure Static NAT Mapping

We want PC0 (192.168.1.10) to be mapped to public IP 203.0.0.10.

R1(config)# ip nat inside source static 192.168.1.10 203.0.0.10

Step 5 — Configure Routing

So both networks know how to reach each other:

On Router R1:

Already has both interfaces, no extra routes needed.

On Server0:

Set gateway = 203.0.0.1 (already done).

Step 6 — Verification

From PC0 → Ping Server0:

PC0> ping 203.0.0.100


✅ Should work because NAT translates 192.168.1.10 → 203.0.0.10.

From Server0 → Ping PC0’s public mapped IP:

Server0> ping 203.0.0.10


✅ Should reply because Router R1 translates back to 192.168.1.10.

Check NAT table on R1:

R1# show ip nat translations
Pro  Inside global      Inside local       Outside local      Outside global
icmp 203.0.0.10:1       192.168.1.10:1     203.0.0.100:1      203.0.0.100:1

🔹 Real-World Tie-In

Static NAT is often used for servers inside a private network (like a web server or mail server) that must be accessible via a fixed public IP.

Example: Internal web server at 10.0.0.5 mapped to 203.0.113.50.
