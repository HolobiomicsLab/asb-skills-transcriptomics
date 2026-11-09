---
name: dag-based-granger-causal-inference
description: Use when you have paired single-cell RNA-seq and ATAC-seq (multimodal)
  data from a dynamic biological process and need causal (not merely correlational)
  peak-gene link inference across cell states.
when_to_use_negative:
- The data lacks a dynamic biological process (e.g. differentiation or disease trajectory)
  providing pseudotemporal context — GrID-Net relies on multiomic data from matching
  tissue undergoing a dynamic process
- The input is static bulk or single-modality data without paired scATAC-seq and scRNA-seq
  measurements of the same cells
- A purely associational peak-gene analysis (e.g. Pearson or pseudocell correlation)
  suffices and causal-lag interpretation is not needed
edam_operation: http://edamontology.org/operation_3890
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3794
- http://edamontology.org/topic_0203
tools:
- name: GrID-Net
  role: 'Core framework: graph neural network performing Granger causal inference
    on the cell-state DAG via lagged message passing; gridnet.run_gridnet scores candidate
    peak-gene pairs'
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint RNA+ATAC cell representations (gridnet.schema_representations)
    that are used to build the kNN graph and infer pseudotime for DAG construction
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Implementation backend for the GrID-Net GNN models, trained on a single NVIDIA
    Tesla V100 GPU
- name: scanpy
  role: Provides the diffusion pseudotime implementation used to orient kNN graph
    edges into a DAG
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:83e01866a83ccfbc1da5af42c763c0afcbecb04b14de6c7313bdf10a1c062243
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
  edam_operation: http://edamontology.org/operation_3890
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3794
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# dag-based-granger-causal-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for performing Granger causal inference over directed acyclic graphs (DAGs) — generalizing Granger causality from total orderings to partial orderings — to infer causal noncoding locus-gene relationships from single-cell multimodal RNA+ATAC data using GrID-Net.

## When to use

Use when you have paired single-cell RNA-seq and ATAC-seq (multimodal) data from a dynamic biological process and need causal (not merely correlational) peak-gene link inference across cell states.

## When NOT to use

- The data lacks a dynamic biological process (e.g. differentiation or disease trajectory) providing pseudotemporal context — GrID-Net relies on multiomic data from matching tissue undergoing a dynamic process
- The input is static bulk or single-modality data without paired scATAC-seq and scRNA-seq measurements of the same cells
- A purely associational peak-gene analysis (e.g. Pearson or pseudocell correlation) suffices and causal-lag interpretation is not needed

## Inputs

- Unprocessed AnnData object for the RNA-seq modality with gene TSS annotations in adata.var (chr_no, txstart)
- Unprocessed AnnData object for the ATAC-seq modality with peak genomic coordinates in adata.var (chr_no, start, end)
- A root cell marker gene (e.g. 'TOP2A') or root cell index for pseudotime inference
- Candidate peak-gene pairs (DataFrame with atac_id and gene columns)
- DAG adjacency matrix over cells (N x N), rows/columns ordered to match X and Y

## Outputs

- DataFrame of candidate peak-gene pairs with Granger causality test statistics (one-tailed F-test) from gridnet.run_gridnet
- Ranked table of peak-gene links ordered by Granger causal strength, suitable for AUROC evaluation against eQTL-supported ground truth

## How to apply

First, preprocess paired RNA/ATAC AnnData objects with gridnet.preprocess_multimodal (count normalization and log(1+x) transformation), filtering genes expressed in fewer than 1% of cells and peaks accessible in fewer than 0.1% of cells. Assemble candidate peak-gene pairs with gridnet.identify_all_peak_gene_link_candidates using distance_thresh=1e6 (restricted to 250 kb for the 10x Multiome dataset). Learn joint RNA+ATAC cell representations with gridnet.schema_representations (Schema), then build a kNN graph (k=15) and orient its edges by diffusion pseudotime ('dpt') to form the DAG of cells via gridnet.construct_dag. Run gridnet.run_gridnet with a GNN of L=10 layers, Adam optimizer, learning rate 0.001, and mini-batch size 1024 candidate peak-gene pairs; GrID-Net uses two forecasting models (reduced and full) and a one-tailed F-test on forecasting improvement to quantify the strength of each Granger causal link. The rationale is that the DAG of cell states captures the inherent causal time lag ('cell-state parallax') between chromatin accessibility and gene expression that correlational methods overlook.

## Related tools

- **GrID-Net** (Core framework: graph neural network performing Granger causal inference on the cell-state DAG via lagged message passing; gridnet.run_gridnet scores candidate peak-gene pairs) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint RNA+ATAC cell representations (gridnet.schema_representations) that are used to build the kNN graph and infer pseudotime for DAG construction) — https://github.com/rs239/schema
- **PyTorch** (Implementation backend for the GrID-Net GNN models, trained on a single NVIDIA Tesla V100 GPU)
- **scanpy** (Provides the diffusion pseudotime implementation used to orient kNN graph edges into a DAG)

## Examples

```
X_joint = gridnet.schema_representations(rna_adata,atac_adata); dag_adjacency_matrix = gridnet.construct_dag(X_joint,iroot,n_neighbors=15,pseudotime_algo='dpt'); results_df = gridnet.run_gridnet(X,Y,X_feature_names,Y_feature_names,candidate_XY_pairs,dag_adjacency_matrix)
```

## Evaluation signals

- AUROC of peak-gene link scores against ground-truth pairs overlapping significant eQTLs (P < 10^-10) from the eQTL Catalogue should exceed the next best baseline (Pearson, pseudocell correlation, Linear Granger, GVAR, ABC) by 7-22% in 3 of 4 datasets
- Welch one-tailed t-test comparing GrID-Net vs next best method per dataset should yield P < 0.01 (only exception: GVAR on sci-CAR)
- DAG adjacency matrix must be acyclic with rows/columns ordered consistently with X and Y; kNN graph uses n_neighbors=15 and pseudotime_algo='dpt'
- Candidate peak-gene pairs must respect distance_thresh (default 1e6; 250 kb for 10x Multiome) and post-filter expression/accessibility thresholds (1% genes, 0.1% peaks)
- GNN training config check: L=10 layers, Adam optimizer, learning rate 0.001, mini-batch size 1024 candidate pairs, up to 20 epochs

## Limitations

- GrID-Net infers causal hypotheses from observational data that require perturbation-based validation; eQTL cross-referencing only mimics such validation
- Requires multiomic single-cell data from matching tissue undergoing a dynamic process to provide pseudotemporal context; tissue matching is imperfect (e.g. corticogenesis data may explain only a subset of SCZ phenotypes)
- Computational burden of large single-cell datasets may exceed GPU memory capacity
- Inferred causal interactions are likely indirect and mediated by additional biophysical events such as transcription factor binding; no Hi-C/3D genome structure or CTCF topology is incorporated
- GrID-Net's mathematical formulation is deferred to Supplementary Information, not provided in the main text

## Evidence

- [intro] Our key mathematical advance is a generalization of Granger causality to partial orderings.: "Our key mathematical advance is a generalization of Granger causality to partial orderings."
- [discussion] GNN training hyperparameters: [evidence span withheld — anchor status: not_found]
- [intro] F-test link strength evaluation: [evidence span withheld — anchor status: not_found]
