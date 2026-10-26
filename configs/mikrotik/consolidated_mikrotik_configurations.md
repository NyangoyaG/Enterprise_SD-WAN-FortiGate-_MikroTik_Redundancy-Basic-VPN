# Enterprise SD-WAN Lab --- Consolidated MikroTik Configurations

**Reference date:** 26 October 2026\
**RouterOS version shown:** 7.22.1

> This is one documentation file for all seven MikroTik devices. It is
> not a single script to import. Each section belongs to a different
> router. The text below transcribes the configuration visible in the
> supplied `/export hide-sensitive` screenshots; lines outside the
> screenshot crop may be missing. Verify against the live router before
> making changes.

## Addressing overview

  -----------------------------------------------------------------------
  Device                              Address/settings shown
  ----------------------------------- -----------------------------------
  Primary-ISP1                        `203.0.113.1/30` on `ether1`;
                                      masquerade out `ether2`

  Secondary-ISP2                      `198.51.100.1/30` on `ether1`;
                                      masquerade out `ether2`

  Core-SW1                            `10.10.10.2/24`, `10.255.255.2/24`;
                                      VRRP priority 150

  Core-SW2                            `10.10.10.3/24`, `10.255.255.3/24`;
                                      department VRRP priority not
                                      explicitly set

  Access-SW1                          `ether2` PVID 10

  Access-SW2                          `10.10.20.50/24` on `bridge-lan`

  Access-SW3                          No static IP shown
  -----------------------------------------------------------------------

Department VLANs on the cores: VLAN 10 Admin (`10.10.11.0/24`), VLAN 20
HR (`10.10.20.0/24`), VLAN 30 IT (`10.10.30.0/24`). The virtual gateway
addresses shown are `10.10.11.254`, `10.10.20.254`, and `10.10.30.254`.
VLAN 99 is named `VLAN-MGMT`; no VLAN 99 IP address is visible.

------------------------------------------------------------------------

## 1. Primary-ISP1

### Exported configuration visible

``` routeros
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
set [ find default-name=ether6 ] disable-running-check=no
set [ find default-name=ether7 ] disable-running-check=no
set [ find default-name=ether8 ] disable-running-check=no
/ip address
add address=203.0.113.1/30 interface=ether1 network=203.0.113.0
/ip dhcp-client
add interface=ether1 name=client1
add interface=ether2 name=client2
/ip firewall nat
add action=masquerade chain=srcnat out-interface=ether2
/system identity
set name=Primary-ISP1
```

### Explanation

-   `/system identity` gives the router its recognizable WinBox/terminal
    name.
-   `203.0.113.1/30` assigns the primary ISP-side address to `ether1`;
    the network is `203.0.113.0/30`.
-   `disable-running-check=no` leaves the normal Ethernet link-running
    check enabled.
-   DHCP clients named `client1` and `client2` are configured on
    `ether1` and `ether2`. Their operational status is not shown.
-   The masquerade rule translates source addresses for traffic leaving
    `ether2`. Confirm `ether2` is the intended egress interface in the
    lab.
-   No static route is visible in the supplied screenshot; inspect the
    complete live export and route table before concluding none exists.

------------------------------------------------------------------------

## 2. Secondary-ISP2

### Exported configuration visible

``` routeros
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
set [ find default-name=ether6 ] disable-running-check=no
set [ find default-name=ether7 ] disable-running-check=no
set [ find default-name=ether8 ] disable-running-check=no
/ip address
add address=198.51.100.1/30 interface=ether1 network=198.51.100.0
/ip dhcp-client
add interface=ether1 name=client1
add interface=ether2 name=client2
/ip firewall nat
add action=masquerade chain=srcnat out-interface=ether2
/system identity
set name=Secondary-ISP2
```

### Explanation

-   `198.51.100.1/30` assigns the secondary ISP-side address to
    `ether1`; the network is `198.51.100.0/30`.
-   DHCP clients are configured on `ether1` and `ether2`; their status
    is not visible.
-   Masquerade translates traffic leaving `ether2`; verify that
    interface's role before modifying it.
-   No static route is visible in the screenshot; verify with
    `/ip route print detail`.

------------------------------------------------------------------------

## 3. Core-SW1

### Exported configuration visible

