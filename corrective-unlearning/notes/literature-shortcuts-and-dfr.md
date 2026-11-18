# Literature notes: shortcut-learning side (17 Nov 2026)

Status tags: [V] = checked on the web this session (abstract or full text); [M] = from memory, NOT re-verified — check before citing.

## 1. Is presence-vs-proportion already named?

No paper found that names this distinction or shows a method working on one type and not the other. Nearest neighbours:

- **Yang, Zhang, Katabi, Ghassemi, "Change is Hard: A Closer Look at Subpopulation Shift", ICML 2023** (PMLR 202) [V] https://proceedings.mlr.press/v202/yang23s.html. Four shift types: spurious correlation (SC), attribute imbalance (AI), class imbalance (CI), attribute generalization (AG), from a decomposition of P(y|x) with an attribute term P(a|y,x_core)/P(a|x_core). Algorithms improve only SC and CI; DFR/CRT-style classifier rebalancing helps SC. Their attribute `a` is defined for every sample, so both our types fall inside "SC" — the split is orthogonal to theirs. Closest *precedent for the claim form* ("methods help on some shift types only"). They file MultiNLI-negation under AI.
- **Hermann, Mobahi, Fel, Mozer, "On the Foundations of Shortcut Learning", ICLR 2024** [V abstract; venue from ICLR listing] https://arxiv.org/abs/2310.16228. Predictivity (how well a feature indicates the label) vs availability (how easily it is extracted). Our ρ sweep is a predictivity sweep; presence/proportion is a third axis (support of the cue), not covered.
- **Bell, Bouchacourt, Sagun, "Reassessing the Validity of Spurious Correlations Benchmarks", arXiv 2409.04188 (2024)** [V abstract] — best methods on one benchmark do poorly on another; three validity desiderata; advice to pick the benchmark nearest one's problem. Supports "benchmark structure decides the winner" but no presence/proportion axis.
- **Li et al., "A Whac-A-Mole Dilemma: Shortcuts Come in Multiples Where Mitigating One Amplifies Others", CVPR 2023** [V] https://arxiv.org/abs/2212.04825. UrbanCars: every image has a background and a co-occurring object, each 0.95-correlated (proportion-type x2). ImageNet-W: watermark present in *some* carton training images (presence-type); mitigations amplify the other shortcut. Does not contrast the two types.
- **Le, Schlötterer, Seifert, "Is Last Layer Re-Training Truly Sufficient for Robustness to Spurious Correlations?", arXiv 2308.00473** [V] — the only paper found running one method (DFR) on a presence-type set (ISIC patches) and a proportion-type set (Waterbirds). ISIC worst group 64.4 -> 85.8 but benign-no-patch 94.3 -> 77.7; neurons still respond to the patch. No type claim.
- XAI "Clever Hans / data artifact" line (Rieger et al. below; Anders et al. ClArC [M]) treats presence-type cues only. Spawrious (Lynch et al., arXiv 2303.05470 [V]) splits one-to-one vs many-to-many — different axis; all images have a background.
- Survey: Ye et al., "Spurious Correlations in Machine Learning: A Survey", arXiv 2402.12715 [V] — taxonomy by method, none by cue type.
- Unlearning side, seen in passing: Kwon et al., "Easy to Learn, Yet Hard to Forget: Towards Robust Unlearning Under Bias" (CUPID; arXiv 2602.21773, AAAI 2026 per OJS listing) [V abstract] — Waterbirds, BAR, Biased NICO++, all proportion-type; no type distinction.

Suggested wording: "to our knowledge unnamed"; cite Yang et al. for the form of the claim, Hermann for the orthogonal axes, Le et al. as nearest empirical precedent.

## 2. Last-layer retraining

- **Kirichenko, Izmailov, Wilson, "Last Layer Re-Training is Sufficient for Robustness to Spurious Correlations", ICLR 2023** [V text; venue M] arXiv 2204.02937. WGA DFR^Val_Tr vs DFR^Tr_Tr: Waterbirds 92.9/90.2, CelebA 88.3/80.7, MultiNLI 74.7/71.5, CivilComments 70.1/58.0 — held-out data matters (matches our result 1). SUBG 89.1/85.6 on WB/CelebA. Feature extractor trained with NO minority groups, then DFR: WB 89.9, CelebA 89.0 (ERM-no-minority 31.9/21.7) — core features are learned even at 100% alignment; directly relevant to our α=1 retrain cliff (features survive, head inverts). Waterbirds DFR uses ~500 held-out examples. I could not locate a minority-count ablation in the text retrieved; do not cite one.
- **Izmailov, Kirichenko, Gruver, Wilson, "On Feature Learning in the Presence of Spurious Correlations", NeurIPS 2022** [V abstract] arXiv 2210.11369: ERM features match group-robust features; architecture and pre-training dominate.
- **LaBonte, Muthukumar, Kumar, "Towards Last-layer Retraining for Group Robustness with Fewer Annotations", NeurIPS 2023** [V] arXiv 2309.08534: class-balanced last-layer retraining with no group labels: WB 92.6, CelebA 73.7, CivilComments 80.4, MultiNLI 64.7; SELF 93.0/83.9/79.1/70.7 using as few as 20 held-out labels (<3%). Notes Waterbirds results with <10 worst-group samples are unreliable — consistent with our 5/group tie.
- **LaBonte, Hill, Zhang, Muthukumar, Kumar, "The Group Robustness is in the Details: Revisiting Finetuning under Spurious Correlations", NeurIPS 2024** [V] arXiv 2407.13957: subsetting hurts when a small minority group sits in the majority class; upsampling/upweighting collapse with long training.
- **Idrissi, Arjovsky, Pezeshki, Lopez-Paz, "Simple data balancing achieves competitive worst-group-accuracy", CLeaR 2022** [V] arXiv 2110.14503: subsampling/reweighting match SOTA; group labels matter most for model selection (our result 6).
- **Qiu, Potapczynski, Izmailov, Wilson, "Simple and Fast Group Robustness by Automatic Feature Reweighting", ICML 2023** (AFR) [title/venue V; authors M]. Bitrate-constrained DRO: not checked.

