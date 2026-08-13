# Topic A — Day 4 Session Log (Kaggle T4×2 Build)

*Continuation from Day 3. Full Save Version run completed successfully end-to-end. Everything done, learned, tested, expected, and actually got today, in one place.*

**Session date:** Aug 9, 2026 (continued)
**Environment:** Kaggle Notebook, GPU T4×2, Save Version run
**Status at end of day:** Cells 1-12 (all cells) ran successfully end-to-end in one Save Version pass, ~9,576 seconds (~2.66 hours) total. No crashes. Regret metric confirmed solid on Prisoner's Dilemma for both models. τ metric found to be **structurally unidentifiable on Prisoner's Dilemma specifically** — a real methodological finding, not a bug, that changes the plan going forward.

---

## 1. What We Set Out To Do Today

Fix yesterday's broken τ (reasoning depth) calculation. Yesterday's version fit τ from a single deterministic (greedy-decoded) action sequence per model, which caused the optimizer to collapse to the search boundary (τ=0.01) — meaningless, not a real estimate. Today's goal: rebuild τ estimation properly, matching the Behavioral Game Theory paper's actual method (sampling-based action-count frequencies), verify it with a diagnostic, and commit a clean Save Version.

---

## 2. What Changed in the Code Since Day 3

| Change | Why |
|---|---|
| Added `sample_action_counts_checkpointed()` function | Runs N independent **sampled** generations (`do_sample=True`, temperature=0.7) at a fixed game-state, counts how many times the model picked C vs D. This is what actually makes τ statistically identifiable — a single deterministic pick gives the fitting procedure nothing to work with. |
| Added 3 representative game-states per model: "early" (round 1, no history), "mid" (after round 5), "late" (after round 9) | Cheaper than repeating full 10-round games — samples the model's distribution at a few meaningful decision points instead. |
| `n_samples=15` for Llama-3-8B, `n_samples=5` for DeepSeek-R1-Distill | Reduced from an original plan of 15 for both, specifically for R1, because R1's per-generation time (up to ~700s in hard game-states) made 15×3 contexts a multi-hour risk against Kaggle's 12-hour session cap. Every sample is checkpointed individually regardless of count. |
| Rewrote `fit_tau_from_counts()` to fit τ from **pooled action counts** across contexts, not a single trajectory | Matches the paper's actual method — TQRE is a model of choice *frequencies*, not single deterministic picks. |
| Added NLL (negative log-likelihood) diagnostic cell, plotting the curve across τ∈[0.01, 8.0] | To verify the fix actually worked — checks whether the fitted τ sits at a genuine interior minimum or has collapsed to the search boundary again. |
| Model loader kept official-repo-first with mirror fallback (unchanged from Day 3) | Still working correctly — official `meta-llama/Meta-Llama-3-8B-Instruct` loaded successfully this run (access was approved yesterday). |

---

## 3. Bugs/Issues Hit and Fixed Today

| # | Issue | Fix |
|---|---|---|
| 1 | `AttributeError: module 'numpy' has no attribute 'math'` in the original τ fitting code | `np.math.factorial` doesn't exist in current numpy — replaced with Python's built-in `math.factorial` (`import math` added) |
| 2 | Rebuilding `result_llama`/`result_r1` from saved checkpoint JSONs (when resuming in a session without the model loaded) | Wrote `rebuild_result_from_checkpoint()` — reconstructs the full result dict (regret, FTRL, FTPL, action sequence) directly from the checkpoint JSON, no GPU/model needed |
| 3 | Uploaded checkpoint files landed as a Kaggle **input dataset** (read-only), not a working-directory upload | Adjusted `CHECKPOINT_DIR` reference to point at `/kaggle/input/datasets/.../checkpoints` for reading; original `/kaggle/working/checkpoints` remains the write path for new runs |
| 4 | Estimated R1 sampling runtime at `n_samples=15` for all 3 contexts (~5.8 hours) risked exceeding safe session budget | Reduced R1 to `n_samples=5`; actual run confirmed this was the right call — total session time landed at ~2.66 hours |

---

## 4. What We Expected vs. What We Actually Got

**Expected:** with real sampling (not deterministic decoding) and multiple independent trials per context, τ should be identifiable — i.e., the NLL curve should show a genuine interior minimum, not a boundary collapse, for both models.

**Got:**

| Model | Regret vs FTRL | Action counts (early / mid / late) | Fitted τ | Min NLL | Boundary NLL (τ=0.01) |
|---|---|---|---|---|---|
| Llama-3-8B | −8 (beats FTRL) | 15C/0D, 0C/15D, 0C/15D | **0.25** | 30.914 | 31.168 |
| DeepSeek-R1-Distill | −15 (beats FTRL by more) | 4C/1D, 4C/1D, 0C/5D | **0.01** | 10.405 | 10.405 (identical) |

**Llama-3-8B:** despite `temperature=0.7`, it produced **zero variation** across all 15 samples in every single context (perfectly unanimous C or D each time) — this model is extremely confident/low-entropy on this game regardless of sampling. Its τ did move slightly off the exact boundary (0.25 vs. 0.01), a small real improvement over yesterday, but the gap between the true minimum and the boundary NLL is tiny (~0.25 nats) — a weak, not strong, signal.

