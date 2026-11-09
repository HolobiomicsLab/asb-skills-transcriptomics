---
name: single-cell-multiome-data-partitioning
description: Use when you need held-out likelihood evaluation of batch-wise single-cell
  models (e.g., MIRA RP models on the 13-batch NEURIPS 10x Multiome bone marrow data)
  and must split cells per batch into train/test sets.
when_to_use_negative:
- The analysis goal is batch correction or latent-space integration rather than held-out
  likelihood model comparison — use CODAL/MIRA joint modeling directly instead
- The dataset has a single batch, so per-batch stratified train/test partitioning
  has no batch structure to preserve
- Input is already a model output (topic compositions or embeddings); partitioning
  applies to cell-level count data with batch labels
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: Preprocessing of the multimodal bone marrow dataset (filtering, normalization,
    HVG selection) prior to partitioning; AnnData slicing for train/test splits
- name: MIRA
  role: Topic/regulatory potential models trained on training cells and scored on
    held-out test cells from each batch
  repo: github.com/cistrome/MIRA
- name: CODAL
  role: Provides fixed technical effect vectors used to augment pooled training models,
    evaluated on the same per-batch test partitions
  repo: https://mira-multiome.readthedocs.io
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:b93a265f30b41a9b2c61e59bc8c4a490354b0573a5f8f44e5796cbe54ceb00cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CODAL
  - MIRA
  - scanpy
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# single-cell-multiome-data-partitioning

## Summary

Partitions single-cell multiome cells from each sequencing batch into training and held-out test sets (4:1 ratio) so that generative models such as MIRA regulatory potential models can be compared by held-out log-likelihood across batches.

## When to use

Use when you need held-out likelihood evaluation of batch-wise single-cell models (e.g., MIRA RP models on the 13-batch NEURIPS 10x Multiome bone marrow data) and must split cells per batch into train/test sets.

## When NOT to use

- The analysis goal is batch correction or latent-space integration rather than held-out likelihood model comparison — use CODAL/MIRA joint modeling directly instead
- The dataset has a single batch, so per-batch stratified train/test partitioning has no batch structure to preserve
- Input is already a model output (topic compositions or embeddings); partitioning applies to cell-level count data with batch labels

## Inputs

- Preprocessed single-cell multiome AnnData (scRNA-seq + scATAC-seq counts) with batch labels, e.g., the 2021 NEURIPS Multimodal Single-Cell Data Integration challenge bone marrow dataset
- Batch identity annotations for each cell (13 sequencing batches in the NEURIPS dataset)
- Optionally, a gene mask restricting to 2841 highly variable genes with UCSC TSS annotations

## Outputs

- Per-batch training and test cell index sets in a 4:1 ratio
- Held-out log-likelihood scores per model regime per batch, assembled into a batch-by-regime matrix for heatmap rendering

## How to apply

Start from the preprocessed multiome AnnData (e.g., NEURIPS bone marrow, restricted to the 2841 genes that are both highly variable and have UCSC TSS annotations for RP modeling). For each of the sequencing batches (13 in the NEURIPS 10x Multiome bone marrow dataset), randomly assign cells to a training set and a test set in a 4:1 ratio, keeping the split within batch so every batch contributes held-out cells. Train candidate models (per-batch, pooled-uncorrected, pooled-augmented with CODAL technical effect vectors) only on the training cells, then evaluate log-likelihood on each batch's test cells. Assemble the scores into a batch-by-regime matrix for heatmap comparison (reproducing the Fig. 4b,c style comparison). Rationale: a per-batch stratified split guarantees the held-out likelihood is comparable across all batches regardless of batch size or cell-type composition.

## Related tools

- **scanpy** (Preprocessing of the multimodal bone marrow dataset (filtering, normalization, HVG selection) prior to partitioning; AnnData slicing for train/test splits)
- **MIRA** (Topic/regulatory potential models trained on training cells and scored on held-out test cells from each batch) — github.com/cistrome/MIRA
- **CODAL** (Provides fixed technical effect vectors used to augment pooled training models, evaluated on the same per-batch test partitions) — https://mira-multiome.readthedocs.io

## Examples

```
for b in adata.obs['batch'].unique(): train_idx, test_idx = train_test_split(adata.obs_names[adata.obs['batch']==b], test_size=0.2, random_state=0)
```

## Evaluation signals

- Each batch contributes both training and test cells; no batch is left without held-out cells
- Train:test ratio within every batch is 4:1
- Held-out log-likelihood can be computed for every trained model on every batch's test cells, forming a complete batch-by-regime matrix
- The 'all batches + CODAL technical effect correction' regime attains the highest total held-out likelihood across batches, as reported in the paper

## Limitations

- Per-batch splits with batch-confounded cell states can make held-out likelihood sensitive to under-correction of technical effects; the authors caution such states should be validated with orthogonal biological evidence
- The partitioning strategy is tied to count-based generative model comparison (Negative Binomial likelihoods); it does not itself remove batch effects
- Likelihood comparability depends on using CODAL's unconfounded compositional accessibility rates as accessibility input for the augmented regime

## Evidence

- [methods] Partition each of the 13 sequencing batches into training and test sets in a 4:1 ratio.: [evidence span withheld — anchor status: not_found]
- [intro] batched dataset NEURIPS challenge: "The batched dataset, created for the 2021 NEURIPS Multimodal Single-Cell Data Integration challenge33, was generated by distributing bone marrow samples"
- [methods] 2841 genes HVG TSS: "2841 genes which were both highly variable and had UCSC TSS annotations78"
- [methods] cells with fewer than 400 counts and genes with fewer than 30 counts filtered out: "cells with fewer than 400 counts and genes with fewer than 30 counts filtered out"
