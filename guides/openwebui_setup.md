# Open WebUI Setup Guide

## Overview
**Open WebUI** is the interface for your AI agents. It provides a ChatGPT-like experience that can talk to both your local Ollama models and cloud APIs.

## Configuration

### Admin Account
1.  Navigate to `https://chat.yourdomain.com`.
2.  The first account created becomes the **Admin**.
3.  Go to **Admin Panel > Settings** to configure global defaults.

## Connecting Models

### Cloud Models (DeepSeek / OpenRouter)
We configure these via environment variables in `docker-compose.yml` for security and ease of management.

```yaml
open-webui:
  environment:
    - OPENAI_API_BASE_URLS=https://api.deepseek.com;https://openrouter.ai/api/v1
    - OPENAI_API_KEYS=${DEEPSEEK_API_KEY};${OPENROUTER_API_KEY}
```

### Local Models (Ollama)
Open WebUI automatically detects Ollama running on the same host if configured with:
`OLLAMA_BASE_URL=http://ollama:11434`

## Retrieval Augmented Generation (RAG)
1.  Go to **Workspace > Knowledge**.
2.  Create a Collection (e.g., "Project Docs").
3.  Upload your files.
4.  In any chat, type `#` to select a collection and "talk" to your documents.
