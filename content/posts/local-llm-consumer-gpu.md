---
title: "How to Run a Local LLM on a Consumer GPU (3090, 4090, etc.)"
date: 2026-04-18
description: "A practical guide to running large language models on consumer NVIDIA GPUs. Which models fit in which VRAM, what to buy, and how to set it all up."
tags: ["llm", "local ai", "gpu", "ollama", "nvidia", "home server"]
categories: ["guides"]
showToc: true
---

Consumer GPUs have become the de facto hardware for running local LLMs. The math is simple: large language models live and die by VRAM, and NVIDIA's consumer cards offer more VRAM per dollar than any cloud alternative or enterprise GPU you could buy new. A used RTX 3090 with 24 GB of VRAM runs models that would cost $2–5/hour on cloud GPU instances — for a one-time hardware investment of $600–800.

This guide covers what hardware to buy, what models fit in it, and how to get everything running.

## Why VRAM Is Everything

When you run a language model, the model weights have to fit in GPU memory (VRAM). If they don't fit entirely, inference falls back to system RAM, which is dramatically slower — often 10–30x. Full GPU inference is the goal.

As a rough rule: 1 billion parameters at 4-bit quantization uses about 700 MB of VRAM. So:

| Model Size | 4-bit VRAM | 8-bit VRAM | 16-bit VRAM |
|-----------|-----------|-----------|------------|
| 7B        | ~5 GB     | ~8 GB     | ~14 GB     |
| 13B       | ~9 GB     | ~14 GB    | ~28 GB     |
| 34B       | ~22 GB    | ~34 GB    | ~68 GB     |
| 70B       | ~44 GB    | ~70 GB    | ~140 GB    |

These are approximations — actual usage includes KV cache overhead and overhead from the runtime, but this gets you in the right ballpark.

## Consumer GPU Recommendations

### RTX 3060 12GB (~$250–300 used)

The best value entry point for local LLM work. 12 GB of GDDR6 comfortably fits 7B models at 8-bit quantization, or 13B models at 4-bit. Fast enough for real-time chat, cheap enough to be a starter GPU.

Fits: Qwen3 8B (q8), Llama 4 Scout (q4), 13B models (q4)

