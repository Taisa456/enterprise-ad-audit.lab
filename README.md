# Enterprise Defense Lab — AEGIS Systems

A small enterprise network designed, deployed, and audited in a virtualised lab: three-zone segmentation, Active Directory authentication, firewall-controlled access between zones, and a full vulnerability assessment with **Nessus Essentials**.

The deliverable is an audit of the lab's own security posture — what works, what doesn't, and the concrete vulnerabilities found across every host.

---

## Table of Contents

- [Lab at a Glance](#lab-at-a-glance)
- [Network Architecture](#network-architecture)
- [Active Directory Setup](#active-directory-setup)
- [Machine Roles](#machine-roles)
- [Vulnerability Assessment](#vulnerability-assessment)
- [Key Findings](#key-findings)
- [Audit Checklist Results](#audit-checklist-results)
- [Summary & Conclusion](#summary--conclusion)
- [Repository Structure](#repository-structure)
- [Ethical Disclaimer](#ethical-disclaimer)
- [License](#license)

---

## Lab at a Glance

| | |
|---|---|
| **Virtual Machines** | 4 |
| **Network Zones** | 3 (WAN · Server · Operations) |
| **AD Users Created** | 5 |
| **Unique Vulnerabilities Found** | 87 (138+ total instances across hosts) |
| **Scan Date** | April 14, 2026 |
| **Scanner** | Nessus Essentials |

---

## Network Architecture

Three-zone segmentation with the **Main Server (AD/DC)** acting as router and authentication authority between the Operations and Server networks. Internet access is mediated through the WAN zone; the Operator workstation is intentionally blocked from the internet.

| Zone | CIDR | Hosts |
|---|---|---|
| **WAN Access** | — | Internet egress |
| **Operations Network** | 172.16.10.0/24 | Main Server (172.16.10.10) · Operator WS (172.16.10.100) · Kali Attacker (172.16.10.200) |
| **Server Network** | 172.16.20.0/24 | Metasploitable 2 (172.16.20.100) |

**Routing policy:**
- Operator → can reach Main Server, can RDP from Attacker, **blocked from internet**
- Attacker → has internet, can reach Operator via RDP, **blocked from Metasploitable**
- Main Server → has internet, routes between Ops and Server zones
- Metasploitable → reachable internally, isolated from the Attacker zone


---

## Active Directory Setup

| Item | Value |
|---|---|
| **Domain** | `kbr.local` |
| **Domain Controller** | `KBR-DC01` |
| **OS** | Windows Server 2022 |
| **IP** | 172.16.10.10 (Operations zone) |

### User Accounts

| Username | Role | Notes |
|---|---|---|
| Administrator | Domain Admin | Audit finding: lacks full privileges (see audit checklist) |
| Support | Support Staff | Standard user |
| Production 1 | Production | Domain-joined |
| Production 2 | Production | Domain-joined |
| Production 3 | Production | Domain-joined |

Passwords are deliberately weak and lab-only, set per the lab specification. Authentication, domain join, and policy application were all verified. *(Specific values redacted in the public version of this material — see [SECURITY.md](SECURITY.md) for sanitization notes.)*

---

## Machine Roles

| Host | OS | IP | Role |
|---|---|---|---|
| `KBR-DC01` | Windows Server 2022 | 172.16.10.10 | Domain Controller · router · firewall |
| Operator WS | Windows 10/11 | 172.16.10.100 | Domain-joined workstation · RDP target |
| Kali Attacker | Kali Linux | 172.16.10.200 | Pentest workstation — full sudo, blocked from Metasploitable |
| Metasploitable 2 | Ubuntu 8.04 | 172.16.20.100 | Deliberately vulnerable target server |

**Attacker capabilities verified:** internet access · AnyDesk installed · RDP to Operator · SMB enumeration via CrackMapExec · full sudo. **Restricted:** cannot reach Metasploitable, cannot reach Server Network directly.

---

## Vulnerability Assessment

Nessus Essentials scan run against all four hosts on April 14, 2026.

| Host | IP | Unique Vulnerabilities |
|---|---|---|
| Metasploitable 2 | 172.16.20.100 | **69** |
| Windows AD DC | 172.16.10.10 | 28 |
| Kali Linux | 172.16.10.200 | 22 |
| Windows Operator | 172.16.10.100 | 19 |
| **Total unique** | | **87** (138+ raw instances) |

> Unique = a single CVE found on N hosts is counted once. Raw instances counts each host occurrence separately.

These hosts share a single architecture, so a foothold on any one machine is a potential pivot to the rest. Metasploitable's 69 vulnerabilities dominate the risk surface and are the primary justification for its zone isolation.

---

## Key Findings

### Critical — Metasploitable 2 (172.16.20.100)

| Finding | CVSS | Summary |
|---|---|---|
| **UnrealIRCd Backdoor** | 10.0 | Unauthenticated remote access. Bypasses all controls. Full system compromise. **Mitigation:** remove backdoor, patch service, reinstall OS if compromise suspected. |
| **Remote Shell / RCE** (Tomcat, VNC default passwords) | 10.0 | Default credentials (`password`) allow arbitrary remote code execution. **Mitigation:** change all default passwords, disable unneeded services, enable NLA. |
| **Web Server RCE** | 9.8 | Web server exploitable without credentials. **Mitigation:** patch, restrict network access, input validation. |
| **Outdated OS — Ubuntu 8.04 EOL** | — | EOL since 2013; permanently unpatched. Root cause of the 69-vuln count. **Mitigation:** upgrade to supported OS. |

### High — Main Server (172.16.10.10)

- **Open NetBIOS Port 137** — leaks machine name and domain details, enabling AD discovery and targeted attacks. **Mitigation:** close port 137 at the firewall or disable NetBIOS over TCP/IP if legacy support isn't needed.

### Medium — Infrastructure

- **VMware Side Channel Vulnerability** — virtualisation-layer flaw allows memory/processor task leakage between VMs sharing the same hardware. Could expose cryptographic keys or credentials. **Mitigation:** enable side channel mitigations in VMware VM advanced options.
- **DNS resolution issue** on Kali — temporary DNS access failure during testing.

### High — Active Directory

- **Administrator account lacks full privileges** — created but with insufficient permissions to perform domain administration. **Mitigation:** ensure the account is correctly added to Domain Admins / Enterprise Admins.

---

## Audit Checklist Results

| Category | Weight | Status |
|---|---|---|
| **Network Configuration** | 30% | ✅ WAN/Server/Ops configured · ✅ Routing on Main Server · ⚠️ NetBIOS open on Win Server · ✅ Internet blocked for Operator · ✅ Attacker scoped to Operator only |
| **Active Directory** | 25% | ✅ AD DS installed · ✅ Domain created · ⚠️ 5 users created (Admin lacks full privileges) · ✅ Production users authenticate · ✅ Operator domain-joined |
| **System Config & Access Control** | 45% | ✅ 3 NICs on Main Server · ✅ Firewall rules · ✅ RDP enabled (NLA disabled) · ✅ Metasploitable internally reachable · ✅ Metasploitable isolated from Attacker · ✅ Operator blocked from internet |

---

## Summary & Conclusion

### Achieved

- 4-machine virtualised lab fully deployed
- 3-zone network segmentation implemented and enforced
- Active Directory with `kbr.local` domain operational
- 5 users created with correct authentication flow
- Operator workstation domain-joined
- RDP with NLA disabled on Operator (per lab requirements)
- Metasploitable isolated from the Attacker zone
- Nessus Essentials scan completed and documented

### Outstanding Issues

| Severity | Issue |
|---|---|
| Critical | Metasploitable 2 — 10 Critical CVEs |
| Critical | 69 total vulnerabilities on Metasploitable host (driven by Ubuntu 8.04 EOL) |
| High | Administrator account missing full domain privileges |
| High | NetBIOS port 137 open on Main Server |
| Medium | VMware side channel vulnerability detected |
| Medium | DNS resolution issue on Kali Linux |

**Bottom line:** most lab requirements were met. The architectural goals (segmentation, AD, controlled access) are in place. The Metasploitable host is intentionally vulnerable, so its findings are expected — but they were quantified and analysed rather than waved through. The two genuine remediations to prioritise are closing NetBIOS port 137 and correctly elevating the Administrator account.

---


## Ethical Disclaimer

All activities documented in this repository were performed exclusively within a self-contained virtualised lab environment for supervised academic learning purposes. No external, production, or third-party systems were targeted, scanned, or affected. Metasploitable 2 is an intentionally vulnerable virtual machine designed specifically for security training.

**This material must not be used to replicate these activities against any real system without explicit written authorisation from the system owner.** Unauthorised vulnerability scanning, credential attacks, or exploitation of network services is illegal in most jurisdictions.

---

## License

MIT — see [LICENSE](LICENSE). The presentation itself is provided for educational reference.
