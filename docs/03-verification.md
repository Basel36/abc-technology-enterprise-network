# Verification and failure tests

| Feature | Command | Expected result |
|---|---|---|
| VLANs | `show vlan brief` | 10–60 and 99 on HQ; 10/20/30 on branches; 300 in the Data Center |
| Trunking | `show interfaces trunk` | Uplinks trunking, native VLAN 99 on HQ |
| EtherChannel | `show etherchannel summary` | `Po1 (SU)`, both members `(P)` |
| ROAS | `show ip interface brief` | All sub-interfaces up/up |
| HSRP | `show standby brief` | HQ-Primary Active for 10/30/50/99; HQ-SECONDARY Active for 20/40/60 |
| OSPF | `show ip ospf neighbor` | HQ-Primary and DATACENTER: 4 FULL neighbors; others: 2 |
| Routing | `show ip route` | OSPF routes to every subnet and an `O*E2` default |
| DHCP | `ipconfig` on a client | Address, mask and gateway from the correct pool |
| Passwords | `enable` | Prompts for the enable secret |
| SSH | `ssh -l admin 192.168.2.130`, `show ip ssh` | Encrypted login, SSH version 2.0 |
| NAT | `show ip nat translations` | 50.50.50.2 ↔ 192.168.30.12 |
| Web | Browse `http://50.50.50.2` from PC0 | Page loads from the Internet side |

## Failure and redundancy tests
1. **HSRP** — continuous ping from an HQ PC to the Web server (192.168.30.12); shut Gi0/0 on HQ-Primary. HQ-SECONDARY becomes Active for every VLAN and the ping recovers in a few seconds. `no shutdown` returns VLANs 10/30/50/99 to HQ-Primary (preempt).
2. **EtherChannel** — shut Fa0/3 on Switch0; Po1 stays up on Fa0/4 and the ping continues (`Fa0/3(D) Fa0/4(P)`).
3. **DHCP** — request an address from each HQ department and each branch group; confirm the pool.
4. **WAN** — disconnect one serial link; OSPF re-converges over the remaining links.
5. **NAT/Web** — browse to 50.50.50.2 from PC0.

## Result
All ten project requirements were met and the failover tests passed.
