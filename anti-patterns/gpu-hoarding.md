# GPU Hoarding

You've got eight A100s reserved 24/7. Your actual utilization? 23%. You're burning $48K a month to feel safe. That's not infrastructure — that's an expensive security blanket.

If this sounds familiar, you're not alone. And you're hemorrhaging money.

---

**TL;DR:** Most teams pay 3–7x more per useful GPU-hour than they need to. Auto-shutdown, spot instances, and honest utilization audits will save you 40–70% of your GPU bill. Go check yours right now.

---

## The Psychology of GPU Hoarding

GPU hoarding is the AI equivalent of keeping a parking spot reserved downtown because you *might* need it on Tuesdays. It feels rational. It feels prudent. It's quietly bankrupting your company.

Here's how it happens:

1. **The scarcity mindset.** In 2023, GPUs were genuinely scarce. Teams learned that if you didn't reserve, you didn't get compute. That trauma persists even as supply has normalized.
2. **The "what if" factor.** "What if we need to retrain the model?" "What if traffic spikes?" "What if the new hire needs GPU access?" Teams hoard for hypotheticals.
3. **Nobody gets fired for over-provisioning.** Under-provisioning causes visible outages. Over-provisioning causes invisible waste. Guess which one gets attention?
4. **The dashboard lie.** Teams look at peak utilization (90%!) and feel justified. But average utilization tells the real story — and it's usually 20–30%.

## The Numbers Are Brutal

Studies consistently show the waste:

