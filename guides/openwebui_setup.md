# Open WebUI Setup Guide

## Overview
**Open WebUI** is the interface for your AI agents. It provides a ChatGPT-like experience that can talk to both your local Ollama models and cloud APIs.

## Configuration

### Admin Account
1.  Go to `https://ai.yourdomain.com`.
2.  The first account created becomes the **Admin**.
3.  Go to **Admin Panel > Settings** to configure global defaults.

## Connecting Models

This stack defaults to free options -- see [Model Registry](model_registry.md) for the full reasoning.

### Local Models (Ollama) -- default, always free
Open WebUI automatically detects Ollama running on the same host, configured via:
`OLLAMA_BASE_URL=http://ollama:11434`. No API key, no setup beyond pulling a model (see Model Registry).

### Cloud Models (optional)
All configured via environment variables in `docker-compose.yml` -- no in-app setup needed once `.env` is filled in.

*   **Gemini** (recommended free option): `ENABLE_GEMINI_API=true` + `GEMINI_API_KEY`.
*   **DeepSeek / OpenRouter** (optional, paid):
    ```yaml
    open-webui:
      environment:
        - OPENAI_API_BASE_URLS=https://api.deepseek.com;https://openrouter.ai/api/v1
        - OPENAI_API_KEYS=${DEEPSEEK_API_KEY};${OPENROUTER_API_KEY}
    ```

## Retrieval Augmented Generation (RAG)
1.  Go to **Workspace > Knowledge**.
2.  Create a Collection (e.g., "Project Docs").
3.  Upload your files.
4.  In any chat, type `#` to select a collection and "talk" to your documents.
