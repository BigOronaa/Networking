# Subnetting Plan for a Growing Organization


## Objective
Design an efficient subnetting plan for a growing organization with six departments, planning for at least **50%** growth in each department over the next five years. Ensure logical separation, addressing efficiency (VLSM), and operational/management guidance.

---

## 1) Requirements & Growth Calculation (digit-by-digit)
Departments and current host needs (given):

- Finance: **50** hosts
- Human Resources: **30** hosts
- Sales: **70** hosts
- Marketing: **40** hosts
- IT: **100** hosts
- Operations: **80** hosts

**Planned growth:** 50% increase for each department over five years → multiply each host requirement by 1.5 and round up to the next whole host requirement.

Calculations (1.5 × current):

- Finance: 50 × 1.5 = **75** → needs ≥75 usable hosts.
- Human Resources: 30 × 1.5 = **45** → needs ≥45 usable hosts.
- Sales: 70 × 1.5 = **105** → needs ≥105 usable hosts.
- Marketing: 40 × 1.5 = **60** → needs ≥60 usable hosts.
- IT: 100 × 1.5 = **150** → needs ≥150 usable hosts.
- Operations: 80 × 1.5 = **120** → needs ≥120 usable hosts.

Now chose a subnet sizes (usable hosts = 2^n − 2):

- /24 → 256 addresses → **254 usable** hosts
- /25 → 128 addresses → **126 usable** hosts
- /26 → 64 addresses → **62 usable** hosts

Match each requirement to the smallest subnet that satisfies it:

- Finance (75) → **/25 (126 usable)** ✅
- HR (45) → **/26 (62 usable)** ✅
- Sales (105) → **/25 (126 usable)** ✅
- Marketing (60) → **/26 (62 usable)** ✅
- IT (150) → **/24 (254 usable)** ✅
- Operations (120) → **/25 (126 usable)** ✅

---

## 2) Chosen Private Address Space & Rationale
Use a private RFC1918 block. Recommendation: **10.10.0.0/16** for the site — large enough to hold many VLSM subnets and future expansion while keeping addressing simple and contiguous.

We will allocate contiguous VLSM subnets inside 10.10.0.0/16 (start from .0 upward), assigning largest subnets first to avoid fragmentation.

---

## 3) Final Subnet Allocation (VLSM, contiguous)
| Dept / Purpose | Required hosts (with 50% growth) | Subnet (CIDR) | Network | Usable IP range | Gateway | Usable hosts |
|---|---:|---:|---:|---|---|---:|
| IT — infrastructure & servers | 150 | **10.10.0.0/24** | 10.10.0.0 | 10.10.0.1 - 10.10.0.254 | 10.10.0.1 | 254 |
| Data Center — servers/DMZ (centralized) | reserve for servers | **10.10.1.0/24** | 10.10.1.0 | 10.10.1.1 - 10.10.1.254 | 10.10.1.1 | 254 |
| Sales — CRM & sales data | 105 | **10.10.2.0/25** | 10.10.2.0 | 10.10.2.1 - 10.10.2.126 | 10.10.2.1 | 126 |
| Operations — production & supply chain | 120 | **10.10.2.128/25** | 10.10.2.128 | 10.10.2.129 - 10.10.2.254 | 10.10.2.129 | 126 |
| Finance — financial systems | 75 | **10.10.3.0/25** | 10.10.3.0 | 10.10.3.1 - 10.10.3.126 | 10.10.3.1 | 126 |
| Marketing — campaigns & analytics | 60 | **10.10.3.128/26** | 10.10.3.128 | 10.10.3.129 - 10.10.3.190 | 10.10.3.129 | 62 |
| HR — employee data & payroll | 45 | **10.10.3.192/26** | 10.10.3.192 | 10.10.3.193 - 10.10.3.254 | 10.10.3.193 | 62 |

**Reserved for future growth / spares:**  
- **10.10.4.0/22** (covers 10.10.4.0 - 10.10.7.255; /22 = 1024 addresses, 1022 usable) — use for new departments, wireless, contractors, or larger expansions.  
- Keep at least one additional /24 block (e.g., 10.10.8.0/24) reserved for unexpected growth or infrastructure services (backups, monitoring, logs).

---

## 4) VLAN & Routing Plan (logical separation)
Assign a VLAN per department and map to the subnet above. Example VLAN IDs:

