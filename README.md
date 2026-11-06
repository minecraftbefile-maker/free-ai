# 🚀 Free AI — Serverless GGUF LLM Proxy

Run **any GGUF model** on free GitHub Actions infrastructure, expose it as an **OpenAI-compatible API** through a public Cloudflare tunnel, and manage everything from a custom web dashboard.

No GPU. No servers. No monthly bill. Just a GitHub repo and a browser.

---

> ⚠️ **WARNING: EPHEMERAL STORAGE** 
> All downloaded models, configuration files, API keys, and chat history will be **permanently wiped** when the GitHub Actions runner shuts down (max 6 hours per run). This environment is strictly for temporary usage, testing, and experimentation. Do not rely on it for production or persistent data.

---

## ✨ Features

| | |
|---|---|
| 🆓 **100% free** | Runs entirely on GitHub Actions free tier |
| 🧠 **Any GGUF model** | Download & load from Hugging Face on demand |
| 🔌 **OpenAI SDK compatible** | Drop-in replacement for `api.openai.com` |
| ⚡ **Streaming** | Server-Sent Events, works with every OpenAI SDK |
| 🛠 **Tool/function calling** | `tools`, `tool_choice`, `response_format` fully supported |
| 👥 **All chat roles** | `system`, `user`, `assistant`, `tool`, `function` |
| 🔐 **Two-layer auth** | First-time admin password + revocable bearer API key |
| 📊 **Speed metrics** | Live tokens/sec + completion time on every response |
| 🌐 **Public URL** | Zero-config Cloudflare Quick Tunnel |
| 🎨 **Custom WebUI** | Load models, rotate keys, inspect performance |
| 🚫 **No PyTorch** | Pure `llama-cpp-python` — small, fast installs |
| 🔄 **Hot model switching** | Download → load → unload → switch, all in-browser |

---

## 🏗 How It Works

The system consists of four main components working together:

| Component | Role |
| :--- | :--- |
| **Web Browser / OpenAI SDK** | The client interface (either the custom dashboard or your Python/Node code). |
| **Cloudflare Quick Tunnel** | Provides a free, temporary public HTTPS URL that routes traffic to the runner. |
| **FastAPI Proxy** | Handles authentication, OpenAI API routing, token/sec calculations, and serves the UI. |
| **llama-cpp-python** | The CPU inference engine that loads `.gguf` files into RAM and generates text. |

### The Lifecycle of a Request

1. **Startup:** The GitHub workflow installs dependencies, starts the FastAPI proxy, and opens the Cloudflare tunnel. 
2. **First Visit:** You visit the public URL. The proxy detects no password is set and forces you to create one (`/setup`).
3. **Model Management:** You use the WebUI to search Hugging Face, download a single-file `.gguf` model, and load it into RAM. 
4. **API Generation:** You use the generated API key to send requests to `/v1/chat/completions` (or use the dashboard interface). The proxy forwards it to `llama-cpp-python`, formats the response, calculates the tokens/sec, and returns it to you.
5. **Shutdown:** The job hits the 6-hour limit (or is cancelled), the runner destroys the VM, and all models/data are lost.

---

## 🚀 Quick Start

### 1. Fork this repository

Click **Fork** in the top-right corner.

### 2. Enable Actions

Go to the **Actions** tab → click **"I understand my workflows, go ahead and enable them"** if prompted.

### 3. Run the workflow

- **Actions** → **Custom-Auth LLM Proxy** → **Run workflow** → **Run workflow**
- Wait ~2–3 minutes for setup to complete.

### 4. Get your URL

In the **"Expose via Cloudflare Tunnel"** step's log, find:
