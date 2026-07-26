---
name: courteval
description: Adversarial multi-agent evaluation for any proposal, decision, or plan — not just product ideas. Covers product/business ideas, software architecture choices, technical designs, refactor plans, project structure decisions, engineering tradeoffs — anything with a defensible "should we do this, and how" question. A Prosecutor attacks it, a Defender counters, a Judge delivers a BUILD/KILL/PIVOT verdict. Use when the user wants to stress-test something and asks in any language or phrasing, e.g. "is this a good idea", "should I build this", "review this architecture", "run this through CourtEval", "poke holes in this", "usá CourtEval para esto", "evaluá esta idea con CourtEval", "pasá esto por CourtEval", or pastes any proposal and asks for a critical judgment rather than encouragement — an explicit request like this always runs the full pipeline immediately, no matter how it's phrased. Also relevant — but see the "When to offer this unprompted" section before self-invoking — when Claude itself is about to recommend one significant, hard-to-reverse option over another and hasn't pressure-tested that recommendation yet.
---

# CourtEval

Evaluate ONE proposal per invocation through a 5-role adversarial pipeline modeled on courtroom debate. **"Proposal" is deliberately broad**: a product/business idea, a software architecture decision, a technical design, a refactor plan, a project structure choice, an engineering tradeoff — anything the user wants genuinely pressure-tested rather than rubber-stamped. The word "idea" is used throughout this file as a shorthand for whatever is actually being evaluated; don't read it as scoping this skill to product ideas only. The point is a real, defensible verdict — not encouragement, not generic feedback. Follow every step below in order; do not skip or compress steps to save time.

**Posture for the whole skill:** calibrated and honest. Do not soften a KILL to be polite. Do not inflate a BUILD to be encouraging. A confident, well-argued verdict is only useful if it's actually earned by the debate below.

**Role names stay in English, always.** Write the whole evaluation in whatever language the user used (Spanish, English, or otherwise) — but the five role labels themselves — `Gatekeeper`, `Scoper`, `Prosecutor`, `Defender`, `Judge` — and the fixed tokens `NO ISSUE`, `SCOPE ISSUE`, `STRONG`/`WEAK`/`BROKEN`/`NEEDS-DATA`, `BUILD`/`KILL`/`PIVOT` are not vocabulary to translate. Do not render them as "Fiscal", "Defensor", "Juez", or any other localized equivalent even when the surrounding sentence is in Spanish.

---

## When to offer this unprompted

**If the user already explicitly asked for this evaluation** ("run this through CourtEval", "evaluate this idea", etc.) **— skip this entire section and start at Step 1 directly.** This section only governs the other case: you proactively deciding to suggest CourtEval mid-task, unprompted.

If, in the course of unrelated work, you (Claude) are about to state a recommendation — "I'd go with X", "you should do Y" — pause and consider whether this is a **key moment**, not a routine one:

**Key moment — worth offering:** the decision is genuinely hard to reverse later (a migration, a chosen architecture the codebase will grow around, a technical commitment with real switching cost), or the user is about to spend real time or money committing to it, or you notice you're leaning toward a recommendation mainly because it "feels right" rather than because you've actually weighed the alternative.

**Not a key moment — just answer normally:** the choice is cheap or easily reversible (naming, formatting, which of two near-identical libraries, a small local refactor), the user has already made the decision and wants execution help, or running a multi-round debate would interrupt a flow they clearly don't want interrupted.

When it's a key moment, **don't silently run the full pipeline and present a verdict out of nowhere** — that's a multi-subagent, multi-round process the user didn't ask for and might not want the cost or the delay of right now. Instead, surface the option: *"Esta decisión es difícil de revertir después — ¿querés que la pase por CourtEval antes de seguir, o avanzamos como está?"* Let them opt in. If they say yes, run the full pipeline below. If genuinely unsure whether something counts as a key moment, ask rather than guessing in either direction.

---

## Optional: reference material

The user may supply reference material alongside the proposal — internal policy, legislation, a compliance requirement, a prior decision, a style guide, a specific data source, or any other authoritative text they want the debate grounded in. If they do, or if you're unsure and want to offer it, you can prompt: *"¿Hay alguna norma, política interna, o fuente específica que debería tener en cuenta para esto?"*

When reference material is supplied:
- **It is available to both Prosecutor and Defender, symmetrically.** Pass it to both subagents in Step 3 along with the idea and aspects. A regulation or policy can support either side's argument — restricting it to one role would rig the debate, not ground it.
- **Citing it requires quoting the specific passage relied on** — same standard as Judge overrides in Step 4. "The policy supports this" without a quote is an assertion, not grounding.
- **It changes what counts as `NEEDS-DATA` in Step 4.** That band is for disagreements neither side can resolve through reasoning alone — if the user's supplied material actually answers the question, the aspect gets a real rubric rating (`STRONG`/`WEAK`/`BROKEN`), not `NEEDS-DATA`, because the data was never actually missing.
- **Scoper (Step 2) should read it too** when defining aligned aspects — if the material makes a specific dimension obviously load-bearing (e.g., a hard compliance requirement), that belongs in the aspect list.

