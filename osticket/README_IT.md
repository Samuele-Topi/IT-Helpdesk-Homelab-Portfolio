# 🎫 Lab osTicket: Gestione dei Processi di Service Desk

## 🎯 Obiettivo del Progetto
Questo laboratorio ha l'obiettivo di simulare un workflow professionale di IT Helpdesk / Service Desk all'interno di un'azienda. L'attività prevede il deploy di una piattaforma di ticketing open source (osTicket) su server Linux, la modellazione dei processi standard ITIL (creazione, categorizzazione, assegnazione di SLA, risoluzione e chiusura dei ticket) e, in una fase avanzata, l'integrazione LDAP con Active Directory per consentire il login centralizzato.

---

## 🏗️ Architettura Prevista
Il sistema risiede su una macchina virtuale Ubuntu Server. Poiché deve scaricare pacchetti da Internet (tramite la scheda NAT di VirtualBox) e contemporaneamente comunicare con il Domain Controller e il Client sulla rete interna (`192.168.10.0/24`), è configurata con **due schede di rete (Dual NIC)**.

```
                      [ Rete VM: 192.168.10.0/24 ]
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
┌────────┴────────┐       ┌────────┴────────┐       ┌────────┴────────┐
│  Domain Controller      │  Macchina Client │      │ Server Ticketing│
│      [DC-01]    │       │    [CLI-W11]    │       │  [osticket-srv] │
│                 │       │                 │       │                 │
│ Win Server 2022 │       │ Windows 11 Ent  │       │ Ubuntu 22.04 LTS│
│ IP: 192.168.10.10│      │ IP: 192.168.10.20│      │ NIC 1: 10.0.2.15│
│ AD DS, DNS      │       │ DNS: 192.168.10.10│     │ NIC 2: 192.168.10.30
└─────────────────┘       └─────────────────┘       └─────────────────┘
```

