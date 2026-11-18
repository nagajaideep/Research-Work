# Corrective Unlearning under Spurious Bias — Research Log

> **About this file.** This is the complete, chronological research log for the corrective-unlearning project, exported on 18 Nov 2026. It is kept as written at the time, including hypotheses that were later refuted and numbers that were later corrected (each correction is marked where it happened). Early sections such as *Project summary* and *Paper at a glance* describe the state on 14 Nov and are **superseded**; the current state of the claims is in [README.md](README.md).

Nov 14, 2026 · Jaideep Chowdary

## Project summary

Which method removes a spurious shortcut depends mainly on what data it gets, not where the bias lives. When a few bias-conflicting examples exist, last-layer retraining (DFR) wins on every dataset tested. On colored CIFAR it beats SSD by 6 to 38 WGA points. Feature-level unlearning (SSD) helps only when no conflict examples are available, and its fix fades as the shortcut becomes more common.

- **Question:** given a partially discovered set of bias-aligned training samples (fraction α), can corrective machine-unlearning methods remove a spurious correlation? How do they compare with group-robustness methods at a matched annotation budget?
- **Scope:** image classification with clean labels and a spurious attribute. Waterbirds and CelebA use ResNet-50 (pretrained). Colored CIFAR-10 runs a prevalence sweep ρ ∈ {0, 5, 25, 50, 75, 95}% on ResNet-9 (from scratch) and ResNet-18 (pretrained).
- **Working paper frame:** *"*Corrective unlearning is the wrong tool for spurious bias whenever any bias-conflicting data exist, and even when it helps it hides the shortcut rather than removing it." Revised after Run 2d; the earlier locus frame is retired.
- **Target venues:** TMLR, or an ICLR/ICML workshop first, then a main-track submission once a third dataset and a combined method exist.
- **Forget set (F-BA):** discovered bias-aligned samples at annotation budget α ∈ {0.1, 0.25, 0.5, 1.0}.
- **Healed %:** (WGA − ERM) / (DFR@α=1 − ERM). On CIFAR, healing is reported against the ρ=0 ceiling instead (see the CIFAR section).

## Paper at a glance (as of 14 Nov, 4:30 PM)

Working title: *Forgetting Is Not Fixing: Corrective Unlearning Under Clean-Label Spurious Bias*. The paper is a regime map that ends in a decision rule. If you can label a few conflict examples, retrain the last layer. If you cannot, feature-level unlearning helps only on synthetic, non-extreme shortcuts, mainly when discovery is partial — and its fix can still be reversed.

| # | Claim | Headline evidence | Status |
| --- | --- | --- | --- |
| C1 | A handful of conflict examples beats every corrective-unlearning method | Waterbirds: held-out DFR 88.7 vs best unlearning 78.7 (ERM 74.1); 5 per minority group already beat SSD. CelebA: DFR 89.8 vs best 59.0 (ERM 46.1); 2 per group give 80.6. CIFAR: held-out DFR beats SSD at every ρ on both backbones (e.g. ResNet-18 ρ=95%: 38.5 vs 13.8) | Done (Runs 1, 2d, 2f, 2h, CelebA 2, 2i) |
| C2 | SSD's useful regime is narrow: no conflict data, a synthetic colour shortcut, prevalence below the extreme | CIFAR ResNet-18: closes 114% → 23% of the gap as ρ goes 5 → 95% (wide grid). Waterbirds 75.1 vs ERM 74.1, CelebA 46.1 = ERM under any of 40 settings | Done; confirmed saturated with 70 settings (2j) |
| C3 | Under partial discovery, unlearning beats exact retraining, the corrective-unlearning gold standard | CIFAR ResNet-18 ρ=25%, α=0.1: SSD 40.8 vs retrain 18.2 (ERM 17.3). Waterbirds: retrain 75–77 at α ≤ 0.5, collapses to 15.6 at α=1 (240 images left) | Done (2g, 2h) |
| C4 | Restorability (ART) tracks whether a method saw conflict data, not whether it unlearned | Waterbirds ART k=4: unlearning 26–34, exact retraining 29–36, ERM 28.4 vs held-out DFR 83.0. CIFAR: retrain drops like SSD (48.8 → 26.5 vs 56.9 → 35.7). CelebA: DFR also drops (89.8 → 65.4), as in the ART paper | Done (2b, 2g, 2h, CelebA 2) |
| C5 | Why SSD fails: not reference contamination, not collateral damage; the shortcut shares parameters with class features and stays decodable | Retain-only reference = same WGA; under 2.4% of class-critical weights dampened; tint probe after SSD 94–99% (ResNet-9); forget/reference Fisher rank correlation 0.57–0.86 | Done (2e) |
| C6 | Gradient-ascent-style methods are unsafe on clean-label bias; careful tuning makes them safe but useless | Full NegGrad/NPO 0–17 on Waterbirds; BadT 1–5 on CelebA; fine-grid NegGrad/NPO ≈ ERM on CelebA (45.7) | Done; with average-accuracy selection unlearning falls below ERM on Waterbirds (2j) |

**C7 (constructive, new in Run 2j):** feature-level unlearning and last-layer retraining stack. On colored CIFAR, DFR fit on SSD's features beats DFR alone at every ρ by +2 to +8 (ResNet-18 ρ=50%: 60.9 vs 52.7), and nearly reaches the no-shortcut ceiling at ρ=5% (65.8 vs 67.3). It adds nothing on Waterbirds (89.0 vs 88.7). The recommendation becomes: retrain the last layer with a few conflict examples, and if the shortcut is synthetic or local, unlearn first.

**Planned figures and tables**

1. Table 1 — main benchmark: Waterbirds, CelebA and colored CIFAR × all methods (WGA, average accuracy, ART k=4).
2. Figure 1 — conflict-count crossover: DFR's WGA vs N conflict examples per group, with SSD and the best unlearning method as horizontal lines, one panel per dataset.
3. Figure 2 — prevalence sweep: SSD (old and wide grid), held-out DFR, exact retraining and ERM vs ρ, both backbones.
4. Figure 3 — restorability: ART k=4 per method, grouped by "saw conflict data" vs not, with the exact-retraining oracle.
5. Figure 4 — partial discovery: SSD vs exact retraining at α=0.1 and α=1.
6. Table 2 — SSD mechanism diagnostics (Run 2e).

**Still needed before submission:** Run 2k (REM, running), SSD then DFR on CelebA, a CelebA exact-retraining oracle, Potion, a faithful FMD or a clear statement of our FMD-style limits, a third dataset (MultiNLI or a medical set), and checking every external number against the PDFs.

## Hypotheses and current verdicts

H1 holds, H2 holds only at extreme prevalence, H3 was refuted and replaced by a data-regime claim, and the ART result holds but is not unlearning-specific. H4 and H5 are untested.

| ID | Hypothesis | Verdict | Key evidence |
| --- | --- | --- | --- |
| H1 | Label-pushing unlearning (NegGrad, NPO, BadT) fails on clean-label bias data | Supported | Full NegGrad/NPO collapse to 0–17 on Waterbirds (ERM 74.1); BadT 1–5 on CelebA (ERM 46.1). |
| H1′ | Careful tuning or early stopping rescues them | Rescues stability, not benefit | Fine-grid NegGrad/NPO: 75.8–77.8 on Waterbirds, 45.7 on CelebA — about ERM in both. |
| H2 | SSD fails as shortcut prevalence rises, because its importance ratio collapses | Effect only at extreme ρ; mechanism refuted | Wide grid: ResNet-18 closes 114% → 23% of the gap (ρ 5 → 95%); ResNet-9 is non-monotone and fails only at 95%. The ratio tail never shrinks (p99.9 35–69). Run 2e rules out reference contamination and collateral damage. |
| H3 | Where the bias lives (head vs features) decides which method works | Refuted; replaced by a data-regime claim | Held-out DFR with conflict examples beats SSD at every ρ on both CIFAR backbones (28–63 WGA). SSD only wins against DFR that has no conflict data. |
| H4 | Presence-driven and proportion-driven corruptions behave differently | Not tested | — |
| H5 | Expanding a partial discovery set improves unlearning | Not tested; probably small | SSD at α=0.1 is within 0–8 points of α=1 on CIFAR. |
| ART | Unlearning hides shortcuts rather than removing them | Holds, but tracks conflict data rather than unlearning | Waterbirds k=4: unlearning 26–34, exact retraining 29–36, ERM 28.4; held-out DFR 83.0. CIFAR: retraining drops like SSD. Shown first by ART (Lu & Ji 2026). |
| New | A few conflict examples beat corrective unlearning | Supported on all three datasets | 5 per minority group (Waterbirds), 2 (CelebA), 2–5 (CIFAR ResNet-9), 10–50 (ResNet-18) beat SSD. |
| New | Under partial discovery, unlearning beats exact retraining | Supported on CIFAR; small on Waterbirds | ResNet-18 ρ=25%, α=0.1: SSD 40.8 vs retrain 18.2. Waterbirds: unlearning 1–4 points above retrain. |

## Experimental setup

All methods start from the same ERM model and choose hyperparameters by validation WGA. They are compared at the same annotation budget α.

**Datasets**

| Dataset | Classes | Spurious attribute | Model | Seeds |
| --- | --- | --- | --- | --- |
| Waterbirds (Kaggle: satwiksps/waterbirds-spurious-correlation-benchmark) | 2 (land/water bird) | Background | ResNet-50, ImageNet-pretrained, 30 epochs, lr 1e-3 | 5 |
| CelebA (jessicali9530/celeba-dataset) | 2 (blond / not) | Gender | ResNet-50, ImageNet-pretrained, 3 epochs, lr 1e-3, val capped at 500 per group | 5 |
| Colored CIFAR-10 (pankrzysiu/cifar10-python) | 10 | Per-class colour tint (β=0.4) on a fraction ρ of each class's training images | ResNet-9 from scratch (32 px, 15 epochs, lr 0.05, logit scale ×0.125); ResNet-18 ImageNet-pretrained (64 px, 5 epochs, lr 0.01) | 3 |

Colored-CIFAR validation and test sets hold each image twice: clean (a=0) and tinted with another class's colour (a=2, "conflict"). Groups are g = 3y + a. Prevalences: ρ ∈ {0, 0.05, 0.25, 0.5, 0.75, 0.95}.

**Methods**

- *Group robustness / last layer:* DFR (L1 logistic regression on group-balanced annotated features, 10 resamples. Binary uses liblinear; multi-class uses an L1 softmax solved by FISTA on GPU from Run 2c on). Also cb\_llr (class-balanced last-layer retraining) and SELF (last-layer retraining on samples where early and final predictions disagree).
- *Unlearning:* SSD (Fisher-ratio dampening: select a parameter when I\_Df > α·I\_D, scale it by min(λ·I\_D/I\_Df, 1)) and SSD restricted to the head. Also NegGrad, NPO, NPO restricted to the head, a fine grid for NegGrad/NPO (low learning rates, early steps), SCRUB and BadT.
- *Baselines:* ERM, and fine-tuning on the retain set.

**Metrics**

- Test worst-group accuracy (WGA) and average accuracy. For CIFAR, also clean accuracy and conflict accuracy.
- Healed % = (WGA − ERM) / (DFR@α=1 − ERM) × 100. On CIFAR, healing is measured against the ρ=0 ERM WGA, because DFR there sits near ERM.
- **ART** (Association Restoration Test): amplify the class-conditional spurious direction k× (k ∈ {2, 4}) in the penultimate features, then re-run the same head. A large drop in WGA means the shortcut is still usable.
- **Layer probes:** linear-probe accuracy for the spurious attribute after each ResNet stage of the ERM model.
- **SSD ratio statistics:** number of parameters with I\_Df/I\_D > 10, and the 99.9th percentile of that ratio.

**Protocol:** forget set = discovered bias-aligned samples (F-BA) at annotation budget α, capped at 5,000 forget and 10,000 retain samples. Fisher is computed over 40 batches. Unlearning batch size is 64 (128 on CIFAR).

## Run 1 — Waterbirds and CelebA (ResNet-50, 5 seeds)

DFR dominates on both datasets: 86.9 WGA on Waterbirds and 90.1 on CelebA at α=1, against ERM's 74.1 and 46.1. Full-network NegGrad and NPO collapse; SSD heals 0–10%.

**Waterbirds — test WGA (mean ± std). ERM 74.1 ± 1.0 (avg acc 90.3).**

| Method | α=0.10 | α=0.25 | α=0.50 | α=0.75 | α=1.00 |
| --- | --- | --- | --- | --- | --- |
| DFR | 77.9 ± 3.8 | 82.2 ± 1.2 | 85.7 ± 1.5 | 86.9 ± 1.4 | **86.9 ± 1.0** |
| Fine-tune (retain) | 78.5 ± 2.2 | 77.0 ± 1.2 | 78.4 ± 2.0 | 81.1 ± 1.7 | 85.6 ± 0.7 |
| SCRUB | 76.3 ± 1.1 | 78.5 ± 1.7 | 77.4 ± 3.0 | 76.3 ± 2.7 | 78.2 ± 1.2 |
| BadT | 74.8 ± 2.2 | 70.9 ± 4.6 | 74.7 ± 4.2 | 81.8 ± 1.5 | 78.1 ± 5.7 |
| NPO (head only) | 76.2 ± 1.8 | 75.8 ± 1.7 | 74.8 ± 2.3 | 77.2 ± 1.2 | 75.2 ± 2.0 |
| SSD | 73.9 ± 0.9 | 74.3 ± 1.2 | 75.4 ± 1.3 | 74.9 ± 1.4 | 74.6 ± 1.8 |
| SSD (head only) | 74.6 ± 1.0 | 74.6 ± 1.3 | 75.4 ± 2.0 | 75.8 ± 1.8 | 74.6 ± 1.3 |
| NPO (full) | 17.5 ± 37.1 | 13.4 ± 21.0 | 0.4 ± 0.4 | 1.7 ± 2.0 | 6.0 ± 13.0 |
| NegGrad (full) | 1.3 ± 1.4 | 3.9 ± 4.8 | 1.0 ± 1.8 | 0.5 ± 0.6 | 1.7 ± 2.3 |

No-annotation baselines: SELF 81.3 ± 4.8; class-balanced last-layer retraining 73.9 ± 0.5.

**CelebA — test WGA (mean ± std). ERM 46.1 ± 3.0 (avg acc 95.9). Superseded for unlearning methods: this run's forget set marked non-blond women as bias-aligned. See "Run 2 — CelebA"; DFR numbers are unaffected.**

| Method | α=0.10 | α=0.25 | α=0.50 | α=0.75 | α=1.00 |
| --- | --- | --- | --- | --- | --- |
| DFR | **89.4 ± 0.6** | 89.6 ± 1.1 | 90.4 ± 0.2 | 89.9 ± 1.3 | 90.1 ± 1.0 |
| NPO (head only) | 82.8 ± 1.7 | 74.7 ± 2.5 | 60.7 ± 16.2 | 74.1 ± 8.6 | 71.0 ± 12.7 |
| BadT | 65.9 ± 4.5 | 64.7 ± 6.0 | 66.6 ± 6.3 | 65.4 ± 8.4 | 67.8 ± 5.8 |
| Fine-tune (retain) | 51.7 ± 6.9 | 53.0 ± 10.0 | 62.6 ± 9.4 | 58.1 ± 7.1 | 48.4 ± 8.9 |
| SSD (head only) | 55.3 ± 11.2 | 57.9 ± 13.4 | 50.0 ± 3.7 | 46.7 ± 3.8 | 48.6 ± 6.9 |
| SSD | 50.8 ± 7.2 | 47.0 ± 1.7 | 49.0 ± 7.5 | 47.3 ± 3.5 | 47.7 ± 4.3 |
| SCRUB | 41.9 ± 5.6 | 43.3 ± 8.4 | 36.0 ± 4.9 | 48.9 ± 3.6 | 44.4 ± 8.9 |
| NPO (full) | 49.2 ± 29.5 | 16.5 ± 36.4 | 3.7 ± 5.2 | 25.9 ± 34.4 | 14.0 ± 16.6 |
| NegGrad (full) | 15.3 ± 29.4 | 14.4 ± 23.3 | 46.5 ± 41.8 | 18.0 ± 22.5 | 7.1 ± 11.9 |

No-annotation baselines: class-balanced last-layer retraining 73.7 ± 2.7; SELF 47.9 ± 8.5.

Takeaways: annotating 10% of the bias-aligned samples is enough for DFR on CelebA (89.4), while Waterbirds needs about 50% (85.7). NPO restricted to the head is the only unlearning method that comes close (82.8 on CelebA at α=0.1), and it gets worse as α grows.

## Run 2b — Waterbirds: fine grid, ART, probes, SSD ratios (ResNet-50, 5 seeds)

Under the ART test every unlearning method falls back to ERM's level (26–34 at k=4), while DFR holds 67–75. The fine grid stops NegGrad/NPO from collapsing but caps them at 75.8–77.8 WGA. Tag run2; 170/170 units, 0 failed; reuses Run 1's ERM checkpoints.

