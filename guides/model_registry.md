# Model Registry (Tutorial Edition)

This document lists the recommended AI setup for this tutorial stack, focusing on simplicity and cost-effectiveness.

---

## 1. Cloud Models (The "Gateway" Strategy)
Instead of managing multiple API keys (OpenAI, Anthropic, Google, etc.), we recommend using **OpenRouter** as a single gateway.

### **OpenRouter**
*   **Role**: **Universal Brain**. Connects to all major models via one API key.
*   **Why?**: No monthly subscriptions, pay-per-token, access to top models (Claude 3.5, GPT-4o, DeepSeek V3) instantly.
*   **Billing**: Prepaid credits (start with $5).
*   **Recommended Models**:
    *   **Coding/Logic**: `anthropic/claude-3.5-sonnet` (Top Tier) or `deepseek/deepseek-chat` (Best Value).
    *   **Reasoning**: `deepseek/deepseek-reasoner` (R1).
    *   **General**: `google/gemini-2.0-flash-exp` (Fast & often free on OpenRouter).

---

## 2. Local Models (Ollama)
*Runs locally on your Oracle Cloud ARM64 instance. Free & Private.*

| Model | Tag | Role |
| :--- | :--- | :--- |
| **Nomic Embed** | `nomic-embed-text` | **Essential**. Powers the "Knowledge" (RAG) feature in Open WebUI. |
| **Qwen 2.5 14B** | `qwen2.5:14b` | Great balance of speed/intelligence for local chat. |
| **Llama 3.2** | `llama3.2` | Lightweight model for basic tasks. |

### 📥 How to Install
Run these commands on your server to download the models:

```bash
# 1. Embeddings (Required for document search)
docker exec -it ollama ollama pull nomic-embed-text

# 2. Local Chat Model
docker exec -it ollama ollama pull qwen2.5:14b
```

---

## 3. Usage in Stack

*   **Open WebUI**:
    *   Go to **Settings > Connections**.
    *   **OpenAI API**: Enter `https://openrouter.ai/api/v1` as the URL and your OpenRouter Key.
    *   **Ollama**: Should be connected automatically (`http://ollama:11434`).

*   **n8n Workflows**:
    *   Use the **OpenAI Chat Model** node.
    *   Create a Credential with:
        *   URL: `https://openrouter.ai/api/v1`
        *   API Key: `sk-or-...`
