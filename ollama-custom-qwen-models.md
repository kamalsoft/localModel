
<img width="595" height="593" alt="image" src="https://github.com/user-attachments/assets/e893b086-a719-4c2b-a16a-dfaa0ba5e375" />

---

# Ollama + VS Code Copilot: Custom Qwen Models — Complete Setup Guide

> Build three custom Ollama models with tuned system prompts, a RAG pipeline that reads your codebase, and full VS Code Copilot integration — all running locally on your Mac.
>
> **Last verified:** March 2026 · **Ollama:** v0.18+ · **All model tags verified against [ollama.com/library](https://ollama.com/library)**

---

## Table of Contents

- [1. Overview & What You Get](#1-overview--what-you-get)
- [2. Prerequisites](#2-prerequisites)
- [3. Install & Launch Ollama](#3-install--launch-ollama)
- [4. Choose & Pull Your Base Model](#4-choose--pull-your-base-model)
- [5. Speed & Performance Optimization](#5-speed--performance-optimization)
- [6. Create Custom Modelfiles](#6-create-custom-modelfiles)
- [7. RAG Pipeline Setup](#7-rag-pipeline-setup)
- [8. VS Code Copilot Integration](#8-vs-code-copilot-integration)
- [9. Ollama Launch — Agent Mode](#9-ollama-launch--agent-mode)
- [10. Quick Test Commands](#10-quick-test-commands)
- [11. Parameter Reference](#11-parameter-reference)
- [12. Automation Scripts](#12-automation-scripts)
- [13. Troubleshooting](#13-troubleshooting)
- [14. Quick Reference Card](#14-quick-reference-card)
- [15. Privacy Note](#15-privacy-note)
- [16. Useful Links](#16-useful-links)

---

## 1. Overview & What You Get

This guide sets up three custom Ollama models, each with a distinct personality and tuning, plus a RAG pipeline and VS Code integration.

| Model Name | Purpose | Temperature | Seed | Behavior |
|---|---|---|---|---|
| `strict-coder` | Code-only output — zero prose | 0.1 | 42 (deterministic) | Returns raw code, no explanations |
| `verbose-explainer` | Detailed teaching mode | 0.7 | 0 (random) | Step-by-step explanations with code |
| `rag-coder` | Codebase-aware answers via RAG | 0.2 | 42 (deterministic) | References your actual project files |

All three models appear in VS Code Copilot's model picker and work with `ollama launch` for agentic coding tools.

---

## 2. Prerequisites

| Requirement | Details |
|---|---|
| **Ollama** | v0.18.0+ ([ollama.com/download](https://ollama.com/download)) |
| **VS Code** | v1.250+ with GitHub Copilot extension |
| **Python** | 3.9+ (for RAG pipeline only) |
| **OS** | macOS 13+, Ubuntu 20.04+, or Windows 10+ |

### RAM Recommendations

| Your RAM | Recommended Base Model | Pull Command |
|---|---|---|
| 16 GB | `qwen3:14b` (9.3 GB, 40K ctx, dense) | `ollama pull qwen3:14b` |
| 24 GB+ | `qwen3-coder:30b` (19 GB, 256K ctx, MoE) ⭐ | `ollama pull qwen3-coder:30b` |
| 36 GB+ | `qwen3-coder:30b` with large context windows | `ollama pull qwen3-coder:30b` |
| 64 GB+ | `qwen3-coder-next` (52 GB, 256K ctx, hybrid MoE) | `ollama pull qwen3-coder-next` |
| Any (cloud) | `qwen3-coder:480b-cloud` (zero local VRAM) | `ollama pull qwen3-coder:480b-cloud` |

> **Why `qwen3-coder:30b` for 24 GB?** Despite the "30B" name, it's a **Mixture-of-Experts** model that only activates **3.3B parameters per token**. This makes it fast and memory-efficient while maintaining high code quality with a massive 256K context window.

---

## 3. Install & Launch Ollama

### macOS

```bash
brew install ollama
# Or download from https://ollama.com/download

# Launch the app (runs as a menu bar service):
open -a Ollama

# Verify:
ollama --version
```

### Linux

```bash
curl -fsSL https://ollama.com/install.sh | sh

# Start the service:
sudo systemctl start ollama
sudo systemctl enable ollama

# Verify:
ollama --version
```

### Windows (PowerShell)

```powershell
# Download and run installer from https://ollama.com/download
# Ollama starts automatically as a background service after installation

# Verify:
ollama --version
```

### Verify the Server

```bash
curl http://localhost:11434/
# Expected: "Ollama is running"
```

> **Note:** `ollama start` is an alias for `ollama serve` — both start the local API server.

---

## 4. Choose & Pull Your Base Model

### Verified Model Tags (from [ollama.com/library](https://ollama.com/library))

#### qwen3-coder (Coding-Specialized MoE)

| Tag | Disk Size | Context | Architecture |
|---|---|---|---|
| `qwen3-coder:30b` (latest) ⭐ | 19 GB | 256K | MoE: 30B total, 3.3B active per token |
| `qwen3-coder:30b-a3b-q8_0` | 32 GB | 256K | Same model, Q8_0 quantization |
| `qwen3-coder:30b-a3b-fp16` | 61 GB | 256K | Same model, FP16 full precision |
| `qwen3-coder:480b` | 290 GB | 256K | MoE: 480B total (250 GB+ RAM needed) |
| `qwen3-coder:480b-cloud` | 0 GB (cloud) | 256K | Cloud-hosted, zero local resources |

#### qwen3-coder-next (Newer, Agentic-Focused)

| Tag | Disk Size | Context | Architecture |
|---|---|---|---|
| `qwen3-coder-next:latest` | 52 GB | 256K | Hybrid MoE: 80B total, 3B active |
| `qwen3-coder-next:q8_0` | 85 GB | 256K | Same, Q8_0 quantization |
| `qwen3-coder-next:cloud` | 0 GB (cloud) | 256K | Cloud-hosted |

#### qwen3 (General-Purpose — Smaller Sizes Available)

| Tag | Disk Size | Context |
|---|---|---|
| `qwen3:0.6b` | 523 MB | 40K |
| `qwen3:1.7b` | 1.4 GB | 40K |
| `qwen3:4b` | 2.5 GB | 256K |
| `qwen3:8b` (latest) | 5.2 GB | 40K |
| `qwen3:14b` | 9.3 GB | 40K |
| `qwen3:30b` | 19 GB | 256K |
| `qwen3:32b` | 20 GB | 40K |

> **Warning:** There is **no** `qwen3-coder:7b` or `qwen3-coder:14b`. These tags do not exist. The smallest coder-specific model is `qwen3-coder:30b`. For smaller machines, use `qwen3:14b` or `qwen3:8b` as the base.

### Pull Your Base Model

```bash
# Recommended for most setups (24 GB+ RAM):
ollama pull qwen3-coder:30b

# Alternative for 16 GB machines:
ollama pull qwen3:14b

# Cloud-powered (no local GPU needed):
ollama pull qwen3-coder:480b-cloud

# Embedding model for RAG pipeline:
ollama pull nomic-embed-text

# Verify everything downloaded:
ollama list
```

---

## 5. Speed & Performance Optimization

> **Every variable below requires restarting Ollama to take effect.** Platform-specific set + restart commands are provided.

### macOS (launchctl)

```bash
# ── Set all variables ──
launchctl setenv OLLAMA_FLASH_ATTENTION 1        # Faster inference, lower VRAM
launchctl setenv OLLAMA_KV_CACHE_TYPE q8_0       # ~40% memory savings on KV cache
launchctl setenv OLLAMA_ORIGINS "*"              # Allow VS Code and other apps to connect
launchctl setenv OLLAMA_KEEP_ALIVE 24h           # Keep model loaded for 24 hours
launchctl setenv OLLAMA_NUM_PARALLEL 4           # Handle 4 concurrent requests
launchctl setenv OLLAMA_MAX_LOADED_MODELS 2      # Max 2 models in memory at once

# ── Restart Ollama ──
pkill -f Ollama
sleep 2
open -a Ollama

# ── Verify a variable took effect ──
launchctl getenv OLLAMA_FLASH_ATTENTION
# Should print: 1
```

### Linux (systemd)

```bash
# ── Create override file with all variables ──
sudo mkdir -p /etc/systemd/system/ollama.service.d
sudo tee /etc/systemd/system/ollama.service.d/override.conf << 'EOF'
[Service]
Environment="OLLAMA_FLASH_ATTENTION=1"
Environment="OLLAMA_KV_CACHE_TYPE=q8_0"
Environment="OLLAMA_ORIGINS=*"
Environment="OLLAMA_KEEP_ALIVE=24h"
Environment="OLLAMA_NUM_PARALLEL=4"
Environment="OLLAMA_MAX_LOADED_MODELS=2"
EOF

# ── Restart Ollama ──
sudo systemctl daemon-reload
sudo systemctl restart ollama

# ── Verify ──
systemctl show ollama.service | grep -i environment
```

### Windows (PowerShell — Run as Admin)

```powershell
# ── Set all variables ──
[System.Environment]::SetEnvironmentVariable("OLLAMA_FLASH_ATTENTION", "1", "Machine")
[System.Environment]::SetEnvironmentVariable("OLLAMA_KV_CACHE_TYPE", "q8_0", "Machine")
[System.Environment]::SetEnvironmentVariable("OLLAMA_ORIGINS", "*", "Machine")
[System.Environment]::SetEnvironmentVariable("OLLAMA_KEEP_ALIVE", "24h", "Machine")
[System.Environment]::SetEnvironmentVariable("OLLAMA_NUM_PARALLEL", "4", "Machine")
[System.Environment]::SetEnvironmentVariable("OLLAMA_MAX_LOADED_MODELS", "2", "Machine")

# ── Restart: close Ollama from System Tray → reopen from Start Menu ──

# ── Verify ──
[System.Environment]::GetEnvironmentVariable("OLLAMA_FLASH_ATTENTION", "Machine")
```

### Environment Variable Reference

| Variable | Value | Effect |
|---|---|---|
| `OLLAMA_FLASH_ATTENTION` | `1` | Enables Flash Attention — faster inference, lower VRAM |
| `OLLAMA_KV_CACHE_TYPE` | `q8_0` | Quantizes KV cache — ~40% memory savings (requires flash attention) |
| `OLLAMA_ORIGINS` | `*` | Allows VS Code, web UIs, and other apps to connect |
| `OLLAMA_KEEP_ALIVE` | `24h` | Keeps model loaded for 24 hours (no reload delay). Default: `5m` |
| `OLLAMA_NUM_PARALLEL` | `4` | Handles up to 4 concurrent requests per model |
| `OLLAMA_MAX_LOADED_MODELS` | `2` | Max models in memory at once (adjust based on RAM) |

### Detect Your Optimal Thread Count

```bash
# macOS — performance cores:
sysctl -n hw.perflevel0.logicalcpu

# Linux:
lscpu | grep "Core(s) per socket" | awk '{print $NF}'

# Windows (PowerShell):
(Get-CimInstance Win32_Processor).NumberOfCores
```

> Use this value for the `PARAMETER num_thread` lines in your Modelfiles. **Do not count hyper-threads.**

---

## 6. Create Custom Modelfiles

A Modelfile is like a Dockerfile for LLMs — it defines the base model, system prompt, and tuning parameters.

### Directory Setup

```bash
mkdir -p ~/ollama-models
cd ~/ollama-models
```

---

### Model 1: `strict-coder` — Code-Only Output

```bash
cat > ~/ollama-models/Modelfile.strict-coder << 'MODELFILE'
# ─────────────────────────────────────────────────
# strict-coder — Pure code output, zero commentary
# ─────────────────────────────────────────────────
FROM qwen3-coder:30b

PARAMETER temperature 0.1
PARAMETER seed 42
PARAMETER top_p 0.9
PARAMETER top_k 20
PARAMETER repeat_penalty 1.1
PARAMETER num_ctx 65536
PARAMETER num_predict -1
# Set num_thread to your physical core count (see Section 5)
PARAMETER num_thread 8

PARAMETER stop "<|im_end|>"
PARAMETER stop "<|endoftext|>"

SYSTEM """You are a strict code generator. Rules:
1. Output ONLY code — no explanations, no markdown fences, no comments unless inside the code itself.
2. If the request is ambiguous, pick the most common interpretation and code it.
3. Use modern best practices, proper error handling, and type hints where applicable.
4. Match the language/framework of the question. If unspecified, default to TypeScript.
5. Never apologize, never say "here is the code", never add preamble or postscript.
6. For multi-file responses, separate with a comment: // --- filename.ext ---
7. Always produce complete, runnable code — never use placeholders like '...' or 'TODO'.
"""
MODELFILE
```

---

### Model 2: `verbose-explainer` — Teaching Mode

```bash
cat > ~/ollama-models/Modelfile.verbose-explainer << 'MODELFILE'
# ─────────────────────────────────────────────────
# verbose-explainer — Teaching mode with deep explanations
# ─────────────────────────────────────────────────
FROM qwen3-coder:30b

PARAMETER temperature 0.7
PARAMETER seed 0
PARAMETER top_p 0.95
PARAMETER top_k 40
PARAMETER repeat_penalty 1.0
PARAMETER num_ctx 65536
PARAMETER num_predict -1
# Set num_thread to your physical core count (see Section 5)
PARAMETER num_thread 8

PARAMETER stop "<|im_end|>"
PARAMETER stop "<|endoftext|>"

SYSTEM """You are a senior software engineer and patient teacher. Rules:
1. Explain concepts step-by-step, as if teaching a mid-level developer.
2. Always show code examples with detailed inline comments.
3. Explain WHY each decision was made, not just WHAT the code does.
4. Highlight potential pitfalls, edge cases, and performance considerations.
5. Use analogies when explaining complex concepts.
6. Structure responses with clear headings: Overview → Code → Explanation → Gotchas.
7. When reviewing code, be constructive — suggest improvements with before/after examples.
8. Include time/space complexity analysis for algorithms.
"""
MODELFILE
```

---

### Model 3: `rag-coder` — Codebase-Aware Mode

```bash
cat > ~/ollama-models/Modelfile.rag-coder << 'MODELFILE'
# ─────────────────────────────────────────────────
# rag-coder — RAG-enhanced codebase reader
# ─────────────────────────────────────────────────
FROM qwen3-coder:30b

PARAMETER temperature 0.2
PARAMETER seed 42
PARAMETER top_p 0.9
PARAMETER top_k 30
PARAMETER repeat_penalty 1.1
PARAMETER num_ctx 65536
PARAMETER num_predict -1
# Set num_thread to your physical core count (see Section 5)
PARAMETER num_thread 8

PARAMETER stop "<|im_end|>"
PARAMETER stop "<|endoftext|>"

SYSTEM """You are a codebase-aware coding assistant powered by retrieval-augmented generation.
Rules:
1. When context from the codebase is provided, reference specific files, functions, and patterns found in it.
2. Follow the existing code style, naming conventions, and architectural patterns of the project.
3. When suggesting changes, specify the exact file and location to modify.
4. If the retrieved context is insufficient, say what additional files you would need to see.
5. Prioritize consistency with the existing codebase over theoretical best practices.
6. Never hallucinate file names, function signatures, or APIs not present in context.
7. Always consider how your suggestion impacts other parts of the codebase.
"""
MODELFILE
```

---

### Build All Three Models

```bash
cd ~/ollama-models

ollama create strict-coder -f Modelfile.strict-coder
ollama create verbose-explainer -f Modelfile.verbose-explainer
ollama create rag-coder -f Modelfile.rag-coder

# Verify all models are created:
ollama list
```

Expected output should show `strict-coder`, `verbose-explainer`, and `rag-coder` alongside the base `qwen3-coder:30b`.

> **For 16 GB machines:** Replace `FROM qwen3-coder:30b` with `FROM qwen3:14b` in all three Modelfiles and reduce `num_ctx` to `16384`.

### Verify a Model's Configuration

```bash
ollama show --modelfile strict-coder
```

This confirms the system prompt, parameters, and base model were applied correctly.

---

## 7. RAG Pipeline Setup

RAG (Retrieval-Augmented Generation) indexes your project files into a vector database, then retrieves relevant code snippets when you ask questions — giving the model awareness of your actual codebase.

### Install Dependencies

```bash
pip install chromadb ollama
ollama pull nomic-embed-text
```

### `requirements.txt`

```
chromadb>=0.5.0
ollama>=0.4.0
```

### `rag_query.py`

```python
#!/usr/bin/env python3
"""
RAG Pipeline for rag-coder model.
Indexes your codebase into ChromaDB, retrieves relevant chunks,
and queries the rag-coder model with full project context.

Usage:
    python rag_query.py --project /path/to/your/project --reindex
    python rag_query.py --project /path/to/your/project --query "How does auth work?"
    python rag_query.py --project /path/to/your/project --query "Add pagination" --top-k 10
"""

import argparse
import hashlib
import os
import sys
from pathlib import Path

import chromadb
import ollama

# ── Configuration ──────────────────────────────────────────
EMBED_MODEL = "nomic-embed-text"
CHAT_MODEL = "rag-coder"
COLLECTION = "codebase"
INDEX_DIR = ".rag-index"
CHUNK_SIZE = 1200          # characters per chunk
CHUNK_OVERLAP = 200        # overlap between chunks
DEFAULT_TOP_K = 5          # context chunks to retrieve

SUPPORTED_EXTENSIONS = {
    ".py", ".js", ".ts", ".tsx", ".jsx", ".java", ".go", ".rs",
    ".cpp", ".c", ".h", ".hpp", ".cs", ".rb", ".php", ".swift",
    ".kt", ".scala", ".sh", ".bash", ".yaml", ".yml", ".toml",
    ".json", ".md", ".txt", ".sql", ".html", ".css", ".scss",
    ".vue", ".svelte", ".dockerfile", ".tf", ".hcl", ".prisma",
    ".graphql",
}

IGNORE_DIRS = {
    "node_modules", ".git", "__pycache__", ".venv", "venv",
    "dist", "build", ".next", ".nuxt", ".rag-index", ".mypy_cache",
    "target", "bin", "obj", ".tox", "coverage", ".turbo",
}


def collect_files(project_path: str) -> list[Path]:
    """Recursively collect supported source files."""
    files = []
    for root, dirs, filenames in os.walk(project_path):
        dirs[:] = [d for d in dirs if d not in IGNORE_DIRS]
        for fname in filenames:
            fpath = Path(root) / fname
            if fpath.suffix.lower() in SUPPORTED_EXTENSIONS:
                if fpath.stat().st_size < 100_000:  # skip files > 100KB
                    files.append(fpath)
    return sorted(files)


def chunk_text(text: str, source: str) -> list[dict]:
    """Split text into overlapping chunks with metadata."""
    chunks = []
    for i in range(0, len(text), CHUNK_SIZE - CHUNK_OVERLAP):
        chunk = text[i : i + CHUNK_SIZE]
        if chunk.strip():
            chunk_id = hashlib.md5(f"{source}:{i}".encode()).hexdigest()
            chunks.append({
                "id": chunk_id,
                "text": f"# FILE: {source}\n{chunk}",
                "metadata": {"source": source, "offset": i},
            })
    return chunks


def index_codebase(project_path: str) -> None:
    """Index the codebase into ChromaDB with Ollama embeddings."""
    index_path = os.path.join(project_path, INDEX_DIR)
    client = chromadb.PersistentClient(path=index_path)

    # Delete existing collection if re-indexing
    try:
        client.delete_collection(COLLECTION)
    except ValueError:
        pass

    collection = client.create_collection(
        name=COLLECTION,
        metadata={"hnsw:space": "cosine"},
    )

    files = collect_files(project_path)
    if not files:
        print("ERROR: No supported files found.")
        sys.exit(1)

    print(f"Indexing {len(files)} files from {project_path}...")

    all_chunks = []
    for fpath in files:
        try:
            text = fpath.read_text(encoding="utf-8", errors="ignore")
            rel = str(fpath.relative_to(project_path))
            all_chunks.extend(chunk_text(text, rel))
        except Exception as e:
            print(f"  Skipped {fpath}: {e}")

    if not all_chunks:
        print("ERROR: No text chunks generated.")
        sys.exit(1)

    # Batch embed and insert
    BATCH = 64
    for i in range(0, len(all_chunks), BATCH):
        batch = all_chunks[i : i + BATCH]
        texts = [c["text"] for c in batch]
        ids = [c["id"] for c in batch]
        metas = [c["metadata"] for c in batch]

        resp = ollama.embed(model=EMBED_MODEL, input=texts)
        embeddings = resp.embeddings

        collection.add(
            ids=ids,
            embeddings=embeddings,
            documents=texts,
            metadatas=metas,
        )
        done = min(i + BATCH, len(all_chunks))
        print(f"  Indexed {done}/{len(all_chunks)} chunks")

    print(f"\nDone! {len(all_chunks)} chunks indexed in {index_path}/")


def query_rag(project_path: str, question: str, top_k: int) -> None:
    """Retrieve context and query the rag-coder model."""
    index_path = os.path.join(project_path, INDEX_DIR)
    if not os.path.exists(index_path):
        print("ERROR: No index found. Run with --reindex first.")
        sys.exit(1)

    client = chromadb.PersistentClient(path=index_path)
    collection = client.get_collection(COLLECTION)

    # Embed the question
    q_resp = ollama.embed(model=EMBED_MODEL, input=[question])
    q_embedding = q_resp.embeddings[0]

    # Retrieve top-k relevant chunks
    results = collection.query(query_embeddings=[q_embedding], n_results=top_k)

    # Build context block
    context_parts = []
    sources = []
    for doc, meta in zip(results["documents"][0], results["metadatas"][0]):
        context_parts.append(doc)
        sources.append(meta["source"])

    context_block = "\n\n---\n\n".join(context_parts)

    prompt = f"""## Retrieved Codebase Context

{context_block}

---

## Question

{question}"""

    print(f"\n  Retrieved {len(sources)} chunks from:")
    for s in dict.fromkeys(sources):  # unique, ordered
        print(f"    - {s}")
    print(f"\n  Querying {CHAT_MODEL}...\n")

    # Stream the response
    stream = ollama.chat(
        model=CHAT_MODEL,
        messages=[{"role": "user", "content": prompt}],
        stream=True,
    )
    for chunk in stream:
        print(chunk.message.content, end="", flush=True)
    print()


def main():
    parser = argparse.ArgumentParser(description="RAG pipeline for rag-coder")
    parser.add_argument("--project", type=str, required=True,
                        help="Path to your project directory")
    parser.add_argument("--query", type=str, default=None,
                        help="Question to ask about the codebase")
    parser.add_argument("--reindex", action="store_true",
                        help="Rebuild the vector index")
    parser.add_argument("--top-k", type=int, default=DEFAULT_TOP_K,
                        help=f"Number of context chunks to retrieve (default: {DEFAULT_TOP_K})")
    args = parser.parse_args()

    if args.reindex:
        index_codebase(args.project)
    elif args.query:
        query_rag(args.project, args.query, args.top_k)
    else:
        parser.print_help()
        print("\nExamples:")
        print(f"  python rag_query.py --project ~/MyProject --reindex")
        print(f'  python rag_query.py --project ~/MyProject --query "How does auth work?"')


if __name__ == "__main__":
    main()
```

### Usage

```bash
# First time — index your codebase (takes a few minutes):
python rag_query.py --project ~/ResumeFlow-UI --reindex

# Ask questions about your codebase:
python rag_query.py --project ~/ResumeFlow-UI --query "How does the resume parser work?"

# Retrieve more context for complex questions:
python rag_query.py --project ~/ResumeFlow-UI --query "Refactor the API routes" --top-k 10

# Re-index after making changes:
python rag_query.py --project ~/ResumeFlow-UI --reindex
```

### Add to `.gitignore`

```gitignore
# Ollama RAG index (generated, machine-specific)
.rag-index/
__pycache__/
*.pyc
```

---

## 8. VS Code Copilot Integration

### Method 1: Manage Models UI (Recommended)

1. Open VS Code
2. Open Copilot Chat (`Cmd+Shift+I` on macOS / `Ctrl+Shift+I` on others)
3. Click the **model dropdown** at the top of the chat panel
4. Click **Manage Models...**
5. Select **Ollama** from the provider list
6. Your custom models (`strict-coder`, `verbose-explainer`, `rag-coder`) appear automatically
7. Toggle them **on** — they now appear in the model picker

### Method 2: `settings.json` Configuration

Open VS Code settings (`Cmd+,` → click the `{}` icon top-right) and add:

```jsonc
{
  "github.copilot.chat.models": [
    {
      "family": "strict-coder",
      "id": "strict-coder",
      "name": "Strict Coder (Code Only)",
      "provider": "ollama",
      "url": "http://localhost:11434"
    },
    {
      "family": "verbose-explainer",
      "id": "verbose-explainer",
      "name": "Verbose Explainer (Teaching)",
      "provider": "ollama",
      "url": "http://localhost:11434"
    },
    {
      "family": "rag-coder",
      "id": "rag-coder",
      "name": "RAG Coder (Codebase-Aware)",
      "provider": "ollama",
      "url": "http://localhost:11434"
    },
    {
      "family": "qwen3-coder",
      "id": "qwen3-coder:30b",
      "name": "Qwen3 Coder 30B (Base)",
      "provider": "ollama",
      "url": "http://localhost:11434"
    }
  ]
}
```

### Verify

Open Copilot Chat → click the model dropdown → you should see all your custom models listed alongside the default Copilot models.

---

## 9. Ollama Launch — Agent Mode

`ollama launch` (v0.15+, January 2026) connects your local models to popular coding agents with zero configuration.

```bash
# Launch Claude Code with your strict-coder model
ollama launch claude --model strict-coder

# Launch Codex
ollama launch codex --model strict-coder

# Launch OpenCode
ollama launch opencode --model verbose-explainer

# Configure a tool without launching it
ollama launch claude --config
```

| Tool | Command | Description |
|---|---|---|
| Claude Code | `ollama launch claude` | Anthropic's agentic coding CLI |
| Codex | `ollama launch codex` | OpenAI's code generation tool |
| OpenCode | `ollama launch opencode` | Terminal-based coding assistant |
| Droid | `ollama launch droid` | Android development assistant |

All tools work with any of your custom models: `strict-coder`, `verbose-explainer`, `rag-coder`.

---

## 10. Quick Test Commands

```bash
# ── Test strict-coder (should output ONLY code, no explanation) ──
ollama run strict-coder "Write a Python function that reverses a linked list"

# ── Test verbose-explainer (should give detailed explanation + code) ──
ollama run verbose-explainer "Explain how React useEffect cleanup works"

# ── Test rag-coder (best via RAG pipeline) ──
python rag_query.py --project ~/ResumeFlow-UI --query "What components handle resume upload?"

# ── Non-interactive mode (for scripts / CI) ──
echo "Write a TypeScript interface for a User object" | ollama run strict-coder

# ── Programmatic single-shot (no chat formatting) ──
ollama generate strict-coder "fibonacci in Rust"
```

### Interactive Session Commands

Inside `ollama run <model>`, you can tune on the fly:

```
/set parameter temperature 0.05     # Even more deterministic
/set parameter num_ctx 32768        # Adjust context window
/show info                          # Show current parameters
/show system                        # Show current system prompt
/clear                              # Clear conversation history
```

> **Note:** `/set` changes are session-only. To persist, edit the Modelfile and re-run `ollama create`.

---

## 11. Parameter Reference

### Side-by-Side Comparison

| Parameter | strict-coder | verbose-explainer | rag-coder | What It Does |
|---|---|---|---|---|
| `temperature` | 0.1 | 0.7 | 0.2 | Randomness: 0 = deterministic, 1 = creative |
| `seed` | 42 | 0 | 42 | Fixed int = reproducible; 0 = random each run |
| `top_p` | 0.9 | 0.95 | 0.9 | Nucleus sampling: lower = more focused |
| `top_k` | 20 | 40 | 30 | Token choices per step: lower = more precise |
| `repeat_penalty` | 1.1 | 1.0 | 1.1 | Penalize repetition: 1.0 = no penalty |
| `num_ctx` | 65536 | 65536 | 65536 | Context window: ~50K words of code + conversation |
| `num_predict` | -1 | -1 | -1 | Max output tokens: -1 = unlimited |

### Seed Behavior

| Seed Value | Behavior | Best For |
|---|---|---|
| `42` (any fixed int) | Same input → same output every time | Code generation, testing, CI pipelines |
| `0` | Different output each run | Creative tasks, brainstorming, varied explanations |

---

## 12. Automation Scripts

### `setup.sh` — One-Command Setup

```bash
#!/bin/bash
set -euo pipefail
echo "=== Ollama Custom Qwen Models Setup ==="

# 1. Check Ollama
if ! command -v ollama &> /dev/null; then
    echo "ERROR: Ollama not found. Install from https://ollama.com/download"
    exit 1
fi
echo "Ollama version: $(ollama --version)"

# 2. Pull base models
echo "Pulling models..."
ollama pull qwen3-coder:30b
ollama pull nomic-embed-text

# 3. Set macOS environment variables
if [[ "$(uname)" == "Darwin" ]]; then
    echo "Setting macOS environment variables..."
    launchctl setenv OLLAMA_FLASH_ATTENTION 1
    launchctl setenv OLLAMA_KV_CACHE_TYPE q8_0
    launchctl setenv OLLAMA_ORIGINS "*"
    launchctl setenv OLLAMA_KEEP_ALIVE 24h
    launchctl setenv OLLAMA_NUM_PARALLEL 4
    launchctl setenv OLLAMA_MAX_LOADED_MODELS 2
    echo "Restarting Ollama..."
    pkill -f Ollama || true
    sleep 2
    open -a Ollama
    sleep 5
fi

# 4. Create Modelfile directory
mkdir -p ~/ollama-models

# 5. Build custom models
echo "Building custom models..."
ollama create strict-coder -f Modelfile.strict-coder
ollama create verbose-explainer -f Modelfile.verbose-explainer
ollama create rag-coder -f Modelfile.rag-coder

# 6. Install Python deps
echo "Installing RAG dependencies..."
pip install chromadb ollama

# 7. Verify
echo ""
echo "=== Setup Complete ==="
ollama list
echo ""
echo "Next steps:"
echo "  1. Open VS Code → Copilot Chat → Model dropdown → Manage Models → Ollama"
echo "  2. Index your codebase: python rag_query.py --project ~/your-project --reindex"
```

### `rebuild_models.sh` — Quick Rebuild

```bash
#!/bin/bash
set -euo pipefail
echo "Pulling latest base model..."
ollama pull qwen3-coder:30b

echo "Rebuilding custom models..."
ollama create strict-coder -f Modelfile.strict-coder
ollama create verbose-explainer -f Modelfile.verbose-explainer
ollama create rag-coder -f Modelfile.rag-coder

echo "Done. Updated models:"
ollama list | grep -E "strict-coder|verbose-explainer|rag-coder"
```

```bash
chmod +x setup.sh rebuild_models.sh
```

> **When to rebuild:** Custom models do NOT auto-update when the base model changes. Run `rebuild_models.sh` after `ollama pull qwen3-coder:30b` fetches a new version.

---

## 13. Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| `pull manifest: file does not exist` | Model tag doesn't exist in registry | Verify exact tags at [ollama.com/library](https://ollama.com/library). There is no `qwen3-coder:7b` or `:14b` |
| Models not showing in VS Code | `OLLAMA_ORIGINS` not set or VS Code needs restart | Set `OLLAMA_ORIGINS=*` → restart Ollama → restart VS Code |
| `connection refused` | Ollama server not running | Run `ollama serve` or `open -a Ollama` (macOS) |
| Slow first response | Model loading from disk (cold start) | Set `OLLAMA_KEEP_ALIVE=24h` → restart Ollama. Warm up with `ollama run strict-coder "ping"` |
| Out of memory | Model too large for RAM | Use smaller base (`qwen3:14b`), or reduce `num_ctx` to `16384` |
| Flash Attention not working | Env var not set before server start | Verify with `launchctl getenv OLLAMA_FLASH_ATTENTION` (macOS) → restart Ollama |
| Strict model outputs explanations | System prompt not applied | Verify with `ollama show --modelfile strict-coder` → rebuild if wrong |
| RAG returns irrelevant results | Stale index after code changes | Re-run `python rag_query.py --project ~/your-project --reindex` |
| `ollama create` fails | Base model not pulled yet | Run `ollama pull qwen3-coder:30b` first |

---

## 14. Quick Reference Card

### Pull Commands

```bash
ollama pull qwen3-coder:30b          # Primary coding model (19 GB, 256K ctx)
ollama pull qwen3:14b                # Lighter alternative (9.3 GB, 40K ctx)
ollama pull qwen3-coder:480b-cloud   # Cloud-hosted (zero local VRAM)
ollama pull nomic-embed-text         # Embeddings for RAG pipeline (274 MB)
```

### Run Your Custom Models

```bash
ollama run strict-coder              # Code only, no explanations
ollama run verbose-explainer         # Teaching mode with explanations
ollama run rag-coder                 # Codebase-aware (use via rag_query.py)
```

### Essential Commands

| Command | Description |
|---|---|
| `ollama list` | List all local models |
| `ollama ps` | Show running models + resource usage |
| `ollama show --modelfile <name>` | Inspect a model's configuration |
| `ollama create <name> -f <file>` | Create or rebuild a custom model |
| `ollama rm <name>` | Delete a model |
| `ollama cp <src> <dst>` | Copy/rename a model |
| `ollama stop <name>` | Unload model from memory |

---

## 15. Privacy Note

All three custom models run **100% locally** on your machine. No data leaves your device. The RAG pipeline stores its index in a local `.rag-index/` directory — no cloud services involved.

**Exception:** `qwen3-coder:480b-cloud` routes inference through Ollama's cloud infrastructure.

---

## 16. Useful Links

- **Ollama:** [ollama.com](https://ollama.com)
- **Model Library:** [ollama.com/library](https://ollama.com/library)
- **qwen3-coder tags:** [ollama.com/library/qwen3-coder/tags](https://ollama.com/library/qwen3-coder/tags)
- **qwen3 tags:** [ollama.com/library/qwen3/tags](https://ollama.com/library/qwen3/tags)
- **VS Code Copilot + Ollama:** [code.visualstudio.com/docs/copilot/language-model-providers](https://code.visualstudio.com/docs/copilot/language-model-providers)
- **ChromaDB Docs:** [docs.trychroma.com](https://docs.trychroma.com)

---

## Recommended Repo Structure

```
ollama-custom-models/
├── ollama-custom-qwen-models.md      ← this file
├── Modelfile.strict-coder
├── Modelfile.verbose-explainer
├── Modelfile.rag-coder
├── rag_query.py
├── requirements.txt
├── setup.sh
├── rebuild_models.sh
├── .gitignore
└── LICENSE
```

---

## License

MIT — use, modify, and share freely.
