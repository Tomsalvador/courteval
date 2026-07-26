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

**Speed matters — this has to run fast enough that people actually use it.** Real subagent dispatch has real overhead; a sequential turn-by-turn debate can take several minutes, which is long enough that nobody will bother. Four things keep it fast without cutting the mechanism itself:

1. **Dispatch Prosecutor and Defender in parallel, not sequentially.** Issue both Task/Agent calls in the same turn so they run concurrently — do not wait for Prosecutor to finish before starting Defender. This roughly halves wall-clock time per round, and if anything makes isolation *stronger*: neither sees the other's output for that round at all, only the transcript through the prior round.
2. **Use a faster/lighter model for Prosecutor and Defender specifically**, if your environment lets you choose a model per subagent dispatch — reserve the strongest available model for the Judge's synthesis in Step 4, where the reasoning actually needs to be deep. The debate roles need to be sharp and concrete, not exhaustive.
3. **Keep each turn short: roughly 100-150 words per still-open aspect.** This is a fast pressure-test, not a legal brief — concrete and concise beats thorough and slow.
4. **Default round cap is 2, not 3.** Round 3 rarely changes the verdict and doubles the worst-case wait for marginal benefit. If the user explicitly asks for deeper scrutiny ("modo profundo", "sé más exhaustivo", "dale otra vuelta"), or the Judge's confidence is still low and volatile after round 2, extend to a third round — but that's opt-in, not the default.

### Prosecutor and Defender, dispatched together each round

Each round, issue **both dispatches in the same turn** so they run in parallel. Give each ONLY: the idea text, the aligned aspects, any user-supplied reference material (see "Optional: reference material" above), and the transcript **through the end of the previous round** — never this round's output from the other role, since it doesn't exist yet when both are dispatched. Do NOT give either any framing, context, or conversation history from how the idea was originally proposed or discussed with the user.

