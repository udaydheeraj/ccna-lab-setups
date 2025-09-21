Allow PC1, PC2, PC3 in the LAN to access the Internet using Static NAT.

Use Router R1 as the NAT device.

Include a DNS server (8.8.8.8) and a Web Server (203.0.0.200) to simulate real Internet services.

🔹 Topology
   Inside LAN (Private)                   Router R1 (NAT)               Outside (Public Internet)
 ┌─────────┐   ┌─────────┐
 │ PC1     │   │ PC2     │     ┌───────────┐       ┌─────────────┐       ┌────────────┐
 │ 192.168.1.10 │ 192.168.1.20 │---│ Fa0/0    R1 │ Fa0/1 203.0.0.1 │---│ DNS Server  │ 8.8.8.8
 │ GW:1.1   │   │ GW:1.1   │     │ 192.168.1.1│       │             │       │ Web Server  │ 203.0.0.200
 └─────────┘   └─────────┘     └───────────┘       └─────────────┘       └────────────┘
            │ PC3: 192.168.1.30

🔹 Step-by-Step Configuration
Step 1 — Place Devices

1 Router (R1)

3 PCs (PC1, PC2, PC3)

1 DNS Server (8.8.8.8)

1 Web Server (203.0.0.200)

1 Switch (to connect PCs to Router)

Step 2 — Assign IP Addresses
PCs:

PC1: 192.168.1.10 /24, Gateway = 192.168.1.1, DNS = 8.8.8.8

PC2: 192.168.1.20 /24, Gateway = 192.168.1.1, DNS = 8.8.8.8

PC3: 192.168.1.30 /24, Gateway = 192.168.1.1, DNS = 8.8.8.8

Router R1:
R1> enable
R1# configure terminal

! Inside interface (LAN)
R1(config)# interface fa0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# ip nat inside
R1(config-if)# exit

! Outside interface (toward Internet)
R1(config)# interface fa0/1
R1(config-if)# ip address 203.0.0.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# ip nat outside
R1(config-if)# exit

DNS Server:

IP: 8.8.8.8 /24

Gateway: 203.0.0.1

Web Server:

IP: 203.0.0.200 /24

Gateway: 203.0.0.1

Enable HTTP service.

Step 3 — Configure Static NAT on R1

We will map private IPs of PCs to public IPs:

PC1 → 203.0.0.10

PC2 → 203.0.0.20

PC3 → 203.0.0.30

! Static NAT mappings
R1(config)# ip nat inside source static 192.168.1.10 203.0.0.10
R1(config)# ip nat inside source static 192.168.1.20 203.0.0.20
R1(config)# ip nat inside source static 192.168.1.30 203.0.0.30

Step 4 — Configure DNS for Web Resolution

On DNS Server (8.8.8.8):

Add A Record:

www.example.com → 203.0.0.200

Step 5 — Verification

Check NAT Table on R1:

R1# show ip nat translations
Pro Inside global   Inside local     Outside local     Outside global
--- 203.0.0.10      192.168.1.10     ---               ---
--- 203.0.0.20      192.168.1.20     ---               ---
--- 203.0.0.30      192.168.1.30     ---               ---


From PC1:

PC1> ping 8.8.8.8


✅ DNS server reachable.

From PC2:

PC2> ping www.example.com


✅ DNS resolves → should reply from 203.0.0.200.

From PC3:

Open web browser → http://www.example.com
✅ Loads page from Web Server.
