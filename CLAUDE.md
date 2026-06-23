# CLAUDE.md — Contesto del Progetto per Claude Code

> **Istruzione per Claude:** All'avvio di ogni sessione in questo progetto, leggi questo file per intero prima di fare qualsiasi cosa. Alla fine di ogni sessione (o quando vengono aggiunti lab/contenuti nuovi), aggiorna questo file per riflettere lo stato attuale. Il file va mantenuto preciso e compatto — non aggiungere sezioni che non corrispondono a qualcosa di reale nel repository.

---

## Identità del progetto

Portfolio tecnico IT / Homelab documentato, usato come resume per posizioni IT Support Tier 1/2 e Sysadmin.
- **Autore:** Samuele Topi
- **Università:** Informatica, Università degli Studi di Parma
- **Certificazioni:** Google IT Support Professional Certificate (in corso)
- **Lingua:** documenti EN + IT in parallelo (`README.md` + `README_IT.md` per ogni sezione)
- **Hypervisor:** Oracle VirtualBox 7.x, host Windows

---

## Struttura repository

```
C:\.github\.helpdesk\
├── CLAUDE.md                  ← questo file
├── README.md                  ← Portfolio root (EN)
├── README_IT.md               ← Portfolio root (IT)
├── Screenshot.png             ← screenshot generico (uso da chiarire)
├── Memory.md                  ← note di pianificazione miste (da altri agent AI)
├── active-directory\
│   ├── README.md              ← writeup lab AD (EN) — COMPLETO
│   ├── README_IT.md           ← writeup lab AD (IT) — COMPLETO
│   ├── Login_Details.md       ← credenziali VM (NON pubblicare su repo pubblico)
│   └── screenshots\           ← 7 screenshot presenti
│       ├── 1_Server_IP.png
│       ├── 2_CLI_Joined.png
│       ├── 3_IPConfig_Ping.png
│       ├── 4_Member_Of.png
│       ├── 5_GPO.png
│       ├── 5_1_ALL_GPO.png
│       └── 6_ControlPanel_Block.png
└── osticket\
    ├── README.md              ← writeup lab osTicket (EN) — struttura completa, screenshot mancanti
    ├── README_IT.md           ← writeup lab osTicket (IT)
    └── Login_Details.md       ← credenziali VM osTicket (NON pubblicare su repo pubblico)
```

---

## Lab 1 — Active Directory & GPO

**Stato: COMPLETATO**

### Ambiente
| Componente | Dettaglio |
|---|---|
| Dominio | `homelab.local` |
| DC | `DC-01` — Windows Server 2022 Standard Eval |
| IP DC | `192.168.10.10` (statico) |
| Ruoli DC | AD DS + DNS |
| Client | `CLI-W11` — Windows 11 Enterprise Eval |
| IP Client | `192.168.10.20` |
| DNS Client | `192.168.10.10` |

### Struttura AD
```
homelab.local
└── Azienda-HQ
    ├── IT
    ├── Sales
    │   └── Giulia Bianchi (gbianchi)
    └── HR
        ├── Mario Rossi (mrossi)
        └── HR_Docs_Access (Security Group globale)
```

### GPO configurate
| Nome | Collegata a | Funzione |
|---|---|---|
| `Map_Drive_HR` | OU HR | Mappa `Z:` → `\\192.168.10.10\HR_Docs` |
| `Restrict_Control_Panel` | OU HR / Sales / dominio | Blocca accesso Pannello di Controllo |

### Screenshot presenti (7)
1. `1_Server_IP.png` — Server Manager, IP statico e ruoli DC-01
2. `2_CLI_Joined.png` — ADUC, CLI-W11 joinato al dominio
3. `3_IPConfig_Ping.png` — CMD client: DNS resolution + ping OK
4. `4_Member_Of.png` — Mario Rossi > tab Member Of
5. `5_GPO.png` — GPO Management: Map_Drive_HR collegata a HR OU
6. `5_1_ALL_GPO.png` — Editor GPO: Drive Maps, Z: → path
7. `6_ControlPanel_Block.png` — Finestra blocco GPO su client

### Screenshot ancora mancanti (non critici)
- Cartella `HR_Docs` condivisa visibile sul server
- Drive `Z:` mappato in Esplora File sul client (login come mrossi)
- Creazione utente Mario Rossi (User creation wizard)

---

## Lab 2 — osTicket Ticketing System

**Stato: INSTALLATO E CONFIGURATO — screenshot mancanti — Phase 4 (LDAP) da fare**

### Ambiente VM
| Componente | Dettaglio |
|---|---|
| Hostname | `osticket-srv` |
| OS | Ubuntu Server 22.04 LTS |
| RAM | 2 GB, 1 vCPU, 30 GB HDD |
| NIC 1 (NAT) | `10.0.2.15/24` — accesso internet |
| NIC 2 (Internal) | `192.168.10.30/24` — rete lab |
| Stack | Apache2 + MySQL + PHP 8.2 + osTicket |

### Accesso
| Tipo | Dettaglio |
|---|---|
| SSH (host) | `ssh admin_srv@127.0.0.1 -p 2222` |
| Web (interno) | `http://192.168.10.30/osticket` |
| Web (host) | `http://127.0.0.1:8080/osticket` |
| Admin panel | `/scp` — `admin_desk` |

### Scenario ticket documentato
- **Utente:** Giulia Bianchi (Sales) — `gbianchi`
- **Problema:** Nessuna connettività internet (APIPA `169.254.x.x`)
- **Risoluzione:** `ipconfig /release` + `ipconfig /renew` sul client CLI-W11
- **SLA:** High / 4h — ticket chiuso come Resolved

### Fasi
- [x] Phase 1 — Ubuntu Server + LAMP stack
- [x] Phase 2 — osTicket installato e configurato
- [x] Phase 3 — Ticket Giulia Bianchi simulato e risolto
- [ ] Phase 4 — LDAP / AD integration (da eseguire)

### Screenshot mancanti (necessari per completare il writeup)
- Interfaccia web osTicket (homepage utente e login)
- Ticket di Giulia Bianchi nel pannello SCP
- Configurazione dipartimenti / SLA nel pannello admin
- Eventuale integrazione LDAP (se eseguita)

---

## Checklist prima del push su GitHub pubblico

- [ ] Compilare placeholder in `README.md` e `README_IT.md` root: nome, LinkedIn, GitHub URL
- [ ] Aggiungere screenshot mancanti osTicket
- [ ] Valutare rimozione o `.gitignore` per i file `Login_Details.md` (contengono credenziali in chiaro)
- [ ] Aggiungere `Screenshot.png` nella sezione giusta o rimuoverlo dalla root
- [ ] Inizializzare `git init` e creare repo GitHub
- [ ] Eseguire Phase 4 LDAP (opzionale ma valorizzante)

---

## Istruzioni di aggiornamento per Claude

Quando l'utente aggiunge nuovi lab, screenshot, sezioni o compie azioni di configurazione:
1. Aggiorna la sezione corrispondente in questo file (stato, screenshot presenti, fasi completate)
2. Se viene aggiunto un nuovo lab, aggiungi una sezione `## Lab N — Nome` con la stessa struttura
3. Aggiorna la checklist "prima del push" se qualcosa viene completato
4. Mantieni il file preciso e sintetico — niente sezioni vuote o future ipotetiche non ancora iniziate
5. Aggiorna anche la memoria in `C:\Users\lavat\.claude\projects\C---github--helpdesk\memory\project_helpdesk_portfolio.md` se lo stato cambia significativamente
