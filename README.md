<div align="center">

# 🏗️ AI Infrastructure Decision Guide

**A practical guide to AI infrastructure — what to use, what to avoid, and how much it actually costs.**

Real costs. Real architectures. Real lessons.

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Last Updated](https://img.shields.io/badge/Updated-February%202026-blue.svg)](#)

</div>

---

You're an engineer. Someone on your team says "let's add AI." You search for how to actually deploy LLMs in production and get 50 blog posts that say "it depends" and link to vendor docs.

This guide is the opposite. **Opinionated, numbers-backed, battle-tested.** We tell you what to use, what to avoid, and how much it'll actually cost — from a $50/month side project to a 5M-pages/month document pipeline.

> Think [og-aws](https://github.com/open-guides/og-aws) for AI infrastructure.

---

## If You Only Read Three Pages

1. **[Provider Comparison Matrix](choosing-your-llm-provider/comparison-matrix.md)** — Azure vs. Bedrock vs. Vertex AI vs. direct API, decided in one table
2. **[RAG vs. Fine-Tuning](rag-architectures/rag-vs-fine-tuning.md)** — The most common architectural decision, with real cost math
3. **[Real-World Failures](real-world/failures.md)** — Companies that burned $50K+ so you don't have to

---

## What's Inside

### Choosing Your LLM Provider

| Guide | The Question It Answers |
|-------|------------------------|
| [Comparison Matrix](choosing-your-llm-provider/comparison-matrix.md) | Which provider should I use? (Spoiler: it depends on lock-in tolerance, not features) |
| [AWS Bedrock](choosing-your-llm-provider/aws-bedrock.md) | Is Bedrock worth the rate-limit pain? What's AgentCore? |
| [Azure OpenAI](choosing-your-llm-provider/azure-openai.md) | How do PTUs work? When does Azure beat direct OpenAI? |
| [GCP Vertex AI](choosing-your-llm-provider/gcp-vertex-ai.md) | Is Gemini Flash really that cheap? What's the grounding pricing trap? |
| [Self-Hosted](choosing-your-llm-provider/self-hosted.md) | When does self-hosting break even? (Hint: $5K+/month API spend) |

### RAG Architectures

| Guide | The Question It Answers |
|-------|------------------------|
| [Naive vs. Advanced RAG](rag-architectures/naive-vs-advanced-rag.md) | Do I need a complex RAG pipeline? (Usually no) |
| [RAG vs. Fine-Tuning](rag-architectures/rag-vs-fine-tuning.md) | Should I fine-tune or build RAG? Cost comparison at every scale |
| [Vector Database Comparison](rag-architectures/vector-db-comparison.md) | Pinecone vs. Weaviate vs. pgvector — pricing, limits, honest reviews |
| [RAG Cost Breakdown](rag-architectures/cost-breakdown.md) | What does RAG actually cost from 1K to 10M queries/month? |

### Architecture Patterns

| Guide | The Question It Answers |
|-------|------------------------|
| [Production Chatbot](architecture-patterns/chatbot.md) | How do I build a chatbot that costs $150/month at 100K users? |
| [Agent Systems](architecture-patterns/agent-systems.md) | How do multi-agent systems fail in production? (Cost multiplication, mostly) |
| [Batch Processing](architecture-patterns/batch-processing.md) | How do I process millions of items at 50% off? |
| [Document Processing](architecture-patterns/document-processing.md) | OCR vs. vision models — why the 10-20× cost difference matters |

### GPU Compute

| Guide | The Question It Answers |
|-------|------------------------|
| [Cloud GPU Comparison](gpu-compute/cloud-gpu-comparison.md) | Who's cheapest for GPUs? ($1.38/hr on Thunder vs. $14/hr on GCP) |
| [Cost Optimization](gpu-compute/cost-optimization.md) | How do I cut GPU costs 80-95%? (Quantization, batching, right-sizing) |
| [Spot vs. Reserved](gpu-compute/spot-vs-reserved.md) | Should I commit or gamble? Interruption rates and break-even math |

### Anti-Patterns

| Guide | The Expensive Lesson |
|-------|---------------------|
| [Over-Engineering RAG](anti-patterns/over-engineering-rag.md) | An intern with Elasticsearch beat a 3-month RAG project |
| [The Fine-Tuning Trap](anti-patterns/fine-tuning-trap.md) | $47K on fine-tuning, beaten by GPT-4 with a good prompt |
| [GPU Hoarding](anti-patterns/gpu-hoarding.md) | 23% utilization on 8× A100s = $48K/month security blanket |
| [Vendor Lock-In](anti-patterns/vendor-lock-in.md) | Model retired, region down, production counting down to death |

### Real-World Case Studies

| Guide | What Happened |
|-------|--------------|
| [Failures](real-world/failures.md) | Real companies, real money lost, real post-mortems |

---

## Principles

1. **Start with the cheapest thing that works.** GPT-4o Mini before GPT-4o. Prompt engineering before fine-tuning. System prompt before RAG.
2. **Measure before you optimize.** Don't quantize until you know your baseline. Don't add caching until you've profiled.
3. **The model is the easy part.** Memory management, guardrails, cost control, and observability are where production lives or dies.
4. **Every page gives you an answer, not "it depends."** Real thresholds. Real dollar amounts. Real decisions.

---

## Contributing

Found something wrong? Discovered a gotcha in production? Have a war story?

Read [CONTRIBUTING.md](CONTRIBUTING.md) for the style guide. The short version: bring numbers, have opinions, cite your sources.

---

## License

[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Share it, adapt it, just give credit and keep it open.

---

<div align="center">

**If this saved you time or money, [⭐ star the repo](../../stargazers).**

</div>
