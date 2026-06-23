# 🧠 Archivio di Memoria Progetto — Helpdesk IT Homelab

Questo documento racchiude e consolida l'intera storia del progetto, il profilo professionale del candidato, le strategie di carriera concordate, i dettagli tecnici dei laboratori (configurati e pianificati) e la cronologia delle sessioni di chat. Sostituisce i vecchi file storici per fungere da unica sorgente di verità del repository.

---

## 👤 Profilo del Candidato & Obiettivo Professionale
* **Candidato:** Studente al primo anno di Informatica presso l'Università degli Studi di Parma (con piano di trasferimento a Milano — UniMi o Bicocca).
* **Obiettivo Geografico:** Milano e hinterland.
* **Ruolo Target:** Help Desk Tier 1 / Service Desk / IT Support Specialist.
* **Certificazioni:** Google IT Support Professional Certificate (in corso/completamento), CompTIA A+ (Core 1 e Core 2 in pianificazione), AZ-900 (Cloud Literacy in pianificazione).
* **Vincoli Tecnici:** Assenza di carta di credito (AWS escluso). Utilizzo esclusivo di **VirtualBox** per homelab locale ed eventualmente **Azure Free Tier** per test cloud.
* **Competenze Chiave:** Amministrazione Windows (avanzata), networking base, hardware troubleshooting, assistenza informale (realizzata per amici e familiari, da valorizzare nel CV ATS).

---

## 🏗️ Struttura del Portfolio e Repository
Il repository è organizzato per essere pronto alla pubblicazione su GitHub come portfolio tecnico:

```
.helpdesk/
├── active-directory/
│   ├── README.md             # Write-up del Lab AD (Inglese)
│   ├── README_IT.md          # Write-up del Lab AD (Italiano)
│   └── screenshots/          # 7 screenshot del Lab AD (configurazioni e test)
├── osticket/
│   ├── README.md             # Documentazione Lab osTicket (Inglese - In corso)
│   └── README_IT.md          # Documentazione Lab osTicket (Italiano - In corso)
├── README.md                 # Intro Portfolio / Executive Summary (Inglese)
└── README_IT.md              # Intro Portfolio / Executive Summary (Italiano)
```

---

## 🛠️ Lab 1 — Active Directory & Group Policies (Completato)

### Ambiente ed Infrastruttura Reale
* **Hypervisor:** Oracle VirtualBox 7.x
* **Rete:** Host-only / Rete Interna isolata sulla subnet `192.168.10.0/24`.
* **Domain Controller (DC-01):**
  * Windows Server 2022 Standard Evaluation (IP statico: `192.168.10.10`, RAM: 6 GB).
  * Ruoli installati: AD DS (Active Directory Domain Services) + Server DNS.
  * Nome Dominio: `homelab.local`.
* **Client (CLI-W11):**
  * Windows 11 Enterprise Evaluation (IP statico/riservato: `192.168.10.20`).
  * DNS primario: puntato a `192.168.10.10`.

### Configurazione Logica Active Directory
1. **Unità Organizzative (OU):**
   * Creata OU madre `Azienda-HQ`, con sub-OU `IT`, `Sales` e `HR`.
2. **Utenti & Gruppi:**
   * Creato utente `Mario Rossi` (mrossi) nella OU `HR` (con opzione password change al login).
   * Creato utente `Giulia Bianchi` (gbianchi) nella OU `Sales`.
   * Creato gruppo di sicurezza globale `HR_Docs_Access` nella OU `HR`, con `mrossi` come membro.
3. **Cartella Condivisa e Permessi:**
   * Creata cartella `HR_Docs` sul DC.
   * Disabilitata l'ereditarietà NTFS e assegnati permessi di Modifica esclusivamente al gruppo `HR_Docs_Access`.
4. **Group Policy Objects (GPO):**
   * GPO **Map_Drive_HR** (collegata alla OU HR): Mappa l'unità `Z:` al percorso `\\192.168.10.10\HR_Docs` al login utente.
   * GPO **Restrict_Control_Panel** (collegata a livello dominio/OU): Blocca l'accesso al Pannello di Controllo e alle impostazioni del PC per evitare manomissioni.

### Log di Troubleshooting del Laboratorio
* **Problema di Join a Dominio:** Il client non contattava il controller di dominio.
  * *Causa:* La scheda di rete virtuale otteneva il DNS in DHCP dal router dell'host anziché dal DC.
  * *Soluzione:* Configurato manualmente l'IP DNS del client su `192.168.10.10` ed eseguito `ipconfig /flushdns`.
