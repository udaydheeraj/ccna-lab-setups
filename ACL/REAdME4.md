Lab 2 — Permit only Admin to SSH to Server (Named extended ACL + logging)

Objective: Allow only Admin PC to SSH (TCP 22) to the Linux management server; block SSH from other hosts. Log attempts.

Topology
Admin PC (10.0.0.10) --- SW --- R2 --- Server (10.0.2.100)
Guest PC (10.0.0.20) ---/

IP Plan

VLAN/Net: 10.0.0.0/24 (Admin & Guest)

Management Server: 10.0.2.100/24

Router R2 connects both networks.

Steps

Configure interfaces & basic routing so Admin & server can reach each other.

Create named extended ACL with logging

R2(config)# ip access-list extended MGMT-SHIELD
R2(config-ext-nacl)# permit tcp host 10.0.0.10 host 10.0.2.100 eq 22 log
R2(config-ext-nacl)# deny tcp any host 10.0.2.100 eq 22 log
R2(config-ext-nacl)# permit ip any any
R2(config-ext-nacl)# exit


Explanation: first ACE permits Admin -> server TCP22 and logs matches; second ACE denies any other host to server TCP22 and logs; final ACE permits everything else.

Apply ACL — place close to source networks (here the source is 10.0.0.0/24); apply inbound on R2 interface that receives traffic from the LAN:

R2(config)# interface GigabitEthernet0/0   ! interface toward LAN
R2(config-if)# ip access-group MGMT-SHIELD in


Test

From Admin: ssh 10.0.2.100 → should connect.

From Guest: ssh 10.0.2.100 → should be denied.