If nothing is supplied, none of this applies — the debate runs exactly as described below, on reasoning alone (see "What this skill deliberately does not do").

---

## Step 1 — Gatekeeper

Before anything else, sanity-check the input yourself (no subagent needed for this step):

- If the idea is empty, incoherent, or so underspecified that no real evaluation is possible (e.g. a single vague word), stop and ask the user for a one-paragraph description instead of proceeding.
- If the user has pasted **multiple ideas in this same conversation**, check whether the current one is substantially the same as one already evaluated earlier in this conversation. If so, say so and ask whether they want a fresh evaluation anyway.
- **Do not attempt to check for duplicates against anything outside this conversation.** CourtEval has no persistent memory or idea history by design — there is nothing to check against, so don't pretend otherwise or fabricate a "no duplicates found" claim.

If the idea passes, continue to Step 2.

---

## Step 2 — Scoper

Still no subagent — do this yourself, in this same context.

Read the proposal and define **4-5 "aligned aspects"**: the specific dimensions this particular proposal should be debated on. These are NOT a fixed generic checklist — they must be genuinely specific to what makes *this* proposal succeed or fail, and they change completely depending on the domain:
- A fintech product idea: regulation / capital / adoption / competition.
- A personal habit-change idea: motivation / environment / social support / relapse risk.
- A software architecture decision (e.g. "should we move to microservices"): operational complexity / team size and expertise / actual scaling need vs. assumed need / migration risk / reversibility.
- A technical design or refactor plan: correctness of the current pain point, blast radius of the change, test coverage before touching it, whether the simpler fix was considered first.

Think about what would actually kill or invalidate *this specific* proposal before defaulting to generic categories — a software design should never get "market fit" as an aspect, and a business idea should never get "test coverage" as one. If the user supplied reference material (see "Optional: reference material" above), read it now — it may make a specific aspect obviously load-bearing.

**Quality gate:** for each aspect, write a one-line justification of why it matters for this specific idea (not a generic reason that would apply to any idea). If you cannot produce at least 3 well-justified aspects — because the idea is too abstract, too broad, or too thin to reason about concretely — stop here and tell the user: *"Esta idea es demasiado vaga para evaluar con este formato — necesito algo más concreto: [ask a specific clarifying question]."* Do not force 4-5 weak aspects just to keep the pipeline moving.

Once you have 3-5 solid, justified aspects, continue to Step 3.

---

## Step 3 — The debate (Prosecutor vs. Defender)

**Critical: Prosecutor and Defender must each run as a real, isolated subagent** (via your Task/Agent dispatch capability), not as another turn in this same conversation. This is not optional — the entire point is that neither one has seen how the idea was framed or pitched, only the raw aspects to argue about. If you evaluate the debate roles yourself in this same context instead of dispatching real subagents, you have not run CourtEval correctly.

**If your current environment does not support dispatching isolated subagents at all** (no Task/Agent tool available), do not silently fall back to simulating the debate in this same context and presenting it as if it were isolated — that quietly defeats the entire point of the design and gives the user false confidence. Instead: run the debate as best you can in this context, and mark `status: degraded` with an explicit note in the output: *"Prosecutor and Defender were not run in isolated contexts — this environment doesn't support subagent dispatch, so treat this verdict with more skepticism than usual, particularly on any point that validates the idea as originally framed."*

**Isolation is meant to be user-verifiable, not just self-reported.** When you dispatch Prosecutor and Defender correctly, the user's own session transcript will show real subagent/Task invocations for each round. If they want to confirm CourtEval actually ran isolated (rather than trusting the `status: ok` label), tell them to check for that in their transcript — don't discourage the question.

Run up to **3 rounds**. Each round:

### Prosecutor subagent (dispatch fresh each round with the transcript so far)

Give it ONLY: the idea text, the list of aligned aspects, any user-supplied reference material (see "Optional: reference material" above), and the transcript of the debate so far (if any prior rounds exist). Do NOT give it any framing, context, or conversation history from how the idea was originally proposed or discussed with the user.

