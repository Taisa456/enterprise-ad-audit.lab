# Security & Sanitization Notes

This repository contains an academic lab audit. Before any commit reaches the public branch, the items below were verified — and should be re-verified by anyone forking, extending, or adding screenshots.

## Pre-push checklist

### Credentials & secrets
- [ ] No plaintext passwords, even "lab" passwords or weak placeholders
- [ ] No password hashes (NTLM, NetNTLMv2, Kerberos AS-REP, etc.)
- [ ] No API keys, license keys (Nessus, Metasploit Pro), or activation codes
- [ ] No SSH private keys, certificates, or `.pem` / `.ppk` files
- [ ] No domain credentials, even for lab domains — patterns can be reused
- [ ] No `.bash_history`, `.zsh_history`, or shell history artefacts
- [ ] Run `git log -p` and grep for `password`, `token`, `secret`, `key`, `cred`

### Network & identity
- [ ] Only RFC1918 / lab IP ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
- [ ] No public IPs, hostnames, or domain names belonging to real organisations
- [ ] No internal corporate naming conventions if this lab was hosted on real infrastructure
- [ ] MAC addresses redacted from screenshots (random-locally-administered or `XX:XX:XX:XX:XX:XX`)
- [ ] VM UUIDs / serial numbers cropped or blurred in screenshots

### Personal data
- [ ] Author name(s) reflect only people who consented to public attribution
- [ ] No email addresses, student IDs, or personal phone numbers
- [ ] No instructor names or course identifiers if your institution's policy forbids this

### Scan output
- [ ] `.nessus` and `.csv` files reviewed before commit (excluded by `.gitignore` by default; use `git add -f` only for sanitized exports)
- [ ] Nessus screenshots: scanner license key cropped out, scan ID redacted if pointing to a private account
- [ ] No proprietary plugin output if you have a paid Nessus tier

### What's intentionally public
- The lab uses Metasploitable 2 (publicly available, intentionally vulnerable). All CVEs and finding details documented here are well-known and the entire point of the target.
- RFC1918 IP space, the made-up domain `kbr.local`, and the hostname `KBR-DC01` are lab fixtures and pose no risk.
- Default Metasploitable credentials referenced in vulnerability findings (e.g., Tomcat / VNC `password`) are documented in the public security literature — they describe the vulnerability, not a credential you're protecting.

## Reporting a security issue with this repository

This is an academic lab repository, not a production service. If you spot something here that you believe is a real security exposure (a leaked credential, a real IP, anything that shouldn't be public), open a private issue or contact the repository owner directly rather than filing a public issue.

