# Spot vs. Reserved GPU Instances: When to Commit, When to Gamble

You're about to kick off a week-long training run. Do you burn $6.88/GPU-hr on on-demand H100s and sleep soundly, or roll the dice at $2.00/GPU-hr on spot and pray AWS doesn't yank your instance at 2 AM? That decision — multiplied across dozens of GPUs and thousands of hours — is often the difference between a $50K cloud bill and a $200K one.

Here's how to make that call with data instead of gut feeling.

> **TL;DR:** Spot GPUs save 60–90% but H100 spot interruption rates hit 15–20% — expect restarts during day-long training runs. Reserved instances (1–3 yr) save 40–72% with zero interruption risk. **Rule of thumb:** If your GPU utilization is >60% steady-state, reserve. If your workload checkpoints cleanly and tolerates restarts, use spot. Never use spot for user-facing inference.

---

## The Three Pricing Tiers

| Tier | Discount vs. On-Demand | Commitment | Interruption Risk |
|------|------------------------|------------|-------------------|
| **On-Demand** | Baseline (0%) | None | None |
| **Spot / Preemptible** | 60–90% off | None | High (reclaimed anytime) |
| **Reserved / Committed** | 40–72% off | 1–3 years | None |

---

## What You'll Actually Save (February 2026)

### Spot Discounts

| Provider | GPU | Spot $/GPU-hr | On-Demand $/GPU-hr | Savings |
|----------|-----|---------------|---------------------|---------|
| AWS | H100 (p5) | ~$2.00–2.50 | $6.88 | **64–71%** |
| AWS | A100 (p4d) | ~$0.80–1.20 | $2.45 | **51–67%** |
| GCP | H100 (a3) | ~$2.50–3.00 | $14.19* | **79–82%** |
| GCP | A100 (a2) | ~$1.10 | $3.67 | **70%** |
| Azure | H100 (NC v5) | ~$2.10 | $6.98 | **70%** |

*GCP's on-demand H100 pricing is unusually high — spot actually brings it to competitive levels.

### Reserved / Committed Use Discounts

| Provider | Mechanism | 1-Year Savings | 3-Year Savings |
|----------|-----------|----------------|----------------|
| **AWS** | Savings Plans / Reserved Instances | ~30–40% | ~55–72% |
| **Azure** | Reserved VM Instances / Savings Plan for Compute | ~30–40% | ~50–72% |
| **GCP** | Committed Use Discounts (CUD) | ~28–37% | ~52–57% |
| **CoreWeave** | Long-term contracts | ~20–30% | ~40–50% |
| **Lambda Labs** | No formal reservations | — | — |

Azure claims up to 72% with reservations (up to 80% combined with Hybrid Benefit). GCP GPU CUDs cap around 57% depending on machine family.

---

## Interruption Rates: The Numbers Nobody Advertises

