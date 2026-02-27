# Cloud GPU Comparison: Every Major Provider, Real Prices

You just got budget approval for your ML workload. You open up AWS, GCP, and Azure pricing pages — and immediately regret it. Three different naming conventions, five different instance families, and a 10× price spread for what looks like the same GPU. Sound familiar?

This guide cuts through the noise. Every major cloud GPU provider, real per-GPU-hour prices, and the tradeoffs nobody puts in their marketing pages.

> **TL;DR:** H100 rental ranges from ~$1.40/GPU-hr (marketplace/spot) to ~$14/GPU-hr (GCP on-demand). AWS cut H100 prices 44% in mid-2025 but *raised* H200 prices 15% in January 2026. Indie clouds like Lambda ($2.86), RunPod ($1.99), and Vast.ai (~$1.47) are 2–5× cheaper than hyperscalers — but you trade ecosystem, compliance, and SLAs. H200 is the new default for large inference; B200 is rolling out at ~2× H100 performance with limited cloud availability.

---

## Per-GPU-Hour Pricing (February 2026)

All prices are on-demand, per single GPU-hour, USD, US regions.

| Provider | GPU | Instance / SKU | $/GPU-hr | Notes |
|----------|-----|----------------|----------|-------|
| **AWS** | H100 80 GB | p5.48xlarge (8×) | **$6.88** | $55.04/hr ÷ 8. Unchanged since mid-2025 cut |
| **AWS** | H200 141 GB | p5e.48xlarge (8×) | **$4.97** | $39.80/hr ÷ 8. Hiked 15% in Jan 2026 (was $4.33) |
| **AWS** | H200 141 GB | p5en.48xlarge (8×) | **$5.20** | $41.61/hr ÷ 8. NVLink-connected |
| **AWS** | A100 80 GB | p4d.24xlarge (8×) | **~$2.45** | Post mid-2025 cuts (~45% reduction) |
| **Azure** | H100 80 GB | NC H100 v5 (1×) | **$6.98** | Single-GPU VM, East US |
| **Azure** | A100 80 GB | NC A100 v4 (1×) | **~$3.40** | Single-GPU VM |
| **GCP** | H100 80 GB | a3-highgpu-1g (1×) | **$14.19** | Expensive; 8-GPU a3 nodes ~$3/GPU-hr after CUD |
| **GCP** | A100 80 GB | a2-ultragpu-1g | **~$3.67** | |
| **Oracle** | H100 80 GB | BM.GPU.H100.8 (8×) | **$10.00** | $80/hr bare-metal ÷ 8 |
| **Lambda Labs** | H100 SXM 80 GB | 8-GPU HGX | **$2.86** | Consistently among cheapest "real" clouds |
| **CoreWeave** | H100 HGX 80 GB | — | **$4.76** | Enterprise-grade GPU cloud |
| **Paperspace** | H100 80 GB | Dedicated | **$5.95** | Long-term commitment discounts available |
| **RunPod** | H100 80 GB PCIe | Community cloud | **$1.99** | Marketplace; reliability varies |
| **Vast.ai** | H100 80 GB | Third-party hosts | **~$1.47** | Marketplace lowest; quality varies wildly |
| **Thunder Compute** | H100 80 GB | — | **$1.38** | Newer entrant; claims top-tier reliability |
| **Cudo Compute** | H100 80 GB | — | **~$1.80** | |
| **TensorDock** | H100 80 GB | — | **~$2.25** | |

### H200 Cloud Pricing (Limited Availability)

| Provider | $/GPU-hr | Notes |
|----------|----------|-------|
| AWS (p5e) | $4.97 | Most available; hiked Jan 2026 |
| AWS (p5en) | $5.20 | NVLink variant |
| Jarvislabs | ~$3.80 | Reported Jan 2026 |
| Lambda Labs | ~$3.50 | Limited availability |

### A100: The Budget Workhorse

A100 80 GB is now sub-$1/GPU-hr on open marketplaces (Vast.ai, RunPod) and $2–3.50 on hyperscalers. It remains the best value for workloads that don't need H100-class bandwidth.

