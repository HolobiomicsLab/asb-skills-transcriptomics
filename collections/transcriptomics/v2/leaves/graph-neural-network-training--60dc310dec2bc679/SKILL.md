---
name: graph-neural-network-training
description: Use when you have paired scATAC-seq peak accessibility (N x C) and scRNA-seq
  expression (N x T) matrices, a candidate peak-gene pair list, and a cell-state DAG,
  and need to rank Granger causal links.
when_to_use_negative:
- Data is static or lacks a dynamic biological process providing pseudotemporal context
  — GrID-Net relies on multiomic data from matching tissue undergoing a process such
  as differentiation to orient the DAG.
- Single-modality data or datasets where scATAC-seq and scRNA-seq are not assayed
  for the same cells (paired multiomic data is required).
- Datasets so large that the GNN training exceeds GPU memory capacity, per the paper's
  stated practical limitation.
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: GrID-Net
  role: Framework whose run_gridnet function trains the reduced and full lagged message-passing
    GNN forecasting models and applies the one-tailed F-test per candidate peak-gene
    pair
  repo: https://github.com/alexw16/gridnet
- name: PyTorch
  role: Backend in which the GrID-Net GNN models are implemented and trained
- name: NVIDIA Tesla V100 GPU
  role: Hardware used to train the GrID-Net models
- name: Schema
  role: Produces the joint RNA+ATAC cell representations used to construct the kNN
    graph and pseudotime for the DAG that the GNN message passing runs on
  repo: https://github.com/rs239/schema
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:cd6bd689287f646ea813ec1af5228fd88ba1a35c18f624a678aa77910fceacd7
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
  - NVIDIA Tesla V100 GPU
  - PyTorch
  - Schema
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# graph-neural-network-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train lagged message-passing GNNs (reduced and full forecasting models) on a DAG of cell states to score Granger causal peak-gene links from paired scATAC-seq/scRNA-seq data, as implemented in GrID-Net (gridnet.run_gridnet) in PyTorch on a single NVIDIA Tesla V100 GPU.

## When to use

Use when you have paired scATAC-seq peak accessibility (N x C) and scRNA-seq expression (N x T) matrices, a candidate peak-gene pair list, and a cell-state DAG, and need to rank Granger causal links.

## When NOT to use

- Data is static or lacks a dynamic biological process providing pseudotemporal context — GrID-Net relies on multiomic data from matching tissue undergoing a process such as differentiation to orient the DAG.
- Single-modality data or datasets where scATAC-seq and scRNA-seq are not assayed for the same cells (paired multiomic data is required).
- Datasets so large that the GNN training exceeds GPU memory capacity, per the paper's stated practical limitation.

## Inputs

- scATAC-seq peak accessibility matrix X (N x C) with peak feature names (AnnData with chr_no, start, end annotations)
- scRNA-seq expression matrix Y (N x T) with gene feature names (AnnData with chr_no, txstart TSS annotations)
- candidate peak-gene pairs table from identify_all_peak_gene_link_candidates (distance_thresh=1e6) converted to (atac_id, gene) tuples
- DAG adjacency matrix (N x N) of cells ordered consistently with rows of X and Y

## Outputs

- results DataFrame annotating candidate peak-gene pairs with Granger causality F-test statistics and p-values (artifact_peakgene_links.csv)
- ranked list of peak-gene pairs by Granger causal strength

## How to apply

First construct the DAG of cells from Schema multimodal representations via a k=15 kNN graph with edges oriented by diffusion pseudotime, ordered consistently with the rows of X and Y. Load the peak accessibility matrix X and gene expression matrix Y with feature names, and convert the candidates table (distance_thresh=1e6; genes expressed in >=1% of cells, peaks accessible in >=0.1% of cells) into candidate_XY_pairs as (atac_id, gene) tuples. Train the reduced and full forecasting models with gridnet.run_gridnet using L=10 GNN layers, the Adam optimizer with learning rate 0.001, mini-batch size 1024 candidate peak-gene pairs, and up to 20 epochs in PyTorch. The reduced model forecasts each gene from its own past; the full model adds the candidate cause, and each pair is scored by a one-tailed F-test comparing the forecasting improvement of the full over the reduced model. Rank pairs by Granger causal strength and write peak, gene, F-test statistic, and p-value to artifact_peakgene_links.csv.

## Related tools

- **GrID-Net** (Framework whose run_gridnet function trains the reduced and full lagged message-passing GNN forecasting models and applies the one-tailed F-test per candidate peak-gene pair) — https://github.com/alexw16/gridnet
- **PyTorch** (Backend in which the GrID-Net GNN models are implemented and trained)
- **NVIDIA Tesla V100 GPU** (Hardware used to train the GrID-Net models)
- **Schema** (Produces the joint RNA+ATAC cell representations used to construct the kNN graph and pseudotime for the DAG that the GNN message passing runs on) — https://github.com/rs239/schema

## Examples

```
results_df = gridnet.run_gridnet(X, Y, X_feature_names, Y_feature_names, candidate_XY_pairs, dag_adjacency_matrix)
```

## Evaluation signals

- Output table schema check: results DataFrame contains peak, gene, F-test statistic, and p-value columns for every candidate_XY_pair
- Recompute F-test statistics on a small held-out candidate set and confirm they match the reported values
- Verify DAG adjacency matrix is square (N x N), acyclic, and ordered consistently with the rows of X and Y
- Confirm training hyperparameters match the paper: L=10 GNN layers, Adam lr=0.001, mini-batch size 1024 pairs, up to 20 epochs
- Sanity-check that candidate pairs obey the filters: peaks within 1 Mb of gene TSS, genes expressed in >=1% of cells, peaks accessible in >=0.1% of cells

## Limitations

- GrID-Net's mathematical formulation is deferred to the Supplementary Information, not the main text.
- Inferred associations are causal hypotheses from observational data that require perturbation-based validation; eQTL cross-referencing only mimics such validation.
- Large single-cell datasets may exceed GPU memory capacity during GNN training.
- Requires multiomic single-cell data from matching tissue undergoing a dynamic process (e.g., differentiation) to supply pseudotemporal context; tissue matching may be imperfect.
- Inferred causal interactions are likely indirect and mediated by additional biophysical events such as transcription factor binding; no Hi-C/CTCF topology information is incorporated.

## Evidence

- [discussion] GNN layers and Adam hyperparameters: [evidence span withheld — anchor status: not_found]
- [intro] one-tailed F-test scoring: [evidence span withheld — anchor status: not_found]
- [discussion] DAG construction from kNN graph: [evidence span withheld — anchor status: not_found]
