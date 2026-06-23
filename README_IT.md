# 🛠️ Portfolio di IT Helpdesk & Systems Administration

Benvenuto nel mio portfolio tecnico. Questo repository documenta una serie di laboratori pratici (homelab) progettati per simulare ambienti aziendali reali di supporto IT e amministrazione sistemi.

L'obiettivo principale di questo progetto è dimostrare la mia capacità di implementare, configurare, proteggere e risolvere problemi relativi a sistemi aziendali (Windows Server, Active Directory, GPO) e operazioni di helpdesk (gestione ticket con osTicket, amministrazione server Linux).

---

## 🏗️ Struttura del Portfolio

Il portfolio è suddiviso in due laboratori principali, allineati con le competenze fondamentali richieste per ruoli IT Support Tier 1/2:

### 1. [Lab Active Directory & Group Policies](active-directory/README_IT.md)
* **Stato:** Completato
* **Tecnologie Chiave:** Windows Server 2022, Windows 10/11 Enterprise, VirtualBox, Active Directory DS, Group Policy (GPO), DNS.
* **Scenari Risolti:** Join al dominio, provisioning utenti (RBAC), implementazione GPO (mappatura dischi di rete, restrizioni Pannello di Controllo), troubleshooting di rete.
* **Documentazione:** [Leggi il write-up in Italiano](active-directory/README_IT.md) | [Read in English](active-directory/README.md)

### 2. [Lab osTicket (Ticketing System)](osticket/README_IT.md)
* **Stato:** Completato (Integrazione AD completata)
* **Tecnologie Chiave:** Ubuntu Server 22.04 LTS, Apache2, MySQL, PHP v8.2, osTicket.
* **Scenari Risolti:** Deploy dello stack LAMP (Apache/MySQL/PHP), installazione ed hardening di osTicket, creazione dipartimenti e SLA, simulazione e risoluzione di un ticket ITIL reale di connettività di rete, e integrazione avanzata LDAP con Active Directory.
* **Documentazione:** [Leggi la documentazione in Italiano](osticket/README_IT.md) | [Read in English](osticket/README.md)

---

## 🛠️ Competenze Tecniche Dimostrate

| Competenza | Applicazione Pratica |
| :--- | :--- |
| **Active Directory DS** | Progettazione struttura OU, gestione utenti/gruppi, controllo degli accessi basato sui ruoli (RBAC). |
| **Amministrazione Windows** | Installazione Windows Server 2022, gestione della sicurezza locale, analisi dei log tramite Visualizzatore Eventi. |
| **Group Policy (GPO)** | Configurazione requisiti minimi password a livello di dominio, mappatura automatica dischi, hardening postazioni client. |
| **Servizi di Rete** | Configurazione DNS per il join a dominio, integrazione DHCP, diagnostica di rete (`ipconfig`, `ping`, `tracert`). |
| **Linux & Stack LAMP** | Deploy di Ubuntu Server, configurazione server web Apache, gestione database MySQL e moduli PHP. |
| **Processi ITIL / Helpdesk** | Ciclo di vita dei ticket, comunicazione con gli utenti, rispetto degli SLA, documentazione delle soluzioni. |

---

## 📂 Contenuto del Repository

* `active-directory/` - Documentazione, schema di rete, credenziali, screenshot e log relativi al lab Active Directory.
* `osticket/` - Documentazione, credenziali, configurazioni e screenshot relativi al lab osTicket.
* `GEMINI.md` - File di memoria e storico delle decisioni/troubleshooting del progetto.

---

## 📬 Contatti e Profilo

* **Nome:** [Il tuo Nome]
* **Località:** Parma / Milano, Italy
* **Formazione:** Studente di Informatica (Università degli Studi di Parma)
* **Certificazioni:** Google IT Support Professional Certificate (In Corso)
* **LinkedIn:** [Link Profilo]
* **GitHub:** [Link GitHub]