[NVIDIA RTX 3060 12GB on Amazon](https://www.amazon.com/s?k=RTX+3060+12GB&tag=YOURTAG-20)

### RTX 3090 24GB (~$800–1,000 used)

Still the value king for local AI in 2026 despite GPU prices rising across the board. 24 GB fits DeepSeek R1 32B at 4-bit comfortably, or Llama 4 Scout at higher precision. XDA Developers called it "the best GPU for local AI value" even after the RTX 50 series launch.

Fits: 7B (full precision), 13B (q8), 32–34B (q4)

[NVIDIA RTX 3090 24GB on Amazon](https://www.amazon.com/s?k=RTX+3090+24GB&tag=YOURTAG-20)

### RTX 4090 24GB (~$2,000+ used)

Same VRAM as the 3090, but 40–60% faster inference due to better memory bandwidth and compute. Prices have risen significantly in 2026 due to AI demand. Unless you need the speed, the 3090 is better value.

Fits: Same as 3090, but faster

[NVIDIA RTX 4090 on Amazon](https://www.amazon.com/s?k=RTX+4090&tag=YOURTAG-20)

### RTX 4060 Ti 16GB (~$450 new)

Underrated for AI workloads. The 16 GB version gives you enough VRAM for 13B at q8 or 32B models at q4 in a power-efficient 165W package. For a mini PC or ITX build where power matters, this is a strong pick.

[NVIDIA RTX 4060 Ti 16GB on Amazon](https://www.amazon.com/s?k=RTX+4060+Ti+16GB&tag=YOURTAG-20)

### What About RTX 5080/5090?

The RTX 5080 (16 GB GDDR7) and 5090 (32 GB GDDR7) are out, but prices are brutal in early 2026 — the 5080 sells for $1,400–1,950, the 5090 for $3,000–5,000 due to memory shortages. For local AI hobbyists, neither is worth the premium right now. Revisit in late 2026 when prices normalize.

### Dual GPU: 2x RTX 3090 (~$1,600–2,000 used)

Two 3090s give you 48 GB of combined VRAM via tensor parallel inference with llama.cpp. This opens up 70B models at 4-bit quantization. Requires a beefy ATX case, 1200W+ PSU, and a motherboard with two full PCIe slots.

## Installing the Stack

### Step 1: NVIDIA Drivers

On Ubuntu/Debian, the easiest path:

```bash
sudo apt update
sudo apt install nvidia-driver-550 nvidia-cuda-toolkit
sudo reboot
```

After reboot:

```bash
nvidia-smi
```

You should see your GPU listed with driver version and VRAM.

### Step 2: Install Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Ollama auto-detects CUDA and uses the GPU. Verify:

```bash
ollama run llama3.1:8b --verbose
```

Look for `GPU layers: XX` in the output — that tells you how many layers are offloaded to the GPU. For a 7B model on a 12 GB GPU, all layers should offload.

### Step 3: Pull and Benchmark Models

Pull a model:

```bash
ollama pull llama3.1:8b
```

Test inference speed:

```bash
ollama run llama3.1:8b "Write a paragraph about the Roman Empire"
```

Watch the tokens/sec in the output. On an RTX 3090, expect 50–80 tokens/sec for a 7B model. On an RTX 4090, 80–120 tokens/sec.

## Model Recommendations by GPU

### 8–12 GB VRAM (RTX 3060/3070, RTX 4060)

Best models in this VRAM range:

- **Qwen3 4B** (`ollama pull qwen3:4b`) — fast, efficient, great for quick tasks and surprisingly capable
- **Llama 4 Scout 17B Q4** (`ollama pull llama4:scout`) — the 2026 standout; fits in 12 GB at 4-bit, excellent general-purpose quality
- **Qwen3-Coder** (`ollama pull qwen3-coder`) — best coding model available, MoE architecture runs fast despite 80B total params
- **Qwen3 8B** (`ollama pull qwen3:8b`) — strong all-rounder, 5 GB VRAM at Q4

### 16–20 GB VRAM (RTX 4060 Ti 16GB, RTX 4080)

Additional models that fit:

- **Llama 4 Scout** at higher precision — more quality headroom
- **DeepSeek R1 14B** — reasoning-focused, shows chain-of-thought, excellent for technical problems
- **Qwen3 14B** — Google-competitive on many benchmarks, fast inference

### 24 GB VRAM (RTX 3090, RTX 4090)

The full range opens up:

- **DeepSeek R1 32B** (`ollama pull deepseek-r1:32b`) — fits comfortably, near GPT-4o quality on reasoning tasks
- **Llama 4 Maverick** (partial GPU) — 400B MoE, most layers need CPU offload at 24 GB
- **Qwen3 32B** — strong general-purpose model, fits cleanly in 24 GB at Q4

To run 70B models fully in VRAM, you need either 48 GB (two 3090s) or 80+ GB (professional hardware like A100).

## llama.cpp for Advanced Control

Ollama is built on llama.cpp but abstracts away the details. For more control — custom quantizations, specific GPU layer counts, CPU+GPU offloading — run llama.cpp directly.

Build it:

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release -j $(nproc)
```

Download a GGUF model from Hugging Face and run:

```bash
./build/bin/llama-cli \
  -m /path/to/model.gguf \
  -ngl 99 \           # offload all layers to GPU
  -c 4096 \           # context length
  -t 8 \              # CPU threads
  -p "Your prompt here"
```

The `-ngl` flag controls how many layers go to GPU. Set it to the maximum your VRAM allows (start with 99 and reduce if you get OOM errors).

## System RAM and CPU Also Matter

VRAM is the constraint for model size, but system RAM and CPU matter for:

- **Context length:** The KV cache for long contexts is large. 4K tokens might need 1–2 GB of additional RAM/VRAM. Running 32K context contexts on a 7B model requires significant extra memory.
- **Tokenization and preprocessing:** Happens on CPU before GPU inference.
- **Multi-user setups:** If multiple people are using the server simultaneously, you need enough CPU/RAM to handle concurrent requests without queueing.

Minimum for a GPU AI server: 32 GB system RAM, modern 8-core CPU.

Recommended: 64 GB RAM, AMD Ryzen 9 or Intel Core i9.

## Full Hardware Build for Local AI

A purpose-built local AI server doesn't need to be a workstation. A mid-tower with a GPU handles everything:

| Component | Recommendation | Approx Price |
|-----------|---------------|-------------|
| GPU | RTX 3090 (used) | $650 |
| CPU | AMD Ryzen 7 5700X | $150 |
| Motherboard | B550 ATX | $120 |
| RAM | 64 GB DDR4 3200 | $100 |
| PSU | 850W 80+ Gold | $100 |
| Case | Mid-tower ATX | $80 |
| NVMe SSD 1TB | Samsung 980 Pro | $90 |

**Total: ~$1,300** for a machine that runs 34B models at full VRAM, handles 70B models with CPU offload, and serves multiple users via Open WebUI.

[AMD Ryzen 7 5700X on Amazon](https://www.amazon.com/s?k=Ryzen+7+5700X&tag=YOURTAG-20)
[64GB DDR4 RAM Kit on Amazon](https://www.amazon.com/s?k=64GB+DDR4+3200+kit&tag=YOURTAG-20)

## Power Draw and Running Costs

GPU inference is power-hungry:

- RTX 3090 at full load: ~350W
- RTX 4090 at full load: ~450W
- Whole system at inference: 400–550W typical

At $0.15/kWh, running at full load 8 hours/day costs ~$18–24/month. Most AI servers don't run at full load constantly — they spike during inference and idle the rest of the time. Real-world costs for typical use are $10–15/month.

This is still far less than running equivalent cloud API calls at scale.

## What's Next

Once you have a GPU AI server running:

1. **Add Open WebUI** for a proper chat interface
2. **Set up Cloudflare Tunnels** for remote access from any device
3. **Try Continue.dev** for AI-assisted coding in VSCode/JetBrains — pointed at your Ollama server
4. **Experiment with embeddings** for building RAG pipelines with local document search

Local AI on consumer hardware is genuinely capable in 2026. The gap between Llama 3.1 70B and GPT-4o is real but narrowing — for many everyday tasks, local models are already good enough, and the privacy and cost advantages make the setup worthwhile.