---

## GPU Specs at a Glance

| GPU | Architecture | VRAM | Memory BW | FP16 TFLOPS | FP8 TFLOPS | Interconnect | TDP |
|-----|-------------|------|-----------|-------------|------------|--------------|-----|
| A100 80 GB | Ampere | 80 GB HBM2e | 2.0 TB/s | 312 | — | NVLink 600 GB/s | 400 W |
| H100 SXM | Hopper | 80 GB HBM3 | 3.35 TB/s | 990 | 1,979 | NVLink 900 GB/s | 700 W |
| H200 SXM | Hopper | 141 GB HBM3e | 4.8 TB/s | 990 | 1,979 | NVLink 900 GB/s | 700 W |
| B200 | Blackwell | 192 GB HBM3e | 8.0 TB/s | ~2,250 | ~4,500 | NVLink 1.8 TB/s | 1000 W |
| L4 | Ada Lovelace | 24 GB GDDR6 | 300 GB/s | 121 | 242 | PCIe Gen4 | 72 W |
| AMD MI300X | CDNA 3 | 192 GB HBM3 | 5.3 TB/s | 1,307 | 2,615 | IF Links | 750 W |

### What the Generational Leaps Actually Mean

- **H100 → H200:** Same compute, but 76% more VRAM and 43% more bandwidth. That's a massive win for large-context inference — your KV cache finally fits.
- **H100 → B200:** ~2.3× compute, 2.4× VRAM, 2.4× bandwidth. MLPerf shows ~57% faster training throughput; inference up to 2× on large models.
- **A100 → H100:** ~3× FP16, 1.7× bandwidth. The generational jump that mattered most for training.

---

## Matching GPUs to Workloads

This is where people waste the most money. An H100 running a 7B model is like renting a semi-truck to move a bookshelf.

| Workload | Recommended GPU | Why |
|----------|----------------|-----|
| **Inference: 7B–13B models** | L4 (24 GB) or A10G | Cheap ($0.50–0.80/hr), sufficient VRAM for quantized small models |
| **Inference: 70B (quantized)** | A100 80 GB or H100 | 70B INT4 ≈ 35 GB; A100 handles it. H100 gives 2× throughput |
| **Inference: 70B (FP16)** | 2× H100 or 1× H200 | ~140 GB needed; H200's 141 GB VRAM is perfect |
| **Inference: 200B+ / MoE (DeepSeek-R1)** | 8× H200 or 4× B200 | 671B params need massive memory; H200×8 = 1.1 TB |
| **Training: 7B from scratch** | 8× A100 80 GB | Still cost-effective; H100 if you need speed |
| **Training: 70B+ from scratch** | 64+ H100/H200 cluster | Multi-node NVLink/InfiniBand required |
| **Fine-tuning: LoRA on 70B** | 1–2× A100 80 GB | LoRA reduces the memory footprint dramatically |
| **Batch inference at scale** | H100 or H200 | Bandwidth-bound; H-series excels |
| **Cost-sensitive experimentation** | RunPod/Vast.ai A100 | Sub-$1/hr; acceptable for non-production |

---

## Market Dynamics: What's Actually Happening (February 2026)

### The Price Rollercoaster

The GPU cloud market has been anything but stable:

1. **2023–early 2025:** H100 scarcity. $8–11/GPU-hr on hyperscalers. Lambda had waitlists.
2. **Mid-2025:** AWS cut H100 prices 44%. GCP and Azure followed with 20–30% cuts. Supply caught up as TSMC capacity expanded.
3. **Late 2025:** H100 market prices stabilized at $2–4/GPU-hr. A100s crashed to sub-$1 on marketplaces.
4. **January 2026:** AWS *raised* H200 prices 15% (p5e: $34.61 → $39.80/hr). Demand for H200/B200 outstrips supply while H100 supply is comfortable.

### Supply and Demand Right Now

