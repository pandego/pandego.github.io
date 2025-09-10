---
date: 2025-07-03
authors:
  - miguelmirandadias
categories:
  - Tools
  - Python
  - Development
description: MCP is a way to help your LLM-powered apps interact with the outside world — tools, databases, documents, APIs — in a standard and structured way.
---

# From Skeptic to Believer: Unpacking the Model Context Protocol

I’ll admit it , when I first heard about the **Model Context Protocol (MCP)**, my eyes nearly rolled out of my head. Having worn the data scientist hat for over a decade, I’ve watched plenty of hyped frameworks come and go. But curiosity got the better of me. I dug in, tried it out, and well, I was pleasantly surprised. **It turns out MCP actually is useful!**

<!-- more -->

---
> *Condensed mini‑blog from my piece on the Model Context Protocol (MCP).*

## MCP in one breath

**It’s a protocol, not a framework.** Think *USB‑C for LLM apps*.

* **Host**: your LLM application (chatbot, editor plugin, desktop app). *The decider.*
* **Client**: the in‑app component that speaks MCP to servers. *The messenger.*
* **Server**: exposes **Resources** (read‑only data), **Tools** (actions), and **Prompts** (reusable templates). *The do‑er.*

One server can power many apps; one app can connect to many servers. No model/vendor lock‑in.

---

## The three primitives (with bite‑sized code)

### 1) Resources — "GET me context"

Read‑only handles to data you want the model to see.

```python
from mcp.server.fastmcp import FastMCP
mcp = FastMCP("My App")

@mcp.resource("config://app")
def get_config() -> str:
    return "App configuration here"

@mcp.resource("users://{user_id}/profile")
def get_user_profile(user_id: str) -> str:
    return f"Profile data for user {user_id}"
```

Use for configs, user metadata, document contents, preloaded business context.

### 2) Tools — "Do the thing"

Side‑effectful actions or computations (sync/async).

```python
import httpx
from mcp.server.fastmcp import FastMCP
mcp = FastMCP("My App")

@mcp.tool()
def calculate_bmi(weight_kg: float, height_m: float) -> float:
    return weight_kg / (height_m ** 2)

@mcp.tool()
async def fetch_weather(city: str) -> str:
    async with httpx.AsyncClient() as client:
        r = await client.get(f"https://api.weather.com/{city}")
        return r.text
```

Great for API calls, business logic, CRUD, automation.

### 3) Prompts — "Stop rewriting templates"

Server‑defined, reusable prompt patterns.

```python
from mcp.server.fastmcp import FastMCP
from mcp.server.fastmcp.prompts import base
mcp = FastMCP("My App")

@mcp.prompt()
def review_code(code: str) -> str:
    return f"Please review this code:\n\n{code}"

@mcp.prompt()
def debug_error(error: str) -> list[base.Message]:
    return [
        base.UserMessage("I'm seeing this error:"),
        base.UserMessage(error),
        base.AssistantMessage("I'll help debug that. What have you tried so far?"),
    ]
```

Handy for code review, support workflows, query templates, standardized outputs.

---

## A practical build: wrap an MLflow model with MCP

Goal: expose a wine‑quality predictor (served via **MLflow**) to any MCP‑compatible host (e.g., Claude Desktop).

**Tool:** `predict_wine_quality`

```python
# server.py
@mcp.tool(name="predict_wine_quality", description="Predict wine quality using MLflow API")
async def predict_wine_quality(inputs: list[list[float]], columns: list[str]) -> list[float]:
    payload = {"dataframe_split": {"data": inputs, "columns": columns}}
    async with httpx.AsyncClient() as client:
        resp = await client.post(MLFLOW_URL, json=payload)
    return resp.json()["predictions"]
```

**Resource:** example payload to guide users

```python
@mcp.resource(
    uri="wine://example",
    name="wine_quality_example",
    description="Example wine quality inputs and outputs",
    mime_type="application/json",
)
def get_input_example() -> str:
    import json
    return json.dumps({
        "columns": [
            "fixed acidity", "volatile acidity", "citric acid", "residual sugar",
            "chlorides", "free sulfur dioxide", "total sulfur dioxide",
            "density", "pH", "sulphates", "alcohol"
        ],
        "data": [
            [7.4, 0.7, 0, 1.9, 0.076, 11, 34, 0.9978, 3.51, 0.56, 9.4],
            [7.8, 0.88, 0, 2.6, 0.098, 25, 67, 0.9968, 3.2, 0.68, 9.8]
        ]
    }, indent=2)
```

