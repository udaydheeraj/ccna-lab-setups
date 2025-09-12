Topology
PC1(.10) --- SW --- R1(G0/0) --- ISP(Simulated Server / cloud) at R1 G0/1
PC2(.20) --- SW

IP Plan

LAN: 192.168.10.0/24

PC1 = 192.168.10.10

PC2 = 192.168.10.20

R1 G0/0 = 192.168.10.1

WAN/Internet (simulate with Server):

R1 G0/1 = 203.0.113.2

Internet Server = 203.0.113.10

Steps

Build & IP configure PCs and router interfaces:

PC1/PC2: Desktop → IP Configuration.

On R1:

enable
conf t
interface GigabitEthernet0/0
  ip address 192.168.10.1 255.255.255.0
  no shutdown
interface GigabitEthernet0/1
  ip address 203.0.113.2 255.255.255.0
  no shutdown
ip route 0.0.0.0 0.0.0.0 203.0.113.10    ! static toward Internet server for lab
end
write


Create Standard ACL to deny PC1 and permit everyone else and log matches:

conf t
access-list 10 deny host 192.168.10.10 log
access-list 10 permit any
exit


Apply ACL outbound on interface toward Internet (G0/1):

interface GigabitEthernet0/1
  ip access-group 10 out
end
write


Test:

From PC1 try ping 203.0.113.10 or browse to the Server → should be blocked.

From PC2 ping/browse → should succeed.

Verify / troubleshoot:

show access-lists 10 → shows counters next to each line (hit counts) and the log entry if console logging is enabled.

show ip interface GigabitEthernet0/1 → shows ACL applied outbound.

If nothing logs, ensure console logging or check show logging.

If both PCs blocked: ensure ACL applied out on correct interface (should be interface toward the Internet) and wildcard/host is correct.