``` routeros
/interface bridge
add name=bridge-lan priority=0x1000 vlan-filtering=yes
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
set [ find default-name=ether6 ] disable-running-check=no
set [ find default-name=ether7 ] disable-running-check=no
set [ find default-name=ether8 ] disable-running-check=no
/interface vrrp
add interface=bridge-lan name=VRRP-LAN priority=150
/interface vlan
add interface=bridge-lan name=VLAN-ADMIN vlan-id=10
add interface=bridge-lan name=VLAN-HR vlan-id=20
add interface=bridge-lan name=VLAN-IT vlan-id=30
add interface=bridge-lan name=VLAN-MGMT vlan-id=99
/interface vrrp
add interface=VLAN-ADMIN name=VRRP-ADMIN priority=150 vrid=10
add interface=VLAN-HR name=VRRP-HR priority=150 vrid=20
add interface=VLAN-IT name=VRRP-IT priority=150 vrid=30
/interface bridge port
add bridge=bridge-lan interface=ether1
add bridge=bridge-lan interface=ether2
add bridge=bridge-lan interface=ether3 pvid=10
add bridge=bridge-lan interface=ether5 pvid=20
add bridge=bridge-lan interface=ether4
/interface bridge vlan
add bridge=bridge-lan tagged=bridge-lan,ether2 untagged=ether3 vlan-ids=10
add bridge=bridge-lan tagged=bridge-lan,ether2 untagged=ether5 vlan-ids=20
add bridge=bridge-lan tagged=bridge-lan,ether2 vlan-ids=30
add bridge=bridge-lan tagged=bridge-lan,ether2 vlan-ids=99
add bridge=bridge-lan untagged=bridge-lan,ether1,ether4 vlan-ids=1
/ip address
add address=10.10.10.2/24 interface=bridge-lan network=10.10.10.0
add address=10.10.10.254/24 interface=VRRP-LAN network=10.10.10.0
add address=10.10.11.1/24 interface=VLAN-ADMIN network=10.10.11.0
add address=10.10.20.1/24 interface=VLAN-HR network=10.10.20.0
add address=10.10.30.1/24 interface=VLAN-IT network=10.10.30.0
add address=10.10.11.254/24 interface=VRRP-ADMIN network=10.10.11.0
add address=10.10.20.254/24 interface=VRRP-HR network=10.10.20.0
add address=10.10.30.254/24 interface=VRRP-IT network=10.10.30.0
add address=10.255.255.2/24 interface=bridge-lan network=10.255.255.0
/ip dhcp-client
add interface=ether1 name=client1
/ip firewall filter
add action=accept chain=forward comment="Allow Admin to FortiGate transit" dst-address=10.10.10.0/24 src-address=10.10.11.0/24
/ip firewall nat
add action=masquerade chain=srcnat comment="Depts to FortiGate" dst-address=10.10.0.0/16 src-address=10.10.0.0/16
/ip route
add dst-address=0.0.0.0/0 gateway=10.10.10.1
/system identity
set name=Core-SW1
/tool sniffer
set filter-interface=ether5
```

### Explanation

-   `bridge-lan` is the Layer 2 bridge. `vlan-filtering=yes` makes the
    bridge VLAN table and PVIDs affect VLAN forwarding.
-   Bridge priority `0x1000` is for spanning-tree bridge election; it is
    not VRRP priority.
-   VLAN interfaces define VLAN IDs 10, 20, 30, and 99. The department
    VRRP instances use VRIDs 10, 20, and 30 with priority 150.
-   `ether3` is configured as untagged/access VLAN 10 and `ether5` as
    untagged/access VLAN 20. `ether2` is tagged for VLANs 10, 20, 30,
    and 99 in the visible table.
-   VLAN 1 is shown untagged on `bridge-lan`, `ether1`, and `ether4`.
-   `10.10.10.2/24` is the core's transit address; `10.10.10.1` is the
    configured default gateway. The `10.10.10.254` address belongs to
    `VRRP-LAN`.
-   `.1` addresses on VLAN interfaces are the switch's own addresses;
    `.254` addresses on the VRRP interfaces are intended virtual gateway
    addresses.
-   `10.255.255.2/24` is also configured on the bridge; confirm its
    intended purpose.
-   DHCP client on `ether1` is configured, but the screenshot includes a
    warning that a DHCP client cannot run on a slave/passthrough
    interface. Verify its state.
-   The filter rule allows forwarded traffic from Admin subnet
    `10.10.11.0/24` to transit subnet `10.10.10.0/24`; it is not a
    complete firewall policy.
-   The NAT rule matches both source and destination within
    `10.10.0.0/16`. This is unusual for typical internet NAT, so
    document it as existing and review before changing.
