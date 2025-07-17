# 🛡️ Enterprise Cybersecurity Lab Setup


This is my personal cybersecurity lab where I built a simulated Windows Server environment to practice essential blue team and system administration skills. I used Windows Server and a client OS to simulate Active Directory, Group Policy, file server security, DNS sinkholes, Sysmon logging — all in VirtualBox.

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




<img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/9ab055417563108d172553165b93d48edaf919ef/networksetting.png?raw=true" alt="Adapter Settings" width="300"/><img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/62aab6ef6b3d6d9b8d16b84fa1d920512f0b3b02/PROPERTIES.png?raw=true" alt="Properties" width="300"/><img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/efafdc69c12ae130628028cc3fda5ff93888fef3/Adapter%20settings.png?raw=true" alt="Adapter Settings" width="300"/>




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


<h3>Joining the Domain (Part 1)</h3>
<img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/62aab6ef6b3d6d9b8d16b84fa1d920512f0b3b02/JOIN%20DOMAIN.png?raw=true" alt="Join Domain Screenshot" width="300"/>

<h3>Joining the Domain (Part 2)</h3>
<img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/a8549ae42e37f6360adf45963b9ceb8bdfb99a11/Join%20domain%20(2).png?raw=true" alt="Join Domain Step 2" width="300"/>




---

### 4. Log In with a Domain User

After rebooting, I logged in using a domain account I created on the DC (`corp\jsmith`).

---

### 5. Group Creation and RBAC

In **Active Directory Users and Computers** on DC01:

- I created these security groups:
  - IT Admins
  - HR users
- Then I created user accounts like `jsmith`, `ajones`, `bwayne`, and `intern1`
- Assigned them to their respective groups

<h3>Role-Based Access Control (RBAC) - Group Creation</h3>
<img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/4875fd86f8a1c5e28a491dc0d69b6b967cabeb0c/Group%20creation.png?raw=true" alt="Group Creation for RBAC" width="600"/>



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

<h3>NTFS File Permissions</h3>
<img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/62aab6ef6b3d6d9b8d16b84fa1d920512f0b3b02/NTFS%20file%20permissions.png?raw=true" alt="NTFS File Permissions Screenshot" width="600"/>

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

<h3>Security Hardening</h3>
<img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/62aab6ef6b3d6d9b8d16b84fa1d920512f0b3b02/security%20hardening.png?raw=true" width="300"/>

<h3>Warning Screen</h3>
<img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/62aab6ef6b3d6d9b8d16b84fa1d920512f0b3b02/warning%20screen.png?raw=true" width="300"/>




---

### 8. Audit & Monitor Logins

I enabled login auditing and viewed results in Event Viewer:

- `Event ID 4624`: Successful login
- `Event ID 4625`: Failed login
  
<h3>Event Viewer - Login Audit</h3>
<img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/4875fd86f8a1c5e28a491dc0d69b6b967cabeb0c/Event%20viewer-login%20audit.png?raw=true" width="600"/>
---

### 9. Simulate a Brute Force Attack

On **Client01**, I tried several failed logins to trigger lockout:

```powershell
for /L %i in (1,1,10) do net use \DC01\C$ /user:corp\jsmith wrongpassword
```

After 3 attempts, the account locked. I verified it in Active Directory Users and Computers.

<h3>Account Lockout Enforcement</h3>
<img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/62aab6ef6b3d6d9b8d16b84fa1d920512f0b3b02/account%20lockout%20enforcement.png?raw=true" width="600"/>


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
<img src="https://github.com/enioluww/activedirectory-Cyberlab/blob/62aab6ef6b3d6d9b8d16b84fa1d920512f0b3b02/sysmon%20logs.png?raw=true" width="600" />


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


### What This Lab Taught Me

Building this lab gave me real-world skills in:

- Setting up and managing Active Directory

- Implementing RBAC and file server permissions

- Hardening systems with Group Policy

- Detecting and simulating security incidents

- Monitoring logs with Sysmon and Event Viewer

- Practicing defense against common red team tactics


## 🙋‍♂️ About Me

**Enioluwa Israel Akinwande**\
Aspiring Cybersecurity Analyst\
📧 [enioluwakinwande@gmail.com](mailto\:enioluwakinwande@gmail.com)\
📍 Based in Canada

