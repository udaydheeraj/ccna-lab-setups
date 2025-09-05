Extended Packet Tracer Lab – RIP vs EIGRP with Backup Path
🔹 1. Extended Topology
 PC1 ---- SW1 ---- R1 ---- R2 ---- R3 ---- SW2 ---- PC2
                  |                         |
                  +-----------(Backup)------+

Devices:

2 PCs (PC1, PC2)

2 Switches (SW1, SW2)

3 Routers (R1, R2, R3)

Extra link between R1 ↔ R3

2. New IP Addressing
Device	Interface	IP Address	Subnet
PC1	NIC	192.168.1.10	255.255.255.0
PC2	NIC	192.168.3.10	255.255.255.0
R1	G0/0	192.168.1.1	/24
R1	G0/1	10.0.12.1	/30
R1	G0/2	10.0.13.1	/30 (NEW)
R2	G0/0	10.0.12.2	/30
R2	G0/1	10.0.23.1	/30
R3	G0/0	10.0.23.2	/30
R3	G0/1	192.168.3.1	/24
R3	G0/2	10.0.13.2	/30 (NEW)
3. Basic Configurations (Extra Link)
On R1:
interface g0/2
 ip address 10.0.13.1 255.255.255.252
 no shutdown

On R3:
interface g0/2
 ip address 10.0.13.2 255.255.255.252
 no shutdown


Now we have a triangle topology (R1-R2-R3 + backup R1-R3).

 4. RIP v2 Configuration (with backup path)
R1:
router rip
 version 2
 network 192.168.1.0
 network 10.0.12.0
 network 10.0.13.0
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
 network 10.0.13.0
 network 192.168.3.0
 no auto-summary

 5. EIGRP Configuration (with backup path)
R1:
router eigrp 100
 network 192.168.1.0 0.0.0.255
 network 10.0.12.0 0.0.0.3
 network 10.0.13.0 0.0.0.3
 no auto-summary

R2:
router eigrp 100
 network 10.0.12.0 0.0.0.3
 network 10.0.23.0 0.0.0.3
 no auto-summary

R3:
router eigrp 100
 network 192.168.3.0 0.0.0.255
 network 10.0.23.0 0.0.0.3
 network 10.0.13.0 0.0.0.3
 no auto-summary

 6. Testing the Backup Path
Step A – Verify Normal Path

PC1 pings PC2.

By default, R1 → R2 → R3 → PC2.

Check with:

traceroute 192.168.3.10


On R1 → should go via R2 first.

Step B – Break Primary Link (R2 → R3)

On R2:

interface g0/1
 shutdown

If Using RIP:

RIP takes ~3 minutes (180 sec) to notice the failure.

During this time, pings fail.

After timeout, RIP updates and traffic reroutes via R1 → R3.

If Using EIGRP:

EIGRP detects failure in a few seconds.

DUAL algorithm immediately reroutes traffic via R1 → R3.

Pings drop only briefly (1–2 packets).

7. Key Observations
Feature	RIP	EIGRP
Metric	Hop count	Bandwidth + Delay
Convergence	Slow (up to 3 min)	Very fast (seconds)
Backup Path	Only after timeout	DUAL finds immediately
Suitability	Very small/simple networks	Medium to large enterprise
Summary of Extended Lab

Added a backup link (R1 ↔ R3) to create redundancy.

RIP fails to reroute quickly (pings fail for minutes).

EIGRP reroutes almost instantly using the backup link.

This shows why EIGRP (or OSPF in real-world) is better for enterprise networks.