* **GPO Mappatura Unità Invisibile:** Il disco Z: non compariva su CLI-W11.
  * *Causa:* Azione impostata su "Crea" anziché "Aggiorna" nella GPO e permessi di condivisione di rete (tab Sharing) non configurati.
  * *Soluzione:* Modificata l'azione in "Aggiorna", configurati i permessi di Sharing inserendo il gruppo `HR_Docs_Access` ed eseguito `gpupdate /force` sul client.

---

## ⏳ Lab 2 — osTicket & LAMP Stack (In Corso - Setup Completato)

### Infrastruttura Configurata
* **Server Ticketing (osticket-srv):** Ubuntu Server 22.04 LTS (NIC 1 NAT: `10.0.2.15` per Internet, NIC 2 Rete Interna: `192.168.10.30` per Homelab).
* **Software Stack (LAMP):** Apache2, MySQL Server, PHP v8.2.3 (aggiornato da 8.1 per compatibilità con osTicket v1.18.3, installato modulo `php8.2-ldap`).

### Flusso di Installazione e Configurazione
1. **Configurazione Rete e SSH:** Configurazione dell'IP statico `192.168.10.30` sulla NIC 2 (Rete Interna) e attivazione del servizio SSH. Configurazione dell'Inoltro Porte in VirtualBox (porta host `2222` -> porta guest `22` dell'IP NAT) per consentire la gestione del server tramite Windows Terminal.
2. **Database:** Creazione DB `osticket_db`, utente `osticket_user` con password `Password123!`.
3. **Deploy Files & Setup Web:** Deploy dei file di osTicket in `/var/www/html/osticket`, configurazione dei permessi per l'utente `www-data` e completamento con successo dell'installatore web da browser. Configurato l'utente amministratore `admin_desk` con password `DeskAdmin2026!` ed email `admin@homelab.local`. Conclusa l'installazione, i permessi di `ost-config.php` sono stati reimpostati in sola lettura e la cartella `/setup` è stata rimossa per hardening.
4. **Simulazione Ticket ITIL (Risolto & Documentato):**
   * **Ticket 1 (Rete - Completato):** Giulia Bianchi (Sales) - "Impossibile navigare su Internet". Risolto tramite diagnostica sulla postazione `CLI-W11` (`ipconfig /release` e `/renew` per ripristinare l'IP ed `nslookup` per verificare la risoluzione DNS). Questo scenario simula un troubleshooting tipico di Tier 1 ed è l'unico documentato attivamente nel portfolio.
5. **Integrazione Avanzata (Completata):** Configurazione con successo del plugin LDAP per consentire agli agenti e agli utenti del dominio `homelab.local` di accedere ad osTicket con credenziali AD.
   * *Troubleshooting eseguito:* 
     * Corretto il DNS di `osticket-srv` configurando `nameservers` su Netplan per puntare al DC (`192.168.10.10`).
     * Risolto il loop di login "Session timed out" applicando una patch a `/var/www/html/osticket/include/class.ostsession.php` alla riga 263 (forzando il cookie `SameSite` a `Lax` invece di `None` per le connessioni HTTP).
     * Risolto l'errore "Access Denied" per l'utente `mrossi` sbloccando l'account e rimuovendo l'obbligo di cambio password al primo accesso sul Domain Controller. Conclusa la configurazione dell'istanza LDAP usando `homelab\mrossi` come utente di ricerca.

---

## 📈 Strategia di Candidatura & Carriera (Dal Coach)

### 1. Curriculum Vitae & LinkedIn
* **CV ATS-Friendly:** Formato plain text strutturato per il mercato italiano. Valorizzare l'assistenza IT informale (amici e parenti) descrivendola con termini tecnici (es. "Supporto tecnico on-demand per sistemi Windows/macOS, configurazione router Soho e rimozione malware") anziché creare ruoli inventati.
* **LinkedIn Profile:** Bio con parole chiave ATS (Help Desk, Supporto Tecnico, Active Directory, Troubleshooting, Windows Server).
* **Certificazioni in Corso:** Inserire la Google IT Support Certificate con dicitura *"In corso - Completamento stimato: [Mese/Anno]"* per massimizzare la visibilità delle keyword senza mentire.

### 2. Canali di Ricerca a Milano
* **MSP (Managed Service Provider):** Target principale per fare esperienza rapida. Candidature spontanee ai responsabili tecnici.
* **Agenzie per il Lavoro (Milano):** Experis, Randstad IT, Hays, Gi Group, Manpower. Contattare direttamente i recruiter IT di queste agenzie su LinkedIn.
* **Job Alert:** Impostare parole chiave esatte (Help Desk, Service Desk, Supporto IT, Tecnico Informatico junior).

### 3. Preparazione al Colloquio (STAR Method)
* Strutturare le risposte sulla risoluzione di problemi reali del laboratorio (es. come è stato risolto il problema di join a dominio o della GPO) per dimostrare attitudine tecnica e problem-solving pratico durante le interviste tecniche.
