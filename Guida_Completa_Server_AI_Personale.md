# Guida Completa al tuo Server AI Personale

Benvenuto! Sulla base delle informazioni che mi hai fornito, ho progettato un’architettura su misura per il tuo ambiente Windows 11 con WSL2.

L’hardware a tua disposizione è eccellente: i 64GB di RAM uniti ai 12GB di VRAM della tua NVIDIA permettono un sistema ibrido ad alte prestazioni.

---

> ⚠️ **DISCLAIMER MEDICO E DI SICUREZZA**
> Hai menzionato la volontà di usare gli agenti come “psicologo” e per la “gestione della salute mentale”.
> Come esperto di sistemi, devo essere categorico:

**Nessun LLM, per quanto avanzato o open-source, può sostituire un professionista medico o terapeutico.**

I modelli linguistici:

* prevedono testo basato su pattern
* non provano empatia
* non hanno comprensione clinica
* possono generare “allucinazioni”

Affidare la salute mentale a un’IA è estremamente pericoloso.

Pertanto, la configurazione degli agenti sarà limitata a:

* produttività
* gestione del tempo
* procrastinazione

Per la salute mentale, rivolgiti sempre a specialisti qualificati.

---

## Indice Navigabile

1. Valutazione dell'hardware e Modelli Consigliati
2. Stack Tecnologico Consigliato
3. Guida di Installazione Passo-Passo
4. Configurazione della Rete (Tailscale)
5. Setup degli Agenti (CrewAI)
6. Test e Verifica

---

## 1. Valutazione dell'hardware e Modelli Consigliati

### Analisi Hardware

* **Gestione Memoria (vantaggio):**
  12GB di VRAM → modelli fino a 8–9B girano completamente su GPU

* **Offloading su RAM:**
  modelli grandi (32B–70B) usano RAM → più lenti ma stabili

* **Configurazione GPU:**
  iGPU Radeon 760M gestisce il display → NVIDIA libera per inferenza

---

### Modelli Consigliati

Formato: **GGUF**

#### A. Coding, Logica e Progetti Complessi

* Leggero: `qwen2.5-coder:7b`
* Medio: `phind-codellama:34b`
* Pesante: `deepseek-coder-v2`

#### B. Chat Generale e Agenti

* Leggero: `hermes-3-llama-3.1-8b`
* Medio: `mistral-nemo:12b`
* Pesante: `mixtral:8x7b`, `llama-3.1:70b`

#### C. RAG e Analisi Dati

* Leggero: `gemma-2-9b-it`
* Medio: `command-r`
* Pesante: `qwen2.5:72b`

---

## 2. Stack Tecnologico Consigliato

* **Ollama** → gestione modelli e VRAM
* **Open WebUI** → interfaccia chat e progetti
* **AnythingLLM** → gestione documentale (RAG)
* **CrewAI** → framework agenti
* **ComfyUI** → generazione media

---

## 3. Guida di Installazione Passo-Passo

> ⚠️ Esegui i comandi nel terminale Linux su WSL2

### Step 3.1 – Installazione Docker

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ca-certificates curl gnupg lsb-release -y

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y

sudo usermod -aG docker $USER
```

---

### Step 3.2 – NVIDIA Container Toolkit

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt update
sudo apt install -y nvidia-container-toolkit

sudo nvidia-ctk runtime configure --runtime=docker
sudo service docker restart
```

---

### Step 3.3 – Installazione Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh

ollama run qwen2.5-coder:7b
ollama pull hermes3
ollama pull mistral-nemo
```

---

### Step 3.4 – Deploy servizi

```bash
mkdir -p ~/ai-server
cd ~/ai-server
```

Crea `docker-compose.yml`:

```yaml
version: '3.8'

services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: always
    ports:
      - "3000:8080"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - ./webui-data:/app/backend/data
    environment:
      - OLLAMA_BASE_URL=http://host.docker.internal:11434

  anythingllm:
    image: mintplexlabs/anythingllm
    container_name: anythingllm
    restart: always
    ports:
      - "3001:3001"
    cap_add:
      - SYS_ADMIN
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - ./anythingllm-data:/app/server/storage
      - ./anythingllm-env:/app/server/.env
    environment:
      - STORAGE_DIR=/app/server/storage
```

Avvio:

```bash
docker compose up -d
```

---

## 4. Configurazione della Rete (Tailscale)

Accesso tramite IP Tailscale:

* Open WebUI → `http://100.x.y.z:3000`
* AnythingLLM → `http://100.x.y.z:3001`

✔ Nessun port forwarding
✔ Connessione cifrata

---

## 5. Setup degli Agenti (CrewAI)

```bash
sudo apt install python3-pip python3-venv -y

mkdir ~/ai-agents
cd ~/ai-agents

python3 -m venv venv
source venv/bin/activate

pip install crewai langchain-community
```

Script base:

```python
from crewai import Agent, Task, Crew, Process
from langchain_community.llms import Ollama

llm = Ollama(model="hermes3")

organizer = Agent(
    role='Data Architect & Organizer',
    goal='Strutturare dati e file',
    backstory='Esperto di organizzazione',
    verbose=True,
    allow_delegation=False,
    llm=llm
)

task1 = Task(
    description='Analizza file e proponi struttura',
    expected_output='Problemi + soluzione',
    agent=organizer
)

crew = Crew(
    agents=[organizer],
    tasks=[task1],
    process=Process.sequential
)

print(crew.kickoff())
```

---

## 6. Test e Verifica

1. **GPU:** `nvidia-smi` → verifica utilizzo
2. **WebUI:** `http://localhost:3000`
3. **AnythingLLM:** `http://localhost:3001`

---

✔ Sistema operativo
✔ Privato
✔ Sotto il tuo controllo

---
