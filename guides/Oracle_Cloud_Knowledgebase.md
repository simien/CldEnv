# Oracle Cloud Environment: Master Knowledgebase

**Version**: 2.0 (Public Tutorial Edition)
**Scope**: Infrastructure, Software Stack, AI Configuration, and Operational Procedures.
**Purpose**: Serving as the Source of Truth for your Sovereign Contextual Stack.

---

## [ Executive Summary ]

This environment is a **Sovereign Contextual Stack** hosted on Oracle Cloud Infrastructure (OCI). It is designed to provide a private, cost-free, production-grade platform for AI automation and coding.

**Core Philosophy (Antigravity v2.0):**
1.  **Sovereignty**: All data, code, and logs reside on a user-controlled server.
2.  **Context**: The system maintains its own context via n8n (workflows) and **Qdrant** (Vector Memory).
3.  **Cloud Brain, Sovereign Memory**: Shifted heavy inference to **OpenRouter** (Cloud) to free up minimal resources for stable local memory.
4.  **Security**: Zero-trust networking with Caddy as the single ingress point.

---

## [ Infrastructure Specifications ]

### 2.1 Hardware
* **Provider**: Oracle Cloud Free Tier.
* **Region**: [Your Region]
* **Instance Type**: **VM.Standard.A1.Flex**.
  * **Architecture**: ARM64 (Ampere Altra).
  * **OCPUs**: 4.
  * **RAM**: 24 GB.
  * **Storage**: Block Volume (Default 50GB+).

### 2.2 Operating System
* **OS**: Ubuntu 24.04 LTS (Minimal, ARM64).
* **Shell**: Bash/Zsh.
* **User**: `ubuntu` (default).

### 2.3 Networking & Security
* **Public IP**: `x.x.x.x` (Reserved/Static IP recommended).
* **Domain**: `yourdomain.com` (Managed via DuckDNS/Cloudflare).
* **Ingress Ports (OCI Firewall & iptables):**
  * **22 (SSH):** Remote specific access.
  * **80 (HTTP):** Open for Let's Encrypt challenges (Auto-redirects to 443).
  * **443 (HTTPS):** Main application traffic.
* **Egress**: All outbound traffic allowed.
* **Internal Networking**: Docker Bridge Network (`caddy_net`) isolates containers. No container ports are exposed to the public internet; Caddy proxies everything.

---

## [ Application Stack (Service Registry) ]

All services run as Docker containers defined in `docker-compose.yml`.

### 3.1 Reverse Proxy & Gateway
* **Service**: **Caddy**.
* **Role**: The "Front Door". Handles SSL termination (Let's Encrypt), routing, and security headers.
* **Routing Table**:
  * `yourdomain.com` -> **n8n** (Automation).
  * `logs.yourdomain.com` -> **Portainer** (Management).
  * `status.yourdomain.com` -> **Uptime Kuma** (Monitoring).
  * `ai.yourdomain.com` -> **Open WebUI** (AI Chat).

### 3.2 Core Services
1. **n8n (Automation):**
   * **URL**: `https://yourdomain.com`
   * **Role**: The central "nervous system". Runs workflows that connect AI, data, and webhooks.
   * **Configuration**: Uses `n8n_data` (SQLite).
2. **Open WebUI (AI Interface):**
   * **URL**: `https://ai.yourdomain.com`
   * **Role**: Chat interface and RAG entry point.
   * **Config**: Connects to OpenRouter (Chat) and local Ollama (Embeddings).
3. **Ollama (Embedding Server):**
   * **Internal Port**: `11434`
   * **Role**: Local embedding generation (`nomic-embed-text`) ONLY. Large models removed to save RAM.

---

## [ AI & Model Strategy ]

The system uses a **"Cloud Brain, Sovereign Memory"** strategy.

### 4.1 Cloud Inference ("The Brain")
* **Primary**: **OpenRouter**.
  * **Logic**: `anthropic/claude-3.5-sonnet`.
  * **Fallback**: `deepseek/deepseek-chat` (Direct API).
* **Key Management**: Centralized in `.env` and injected via `docker-compose.yml`.

### 4.2 Local Context ("The Memory")
* **Embeddings**: **Ollama** running `nomic-embed-text` (CPU optimized).
* **Vector Store**: **Qdrant** (Rust-based, high performance).

---

## [ Operational Procedures ]

### 5.1 Updates & Maintenance
* **Update System**: `sudo apt update && sudo apt upgrade -y`.
* **Update Containers**:
  ```bash
  docker compose pull
  docker compose up -d
  docker image prune -f
  ```

### 5.2 Backup Strategy
* **n8n Backup**: Ensure workflows are exported periodically or backed up via volume snapshots.
* **Open WebUI Backup**: Export chat history and prompts from the setting menu.

### 5.3 Troubleshooting
* **Container Status**: `docker ps`.
* **Logs**: `docker logs -f [container_name]`.