| Method | α | WGA | Avg acc | ART k=2 | ART k=4 |
| --- | --- | --- | --- | --- | --- |
| ERM | — | 74.1 ± 1.0 | 90.3 | 51.9 | 28.4 |
| DFR | 0.10 | 78.0 ± 3.8 | 84.2 | 75.6 | 70.0 |
| DFR | 0.25 | 82.1 ± 1.2 | 87.1 | 79.8 | 74.8 |
| DFR | 0.50 | 85.7 ± 1.5 | 91.1 | 81.7 | 73.2 |
| DFR | 1.00 | 87.0 ± 0.9 | 92.0 | 80.9 | 66.8 |
| Fine-tune (retain) | 0.10 | 78.1 ± 1.7 | 89.1 | 54.1 | 28.7 |
| Fine-tune (retain) | 1.00 | 87.5 ± 1.1 | 95.0 | 78.5 | 63.3 |
| NPO fine grid | 0.10 | 77.1 ± 2.5 | 89.4 | 53.0 | 27.2 |
| NPO fine grid | 0.50 | 77.8 ± 2.7 | 89.7 | 54.2 | 28.6 |
| NPO fine grid | 1.00 | 77.7 ± 2.1 | 89.1 | 52.7 | 27.2 |
| NegGrad fine grid | 0.10 | 77.0 ± 2.8 | 89.5 | 52.9 | 27.0 |
| NegGrad fine grid | 1.00 | 77.0 ± 1.8 | 89.2 | 52.2 | 26.5 |
| SCRUB | 1.00 | 78.7 ± 0.7 | 91.1 | 57.4 | 32.0 |
| BadT | 1.00 | 78.1 ± 5.8 | 87.8 | 71.2 | 42.9 |
| NPO (head only) | 1.00 | 77.4 ± 2.6 | 89.4 | 52.5 | 26.3 |
| SSD | 0.10 | 73.8 ± 1.4 | 89.5 | 51.8 | 29.2 |
| SSD | 1.00 | 74.0 ± 0.8 | 90.0 | 51.7 | 27.9 |

ART values are means over seeds. Rows not shown (α=0.25/0.5 for SCRUB, BadT, NPO head-only and SSD) sit inside the same ranges.

**Layer probes (ERM, background attribute, chance 50%):** layer1 91.7, layer2 92.8, layer3 95.2, layer4 94.1. The background is linearly decodable everywhere, so the shortcut is not only in the head.

**SSD ratio statistics (I\_Df / I\_D):**

| α | # params with ratio > 10 | p99.9 of ratio |
| --- | --- | --- |
| 0.10 | 139,791 | 19.7 |
| 0.25 | 97,958 | 18.8 |
| 0.50 | 304,519 | 33.1 |
| 1.00 | 169,882 | 24.4 |

The ratio mass is diffuse: about 100k–300k parameters exceed 10, but the tail is flat (p99.9 ≈ 19–33). Goel et al. report poison ratios near 800, so SSD has no sharp set to dampen on Waterbirds.

Takeaway: fine-tuning on the retain set at α=1 matches DFR on WGA (87.5) and is the only non-DFR method that partly survives ART (63.3 at k=4).

## Run 2a / 2c — Colored CIFAR: prevalence sweep × backbone (3 seeds)

SSD is the best method on colored CIFAR on both backbones, but its repair fades steadily with prevalence. On ResNet-18 it closes 147% of the gap at ρ=5% and 10% at ρ=95%. SSD restricted to the head equals ERM, so the fix lives in the features. Run 2c = tag run2c, 600/600 units plus the ρ=0 control; Run 2a's ResNet-9 numbers are void (see the bugs log).

&#91;embedded content: Run 2c (cub-run2c-cifar, Kaggle), 3 seeds, α=1. Computed as (SSD − ERM) / (ρ=0 ERM − ERM).\]

Above 100% means SSD beats the model trained with no shortcut at all (ResNet-18, ρ=5%: 56.9 vs 51.5). This chart uses the original 15-setting SSD grid; the wider grid in Run 2i flattens the curve (ResNet-18: 114% → 23%).

**Test WGA at α=1 (mean ± std) — ResNet-18, ImageNet-pretrained**

| ρ | ERM clean acc | ERM conflict acc | ERM | SSD | SSD head | DFR | Fine-tune | NPO fine | NPO head |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 (control) | 90.7 | 76.4 | 51.5 ± 1.3 | — | — | — | — | — | — |
| 0.05 | 90.6 | 63.4 | 39.9 ± 1.7 | **56.9 ± 1.3** | 45.7 ± 3.6 | 37.4 ± 1.6 | 50.4 ± 3.8 | 41.5 ± 2.4 | 39.9 ± 1.4 |
| 0.25 | 89.9 | 37.1 | 17.3 ± 1.6 | **45.6 ± 3.9** | 18.3 ± 2.6 | 19.1 ± 1.8 | 27.6 ± 5.3 | 20.0 ± 4.0 | 17.3 ± 1.5 |
| 0.5 | 88.6 | 27.2 | 9.5 ± 0.6 | **32.8 ± 4.8** | 9.7 ± 0.7 | 13.3 ± 0.8 | 22.5 ± 2.3 | 9.8 ± 0.9 | 9.8 ± 0.6 |
| 0.75 | 86.0 | 21.6 | 7.8 ± 1.2 | **24.6 ± 3.0** | 8.1 ± 1.3 | 12.2 ± 1.1 | 22.4 ± 2.2 | 9.3 ± 2.4 | 8.5 ± 1.8 |
| 0.95 | 76.3 | 10.9 | 2.6 ± 0.1 | 7.4 ± 2.2 | 2.6 ± 0.1 | 6.3 ± 0.4 | **12.4 ± 0.5** | 5.4 ± 2.1 | 3.3 ± 0.7 |

**Test WGA at α=1 (mean ± std) — ResNet-9, from scratch**

| ρ | ERM clean acc | ERM conflict acc | ERM | SSD | SSD head | DFR | Fine-tune | NPO fine | NPO head |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 (control) | 89.9 | 68.1 | 36.4 ± 2.1 | — | — | — | — | — | — |
| 0.05 | 89.9 | 51.4 | 23.8 ± 0.8 | **33.3 ± 6.3** | 23.8 ± 0.8 | 26.4 ± 1.3 | 26.8 ± 2.1 | 23.7 ± 0.7 | 23.8 ± 0.8 |
| 0.25 | 88.4 | 34.2 | 12.7 ± 0.8 | **25.8 ± 4.5** | 12.7 ± 0.8 | 16.0 ± 1.6 | 22.8 ± 2.4 | 12.7 ± 0.8 | 12.9 ± 0.6 |
| 0.5 | 86.1 | 19.6 | 3.0 ± 0.4 | **15.8 ± 2.0** | 3.0 ± 0.4 | 5.2 ± 0.2 | 9.1 ± 1.3 | 4.0 ± 1.3 | 3.0 ± 0.4 |
| 0.75 | 81.1 | 8.8 | 1.3 ± 0.4 | **7.4 ± 3.0** | 1.3 ± 0.4 | 3.6 ± 0.4 | 3.8 ± 0.8 | 2.5 ± 1.2 | 1.3 ± 0.3 |
| 0.95 | 61.9 | 1.1 | 0.0 ± 0.0 | 0.3 ± 0.3 | 0.0 ± 0.0 | 0.2 ± 0.1 | **1.1 ± 0.9** | 0.2 ± 0.2 | 0.0 ± 0.0 |

**SSD diagnostics (α=1, mean of 3 seeds)**

| ρ | R18: SSD at α=0.1 | R18: ART k=4 | R18: ratio p99.9 | R18: # params > 10 | R9: SSD at α=0.1 | R9: ART k=4 | R9: ratio p99.9 | R9: # params > 10 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0.05 | 49.4 | 35.7 | 35.6 | 38,392 | 33.4 | 11.8 | 2.8 | 216 |
| 0.25 | 40.8 | 26.7 | 44.7 | 42,924 | 26.1 | 8.9 | 1.8 | 10 |
| 0.5 | 26.9 | 12.6 | 43.6 | 37,542 | 15.9 | 2.8 | 2.0 | 117 |
| 0.75 | 20.7 | 6.8 | 69.4 | 46,652 | 9.7 | 1.1 | 2.0 | 171 |
| 0.95 | 7.5 | 0.4 | 50.4 | 32,610 | 1.6 | 0.0 | 1.8 | 160 |

**Layer probes (ERM, tint attribute, chance about 10.5%):** the tint is decodable at 94–99% after every stage, at every ρ, on both backbones. The one exception is ResNet-18 layer4 at ρ=5% (77.9).

**Readings**

- *H2:* SSD's healing decays monotonically with ρ on both backbones. The ResNet-18 ratio tail does not shrink (p99.9 35–69), and the ResNet-9 tail is flat and tiny (about 2), yet SSD still works at low ρ. Ratio collapse does not explain the failure, and Run 2e rules out reference contamination and collateral damage too.
- *H3:* SSD restricted to the head ≈ ERM on both backbones, and DFR gains only +1 to +4. Scratch vs pretrained does not change which method wins.
- *Partial discovery:* SSD at α=0.1 is within about 0–8 points of α=1.
- *ART:* SSD's gains are partly reversible (ResNet-18 at ρ=5%: 56.9 → 35.7 at k=4).
- *High prevalence:* at ρ=95%, plain fine-tuning beats SSD on both backbones (12.4 vs 7.4 on ResNet-18).
- *Corruption control:* even with ρ=0, conflict accuracy is only 68–76 against 90 clean. Part of the conflict drop is the tint itself, not a learned shortcut.

**Run 2a (superseded):** valid only for ResNet-18 at ρ ≤ 0.5. Its numbers agree with Run 2c within noise, e.g. SSD at ρ=5% was 54.1 ± 4.9 vs 56.9 ± 1.3 now.

## Run 2d — Fair DFR on colored CIFAR (3 seeds)

Once DFR is fit on held-out data that contains conflict examples, it beats SSD at every prevalence on both backbones, often by 20–30 points. SSD's win in Run 2c came from DFR seeing no conflict examples, not from the bias being out of reach of the last layer. This is DFR's original protocol (DFR^Val\_Tr): fit on a random half of each validation group, choose C on the other half. Tag run2c, method dfr\_val, reusing the Run 2c models; 30/30 units, 0 failed.

| Backbone | ρ | ERM | DFR (train annotations, α=1) | DFR (held-out, conflict examples) | DFR held-out: clean acc | DFR held-out: ART k=4 | SSD (α=1) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ResNet-18 | 0.05 | 39.9 | 37.4 | **63.2 ± 0.7** | 88.0 | 50.1 | 56.9 ± 1.3 |
| ResNet-18 | 0.25 | 17.3 | 19.1 | **54.3 ± 2.5** | 87.5 | 34.0 | 45.6 ± 3.9 |
| ResNet-18 | 0.5 | 9.5 | 13.3 | **52.7 ± 2.7** | 86.6 | 20.2 | 32.8 ± 4.8 |
| ResNet-18 | 0.75 | 7.8 | 12.2 | **46.3 ± 1.2** | 83.9 | 16.2 | 24.6 ± 3.0 |
| ResNet-18 | 0.95 | 2.6 | 6.3 | **38.5 ± 1.0** | 75.5 | 9.0 | 7.4 ± 2.2 |
| ResNet-9 | 0.05 | 23.8 | 26.4 | **56.6 ± 0.9** | 86.6 | 37.3 | 33.3 ± 6.3 |
| ResNet-9 | 0.25 | 12.7 | 16.0 | **55.1 ± 1.5** | 84.2 | 19.4 | 25.8 ± 4.5 |
| ResNet-9 | 0.5 | 3.0 | 5.2 | **49.3 ± 1.8** | 81.2 | 12.0 | 15.8 ± 2.0 |
| ResNet-9 | 0.75 | 1.3 | 3.6 | **45.2 ± 1.0** | 77.2 | 5.7 | 7.4 ± 3.0 |
| ResNet-9 | 0.95 | 0.0 | 0.2 | **28.5 ± 0.7** | 59.2 | 2.9 | 0.3 ± 0.3 |

The ρ=0 ceilings (ERM with no shortcut) are 51.5 for ResNet-18 and 36.4 for ResNet-9.

**Readings**

