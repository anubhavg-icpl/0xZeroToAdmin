# From Zero Creds to Enterprise Admin

A comprehensive technical write-up demonstrating how Active Directory misconfigurations can be chained together to achieve complete domain compromise - without exploiting any software vulnerabilities.

## Overview

This repository documents a real-world attack chain that escalates from zero credentials to Enterprise Admin access by exploiting common Active Directory misconfigurations:

- **SMB Signing Disabled** - Enables NTLM relay attacks
- **LLMNR/NBT-NS Enabled** - Allows network poisoning and credential capture
- **IPv6 DNS Misconfiguration** - Permits MITM6 attacks
- **Weak Domain ACLs** - Enables privilege escalation to DCSync
- **Default Machine Account Quota** - Allows unauthorized computer account creation

## Attack Chain Summary

```
No Credentials
      │
      ▼
Network Poisoning (Responder + MITM6)
      │
      ▼
NTLM Hash Capture
      │
      ▼
NTLM Relay to LDAP (ntlmrelayx)
      │
      ▼
Machine Account Creation + ACL Manipulation
      │
      ▼
DCSync Privileges Obtained
      │
      ▼
Domain Credential Extraction (secretsdump)
      │
      ▼
Enterprise Admin Access
```

## Tools Used

| Tool | Purpose |
|------|---------|
| Nmap | Network reconnaissance and port scanning |
| NetExec (CrackMapExec) | SMB signing enumeration and post-exploitation |
| Responder | LLMNR/NBT-NS poisoning and hash capture |
| MITM6 | IPv6 DNS spoofing |
| Impacket (ntlmrelayx) | NTLM relay attacks |
| Impacket (secretsdump) | DCSync credential extraction |
| Evil-WinRM | Remote PowerShell access |

## Repository Structure

```
.
├── README.md                 # This file
├── article.md                # Full technical write-up
├── LICENSE                   # MIT License
└── scripts/                  # Helper scripts and commands
    └── commands.md           # Quick reference command cheat sheet
```

## Key Takeaways

1. **Patching alone is not enough** - This attack requires no CVE exploitation
2. **Default configurations are dangerous** - Many attack vectors exist due to Windows defaults
3. **Defense in depth matters** - Multiple security controls could have stopped this chain
4. **SMB signing is critical** - Enforce it on all domain-joined systems

## Defensive Recommendations

- Enforce SMB signing across all domain systems
- Disable LLMNR and NBT-NS via Group Policy
- Properly configure or disable IPv6 if not in use
- Set Machine Account Quota to 0
- Implement tiered administration model
- Monitor for DCSync attacks (Event ID 4662)
- Use Protected Users security group for privileged accounts

## Disclaimer

This content is provided for **educational and authorized security testing purposes only**. The techniques described should only be used in environments where you have explicit written authorization. Always follow responsible disclosure practices and operate within legal and ethical boundaries.

## Author

**Anubhav Gain**
Security Researcher

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
