# Topic A — Complete Consolidated Reference

*Everything on this topic, pulled from the uploaded research notes doc + all analysis done across conversations, in one place. This file is the source of truth going forward — no need to re-derive from chat history.*

**Last updated:** Aug 5, 2026 (Day 2 consolidation pass)

---

## 1. Main Working Topic (never changed)
> **No-Regret Methodologies in Multi-Agent Systems**

---

## 2. All Candidate Titles (in order they were generated)

| # | Title | Stage | Outcome it assumes |
|---|---|---|---|
| 1 | *"Do Reasoning Models Learn to Play No-Regret? Test-Time Deliberation in Repeated Games and Online Learning"* | Original starting title (pre-experiment, doc Section 1) | Neutral — just states the question |
| 2 | *"Thinking Their Way to No-Regret: Test-Time Deliberation Closes the Online-Learning Gap in Language Model Agents"* | Post-execution candidate (doc Section 6) | **If regret cleanly declines** across model generations — flagged in the doc as the **weakest, least interesting** outcome (unsurprising, weak citation value) |
| 3 | *"Some Rounds of Thinking: Partial and Game-Dependent Gains from Test-Time Deliberation in Repeated Play"* (reconstructed placeholder — original doc line was truncated; **replace once real results define the actual middle pattern**) | Post-execution candidate (doc Section 6, originally truncated) | Mixed result — regret improves on some games/generations but not others |
| 4 | **"Deep but Not No-Regret: Dissociating Reasoning Depth from Online-Learning Optimality in LLM Agents"** | Post-execution candidate (doc Section 6) — **currently recommended** | **If reasoning depth (τ) and regret diverge** — the doc's own explicit recommendation, and the most citable/non-obvious finding |

**Doc's own guidance (Section 6, verbatim intent):** aim for outcome #3 or #4 rather than #2's clean-improvement story — a "models keep getting better at everything" story is unsurprising; a divergence/failure-mode story gets cited, and is also more consistent with what existing literature (mixed CoT effects across GTBench and the Behavioral GT paper) already suggests is likely.

**Current status:** Title #4 is the working title, to stay flexible until real results come in. **Do not lock the final title until Stage 4 (Merge & Compare) produces the actual dissociation pattern** — pick whichever of #2/#3/#4 the data actually matches.

---

## 3. Core Research Question
Do reasoning models (o1, o3-mini, DeepSeek-R1, current-gen open models) show measurably different **regret** behavior than non-reasoning models across model *generations* — and, critically, does a model's **reasoning depth (τ)** track its **regret performance**, or can the two diverge (a model reasons "deeply" by one measure but still plays poorly by the other)?

**Axis of study:** Longitudinal — GPT-4 → o1 → o3-mini → DeepSeek-R1 → current-generation open models.

---

## 4. The Literature Gap (verified across two research passes)

**Confirmed still open, as of Aug 5, 2026 deep search:**
- No paper computes **both** regret-vs-FTRL/FTPL (Park et al.'s metric) **and** reasoning-depth τ (Behavioral GT paper's TQRE metric) **on the same models, on the same games.**
- No paper tracks a **longitudinal regret trend** across reasoning-model generations — everyone benchmarks a snapshot, not a trend line.

**Three papers that define the gap (each covers only part of the picture):**

| Paper | What it has | What it's missing |
|---|---|---|
| Park et al., "Do LLM Agents Have Regret?" (ICLR 2025) | Formal regret-vs-FTRL/FTPL framework, 6 repeated games + 4 non-stationary online-learning problems | Only non-reasoning models (GPT-3.5, GPT-4, GPT-4o) |
| GTBench (NeurIPS 2024 D&B) | "Regret" terminology, but per Appendix A10 it's just hardcoded if/else scoring, not a real FTRL/FTPL comparison | Also non-reasoning models only |
| Behavioral GT paper (NeurIPS 2025) | Tests reasoning models (o1, o3-mini, R1) on 22 models × 13 games, using τ/TQRE | Different metric entirely (reasoning depth, not regret); one-shot games only, not repeated/sequential |

