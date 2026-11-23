# Design notes

## Layer 2
- **VLANs** 10–60 and 99 (Mgmt) on all four HQ switches; 10/20/30 on branch switches; 300 (Servers) in the Data Center.
- **Access ports** use PortFast (`spanning-tree portfast`) so PCs get DHCP without waiting for STP.
- **802.1Q trunks** use native VLAN 99 on both ends of every HQ trunk.
- **LACP EtherChannel (Po1)**: HQ core pair on Fa0/3–4, Data Center pair on Fa0/4–5, `channel-group 1 mode active`.
- **STP roots match HSRP roles** so each VLAN's L2 path leads to the router that is Active for it:
  - Switch0: root primary for VLANs 10, 30, 50, 99; secondary for 20, 40, 60
  - Switch1: root primary for VLANs 20, 40, 60; secondary for 10, 30, 50, 99

## Layer 3
- **Router-on-a-stick**: one 802.1Q sub-interface per VLAN on each router's physical port.
- **HSRP load sharing**: HQ-Primary Active (priority 110) for VLANs 10/30/50/99, HQ-SECONDARY Active for 20/40/60. Preempt enabled so recovered routers reclaim their VLANs.
- **OSPF process 1, Area 0**: one `network 192.168.0.0 0.0.255.255 area 0` statement, `passive-interface default`, only serial links active. Router IDs 1.1.1.1 (HQ-Primary), 2.2.2.2 (HQ-SECONDARY), 3.3.3.3 (BR-1), 4.4.4.4 (BR-2), 5.5.5.5 (DATACENTER).
- **Default route**: DATACENTER has a static default to the ISP (200.1.1.2) and advertises it with `default-information originate`, appearing as `O*E2` elsewhere.

## Services and security
- **DHCP**: standalone server in the Data Center; `ip helper-address 192.168.30.10` on every client-facing gateway forwards Discovers as unicast.
- **AAA**: `aaa new-model`; console/aux use line passwords; VTY uses RADIUS first with local fallback. Note: if RADIUS answers with a reject, the router does not fall back.
- **SSH**: v2, RSA 1024, `transport input ssh` on VTY lines.
- **Static NAT**: `192.168.30.12` ↔ `50.50.50.2`, `ip nat inside` on Gi0/0.300 and `ip nat outside` on Gi0/2; ISP routes `50.50.50.0/24` back to DATACENTER.

## Possible improvements
- RSA keys of 2048 bits or more, and `login block-for` on VTY.
- OSPF authentication on serial links.
- Replace the HQ ROAS pair with L3 switches for inter-VLAN routing at scale.
- Add NAT overload (PAT) for outbound internet access.