-   The default route points to FortiGate transit IP `10.10.10.1`;
    sniffer filter is set to `ether5`.

------------------------------------------------------------------------

## 4. Core-SW2

### Exported configuration visible

``` routeros
/interface bridge
add name=bridge-lan priority=0x7D0 vlan-filtering=yes
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
set [ find default-name=ether6 ] disable-running-check=no
set [ find default-name=ether7 ] disable-running-check=no
set [ find default-name=ether8 ] disable-running-check=no
/interface vrrp
add interface=bridge-lan name=VRRP-LAN
/interface vlan
add interface=bridge-lan name=VLAN-ADMIN vlan-id=10
add interface=bridge-lan name=VLAN-HR vlan-id=20
add interface=bridge-lan name=VLAN-IT vlan-id=30
add interface=bridge-lan name=VLAN-MGMT vlan-id=99
/interface vrrp
add interface=VLAN-ADMIN name=VRRP-ADMIN vrid=10
add interface=VLAN-HR name=VRRP-HR vrid=20
add interface=VLAN-IT name=VRRP-IT vrid=30
/interface bridge port
add bridge=bridge-lan interface=ether1
add bridge=bridge-lan interface=ether2
add bridge=bridge-lan interface=ether3 pvid=30
/interface bridge vlan
add bridge=bridge-lan tagged=bridge-lan,ether2 vlan-ids=10
# Screenshot notes: ether5 is not a bridge port
add bridge=bridge-lan tagged=bridge-lan,ether2 untagged=ether5 vlan-ids=20
add bridge=bridge-lan tagged=bridge-lan,ether2 untagged=ether3 vlan-ids=30
add bridge=bridge-lan tagged=bridge-lan,ether2 vlan-ids=99
add bridge=bridge-lan untagged=bridge-lan,ether1,ether2 vlan-ids=1
/ip address
add address=10.10.10.3/24 interface=bridge-lan network=10.10.10.0
add address=10.255.255.3/24 interface=bridge-lan network=10.255.255.0
add address=10.10.10.254/24 interface=VRRP-LAN network=10.10.10.0
add address=10.10.11.2/24 interface=VLAN-ADMIN network=10.10.11.0
add address=10.10.20.2/24 interface=VLAN-HR network=10.10.20.0
add address=10.10.30.2/24 interface=VLAN-IT network=10.10.30.0
add address=10.10.11.254/24 interface=VRRP-ADMIN network=10.10.11.0
add address=10.10.20.254/24 interface=VRRP-HR network=10.10.20.0
add address=10.10.30.254/24 interface=VRRP-IT network=10.10.30.0
/ip dhcp-client
add interface=ether1 name=client1
/ip route
add dst-address=0.0.0.0/0 gateway=10.10.10.1
add dst-address=0.0.0.0/0 gateway=10.10.10.1
/system identity
set name=Core-SW2
```

### Explanation

-   Bridge `bridge-lan` has VLAN filtering enabled and bridge priority
    `0x7D0`, which is a spanning-tree setting, not VRRP priority.
-   VLAN IDs 10, 20, 30, and 99 are configured. `ether2` is tagged for
    these VLANs in the visible table.
-   `ether3` has PVID 30, making it the intended untagged/access port
    for VLAN 30.
-   The screenshot says `ether5 not a bridge port`, but the VLAN 20 row
    lists `ether5` as untagged. Verify with live
    `/interface bridge port print detail` and
    `/interface bridge vlan print detail`; this appears inconsistent.
-   VRRP IDs 10, 20, and 30 match the department VLANs. The screenshot
    does not explicitly set VRRP priority on these instances, so do not
    claim a custom priority from this export.
-   The `.2` addresses are the switch's own VLAN addresses. `.254`
    addresses are configured on VRRP interfaces as shared gateway
    addresses. Confirm VRRP states and peer compatibility.
-   `10.10.10.3/24` is the transit address; `10.10.10.1` is the
    configured default gateway.
-   `10.255.255.3/24` is also configured on the bridge; confirm its
    purpose.
-   A DHCP client is configured on `ether1`; the export warns this may
    not work on a slave/passthrough interface.
-   Two identical default-route entries are visible. Check
    `/ip route print detail` to see whether both are active and whether
    distances/status differ before considering cleanup.
-   No firewall or NAT section is visible in the supplied Core-SW2
    screenshot.

------------------------------------------------------------------------

