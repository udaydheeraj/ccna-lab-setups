Step-by-Step Configuration
Step 1 — Assign IPs

PCs:

PC	IP Address	Subnet Mask	Gateway	DNS
PC1	192.168.1.10	255.255.255.0	192.168.1.1	203.0.0.8
PC2	192.168.1.20	255.255.255.0	192.168.1.1	203.0.0.8
PC3	192.168.1.30	255.255.255.0	192.168.1.1	203.0.0.8
PC4	192.168.1.40	255.255.255.0	192.168.1.1	203.0.0.8
PC5	192.168.1.50	255.255.255.0	192.168.1.1	203.0.0.8

Router R1:

R1> enable
R1# configure terminal

! Inside (LAN)
R1(config)# interface fa0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# ip nat inside
R1(config-if)# exit

! Outside (WAN)
R1(config)# interface fa0/1
R1(config-if)# ip address 203.0.0.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# ip nat outside
R1(config-if)# exit


DNS Server:

IP: 203.0.0.8 /24

Gateway: 203.0.0.1

Add record: www.example.com = 203.0.0.200

Web Server:

IP: 203.0.0.200 /24

Gateway: 203.0.0.1

Enable HTTP service

Step 2 — Configure PAT (Port Address Translation)

Define ACL to match inside traffic:

R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255


Configure PAT using outside interface:

R1(config)# ip nat inside source list 1 interface fa0/1 overload


⚠️ overload enables PAT, allowing multiple PCs to share the single public IP 203.0.0.1.

Step 3 — Verification

Ping DNS from PC1

PC1> ping 203.0.0.8


✅ Should reply.

Ping Web Server directly from PC2

PC2> ping 203.0.0.200


✅ Should reply.

Ping by Name from PC3

PC3> ping www.example.com


✅ DNS resolves & ping replies.

Test Web from PC4 or PC5

Open Web Browser → type http://www.example.com
✅ Should load page.

Check NAT table on Router

R1# show ip nat translations
Pro  Inside global   Inside local    Outside local    Outside global
icmp 203.0.0.1:1025  192.168.1.10:1  203.0.0.200:1    203.0.0.200:1
icmp 203.0.0.1:1026  192.168.1.20:1  203.0.0.200:1    203.0.0.200:1
tcp  203.0.0.1:8080  192.168.1.40:80 203.0.0.200:80   203.0.0.200:80


✅ Notice: All inside PCs are using one global IP (203.0.0.1), but with different port numbers → this is PAT in action.