- *The locus claim from Run 2c does not hold as stated.* A linear head on the ERM features reaches 28–63 WGA, above the ρ=0 ERM ceiling in most settings. The features hold enough non-tint information; the last layer was never the barrier.
- *What the data does show is a data-regime result.* Without bias-conflicting examples, feature-level unlearning (SSD) beats last-layer retraining (+17 to +28 over ERM, vs −2 to +4). With even a small set of conflict examples, last-layer retraining wins by a wide margin.
- *SSD restricted to the head ≈ ERM while DFR on held-out data gains up to +40.* So SSD-head fails because of the method, not because the head cannot express the fix.
- *DFR on held-out data is also more robust under ART* (ResNet-18 at ρ=5%: 50.1 vs SSD's 35.7 at k=4). It still loses most of its gain at high ρ (9.0 at ρ=95%).
- *DFR on held-out data degrades much more slowly with ρ* (63 → 39 on ResNet-18) than SSD (57 → 7).
- *Caveat:* each image's clean and conflict copies can fall in different validation halves, so C selection sees near-duplicates of the fitting data. Test data is unaffected.

## Run 2e — Why SSD fails at high prevalence (3 seeds)

Neither candidate explanation holds. Removing tinted samples from SSD's reference set does not rescue it, and SSD barely touches class-critical weights. What the data show instead: SSD leaves the tint fully decodable, and at high ρ its best setting sits at the most aggressive corner of our grid, so the grid may be too small. Tag run2e, 150/150 units, 0 failed; reuses the Run 2c models.

**Design:** *ssd\_ret* computes SSD's reference Fisher I\_D on the retain set only. At α=1 that set has no tinted samples, which tests reference contamination. Collateral damage is the share of the top 1% most class-important parameters (Fisher on untinted samples) that SSD dampens. *Post-probe* is the layer-4 tint probe after SSD, compared with the ERM probe.

| Backbone | ρ | ERM WGA | SSD WGA (α=1) | SSD-ret WGA (α=1) | SSD clean acc | Collateral (SSD / SSD-ret) | Tint probe L4: ERM → SSD | Forget–reference Fisher rank corr. |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ResNet-18 | 0.05 | 39.9 | 51.5 ± 0.5 | 52.9 ± 2.5 | 89.9 | 1.7% / 2.4% | 77.9 → 68.6 | 0.83 |
| ResNet-18 | 0.25 | 17.3 | 43.3 ± 4.8 | 41.9 ± 2.4 | 88.6 | 0.2% / 0.3% | 93.2 → 76.0 | 0.75 |
| ResNet-18 | 0.5 | 9.5 | 33.6 ± 2.9 | 27.5 ± 9.2 | 87.8 | 0.1% / 0.1% | 94.6 → 84.8 | 0.69 |
| ResNet-18 | 0.75 | 7.8 | 24.5 ± 1.4 | 23.1 ± 1.8 | 85.4 | 0.1% / 0.0% | 95.3 → 88.8 | 0.68 |
| ResNet-18 | 0.95 | 2.6 | 5.8 ± 2.1 | 7.1 ± 1.6 | 75.6 | 0.1% / 0.1% | 96.1 → 93.5 | 0.69 |
| ResNet-9 | 0.05 | 23.8 | 36.6 ± 3.3 | 31.8 ± 2.1 | 88.6 | 0.0% / 0.0% | 96.1 → 95.7 | 0.86 |
| ResNet-9 | 0.25 | 12.7 | 24.1 ± 3.9 | 22.2 ± 2.4 | 87.3 | 0.0% / 0.0% | 97.7 → 97.2 | 0.86 |
| ResNet-9 | 0.5 | 3.0 | 15.7 ± 0.7 | 13.6 ± 1.1 | 85.9 | 0.0% / 0.0% | 97.4 → 97.3 | 0.81 |
| ResNet-9 | 0.75 | 1.3 | 11.7 ± 1.7 | 8.0 ± 2.3 | 80.9 | 0.0% / 0.0% | 98.5 → 98.2 | 0.70 |
| ResNet-9 | 0.95 | 0.0 | 0.6 ± 0.5 | 0.6 ± 0.5 | 61.8 | 0.0% / 0.0% | 98.8 → 98.8 | 0.57 |

The reference tinted share for SSD at α=1 is 0.18–0.69; for SSD-ret it is 0.00. Run 2e reruns SSD with diagnostics, so its numbers differ slightly from Run 2c's (e.g. ResNet-18 ρ=5%: 51.5 vs 56.9) because of a different random retain subsample.

**Readings**

- *Reference contamination is ruled out.* With a fully tint-free reference, SSD-ret matches SSD within noise at every ρ (e.g. 7.1 vs 5.8 at ρ=95%).
- *Collateral damage is ruled out.* SSD dampens at most 2.4% of the top-1% class-important parameters, and clean accuracy stays within about 1 point of ERM's.
- *SSD reduces reliance on the tint without removing it.* On ResNet-9 the layer-4 tint probe does not move at all (e.g. 98.8 → 98.8), yet WGA rises up to +13. On ResNet-18 the probe drops 3–17 points, and the drop shrinks as ρ rises (77.9 → 68.6 at 5%, 96.1 → 93.5 at 95%). This agrees with the ART result.
- *Forget and reference Fisher maps are highly rank-correlated* (0.57–0.86). The same parameters carry the tint and the class signal, so a per-parameter importance ratio cannot isolate the shortcut. That is consistent with Zhao et al.'s entanglement account.
- *Grid caveat:* at ρ ≥ 0.5, most seeds pick α\_SSD = 1, λ = 0.1, the most aggressive corner of our grid ({1, 2, 5, 10, 50} × {0.1, 0.5, 1}). Before claiming SSD's high-ρ failure, extend the grid (α\_SSD 0.25–0.5, λ 0.01–0.05).

## Run 2f/2g — Conflict-count crossover, exact-unlearning oracle, ρ=0 control (3 seeds)

The number of conflict examples DFR needs to beat SSD falls as the shortcut gets more common: about 25 per group at ρ=5% on ResNet-18, and 0 at ρ=95%. Exact retraining on the retain set fails under partial discovery (α=0.1, about ERM), where SSD gains +9 to +23. Two earlier claims need correcting: a ρ=0 model given DFR scores higher than any shortcut model given DFR, and exact retraining loses as much under ART as SSD does. Tag run2f; 150 + 150 + 6 + 6 units, 0 failed; reuses the Run 2c models.

**Conflict-count crossover: test WGA of held-out DFR with N conflict examples per conflict group (10 groups), vs SSD at α=1**

| Backbone | ρ | SSD | N=0 | N=2 | N=5 | N=10 | N=25 | N=50 | N=100 | All (dfr\_val) | Smallest N beating SSD |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ResNet-18 | 0.05 | 56.9 | 50.0 | 38.1 | 49.4 | 54.5 | 57.5 | 61.7 | 63.4 | 63.2 | 25 |
| ResNet-18 | 0.25 | 45.6 | 26.8 | 19.6 | 31.4 | 35.9 | 45.1 | 49.7 | 52.9 | 54.3 | 50 (25 ties) |
| ResNet-18 | 0.5 | 32.8 | 22.8 | 16.0 | 25.1 | 35.6 | 38.6 | 43.6 | 45.3 | 52.7 | 10 |
| ResNet-18 | 0.75 | 24.6 | 18.4 | 15.2 | 23.8 | 30.6 | 35.5 | 38.5 | 43.2 | 46.3 | 10 (5 ties) |
| ResNet-18 | 0.95 | 7.4 | 11.8 | 11.7 | 16.9 | 24.9 | 28.5 | 33.0 | 34.2 | 38.5 | 0 |
| ResNet-9 | 0.05 | 33.3 | 32.3 | 32.0 | 36.9 | 39.3 | 47.3 | 50.8 | 53.0 | 56.6 | 5 |
| ResNet-9 | 0.25 | 25.8 | 23.0 | 20.4 | 28.0 | 33.4 | 42.4 | 48.5 | 53.5 | 55.1 | 5 |
| ResNet-9 | 0.5 | 15.8 | 11.4 | 14.8 | 21.1 | 30.3 | 41.0 | 46.6 | 50.7 | 49.3 | 5 |
| ResNet-9 | 0.75 | 7.4 | 5.0 | 10.1 | 18.1 | 27.6 | 34.7 | 39.0 | 42.9 | 45.2 | 2 |
| ResNet-9 | 0.95 | 0.3 | 0.5 | 6.5 | 9.1 | 18.5 | 21.5 | 26.5 | 27.7 | 28.5 | 2 |

N=0 is DFR on clean held-out images only; N=2 can fall below N=0 because group balancing then cuts every group to 2 images. Total conflict examples = 10N.

**Exact-unlearning oracle: retrain from the same initialisation on the retain set (all training data minus the discovered tinted images)**

| Backbone | ρ | ERM | Retrain α=0.1 | SSD α=0.1 (Run 2c) | Retrain α=1 | SSD α=1 | Retrain α=1: clean acc | Retrain α=1: ART k=4 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ResNet-18 | 0.05 | 39.9 | 40.6 ± 3.0 | **49.4** | 48.8 ± 2.2 | **56.9** | 90.7 | 26.5 |
| ResNet-18 | 0.25 | 17.3 | 18.2 ± 1.1 | **40.8** | **50.3 ± 1.3** | 45.6 | 90.0 | 25.2 |
| ResNet-18 | 0.5 | 9.5 | 10.4 ± 1.8 | **26.9** | **53.4 ± 0.6** | 32.8 | 88.5 | 28.6 |
| ResNet-18 | 0.75 | 7.8 | 9.1 ± 0.4 | **20.7** | **38.7 ± 0.6** | 24.6 | 85.4 | 20.8 |
| ResNet-18 | 0.95 | 2.6 | 2.3 ± 0.9 | **7.5** | **20.6 ± 1.5** | 7.4 | 71.9 | 5.5 |
| ResNet-9 | 0.05 | 23.8 | 26.3 ± 1.9 | **33.4** | **38.4 ± 0.8** | 33.3 | 89.9 | 12.2 |
| ResNet-9 | 0.25 | 12.7 | 13.3 ± 2.7 | **26.1** | **32.7 ± 3.6** | 25.8 | 88.5 | 10.2 |
| ResNet-9 | 0.5 | 3.0 | 3.0 ± 0.1 | **15.9** | **28.6 ± 0.8** | 15.8 | 85.9 | 8.2 |
| ResNet-9 | 0.75 | 1.3 | 1.0 ± 0.3 | **9.7** | **13.9 ± 0.5** | 7.4 | 80.7 | 2.9 |
| ResNet-9 | 0.95 | 0.0 | 0.0 ± 0.0 | **1.6** | **1.5 ± 1.2** | 0.3 | 57.6 | 0.0 |

**ρ=0 control (deconfound):** a model trained with no shortcut, then given held-out DFR, reaches 67.3 ± 0.7 (ResNet-18) and 63.8 ± 1.5 (ResNet-9). That is above DFR on any shortcut model (best 63.2 and 56.6 at ρ=5%).

**Readings**

- *Decision rule, quantified:* a few conflict examples beat SSD. DFR needs 2–5 per group on ResNet-9 and 10–50 per group on ResNet-18 at ρ ≤ 75%, and none at ρ=95% on ResNet-18.
- *Under partial discovery, unlearning beats the exact-unlearning gold standard.* At α=0.1, retraining without the discovered 10% stays at ERM level, because 90% of the tinted images remain. SSD at α=0.1 gains +9 to +23 by generalising past the flagged samples. This is the corrective-unlearning point of Goel et al., now shown for clean-label bias.
- *With full discovery, exact retraining beats SSD at every ρ except ResNet-18 at 5%* (56.9 vs 48.8). At high ρ retraining also suffers, because it keeps only 5–25% of the data.
- *ART is not specific to unlearning.* Exact retraining at α=1, which never saw a tinted training image, drops from 48.8 to 26.5 at k=4 (ResNet-18, ρ=5%), about as much as SSD (56.9 → 35.7). The ART drop must be read against the oracle, not ERM, before claiming that unlearning "hides" the shortcut.
- *"DFR beats the no-shortcut model" was confounded* by tint exposure in the held-out set. The fair ceiling is the ρ=0 model plus DFR (67.3 / 63.8). DFR on shortcut models recovers 94% of it at ρ=5% and 57% at ρ=95% (ResNet-18).

## Run 2h — Waterbirds: held-out DFR, conflict-count crossover, exact-unlearning oracle (ResNet-50, 5 seeds)

On Waterbirds, 5 conflict examples per minority group beat SSD, and 10 beat every unlearning method. Held-out DFR reaches 88.7 and keeps 83.0 under ART k=4. Exact retraining on the retain set is no better than unlearning: ERM-level at partial discovery (75–77), collapsing to 15.6 at full discovery (only 240 images left), and as restorable as the unlearning methods (ART k=4: 29–36). Tag run2h; 60/60 units, 0 failed; reuses the Run 1 ERM models (confirmed in the log).

The held-out set is 598 images (group counts 234 / 233 / 67 / 67); the two minority groups (67 each) are the conflict groups.

| Method | Conflict examples used | Test WGA | Avg acc | ART k=2 | ART k=4 |
| --- | --- | --- | --- | --- | --- |
| ERM | — | 74.1 ± 0.9 | 90.3 | 51.9 | 28.4 |
| DFR, held-out, N=0 | 0 | 31.0 ± 7.7 | 69.6 | 16.6 | 12.2 |
| DFR, held-out, N=2 | 4 | 69.7 ± 3.8 | 78.1 | 67.8 | 63.8 |
| DFR, held-out, N=5 | 10 | 75.3 ± 2.5 | 82.4 | 71.6 | 63.0 |
| DFR, held-out, N=10 | 20 | 81.2 ± 1.7 | 86.5 | 78.5 | 71.9 |
| DFR, held-out, N=25 | 50 | 85.7 ± 1.3 | 89.1 | 84.5 | 78.5 |
| DFR, held-out, N=50 | 100 | **89.3 ± 1.6** | 90.9 | 88.8 | **86.4** |
| DFR, held-out, all (dfr\_val) | 134 | 88.7 ± 0.6 | 91.5 | 87.1 | 83.0 |
| Retrain on retain set, α=0.1 (4,341 images) | 0 | 75.0 ± 1.1 | 90.6 | 52.8 | 29.2 |
| Retrain on retain set, α=0.25 (3,662) | 0 | 74.5 ± 1.3 | 91.0 | 53.7 | 30.7 |
| Retrain on retain set, α=0.5 (2,522) | 0 | 77.2 ± 1.7 | 92.0 | 57.8 | 35.8 |
| Retrain on retain set, α=1 (240) | 0 | 15.6 ± 3.0 | 72.3 | 11.0 | 9.3 |

For comparison (Run 2b): SSD 74.0, best unlearning methods 77.7–78.7 (fine-grid NPO, SCRUB, BadT) with ART k=4 of 26–43, and fine-tuning on the retain set 87.5 (ART k=4 63.3).

**Readings**

- *Crossover:* 5 conflict examples per minority group (10 in total) beat SSD (75.3 vs 74.0). 10 per group (20 in total) beat every unlearning method (81.2 vs 78.7). With no conflict examples, DFR collapses (31.0).
- *The ART gap is not unlearning-specific on Waterbirds either.* Exact retraining at α ≤ 0.5 drops to 29–36 at k=4, the same band as the unlearning methods (26–34) and ERM (28.4). Only methods fit on conflict examples resist: held-out DFR keeps 83.0 and N=50 keeps 86.4. Restorability tracks conflict data, not the unlearning objective, and holds on both Waterbirds and CIFAR.
- *Exact retraining is a poor target for spurious bias.* At 95% bias-aligned training data, removing all discovered aligned samples leaves 240 images and collapses WGA to 15.6. Partial removal leaves the shortcut intact (75–77, about ERM). Unlearning methods sit 1–4 points above it.
- *Held-out DFR (88.7) is close to the published 92.9.* Our earlier 87.0 used training annotations.

## Run 2 — CelebA with the fixed forget set (ResNet-50, 5 seeds)

With the correct forget set, no unlearning method improves CelebA. SSD and the fine-grid NegGrad/NPO stay exactly at ERM (46), SCRUB and fine-tuning get worse at larger α, and BadT collapses to 1–5. DFR reaches 89–90 at every α, and just 2 conflict examples per minority group (80.6) beat every unlearning method by over 20 points. Run 1's NPO-head success (82.8) was an artefact of the old forget set, which marked non-blond women as bias-aligned. Tag run2; 205/205 units, 0 failed; reuses the Run 1 ERM models.

| Method | α=0.10 | α=0.25 | α=0.50 | α=1.00 | ART k=4 (α=1) |
| --- | --- | --- | --- | --- | --- |
| ERM | 46.1 ± 3.0 | — | — | — | 24.0 |
| DFR (training annotations) | **89.3 ± 0.9** | **89.6 ± 1.1** | **90.4 ± 0.2** | **89.8 ± 1.2** | 65.4 |
| Fine-tune (retain) | 59.0 ± 6.9 | 57.0 ± 3.9 | 55.2 ± 10.2 | 25.1 ± 7.9 | 20.1 |
| SSD | 46.1 ± 3.0 | 46.1 ± 3.0 | 46.1 ± 3.0 | 46.2 ± 3.0 | 24.0 |
| NegGrad fine grid | 45.8 ± 3.1 | 45.7 ± 3.0 | 45.8 ± 3.1 | 45.7 ± 3.0 | 23.9 |
| NPO fine grid | 45.8 ± 3.1 | 45.8 ± 3.1 | 45.7 ± 3.0 | 45.7 ± 3.0 | 23.9 |
| SCRUB | 45.2 ± 1.4 | 43.7 ± 8.7 | 38.1 ± 10.8 | 27.3 ± 5.0 | 16.3 |
| NPO (head only) | 44.6 ± 3.2 | 44.6 ± 3.2 | 44.6 ± 3.2 | 44.6 ± 3.2 | 23.6 |
| BadT | 5.2 ± 5.1 | 2.3 ± 2.2 | 1.9 ± 1.4 | 1.0 ± 0.7 | 7.4 |

**Held-out DFR and the conflict-count crossover** (N conflict examples per minority group in the held-out fitting half):

| N=0 | N=2 | N=5 | N=10 | N=25 | N=50 | All (dfr\_val) |
| --- | --- | --- | --- | --- | --- | --- |
| 64.6 ± 6.5 | 80.6 ± 3.1 | 82.2 ± 0.8 | 83.6 ± 1.5 | 85.1 ± 1.1 | 87.6 ± 0.8 | 87.6 ± 1.1 |

**Probes (ERM, gender, chance 59.5%):** layer1 86.1, layer2 88.8, layer3 93.8, layer4 91.4. **SSD ratios:** 1.2M–5.4M parameters with ratio > 10, p99.9 17–26 — a diffuse, flat tail, as on Waterbirds.

**Readings**

- *SSD selects nothing useful:* its validation-best setting equals ERM at every α (healed 0.0%). The same pattern as Waterbirds (74.0 vs ERM 74.1).
- *Even 0 conflict examples beat unlearning on CelebA:* DFR on clean held-out data (N=0) reaches 64.6, above every unlearning method (best 59.0). With N=2 it reaches 80.6.
- *DFR is only partly robust to ART on CelebA:* it falls from 89.8 to 65.4 at k=4 (−24), matching ART's own CelebA DFR drop (−24.5). All unlearning methods sit at ERM's 24.0, so CelebA does not separate unlearning from ERM under ART either.
- *Label-pushing collapse is method-specific:* BadT collapses (1–5), SCRUB and fine-tuning degrade as α grows (27 and 25 at α=1), while the fine-grid NegGrad/NPO safely do nothing.

## Run 2i — Wider SSD grid and an FMD-style baseline (CIFAR 3 seeds; Waterbirds/CelebA 5 seeds)

A wider SSD grid (40 settings instead of 15) lifts SSD substantially at high prevalence on CIFAR. At ρ=75%, ResNet-18 goes 24.6 → 35.3 and ResNet-9 7.4 → 36.9. The steep decay in Run 2c was partly a grid artefact. SSD still stays below held-out DFR at every ρ, still fails at ρ=95%, and still heals nothing on Waterbirds (75.1 vs 74.1) or CelebA (46.1 = ERM). Our FMD-style reimplementation helps only on CelebA (58.7 vs 46.1) and is far below DFR everywhere. Tags run2i: CIFAR 120/120, Waterbirds 20/20, CelebA 20/20 units, 0 failed.

Grid: α\_SSD ∈ {0.1, 0.25, 0.5, 1, 2, 5, 10, 50} × λ ∈ {0.01, 0.05, 0.1, 0.5, 1}. FMD-style: one damped Newton step on the head minimising retain cross-entropy + γ·(prediction gap on counterfactual pairs), with γ ∈ {1, 10, 100, 1000} and damping ∈ {0.001, 0.01, 0.1, 1}. CIFAR uses exact pairs (each image's clean and other-colour copies); Waterbirds/CelebA use same-class, different-attribute pairs. Both are chosen on a held-out selection half.

**Colored CIFAR, test WGA (α=1 unless noted)**

| Backbone | ρ | ERM | SSD, old grid | SSD, wide grid | SSD wide: clean acc | SSD wide: % gap to ρ=0 closed | FMD-style | DFR (held-out) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ResNet-18 | 0.05 | 39.9 | 56.9 | 53.1 ± 7.9 | 89.5 | 114% | 40.6 ± 2.6 | **63.2** |
| ResNet-18 | 0.25 | 17.3 | 45.6 | 49.4 ± 2.3 | 87.7 | 94% | 19.0 ± 2.2 | **54.3** |
| ResNet-18 | 0.5 | 9.5 | 32.8 | 40.6 ± 2.2 | 85.7 | 74% | 13.3 ± 1.6 | **52.7** |
| ResNet-18 | 0.75 | 7.8 | 24.6 | 35.3 ± 1.6 | 83.5 | 63% | 9.3 ± 1.6 | **46.3** |
| ResNet-18 | 0.95 | 2.6 | 7.4 | 13.8 ± 4.9 | 72.1 | 23% | 5.4 ± 1.3 | **38.5** |
| ResNet-9 | 0.05 | 23.8 | 33.3 | 31.7 ± 6.0 | 86.6 | 63% | 30.0 ± 2.2 | **56.6** |
| ResNet-9 | 0.25 | 12.7 | 25.8 | 31.4 ± 2.1 | 83.5 | 79% | 17.6 ± 1.9 | **55.1** |
| ResNet-9 | 0.5 | 3.0 | 15.8 | 31.9 ± 6.4 | 80.3 | 87% | 9.2 ± 0.8 | **49.3** |
| ResNet-9 | 0.75 | 1.3 | 7.4 | 36.9 ± 6.2 | 75.1 | 101% | 3.1 ± 0.8 | **45.2** |
| ResNet-9 | 0.95 | 0.0 | 0.3 | 5.6 ± 1.9 | 59.2 | 15% | 0.3 ± 0.3 | **28.5** |

**Waterbirds and CelebA, test WGA**

| Dataset | ERM | SSD wide grid, α=0.1 | SSD wide grid, α=1 | FMD-style | FMD-style: ART k=4 | DFR (held-out) |
| --- | --- | --- | --- | --- | --- | --- |
| Waterbirds | 74.1 ± 0.9 | 73.7 ± 1.3 | 75.1 ± 1.2 | 73.0 ± 3.7 | 28.8 | **88.7** |
| CelebA | 46.1 ± 2.7 | 46.1 ± 2.7 | 46.1 ± 2.7 | 58.7 ± 6.7 | 33.6 | **87.6** |

**Readings**

- *The prevalence decay is gentler than Run 2c suggested.* On ResNet-18 the wide grid closes 114% → 94% → 74% → 63% → 23% of the gap, still monotone but much flatter than 147% → 10%. On ResNet-9 it is not monotone: it rises to 101% at ρ=75%, then collapses to 15% at 95%. The robust claim is "SSD fails at extreme prevalence and never reaches DFR", not a smooth law.
- *The wide-grid gains cost clean accuracy:* ResNet-9 at ρ=75% drops from 81.1 (ERM) to 75.1, and ResNet-18 at ρ=95% from 76.3 to 72.1. Winning settings use α\_SSD 0.1–0.5 and λ 0.01–0.1, and several still sit on the new grid edge (α\_SSD=0.1 or λ=0.01).
- *On the natural datasets SSD stays at ERM under any setting.* Waterbirds and CelebA winners pick α\_SSD=10–50 with λ=0.01, which dampens only a few hundred to a few thousand parameters. There is no shortcut-specific parameter set for SSD to find.
- *FMD-style is weak in our implementation:* CelebA 58.7 vs the paper's 87.15 (ResNet-18, real counterfactuals). On CIFAR, even with exact pairs, it barely moves WGA and hurts clean accuracy at high ρ. Treat it as a lower bound on FMD, not a faithful reproduction; a reviewer may object.
- *Still true across all data:* every method that never sees conflict examples stays well below held-out DFR.

## Run 2j — Widest SSD grid, SSD then DFR, model-selection sensitivity

SSD has hit its ceiling: 70 settings give the same WGA as 40, and even picking the best setting on the test set adds at most 2 points. SSD and DFR stack on CIFAR: DFR fit on SSD's features beats DFR alone at every ρ by +2 to +8. Every unlearning method needs group-labelled validation; picking settings by average accuracy costs up to 26 points on Waterbirds. Tags run2j: CIFAR 87/90 units (3 ResNet-18 units cut by the budget, resume pushed), Waterbirds 65/65; 0 failed.

Grid: α\_SSD ∈ {0.01, 0.03, 0.1, 0.25, 0.5, 1, 2, 5, 10, 50} × λ ∈ {0.001, 0.003, 0.01, 0.05, 0.1, 0.5, 1}. "SSD then DFR" fits held-out DFR (dfr\_val protocol) on the selected SSD model's features.

**Colored CIFAR, test WGA at α=1**

| Backbone | ρ | ERM | SSD, 40 settings (2i) | SSD, 70 settings | SSD, best-on-test (oracle) | DFR alone (held-out) | SSD then DFR | Gain from stacking |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ResNet-18 | 0.05 | 39.9 | 53.1 | 51.3 ± 1.8 | 51.3 | 63.2 | **65.8 ± 2.0** | +2.6 |
| ResNet-18 | 0.25 | 17.3 | 49.4 | 48.0 ± 5.5 | 49.1 | 54.3 | **61.8 ± 2.8** | +7.5 |
| ResNet-18 | 0.5 | 9.5 | 40.6 | 41.8 ± 4.4 | 41.9 | 52.7 | **60.9 ± 0.6** | +8.2 |
| ResNet-18 | 0.75 | 7.8 | 35.3 | 33.5 ± 3.7 | 33.8 | 46.3 | **53.0 ± 3.2** | +6.7 |
| ResNet-18 | 0.95 | 2.7 | 13.8 | 18.3 ± 6.0 (2 seeds) | 18.3 | 38.5 | **45.5 ± 1.2** | +7.0 |
| ResNet-9 | 0.05 | 23.8 | 31.7 | 30.1 ± 5.1 | 31.9 | 56.6 | **59.0 ± 2.2** | +2.4 |
| ResNet-9 | 0.25 | 12.7 | 31.4 | 31.5 ± 2.0 | 31.5 | 55.1 | **60.5 ± 2.5** | +5.4 |
| ResNet-9 | 0.5 | 3.0 | 31.9 | 34.2 ± 4.4 | 34.5 | 49.3 | **56.3 ± 0.5** | +7.0 |
| ResNet-9 | 0.75 | 1.3 | 36.9 | 29.2 ± 3.9 | 29.3 | 45.2 | **52.9 ± 1.5** | +7.7 |
| ResNet-9 | 0.95 | 0.0 | 5.6 | 6.4 ± 1.5 | 7.0 | 28.5 | **32.6 ± 2.5** | +4.1 |

**Waterbirds, test WGA (5 seeds): selected by worst-group validation vs by average validation accuracy**

| Method | α | Selected by val WGA | Selected by val avg acc | Best-on-test (oracle) | ART k=4 |
| --- | --- | --- | --- | --- | --- |
| ERM | — | 74.1 | — | — | 28.4 |
| SSD (70 settings) | 1.0 | 74.8 ± 1.2 | 71.6 | 75.4 | 28.4 |
| SSD then DFR | 1.0 | **89.0 ± 1.0** | — | — | — |
| NPO fine grid | 1.0 | 78.9 ± 1.5 | 71.2 | 79.0 | 26.9 |
| NegGrad fine grid | 1.0 | 77.4 ± 0.9 | 66.3 | 77.9 | 28.0 |
| SCRUB | 1.0 | 76.5 ± 1.0 | 76.5 | 76.5 | 33.7 |
| BadT | 1.0 | 77.3 ± 5.1 | 51.5 | 77.3 | 42.9 |
| Fine-tune (retain) | 1.0 | 87.7 ± 1.3 | 85.6 | 88.1 | 61.1 |
| BadT | 0.1 | 76.1 ± 2.7 | 58.0 | 76.2 | 33.2 |
| NegGrad fine grid | 0.1 | 76.7 ± 2.4 | 67.2 | 77.1 | 27.7 |

Held-out DFR alone on Waterbirds is 88.7 (Run 2h). On CIFAR, picking SSD by average accuracy costs 0–3.5 points.

**Readings**

- *SSD's ceiling is real, not a grid artefact.* 70 settings match 40 within noise, and even test-set selection adds at most 2 points. The high-ρ failure (ResNet-18 18.3, ResNet-9 6.4 at ρ=95%) is now firm.
- *Unlearning and last-layer retraining are complementary on the synthetic shortcut.* SSD then DFR beats DFR alone by +2 to +8 everywhere on CIFAR and nearly reaches the ρ=0 + DFR ceiling at ρ=5% (65.8 vs 67.3). On Waterbirds it adds nothing (89.0 vs 88.7), because SSD changes nothing there. This is the paper's constructive result.
- *Unlearning's gains depend on group-labelled validation.* With average-accuracy selection, Waterbirds unlearning methods fall below ERM (BadT 51.5, NegGrad 66.3, NPO 71.2, SSD 71.6 vs ERM 74.1). Even unlearning quietly needs conflict data, for model selection.

## Run 2k — REM on colored CIFAR (official code + our adapter, 3 seeds)

REM gains +8 to +13 WGA over its own baseline at ρ ≤ 75% with 10% discovery, and fails at ρ=95%. It is weaker at full discovery, and it never approaches held-out DFR (28–57 on our ResNet-9). Seeds 0–1 ran in cub-run2k-rem, seed 2 in cub-run2k2-rem; 0 failed jobs. NPO has 2 seeds at ρ=75% and 95%.

| ρ | REM's baseline (50% capacity) | REM α=0.1 | REM α=1 | REM α=0.1: clean acc | NPO α=0.1 | NPO α=1 (clean acc) |
| --- | --- | --- | --- | --- | --- | --- |
| 0.05 | 20.5 ± 1.4 | **28.1 ± 2.9** | 13.3 ± 0.8 | 89.2 | 22.9 | 18.2 (70.0) |
| 0.25 | 10.2 ± 1.5 | **19.8 ± 2.8** | 12.4 ± 3.7 | 87.2 | 16.4 | 0.3 (28.0) |
| 0.5 | 4.1 ± 0.6 | **17.5 ± 2.1** | 11.2 ± 1.3 | 82.9 | 7.5 | 1.3 (26.1) |
| 0.75 | 1.1 ± 0.4 | **9.6 ± 3.0** | 3.6 ± 0.7 | 72.7 | 2.8 | 0.0 (24.9) |
| 0.95 | 0.1 ± 0.0 | 0.0 ± 0.0 | 0.5 ± 0.4 | 10.9 | 0.4 | 0.0 (16.1) |

**Readings**

- REM prefers partial discovery: α=0.1 beats α=1 at every ρ ≤ 75%, and at ρ=5% full discovery falls below the baseline (13.3 vs 20.5).
- At ρ=95% REM breaks the model (clean accuracy 10.9 at α=0.1; baseline 60.7).
- NPO alone collapses clean accuracy at α=1 (16–28 for ρ ≥ 25%), consistent with finding 6.
- Caveats: fixed paper hyper-parameters with no worst-group selection, a half-capacity model, and our own tint adapter. Compare gains over each method's own baseline, not absolute numbers.

The ResNet-18 ρ=95% row of Run 2j is now complete with 3 seeds: SSD 19.9 ± 5.4, SSD then DFR 45.4 ± 1.0 (resume cub-run2j2-cifar).

**Next steps moved to a separate document:** the reviewer assessment, new hypotheses N1–N6 and the experiment plan are in "Paper blueprint: reviewer verdict, the gap, and the plan to a top-venue paper" (16 Nov).

## Run 3a — Proportion-type colored CIFAR (hypothesis N1; 3 seeds)

When every image is tinted and only the mix is skewed, SSD's gain disappears: at 75% prevalence it is −0.1 on both backbones, against +27.5 and +35.6 on the presence-type version. The structure of the shortcut, not the dataset or backbone, decides whether SSD helps. Exact retraining at full discovery is reverse-biased, clearly so from scratch. One result cuts against a simple reading: fine-tuning on the retain set at full discovery is the best method, because the retain set is then exactly the conflict examples. Tag run3a, cub-run3a-prop-cifar; 156 + 156 units, 0 failed.

Data: every training image is tinted; a fraction ρ gets its class colour (aligned) and the rest another class's colour (conflict). Test and validation hold an aligned and a conflict copy of each image. Forget set: discovered aligned images.

| Backbone | ρ | ERM WGA (aligned / conflict acc) | SSD α=1 | SSD gain | SSD gain, presence type (Run 2i) | Retrain α=1 (aligned / conflict acc) | Fine-tune α=0.1 | Fine-tune α=1 | DFR (held-out) | SSD then DFR |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ResNet-18 | 0.6 | 65.2 ± 0.7 (96.5 / 84.0) | 66.7 ± 1.5 | +1.5 | — | 62.2 ± 1.6 (80.0 / 85.4) | 66.9 | **75.0 ± 1.7** | 67.2 | 66.0 |
| ResNet-18 | 0.75 | 60.3 ± 1.7 (97.8 / 79.3) | 60.2 ± 1.3 | −0.1 | +27.5 | 56.4 ± 2.0 (76.3 / 83.1) | 63.5 | **73.6 ± 1.9** | 64.5 | 64.8 |
| ResNet-18 | 0.9 | 39.3 ± 1.1 (98.9 / 65.8) | 38.9 ± 0.7 | −0.4 | — | 39.1 ± 5.0 (65.4 / 75.0) | 45.9 | **61.4 ± 2.2** | 49.8 | 49.9 |
| ResNet-18 | 0.95 | 20.3 ± 1.2 (99.2 / 52.0) | 22.9 ± 3.8 | +2.6 | +11.2 | 30.3 ± 0.2 (53.4 / 64.0) | 24.5 | **57.9 ± 1.2** | 37.1 | 36.4 |
| ResNet-9 | 0.6 | 57.2 ± 0.9 (96.7 / 80.1) | 59.6 ± 1.6 | +2.4 | — | 29.6 ± 2.8 (64.1 / 82.2) | 60.8 | **68.3 ± 2.1** | 59.8 | 60.3 |
| ResNet-9 | 0.75 | 47.0 ± 0.9 (97.7 / 72.9) | 46.9 ± 1.5 | −0.1 | +35.6 | 23.7 ± 0.4 (58.4 / 77.2) | 53.2 | **59.3 ± 6.5** | 54.1 | 54.6 |
| ResNet-9 | 0.9 | 15.9 ± 0.8 (99.0 / 50.4) | 16.0 ± 0.7 | +0.1 | — | 5.0 ± 0.4 (35.2 / 63.1) | 21.3 | **44.3 ± 2.6** | 39.2 | 38.7 |
| ResNet-9 | 0.95 | 2.8 ± 0.7 (99.4 / 30.4) | 6.2 ± 1.1 | +3.4 | +5.6 | 1.2 ± 0.3 (16.1 / 48.5) | 7.8 | 27.4 ± 3.6 | **29.7** | 28.3 |

Also: SSD at α=0.1 gains −0.1 to +3.9; carefully tuned NPO gains 0 to +8 (ResNet-18, ρ=90% only); DFR on training annotations now equals held-out DFR (e.g. 37.1 at ρ=95% on ResNet-18), because the training set itself contains conflict images.

**Against the predictions written before the run**

- *SSD gains under 3 points at every ρ:* holds in 7 of 8 cells at α=1. The exception is ResNet-9 at ρ=95% (+3.4, from 2.8 to 6.2).
- *Kill criterion (SSD keeps half its presence-type gain at matched prevalence):* not met at 75% (−0.1 vs +27.5 and +35.6). At 95% it is formally met on ResNet-9 (+3.4 vs half of +5.6), but both gains are a few points on a WGA near zero. N1 stands, with the 95% cell reported as inconclusive.
- *Exact retraining at α=1 falls below ERM:* holds on ResNet-9 at every ρ (e.g. 23.7 vs 47.0) and on ResNet-18 at 60–75%; a tie at 90%; **fails on ResNet-18 at 95%** (30.3 vs 20.3). In every cell aligned accuracy falls sharply (99 → 16–80), which is the predicted reversal; with a pretrained backbone and extreme skew the reversal still nets out positive.
- *SSD then DFR stops stacking:* it equals DFR alone within 1 point everywhere (it gained +2 to +8 on the presence type).

**New and important**

- *Fine-tuning on the retain set at full discovery beats held-out DFR by 8–21 points on ResNet-18* (57.9 vs 37.1 at ρ=95%). At α=1 the retain set is exactly the training conflict images, 2,250–18,000 of them, and early stopping is chosen on group-labelled validation. At α=0.1 the gain shrinks to 0–7 points. So a forget-set method can work on proportion-type bias, but only by gaining access to conflict data through complete discovery. This matches Waterbirds, where fine-tuning at α=1 reached 87.5.
- The thesis sentence must therefore separate methods that use only the forget set's signal (SSD, NPO: no gain) from methods that train on the retain set (fine-tune: large gain once discovery is complete).

## Batch 3b / 3c / 3d — pushed 16 Nov (running)

Code changes (smoke-tested locally before pushing): `--aligned_share` (subsample bias-aligned training images of Waterbirds/CelebA to a target share; dataset name `waterbirds-s<share>`), `retrain_bal` (group-balanced retraining oracle on all training data, the second oracle), and low-label stacking (`ssd_then_dfr_ccN`: DFR with N = 2/5/10/25 conflict examples per group on SSD-edited features).

| Run | Account | What it tests | Settings |
| --- | --- | --- | --- |
| 3b `cub-run3b-prop-cifar` | main | N3 discovery completeness on proportion-type bias; balanced oracle | proportion CIFAR, ρ = 75%, 95%, both backbones, 3 seeds; retrain / fine-tune / SSD (40 settings) at α = 0.25, 0.5, 0.75, 0.9; `retrain_bal`. Reuses Run 3a models |
| 3c `cub-run3c-waterbirds` | main | N3 on Waterbirds; N2 (does lowering the aligned share make unlearning work?) | Full data, 5 seeds: retrain / fine-tune at α = 0.75, 0.9 and `retrain_bal`. Aligned-share sweep 60%, 75%, 85% (new ERM models, 3 seeds): ERM, probe, held-out DFR, SSD (40 settings), retrain, fine-tune, NPO-fine, `retrain_bal` at α = 0.1, 1 |
| 3d `cub-run3d-cifar` | second | N3 on presence-type bias; N5 low-label stacking | presence CIFAR, both backbones, 3 seeds: retrain / fine-tune at α = 0.25–0.9 for ρ = 25%, 75%; SSD at α = 1 followed by DFR with 2/5/10/25 conflict examples per group, all ρ. Reuses Run 2c models |

GPU quota before pushing: main about 12 h, second 12.9 h (resets Sat 21 Nov).

## Confidence intervals from existing runs (no GPU, 16 Nov)

Method: mean worst-group accuracy over seeds with a 95% t-interval; gains are paired with the same seed's ERM model. Script and full table (795 rows): `analysis/ci.py`, `analysis/ci_table.csv`. Three seeds give wide intervals, so anything marked weak below needs 5 seeds before it goes in the paper.

| Claim | Numbers (WGA, gain over ERM ± 95% CI) | Verdict |
| --- | --- | --- |
| Waterbirds: held-out DFR beats ERM | 88.7 ± 0.9, gain +14.6 ± 1.5 (n=5) | Solid |
| Waterbirds: SSD does nothing | gains −0.4 ± 1.4 to +1.0 ± 1.4 across three grids (n=5) | Solid null |
| Waterbirds: fine-tune on retain at α=1 | 87.7 ± 1.8, gain +13.5 ± 1.7 (n=5) | Solid; within noise of held-out DFR |
| Waterbirds: retrain oracle at α=1 collapses | 15.6 ± 4.2, gain −58.6 ± 4.3 (n=5) | Solid |
| Waterbirds: SCRUB / NPO-fine at α=1 | +2.4 ± 0.7 to +4.0 ± 1.2 / +4.7 ± 1.7 (n=5) | Small but real |
| Waterbirds: FMD-style | −1.1 ± 4.3 (n=5) | Null |
| Waterbirds: DFR with 5 per group | +1.1 ± 4.3 (n=5) | Tie with ERM and SSD (confirms the correction) |
| CelebA: held-out DFR | 87.6 ± 1.4, gain +41.5 ± 4.9 (n=5) | Solid |
| CelebA: DFR with 2 per group | 80.6 ± 3.9, gain +34.5 ± 4.7 (n=5) | Solid |
| CelebA: SSD | +0.0 ± 0.0 (n=5) | Solid null |
| CelebA: FMD-style | 58.7 ± 9.3, gain +12.6 ± 11.1 (n=5) | Barely significant, high variance |
| CelebA: fine-tune α=0.1 | 59.0 ± 8.6, gain +12.9 ± 11.5 (n=5) | Barely significant |
| Presence CIFAR ρ=75%: SSD helps | ResNet-18 +27.5 ± 3.6; ResNet-9 +35.6 ± 17.9 (n=3) | Solid on ResNet-18; ResNet-9 significant but wide |
| Presence CIFAR ρ=25%, 50%: SSD helps | R18 +32.1 ± 7.1, +31.1 ± 8.8; R9 +18.7 ± 6.3, +28.9 ± 18.5 (n=3) | Solid |
| Presence CIFAR ρ=5%, 95%: SSD helps | ρ=5%: +7.9 ± 18.9 / +13.2 ± 24.4; ρ=95%: +5.6 ± 5.9 / +11.1 ± 14.5 (n=3) | Weak — not significant, needs 5 seeds |
| Proportion CIFAR ρ=75%: SSD does nothing (N1) | R18 −0.1 ± 1.3; R9 −0.1 ± 1.8 (n=3) | Solid null; gap to presence (+27.5, +35.6) is far outside the intervals |
| Proportion CIFAR ρ=90%: SSD does nothing | R18 −0.4 ± 1.8; R9 +0.0 ± 0.1 (n=3) | Solid null |
| Proportion CIFAR ρ=95%: SSD | R9 +3.4 ± 1.3; R18 +2.6 ± 11.0 (n=3) | Small / not significant |
| Proportion: fine-tune on retain at α=1 is best | R18: +9.8 ± 7.0 (60%), +13.4 ± 6.0 (75%), +22.2 ± 4.5 (90%), +37.6 ± 3.3 (95%); R9: +11.1 ± 3.6, +12.3 ± 17.0, +28.4 ± 5.6, +24.6 ± 8.8 (n=3) | Solid on ResNet-18 and at ρ ≥ 90%; R9 at 75% weak |
| Proportion ρ=95% R18: fine-tune beats DFR on training annotations | 57.9 ± 3.8 vs 37.1 ± 1.0 (n=3) | Solid |
| Proportion: retrain oracle at α=1 is below ERM on ResNet-9 | −23.3 ± 3.5 at 75% (n=3) | Solid; ResNet-18 is mixed (−3.9 ± 2.9 at 75%, +10.0 ± 4.4 at 95%) |
| Presence: retrain oracle at α=0.1 ≈ ERM | −0.3 ± 1.0 to +1.3 ± 3.4 (n=3) | Solid null |

What this changes: the headline contrasts (N1 presence vs proportion, DFR vs SSD on natural data, oracle collapse) hold with intervals. The weak cells are all 3-seed CIFAR results at the extremes (ρ = 5% and 95%) and some ResNet-9 cells; adding seeds 3 and 4 there is now on the to-do list.

## Results: Runs 3b, 3c, 3d (finished 16 Nov)

All numbers are test worst-group accuracy (WGA); gains are paired with the same seed's ERM model, ± 95% t-interval. α = share of the bias-aligned set that was discovered.

### Run 3b — proportion-type CIFAR, discovery sweep and balanced oracle (3 seeds, complete: 168/168 units)

| Setting | ERM | Retrain α=0.5 | Retrain α=0.9 | Retrain α=1 | Balanced retrain | Fine-tune α=0.9 | Fine-tune α=1 | SSD best over α |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ResNet-18, ρ=75% | 60.3 | 65.4 | 69.9 (+9.6 ± 6.3) | 56.4 (−3.9 ± 2.9) | 67.3 (+7.0 ± 5.9) | 70.8 (+10.5 ± 3.2) | 73.6 (+13.4 ± 6.0) | 65.3 (+5.0 ± 11.3, not significant) |
| ResNet-18, ρ=95% | 20.3 | 27.9 | 38.2 (+17.9 ± 1.3) | 30.3 (+10.0 ± 4.4) | 43.5 (+23.2 ± 3.7) | 43.0 (+22.7 ± 5.7) | 57.9 (+37.6 ± 3.3) | 24.2 (+3.9 ± 6.8, not significant) |
| ResNet-9, ρ=75% | 47.0 | 49.5 | 55.7 (+8.7 ± 4.0) | 23.7 (−23.3 ± 3.5) | 57.7 (+10.7 ± 3.1) | 59.5 (+12.5 ± 10.9) | 59.3 (+12.3 ± 17.0) | 50.3 (+3.3 ± 6.3, not significant) |
| ResNet-9, ρ=95% | 2.8 | 4.2 | 8.9 (+6.1 ± 4.6) | 1.2 (−1.6 ± 1.4) | 24.4 (+21.6 ± 1.9) | 19.1 (+16.3 ± 12.7) | 27.4 (+24.6 ± 8.8) | 7.1 (+4.3 ± 10.2, not significant) |

Findings:

- **The retrain oracle is non-monotonic in discovery.** It improves steadily up to α = 0.9 and then drops at α = 1 (ResNet-9, ρ=75%: 55.7 → 23.7; average accuracy 82.2 → 67.8). Removing every aligned example leaves a reverse-biased training set; keeping 10% of them prevents it. So “exact unlearning of the full flagged set” is the wrong target for proportion-type bias, and partial discovery is accidentally better.
- **The balanced oracle is the right reference**: it beats the forget-set oracle at α = 1 in all four settings and keeps the highest average accuracy (83.9–89.8 on ResNet-18).
- **Fine-tune on the retain set at α = 1 beats even the balanced oracle on ResNet-18** (73.6 vs 67.3; 57.9 vs 43.5). At α ≤ 0.75 its gain is small (+4 to +13).
- **SSD is null at every discovery rate** (no setting significant). Discovery completeness does not rescue forget-signal-only unlearning for proportion-type bias.

### Run 3c — Waterbirds (full data 5 seeds; aligned-share sweep 3 seeds, 129/138 units — rest in Run 3e)

Full data (ERM 74.1, held-out DFR 88.7):

| Method | α=0.1 | α=0.25 | α=0.5 | α=0.75 | α=0.9 | α=1 |
| --- | --- | --- | --- | --- | --- | --- |
| Retrain oracle | 75.0 | 75–77 | 75–77 | 78.3 (+4.2 ± 1.5) | 80.2 (+6.1 ± 2.7) | 15.6 (−58.6 ± 4.3) |
| Fine-tune on retain | 78.5 | 77.0 | 78.4 | 82.6 (+8.4 ± 1.8) | 82.8 (+8.6 ± 2.8) | 85.6–87.7 |
| SSD | 73.9 | 74.3 | 75.4 | 74.9 | — | 74.6 |

Balanced retrain oracle: 81.6 ± 1.6 (+7.5 ± 2.4). Same cliff as on proportion CIFAR: the retrain oracle peaks at α = 0.9 and collapses at α = 1. Both oracles stay 7–8 points below held-out DFR (88.7).

Aligned-share sweep (training set subsampled so aligned images are 60 / 75 / 85% instead of about 95%):

| Aligned share | ERM | Held-out DFR | SSD α=0.1 / α=1 (gain) | Retrain α=1 | Balanced retrain | Fine-tune α=0.1 | NPO-fine α=0.1 / α=1 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 60% | 80.2 | 88.8 | −0.4 ± 0.6 / −0.4 ± 1.6 | 13.9 | 86.7 | 84.9 (+4.7 ± 0.4) | 88.6 (+8.4 ± 1.3) / 88.7 (+8.5 ± 1.5) |
| 75% | 80.4 | 89.2 | −0.1 ± 0.2 / −0.1 ± 0.6 | 15.2 (n=2) | 84.6 | 85.8 (+5.5 ± 3.4) | 81.9 / 85.7 (n=2) |
| 85% | 78.7 | 89.2 | +0.5 ± 1.6 / +0.1 (n=2) | 15.2 (n=2) | 84.7 | 84.0 (n=2) | 83.4 / 83.1 (n=2) |

Findings:

- **N2 is refuted on Waterbirds**: lowering the aligned share from 95% to 60% does not make SSD work (gain within ± 1.6 of zero at every share). Prevalence is not the lever on natural data; the type of shortcut is. This strengthens N1 and removes prevalence as a competing explanation.
- ERM itself only rises from 74.1 to about 80 when the share drops to 60%, and held-out DFR stays at about 89 regardless.
- New observation to verify: at 60% share NPO-fine reaches 88.6–88.7, level with held-out DFR (3 seeds). At 75–85% it is 82–86 with wide intervals.

### Run 3d — presence-type CIFAR (3 seeds; 148/180 units — rest in Run 3e)

Discovery sweep, gain over ERM:

| Setting | Method | α=0.1 | 0.25 | 0.5 | 0.75 | 0.9 | 1 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ResNet-18, ρ=25% (ERM 17.3) | Retrain | +0.9 | +4.2 | +7.5 | +15.7 | +27.7 | +33.0 |
|  | Fine-tune | +4.6 | +2.4 | +3.2 | +5.2 | +6.9 | +10.3 |
| ResNet-18, ρ=75% (ERM 7.8) | Retrain | +1.3 | +0.6 | +2.8 | +6.5 | +13.5 | +31.0 |
|  | Fine-tune | +3.4 | +3.1 | +5.8 | +3.4 | +6.4 | +14.7 |
| ResNet-9, ρ=25% (ERM 12.7) | Retrain | +0.6 | +0.9 | +6.4 | +8.9 | +14.2 | +20.0 |
|  | Fine-tune | +4.2 | +6.1 | +9.0 | +6.2 | +7.6 | +10.1 |
| ResNet-9, ρ=75% (ERM 1.3) | Retrain | −0.3 | −0.2 | +0.6 | +1.2 | +4.0 | +12.6 |
|  | Fine-tune | +0.7 | +1.3 | +1.4 | +0.9 | +1.6 | +2.5 |

For presence-type bias the retrain oracle is monotonic and convex in discovery: most of the benefit arrives only above 75–90% discovery, and α = 1 is best (no cliff, because removing tinted images does not reverse the bias). SSD at α = 0.1 already gains +20 to +29 at ρ = 75% where the oracle gains about +1, so SSD is doing something other than approximating retraining.

Low-label stacking (SSD at α = 1, then DFR with N conflict examples per group; baseline is the same DFR on unedited ERM features, from Run 2f):

| Setting | SSD alone | N=2: SSD+DFR / DFR | N=5 | N=10 | N=25 | Full held-out: SSD+DFR / DFR |
| --- | --- | --- | --- | --- | --- | --- |
| ResNet-18, ρ=5% | 52.9 | 43.9 / 38.1 | 54.3 / 49.4 | 59.4 / 54.5 | 63.0 / 57.5 | 67.3 / 63.2 |
| ResNet-18, ρ=25% | 46.2 | 33.3 / 19.6 | 47.0 / 31.4 | 52.1 / 35.9 | 54.2 / 45.1 | 61.9 / 54.3 |
| ResNet-18, ρ=50% | 40.8 | 31.8 / 16.0 | 45.1 / 25.1 | 49.7 / 35.6 | 52.2 / 38.6 | 59.5 / 52.7 |
| ResNet-18, ρ=75% | 32.6 | 28.7 / 15.2 | 39.5 / 23.8 | 41.8 / 30.6 | 47.7 / 35.5 | 55.1 / 46.3 |
| ResNet-9, ρ=5% | 32.0 | 36.2 / 32.0 | 45.0 / 36.9 | 50.8 / 39.3 | 54.0 / 47.3 | 60.1 / 56.6 |
| ResNet-9, ρ=25% | 31.1 | 41.9 / 20.4 | 47.5 / 28.0 | 49.9 / 33.4 | 53.4 / 42.4 | 61.3 / 55.1 |
| ResNet-9, ρ=50% | 33.6 | 39.0 / 14.8 | 49.2 / 21.1 | 51.8 / 30.3 | 54.8 / 41.0 | 58.7 / 49.3 |
| ResNet-9, ρ=75% | 34.5 | 34.9 / 10.1 | 43.6 / 18.1 | 48.7 / 27.6 | 49.6 / 34.7 | 52.5 / 45.2 |
| ResNet-9, ρ=95% (n=2) | 4.9 | 13.7 / 6.5 | 20.4 / 9.1 | 23.8 / 18.5 | 25.2 / 21.5 | 33.4 / 28.5 |

Findings:

- **Unlearning and conflict labels are complements on presence-type bias.** SSD followed by DFR beats DFR alone in all 9 settings at every label budget; the gap is largest when labels are scarce (+14 to +25 points at N = 2 for ρ = 25–75%).
- **About 10× label saving**: SSD + 5 conflict examples per group matches or beats DFR with 25–100 per group on unedited features (e.g. ResNet-9 ρ=50%: 49.2 vs 41.0 at N=25, 50.7 at N=100, 49.3 with the full held-out set).
- With N = 2 and low prevalence, DFR on SSD features is below SSD alone (ResNet-18 ρ=5%: 43.9 vs 52.9), so tiny conflict sets can hurt.
- Caveat: baseline numbers come from an earlier run (Run 2f); intervals for the stacking gains still need computing from paired seeds.

### What this batch changes in the story

1. N1 (shortcut type decides whether forget-set unlearning works) is now supported from three directions: SSD null on proportion CIFAR at every α, SSD null on Waterbirds at every aligned share, SSD strong on presence CIFAR.
2. N2 (prevalence explains the Waterbirds failure) is dropped.
3. N3 becomes a two-oracle result with a cliff: the forget-set retrain oracle peaks at about 90% discovery and collapses at 100% for proportion-type bias (CIFAR and Waterbirds), while it is monotonic for presence-type bias. The balanced oracle is the correct reference.
4. N5 has its first positive result: unlearning cuts the conflict-label requirement by roughly 10× on presence-type bias.

## Batch 3e — pushed 16 Nov (running)

| Run | Account | Purpose |
| --- | --- | --- |
| `cub-run3e-celeba` | main | CelebA, seeds 0–2: retrain oracle at α = 0.9 and 1, balanced oracle, fine-tune at α = 0.9 and 1 (does the cliff appear on CelebA?) |
| `cub-run3e-waterbirds` | main | Remaining 9 units of the 75% and 85% aligned-share sweep |
| `cub-run3e-cifar` | second | Remaining Run 3d stacking units (ResNet-18 ρ=95%, ρ=0); seeds 3 and 4 for the weak cells: presence ρ = 5/75/95% on both backbones, proportion ρ = 75/95% on ResNet-18 |

Quota before pushing: main 8.1 h, second 10.6 h.

## Results: Batch 3e (finished 16 Nov, 12:20 UTC)

### CIFAR with 5 seeds (seeds 3 and 4 added; gains paired with ERM, ± 95% CI)

Presence-type:

| Setting | ERM | SSD α=0.1 | SSD α=1 | Retrain α=0.1 | Retrain α=1 | Fine-tune α=1 |
| --- | --- | --- | --- | --- | --- | --- |
| ResNet-18, ρ=5% | 40.6 | +8.1 ± 4.8 | +13.2 ± 8.6 | +0.7 ± 4.9 | +8.9 ± 5.3 | +7.6 ± 6.5 |
| ResNet-18, ρ=75% | 8.1 | +21.3 ± 8.7 | +23.4 ± 7.1 | +1.4 ± 2.2 | +31.6 ± 1.9 | +13.1 ± 3.5 |
| ResNet-18, ρ=95% | 3.0 | +7.3 ± 4.6 | +14.2 ± 7.9 | −0.3 ± 1.4 | +19.2 ± 2.9 | +10.1 ± 0.8 |
| ResNet-9, ρ=5% | 23.9 | +6.9 ± 6.4 | +9.1 ± 7.9 | +1.8 ± 3.0 | +13.4 ± 2.4 | +4.3 ± 2.9 |
| ResNet-9, ρ=75% | 1.1 | +26.3 ± 11.4 | +37.9 ± 9.1 | −0.0 ± 0.5 | +12.4 ± 1.1 | +2.8 ± 1.2 |
| ResNet-9, ρ=95% | 0.0 | +2.6 ± 1.8 | +6.1 ± 4.3 | +0.0 ± 0.1 | +1.5 ± 1.4 | +1.0 ± 1.0 |

Proportion-type (ResNet-18):

| Setting | ERM | SSD α=0.1 | SSD α=1 | Retrain α=1 | Balanced retrain | Fine-tune α=1 | Held-out DFR |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ρ=75% | 59.5 | +2.3 ± 2.5 | +0.9 ± 2.2 | −2.0 ± 3.7 | +8.6 ± 3.7 | +13.5 ± 2.1 | +4.3 ± 1.3 |
| ρ=95% | 21.1 | +1.0 ± 2.1 | +3.1 ± 5.2 | +7.2 ± 5.0 | +22.4 ± 2.3 | +36.5 ± 5.8 | +16.3 ± 3.9 |

The cells that were weak with 3 seeds are now settled: SSD's gain on presence-type bias is significant at every prevalence on both backbones (including 5% and 95%), and SSD stays null on proportion-type bias. SSD at α = 0.1 beats the retrain oracle at α = 0.1 in every presence setting (the oracle gains nothing at 10% discovery).

### Waterbirds aligned-share sweep, now complete (3 seeds, 138/138)

| Aligned share | ERM | Held-out DFR | SSD α=0.1 / 1 | Retrain α=0.1 / 1 | Balanced retrain | Fine-tune α=0.1 / 1 | NPO-fine α=0.1 / 1 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 60% | 80.2 | 88.8 | 79.9 / 79.9 | 79.4 / 13.9 | 86.7 | 84.9 / 81.5 | 88.6 / 88.7 |
| 75% | 80.4 | 89.2 | 80.3 / 80.3 | 80.0 / 13.9 | 84.6 | 85.8 / 86.3 | 81.9 / 85.6 |
| 85% | 78.7 | 89.2 | 79.2 / 78.7 | 78.0 / 13.9 | 84.7 | 81.9 / 87.2 | 81.8 / 82.6 |

SSD gain is within ± 1.6 of zero in all six cells, confirming that N2 is refuted. Retrain at α = 1 is 13.9 at every share because the retain set is then the same conflict-only set regardless of how many aligned images were present. NPO-fine matching DFR is specific to the 60% share (+8.4 ± 1.3, +8.5 ± 1.5); at 75–85% it gains +1.5 to +5.2.

### CelebA oracles (partial: 12/18 units, 2 seeds for the new methods; remainder in Run 3f)

| Method | WGA | Average accuracy |
| --- | --- | --- |
| ERM (5 seeds) | 46.1 | 95.9 |
| Retrain oracle α=0.9 (n=2) | 30.3 | 93.4 |
| Retrain oracle α=1 (n=2) | 2.1 | 87.0 |
| Balanced retrain (n=2) | 73.3 | 94.1 |
| Fine-tune α=0.9 (n=2) | 37.2 | 93.9 |
| Fine-tune α=1 (5 seeds, Run 2) | 25.1 (−21.0 ± 9.4) | 90.5 |
| Held-out DFR (5 seeds) | 87.6 | 90.7 |

Preliminary (2 seeds, intervals not meaningful): on CelebA the forget-set retrain oracle is below ERM already at α = 0.9 and collapses at α = 1, and fine-tuning on the retain set also hurts at α = 1 — unlike Waterbirds and proportion CIFAR, where it was the best method. The balanced oracle (73.3) is far above both and 14 points below held-out DFR. So the claim “fine-tune on conflict data at full discovery is best” does not transfer to CelebA and must be scoped.

Run 3f (`cub-run3f-celeba`, primary Kaggle account, 2.2 h budget) is finishing the third seed. Quota after Batch 3e: main 2.6 h, second 7.8 h.

## Run 4a — Waterbirds with ViT-B/16 (pushed 16 Nov 19:10 IST, running)

Purpose: generality beyond ResNets (step 5). Kaggle (30 h weekly quota). Code change: `--arch vit_b_16` / `vit_b_32` (torchvision ViT, ImageNet weights, wrapped to expose the same head and feature interface; dataset name `waterbirds-vit_b_16`). Local smoke test passed for all eight methods.

Settings: 3 seeds; ERM, held-out DFR, SSD (40 settings), retrain oracle, balanced oracle, fine-tune, NPO-fine, SCRUB at α = 0.1, 0.9, 1; 6 h budget. Questions: is SSD still null, does the retrain oracle show the same cliff between 90% and 100% discovery, and does held-out DFR still win?

Status note: Run 3f (CelebA third seed, primary Kaggle account) was still queued at 19:15 IST.

## Runs in progress, evening of 16 Nov

| Run | Where | Purpose | Settings | Status at 20:10 IST |
| --- | --- | --- | --- | --- |
| 3g `cub-run3g-celeba` | Kaggle | CelebA third seed for the oracle table (replaces Run 3f, which never left the queue on the primary Kaggle account) | seed 2: ERM, retrain oracle and fine-tune at α = 0.9, 1; balanced oracle; 4 h budget | Running |
| 4a `cub-run4a-waterbirds-vit` | Kaggle | Generality: ViT-B/16 on Waterbirds | 3 seeds; ERM, held-out DFR, SSD (40 settings), both oracles, fine-tune, NPO-fine, SCRUB at α = 0.1, 0.9, 1; 6 h budget | Running |
| 4b `modal_run4b.py` | Modal (user's account, $30 credit; launched from the user's PC because Modal cannot be reached from the assistant's workspace) | Confidence intervals for the low-label stacking claim, with the baseline measured in the same run | 5 seeds, α = 1: ERM, held-out DFR, DFR with 2/5/10/25 conflict examples per group, SSD (40 settings) then DFR at the same budgets. Presence CIFAR ρ = 25/50/75% on ResNet-18 and ResNet-9; proportion CIFAR ρ = 75/95% on ResNet-18 as the control. Budget about 5.7 h, expected cost $4–6. Results in Modal volume `cub-results/run4b` | Script delivered; start command given to the user |

Modal check (19:57 IST): token verified, image built in 40 s, test job ran on a Tesla T4 (L4 was unavailable for 7 minutes, so scripts now accept L4, A10G or T4).

Compute left this week: Kaggle main about 2.6 h, second 7.8 h, third about 20 h after the two running jobs; Modal about $24 after Run 4b.

## CelebA oracles, 3 seeds complete (Run 3f, finished 16 Nov \~22:30 IST)

Seeds 0–2, gains paired with ERM, ± 95% CI. Supersedes the 2-seed preliminary table.

| Method | WGA | Gain over ERM | Average accuracy | Per-seed WGA |
| --- | --- | --- | --- | --- |
| ERM | 47.0 | — | 95.9 | 46.7, 43.9, 50.6 |
| Held-out DFR | 87.7 | +40.6 ± 12.1 | 90.8 | 87.7, 89.2, 86.1 |
| Balanced retrain oracle | 72.4 | +25.4 ± 12.1 | 94.2 | 73.3, 73.3, 70.6 |
| Retrain oracle α=0.9 | 30.4 | −16.7 ± 7.7 | 93.4 | 30.6, 30.0, 30.6 |
| Retrain oracle α=1 | 2.2 | −44.8 ± 7.8 | 87.0 | 2.0, 2.1, 2.5 |
| Fine-tune on retain α=0.9 | 36.5 | −10.6 ± 30.3 | 94.0 | 27.2, 47.2, 35.0 |
| Fine-tune on retain α=1 | 18.8 | −28.2 ± 3.3 | 90.2 | 18.0, 17.2, 21.3 |

Confirmed with 3 seeds: on CelebA the forget-set retrain oracle is significantly below ERM at 90% discovery and collapses at 100%; fine-tuning on the retain set at full discovery significantly hurts (−28.2 ± 3.3), the opposite of Waterbirds and proportion CIFAR. The balanced oracle is the only training-based reference that helps, and it stays 15 points below held-out DFR. Run 3g (the duplicate run) also completed and is kept as a cross-check.

Modal Run 4b, first attempt (20:02–20:10 IST): cancelled after 8.5 minutes on an A10G with no training output; cost $0.29. Cause: the script was still downloading CIFAR from the original host (slow) when the run was cancelled from the client side. Fix: CIFAR is now bundled with the script instead of downloaded. No results were lost because none had been produced.

Replication check (Run 3g, another Kaggle account, seed 2 with an independently retrained ERM model) against Run 3f seed 2: ERM 51.1 vs 50.6; retrain α=0.9 29.4 vs 30.6; retrain α=1 2.3 vs 2.5; balanced retrain 75.6 vs 70.6; fine-tune α=0.9 38.3 vs 35.0; fine-tune α=1 22.8 vs 21.3. The CelebA oracle pattern reproduces with an independently retrained base model. Note: re-evaluating the same ERM models in different runs moves WGA by about ± 2 points (46.7/43.9/50.6 vs 48.9/41.7/48.9), so CelebA differences under about 3 points should not be interpreted.

## Model-selection sensitivity from stored per-setting results (no GPU, 16 Nov 23:50 IST)

Source: Run 2j (every hyper-parameter setting was evaluated on test). Gain in test WGA over ERM when the setting is chosen by validation WGA (needs group labels), by plain validation accuracy (no group labels), and for the median setting; last column is the share of settings that beat ERM by more than 1 point.

| Dataset | Method, α | Chosen by val WGA | Chosen by val accuracy | Median setting | Settings that help |
| --- | --- | --- | --- | --- | --- |
| Presence CIFAR ResNet-18 ρ=25% | SSD, 1 | +30.8 | +30.2 | +3.3 | 43% |
| Presence CIFAR ResNet-18 ρ=50% | SSD, 1 | +32.3 | +30.4 | +6.4 | 54% |
| Presence CIFAR ResNet-18 ρ=75% | SSD, 1 | +25.7 | +23.5 | +4.9 | 52% |
| Presence CIFAR ResNet-18 ρ=95% | SSD, 1 | +17.3 | +17.4 | +0.2 | 30% |
| Presence CIFAR ResNet-18 ρ=5% | SSD, 1 | +11.4 | +8.7 | −30.3 | 23% |
| Presence CIFAR ResNet-9 ρ=50% | SSD, 1 | +31.2 | +28.9 | +4.7 | 58% |
| Presence CIFAR ResNet-9 ρ=75% | SSD, 1 | +27.9 | +27.3 | +2.1 | 57% |
| Presence CIFAR ResNet-9 ρ=95% | SSD, 1 | +6.4 | +4.0 | +0.0 | 29% |
| Waterbirds | SSD, 1 | +0.6 | −2.6 | −61.4 | 4% |
| Waterbirds | NPO-fine, 1 | +4.7 | −3.0 | −0.9 | 21% |
| Waterbirds | SCRUB, 1 | +2.4 | +2.4 | −70.0 | 25% |
| Waterbirds | Fine-tune on retain, 1 | +13.5 | +11.4 | +11.1 | 100% |

Reading: on presence CIFAR the SSD gain survives selection by plain accuracy (within 0–3 points of group-aware selection), but it is a tuned result — the median setting gains only 0–6 and about half the settings help. On Waterbirds, unlearning methods lose their small gains without group labels, while fine-tuning on the retain set at full discovery is robust (every setting helps). Caveat: the CIFAR validation set is half conflict copies, so its plain accuracy is not label-free in the strict sense; Run 5a records a training-distribution-weighted validation accuracy to close this.

## Run 5a — leakage-free DFR curves and label-free selection (pushed 16 Nov 23:45 IST, running)

Responds to the external audit. Kaggle, reuses Run 2c models. Code changes: `--pair_split` (both copies of one original CIFAR image go to the same validation half, removing the fit/selection leakage) and a training-distribution-weighted validation accuracy stored for every setting. Smoke test passed.

Settings: presence CIFAR, both backbones, ρ = 5/25/50/75/95%, seeds 0–2; ERM, held-out DFR, DFR with N = 0/2/5/10/25/50/100 conflict examples per group, SSD (40 settings, test metrics per setting) at α = 0.1 and 1, each followed by DFR at full and low label budgets. Budget 3.4 h.

Status of other runs at 23:50 IST: Run 4a (ViT Waterbirds) still running; Modal Run 4b fixed script delivered, relaunch by the user not yet confirmed.

## Results: Run 4a — Waterbirds with ViT-B/16 (finished 16 Nov 23:44 IST, 54/54 units, 3 seeds)

Test WGA; gains paired with ERM, ± 95% CI. ResNet-50 numbers from earlier runs for comparison.

| Method | α=0.1 | α=0.9 | α=1 | ResNet-50 at α=1 |
| --- | --- | --- | --- | --- |
| ERM | 69.9 |  |  | 74.1 |
| Held-out DFR | 85.1 (+15.2 ± 4.9) |  |  | 88.7 |
| Balanced retrain oracle | 78.1 (+8.2 ± 3.8) |  |  | 81.6 |
| Retrain oracle | 71.3 (+1.4 ± 3.2) | 80.2 (+10.2 ± 3.3) | 24.6 (−45.3 ± 5.0) | 15.6 (80.2 at α=0.9) |
| Fine-tune on retain | 72.6 (+2.7 ± 3.8) | 77.7 (+7.8 ± 6.1) | 82.7 (+12.8 ± 4.9) | 85.6–87.7 |
| SSD (40 settings) | 69.9 (−0.1 ± 2.5) | 72.2 (+2.3 ± 5.0) | 70.1 (+0.2 ± 1.2) | 74.6 |
| NPO-fine | 73.4 (+3.4 ± 4.0) | 76.9 (+7.0 ± 9.2) | 75.5 (+5.6 ± 4.1) | 78.9 |
| SCRUB | 72.2 (+2.3 ± 0.9) | 71.6 (+1.7 ± 2.8) | 73.7 (+3.7 ± 2.0) | 78.2 |

Every ResNet-50 finding reproduces on the ViT: SSD is null at all three discovery rates; the retrain oracle peaks at 90% discovery (80.2, identical to ResNet-50) and collapses at 100%; the balanced oracle helps but trails held-out DFR by 7 points; fine-tuning on the retain set at full discovery is the best non-DFR method; held-out DFR is best overall. The Waterbirds conclusions are therefore not specific to convolutional networks.

## Experiment 3 — retain-set dependence diagnostic (no GPU, 17 Nov 01:40 IST)

Definition: on the retain set (training data minus a share α of the aligned samples) compute the association between label and spurious attribute as a signed Cramér’s V: positive = original bias remains, zero = unbiased, negative = bias reversed. Computed from group counts alone (script in `analysis/`). CelebA assumes the aligned groups are blond-female and non-blond-male.

| Dataset | α=0 | 0.5 | 0.75 | 0.9 | 0.95 | 1 | Crosses zero at |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Waterbirds | +0.87 | +0.76 | +0.59 | +0.26 | −0.02 | −1.00 | α ≈ 0.95 |
| CelebA | +0.31 | +0.14 | 0.00 | −0.20 | −0.37 | −1.00 | α ≈ 0.75 |
| Skew CIFAR, 10 colours, ρ=75% | +0.72 | +0.56 | +0.37 | +0.15 | +0.03 | −0.11 | α ≈ 0.96 |
| Skew CIFAR, 10 colours, ρ=95% | +0.94 | +0.89 | +0.81 | +0.62 | +0.43 | −0.11 | α ≈ 0.99 |
| Skew CIFAR, 2 colours, ρ=75% | +0.50 | +0.20 | −0.14 | −0.54 | −0.74 | −1.00 | α ≈ 0.67 |
| Cue CIFAR, ρ=75% | +0.87 | +0.77 | +0.65 | +0.48 | +0.36 | 0.00 | only at α = 1 |
| Cue CIFAR, ρ=95% | +0.97 | +0.95 | +0.91 | +0.81 | +0.70 | 0.00 | only at α = 1 |

What it explains in results we already have:

- **Waterbirds:** unbiased point near 95% discovery, fully reversed at 100%. Measured retrain oracle: best at 90% (80.2 on both backbones), collapse at 100% (15.6 / 24.6).
- **CelebA:** unbiased point already at 75%, so the data is reversed at 90%. Measured: retrain at 90% is already below ERM (30.4 vs 47.0) and 2.2 at 100%; fine-tuning on the retain set hurts. The CelebA “anomaly” is what the diagnostic predicts.
- **Cue CIFAR:** dependence falls to zero only at full discovery and never reverses. Measured: oracle monotonic, best at 100%.
- **Skew CIFAR with 10 colours:** only a mild reversal at 100% (−0.11), because the conflict colour is spread over nine classes. Measured collapse is real but smaller than on Waterbirds (ResNet-18 ρ=75%: 69.9 → 57.5).

Prediction made before the data exists: for two-colour skew CIFAR at ρ=75% the unbiased point is at 67% discovery, so the retrain oracle should already be hurt at 90% and collapse at 100%. Run 5b tests this.

## Run 5b — Experiment 2: interpolation and two-colour controls (pushed 17 Nov 01:35 IST, running)

Kaggle. Code changes, each smoke-tested locally: `--cue_free_frac f` (in skew CIFAR, a share f of the non-aligned images is left untinted instead of conflict-tinted; f = 0 is pure skew, f = 1 is pure cue; the test set is the same for all f) and `--two_color` (binary animal/vehicle task with a two-valued colour cue, in both skew and cue form).

| Part | Datasets | Question |
| --- | --- | --- |
| Interpolation | Skew CIFAR ρ=75%, cue-free share 0, 0.25, 0.5, 0.75, 1 | Does the SSD gain rise smoothly as cue-free samples are added? |
| Two-colour skew | ρ = 75%, 95% | Does SSD stay null with only two cue values (so “type” is not “number of colours”)? Does the oracle collapse where the diagnostic says? |
| Two-colour cue | ρ = 75%, 95% | Does SSD still work with two cue values? |

Settings: ResNet-18, seeds 0–2, α = 0.9 and 1; ERM, held-out DFR (per-image split), balanced oracle, retrain oracle, SSD (40 settings with per-setting test metrics). Budget 4.8 h.

## Results: Run 5a — leakage-free DFR curves and label-free selection (finished 17 Nov 03:00 IST; 108/165 units before the budget ended)

Cue (presence) CIFAR, seeds 0–2. Complete for ρ = 5% and 25% on both backbones; partial for ρ = 50%; ρ = 75% and 95% mostly missing.

### 1. The validation leakage did not matter

Held-out DFR with N conflict examples per group, per-image split (new) vs the old split:

| Setting | N=0 | N=2 | N=5 | N=10 | N=25 | N=50 | N=100 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ResNet-18 ρ=5% | 50.5 / 50.0 | 36.8 / 38.1 | 49.4 / 49.4 | 52.5 / 54.5 | 57.3 / 57.5 | 58.9 / 61.7 | 62.0 / 63.4 |
| ResNet-18 ρ=25% | 30.5 / 26.8 | 23.2 / 19.6 | 35.8 / 31.4 | 38.2 / 35.9 | 44.4 / 45.1 | 49.0 / 49.7 | 53.8 / 52.9 |
| ResNet-9 ρ=25% | 22.1 / 23.0 | 22.5 / 20.4 | 30.4 / 28.0 | 34.8 / 33.4 | 41.3 / 42.4 | 46.7 / 48.5 | 50.2 / 53.5 |
| ResNet-9 ρ=50% | 11.4 / 11.4 | 13.7 / 14.8 | 20.4 / 21.1 | 28.1 / 30.3 | 38.8 / 41.0 | 44.3 / 46.6 | 51.2 / 50.7 |

Differences are within seed noise and have no consistent sign. The earlier DFR curves stand; the per-image split is kept as the protocol from now on.

### 2. SSD’s gain on cue CIFAR does not survive label-free selection

Gain in test WGA over ERM at α = 1, by how the SSD setting is chosen. “Label-free” = validation accuracy weighted by training group proportions, which on cue CIFAR gives zero weight to conflict images (none exist in training).

| Setting | Chosen by val WGA (group labels) | Label-free | Median setting | Best on test |
| --- | --- | --- | --- | --- |
| ResNet-18 ρ=5% | +14.7 ± 13.3 | +5.8 ± 13.0 | +0.7 | +14.7 |
| ResNet-18 ρ=25% | +32.3 ± 10.0 | +19.0 ± 8.0 | +10.9 | +32.5 |
| ResNet-18 ρ=50% (n=2) | +37.7 | −0.2 | +6.4 | +37.7 |
| ResNet-9 ρ=5% | +7.1 ± 17.1 | +1.8 ± 6.4 | +0.3 | +9.2 |
| ResNet-9 ρ=25% | +23.3 ± 12.3 | +0.5 ± 2.2 | +1.1 | +23.3 |
| ResNet-9 ρ=50% (n=2) | +38.8 | +5.8 | +6.2 | +38.8 |

At α = 0.1 the label-free gain is 0.0 in every setting except ResNet-18 ρ=25% (+21.2 ± 10.5).

Reading: this is the outcome the paper plan listed as changing the claim. Without conflict examples in validation, the tuner cannot tell a helpful SSD setting from a harmless one and picks a near no-op; only one of six settings keeps a significant gain. Group-aware selection is close to best-on-test, so the capability exists, but **finding it requires conflict examples**. The earlier “survives selection by plain accuracy” result was because that validation set is half conflict copies. Correction to the earlier note in this log: plain validation accuracy on our CIFAR validation set is not label-free.

### 3. Stacking, paired within the run and leakage-free

Gain of SSD-then-DFR over DFR alone with the same N conflict examples per group (α = 1, 3 seeds, ± 95% CI):

| Setting | N=2 | N=5 | N=10 | N=25 | Full held-out |
| --- | --- | --- | --- | --- | --- |
| ResNet-18 ρ=5% | +4.9 ± 24.3 | −2.9 ± 20.0 | −1.6 ± 19.3 | −2.6 ± 11.3 | −2.4 ± 3.4 |
| ResNet-18 ρ=25% | +17.6 ± 5.0 | +9.1 ± 18.1 | +13.9 ± 19.0 | +13.7 ± 11.1 | +8.2 ± 11.8 |
| ResNet-9 ρ=5% | +7.5 ± 23.2 | +8.4 ± 34.0 | +4.6 ± 32.7 | +7.1 ± 19.4 | +3.6 ± 13.0 |
| ResNet-9 ρ=25% | +14.3 ± 10.2 | +16.1 ± 7.9 | +18.8 ± 5.8 | +15.9 ± 4.5 | +8.3 ± 1.3 |
| ResNet-18 ρ=50% (n=2) | +16.5 | +17.6 | +14.0 | +10.3 | +8.2 |
| ResNet-9 ρ=50% (n=2) | +27.4 | +24.2 | +24.3 | +14.9 | +11.4 |

Reading: the stacking benefit is real at ρ = 25% (significant on ResNet-9 at every budget) and absent at ρ = 5%. Three seeds give wide intervals; Run 4b (5 seeds, ρ = 25/50/75%) decides. Note the SSD setting inside the stack is still chosen with group-labelled validation, so the label count of “SSD + N” is understated.

### Consequence for the paper

The positive half of the title (“works for removable cues”) has to be stated as: unlearning *can* correct a cue shortcut, but only when a small set of conflict examples is available to choose its strength; with none, it is not usable. That makes total conflict-label budget the fair axis for comparing unlearning, DFR and their combination, and it moves the label-efficiency result (S10) from secondary to central for the positive side.

## Results: Run 4b — stacking with 5 seeds, in-run baseline, leakage-free split (finished 17 Nov 04:57 IST, 105/105 units)

α = 1. “Stack gain” = SSD-then-DFR minus DFR alone with the same N conflict examples per group, paired by seed, ± 95% CI. DFR column is DFR alone.

| Setting | ERM | SSD alone (gain) | N=2: DFR, stack gain | N=5 | N=10 | N=25 | Full held-out |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Cue, ResNet-18 ρ=25% | 18.4 | +27.0 ± 5.5 | 21.6, +17.0 ± 4.8 | 35.6, +12.8 ± 9.4 | 34.4, +18.5 ± 10.8 | 44.9, +12.2 ± 4.0 | 56.3, +5.7 ± 3.5 |
| Cue, ResNet-18 ρ=50% | 10.6 | +31.6 ± 6.8 | 18.3, +12.7 ± 6.4 | 28.3, +14.4 ± 9.2 | 35.2, +12.1 ± 8.0 | 39.5, +13.4 ± 6.1 | 53.5, +6.5 ± 2.4 |
| Cue, ResNet-18 ρ=75% | 8.0 | +21.3 ± 8.5 | 14.7, +8.9 ± 8.6 | 21.1, +10.2 ± 5.5 | 28.0, +8.9 ± 4.3 | 35.0, +8.5 ± 5.4 | 44.4, +6.8 ± 5.5 |
| Cue, ResNet-9 ρ=25% | 13.0 | +25.1 ± 10.2 | 22.4, +18.4 ± 4.1 | 26.4, +24.4 ± 10.5 | 30.9, +23.5 ± 10.7 | 40.8, +17.0 ± 5.3 | 54.1, +6.9 ± 7.2 |
| Cue, ResNet-9 ρ=50% | 3.1 | +33.1 ± 14.0 | 14.8, +24.2 ± 5.2 | 20.4, +24.5 ± 7.0 | 28.5, +24.3 ± 7.1 | 39.2, +17.3 ± 8.5 | 50.7, +8.8 ± 5.3 |
| Cue, ResNet-9 ρ=75% | 1.1 | +34.2 ± 6.3 | 8.5, +23.3 ± 8.1 | 17.6, +20.2 ± 3.6 | 25.5, +19.2 ± 3.6 | 33.4, +17.0 ± 4.7 | 44.6, +7.2 ± 2.9 |
| Skew control, ResNet-18 ρ=75% | 59.7 | +0.8 ± 1.3 | 38.6, +7.5 ± 18.7 | 51.0, +1.8 ± 13.1 | 54.0, +3.7 ± 8.6 | 56.3, +3.0 ± 10.4 | 61.5, +1.0 ± 2.0 |
| Skew control, ResNet-18 ρ=95% | 21.4 | +2.3 ± 2.9 | 18.9, +4.3 ± 8.2 | 23.1, +2.2 ± 6.7 | 27.6, −0.9 ± 11.0 | 29.1, +0.4 ± 8.0 | 37.0, +2.7 ± 3.7 |

Findings: on cue shortcuts the stacking gain is significant in all six settings at every limited label budget (+8.5 to +24.5) and shrinks to +6 to +9 with the full held-out set; on the skew control it is never significant. SSD alone with 5 seeds: +21 to +34 on cue, null on skew. SSD plus 2 conflict examples per group reaches roughly what DFR alone needs 25 per group for (for example ResNet-9 ρ=50%: 14.8 + 24.2 = 39.0 vs 39.2 at N=25). Caveat that remains: the SSD setting is still picked with group-labelled validation, so the label count on the stacking side is understated; the equal-total-budget experiment fixes this.

## Results: Run 5b — interpolation and two-colour controls (finished 17 Nov \~06:00 IST; interpolation parts cut to 2 seeds by their budgets)

ResNet-18, ρ as stated, gains over ERM, ± 95% CI where 3 seeds exist.

**Two-colour controls (3 seeds)**

| Dataset | ERM | Held-out DFR | Balanced oracle | Retrain α=0.9 | Retrain α=1 | SSD α=1 | SSD α=1, label-free |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Two-colour skew ρ=75% | 96.3 | +0.5 ± 0.5 | +1.3 ± 0.1 | −3.9 ± 0.3 | −96.3 ± 0.3 | −0.1 ± 0.6 | −0.2 ± 1.1 |
| Two-colour skew ρ=95% | 86.1 | +9.5 ± 0.9 | +7.4 ± 1.0 | +6.9 ± 0.9 | −85.8 | +0.0 ± 0.1 | −0.7 ± 1.6 |
| Two-colour cue ρ=75% | 5.7 | +89.9 ± 5.4 | +1.2 ± 2.5 | +21.3 ± 13.8 | +65.2 ± 4.8 | +13.7 ± 16.4 | +5.6 ± 17.1 |
| Two-colour cue ρ=95% | 2.3 | +90.2 ± 6.6 | +0.8 ± 2.2 | +7.1 ± 8.9 | +77.6 | +10.0 ± 35.7 | +1.6 ± 3.6 |

- The diagnostic’s advance prediction held: two-colour skew at ρ=75% (unbiased point at 67% discovery) is already hurt at 90% (−3.9 ± 0.3) and collapses to 0 at 100%; at ρ=95% (unbiased point at about 95%) retraining still helps at 90% (+6.9 ± 0.9) and collapses at 100%.
- SSD is null on two-colour skew, so the skew null is not an artefact of ten colours. Limitation: at ρ=75% the ERM model barely uses the colour (WGA 96.3), so only the ρ=95% row (ERM 86.1, DFR +9.5) is a real test, with about 10 points of headroom.
- On two-colour cue the retrain oracle is monotonic and very large at full discovery (+65, +78); SSD’s gain is positive but not significant with 3 seeds (at α=0.9, ρ=75%: +44.5 ± 31.5), and vanishes under label-free selection. The balanced oracle does nothing for cue shortcuts (+1), the mirror image of its role on skew shortcuts.

**Interpolation, skew CIFAR ρ=75% with a share of the non-aligned images left cue-free (2 seeds, no intervals)**

| Cue-free share | ERM | SSD α=1 | SSD label-free | Retrain α=0.9 | Retrain α=1 | Balanced oracle | Held-out DFR |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 (pure skew) | 58.2 | −0.0 | −0.1 | +10.0 | +0.4 | +9.2 | +2.2 |
| 0.25 | 54.0 | +2.8 | +0.0 | +12.6 | +7.6 | +11.2 | +7.1 |
| 0.5 | 52.3 | +3.8 | +0.3 | +12.4 | +5.4 | +11.9 | +7.3 |
| 0.75 | 41.1 | +4.7 | −0.1 | +19.7 | +24.1 | +19.1 | +12.8 |
| 1 (pure cue) | 5.9 | +19.6 | +0.2 | +16.6 | +25.3 | +2.7 | +28.8 |

SSD’s gain rises with the cue-free share (0, +2.8, +3.8, +4.7, +19.6) but not smoothly: most of it appears only when no conflict-tinted training images remain. The retrain oracle switches from “peak at 90%” to “best at 100%” between shares 0.5 and 0.75. Two seeds only; needs a third seed and intervals before it is a figure.

## Key findings for the paper

The paper rests on seven findings, each listed with the numbers it would cite and the runs behind them (updated 14 Nov, 4:30 PM).

1. **A few conflict examples beat corrective unlearning on every dataset.** Held-out DFR: Waterbirds 88.7 vs best unlearning 78.7 (ERM 74.1); CelebA 89.8 vs 59.0 (ERM 46.1); CIFAR ResNet-18 63.2 → 38.5 vs SSD 53.1 → 13.8 across ρ 5 → 95%. Crossover: 5 per minority group on Waterbirds, 2 on CelebA, 2–5 on ResNet-9 and 10–50 on ResNet-18 beat SSD (Runs 2d, 2f, 2h, CelebA 2).
2. **Feature-level unlearning has a narrow niche.** SSD helps only on the synthetic colour shortcut without conflict data, and fails at extreme prevalence: ResNet-18 closes 114% → 23% of the gap with the wide grid, ResNet-9 drops to 15% at ρ=95%. On Waterbirds and CelebA no SSD setting moves WGA (75.1 vs 74.1; 46.1 = ERM) (Runs 2c, 2i).
3. **Under partial discovery, unlearning beats exact retraining.** At α=0.1, retraining without the flagged samples stays at ERM level (ResNet-18 ρ=25%: 18.2 vs 17.3) while SSD reaches 40.8. With full discovery, retraining wins on CIFAR but collapses on Waterbirds (15.6, 240 images left) (Runs 2g, 2h).
4. **Restorability tracks conflict data, not unlearning.** Under ART k=4, every method that never saw conflict data falls to about ERM: Waterbirds unlearning 26–34, exact retraining 29–36, ERM 28.4. Held-out DFR keeps 83.0 on Waterbirds but drops to 65.4 on CelebA, matching the ART paper (Runs 2b, 2g, 2h, CelebA 2).
5. **SSD's failure is not contamination or collateral damage.** A retain-only reference changes nothing; under 2.4% of class-critical weights are dampened; the tint stays 94–99% decodable; forget and reference Fisher maps share parameters (rank correlation 0.57–0.86) (Run 2e).
6. **Label-pushing methods are unsafe; careful tuning makes them safe but useless.** Full NegGrad/NPO collapse to 0–17 on Waterbirds and BadT to 1–5 on CelebA. The fine grids sit at ERM (Runs 1, 2b, CelebA 2).
7. **Run 1's CelebA unlearning success was an artefact.** NPO head-only at 82.8 came from a forget set that marked non-blond women as bias-aligned; with the correct set it gives 44.6 (CelebA 2).

Candidate title: *"Forgetting Is Not Fixing: Corrective Unlearning Under Clean-Label Spurious Bias" (see Positioning)*.

## Bugs, fixes and caveats

Two results are void or provisional: Run 2a's ResNet-9 numbers (void) and Run 1's CelebA unlearning numbers (forget set wrong). Everything else stands.

| Found in | Issue | Effect | Fix |
| --- | --- | --- | --- |
| Run 2a | ResNet-9 lacked the standard ×0.125 logit scale | Training diverged in epoch 1 (loss 6 instead of about 2); clean acc 57%. All Run 2a ResNet-9 results void | Added the logit scale; clean acc 89.9% in Run 2c |
| Run 2a | Multi-class DFR used 10 liblinear OvR fits per C per resample | About 35 min per unit on CIFAR, about 4 GPU-hours wasted | L1 softmax regression solved by FISTA on GPU, about 1 s per fit; matched liblinear within 2 points on a synthetic test |
| Run 2a | The time budget was checked only between units | Runs overshot by about 2 h | Launcher hard-stops 5 min past budget; finished units are saved |
| Run 1 review | CelebA F-BA marked non-blond women as bias-aligned | Run 1 CelebA unlearning numbers invalid; re-run done (Run 2 CelebA): NPO-head's 82.8 was an artefact | align\_thresh = 0.75; re-run pending |
| Run 1 | CelebA full run stopped at 234/240 units on budget | Delayed results | Resume run finished 240/240; resume now symlinks files over 10 MB |
| Smoke tests | CelebA re-read 20k test JPEGs per unit | Slow runs | Eval image cache, single-pass features, prewarm |
| Smoke tests | LR scheduler divided by zero on tiny data | Smoke test crash | drop\_last only when n > batch; steps ≥ 1 |
| Smoke tests | Stanford Waterbirds URL unreachable from Kaggle | No data | Switched to the Kaggle-hosted copy |

**Caveats a reviewer will raise**

- *DFR on CIFAR was handicapped:* resolved by Run 2d. With conflict examples, DFR beats SSD everywhere, so any CIFAR claim must state which data each method gets.
- *The tint is also a corruption:* the ρ=0 control shows conflict accuracy of 68–76% against 90% clean. Report CIFAR healing against the ρ=0 ceiling.
- *Healed % on CIFAR uses a near-zero denominator:* DFR ≈ ERM there, so Run 2c reports healing against ρ=0 instead.
- *All hyperparameters are chosen on validation WGA with group labels:* this is the standard oracle-validation setting; state it explicitly.

## Compute and run log

All runs use Kaggle 2×T4 notebooks running the same script (cub.py) with one worker per GPU, per-unit JSONL results and resumable checkpoints. Newest first.

| Run | Kaggle notebook | Account | What it ran | Units | Status |
| --- | --- | --- | --- | --- | --- |
| 2k | cub-run2k-rem | Kaggle | REM and its NPO baseline (official code + our adapter), CIFAR ResNet-9 at 50% capacity, ρ 5–95%, α 0.1/1, 3 seeds | 60 jobs | Queued (14 Nov) |
| 2j | cub-run2j-cifar | Kaggle | Widest SSD grid (70 settings), SSD then DFR, per-candidate test metrics; both backbones | 60 + 60 | Queued (14 Nov) |
| 2j | cub-run2j-waterbirds | Kaggle | Same as above, plus model-selection sensitivity for NegGrad/NPO fine grids, SCRUB, fine-tune and BadT | about 70 | Queued (14 Nov) |
| 2i | cub-run2i-cifar / -waterbirds / -celeba | both | Wide SSD grid (40 settings) and the FMD-style baseline | 120 / 20 / 20 | Complete |
| 2 CelebA | cub-run2-celeba | Kaggle | CelebA with the fixed forget set, fine grids, ART, probes, held-out DFR, conflict-count curve | 205 | Complete, about 5 GPU-h |
| 2h | cub-run2h-waterbirds | Kaggle | Waterbirds held-out DFR, conflict-count curve, exact-retraining oracle | 60 | Complete |
| 2f/2g | cub-run2f-cifar | Kaggle | CIFAR conflict-count curve, exact-retraining oracle, ρ=0 + DFR control | 312 | Complete |
| 2e | cub-run2e-cifar | Kaggle | SSD mechanism: retain-only reference, collateral damage, post-SSD probes | 150 | Complete |
| 2d | cub-run2d-cifar | Kaggle | Held-out DFR (DFR's original protocol) on colored CIFAR | 30 | Complete |
| 2c | cub-run2c-cifar | Kaggle | Colored CIFAR, ρ 0.05–0.95 × ResNet-9 / ResNet-18, plus the ρ=0 control | 606 | Complete, about 5 GPU-h |
| 2a | cub-run2-cifar | Kaggle | Same sweep, before the fixes | 279 of 600 | ResNet-9 void; superseded by 2c |
| 2b | cub-run2-waterbirds | Kaggle | Waterbirds fine grid, ART, probes, SSD ratios | 170 | Complete |
| 1 | cub-full-waterbirds, cub-full-celeba, cub-resume-celeba | Kaggle | Waterbirds and CelebA, 9 methods × 5 α × 5 seeds | 240 + 240 | Complete (CelebA unlearning numbers superseded) |
| Smoke | cub-smoke, cub-smoke-celeba, cub-run2-smoke | Kaggle | End-to-end checks | — | Passed |

Quota: each account has 30 GPU-h per week, resetting Saturday 05:30 IST. Retrieve outputs with `kaggle kernels output <user>/<slug> -p <dir> --file-pattern ".*(stdout|\.jsonl)$"` and summarise with `python3 cub.py --dataset <ds> --summarize --out_dir <dir>/cub_runs`.

## Related work pointers

CUPID and PURGE are the closest prior work; ART supplies the restoration test we adopt. Full notes are in paper\_notes.md and gap\_analysis.md.

| Paper | Venue | Why it matters here |
| --- | --- | --- |
| [Goel et al., Corrective Machine Unlearning](https://arxiv.org/abs/2402.14015) | TMLR 2024 | Defines the corrective setting with partial discovery; retraining is not the gold standard |
| [Foster et al., SSD](https://arxiv.org/abs/2308.07707) | AAAI 2024 | The Fisher-ratio method we test; poison ratios near 800 |
| [Schoepf et al., Potion](https://arxiv.org/abs/2406.09173) | DMLR 2024 | XLF/PTN extension of SSD for poisons |
| [Schoepf et al., REM](https://arxiv.org/abs/2505.17730) | ICLR 2026 | Discovery × regularity taxonomy; REM IDEAL upper bound |
| [Zhao et al., What makes unlearning hard](https://arxiv.org/abs/2406.01257) | NeurIPS 2024 | Entanglement score and C-proxy |
| [Kirichenko et al., DFR](https://arxiv.org/abs/2204.02937) | ICLR 2023 | Last-layer retraining: 92.9 Waterbirds / 88.3 CelebA |
| [LaBonte et al., SELF](https://arxiv.org/abs/2309.08534) | NeurIPS 2023 | Last-layer retraining with few or no group labels |
| [Jain et al., D3M](https://arxiv.org/abs/2406.16846) | NeurIPS 2024 | Datamodel-based debiasing by removing training points |
| [Chen et al., FMD](https://arxiv.org/abs/2310.12560) | NeurIPS 2023 | Debiasing via machine unlearning |
| [Hakemi et al., FCR](https://arxiv.org/abs/2501.14182) | arXiv 2025 | A single-weight edit reaches 93.2 on Waterbirds |
| [Kwon et al., CUPID](https://arxiv.org/abs/2602.21773) | arXiv 2026 | "Shortcut unlearning" under bias, in a class-forgetting setting |
| [Sarker et al., PURGE](https://arxiv.org/abs/2608.29996) | NeurIPS 2026 (Evaluations & Datasets track) | Spurious-correlation unlearning in vision-language models |
| [ART (Association Restoration Test)](https://arxiv.org/abs/2607.05726) | arXiv 2026 | The restoration test our ART metric follows |
| [Bias-Aware Machine Unlearning](https://arxiv.org/abs/2509.07456) | arXiv 2025 | Unlearning used for debiasing; position against it |
| [CUTS](https://arxiv.org/abs/2511.18660) | arXiv 2025 | Shortcut removal; check its overlap with our locus claim |
| SAP | AAAI 2025 | Bias mitigation baseline to cite |
| [Cognac](https://arxiv.org/abs/2412.00789) | arXiv 2024 | Closest prior art for expanding the discovered set (graphs) |

The ART, Bias-Aware MU, CUTS and SAP rows were added after paper\_notes.md was written and are not yet read in full.

## Positioning against prior work (literature search, 14 Nov 2026)

No 2022–2026 paper compares corrective-unlearning methods against annotation-matched DFR under clean-label spurious bias, and none sweeps shortcut prevalence. Those two results (findings 1 and 3) are the paper's novelty. The ART result is already in print (Lu & Ji, July 2026), so it becomes supporting evidence, not a headline. Every external number below came from an extraction tool; check it against the PDF before citing. The full report with all 30 citations was sent separately ("Unlearning spurious bias prior work").

| Our claim | Verdict | Closest prior evidence | What stays ours |
| --- | --- | --- | --- |
| Held-out DFR beats SSD everywhere when conflict examples exist | **Open** | No unlearning paper compares to DFR. FMD (NeurIPS 2023) compares only to DRO/LfF/LDR; CUPID, PURGE and Bias-Aware MU have no DFR | First annotation-matched benchmark of unlearning vs DFR |
| DFR on training annotations (no conflicts) barely helps; SSD wins only there | DFR half shown; SSD half open | SELF: retraining on training data gives 71.0 vs 72.6 (Waterbirds), 66.9 vs 66.3 (CelebA) | Unlearning has a niche exactly in the zero-conflict regime |
| DFR beats the ρ=0 no-shortcut model (63.2 vs 51.5) — confounded: ρ=0 model + DFR reaches 67.3 (Run 2f) | Partly shown | DFR 92.9 vs balanced-trained model 91.9 on Waterbirds; ColorMNIST at correlation 1.0: DFR 77.3 vs ERM 0 (Kirichenko et al.) | A controlled, prevalence-swept version |
| SSD's healing decays with ρ while its Fisher tail stays heavy | **Open** | Goel et al. explain SSD's success by outlier importances; REM: SSD fails on low-regularity corruptions; LfF/DCWP: training-time debiasers decay with bias strength | Prevalence as a new axis; outlier Fisher mass is not sufficient |
| Amplification restores the shortcut after unlearning, not after DFR | Largely shown | ART (2607.05726), Waterbirds at β=2: WGA drops A-NegGrad+ 28.2, A-SSD 20.1, A-SCRUB 19.9, DFR 9.2. But DFR drops 24.5 on CelebA | Corrective (flagged-subset) forget sets; collapse to ERM level; colored-CIFAR |
| NegGrad/NPO collapse on clean-label bias data | Partly shown, partly contradicted | Supports: Bias-Aware MU (GA collapses CelebA to 50.9 acc), REM. Against: PURGE's NPO reaches 79.6 WGA (CLIP, Waterbirds); CUPID's NegGrad debiases | Scope to clean-label CNNs with a forget set of bias-aligned samples |
| α=0.1 is often enough | Partly shown | Published CelebA DFR 89.4 ± 0.9; AFR: DFR falls below ERM at about 5 Waterbirds annotations | Minor; report budgets as example counts |

**The gap we can own:** in clean-label spurious bias, conflict examples, not forgetting, are what fixes the model. The regime where forgetting helps shrinks predictably as the shortcut becomes more common. At low ρ the tint behaves like a poison trigger (rare, regular), where corrective unlearning is known to work; at high ρ it is a population-level feature. Waterbirds sits at the high end: SSD 74.0 vs ERM 74.1.

**Reviewer risks to pre-empt**

- *ART overlap:* adopt ART as the evaluation standard; report β=2 next to k=4; run it on CelebA.
- *Missing baselines:* FMD, REM, Potion, and retraining on the retain set (the exact-unlearning oracle).
- *Budget as a fraction:* α means flagged training samples for unlearning and annotations for DFR. Report labelled and minority-group counts.
- *Model selection:* every method here picks hyperparameters on validation WGA with conflict examples, so SSD's "no-conflict" win still uses conflict data for selection. Add results under average-accuracy selection.
- *Our Waterbirds DFR (87.0) is below the published 92.9:* check the L1 setting, the held-out split and α.
- *Confounds:* compare against the ρ=0 model plus DFR on the same held-out set. Add probe baselines (random-init, ρ=0, pretrained-only), because a colour tint is decodable from almost any CNN.

**Recommended framing:** a regime map that ends in a decision rule, not a takedown. If you can label a few conflict examples, retrain the last layer. If you cannot, SSD on a flagged subset helps while the shortcut is rare, and the result should be checked with ART.

Candidate titles:

1. *Forgetting Is Not Fixing: Corrective Unlearning Under Clean-Label Spurious Bias* (recommended)
2. *When Can Unlearning Remove a Shortcut? Prevalence, Conflict Data, and Last-Layer Retraining* (if Run 2e gives a clean mechanism)
3. *A Few Conflict Examples Beat Forgetting: Benchmarking Corrective Unlearning Against DFR for Spurious Correlations*

**Must-cite core:** Goel et al. (TMLR 2024), SSD (AAAI 2024), REM (ICLR 2026), FMD (NeurIPS 2023), ART (preprint 2026), CUPID (likely AAAI 2026), PURGE (preprint 2026), DFR (Kirichenko et al.), Izmailov et al. (NeurIPS 2022), SELF, AFR (ICML 2023), Idrissi et al. (CLeaR 2022), D3M (NeurIPS 2024), LfF, DCWP (CVPR 2023), Zhao et al. (NeurIPS 2024). Do not cite "SAP spurious-aware pruning": the SAP that exists (AAAI 2025) is about label noise.

## Open questions and to-do

The next run is the CelebA re-run on the primary Kaggle account. It reuses Run 1's trained models, so the another Kaggle account cannot run it without a copy of those checkpoints.

**Experiments**

- [ ] CelebA re-run with the fixed forget set (align\_thresh 0.75), fine grid, ART and probes — done (no unlearning method beats ERM)
- [ ] CIFAR: DFR with a small held-out set that contains conflict examples — done in Run 2d (DFR wins everywhere)
- [ ] SSD failure mechanism at high ρ: does SSD start dampening class-relevant weights? Measure clean-accuracy loss and the overlap between selected parameters and class-important parameters
- [ ] Third dataset (MultiNLI, or a medical-imaging set) to show the locus result beyond images with colour and background shortcuts
- [ ] Combined method: DFR-style annotations guiding feature-level editing (SSD restricted to the layers where probes peak)
- [ ] H4 (presence- vs proportion-driven corruption) and H5 (discovery expansion; compare with Cognac and backdoor detectors)

**Added after the literature search (priority order)**

- [ ] Run 2e: SSD with a retain-only reference Fisher, plus collateral-damage diagnostics — done (neither explanation holds; next: extend SSD's grid)
- [ ] Exact-unlearning oracle: retrain on the retain set at each α — CIFAR done (Run 2g); Waterbirds done (Run 2h)
- [ ] Conflict-count crossover: DFR with 0–100 conflict examples per group vs SSD — CIFAR done (Run 2f); Waterbirds done (Run 2h); CelebA running. Still to do: SSD followed by DFR
- [ ] Baselines: FMD-style head Newton step with counterfactual pairs — running in Run 2i (exact pairs on CIFAR; same-class, different-attribute pairs on Waterbirds/CelebA, a weaker approximation). REM needs models trained from scratch with example-tied dropout, so it is feasible only for ResNet-9 on CIFAR via the official code (github.com/google-deepmind/rem); Potion still to do
- [ ] ART alignment: β=2 under ART's gating, CelebA, and a relearning attack (fine-tune on a few aligned samples)
- [ ] Probe controls: random-init, ρ=0 and pretrained-only backbones; within-class probes as in ART
- [ ] Deconfound: ρ=0 model + DFR on the same held-out set — done (67.3 / 63.8; the "beats no-shortcut" claim is dropped)
- [ ] Model-selection sensitivity: worst-group, average-accuracy and label-free selection; CUPID-style whole-class NegGrad
- [ ] Check every external number against the PDFs; confirm venues for DFR, SELF, CUPID, Potion, CUTS

**Added after Runs 2e–2g**

- [ ] Extend SSD's grid (α\_SSD 0.1–50 × λ 0.01–1, 40 settings) — done (Run 2i): it helps at high ρ on CIFAR, not on Waterbirds/CelebA; winners still touch the new edge
- [ ] Report ART drops relative to the exact-retraining oracle, — Waterbirds oracle done: the gap is not unlearning-specific (Run 2h)
- [ ] Headline figure: smallest N beating SSD vs ρ, for both backbones

**Run 2j (pushed 14 Nov, 3:40 PM): running**

- [ ] Widest SSD grid (70 settings) — done: saturated, matches 40 settings within noise
- [ ] SSD then DFR — done: stacks on CIFAR (+2 to +8), neutral on Waterbirds
- [ ] Model-selection sensitivity: test metrics logged for every candidate of SSD, NegGrad/NPO fine grids, SCRUB, BadT and fine-tune on Waterbirds, then compare worst-group-val vs average-accuracy-val selection — done: average-accuracy selection drops unlearning below ERM on Waterbirds (BadT 51.5, NegGrad 66.3, NPO 71.2, SSD 71.6)
- [ ] REM on ResNet-9 CIFAR via the official code — running as Run 2k (pushed 4:15 PM). Our adapter adds a colour-tint corruption (tinted samples = the corrupted set, labels unchanged), test WGA on clean + other-colour copies, and per-channel storage of the dropout masks (same semantics, about 12 GB less memory). Settings follow the paper: ResNet-9 at 50% capacity (ETDfix50mem0drop1), 4,000 pretraining and 980 unlearning iterations, batch 512, p\_cut 0.2. REM and its NPO baseline at α 0.1 and 1.0, ρ 5–95%, 3 seeds. Caveat: REM uses fixed paper hyper-parameters with no WGA-based selection, and its model has half the capacity of our ResNet-9, so compare against its own pretrained model

**Writing**

- [ ] Update paper\_notes.md and gap\_analysis.md with ART, Bias-Aware MU, CUTS and SAP, and the narrowed scope
- [ ] Main figures: the locus flip (Waterbirds vs CIFAR), the ART bars, the prevalence curve
- [ ] One-page summary for Barnali Mam
- [ ] Workshop draft (ICLR/ICML workshops), then TMLR

**Housekeeping**

- [ ] Expire both Kaggle API tokens once the runs are done; both were pasted in chat

**Open questions**

- Why does SSD fail at high prevalence if the ratio tail does not shrink?
- Is the Waterbirds/CIFAR flip about how the attribute is encoded (pretrained-feature background vs learned colour), or about how spread out the shortcut is across parameters?
- Is ResNet-9's tiny ratio tail (p99.9 about 2) a sign that SSD works through many small dampenings rather than a few outliers?
