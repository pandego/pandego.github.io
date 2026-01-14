---
date: 2023-03-05
authors:
  - miguelmirandadias
categories:
  - Tools
  - Python
  - Development
description: Explore alternatives to OpenAI's services and learn how to leverage open-source models for your AI projects.
---

# No OpenAI Account, No Problem!

If you want the OpenAI developer experience without the OpenAI account (or tokens), Ollama now exposes an **OpenAI-style `/v1` endpoint**. That means you can point existing clients and frameworks at your **local** models and ship.

<!-- more -->

---
> *Condensed mini-blog from my piece on crafting your own OpenAI-compatible API with Ollama.*

> *Read full article on [Medium](https://medium.com/data-science-collective/no-openai-account-no-problem-8a5835739572).*

## Why this rocks

* **Drop‑in compatibility:** Keep using the OpenAI SDKs and patterns.
* **Local first:** Your data and prompts stay on your machine.
* **Costs:** \$0 per token, just your hardware.

## 1) Spin up Ollama with Docker

**CPU only**

```bash
docker run -d -v /data/ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

**GPU (NVIDIA)**

1. Install **NVIDIA Container Toolkit**.
2. Run with GPU access:

```bash
docker run -d --gpus=all -v /data/ollama:/root/.ollama --restart always -p 11434:11434 --name ollama ollama/ollama
```

*Pro tip:* Want specific GPUs? Use `--gpus "device=0,1"`.

**Where are models stored?** In this compose, they’ll live on your host at `/data/ollama`.

## 2) Sanity‑check Ollama

Run a model inside the container:

```bash
docker exec -it ollama ollama run llama2
# >>> Send a message (/? for help)
```

Hit the OpenAI‑style chat completions endpoint:

```bash
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
        "model": "llama2",
        "messages": [
          {"role": "system", "content": "You are a helpful assistant."},
          {"role": "user",   "content": "Hello!"}
        ]
      }'
```

If you get a response with `choices[0].message.content`, you’re golden.

## 3) Build a local “OpenAI” chatbot

**Requirements**

```txt
streamlit>=1.28
langchain>=0.0.217
openai>=1.2
duckduckgo-search
anthropic>=0.3.0
trubrics>=1.4.3
streamlit-feedback
```

**Minimal app: `Chatbot.py`**

```python
from openai import OpenAI
import streamlit as st

# This key is required by the SDK but unused by Ollama; leave any string
OPENAI_API_KEY = "ollama-baby"

st.title("Chatbot")
st.caption("A Streamlit chatbot powered by OpenAI API... I mean Ollama!!!")

if "messages" not in st.session_state:
    st.session_state["messages"] = [
        {"role": "assistant", "content": "How can I help you?"}
    ]

# Show history
for msg in st.session_state["messages"]:
    st.chat_message(msg["role"]).write(msg["content"])

# Input
prompt = st.chat_input("Say something…")
if prompt:
    client = OpenAI(
        api_key=OPENAI_API_KEY,
        base_url="http://localhost:11434/v1",  # ← point at Ollama
    )

    st.session_state["messages"].append({"role": "user", "content": prompt})
    st.chat_message("user").write(prompt)

    response = client.chat.completions.create(
        model="llama2",
        messages=st.session_state["messages"],
    )
    msg = response.choices[0].message.content

    st.session_state["messages"].append({"role": "assistant", "content": msg})
    st.chat_message("assistant").write(msg)
```

Run it:

```bash
streamlit run Chatbot.py
```

![Chatbot](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*BzXcDnH_wHptJa1S42blGQ.png)
Still not a standup comedian 😬

### Notes

* You can swap `llama2` for any local model you’ve pulled with Ollama.
* Keep an eye on **VRAM/CPU** footprints; bigger models need beefier hardware.
* LangChain, agents, and retrieval components can ride along since the client looks like OpenAI.

## Wrap‑up

With Ollama’s `/v1` endpoint, you can prototype and ship **OpenAI‑compatible** apps **locally**. The DX you know, the privacy you want, and zero token anxiety.

---

## 📖 Read the Full Article

<div class="medium-card" style="border: 1px solid var(--md-default-fg-color--lightest, #e1e5e9); border-radius: 8px; padding: 16px; margin: 20px 0; background: var(--md-code-bg-color, #f8f9fa); transition: all 0.2s ease;">
  <div style="display: flex; align-items: center; gap: 12px;">
    <div style="flex-shrink: 0;">
      <!-- Replace with your article image -->
      <img src="https://miro.medium.com/v2/resize:fit:4800/format:webp/1*KdlhJSmqmQJGWlg2vzD0sQ.png" alt="Article Preview" style="width: 80px; height: 80px; border-radius: 8px; object-fit: cover; border: 1px solid var(--md-default-fg-color--lightest, #e1e5e9);">
    </div>
    <div style="flex: 1;">
      <h4 style="margin: 0 0 8px 0; font-size: 16px; color: var(--md-default-fg-color, #1a1a1a);">No OpenAI Account, No Problem! Crafting Your Own OpenAI API with Ollama 🦙</h4>
      <!-- Replace with your custom description -->
      <p style="margin: 0 0 12px 0; font-size: 14px; color: var(--md-default-fg-color--light, #6b7280); line-height: 1.4;">Explore alternatives to OpenAI's services and learn how to leverage open-source models for your AI projects.</p>
      <a href="https://medium.com/@miguelmirandadias/no-openai-account-no-problem-8a5835739572" target="_blank" style="display: inline-flex; align-items: center; gap: 4px; color: var(--md-accent-fg-color, #059669); text-decoration: none; font-weight: 500; font-size: 14px; transition: color 0.2s ease;">
        📖 Full article available on Medium
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M7 17L17 7M17 7H7M17 7V17"/>
        </svg>
      </a>
    </div>
  </div>
</div>