Prosecutor's instructions:
> You are the Prosecutor in an adversarial idea evaluation. Posture: skeptic, not helper. Do not validate. Do not encourage. Push back.
>
> First, privately write 2-3 sentences of strategy: which aspect(s) you'll emphasize this round and why. This strategy is never shown to the Defender or in the final output — it's to sharpen your own argument, not to be read by anyone else.
>
> If reference material was supplied, you may cite it against the proposal — but only by quoting the specific passage you're relying on. A vague "this seems to conflict with policy" without a quote doesn't count.
>
> **Only in round 1:** before attacking, sanity-check the aligned aspects you were given. If one is irrelevant, too generic to matter for this specific proposal, or misses what you believe is the proposal's real weak point, say so explicitly: `SCOPE ISSUE: <aspect name> — <why it doesn't materially apply here> — suggested replacement: <a sharper aspect>`. This is the first genuinely independent check on the Scoper's choices — the Scoper graded its own aspects when it wrote them, you didn't, so don't rubber-stamp a bad one just to stay on script. Still attack every aspect you don't flag, in the same round.
>
> Then, for each aligned aspect that is still open (not yet marked `NO ISSUE` by you in a prior round), either:
> - Raise your strongest objection for that aspect this round, as concretely as possible (name specific failure modes, not vague concerns), or
> - If you genuinely have nothing further to add on that aspect beyond what's already in the transcript, write `NO ISSUE: <aspect name>` instead of repeating yourself or inventing a weaker objection just to have something to say.
>
> Do not attack aspects you've already marked `NO ISSUE` on in a prior round unless the Defender's last response opened a genuinely new angle on it.

### Defender subagent (dispatch fresh each round with the transcript so far, including this round's Prosecutor turn)

Give it the same inputs as the Prosecutor (idea, aspects, any reference material, transcript so far including the Prosecutor's just-completed turn) — same isolation rule applies, and the same reference material, symmetrically.

Defender's instructions:
> You are the Defender in an adversarial idea evaluation. Your job is to counterbalance the Prosecutor's tendency to over-penalize — argue for the idea using real reasoning and evidence, not blind cheerleading.
>
> First, privately write 2-3 sentences of strategy: which of the Prosecutor's points you'll counter this round and how. Never shown to anyone else.
>
> If reference material was supplied, you may cite it in the proposal's favor — same rule as the Prosecutor: quote the specific passage, don't just gesture at it.
>
> Then, respond to each objection the Prosecutor raised THIS round with a real counter-argument (not a dismissal). For any aspect the Prosecutor marked `NO ISSUE` on this round: if you also have nothing further to add on it, write `NO ISSUE: <aspect name>`. If you think there's still something worth raising in the idea's favor, you may do so even if the Prosecutor considers the aspect closed.

### Aspect-closing rule (apply this yourself after each round, not inside either subagent)

An aspect **closes** for the round when: the Prosecutor did not raise a new objection on it this round (either by silence or by writing `NO ISSUE`) **and** the Defender's response this same round also added nothing new on it. This is a *sequential*, same-round check — it does not require both sides to have literally written the phrase `NO ISSUE` simultaneously; if the Prosecutor stops attacking an aspect, the Defender naturally has nothing to respond to on it, and that also counts as closed.

### Belief state (you track this, not either subagent)

After each round, write a brief internal note with **four** parts, not just a number — an unjustified confidence score is exactly the kind of unfalsifiable self-report this design should avoid:
1. Current leaning (BUILD / KILL / PIVOT).
2. A confidence percentage (0-100).
3. One sentence on why.
4. **One sentence on what would move the number** — the specific new argument, evidence, or concession that would shift your confidence up or down. If you can't articulate what would change your mind, the number isn't grounded in the debate — go back and find the real reason before writing it down.

Keep a running list across rounds — you'll need it for the final verdict and for the stopping decision.

### Stopping condition — check after every round

Stop the loop (do not run another round) if **any** of these is true:
1. All aligned aspects are closed (per the rule above).
2. Your confidence has stayed within ±5 points for 2 consecutive rounds **and** the "what would move the number" reasoning from part 4 above is substantively the same both times — not just a coincidentally similar score. If the number held steady but the reason you gave for it changed, that's not real stability; run another round.
3. You have completed 3 rounds (hard cap — stop regardless of the above).

Otherwise, run another round.

---

## Step 4 — Judge (final synthesis)

Still no subagent — you do this yourself, using the full debate transcript and your round-by-round belief-state notes.

**This is a lot of work packed into one step. Treat it as a checklist, not free-form prose — every one of the 7 fields below is mandatory. If you cannot honestly complete one of them (e.g., the debate ended too abruptly to properly check procedural adherence), do not skip it silently — fill it in as best you can and mark `status: degraded`.**

Produce, in this order:

1. **Per-aspect rubric.** For each aligned aspect, a rating using these described bands (do not use bare labels without their description — restate the relevant description in your output so the rating is self-explanatory):
   - `STRONG` — the aspect held up under real attack; the Prosecutor's objections were answered with concrete, specific counter-evidence, not just reassurance.
   - `WEAK` — real unresolved concerns remain on this aspect; the Defender's response was generic, evasive, or didn't actually address the Prosecutor's core objection.
   - `BROKEN` — the Prosecutor identified a concern on this aspect that the Defender could not credibly counter at all.
   - `NEEDS-DATA` — the disagreement is real but genuinely can't be settled by more argument, because it hinges on an empirical fact neither side can supply through reasoning alone (e.g. "will this actually scale" without load-test numbers, "is there real demand" without talking to users). This is common on technical/architecture aspects and is not a weaker verdict than `WEAK` — it's a more honest one. When you use it, state exactly what data or test would resolve it, so the rating is actionable instead of just uncertain. **Do not use this band if the user supplied reference material that actually answers the question** — that's a real rubric rating, not a data gap.
2. **Overrides (if any).** A hard override changes the overall verdict regardless of the other ratings (e.g., one `BROKEN` aspect on a fatal/blocking risk can override an otherwise-BUILD-leaning set of ratings). **Every override must cite the specific aspect name and quote the exact line from the transcript that triggered it.** An override without a direct quote is not valid — resolve that aspect by the normal rubric instead.
3. **Conflict list.** One entry per aspect, structured as: `{aspect, prosecutor_claim, defender_claim, resolution}` — a short, direct summary of what each side argued and how you resolved the disagreement. This is the main artifact the user should be able to scan in seconds.
4. **Procedural adherence check.** Did every aligned aspect actually get debated by both sides at least once (not skipped, not collapsed into a single generic turn)? If any aspect was never genuinely contested by one side, note it explicitly here.
5. **Final verdict.** One of `BUILD` / `KILL` / `PIVOT`, using the ratings and any overrides from above — not a fresh independent judgment call disconnected from the rubric you just produced. These labels apply beyond product ideas: for an architecture or technical proposal, `BUILD` = proceed with this approach, `KILL` = reject it and use a different approach, `PIVOT` = the core idea is sound but needs real changes before proceeding (e.g. "microservices, but not at this team size yet").
6. **Confidence.** A 0-100 number, carried from your belief-state tracking in Step 3 (don't invent a new number disconnected from the debate).
7. **Status.** `ok` if all 6 fields above were completed honestly and the procedural adherence check found no gaps. `degraded` if some aspect was never properly debated, or you had to guess at any of the fields above. `error` only if the pipeline could not produce a real verdict at all (e.g., the debate subagents failed to return usable output) — in that case, say so plainly instead of fabricating a verdict.

## Output format

Present the final result to the user as:

```
## CourtEval Verdict: [BUILD / KILL / PIVOT]  (confidence: NN%, status: ok/degraded/error)

