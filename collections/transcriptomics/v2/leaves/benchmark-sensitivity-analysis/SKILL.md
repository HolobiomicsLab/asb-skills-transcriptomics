---
name: benchmark-sensitivity-analysis
description: Use when comparing module-detection or sub-network-inference methods
  on RNA-seq data where ground-truth sub-networks are known or embedded, and sensitivity/robustness
  must be quantified and statistically compared.
when_to_use_negative:
- When no ground-truth (embedded or curated) sub-network is available, since sensitivity
  against a known target cannot be computed
- When the goal is detecting modules in a single real dataset rather than comparing
  competing detection methods on simulated replicates
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3361
tools:
- name: fRNC
  role: Module-detection method under evaluation; runmodule with method='global' detects
    RNCs from edgeR-scored networks
  repo: https://github.com/leiming8886/fRNC
- name: edgeR
  role: Differential expression scoring of simulated miRNAs, mRNAs and lncRNAs feeding
    combinp-derived gene2weight node scores
- name: devtools
  role: Used to install the fRNC R package from GitHub prior to benchmarking
  repo: https://github.com/leiming8886/fRNC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:7c16158c094456748c6c0d777863643b18131251abea3f5b33b3c15e08ba4b3d
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
  - R
  - devtools
  - edgeR
  - fRNC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# benchmark-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A head-to-head benchmarking procedure that measures how sensitively and robustly module-detection methods (fRNC vs WGCNA, LncACT) recover known embedded RBP-ncRNA sub-networks in SimSeq-simulated RNA-seq data, compared via ANOVA and 100-times half-sample resampling.

## When to use

Use when comparing module-detection or sub-network-inference methods on RNA-seq data where ground-truth sub-networks are known or embedded, and sensitivity/robustness must be quantified and statistically compared.

## When NOT to use

- When no ground-truth (embedded or curated) sub-network is available, since sensitivity against a known target cannot be computed
- When the goal is detecting modules in a single real dataset rather than comparing competing detection methods on simulated replicates

## Inputs

- SimSeq-simulated RNA-seq count matrices with embedded true sub-networks (grid of abundance fold ratios 0.5/1/2/5 and sizes 30/50/70 nodes, 200 replicates per condition)
- TCGA ESCA and BRCA expression datasets (case.exp_miRNA, control.exp_miRNA, case.exp_rna, control.exp_rna, brca_miRNA_re_se, brca_RNA_re_se, brca_pro_re_se)
- RBP-ncRNA interaction network from ENCORI (high stringency, CLIP evidence >= 3)

## Outputs

- Per-replicate sensitivity values for fRNC, WGCNA, and LncACT under each condition
- ANOVA comparison of sensitivity across methods grouped by abundance-ratio and true-subnetwork-size
- Robustness (recovery consistency) statistics from 100 half-sample resampling repetitions
- Tabulated benchmark summary confirming whether fRNC outperforms competing tools

## How to apply

Generate SimSeq-based simulated RNA-seq count datasets spanning a parameter grid of RBP-to-miRNA abundance fold ratios (0.5, 1, 2, 5) and embedded true sub-network sizes (30, 50, 70 nodes), with 200 simulations per condition for both ESCA and BRCA TCGA settings. For each replicate, score differential expression with edgeR, run fRNC's global module detection (runmodule with method='global', e.g. FDR=1e-5), and run WGCNA and LncACT as competitors. Score each detected sub-network against the embedded true sub-network to compute sensitivity per method and condition, then compare methods with ANOVA grouped by abundance-ratio and subnetwork-size conditions. Assess robustness by repeating detection on 100 times half-sample resampled datasets and recording recovery consistency; validate that fRNC achieves higher, more consistent detection than WGCNA and LncACT.

## Related tools

- **fRNC** (Module-detection method under evaluation; runmodule with method='global' detects RNCs from edgeR-scored networks) — https://github.com/leiming8886/fRNC
- **edgeR** (Differential expression scoring of simulated miRNAs, mRNAs and lncRNAs feeding combinp-derived gene2weight node scores)
- **devtools** (Used to install the fRNC R package from GitHub prior to benchmarking) — https://github.com/leiming8886/fRNC

## Examples

```
res.list_global <- runmodule(network = interac_rbp, gene2weight, method = "global", FDR = 1e-5)
```

## Evaluation signals

- Sensitivity is recomputable per condition: each detected sub-network must be matched against the known embedded true sub-network of 30/50/70 nodes
- ANOVA across methods and conditions should reproduce the reported pattern that fRNC shows higher sensitivity than WGCNA and LncACT
- Robustness check: recovery consistency across the 100 half-sample resampled datasets should be higher for fRNC than competitors
- Each of the 4 abundance ratios x 3 sub-network sizes x 200 simulations must complete for both ESCA and BRCA settings (no missing condition cells)
- Detected RNCs from runmodule should honor the specified FDR threshold (e.g. 1e-5 global, 1e-20 in BRCA protein-level example)

## Limitations

- Conclusions are tied to simulated SimSeq datasets with the tested parameter grid (fold ratios 0.5-5, sizes 30-70); other regimes may not generalize
- Comparative claims depend on the specific competing implementations of WGCNA and LncACT and their parameterization
- The fRNC global module search with strict FDR thresholds (1e-5 to 1e-20) can be computationally heavy; the BRCA case pre-computed DE results internally to save running time

## Evidence

- [abstract] fRNC detects more accurate and robust sub-networks with scalability: "When compared with existing tools, f RNC can detect more accurate and robust sub-network with scalability."
- [methods] edgeR differential analysis: "The R package edgeR was used to analyze the differentially expressed miRNAs, mRNAs and lncRNAs."
- [methods] global module detection call: "res.list_global <- runmodule(network = interac_rbp, gene2weight, method = "global", FDR = 1e-5)"
- [readme] high stringency interaction filtering: "with a high stringency, where the number of the supported CLIP experimental evidence is 3 or greater."
- [readme] installing fRNC from GitHub: "devtools::install_github("leiming8886/fRNC",ref = "main")"
