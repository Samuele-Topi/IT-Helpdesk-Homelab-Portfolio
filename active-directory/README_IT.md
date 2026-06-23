# 🏢 Infrastruttura Windows Server & Active Directory Homelab

## 🎯 Obiettivo del Progetto
L'obiettivo di questo laboratorio è progettare, implementare e mettere in sicurezza un ambiente di rete aziendale simulato per una Piccola-Media Impresa (PMI). Questo laboratorio dimostra le competenze pratiche di amministrazione dei sistemi IT, tra cui la configurazione di un Domain Controller, la strutturazione di una directory centralizzata, il provisioning degli utenti, il controllo degli accessi tramite RBAC (Role-Based Access Control), l'automazione tramite Group Policy (GPO) e la risoluzione di problemi di rete (troubleshooting).

---

## 🏗️ Architettura del Lab
Il laboratorio è interamente virtualizzato in locale tramite Oracle VirtualBox, utilizzando uno schema di rete host-only o interno (`192.168.10.0/24`) per garantire comunicazioni isolate e sicure.

```
                  ┌──────────────────────────────┐
                  │          PC Host             │
                  │   (Oracle VirtualBox 7.x)    │
                  └──────────────┬───────────────┘
                                 │
                   [ Rete VM: 192.168.10.0/24 ]
                                 │
         ┌───────────────────────┴───────────────────────┐
         │                                               │
┌────────┴────────┐                             ┌────────┴────────┐
│  Domain Controller                             │  Macchina Client │
│      [DC-01]    │                             │    [CLI-W11]    │
│                 │                             │                 │
│ Win Server 2022 │                             │ Windows 11 Ent  │
│ IP: 192.168.10.10│                            │ IP: 192.168.10.20│
│ AD DS, DNS      │                             │ DNS: 192.168.10.10│
└─────────────────┘                             └─────────────────┘
```

### Specifiche delle Macchine Virtuali
* **Hypervisor:** Oracle VirtualBox 7.x
* **Domain Controller (DC-01):**
  * **OS:** Windows Server 2022 Standard Evaluation (Trial di 180 giorni)
  * **Risorse:** 2 vCPU, 6 GB RAM, 50 GB HDD
  * **Indirizzo IP:** `192.168.10.10` (Statico)
  * **Subnet Mask:** `255.255.255.0`
  * **Gateway:** `192.168.10.1` (o gateway NAT/router locale)
  * **Server DNS:** `127.0.0.1` (punta a se stesso)
  * **Ruoli Attivi:** AD DS (Active Directory Domain Services) e Server DNS
  * **Nome Dominio:** `homelab.local`
* **Client Workstation (CLI-W11):**
  * **OS:** Windows 11 Enterprise Evaluation
  * **Risorse:** 2 vCPU, 4 GB RAM, 60 GB HDD
  * **Indirizzo IP:** `192.168.10.20` (Statico/DHCP riservato)
  * **Server DNS:** `192.168.10.10` (punta al Domain Controller)

---

## 🛠️ Fasi di Configurazione e Implementazione

### Fase 1: Configurazione di Rete e Promozione a DC
1. Installato Windows Server 2022 e configurato un indirizzo IP statico (`192.168.10.10`) sulla scheda di rete principale.
2. Installati i ruoli Servizi di dominio Active Directory (AD DS) e Server DNS tramite il Server Manager.
3. Promosso il server a Domain Controller, creando una nuova foresta con dominio `homelab.local`.
4. Verificato il corretto avvio del DNS e la creazione automatica dei record host locali.
*Screenshot di riferimento:* [Server Manager Setup](screenshots/1_Server_IP.png)

### Fase 2: Progettazione Logica di Active Directory (OU, Utenti e Gruppi)
È stata definita una struttura a Unità Organizzative (OU) che rispecchia una reale gerarchia aziendale. Tutti gli oggetti creati sono posizionati sotto un'OU madre chiamata `Azienda-HQ` per evitare la frammentazione nei container predefiniti di Windows.

```
homelab.local (Radice Dominio)
└── Azienda-HQ (OU Principale)
    ├── IT (Sub-OU)
    ├── Sales (Sub-OU)
    └── HR (Sub-OU)
        ├── Mario Rossi (Utente: mrossi)
        └── Gruppi di Sicurezza:
            └── HR_Docs_Access (Gruppo di Sicurezza Globale)
```

1. **Creazione Utenti:**
   * **Mario Rossi** (mrossi) nella sub-OU HR. L'account è stato impostato con l'obbligo di cambiare la password al primo accesso.
2. **Controllo Accessi (RBAC):**
   * Creato il gruppo di sicurezza globale `HR_Docs_Access` nella sub-OU HR.
   * Aggiunto l'utente `Mario Rossi` come membro di questo gruppo.
*Screenshot di riferimento:* [Proprietà Utente e Membership](screenshots/4_Member_Of.png)

### Fase 3: Cartella Condivisa e Permessi NTFS
Per simulare un file server aziendale:
1. Creata sul Domain Controller una cartella di condivisione `HR_Docs`.
2. Abilitata la condivisione avanzata sulla cartella.
3. Applicati **permessi di sicurezza NTFS** restrittivi:
   * Disabilitata l'ereditarietà per escludere gli utenti generici del dominio.
   * Assegnati permessi di **Modifica (Scrittura/Lettura)** esclusivamente al gruppo `HR_Docs_Access`.
   * Questo garantisce che solo i membri del reparto HR autorizzati possano accedere a questi file.

