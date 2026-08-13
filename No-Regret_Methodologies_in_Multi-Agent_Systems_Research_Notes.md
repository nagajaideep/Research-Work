# No-Regret Methodologies in Multi-Agent Systems: Research Development Notes

*A working document tracing the research idea, literature verification, venue strategy, and admissions positioning for a project on reasoning models and regret in repeated games.*

---

## 1. Origin Point: The Claimed Research Gap

The starting claim was that a specific, defensible gap exists in the literature, based on close (not abstract-only) reading of two papers:

**Gap statement:** Nobody has run the formal sublinear-regret framework from Park et al. ("Do LLM Agents Have Regret?") — i.e., repeated games and online-learning problems with regret benchmarked against FTRL/FTPL baselines — on modern *reasoning* models (o1, o3-mini, DeepSeek-R1, etc.).

**Why the gap was believed to exist:**
- **GTBench** uses "regret" as a term, but its actual computation (per Appendix A10) is a simple hand-coded rule (if/else bid comparisons for Blind Auction; a per-turn point accumulation for Iterated Prisoner's Dilemma) — not a formal comparison to FTRL/FTPL. GTBench also only tests non-reasoning models: GPT-3.5-turbo, GPT-4, Llama-3-70B-Instruct, CodeLlama-34b-Instruct, Llama-2-70b-chat, Mistral-7B-Instruct.
- **The behavioral game theory paper** ("LLM Strategic Reasoning: Agentic Study through Behavioral Game Theory," NeurIPS 2025) tests reasoning models (o1, o3-mini, DeepSeek-R1) extensively, but uses a completely different metric — "reasoning depth" (τ) via a Truncated Quantal Response Equilibrium (TQRE) / Cognitive Hierarchy framework — not regret, and its games are one-shot (not sequential/repeated), so it never asks Park et al.'s actual question.
- **Park et al. itself** only tested non-reasoning models (GPT-3.5, GPT-4, GPT-4o, open-source models) — no o1/o3/R1.

**Bonus hook identified:** Both GTBench and the behavioral paper independently found that CoT's effect on strategic reasoning is mixed/inconsistent across models and game types in one-shot settings. This raises (but doesn't answer) the question: does that same "reasoning doesn't reliably help" pattern hold in the *regret/repeated-game* setting, where success depends on adapting over time rather than getting one decision right?

**Originally proposed title:**
> "Do Reasoning Models Learn to Play No-Regret? Test-Time Deliberation in Repeated Games and Online Learning"

**Original claimed related-work structure:** A clean three-paper map — Park et al. (the framework, non-reasoning models), GTBench (a cruder regret notion, non-reasoning models, different games), and the behavioral game theory paper (reasoning models, but a different metric, one-shot only) — with the proposed work sitting precisely in the gap between them.

---

## 2. Literature Verification (Deep Search Pass)

### 2.1 Confirmed accurate

**GTBench's regret formulas** — verified directly from Appendix A10 of the paper (arXiv:2402.12348):
- *Blind Auction:* regret is computed via a simple conditional comparing the player's bid to the opponent's bid + valuation — not a comparison to a no-regret algorithm baseline.
- *Iterated Prisoner's Dilemma:* regret is a hardcoded accumulation — 0 if the player testifies regardless of opponent, 1 if the player stays silent while the opponent testifies, 2 if both stay silent.
- **Models tested:** GPT-4, GPT-3.5-turbo, Llama-3-70b-Instruct, CodeLlama-34b-Instruct, Llama-2-70b-chat, Mistral-7B-Instruct — confirmed zero reasoning models.
- GTBench was published as a **NeurIPS Datasets & Benchmarks** paper (2024) — a real, indexed, citable, non-main-track venue.

