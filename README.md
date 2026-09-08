# CldEnv

**Self-Hosted Cloud Server Stack**

`CldEnv` is a Docker Compose stack for running your own server on a free-tier VPS: automation, AI chat and local inference, git hosting, container/uptime monitoring, a file browser, and a set of developer utilities, all behind one reverse proxy, with your data staying on your own server.

This repository provides the configuration patterns, scripts, and guides to deploy this stack on a free **Oracle Cloud** instance or any standard VPS.

---

## [ Architecture ]

*   **Reverse proxy**: [Caddy](https://caddyserver.com) - TLS and routing for every service below.
*   **Automation**: pick one -- [n8n](https://n8n.io) (visual, node-based workflows; what this repo's example compose ships) or [Windmill](https://www.windmill.dev) (script-first, Python/TypeScript/Bash jobs with built-in scheduling; what the reference deployment behind this tutorial actually runs today). Both are included as separate, clearly-labeled blocks in `docker-compose.example.yml` -- keep the one you want, delete the other.
*   **AI chat**: [Open WebUI](https://openwebui.com) - a ChatGPT-style chat UI, backed by [Ollama](https://ollama.com) for free local inference by default, with [free cloud models](guides/model_registry.md) as an optional add-on, and [Qdrant](https://qdrant.tech) for vector search.
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

### Domains & Networking

Each service in `Caddyfile.example` gets its own subdomain (`ai.yourdomain.com`, `git.yourdomain.com`, etc.). Caddy is what routes each one to the right container once DNS resolves -- it isn't involved in making the DNS itself work. Either path below gets you there; pick whichever matches what you already have.

**Option A: You own a domain (recommended if you have one)**

Most registrars and DNS providers support wildcard records, which cover every subdomain in one shot:

1.  In your domain's DNS settings, add a single **A record**: host `*`, value your server's public IP (e.g., `*.yourdomain.com -> 203.0.113.10`).
2.  That's it -- `ai.yourdomain.com`, `git.yourdomain.com`, and every other subdomain in `Caddyfile.example` now resolve automatically, present or future, with no further DNS changes.
3.  If your provider doesn't support wildcards, add one A record per subdomain instead.

**Option B: Free subdomain via DuckDNS**

DuckDNS behaves like a wildcard once you've registered one name -- it resolves *any* subdomain under that name to the same IP automatically, with nothing extra to configure. Confirmed directly: `made-up-name.sapcloud.duckdns.org` resolves the same as `sapcloud.duckdns.org` itself, no prior registration needed for that specific subdomain.

1.  Get a free domain from `duckdns.org` (e.g., `my-ai-stack.duckdns.org`), pointed at your server's IP.
2.  That's it -- `ai.my-ai-stack.duckdns.org`, `git.my-ai-stack.duckdns.org`, and every other subdomain in `Caddyfile.example` already resolve to that same IP, present or future, with no further DNS changes. The dashboard's "domains X/5" counter is how many separate *root* names you've registered (useful for running entirely separate projects), not a limit on subdomains under the one you're using here.

Either way, the included `Caddyfile.example` shows the routing for every service once DNS resolves -- Caddy requests and renews TLS certificates automatically, the same way, regardless of which option you used.

---


## [ Quick Start ]

### 1. Requirements
*   A VPS (Oracle Cloud ARM64 or generic).
*   **Docker & Docker Compose**.
*   A domain name (or DuckDNS subdomain).
*   **On Oracle Cloud specifically**: ports 80 and 443 also need opening in the instance's own **Security List / Network Security Group** (Networking > Virtual Cloud Networks in the OCI console) -- this is a separate firewall from the instance's own `ufw`, and traffic gets silently dropped at this layer if it's not opened here too, regardless of how `ufw` is configured.


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
We provide example configurations that need to be customized. `yourdomain.com` appears in both files below -- replace every instance of it in both, not just one.

**A. Networking (Caddy)**
```bash
cp Caddyfile.example Caddyfile
nano Caddyfile
```
*   Replace `yourdomain.com` with your actual domain/subdomain.
*   Update the email address for Let's Encrypt notifications.
*   Fix up the basic-auth line on the `tools.` route (and any others you add one to): generate a real hash after Caddy is running, with `docker exec caddy caddy hash-password`, and paste it in.

**B. Secrets (.env)**
```bash
cp .env.example .env
nano .env
```
*   Fill in `WEBUI_SECRET_KEY` (required). Local models via Ollama need no key at all; if you want cloud models too, `GEMINI_API_KEY` is the free option (see [Model Registry](guides/model_registry.md) for why). See the comments in `.env.example` for every variable and which ones are optional.
*   Docker Compose loads `.env` automatically from this directory -- no extra flag or step needed.

**C. Services (Docker)**
```bash
cp docker-compose.example.yml docker-compose.yml
nano docker-compose.yml
```
*   Replace `yourdomain.com` here too (see the note above).
*   Pick your automation engine: delete the `n8n` block or the `windmill_server`/`windmill_lsp`/`windmill_db` blocks (see Architecture above), plus their matching Caddy route.

**D. Local AI (Ollama)**
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
*   **Open WebUI**: `https://ai.yourdomain.com`

---


## [ Documentation ]

*   [**Model Registry**](guides/model_registry.md): How to configure Ollama and OpenRouter.
*   [**Open WebUI Setup**](guides/openwebui_setup.md): First-login admin setup and connecting models.
*   [**Oracle Cloud Knowledgebase**](guides/Oracle_Cloud_Knowledgebase.md): Full service reference, routing table, AI strategy, and troubleshooting.

## License
MIT
