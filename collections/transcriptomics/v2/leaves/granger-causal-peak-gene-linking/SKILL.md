---
name: granger-causal-peak-gene-linking
description: Use when paired scRNA-seq/scATAC-seq data from a dynamic process (e.g.
  10x Multiome corticogenesis) must be prioritized for causal peak-gene or variant-locus-to-gene
  links beyond correlation-based proximity.
when_to_use_negative:
- The input data are not paired multimodal single-cell measurements (RNA + ATAC from
  the same cells); GrID-Net's DAG-of-cells construction and pseudotime orientation
  require jointly assayed modalities
- The tissue is static or lacks a dynamic process (differentiation/disease trajectory),
  since the method relies on pseudotemporal context from a dynamic biological process
- Data exceed GPU memory capacity, or a quick exploratory correlation-based peak-gene
  ranking suffices and causal lag inference is unnecessary
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0203
tools:
- name: GrID-Net
  role: 'Core framework: trains a graph neural network with lagged message passing
    on the cell-state DAG and scores peak-gene pairs with a one-tailed F-test on forecasting
    improvement'
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint multimodal cell embeddings unifying gene expression and chromatin
    accessibility, used to build the kNN graph and diffusion pseudotime DAG
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Backend in which GrID-Net models are implemented and trained
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:793a5ebf55d0ac40c44b5ea1843fe130699b7d2e9f5785a9c7a3e6c1064d0749
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
  - CIS-BP
  - FIMO
  - GrID-Net
  - HOCOMOCO v11
  - JASPAR CORE (Non-Redundant) v2
  - PyTorch
  - Schema
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# granger-causal-peak-gene-linking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies GrID-Net, a graph neural network that generalizes Granger causal inference to DAGs of cell states, to identify causal noncoding peak-gene links from paired single-cell RNA-seq and ATAC-seq multimodal data, exploiting the 'cell-state parallax' time lag between chromatin accessibility and expression that correlation-based methods miss.

## When to use

Use when paired scRNA-seq/scATAC-seq data from a dynamic process (e.g. 10x Multiome corticogenesis) must be prioritized for causal peak-gene or variant-locus-to-gene links beyond correlation-based proximity.

## When NOT to use

- The input data are not paired multimodal single-cell measurements (RNA + ATAC from the same cells); GrID-Net's DAG-of-cells construction and pseudotime orientation require jointly assayed modalities
- The tissue is static or lacks a dynamic process (differentiation/disease trajectory), since the method relies on pseudotemporal context from a dynamic biological process
- Data exceed GPU memory capacity, or a quick exploratory correlation-based peak-gene ranking suffices and causal lag inference is unnecessary

## Inputs

- Unprocessed AnnData object for RNA-seq with gene TSS annotations in adata.var (columns chr_no, txstart)
- Unprocessed AnnData object for ATAC-seq with peak start/end annotations in adata.var (columns chr_no, start, end)
- Root cell marker gene (e.g. 'TOP2A') or root cell index for pseudotime inference
- DAG adjacency matrix (N x N) when using gridnet.run_gridnet directly

## Outputs

- DataFrame of candidate peak-gene pairs with Granger causal test statistics (results_df from gridnet.run_gridnet)
- Distance-dependent locus-gene link assignments (up to 3 genes per variant-overlapping peak, top 50% of scores)

## How to apply

First normalize counts with log(1+x) via gridnet.preprocess_multimodal, then assemble candidate peak-gene pairs by genomic distance between peak centers and gene TSSs (distance_thresh=1e6, or 250 kb for the 10x Multiome corticogenesis dataset) while filtering genes expressed in <1% of cells and peaks accessible in <0.1% of cells. Learn joint multimodal cell representations with Schema (gridnet.schema_representations), build a k-nearest neighbor graph (k=15) with Euclidean distances, orient edges by diffusion pseudotime (dpt) to form a cell-state DAG via gridnet.construct_dag, and run gridnet.run_gridnet, which trains 10-layer GNN models (Adam, lr 0.001, batch size 1024 candidate pairs, up to 20 epochs) with lagged message passing and scores each candidate pair with a one-tailed F-test comparing reduced vs. full forecasting models. For variant-to-gene mapping, link each SCZ variant overlapping an ATAC peak (fine-mapped PP>=0.05) to up to 3 of the peak's highest-scoring genes using distance-dependent GrID-Net scores, retaining only pairs in the top 50% of all peak-gene scores.

## Related tools

- **GrID-Net** (Core framework: trains a graph neural network with lagged message passing on the cell-state DAG and scores peak-gene pairs with a one-tailed F-test on forecasting improvement) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint multimodal cell embeddings unifying gene expression and chromatin accessibility, used to build the kNN graph and diffusion pseudotime DAG) — https://github.com/rs239/schema
- **PyTorch** (Backend in which GrID-Net models are implemented and trained)

## Examples

```
results_df = gridnet.gridnet_multimodal(rna_adata, atac_adata, root_cell_marker_gene='TOP2A')
```

## Evaluation signals

- Variant coverage gain: applying GrID-Net to schizophrenia variants should increase variant coverage by 36% relative to nearest-gene or correlation baselines
- eQTL cross-validation: trained logistic regression on GrID-Net scores + log-distance should predict peak-gene pairs overlapping significant skin eQTLs (P < 10^-3) on the SHARE-seq mouse skin dataset
- Pseudotime lag test: GrID-Net-prioritized top/bottom 1% peak-gene pairs (n=14,802 per group) should show significantly longer accessibility-to-expression lags than Pearson or pseudocell correlation pairs (Welch one-tailed P < 10^-26 on the 10x Multiome corticogenesis dataset)
- Results DataFrame from gridnet.run_gridnet must annotate every candidate pair passed in candidate_XY_pairs with Granger causality test statistics
- DAG sanity check: the adjacency matrix from gridnet.construct_dag must be acyclic with edges oriented along increasing diffusion pseudotime (dpt), k=15 neighbors

## Limitations

- Inferred locus-gene associations are causal hypotheses that require perturbation-based validation; eQTL cross-referencing only mimics such validation
- Relies on multiomic single-cell data from a matching tissue undergoing a dynamic process (e.g. corticogenesis), which may explain only a subset of disease phenotypes such as SCZ
- Computational burden of large single-cell datasets may exceed GPU memory capacity (models trained on a single NVIDIA Tesla V100)
- Inferred causal interactions are likely indirect and mediated by additional biophysical events such as TF binding; no Hi-C/3D genome or CTCF topology information is incorporated
- GrID-Net's full mathematical formulation is deferred to the Supplementary Information

## Evidence

- [intro] F-test scoring of Granger causal links: [evidence span withheld — anchor status: not_found]
