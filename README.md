# CourtEval

Adversarial multi-agent evaluation for any proposal, as a Claude Code Skill — not just product ideas. Use it on a business idea, a software architecture decision, a technical design, a refactor plan, or any other "should we do this, and how" question. Instead of asking one model for its opinion, CourtEval runs a courtroom-inspired debate:

- **Prosecutor** attacks the idea, aspect by aspect — isolated from however the idea was originally pitched, so it can't just agree with the framing.
- **Defender** counters with real evidence, isolated the same way — exists specifically so the Prosecutor's skepticism doesn't just win by default.
- **Judge** referees the exchange, tracks confidence round over round, and closes with a rubric, a per-aspect conflict list, and a final **BUILD / KILL / PIVOT** verdict.

No fixed checklist — a **Scoper** step defines 4-5 aspects specific to *this* proposal before the debate starts, so a fintech idea and a microservices-vs-monolith decision get debated on genuinely different terms (regulation/capital/adoption for one, operational complexity/team size/reversibility for the other).

**Optional grounding:** if you have relevant source material — a compliance policy, legislation, an internal standard, a prior decision — hand it over with the proposal. Both Prosecutor and Defender get it, symmetrically, and have to quote the exact passage they're relying on to use it. Without it, the debate runs on reasoning alone.

## Install

```bash
git clone https://github.com/Tomsalvador/courteval.git
```

Then add it as a Claude Code plugin (via `.claude-plugin/marketplace.json` in this repo, or by pointing Claude Code at the cloned path per its plugin-loading docs).

## Use

In a Claude Code session with the plugin loaded, just describe what you want evaluated and ask for a verdict — e.g.:

> "Evaluate this idea with CourtEval: a subscription box that sends you a random book chosen by AI based on your mood that week."

> "Run this architecture decision through CourtEval: moving our monolith to microservices before we've hit any real scaling pain, mainly because it feels like the 'right' way to do it."

Or more directly: `run this through CourtEval: <your proposal>`.

CourtEval will:
1. Sanity-check the input (Gatekeeper)
2. Define the aspects worth debating for this specific idea (Scoper)
3. Run up to 3 rounds of real debate between isolated Prosecutor/Defender subagents, stopping early once nothing new is being said
4. Synthesize a verdict with a rubric, cited overrides, and a conflict list you can scan in seconds

## What it doesn't do

- No built-in web search or fact-checking — the debate runs on the model's own reasoning, not verified market data. Pair it with a research skill if you need that.
- No persistent memory across sessions — there's no idea history or duplicate-detection beyond the current conversation.
- No cost tracking — a typical evaluation costs roughly 4-9 model calls, usually resolving in 1-2 debate rounds.

See [`skills/courteval/SKILL.md`](skills/courteval/SKILL.md) for the full orchestration logic.

## License

MIT — see [LICENSE](LICENSE).