Prosecutor's instructions:
> You are the Prosecutor in an adversarial idea evaluation. Posture: skeptic, not helper. Do not validate. Do not encourage. Push back. Keep it concise: roughly 100-150 words per aspect you address, no padding.
>
> First, privately write 1-2 sentences of strategy: which aspect(s) you'll emphasize this round and why. Never shown to the Defender or in the final output.
>
> If reference material was supplied, you may cite it against the proposal — but only by quoting the specific passage you're relying on. A vague "this seems to conflict with policy" without a quote doesn't count.
>
> **Only in round 1:** before attacking, sanity-check the aligned aspects you were given. If one is irrelevant, too generic to matter for this specific proposal, or misses what you believe is the proposal's real weak point, say so explicitly: `SCOPE ISSUE: <aspect name> — <why it doesn't materially apply here> — suggested replacement: <a sharper aspect>`. This is the first genuinely independent check on the Scoper's choices — the Scoper graded its own aspects when it wrote them, you didn't, so don't rubber-stamp a bad one just to stay on script. Still attack every aspect you don't flag, in the same round.
>
> Then, for each aligned aspect that is still open (not yet marked `NO ISSUE` by you in a prior round), either:
> - Raise your strongest objection for that aspect this round, as concretely as possible (name specific failure modes, not vague concerns) — respond to the Defender's most recent argument on it if the transcript already has one, or
> - If you genuinely have nothing further to add on that aspect beyond what's already in the transcript, write `NO ISSUE: <aspect name>` instead of repeating yourself or inventing a weaker objection just to have something to say.
>
> Do not attack aspects you've already marked `NO ISSUE` on in a prior round unless the transcript shows the Defender opened a genuinely new angle on it since.

Defender's instructions:
> You are the Defender in an adversarial idea evaluation. Your job is to counterbalance the Prosecutor's tendency to over-penalize — argue for the idea using real reasoning and evidence, not blind cheerleading. Keep it concise: roughly 100-150 words per aspect you address, no padding.
>
> First, privately write 1-2 sentences of strategy: what you'll emphasize this round and why. Never shown to the Prosecutor or in the final output.
>
> If reference material was supplied, you may cite it in the proposal's favor — same rule as the Prosecutor: quote the specific passage, don't just gesture at it.
>
> You are being dispatched **at the same time** as the Prosecutor's round — you will not see this round's Prosecutor turn, only the transcript through the previous round. For round 1 (no prior transcript), defend each aspect proactively: state the strongest case for the proposal on that aspect, anticipating the obvious objection rather than reacting to one you haven't seen yet. From round 2 onward, respond directly to the Prosecutor's most recent (prior-round) objection on each still-open aspect with a real counter-argument. For any aspect where the transcript shows nothing further worth adding, write `NO ISSUE: <aspect name>`.

### Aspect-closing rule (apply this yourself after both dispatches return, not inside either subagent)

An aspect **closes** as of this round when: the Prosecutor's turn this round did not raise a new objection on it (silence or explicit `NO ISSUE`) **and** the Defender's turn this round also added nothing new on it. Because both were generated in parallel, this is a check you make after both come back — not something either subagent judges mid-round.

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
3. You have completed **2 rounds** (default cap — stop here unless the user asked for the deeper mode described above, in which case the cap is 3).

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

The 7 fields from Step 4 are how you *think*, not how you *talk*. Do not present them to the user as a literal template with headers, a per-aspect rubric list, and a labeled verdict line — that reads like a compliance report, not an answer. Write the final result as natural prose: the same register you'd use for a direct, un-debated answer to the same question, just backed by what the adversarial debate actually surfaced instead of generic reasoning.

Requirements for the prose:
- Lead with whatever is most decision-relevant — usually the strongest point in the proposal's favor and/or the most serious unresolved gap — not a march through the aspects in the order they were defined.
- Name the specific, concrete findings from the transcript (what actually got raised, what actually held up or fell apart, and say plainly when a round cap cut off a rebuttal before it happened) — but state them as your own direct analysis, in your own voice. **Never surface the machinery**: no "Prosecutor", "Defender", "Judge", "el que atacaba", "quien defendía", "el debate", "la otra parte concedió", round numbers, or any other language that reveals this ran as a multi-role process. The reader should not be able to tell this wasn't a single, well-reasoned answer — the adversarial process is how you got here, not something to narrate. Say "el punto más débil es X" or "esto no se sostiene porque Y", never "el Fiscal objetó X" or "el Defensor no pudo responder Y".
- Never show the literal band labels (`STRONG`/`WEAK`/`BROKEN`/`NEEDS-DATA`) or a conflicts/rubric list to the user — translate each into what it actually means for the proposal in plain language instead.
- Close with your actual take stated as an opinion, in a sentence: mention the BUILD/KILL/PIVOT lean and a calibrated confidence in words (e.g. "confianza media-alta", "poco convencido de esto tal como está") — not as a bracketed label, header, or bare percentage.
- If `status` is `degraded` or `error`, say so plainly within the prose (e.g. "un aspecto no llegó a debatirse del todo, así que tomá esto con más pinzas") — the conversational format doesn't excuse dropping this disclosure.
- Keep it as tight as the substance allows. This is a more carefully verified answer, not a longer one for its own sake — if the debate genuinely found nothing beyond what a plain answer would say, say that too, don't pad it out to look thorough.

## What this skill deliberately does not do

Be upfront about these if the user asks, rather than implying more rigor than actually exists:

- No external web search or fact-checking by default — the debate runs on the reasoning and general knowledge of the model, not verified market data or live competitor research. If the user wants that, suggest they pair this with a research skill and feed CourtEval the findings as part of the idea brief, or supply it directly as reference material (see "Optional: reference material" above) — that's user-supplied grounding, not automatic search, and it only helps on the aspects it actually covers. This is also why the `NEEDS-DATA` rubric band exists — some aspects, especially technical ones, honestly can't be resolved by this skill at all without data nobody supplied.
- `NO ISSUE` declarations and the Judge's confidence number are still the model's own self-report, not a deterministic calculation — there is no code verifying them. Requiring a stated reason the number would move (Step 3) makes it harder to hand-wave, but it isn't proof.
- Override citations are required but not independently verified against the actual transcript by anything other than the Judge's own diligence.
- Subagent isolation for Prosecutor/Defender depends on the host actually supporting Task/Agent dispatch and this skill's instructions being followed — there is no code-level enforcement. If it can't run isolated, the skill is instructed to disclose that in `status` rather than fake it, and the user can verify real isolation happened by checking their own session transcript for actual subagent invocations.
- No cost or usage tracking — a full evaluation costs roughly 4-7 model calls by default (Gatekeeper and Scoper are folded into this context; Prosecutor and Defender are separate parallel subagent calls, up to 2 rounds by default; Judge is folded into this context). Most proposals resolve in 1-2 rounds because of the aspect-closing rule — technical/architecture aspects that hinge on real data may still land on `NEEDS-DATA` after the default cap rather than a clean resolution, which is the debate correctly surfacing a real limit, not the skill failing. Parallel dispatch and a faster model on the debate roles (see Step 3) keep this to roughly a minute or two in practice rather than several minutes — ask for the deeper mode (up to 3 sequential-context rounds) only when you actually want to trade speed for extra scrutiny.
