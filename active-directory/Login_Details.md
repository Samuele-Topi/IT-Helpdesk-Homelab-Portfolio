# 🔑 Active Directory Credentials

This file contains the access details and credentials for the Domain Controller and clients of the `homelab.local` domain.

---

## 🖥️ Domain Controller (SRV-DC01)
*   **Role:** Domain Controller (AD DS, DNS)
*   **Static IP:** `192.168.10.10`
*   **Username:** `Administrator` (or `homelab\Administrator`)
*   **Password:** `Admin2026!`

---

## 💻 Windows 11 Client (CLI-W11)
Below are the test users configured in their respective Organizational Units (OUs):

### 🏢 Organizational Unit: `HR`
*   **Full Name:** Mario Rossi
*   **Username:** `mrossi` (or `mrossi@homelab.local`)
*   **Password:** `HomelabHR2026!`
*   **Security Group:** `HR_Docs_Access`
*   **Mapped Network Drive (Z:):** `\\192.168.10.10\HR_Docs`

### 🏢 Organizational Unit: `Sales`
*   **Full Name:** Giulia Bianchi
*   **Username:** `gbianchi` (or `gbianchi@homelab.local`)
*   **Password:** `HomelabSales2026!`

---

> [!WARNING]
> These credentials are for educational purposes and intended exclusively for the isolated Homelab laboratory environment. Do not reuse these passwords in production environments.