## 3. Benchmarks

| Benchmark | Type | Size / access |
|---|---|---|
| ISIC + colour patches (Rieger, Singh, Murdoch, Yu, ICML 2020, arXiv 1909.13584 [V]) | **Presence**: patches in ~50% of benign, 0% of malignant | 21,654 images (19,372 benign / 2,282 malignant); ISIC archive is free; Kaggle mirrors exist [M] |
| Hard ImageNet (Moayeri, Singla, Feizi, NeurIPS 2022 D&B [V page; authors M]) | Presence-like, graded; has object masks + spuriosity ranks | 15 classes, ~19k train / 750 test; GitHub/Box |
| ImageNet-W (Li et al.) | Presence (watermark in some carton images) | needs full ImageNet; too heavy for T4 |
| UrbanCars, Waterbirds, CelebA, Spawrious (~152k), MetaShift, bFFHQ, BAR, NICO [last four M] | Proportion | — |
| NIH/CheXpert chest drain (Oakden-Rayner et al., CHIL 2020 [M]) | Presence | drain labels not fully public [M] |
| MultiNLI negation (Sagawa et al. group-DRO split) | **Presence**: negation words in a minority of hypotheses | 206,175 train [M]; free |
| CivilComments-WILDS | **Presence**: identity mention present/absent | ~448k total [V]; original Jigsaw data is a Kaggle competition [M] |

**Recommend image: ISIC colour patches.** Only natural, cue-on-a-subset set that fits T4, has precedent with both explanation-penalty and DFR, and has patch masks (Rieger repo [M]). Caveat: the cue sits in one class only, so the conflict group (malignant+patch) must be synthesised by pasting patches (as Le et al. do). Fallback: Hard ImageNet.
**Recommend text: MultiNLI negation** (single lexical cue, standard 6 groups, DFR numbers above). Caveat: headroom is small (ERM ~66-67 [M] vs DFR 74.7) and Yang et al. label it attribute imbalance. CivilComments has more headroom (DFR^Tr 58.0 vs DFR^Val 70.1) but 8 overlapping identities and noisy labels.

## 4. Theory to build on

- **Nagarajan, Andreassen, Neyshabur, "Understanding the Failure Modes of Out-of-Distribution Generalization", ICLR 2021** [V] arXiv 2010.15775. Spurious feature takes +B or -B on EVERY sample, agreeing with y w.p. p — exactly our proportion-type. Geometric skew (max-margin uses the cue for the majority because the minority needs a smaller norm) and statistical skew (finite-time GD). Theorem 1: with no minority group the spurious weight has a positive lower bound. Removing all aligned data is the mirror image: cue becomes perfectly anti-correlated => reversed shortcut = our α=1 cliff; partial removal moves p toward 0.5 = our α=0.9 optimum.
- **Sagawa, Raghunathan, Koh, Liang, "An Investigation of Why Overparameterization Exacerbates Spurious Correlations", ICML 2020** [V abstract] arXiv 2005.04345: majority/minority ratio and spurious SNR; subsampling works where upweighting fails.
- **Chaudhuri, Ahuja, Arjovsky, Lopez-Paz, "Why does Throwing Away Data Improve Worst-Group Error?", ICML 2023** [V] arXiv 2205.11672: tails + extreme-value theory.
- **Puli, Zhang, Wald, Ranganath, "Don't blame Dataset Shift! Shortcut Learning due to Gradients and Cross Entropy", NeurIPS 2023** [V] arXiv 2308.12553: max-margin bias prefers the shortcut even when the stable feature is perfect.
- Shah et al., "The Pitfalls of Simplicity Bias in Neural Networks", NeurIPS 2020 [M, not checked].

**Proposed mechanism (ours, not in literature):** x = (core, s). Presence: s in {0, B·y}. Deleting aligned data leaves s ≡ 0 on the retain set: the cue weight gets no gradient and no sign flip (monotone oracle), and a forget-only importance score isolates it because the cue unit is silent on retain data. Proportion: s in {+B, -B}. The cue unit fires on every retain sample, so forget/retain importance ratio ~1 (SSD null), and deleting aligned data flips corr(s, y) to -1 (cliff). Prevalence ρ enters neither argument.
