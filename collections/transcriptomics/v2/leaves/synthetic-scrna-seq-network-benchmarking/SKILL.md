---
name: synthetic-scrna-seq-network-benchmarking
description: Use when a new GRN inference method must be validated against baselines
  on synthetic scRNA-seq data where the true TF-target network (BoolODE toggle-switch
  ground truth) is known, so AUROC/AUPRC/EPR can be computed exactly.
when_to_use_negative:
- Input is real scRNA-seq data without a known ground-truth network — use context-specific
  inference (e.g. with scATAC-seq/motif-filtered base GRNs) rather than synthetic
  benchmarking
- Target gene set is large (>200 genes) or candidate TFs are not pre-selected — scGATE
  is documented as applicable only to a small (<200) gene subset with refined candidate
  TFs, making a synthetic head-to-head of full-network tools misleading
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scGATE
  role: Method under benchmark; infers TF-target edges and Boolean logic gates from
    scRNA-seq via Bayesian Hill-function modeling
  repo: https://github.com/CompBioIPM/scGATE
- name: CellOracle
  role: Baseline GRN inference tool run on the same simulated expression matrices
    for comparison
- name: BoolODE
  role: Generates synthetic scRNA-seq data from toggle-switch networks providing the
    ground truth
- name: R 4.1.3
  role: Runtime environment for the scGATE package and benchmark scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:9be907938296d81e069aa0cd9ec9bb48d8f0be733c73732f62e18e569eb281f3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  repo_url: https://github.com/CompBioIPM/scGATE
  tools:
  - BoolODE
  - CellOracle
  - R 4.1.3
  - VGAM
  - arrow
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# synthetic-scRNA-seq-network-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark GRN inference tools (scGATE vs. GRNBOOST2, LEAP, CellOracle) on BoolODE-simulated single-cell expression data with known ground-truth networks, scoring predicted TF-target edges by AUROC and EPR across cell types, network-complexity variants, and dropout levels.

## When to use

Use when a new GRN inference method must be validated against baselines on synthetic scRNA-seq data where the true TF-target network (BoolODE toggle-switch ground truth) is known, so AUROC/AUPRC/EPR can be computed exactly.

## When NOT to use

- Input is real scRNA-seq data without a known ground-truth network — use context-specific inference (e.g. with scATAC-seq/motif-filtered base GRNs) rather than synthetic benchmarking
- Target gene set is large (>200 genes) or candidate TFs are not pre-selected — scGATE is documented as applicable only to a small (<200) gene subset with refined candidate TFs, making a synthetic head-to-head of full-network tools misleading

## Inputs

- BoolODE-simulated toggle-switch scRNA-seq expression matrix (ClusterI.csv from scGATE example_data)
- Network-complexity and decoy-TF variants of the BoolODE toggle-switch simulation
- Known BoolODE ground-truth TF-target network per simulated condition

## Outputs

- benchmark_performance.csv with AUROC, AUPRC/EPR per tool per cell type and dropout level
- scGATE outputs: log10 Bayes Factor logic gates and BF-ranked TF-target edge lists

## How to apply

Load the BoolODE-simulated toggle-switch Cluster I expression matrix (ClusterI.csv) from the scGATE example_data folder, including its network-complexity and decoy-TF variants. Preprocess with scRNA_seq_preprocessing() using library_size_normalization="False" so counts are quantile-rescaled into the (0,1) interval, and drop genes with standard deviation <= 0.20 to retain informative genes. Run scGATE_logic() with run_mode="simple" and top_gates=1 to get logic gates ranked by log10 Bayes Factor, and scGATE_edge() with h_act=7 for BF-ranked TF-target edge lists. Run each baseline (e.g. CellOracle) on the same simulated expression matrices and conditions, then score every tool's predicted edges against the BoolODE ground-truth network, computing AUROC and AUPRC/EPR per tool and per condition. Assemble results into benchmark_performance.csv and confirm scGATE exceeds baselines, consistent with the article's reported AUROC > 0.98 superiority.

## Related tools

- **scGATE** (Method under benchmark; infers TF-target edges and Boolean logic gates from scRNA-seq via Bayesian Hill-function modeling) — https://github.com/CompBioIPM/scGATE
- **CellOracle** (Baseline GRN inference tool run on the same simulated expression matrices for comparison)
- **BoolODE** (Generates synthetic scRNA-seq data from toggle-switch networks providing the ground truth)
- **R 4.1.3** (Runtime environment for the scGATE package and benchmark scripts)

## Examples

```
normalized_counts <- scRNA_seq_preprocessing(data = ClusterI, library_size_normalization = "False"); gates <- scGATE_logic(data = normalized_counts, top_gates = 1, run_mode = "simple"); edges <- scGATE_edge(data = normalized_counts, h_act = 7)
```

## Evaluation signals

- benchmark_performance.csv shows scGATE AUROC/AUPRC exceeding GRNBOOST2, LEAP, and CellOracle in every cell type and dropout condition
- scGATE achieves AUROC > 0.98 on the synthetic toggle-switch datasets, matching the article's reported result
- Genes with standard deviation <= 0.20 were removed before inference; surviving genes have expression rescaled into the (0,1) interval
- Rankings are stable across the network-complexity and decoy-TF variants of the toggle switch
- Baseline tools are evaluated on identical expression matrices and scored against the identical BoolODE ground truth (no per-tool preprocessing differences)

## Limitations

- Synthetic toggle-switch ground truth may miss interactions such as large protein complexes, which are absent from both the model and the ground-truth networks
- scGATE is applicable only to a small (<200) subset of genes with a refined candidate TF list, so benchmark scope is constrained
- Boolean logic involving more than ~5 regulators is not decodable due to overfitting, bounding the complexity of networks fairly testable
- No software version numbers are stated for baseline tools in this section, which hampers exact reproducibility of the comparison

## Evidence
