# Ollama Custom Qwen Models — Speed, Strict Mode & VS Code Copilot Integration

> **Production-ready Modelfiles** for three custom Qwen3-Coder personas: a strict code-only engine, a verbose explainer, and a RAG-enhanced codebase reader — all tuned for maximum speed and visible in VS Code Copilot's model selector.

---

## Table of Contents

- [1. Prerequisites \& Installation](#1-prerequisites--installation)
- [2. Mandatory Parameters Reference](#2-mandatory-parameters-reference)
- [3. Model 1 — Strict Code-Only Mode](#3-model-1--strict-code-only-mode-qwen-strict-coder)
- [4. Model 2 — Verbose Explainer Mode](#4-model-2--verbose-explainer-mode-qwen-explainer)
- [5. Model 3 — RAG-Enhanced Codebase Reader](#5-model-3--rag-enhanced-codebase-reader-qwen-rag-coder)
- [6. Speed Optimization Checklist](#6-speed-optimization-checklist)
- [7. VS Code Copilot Integration](#7-vs-code-copilot-integration)
- [8. Quick Command Reference](#8-quick-command-reference)
- [9. Troubleshooting](#9-troubleshooting)

---

## 1. Prerequisites & Installation

### Install Ollama

| Platform | Command |
|----------|---------|
| macOS / Linux | `curl -fsSL https://ollama.com/install.sh \| sh` |
| Windows | Download installer from [ollama.com/download](https://ollama.com/download) |

Verify the installation:

```bash
ollama --version
```

### Pull the Base Model

```bash
ollama pull qwen3-coder:30b
```

#### Available Qwen3-Coder Tags

| Tag | Size | Active Params | Context | Best For |
|-----|------|---------------|---------|----------|
| `qwen3-coder:30b` | 19 GB | 3.3B (MoE) | 256K | Local coding, fast inference |
| `qwen3-coder:480b` | 290 GB | 35B (MoE) | 256K | Maximum quality, needs 250 GB+ RAM |
| `qwen3-coder:480b-cloud` | Cloud | 35B | 256K | Cloud-hosted, no local resources |
| `qwen3-coder-next` | ~5 GB | 3B hybrid | 256K | Ultra-lightweight, local dev |

### Enable CORS for VS Code

| Platform | How |
|----------|-----|
| macOS | `launchctl setenv OLLAMA_ORIGINS "*"` |
| Linux | Edit systemd service → `Environment="OLLAMA_ORIGINS=*"` then `sudo systemctl daemon-reload && sudo systemctl restart ollama` |
| Windows | System Environment Variable → `OLLAMA_ORIGINS` = `*` |

**Restart Ollama after setting the variable.**

### Enable Flash Attention (Speed Boost)

Set this environment variable **before** starting `ollama serve`:

```bash
export OLLAMA_FLASH_ATTENTION=1
```

> This reduces memory usage and accelerates inference on long contexts.

---

## 2. Mandatory Parameters Reference

| Parameter | Range | Default | Recommended (Code) | What It Does |
|-----------|-------|---------|---------------------|--------------|
| `temperature` | 0.0 – 2.0 | 0.8 | **0.1** | Controls randomness. Lower = deterministic, precise code |
| `num_ctx` | 1 – 256000 | 2048 | **16384** | Context window in tokens. Higher = more VRAM |
| `num_predict` | -1 to ∞ | -1 (infinite) | **4096** | Max tokens per response. Caps length for speed |
| `top_k` | 1 – 100 | 40 | **10** | Limits token pool to top K. Lower = more focused, faster |
| `top_p` | 0.0 – 1.0 | 0.9 | **0.85** | Nucleus sampling. Lower = less randomness |
| `min_p` | 0.0 – 1.0 | 0.0 | **0.05** | Filters tokens below this probability |
| `repeat_penalty` | 0.5 – 2.0 | 1.1 | **1.15** | Penalizes repeated tokens |
| `repeat_last_n` | 0 – num_ctx | 64 | **128** | How far back to check for repetition |
| `seed` | any integer | random | **42** | Fixed seed for reproducible output |
| `num_gpu` | 0 – 999 | auto | **999** | GPU layers. 999 = all on GPU (fastest) |
| `num_thread` | 1 – N | auto | **CPU cores / 2** | CPU threads. Set to physical core count |
| `mirostat` | 0, 1, 2 | 0 | **0** | Alternative sampling. Keep 0 for code |
| `stop` | string(s) | model default | See Modelfiles | Stop sequences that terminate generation |

> **⚡ Speed Rule of Thumb:** Lower temperature + lower top_k + capped num_predict + max GPU offload = fastest possible inference.

---

## 3. Model 1 — Strict Code-Only Mode (`qwen-strict-coder`)

**Purpose:** Outputs ONLY raw code. No explanations, no comments (unless in-code), no markdown prose.

### Modelfile: `Modelfile.strict-coder`

```dockerfile
# ============================================================
# MODEL: qwen-strict-coder
# PURPOSE: Pure code output — no prose, no explanations
# ============================================================

FROM qwen3-coder:30b

# ── System Prompt (Strict Code-Only Enforcement) ──────────
SYSTEM """
You are a strict code generation engine. You follow these rules WITHOUT EXCEPTION:

1. OUTPUT ONLY CODE. Never output explanations, descriptions, summaries, or commentary.
2. Do NOT wrap code in markdown code fences (no triple backticks).
3. Do NOT add introductory text like "Here is the code" or "Sure, here you go".
4. Do NOT add concluding text like "This code does X" or "Let me know if you need help".
5. Include inline comments ONLY when they clarify non-obvious logic.
6. If the user asks a question that cannot be answered with code, respond with a single-line code comment explaining why.
7. Always use the language specified or inferred from context. Default to Python if ambiguous.
8. Follow best practices: proper error handling, type hints, meaningful variable names.
9. If modifying existing code, return the COMPLETE modified file, not just the diff.
10. For multiple files, separate them with the comment: # --- FILE: filename.ext ---
"""

# ── Speed-Optimized Parameters ────────────────────────────
PARAMETER temperature 0.1
PARAMETER top_k 10
PARAMETER top_p 0.85
PARAMETER min_p 0.05
PARAMETER num_predict 4096
PARAMETER num_ctx 16384
PARAMETER repeat_penalty 1.15
PARAMETER repeat_last_n 128
PARAMETER seed 42
PARAMETER num_gpu 999
PARAMETER num_thread 8

# ── Stop Sequences ────────────────────────────────────────
PARAMETER stop "<|im_start|>"
PARAMETER stop "<|im_end|>"

# ── Few-Shot Training Messages (Teach Strict Behavior) ───
MESSAGE user "Write a Python function to reverse a string"
MESSAGE assistant "def reverse_string(s: str) -> str:
    return s[::-1]"

MESSAGE user "Create a TypeScript interface for a user profile"
MESSAGE assistant "interface UserProfile {
  id: string;
  email: string;
  displayName: string;
  avatarUrl?: string;
  createdAt: Date;
  updatedAt: Date;
  isActive: boolean;
}"

MESSAGE user "What is recursion?"
MESSAGE assistant "# This question cannot be answered with code alone.
# Recursion is a function calling itself with a base case to terminate."
```

### Build & Test

```bash
# Create the model
ollama create qwen-strict-coder -f Modelfile.strict-coder

# Test strict mode (should output ONLY code)
ollama run qwen-strict-coder "Write a FastAPI health check endpoint"

# Verify no prose leaks
ollama run qwen-strict-coder "Explain how async works in Python"
```

---

## 4. Model 2 — Verbose Explainer Mode (`qwen-explainer`)

**Purpose:** Provides detailed, educational explanations with code. Teaches, not just generates.

### Modelfile: `Modelfile.explainer`

```dockerfile
# ============================================================
# MODEL: qwen-explainer
# PURPOSE: Verbose teaching mode — explains everything in depth
# ============================================================

FROM qwen3-coder:30b

# ── System Prompt (Verbose Explainer) ─────────────────────
SYSTEM """
You are an expert programming tutor and technical writer. Your responses must be thorough, educational, and well-structured. Follow these guidelines:

1. ALWAYS explain the "why" behind every code decision, not just the "what".
2. Structure every response with:
   - **Concept Overview**: Brief explanation of the core concept
   - **Step-by-Step Walkthrough**: Numbered breakdown of the approach
   - **Complete Code**: Full working implementation with extensive inline comments
   - **Key Takeaways**: Bullet points summarizing what the user should remember
   - **Common Pitfalls**: Mistakes beginners make with this pattern
   - **Further Reading**: Suggest related topics to explore
3. Use analogies to explain complex concepts (e.g., "Think of a mutex like a bathroom lock").
4. When showing code, comment EVERY significant line explaining its purpose.
5. If multiple approaches exist, show at least two and compare their trade-offs.
6. Use markdown formatting: headers, bold, code blocks, tables, bullet points.
7. Assume the user is an intermediate developer who wants to deeply understand the topic.
8. Include time and space complexity analysis for algorithms.
9. Show both the naive approach and the optimized approach when applicable.
"""

# ── Balanced Parameters (Quality over Speed) ──────────────
PARAMETER temperature 0.4
PARAMETER top_k 40
PARAMETER top_p 0.92
PARAMETER min_p 0.03
PARAMETER num_predict 8192
PARAMETER num_ctx 32768
PARAMETER repeat_penalty 1.1
PARAMETER repeat_last_n 256
PARAMETER seed 0
PARAMETER num_gpu 999
PARAMETER num_thread 8

# ── Stop Sequences ────────────────────────────────────────
PARAMETER stop "<|im_start|>"
PARAMETER stop "<|im_end|>"
```

### Build & Test

```bash
ollama create qwen-explainer -f Modelfile.explainer
ollama run qwen-explainer "Explain Python decorators with examples"
```

---

## 5. Model 3 — RAG-Enhanced Codebase Reader (`qwen-rag-coder`)

**Purpose:** Tuned for Retrieval-Augmented Generation. Expects codebase context injected into the prompt, responds with awareness of the user's project structure, style, and dependencies.

### Modelfile: `Modelfile.rag-coder`

```dockerfile
# ============================================================
# MODEL: qwen-rag-coder
# PURPOSE: RAG-enhanced — reads injected codebase context
# ============================================================

FROM qwen3-coder:30b

# ── System Prompt (RAG-Aware Codebase Assistant) ──────────
SYSTEM """
You are a codebase-aware AI assistant operating in RAG (Retrieval-Augmented Generation) mode. Context from the user's codebase will be provided at the beginning of each prompt between <codebase_context> and </codebase_context> tags.

RULES FOR RAG MODE:

1. ALWAYS read and analyze the provided codebase context before responding.
2. Your answers must be consistent with the existing codebase:
   - Match the coding style, naming conventions, and patterns already in use.
   - Import from existing modules rather than suggesting new dependencies.
   - Follow the project's established architecture and folder structure.
   - Use existing utility functions and helpers instead of reinventing them.
3. When suggesting new code:
   - Specify exactly which file it belongs in (use the project's file paths).
   - Show where it integrates with existing code (reference specific functions/classes).
   - Ensure imports are correct relative to the project structure.
4. When debugging:
   - Reference specific line numbers and file paths from the context.
   - Trace the execution flow through the actual codebase.
   - Check for conflicts with existing implementations.
5. If the context is insufficient to answer accurately, explicitly state what additional files or context you need.
6. Never hallucinate file paths, function names, or APIs not present in the context.
7. When the context includes package.json, requirements.txt, Cargo.toml, or similar, respect the existing dependency versions.
8. Format file references as: path/to/file.ext:line_number when referencing specific locations.
"""

# ── RAG-Optimized Parameters (Large Context, Precise) ────
PARAMETER temperature 0.15
PARAMETER top_k 15
PARAMETER top_p 0.88
PARAMETER min_p 0.05
PARAMETER num_predict 6144
PARAMETER num_ctx 65536
PARAMETER repeat_penalty 1.12
PARAMETER repeat_last_n 256
PARAMETER seed 42
PARAMETER num_gpu 999
PARAMETER num_thread 8

# ── Stop Sequences ────────────────────────────────────────
PARAMETER stop "<|im_start|>"
PARAMETER stop "<|im_end|>"

# ── Few-Shot: Teaching RAG Behavior ───────────────────────
MESSAGE user "<codebase_context>\n# FILE: src/utils/logger.py\nimport logging\n\ndef get_logger(name: str) -> logging.Logger:\n    logger = logging.getLogger(name)\n    logger.setLevel(logging.INFO)\n    return logger\n</codebase_context>\n\nAdd error logging to the authentication module"
MESSAGE assistant "Based on your existing `src/utils/logger.py`, here is the integration for the auth module:\n\n```python\n# FILE: src/auth/handler.py\nfrom src.utils.logger import get_logger\n\nlogger = get_logger(__name__)\n\ndef authenticate(token: str) -> bool:\n    try:\n        # validation logic here\n        return True\n    except Exception as e:\n        logger.error(f'Authentication failed: {e}')\n        return False\n```\n\nThis uses your existing `get_logger` utility to maintain consistent logging across the project."
```

### Build & Test

```bash
ollama create qwen-rag-coder -f Modelfile.rag-coder

# Test with injected context
ollama run qwen-rag-coder "<codebase_context>
# FILE: app/main.py
from fastapi import FastAPI
app = FastAPI(title='MyAPI')
</codebase_context>

Add a /users endpoint with CRUD operations"
```

### RAG Pipeline Setup (Python Script)

Save the following as `rag_codebase.py` in your project root.

**Install dependencies first:**

```bash
pip install chromadb ollama
ollama pull nomic-embed-text
```

```python
#!/usr/bin/env python3
"""
rag_codebase.py — Local RAG pipeline for qwen-rag-coder
Indexes your codebase, retrieves relevant files, and queries the model.

Usage:
    python rag_codebase.py --index /path/to/your/project
    python rag_codebase.py --query "Add rate limiting to the API"
"""

import os
import sys
import json
import argparse
import hashlib
from pathlib import Path
from typing import List, Dict

import chromadb
import ollama

# ── Configuration ─────────────────────────────────────────
COLLECTION_NAME = "codebase"
EMBED_MODEL = "nomic-embed-text"
CHAT_MODEL = "qwen-rag-coder"
CHROMA_PATH = "./.rag_index"
MAX_CONTEXT_FILES = 10
MAX_FILE_SIZE_KB = 100

# File extensions to index
CODE_EXTENSIONS = {
    ".py", ".js", ".ts", ".tsx", ".jsx", ".java", ".go", ".rs",
    ".cpp", ".c", ".h", ".cs", ".rb", ".php", ".swift", ".kt",
    ".vue", ".svelte", ".html", ".css", ".scss", ".sql",
    ".yaml", ".yml", ".toml", ".json", ".md", ".sh", ".bash",
    ".dockerfile", ".tf", ".prisma", ".graphql",
}

# Directories to skip
SKIP_DIRS = {
    "node_modules", ".git", "__pycache__", ".venv", "venv",
    "dist", "build", ".next", ".nuxt", "target", "vendor",
    ".rag_index", ".ollama",
}


def collect_files(project_path: str) -> List[Dict]:
    """Walk the project tree and collect indexable source files."""
    files = []
    for root, dirs, filenames in os.walk(project_path):
        dirs[:] = [d for d in dirs if d not in SKIP_DIRS]
        for fname in filenames:
            fpath = Path(root) / fname
            if fpath.suffix.lower() in CODE_EXTENSIONS:
                size_kb = fpath.stat().st_size / 1024
                if size_kb <= MAX_FILE_SIZE_KB:
                    try:
                        content = fpath.read_text(encoding="utf-8", errors="ignore")
                        rel_path = str(fpath.relative_to(project_path))
                        files.append({
                            "path": rel_path,
                            "content": content,
                            "id": hashlib.md5(rel_path.encode()).hexdigest(),
                        })
                    except Exception:
                        pass
    return files


def index_codebase(project_path: str):
    """Index all source files into ChromaDB with Ollama embeddings."""
    print(f"Scanning: {project_path}")
    files = collect_files(project_path)
    print(f"Found {len(files)} indexable files")

    client = chromadb.PersistentClient(path=CHROMA_PATH)

    # Delete existing collection if re-indexing
    try:
        client.delete_collection(COLLECTION_NAME)
    except Exception:
        pass

    collection = client.create_collection(name=COLLECTION_NAME)

    for i, f in enumerate(files):
        doc_text = f"# FILE: {f['path']}\n{f['content']}"
        # Generate embedding via Ollama
        response = ollama.embed(model=EMBED_MODEL, input=doc_text)
        embedding = response["embeddings"][0]

        collection.add(
            ids=[f["id"]],
            documents=[doc_text],
            embeddings=[embedding],
            metadatas=[{"path": f["path"]}],
        )
        print(f"  [{i+1}/{len(files)}] Indexed: {f['path']}")

    print(f"\nIndex complete: {len(files)} files -> {CHROMA_PATH}")


def query_codebase(question: str):
    """Retrieve relevant files and query the RAG model."""
    client = chromadb.PersistentClient(path=CHROMA_PATH)
    collection = client.get_collection(name=COLLECTION_NAME)

    # Embed the query
    q_response = ollama.embed(model=EMBED_MODEL, input=question)
    q_embedding = q_response["embeddings"][0]

    # Retrieve top-N relevant files
    results = collection.query(
        query_embeddings=[q_embedding],
        n_results=MAX_CONTEXT_FILES,
    )

    # Build context block
    context_parts = []
    for doc, meta in zip(results["documents"][0], results["metadatas"][0]):
        context_parts.append(doc)
    context_block = "\n\n".join(context_parts)

    # Build the RAG prompt
    rag_prompt = f"""<codebase_context>
{context_block}
</codebase_context>

{question}"""

    print(f"\n-- Retrieved {len(context_parts)} files --")
    for meta in results["metadatas"][0]:
        print(f"  * {meta['path']}")
    print(f"-- Querying {CHAT_MODEL} --\n")

    # Stream the response
    stream = ollama.chat(
        model=CHAT_MODEL,
        messages=[{"role": "user", "content": rag_prompt}],
        stream=True,
    )
    for chunk in stream:
        print(chunk["message"]["content"], end="", flush=True)
    print()


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="RAG Codebase Assistant")
    parser.add_argument("--index", type=str, help="Path to project to index")
    parser.add_argument("--query", type=str, help="Question about your codebase")
    args = parser.parse_args()

    if args.index:
        index_codebase(args.index)
    elif args.query:
        query_codebase(args.query)
    else:
        parser.print_help()
```

**Usage:**

```bash
# Step 1: Index your project
python rag_codebase.py --index /path/to/your/project

# Step 2: Query with codebase awareness
python rag_codebase.py --query "How does the authentication middleware work?"
python rag_codebase.py --query "Add pagination to the /users endpoint"
```

---

## 6. Speed Optimization Checklist

1. **GPU Offload Everything** — `PARAMETER num_gpu 999` forces all layers onto GPU. If you get OOM errors, reduce by 4 at a time.
2. **Enable Flash Attention** — Set `OLLAMA_FLASH_ATTENTION=1` as an environment variable before starting `ollama serve`.
3. **Cap Output Length** — `PARAMETER num_predict 4096` prevents runaway generation. Increase for explainer mode.
4. **Lower Temperature** — `PARAMETER temperature 0.1` reduces sampling computation and produces deterministic output.
5. **Restrict Token Pool** — `PARAMETER top_k 10` + `PARAMETER top_p 0.85` = fewer candidates = faster selection.
6. **Set Thread Count** — `PARAMETER num_thread N` — set to your physical CPU core count (not hyperthreads):
   - Linux: `nproc`
   - macOS: `sysctl -n hw.physicalcpu`
   - Windows: Task Manager → Performance → Cores
7. **Keep Model Loaded** — `OLLAMA_KEEP_ALIVE=-1` keeps the model in memory permanently (no cold-start delay).
8. **Use Quantized Models** — Prefer Q4_K_M (default for 30b) for speed. Use Q8_0 only with VRAM headroom.
9. **Reduce Context Window** — Use `num_ctx 16384` instead of 65536 unless you need long context. VRAM scales quadratically with context.
10. **Monitor Performance** — `ollama ps` to check loaded models and VRAM. `nvidia-smi` for GPU utilization.

---

## 7. VS Code Copilot Integration

### Step 1: Ensure Ollama Is Running

```bash
ollama serve
# Verify:
curl http://localhost:11434/api/tags
```

### Step 2: Add Ollama as a Model Provider

1. Open **VS Code**
2. Open the **Copilot Chat** sidebar (`Ctrl+Shift+I` / `Cmd+Shift+I`)
3. Click the **model dropdown** at the top of the chat panel
4. Click **"Manage Models"**
5. Click **"Add Models"** (top-right)
6. Select **"Ollama"** as the provider
7. Enter the endpoint: `http://localhost:11434/`
8. Press **Enter**

### Step 3: Enable Your Custom Models

After adding the Ollama provider:

- Your three models (`qwen-strict-coder`, `qwen-explainer`, `qwen-rag-coder`) appear in the model list
- Click the **eye icon** (👁️) next to each model to make it visible in the dropdown
- Models appear in **Ask mode** by default
- For **Agent mode** (tool-calling), the model must support tool calling — Qwen3-Coder supports this natively. If it doesn't appear in Agent mode, use **VS Code Insiders** (known bug in stable, patched in Insiders)

### Step 4: Manual `settings.json` Config (Optional)

```json
{
  "github.copilot.chat.models": [
    {
      "provider": "ollama",
      "model": "qwen-strict-coder",
      "endpoint": "http://localhost:11434"
    },
    {
      "provider": "ollama",
      "model": "qwen-explainer",
      "endpoint": "http://localhost:11434"
    },
    {
      "provider": "ollama",
      "model": "qwen-rag-coder",
      "endpoint": "http://localhost:11434"
    }
  ]
}
```

### Mode Compatibility

| Model | Ask Mode | Agent Mode | Edit Mode |
|-------|----------|------------|-----------|
| `qwen-strict-coder` | ✅ | ✅ (tool-calling inherited) | ✅ |
| `qwen-explainer` | ✅ | ✅ | ✅ |
| `qwen-rag-coder` | ✅ | ✅ | ✅ |

---

## 8. Quick Command Reference

| Action | Command |
|--------|---------|
| Pull base model | `ollama pull qwen3-coder:30b` |
| Pull embedding model | `ollama pull nomic-embed-text` |
| Create strict coder | `ollama create qwen-strict-coder -f Modelfile.strict-coder` |
| Create explainer | `ollama create qwen-explainer -f Modelfile.explainer` |
| Create RAG coder | `ollama create qwen-rag-coder -f Modelfile.rag-coder` |
| List all models | `ollama list` |
| Show model config | `ollama show --modelfile qwen-strict-coder` |
| Test a model | `ollama run qwen-strict-coder "Write a Python fizzbuzz"` |
| Check running models | `ollama ps` |
| Delete a model | `ollama rm model-name` |
| Index codebase (RAG) | `python rag_codebase.py --index /path/to/project` |
| Query codebase (RAG) | `python rag_codebase.py --query "How does auth work?"` |
| Start Ollama server | `ollama serve` |
| Check GPU usage | `nvidia-smi` (NVIDIA) or `ollama ps` |

---

## 9. Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| Models not visible in Copilot | Eye icon not toggled | Click 👁️ next to each model in Manage Models |
| Models missing from Agent mode | Lack of tool-calling or VS Code bug | Use Ask mode, or switch to VS Code Insiders |
| "Connection refused" from VS Code | Ollama not running or CORS blocked | Run `ollama serve` and set `OLLAMA_ORIGINS=*` |
| Slow first response | Cold start — model loading into memory | Set `OLLAMA_KEEP_ALIVE=-1` to keep model resident |
| Out of Memory (OOM) | Too many GPU layers or context too large | Reduce `num_gpu` by 4, or lower `num_ctx` |
| Repetitive output | Repeat penalty too low | Increase `repeat_penalty` to 1.2 and `repeat_last_n` to 256 |
| Strict model still outputs prose | System prompt not enforced | Verify with `ollama show --modelfile qwen-strict-coder`, rebuild if needed |
| RAG returns irrelevant files | Stale index | Re-run `python rag_codebase.py --index /path/to/project` |

---
``` repo
repo/
├── ollama-custom-qwen-models.md
├── Modelfile.strict-coder
├── Modelfile.explainer
├── Modelfile.rag-coder
└── rag_codebase.py
```
---

## License

MIT — use, modify, and share freely.
