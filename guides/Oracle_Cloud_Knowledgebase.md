# Oracle Cloud Environment: Knowledgebase

**Version**: 2.0 (Public Tutorial Edition)
**Scope**: Infrastructure, software stack, AI configuration, and operational procedures.
**Purpose**: Reference documentation for this stack's configuration and behavior.

---

## [ Executive Summary ]

This environment is a self-hosted server stack on Oracle Cloud Infrastructure (OCI): a free, private platform for automation, AI, and general server tooling.

**Design principles:**
1.  **Data ownership**: All data, code, and logs reside on a user-controlled server.
2.  **Context**: The system maintains its own context via its automation engine (n8n or Windmill workflows) and **Qdrant** (vector memory).
3.  **Split inference**: heavy inference runs on **OpenRouter** (cloud), keeping local resources free for embeddings and stable local memory.
4.  **Security**: zero-trust networking with Caddy as the single ingress point.

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
* **Role**: Single ingress point. Handles TLS termination (Let's Encrypt), routing, and security headers.
* **Routing Table**:
  * `yourdomain.com` -> **n8n** (Automation, if kept).
  * `windmill.yourdomain.com` -> **Windmill** (Automation, if kept instead of n8n).
  * `logs.yourdomain.com` -> **Portainer** (Management).
  * `status.yourdomain.com` -> **Uptime Kuma** (Monitoring).
  * `ai.yourdomain.com` -> **Open WebUI** (AI Chat).

### 3.2 Core Services

**Automation (pick one):**
1. **n8n** -- visual, node-based workflows.
   * **URL**: `https://yourdomain.com`
   * **Role**: Runs workflows that connect AI, data, and webhooks.
   * **Configuration**: Uses `n8n_data` (SQLite).
2. **Windmill** -- script-first jobs (Python/TypeScript/Bash) with built-in scheduling. What the reference deployment behind this tutorial actually runs.
   * **URL**: `https://windmill.yourdomain.com`
   * **Role**: Same job as n8n above -- scheduled and webhook-triggered automation -- written as scripts instead of visual flows.
   * **Configuration**: `windmill_server` + `windmill_lsp` (editor language support) + `windmill_db` (Postgres job store).

**Everything else:**
3. **Open WebUI (AI Interface):**
   * **URL**: `https://ai.yourdomain.com`
   * **Role**: Chat interface and RAG entry point.
   * **Config**: Connects to OpenRouter (Chat) and local Ollama (Embeddings).
4. **Ollama (Embedding Server):**
   * **Internal Port**: `11434`
   * **Role**: Local embedding generation (`nomic-embed-text`) ONLY. Large models removed to save RAM.

---

## [ AI & Model Strategy ]

The model strategy follows the same free-by-default rule as the rest of this stack: paid only if chosen. See `guides/model_registry.md` for the full reasoning; summary below.

### 4.1 Local Inference (default, always free)
* **Embeddings**: **Ollama** running `nomic-embed-text` (CPU optimized).
* **Chat**: **Ollama** running `llama3.2` (or `qwen2.5:14b` for better quality at the cost of speed on CPU-only inference).
* **Vector store**: **Qdrant** (Rust-based, high performance).

### 4.2 Cloud Inference (optional, still free)
* **Recommended**: **Google Gemini** -- a genuinely free tier (rate-limited, not credit-limited) covering current-generation models. No billing setup required.
* **Optional gateway**: **OpenRouter** -- reaches many providers through one key, but its free-tier model list is small and rotates; almost everything else on it is paid, prepaid-credit based. Treat it as the upgrade path to a specific premium model, not the default.
* **Key management**: centralized in `.env` and injected via `docker-compose.yml`.

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
