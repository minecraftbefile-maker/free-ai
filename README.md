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

In the **"Expose via Cloudflare Tunnel"** step's log, find the block that looks like this:

    INF |  Your quick Tunnel has been created! Visit it at:
    INF |  https://random-words-here.trycloudflare.com

That's your live URL. Copy it.

### 5. First-time setup

Open the URL in a browser → create your admin password.

### 6. Download & load a model

From the dashboard, go to the **Model Store**:

| Field | Example (fast, small) | Example (smart, slow) |
|-------|-----------------------|-----------------------|
| Repository ID | `TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF` | `bartowski/Meta-Llama-3.1-8B-Instruct-GGUF` |
| Filename | `tinyllama-1.1b-chat-v1.0.Q8_0.gguf` | `Meta-Llama-3.1-8B-Instruct-Q4_K_M.gguf` |

Click **Download** → then head to the Dashboard and click **Load**.

### 7. Copy your API key

Scroll down on the dashboard → **API Access** → click **Show** → **Copy** the key (`sk-...`).

---

## 📡 Using the API

### Python (OpenAI SDK)

    from openai import OpenAI

    client = OpenAI(
        base_url="https://your-tunnel.trycloudflare.com/v1",
        api_key="sk-your-api-key",
    )

    response = client.chat.completions.create(
        model="anything",                       # currently loaded model is used
        messages=[{"role": "user", "content": "Explain quantum computing in one sentence."}],
        max_tokens=200,
    )

    print(response.choices[0].message.content)
    print(f"\n⚡ {response.timings['tokens_per_second']} tokens/sec")

### Python (streaming)

    stream = client.chat.completions.create(
        model="anything",
        messages=[{"role": "user", "content": "Count from 1 to 20."}],
        stream=True,
    )

    for chunk in stream:
        if chunk.choices and chunk.choices[0].delta.content:
            print(chunk.choices[0].delta.content, end="", flush=True)
        if hasattr(chunk, "timings") and chunk.timings:
            print(f"\n\n⚡ {chunk.timings['tokens_per_second']} t/s")

### Python (tool calling)

    response = client.chat.completions.create(
        model="anything",
        messages=[{"role": "user", "content": "What's the weather in Paris?"}],
        tools=[{
            "type": "function",
            "function": {
                "name": "get_weather",
                "description": "Get current weather for a city",
                "parameters": {
                    "type": "object",
                    "properties": {"city": {"type": "string"}},
                    "required": ["city"],
                },
            },
        }],
        tool_choice="auto",
    )

    print(response.choices[0].message.tool_calls)

### curl

    curl -X POST https://your-tunnel.trycloudflare.com/v1/chat/completions \
      -H "Authorization: Bearer sk-your-api-key" \
      -H "Content-Type: application/json" \
      -d '{
        "model": "anything",
        "messages": [{"role": "user", "content": "Hello!"}],
        "max_tokens": 100
      }'

---