## 5. Access-SW1

### Exported configuration visible

``` routeros
/interface bridge
add name=bridge-lan
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
set [ find default-name=ether6 ] disable-running-check=no
set [ find default-name=ether7 ] disable-running-check=no
set [ find default-name=ether8 ] disable-running-check=no
/interface bridge port
add bridge=bridge-lan interface=ether1
add bridge=bridge-lan interface=ether2 pvid=10
/ip dhcp-client
add interface=ether1 name=client1
/system identity
set name=Access-SW1
```

### Explanation

-   `bridge-lan` joins `ether1` and `ether2` at Layer 2.
-   `ether2` is assigned PVID 10. This classifies untagged ingress
    frames into VLAN 10 only if the bridge's VLAN design supports it;
    this export does not show VLAN filtering enabled here.
-   DHCP client `client1` is configured on `ether1`; status is not
    shown.
-   No static management IP is visible in the screenshot.

------------------------------------------------------------------------

## 6. Access-SW2

### Exported configuration visible

``` routeros
/interface bridge
add name=bridge-lan protocol-mode=none
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
set [ find default-name=ether6 ] disable-running-check=no
set [ find default-name=ether7 ] disable-running-check=no
set [ find default-name=ether8 ] disable-running-check=no
/interface bridge port
add bridge=bridge-lan interface=ether1
add bridge=bridge-lan interface=ether2
/ip address
add address=10.10.20.50/24 interface=bridge-lan network=10.10.20.0
/ip dhcp-client
add interface=ether1 name=client1
/system identity
set name=Access-SW2
/tool sniffer
set filter-interface=ether1 filter-mac-protocol=arp
```

### Explanation

-   `protocol-mode=none` disables spanning-tree protection on the
    bridge. This is the actual exported setting; ensure the topology
    cannot form Layer 2 loops.
-   `ether1` and `ether2` are bridge ports.
-   `10.10.20.50/24` is the management address shown in the HR subnet.
-   A DHCP client is also configured on `ether1`; verify whether it is
    needed and active alongside the static management address.
-   Packet sniffer filtering is set to ARP on `ether1`.

------------------------------------------------------------------------

## 7. Access-SW3

### Exported configuration visible

``` routeros
/interface bridge
add name=bridge-lan
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
set [ find default-name=ether6 ] disable-running-check=no
set [ find default-name=ether7 ] disable-running-check=no
set [ find default-name=ether8 ] disable-running-check=no
/interface bridge port
add bridge=bridge-lan interface=ether1
add bridge=bridge-lan interface=ether2
/ip dhcp-client
add interface=ether1 name=client1
/system identity
set name=Access-SW3
```

### Explanation

-   `bridge-lan` connects `ether1` and `ether2` at Layer 2.
-   DHCP client `client1` is configured on `ether1`; status is not
    shown.
-   No static management IP is visible in the screenshot.

------------------------------------------------------------------------

## 8. Verification commands

Run these on the individual router being checked. These are inspection
commands, not configuration changes.

``` routeros
/export hide-sensitive
/system identity print
/ip address print detail
/interface ethernet print detail
/interface bridge port print detail
/interface bridge vlan print detail
/interface vlan print detail
/interface vrrp print detail
/ip route print detail
/ip dhcp-client print detail
/ip firewall nat print detail
/ip firewall filter print detail
```

### Validation checklist

1.  Confirm VRRP IDs, virtual addresses, priorities, and runtime states
    on both core switches.
2.  Confirm VLAN 10/20/30/99 tagging and untagged membership matches the
    actual links.
3.  Verify the `10.10.10.1` FortiGate transit gateway is reachable from
    both cores.
4.  Check the duplicate default route shown on Core-SW2.
5.  Investigate Core-SW2's VLAN 20 row that references `ether5`, despite
    the export noting `ether5` is not a bridge port.
6.  Verify whether DHCP clients configured on bridge slave/passthrough
    interfaces are needed and operational.
7.  Review Core-SW1's NAT match (`src-address=10.10.0.0/16` and
    `dst-address=10.10.0.0/16`) before altering it; it is preserved here
    as exported.
8.  Back up each router before making changes, and make changes one
    device at a time.

## Important limitations

This document records what is visible in the screenshots. It is not
proof that every configuration line or live state is captured. Do not
import the whole document into any router: it contains several devices'
configurations and explanatory material. Use each code block only as a
reference for that named device, and verify the live export before
applying corrections.