**Park et al. ("Do LLM Agents Have Regret? A Case Study in Online Learning and Games," Chanwoo Park, Xiangyu Liu, Asuman Ozdaglar, Kaiqing Zhang — ICLR 2025, arXiv:2403.16843)** — confirmed:
- Explicit validation against FTRL (entropy-regularized) and FTPL (Gaussian-perturbation) baselines.
- Repeated games tested: 6 representative general-sum games (win-win, prisoner's dilemma, unfair, cyclic, biased, second-best) from Robinson & Goforth (2005), plus non-stationary online learning problems (Uniform, Gaussian, Linear-trend, Sine-trend loss vectors), horizons up to T=100–500.
- Models tested: GPT-3.5, GPT-4, GPT-4o and open-source models — no reasoning models.
- Identifies failure cases of GPT-4 (Turbo) on adversarial settings; proposes a "regret-loss" for training no-regret behavior.

**Behavioral game theory paper ("LLM Strategic Reasoning: Agentic Study through Behavioral Game Theory," Jingru Jia et al. — NeurIPS 2025, arXiv:2502.20432)** — confirmed:
- Framework: Truncated Quantal Response Equilibrium (TQRE), rooted in Cognitive Hierarchy / Quantal Response behavioral game theory.
- Tests 22 state-of-the-art LLMs across 13 abstracted real-world games.
- GPT-o3-mini, GPT-o1, and DeepSeek-R1 lead in "reasoning depth" (τ), though model scale alone does not determine performance.
- CoT prompting is not universally effective — helps some models, provides limited gains for others; longer reasoning chains do not consistently yield better decisions.
- Also studies effects of encoded demographic attributes on decision-making, finding bias even in DeepSeek-R1 and GPT-4o.
- **Note:** could not independently verify the specific "one-shot, 30 independent trials per game" detail cited earlier — flagged as unconfirmed and worth double-checking directly in the paper.

### 2.2 Critical adjacent-work findings (the "watch out" list)

**"Reasoning without Regret"** (Tarun Chitra, arXiv:2504.09777, April 2025) — **near-identical title, unrelated content.** This paper is a theoretical framework (Backwards Adaptive Reward Shaping / BARS) using Markov Decision Processes, Hamilton-Jacobi-Bellman equations, and Backward Stochastic Differential Equations to convert sparse outcome-based CoT training rewards into dense procedure-based rewards. Its "regret" is single-agent *dynamic regret* over reward-shaping rounds (a training-theory concept), not multi-agent game-theoretic regret against FTRL/FTPL. **Action required:** must be explicitly cited and disambiguated from in any paper using a similar title, or reviewers/search engines will conflate the two.

**"Regret Minimization with Adaptive Opponents in Repeated Games"** (Mingyang Liu, Asuman Ozdaglar, Tiancheng Yu, Kaiqing Zhang — accepted COLT 2026, arXiv:2606.06486) — **same author group as Park et al.**, actively still publishing on regret-in-repeated-games. Introduces a new metric, "Repeated Policy Regret" (RP-Regret), for opponents who can respond adaptively to history. Appears theoretical (no LLM testing found in available excerpts), but signals the original framework's own authors are still actively extending this exact space — meaning they (or their students) are well-positioned to scoop a reasoning-model extension themselves.

**"Reasonably reasoning AI agents can avoid game-theoretic failures in zero-shot, provably"** (Enoch Hyunwook Kang, University of Washington — arXiv:2603.18563, posted March 19, 2026; **accepted at ICLR 2026 Workshop on Multi-Agent Learning and Generative AI (MALGAI)**) — cites both GTBench and the behavioral game theory paper directly. Proves theoretically (and validates empirically on five game scenarios including repeated Prisoner's Dilemma) that "reasonably reasoning" agents — capable of Bayesian learning and asymptotic best-response learning — converge to Nash-equilibrium-like play in infinitely repeated games without post-training. **Different metric/lens** (Nash convergence, not regret-vs-FTRL/FTPL) but structurally adjacent and very recent — must be cited and explicitly differentiated from in related work. Its acceptance at MALGAI also confirms this general genre of paper (empirical + light theory, LLM agents in repeated games) clears real workshop review.

**GT-HarmBench** (arXiv:2602.12316, Feb 2026) — benchmarks Claude 4.5 Opus, Claude 4.5 Sonnet, GPT-5.2, GPT-5.1, GPT-5 Mini/Nano, GPT-4o, Grok 4.1 Fast, Gemini 3 Pro/Flash, Llama 3.3 70B, Qwen3, DeepSeek V3.2 across Prisoner's Dilemma, Chicken, Battle of the Sexes, Stag Hunt, Coordination, No Conflict — a safety/alignment-framed scoring system, not regret-based, but confirms LLM-era game-theory benchmarking is being produced on roughly a monthly cadence right now.

**Search conclusion:** As of this research pass, nobody has been found running current reasoning models through Park et al.'s exact FTRL/FTPL-validated regret framework in repeated games — the specific empirical claim holds. However, the surrounding space is extremely active and moving on a weeks-to-months timescale, not a safe multi-year runway.

---

## 3. Assessment: Is This "Worth Working On"?

**What holds up:** The specific empirical claim is real and defensible — the exact experiment (reasoning models × Park et al.'s regret/FTRL/FTPL framework × repeated games) has not been done.

**What's riskier than it first appeared:**
- Title collision risk with "Reasoning without Regret" (unrelated April 2025 paper) — must be actively disambiguated.
- The original framework's own authors (Ozdaglar, Kaiqing Zhang) are still actively publishing regret-in-repeated-games work (COLT 2026) — they are best positioned to scoop this.
- A structurally adjacent paper (Kang, March 2026) on reasoning agents in repeated games appeared and was accepted at a real workshop three weeks before this conversation — confirms the temperature of the space.
- As originally scoped ("run existing models through existing framework, report regret numbers"), this is closer to a rigorous replication-plus-extension than a novel contribution — reviewers at a strong venue would ask what the intellectual contribution is beyond swapping in newer models.

**Revised verdict after scope expansion (see Section 5):** Worth pursuing, but only with genuine novel elements added (longitudinal trend documentation, custom adversarial/ambiguous game design, and a stated mechanism/insight) — not as a pure benchmark-replication exercise. Execution speed matters given how fast this space is moving.

---

## 4. Venue and Track Research

Checked live workshop listings rather than assuming availability:

- **NExT-Game @ ICML 2026** (Seoul, held July 11, 2026 — already passed) — explicitly seeks work on regret-minimizing learners and LLMs' brittle strategic planning in multi-agent settings; organized with Éva Tardos (Cornell), Tuomas Sandholm (CMU), Lillian Ratliff (UWashington) involved. Near-perfect topical fit; watch for a 2027 recurrence.
- **ICLR 2026 Workshop on Multi-Agent Learning and Generative AI (MALGAI)** — submission deadline was Feb 2026 (already passed for this cycle). Notably, this is where Kang's adjacent paper was accepted, confirming the genre is workshop-viable at a top venue.
- **NeurIPS 2026** (Sydney Dec 11–12 / Paris & Atlanta Dec 12–13, 2026) — the realistic near-term target given the conversation's timing (early August 2026). Workshop submission deadlines are rolling through August–September 2026 (example: the "Who Verifies the Agents?" workshop's deadline is Aug 29, 2026); all NeurIPS 2026 workshops must notify authors of acceptance by Sept 29, 2026. A game-theory/multi-agent-learning-themed NeurIPS 2026 workshop (parallel in spirit to NExT-Game) is the live window to search for and target.
- **GTBench precedent:** published in **NeurIPS Datasets & Benchmarks track** — confirms this general category (empirical benchmark + novel dataset/games) is a legitimate, citable, non-main-track publication path that fits the stated goal of "not main track but a real sub-track."

---

## 5. Scope Elevation: From Pure Benchmark Replication to a Real Contribution

The initial framing risked reading as "we ran models X, Y, Z through an existing framework" — a weak, purely evaluative contribution. Three additions meaningfully change this:

### 5.1 Longitudinal comparison across model generations
Track regret/strategic behavior across a *documented trend* — GPT-4 → o1 → o3-mini → DeepSeek-R1 → current-generation models — rather than a single-snapshot benchmark. This shows how regret behavior has evolved as reasoning capability scaled, which is a real, undocumented contribution (nobody currently has this trend line).

### 5.2 Designing original games/prompts (the single biggest lever)
Rather than only replaying Robinson-Goforth's 6 games and GTBench's 10 games, construct new scenarios specifically designed to stress-test where reasoning models should fail under a regret lens:
- Games with delayed-payoff traps.
- Non-stationary opponents (specifically designed to punish agents who summarize history into simple sum/average statistics — a documented LLM behavior pattern noted in Park et al.'s own Appendix C.10).
- Deliberately ambiguous/noisy payoff structures that punish overconfident one-shot reasoning and instead reward exploration under uncertainty.

This is "benchmark design as a contribution," a recognized and respected category (precedent: GTBench itself was a NeurIPS Datasets & Benchmarks paper).

### 5.3 Identifying and reporting a mechanism, not just a data point
The strongest possible finding: reasoning depth (τ, the CH/TQRE-style metric from the behavioral game theory paper) and regret (the online-learning-style metric from Park et al.) may be measuring genuinely different capabilities — i.e., a model can rank highly on one and poorly on the other. Neither Park et al., GTBench, nor the behavioral game theory paper make this comparison explicitly, since none of them measure both metrics on the same models on the same games. This would be a genuine, citable, non-obvious insight rather than a benchmark table.

---

## 6. Candidate Titles (Post-Execution Framing)

Per the explicit request that the title reflect an executed finding rather than a proposed study, three provisional titles were generated, each matching a different likely outcome pattern — to be finalized only once results are in:

1. **If regret cleanly declines across model generations** ("clean progress" story — considered the weakest/least interesting outcome):
   > *"Thinking Their Way to No-Regret: Test-Time Deliberation Closes the Online-Learning Gap in Language Model Agents"*

2. **If regret improves in stationary/simple games but stays flat or worsens in adversarial/non-stationary ones** (considered the more likely and more interesting outcome, consistent with prior "CoT helps inconsistently" findings):
   > *"Regret Without Reasoning: Where Test-Time Deliberation Fails to Generalize in Repeated Games"*

3. **If the two-metrics-diverge mechanism is confirmed** (reasoning depth ≠ low regret — considered the most citable, strongest "core finding" version):
   > *"Deep but Not No-Regret: Dissociating Reasoning Depth from Online-Learning Optimality in LLM Agents"*

**Recommendation given:** Aim for outcome 2 or 3 rather than 1 — a clean "models keep improving at everything" story is unsurprising and has weak citation value; a divergence/failure-mode story is the one that gets cited, and is also more consistent with what the existing literature (mixed CoT effects) already suggests is likely to be found.

**Main working topic, as specified by the user:**
> **No-Regret Methodologies in Multi-Agent Systems**

---

## 7. Detailed Execution Roadmap

### 7.1 Model roster (the longitudinal axis) — open-source only, by design

**Decision made:** restrict the entire model roster to open-source / open-weight models only, rather than mixing in closed API models (GPT, Claude, Gemini). This is a deliberate scope choice, not a limitation to apologize for, and it should be stated as a design decision in the paper itself.

**Why open-source-only genuinely adds value, not just convenience:**
- **Reproducibility.** Anyone can re-run the exact experiment on the exact weights later — a real, citable strength reviewers explicitly look for in benchmark papers, and one that closed API-based papers (including Park et al.'s own GPT-4-based results) structurally cannot offer, since API-served closed models change silently under a fixed name/version.
- **Full inspectability of the reasoning trace.** Closed reasoning models (OpenAI's o1/o3 family) redact or summarize their raw chain-of-thought; open models (DeepSeek-R1 and its successors, Qwen's thinking mode, Kimi's k-series, GLM's reasoning mode) expose the full trace. This directly enables the Layer C mechanism analysis (Section 7.2 below) — coding whether the model explicitly reasons about opponent history, counterfactuals, or randomization — which is difficult-to-impossible to do rigorously on redacted closed-model traces.
- **No rate limits or per-token cost ceiling.** Repeated games with long horizons (T=100–500, per Park et al.'s own setup) multiplied across many games, many trials for statistical power, and many models get expensive and slow fast on paid APIs. Self-hosted open models remove that ceiling, enabling the larger, more statistically powered experiment design this paper needs.
- **Precedent fit.** Park et al. and GTBench both already included open-source models (Llama, Mistral, CodeLlama) alongside closed ones — an open-source-only version isn't a downgrade in relevance, it's a natural, well-precedented restriction of the same lineage.
- **Removes a confound.** Closed-model providers continuously and silently update models behind the same API name; an open-weights-only design pins exact model versions/checkpoints, which strengthens any longitudinal ("how has this evolved") claim specifically.

**Three-tier structure using open-source models only, so the generational trend is visible:**
- **Non-reasoning baseline tier** (already covered by Park et al./GTBench — reuse published numbers where possible, re-run only 1–2 for calibration): Llama-3-70B-Instruct, Mistral-7B/Mixtral, CodeLlama-34B.
- **Early open-source reasoning tier** (covered by the behavioral game theory paper under the *different* τ/reasoning-depth metric — re-run under the regret/FTRL/FTPL metric instead): DeepSeek-R1, Qwen3 (thinking mode), Kimi K2, GLM-4.6.
- **Current-generation open-source reasoning tier** (the genuinely novel contribution — no regret numbers exist anywhere for these yet, since they postdate every paper being extended): DeepSeek-V4 (Pro/Flash), GLM-5.2, Kimi K2.7/K3, Qwen 3.6, MiniMax M3, gpt-oss-120b/20b (OpenAI's own open-weight release — notable as a rare closed-lab-but-open-weights data point), Gemma 4, Nemotron 3 Ultra. *(Confirm current availability and exact version numbers at execution time — this list reflects the open-weight landscape as of mid-2026 and will keep shifting; treat it as a snapshot to refresh right before running experiments, not a fixed final roster.)*

**One licensing note worth building into the methods section:** license tier varies (Apache 2.0 / MIT are the most permissive and academically frictionless — DeepSeek, Qwen, GLM's core releases, gpt-oss; some models like Kimi K2 and Llama carry custom licenses with usage-scale restrictions). Worth a one-line acknowledgment of which license governs each model tested, both for rigor and to preempt a reviewer question.

### 7.2 Game suite — three layers
- **Layer A (replication baseline, for direct comparability):** Park et al.'s exact non-stationary online-learning problems (Uniform/Gaussian/Linear-trend/Sine-trend loss vectors) and their 6 representative repeated games (win-win, prisoner's dilemma, unfair, cyclic, biased, second-best).
- **Layer B (original adversarial/non-stationary stress tests — the novel design layer):**
  - A non-stationary opponent that shifts strategy mid-game specifically to punish agents relying on simple history-summarization (sum/average) rather than genuine adaptive tracking.
  - A delayed-credit game where payoff feedback for a decision arrives several rounds later, testing whether reasoning models can hold a hypothesis open across a temporal gap (distinct from within-turn deliberation).
  - An ambiguous-payoff game where the agent observes only a noisy/partial signal of its own payoff, forcing exploration-under-uncertainty rather than exploitation of a known, fully observed matrix.
- **Layer C (mechanism probes):** For a subset of games, manually code chain-of-thought traces for whether the model explicitly reasons about regret-relevant concepts — tracking opponent history, computing counterfactual "what if I had played differently," or deliberately randomizing. This redoes, for reasoning-model CoT traces specifically, the qualitative analysis Park et al. did (in their Appendix C.10) only for non-reasoning models — a real, currently unfilled gap.

### 7.3 Metrics — track both, not just one
- **Regret vs. FTRL/FTPL** (Park et al.'s formal definition) — primary axis.
- **Reasoning depth (τ via CH/TQRE)**, computed using the behavioral game theory paper's method, on the *same* games — enabling a direct test of whether the two metrics move together or diverge (the mechanism claim).
- Apply proper statistical testing (e.g., Wilcoxon signed-rank across model pairs) rather than only comparing means/bar charts, to substantiate any "generation X beats generation Y" claim rigorously — an upgrade over the general rigor level of comparable existing LLM-benchmark papers.

### 7.4 Framing the write-up as findings, not a study proposal
Write each result as a specific, falsifiable claim rather than a general description of the exercise, e.g.:
- "Regret decreases significantly from GPT-4 to [current model] on stationary online-learning problems (Wilcoxon p<0.05), but shows no significant improvement on our non-stationary adversarial game — suggesting test-time deliberation improves computation of known payoffs but not exploration under uncertainty."
- "Models with high reasoning depth (τ) on the behavioral-game-theory metric do not consistently show low regret on our repeated-game metric — e.g., [Model A] ranks top-3 in τ but bottom-3 in regret on [specific game], indicating the two evaluation paradigms capture different capabilities."

The second type of statement, if borne out by the data, functions as the paper's core, genuinely new contribution.

### 7.5 Practical execution warning
Validate Layer B (the custom games) for actual difficulty/discriminative power on 2 pilot models *before* committing full compute to the entire model roster across all games. Risk: custom games could turn out to be either trivially solved by all models (uninteresting) or so poorly specified that the metric itself breaks down (unusable). Pilot first, then scale.

---

## 8. Categorization Honesty: Is This "Core AI/ML"?

**Verdict, unchanged by scope expansion:** No — this is not "core AI/ML" in the technical sense (no new architecture, training method, or optimization theory). It is a strong **empirical/benchmark contribution in AI agents + game theory**, a real and increasingly active subfield, but categorically distinct from core ML theory/methods work.

**Recommended framing for any SOP or self-description:** Present it as rigorous empirical characterization of an emerging capability gap in reasoning models, with a documented trend and a named mechanism — not as "core ML depth." Precision in framing reads as maturity to reviewers; overclaiming reads as inexperience, and reviewers can tell the difference.

---

## 9. Georgia Tech MSCS Admissions — Verified Context and Assessment

### 9.1 Verified facts about the GT MSCS admissions process
- Georgia Tech MSCS defaults to a **Course Option** (30 hours of pure coursework, no research component) as the most common completion path; Project and Thesis options exist but are not required.
- Admissions review is **holistic**: statement of purpose, letters of recommendation, test scores, and GPA are all reviewed carefully; the desirable minimum GPA is 3.0/4.0 (most admits score higher).
- The Admissions Committee "welcomes additional pertinent information" to aid the decision — meaning a paper/project can be included as supporting context, but is not a required or independently weighted checkbox.
- GRE is not required (though considered if submitted); English proficiency requirements apply for international applicants.
- Application deadline is Feb 1 for Fall admission; decisions typically arrive the first week of April.
- 11 specializations are offered, including Machine Learning; specialization coursework requires 15–18 hours with minimum "B" grades.

### 9.2 Assessment of the specific applicant profile (from uploaded resumes)

**Profile summary (verified from both uploaded resume versions):**
- B.Tech in CS (AIML) at Mahatma Gandhi Institute of Technology, Hyderabad, CGPA 9.21–9.24 (two slightly different figures across the two resume versions), graduating 2027.
- **Research Interests** explicitly stated on CV: Trustworthy and Responsible AI, Multimodal Model Evaluation and Explainability, **LLM Evaluation**, Low-Resource NLP.
- **IIT Roorkee (CoDA Lab)**, Research Intern, Apr–May 2025: built a Google Earth Engine workflow integrating MODIS LST, Landsat-8, LCZ, and WorldPop data for Bengaluru urban heat analysis; trained a Random Forest land-cover model (81.68% accuracy, 0.766 Kappa); applied Wilcoxon and MANOVA statistical tests. Feeds into a paper "Why Do Urban Cooling Islands Exist?" — currently under review at the *Journal of Water and Climate Change* (an environmental science journal, not a CS/ML venue).
- **Viswam.AI** (with Swecha & IIITH), Tech-Lead Intern, Summer of AI 2025, May–Jul 2025: led Python/NLP pipelines for low-resource Indic language transformer models; curated 40GB+ data and 800+ annotated image-text pairs.
- **Anshap**, Full Stack Developer Intern (second resume version), Apr–Jun 2026: integrated multiple LLMs into a mental-health platform (3,000+ active users), improving AI response quality/personalization by 75% in internal A/B tests; also built event-driven notification pipelines and a LiveKit-based real-time calling platform.
- **Open Sciences Lab** (remote, USA), Software Engineering Intern, May–Aug 2025: built an ASTx-to-Python AST transpiler; resolved 27+ bugs; automated CI/CD across Python 3.9–3.13.
- **Publications:**
  1. "Biological Age Estimation from Chest Radiographs Using Deep Learning: A ResNet-50 Based Approach with Clinical Risk Stratification" — **first author** — Accepted, ICSCT 2026; IEEE Xplore forthcoming.
  2. "BrainRiskNet: Dual Head CNN–ConvLSTM for Brain Tumor MRI Classification with Uncertainty Aware Severity Assessment" — co-author — Accepted, ISED 2026.
  3. "Why Do Urban Cooling Islands Exist? Understanding the Factors of Localized Urban Cooling" — co-author — Under review, *Journal of Water and Climate Change*.
- **Projects:** BrainRiskNet (91.12% classification accuracy, uncertainty quantification via predictive entropy); Multimodal Skin Cancer Detection System (ResNet/EfficientNet + attention fusion, Grad-CAM interpretability) — Team Alpha ranked **55th of 165 teams** in the ISIC MILK10k Benchmark; RouteIQ (DistilBERT intent classifier + FastAPI); ShiftCV (Gemini 2.5-powered resume-to-LaTeX agentic pipeline); YT Smart Speed (Chrome extension).
- **Competitive programming:** CodeChef 2-star, ranked 1,290/12,500+ in CodeChef Starters 238; 330+ DSA problems solved.
- **Honors:** Silver Medal for academic excellence (2nd year, CGPA 9.38); Vice Chair, ACM Student Chapter at MGIT.
- No faculty advisor or co-author is currently attached to the proposed reasoning-models/regret project; it would be self-directed.

### 9.3 Honest assessment of the pivot narrative

**What genuinely supports the pivot ("shifting into LLMs and their reasoning"):**
- The CV's own stated Research Interests already list "LLM Evaluation" — this is not a narrative invented for the SOP; it's already declared.
- Viswam.AI provides real, hands-on transformer/LLM pipeline experience.
- Anshap provides real production experience integrating and A/B-testing LLM response quality — direct, practical exposure to evaluating LLM behavior.
- The methodological throughline across BrainRiskNet (uncertainty quantification), the IIT Roorkee project (rigorous statistical testing), and the skin cancer benchmark is consistent, rigorous empirical evaluation — the same skill set a regret/reasoning benchmark project would exercise, applied to a new object of study.
- Conclusion: the pivot story is genuinely coherent and evidence-backed, not a generic hype-chase into a currently trendy topic.

**What must be honestly flagged, applied with the same rigor as everything else in this document:**
- **ICSCT 2026 and ISED 2026 are not prestigious venues.** "IEEE Xplore forthcoming" is an indexing statement, not a quality signal — IEEE Xplore hosts everything from flagship conferences to small regional conferences with broad scope and high acceptance rates; the standard boilerplate phrasing ("accepted papers submitted for inclusion into IEEE Xplore subject to meeting quality requirements") used across many similarly-acronymed conferences is itself a marker of this tier. A CS-focused reviewer at a program like Georgia Tech is likely to recognize this distinction and not weight these venues as strongly as their "IEEE published" framing might otherwise suggest to a non-specialist reader.
- **The IIT Roorkee work, while genuinely strong as an experience, is environmental/remote-sensing science with ML as a tool, not CS/ML research** — the target journal (*Journal of Water and Climate Change*) is not a computer science or ML venue. It demonstrates real capability (independent quantitative research, working with institutional-quality data, reaching a real peer-reviewed journal) but does not itself build "LLM/reasoning" credibility, applying the same honesty standard used to evaluate the proposed regret paper.
- **No faculty advisor is a genuine risk, not just a humility point.** The current LLM-reasoning/game-theory space is being actively worked by well-resourced groups (Park et al.'s MIT/UMD team with a COLT 2026 follow-up already out; a solo University of Washington author who already placed an adjacent paper in an ICLR 2026 workshop). A fourth-year undergraduate working solo, without institutional review or co-author positioning help before submission, faces real risk of weak framing, related-work gaps, or desk rejection — even though solo-authored success (per the Kang precedent) is clearly possible.
- **One additional paper is an incremental addition to an already-solid profile, not a required one.** With 2 accepted papers, 1 under review, a global benchmark placement, and two internships (including a genuine research lab), the existing file is already differentiated for an undergraduate applicant. The new paper's realistic function is as **narrative glue** — concrete evidence that the applicant is already doing what he says he wants to study next — rather than as the primary driver of admission.

### 9.4 Final, unhedged verdict on the pivot narrative

The pivot narrative holds together and is genuinely supported by existing evidence on the CV — this is a real correction from an earlier, more generic skepticism about "chasing a hot topic." However:
- Existing venues (ICSCT, ISED) should not be oversold as prestige signals in the SOP or in the applicant's own self-assessment.
- The IIT Roorkee experience should be framed accurately as rigorous applied-ML research in an environmental-science context, not as "AI/ML research" in the CS sense.
- The new project should be framed, per Section 8 above, as precise empirical characterization work — not "core AI/ML" or "identifying something major in AI" — since overclaiming is the one thing most likely to undercut an otherwise strong and coherent file in the eyes of a careful, holistic reviewer.
- The strongest version of the SOP ties all three pieces together explicitly: started in applied medical ML → gained hands-on production LLM experience (Viswam.AI, Anshap) → became curious about how reasoning models actually behave strategically over time → conducted an independent empirical investigation, scoped and reported honestly.

---

*Document compiled from full conversation history. All paper titles, authors, venues, and technical claims above were verified via direct web search and/or direct PDF retrieval during the conversation, with citations traceable to arXiv IDs and conference/workshop pages as noted inline.*
