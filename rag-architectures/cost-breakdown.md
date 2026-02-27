# RAG Cost Breakdown: What You'll Actually Pay

> **TL;DR:** RAG costs are dominated by LLM inference (60–80% of total), not vector storage or embeddings. At startup scale (100 queries/day), expect $50–150/mo. At mid-scale (1K queries/day), $300–1,500/mo. At enterprise scale (10K queries/day), $2,000–15,000/mo. The single biggest cost lever is your choice of LLM — switching from GPT-4o to GPT-4o-mini cuts costs 10–15× with modest quality loss. The second biggest lever is smart model routing.

You've built your RAG pipeline. It works. Now comes the question nobody asked during the hackathon: "What's this going to cost in production?" The answer depends almost entirely on one thing — and it's probably not what you think.

---

## The Five Cost Buckets

Every RAG pipeline breaks down into five components. Understanding which ones actually matter saves you from optimizing the wrong thing.

1. **Embedding costs** — converting documents and queries to vectors
2. **Vector storage** — storing and indexing your embeddings
3. **Retrieval costs** — querying the vector database
4. **LLM inference** — generating answers from retrieved context
5. **Infrastructure** — compute, networking, orchestration

Spoiler: bucket #4 dwarfs everything else. But let's prove it with real numbers.

---

## Pricing Inputs

### Embedding Models

| Model | Price per 1M tokens | Dimensions |
|---|---|---|
| text-embedding-3-small | $0.02 | 1536 |
| text-embedding-3-large | $0.13 | 3072 |
| Cohere embed-v3 | $0.10 | 1024 |
| Voyage voyage-3 | $0.06 | 1024 |

### LLM Models (per 1M tokens)

| Model | Input | Output | Notes |
|---|---|---|---|
| GPT-5-nano | $0.05 | $0.40 | Cheapest GPT-5 family |
| GPT-4o-mini | $0.15 | $0.60 | Best cost/quality for most RAG |
| GPT-4.1-mini | $0.40 | $1.60 | Newer, slightly better |
| GPT-4o | $2.50 | $10.00 | High quality, high cost |
| GPT-4.1 | $2.00 | $8.00 | Latest flagship |
| Claude Haiku 4.5 | ~$1.00 | ~$5.00 | Anthropic's value option |
| Claude Sonnet 4 | ~$3.00 | ~$15.00 | Anthropic's workhorse |
| Gemini 2.0 Flash | ~$0.10 | ~$0.40 | Google's value option |

### Vector Database ([Pinecone](https://www.pinecone.io/) Serverless)

| Metric | Price |
|---|---|
| Storage | $0.33/GB/month |
| Read units | $8.25/million RUs |
| Write units | $2.00/million WUs |

---

## Scale 1: Startup — 10K Docs, 100 Queries/Day

**Setup:** 10,000 documents averaging 500 tokens each (5M total tokens). ~10K vectors at 1536 dimensions. 100 queries/day (3,000/month). Top-5 retrieval, ~2,000 tokens context per query, ~500 token output. Using text-embedding-3-small + GPT-4o-mini.

| Component | Monthly Cost |
|---|---|
| Document embedding (amortized) | $0.10 |
| Query embedding | $0.006 |
| Vector storage (60 MB) | $0.02 |
| Retrieval (read units) | $0.15 |
| LLM input (3K × 2,500 tokens) | $1.13 |
| LLM output (3K × 500 tokens) | $0.90 |
| Pinecone Standard minimum | $50.00 |
| Infrastructure (Railway/Fly.io) | $5–20 |
| **Total** | **~$55–75/mo** |

Notice the problem? At startup scale, **the Pinecone minimum ($50/mo) dominates**. You're paying for the platform, not usage. The actual RAG costs are ~$2.

### The Smarter Startup Setup

