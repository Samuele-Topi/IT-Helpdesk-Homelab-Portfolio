# 🔑 osTicket Server Login Details

This file contains the configuration and credentials for the osTicket Linux virtual machine.

---

## 🖥️ Virtual Machine Specifications
* **Operating System:** Ubuntu Server 22.04 LTS
* **Server Hostname:** `osticket-srv`
* **Admin Name:** Samuele Topi

## 🌐 Network Interfaces
* **Interface 1 (NAT):** `10.0.2.15/24` (Used for internet access to download software packages)
* **Interface 2 (Internal Network - Planned):** `192.168.10.30/24` (For communication with `homelab.local` domain)

## 👤 Credentials
* **SSH/System Username:** `admin_srv`
* **SSH/System Password:** `osticketpass0`

## 🗄️ Database Credentials
* **Database Name:** `osticket_db`
* **Database User:** `osticket_user`
* **Database Password:** `Password123!`

## 🌐 osTicket Web Configuration
* **Access URL (Internal Net):** `http://192.168.10.30/osticket`
* **Access URL (Host Forwarded):** `http://127.0.0.1:8080/osticket`
* **Helpdesk Name:** `Homelab IT Helpdesk`
* **Default Email:** `support@homelab.local`

## 👤 osTicket Web Console Admin User
* **Name:** `Samuele Topi`
* **Email:** `admin@homelab.local`
* **Username:** `admin_desk`
* **Password:** `DeskAdmin2026!`
