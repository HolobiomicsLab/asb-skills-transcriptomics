---
name: eqtl-retrieval-auroc-benchmarking
description: Use when you have peak-gene scores from one or more causal-inference
  configurations (GrID-Net, linear Granger, correlational baselines) on the SHARE-seq
  mouse skin dataset and need a quantitative gold-standard comparison against significant
  eQTLs from the eQTL Catalogue.
when_to_use_negative:
- The dataset has no matching tissue eQTL catalog available to serve as a perturbation-mimicking
  gold standard (GrID-Net associations are causal hypotheses validated only by such
  cross-referencing).
- You want to validate predictions against physical chromatin contacts instead of
  genetic perturbation evidence — the paper used 5C data for SHARE-seq only because
  mouse skin Hi-C was unavailable, and that is a different evaluation.
- The biological system is not a dynamic differentiation process, so no pseudotemporal
  DAG (and thus no GrID-Net or branch decomposition) can be constructed for the configurations
  being benchmarked.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: GrID-Net
  role: Produces the Granger causal peak-gene scores on the cell-state DAG whose eQTL-retrieval
    AUROC is benchmarked; also runs on full DAG vs branch-subset DAGs in the ablation.
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint RNA+ATAC cell representations (gridnet.schema_representations)
    used to build the kNN DAG underpinning the GrID-Net scores being evaluated.
  repo: https://github.com/rs239/schema
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:6d62866286ebe627db94824ce19fa9e6b11b3cfbb3b6ee62cbcb8cd1fa7cc965
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/alexw16/gridnet
  tools:
  - GrID-Net
  - Schema
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# eqtl-retrieval-auroc-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark locus-gene causal-scoring methods (e.g., GrID-Net on a cell-state DAG versus linear Granger baselines on full trajectories or individual branches) by how well their scores retrieve significant skin eQTLs (P < 1e-3) from the eQTL Catalogue, evaluated via AUROC of eQTL retrieval using a distance-aware logistic regression.

## When to use

Use when you have peak-gene scores from one or more causal-inference configurations (GrID-Net, linear Granger, correlational baselines) on the SHARE-seq mouse skin dataset and need a quantitative gold-standard comparison against significant eQTLs from the eQTL Catalogue.

## When NOT to use

- The dataset has no matching tissue eQTL catalog available to serve as a perturbation-mimicking gold standard (GrID-Net associations are causal hypotheses validated only by such cross-referencing).
- You want to validate predictions against physical chromatin contacts instead of genetic perturbation evidence — the paper used 5C data for SHARE-seq only because mouse skin Hi-C was unavailable, and that is a different evaluation.
- The biological system is not a dynamic differentiation process, so no pseudotemporal DAG (and thus no GrID-Net or branch decomposition) can be constructed for the configurations being benchmarked.

## Inputs

- SHARE-seq mouse skin RNA-seq AnnData with gene TSS annotations (chr_no, txstart in adata.var)
- SHARE-seq ATAC-seq AnnData with peak coordinates (chr_no, start, end in adata.var)
- Candidate peak-gene pairs from gridnet.identify_all_peak_gene_link_candidates(distance_thresh=1e6)
- GrID-Net Granger statistics (results_df from gridnet.run_gridnet) and/or linear Granger scores for each configuration (full DAG, cortex, medulla, IRS branches)
- Significant skin eQTLs (P < 1e-3) from the eQTL Catalogue

## Outputs

- AUROC of eQTL retrieval for each scoring configuration
- Significance statistics for AUROC differences between configurations
- Comparison table (e.g., dag_vs_branches_auroc.csv) of linear-Granger versus GrID-Net AUROCs

## How to apply

For each configuration (GrID-Net on the full three-branch DAG built with gridnet.construct_dag using n_neighbors=15 and pseudotime_algo='dpt', or linear Granger on the full trajectory or the cortex/medulla/IRS branches separately), rank candidate peak-gene pairs restricted to peaks within 1 Mb of the gene TSS (distance_thresh=1e6). Assemble the gold standard as peak-gene pairs overlapping significant skin eQTLs (P < 10^-3) from the eQTL Catalogue. Train a logistic regression that combines each configuration's causal scores with log genomic distance as a feature, mirroring the paper's evaluation of distance-dependent regulatory effects. Score every configuration by AUROC of eQTL retrieval, compute significance statistics for pairwise differences (e.g., GrID-Net versus each linear-Granger configuration), and tabulate the results (e.g., dag_vs_branches_auroc.csv). Correct application reproduces the Fig. 2c ablation: GrID-Net on the full DAG significantly exceeds all linear-Granger configurations.

## Related tools

- **GrID-Net** (Produces the Granger causal peak-gene scores on the cell-state DAG whose eQTL-retrieval AUROC is benchmarked; also runs on full DAG vs branch-subset DAGs in the ablation.) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint RNA+ATAC cell representations (gridnet.schema_representations) used to build the kNN DAG underpinning the GrID-Net scores being evaluated.) — https://github.com/rs239/schema

## Examples

```
results_df = gridnet.run_gridnet(X,Y,X_feature_names,Y_feature_names,candidate_XY_pairs,dag_adjacency_matrix)
```

## Evaluation signals

- Every configuration yields better-than-random eQTL retrieval (AUROC > 0.5) on significant skin eQTLs (P < 1e-3).
- GrID-Net on the full DAG achieves a significantly higher AUROC than every linear-Granger configuration (full trajectory or individual cortex/medulla/IRS branches), reproducing the Fig. 2c ablation.
- The logistic regression trains with both the causal score and log genomic distance features, so the evaluation is distance-aware rather than purely score-ranked.
- Candidate pairs are correctly restricted to peaks within 1 Mb (distance_thresh=1e6) of gene TSS before scoring, matching the paper's candidate set.
- Output table (dag_vs_branches_auroc.csv) contains one AUROC per configuration plus pairwise significance statistics.

## Limitations

- eQTL cross-referencing only mimics perturbation-based validation; the inferred locus-gene associations remain causal hypotheses that need experimental perturbation to confirm.
- For SHARE-seq, mouse skin Hi-C was unavailable, so any complementary chromatin-contact evaluation had to rely on 5C data.
- Tissue matching is imperfect: the mouse skin eQTL gold standard reflects that tissue only, and comparable benchmarks in other tissues may explain only a subset of regulatory phenotypes.

## Evidence
