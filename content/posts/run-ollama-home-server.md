---
title: "How to Run Ollama on a Home Server (Local AI Without the Cloud)"
date: 2026-04-18
description: "Run Llama 4, Qwen3, and DeepSeek R1 on your own hardware with Ollama. This guide covers install, model selection, network setup, and real performance numbers — no fluff."
tags: ["ollama", "local ai", "home server", "llm", "self-hosting"]
categories: ["guides"]
showToc: true
---

If you're tired of sending your prompts to someone else's server, Ollama is the cleanest way out. It's a tool that runs large language models locally — one install command, one pull command, and you're talking to a capable AI that never leaves your machine.

This guide is for people who want to actually get it working, not just understand what it is. By the end you'll have a local model running, accessible from other devices on your network, and paired with a web interface you can use daily.

## Is Local AI Actually Good in 2026?

Yes, meaningfully so. A year ago the quality gap between local models and GPT-4o was obvious. Today it's much smaller. Qwen3 8B and Llama 4 Scout score within 5–10% of GPT-4o on standard benchmarks like MMLU and HumanEval. For writing, summarization, coding help, and Q&A, most people won't notice the difference on everyday tasks.

Where cloud models still win: very long context windows (1M+ tokens), multimodal tasks, and the absolute cutting edge of reasoning. For everything else, local is competitive.

## What Hardware You Need

The single most important factor is whether you have a GPU with enough VRAM to hold the model in memory. If the model fits in VRAM, inference is fast. If it doesn't, it spills to system RAM and becomes 5–10x slower.

| Hardware | What You Can Run | Speed (approx.) |
|----------|-----------------|-----------------|
| CPU only, 16 GB RAM | Qwen3 4B, small models | 2–5 tok/s |
| CPU only, 32 GB RAM | Qwen3 8B (slow) | 3–8 tok/s |
| GPU 8 GB VRAM | Qwen3 8B Q4 | 40–55 tok/s |
— | GPU 12 GB VRAM | Llama 4 Scout Q4 | 35–50 tok/s |
| GPU 24 GB VRAM | DeepSeek R1 32B Q4 | 25–40 tok/s |

Speeds are approximate for Ollama on a single GPU. CPU-only is usable for non-time-sensitive work — slow enough to read as it generates, but functional.

