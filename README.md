# 🧠 MANIFESTO ARCHITETTURALE: IL SECONDO CERVELLO DIGITALE

Questo documento descrive l'architettura completa del sistema "Memoria 2", un progetto di **Rapid Application Development (RAD)** basato su Python e n8n, destinato all'archiviazione e all'analisi ontologica di input personali.

---

## 1. STRUTTURA E OBIETTIVI (RAD vs INDUSTRIALE)

Il progetto è una catena di Microservizi disaccoppiati (ogni pezzo fa un lavoro specifico):

| Componente | Ruolo nel Sistema | Equivalente Industriale |
| :--- | :--- | :--- |
| **n8n** | Orchestratore e Logic Layer | Workflow Automation / BPM |
| **Python** | Worker (Data Ingestor) | ETL (Extract, Transform, Load) |
| **Streamlit** | Frontend (Dashboard) | React / Vue.js |
| **SQLite** | Database | PostgreSQL (per server multi-utente) |

L'obiettivo è usare n8n (strumento RAD) per definire la logica velocemente, ma con un backend (Python/SQLite) che sia tecnicamente solido e pronto per evolvere.

---

## 2. ARCHITETTURA DI RETE E SICUREZZA

### A. Il Tunnel Remoto (Ngrok / Localtunnel)
Il tuo PC Fisso agisce da server. Per renderlo accessibile da fuori (es. dal tuo Mac all'università), usiamo due tunnel:

| Servizio | Porta | Dominio | Funzione |
| :--- | :--- | :--- | :--- |
| **Backend/n8n** | 5678 | `lenslike...ngrok-free.dev` | Riceve il **Webhook** di Telegram e permette la modifica remota del workflow. |
| **Frontend/Streamlit** | 8501 | `lucky-cat-45.loca.lt` | Espone la Dashboard con i grafici UMAP. |

### B. Il Webhook (Il "Campanello" di Telegram)
* **Cos'è:** È un meccanismo di comunicazione **push** (l'opposto della API Call). Telegram ti avvisa (ti "chiama") quando arriva un messaggio al tuo bot.
* **Perché non funzionava subito:** Il tuo router (NAT/Firewall) blocca le chiamate in entrata sul tuo IP di casa. Ngrok crea un "buco" sicuro (Tunnel) per permettere a Telegram di bussare.

### C. Sicurezza
* L'accesso ai dati è protetto dal login di n8n (per la programmazione) e dal tuo IP/password (per Localtunnel).
* Ogni volta che riavvii il sistema, devi rilanciare i **quattro comandi** nei terminali.

---

## 3. IL FLUSSO DATI (Lo Schema)

L'intero processo è basato sulla **De-duplicazione** e sulla **Standardizzazione** del dato.

### A. La Catena di Ingestione (n8n Workflow)
1.  **[Telegram Trigger]**
2.  **[Execute Command]**: Controlla l'esistenza del file JSON (messaggio già elaborato?). **STOP -> se già pagato** (Anti-Spreco).
3.  **[Switch]**: Decide se è Foto/File (Ramo Immagini) o Testo (Ramo Testo).
4.  **[Save File Fisico]**: Scrive il file su disco (`IMG_timestamp.jpg`). **[NON_APPLICABILE_A_RAMO_TESTO]**
5.  **[OpenAI Vision/Text]**: Genera l'analisi strutturalista.
6.  **[Code Node]**: Pulisce il JSON sporco (toglie ````json`), recupera il **testo integrale** (dal nodo Telegram) e il **percorso file** (dal nodo `Save File Fisico`).
7.  **[Merge]**: Riunisce le due strade (Immagini e Testo) in un unico flusso.
8.  **[Read/Write Files]**: Salva il log JSON completo (`analisi_ID.json`) come archivio grezzo.
9.  **[Execute Command]**: Chiama lo script Python (`ingest_sqlite.py`) passandogli il percorso del JSON appena salvato.

### B. Il Lavoro del Python Worker
Il file `ingest_sqlite.py` (lo Script Muratore) esegue la fase finale di **Load (L)**:
* Legge il file JSON.
* Converte la lista di tag in stringhe.
* Inserisce la riga nella tabella **`memories`** in SQLite.
* Il database usa il `file_path` (nome del file: `IMG_...jpg`) come riferimento alla posizione dell'immagine sul disco.

---

## 4. IL RISULTATO (L'App Streamlit)

La Dashboard è il **Frontend** che legge il database per te.

### A. La Mappa Neurale (Grafo UMAP)
* **Concetto:** Trasforma il significato delle tue note in coordinate spaziali.
* **Processo:** Prende il Titolo, la Sintesi e i Tag -> li trasforma in un vettore (Embedding) -> usa **UMAP** per schiacciare le 384 dimensioni in 2D (X/Y).
* **Significato:** I punti vicini sul grafico sono concetti **semanticamente correlati** (es. "Fisica" vicino a "Amore" se entrambi parlano di "Entropia").

### B. Le Colonne Essenziali (Il Modello del Dato)

Il tuo database è basato sulla funzione del contenuto, non sul formato:

| Colonna | Esempio | Funzione |
| :--- | :--- | :--- |
| `meta_type` | Strategia Personale | La macro-categoria funzionale (perché hai salvato il dato?). |
| `content_format` | Formula & Schema | La forma fisica del contenuto (è testo, una foto o un grafico?). |
| `human_title` | Analisi Finanza Logica | Il titolo breve estratto dalla tua nota. |
| `robo_connections`| Isomorfismo fra onde e politica. | L'analisi laterale e il valore aggiunto dell'AI. |
| `testo_integrale` | "Non mi piace stare a letto..." | Il testo completo e grezzo, non modificato (per l'archivio). |
| `file_path` | `IMG_20251130_103441.jpg` | L'indirizzo di casa del file sul disco (per l'app). |