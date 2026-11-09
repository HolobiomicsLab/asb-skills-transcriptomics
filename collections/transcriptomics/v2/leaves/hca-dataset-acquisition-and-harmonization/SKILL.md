---
name: hca-dataset-acquisition-and-harmonization
description: Use when benchmarking batch-effect insensitivity of a representation
  method on a large multi-donor dataset, or reproducing the tGPT vs. ComBat/MNN/Harmony/Seurat/BBKNN/Scanorama/Pegasus/scVI/scArches/iMAP/DESC
  kBET comparison on HCA bone marrow.
when_to_use_negative:
- The dataset lacks batch/donor labels — kBET requires a batch variable to compute
  acceptance rates
- The analysis depends on actual expression levels or fold changes among genes — tGPT
  uses only expression rankings of top-expressing genes and neglects fold changes
- The question concerns low-expressing genes — tGPT uses only the top-expressing genes
  and may miss information represented within low-expressing genes
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: tGPT
  role: Pretrained GPT-2 transformer used to extract cell embeddings from per-cell
    gene rankings (checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64)
  repo: github.com/deeplearningplus/tGPT
- name: Scanpy
  role: Neighbor graph computation and downstream clustering/UMAP on extracted embeddings
    (sc.pp.neighbors, n_neighbors=20)
  repo: github.com/theislab/scanpy
- name: ComBat
  role: Batch-correction baseline providing corrected embeddings for kBET comparison
  repo: github.com/Jfortin1/ComBatHarmonization
- name: MNN
  role: Batch-correction baseline providing corrected embeddings for kBET comparison
  repo: github.com/MarioniLab/MNN2017
- name: Harmony
  role: Batch-correction baseline providing corrected embeddings for kBET comparison
  repo: github.com/immunogenomics/harmony
- name: Scanorama
  role: Batch-correction baseline providing corrected embeddings for kBET comparison
  repo: github.com/brianhie/scanorama
- name: iMAP
  role: Batch-correction baseline providing corrected embeddings for kBET comparison
  repo: github.com/Svvord/iMAP
- name: scArches
  role: Batch-correction baseline providing corrected embeddings for kBET comparison
  repo: github.com/theislab/scarches
- name: Pegasus
  role: Single-cell analysis baseline (v1.4.3) with recommended filter parameters
    (min_genes 500, max_genes 6000, percent_mito 10)
  repo: github.com/lilab-bcb/pegasus
- name: scVI
  role: Deep-learning batch-correction baseline providing corrected embeddings for
    kBET comparison
  repo: github.com/theislab/scvelo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:3af0f2a1d72bb9a9f0d31df63e9bd91ca1096318cd047bb4881442a9a7143949
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  tools:
  - BBKNN
  - ComBat
  - DESC
  - Harmony
  - MNN
  - Pegasus
  - Scanorama
  - Scanpy
  - Seurat
  - iMAP
  - scArches
  - scVI
  - tGPT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# hca-dataset-acquisition-and-harmonization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Acquire the Human Cell Atlas Census of Immune Cells bone-marrow dataset (n=282,588 cells from 64 healthy donors), harmonize it with 11 batch-correction methods, and benchmark tGPT's batch-insensitivity via kBET acceptance rates.

## When to use

Use when benchmarking batch-effect insensitivity of a representation method on a large multi-donor dataset, or reproducing the tGPT vs. ComBat/MNN/Harmony/Seurat/BBKNN/Scanorama/Pegasus/scVI/scArches/iMAP/DESC kBET comparison on HCA bone marrow.

## When NOT to use

- The dataset lacks batch/donor labels — kBET requires a batch variable to compute acceptance rates
- The analysis depends on actual expression levels or fold changes among genes — tGPT uses only expression rankings of top-expressing genes and neglects fold changes
- The question concerns low-expressing genes — tGPT uses only the top-expressing genes and may miss information represented within low-expressing genes

## Inputs

- HCA Census of Immune Cells bone-marrow count matrix (project cc95ff89-2e68-4a08-a234-480eca21ce79, 282,588 cells)
- Donor/batch labels for the 64 healthy donors (kBET batch variable)
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64'
- Per-cell gene rankings of top-expressing genes (gene rankings file, e.g. gene_rankings.txt.gz)

