---
name: resampling-robustness-evaluation
description: 'Use when comparing module-detection methods (fRNC, WGCNA, LncACT) for
  stability: you have ground-truth embedded sub-networks or matched case/control RNA-seq
  counts and need recovery consistency across resamples plus sensitivity comparisons
  via ANOVA.'
when_to_use_negative:
- The goal is de novo discovery of condition-specific RNCs on a single real dataset
  rather than benchmarking detection stability against a known truth
- Input data are protein-level RNCs from precomputed BRCA differential results (brca_*_re_se)
  without any resampling or ground-truth design
- Only one sample group is available, so half-sample resampling is undefined
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: fRNC
  role: Module detection via runmodule(method='global') on scored RBP-ncRNA networks;
    the method under robustness evaluation
  repo: https://github.com/leiming8886/fRNC
- name: edgeR
  role: Differential expression analysis of simulated miRNA, mRNA and lncRNA counts
    to produce node scores
- name: devtools
  role: Installs the fRNC package from GitHub (devtools::install_github)
  repo: https://github.com/leiming8886/fRNC
- name: WGCNA
  role: Competing module-detection method used as baseline in sensitivity/robustness
    comparison
- name: LncACT
  role: Competing module-detection method used as baseline in sensitivity/robustness
    comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:087d7ca97aefbd2708ec2821431fce3afda6fc0dc783e70c50ad54c6de6ab8f4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/fRNC
  repo_url: https://github.com/leiming8886/fRNC
  tools:
  - LncACT
  - R
  - WGCNA
  - devtools
  - edgeR
  - fRNC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# resampling-robustness-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol for assessing whether detected RBP-ncRNA sub-networks (RNCs) are robust to sampling variability, by re-running fRNC module detection on 100 half-sample resampled datasets and comparing recovery consistency against WGCNA and LncACT on SimSeq-simulated RNA-seq data.

## When to use

Use when comparing module-detection methods (fRNC, WGCNA, LncACT) for stability: you have ground-truth embedded sub-networks or matched case/control RNA-seq counts and need recovery consistency across resamples plus sensitivity comparisons via ANOVA.

## When NOT to use

- The goal is de novo discovery of condition-specific RNCs on a single real dataset rather than benchmarking detection stability against a known truth
- Input data are protein-level RNCs from precomputed BRCA differential results (brca_*_re_se) without any resampling or ground-truth design
- Only one sample group is available, so half-sample resampling is undefined

## Inputs

- Simulated RNA-seq count matrices (SimSeq, seeded from TCGA ESCA/BRCA case.exp_miRNA, control.exp_miRNA, case.exp_rna, control.exp_rna)
- Known embedded true sub-network node sets (30/50/70 nodes per simulation condition)
- High-stringency RBP-ncRNA interaction network from ENCORI (CLIP evidence >= 3)
- edgeR differential expression results (logFC, PValue per gene type)

## Outputs

- Per-method sensitivity values for each abundance-ratio and sub-network-size condition
- Per-method robustness values (recovery consistency over 100 half-sample resamples)
- ANOVA comparison of sensitivity across fRNC, WGCNA and LncACT
- Detected sub-network lists per replicate (optionally exported as XGMML via saveNetwork)

## How to apply

Generate SimSeq-based simulated RNA-seq count datasets seeded from TCGA ESCA and BRCA expression data, varying RBP-to-miRNA abundance fold ratios (0.5, 1, 2, 5) and embedded true sub-network sizes (30, 50, 70 nodes) with ~200 simulations per condition. For each replicate, score differential expression with edgeR (DEGs on miRNA and RNA count matrices), build node weights with combinp, load high-stringency ENCORI interactions via interStringency, and detect modules with runmodule(method='global', FDR=1e-5), running WGCNA and LncACT in parallel. Score each detected sub-network against the embedded truth for sensitivity, then compare methods with ANOVA grouped by abundance-ratio and sub-network-size conditions. Finally, repeat detection on 100 half-sample resampled datasets and record per-method recovery consistency as the robustness metric.

## Related tools

- **fRNC** (Module detection via runmodule(method='global') on scored RBP-ncRNA networks; the method under robustness evaluation) — https://github.com/leiming8886/fRNC
- **edgeR** (Differential expression analysis of simulated miRNA, mRNA and lncRNA counts to produce node scores)
- **devtools** (Installs the fRNC package from GitHub (devtools::install_github)) — https://github.com/leiming8886/fRNC
- **WGCNA** (Competing module-detection method used as baseline in sensitivity/robustness comparison)
- **LncACT** (Competing module-detection method used as baseline in sensitivity/robustness comparison)

## Examples

```
res.list_global <- runmodule(network = interac_rbp, gene2weight, method = "global", FDR = 1e-5)
```

## Evaluation signals

- Sensitivity of fRNC-detected sub-networks (measured against embedded true sub-networks) exceeds that of WGCNA and LncACT across the 0.5/1/2/5 fold-ratio and 30/50/70-node conditions
- ANOVA on sensitivity grouped by abundance-ratio and sub-network-size conditions supports a significant method effect
- Recovery consistency over 100 half-sample resamples is higher and more stable for fRNC than for the competing tools
- Each simulated condition yields ~200 replicate detections with edgeR-based scoring completed without failure before sensitivity scoring
- Detected modules reproduce the published pattern that fRNC yields 'more accurate and robust sub-network with scalability'

## Limitations

- Robustness estimates depend on SimSeq simulation parameters (fold ratios 0.5-5, 30-70 node truths); results may not transfer to other simulation designs or real datasets without ground truth
- Half-sample resampling halves effective sample size, which can reduce detection power for all methods and shrink true-positive recovery
- BRCA case 2 relies on precomputed differential results stored in the package (brca_miRNA_re_se, brca_RNA_re_se, brca_pro_re_se), so re-simulation from raw counts requires rebuilding those objects

## Evidence
