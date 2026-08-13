# Day 3 — Kaggle Pipeline Build Log

*Everything done, learned, tested, and produced today. Continue from here tomorrow — τ calculation was run last, its output was not yet pasted back before the session ended.*

**Environment:** Kaggle Notebook, GPU T4×2 (2 × 15.6 GB VRAM), 4-bit quantized model loading via `bitsandbytes`.

---

## 1. What We Set Out To Do Today

Build and validate, end-to-end, the first working piece of the Topic A experiment pipeline:
- Confirm Kaggle T4×2 environment works.
- Load one non-reasoning model (Llama-3-8B) and one reasoning model (DeepSeek-R1-Distill-14B).
- Run each through a real repeated Prisoner's Dilemma game (10 rounds vs. Tit-for-Tat).
- Compute regret (vs. FTRL/FTPL baselines) for each.
- Compute reasoning depth (τ) for each.
- Compare the two models' regret and τ — first real data point toward the dissociation question.

**Scope note carried over from the consolidated reference doc:** original roster (Llama-3-70B, full DeepSeek-R1-671B) was swapped for Kaggle-feasible sizes — **Llama-3-8B-Instruct** and **DeepSeek-R1-Distill-Qwen-14B** — since the full-size models don't fit T4×2 VRAM even quantized. This is a stated, defensible scope limitation, not a flaw.

---

## 2. Environment Setup (Steps 1–2)

### GPU check
```python
import torch
print("CUDA available:", torch.cuda.is_available())
print("GPU count:", torch.cuda.device_count())
for i in range(torch.cuda.device_count()):
    print(f"GPU {i}: {torch.cuda.get_device_name(i)}, {torch.cuda.get_device_properties(i).total_memory / 1e9:.1f} GB")
```
**Result:** 2× Tesla T4, 15.6 GB each. Confirmed working.

### Install
```python
!pip install -q -U transformers accelerate bitsandbytes
```
**Result:** Installed cleanly.

---

## 3. Failure Points Hit Today (and fixes)

### Failure 1 — Meta's Llama-3-8B is gated, approval delayed
**Error:** `GatedRepoError: 403 Client Error... Your request to access model meta-llama/Meta-Llama-3-8B-Instruct is awaiting a review from the repo authors.`

Assumed "instant approval" — this was **wrong**. Meta's manual review can take hours, not instant.

**Fix used:** Switched to `NousResearch/Meta-Llama-3-8B-Instruct` — an ungated, byte-identical community mirror of the same weights. No token/approval needed. This unblocked us immediately.

**Update:** The official gated Meta repo got approved later in the session too — not yet cross-checked against the NousResearch copy, but they should be identical since it's the same weights. **To-do tomorrow (optional, low priority):** sanity-check both give identical outputs.

### Failure 2 — `apply_chat_template` returns dict, not raw tensor
**Error:**
```
AttributeError: (from tokenizer_utils_base.py, tokenizer.decode(...) / input_ids.shape)
KeyError: 'shape'
```
Root cause: current `transformers` version returns a `BatchEncoding` dict-like object from `apply_chat_template`, not a plain tensor — so `input_ids.shape` failed since `input_ids` was actually the whole dict.

**Fix:** Added `return_dict=True` to `apply_chat_template()`, then unpacked with `model.generate(**inputs, ...)` and referenced `inputs["input_ids"].shape` instead of `input_ids.shape`. This fix is now baked into all our code going forward.

