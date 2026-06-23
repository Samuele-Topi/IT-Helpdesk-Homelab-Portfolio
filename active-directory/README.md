# 🏢 Windows Server & Active Directory Infrastructure Lab

## 🎯 Objective
The goal of this laboratory is to design, deploy, and secure a simulated corporate network environment representing a small-to-medium enterprise (SME). This lab demonstrates standard systems administration skills, including domain controller setup, directory virtualization, user provisioning, access control via Role-Based Access Control (RBAC), Group Policy automation, and networking troubleshooting.

---

## 🏗️ Lab Architecture
The lab runs locally on Oracle VirtualBox, using a host-only or custom network layout (`192.168.10.0/24`) to ensure isolated, secure communication.

```
                  ┌──────────────────────────────┐
                  │          PC Host             │
                  │   (Oracle VirtualBox 7.x)    │
                  └──────────────┬───────────────┘
                                 │
                   [ VM Network: 192.168.10.0/24 ]
                                 │
         ┌───────────────────────┴───────────────────────┐
         │                                               │
┌────────┴────────┐                             ┌────────┴────────┐
│  Domain Controller                             │  Client Machine  │
│      [DC-01]    │                             │    [CLI-W11]    │
│                 │                             │                 │
│ Win Server 2022 │                             │ Windows 11 Ent  │
│ IP: 192.168.10.10│                            │ IP: 192.168.10.20│
│ AD DS, DNS      │                             │ DNS: 192.168.10.10│
└─────────────────┘                             └─────────────────┘
```

### Virtual Machine Specifications
* **Hypervisor:** Oracle VirtualBox 7.x
* **Domain Controller (DC-01):**
  * **OS:** Windows Server 2022 Standard Evaluation (180-day trial)
  * **Hardware:** 2 vCPUs, 6 GB RAM, 50 GB HDD
  * **IP Address:** `192.168.10.10` (Static)
  * **Subnet Mask:** `255.255.255.0`
  * **Gateway:** `192.168.10.1` (or local router/NAT gateway)
  * **DNS Server:** `127.0.0.1` (pointing to itself)
  * **Active Roles:** AD DS (Active Directory Domain Services) & DNS
  * **Domain Name:** `homelab.local`
* **Client Workstation (CLI-W11):**
  * **OS:** Windows 11 Enterprise Evaluation
  * **Hardware:** 2 vCPUs, 4 GB RAM, 60 GB HDD
  * **IP Address:** `192.168.10.20` (Static/DHCP reserved)
  * **DNS Server:** `192.168.10.10` (pointing to the DC)

---

## 🛠️ Step-by-Step Implementation

### Phase 1: DC Promotion & Network Setup
1. Installed Windows Server 2022 and configured a static IP address (`192.168.10.10`) on the primary network interface.
2. Installed Active Directory Domain Services (AD DS) and DNS Server roles through the Server Manager.
3. Promoted the server to Domain Controller, creating a new forest under the domain `homelab.local`.
4. Verified that DNS was running and that local host records were created automatically.
*Screenshot proof:* Refer to [Server Manager Setup](screenshots/1_Server_IP.png)

### Phase 2: Active Directory Logical Design (OUs, Users & Groups)
An Organizational Unit (OU) structure was designed to mirror a real corporate hierarchy. Users were organized under `Azienda-HQ` to avoid cluttering default container systems.

```
homelab.local (Domain Root)
└── Azienda-HQ (Parent OU)
    ├── IT (Sub-OU)
    ├── Sales (Sub-OU)
    └── HR (Sub-OU)
        ├── Mario Rossi (User: mrossi)
        └── Security Groups:
            └── HR_Docs_Access (Global Security Group)
```

1. **User Provisioning:**
   * **Mario Rossi** (mrossi) was created in the HR sub-OU. The account was configured to require a password change on next login.
2. **Access Control (RBAC):**
   * Created a global security group called `HR_Docs_Access` inside the HR OU.
   * Added `Mario Rossi` as a member of this security group.
*Screenshot proof:* Refer to [User Security Membership](screenshots/4_Member_Of.png)

### Phase 3: Shared Folder & NTFS Permissions Setup
To simulate an enterprise file server:
1. Created a sharing directory `HR_Docs` on the Domain Controller.
2. Configured advanced sharing on the folder.
3. Applied strict **NTFS security permissions**:
   * Removed inheritance to exclude generic domain users.
   * Assigned **Modify (Full Write/Read)** permissions exclusively to the `HR_Docs_Access` security group.
   * This ensures that only members of the HR group can access or modify HR files, proving domain-level authorization logic.

