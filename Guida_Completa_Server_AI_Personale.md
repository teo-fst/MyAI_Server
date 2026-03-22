# 🚀 Guida alla Configurazione: Server AI Locale Multi-Agente

Questa guida descrive l'installazione e la configurazione di un ecosistema AI autonomo su **Windows 11 (WSL2)**, ottimizzato per:

* 🧠 **GPU:** NVIDIA RTX 5070 (12GB VRAM)
* 💾 **RAM:** 64GB
* ⚙️ **CPU:** Ryzen 5 8600G

---

## 📑 Indice

* [1. Valutazione Hardware e Modelli](#1-valutazione-hardware-e-modelli)
* [2. Stack Tecnologico](#2-stack-tecnologico-consigliato)
* [3. Installazione](#3-guida-di-installazione-passo-passo)
* [4. Rete e Accessibilità](#4-configurazione-della-rete-e-accessibilità)
* [5. Agenti e Automazione](#5-setup-degli-agenti-e-automazione)
* [6. Test e Verifica](#6-test-e-verifica)

---

# 1. Valutazione Hardware e Modelli

## 🔍 Analisi Risorse

<details>
<summary>Espandi dettagli</summary>

* **GPU (RTX 5070 12GB):**
  Motore primario. 12GB di VRAM permettono modelli fino a 14B completamente in memoria video.

* **System RAM (64GB):**
  Fondamentale per modelli Large (fino a 70B) tramite offloading (GGUF).

* **CPU (Ryzen 5 8600G):**
  Gestione container Docker e calcolo layer non in GPU.

</details>

---

## 🤖 Selezione Modelli (Senza Filtri / Open)

Ho selezionato tre categorie basate sulla tua richiesta di flessibilità e assenza di censure.

| Categoria                      | Modello Small (Fast)       | Modello Medium (Balanced) | Modello Large (High Logic) |
| :----------------------------- | :------------------------- | :------------------------ | :------------------------- |
| **General / Roleplay / Psico** | **Llama-3.1-8B-Lexi**      | **Mistral-Nemo-12B-Aura** | **Llama-3.1-70B-Instruct** |
| **Coding & Workflow**          | **DeepSeek-Coder-V2-Lite** | **Qwen2.5-Coder-14B**     | **Codestral-22B**          |
| **Multimodale (Vision)**       | **Moondream2**             | **Llava-v1.6-Mistral-7B** | **Llava-v1.5-13B**         |

> 💡 **PERCHÉ**
> Il modello 70B userà la RAM → lento (~1–2 token/s) ma estremamente potente
> I modelli 8B–14B girano interamente su GPU → velocità istantanea

---

# 2. Stack Tecnologico Consigliato

L’architettura si basa su **Docker + WSL2** per isolamento e controllo locale dei dati.

<details>
<summary>Componenti dello stack</summary>

* **Ollama (Backend)**
  Gestione dinamica VRAM/RAM

* **Open WebUI (Frontend)**
  Interfaccia tipo ChatGPT + Modelfiles

* **AnythingLLM (RAG)**
  Gestione documenti e indicizzazione locale

* **ComfyUI (Media)**
  Generazione immagini/video con API

</details>

---

# 3. Guida di Installazione Passo-Passo

## A. Preparazione WSL2 e Driver

```powershell
# Apri PowerShell come amministratore
wsl --update
wsl --install
```

---

## B. Installazione Docker Desktop

Scarica e installa:
👉 [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

✔ Abilita **WSL2 backend**

---

## C. Deployment dello Stack

Crea `docker-compose.yaml`:

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    volumes:
      - ./ollama:/root/.ollama
    ports:
      - "11434:11434"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    volumes:
      - ./open-webui:/app/backend/data
    ports:
      - "3000:8080"
    environment:
      - 'OLLAMA_BASE_URL=http://ollama:11434'
    depends_on:
      - ollama
    restart: always
```

Avvio:

```bash
docker compose up -d
```

> ⚠️ Primo avvio: ~2GB download immagini Docker

---

## D. Installazione Modelli

```bash
docker exec -it ollama ollama run llama3.1:8b
docker exec -it ollama ollama run qwen2.5-coder:14b
docker exec -it ollama ollama run llama3.1:70b
```

---

# 4. Configurazione della Rete e Accessibilità

✔ Nessun port forwarding
✔ Sicurezza elevata con Tailscale

---

## Configurazione

<details>
<summary>Espandi</summary>

1. **Accesso Statico**
   Configura Open WebUI su `0.0.0.0`

2. **Tailscale Funnel (Opzionale)**

```bash
tailscale funnel 3000
```

3. **Sicurezza**
   Disabilita registrazioni pubbliche in Open WebUI

</details>

---

# 5. Setup degli Agenti e Automazione

## 📚 AnythingLLM

<details>
<summary>Setup</summary>

1. Installa versione Desktop o Docker
2. LLM Engine → `http://localhost:11434`
3. Crea Workspace
4. Aggiungi cartelle locali → indicizzazione vettoriale

</details>

---

## 🤖 Agenti (Open WebUI)

### Creazione "Coach Benessere"

1. Workspace → Modelli → Crea
2. Nome: **Coach Benessere**
3. Base Model: `llama3.1:70b`
4. System Prompt:

> Agisci come uno psicologo esperto in terapia cognitivo-comportamentale...

---

## ⚙️ Automazione con CrewAI

```bash
pip install crewai langchain_community
```

Uso:

* analisi file
* verifica template
* riorganizzazione automatica

---

# 6. Test e Verifica

## ✅ Checklist

* **GPU**

```bash
nvidia-smi
```

* **RAM**
  → test modello 70B

* **Accesso remoto**

```
http://[IP-TAILSCALE]:3000
```

---

# ✅ Stato Finale

✔ Sistema locale
✔ Nessun cloud
✔ Open-source
✔ Multi-agente
✔ Sicuro

---

## 🏁 Fine della guida
