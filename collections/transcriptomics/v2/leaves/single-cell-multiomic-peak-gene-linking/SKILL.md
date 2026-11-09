---
name: single-cell-multiomic-peak-gene-linking
description: Use when you have paired scATAC-seq/scRNA-seq data from a dynamic single-cell
  process and need to rank candidate chromatin peak-gene regulatory links by causal
  strength rather than correlation.
when_to_use_negative:
- 'Data lack pseudotemporal context: the method relies on multiomic single-cell data
  from matching tissue undergoing a dynamic process such as differentiation or disease;
  static or non-dynamic snapshots are out of scope.'
- Only one modality is available (RNA-seq or ATAC-seq alone) — the wrapper requires
  paired scATAC-seq and scRNA-seq for the same cells.
- The goal is total-order Granger causality on regular time series rather than causal
  inference on a partially ordered DAG of cell states.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GrID-Net
  role: 'Core framework: runs lagged message-passing GNN forecasting models and the
    one-tailed F-test to quantify Granger causal peak-gene link strength on the cell-state
    DAG'
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint multimodal cell representations from RNA and ATAC modalities
    used to infer pseudotime and construct the DAG of cells
  repo: https://github.com/rs239/schema
- name: PyTorch
  role: Implementation backend for the GrID-Net GNN models
- name: NVIDIA Tesla V100 GPU
  role: Hardware used to train the GrID-Net models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:0bf8471876f81b1c4975fbf74e2c8956dcf871d08d7e864af061bdbdb457e686
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-multiomic-peak-gene-linking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers causal noncoding peak-gene links from paired single-cell ATAC-seq and RNA-seq data by generalizing Granger causality to DAGs: GrID-Net trains lagged message-passing GNN forecasting models (reduced vs. full) and scores each candidate peak-gene pair with a one-tailed F-test on forecasting improvement.

## When to use

Use when you have paired scATAC-seq/scRNA-seq data from a dynamic single-cell process and need to rank candidate chromatin peak-gene regulatory links by causal strength rather than correlation.

## When NOT to use

- Data lack pseudotemporal context: the method relies on multiomic single-cell data from matching tissue undergoing a dynamic process such as differentiation or disease; static or non-dynamic snapshots are out of scope.
- Only one modality is available (RNA-seq or ATAC-seq alone) — the wrapper requires paired scATAC-seq and scRNA-seq for the same cells.
- The goal is total-order Granger causality on regular time series rather than causal inference on a partially ordered DAG of cell states.

## Inputs

- Paired unprocessed AnnData objects for RNA-seq (with gene TSS annotations: chr_no, txstart in adata.var) and ATAC-seq (with peak annotations: chr_no, start, end in adata.var)
- Peak accessibility matrix X (N x C numpy.ndarray) with X_feature_names
- Gene expression matrix Y (N x T numpy.ndarray) with Y_feature_names
- Candidate peak-gene pairs table from gridnet.identify_all_peak_gene_link_candidates with distance_thresh=1e6, converted to (atac_id, gene) tuples
- DAG adjacency matrix (N x N) of cell states ordered consistently with rows of X and Y, from a k=15 kNN graph oriented by diffusion pseudotime
- Root cell specification: a marker gene for the root cell type (e.g. TOP2A) or a root cell index

## Outputs

- results DataFrame annotating each candidate peak-gene pair with Granger causality one-tailed F-test statistics (peak, gene, F-test statistic, p-value)
- Ranked artifact table of peak-gene links (e.g. artifact_peakgene_links.csv)

## How to apply

First pre-process the paired AnnData objects with count normalization and log(1+x) transformation (gridnet.preprocess_multimodal), then assemble candidate peak-gene pairs whose peak centers fall within 1 Mb of the gene TSS (distance_thresh=1e6), keeping genes expressed in >=1% of cells and peaks accessible in >=0.1% of cells. Learn joint multimodal cell representations with Schema, build a k=15 kNN graph, orient edges by diffusion pseudotime from a root cell (marker gene like TOP2A or explicit root index) to form a DAG of cell states. Load the peak accessibility matrix X (N x C), gene expression matrix Y (N x T), feature names, and candidate_XY_pairs as (atac_id, gene) tuples, ordering the DAG adjacency matrix consistently with the rows of X and Y. Train the reduced and full lagged message-passing GNN forecasting models via gridnet.run_gridnet with L=10 layers, Adam optimizer (lr 0.001), mini-batch size 1024 candidate pairs, up to 20 epochs in PyTorch on a Tesla V100 GPU. Rank each pair by Granger causal strength computed from a one-tailed F-test comparing full-model forecasting accuracy against the reduced model that uses only the effect's own past, and write the results DataFrame to a CSV artifact.

## Related tools

- **GrID-Net** (Core framework: runs lagged message-passing GNN forecasting models and the one-tailed F-test to quantify Granger causal peak-gene link strength on the cell-state DAG) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint multimodal cell representations from RNA and ATAC modalities used to infer pseudotime and construct the DAG of cells) — https://github.com/rs239/schema
- **PyTorch** (Implementation backend for the GrID-Net GNN models)
- **NVIDIA Tesla V100 GPU** (Hardware used to train the GrID-Net models)

## Examples

```
results_df = gridnet.gridnet_multimodal(rna_adata,atac_adata,root_cell_marker_gene='TOP2A',distance_thresh=1e6)
```

## Evaluation signals

- Verify the output table schema contains peak, gene, F-test statistic, and p-value columns for each candidate pair.
- Confirm recomputed F-test statistics on a small held-out candidate set match the reported values.
- Check the DAG adjacency matrix is square (N x N), acyclic, and ordered consistently with the rows of X and Y.
- Confirm candidate pairs satisfy the filters: peaks within 1 Mb of gene TSS, genes expressed in >=1% of cells, peaks accessible in >=0.1% of cells.
- Sanity-check the underlying GNN training configuration: L=10 layers, Adam lr 0.001, mini-batch size 1024, up to 20 epochs.

## Limitations

- Inferred locus-gene associations are causal hypotheses that require perturbation-based validation; eQTL cross-referencing only mimics such validation.
- Large single-cell datasets may impose computational burden exceeding GPU memory capacity.
- Relies on multiomic single-cell data from matching tissue undergoing a dynamic process to provide pseudotemporal context; tissue matching may be imperfect (corticogenesis data may explain only a subset of SCZ phenotypes).
- Inferred causal interactions are likely indirect and mediated by additional biophysical events such as transcription factor binding.
- No Hi-C/3D genome structure or CTCF topology information is incorporated; adding such modalities could refine predictions.

## Evidence

- [full_text] GrID-Net scores candidate causal links by evaluating each pair with a one-tailed F-test that quantifies how much the full forecasting model: [evidence span withheld — anchor status: not_found]
- [discussion] All GrID-Net models consisted of L=10 GNN layers and were trained using the Adam optimizer with a learning rate of 0.001: [evidence span withheld — anchor status: not_found]
