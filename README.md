# Second Brain - Automated Neural Archive

This project is a self-hosted system designed to capture, analyze, and visualize data (images and text) sent via instant messaging. It's basically a backend for a future app I'm building. Right now, the focus is on applying semantic organization to personal data.

**Core Purpose:** Transform ephemeral Telegram messages into a structured, navigable knowledge graph without using third-party cloud storage for the backend.

---

## System Architecture

It runs on a local Windows Server (Self-Hosted). The architecture is microservices-like, orchestrated by n8n.

### The Stack
- **Orchestrator:** n8n (Self-hosted via npm) - Handles logic and workflows.
- **Ingestion:** Python Scripts - Handles the ETL pipeline.
- **Database:** SQLite - Local file-based storage.
- **AI:** OpenAI API (GPT-4o) - For analysis and tagging.
- **Frontend:** Streamlit + P5.js (Interactive Graph) + UMAP (Semantic Mapping).
- **Network:** Ngrok (Backend Tunnel) & Localtunnel (Frontend Access).

---

## Data Flow

1. **Ingestion:** I send text or an image to my Telegram Bot.
2. **Validation:** n8n receives the webhook via Ngrok. It runs a check against local logs to see if the message ID exists. If it does, it stops to save API costs (Anti-Duplication).
3. **Processing:**
   - Images are saved physically to the local filesystem.
   - Content is sent to OpenAI to extract title, tags, semantic connections, and scientific analogies.
4. **Standardization:** A JavaScript node normalizes the output from parallel branches (Text vs Image) into a single JSON format.
5. **Archiving:**
   - A raw JSON log is saved to disk for backup.
   - n8n triggers a Python script (`ingest_sqlite.py`) to insert the cleaned data into SQLite.
6. **Visualization:** Streamlit reads the DB, calculates Vector Embeddings, generates a UMAP projection, and renders an interactive P5.js graph.

---

## File Structure

Root Path: `[LOCAL_PROJECT_PATH]\memoria2`

- `n8n/`: Workflow JSON configurations.
- `files/`: Where binary assets (images) are stored.
- `json_logs/`: Raw JSON responses from AI (for debugging/backup).
- `cervello.sqlite`: The main relational database.
- `ingest_sqlite.py`: Python worker for DB insertion.
- `dashboard.py`: Streamlit App code (UI + UMAP + P5.js).
- `generate_graph.py`: Script for static graph generation.

---

## Challenges & Solutions

**1. No Native SQLite in n8n (Windows)**
Problem: Missing drivers/nodes when installing n8n on Windows via npm.
Solution: Decoupled the "Load" phase. n8n saves JSON to disk, then triggers a Python script to handle the actual SQL insertion.

**2. Data Duplication**
Problem: Telegram retries were causing double API billing.
Solution: Implemented "Pre-Check Logic". First check if File ID/JSON exists, then check DB for identical content strings.

**3. Mobile Visualization**
Problem: Heavy Plotly/WebGL graphs failed on mobile browsers.
Solution: Switched to P5.js. Implemented Base64 encoding to serve local images directly inside the HTML canvas, bypassing browser security blocks on local files.

---

## Roadmap

**Immediate:**
- Online: buy or use a free server to run a port for n8n and one for the visualization ******************************important
- Multi-format: Add handlers for PDF parsing and Audio (Whisper).
- Query: implement a system to make more data analysis on the db, looking for specific cluster of files.
- Visualization: Use a good framework like p5 or other js based ones to visualize the data.
- - Refactor File Handling: Validate file existence on disk before DB write (fix naming conventions).

**Future:**

- - Search Engine: Implement Vector Search (Semantic search) instead of simple keyword matching.
- Migration: Dockerize the stack to move from Local Windows to a Linux VPS (24/7 availability).
- Optimization: Improve P5.js performance for large datasets (>1,000 nodes).

---

## Usage (Localhost)

Launch 4 separate PowerShell terminals:

1. Backend Tunnel: `ngrok http --domain=[FIXED_DOMAIN] 5678`
2. Engine: `n8n start`
3. Frontend App: `python -m streamlit run dashboard.py --server.address 0.0.0.0`
4. Frontend Tunnel: `npx localtunnel --port 8501`