### Specifiche della Macchina Virtuale
* **Nome Host:** `osticket-srv`
* **OS:** Ubuntu Server 22.04 LTS (Long Term Support)
* **Risorse:** 1 vCPU, 2 GB RAM, 30 GB HDD
* **Schede di Rete:**
  * **NIC 1 (NAT):** `10.0.2.15/24` (Per l'accesso a Internet / installazione pacchetti LAMP)
  * **NIC 2 (Rete Interna - IP Statico):** `192.168.10.30/24` (Per le comunicazioni con il dominio locale e integrazione AD LDAP)
* **Stack Software (LAMP):**
  * **Web Server:** Apache2
  * **Database:** MySQL Server
  * **Linguaggio scripting:** PHP (con moduli: `php-mysql`, `php-gd`, `php-imap`, `php-xml`, `php-mbstring`, etc.)
  * **Applicativo:** osTicket (ultima versione stabile)

---

## 📋 Fasi di Installazione Previste

### Fase 1: Setup di Linux e Installazione dello Stack LAMP
1. Creazione della VM Ubuntu Server (`osticket-srv`) su VirtualBox, configurando la scheda NIC 1 su NAT e la NIC 2 sulla rete interna. Configura l'IP statico `192.168.10.30` sulla NIC 2.
2. **Configurazione dell'accesso SSH dall'host Windows (Opzionale):**
   * Assicurarsi che SSH sia attivo sulla VM: `sudo apt update && sudo apt install openssh-server -y`.
   * Configurare l'Inoltro Porte (Port Forwarding) in VirtualBox per la **NIC 1 (NAT)**:
     * Regola: Protocollo `TCP`, IP Host `127.0.0.1`, Porta Host `2222`, IP Guest *lasciato vuoto* (o `10.0.2.15`), Porta Guest `22`.
   * Collegamento dall'host tramite Windows Terminal (PowerShell):
     ```powershell
     ssh admin_srv@127.0.0.1 -p 2222
     ```
3. Aggiornamento dei repository e installazione dei pacchetti necessari:
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install apache2 mysql-server php php-mysql php-gd php-imap php-xml php-mbstring php-intl php-apcu unzip wget -y
   ```
4. Configurazione sicura del database MySQL, creando un utente dedicato e lo schema del database:
   ```sql
   CREATE DATABASE osticket_db;
   CREATE USER 'osticket_user'@'localhost' IDENTIFIED BY 'Password123!';
   GRANT ALL PRIVILEGES ON osticket_db.* TO 'osticket_user'@'localhost';
   FLUSH PRIVILEGES;
   ```

### Fase 2: Configurazione di osTicket
1. Download e scompattazione dei file sorgente di osTicket nella cartella root del server web Apache: `/var/www/html/osticket`.
2. Impostazione dei permessi e copia del file di configurazione di esempio:
   ```bash
   sudo chown -R www-data:www-data /var/www/html/osticket
   ```
3. Accesso all'installatore web da browser client (`http://192.168.10.30/osticket`) e inserimento delle credenziali del database e del profilo amministratore.
4. Configurazione dei Dipartimenti di supporto e delle regole di assegnazione automatica in base agli argomenti di aiuto (Help Topics).

![Configurazione dei Dipartimenti](screenshots/1_Department_Config.png)

### Fase 3: Simulazione e Risoluzione dei Ticket (Caso Reale)
Per dimostrare la gestione operativa dei ticket secondo le linee guida ITIL, è stato simulato, gestito e risolto un ticket reale all'interno della piattaforma:

#### **Scenario: Impossibile navigare su Internet (Utente: Giulia Bianchi - Ufficio Vendite)**
* **Apertura del Ticket:** L'utente ha aperto una segnalazione tramite il portale pubblico di osTicket selezionando l'argomento di aiuto **"Problemi di Rete / Connettività"**. Il sistema ha indirizzato automaticamente il ticket al dipartimento **Supporto IT** impostando la priorità su **Alta** (SLA Urgente: risoluzione entro 4 ore).
* **Presa in carico:** L'amministratore/agente (`admin_desk`) ha preso in carico il ticket tramite la console di controllo (`/scp`).
* **Diagnostica e Troubleshooting (Postazione Client CLI-W11):**
  1. È stata aperta la PowerShell sulla macchina client e verificata la configurazione di rete:
     ```powershell
     ipconfig /all
     ```
     *Diagnosi:* Il client non presentava un IP valido assegnato dal DHCP (generando un indirizzo di autoconfigurazione APIPA `169.254.x.x`).
  2. È stato forzato il rilascio ed il rinnovo della richiesta DHCP:
     ```powershell
     ipconfig /release
     ipconfig /renew
     ```
  3. È stata verificata la connettività locale e la risoluzione DNS dei domini esterni:
     ```powershell
     ping 192.168.10.10   # Test connettività locale verso il DC
     nslookup google.com  # Verifica risoluzione DNS esterna
     ```
* **Chiusura del Ticket:** L'agente ha risposto all'utente tramite osTicket dettagliando le operazioni eseguite e ha impostato lo stato del ticket su **Chiuso/Risolto**.

### Fase 4: Integrazione LDAP con Active Directory (Avanzato)
* Indirizzo del server LDAP: `192.168.10.10`.
* Installazione del plugin LDAP per osTicket e configurazione dei parametri di collegamento con il Domain Controller (`homelab.local`).

![Configurazione del Plugin LDAP](screenshots/3_LDAP_Config.png)

* Mappatura degli agenti e degli utenti sui relativi Gruppi di Sicurezza di Active Directory. Per far accedere un agente con le credenziali di dominio, l'account viene creato in osTicket e associato al backend di autenticazione LDAP.

![Associazione dell'Agente al Backend LDAP](screenshots/2_Agent_Page.png)

* Verifica del login centralizzato SSO (Single Sign-On) — **Completata con successo per l'utente `mrossi`**.

#### 🔍 Log di Troubleshooting del Laboratorio LDAP & Sessioni
Durante l'integrazione LDAP sono stati riscontrati e risolti i seguenti problemi critici:

1. **Risoluzione DNS di `homelab.local` Fallita (SERVFAIL):**
   * *Causa:* Il server Ubuntu chiedeva ai DNS pubblici della scheda NAT (`enp0s3`) anziché al Domain Controller (`192.168.10.10`).
   * *Soluzione:* Modificato il file di configurazione Netplan `/etc/netplan/50-cloud-init.yaml` per aggiungere i `nameservers` sulla scheda interna `enp0s8`:
     ```yaml
     enp0s8:
      dhcp4: no
      addresses:
       - 192.168.10.30/24
      nameservers:
       addresses:
        - 192.168.10.10
       search:
        - homelab.local
     ```
     Applicato con `sudo netplan apply`.

2. **Loop di Login "Session timed out due to inactivity" (HTTP/SameSite Bug):**
   * *Causa:* Con la configurazione Iframe attiva, osTicket imposta il cookie di sessione con `SameSite=None`. Sui browser moderni in HTTP (senza HTTPS e flag `Secure`), il cookie viene scartato ad ogni richiesta, disconnettendo l'utente immediatamente dopo il login.
   * *Soluzione:* Modificato il file `/var/www/html/osticket/include/class.ostsession.php` alla riga 263 per forzare `Lax` invece di `None` in ambienti HTTP:
     ```php
     // Sostituito 'None' con 'Lax'
     'samesite' => !empty($ost->getConfig()->getAllowIframes()) ? 'Lax' : 'Strict'
     ```
     Riavviato Apache con `sudo systemctl restart apache2` e svuotata la tabella sessioni con `TRUNCATE TABLE ost_session;`.

3. **Errore "Access Denied" al Login di Mario Rossi (`mrossi`):**
   * *Causa:* L'account in Active Directory aveva attiva l'opzione "L'utente deve cambiare la password al prossimo accesso", bloccando il Bind LDAP. Inoltre, le credenziali del "Search User" impostate nel plugin LDAP erano errate (usava il gruppo `Administrators` invece dell'utente singolo).
   * *Soluzione:* Disabilitata l'opzione di cambio password sul Domain Controller (tramite console ADUC o PowerShell `Set-ADUser -Identity mrossi -ChangePasswordAtLogon $false`) e impostata una password nota (`HomelabHR2026!`). Configurata l'istanza LDAP su osTicket usando `homelab\mrossi` come utente di ricerca funzionante.

---

## 🛠️ Competenze Tecniche da Sviluppare
* **Amministrazione Sistemi Linux:** Navigazione da riga di comando (CLI), gestione pacchetti, configurazione di rete e permessi del file system.
* **Configurazione Stack LAMP:** Configurazione virtual host web, gestione database relazionali e moduli PHP.
* **Processi di Service Desk:** Allineamento alle linee guida ITIL, categorizzazione degli incidenti, monitoraggio dei tempi di risposta (SLA) e documentazione tecnica delle soluzioni.
