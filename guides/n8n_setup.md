# n8n Setup Guide

## Overview
**n8n** is the nervous system of your stack. It orchestrates data flow between your AI agents and external services.

## Configuration

### Environment Variables
In your `docker-compose.yml`, ensure these are set correctly:

```yaml
n8n:
  environment:
    - N8N_HOST=yourdomain.com
    - WEBHOOK_URL=https://yourdomain.com/
```

### First Launch
1.  Navigate to `https://yourdomain.com`.
2.  Create your owner account.
3.  Go to **Settings > Community Nodes** and install any needed extensions.

## Connecting to Local AI (Ollama)
To make n8n talk to the local Ollama instance:

1.  Use the **HTTP Request** node or **Ollama Chat Model** node.
2.  **Base URL**: `http://host.docker.internal:11434`
    *   *Note*: `localhost` will NOT work because n8n is inside a Docker container. You must use the internal Docker host address.

## Connecting to Cloud AI
For robust production workflows, we recommend using the **OpenAI Chat Model** node with a custom Base URL:

*   **DeepSeek**: `https://api.deepseek.com`
*   **OpenRouter**: `https://openrouter.ai/api/v1`
