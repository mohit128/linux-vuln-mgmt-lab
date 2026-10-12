# Linux Vulnerability Management Lab

A hands-on home lab for practicing the full vulnerability management cycle on
RHEL-family Linux: **find → prioritize → fix → verify**.

The goal is to work the way a production Linux/SecOps team does: scan real
(deliberately outdated) systems, verify findings against vendor data instead of
trusting a single tool, remediate with automation, and document every decision.

> Status: **in progress.** Findings are added as they are investigated.

## Lab setup

| Component | Details |
|---|---|
| Host | Rocky Linux 10.2 laptop, KVM / virt-manager |
| Network | libvirt default NAT (`192.168.122.0/24`), isolated from the home LAN |
| Target 1 | Rocky Linux 8.6 minimal, intentionally unpatched (vulnerable target) |
| Target 2 | Rocky Linux 10.1 minimal (current baseline for comparison) |
| Control node | The host laptop, running Ansible over SSH |

Snapshots are taken before scanning or fixing, so every target can be reset to
its original vulnerable state.

## Tools

| Purpose | Tool |
|---|---|
| Compliance scanning | OpenSCAP + SCAP Security Guide (PCI DSS v4.0.1 profile) |
| CVE / errata checks | `dnf updateinfo`, `rpm -q --changelog` |
| Vendor verification | Red Hat CVE pages and RHSA advisories |
| Remediation | Ansible |
| Network scanning (planned) | Greenbone OpenVAS |

## Workflow

1. **Find**: run an OpenSCAP compliance scan and list pending security errata.
2. **Prioritize**: research each finding (CVSS, EPSS, CISA KEV, exposure) and
   rule out false positives caused by Red Hat backporting.
3. **Fix**: apply patches and configuration hardening with Ansible.
4. **Verify**: rescan and record before/after results.

## Findings

| # | Finding | Host | Status |
|---|---|---|---|
| 1 | [CVE-2023-48795 (Terrapin)](findings.md#finding-1-cve-2023-48795-terrapin) | rocky86 | Investigating |

## Key lessons so far

- An empty `dnf updateinfo --cve` result does not prove a host is safe. Rocky's
  errata metadata can miss CVE tags, so findings are cross-checked against Red
  Hat's own data.
- Red Hat backports fixes without changing upstream version numbers, so scanners
  that only read versions produce false positives. The package changelog is the
  evidence.
- Some fixes exist only in Red Hat EUS streams, which Rocky Linux does not
  rebuild.

## Roadmap

- [x] Build KVM lab with Rocky 8.6 and Rocky 10.1 targets
- [x] Investigate first CVE finding (Terrapin)
- [ ] OpenSCAP PCI DSS scan reports for both targets
- [ ] Document 5 prioritized findings
- [ ] Ansible remediation playbook
- [ ] Rescan and publish before/after results
- [ ] OpenVAS network scan comparison

## Safety

All targets run on an isolated NAT network on a personal machine. No employer
systems, data, or configurations are included in this repository.

## License

MIT