## 📚 API Reference

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/v1/chat/completions` | POST | Bearer | Chat completions (streaming + tools) |
| `/v1/completions` | POST | Bearer | Legacy text completions |
| `/v1/models` | GET | Bearer | Returns the currently loaded model |
| `/health` | GET | none | Health check + current model + last stats |

### Supported OpenAI parameters

Every request forwards all standard OpenAI fields to `llama-cpp-python`:

- `messages[]` with roles: `system`, `user`, `assistant`, `tool`, `function`
- `tools` + `tool_choice` (string or object)
- `functions` + `function_call` (legacy)
- `response_format` (`text`, `json_object`, `json_schema`)
- `temperature`, `top_p`, `top_k`, `min_p`
- `max_tokens`, `stop`, `seed`
- `presence_penalty`, `frequency_penalty`
- `stream` (SSE)

### Response extensions

Every response includes a `timings` object:

    {
      "timings": {
        "total_time": 4.213,
        "eval_time": 3.842,
        "completion_tokens": 18,
        "tokens_per_second": 4.69
      }
    }

During streaming, the final chunk carries `timings` + `usage` before `data: [DONE]`.

---

## 🖥 Dashboard

The built-in web dashboard gives you:

- **Model status** — which model is loaded, or "no model loaded"
- **Performance card** — last request's tokens/sec, token count, total time
- **Model Store** — search Hugging Face for single-file GGUF models (including Vision models)
- **Downloader** — manually paste a Hugging Face `repo_id` + `filename`
- **API keys** — copy or regenerate your bearer token (hidden by default)
- **Security** — IP ban after 10 failed login attempts

---

## 🔒 Security

- **First-run setup**: the first visitor creates the admin password
- **Password hashing**: PBKDF2-SHA256, 100,000 iterations, random salt
- **Session cookies**: signed with a per-run random secret
- **API keys**: 48-hex-char, `sk-` prefixed, revocable from dashboard
- **IP Banning**: 10 failed login attempts from a single IP will ban it.
- **Isolated tunnel**: each run gets a fresh Cloudflare subdomain
- **Ephemeral state**: config, key, models, and session die when the job ends

> ⚠️ **Cloudflare Quick Tunnels have no uptime guarantee** and are for experimentation. Do not use this for production workloads or sensitive data.

---

## 📊 Performance Expectations

GitHub-hosted runners have **4 vCPUs** and **~16 GB RAM** (no GPU). Real-world throughput:

| Model | Size | Approx speed |
|-------|------|--------------|
| TinyLlama 1.1B Q8_0 | 1.1 GB | 20–40 t/s |
| Phi-3 mini 3.8B Q4_K_M | 2.3 GB | 10–20 t/s |
| Llama 3.1 8B Q4_K_M | 4.7 GB | 3–8 t/s |
| Llama 3.1 8B Q8_0 | 8.5 GB | 1–4 t/s (swap) |

For faster responses: use smaller models, smaller quantizations (Q4_K_M > Q8_0), or shorter `max_tokens`.

---

## ⚠️ Limitations

| Limitation | Details |
|------------|---------|
| **6-hour job limit** | Free tier kills the job after 6 hours |
| **Ephemeral URL** | Tunnel URL changes on every workflow run |
| **16 GB RAM** | Larger models rely on disk swap → slow |
| **No persistence** | Downloaded models are deleted when the job ends |
| **CPU-only** | No GPU acceleration on free runners |
| **1 model at a time** | Loaded model is unloaded when switching |
| **Quick Tunnel** | No SLA, no custom domain, random URL |

**Want more?** Use a self-hosted runner, upgrade to a named Cloudflare Tunnel, or deploy to a VPS.

---

## ❓ FAQ

<details>
<summary><b>Why is the first install slow?</b></summary>

If pip compiles `llama-cpp-python` from source instead of downloading the wheel, it takes 3–5 minutes. This workflow pins the prebuilt wheel index, so it should install in seconds. If you still see "Building wheel...", the wheel for your Python version may not exist yet — downgrade Python to 3.11 (already set).

</details>

<details>
<summary><b>I got banned from my own dashboard (403 Forbidden)</b></summary>

You entered the wrong password 10 times. Since the app is ephemeral, simply stop the GitHub Action and re-run it. The ban list is stored in memory and resets on restart.

</details>

<details>
<summary><b>The dashboard shows "502 Bad Gateway" / "530"</b></summary>

The workflow has stopped, or the tunnel died. Re-run the workflow from the Actions tab. The new URL will appear in the "Expose via Cloudflare Tunnel" step.

</details>

<details>
<summary><b>`/setup` shows "Internal Server Error"</b></summary>

Check the "Dump proxy log on failure" step's output. Most likely the template directory or config permissions are wrong. If you've been editing `main.py`, verify the `TemplateResponse(request, "x.html", {...})` signature — Starlette 1.x only accepts this form.

</details>

<details>
<summary><b>Where do models get downloaded?</b></summary>

Into `proxy/models/` on the runner. They survive for the duration of the workflow only. To use a model again, re-download it or set up an `actions/cache` step.

</details>

<details>
<summary><b>Does this work with any GGUF model?</b></summary>

Yes. Any `.gguf` file hosted on Hugging Face works. Chat templates are read from the GGUF metadata, so Llama 3, Mistral, Qwen, Phi, Gemma, etc. all behave correctly — but for tool calling, prefer models with native tool-use support (Llama 3.1+, Qwen 2.5, Mistral v0.3+).

</details>

---

## 📄 License

MIT — do what you want, no warranty.

---

## 🙏 Credits

- [llama.cpp](https://github.com/ggerganov/llama.cpp) — the inference engine
- [llama-cpp-python](https://github.com/abetlen/llama-cpp-python) — Python bindings
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/) — free public URLs
- [Hugging Face](https://huggingface.co/models?library=gguf) — the model library

---

**⭐ If this saves you money, give the repo a star.**
