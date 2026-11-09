---
name: eqtl-groundtruth-benchmark-evaluation
description: Use when a peak-gene (locus-gene) inference method must be quantitatively
  benchmarked against eQTL-supported ground-truth pairs on paired scRNA-seq/scATAC-seq
  data with multiple samples per dataset.
when_to_use_negative:
- For pure locus-gene discovery rather than method benchmarking — this skill requires
  eQTL ground-truth data and is an evaluation, not a discovery, workflow
- When the input lacks paired single-cell multimodal data from a dynamic process (e.g.
  differentiation), since pseudotime-oriented DAGs cannot be constructed and Granger
  causal scores are undefined
- When only total-variant-to-nearest-gene annotation is needed — distal locus-gene
  link inference and its benchmarking are unnecessary overhead
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: GrID-Net
  role: Produces the Granger causal pair-level scores being benchmarked (GNN with
    L=10 layers, Adam, lr 0.001, batch size 1024)
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint RNA+ATAC cell representations used to build the shared DAG so
    all methods are scored on the same cell ordering
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Implementation framework for the GrID-Net GNN models used to generate scores
    under evaluation
  repo: https://github.com/alexw16/gridnet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:ccb71365ef08ab63a9a2a0adef95cb4529f3a094cc34adc5dd52d7cd4acf6f0e
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
  - PyTorch
  - Schema
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# eqtl-groundtruth-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol for validating inferred noncoding peak-gene links by scoring candidate pairs against significant eQTLs from the eQTL Catalogue and comparing methods via pair-level AUROC across multimodal single-cell datasets.

## When to use

Use when a peak-gene (locus-gene) inference method must be quantitatively benchmarked against eQTL-supported ground-truth pairs on paired scRNA-seq/scATAC-seq data with multiple samples per dataset.

## When NOT to use

- For pure locus-gene discovery rather than method benchmarking — this skill requires eQTL ground-truth data and is an evaluation, not a discovery, workflow
- When the input lacks paired single-cell multimodal data from a dynamic process (e.g. differentiation), since pseudotime-oriented DAGs cannot be constructed and Granger causal scores are undefined
- When only total-variant-to-nearest-gene annotation is needed — distal locus-gene link inference and its benchmarking are unnecessary overhead

## Inputs

- Preprocessed paired RNA-seq and ATAC-seq AnnData objects per sample (normalized, log(1+x)-transformed, genes filtered at 1% cells, peaks at 0.1% cells)
- Candidate peak-gene pairs DataFrame from gridnet.identify_all_peak_gene_link_candidates (distance_thresh=1e6; 250 kb for 10x Multiome)
- DAG adjacency matrix of cells (kNN k=15, pseudotime_algo='dpt')
- Method-specific pair-level scores (GrID-Net Granger statistics, Pearson/pseudocell correlation, Linear Granger, GVAR, ABC) on identical candidate pairs
- Significant eQTLs (P < 10^-10) from the eQTL Catalogue defining ground-truth peak-gene pairs

## Outputs

- Per-sample, per-method AUROC values against eQTL-supported peak-gene pairs
- Per-dataset table of AUROC mean ± SD across 5 samples for each method
- Welch one-tailed t-test P values for GrID-Net vs next best method per dataset
- Consolidated benchmark file eqtl_benchmark_auroc.csv

## How to apply

Run every method (GrID-Net plus baselines: Pearson correlation, pseudocell correlation, Linear Granger, GVAR, ABC) on the identical candidate peak-gene pairs and identical DAG of cells per sample. For GrID-Net, score pairs using the GNN with L=10 layers, Adam optimizer, learning rate 0.001, and mini-batch size 1024 candidate peak-gene pairs; pairs are candidates selected with distance_thresh=1e6 (250 kb for the 10x Multiome dataset). For each method and sample, compute AUROC using ground-truth peak-gene pairs that overlap significant eQTLs (P < 10^-10) from the eQTL Catalogue; the eQTL overlap serves as a surrogate for perturbation-based validation of causal links. Aggregate mean ± SD AUROC across the 5 samples per dataset (10x Multiome, SHARE-seq, SNARE-seq, sci-CAR) and compare GrID-Net to the next best method with a Welch one-tailed t-test. Write the consolidated per-method AUROC table and t-test results to eqtl_benchmark_auroc.csv.

## Related tools

- **GrID-Net** (Produces the Granger causal pair-level scores being benchmarked (GNN with L=10 layers, Adam, lr 0.001, batch size 1024)) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint RNA+ATAC cell representations used to build the shared DAG so all methods are scored on the same cell ordering) — https://github.com/rs239/schema
- **PyTorch** (Implementation framework for the GrID-Net GNN models used to generate scores under evaluation) — https://github.com/alexw16/gridnet

## Examples

```
results_df = gridnet.run_gridnet(X, Y, X_feature_names, Y_feature_names, candidate_XY_pairs, dag_adjacency_matrix)
```

## Evaluation signals

- Welch one-tailed t-test comparing GrID-Net AUROC to the next best method should reach P < 0.01 in each dataset
- Expected 7-22% higher AUROC than the next best method in 3 of 4 datasets; GVAR on sci-CAR is the only exception where GrID-Net is not significantly more accurate
- All methods must be scored on the identical candidate peak-gene pairs and DAG per sample; any divergence invalidates the comparison
- Ground-truth labels restricted to peak-gene pairs overlapping significant eQTLs (P < 10^-10) from the eQTL Catalogue
- Output CSV eqtl_benchmark_auroc.csv must contain per-method AUROC mean ± SD for all four datasets with 5 samples each

## Limitations

- eQTL cross-referencing only mimics perturbational validation; inferred locus-gene associations remain causal hypotheses that need to be validated by perturbations
- Relies on multimodal single-cell data from matching tissue undergoing a dynamic process to provide pseudotemporal context
- Computational burden of large single-cell datasets may exceed GPU memory capacity

## Evidence

- [discussion] All GrID-Net models consisted of L=10 GNN layers and were trained using the Adam optimizer with a learning rate of 0.001 and a mini-batch size of 1024 candidate peak-gene pairs.: [evidence span withheld — anchor status: not_found]