### Fase 4: Automazione con GPO (Group Policy Objects)
Sono state implementate due policy per automatizzare e proteggere le postazioni degli utenti:

| Nome GPO | Collegata A | Percorso e Funzione |
| :--- | :--- | :--- |
| **Map_Drive_HR** | OU `HR` | `Configurazione utente -> Preferenze -> Impostazioni di Windows -> Mapping unità`. Mappa il percorso di rete `\\192.168.10.10\HR_Docs` sull'unità `Z:` al login di qualsiasi utente HR. |
| **Restrict_Control_Panel** | (Livello dominio / OU HR / Sales) | `Configurazione utente -> Criteri -> Modelli amministrativi -> Pannello di controllo`. Abilita la restrizione "Proibisci l'accesso al Pannello di controllo e alle impostazioni del PC" per evitare modifiche non autorizzate. |

*Screenshot di riferimento:*
* [Collegamento GPO su OU](screenshots/5_GPO.png)
* [Configurazione Mapping Disco GPO](screenshots/5_1_ALL_GPO.png)

### Fase 5: Integration Client e Verifica dei Criteri
1. Installato il sistema operativo Windows 11 sulla macchina client e configurata la scheda di rete.
2. Configurate le proprietà IPv4 del client impostando l'IP `192.168.10.20` e come DNS primario l'IP del Domain Controller (`192.168.10.10`).
3. Effettuato il join della workstation al dominio `homelab.local` utilizzando le credenziali di Domain Admin.
*Screenshot di riferimento:*
* [Risoluzione DNS e Ping dal Client](screenshots/3_IPConfig_Ping.png)
* [Client Joinato in Active Directory](screenshots/2_CLI_Joined.png)

#### Esito dei Test sul Client:
* Accesso eseguito come **Mario Rossi**:
  * Verificato che il client richiede il cambio password obbligatorio al primo avvio.
  * In Esplora File, confermata la presenza dell'unità di rete `Z:` mappata automaticamente su `\\192.168.10.10\HR_Docs`. Verificati i permessi di scrittura creando con successo file al suo interno.
* Hardening e Postazione Client:
  * Tentato l'accesso al Pannello di Controllo. Il sistema ha bloccato l'operazione mostrando la finestra di avviso GPO prevista.
*Screenshot di riferimento:* [Finestra Blocco Pannello di Controllo](screenshots/6_ControlPanel_Block.png)

---

## 🔍 Log di Troubleshooting (Risoluzione Problemi)

Di seguito sono dettagliati i principali problemi tecnici riscontrati durante il laboratorio e le relative soluzioni:

### 1. Impossibile joinare il client al dominio
* **Sintomo:** Errore "Impossibile contattare un controller di dominio Active Directory" durante il tentativo di join dal client.
* **Diagnostica:** Eseguito `nslookup homelab.local` sul prompt dei comandi del client. Il comando non riusciva a risolvere l'IP, indicando che il client non stava interrogando il server DNS locale.
* **Causa:** La scheda di rete del client otteneva i server DNS in automatico (DHCP) dal gateway di VirtualBox, invece di puntare al Domain Controller.
* **Risoluzione:** Aperto il Centro connessioni di rete sul client, modificate le proprietà di IPv4 e inserito manualmente l'IP del DC (`192.168.10.10`) come server DNS. Eseguito `ipconfig /flushdns` e riprovato il join. La macchina si è unita al dominio immediatamente.

### 2. L'unità mappata GPO (Disco Z:) non appare sul client
* **Sintomo:** L'unità `Z:` non compare in Esplora File dopo l'accesso dell'utente Mario Rossi.
* **Diagnostica:** Eseguito `gpresult /r` sul prompt dei comandi del client. La policy `Map_Drive_HR` risultava applicata correttamente nell'elenco degli oggetti utente, ma l'unità era assente.
* **Causa:** La GPO era configurata con azione "Crea" (Create) ma necessitava di "Aggiorna" (Update) per forzare l'applicazione immediata al login. Inoltre, i permessi di condivisione di rete (tab Sharing) bloccavano l'accesso prima dei permessi NTFS (tab Security).
* **Risoluzione:** Modificata l'azione della GPO in "Aggiorna" nell'Editor Gestione Criteri di Gruppo su `DC-01`. Aggiunto il gruppo `HR_Docs_Access` anche nella scheda dei permessi di Condivisione (oltre a Sicurezza) con controllo di modifica. Eseguito `gpupdate /force` sul client e riavviato. L'unità è apparsa correttamente.

---

## 🛠️ Competenze Tecniche Sviluppate
* **Servizi di dominio Active Directory (AD DS):** Progettazione gerarchica, gestione utenti/gruppi, provisioning ruoli.
* **Gestione Criteri di Gruppo (GPO):** Automazione delle configurazioni, hardening delle postazioni client, scriptless mapping.
* **Amministrazione di Rete:** Gestione DNS in AD, troubleshooting IP e conflitti di routing su schede di rete virtuali.
* **Access Control Lists (Permessi NTFS e Condivisione):** Implementazione di politiche di sicurezza a "privilegio minimo" (Role-Based Access Control).
