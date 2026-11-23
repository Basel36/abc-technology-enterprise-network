# Addressing plan

Block: `192.168.0.0/16`, allocated largest-first (usable hosts = 2^h − 2). At HQ each VLAN reserves three extra addresses: the HSRP virtual IP and one real address per router.

## HQ

| VLAN | Dept | Hosts | Network | Mask | Virtual GW | Routers (Pri / Sec) |
|---|---|---|---|---|---|---|
| 30 | HR | 300 | 192.168.0.0/23 | 255.255.254.0 | .0.1 | .0.2 / .0.3 |
| 50 | Sales | 100 | 192.168.2.0/25 | 255.255.255.128 | .2.1 | .2.2 / .2.3 |
| 10 | Admin | 50 | 192.168.2.128/26 | 255.255.255.192 | .2.129 | .2.130 / .2.131 |
| 20 | IT | 30 | 192.168.2.192/26 | 255.255.255.192 | .2.193 | .2.194 / .2.195 |
| 40 | Finance | 20 | 192.168.3.0/27 | 255.255.255.224 | .3.1 | .3.2 / .3.3 |
| 60 | R&D | 15 | 192.168.3.32/27 | 255.255.255.224 | .3.33 | .3.34 / .3.35 |
| 99 | Mgmt | 10 | 192.168.3.64/28 | 255.255.255.240 | .3.65 | .3.66 / .3.67 |

## Branches

| Site | VLAN | Group | Hosts | Network | Gateway |
|---|---|---|---|---|---|
| BR-1 | 30 | Dept 3 | 50 | 192.168.16.0/26 | 192.168.16.1 |
| BR-1 | 10 | Dept 1 | 20 | 192.168.16.64/27 | 192.168.16.65 |
| BR-1 | 20 | Dept 2 | 10 | 192.168.16.96/28 | 192.168.16.97 |
| BR-2 | 30 | Dept 3 | 50 | 192.168.18.0/26 | 192.168.18.1 |
| BR-2 | 10 | Dept 1 | 20 | 192.168.18.64/27 | 192.168.18.65 |
| BR-2 | 20 | Dept 2 | 10 | 192.168.18.96/28 | 192.168.18.97 |

## Data Center and Internet edge

| Network | Use |
|---|---|
| 192.168.30.0/27 (VLAN 300) | Servers: DHCP `.10`, AAA `.11`, Web `.12`, gateway `.1` |
| 200.1.1.0/30 | DATACENTER `.1` ↔ ISP `.2` |
| 8.8.8.0/24 | ISP LAN with external PC0 `8.8.8.8`, gateway `8.8.8.1` |
| 50.50.50.2 | Public address of the Web server (static NAT) |

## WAN links (192.168.100.0/24, /30 each)

| Link | Router A (port, IP) | Router B (port, IP) | Subnet |
|---|---|---|---|
| A | HQ-Primary Se0/1/1 .1 | DATACENTER Se0/1/1 .2 | 100.0/30 |
| B | HQ-Primary Se0/1/0 .5 | HQ-SECONDARY Se0/0/0 .6 | 100.4/30 |
| C | HQ-Primary Se0/0/0 .9 | BR-1 Se0/0/1 .10 | 100.8/30 |
| D | HQ-Primary Se0/0/1 .13 | BR-2 Se0/0/1 .14 | 100.12/30 |
| E | HQ-SECONDARY Se0/0/1 .17 | DATACENTER Se0/1/0 .18 | 100.16/30 |
| F | DATACENTER Se0/0/0 .21 | BR-1 Se0/0/0 .22 | 100.20/30 |
| G | DATACENTER Se0/0/1 .25 | BR-2 Se0/0/0 .26 | 100.24/30 |

## DHCP pools (server 192.168.30.10)

| Pool | Gateway | Start IP | Mask | Max users |
|---|---|---|---|---|
| HQ-HR | 192.168.0.1 | 192.168.0.11 | 255.255.254.0 | 300 |
| HQ-Sales | 192.168.2.1 | 192.168.2.11 | 255.255.255.128 | 100 |
| HQ-Admin | 192.168.2.129 | 192.168.2.134 | 255.255.255.192 | 50 |
| HQ-IT | 192.168.2.193 | 192.168.2.198 | 255.255.255.192 | 30 |
| HQ-Finance | 192.168.3.1 | 192.168.3.6 | 255.255.255.224 | 20 |
| HQ-RnD | 192.168.3.33 | 192.168.3.38 | 255.255.255.224 | 15 |
| BR1-Dept3 / Dept1 / Dept2 | .16.1 / .16.65 / .16.97 | .16.5 / .16.70 / .16.100 | /26 /27 /28 | 50 / 20 / 10 |
| BR2-Dept3 / Dept1 / Dept2 | .18.1 / .18.65 / .18.97 | .18.5 / .18.70 / .18.100 | /26 /27 /28 | 50 / 20 / 10 |
