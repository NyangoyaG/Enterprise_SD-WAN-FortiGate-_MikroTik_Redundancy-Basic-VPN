# Core Switches, VLANs and VRRP

## Overview

Core-SW1 and Core-SW2 provide Layer 3 connectivity for the department VLANs and use VRRP to provide redundant default gateways.

The documented design uses Core-SW1 as the preferred master and Core-SW2 as the backup.

## Transit Network

| Device | Transit Address |
|---|---|
| FortiGate port3 | `10.10.10.1/24` |
| Core-SW1 | `10.10.10.2/24` |
| Core-SW2 | `10.10.10.3/24` |

The core switches use FortiGate as their upstream next hop where configured. Confirm the live routing table before applying or modifying routes.

## Department VLANs

| VLAN | Department | Subnet | Virtual Gateway |
|---|---|---|---|
| 10 | Admin | `10.10.11.0/24` | `10.10.11.254` |
| 20 | HR | `10.10.20.0/24` | `10.10.20.254` |
| 30 | IT | `10.10.30.0/24` | `10.10.30.254` |
| 99 | Management | `10.10.99.0/24` | Verify configuration |

The VLAN 99 management network is a documented design item; do not assume it is implemented until confirmed in the current configuration.

## VRRP Design

The documented VRRP priority is higher on Core-SW1 than Core-SW2:

- Core-SW1: priority 150.
- Core-SW2: priority 100.

The intended state is Core-SW1 as master and Core-SW2 as backup for the department VLAN gateways.

## Verification Commands

Run these commands on each core router:

```routeros
/system identity print
/interface print
/interface bridge print
/interface bridge port print
/interface bridge vlan print
/interface vlan print
/ip address print
/ip route print detail
/interface vrrp print detail
```

Confirm that:

- VLAN IDs and subnets match the addressing plan.
- Trunk links carry the required tagged VLANs.
- Access ports are assigned to the intended VLANs.
- VRRP virtual addresses match the department gateways.
- The active master and backup states are correct.

## VRRP Failover Test

1. Verify that Core-SW1 is master and Core-SW2 is backup.
2. Start continuous pings from an Admin, HR, or IT PC to its virtual gateway.
3. Disable the active core path in the lab.
4. Confirm that Core-SW2 becomes master.
5. Confirm that the client can still reach its gateway and permitted destinations.
6. Restore the original path and record the resulting VRRP state.

Do not describe redundancy as fully validated until this controlled test succeeds.

## Routing Considerations

A working VLAN interface does not guarantee that traffic can reach other VLANs or the internet. Verify connected routes, default routes, return routes, firewall policies, and any routing filters.

In this lab, internal routing and reverse-path behaviour have previously required troubleshooting. Record any unresolved routing issue instead of masking it by disabling security controls.

## Security Considerations

Restrict management access to trusted management hosts. Apply explicit inter-VLAN policies based on business requirements and avoid exposing router management services to user VLANs unnecessarily.


## Screenshot Evidence

### VLAN and Bridge Configuration

![Core switch VLAN and bridge configuration](../../evidence/captured-2026-10-26/04-core-switch-vlan-and-bridge-configuration.png)

Shows the captured VLAN and bridge configuration used to separate departmental traffic.

### VRRP Master and Backup Roles

![Core switch VRRP master and backup](../../evidence/captured-2026-10-26/05-core-switch-vrrp-master-backup.png)

Records the VRRP roles at capture time. A separate interruption test is needed to demonstrate successful gateway failover.

### VRRP and Address Details

![Core switch VRRP and address details](../../evidence/captured-2026-10-26/06-core-switch-vrrp-and-address-details.png)

Shows the captured VRRP and IP address settings for the redundant gateways.

### WinBox Core Switch Exports

![WinBox core switch exports](../../evidence/captured-2026-10-26/13-winbox-core-switch-exports.png)

Provides additional WinBox evidence of the core-switch configuration.
