# Vulnerability Lab Findings

## Finding 1: CVE-2023-48795 (Terrapin)

| Field | Detail |
|---|---|
| Host | rocky86 (Rocky Linux 8.6, 192.168.122.111) |
| Package | openssh / openssh-server / openssh-clients (8.0p1, 8.6-era build) |
| CVE | CVE-2023-48795, "Terrapin" SSH prefix truncation attack |
| Severity | Medium (CVSS 3.1: 5.9). Network attack, but needs a man-in-the-middle position |
| Status | Vulnerable |

### Evidence

**Installed package is not fixed**

```
$ rpm -q --changelog openssh | grep CVE-2023-48795
(no output)
```

**Rocky errata do not tag this CVE**

```
$ dnf updateinfo list --cve CVE-2023-48795
(no output)
```

All 9 pending OpenSSH security advisories were checked one by one
(`dnf updateinfo info <advisory>`). None lists CVE-2023-48795.

**Red Hat fixed it only in Extended Update Support (EUS) streams**

- RHSA-2024:0429: RHEL 8.6 EUS
- RHSA-2024:0594: RHEL 8.8 EUS, fixed package `openssh-8.0p1-19.el8_8.2`

Rocky Linux rebuilds only the main RHEL 8 stream, not EUS. Rocky's errata
list contains `openssh-8.0p1-19.el8_8` (RLSA-2023:4419) but not the EUS
respin `8.0p1-19.el8_8.2`.

**Latest Rocky package changelog**

```
$ dnf download openssh --destdir /tmp/latest
$ rpm -qp --changelog /tmp/latest/openssh-*.rpm | grep 48795
<PASTE RESULT HERE>
```

### Conclusion

<CHOOSE ONE after running the check above>

- **Match found:** The fix exists in Rocky's latest openssh build, but Rocky's
  errata metadata does not tag the CVE. `dnf updateinfo --cve` gives a false
  "nothing to fix" result. Patching to the latest build resolves it.
- **No match:** Rocky 8's main stream has no Terrapin fix. The only protection
  is configuration hardening.

### Remediation

1. **Patch** (if the fix exists): update openssh to the latest Rocky build.
   Keep the current SSH session open while restarting `sshd`, and test a new
   login before closing it.
2. **Mitigate by configuration** (always worth doing): disable the affected
   cipher and MACs in `/etc/ssh/sshd_config` (on Rocky 8, via the system
   crypto policy), specifically `chacha20-poly1305@openssh.com` and all
   `*-etm@openssh.com` MACs.
3. **Verify**: rescan and confirm the cipher/MAC are no longer offered
   (`ssh -Q cipher`, or `nmap --script ssh2-enum-algos -p22 <host>`).

### Lesson learned

An empty `dnf updateinfo --cve` result does not prove a host is safe. Always
cross-check against the upstream vendor (Red Hat CVE pages and RHSA advisories).
Commercial scanners such as Nessus would still flag this CVE.

### References

- https://access.redhat.com/security/cve/CVE-2023-48795
- https://access.redhat.com/errata/RHSA-2024:0594
- https://www.tenable.com/plugins/nessus/189563