## Outputs

- tGPT cell embeddings (mean-pooled over non-padding positions, max_len=64 top-expressing genes)
- Batch-corrected embeddings from ComBat, MNN, Harmony, Seurat, BBKNN, Scanorama, Pegasus, scVI, scArches, iMAP, DESC
- kbet_acceptance_rates.csv: per-method kBET acceptance rates (tGPT ~0.87, Figure S1L)

## How to apply

Download the HCA bone-marrow dataset (project cc95ff89-2e68-4a08-a234-480eca21ce79) from data.humancellatlas.org. Per cell, compute gene expression rankings and feed the top 64 expressing genes (max_len=64) through the pretrained checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (GPT2Model with output_hidden_states=True), mean-pooling token states over non-padding positions (item[1:eos_idx].mean(dim=0)) to obtain cell embeddings. Run the eleven batch-correction baselines (ComBat, MNN, Harmony, Seurat, BBKNN, Scanorama, Pegasus, scVI, scArches, iMAP, DESC) on the same count matrix to get corrected embeddings. Compute kBET acceptance rates on tGPT embeddings and each corrected embedding using donor/batch labels as the batch variable, then assemble per-method rates into kbet_acceptance_rates.csv. The rationale is that tGPT's rank-based input makes it insensitive to batch effects, so its acceptance rate (~0.87, Figure S1L) should be comparable to methods that explicitly correct for batch.

## Related tools

- **tGPT** (Pretrained GPT-2 transformer used to extract cell embeddings from per-cell gene rankings (checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64)) — github.com/deeplearningplus/tGPT
- **Scanpy** (Neighbor graph computation and downstream clustering/UMAP on extracted embeddings (sc.pp.neighbors, n_neighbors=20)) — github.com/theislab/scanpy
- **ComBat** (Batch-correction baseline providing corrected embeddings for kBET comparison) — github.com/Jfortin1/ComBatHarmonization
- **MNN** (Batch-correction baseline providing corrected embeddings for kBET comparison) — github.com/MarioniLab/MNN2017
- **Harmony** (Batch-correction baseline providing corrected embeddings for kBET comparison) — github.com/immunogenomics/harmony
- **Scanorama** (Batch-correction baseline providing corrected embeddings for kBET comparison) — github.com/brianhie/scanorama
- **iMAP** (Batch-correction baseline providing corrected embeddings for kBET comparison) — github.com/Svvord/iMAP
- **scArches** (Batch-correction baseline providing corrected embeddings for kBET comparison) — github.com/theislab/scarches
- **Pegasus** (Single-cell analysis baseline (v1.4.3) with recommended filter parameters (min_genes 500, max_genes 6000, percent_mito 10)) — github.com/lilab-bcb/pegasus
- **scVI** (Deep-learning batch-correction baseline providing corrected embeddings for kBET comparison) — github.com/theislab/scvelo

## Examples

```
model = GPT2LMHeadModel.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64', output_hidden_states=True).transformer; batch = tokenizer(lines, max_length=64, truncation=True, padding=True, return_tensors='pt')
```

## Evaluation signals

- tGPT kBET acceptance rate on HCA embeddings falls within tolerance of the reported ~0.87 (Figure S1L)
- Cell count matches the expected 282,588 bone-marrow cells across 64 healthy donors
- kbet_acceptance_rates.csv contains 12 rows (tGPT plus the 11 batch-correction methods) with valid acceptance rates in [0, 1]
- tGPT acceptance rate is comparable to (not significantly worse than) all 11 batch-corrected baselines
- Embedding extraction used max_len=64 top-expressing genes with mean pooling restricted to non-padding token positions (masked by attention_mask)

## Limitations

- tGPT uses only the top-expressing genes; it may miss information specifically represented within low-expressing genes
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected and this can affect biological interpretation
- Batch information is not always available and often neglected by researchers, so kBET-based harmonization evaluation cannot be run without batch labels

## Evidence