**Adjacent "watch out" papers** (verified via deep search, must be cited/differentiated but don't close the gap):
- *"Reasoning without Regret"* (Chitra, Apr 2025) — near-identical title, unrelated content (single-agent training-theory "regret," not multi-agent game regret). Must be disambiguated explicitly.
- *"Regret Minimization with Adaptive Opponents in Repeated Games"* (Ozdaglar/Kaiqing Zhang group — **COLT 2026**, June 2026) — same authors as Park et al., still actively extending regret theory (new "RP-Regret" metric). Purely theoretical, no LLM testing, but signals this group could pivot to LLMs at any time — a real (not hypothetical) scoop risk.
- Kang (ICLR 2026 MALGAI workshop) — Nash-convergence lens, not regret; structurally adjacent, confirms this genre clears real workshop review.
- **"When Agents Lie"** (Best Paper, ICML NExT-Game 2026) — checked and confirmed this does **not** overlap Topic A; its lens is deception/commitment-breaking, completely different from reasoning-depth/regret dissociation.

---

## 5. Scope Decisions (locked)

- **Model roster: open-source/open-weight only** — deliberate design choice, not a limitation. Reasons: full reproducibility (fixed weights, unlike closed APIs that change silently), full CoT trace inspectability (needed for mechanism analysis — closed reasoning models redact their traces), no rate-limit/cost ceiling, and it removes the "model silently updated" confound for a longitudinal claim.
- **Not "core AI/ML"** — honest categorization is empirical/benchmark contribution in AI agents + game theory. Frame it that way in any write-up; overclaiming reads as inexperience to reviewers.
- **Not theoretical** — purely experimental: run existing models through two established, already-published metrics and report where they agree/disagree.

---

## 6. Model Roster (full version, from the doc — trim to 4 for the initial pipeline build)

| Tier | Models |
|---|---|
| Non-reasoning baseline | Llama-3-70B-Instruct, Mistral-7B/Mixtral, CodeLlama-34B |
| Early open reasoning | DeepSeek-R1, Qwen3 (thinking mode), Kimi K2, GLM-4.6 |
| Current-gen open reasoning | DeepSeek-V4, GLM-5.2, Kimi K3, Qwen 3.6, MiniMax M3, gpt-oss-120b/20b, Gemma 4, Nemotron 3 Ultra |

*(Confirm exact current version numbers right before running — this list drifts fast.)*

**Simplified starter set (build/validate pipeline on these 4 first):** Llama-3-70B-Instruct, DeepSeek-R1, Qwen 3.6, GLM-5.2.

---

## 7. Game Suite (three layers)

| Layer | Purpose | Games |
|---|---|---|
| **A — Replication baseline** | Direct comparability, lets you cite Park et al.'s published numbers | Their 6 repeated games (win-win, prisoner's dilemma, unfair, cyclic, biased, second-best) + 4 non-stationary online-learning problems (Uniform/Gaussian/Linear-trend/Sine-trend) |
| **B — Original stress tests (your novel design contribution)** | Break models that rely on simple history-averaging rather than real tracking | Non-stationary opponent that flips strategy mid-game; delayed-credit game (payoff feedback arrives several rounds later); ambiguous/noisy-payoff game |
| **C — Mechanism probes** | Explain *why* dissociation happens, not just that it does | Manual CoT-trace coding for opponent-history reasoning, counterfactual reasoning, deliberate randomization — only possible because open-weight models expose full traces |