**Prompt:** tidy the output for chat

```python
@mcp.prompt(name="format_predictions", description="Format wine quality predictions for chatbot")
def format_predictions(predictions: list[float]) -> str:
    formatted = "\n".join(f"- Sample {i+1}: **{s:.2f}/10**" for i, s in enumerate(predictions))
    return f"## Predicted Wine Quality Scores\n\n{formatted}"
```

---

## Dev UX: test, containerize, connect

**Local dev & inspector**

```bash
mcp dev src/mcp_server_mlflow/server.py
```

**Containerize it (recommended)**

```dockerfile
FROM python:3.11-slim-bookworm
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential curl git && rm -rf /var/lib/apt/lists/*
COPY pyproject.toml uv.lock README.md ./
COPY src/ ./src/
RUN pip install uv && uv venv && uv sync
RUN useradd -ms /bin/bash appuser
USER appuser
ENV PATH="/app/.venv/bin:$PATH"
ENV MLFLOW_URL="http://host.docker.internal:1234/invocations"
ENTRYPOINT ["mcp-server-mlflow"]
```

Build it:

```bash
docker build -t mcp/wine .
```

**Wire it to Claude Desktop (or any MCP host)**
Option A — Docker:

```jsonc
{
  "mcpServers": {
    "My Wine Quality Server (docker)": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "--init", "mcp/wine"]
    }
  }
}
```

Option B — `uv`:

```jsonc
{
  "mcpServers": {
    "My Wine Quality Server (uv)": {
      "command": "uv",
      "args": ["--directory", "/PATH/TO/PROJECT", "run", "mcp-server-mlflow"]
    }
  }
}
```

---

## When MCP shines (and when it’s overkill)

**Use MCP when:**

* You have multiple apps/agents that need the same tools/data/prompts.
* You want *model‑agnostic* integrations (swap providers without rewiring).
* Teams should share capabilities via a common, versionable interface.

**Skip MCP when:**

* It’s a one‑off script with a single hardcoded API call.
* There’s no reuse across apps or teammates.

---

## Takeaways

* MCP standardizes how LLM apps access **resources**, **tools**, and **prompts**.
* It trades brittle per‑app glue for **composable**, **reusable**, **portable** servers.
* The Python SDK + `mcp dev` + Docker make it straightforward to ship.

---

## Wrap‑up

I started a skeptic. After wiring a real MLflow model through MCP and dropping it into a chat host, I’m… converted. If your LLM features are stuck in “autocomplete” mode, MCP is the cleanest path I’ve seen to make them **world‑aware** without bespoke spaghetti.

---

## 📖 Read the Full Article

<div class="medium-card" style="border: 1px solid var(--md-default-fg-color--lightest, #e1e5e9); border-radius: 8px; padding: 16px; margin: 20px 0; background: var(--md-code-bg-color, #f8f9fa); transition: all 0.2s ease;">
  <div style="display: flex; align-items: center; gap: 12px;">
    <div style="flex-shrink: 0;">
      <!-- Replace with your article image -->
      <img src="https://miro.medium.com/v2/resize:fit:4800/format:webp/1*0A-asKQ7uOC6gswthNoh3A.png" alt="Article Preview" style="width: 80px; height: 80px; border-radius: 8px; object-fit: cover; border: 1px solid var(--md-default-fg-color--lightest, #e1e5e9);">
    </div>
    <div style="flex: 1;">
      <h4 style="margin: 0 0 8px 0; font-size: 16px; color: var(--md-default-fg-color, #1a1a1a);">From Skeptic to Believer: Unpacking the Model Context Protocol</h4>
      <!-- Replace with your custom description -->
      <p style="margin: 0 0 12px 0; font-size: 14px; color: var(--md-default-fg-color--light, #6b7280); line-height: 1.4;">MCP is a powerful open protocol that lets LLM apps connect to tools and data sources in a standardized, plug-and-play way—kind of like REST for AI.</p>
      <a href="https://medium.com/data-science-collective/from-skeptic-to-believer-1a395066387f" target="_blank" style="display: inline-flex; align-items: center; gap: 4px; color: var(--md-accent-fg-color, #059669); text-decoration: none; font-weight: 500; font-size: 14px; transition: color 0.2s ease;">
        📖 Full article available on Medium
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M7 17L17 7M17 7H7M17 7V17"/>
        </svg>
      </a>
    </div>
  </div>
</div>
