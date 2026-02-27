# Contributing to the AI Infrastructure Decision Guide

Thanks for helping keep this guide honest and current. Here's how to contribute effectively.

---

## What We Want

### ✅ Great Contributions

- **Gotchas you discovered in production.** "When you do X, Y breaks because Z." The stuff that's not in vendor docs.
- **Pricing updates.** Verified, sourced, with a date. AI infra pricing changes monthly.
- **War stories / case studies.** Real companies, real numbers, real lessons. Anonymize if needed, but keep the specifics.
- **Corrections.** If something is wrong, fix it. Include a source.
- **Cross-references.** "See also: [anti-patterns/vendor-lock-in.md](anti-patterns/vendor-lock-in.md)" — help readers navigate.
- **New anti-patterns or architecture patterns.** If you've seen a common mistake or a proven pattern that's missing, write it up.

### ❌ What We Don't Want

- **Vendor marketing.** "AWS Bedrock is the best because it integrates seamlessly with..." — No. Tell us what actually works and what doesn't.
- **Unsourced claims.** "LLMs are 10x cheaper on provider X." Show the math or it doesn't get merged.
- **"It depends" answers.** Every section should give a decision framework with real thresholds. "Use X when your monthly spend exceeds $Y" beats "it depends on your use case."
- **Tutorials or how-tos.** This isn't a tutorial site. We explain *what* to choose and *why*, not *how* to set it up step by step.
- **Outdated information without dates.** Always include "as of [month] [year]" with pricing and benchmarks.

---

## Style Guide

### Voice & Tone
- **Opinionated.** We have opinions. We state them clearly. "Use X" not "you might consider X."
- **Engineer-first.** Write like you're explaining to a senior engineer over coffee, not writing a whitepaper.
- **Concise.** Every sentence should earn its place. Cut filler words.
- **Honest.** If something sucks, say it sucks. If something is great, say it's great. No hedging for vendor relationships.

### Formatting
- Every file starts with an H1 title and a narrative hook (not a dry overview)
- Include a `> **TL;DR**` block near the top
- Use tables for comparisons (with real numbers)
- Use Mermaid diagrams for architecture flows where it helps
- Cite sources naturally in prose ("as Vellum's 2025 benchmark showed") — no numbered footnotes
- End with a `## Further Reading` section with linked resources and brief descriptions
- Use `**bold**` for key terms on first use
- Use `code blocks` for technical terms, commands, and architecture diagrams

### Numbers Are Required
- Don't say "expensive" — say "$15,000/month at 500K requests"
- Don't say "faster" — say "2.3× throughput improvement"
- Don't say "most teams" — say "in our experience" or cite a source
- Always include the date pricing was verified

### File Structure
Every guide should follow this general structure:
1. **Title + TL;DR** — The whole page in 3 sentences
2. **Overview / Architecture** — What this is and how it works
3. **Pricing / Cost at Three Scales** — Small, medium, large
4. **The Gotchas** — What nobody tells you
5. **When to Use / When NOT to Use** — Clear decision criteria
6. **Sources** — Numbered, linked

---

## How to Contribute

### Small Fixes (typos, pricing updates, broken links)
1. Fork the repo
2. Make your change
3. Submit a PR with a brief description

### New Content (gotchas, case studies, new pages)
1. Open an issue first describing what you want to add
2. Get a thumbs up from a maintainer
3. Write it following the style guide above
4. Submit a PR

### Pricing Updates
- Include the source URL
- Include the date you verified the price
- If the pricing page is behind a login, screenshot it

---

## Code of Conduct

Be respectful. Disagree on facts, not on people. This is a technical resource, not a battleground.

We have one rule: **back it up with evidence.** Opinions are welcome. Unsupported opinions are not.
