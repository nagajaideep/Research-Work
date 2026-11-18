# Lit notes: unlearning for spurious correlations / corrective unlearning (searched 17 Nov 2026)

Caveat: details below were read through an auto-summarising fetch of arXiv HTML/PDF. Re-check every number and quote against the PDF before citing. Repos verified with `git ls-remote` only (exist, public); contents not inspected.

## Verified papers

| Paper | Venue | Link | Code |
|---|---|---|---|
| Goel, Prabhu, Torr, Kumaraguru, Sanyal, *Corrective Machine Unlearning* | TMLR 2024 | arXiv 2402.14015 | github.com/drimpossible/corrective-unlearning-bench |
| Foster, Schoepf, Brintrup, *Fast Machine Unlearning Without Retraining Through Selective Synaptic Dampening* (SSD) | AAAI 2024 | arXiv 2308.07707 | github.com/if-loops/selective-synaptic-dampening |
| Schoepf, Foster, Brintrup, *Potion: Towards Poison Unlearning* | TMLR (OpenReview 4eSiRnWWaF; venue inferred, check) | arXiv 2406.09173 | github.com/if-loops/towards_poison_unlearning (exists; link not found in paper text I saw) |
| Chen et al. (Ruizhe Chen, ..., Zuozhu Liu), *Fast Model Debias with Machine Unlearning* (FMD) | NeurIPS 2023 | arXiv 2310.12560 | **No official repo found** (paper, GitHub search) |
| Kwon, Kim, Lee, Lee, Lee, Kim, *Easy to Learn, Yet Hard to Forget: Towards Robust Unlearning Under Bias* (CUPID) | AAAI 2026 | arXiv 2602.21773 | **No repo found** |
| Aylapuram, Elluru, Agarwal, *Bias-Aware Machine Unlearning: Towards Fairer Vision Models via Controllable Forgetting* | ICCV 2025 Workshop (U&ME) | arXiv 2509.07456 | none found |
| Sarker, Shah, Sultan, Jang, Zhu, Khanduri, *PURGE: Partition-Aware Unlearning for Removing Spurious Correlations in Large VLMs* | NeurIPS 2026 | arXiv 2608.29996 | none found |
| Schoepf, Mozer, Mitchell, Brintrup, Kaissis, Kairouz, Triantafillou, *Redirection for Erasing Memory (REM)* | ICLR 2026 | arXiv 2505.17730 | github.com/google-deepmind/rem |
| Cheng, Li, Liu, Zhang, *Machine Unlearning under Retain-Forget Entanglement* | ICLR 2026 | arXiv 2603.26569 | "available here" in paper; URL not recovered |
| De Min, Roy, Lathuilière, Ricci, Mancini, *Group-robust Machine Unlearning* | TMLR 2025 | arXiv 2503.09330 | github.com/tdemin16/group-robust_machine_unlearning |
| Lu, Ji, *Association Restoration Test: Revealing Restorable Shortcuts after Unlearning* (ART) | arXiv, Jul 2026 | arXiv 2607.05726 | none found |
| Jain, Hamidieh, Georgiev, Ilyas, Ghassemi, Mądry, *Data Debiasing with Datamodels (D3M)* | NeurIPS 2024 | arXiv 2406.16846 | github.com/MadryLab/D3M |

Also relevant: Mulchandani & Kim, *Severing Spurious Correlations with Data Pruning* (ICLR 2025 spotlight, 2503.18258); Hakemi et al., *Post-hoc Spurious Correlation Neutralization with Single-Weight Fictitious Class Unlearning* (2501.14182); Fazla & Hashemi, *Memorization-Guided Dataset De-Biasing* (2606.02830); Siddiqui et al., *From Dormant to Deleted* (NeurIPS 2025, 2505.22310; relearning by retain-only fine-tuning); *Safety Mirage* (ICLR 2026, 2503.11832; unlearning vs spurious correlations in VLM safety tuning); Idrissi et al., *Simple data balancing* (CLeaR 2022, 2110.14503; SUBG/RWG); LaBonte et al., *Last-layer retraining with fewer annotations* (NeurIPS 2023, 2309.08534).

## What each does (one line)

