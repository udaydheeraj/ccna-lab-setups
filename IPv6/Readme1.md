Topology (place these devices)

R1, R2 (Cisco routers, e.g. 1941)

SW1, SW2 (2960 switches)

PC1, PC2

Use copper straight-through for PC↔SW and SW↔Router; router-to-router use Gig0/1↔Gig0/1.

ASCII:

PC1 --- SW1 --- R1(G0/0) --- R1(G0/1)--- R2(G0/1) --- R2(G0/0) --- SW2 --- PC2

IPv6 plan

LAN1 (PC1): 2001:db8:acad:1::/64

R1 G0/0 = 2001:db8:acad:1::1/64

Link R1↔R2: 2001:db8:acad:12::/64

R1 G0/1 = 2001:db8:acad:12::1/64

R2 G0/1 = 2001:db8:acad:12::2/64

LAN2 (PC2): 2001:db8:acad:2::/64

R2 G0/0 = 2001:db8:acad:2::1/64

Notes: PCs will auto-configure global addresses via SLAAC. Each router also has a link-local address (FE80::/10) on each interface — used by NDP and can be used as next-hop.

STEP 1 — Cable and power on devices

Place devices and connect as topology shows.

Turn on devices (they boot automatically in Packet Tracer).

STEP 2 — Router global prep (on both R1 and R2)

Open CLI on R1, then R2, and run:

enable
configure terminal
no ip domain-lookup
ipv6 unicast-routing        ! enable IPv6 routing globally
exit
write memory

STEP 3 — Configure IPv6 addresses on router interfaces
On R1
enable
configure terminal
hostname R1
interface GigabitEthernet0/0
 description LAN1 - PC1/SW1
 ipv6 address 2001:db8:acad:1::1/64
 no shutdown
!
interface GigabitEthernet0/1
 description Link to R2
 ipv6 address 2001:db8:acad:12::1/64
 no shutdown
end
write memory

On R2
enable
configure terminal
hostname R2
interface GigabitEthernet0/1
 description Link to R1
 ipv6 address 2001:db8:acad:12::2/64
 no shutdown
!
interface GigabitEthernet0/0
 description LAN2 - PC2/SW2
 ipv6 address 2001:db8:acad:2::1/64
 no shutdown
end
write memory


Why this is enough for SLAAC: when an interface has a global prefix and the router sends Router Advertisements (default behavior), hosts on that LAN receive the RA and can autoconfigure.

STEP 4 — Configure switches (optional simple setup)

Switches require no IPv6 addressing for L2 behavior. Just ensure ports are up and PCs are connected to access ports (default is fine). If you want:

On each switch, nothing required; Packet Tracer switches work out of the box.

STEP 5 — Configure PCs to autoconfigure (SLAAC)

On PC1:

Click PC1 → Desktop → IPv6 Configuration (or IP Configuration).

Choose Autoconfigure / Enable IPv6 Auto-configuration (SLAAC) or set to obtain IPv6 automatically.

In Packet Tracer the exact UI wording can vary — the idea is: set IPv6 to auto (do not manually type a global address).

(Alternatively) leave IPv6 fields empty and reboot the PC interface (turn off/on) — it will perform SLAAC.

Repeat for PC2 on LAN2.

STEP 6 — Verify SLAAC & NDP on PCs and routers
Wait a few seconds — Router will send RAs and PCs will generate addresses.

On PC1:

Desktop → Command Prompt → run:

> ipconfig             (or)  > ipv6 address show   (Packet Tracer shows IPs in IPv6 config)
> ping 2001:db8:acad:1::1    (ping R1 link)


You should see a global IPv6 like 2001:db8:acad:1:xxxx:xxxx:xxxx:xxxx (the interface ID generated via Modified EUI-64 or privacy extension).

On R1 (CLI):

show ipv6 interface brief


Expected: interfaces listed with their global and link-local addresses. Example shows GigabitEthernet0/0 2001:db8:acad:1::1/64 FE80::xxxx.

Show the IPv6 neighbor cache (NDP entries):

show ipv6 neighbors


You should see PC1’s link-local and MAC mapping (neighbor entry) for G0/0.
