# Topic A — Day 5 Session Log

*No new GPU run happened today (that's queued as Save Version v5) — today was a research and planning day. Everything below is what was found, decided, and changed as a result. Once v5 finishes running, its actual numbers get appended to this file as an update.*

**Session date:** Aug 10, 2026
**Status at end of day:** Deep literature check completed (real citations, no hallucination). Honest status assessment of the project done. Notebook bug found and fixed. Roster scaled from 2 models to 4. v5 Save Version queued but not yet run.

---

## 1. What We Set Out To Do Today

Two things, explicitly requested: (1) an honest, grounded reality-check — what is Topic A actually trying to prove, what has *actually* happened so far (no exaggeration), and is this even a real, provable thing, or has someone already shown it's not possible; (2) based on that, decide what to do next, then actually build it.

---

## 2. The Honest Status Check (before any new research)

Restated plainly: **does a model's reasoning depth (τ, via Cognitive Hierarchy/TQRE) predict its regret performance (vs. FTRL/FTPL), or can the two diverge?**

**What had actually happened, with no rounding up:**

| | Prisoner's Dilemma | Cyclic (2 models) |
|---|---|---|
| Regret | Both models beat FTRL/FTPL — valid data | Llama lost to FTRL (+5), R1 beat it (−4) — valid data |
| τ | **Unusable** — boundary collapse, since PD has a dominant strategy so every reasoning depth converges to the same action | **Usable for the first time** — genuine interior minimum, but **both models landed on the identical τ (4.36)** because both were unanimous on the same one-shot action |

**Honest conclusion at the start of today:** zero clean evidence yet of τ and regret actually diverging. The τ *instrument* finally works (Cyclic fixed PD's structural problem), but 2 models tied at the same τ tells us nothing about whether τ generally discriminates model capability — it's not enough data to draw any conclusion either way.

---

## 3. Deep Literature Check — Is This Even Possible? (real citations, verified by reading full papers)

**Confirmed still true:** no paper directly tests our exact pairing — Cognitive-Hierarchy/TQRE τ vs. formal FTRL/FTPL regret, same models, same repeated games. That gap remains open.

**But two adjacent papers, read in full, both bear directly on whether dissociation is even plausible:**

### Finding A — evidence *against* easy dissociation
**Zhang, Wang, Chen, Mansur & Sarhangian, "Comparing Exploration-Exploitation Strategies of LLMs and Humans"** (arXiv:2505.09901, updated May 2026, under review at *INFORMS Journal on Data Science*).

- Directly tested whether enabling "thinking" in LLMs (via CoT prompting or built-in thinking modes) reduces regret in multi-armed bandit tasks (stationary 2-armed and non-stationary 4-armed).
- **Result: thinking capabilities significantly reduce regret** — models with reasoning enabled shift toward more human-like, lower-regret exploration, in both settings.
- This is real, citable evidence for the "boring," non-dissociation outcome (title #2 from the original candidate list) — in an adjacent setting.
- **Key differences from our setup, stated plainly:** their "reasoning" indicator is a binary thinking on/off toggle or thinking-budget dial, not a graded Cognitive-Hierarchy τ. Their task is single-agent bandit exploration, not multi-agent strategic games against an opponent. So this is a real, relevant data point *against* dissociation, but not a direct scoop of our exact metric pairing.

### Finding B — evidence *for* dissociation being structurally plausible
**Po Han Teo, "LLM Agents as Static Level-k Players in Behavioural Games"** (arXiv:2606.27845, working draft, June 2026 — independent researcher, **not peer-reviewed**, lower-confidence source, flagged as such explicitly).

- Tested level-k reasoning depth on a p-beauty contest and public goods game, including multi-round horizons.
- **Central finding: LLMs behave as static level-k players — they do not run within-game belief-updating or backward induction across rounds**, even when human players visibly adapt/decay over repeated play. Also found LLMs ignore relative round position (no last-round defection), suggesting they retrieve a fixed reasoning level by category rather than reasoning iteratively from the specific game state.
- **Why this matters for us:** if reasoning depth is a fixed, non-adaptive trait, and regret is fundamentally about *adaptation over rounds*, then by this paper's own account the two are capturing genuinely different mechanisms — structural support for the dissociation hypothesis.
- **Caveats stated honestly:** single-author, unreviewed working paper; different games (p-beauty contest, public goods) than ours; no regret/FTRL metric computed at all — this is an indirect, not direct, precedent.

**Honest synthesis:** the two closest adjacent findings genuinely point in different directions — one (bandits) suggests reasoning helps regret; one (repeated games) suggests reasoning-depth and round-by-round adaptation are separate mechanisms. **We should not assume dissociation is the likely outcome going in** — Zhang et al.'s finding is real, adjacent, peer-review-track evidence that a null/boring result (they correlate) is a live possibility, not just a fallback.

---

## 4. Decision Made Today: Prepare For Either Outcome, Explicitly

Given Finding A vs. Finding B disagree, we locked in a framing plan now rather than being caught out later:

- **If regret and τ rank together on our data** → the paper becomes an extension of Zhang et al.'s finding into strategic multi-agent games (not just bandits), using a formal metric (FTRL/FTPL regret) instead of their model-free regret. Still a real, citable contribution.
- **If they diverge** → the paper becomes the dissociation finding, with Teo's static-level-k paper as the supporting mechanistic citation for *why* it makes sense.

Either way there's a real paper — deciding this now means the project doesn't stall if the "boring" result shows up.

---

## 5. What This Means Practically — The Actual Next Step

Since 2 models tied on τ is not enough data to know whether τ discriminates at all, **the single highest-value next action is scaling Cyclic from 2 models to 4** (Qwen3-8B, GLM-4-9B-Chat) before drawing any conclusion. Everything downstream — the rank comparison (Stage 4), the statistical test (Stage 5) — is blocked on this.

---

## 6. Notebook Work Done Today

### Bug found and fixed
Cell 13 (save results) from the v4 Cyclic notebook crashed with `TypeError: keys must be str, int, float, bool or None, not tuple` — `A_MATRIX`/`B_MATRIX` use tuple keys like `("1","1")`, which `json.dump` can't serialize. Fixed by converting to string keys (`f"{k[0]},{k[1]}"`) just for the save step. Confirmed: **cells 1-12 all ran successfully** in the v4 run — this was a save-step bug only, not a run failure. Real results existed in memory/checkpoints the whole time.

### Model substitution finding — GLM-4.6 too large for T4×2
Checked zai-org's own model page: **GLM-4.6 is a 100B+ parameter MoE flagship model** — does not fit on Kaggle's T4×2 (2×15.6GB) even in 4-bit quantization. Same substitution logic already used for Llama-70B→8B and full DeepSeek-R1→Distill applies here: swapped in **GLM-4-9B-Chat** (zai-org/glm-4-9b-chat, 9B, fits easily) as the legitimate, smaller stand-in. Flagged explicitly as a stated scope limitation, not a silent swap.

### Roster and pipeline changes
- Added a new checkpoint-carryover cell (Cell 3B) that copies the previous run's Llama/R1 Cyclic checkpoint JSONs from an attached Kaggle input dataset (`/kaggle/input/datasets/nagajaideepchowdary/cyclic-lamma-deepseek/checkpoints`) into the working checkpoint directory — so Cells 9-10 skip instantly instead of re-running R1's ~2-hour job.
- Added two new model cells: **Qwen3-8B** (thinking mode on by default, treated as reasoning-tier, same generous 4000-token budget as R1 — applied consistently, not a new per-model citation) and **GLM-4-9B-Chat** (non-reasoning-tier, fast, 250-token budget like Llama).
- Updated the tau-fitting, NLL diagnostic, and save-results cells to cover all 4 models.
- Final notebook order locked: Cells 1-3 → **3B (new)** → 4-10 (unchanged) → **10B (new, Qwen3-8B)** → **10C (new, GLM-4-9B-Chat)** → 11-13 (edited for 4 models).

### Save Version queued
**v5**, name: `v5-cyclic-4models-llama-r1-qwen3-glm4`. Not yet run as of this log.

---

## 7. Where This Leaves the Project

| Stage | Status |
|---|---|
| 1. Game Engine | ✅ Done, generalized past PD |
| 2. Regret Calculator | ✅ Done, validated on 2 games |
| 3. τ Calculator | ✅ Working on Cyclic (fixed PD's collapse) — **but only tested on 2 models so far** |
| 4. Merge & Compare | Not started — blocked on 4-model Cyclic data |
| 5. Statistical Test | Not started |

**Literature positioning, updated:** the core gap (τ vs. formal regret, same models/games) is still open. But we now have two real, cited adjacent findings that pull in opposite directions, and an explicit plan for framing the paper depending on which way our own data goes.

---

## 8. External Peer Review — Two Independent AI Reviews, Cross-Verified

After the honest status check above, the full research-direction brief (Section "Independent Review Request" — see companion file `Independent_Review_Prompt_Topic_A.md`) was given to **two different AI systems**, independently, for critical review. Neither saw the other's answer. Both were then checked against primary sources rather than accepted at face value.

### What Both Reviews Independently Agreed On (strongest signal — convergent without collusion)
1. **τ sample count is too low.** Both flagged 8-20 one-shot samples as insufficient.
2. **γ should be jointly fit with τ, not fixed at 1.0.** Both independently caught this as a real simplification, not just a stated limitation.
3. **2 models (soon 4) is not enough to know if τ discriminates model capability at all.**
4. **10 rounds is short for a regret claim.**
5. **The τ=4.36 tie should be read as "non-discriminative measurement," not "equal reasoning depth."** Both explained *why* more precisely than we had: near-deterministic action distributions produce flat/unstable likelihood surfaces, not genuine equality.
6. **Core research question is coherent and the gap is genuinely open** — both independently confirmed this via their own literature search.
7. **AAMAS 2027 is realistic only with real scope cuts.**

### Claims Checked Against Primary Sources (not just accepted)

**Confirmed true, independently verified:**
- García-Pola (2020), *"Do People Minimize Regret in Strategic Situations? A Level-k Comparison,"* *Games and Economic Behavior* Vol. 124, pp. 82-104 — confirmed real. Shows regret-minimization and level-k reasoning are established, sometimes-agreeing/sometimes-diverging competing models in human behavioral economics. Genuinely useful additional theoretical citation, not found in our own earlier search.
- AAMAS 2027 dates confirmed via the official Calls page: abstract Sept/Oct 1, 2026; full paper Oct 8, 2026.
- Jia et al.'s actual method, re-checked against the full text we already had fetched: **30 independent trials per game**, and **τ and γ are jointly estimated via MLE** — not τ alone with γ fixed. Both reviews were right; our implementation deviates from this on both counts.

**Pushed back on, with citation:**
- Reviewer 1 called the reactive-scripted-opponent regret setup "the most serious issue, fix before scaling — priority #1." Checked this against **Park et al.'s own paper** (already fetched in full earlier this week): their repeated-games experiments use LLM-vs-LLM (fully reactive/adaptive opponents) and compute standard external regret exactly the same way we do. Their own concluding limitations section explicitly lists **"policy regret (Arora et al., 2012a), which accounts for adaptive adversaries"** as a stated **future direction**, not something invalidating their published results. Conclusion: this is a real, worth-stating limitation (and the suggested oblivious-opponent control is cheap and good to add) — but it does not invalidate current numbers; we're using the same standard as the field's seminal paper, not violating it.
- Reviewer 2's claim that 4-bit quantization "differentially harms reasoning models" — plausible but no specific source cited, could not independently verify as an established phenomenon. Noted as unverified, not treated as confirmed.

### Synthesized Action Items From This Review
1. Bump one-shot τ samples toward ~30 (Jia et al.'s own standard) before trusting any model comparison.
2. Jointly fit τ and γ instead of fixing γ=1 — a real implementation fix, not just a caveat.
3. Add an oblivious (pre-generated, non-reactive) opponent condition alongside the current reactive one for regret, report both.
4. Increase repeated-game rounds from 10 toward 25-50 where compute allows.
5. Cite García-Pola (2020) in the theoretical motivation section.
6. Do not lock the title ("Deep but Not No-Regret") — it pre-announces an outcome not yet in evidence.
7. Treat 4 models as still-directional, not conclusive, even once v5 completes.
8. Frame the eventual analysis as a correlation/effect-size question (ρ(τ, −regret)) rather than a binary "did it dissociate."

---

## 9. Decision: v5 Left Running, Not Stopped

v5 (adding Qwen3-8B and GLM-4-9B-Chat to Cyclic) was already ~30 minutes into its run when the review above came back — Llama and R1 skip instantly via checkpoint, so that 30 minutes is almost entirely fresh work on Qwen3-8B, a slow reasoning model. **Decision: let it finish rather than stop and rebuild immediately.**

**Reasoning:** the methodology fixes above (more samples, joint τ/γ, oblivious opponent, longer rounds) are real and worth doing, but v5 as currently configured is still a useful **cheap directional pilot** — it answers the immediate blocking question (does τ vary at all across 4 structurally different models under current settings, or does it keep landing near the same value?) before spending more compute on a fully rigorous re-run. v5's output will be treated as a pilot signal, not the final dataset. The rigorous version (v6) will apply the fixes above.

---

## 10. Next Steps (in order, updated)

1. **Let v5 finish** (Qwen3-8B + GLM-4-9B-Chat on Cyclic, current methodology) — read as a pilot signal only.
2. Once v5 completes: check whether τ varies across the 4 models at all. This alone is informative regardless of methodology fixes — if it doesn't vary even with 4 structurally different models, that's worth knowing before investing in v6.
3. **Read R1's raw `full_response` text** across the 10 PD and 10 Cyclic rounds (data already collected, free to check) — does its stated reasoning visibly deepen over rounds, or stay static? Direct, cheap test of Teo's "static level-k" claim on our own data.
4. **Build v6** with the review's fixes: ~30 one-shot τ samples, joint τ/γ MLE fitting, an added oblivious-opponent regret condition alongside the reactive one, and longer repeated-game rounds (25-50) where compute allows.
5. Do the first real Stage 4 comparison on v6's data — sort by regret, sort by τ, check rank agreement, report as correlation/effect size rather than binary dissociation.
6. Add a second τ-capable game (Biased, previously identified as backup) as a robustness check.
7. Only after a properly-fit dataset (v6-quality) across 4+ models × 2 games: run statistical analysis (Wilcoxon signed-rank, but reported alongside effect sizes and confidence intervals given the small model count, per the reviews' caution about over-relying on a tiny-sample significance test).

*(This file will be updated with v5's actual pilot results once the run completes, and again once v6 is built and run.)*
