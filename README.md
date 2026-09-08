# CldEnv

**Self-Hosted Cloud Server Stack**

`CldEnv` is a Docker Compose stack for running your own server on a free-tier VPS: automation, AI chat and local inference, git hosting, container/uptime monitoring, a file browser, and a set of developer utilities, all behind one reverse proxy, with your data staying on your own server.

This repository provides the configuration patterns, scripts, and guides to deploy this stack on a free **Oracle Cloud** instance or any standard VPS.

---

## [ Architecture ]

*   **Reverse proxy**: [Caddy](https://caddyserver.com) - TLS and routing for every service below.
*   **Automation**: pick one -- [n8n](https://n8n.io) (visual, node-based workflows; what this repo's example compose ships) or [Windmill](https://www.windmill.dev) (script-first, Python/TypeScript/Bash jobs with built-in scheduling; what the reference deployment behind this tutorial actually runs today). Both are included as separate, clearly-labeled blocks in `docker-compose.example.yml` -- keep the one you want, delete the other.
*   **AI chat**: [Open WebUI](https://openwebui.com) - a ChatGPT-style chat UI, backed by [Ollama](https://ollama.com) (local inference) and [OpenRouter](guides/model_registry.md) (cloud models), with [Qdrant](https://qdrant.tech) for vector search.
*   **Git hosting**: [Gitea](https://about.gitea.com) - a self-hosted git server.
*   **Container management**: [Portainer](https://www.portainer.io) - a web UI for Docker.
*   **Monitoring**: [Uptime Kuma](https://github.com/louislam/uptime-kuma) (service uptime) and [Beszel](https://github.com/henrygd/beszel) (resource metrics).
*   **Files**: [File Browser](https://filebrowser.org) - a web file manager.
*   **Developer utilities**: [IT-Tools](https://it-tools.tech) (everyday dev conversions/generators) and [ChangeDetection.io](https://changedetection.io) (page-change monitoring).
*   **Housekeeping**: [Watchtower](https://containrrr.dev/watchtower/) (update monitoring) and a keepalive container for free-tier instance reclamation.
*   **Orchestration**: Docker Compose.

---

## [ Infrastructure: Free Tier Specs ]

This stack is optimized for the **Oracle Cloud Always Free** tier, specifically the ARM64 Ampere instances.


### [ Recommended Specs ]
*   **Instance**: **VM.Standard.A1.Flex**
*   **CPU**: 4 OCPUs (ARM64).
*   **RAM**: 24 GB - essential for running local embeddings and vector DBs.
*   **Storage**: **200 GB Block Volume**.
    *   *Tip*: Oracle offers 200 GB of free block storage. You can assign it all to this one instance for maximum space (logging, vector DBs, backups) OR split it (e.g., 100GB/100GB) if you plan to run a second free instance. Maximizing it here ensures you never run out of space for Docker images.
*   **OS**: Ubuntu 22.04 or 24.04 (ARM64).

### Other VPS Options
While optimized for Oracle ARM, this stack runs perfectly on any x86/ARM VPS (DigitalOcean, Hetzner, AWS) with Docker installed.

### Domains & Networking (DuckDNS)
You don't need a paid domain. This stack is configured to work with **DuckDNS** or any dynamic DNS provider.

1.  Get a free subdomain from `duckdns.org` (e.g., `my-ai-stack.duckdns.org`).
2.  Use the wildcard feature: Caddy will automatically route subdomains like `n8n.my-ai-stack.duckdns.org` or `ai.my-ai-stack.duckdns.org` if you configure your DNS records or use Caddy's internal routing capabilities.
3.  The included `Caddyfile.example` shows how to set this up easily.

---


## [ Quick Start ]

### 1. Requirements
*   A VPS (Oracle Cloud ARM64 or generic).
*   **Docker & Docker Compose**.
*   A domain name (or DuckDNS subdomain).


#### [ Essential Setup: Installing Docker ]

If you are on a fresh Oracle Ubuntu instance, you can install the latest Docker engine quickly:

```bash
# 1. Update and Install Essentials
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git ufw

# 2. Install Docker (Official Script)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 3. Enable Non-Root Docker Access (Vital!)
# Allows running 'docker' without 'sudo'
sudo usermod -aG docker $USER
newgrp docker

# 4. Verify
docker run hello-world
```

### 2. Setup
Clone this repository to your server:

```bash
git clone https://github.com/simien/CldEnv.git
cd CldEnv
```

### 3. Configuration
We provide example configurations that need to be customized.

**A. Networking (Caddy)**
```bash
cp Caddyfile.example Caddyfile
nano Caddyfile
```
*   Replace `yourdomain.com` with your actual domain/subdomain.
*   Update the email address for Let's Encrypt notifications.

**B. Services (Docker)**
```bash
cp docker-compose.example.yml docker-compose.yml
nano docker-compose.yml
```
*   Set your secure passwords (API Key, Basic Auth).
*   Add your **OpenRouter API Key** (and OpenAI/Anthropic if using directly).
*   Pick your automation engine: delete the `n8n` block or the `windmill_server`/`windmill_lsp`/`windmill_db` blocks (see Architecture above), plus their matching Caddy route. If keeping Windmill, also set `WINDMILL_DB_PASSWORD`.

**C. Local AI (Ollama)**
To enable local RAG and chat, pull the essential models:
```bash
# Embeddings (Required for RAG)
docker exec -it ollama ollama pull nomic-embed-text

# Chat (Optional Local Fallback)
docker exec -it ollama ollama pull qwen2.5:14b
# or for smaller instances
docker exec -it ollama ollama pull llama3.2
```


### 4. Deploy
Start the stack:

```bash
docker compose up -d
```

The services are now running:
*   **n8n**: `https://yourdomain.com` (or configured subdomain) -- if you kept n8n
*   **Windmill**: `https://windmill.yourdomain.com` -- if you kept Windmill instead
*   **Open WebUI**: `https://chat.yourdomain.com`

---


## [ Documentation ]

*   [**Model Registry**](guides/model_registry.md): How to configure Ollama and OpenRouter.

## License
MIT
