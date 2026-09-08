# Model Registry (Tutorial Edition)

This stack is built around a free-tier setup, so the model recommendations follow the same rule: **free by default, paid only if you choose it.** Nothing here requires a credit card or prepaid balance to get a working setup.

---

## 1. Local Models (Ollama)
*Runs on your own instance. Free, private, no API key, no rate limit beyond your own hardware.*

| Model | Tag | Role |
| :--- | :--- | :--- |
| **Nomic Embed** | `nomic-embed-text` | **Essential**. Powers the "Knowledge" (RAG) feature in Open WebUI. |
| **Llama 3.2** | `llama3.2` | Lightweight chat model -- realistic for the free-tier ARM64 CPU. |
| **Qwen 2.5 14B** | `qwen2.5:14b` | Noticeably better answers, noticeably slower on CPU-only inference. Try it; drop to `llama3.2` if responses feel too slow. |

### How to Install
Run these commands on your server to download the models:

```bash
# 1. Embeddings (Required for document search)
docker exec -it ollama ollama pull nomic-embed-text

# 2. Local Chat Model
docker exec -it ollama ollama pull llama3.2
```

This alone is a complete, zero-cost setup: no cloud account, no API key, nothing to configure below.

---

## 2. Cloud Models (Optional, Still Free)

Local models are limited by the free-tier instance's CPU-only inference -- fine for short answers, slow for anything long or complex. If you want faster or more capable responses without paying, two providers have real, sustainable free tiers:

### **Google Gemini (recommended free cloud option)**
*   **Get a key**: [Google AI Studio](https://aistudio.google.com/apikey) -- no billing setup required for the free tier.
*   **Why**: A genuinely free tier (not a trial credit that runs out) covering current-generation models, including Gemini 2.5 Flash and Flash-Lite. Rate-limited, not capped by a dollar amount.
*   **Use directly**: set `GEMINI_API_KEY` in `.env` -- already wired into Open WebUI and n8n.

### **OpenRouter (optional gateway, mostly a paid service)**
*   **Role**: one API key that can reach many providers' models (Claude, GPT, Gemini, DeepSeek, and others) through a single endpoint.
*   **Free models**: OpenRouter does list a handful of `:free`-suffixed models with no cost, but that list is small and rotates -- check [openrouter.ai/models?max_price=0](https://openrouter.ai/models?max_price=0) for what's currently available rather than relying on any specific model name staying free.
*   **Everything else on OpenRouter is paid**, prepaid-credit based. Treat it as the upgrade path if you decide you want a specific premium model (e.g. `anthropic/claude-3.5-sonnet`) rather than as this tutorial's default.
*   **Use directly**: set `OPENROUTER_API_KEY` in `.env` -- already wired into Open WebUI, n8n, and Windmill.

### DeepSeek / other direct APIs
DeepSeek's own API is inexpensive but not free -- it's included as an optional direct-API alternative (`DEEPSEEK_API_KEY`) if you specifically want their models, not a default recommendation.

---

## 3. Usage in Stack

*   **Open WebUI**:
    *   Go to **Settings > Connections**.
    *   **Ollama**: connected automatically (`http://ollama:11434`) -- works with no further setup.
    *   **Gemini**: already wired via `GEMINI_API_KEY` in `.env`.
    *   **OpenRouter** (optional): enter `https://openrouter.ai/api/v1` as the URL and your OpenRouter key.

*   **n8n / Windmill workflows**:
    *   Use each tool's HTTP/AI node with the provider's API URL and your key from `.env`.
    *   Gemini: `https://generativelanguage.googleapis.com/v1beta` with `GEMINI_API_KEY`.
    *   OpenRouter (optional): `https://openrouter.ai/api/v1` with `OPENROUTER_API_KEY`.
