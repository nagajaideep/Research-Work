# When Does Forgetting Fix Bias? Corrective Unlearning under Clean-Label Spurious Correlations

![Status](https://img.shields.io/badge/Status-Experiments_in_progress-blue)
![Target](https://img.shields.io/badge/Target-TMLR-orange)
![Code](https://img.shields.io/badge/Code-Upon_Publication-lightgrey)

**Full research record, as of 18 Nov 2026.** This file is the current, consolidated state of the project: the question, the setup, every experiment and its numbers, what holds and what failed, the literature, and the plan to a paper. The day-by-day record, including refuted hypotheses and corrected numbers, is in [`research-log.md`](research-log.md). Literature notes and an independent referee report are in [`notes/`](notes/).

Working title: **"Deleting the Aligned Samples Is Not Debiasing: When Corrective Unlearning Can and Cannot Fix Spurious Correlations."**

---

## Contents

1. [The question](#1-the-question)
2. [The answer so far, in three sentences](#2-the-answer-so-far-in-three-sentences)
3. [Setup](#3-setup)
4. [Core concept: cue shortcuts vs skew shortcuts](#4-core-concept-cue-shortcuts-vs-skew-shortcuts)
5. [Results](#5-results)
6. [Status of every claim](#6-status-of-every-claim)
7. [Literature and novelty](#7-literature-and-novelty)
8. [Paper plan](#8-paper-plan)
9. [What is still required](#9-what-is-still-required)
10. [What failed, what was corrected](#10-what-failed-what-was-corrected)
11. [Compute and reproducibility](#11-compute-and-reproducibility)
12. [Project history](#12-project-history)

All numbers are **test worst-group accuracy (WGA, %)** unless stated. "Gain" is the paired difference against the same seed's unedited (ERM) model, with a 95% t-interval over seeds. α is the share of bias-aligned training samples that were discovered (flagged).

---

## 1. The question

A trained classifier relies on a spurious shortcut (a bird's background instead of the bird; a colour tint instead of the object). Labels are correct — the bias comes from which samples exist, not from label noise. Someone flags a share α of the training samples in which the shortcut and the label agree. This is the **corrective machine unlearning** setting of Goel et al. (TMLR 2024), applied to clean-label spurious bias.

**Can an unlearning method, told to forget the flagged samples, fix the model?** And how does that compare with the simple alternative of retraining only the last layer on a small group-balanced set (DFR, Kirichenko et al.)?

## 2. The answer so far, in three sentences

1. **Deleting flagged aligned samples does not debias models whose shortcut is a skewed attribute** (Waterbirds, CelebA, and synthetic analogues): approximate unlearning gives no gain at any discovery rate, and even exact retraining without the flagged samples improves only until the remaining data becomes unbiased and then collapses — at a point that a simple count-based diagnostic predicts in advance.
2. **For removable-cue shortcuts unlearning has a real effect**, but that effect cannot be found without conflict examples: with label-free tuning the gain disappears.
3. **Given a handful of conflict examples, unlearning followed by last-layer retraining beats last-layer retraining alone** at the same number of fitting labels, on cue shortcuts only (5 seeds, paired).

## 3. Setup

### Datasets

| Dataset | Task | Spurious attribute | Backbones | Seeds |
|---|---|---|---|---|
| Waterbirds | landbird vs waterbird | background (land / water) | ResNet-50 (ImageNet), ViT-B/16 (ImageNet) | 5 (R50), 3 (ViT) |
| CelebA | blond vs not blond | gender | ResNet-50 | 3–5 |
| Colored CIFAR-10, **cue** (presence) type | 10 classes | a class-specific colour tint on a share ρ of each class's training images; the rest are untinted | ResNet-9 (scratch), ResNet-18 (pretrained) | 3–5 |
| Colored CIFAR-10, **skew** (proportion) type | 10 classes | every image is tinted: share ρ with its class colour, the rest with another class's colour | ResNet-9, ResNet-18 | 3–5 |
| Two-colour CIFAR (cue and skew) | animal vs vehicle | two-valued colour cue | ResNet-18 | 3 |
| Cue/skew interpolation | 10 classes | skew CIFAR with a share of the non-aligned images left untinted (0 → 1) | ResNet-18 | 2 |
| Waterbirds, aligned-share sweep | as Waterbirds | aligned training images subsampled to 60 / 75 / 85% share | ResNet-50 | 3 |

CIFAR validation and test sets hold each image twice (aligned or clean copy, and a conflict-tinted copy).

### Methods

| Family | Methods |
|---|---|
| Forget-signal-only unlearning | SSD (Selective Synaptic Dampening; 40–70 hyper-parameter settings), SSD-head, SSD with a retain-only reference Fisher, NegGrad, NPO, NPO-head, NegGrad/NPO with retain fine-tuning, SCRUB, Bad Teacher, an FMD-style head Newton step (our approximation, **not** a faithful FMD), REM (official code, half-capacity model) |
| Uses the retain set | Fine-tune on retain; **retrain oracle** (exact unlearning: retrain from scratch without the flagged samples); **balanced oracle** (group-balanced retrain on all training data) |
| Last-layer retraining | DFR on training annotations; **held-out DFR** (DFR^Val_Tr); DFR with N conflict examples per group (N = 0…100); class-balanced last-layer retraining |
| Combination | SSD followed by DFR at N = 2/5/10/25 or the full held-out set ("stacking") |

### Metrics and diagnostics

Worst-group and average accuracy; an Association-Restoration-style test (amplify the class-conditional spurious direction in penultimate features, k = 2, 4); per-layer linear probes for the spurious attribute; SSD Fisher-ratio distributions; per-setting test metrics for model-selection sensitivity; the retain-set dependence diagnostic (Section 4).

### Protocol

Forget set = discovered bias-aligned samples. Hyper-parameters are chosen on validation WGA (group labels) unless a label-free rule is stated. From Run 5a on, the CIFAR held-out DFR split puts both copies of an image in the same half.

## 4. Core concept: cue shortcuts vs skew shortcuts

Let *a* be the spurious attribute, *F* the full set of aligned training samples and *R* the training data that remains after deleting the discovered part of *F*.

- **Cue shortcut:** *a* has a "not present" value (untinted, no patch, no watermark), and on the fully-deleted retain set the attribute carries no information about the label. Deleting *F* leaves unbiased data.
- **Skew shortcut:** every sample carries some value of *a* (every bird has a background, every face a gender); only the proportions are lopsided. Deleting *F* leaves the opposite bias.

**Diagnostic.** Compute the label–attribute association on the retain set (signed Cramér's V: positive = original bias, 0 = unbiased, negative = reversed) as a function of α. It needs only group counts.

| Dataset | α = 0 | 0.5 | 0.75 | 0.9 | 0.95 | 1 | Unbiased at |
|---|---|---|---|---|---|---|---|
| Waterbirds | +0.87 | +0.76 | +0.59 | +0.26 | −0.02 | −1.00 | α ≈ 0.95 |
| CelebA | +0.31 | +0.14 | 0.00 | −0.20 | −0.37 | −1.00 | α ≈ 0.75 |
| Skew CIFAR, 10 colours, ρ = 75% | +0.72 | +0.56 | +0.37 | +0.15 | +0.03 | −0.11 | α ≈ 0.96 |
| Skew CIFAR, 10 colours, ρ = 95% | +0.94 | +0.89 | +0.81 | +0.62 | +0.43 | −0.11 | α ≈ 0.99 |
| Skew CIFAR, 2 colours, ρ = 75% | +0.50 | +0.20 | −0.14 | −0.54 | −0.74 | −1.00 | α ≈ 0.67 |
| Cue CIFAR, ρ = 75% | +0.87 | +0.77 | +0.65 | +0.48 | +0.36 | 0.00 | only at 1 |
| Cue CIFAR, ρ = 95% | +0.97 | +0.95 | +0.91 | +0.81 | +0.70 | 0.00 | only at 1 |

(CelebA assumes the aligned groups are blond-female and non-blond-male; to be confirmed against the forget set used in code.)

## 5. Results

### 5.1 Natural data: last-layer retraining dominates, unlearning does nothing

**Waterbirds, ResNet-50, 5 seeds (ERM 74.1)**

| Method | Result |
|---|---|
| Held-out DFR | 88.7 ± 0.9 (gain +14.6 ± 1.5) |
| SSD, three grids up to 70 settings, all α | gains −0.4 to +1.0 (± ≈ 1.4) |
| SCRUB, α = 1 | +2.4 to +4.0 |
| NPO-fine, α = 1 | +4.7 ± 1.7 |
| NegGrad / NPO (full) | collapse (1–17) |
| FMD-style head step | −1.1 ± 4.3 |
| Fine-tune on retain, α = 1 | 85.6–87.7 (+11.5 to +13.5) |
| DFR with 5 / 10 conflict examples per group | 75.3 (+1.1 ± 4.3, a tie) / 81.2 |
| SSD then held-out DFR | 89.0 (vs 88.7, no meaningful stacking gain) |

**CelebA, ResNet-50 (ERM 46–47)**

| Method | Result |
|---|---|
| Held-out DFR | 87.6 ± 1.4 (+41.5 ± 4.9) |
| DFR with 2 / 5 conflict examples per group | 80.6 ± 3.9 / 82.2 ± 1.0 |
| SSD | +0.0 ± 0.0 (the tuner picks a no-op) |
| Bad Teacher | 1–5 |
| FMD-style | 58.7 ± 9.3 (+12.6 ± 11.1) |
| Fine-tune on retain, α = 0.1 / α = 1 | 59.0 (+12.9 ± 11.5) / 18.8 (−28.2 ± 3.3) |

**Waterbirds, ViT-B/16, 3 seeds (ERM 69.9).** Every ResNet-50 finding reproduces: held-out DFR 85.1 (+15.2 ± 4.9); balanced oracle 78.1; retrain oracle 80.2 at α = 0.9 then 24.6 at α = 1; fine-tune on retain α = 1 82.7 (+12.8 ± 4.9); SSD −0.1 / +2.3 / +0.2 at α = 0.1 / 0.9 / 1 (all n.s.); NPO-fine +3.4 to +7.0; SCRUB +1.7 to +3.7.

### 5.2 Prevalence is not the explanation

Waterbirds with the aligned training share cut from ≈ 95% to 60 / 75 / 85% (new ERM models, 3 seeds): ERM rises only to ≈ 80, held-out DFR stays at ≈ 89, and **SSD's gain stays within ± 1.6 of zero at every share and both α**. The retrain oracle at α = 1 is 13.9 at every share (the retain set is the same conflict-only set). NPO-fine matches DFR only at the 60% share (88.6 / 88.7); at 75–85% it gains +1.5 to +5.2.

### 5.3 Shortcut type decides whether unlearning works

SSD gain at α = 1 (5 seeds on most cells):

| Setting | ResNet-18 | ResNet-9 |
|---|---|---|
| Cue CIFAR ρ = 5% | +13.2 ± 8.6 | +9.1 ± 7.9 |
| Cue CIFAR ρ = 25% | +32.1 ± 7.1 (3 seeds) | +18.7 ± 6.3 (3) |
| Cue CIFAR ρ = 50% | +31.1 ± 8.8 (3) | +28.9 ± 18.5 (3) |
| Cue CIFAR ρ = 75% | +23.4 ± 7.1 | +37.9 ± 9.1 |
| Cue CIFAR ρ = 95% | +14.2 ± 7.9 | +6.1 ± 4.3 |
| Skew CIFAR ρ = 75% | +0.9 ± 2.2 | −0.1 ± 1.8 |
| Skew CIFAR ρ = 95% | +3.1 ± 5.2 | +3.4 ± 1.3 |
| Two-colour skew ρ = 95% (ERM 86.1, DFR +9.5) | +0.0 ± 0.1 | — |
| Two-colour cue ρ = 75% | +13.7 ± 16.4 (n.s.) | — |

On skew CIFAR SSD is null at every discovery rate α ∈ {0.1, 0.25, 0.5, 0.75, 0.9, 1}. On cue CIFAR SSD at α = 0.1 already gains +21 to +26 at ρ = 75%, where the retrain oracle at α = 0.1 gains ≈ 0 — SSD is doing something other than approximating retraining.

**Interpolation** (skew CIFAR ρ = 75% with a share of non-aligned images left cue-free; 2 seeds): SSD gain 0.0, +2.8, +3.8, +4.7, +19.6 for shares 0, 0.25, 0.5, 0.75, 1. It rises, but most of the gain appears only at the pure-cue end. The retrain oracle switches from "peak at 90%" to "best at 100%" between shares 0.5 and 0.75.

### 5.4 Exact unlearning fails on skew shortcuts, where the diagnostic says

Retrain-without-flagged (exact unlearning oracle) by discovery rate:

| Setting | α = 0.1 | 0.75 | 0.9 | 1 | Balanced oracle | Held-out DFR |
|---|---|---|---|---|---|---|
| Waterbirds R50 (ERM 74.1) | 75.0 | 78.3 | **80.2** (+6.1 ± 2.7) | **15.6** (−58.6 ± 4.3) | 81.6 (+7.5 ± 2.4) | 88.7 |
| Waterbirds ViT (ERM 69.9) | 71.3 | — | **80.2** (+10.2 ± 3.3) | **24.6** (−45.3 ± 5.0) | 78.1 | 85.1 |
| CelebA (ERM 47.0) | — | — | **30.4** (−16.7 ± 7.7) | **2.2** (−44.8 ± 7.8) | 72.4 (+25.4 ± 12.1) | 87.7 |
| Skew CIFAR R9 ρ = 75% (ERM 47.0) | 47.3 | 51.8 | **55.7** (+8.7 ± 4.0) | **23.7** (−23.3 ± 3.5) | 57.7 | 56.2 |
| Skew CIFAR R18 ρ = 75% | 60.2 | 66.0 | 69.9 (+9.6 ± 6.3) | 57.5 (−2.0 ± 3.7) | 68.1 (+8.6 ± 3.7) | 63.8 |
| Two-colour skew ρ = 75% (ERM 96.3) | — | — | −3.9 ± 0.3 | −96.3 ± 0.3 | +1.3 ± 0.1 | +0.5 ± 0.5 |
| Two-colour skew ρ = 95% (ERM 86.1) | — | — | +6.9 ± 0.9 | −85.8 | +7.4 ± 1.0 | +9.5 ± 0.9 |
| Cue CIFAR R18 ρ = 75% (ERM 8.1) | +1.4 | +6.5 | +13.5 | **+31.6 ± 1.9** (best) | — | — |
| Two-colour cue ρ = 75% (ERM 5.7) | — | — | +21.3 | **+65.2 ± 4.8** (best) | +1.2 | +89.9 |

- The oracle peaks near the diagnostic's unbiased point and collapses beyond it. On CelebA the unbiased point is at 75%, so the oracle is already below ERM at 90%; this "anomaly" is what the diagnostic predicts.
- **Advance prediction held:** two-colour skew at ρ = 75% (unbiased at 67%) was predicted to be hurt already at 90% before the run existed; measured −3.9 ± 0.3. At ρ = 95% (unbiased ≈ 95%) it was predicted to still help at 90%; measured +6.9 ± 0.9.
- On cue shortcuts the oracle is monotonic and best at full discovery, and the balanced oracle does nothing — the mirror image of skew shortcuts.
- The CelebA oracle pattern reproduced with an independently retrained base model (seed 2: 29.4 vs 30.6 at 90%, 2.3 vs 2.5 at 100%).

**Fine-tune on retain** at α = 1 (retain set = conflict examples) is the best non-DFR method on Waterbirds (85.6–87.7) and on skew CIFAR R18 (ρ = 95%: 57.6, +36.5 ± 5.8, vs held-out DFR 37.4), but **hurts on CelebA** (18.8, −28.2 ± 3.3). It is dataset-dependent and is not claimed as a method.

### 5.5 SSD's cue gain does not survive label-free tuning (Run 5a)

Gain over ERM at α = 1 by how the SSD setting is chosen ("label-free" = validation accuracy weighted by training group proportions; conflict images get zero weight):

| Setting | Val WGA (group labels) | Label-free | Median setting | Best on test |
|---|---|---|---|---|
| Cue R18 ρ = 5% | +14.7 ± 13.3 | +5.8 ± 13.0 | +0.7 | +14.7 |
| Cue R18 ρ = 25% | +32.3 ± 10.0 | +19.0 ± 8.0 | +10.9 | +32.5 |
| Cue R18 ρ = 50% (n = 2) | +37.7 | −0.2 | +6.4 | +37.7 |
| Cue R9 ρ = 5% | +7.1 ± 17.1 | +1.8 ± 6.4 | +0.3 | +9.2 |
| Cue R9 ρ = 25% | +23.3 ± 12.3 | +0.5 ± 2.2 | +1.1 | +23.3 |
| Cue R9 ρ = 50% (n = 2) | +38.8 | +5.8 | +6.2 | +38.8 |

Only one of six settings keeps a significant label-free gain. Group-aware selection is close to best-on-test, so the capability exists, but finding it needs conflict examples. On Waterbirds, unlearning methods also lose their small gains without group-labelled validation, while fine-tuning on retain at α = 1 is robust (every setting helps). Earlier "survives selection by plain accuracy" was wrong: that CIFAR validation set is half conflict copies.

### 5.6 Unlearning then DFR saves conflict labels on cue shortcuts (Run 4b)

5 seeds, α = 1, paired within the run, leakage-free split. Cells: DFR alone with N conflict examples per group, then stacking gain (SSD-then-DFR minus DFR alone):

| Setting | ERM | SSD alone | N = 2 | N = 5 | N = 10 | N = 25 | Full held-out |
|---|---|---|---|---|---|---|---|
| Cue R18 ρ = 25% | 18.4 | +27.0 ± 5.5 | 21.6, +17.0 ± 4.8 | 35.6, +12.8 ± 9.4 | 34.4, +18.5 ± 10.8 | 44.9, +12.2 ± 4.0 | 56.3, +5.7 ± 3.5 |
| Cue R18 ρ = 50% | 10.6 | +31.6 ± 6.8 | 18.3, +12.7 ± 6.4 | 28.3, +14.4 ± 9.2 | 35.2, +12.1 ± 8.0 | 39.5, +13.4 ± 6.1 | 53.5, +6.5 ± 2.4 |
| Cue R18 ρ = 75% | 8.0 | +21.3 ± 8.5 | 14.7, +8.9 ± 8.6 | 21.1, +10.2 ± 5.5 | 28.0, +8.9 ± 4.3 | 35.0, +8.5 ± 5.4 | 44.4, +6.8 ± 5.5 |
| Cue R9 ρ = 25% | 13.0 | +25.1 ± 10.2 | 22.4, +18.4 ± 4.1 | 26.4, +24.4 ± 10.5 | 30.9, +23.5 ± 10.7 | 40.8, +17.0 ± 5.3 | 54.1, +6.9 ± 7.2 |
| Cue R9 ρ = 50% | 3.1 | +33.1 ± 14.0 | 14.8, +24.2 ± 5.2 | 20.4, +24.5 ± 7.0 | 28.5, +24.3 ± 7.1 | 39.2, +17.3 ± 8.5 | 50.7, +8.8 ± 5.3 |
| Cue R9 ρ = 75% | 1.1 | +34.2 ± 6.3 | 8.5, +23.3 ± 8.1 | 17.6, +20.2 ± 3.6 | 25.5, +19.2 ± 3.6 | 33.4, +17.0 ± 4.7 | 44.6, +7.2 ± 2.9 |
| Skew R18 ρ = 75% (control) | 59.7 | +0.8 ± 1.3 | 38.6, +7.5 ± 18.7 | 51.0, +1.8 ± 13.1 | 54.0, +3.7 ± 8.6 | 56.3, +3.0 ± 10.4 | 61.5, +1.0 ± 2.0 |
| Skew R18 ρ = 95% (control) | 21.4 | +2.3 ± 2.9 | 18.9, +4.3 ± 8.2 | 23.1, +2.2 ± 6.7 | 27.6, −0.9 ± 11.0 | 29.1, +0.4 ± 8.0 | 37.0, +2.7 ± 3.7 |

Significant in all six cue settings at every limited budget (+8.5 to +24.5); never significant on the skew control. SSD + 2 conflict examples per group ≈ DFR alone with 25 per group (e.g. R9 ρ = 50%: 39.0 vs 39.2). **Caveat:** the SSD setting is still chosen with group-labelled validation, so the label count on the stacking side is understated. An equal-total-label-budget comparison is required.

### 5.7 Last-layer retraining needs held-out conflict data

DFR on the training annotations is weak; held-out DFR is strong (Waterbirds 88.7, CelebA 87.6). The number of conflict examples DFR needs (CIFAR, per-image split; mean of 3 seeds):

| Setting | N = 0 | 2 | 5 | 10 | 25 | 50 | 100 | Full held-out |
|---|---|---|---|---|---|---|---|---|
| Cue R18 ρ = 5% | 50.5 | 36.8 | 49.4 | 52.5 | 57.3 | 58.9 | 62.0 | 64.6 |
| Cue R18 ρ = 25% | 30.5 | 23.2 | 35.8 | 38.2 | 44.4 | 49.0 | 53.8 | 56.1 |
| Cue R9 ρ = 25% | 22.1 | 22.5 | 30.4 | 34.8 | 41.3 | 46.7 | 50.2 | 55.5 |
| Cue R9 ρ = 50% | 11.4 | 13.7 | 20.4 | 28.1 | 38.8 | 44.3 | 51.2 | 49.5 |

The validation-split leakage flagged by an external audit had no measurable effect (old vs new split within seed noise, no consistent sign).

### 5.8 Other findings (appendix-level)

- **Restoration test:** amplifying the spurious direction (k = 4) brings the shortcut back after unlearning. On Waterbirds every method that never saw conflict data falls to about ERM (unlearning 26–34, exact retraining at α ≤ 0.5 29–36, ERM 28.4), while held-out DFR keeps 83.0. Restorability therefore tracks access to conflict data, not unlearning as such. The basic "hides rather than removes" result replicates the published ART paper and is **not** claimed as ours.
- **Layer probes:** the spurious attribute stays linearly decodable (≈ 86–97%) at every ResNet stage before and after SSD.
- **SSD mechanism (Run 2e):** a retain-only reference Fisher does not rescue SSD at high prevalence; the Fisher-ratio tail does not shrink with prevalence.
- **REM (official code, half-capacity ResNet-9, paper hyper-parameters):** +7.6 / +9.6 / +13.4 / +8.5 / 0 over its own baseline on cue CIFAR at α = 0.1 across prevalence; fails at ρ = 95%. A limited comparison, not evidence against REM.
- **ρ = 0 control:** the tint itself degrades accuracy (conflict accuracy 68–76% vs 90% clean), so CIFAR healing is reported against the ρ = 0 ceiling.
- **Run-to-run noise on CelebA:** re-evaluating the same ERM models moves WGA by about ± 2 points; CelebA differences under ≈ 3 points are not interpreted.

## 6. Status of every claim

| # | Claim | Status |
|---|---|---|
| S1 | Unlearning is null on skew shortcuts (any α, any backbone) | **Holds** (Waterbirds R50 + ViT, CelebA, skew CIFAR 10- and 2-colour). Still to add: show settings did move the weights; best-on-test upper bounds |
| S2 | Prevalence is not the reason | **Holds**; size-matched control still to run |
| S3 | Exact retraining fails on skew shortcuts, where the diagnostic says | **Holds, with an advance prediction confirmed** |
| S4 | Unlearning gains on cue shortcuts | Holds with group-labelled tuning (+21 to +34, 5 seeds); two-colour cue n.s. with 3 seeds |
| S5 | The cue gain survives label-free tuning | **Fails** |
| S6 | The regime switch is driven by cue-free samples, not colour count | Partly supported; interpolation needs a third seed |
| S7 | Holds on a natural removable-cue dataset | Not started (ISIC colour patches planned) |
| S8 | A two-feature model explains both regimes | Not started |
| S9 | Closest baselines (Potion, tuned REM, CUPID, faithful FMD) do not already solve it | Not started |
| S10 | Unlearning then DFR saves conflict labels on cue shortcuts | **Confirmed with 5 seeds**; tuning labels not yet counted |

**Not claimed:** that unlearning is the wrong tool for bias in general; that we are first to study unlearning under bias or to show shortcuts survive unlearning; that DFR beating unlearning is our finding; that a balanced retrain reference is our idea; any comparison with FMD or REM based on our approximations; "removal" of a shortcut without a relearning attack.

## 7. Literature and novelty

Searched 17 Nov 2026 (arXiv, OpenReview, proceedings). Details were read through summaries; every number must be re-checked against the PDFs before citing. Full notes: [`notes/literature-unlearning-for-bias.md`](notes/literature-unlearning-for-bias.md), [`notes/literature-shortcuts-and-dfr.md`](notes/literature-shortcuts-and-dfr.md).

| Part of our claim | Closest prior work | Verdict |
|---|---|---|
| Unlearning removes a cue shortcut | Goel et al., *Corrective Machine Unlearning* (TMLR 2024, arXiv 2402.14015); Potion (arXiv 2406.09173); REM (ICLR 2026, arXiv 2505.17730) | **Anticipated** — our tint is a clean-label trigger; cite as origin |
| Unlearning with a sample forget set is null on Waterbirds / CelebA, and prevalence is not the reason | ART (arXiv 2607.05726) uses association-objective variants; Bias-Aware MU (ICCVW 2025, arXiv 2509.07456) contrasts "localised" vs "entrenched" bias informally | **Open** |
| Cue vs skew as one testable distinction | Yang et al., *Change is Hard* (ICML 2023); Hermann et al. (ICLR 2024); Le et al. (arXiv 2308.00473) | **Open, unnamed** — the main contribution |
| Exact retraining is the wrong reference | Goel and REM (partial-discovery failure, the opposite regime); Group-robust MU (TMLR 2025, arXiv 2503.09330) and ART use reweighted/balanced retrain | **Partly shown** — the peak-then-collapse curve and its diagnostic are new |
| Unlearning then last-layer retraining saves labels | FMD (NeurIPS 2023, arXiv 2310.12560); LaBonte et al. (NeurIPS 2023, arXiv 2309.08534) | **Open** |
| Unlearning hides rather than removes | ART | **Taken** — cite, do not claim |

**Closest competitor:** CUPID — *Easy to Learn, Yet Hard to Forget* (Kwon et al., arXiv 2602.21773, AAAI 2026 per listing). Class-wise forgetting on biased data (Waterbirds, BAR, Biased NICO++); goal is forgetting, not debiasing; keeps exact retraining as gold standard; skew-type datasets only. Must be read in full before writing.

**Theory to build on:** Nagarajan et al. (ICLR 2021) — their spurious feature takes a value on every sample (exactly a skew shortcut); removing all aligned data is the mirror image of their theorem. Also Sagawa et al. (ICML 2020), Chaudhuri et al. (ICML 2023), Idrissi et al. (CLeaR 2022), and Kirichenko et al. (features trained without minority data still support ≈ 89–90 with a retrained head).

**Reviewer expectations:** Waterbirds + CelebA + one more natural set (ISIC colour patches for cue; MultiNLI negation for text); baselines Retrain, NegGrad+, SCRUB, BadT, SalUn, SSD/Potion, NPO; GroupDRO, JTT, DFR, subsampling, D3M. Official code exists for REM, Potion, SSD, the corrective-unlearning benchmark and Group-robust MU; none found for CUPID or FMD.

## 8. Paper plan

**Venue judgement** (one independent referee, see [`notes/independent-referee-report.md`](notes/independent-referee-report.md)): TMLR likely (≈ 55–65% after the fixes); ICML/NeurIPS/ICLR main track unlikely as things stand (≈ 10%, ≈ 20% with a natural cue dataset and a mechanism); top-venue workshop very likely. Write for TMLR; run the natural-dataset test in parallel.

**Structure**

| Section | Content |
|---|---|
| 1. Introduction | When does forgetting flagged aligned samples fix a biased model? One figure; contributions: the cue/skew boundary with a testable diagnostic, the exact-retraining failure on skew shortcuts, the label-budget result on cue shortcuts |
| 2. Related work | Corrective unlearning; unlearning under bias; shortcut taxonomies; last-layer retraining |
| 3. Setting and definitions | Discovery rate, forget/retain sets, cue vs skew, the diagnostic, annotation accounting |
| 4. Protocol | Datasets, backbones, methods, seeds, leakage-free splits, label-free and fixed-budget tuning, both oracles |
| 5. Skew shortcuts: forgetting does not fix them | S1, S2 |
| 6. Even exact unlearning fails on skew shortcuts | S3, the diagnostic, both oracles vs DFR |
| 7. Cue shortcuts: forgetting works only with conflict labels | S4–S7, S10 |
| 8. Why: a two-feature model | S8 |
| 9. Decision rule | Compute the diagnostic; skew → collect a few conflict examples and retrain the last layer; cue → unlearn then retrain the last layer |
| 10. Limitations | Synthetic cue constructions, SSD-family focus, free-tier compute, re-implemented baselines |

**Figures:** (1) two panels, cue vs skew, WGA against discovery rate for SSD, retrain oracle, balanced oracle and DFR; (2) the diagnostic against α with the oracle peak marked; (3) interpolation; (4) tuning-sensitivity distributions with the label-free choice; (5) prevalence control.

## 9. What is still required

| Order | Experiment | Decides |
|---|---|---|
| 1 | Equal-**total**-label-budget comparison on cue CIFAR (tuning + fitting labels counted, e.g. 20 / 50 / 100 in all) | Whether the positive side survives a fair budget |
| 2 | Third seed for the interpolation; a two-colour skew row with more headroom; size-matched Waterbirds control | S6, S2 |
| 3 | ISIC with colour patches (natural cue dataset) | S7 — main track vs TMLR |
| 4 | Two-feature linear model; check that the collapsed oracle has intact features under a retrained head | S8 |
| 5 | Potion and tuned REM (public code); read CUPID and FMD fully and re-implement or justify omission | S9 |
| 6 | MultiNLI negation with BERT (or CivilComments) | Generality beyond images |
| 7 | Relearning attack, only if any sentence claims removal | — |

## 10. What failed, what was corrected

| Item | What happened | Current rule |
|---|---|---|
| First thesis: "unlearning is the wrong tool for bias" | Too broad; parts already published | Narrowed to the cue/skew boundary |
| "SSD's healing decays steeply with prevalence" (147% → 10% of the gap) | Partly a grid artefact: with 40 settings the decay is much flatter on ResNet-18 (114% → 23%) and non-monotone on ResNet-9; the Fisher-ratio mechanism proposed for it was refuted | Prevalence is not the headline variable; the shortcut type is (S2, S3) |
| "Prevalence explains the Waterbirds failure" (N2) | Refuted by the aligned-share sweep | Dropped |
| "Unlearning hides rather than removes" as a contribution | Already shown by ART | Cited, not claimed |
| "SSD gain survives plain-accuracy selection" | The CIFAR validation set is half conflict copies, so it was not label-free | Label-free = training-weighted accuracy; result fails (S5) |
| "DFR with 5 per group beats SSD on Waterbirds" | A tie (+1.1 ± 4.3) | Reported as a tie |
| Run 2a ResNet-9 | Missing ×0.125 logit scale; training diverged | Results void; fixed in Run 2c |
| Run 1 CelebA forget set | Non-blond women marked bias-aligned; NPO-head 82.8 was an artefact | align_thresh = 0.75; re-run (Run 2 CelebA) |
| CelebA comparator "58.7" | That number was fine-tune, not unlearning | Best unlearning-family is FMD-style 58.7; fine-tune 59.0 |
| N4 "balanced oracle passes ART" | Already in ART | Folded into S3 |
| CIFAR validation leakage | Clean and tinted copies of one image could fall in different DFR halves | Per-image split; no measurable effect |
| Multi-class DFR too slow | 35 min per unit | GPU FISTA L1 softmax regression (≈ 1 s) |
| Budget overshoot | Checked only between units | Launcher hard stop |

## 11. Compute and reproducibility

- All training on free Kaggle 2×T4 notebooks (about 30 GPU-hours per week per account) running one script with one worker per GPU, per-unit JSONL results and resumable checkpoints. One test job on Modal.
- Roughly 3,000 result units across 25+ runs (Runs 1, 2a–2k, 3a–3g, 4a–4b, 5a–5b; see the run table in [`research-log.md`](research-log.md)).
- Statistics: seed-paired differences against ERM, 95% t-intervals; 3 seeds on most CIFAR cells, 5 seeds on Waterbirds, CelebA and the headline CIFAR cells.
- Code, configs, seeds and raw results are withheld until publication, in line with this repository's release policy.

## 12. Project history

| Date (2026) | Milestone |
|---|---|
| 12–13 Nov | Run 1 (Waterbirds, CelebA, 9 methods × 5 α × 5 seeds); first literature positioning |
| 13–14 Nov | Runs 2a–2h: colored CIFAR prevalence sweep, fair DFR, SSD mechanism, conflict-count curve, exact-unlearning oracle, CelebA re-run |
| 14–15 Nov | Runs 2i–2k: wide SSD grids (40–70 settings), FMD-style baseline, SSD→DFR, REM |
| 15–16 Nov | Run 3a: proportion-type CIFAR — the cue/skew split appears; blueprint and independent reviews |
| 16 Nov | Runs 3b–3g: discovery sweeps, balanced oracle, aligned-share sweep, 5 seeds on weak cells, CelebA oracles; confidence intervals |
| 16–17 Nov | Run 4a (ViT Waterbirds); external audit; Run 5a (leakage fix, label-free selection); retain-set diagnostic; Run 4b (5-seed stacking); Run 5b (two-colour controls, interpolation); paper plan with literature search and independent referee |
| 17 Nov | All runs stopped; state frozen in this file |

---

*Author: Jaideep Chowdary. Questions or collaboration: open an issue with the `[collab]` prefix.*