- VLAN 10 — Finance — 10.10.3.0/25
- VLAN 20 — HR — 10.10.3.192/26
- VLAN 30 — Sales — 10.10.2.0/25
- VLAN 40 — Marketing — 10.10.3.128/26
- VLAN 50 — IT — 10.10.0.0/24
- VLAN 60 — Operations — 10.10.2.128/25
- VLAN 100 — Data Center / Servers — 10.10.1.0/24
- VLAN 200 — Reserved/Guest/Wireless — 10.10.4.0/22 (subdivided as required)

**Inter-VLAN routing:** Use a Layer 3 switch or router (SVI or sub-interfaces) to route between VLANs. Example SVI config (Cisco-like):

```
interface Vlan10
 ip address 10.10.3.1 255.255.255.128
!
interface Vlan20
 ip address 10.10.3.193 255.255.255.192
!
interface Vlan30
 ip address 10.10.2.1 255.255.255.128
!
interface Vlan40
 ip address 10.10.3.129 255.255.255.192
!
interface Vlan50
 ip address 10.10.0.1 255.255.255.0
!
interface Vlan60
 ip address 10.10.2.129 255.255.255.128
!
interface Vlan100
 ip address 10.10.1.1 255.255.255.0
```

---

## 5) DHCP & Static IP Recommendations
- **DHCP** for user workstations: configure DHCP scopes per VLAN/subnet but **exclude** the gateway (.1) and several low addresses for static assignment. Example DHCP pool for Finance (10.10.3.0/25):
  - Scope: 10.10.3.2 - 10.10.3.120
  - Exclusions: 10.10.3.1 (gateway), 10.10.3.2–10.10.3.10 (reserved for printers/network devices), 10.10.3.120–10.10.3.126 (reserved)
- **Static IPs** for critical infrastructure (servers, routers, switches, printers):
  - Place in the low-address range of each subnet (e.g., .2–.20)
  - Use a consistent naming and documentation policy.

---

## 6) Security & Access Controls
- **ACLs / Firewall rules**: implement east-west controls — restrict which departments can reach sensitive servers (e.g., Finance should have restricted access to payroll servers).  
- **Zero trust principles**: use identity-aware proxies for sensitive services.  
- **Segmentation**: keep servers in the Data Center VLAN; use firewalls between user VLANs and the server VLAN for sensitive services (e.g., Finance DB).

---

## 7) Troubleshooting Scenarios & Commands (quick-reference)

**Connectivity Issues**
- Check host IP & gateway: `ip addr` (Linux), `ipconfig /all` (Windows)
- Ping/gateway: `ping 10.10.3.1`
- Traceroute: `traceroute 10.10.1.10` (Linux) / `tracert` (Windows)
- Check routing table: `ip route` (Linux) / `route print` (Windows) / `show ip route` (Cisco)

**IP Conflicts**
- Check ARP table: `arp -a`
- DHCP logs to identify conflicting leases (DHCP server)
- Temporarily assign a unique static IP to the affected host to isolate

**Overlapping subnets**
- Verify all SVI/subinterfaces and DHCP scopes do not overlap; use `show running-config` on routers/switches to confirm.
- Use `ipcalc` or subnet calculators to verify ranges.

**DHCP troubleshooting**
- Verify DHCP scope and exclusions
- Check DHCP server logs
- `dhcpd -t` (test config) or Windows Server DHCP console
- Use `tcpdump -i <iface> port 67 or 68` to capture DHCP traffic

**Firewall / ACL problems**
- Inspect ACLs: `show access-lists` or firewall GUI
- Temporarily relax rules to narrow down cause; use policy as code to version control.

---

## 8) Future Growth & Migration Strategies
**Option A — Scale inside reserved space:** Use the reserved `10.10.4.0/22` to allocate new teams/units or expand departments without renumbering current subnets.

**Option B — Grow existing subnets (readdressing):**
- If a department outgrows its assigned subnet, plan readdressing to a larger adjacent block (requires careful migration and downtime planning). Example: upgrade Finance from /25 → /24 if needed (requires moving to 10.10.3.0/24 or another free /24).

**Documentation & Change Control**
- Maintain an IPAM spreadsheet or IPAM tool (phpIPAM, NetBox) and version control all network configs.
- Schedule regular address utilization reviews (quarterly).

---

## 9) Optimization Recommendations
- **Efficient routing:** use OSPF or EIGRP internally if you have multiple routers; for a single-site use static routes with a core L3 switch.
- **VLANs & QoS:** tag VLANs, use QoS for voice/critical traffic.
- **IP Address Reuse:** reclaim unused addresses and shrink oversized subnets (if possible).
- **Monitoring:** implement NetFlow/sFlow and centralized logging for traffic patterns.

---





