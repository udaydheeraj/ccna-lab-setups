Topology
PC1 --- SW1 --- R1 --- (optional other LAN)
Server (DHCP) connected to same switch or to router on same subnet
PC2 ---/

IP Plan

LAN: 10.10.10.0/24

Server = 10.10.10.10 (static)

Gateway R1 G0/0 = 10.10.10.1

DHCP pool: 10.10.10.100 – 10.10.10.200

Reserved IP for printer (or PC1) = 10.10.10.50 via static binding

Devices

Router R1

Switch SW1

Server (Packet Tracer device) — name DHCP-SRV

PC1 (will be static reserved), PC2 (DHCP client)

Step-by-step

Cabling

Place devices and connect: Server Fa0 → SW1 Fa0/3, PC1 Fa0 → SW1 Fa0/1, PC2 Fa0 → SW1 Fa0/2, SW1 Fa0/24 → R1 G0/0.

Configure router interface (gateway)

R1> enable
R1# configure terminal
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 10.10.10.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# exit
R1# write


Configure the Server (Packet Tracer GUI)

Click the Server → Desktop → IP Configuration:

IP Address: 10.10.10.10

Subnet Mask: 255.255.255.0

Default Gateway: 10.10.10.1

Click Services tab → DHCP:

Pool Name: OFFICE_POOL

Default Gateway: 10.10.10.1

DNS Server: 8.8.8.8

Starting IP Address: 10.10.10.100

Subnet Mask: 255.255.255.0

Maximum number of users: 101 (or leave default)

Click Add / On to enable the pool.

Static Reservation (Packet Tracer Server has a Static section in DHCP service):

Go to Static (or Bindings depending on PT version) → Add:

MAC Address: get actual MAC of PC1 (on PC1 Desktop → Config → FastEthernet → MAC address shown)

IP Address: 10.10.10.50

Click Add / Save.

Set PC1 to DHCP (but reservation exists) and PC2 to DHCP

PC1 → Desktop → IP Configuration → DHCP (or Auto). Because a static binding exists for its MAC, server will give 10.10.10.50.

PC2 → Desktop → DHCP → should get 10.10.10.100 (or next free).

Observe Simulation (DORA)

Enable Simulation mode. Filter DHCP.

Trigger renew on PC2 (ipconfig /renew) or power-cycle NIC.

Watch Discover → Offer → Request → Ack. Inspect DHCP Offer payload (server IP, lease, DNS, gateway).

Verify on Server & Router

On Server -> DHCP service UI: you should see Active bindings and the static binding for PC1.

On Router, show ip route to ensure routing if server on different subnet. Not needed here since same subnet.

Verify on PCs

PC1 should show 10.10.10.50 (static reservation).

PC2 should show a pool address in the .100 range.