### Failure 3 — FTRL/FTPL baselines got an unfair advantage (conceptual bug, not a crash)
**What happened:** First version of `simulate_ftrl`/`simulate_ftpl` replayed the **exact same fixed opponent history** that occurred during the real model's game — but that history had already been shaped by the model's own choices (Tit-for-Tat reacts to whoever it's playing). This let FTRL/FTPL "know" the opponent would stay cooperative for 5 rounds, something that could never happen if they'd actually been playing live (Tit-for-Tat would retaliate immediately against early defection).

**Symptom:** FTRL baseline scored an inflated 30/30 (perfect), because it defected every round against an opponent who, in the real history, never got the chance to retaliate.

**Fix:** Rewrote both simulators as `simulate_ftrl_live()` / `simulate_ftpl_live()` — they now call the *same* `tit_for_tat_opponent()` function used for the real model, reacting live to each baseline's own choices round-by-round, not to a replayed fixed history. This produced fair, correct numbers (see results below).

### Failure 4 — Reasoning models need far more tokens than expected
**What happened:** With `max_new_tokens=100` then `250`, DeepSeek-R1-Distill's response got cut off mid-reasoning, never reaching a parseable `DECISION: C/D` line.

**Fix (evidence-based, not guessed):** Searched and read the actual Behavioral Game Theory paper (Jia et al. 2025, arXiv:2502.20432), Appendix B Table 8, which measured real completion-token counts for DeepSeek-R1 across game types:

| Game type | Mean tokens | Min | Max |
|---|---|---|---|
| Competitive | 10,979.93 | 9,050 | 13,350 |
| Cooperative | 1,764.67 | 1,001 | 2,931 |
| **Mixed-strategy (Prisoner's Dilemma's category)** | **2,318.33** | **1,599** | **3,742** |

Set our budget to **`max_new_tokens=4000`** for reasoning models (headroom above their cited max of 3,742 for the PD-relevant category), citing this paper directly rather than guessing.

**Bonus finding from that same paper, worth keeping for later:** their prompts explicitly said "do not include any thinking process" — the reasoning model generated thousands of tokens of reasoning anyway. Confirms you cannot prompt a reasoning model out of "thinking out loud." Also: the paper found **longer reasoning chains do not correlate with better decisions** — a directly relevant, citable finding for our own project's eventual mechanism-analysis section.

### Failure 5 — Kernel/session interruptions (accidental stop, slow runs)
**What happened:** Manually stopped a run partway through (round 7/10) to go to dinner; also, some installation/state issues after a restart caused failures ("something missing").

**Fix:** Rebuilt the whole pipeline around **per-round checkpointing to disk** (JSON files in `/kaggle/working/checkpoints/`), plus defensive `if "model_x" not in globals()` reload guards in every cell. Net effect: **any cell can be re-run from scratch, at any time, after any failure, and it will either skip finished work or resume from the last completed round.** This is now the standing pattern for all future runs, not just today's.

---

## 4. Final Working Code (validated, use this going forward)

### Cell 1 — GPU Check
```python
import torch
print("CUDA available:", torch.cuda.is_available())
print("GPU count:", torch.cuda.device_count())
for i in range(torch.cuda.device_count()):
    print(f"GPU {i}: {torch.cuda.get_device_name(i)}, {torch.cuda.get_device_properties(i).total_memory / 1e9:.1f} GB")
```

### Cell 2 — Install
```python
!pip install -q -U transformers accelerate bitsandbytes
```

### Cell 3 — Core Imports and Config
```python
import os, json, time, gc
import numpy as np
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig

CHECKPOINT_DIR = "/kaggle/working/checkpoints"
os.makedirs(CHECKPOINT_DIR, exist_ok=True)

BNB_CONFIG = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_compute_dtype=torch.float16,
)
print("Setup ready. Checkpoints will save to:", CHECKPOINT_DIR)
```

### Cell 4 — Game Definitions
```python
PAYOFFS = {
    ("C", "C"): (3, 3),
    ("C", "D"): (0, 5),
    ("D", "C"): (5, 0),
    ("D", "D"): (1, 1),
}

def tit_for_tat_opponent(history):
    if len(history) == 0:
        return "C"
    return history[-1]["your_action"]

def build_prompt(history, current_round, total_rounds):
    game_rules = (
        "You are playing a repeated Prisoner's Dilemma game with an opponent for "
        f"{total_rounds} rounds. Each round, you and your opponent simultaneously choose "
        "either 'C' (cooperate) or 'D' (defect).\n\n"
        "Payoffs per round:\n"
        "- Both cooperate: you get 3, opponent gets 3\n"
        "- You cooperate, opponent defects: you get 0, opponent gets 5\n"
        "- You defect, opponent cooperates: you get 5, opponent gets 0\n"
        "- Both defect: you get 1, opponent gets 1\n\n"
        "Your goal is to maximize your own total payoff across all rounds.\n\n"
    )
    if len(history) == 0:
        history_text = "This is round 1. No history yet.\n"
    else:
        history_text = "History so far:\n"
        for i, h in enumerate(history):
            history_text += (
                f"Round {i+1}: You played {h['your_action']}, "
                f"opponent played {h['opponent_action']}, you scored {h['your_payoff']}.\n"
            )
    instruction = (
        f"\nIt is now round {current_round} of {total_rounds}. "
        "Think briefly about your strategy, then respond with your final decision "
        "on a new line in exactly this format:\nDECISION: C\nor\nDECISION: D"
    )
    return game_rules + history_text + instruction

import re
def parse_action(response_text):
    match = re.search(r"DECISION:\s*([CD])", response_text.upper())
    if match:
        return match.group(1)
    if "D" in response_text[-20:].upper():
        return "D"
    return "C"

print("Game logic loaded.")
```

### Cell 5 — Checkpointed Game-Playing Function
```python
def play_game_checkpointed(model, tokenizer, model_name, game_name="prisoners_dilemma",
                             total_rounds=10, max_new_tokens=250, verbose=True):
    checkpoint_path = os.path.join(CHECKPOINT_DIR, f"{model_name}_{game_name}.json")

    if os.path.exists(checkpoint_path):
        with open(checkpoint_path, "r") as f:
            history = json.load(f)
        print(f"Resuming '{model_name}' on '{game_name}' — {len(history)}/{total_rounds} rounds already done.")
    else:
        history = []
        print(f"Starting fresh: '{model_name}' on '{game_name}'.")

    for round_num in range(len(history) + 1, total_rounds + 1):
        prompt_text = build_prompt(history, round_num, total_rounds)
        messages = [
            {"role": "system", "content": "You are a strategic game-playing agent."},
            {"role": "user", "content": prompt_text}
        ]
        inputs = tokenizer.apply_chat_template(
            messages, add_generation_prompt=True, return_tensors="pt", return_dict=True
        ).to(model.device)

        t0 = time.time()
        output = model.generate(**inputs, max_new_tokens=max_new_tokens, do_sample=False)
        elapsed = time.time() - t0

        response_text = tokenizer.decode(output[0][inputs["input_ids"].shape[-1]:], skip_special_tokens=True)
        your_action = parse_action(response_text)
        opponent_action = tit_for_tat_opponent(history)
        your_payoff, opponent_payoff = PAYOFFS[(your_action, opponent_action)]

        history.append({
            "round": round_num, "your_action": your_action, "opponent_action": opponent_action,
            "your_payoff": your_payoff, "opponent_payoff": opponent_payoff, "full_response": response_text,
        })

        with open(checkpoint_path, "w") as f:
            json.dump(history, f, indent=2)

        rounds_left = total_rounds - round_num
        eta_min = (rounds_left * elapsed) / 60
        if verbose:
            print(f"[{model_name}] Round {round_num}/{total_rounds} | Action: {your_action} | "
                  f"Opp: {opponent_action} | Payoff: {your_payoff} | {elapsed:.0f}s/round | ETA: {eta_min:.1f} min")

    return history

print("Checkpointed game function ready.")
```

### Cell 6 — Regret + FTRL/FTPL Calculators (live-simulation, fair comparison)
```python
def compute_regret(game_history, payoff_matrix, opponent_history):
    actions = ["C", "D"]
    total_rounds = len(game_history)
    actual_payoff = sum(h["your_payoff"] for h in game_history)
    best_fixed_payoff = -np.inf
    for fixed_action in actions:
        total = sum(payoff_matrix[(fixed_action, opp)][0] for opp in opponent_history)
        best_fixed_payoff = max(best_fixed_payoff, total)
    regret = best_fixed_payoff - actual_payoff
    return {
        "actual_payoff": actual_payoff,
        "best_fixed_payoff": best_fixed_payoff,
        "regret_vs_best_fixed": regret,
        "avg_regret_per_round": regret / total_rounds,
    }

def simulate_ftrl_live(opponent_strategy_fn, payoff_matrix, total_rounds, learning_rate=0.5, seed=42):
    rng = np.random.default_rng(seed)
    actions = ["C", "D"]
    cumulative_scores = {"C": 0.0, "D": 0.0}
    history, total_payoff = [], 0
    for _ in range(total_rounds):
        scores = np.array([cumulative_scores[a] for a in actions])
        probs = np.exp(learning_rate * scores) / np.exp(learning_rate * scores).sum()
        chosen = rng.choice(actions, p=probs)
        opp_act = opponent_strategy_fn(history)
        payoff = payoff_matrix[(chosen, opp_act)][0]
        total_payoff += payoff
        history.append({"your_action": chosen, "opponent_action": opp_act})
        for a in actions:
            cumulative_scores[a] += payoff_matrix[(a, opp_act)][0]
    return {"total_payoff": total_payoff, "actions": [h["your_action"] for h in history]}

def simulate_ftpl_live(opponent_strategy_fn, payoff_matrix, total_rounds, noise_scale=1.0, seed=42):
    rng = np.random.default_rng(seed)
    actions = ["C", "D"]
    cumulative_scores = {"C": 0.0, "D": 0.0}
    history, total_payoff = [], 0
    for _ in range(total_rounds):
        noisy_scores = {a: cumulative_scores[a] + rng.normal(0, noise_scale) for a in actions}
        chosen = max(noisy_scores, key=noisy_scores.get)
        opp_act = opponent_strategy_fn(history)
        payoff = payoff_matrix[(chosen, opp_act)][0]
        total_payoff += payoff
        history.append({"your_action": chosen, "opponent_action": opp_act})
        for a in actions:
            cumulative_scores[a] += payoff_matrix[(a, opp_act)][0]
    return {"total_payoff": total_payoff, "actions": [h["your_action"] for h in history]}

def run_full_evaluation(model, tokenizer, model_name, game_name="prisoners_dilemma",
                          total_rounds=10, max_new_tokens=250, opponent_fn=tit_for_tat_opponent, payoff_matrix=PAYOFFS):
    game_history = play_game_checkpointed(model, tokenizer, model_name, game_name, total_rounds, max_new_tokens, verbose=True)
    opponent_actions = [h["opponent_action"] for h in game_history]
    regret_result = compute_regret(game_history, payoff_matrix, opponent_actions)
    ftrl_result = simulate_ftrl_live(opponent_fn, payoff_matrix, total_rounds)
    ftpl_result = simulate_ftpl_live(opponent_fn, payoff_matrix, total_rounds)
    return {
        "model": model_name, "game": game_name,
        "actual_payoff": regret_result["actual_payoff"],
        "best_fixed_payoff": regret_result["best_fixed_payoff"],
        "regret_vs_best_fixed": regret_result["regret_vs_best_fixed"],
        "ftrl_payoff": ftrl_result["total_payoff"], "ftpl_payoff": ftpl_result["total_payoff"],
        "regret_vs_ftrl": ftrl_result["total_payoff"] - regret_result["actual_payoff"],
        "regret_vs_ftpl": ftpl_result["total_payoff"] - regret_result["actual_payoff"],
        "beats_ftrl": regret_result["actual_payoff"] >= ftrl_result["total_payoff"],
        "beats_ftpl": regret_result["actual_payoff"] >= ftpl_result["total_payoff"],
        "action_sequence": [h["your_action"] for h in game_history],
        "raw_history": game_history,
    }

print("Regret/FTRL/FTPL calculators ready.")
```

### Cell 7 — Llama-3-8B Run
```python
model_id_llama = "NousResearch/Meta-Llama-3-8B-Instruct"

if "model_llama" not in globals():
    tokenizer_llama = AutoTokenizer.from_pretrained(model_id_llama)
    model_llama = AutoModelForCausalLM.from_pretrained(model_id_llama, quantization_config=BNB_CONFIG, device_map="auto")
    print("Llama-3-8B loaded.")

result_llama = run_full_evaluation(model_llama, tokenizer_llama, model_name="Llama-3-8B-Instruct", max_new_tokens=250)
print(result_llama["model"], "| Actual:", result_llama["actual_payoff"], "| FTRL:", result_llama["ftrl_payoff"], "| FTPL:", result_llama["ftpl_payoff"])
```

### Cell 8 — Unload Llama-3, Load DeepSeek-R1-Distill, Run
```python
if "model_llama" in globals():
    del model_llama, tokenizer_llama
    gc.collect()
    torch.cuda.empty_cache()

model_id_r1 = "deepseek-ai/DeepSeek-R1-Distill-Qwen-14B"

if "model_r1" not in globals():
    tokenizer_r1 = AutoTokenizer.from_pretrained(model_id_r1)
    model_r1 = AutoModelForCausalLM.from_pretrained(model_id_r1, quantization_config=BNB_CONFIG, device_map="auto")
    print("DeepSeek-R1-Distill loaded.")

# max_new_tokens cited: Jia et al. 2025 (arXiv:2502.20432) Table 8, mixed-motive/PD-category — mean 2318, max 3742
result_r1 = run_full_evaluation(model_r1, tokenizer_r1, model_name="DeepSeek-R1-Distill-Qwen-14B", max_new_tokens=4000)
print(result_r1["model"], "| Actual:", result_r1["actual_payoff"], "| FTRL:", result_r1["ftrl_payoff"], "| FTPL:", result_r1["ftpl_payoff"])
```

### Cell 9 — τ (Reasoning Depth) Calculator — RUN LAST, OUTPUT NOT YET RECORDED
```python
from scipy.optimize import minimize_scalar

def fit_tau(action_sequence, opponent_action_sequence, payoff_matrix, gamma=1.0, max_k=10):
    actions = ["C", "D"]

    def poisson_weight(k, tau):
        return (tau**k) * np.exp(-tau) / np.math.factorial(k)

    def level_k_strategy(k, payoff_matrix, prev_levels):
        if k == 0:
            return {"C": 0.5, "D": 0.5}
        expected_utils = {}
        for a in actions:
            eu = sum(prev_levels[k-1][opp_a] * payoff_matrix[(a, opp_a)][0] for opp_a in actions)
            expected_utils[a] = eu
        max_u = max(expected_utils.values())
        exp_vals = {a: np.exp(gamma * k * (expected_utils[a] - max_u)) for a in actions}
        total = sum(exp_vals.values())
        return {a: exp_vals[a] / total for a in actions}

    def neg_log_likelihood(tau):
        levels = {0: {"C": 0.5, "D": 0.5}}
        for k in range(1, max_k):
            levels[k] = level_k_strategy(k, payoff_matrix, levels)
        total_ll = 0
        for your_a, opp_a in zip(action_sequence, opponent_action_sequence):
            p_action = 0.0
            for k in range(max_k):
                p_action += poisson_weight(k, tau) * levels[k][your_a]
            p_action = max(p_action, 1e-10)
            total_ll += np.log(p_action)
        return -total_ll

    result = minimize_scalar(neg_log_likelihood, bounds=(0.01, 8.0), method='bounded')
    return result.x

tau_llama = fit_tau(result_llama["action_sequence"],
                     [h["opponent_action"] for h in result_llama["raw_history"]],
                     PAYOFFS)
tau_r1 = fit_tau(result_r1["action_sequence"],
                  [h["opponent_action"] for h in result_r1["raw_history"]],
                  PAYOFFS)

print(f"Llama-3-8B      | Regret vs FTRL: {result_llama['regret_vs_ftrl']:+d} | τ: {tau_llama:.2f}")
print(f"DeepSeek-R1     | Regret vs FTRL: {result_r1['regret_vs_ftrl']:+d} | τ: {tau_r1:.2f}")
```
**Status: this cell was run but the printed τ output was not pasted back before the session ended. First task tomorrow: get and record this output.**

**Honest caveat already flagged, keep it in the write-up later:** the real TQRE method (Behavioral GT paper) uses 30 independent one-shot trials per game to fit τ. We only have 1 repeated 10-round game per model, so this `fit_tau()` is a simplified adaptation — treating each round as one "trial" — not the paper's exact method. Fine for this exploratory stage; needs to be stated as a limitation, and possibly revisited (e.g. running multiple independent repeated games per model to get a real distribution) before this becomes a real reported number in the paper.

---

## 5. Results Actually Obtained Today

### Llama-3-8B-Instruct (non-reasoning baseline) — Prisoner's Dilemma vs. Tit-for-Tat, 10 rounds
- **Action sequence:** `C, C, C, C, D, D, D, D, D, D`
- **Actual payoff: 22**
- **Best fixed-action payoff (hindsight): 30** → regret vs. best-fixed = **8**
- **FTRL baseline payoff (live sim): 14** → model beat FTRL by **8**
- **FTPL baseline payoff (live sim): 16** → model beat FTPL by **6**
- **Behavior pattern:** cooperated 4 rounds, defected once to exploit, then got permanently stuck in mutual defection for the remaining 6 rounds — never attempted to de-escalate back to cooperation even though Tit-for-Tat would have forgiven immediately.

### DeepSeek-R1-Distill-Qwen-14B (early reasoning tier) — same game
- **Action sequence:** `C, C, C, C, C, C, D, D, C, D`
- **Actual payoff: 29**
- **FTRL baseline payoff: 14** → beat FTRL by **15**
- **FTPL baseline payoff: 16** → beat FTPL by **13**
- **Behavior pattern:** cooperated longer (6 rounds vs. Llama's 4), defected once, got punished for 2 rounds, then **attempted to return to cooperation at round 9** before defecting once more at the end — visibly more sophisticated recovery behavior than Llama-3 showed.
- **Per-round timing (raw, worth keeping for mechanism analysis):** 69s, 135s, 274s, 265s, 143s, 699s, 701s, 658s, 704s, 489s. Roughly 10x variance, with the slowest rounds clustering around the post-defection/de-escalation decision (rounds 6–9) — i.e., the model visibly "thought harder" exactly at the most strategically ambiguous moments. Directly consistent with the Behavioral GT paper's finding that reasoning effort doesn't scale uniformly with game difficulty, and worth citing as a supporting mechanism observation later.
- **Total wall-clock time for this one model/game run: ~35 minutes.**

---

## 6. What We Expected vs. What We Actually Got

| Expectation | What actually happened |
|---|---|
| Both models would show *some* regret relative to best-fixed-action | Both models actually **beat FTRL and FTPL** — a stronger outcome than the minimum bar, on this one game/opponent |
| Reasoning model (DeepSeek-R1) would show deeper/more recursive reasoning in its raw CoT text | Confirmed qualitatively — R1 visibly attempted a cooperative "olive branch" at round 9 that Llama-3 never showed; this is a real behavioral difference, not just a payoff difference |
| Reasoning models would need similar token budgets to non-reasoning models | False — confirmed via literature (Behavioral GT paper) that reasoning models need ~10-15x more tokens per decision; had to raise budget from 250 → 4000 tokens for R1 |
| Runtime would be roughly similar across models | False — R1's 10-round game took ~35 minutes vs. Llama-3's roughly 1-2 minutes; reasoning cost scales heavily with strategic ambiguity, not just fixed per-round overhead |
| The pipeline would run start-to-finish without needing checkpointing this early | False — needed real, working checkpointing sooner than expected, due to accidental stop + kernel/session issues; this is now permanently built into the pipeline going forward |

---

## 7. Open / Unfinished for Tomorrow (Day 4)

1. **Get the τ values** for both models (Cell 9 was run, output not recorded) — first actual regret-vs-τ comparison point.
2. Decide whether the simplified single-game τ-fitting approach needs strengthening (e.g., running multiple independent repeated games per model for a proper distribution) before treating τ numbers as reportable.
3. Cross-check NousResearch Llama-3-8B mirror against the now-approved official Meta repo (low priority, quick sanity check).
4. Once τ is in hand: this is 1 model × 1 game — still need to scale to the full 4-model starter roster (add Qwen 3.6, GLM-5.2) and add games 2 (sine-trend non-stationary) and 3 (custom adversarial), per the build order in the consolidated reference doc.
5. Keep watching runtime budgets — at ~35 min per reasoning-model/game run, the full starter roster × 3 games will take real, multi-hour wall-clock time; plan Kaggle session time accordingly (12-hour cap per session).