Here's what you're actually signing up for with spot instances. Data from [AWS Spot Instance Advisor](https://aws.amazon.com/ec2/spot/instance-advisor/) and real-world tracking (February 2026):

| Instance Type | GPU | Typical Interruption Rate | What That Means |
|---------------|-----|--------------------------|-----------------|
| p4d.24xlarge | 8× A100 | **5–10%** | Relatively stable; your most reliable spot option for ML |
| p5.48xlarge | 8× H100 | **15–20%** | Scarce capacity; trending toward the higher end |
| g6.xlarge | L4 | **5–10%** | Decent budget option |
| g5.xlarge | A10G | **<5%** | Very stable |

**Reading those percentages:**
- **<5%:** Fairly safe. A day-long run will probably complete.
- **5–10%:** Expect roughly 1 interruption per day-long run. Checkpoint every 15 min.
- **15–20%:** Expect multiple interruptions per day. Only for highly fault-tolerant workloads.

A few things worth knowing: AWS claims "95% of Spot instances run to completion" across all types — but GPU instances live in the noisy 5%. H100 spot is particularly constrained. GCP gives you **30 seconds** of preemption notice. AWS gives **2 minutes**. Azure gives **30 seconds**. That's your checkpoint window — plan accordingly.

---

## Commitment Terms: Provider by Provider

### AWS

- **Savings Plans:** 1-yr or 3-yr. Commit to a $/hr spend (not a specific instance). Flexible across instance families. Up to 66% off with Compute Savings Plans, up to 72% with EC2 Instance Savings Plans.
- **Reserved Instances:** 1-yr or 3-yr. Locked to instance type + region. All Upfront gets the maximum discount.
- **Capacity Blocks for ML:** Reserve specific GPU instances for defined time windows (hours to weeks). Prices float with demand — AWS hiked these ~15% in January 2026 for p5e/p5en. This didn't affect standard on-demand or Savings Plan pricing.
- **No early termination** on RIs. You pay regardless of usage.

### Azure

- **Reserved VM Instances / Savings Plan for Compute:** 1-yr or 3-yr. Up to 72% off (up to 80% combined with Azure Hybrid Benefit for existing on-prem licenses).
- **More flexible** than AWS RIs — exchange/refund policies exist (with penalties).
- **Spot priority:** Azure lets you set a max price and evicts when the market exceeds it.

### GCP

- **Committed Use Discounts (CUD):** 1-yr or 3-yr. Commit to vCPU + memory amounts (not specific instances). Applies automatically. GPU CUDs offer up to ~57% off depending on machine family and term.
- **Resource-based CUDs** are more flexible than AWS RIs — you commit to resources, not instance types.
- **Spend-based CUDs** also available for broader flexibility.
- **Spot VMs (formerly Preemptible):** 60–91% discount on unused GPU capacity for fault-tolerant workloads.

### Indie Clouds

- **CoreWeave** offers contract-based pricing (negotiated, typically 6–24 months).
- **Lambda Labs** has no formal reservation system — pricing is flat on-demand.
- **RunPod/Vast.ai** — no reservations; marketplace pricing only.

---

## When Spot Makes Sense (and When It Doesn't)

### Use Spot For

1. **Fault-tolerant distributed training.** Checkpoint every 15–30 min. If a node dies, restart from last checkpoint. The 60–70% savings easily justify the overhead.
2. **Batch inference and offline processing.** Processing a backlog of 10M images? Queue-based architecture. Interrupted batches get re-queued automatically.
3. **Hyperparameter sweeps.** Running 100 experiments at 2–4 hours each. Some get interrupted? Just re-run them.
4. **CI/CD GPU testing.** Short jobs with automatic retry.

### Never Use Spot For

1. **User-facing inference.** A 2-minute preemption kills your API's SLA. Full stop.
2. **Multi-node training without checkpointing.** One spot interruption kills ALL nodes' progress.
3. **Jobs that can't checkpoint.** If your framework doesn't support mid-run saves, spot is pure gambling.
4. **Time-sensitive deadlines.** "We need this model by Friday" + spot interruptions = missed deadline.

---

## Architecting for Spot: The Playbook

### Your Checkpointing Strategy

```
Every 15 minutes:
  1. Save model weights to persistent storage (S3/GCS)
  2. Save optimizer state
  3. Save data loader position
  4. Save RNG states (for reproducibility)
  
On interruption signal (2 min warning on AWS):
  1. Trigger emergency checkpoint
  2. Graceful shutdown
  
On restart:
  1. Load latest checkpoint
  2. Resume from exact position
```

### Key Patterns That Work

1. **Persistent Spot Requests (AWS).** Configure `maintain` fleet type so AWS automatically relaunches your instance when capacity returns.

2. **Multi-AZ / Multi-Region diversification.** Don't put all spot instances in one AZ. Spread across 3+ AZs to reduce correlated interruptions.

3. **Spot + On-Demand hybrid.** Run 70% spot, 30% on-demand. The on-demand nodes maintain cluster state and coordination; spot nodes are elastic workers.

4. **Instance type diversification.** Request multiple types (p5 + p4d + g5) with Spot Fleet. AWS picks whatever has capacity.

5. **Checkpoint to object storage, not local disk.** Local NVMe dies with the instance. Always checkpoint to S3/GCS/ABS.

### Framework Support for Checkpointing

| Framework | Checkpointing | Spot-Friendly? |
|-----------|---------------|----------------|
| PyTorch Lightning | Built-in `ModelCheckpoint` | Excellent |
| HuggingFace Trainer | `save_steps` parameter | Good |
| DeepSpeed | `save_checkpoint()` | Good (ZeRO checkpoints are large) |
| Raw PyTorch | Manual `torch.save()` | Works but you own it |
| Megatron-LM | Built-in periodic saving | Good |

---

## Reserved Instance War Stories

These are the gotchas that cost real teams real money:

1. **The utilization trap.** You reserved 8× H100 for a year but only use them 40% of the time. You just paid 60% for nothing. **Always do the math:** `reserved_cost < on_demand_cost × expected_utilization`.

2. **Technology lock-in.** You locked into H100 RIs for 3 years. B200 instances launch at competitive prices 6 months later. You're stuck paying for last-gen hardware.

3. **Region lock (AWS).** AWS RIs are region-locked. If you need to move workloads, the RI doesn't travel with you.

4. **No refunds (mostly).** AWS RIs have no early termination. Azure is slightly better (exchanges with penalties). GCP CUDs can't be cancelled.

5. **Capacity ≠ guaranteed.** A reserved instance guarantees the *price*, but in rare cases capacity shortages can still delay launches. AWS Capacity Reservations (a separate product) guarantee actual capacity.

6. **The price cut risk.** This actually happened: AWS cut H100 on-demand by 44% in mid-2025. Anyone who bought a 3-year RI before the cut found their "discounted" rate was now *more expensive* than the new on-demand price.

7. **Accounting complexity.** Reserved capacity shows up differently in cost allocation. Make sure your finance team understands the amortized vs. cash-flow view before you commit.

---

## The Decision Matrix

| Scenario | Best Option | Expected Savings |
|----------|-------------|-----------------|
| Steady-state inference (24/7) | 1-yr or 3-yr Reserved | 40–72% vs. on-demand |
| Training runs (days–weeks, fault-tolerant) | Spot with checkpointing | 60–90% vs. on-demand |
| Training runs (deadline-driven) | On-demand or Capacity Blocks | 0% (you're paying for reliability) |
| Burst experimentation | Spot or indie cloud on-demand | 40–80% |
| >60% utilization, predictable | Reserved | 40–60% |
| <30% utilization, bursty | On-demand | Pay only what you use |
| Multi-node distributed training | On-demand or reserved | 0–60% (spot too risky for H100) |

---

## The Opinionated Take

**If you're a startup:** Don't buy reserved instances until you've been running at >60% utilization for 3+ months. Start with on-demand or spot, understand your usage pattern, *then* commit.

**If you're an enterprise:** Reserved instances are almost always worth it for your base load. Layer spot on top for elastic training. Budget for on-demand as overflow.

**If you're a researcher:** Spot everything. Checkpoint religiously. Your time isn't billed to a customer SLA. And a tip: p4d (A100) spot is significantly more stable than p5 (H100) spot.

**The January 2026 lesson:** AWS raised high-end GPU Capacity Block prices ~15% with minimal notice (affecting p5e/p5en). Standard on-demand and Savings Plan pricing was unaffected — but it's a reminder that reserved pricing can shift. Factor this into multi-year commitments.

---

## Further Reading

- [AWS Spot Instance Advisor](https://aws.amazon.com/ec2/spot/instance-advisor/) — Real-time interruption rate data by instance type
- [AWS EC2 Savings Plans documentation](https://aws.amazon.com/savingsplans/) — Deep dive on commitment options and discount tiers
- [Azure Reservations documentation](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/) — Exchange policies and hybrid benefit details
- [GCP Committed Use Discounts](https://cloud.google.com/compute/docs/instances/committed-use-discounts-overview) — How resource-based and spend-based CUDs work
- [ThunderCompute: Cloud GPU Spot Availability](https://www.thundercompute.com/) — Independent tracker of spot capacity across providers
- [Cast.ai: Cloud Pricing Comparison 2025](https://cast.ai/) — Cross-cloud cost analysis with GPU-specific breakdowns
- [The Register: AWS raises GPU prices 15%](https://www.theregister.com/) — Coverage of the January 2026 Capacity Block price hike
