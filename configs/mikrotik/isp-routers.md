# MikroTik ISP Router Configuration

## Overview

The lab uses two MikroTik RouterOS routers to simulate redundant internet service providers.

- **Primary-ISP1:** Provides the primary WAN gateway.
- **Secondary-ISP2:** Provides the secondary WAN gateway.

Both routers connect to the FortiGate WAN interfaces.

## Addressing

| Router | Interface | Address | Purpose |
|---|---|---|---|
| Primary-ISP1 | ether1 | 203.0.113.1/30 | FortiGate primary WAN gateway |
| Secondary-ISP2 | ether1 | 198.51.100.1/30 | FortiGate secondary WAN gateway |
| Both routers | ether2 | DHCP client | Upstream/NAT connectivity in the lab |

The captured configuration shows DHCP clients on `ether2` and source NAT using masquerade out that interface. Verify the live configuration before relying on this path.

## Verification Commands

Run these commands on each MikroTik router:

```routeros
/system identity print
/interface print
/ip address print
/ip dhcp-client print detail
/ip route print detail
/ip firewall nat print
```

Export the configuration for inspection with:

```routeros
/export
```

Review exports carefully before publishing. Remove secrets and any sensitive information.

## Primary ISP Verification

On Primary-ISP1, verify that:

- `ether1` has `203.0.113.1/30`.
- The connected route for `203.0.113.0/30` exists.
- The upstream route through `ether2` is active.
- The NAT rule matches the intended upstream interface.

## Secondary ISP Verification

On Secondary-ISP2, verify that:

- `ether1` has `198.51.100.1/30`.
- The connected route for `198.51.100.0/30` exists.
- The upstream route through `ether2` is active.
- The NAT rule matches the intended upstream interface.

## Failover Testing

1. Confirm both gateways respond to pings from FortiGate.
2. Confirm both SD-WAN health checks are alive.
3. Start continuous pings from the department PCs.
4. Disconnect the primary ISP path.
5. Confirm FortiGate marks the primary path unavailable and selects the secondary path.
6. Restore the primary link and record the recovery behaviour.

A default route on a MikroTik router does not by itself establish that end-to-end internet access is available. Verify upstream connectivity and return traffic.

## Security Notes

These routers are lab simulations. Do not expose their management services to untrusted networks. Use unique credentials and remove sensitive values from configuration exports before uploading them to a public GitHub repository.


## Screenshot Evidence

### ISP Interfaces and Route Exports

![ISP router interfaces and route exports](../../evidence/captured-2026-10-26/03-isp-router-interface-and-route-exports.png)

Shows the captured ISP interface addressing and route information for the simulated WAN paths. Compare it with the current router configuration before reproducing the setup.
