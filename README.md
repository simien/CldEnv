# CldEnv

**Sovereign AI Automation Infrastructure**

`CldEnv` is a production-grade, secure, and cost-effective environment for running your own AI agents, automation workflows, and "Sovereign Contextual Stacks". It combines the power of **n8n** (Logic), **Open WebUI** (Interface), and **Ollama** (Local AI) into a unified, privacy-first platform.

This repository provides the configuration patterns, scripts, and guides to deploy this stack on a free **Oracle Cloud** instance or any standard VPS.

---

## [ Architecture ]

*   **Logic**: [n8n](https://n8n.io) (Workflow Automation) - The nervous system of your stack.
*   **Interface**: [Open WebUI](https://openwebui.com) (ChatGPT-style UI) - The friendly face for your AI models.
*   **Intelligence**: [Ollama](https://ollama.com) (Local Inference) + [OpenRouter](guides/model_registry.md) (Cloud Brain).
*   **Security**: [Caddy](https://caddyserver.com) (Reverse Proxy) - Handles auto-SSL and routing.
*   **Orchestration**: Docker Compose + **Antigravity Kit**.

---

## [ Infrastructure: The "Free Tier" Powerhouse ]

This stack is optimized for the **Oracle Cloud Always Free** tier, specifically the ARM64 Ampere instances.


### [ recommended Specs ]
*   **Instance**: **VM.Standard.A1.Flex**
*   **CPU**: 4 OCPUs (ARM64) - surprisingly powerful for AI workloads.
*   **RAM**: 24 GB - essential for running local embeddings and vector DBs.
*   **Storage**: **200 GB Block Volume**.
    *   *Tip*: Oracle offers 200 GB of free block storage. You can assign it all to this one instance for maximum space (logging, vector DBs, backups) OR split it (e.g., 100GB/100GB) if you plan to run a second free instance. Maximizing it here ensures you never run out of space for Docker images.
*   **OS**: Ubuntu 22.04 or 24.04 (ARM64).

### Other VPS Options
While optimized for Oracle ARM, this stack runs perfectly on any x86/ARM VPS (DigitalOcean, Hetzner, AWS) with Docker installed.

### 🌐 Domains & Networking (DuckDNS)
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
*   A domain name (or DuckDNS subdomain).

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
*   Set your secure passwords (API Key, Basic Auth).
*   Add your **OpenRouter API Key** (and OpenAI/Anthropic if using directly).

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
Fire it up!

```bash
docker compose up -d
```

Your stack is now live!
*   **n8n**: `https://yourdomain.com` (or configured subdomain)
*   **Open WebUI**: `https://chat.yourdomain.com`

---


## [ Antigravity Integration ]

This environment is designed to work with **Antigravity**, an agentic AI framework.

*   **Workflows**: The `.agent/workflows` directory contains automation protocols.
*   **Scripts**: The `scripts/` directory includes helper tools for maintenance.
    *   `publish_tutorial.sh`: The script used to generate THIS public repo from our private internal development stack!


## [ Documentation ]

*   [**Model Registry**](guides/model_registry.md): How to configure Ollama and OpenRouter.
*   [**n8n Setup**](guides/n8n_setup.md): Getting started with workflows. (Coming Soon)
*   [**Open WebUI**](guides/openwebui_setup.md): Connecing your first model. (Coming Soon)

## License
MIT
