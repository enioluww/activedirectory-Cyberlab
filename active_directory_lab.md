# 🛡️ My Active Directory Cybersecurity Lab

This is my personal cybersecurity lab where I built a simulated Windows Server environment to practice essential blue team and system administration skills. I used Windows Server and a client OS to simulate Active Directory, Group Policy, file server security, DNS sinkholes, Sysmon logging, and red team tactics — all in VirtualBox.

---

## 💻 My Lab Architecture

| Machine Name | Role              | OS             | IP Address    |
| ------------ | ----------------- | -------------- | ------------- |
| DC01         | Domain Controller | Windows Server | 192.168.56.10 |
| Client01     | Domain Member     | Windows 10     | 192.168.56.X  |

> Note: I configured both machines in Host-Only Network mode in VirtualBox so they can communicate.

---

## 🚀 How to Recreate My Lab

### 1. Set a Static IP Address

On **Client01**, I:

- Opened `Control Panel > Network and Sharing Center > Adapter Settings`
- Right-clicked the network adapter > Properties > IPv4
- Set:
  - IP Address: `192.168.xx.xx`
  - Subnet Mask: `255.255.255.0`
  - Default Gateway: `192.168.xx.xx`
  - Preferred DNS: `192.168.xx.xx` (my DC)

📸&#x20;

---

### 2. Test Connection to Domain Controller

I opened Command Prompt and ran:

```bash
ping 192.168.xx.xx
```

---

### 3. Join the Domain

On **Client01**:

- Opened `System Properties > Change Settings > Computer Name`
- Clicked **Domain**, typed `corp.local`
- Entered domain admin credentials
- Restarted after the prompt

📸  📸;

---

### 4. Log In with a Domain User

After rebooting, I logged in using a domain account I created on the DC (`corp\jsmith`).

---

### 5. Group Creation and RBAC

In **Active Directory Users and Computers** on DC01:

- I created these security groups:
  - IT Admins
  - HR
  - Finance
- Then I created user accounts like `jsmith`, `ajones`, `bwayne`, and `intern1`
- Assigned them to their respective groups

📸&#x20;
[alt text](GPO's.png)

---

### 6. File Server + NTFS Permissions

I created and shared these folders on DC01:

```
C:\Shares\HR
C:\Shares\Finance
C:\Shares\Public
```

Then I applied NTFS permissions so that:

- Only `HR` group can access the HR share
- Only `Finance` group can access the Finance share
- All domain users can access Public (read-only)

📸  📸&#x20;

---

### 7. Group Policy Hardening

Using Group Policy Management, I applied:

| Policy                 | Scope     | Purpose                    |
| ---------------------- | --------- | -------------------------- |
| Disable Control Panel  | Interns   | Limit settings access      |
| Disable Command Prompt | Interns   | Prevent CMD use            |
| Account Lockout Policy | All Users | Lock after 3 failed logins |
| Logon Warning Banner   | All Users | Legal warning at login     |
| Block USB Storage      | All Users | Prevent data exfiltration  |

📸  📸  📸&#x20;

---

### 8. Audit & Monitor Logins

I enabled login auditing and viewed results in Event Viewer:

- `Event ID 4624`: Successful login
- `Event ID 4625`: Failed login

📸&#x20;

---

### 9. Simulate a Brute Force Attack

On **Client01**, I tried several failed logins to trigger lockout:

```powershell
for /L %i in (1,1,10) do net use \DC01\C$ /user:corp\jsmith wrongpassword
```

After 3 attempts, the account locked. I verified it in Active Directory Users and Computers.

---

### 10. Deploy Sysmon for Deep Logging

On both DC01 and Client01:

- I downloaded Sysmon from Microsoft
- Got SwiftOnSecurity's sysmon-config.xml
- Installed it via PowerShell:

```powershell
Sysmon64.exe -accepteula -i sysmonconfig.xml
```

- Then checked logs in:

```
Event Viewer > Applications and Services Logs > Microsoft > Windows > Sysmon > Operational
```

---

### 11. DNS Sinkhole / Web Filtering

To block known malicious domains:

- I edited the `hosts` file on both machines:

```plaintext
127.0.0.1 malicious-site.com
127.0.0.1 phishing-site.com
```

- This redirected any access attempts back to localhost.

---

### 12. Red Team Simulation

To simulate basic attacker actions:

```powershell
# List domain users
net user /domain

# List domain admins
net group "Domain Admins" /domain

# Add scheduled task for persistence
schtasks /create /tn "Backdoor" /tr "calc.exe" /sc onlogon /ru SYSTEM /f
```

---

## 🖼️ Screenshots

| Feature                     | Screenshot |
| --------------------------- | ---------- |
| Adapter Settings            |            |
| Domain Join                 |            |
| Group Creation              |            |
| NTFS Permissions            |            |
| Group Policy Editor         |            |
| Account Lockout Policy      |            |
| Event Viewer (Login Audits) |            |

---

## 📄 License

MIT License — use this for learning or your own projects!

## 🙋‍♂️ About Me

**Enioluwa Israel Akinwande**\
Aspiring Cybersecurity Analyst\
📧 [enioluwakinwande@gmail.com](mailto\:enioluwakinwande@gmail.com)\
📍 Based in Canada

