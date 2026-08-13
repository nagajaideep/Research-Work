# Topic B — Complete Reference: "The Cost of Mixing Models"

*For a collaborator picking this up independently. Everything known about this direction, including the honest risk assessment, in one place.*

---

## 1. Main Working Topic (the shared umbrella)
> **No-Regret Methodologies in Multi-Agent Systems**

This is one of two sibling directions under the same umbrella topic. The other (Topic A — "Deep but Not No-Regret," reasoning-depth-vs-regret dissociation) is being actively built by Tamada. This document is Topic B — parked as a backup/alternate direction, available for a collaborator to take forward independently.

---

## 2. Title

> **"The Cost of Mixing Models: No-Regret Exploitability in Heterogeneous LLM Agent Populations"**

---

## 3. Core Research Question

Does the classical theoretical prediction that **no-regret learners are exploitable** (Deng, Schneider & Sivan — "Strategizing against no-regret learners") actually hold when tested empirically on real, heterogeneous LLM agent populations?

Specifically: **can a stronger LLM agent exploit a weaker one for above-equilibrium payoff, in a mixed population, measured via the formal regret metric** — and does this differ from a homogeneous population of same-capability agents?

This exact empirical test — regret-based exploitability across *heterogeneous* LLM populations — had not been done as of Aug 1, 2026, when this direction was locked. Update as of a later comparative pass (see Section 6 below): the empirical gap narrowed, though the specific formal-metric angle remains technically open.

---

## 4. Why This Was Chosen (original reasoning, Aug 1, 2026)

- No paper had tested this exact empirical question — genuinely open at the time, not an incremental ablation.
- The nearest neighbor identified then, **"When Agents Lie"** (July 2026), was read as measuring **deception/commitment-breaking**, not regret/exploitability — seen as a clean, defensible differentiation.
- Lower perceived risk than the sibling direction (Topic A) of the "just another ablation on reasoning models in games" criticism, since NeurIPS 2025's Behavioral Game Theory paper already covers reasoning-model benchmarking territory.

---

## 5. Design Plan (as originally locked)

- **Population design:** vary composition systematically — homogeneous strong, homogeneous weak, and mixed (strong+weak) — across the **same canonical games Park et al. (2025, "Do LLM Agents Have Regret?") used**, so their published regret numbers can be cited directly as a baseline.
- **Metric:** Regret vs. FTRL/FTPL (Park et al.'s formal definition) — **not** a payoff-gap or deception-framing metric. This is the core differentiator from "When Agents Lie."
- **Compute:** T4-feasible, no training required — same infrastructure philosophy as Topic A (open-weight models, no API dependency).
- **Model roster (not fully finalized):** implied mixed-capability pairs — 2-3 open-weight reasoning/non-reasoning pairs on T4, possibly 1-2 frontier models via API if budget allows. Less fully specified than Topic A's roster; this needs deciding before building.
- **Game suite:** reuse Park et al.'s canonical games (win-win, prisoner's dilemma, unfair, cyclic, biased, second-best — the same six from Robinson & Goforth 2005, Appendix B.4) rather than designing new games — no custom game-design layer was planned here, unlike Topic A.

---

## 6. The Risk Update — Read This Before Committing

This is the single most important thing to internalize before building on Topic B: **a later deep-research pass (done after the initial Aug 1 lock) found the overlap risk was underestimated.**

**"When Agents Lie: Premeditation, Persistence, and Exploitation in Repeated Games"** (Shi, Zhang, Schölkopf, Conitzer, Jin — arXiv:2607.05132, July 2026):
- **Won Best Paper at the 2026 ICML NExT-Game Workshop** — high visibility, strong author group (Conitzer, Schölkopf).
- Tests **three frontier models across six games in homogeneous AND heterogeneous groups over 10 rounds** — structurally almost identical experimental setup to what Topic B proposes.
- Has a named result called **"Heterogeneous Exploitation"**: mixing models causes systematic payoff asymmetries because models interpret public commitments incompatibly (some treat announcements as binding, others as cheap talk to exploit) — this gap **emerges in Round 0 and persists for all 10 rounds**.
- Their own related-work section explicitly stakes out the gap as: *"homogeneous-model evaluations miss the exploitation risks that arise when deployed systems combine models from different providers"* — i.e., nearly the exact territory Topic B was designed to claim as open.

**The technical differentiator that still holds:** their mechanism is *commitment/announcement semantics* (cheap talk vs. binding), not the *formal regret-vs-FTRL/FTPL metric*. The specific empirical test (regret computation under heterogeneity) remains, strictly, undone. But this is a **narrower gap than originally believed**, occupied by a recent, awarded, highly visible paper — a careful reviewer at AAMAS/NeurIPS will very likely cite it and ask "how is this different from When Agents Lie's heterogeneous exploitation finding?"

**Corroborating signal the space is now crowded:** ACM EC 2026 ran a full workshop — "Game Theory and Mechanism Design with Large Language Models" — built specifically around this heterogeneity-of-LLM-agents question. Confirms the general theme is mainstream-hot, not a hidden niche.

---

## 7. What a Collaborator Should Do Before Committing to This

1. **Read "When Agents Lie" in full, firsthand** (arXiv:2607.05132) — not secondhand from this document. Judge the overlap yourself; this is exactly the kind of call that shouldn't be inherited from someone else's summary.
2. **Re-run the literature search closer to actually starting** — this is a fast-moving space (weeks-to-months cycle), and the picture could have shifted further since this document was written.
3. **Write a half-page differentiation** against "When Agents Lie" specifically (regret/exploitability vs. deception/commitment) before writing any code — if this can't be stated crisply and defensibly, that's a signal to reconsider.

---

## 8. Target Venues (same as Topic A)

| Venue | Deadline | Notes |
|---|---|---|
| NeurIPS 2026 workshop | Rolling, through Aug–Sept 2026 | Faster dry-run option, smaller model-pair subset |
| **AAMAS 2027 main track** | "TBC," historically Oct 8–28 | Primary target — real, respected venue |
| ACM EC 2027 (future cycle) | Not yet announced | Given EC 2026 ran a directly relevant workshop, a 2027 edition may be a good fit if timing allows |

**Honest tier assessment:** workshop/AAMAS-tier, not ICML/NeurIPS/ICLR main-track tier — same honest framing as Topic A.

---

## 9. Side-by-Side vs. Topic A (for context on why Topic A was chosen instead)

| | Topic A (chosen, active) | Topic B (this document) |
|---|---|---|
| Core question | Does reasoning depth track regret across model generations? | Can a stronger LLM exploit a weaker one in a mixed population? |
| Nearest competitor | None found doing the exact dual-metric test | "When Agents Lie" (Best Paper, ICML NExT-Game 2026) — real overlap on the core empirical claim |
| Novelty risk | Low | Medium-High |
| Status | Chosen, actively being built (Prisoner's Dilemma + Cyclic done/in progress) | Parked — available for independent pickup |

---

## 10. Bottom Line for Your Friend

This is a **real, buildable, technically-still-differentiated idea** — not a dead end. The regret-vs-FTRL/FTPL angle on heterogeneous populations genuinely hasn't been done. But it now requires a sharper, more explicit differentiation argument against "When Agents Lie" than it would have needed on Aug 1, and that argument should be verified firsthand, not taken on faith from this summary. If the differentiation holds up under her own scrutiny, this is a legitimate parallel paper to Topic A, and the two would make a nice complementary pair (same umbrella topic, citing each other) for both your applications.
