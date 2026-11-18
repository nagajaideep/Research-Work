# Referee report: corrective unlearning under clean-label spurious bias

Basis: `evidence.md` (17 Nov 2026) plus a brief novelty check (abstracts only). Nearest prior work: Goel et al., Corrective Machine Unlearning (TMLR 2024, arXiv 2402.14015); Kirichenko et al., DFR (2204.02937); "Easy to Learn, Yet Hard to Forget" / CUPID (2602.21773), which already studies unlearning under spurious bias on Waterbirds. I did not read the full CUPID paper; do so before writing.

## 1. Publishability (after ~3–4 weeks of fixes)

| Venue | P(accept) | Driver |
|---|---|---|
| TMLR | 55–65% | TMLR asks "are claims supported", not "is it surprising". A scoped negative/boundary study qualifies if the leakage and selection problems are fixed and claims are narrowed. |
| ICLR/NeurIPS/ICML main | 8–12% | No mechanism, the positive result exists only on a synthetic tint, half the findings are already known (DFR wins; retrain needs full discovery), and CUPID occupies the neighbouring space. Rises to about 20% only with a natural cue-type dataset plus a mechanism. |
| Top-venue workshop | 80–85% | Clean message, honest negatives. |

## 2. Core claim

Pick (a), restated: **forget-set unlearning corrects a shortcut only when deleting the flagged samples leaves a retain distribution in which the cue is uninformative; when deletion merely re-weights or reverses an attribute everyone carries, both approximate unlearning and its exact oracle fail.** Result (b) becomes the natural-data pillar of (a), not a separate headline.

- (b) alone: the α=1 cliff is close to arithmetic (the retain set is 100% conflict, so the correlation is reversed). Reviewers will call it obvious. Only the non-monotonicity and the CelebA drop at α=0.9 are non-trivial.
- (c): this is Kirichenko et al. restated; and it is false at 5/group on Waterbirds (tie with ERM).
- (d): 3 seeds, unpaired baseline, synthetic only, absent on Waterbirds. Not a headline.

Fallback: if the natural cue-type test (5.1) fails, build a TMLR-only paper on (b).

## 3. Titles (ranked)

1. "When Does Forgetting Fix Bias? Corrective Unlearning Works for Removable Cues, Not for Skewed Attributes"
2. "Deleting the Aligned Samples Is Not Debiasing: Limits of Corrective Unlearning under Spurious Correlation"
3. "The Retrain Oracle Is the Wrong Target for Bias: A Study of Corrective Unlearning under Clean-Label Shortcuts"

## 4. Cut, demote, artefact risk

Cut or move to appendix: stacking / "10x label saving" (until the paired 5-seed rerun); FMD-style (not faithful; ± 11.1); half-capacity REM; restoration test (own variant only); fine-tune-on-retain as a "method" (it is fine-tuning on conflict samples, sign flips on CelebA); "DFR dominates" as a contribution (keep as baseline).

Artefact risks:
- **Presence-CIFAR SSD gains (+13 to +38): hyper-parameter oracle.** 40–70 settings picked on val WGA, median setting gains 0–6, half the settings hurt. Add CIFAR validation leakage. CIs of ± 7–9 are already wide.
- **SSD "+0.0 ± 0.0" on CelebA** is the selector choosing a no-op, not a measured failure. Report whether any setting moved the weights and the best-on-test upper bound.
- **Proportion-CIFAR null**: confounded with 10-way cue (conflict tint points to a specific wrong class), not only with "skew".
- **Presence = clean-label backdoor.** The positive result may just reproduce Goel et al.'s poison result.
- **Waterbirds share sweep**: subsampling changes dataset size alongside share.
- **Stacking**: baseline from a different run.

## 5. Required experiments (ordered)

TMLR minimum:
1. Fix leakage; select SSD without group labels; report median/quantiles over settings and a fixed-budget random search. Confirm: presence gain ≥ +10 with CI excluding 0 under label-free selection. Kill: gain falls to the median (0–6).
2. Interpolation: proportion CIFAR with a growing fraction of cue-free images (0→100%), and a 2-colour proportion variant. Confirm: SSD gain rises smoothly with cue-free fraction, 2-colour stays null. Kill: 2-colour proportion works (then "type" is really cue cardinality).
3. 5 seeds on all headline cells, paired in-run.
4. Faithful CUPID baseline on Waterbirds. Kill for novelty: CUPID succeeds with flagged aligned samples only.

Main-track additions:
5. **One natural cue-type dataset** (ISIC with colour patches, MultiNLI negation words, or watermark/tag artefacts). The taxonomy predicts SSD and retrain-at-α=1 work there. Confirm: both gain. Kill: null, meaning the positive result is a tint artefact.
6. Mechanism: two-feature linear model predicting the cliff location and sign; plus evidence that the forget-vs-retain Fisher contrast localises cue parameters only in the cue-type case.
7. Second architecture/dataset family for the positive side (ViT on cue-type).
8. A standard relearning attack if "removal" is claimed anywhere.

## 6. Hardest questions

1. "Isn't presence-type just a backdoor, already covered by Goel et al.?" Answer with experiment 5 and an explicit statement that the contribution is the boundary: benchmark biases are not trigger-like.
2. "Isn't the cliff trivial?" Answer with the mechanism (6), the α=0.9 non-monotonicity, and the fact that the balanced oracle is also below DFR, so no retraining reference is adequate.
3. "Did you tune SSD into success on CIFAR and fail to tune it on Waterbirds?" Answer with experiment 1, per-dataset hyper-parameter sensitivity plots, and best-on-test upper bounds for the nulls.

## 7. Framing

Not sharp enough. "Presence" vs "proportion" describes dataset construction, not a property one can test on new data. Proposed definition: let a be the spurious attribute, F the full set of aligned samples, R = D \ F.
- **Cue-type (removable)**: a has a null value in-distribution, and on R the mutual information I(y; a) ≈ 0. Deletion yields an unbiased dataset.
- **Skew-type (confounded)**: every sample carries some value of a, and on R the dependence I(y; a) stays large with reversed sign. Deletion yields the opposite bias.

Report the diagnostic "residual label–attribute dependence on the retain set as a function of α" for every dataset; it predicts the cliff. Names: **cue shortcuts** vs **skew shortcuts** (or "trigger-like" vs "group-imbalance").
