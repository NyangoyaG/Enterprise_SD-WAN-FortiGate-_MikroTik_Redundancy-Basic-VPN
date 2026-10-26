# Troubleshooting Guide

## 1. FortiGate Has No Working Default Route

Check:

```text
get router info routing-table all
diagnose sys sdwan member
diagnose sys sdwan health-check
show router static
show system sdwan
```

Verify that the SD-WAN members have the correct gateways, the health checks can reach their targets, and the default route uses the intended SD-WAN zone.

## 2. ISP Failover Does Not Occur

Check:

- The affected ISP interface and simulated gateway are reachable.
- The SD-WAN health check detects the failure.
- The SD-WAN rule references the intended members.
- The routing table reflects the expected active path.
- The secondary ISP has a valid upstream route and NAT configuration.

Capture the health-check and route output before and after the test.

## 3. Department PC Cannot Reach Its Gateway

Check:

1. The PC's IP address, subnet mask, and gateway.
2. The access port's VLAN assignment.
3. The trunk configuration between access and core switches.
4. The VLAN interface and VRRP state.
5. The return route and any filtering policies.

Use the appropriate VPCS, MikroTik, or FortiGate diagnostic commands to isolate the failing hop.

## 4. FortiGate Drops Traffic to an Internal Subnet

Check:

```text
get router info routing-table all
diagnose debug flow
```

Review the route to the destination subnet, the return path, firewall policy, and reverse-path check messages.

A previous lab troubleshooting session observed traffic being dropped by reverse-path checking and a missing specific route to a department subnet. Treat this as an outstanding item until current tests confirm it has been resolved.

Do not disable reverse-path checking as a first response. Correct the routing or interface configuration where appropriate.

## 5. VRRP Does Not Fail Over

Check:

```routeros
/interface vrrp print detail
/ip address print
/ip route print detail
```

Confirm the VRRP virtual addresses, instance configuration, priorities, interface state, and master/backup roles on both cores. Verify that both peers can communicate over the relevant network.

## 6. VPN Tunnel Does Not Establish

Check:

```text
show vpn ipsec phase1-interface
show vpn ipsec phase2-interface
diagnose vpn tunnel list
```

Verify peer settings, authentication, phase 1 and phase 2 parameters, selectors, routing, and firewall rules. Never publish pre-shared keys or passwords in diagnostic evidence.

## 7. Safe Troubleshooting Practice

- Change one setting at a time.
- Capture the current configuration before changing it.
- Record the command and its output.
- Test the expected behaviour after each change.
- Remove credentials and sensitive information from evidence.
- Keep unresolved issues documented until verified.

