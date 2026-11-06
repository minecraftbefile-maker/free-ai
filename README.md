# 🚀 Free AI — Serverless GGUF LLM Proxy

Run **any GGUF model** on free GitHub Actions infrastructure, expose it as an **OpenAI-compatible API** through a public Cloudflare tunnel, and manage everything from a custom web dashboard.

No GPU. No servers. No monthly bill. Just a GitHub repo and a browser.

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

```mermaid
graph TD
    Client[OpenAI SDK / Browser UI] -->|HTTPS| CF[Cloudflare Quick Tunnel]
    CF -->|Proxy| API[FastAPI App]
    
    API -->|Manage| Dashboard[WebUI Dashboard]
    API -->|Inference| LLM[llama-cpp-python]
    
    LLM -->|Reads GGUF| HF[(Hugging Face Hub)]