- [Up to 60% of GPU time](https://wetranscloud.com/gpu-utilization-in-mlops-maximizing-performance-without-overspending/) in ML workflows is wasted due to misallocation and idle capacity
- Teams [waste 30–50% of their budget](https://www.gmicloud.ai/blog/2025-gpu-cloud-cost-comparison) on GPUs that are provisioned but not actively training
- GPUs left running during debugging or overnight [waste 30–50% of spending](https://www.gmicloud.ai/blog/best-cloud-gpu-rentals-for-startups-in-2025-complete-comparison-guide)
- Traditional cloud providers force you to [pay for entire instances](https://www.runpod.io/articles/guides/cloud-gpu-pricing) even when GPU utilization runs at 20–30%

Let's do the math nobody wants to do.

### The $100K napkin calculation

| Scenario | Monthly Cost | Annual Cost | Avg Utilization | Effective $/useful-GPU-hour |
|---|---|---|---|---|
| 8× A100 reserved (AWS p4d.24xlarge) | ~$25K/mo | ~$300K/yr | 25% | $6.85/hr |
| 8× A100 on-demand, auto-scaled | ~$8K/mo | ~$96K/yr | 80% | $2.06/hr |
| Spot instances + checkpointing | ~$4K/mo | ~$48K/yr | 85% | $0.97/hr |
| API calls (for inference) | ~$2–5K/mo | ~$24–60K/yr | 100% | Pay per token |

**The reserved instance team is paying 3–7x more per useful compute hour.** That delta is a senior engineer's salary. Or your entire marketing budget. Or six months of runway.

## When APIs Are Cheaper Than GPUs

This is the calculation most teams refuse to do:

**Self-hosted inference makes sense when:**
- You're doing >1M+ requests/day consistently
- You need sub-50ms latency
- You're running a model that isn't available via API
- You have regulatory requirements for on-prem

**APIs are cheaper when:**
- Your traffic is bursty (most startups)
- You're still iterating on which model to use
- You're running <100K requests/day
- You don't have dedicated MLOps staff

A single A100 costs ~$2–3/hr on-demand. That's ~$2,000/mo. For that money, you can make roughly **2–4 million GPT-4o-mini API calls** at current pricing. Unless you're doing serious volume, the API is almost always cheaper — and you get zero ops burden.

## The Spot Instance Cheat Code

For training workloads (not serving), spot/preemptible instances are absurdly good value:

- **AWS Spot**: 60–90% cheaper than on-demand for GPU instances
- **GCP Preemptible**: Similar savings
- **RunPod / Vast.ai / Lambda**: Even cheaper, with marketplace-style bidding

The catch: they can be interrupted. The solution: **checkpointing.** Save model state every N steps. If your instance gets yanked, restart from the last checkpoint. This is a solved problem — every major training framework supports it.

Teams that implement checkpointing + spot instances typically save **60–80%** on training costs. Teams that don't are lighting money on fire for the convenience of not writing a checkpoint callback.

## The 5-Minute Audit

Before your next GPU purchase or reservation, answer these:

1. **What's your average GPU utilization over the last 30 days?** Not peak. Average.
   - Below 40%? You're hoarding.
   - Below 60%? You're probably hoarding.
   - Above 70%? You might be right-sized. Maybe.

2. **How many hours per day are your GPUs actively computing?**
   - Under 8 hours → you don't need reserved instances
   - Under 4 hours → you might not need GPUs at all

3. **Could this workload run on a smaller GPU?**
   - Most inference workloads that "need" an A100 actually run fine on an A10G or even a T4 with quantization
   - Training a 7B parameter model? You don't need 8× H100s

4. **Are you paying for GPUs to sit idle on weekends?**
   - If yes, implement auto-scaling or scheduled shutdowns. Today.

## The Auto-Shutdown Rule

This is the single highest-ROI change any ML team can make:

> **Set up auto-shutdown for idle GPU instances.**

Every cloud provider supports this. RunPod does it natively. AWS has Lambda-based solutions. It takes 30 minutes to implement. It saves 30–50% of your GPU bill overnight.

If your team doesn't have auto-shutdown configured, stop reading this guide and go set it up. Right now. I'll wait.

## Treat GPUs Like Taxis, Not Parking Spots

The mental model shift you need:

```
┌─────────────────────────────────────────────────────┐
│  ❌ Hoarding mindset                                │
│  "I need this GPU reserved because I might need it" │
├─────────────────────────────────────────────────────┤
│  ✅ Right-sizing mindset                            │
│  "I'll get a GPU when I need one and release it     │
│   when I'm done"                                    │
└─────────────────────────────────────────────────────┘
```

Making this work requires:
1. **Serverless GPU inference** (RunPod Serverless, Modal, Banana, etc.) for bursty inference
2. **Spot instances + checkpointing** for training
3. **Auto-scaling** for serving — scale to zero when idle
4. **Monitoring and alerts** on utilization — set alerts at <30% sustained utilization

## The CoreWeave Cautionary Tale

Even at the infrastructure *provider* level, GPU over-investment is a risk. CoreWeave's March 2025 IPO told the story: the company disclosed [**$12.9 billion in cumulative asset-backed debt**](https://www.reuters.com/markets/deals/cloud-firm-coreweave-files-us-ipo-2025-03-03/) raised through end of 2024. By August 2025, that had grown to [$11 billion in debt](https://fortune.com/2025/11/08/coreweave-earnings-debt-ai-infrastructure-bubble/) with $7.6 billion in current liabilities. By late 2025, total capital raised (debt + equity) [exceeded $25 billion](https://investors.coreweave.com/).

When even GPU *providers* are overleveraged, maybe individual teams should think twice about reserving capacity they don't need.

## The Bottom Line

Every GPU-hour you pay for but don't use is money directly subtracted from your runway, your team, or your product. The AI industry has a bizarre cultural norm where over-provisioning is seen as "being prepared" rather than what it actually is: **waste**.

Audit your GPU spend this week. I guarantee you'll find money to save.

---

## Further Reading

- [GPU Utilization in MLOps](https://wetranscloud.com/gpu-utilization-in-mlops-maximizing-performance-without-overspending/) — Transcloud's breakdown of where GPU time goes (and where it's wasted).
- [2025 GPU Cloud Cost Comparison](https://www.gmicloud.ai/blog/2025-gpu-cloud-cost-comparison) — GMI Cloud's head-to-head pricing analysis across providers.
- [Best Cloud GPU Rentals for Startups](https://www.gmicloud.ai/blog/best-cloud-gpu-rentals-for-startups-in-2025-complete-comparison-guide) — Practical guide for teams picking their first GPU provider.
- [Cloud GPU Pricing Guide](https://www.runpod.io/articles/guides/cloud-gpu-pricing) — RunPod's overview of pricing models and hidden costs.
- [The AI Infrastructure Bubble](https://developmentcorporate.com/saas/the-ai-infrastructure-bubble-4-surprising-reasons-the-90-billion-data-center-boom-could-end-in-a-bust/) — A sobering look at whether the $90B data center boom is sustainable.
