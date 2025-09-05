Packet Tracer Lab – RIP vs EIGRP
1. 🔹 Lab Objective

Configure a network of 3 routers with PCs on both ends.

First configure RIP v2 and test connectivity.

Then configure EIGRP and test connectivity.

Compare behavior when a link fails (convergence).

2. 🔹 Topology
 PC1 ---- SW1 ---- R1 ---- R2 ---- R3 ---- SW2 ---- PC2

Devices:

2 PCs (PC1, PC2)

2 Switches (SW1, SW2)

3 Routers (R1, R2, R3)

3. 🔹 IP Addressing Plan
Device	Interface	IP Address	Subnet
PC1	NIC	192.168.1.10	255.255.255.0
PC2	NIC	192.168.3.10	255.255.255.0
R1	G0/0	192.168.1.1	/24
R1	G0/1	10.0.12.1	/30
R2	G0/0	10.0.12.2	/30
R2	G0/1	10.0.23.1	/30
R3	G0/0	10.0.23.2	/30
R3	G0/1	192.168.3.1	/24
4. 🔹 Basic Configurations (All Routers)

Do this on each router:

enable
conf t
no ip domain-lookup
hostname R1   <-- (R2, R3 accordingly)

Set IPs (Example R1)
interface g0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
interface g0/1
 ip address 10.0.12.1 255.255.255.252
 no shutdown


Do similar for R2 and R3 using the IP table above.

PC Configuration

PC1: IP = 192.168.1.10, GW = 192.168.1.1

PC2: IP = 192.168.3.10, GW = 192.168.3.1

✅ At this point, direct neighbors should ping (R1 ↔ R2, R2 ↔ R3).

5. 🔹 RIP v2 Configuration
R1:
router rip
 version 2
 network 192.168.1.0
 network 10.0.12.0
 no auto-summary

R2:
router rip
 version 2
 network 10.0.12.0
 network 10.0.23.0
 no auto-summary

R3:
router rip
 version 2
 network 10.0.23.0
 network 192.168.3.0
 no auto-summary

6. 🔹 Test RIP

On R1: show ip route rip → should see 192.168.3.0/24.

On R3: show ip route rip → should see 192.168.1.0/24.

PC1 ping PC2 → success.

7. 🔹 Break the Link (Failure Test with RIP)

Shut down interface g0/1 on R2:

conf t
interface g0/1
 shutdown


Try to ping PC2 from PC1 again.

❌ RIP takes ~180 seconds to converge → ping fails until timeout passes.

8. 🔹 EIGRP Configuration

Now let’s reconfigure using EIGRP AS 100.

Remove RIP

On each router:

no router rip

R1:
router eigrp 100
 network 192.168.1.0 0.0.0.255
 network 10.0.12.0 0.0.0.3
 no auto-summary

R2:
router eigrp 100
 network 10.0.12.0 0.0.0.3
 network 10.0.23.0 0.0.0.3
 no auto-summary

R3:
router eigrp 100
 network 10.0.23.0 0.0.0.3
 network 192.168.3.0 0.0.0.255
 no auto-summary

9. 🔹 Test EIGRP

On R1: show ip route eigrp → should see 192.168.3.0/24.

On R3: show ip route eigrp → should see 192.168.1.0/24.

PC1 ping PC2 → success.

10. 🔹 Break the Link (Failure Test with EIGRP)

Shut down interface g0/1 on R2:

conf t
interface g0/1
 shutdown


Try to ping PC2 from PC1.

✅ EIGRP reconverges within seconds (DUAL finds alternate path quickly).

11. 🔹 Observations

RIP: Simple, but very slow in failure recovery.

EIGRP: Much faster convergence, smarter metrics.