**Best value GPU for local AI:** [MSI RTX 4060 Ti Gaming X Slim 16GB](https://www.amazon.com/dp/B0CBK7H19M?tag=YOURTAG-20) (~$450 new) — 16 GB VRAM handles Llama 4 Scout and most 13B models comfortably.

## Installing Ollama

Ollama runs on Linux, macOS, and Windows. On Linux (the most common home server OS):

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

That's the whole install. The script downloads the binary, creates a systemd service, and starts it automatically. It also detects your NVIDIA GPU automatically if drivers are installed — no manual CUDA setup needed.

Verify it worked:

```bash
ollama --version
systemctl status ollama
```

Current stable version is **v0.21.0** (April 2026). If you see an older version, re-run the install script to update.

## The Model Landscape Right Now

Before pulling anything, here's an honest picture of what's worth running in 2026:

**Qwen3** (Alibaba) — The current best all-rounder for consumer hardware. Available from 0.6B to 235B. The 8B model outperforms Llama 3.1 70B on most tasks while needing a fraction of the VRAM. Has an optional "thinking mode" for chain-of-thought reasoning.

**Llama 4 Scout** (Meta) — A 17B Mixture-of-Experts model where only a portion of parameters activate per inference. Runs faster than its size suggests, excellent for 12 GB VRAM setups.

**DeepSeek R1** — Reasoning-focused. Before answering, it generates a visible chain of thought — useful for math, logic, and complex technical problems. The 32B version is the best local reasoning model if you have 24 GB VRAM.

**Qwen3-Coder** — Specialized for code. MoE architecture means only ~3B parameters are active per token despite an 80B total. Best local coding model currently available.

**What to skip:** Older Llama 3.1/3.2 models, Mistral 7B, and CodeLlama are all outclassed by Qwen3 equivalents in 2026. Only reach for them if you have a specific reason.

## Pulling and Running Models

Pull a model by name — Ollama downloads it from its own model library:

```bash
# Best starting point — runs on almost any hardware
ollama pull qwen3:4b

# Best all-rounder for 8 GB VRAM or 16+ GB RAM
ollama pull qwen3:8b

# Best for 12 GB VRAM — excellent quality
ollama pull llama4:scout

# Best reasoning model, needs 24 GB VRAM
ollama pull deepseek-r1:32b

# Best for coding, any GPU with 12+ GB VRAM
ollama pull qwen3-coder
```

Check what you have installed:

```bash
ollama list
```

Start a chat session:

```bash
ollama run qwen3:8b
```

You'll get a `>>>` prompt. Type your message, hit Enter, get a response. Type `/bye` or press `Ctrl+D` to exit.

One-shot from the terminal (useful for scripting):

```bash
echo "Summarize this in 3 bullet points: $(cat notes.txt)" | ollama run qwen3:8b
```

## Making Ollama Available on Your Network

By default, Ollama listens only on `localhost:11434`. To reach it from other machines — your phone, laptop, or a Docker container running a web UI — you need to change this.

Edit the systemd override file:

```bash
sudo systemctl edit ollama
```

This opens a blank file. Add:

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
```

Save, then apply:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

Test from another device on your network:

```bash
curl http://YOUR_SERVER_IP:11434/api/tags
```

You should get a JSON response listing your models. If you get "connection refused", Ollama is still on localhost — double-check the override file saved correctly.

## Using the API

Ollama exposes an OpenAI-compatible REST API. Any tool built for OpenAI can be pointed at your local Ollama by changing the base URL to `http://localhost:11434/v1`. The API key value is ignored — pass any string.

A basic chat request:

```bash
curl http://localhost:11434/api/chat \
  -d '{
    "model": "qwen3:8b",
    "messages": [{"role": "user", "content": "What is a reverse proxy?"}],
    "stream": false
  }'
```

This is how tools like Open WebUI, Continue.dev (VS Code AI assistant), and others connect to Ollama.

## Adding Open WebUI

The terminal works but a proper web interface is much nicer for daily use. Open WebUI is a polished, actively maintained ChatGPT-style frontend that talks directly to Ollama.

Requirements: Docker installed. If you don't have it: `curl -fsSL https://get.docker.com | sh`

```bash
docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

Visit `http://YOUR_SERVER_IP:3000`. The first account you create becomes the admin.

The `--add-host=host.docker.internal:host-gateway` line is Linux-specific — it lets the Docker container reach Ollama running on the host machine. Without it, the container can't find Ollama.

From Open WebUI you can pull new models, switch between them mid-conversation, and connect additional backends like the OpenAI API or Anthropic Claude alongside your local models.

See our [Docker Compose guide](/posts/docker-compose-beginners/) for more on running services like this.

## Managing Disk Space

Models live at `~/.ollama/models` (or `/usr/share/ollama/.ollama/models` for system installs). Sizes at Q4 quantization:

- 4B model: ~2.5 GB
- 8B model: ~5 GB
- 14B model: ~9 GB
- 32B model: ~20 GB

Remove a model you're not using:

```bash
ollama rm qwen3:4b
```

## Keeping Ollama Updated

Re-run the install script:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Ollama updates frequently — new model support, performance improvements, bug fixes. Worth updating monthly.

## Troubleshooting

**Getting CPU speeds instead of GPU:** Verify your drivers are working (`nvidia-smi` should show your GPU), then check whether Ollama is using it:

```bash
ollama run qwen3:8b --verbose 2>&1 | grep -i "gpu layers"
```

`gpu layers: 0` means Ollama isn't using the GPU. Common causes: missing NVIDIA drivers, or Ollama installed before drivers were present (reinstall Ollama after installing drivers).

**Out of memory error:** The model doesn't fit in your VRAM. Options: use a smaller model, or use a more aggressively quantized version. Instead of `qwen3:14b`, try `qwen3:14b-q4_K_M`. The suffix means 4-bit quantization — roughly half the VRAM at modest quality cost.

**Another device can't connect:** Check that Ollama is actually listening on all interfaces:

```bash
ss -tulpn | grep 11434
```

Should show `0.0.0.0:11434`. If it shows `127.0.0.1:11434`, the environment variable override didn't take — check the systemd override file with `sudo systemctl cat ollama`.

**Model outputs look wrong or repetitive:** Try a different quantization variant. Very aggressive quantizations (q2, q3) can noticeably degrade quality. Stick to q4_K_M or higher for reliable outputs.

## Frequently Asked Questions

**Can Ollama run without a GPU?**
Yes. CPU-only inference works but is slow — 2–8 tokens per second depending on model size and CPU. Usable for non-real-time tasks. For daily interactive use, a GPU makes a significant difference.

**Does Ollama work on Windows?**
Yes, Ollama has a native Windows installer at ollama.com/download. GPU acceleration via CUDA works on Windows too.

**How is this different from ChatGPT?**
Everything runs on your hardware. Your prompts never leave your machine, there's no subscription, and it works offline. The tradeoff is you're limited to what your hardware can run, and current local models are slightly behind the frontier cloud models.

**Can I run multiple models at the same time?**
Yes, if you have enough VRAM. Ollama loads models on demand and unloads them after a timeout. You can run concurrent requests to different models, but they'll compete for VRAM.

**What if I want to access it from outside my home network?**
Don't expose port 11434 directly to the internet — it has no authentication. Use a Cloudflare Tunnel with Cloudflare Access to put an auth gate in front of Open WebUI instead. We cover this in a separate guide.