Switch to pgvector on [Supabase](https://supabase.com/) or [Neon](https://neon.tech/) and the picture changes dramatically:

| Component | Monthly Cost |
|---|---|
| Supabase Pro (includes pgvector) | $25.00 |
| Embeddings + LLM | ~$2.20 |
| App hosting | $5–10 |
| **Total** | **~$32–37/mo** |

---

## Scale 2: Mid-Stage — 100K Docs, 1K Queries/Day

**Setup:** 100,000 documents (50M tokens). ~200K chunks at 1536 dimensions. 1,000 queries/day (30,000/month). Top-5 retrieval with reranking. ~3,000 tokens context, ~500 token output. Using text-embedding-3-small + GPT-4o-mini + [Cohere reranking](https://cohere.com/rerank).

| Component | Monthly Cost |
|---|---|
| Document embedding (quarterly, amortized) | $0.33 |
| Query embedding | $0.06 |
| Vector storage (1.2 GB) | $0.40 |
| Retrieval (read units) | $2.48 |
| Reranking (Cohere) | $30.00 |
| LLM input (30K × 3,500 tokens) | $15.75 |
| LLM output (30K × 500 tokens) | $9.00 |
| Pinecone Standard | $50–80 |
| Infrastructure | $50–100 |
| **Total** | **~$160–240/mo** |

### The LLM Choice Is the Whole Ballgame

What happens if you swap GPT-4o-mini for GPT-4o?

| Component | GPT-4o-mini | GPT-4o |
|---|---|---|
| LLM input | $15.75 | $262.50 |
| LLM output | $9.00 | $150.00 |
| **Total LLM cost** | **$24.75** | **$412.50** |

That's **~17×** more expensive. For most RAG tasks, the quality difference is marginal. This is the single most important cost decision you'll make.

### Smart Model Routing

Don't use one model for everything. Route 70% of queries to GPT-4o-mini, 30% of complex queries to GPT-4o:

| Strategy | Monthly LLM Cost |
|---|---|
| All GPT-4o | ~$490 |
| All GPT-4o-mini | ~$25 |
| Smart routing (70/30 split) | ~$165 |
| **Savings vs all GPT-4o** | **~66%** |

---

## Scale 3: Enterprise — 1M Docs, 10K Queries/Day

**Setup:** 1,000,000 documents (500M tokens). ~3M chunks at 1536 dimensions. 10,000 queries/day (300,000/month). Top-10 retrieval with reranking. ~5,000 tokens context, ~800 token output. Mixed model routing: 60% GPT-4o-mini, 30% GPT-4.1-mini, 10% GPT-4.1.

| Component | Monthly Cost |
|---|---|
| Document embedding (quarterly, amortized) | $3.33 |
| Query embedding | $0.60 |
| Vector storage (18.4 GB) | $6.07 |
| Retrieval (read units) | $37.13 |
| Reranking | $300.00 |
| LLM — 60% GPT-4o-mini (180K queries) | $243.00 |
| LLM — 30% GPT-4.1-mini (90K queries) | $324.00 |
| LLM — 10% GPT-4.1 (30K queries) | $540.00 |
| Pinecone Enterprise | $500–800 |
| Infrastructure (dedicated, monitoring, redundancy) | $500–1,500 |
| **Total** | **~$2,500–3,800/mo** |

### Where the Money Goes at Scale

```
LLM inference    ████████████████████████████████  ~60-70%
Infrastructure   ████████████                      ~15-20%
Vector DB        ████████                          ~10-15%
Reranking        ████                              ~5-8%
Embeddings       █                                 ~0.1%
```

LLM inference dominates. Everything else is noise. If you're optimizing embedding costs before you've optimized your LLM strategy, you're rearranging deck chairs.

---

## Cost Optimization Playbook

### Quick Wins (Do These First)

1. **Use the cheapest embedding model that works.** `text-embedding-3-small` at $0.02/1M is 6.5× cheaper than `text-embedding-3-large`. Test both — the quality difference may not matter for your use case.

2. **Use GPT-4o-mini (or GPT-5-nano) as your default.** Only route to expensive models when quality requires it.

3. **Cache aggressively.** If you see the same or similar queries, cache the LLM response. At 10K queries/day, a 20–30% cache hit rate saves hundreds per month.

4. **Use prompt caching.** OpenAI cached input pricing is 50–90% cheaper. Structure your prompts so the system message + retrieved context is consistent enough to hit the cache.

5. **Reduce context size.** Don't stuff all 10 retrieved chunks into the prompt. Use reranking to select top-3, or use contextual compression to extract only relevant sentences.

### Medium-Effort Optimizations

6. **Binary quantization.** Cuts vector storage 32× with ~96% accuracy retention. Essential at 1M+ vectors.

7. **Embedding dimension reduction.** `text-embedding-3-large` supports Matryoshka dimensions — use 1024 instead of 3072 for 3× storage savings with <2% quality loss.

8. **Batch API for non-real-time tasks.** OpenAI's Batch API gives a 50% discount. Use for document processing, evaluation runs, and anything not user-facing.

9. **pgvector instead of managed vector DB** at small-to-medium scale. Saves $50–500/mo in platform fees.

### Advanced Optimizations

10. **Self-host embedding models.** Run `nomic-embed-text` or `bge-large` on your own GPU for ~$0 marginal cost (after $200–500/mo GPU rental).

11. **Self-host LLMs for high volume.** At 10K+ queries/day, a dedicated GPU running Llama 3.1 70B may be cheaper than API costs. (See the [self-hosted LLM guide](../choosing-your-llm-provider/self-hosted.md) for the full breakeven math.)

12. **Smart model routing with a classifier.** Train a tiny classifier to route queries by complexity. 60–80% savings over using expensive models for everything.

---

## Cost Comparison Summary

| Scale | Monthly Queries | Budget Option | Standard Option | Premium Option |
|---|---|---|---|---|
| **Startup** | 3K | $30–40 (pgvector + GPT-4o-mini) | $55–75 (Pinecone + GPT-4o-mini) | $200–400 (Pinecone + GPT-4o) |
| **Mid** | 30K | $100–160 (pgvector + mixed routing) | $160–240 (Pinecone + GPT-4o-mini) | $500–700 (Pinecone + GPT-4o) |
| **Enterprise** | 300K | $1,500–2,500 (self-hosted + routing) | $2,500–3,800 (managed + mixed routing) | $8,000–15,000 (managed + GPT-4o/4.1) |

---

## Gotchas

1. **Embedding costs are negligible — don't optimize them first.** Teams spend hours comparing embedding prices when the LLM cost is 100× larger.
2. **Vector DB minimum fees matter at small scale.** Pinecone's $50/mo minimum costs more than your actual usage for the first year.
3. **Reranking adds up.** At 10K queries/day, Cohere reranking runs ~$300/mo. Consider self-hosted rerankers (ColBERT, bge-reranker) if this matters.
4. **Data egress costs are hidden.** If your vector DB is in a different cloud than your app, data transfer fees add 5–15%.
5. **Monitoring and observability aren't free.** Budget $50–200/mo for [LangSmith](https://smith.langchain.com/), [Helicone](https://helicone.ai/), or equivalent to understand where your money goes.
6. **The "free tier" trap.** Chroma, pgvector, and self-hosted options are "free" but you're paying in engineering time. At $150K/yr loaded cost, every hour of infra debugging costs $75.

---

## Opinionated Recommendations

1. **Under $100/mo budget:** pgvector + GPT-4o-mini. No managed vector DB.
2. **$100–500/mo budget:** Pinecone Standard or Qdrant Cloud + GPT-4o-mini + Cohere reranking. This is the sweet spot for most products.
3. **$500–5,000/mo budget:** Start implementing smart model routing. Consider self-hosted embedding models. This is where optimization pays off.
4. **$5,000+/mo budget:** Seriously evaluate self-hosted LLMs. A single A100 GPU (~$2–3/hr ≈ $1,500–2,200/mo) running Llama 3.1 70B can handle 50K+ queries/day.
5. **Always measure before optimizing.** Use Helicone or LangSmith to see your actual cost breakdown before changing anything.

---

## Further Reading

- [OpenAI API Pricing](https://platform.openai.com/docs/pricing) — Current pricing for all models, including cached input and batch discounts
- [Pinecone Pricing](https://www.pinecone.io/pricing/) — Serverless pricing calculator and read/write unit explanation
- [Cohere Reranking Pricing](https://cohere.com/pricing) — Per-search pricing for the Rerank API
- [The Economics of RAG: Cost Optimization for Production Systems](https://thedataguy.pro/blog/2025/07/the-economics-of-rag-cost-optimization-for-production-systems/) — Deep dive into cost modeling at scale
- [Helicone](https://helicone.ai/) — LLM observability platform for tracking per-query costs in production