### Phase 4: Group Policy (GPO) Automation
Two policies were implemented to automate workstation management and secure user environments:

| GPO Name | Linked To | Configuration Path & Purpose |
| :--- | :--- | :--- |
| **Map_Drive_HR** | `HR` OU | `User Configuration -> Preferences -> Windows Settings -> Drive Maps`. Mapped the network path `\\192.168.10.10\HR_Docs` to drive letter `Z:` for all users logged in from this OU. |
| **Restrict_Control_Panel** | (OU Domain level / HR / Sales) | `User Configuration -> Policies -> Administrative Templates -> Control Panel`. Enabled the rule "Prohibit access to Control Panel and PC settings" to restrict unauthorized personnel from tampering with workstation settings. |

*Screenshot proof:*
* Refer to [GPO Link Structure](screenshots/5_GPO.png)
* Refer to [GPO Mapped Drive Configuration](screenshots/5_1_ALL_GPO.png)

### Phase 5: Client Integration & Policy Verification
1. Installed Windows 11 Client VM and configured the network card to use the same internal virtual network.
2. Modified the client's IPv4 properties to point its primary DNS to the DC (`192.168.10.10`) and assigned client IP `192.168.10.20`.
3. Joined the machine to the domain `homelab.local` using Domain Admin credentials.
*Screenshot proof:*
* Refer to [Client DNS & Resolution Verification](screenshots/3_IPConfig_Ping.png)
* Refer to [Active Directory Domain Joined Computer](screenshots/2_CLI_Joined.png)

#### Client-side Verification Results:
* Logged in as **Mario Rossi**:
  * Verified that the client requested a password change during the initial login.
  * Opened File Explorer and confirmed that drive letter `Z:` was successfully mapped to `\\192.168.10.10\HR_Docs`. Verified write permissions by successfully saving files in `Z:`.
* Workstation Restriction Checks:
  * Attempted to open the Control Panel. The system successfully blocked the request, displaying the expected GPO restriction dialog.
*Screenshot proof:* Refer to [Control Panel Block Dialog](screenshots/6_ControlPanel_Block.png)

---

## 🔍 Troubleshooting Log

This section details technical challenges encountered during the implementation and how they were resolved:

### 1. Client unable to join domain
* **Symptoms:** Error message "An Active Directory Domain Controller (AD DC) for the domain 'homelab.local' could not be contacted" was displayed on the client.
* **Diagnostics:** Ran `nslookup homelab.local` on the client CMD. It failed to resolve, meaning the client was not resolving domain names through the Domain Controller.
* **Root Cause:** The client was obtaining DNS automatically from VirtualBox DHCP instead of routing DNS requests to the Windows Server DNS service.
* **Resolution:** Opened the Network Connections control panel on the client, accessed IPv4 properties, and manually changed the DNS server to the DC static IP (`192.168.10.10`). Ran `ipconfig /flushdns` and tried the join process again. The client successfully contacted the domain and joined.

### 2. Drive Map GPO not appearing on client
* **Symptoms:** Mapped drive `Z:` did not appear in File Explorer after logging in as Mario Rossi.
* **Diagnostics:** Ran the command `gpresult /r` in CMD as Mario Rossi. The output listed `Map_Drive_HR` under "Applied Group Policy Objects" but the drive was still missing.
* **Root Cause:** The GPO was configured to "Create" the drive map but required the "Update" action to force instant execution upon first login, and sharing permissions needed auditing.
* **Resolution:** Changed the GPO drive map action to "Update" in the Group Policy Management Editor on `DC-01`. Added the `HR_Docs_Access` group to the Sharing permissions tab with modification rights. Ran `gpupdate /force` on the client and restarted. The network drive appeared immediately.

---

## 🛠️ Demonstrated Competencies
* **Active Directory Domain Services (AD DS):** Directory structures, object grouping, domain management, role provisioning.
* **Group Policy Management:** Script-free configuration automation, security hardening, user environment provisioning.
* **Network Administration:** Configuration of DNS static hosts, resolving IP routing conflicts, CLI-based diagnostic tools.
* **Access Control Lists (NTFS/Share Permissions):** Enforcement of least-privilege security policies (RBAC).