**DeepSeek-R1-Distill:** genuinely showed sampling variation this time (4C/1D and 4C/1D, not unanimous) — the sampling fix itself worked as intended. But τ **still collapsed to the exact boundary** (0.01), with the minimum NLL identical to the boundary NLL. This is a different failure mode than yesterday's (yesterday: no variation in the data at all; today: real variation in the data, but the fit still can't identify τ).

---

## 5. The Actual Finding — Why R1 Still Collapsed (Not a Bug)

This is today's most important result, and it's a real methodological insight rather than a coding error.

**Prisoner's Dilemma has a dominant strategy: defect is always at least as good as cooperate, regardless of what the opponent does.** In the level-k reasoning model TQRE is built on, this means **every reasoning level k≥1 predicts the exact same action** (defect) — there's no divergence between "shallow" and "deep" strategic reasoning in this specific game. The only thing τ can distinguish is level-0 (uniform random, 50/50) versus "any positive level" (defect). With R1's counts split across contexts that sometimes lean cooperate-heavy and sometimes defect-heavy, the optimizer is stuck choosing between raising τ (which fits "late" well but wrecks "early"/"mid") or keeping τ near zero (a uniform compromise). The compromise wins, and τ collapses toward 0 — this is the mathematically correct answer given the data and the game, not a broken fit.

**Practical conclusion: Prisoner's Dilemma is structurally a poor game for measuring τ via this method**, independent of how much data we sample. This isn't a wasted effort — it's exactly the kind of finding worth having *before* scaling up model count, and it directly validates why the original game suite plan always included multiple, structurally different games (win-win, unfair, cyclic, biased, second-best) — some of those have mixed-strategy equilibria where different reasoning depths genuinely predict different actions, which is where τ should actually become measurable.

**Regret, by contrast, is unaffected by this issue** — that metric doesn't depend on reasoning-level divergence, so today's Prisoner's Dilemma regret numbers for both models remain valid and usable as-is.

---

## 6. Full Code State (Cells 1-12, as run in this Save Version)

All cells ran successfully in one pass. No changes needed from the version already saved — this Save Version is the working, correct state. Summary of what each cell does:

1. GPU check
2. Install dependencies
3. Imports, config, checkpoint dir, HF_TOKEN load
4. Model loader with official-repo-first, mirror-fallback logic
5. Game definitions (PAYOFFS, tit_for_tat_opponent, build_prompt, parse_action)
6. Checkpointed game-playing function (`play_game_checkpointed`)
7. Regret + FTRL/FTPL calculators + `run_full_evaluation` wrapper
8. Load Llama-3-8B (official-first) + run full evaluation on Prisoner's Dilemma
9. `sample_action_counts_checkpointed()` definition + Llama-3-8B sampling across 3 contexts (n=15 each)
10. Unload Llama-3, load DeepSeek-R1-Distill (official-first) + run full evaluation
11. DeepSeek-R1 sampling across 3 contexts (n=5 each, reduced for time budget)
12. `fit_tau_from_counts()` — τ fitting from pooled counts + final printout
13. NLL diagnostic — plots the curve, prints min NLL and boundary NLL for both models

*(Full code text preserved in the Day 3 log file and the Save Version notebook itself — not re-pasted here since nothing changed from what's already saved.)*

---

## 7. Where This Leaves Us in the Overall Project

| Stage | Status |
|---|---|
| 1. Game Engine | ✅ Done |
| 2. Regret Calculator | ✅ Done, validated, working correctly on Prisoner's Dilemma for both models |
| 3. τ Calculator | ⚠️ Correctly implemented (sampling-based, matches paper's method) — but **found to be unidentifiable on Prisoner's Dilemma specifically**, due to PD's dominant-strategy structure. Not a bug to fix; a game-choice issue to work around. |
| 4. Merge & Compare | Not started |
| 5. Statistical Test | Not started |

**Still nothing paper-reportable yet** — same honest assessment as Day 3, now with one additional, genuinely useful negative result: we know *why* τ won't work on Prisoner's Dilemma, which tells us exactly what property the next game needs to have.

---

## 8. What's Next

1. **Move to a game with a genuine mixed-strategy equilibrium** for the next τ test — candidates from Park et al.'s original suite: "biased," "cyclic," or "second-best." These have equilibria where different reasoning depths (k=1 vs k=2 vs k=3) predict *different* actions, unlike PD where they all agree. This is the actual fix — not more sampling, not a different fitting method.
2. **Keep Prisoner's Dilemma's regret numbers** — they're valid and usable; no need to redo them.
3. **Re-run the sampling + τ pipeline on the new game** for the same 2 models (Llama-3-8B, DeepSeek-R1-Distill) first, to confirm τ becomes identifiable before scaling up.
4. **Once τ is confirmed identifiable on at least one game:** add the 2 remaining starter-set models (Qwen3-8B, GLM-4.6 or similar).
5. **Then:** add the non-stationary/sine-trend game (tests adapting to a changing environment — a different axis than opponent-modeling games).
6. **Stage 4 (Merge & Compare):** once regret + τ exist for all 4 models × multiple games, build the comparison table and scatter plot (regret rank vs. τ rank).
7. **Stage 5 (Statistical test):** Wilcoxon signed-rank on the rank gap.
8. **Only then:** expand to the fuller model roster and full game suite per original scope.

**One thing worth deciding before the next session:** which of "biased," "cyclic," or "second-best" to build first — worth a quick look at Park et al.'s actual payoff matrices for each to pick whichever has the cleanest, most clearly-separated level-k predictions, since that will give the clearest τ signal fastest.