- **H100:** Oversupply. Prices still falling on marketplaces. Good buyer's market.
- **H200:** Tight supply. AWS raising prices signals real scarcity. Only hyperscalers and Lambda have reliable inventory.
- **B200:** Just entering cloud availability (AWS p6 expected H1 2026). Expect launch pricing to be eye-watering ($8–12/GPU-hr) before normalizing.
- **AMD MI300X:** Aggressive pricing (~$3–4/GPU-hr) but the software ecosystem is still catching up. Worth evaluating for inference if your stack supports ROCm.

---

## Hyperscaler vs. Indie Cloud: The Real Tradeoff

This is the decision that trips up most teams. Here's the honest version.

### Why Hyperscalers (AWS / Azure / GCP)

- **Compliance and certifications** — SOC2, HIPAA, FedRAMP
- **Ecosystem integration** — S3, IAM, VPC, managed Kubernetes
- **Multi-region availability** with SLAs
- **Enterprise support contracts**
- **Capacity Blocks** — guaranteed GPU for defined windows
- But: **2–5× more expensive** per GPU-hour
- And: **vendor lock-in** through ecosystem gravity

### Why Indie Clouds (Lambda, CoreWeave, RunPod, Vast.ai)

- **40–70% cheaper** per GPU-hour
- **Simpler** — less config overhead, faster to spin up
- **Often bare-metal** — no virtualization overhead
- But: **limited regions** (often 1–3 US locations)
- And: **no compliance certifications** for most
- And: **variable reliability** (especially marketplaces like Vast.ai)
- And: **limited ecosystem** — you bring your own storage, networking, orchestration

### The Opinionated Take

**For production inference serving:** Use hyperscalers. The 2× premium pays for itself in SLAs, autoscaling, and not getting paged at 3 AM.

**For training and experimentation:** Use Lambda Labs or CoreWeave. They're reliable enough, and the savings compound fast on multi-day training runs.

**For batch/offline processing:** Spot instances on any provider, or marketplace GPUs (RunPod, Vast.ai). Checkpoint aggressively.

**For startups burning through credits:** Most hyperscalers offer $100K–$300K in GPU credits. Use those first. When credits run out, migrate training to Lambda/CoreWeave and keep serving on the hyperscaler.

---

## Gotchas That'll Bite You

1. **Per-instance vs. per-GPU pricing.** AWS and GCP only sell 8-GPU instances for H100/H200. You can't rent 1 GPU. If you need a single GPU, Azure (NC H100 v5) or Lambda are your options.

2. **Egress costs add up fast.** Hyperscalers charge $0.08–0.12/GB for data out. A training run downloading 10 TB of data and uploading checkpoints adds a surprising line item.

3. **Regional price variance.** AWS US-West is 10–30% more expensive than US-East for the same instance. Check before you deploy.

4. **Capacity Blocks ≠ Reserved Instances.** AWS Capacity Blocks (for ML) are time-boxed reservations, not the same as traditional RIs. Prices float with demand.

5. **"Per GPU-hour" is misleading on 8-GPU nodes.** You pay for the full node. If your job only uses 4 GPUs, you're wasting 50%.

6. **Marketplace GPUs have no SLA.** Your Vast.ai instance can disappear without warning. Treat it as ephemeral compute only.

---

## Further Reading

- [AWS EC2 P5 pricing](https://aws.amazon.com/ec2/pricing/) — Current on-demand and spot rates for GPU instances
- [ThunderCompute H100 Pricing Comparison](https://www.thundercompute.com/) — Independent cross-provider GPU price tracker
- [IntuitionLabs H100 Rental Analysis](https://www.intuitlabs.ai/) — Regularly updated marketplace pricing analysis
- [The Register: "AWS raises GPU prices 15%"](https://www.theregister.com/) — Coverage of the January 2026 H200 price hike
- [BentoML LLM Inference Handbook](https://www.bentoml.com/) — Practical guidance on choosing the right GPU for your model
- [Lightly.ai B200 vs H100 Benchmarks](https://www.lightly.ai/) — Real-world performance comparisons across GPU generations
