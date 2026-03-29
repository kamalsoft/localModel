# Ollama Custom Qwen Models — Speed, Strict Mode & VS Code Copilot Integration, Complete Setup Guide

> **Production-ready Modelfiles** for three custom Qwen3-Coder personas: a strict code-only engine, a verbose explainer, and a RAG-enhanced codebase reader — all tuned for maximum speed and visible in VS Code Copilot's model selector.

> Covers Ollama **v0.18+**, `ollama launch`, Web Search API, cloud models, VS Code Copilot native integration, and three production-ready custom Modelfiles (Strict Coder · Verbose Explainer · RAG-Enhanced Coder).
> 
---

<img width="595" height="593" alt="image" src="https://github.com/user-attachments/assets/e893b086-a719-4c2b-a16a-dfaa0ba5e375" />
---

## Table of Contents

1. [Prerequisites & Hardware](#1-prerequisites--hardware)
2. [Install & Verify Ollama](#2-install--verify-ollama)
3. [Choose Your Base Model](#3-choose-your-base-model)
4. [Speed & Performance Optimization](#4-speed--performance-optimization)
5. [Custom Modelfiles](#5-custom-modelfiles)
6. [RAG Pipeline Setup](#6-rag-pipeline-setup)
7. [VS Code Copilot Integration](#7-vs-code-copilot-integration)
8. [`ollama launch` — Modern Coding Tools](#8-ollama-launch--modern-coding-tools)
9. [Runtime Tuning & Interactive Commands](#9-runtime-tuning--interactive-commands)
10. [Updating & Rebuilding Models](#10-updating--rebuilding-models)
11. [Quick Reference](#11-quick-reference)
12. [Troubleshooting](#12-troubleshooting)
13. [Recommended Repo Structure](#13-recommended-repo-structure)

---

## 1. Prerequisites & Hardware

| Requirement | Minimum | Recommended |
|---|---|---|
| **Ollama version** | v0.18.0+ | v0.19.0+ (latest) |
| **OS** | macOS 13+, Ubuntu 20.04+, Windows 10+ | macOS 14+, Ubuntu 22.04+, Windows 11 |
| **RAM** | 16 GB | 32 GB+ |
| **VRAM (GPU)** | 8 GB (7B models) | 24 GB+ (30B models) |
| **Disk** | 20 GB free | 100 GB+ SSD |

### VRAM Quick Calculator

| Model | Parameters (Total / Active) | Quantization | Disk Size | VRAM Needed |
|---|---|---|---|---|
| `qwen3-coder:7b` | 7B / 7B (dense) | Q4_K_M | ~4.5 GB | ~6 GB |
| `qwen3-coder:14b` | 14B / 14B (dense) | Q4_K_M | ~9 GB | ~12 GB |
| `qwen3-coder:30b` | 30.5B / 3.3B (MoE, 128 experts) | Q4_K_M | ~19 GB | ~22 GB |
| `qwen3-coder-next` | 80B / 3B (hybrid MoE, 512 experts) | Q4_K_M | ~52 GB | ~56 GB |
| `qwen3-coder:480b` | 480B / 35B (MoE, 160 experts) | Q4_K_M | ~290 GB | ~300 GB |
| `qwen3-coder:480b-cloud` | 480B / 35B (cloud-hosted) | Full | 0 GB | 0 GB |

> **Tip:** If your hardware can't fit a model, use cloud variants (e.g. `qwen3-coder:480b-cloud`) — see [Cloud Models](#cloud-models-zero-vram-fallback).

---

## 2. Install & Verify Ollama

<details>
<summary><strong>macOS</strong></summary>

```bash
brew install ollama
```

</details>

<details>
<summary><strong>Linux</strong></summary>

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

</details>

<details>
<summary><strong>Windows</strong></summary>

Download the installer from [ollama.com/download](https://ollama.com/download) and run it.

</details>

### Verify Installation

```bash
ollama --version
# Expected: ollama version is 0.18.x or higher

ollama serve
# Or equivalently:
ollama start
```

> `ollama start` is an alias for `ollama serve` — both start the local API server on `http://localhost:11434`.

### Verify the Server is Running

```bash
curl http://localhost:11434/
# Expected: "Ollama is running"
```

---

## 3. Choose Your Base Model

### Recommended Models (March 2026)

| Model | Architecture | Context | Best For | Pull Command |
|---|---|---|---|---|
| `qwen3-coder:7b` | Dense | 256K | Fast iteration, low VRAM | `ollama pull qwen3-coder:7b` |
| `qwen3-coder:14b` | Dense | 256K | Balanced quality/speed | `ollama pull qwen3-coder:14b` |
| `qwen3-coder:30b` | MoE (128 experts, 8 active) | 256K | High-quality coding, 24GB+ VRAM | `ollama pull qwen3-coder:30b` |
| `qwen3-coder-next` ⭐ | Hybrid DeltaNet+Attention+MoE (512 experts, 10+1 active) | 256K | Cutting-edge, non-thinking mode only | `ollama pull qwen3-coder-next` |
| `qwen3-coder:480b-cloud` | MoE (cloud) | 256K | Maximum quality, zero VRAM | `ollama pull qwen3-coder:480b-cloud` |

> **Recommendation:** Start with `qwen3-coder:30b` for the best local balance. Use `qwen3-coder-next` if you have 56 GB+ VRAM and want the newest hybrid architecture. Fall back to `qwen3-coder:480b-cloud` for maximum quality without hardware constraints.

### Cloud Models (Zero-VRAM Fallback)

Ollama now supports **cloud-hosted models** (preview, Sep 2025+). These run on Ollama's infrastructure — no local VRAM needed.

```bash
# Pull a cloud model (downloads only a tiny manifest, not weights)
ollama pull qwen3-coder:480b-cloud

# Run it like any local model
ollama run qwen3-coder:480b-cloud

# Use in custom Modelfiles
# FROM qwen3-coder:480b-cloud
```

> **Note:** Cloud models require an internet connection and an Ollama account. They may have usage limits. Local models remain fully offline-capable.

### Pull Your Chosen Base Model

```bash
# Example: pull the 30B model (recommended for most setups)
ollama pull qwen3-coder:30b

# Verify it's available
ollama list
```

---

## 4. Speed & Performance Optimization

Every environment variable below requires **restarting the Ollama server** to take effect. Platform-specific set + restart commands are provided for each.

---

### 4.1 Enable CORS (Required for VS Code)

<details>
<summary><strong>macOS</strong></summary>

```bash
launchctl setenv OLLAMA_ORIGINS "*"
# Restart: quit the Ollama app from the menu bar, then reopen it
```

</details>

<details>
<summary><strong>Linux (systemd)</strong></summary>

```bash
sudo mkdir -p /etc/systemd/system/ollama.service.d
sudo tee /etc/systemd/system/ollama.service.d/override.conf <<'EOF'
[Service]
Environment="OLLAMA_ORIGINS=*"
EOF

sudo systemctl daemon-reload
sudo systemctl restart ollama
```

</details>

<details>
<summary><strong>Windows (PowerShell — Run as Admin)</strong></summary>

```powershell
[System.Environment]::SetEnvironmentVariable("OLLAMA_ORIGINS", "*", "Machine")
# Restart: close Ollama from System Tray → reopen Ollama from Start Menu
```

</details>

---

### 4.2 Enable Flash Attention (Speed Boost)

<details>
<summary><strong>macOS</strong></summary>

```bash
launchctl setenv OLLAMA_FLASH_ATTENTION 1
# Restart: quit Ollama from menu bar → reopen
```

</details>

<details>
<summary><strong>Linux (systemd)</strong></summary>

```bash
# Add to the override.conf (or create if not exists):
sudo tee -a /etc/systemd/system/ollama.service.d/override.conf <<'EOF'
Environment="OLLAMA_FLASH_ATTENTION=1"
EOF

sudo systemctl daemon-reload
sudo systemctl restart ollama
```

</details>

<details>
<summary><strong>Windows (PowerShell — Run as Admin)</strong></summary>

```powershell
[System.Environment]::SetEnvironmentVariable("OLLAMA_FLASH_ATTENTION", "1", "Machine")
# Restart: close Ollama from System Tray → reopen
```

</details>

---

### 4.3 Enable Quantized KV Cache (Memory Savings)

```
OLLAMA_KV_CACHE_TYPE=q8_0
```

Reduces KV cache memory by ~50% with minimal quality loss. Options: `f16` (default), `q8_0`, `q4_0`.

<details>
<summary><strong>macOS</strong></summary>

```bash
launchctl setenv OLLAMA_KV_CACHE_TYPE q8_0
# Restart: quit Ollama from menu bar → reopen
```

</details>

<details>
<summary><strong>Linux (systemd)</strong></summary>

```bash
sudo tee -a /etc/systemd/system/ollama.service.d/override.conf <<'EOF'
Environment="OLLAMA_KV_CACHE_TYPE=q8_0"
EOF

sudo systemctl daemon-reload
sudo systemctl restart ollama
```

</details>

<details>
<summary><strong>Windows (PowerShell — Run as Admin)</strong></summary>

```powershell
[System.Environment]::SetEnvironmentVariable("OLLAMA_KV_CACHE_TYPE", "q8_0", "Machine")
# Restart: close Ollama from System Tray → reopen
```

</details>

---

### 4.4 Keep Models in Memory (Eliminate Cold Starts)

```
OLLAMA_KEEP_ALIVE=24h
```

Keeps the model loaded in memory for the specified duration after the last request. Default is `5m`.

<details>
<summary><strong>macOS</strong></summary>

```bash
launchctl setenv OLLAMA_KEEP_ALIVE "24h"
# Restart: quit Ollama from menu bar → reopen
```

</details>

<details>
<summary><strong>Linux (systemd)</strong></summary>

```bash
sudo tee -a /etc/systemd/system/ollama.service.d/override.conf <<'EOF'
Environment="OLLAMA_KEEP_ALIVE=24h"
EOF

sudo systemctl daemon-reload
sudo systemctl restart ollama
```

</details>

<details>
<summary><strong>Windows (PowerShell — Run as Admin)</strong></summary>

```powershell
[System.Environment]::SetEnvironmentVariable("OLLAMA_KEEP_ALIVE", "24h", "Machine")
# Restart: close Ollama from System Tray → reopen
```

</details>

---

### 4.5 Enable Parallel Requests

```
OLLAMA_NUM_PARALLEL=4
OLLAMA_MAX_LOADED_MODELS=2
```

<details>
<summary><strong>macOS</strong></summary>

```bash
launchctl setenv OLLAMA_NUM_PARALLEL 4
launchctl setenv OLLAMA_MAX_LOADED_MODELS 2
# Restart: quit Ollama from menu bar → reopen
```

</details>

<details>
<summary><strong>Linux (systemd)</strong></summary>

```bash
sudo tee -a /etc/systemd/system/ollama.service.d/override.conf <<'EOF'
Environment="OLLAMA_NUM_PARALLEL=4"
Environment="OLLAMA_MAX_LOADED_MODELS=2"
EOF

sudo systemctl daemon-reload
sudo systemctl restart ollama
```

</details>

<details>
<summary><strong>Windows (PowerShell — Run as Admin)</strong></summary>

```powershell
[System.Environment]::SetEnvironmentVariable("OLLAMA_NUM_PARALLEL", "4", "Machine")
[System.Environment]::SetEnvironmentVariable("OLLAMA_MAX_LOADED_MODELS", "2", "Machine")
# Restart: close Ollama from System Tray → reopen
```

</details>

---

### 4.6 Detect Optimal Thread Count

Run this to find your physical core count (use for `num_thread` in Modelfiles):

```bash
# macOS
sysctl -n hw.physicalcpu

# Linux
lscpu | grep "Core(s) per socket" | awk '{print $NF}'

# Windows (PowerShell)
(Get-CimInstance Win32_Processor).NumberOfCores
```

> Use the output value for the `PARAMETER num_thread` lines in your Modelfiles. Do **not** count hyper-threads.

---

### 4.7 Combined systemd Override (Linux Only)

For convenience, here's a single override file with all recommended settings:

```bash
sudo mkdir -p /etc/systemd/system/ollama.service.d
sudo tee /etc/systemd/system/ollama.service.d/override.conf <<'EOF'
[Service]
Environment="OLLAMA_ORIGINS=*"
Environment="OLLAMA_FLASH_ATTENTION=1"
Environment="OLLAMA_KV_CACHE_TYPE=q8_0"
Environment="OLLAMA_KEEP_ALIVE=24h"
Environment="OLLAMA_NUM_PARALLEL=4"
Environment="OLLAMA_MAX_LOADED_MODELS=2"
EOF

sudo systemctl daemon-reload
sudo systemctl restart ollama
```

---

### 4.8 Verify Environment Variables

```bash
# macOS
launchctl getenv OLLAMA_FLASH_ATTENTION
launchctl getenv OLLAMA_ORIGINS

# Linux
systemctl show ollama.service | grep -i environment

# Windows (PowerShell)
[System.Environment]::GetEnvironmentVariable("OLLAMA_FLASH_ATTENTION", "Machine")
```

---

### 4.9 Warm Up & Verify Model is Loaded

After setting env vars and restarting, warm up your model to pre-load it into memory:

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "strict-coder",
  "prompt": "ping",
  "stream": false
}'
```

Then verify it's loaded:

```bash
ollama ps --verbose
```

> `ollama ps --verbose` shows detailed resource usage: VRAM, RAM, quantization, and context window per loaded model.

---

## 5. Custom Modelfiles

### Context Length Best Practice

Ollama's coding tool integrations (v0.19+) recommend a minimum context window of **65,536 tokens** for agentic coding workflows. All Modelfiles below use `num_ctx 65536`. If you're on constrained hardware, you can reduce to `16384`, but expect reduced performance in multi-file tasks.

---

### Model 1: `strict-coder` — Code-Only Output

Create a file named `Modelfile.strict-coder`:

```dockerfile
# ─────────────────────────────────────────────────
# strict-coder — Pure code output, zero commentary
# ─────────────────────────────────────────────────
FROM qwen3-coder:30b

SYSTEM """You are a strict code generator. Rules:
1. Output ONLY code — no explanations, no markdown, no comments unless inside the code itself.
2. Use the language specified in the prompt. If unspecified, use Python.
3. Follow best practices: type hints, error handling, docstrings.
4. Never apologize, never add disclaimers, never say "here is the code".
5. If the request is ambiguous, make a reasonable assumption and generate code.
6. Always produce complete, runnable code — never use placeholders like '...'.
7. For /no_think requests, skip internal reasoning entirely."""

# ── Speed & Determinism ──
PARAMETER temperature 0.1
PARAMETER top_p 0.85
PARAMETER top_k 20
PARAMETER repeat_penalty 1.15
PARAMETER num_predict 8192
PARAMETER num_ctx 65536
PARAMETER seed 42
# Set num_thread to your physical core count (see Section 4.6)
PARAMETER num_thread 8

# ── Stop sequences ──
PARAMETER stop "<|im_end|>"
PARAMETER stop "<|endoftext|>"
PARAMETER stop "```"
```

```bash
ollama create strict-coder -f Modelfile.strict-coder
ollama show --modelfile strict-coder    # Verify parameters were applied
ollama run strict-coder "Write a Python FastAPI CRUD endpoint for a user model"
```

---

### Model 2: `verbose-explainer` — Detailed Teaching Mode

Create a file named `Modelfile.verbose-explainer`:

```dockerfile
# ─────────────────────────────────────────────────
# verbose-explainer — Teaching mode with deep explanations
# ─────────────────────────────────────────────────
FROM qwen3-coder:30b

SYSTEM """You are a senior software engineering mentor. Rules:
1. Explain concepts thoroughly before showing code.
2. Use analogies and real-world examples to clarify complex ideas.
3. Break down every code block with line-by-line explanations.
4. Discuss trade-offs, alternatives, and potential pitfalls.
5. Use markdown formatting: headings, bullet points, code blocks.
6. Include "What could go wrong?" and "Production tips" sections.
7. If the user's approach has issues, explain why and suggest improvements.
8. End every response with "Next steps" to guide continued learning."""

# ── Creativity & Depth ──
PARAMETER temperature 0.7
PARAMETER top_p 0.9
PARAMETER top_k 50
PARAMETER repeat_penalty 1.1
PARAMETER num_predict 12288
PARAMETER num_ctx 65536
PARAMETER seed 0
# Set num_thread to your physical core count (see Section 4.6)
PARAMETER num_thread 8

# ── Stop sequences ──
PARAMETER stop "<|im_end|>"
PARAMETER stop "<|endoftext|>"
```

```bash
ollama create verbose-explainer -f Modelfile.verbose-explainer
ollama show --modelfile verbose-explainer    # Verify
ollama run verbose-explainer "Explain async/await in Python with practical examples"
```

---

### Model 3: `rag-coder` — Codebase-Aware RAG Model

Create a file named `Modelfile.rag-coder`:

```dockerfile
# ─────────────────────────────────────────────────
# rag-coder — RAG-enhanced codebase reader
# ─────────────────────────────────────────────────
FROM qwen3-coder:30b

SYSTEM """You are a codebase-aware coding assistant powered by retrieval-augmented generation.
Rules:
1. You will receive context chunks from the user's actual codebase before their question.
2. Base your answers ONLY on the provided context + the user's question.
3. Reference specific files, functions, and line patterns from the context.
4. If the context doesn't contain enough information, say so explicitly.
5. Suggest improvements that fit the existing codebase style and conventions.
6. Never hallucinate file names, function signatures, or APIs not in context.
7. When modifying code, show the full updated function/class, not just a diff."""

# ── Balanced for RAG ──
PARAMETER temperature 0.2
PARAMETER top_p 0.9
PARAMETER top_k 30
PARAMETER repeat_penalty 1.12
PARAMETER num_predict 10240
PARAMETER num_ctx 65536
PARAMETER seed 42
# Set num_thread to your physical core count (see Section 4.6)
PARAMETER num_thread 8

# ── Stop sequences ──
PARAMETER stop "<|im_end|>"
PARAMETER stop "<|endoftext|>"
```

```bash
ollama create rag-coder -f Modelfile.rag-coder
ollama show --modelfile rag-coder    # Verify
```

---

### Seed Behavior Reference

| Setting | Behavior | Use Case |
|---|---|---|
| `seed 42` (or any fixed int) | Deterministic — same input → same output | Code generation, testing, reproducibility |
| `seed 0` | Random seed each run | Creative tasks, brainstorming, varied output |
| `seed` (omitted) | Model default (usually random) | General use |

---

## 6. RAG Pipeline Setup

### 6.1 Requirements

Create `requirements.txt`:

```
ollama>=0.6.0
chromadb>=0.4.22
rich>=13.7.0
```

```bash
pip install -r requirements.txt

# Also pull the embedding model
ollama pull nomic-embed-text
```

### 6.2 RAG Script — `rag_query.py`

```python
#!/usr/bin/env python3
"""
RAG Pipeline for rag-coder model.
Indexes your codebase into ChromaDB, retrieves relevant chunks,
and queries the rag-coder model with context.

Usage:
    python rag_query.py --index /path/to/your/project
    python rag_query.py --query "How does the auth middleware work?"
    python rag_query.py --query "Refactor the database connection pool" --top-k 10
"""

import argparse
import hashlib
import os
import sys
from pathlib import Path

import chromadb
import ollama
from rich.console import Console
from rich.markdown import Markdown

# ── Configuration ──────────────────────────────────────────
EMBED_MODEL = "nomic-embed-text"
CHAT_MODEL = "rag-coder"
COLLECTION_NAME = "codebase"
CHUNK_SIZE = 1500        # characters per chunk
CHUNK_OVERLAP = 200      # overlap between chunks
INDEX_DIR = ".rag_index"

SUPPORTED_EXTENSIONS = {
    ".py", ".js", ".ts", ".tsx", ".jsx", ".java", ".go", ".rs",
    ".cpp", ".c", ".h", ".hpp", ".cs", ".rb", ".php", ".swift",
    ".kt", ".scala", ".sh", ".bash", ".yaml", ".yml", ".toml",
    ".json", ".md", ".txt", ".sql", ".html", ".css", ".scss",
    ".vue", ".svelte", ".dockerfile", ".tf", ".hcl",
}

IGNORE_DIRS = {
    "node_modules", ".git", "__pycache__", ".venv", "venv",
    "dist", "build", ".next", ".rag_index", ".mypy_cache",
    "target", "bin", "obj", ".tox", "coverage",
}

console = Console()


def chunk_text(text: str, source: str) -> list[dict]:
    """Split text into overlapping chunks with metadata."""
    chunks = []
    for i in range(0, len(text), CHUNK_SIZE - CHUNK_OVERLAP):
        chunk = text[i : i + CHUNK_SIZE]
        if chunk.strip():
            chunk_id = hashlib.md5(f"{source}:{i}".encode()).hexdigest()
            chunks.append({
                "id": chunk_id,
                "text": chunk,
                "metadata": {"source": source, "offset": i},
            })
    return chunks


def collect_files(project_path: str) -> list[Path]:
    """Recursively collect supported source files."""
    files = []
    for root, dirs, filenames in os.walk(project_path):
        dirs[:] = [d for d in dirs if d not in IGNORE_DIRS]
        for fname in filenames:
            fpath = Path(root) / fname
            if fpath.suffix.lower() in SUPPORTED_EXTENSIONS:
                files.append(fpath)
    return sorted(files)


def index_codebase(project_path: str) -> None:
    """Index the codebase into ChromaDB with embeddings."""
    client = chromadb.PersistentClient(path=INDEX_DIR)

    # Delete existing collection if re-indexing
    try:
        client.delete_collection(COLLECTION_NAME)
    except ValueError:
        pass

    collection = client.create_collection(
        name=COLLECTION_NAME,
        metadata={"hnsw:space": "cosine"},
    )

    files = collect_files(project_path)
    if not files:
        console.print("[red]No supported files found.[/red]")
        sys.exit(1)

    console.print(f"[cyan]Indexing {len(files)} files...[/cyan]")

    all_chunks = []
    for fpath in files:
        try:
            text = fpath.read_text(encoding="utf-8", errors="ignore")
            rel = str(fpath.relative_to(project_path))
            all_chunks.extend(chunk_text(text, rel))
        except Exception as e:
            console.print(f"[yellow]Skipped {fpath}: {e}[/yellow]")

    if not all_chunks:
        console.print("[red]No text chunks generated.[/red]")
        sys.exit(1)

    # Batch embed and insert
    BATCH = 64
    for i in range(0, len(all_chunks), BATCH):
        batch = all_chunks[i : i + BATCH]
        texts = [c["text"] for c in batch]
        ids = [c["id"] for c in batch]
        metas = [c["metadata"] for c in batch]

        embeddings_resp = ollama.embed(model=EMBED_MODEL, input=texts)
        embeddings = embeddings_resp.embeddings

        collection.add(
            ids=ids,
            embeddings=embeddings,
            documents=texts,
            metadatas=metas,
        )
        console.print(f"  Indexed {min(i + BATCH, len(all_chunks))}/{len(all_chunks)} chunks")

    console.print(f"[green]Done! {len(all_chunks)} chunks indexed in '{INDEX_DIR}/'[/green]")


def query_rag(question: str, top_k: int = 5) -> None:
    """Retrieve context and query the rag-coder model."""
    if not os.path.exists(INDEX_DIR):
        console.print("[red]No index found. Run with --index first.[/red]")
        sys.exit(1)

    client = chromadb.PersistentClient(path=INDEX_DIR)
    collection = client.get_collection(COLLECTION_NAME)

    # Embed the question
    q_embedding = ollama.embed(model=EMBED_MODEL, input=[question]).embeddings[0]

    # Retrieve top-k relevant chunks
    results = collection.query(query_embeddings=[q_embedding], n_results=top_k)

    # Build context block
    context_parts = []
    for doc, meta in zip(results["documents"][0], results["metadatas"][0]):
        context_parts.append(f"### File: {meta['source']} (offset {meta['offset']})\n```\n{doc}\n```")

    context_block = "\n\n".join(context_parts)

    # Query the model
    prompt = f"""## Retrieved Codebase Context

{context_block}

## Question
{question}"""

    console.print("[cyan]Querying rag-coder...[/cyan]\n")

    response = ollama.chat(
        model=CHAT_MODEL,
        messages=[{"role": "user", "content": prompt}],
    )

    console.print(Markdown(response.message.content))


def main():
    parser = argparse.ArgumentParser(description="RAG pipeline for rag-coder")
    parser.add_argument("--index", type=str, help="Path to project directory to index")
    parser.add_argument("--query", type=str, help="Question to ask about the codebase")
    parser.add_argument("--top-k", type=int, default=5, help="Number of context chunks (default: 5)")
    args = parser.parse_args()

    if args.index:
        index_codebase(args.index)
    elif args.query:
        query_rag(args.query, top_k=args.top_k)
    else:
        parser.print_help()


if __name__ == "__main__":
    main()
```

### 6.3 `.gitignore`

Add to your project's `.gitignore`:

```
.rag_index/
__pycache__/
*.pyc
```

### 6.4 Web Search API Enhancement (Optional)

Ollama now provides a **built-in Web Search API** (Sep 2025+) that can augment your RAG pipeline with real-time web data. This is useful for the `verbose-explainer` model or any context that needs current documentation.

```python
import ollama

# Requires: export OLLAMA_API_KEY="your_api_key"
# Get your API key at https://ollama.com (free account)

# Search the web
results = ollama.web_search("FastAPI dependency injection best practices", max_results=5)
for r in results.results:
    print(f"  {r.title}: {r.url}")

# Fetch full page content as markdown
page = ollama.web_fetch("https://fastapi.tiangolo.com/tutorial/dependencies/")
print(page.content[:500])
```

> **Tip:** Combine `web_search` results with your local RAG context to give the `rag-coder` model both codebase awareness and current documentation.

---

## 7. VS Code Copilot Integration

### 7.1 Native Integration (Official Method — Ollama v0.18.3+)

As of Ollama v0.18.3, VS Code Copilot **natively integrates** with Ollama. No extensions needed.

1. Open VS Code
2. Open the **Copilot Chat** sidebar (`Ctrl+Shift+I` / `Cmd+Shift+I`)
3. Click the **model dropdown** at the top of the chat panel
4. Click **Manage Models...**
5. Under **Provider**, select or type **Ollama**
6. Your custom models (`strict-coder`, `verbose-explainer`, `rag-coder`) will appear in the model list
7. Select a model and start chatting

### 7.2 Alternative: `settings.json` Configuration

If you prefer explicit configuration, add this to your VS Code `settings.json` (`Ctrl+,` → Open Settings JSON):

```jsonc
{
    "github.copilot.chat.models": [
        {
            "vendor": "ollama",
            "provider": "ollama",
            "family": "strict-coder",
            "id": "strict-coder",
            "name": "Strict Coder (Ollama)",
            "url": "http://localhost:11434"
        },
        {
            "vendor": "ollama",
            "provider": "ollama",
            "family": "verbose-explainer",
            "id": "verbose-explainer",
            "name": "Verbose Explainer (Ollama)",
            "url": "http://localhost:11434"
        },
        {
            "vendor": "ollama",
            "provider": "ollama",
            "family": "rag-coder",
            "id": "rag-coder",
            "name": "RAG Coder (Ollama)",
            "url": "http://localhost:11434"
        }
    ]
}
```

### 7.3 Inline Completions (Optional)

For tab-completion suggestions powered by Ollama:

```jsonc
{
    "github.copilot.advanced.inlineSuggestProvider": "ollama",
    "ollama.model": "strict-coder",
    "ollama.endpoint": "http://localhost:11434"
}
```

### 7.4 Alternative Extension: Ollama Copilot

If the native integration doesn't suit your workflow, install the **[Ollama Copilot](https://marketplace.visualstudio.com/items?itemName=ollama-copilot)** extension from the VS Code Marketplace:

```jsonc
{
    "ollama-copilot.baseUrl": "http://127.0.0.1:11434",
    "ollama-copilot.model": "strict-coder"
}
```

### ⚠️ Privacy Note

> Even with a local Ollama endpoint, GitHub Copilot **may still transmit usage telemetry or prompt metadata** depending on your VS Code and Copilot settings. To minimize this:
>
> 1. Disable telemetry: `"telemetry.telemetryLevel": "off"`
> 2. Review Copilot settings: `"github.copilot.advanced.debug.telemetry": false`
> 3. For fully air-gapped setups, consider using **Continue.dev** or **ollama launch** (Section 8) instead.

---

## 8. `ollama launch` — Modern Coding Tools

**`ollama launch`** (v0.15+, January 2026) is Ollama's flagship command for coding tool integrations. It sets up and runs coding agents with your local (or cloud) models in a single command — no environment variables or config files needed.

### Supported Coding Tools

| Tool | Command | Description |
|---|---|---|
| **Claude Code** | `ollama launch claude` | Anthropic's agentic coding CLI |
| **OpenCode** | `ollama launch opencode` | Terminal-based coding assistant |
| **Codex** | `ollama launch codex` | OpenAI's code generation tool |
| **Droid** | `ollama launch droid` | Android development assistant |
| **OpenClaw** | `ollama launch openclaw` | Open-source coding agent |

### Using Custom Models with `ollama launch`

Your custom Modelfile-based models work seamlessly with `ollama launch`:

```bash
# Launch Claude Code with your strict-coder model
ollama launch claude --model strict-coder

# Launch OpenCode with your verbose-explainer
ollama launch opencode --model verbose-explainer

# Launch with the latest Qwen model
ollama launch claude --model qwen3-coder-next

# Configure a tool without launching it
ollama launch claude --config
```

### When to Use `ollama launch` vs VS Code Copilot

| Feature | `ollama launch` | VS Code Copilot + Ollama |
|---|---|---|
| **Setup complexity** | Zero config | Requires env vars + settings |
| **Interface** | Terminal / agent CLI | VS Code IDE |
| **Agentic capabilities** | Full agent loop (file editing, commands) | Chat + inline completions |
| **Custom models** | ✅ `--model strict-coder` | ✅ Via model selector |
| **Offline** | ✅ Fully local | ⚠️ May send telemetry |
| **Best for** | Terminal-first workflows, agentic coding | IDE-integrated coding |

> **Recommendation:** Use `ollama launch` for agentic, terminal-first coding workflows. Use VS Code Copilot integration for IDE-embedded assistance. Both work with your custom Modelfiles.

---

## 9. Runtime Tuning & Interactive Commands

### 9.1 Interactive `/set` Commands

When running a model interactively (`ollama run <model>`), you can tune parameters on the fly without rebuilding the Modelfile:

```
>>> /set parameter temperature 0.3
Set parameter 'temperature' to '0.3'

>>> /set parameter num_ctx 32768
Set parameter 'num_ctx' to '32768'

>>> /set parameter top_p 0.8
Set parameter 'top_p' to '0.8'

>>> /set parameter seed 42
Set parameter 'seed' to '42'

>>> /set system "You are a Python expert. Output only code."
Set system prompt.

>>> /show parameter
Displays current parameter values.

>>> /show system
Displays current system prompt.
```

> **Note:** `/set` changes are session-only. To make them permanent, update the Modelfile and re-create the model.

### 9.2 `ollama generate` — Scripted / CI Usage

For non-interactive, scripted, or CI/CD usage:

```bash
# Single-shot generation (no conversation context)
ollama generate strict-coder "Write a Python decorator for retry with exponential backoff"

# JSON output (structured responses)
ollama generate strict-coder "Return a JSON schema for a User model" --format json

# Pipe into a file
ollama generate strict-coder "Write a Dockerfile for a Python FastAPI app" > Dockerfile
```

---

## 10. Updating & Rebuilding Models

When the base model is updated by Ollama (e.g. `qwen3-coder:30b` gets a new version), your custom models **do not** auto-update. Rebuild them:

```bash
# 1. Pull the latest base model
ollama pull qwen3-coder:30b

# 2. Rebuild each custom model from its Modelfile
ollama create strict-coder -f Modelfile.strict-coder
ollama create verbose-explainer -f Modelfile.verbose-explainer
ollama create rag-coder -f Modelfile.rag-coder

# 3. Verify each rebuild
ollama show --modelfile strict-coder
ollama show --modelfile verbose-explainer
ollama show --modelfile rag-coder

# 4. (Optional) Clean up old model cache
ollama prune
```

> **Tip:** Add a rebuild script to your repo (see [Recommended Repo Structure](#13-recommended-repo-structure)).

---

## 11. Quick Reference

### Essential Commands

| Command | Description |
|---|---|
| `ollama serve` / `ollama start` | Start the Ollama server |
| `ollama pull <model>` | Download a model |
| `ollama create <name> -f <file>` | Create custom model from Modelfile |
| `ollama run <model>` | Interactive chat session |
| `ollama generate <model> "prompt"` | Single-shot generation (non-interactive) |
| `ollama generate <model> "prompt" --format json` | Structured JSON output |
| `ollama list` | List all local models |
| `ollama ps` | Show running models |
| `ollama ps --verbose` | Detailed resource usage per model |
| `ollama show --modelfile <model>` | Inspect a model's Modelfile |
| `ollama stop <model>` | Unload model from memory (keep server running) |
| `ollama rm <model>` | Delete a model |
| `ollama prune` | Clear model cache |
| `ollama logs` | View server logs |
| `ollama launch <tool> --model <model>` | Launch a coding tool with a specific model |
| `ollama launch <tool> --config` | Configure a coding tool without launching |
| `ollama cp <src> <dst>` | Copy/rename a model |

### API Endpoints

| Endpoint | Method | Description |
|---|---|---|
| `http://localhost:11434/` | GET | Health check |
| `http://localhost:11434/api/generate` | POST | Single-shot generation |
| `http://localhost:11434/api/chat` | POST | Chat completion |
| `http://localhost:11434/api/embed` | POST | Generate embeddings |
| `http://localhost:11434/api/tags` | GET | List models |
| `http://localhost:11434/api/ps` | GET | List running models |
| `https://ollama.com/api/web_search` | POST | Web search (requires API key) |
| `https://ollama.com/api/web_fetch` | POST | Fetch URL content (requires API key) |

### Parameter Tuning Cheat Sheet

| Parameter | Strict Code | Verbose Explainer | RAG Coder | What It Does |
|---|---|---|---|---|
| `temperature` | 0.1 | 0.7 | 0.2 | Randomness (0 = deterministic, 1 = creative) |
| `top_p` | 0.85 | 0.9 | 0.9 | Nucleus sampling threshold |
| `top_k` | 20 | 50 | 30 | Vocabulary cutoff per token |
| `repeat_penalty` | 1.15 | 1.1 | 1.12 | Penalize repeated tokens |
| `num_predict` | 8192 | 12288 | 10240 | Max tokens to generate |
| `num_ctx` | 65536 | 65536 | 65536 | Context window (tokens) |
| `seed` | 42 | 0 | 42 | Reproducibility (42=fixed, 0=random) |

---

## 12. Troubleshooting

<details>
<summary><strong>Model not appearing in VS Code Copilot</strong></summary>

1. Verify the model exists: `ollama list` — your custom models should appear
2. Verify Ollama is running: `curl http://localhost:11434/`
3. Verify CORS is set: check `OLLAMA_ORIGINS` (see Section 4.8)
4. Restart VS Code after changing Ollama settings
5. In Copilot Chat, click model dropdown → Manage Models → ensure "Ollama" provider is selected

</details>

<details>
<summary><strong>Slow first response (cold start)</strong></summary>

1. Set `OLLAMA_KEEP_ALIVE=24h` (Section 4.4)
2. Warm up the model after restart (Section 4.9)
3. Enable Flash Attention (Section 4.2)
4. Enable quantized KV cache (Section 4.3)
5. Check `ollama ps --verbose` to confirm model is loaded in GPU memory

</details>

<details>
<summary><strong>Out of memory (OOM) errors</strong></summary>

1. Use a smaller model: `qwen3-coder:7b` or `qwen3-coder:14b`
2. Reduce `num_ctx` to `16384` or `32768`
3. Enable `OLLAMA_KV_CACHE_TYPE=q4_0` for maximum memory savings
4. Set `OLLAMA_MAX_LOADED_MODELS=1` to prevent multiple models from loading
5. Close other GPU-heavy applications
6. Consider cloud models: `qwen3-coder:480b-cloud` uses zero local VRAM

</details>

<details>
<summary><strong>Model outputs explanations instead of code (strict-coder)</strong></summary>

1. Verify the Modelfile was applied: `ollama show --modelfile strict-coder`
2. Check `temperature` is `0.1` and `top_k` is `20`
3. Re-create the model: `ollama create strict-coder -f Modelfile.strict-coder`
4. Use `/no_think` prefix in prompts to disable internal reasoning

</details>

<details>
<summary><strong>RAG script errors</strong></summary>

1. Ensure `ollama>=0.6.0` is installed: `pip show ollama`
2. Ensure `nomic-embed-text` is pulled: `ollama pull nomic-embed-text`
3. Ensure `rag-coder` model exists: `ollama list | grep rag-coder`
4. Re-index if files changed: `python rag_query.py --index /path/to/project`
5. Check ChromaDB index exists: `ls -la .rag_index/`

</details>

<details>
<summary><strong>Connection refused / server not running</strong></summary>

```bash
# Check if Ollama is running
ollama ps

# Check server logs
ollama logs

# Start the server
ollama serve

# If port is in use, find and kill the process:
# macOS/Linux
lsof -i :11434
kill -9 <PID>

# Windows (PowerShell)
netstat -ano | findstr :11434
taskkill /PID <PID> /F
```

</details>

---

## 13. Recommended Repo Structure

```
ollama-custom-models/
├── README.md                          # This guide
├── modelfiles/
│   ├── Modelfile.strict-coder
│   ├── Modelfile.verbose-explainer
│   └── Modelfile.rag-coder
├── scripts/
│   ├── rag_query.py                   # RAG pipeline
│   ├── rebuild_models.sh              # Rebuild all models after base update
│   └── setup.sh                       # One-command setup
├── vscode/
│   └── settings.json                  # VS Code settings snippet
├── requirements.txt
├── .gitignore
└── LICENSE
```

### `scripts/rebuild_models.sh`

```bash
#!/bin/bash
set -euo pipefail

echo "Pulling latest base model..."
ollama pull qwen3-coder:30b

echo "Rebuilding custom models..."
ollama create strict-coder -f modelfiles/Modelfile.strict-coder
ollama create verbose-explainer -f modelfiles/Modelfile.verbose-explainer
ollama create rag-coder -f modelfiles/Modelfile.rag-coder

echo "Verifying..."
ollama list | grep -E "strict-coder|verbose-explainer|rag-coder"

echo "All models rebuilt successfully."
```

```bash
chmod +x scripts/rebuild_models.sh
./scripts/rebuild_models.sh
```

### `scripts/setup.sh`

```bash
#!/bin/bash
set -euo pipefail

echo "=== Ollama Custom Models Setup ==="

# 1. Check Ollama is installed
if ! command -v ollama &>/dev/null; then
    echo "Ollama not found. Install from https://ollama.com/download"
    exit 1
fi

echo "Ollama version: $(ollama --version)"

# 2. Pull base model + embedding model
echo "Pulling models..."
ollama pull qwen3-coder:30b
ollama pull nomic-embed-text

# 3. Create custom models
echo "Creating custom models..."
ollama create strict-coder -f modelfiles/Modelfile.strict-coder
ollama create verbose-explainer -f modelfiles/Modelfile.verbose-explainer
ollama create rag-coder -f modelfiles/Modelfile.rag-coder

# 4. Install Python dependencies
echo "Installing Python dependencies..."
pip install -r requirements.txt

# 5. Verify
echo ""
echo "=== Setup Complete ==="
echo "Available custom models:"
ollama list | grep -E "strict-coder|verbose-explainer|rag-coder"
echo ""
echo "Next steps:"
echo "  1. Set environment variables (see README Section 4)"
echo "  2. Open VS Code → Copilot Chat → Model dropdown → Manage Models → Ollama"
echo "  3. Index your codebase: python scripts/rag_query.py --index /path/to/project"
```

---

## Recommended Repo Structure

```
repo/
├── ollama-custom-qwen-models.md   ← this file
├── Modelfile.strict-coder
├── Modelfile.explainer
├── Modelfile.rag-coder
├── rag_codebase.py
├── requirements.txt
├── .gitignore                     ← include .rag_index/
└── README.md
```

---

## License

MIT — use freely, attribute if you share.