- **Goel**: BadNet poison + interclass confusion (IC), CIFAR-10 R9 / CIFAR-100 WRN / PCam. No clean-label bias. SSD heals poison at 10% discovery, fails on IC; retrain-without-deletion-set needs >=80% discovery for poison. Baselines: RewoD, CF, SSD, BadT, SCRUB.
- **Potion**: SSD variant (XLF importance + PTN search). Poison only (BadNet, sine, moving trigger). Excludes label-swap because no shared trigger.
- **REM**: 2D task space = discovery rate x "statistical regularity" (C-score): random mislabel / IC / poison. No spurious-correlation task. Potion works at high regularity only; retain-set methods fail at partial discovery.
- **FMD**: influence-function Newton update of the **last layer only**, using an **external counterfactual set**. Not forget-set unlearning. Colored MNIST, CelebA, Adult, StereoSet. An ablation variant (Eq. 8) pairs harmful samples with bias-conflicting ones.
- **CUPID**: class-wise forgetting under 99.5% bias; goal is forgetting, not debiasing. Observes "shortcut unlearning" (unlearning erases the shortcut, conflict-group accuracy of the forgotten class goes up). Waterbirds, BAR, Biased NICO++. Retrain = gold standard, unquestioned.
- **Bias-Aware MU**: forget set = biased subset (CUB pose bin, CIFAR-10 red patch on 50% of birds, CelebA gender-smiling). ResNet-18. GA, LoRA, SCRUB-style, FMD, hard unlearning as oracle. Metrics DP/EO, not WGA. No DFR.
- **Group-robust MU**: forget set non-uniform over groups; plain Retrain degrades the dominant forget group; proposes **Retrain+reweight as the reference** and MIU. CelebA, Waterbirds, FairFace. Goal is preserving robustness, not debiasing.
- **ART**: audits A-NegGrad+, A-SCRUB, A-SalUn, **A-SSD** (association-objective variants, no sample forget set) on Waterbirds, CelebA, SpuCoDogs, ISIC. References: ERM, **Balanced Retrain**, DFR, GroupDRO, JTT. Shortcut stays linearly readable and restorable after unlearning; Balanced Retrain is the low-restoration reference.
- **D3M**: TRAK attribution, remove harmful (mostly majority) examples, retrain. Waterbirds 87.2 vs DFR 89.0; CelebA-Blond 90.0. Needs group-labelled validation.
- **PURGE**: GA/KL/NPO with LoRA on structured forget/retain partitions built from counterfactual images; COCO hallucination benchmarks + Waterbirds CLIP.

## Our claims vs prior work

**C1 (shortcut type decides) — positive half anticipated, negative half open.**
Our presence-type tint is a clean-label trigger; Goel and Potion already show SSD-family removes trigger-like corruption at low discovery and fails on non-localised corruption (IC), and REM formalises this as a "regularity" axis. Reviewers will call presence-CIFAR "poison without the label flip". Bias-Aware MU informally contrasts "localized" (patch) vs "diffuse" (pose) vs "entrenched" (CelebA) bias, but with no controlled construction, different methods per dataset, and DP/EO only. Nobody found runs a controlled presence vs proportion pair, an SSD null on Waterbirds/CelebA with a sample forget set, or a prevalence control. ART's A-SSD is not comparable (association objective). Verdict: partially shown; the proportion-type null plus the aligned-share sweep is the new part. Must cite Goel/REM as the origin of the taxonomy.

**C2 (retrain-on-retain mis-specified, cliff at 100%) — partially shown; the cliff curve is open.**
Goel and REM say retrain is not a gold standard, but for the opposite reason (partial discovery leaves corruption in the retain set); ours fails at full discovery. Group-robust MU is the closest: plain retrain shifts the retain group distribution, and a reweighted retrain is proposed as reference. ART already uses Balanced Retrain as its reference without arguing for it. Bias-Aware MU notes in passing that patch absence can become a new shortcut, yet keeps hard unlearning as oracle; CUPID keeps Retrain. Idrissi (SUBG) is the classical background that full removal of a majority group is not balancing. No paper found reports WGA of retrain vs discovered fraction with a 90% peak and 100% collapse. "Balanced retrain is the right reference" cannot be sold as new practice; the non-monotone curve can.

**C3 (unlearning + few conflict labels are complements, ~10x) — open.**
No paper found measures a label-budget curve for unlearning followed by last-layer retraining. Nearest: FMD is itself a last-layer correction using counterfactual/conflict data; ART Sec. 5.7 retrains a head after A-NegGrad+ for a restoration test; LaBonte et al. is the label-efficiency baseline for DFR. Nothing contradicts it. Our own evidence is the weak point (3 seeds, cross-run baseline).

**Scoop warnings.** (i) Evidence item 7 is ART's result, on the same datasets, with SSD included; ours is a replication. (ii) "Shortcut unlearning" as a term is CUPID's, with a different meaning (side effect of class forgetting). (iii) "Bias-aware unlearning with forget set = biased samples" as a setting exists (ICCVW 2025), on a CIFAR-10 patch construction close to our presence type.

## Reviewer expectations

Datasets: Waterbirds, CelebA (both, always), plus one of SpuCoDogs / BAR / NICO++ / MetaShift / MultiNLI or CivilComments; CIFAR-10 R9 and CIFAR-100 WRN-28x10 for corrective unlearning. Baselines: Retrain, NegGrad(+), SCRUB, BadT, SalUn, SSD/Potion, NPO, L1-sparse, DELETE; debiasing side GroupDRO, JTT, DFR, RWG/SUBG, D3M. Metrics: WGA, average accuracy, EO/DP, MIA, restoration (ART) or relearning (Siddiqui et al.). Missing Potion and REM-tuned baselines will be noticed; faithful FMD and CUPID have no public code, which is a defensible reason to omit or reimplement with a disclaimer.
