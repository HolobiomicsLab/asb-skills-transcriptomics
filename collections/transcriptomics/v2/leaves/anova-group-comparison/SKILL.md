---
name: anova-group-comparison
description: Use when comparing a continuous performance score (e.g., sub-network
  detection sensitivity on simulated RNA-seq data) across three or more competing
  methods or simulation conditions and you need an omnibus test of mean differences.
when_to_use_negative:
- Only two methods or two conditions are being compared, where a direct pairwise test
  suffices instead of an omnibus ANOVA
- The evaluation data are real tumor/normal samples without an embedded ground-truth
  sub-network, since sensitivity cannot be computed against a known true module
- The aim is module detection itself (use runmodule global/local) rather than statistically
  comparing detection performance across tools
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: fRNC
  role: Primary module-detection method under benchmark; detects RBP-ncRNA sub-networks
    via global maximum significant RNC search to be scored for sensitivity
  repo: https://github.com/BioinformaticsSTU/fRNC
- name: edgeR
  role: Provides the differential expression statistics used to weight network nodes
    before module detection on each simulated replicate
- name: devtools
  role: Installs the fRNC package from GitHub so runmodule and the simulation benchmark
    can be executed in R
  repo: https://github.com/leiming8886/fRNC
- name: R
  role: Execution environment for simulation, module detection, sensitivity scoring
    and the ANOVA comparison
  repo: https://github.com/leiming8886/fRNC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:66de0bf4c19785b2ad619140b9a59d6e51550dccf0e599e4aa94f059075f7db3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/fRNC
  repo_url: https://github.com/BioinformaticsSTU/fRNC
  tools:
  - R
  - devtools
  - edgeR
  - fRNC
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# anova-group-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Method for benchmarking competing sub-network detection tools (fRNC vs WGCNA vs LncACT) by running ANOVA on per-replicate sensitivity scores computed across factorial simulation conditions, supporting the claim that fRNC detects more accurate and robust RBP-ncRNA sub-networks.

## When to use

Use when comparing a continuous performance score (e.g., sub-network detection sensitivity on simulated RNA-seq data) across three or more competing methods or simulation conditions and you need an omnibus test of mean differences.

## When NOT to use

- Only two methods or two conditions are being compared, where a direct pairwise test suffices instead of an omnibus ANOVA
- The evaluation data are real tumor/normal samples without an embedded ground-truth sub-network, since sensitivity cannot be computed against a known true module
- The aim is module detection itself (use runmodule global/local) rather than statistically comparing detection performance across tools

## Inputs

- SimSeq-simulated RNA-seq count matrices (200 replicates per abundance-ratio x subnetwork-size condition)
- TCGA ESCA and BRCA expression data (case.exp_miRNA, control.exp_miRNA, case.exp_rna, control.exp_rna; brca_miRNA_re_se, brca_RNA_re_se, brca_pro_re_se)
- edgeR differential expression results (logFC, PValue per gene/miRNA)
- Known embedded true sub-networks used as detection ground truth
- Per-replicate sensitivity table with columns for method, abundance_ratio, true_size

## Outputs

- ANOVA table of sensitivity by method, abundance-ratio and subnetwork-size factors
- Per-method sensitivity summaries for each simulation condition
- Robustness (recovery consistency) rates from 100 half-sample resampling repeats per method

## How to apply

First generate SimSeq-based simulated RNA-seq count datasets with RBP-to-miRNA abundance fold ratios of 0.5, 1, 2 and 5, embedded true sub-network sizes of 30, 50 and 70 nodes, and 200 simulations per condition (ESCA and BRCA settings from TCGA). For each replicate, score differential expression of miRNAs, mRNAs and lncRNAs with edgeR, then detect modules with fRNC runmodule(method='global', FDR=1e-5) alongside WGCNA and LncACT as competitors. Score each detected sub-network against the known embedded true sub-network to obtain a sensitivity value per method and condition. Compare sensitivity across methods with ANOVA, grouping by abundance-ratio and true-subnetwork-size factors. Assess robustness by repeating detection on 100 half-sample resampled datasets and recording per-method recovery consistency before drawing the comparison conclusion.

## Related tools

- **fRNC** (Primary module-detection method under benchmark; detects RBP-ncRNA sub-networks via global maximum significant RNC search to be scored for sensitivity) — https://github.com/BioinformaticsSTU/fRNC
- **edgeR** (Provides the differential expression statistics used to weight network nodes before module detection on each simulated replicate)
- **devtools** (Installs the fRNC package from GitHub so runmodule and the simulation benchmark can be executed in R) — https://github.com/leiming8886/fRNC
- **R** (Execution environment for simulation, module detection, sensitivity scoring and the ANOVA comparison) — https://github.com/leiming8886/fRNC

## Examples

```
summary(aov(sensitivity ~ method + abundance_ratio + true_size, data = sim_bench))  # sim_bench: 200 replicates x {fRNC, WGCNA, LncACT} x ratios {0.5,1,2,5} x sizes {30,50,70}
```

## Evaluation signals

- ANOVA on sensitivity shows a significant method effect, with fRNC mean sensitivity exceeding WGCNA and LncACT across abundance-ratio (0.5/1/2/5) and size (30/50/70) conditions
- Each condition cell contains exactly 200 simulated replicates as specified in the benchmark design
- Sensitivity values are computed by scoring detected sub-networks against the known embedded true sub-networks, not by proxy metrics
- Robustness records per-method recovery consistency over 100 half-sample resampled datasets, with fRNC expected to be the most consistent
- The tabulated pattern reproduces the reported finding that fRNC detects more accurate and robust sub-networks with scalability

## Limitations

- The benchmark relies on SimSeq-based simulated data with embedded true sub-networks (sizes 30, 50, 70 nodes), so conclusions are conditioned on those simulation assumptions
- Only four RBP-to-miRNA abundance fold ratios (0.5, 1, 2, 5) were tested; performance outside this ratio range is not established
- The article's discussion section content is absent from the provided source, so caveats stated by the authors post-results cannot be verified

## Evidence

- [abstract] fRNC outperforms competitors in sensitivity: "When compared with existing tools, f RNC can detect more accurate and robust sub-network with scalability."
- [methods] edgeR differential expression scoring: "The R package edgeR was used to analyze the differentially expressed miRNAs, mRNAs and lncRNAs."
- [readme] fRNC global module detection call: "res.list_global <- runmodule(network = interac_rbp, gene2weight, method = "global", FDR = 1e-5)"
- [readme] Package installation via devtools: "devtools::install_github("leiming8886/fRNC",ref = "main")"