**Simplified starter set:** 1 repeated game (Prisoner's Dilemma) + 1 non-stationary problem (sine-trend) + 1 custom adversarial game, before expanding to the full suite.

---

## 8. Metrics — Quick-Reference Definitions *(new)*

So these don't need to be re-explained every session:

- **Regret** = how much worse a model's actual cumulative payoff was compared to the best fixed action in hindsight. Computed against two textbook baselines:
  - **FTRL** (Follow-The-Regularized-Leader) — an entropy-regularized online-learning algorithm, mathematically guaranteed to be low-regret.
  - **FTPL** (Follow-The-Perturbed-Leader) — a Gaussian-noise-perturbation variant, also guaranteed low-regret.
  - A model "beats" or "matches" these baselines → it's behaving like a competent online learner. This is Park et al.'s exact formal definition — reuse it as-is, don't reinvent it.
- **Reasoning depth (τ)** = a single number describing how many levels of strategic "I think that you think..." a model is actually doing, fit via **TQRE** (Truncated Quantal Response Equilibrium — combines Quantal Response Equilibrium's probabilistic-choice modeling with Cognitive Hierarchy's bounded-depth-of-reasoning modeling). Level-0 = non-strategic/random, Level-1 = best-responds to a Level-0 opponent, Level-2 = best-responds to Level-1, etc. This is the Behavioral GT paper's exact method — reuse it as-is.
- **The dissociation claim** = the core thing this whole project measures: does a model's τ rank match its regret rank, or can a model be "deep" (high τ) but still not low-regret (bad FTRL/FTPL comparison), or vice versa?

---

## 9. The Pipeline (5 stages, in build order)

1. **Game Engine** — runs a model through a game round-by-round, logs actions/payoffs/full reasoning trace.
2. **Regret Calculator** — reads the raw log, computes cumulative regret vs. best-fixed-action, FTRL baseline, FTPL baseline.
3. **τ Calculator** — reads the same log, fits action distribution to TQRE, outputs best-fit τ.
4. **Merge & Compare** — joins regret + τ tables, flags models where the two ranks disagree sharply (your dissociation candidates), produces a scatter plot.
5. **Statistical Test** — Wilcoxon signed-rank on the rank gap, one p-value per game.

**Build order:** 1 model × 1 game end-to-end first → validate regret calc against Park et al.'s published numbers → validate τ calc against Behavioral GT paper's published values → scale to 4 models × 1 game → add game 2 → add game 3 (pilot on 2 models first) → run merge/stats → only then expand to full roster/game suite.

---

## 10. Your Standard Build Spec (applied)
- Checkpoint every 20 rounds per (model, game) run, resume-by-default.
- 60-worker concurrency across different (model, game) pairs (not within a single game — rounds are sequential).
- Rolling ETA display in terminal.
- `--dry-run` flag (test on 2 rounds before full runs).
- Output raw logs → `/mnt/user-data/outputs/raw_logs/`, final tables → `/mnt/user-data/outputs/results/`.

---

## 11. Target Venues

| Venue | Deadline | Notes |
|---|---|---|
| NeurIPS 2026 workshop | Rolling, through Aug–Sept 2026 (e.g. Aug 29 example deadline) | Fast dry-run/proof-of-concept option, smaller model subset |
| **AAMAS 2027 main track** | "TBC," historically Oct 8–28 | **Primary target** — real, respected venue; ~2–3 months runway from today |

**Honest tier assessment:** this is workshop/AAMAS-tier, not ICML/NeurIPS/ICLR main-track tier. A legitimate, citable empirical contribution — not a "top-3 venue" claim.

---

## 12. Confidence Summary (from direct assessment)
- **"Genuinely open" as of today's search:** fairly confident — verified no paper does this exact dual-metric comparison. Not certain — search isn't exhaustive, and the active COLT 2026 author group is a real (if not immediate) scoop risk.
- **"Top conferences":** don't oversell this — realistic ceiling is AAMAS main track / strong NeurIPS workshop, not ICML/NeurIPS/ICLR main track.
- **Re-verify before submission:** re-run the literature search once more a few weeks before the actual AAMAS deadline — this space moves on a weeks-to-months cycle, and "open today" isn't a permanent guarantee.

---

## 13. Side-by-Side vs. Topic B (why A was chosen over B)

| | Topic A (this one) | Topic B — "Cost of Mixing Models" |
|---|---|---|
| Core question | Does reasoning depth track regret across model generations? | Can a stronger LLM exploit a weaker one in a mixed population? |
| Nearest competitor found | None doing the exact dual-metric test | "When Agents Lie" (Best Paper, ICML NExT-Game 2026) — has a named "Heterogeneous Exploitation" finding uncomfortably close to B's core claim |
| Novelty risk | Low | Medium-High (elevated after full-text read of "When Agents Lie") |
| Status | **Chosen, being built** | Not dead, but shelved due to elevated overlap risk |

**Topic B is not deleted, just backed up** — see the filename. If Topic A's dissociation pattern turns out weak/uninteresting once real data comes in, Topic B remains a fallback, with the explicit caveat that its overlap with "When Agents Lie" needs a full-text re-read (not just the abstract) before recommitting to it.

---

## 14. Changelog *(new)*
- **Day 1 (uploaded doc):** Original research notes — literature verification, GT MSCS admissions assessment, initial title candidates.
- **Day 1 (separate chat, Aug 1):** Topic B locked ("Cost of Mixing Models") after independent comparison — this was not reconciled with the Day 1 doc at the time.
- **Day 2 (this consolidation):** Full comparative deep-research pass across both topics using current literature; Topic A chosen over Topic B due to lower novelty-collision risk (specifically, "When Agents Lie" winning Best Paper at ICML NExT-Game 2026 raised Topic B's risk materially). Pipeline, model roster, game suite, and build spec locked for Topic A. This file is the resulting source of truth.

---

## 15. Next Action
Start building **Stage 1 — Game Engine**, single model (Llama-3-70B-Instruct or DeepSeek-R1) × single game (Prisoner's Dilemma), per the build order in Section 9. Everything downstream (regret calc, τ calc, merge, stats) depends on this working cleanly first.