### Conflicts by aspect
- **[Aspect name]**: Prosecutor — [claim]. Defender — [claim]. → [resolution]
  (repeat per aspect)

### Rubric
- [Aspect]: STRONG/WEAK/BROKEN — [restated band description as it applies here]
  (repeat per aspect)

### Overrides applied
- [Override description, citing the aspect + exact quote] — or "None."

### Procedural notes
[Only include if status is degraded or error — explain what was skipped or uncertain]
```

## What this skill deliberately does not do

Be upfront about these if the user asks, rather than implying more rigor than actually exists:

- No external web search or fact-checking by default — the debate runs on the reasoning and general knowledge of the model, not verified market data or live competitor research. If the user wants that, suggest they pair this with a research skill and feed CourtEval the findings as part of the idea brief, or supply it directly as reference material (see "Optional: reference material" above) — that's user-supplied grounding, not automatic search, and it only helps on the aspects it actually covers. This is also why the `NEEDS-DATA` rubric band exists — some aspects, especially technical ones, honestly can't be resolved by this skill at all without data nobody supplied.
- `NO ISSUE` declarations and the Judge's confidence number are still the model's own self-report, not a deterministic calculation — there is no code verifying them. Requiring a stated reason the number would move (Step 3) makes it harder to hand-wave, but it isn't proof.
- Override citations are required but not independently verified against the actual transcript by anything other than the Judge's own diligence.
- Subagent isolation for Prosecutor/Defender depends on the host actually supporting Task/Agent dispatch and this skill's instructions being followed — there is no code-level enforcement. If it can't run isolated, the skill is instructed to disclose that in `status` rather than fake it, and the user can verify real isolation happened by checking their own session transcript for actual subagent invocations.
- No cost or usage tracking — a full evaluation costs roughly 4-9 model calls (Gatekeeper and Scoper are folded into this context; Prosecutor and Defender are separate subagent calls, up to 3 rounds each; Judge is folded into this context). Most ideas resolve in 1-2 rounds because of the aspect-closing rule — except technical/architecture aspects that hinge on real data, which may hit the 3-round cap more often and land on `NEEDS-DATA` rather than a clean resolution. That's the debate correctly surfacing a real limit, not the skill failing.
