---
name: spatial-transcriptomic-embedding-training
description: Use when you have matched single-cell RNA-seq and spatial transcriptomics
  count matrices (e.g., SMART-seq SC + MERFISH or Visium ST) and need to transfer
  spot-level spatial coordinates or zonation information onto individual cells.
when_to_use_negative:
- When the goal is spot-level deconvolution into a fixed number of cell types — the
  article notes such deconvolution 'cannot link the SC and ST at the single cell level'
  and limits discovery of sub-clusters or continuous spatial variation.
- When SC and ST data come from different donors in the same disease state — the article
  explicitly states this remains untested future work.
- When no spatial coordinate metadata exists for the ST spots, since sigma depends
  on spot adjacent distance and all_coord requires spot coordinates.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STEM / scSTEM
  role: 'Core model: SOmodel with train_wholedata, netE encoder, and all_coord used
    to learn spatially aware embeddings and infer SC-ST mappings'
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Deep-learning backend used to train SOmodel on CUDA for 400 epochs and compute
    softmax embedding dot products
- name: scanpy
  role: 'Preprocessing: sc.pp.normalize_total, sc.pp.log1p, and sc.pp.calculate_qc_metrics
    for computing the dropout-rate domain gap'
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:e2696662bedb40f725210747f5cfba1c442d97b38e860f8c8d117521bc16edbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - PyTorch
  - STEM
  - STEM / scSTEM
  - pandas
  - scSTEM
  - scanpy
  - seaborn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# spatial-transcriptomic-embedding-training

## Summary

Trains the STEM (SpaTially aware EMbedding) deep transfer-learning model to encode SC RNA-seq and spatial transcriptomics data into a unified spatially aware embedding space, from which SC-ST and SC-SC spatial mappings are inferred. This is the core training step that eliminates the SC-ST domain gap and enables single-cell spatial reconstruction.

## When to use

Use when you have matched single-cell RNA-seq and spatial transcriptomics count matrices (e.g., SMART-seq SC + MERFISH or Visium ST) and need to transfer spot-level spatial coordinates or zonation information onto individual cells.

## When NOT to use

- When the goal is spot-level deconvolution into a fixed number of cell types — the article notes such deconvolution 'cannot link the SC and ST at the single cell level' and limits discovery of sub-clusters or continuous spatial variation.
- When SC and ST data come from different donors in the same disease state — the article explicitly states this remains untested future work.
- When no spatial coordinate metadata exists for the ST spots, since sigma depends on spot adjacent distance and all_coord requires spot coordinates.

## Inputs

- Single-cell RNA-seq raw count matrix (e.g., SMART-seq SC counts, CSV/AnnData) with cell metadata
- Spatial transcriptomics raw count matrix (e.g., MERFISH or Visium ST counts) with spot spatial coordinates
- Spot adjacency distance / spatial metadata (for setting sigma)
- Processed data and trained models from Figshare deposit DOI 10.6084/m9.figshare.24452812 (for reproduction)

## Outputs

- Trained STEM SOmodel and loss curve (saved to outf folder, e.g., log/test)
- SC and ST embeddings from netE
- Softmax SC-ST and ST-SC mapping matrices (netsc2st, netst2sc; each row sums to 1)
- Per-cell spatial coordinates in adata.obsm['spatialDA']

## How to apply

Load the SC and ST count matrices and normalize both with sc.pp.normalize_total followed by sc.pp.log1p. Quantify the domain gap as a dropout rate dp = 1 - median(n_genes_by_counts SC) / median(n_genes_by_counts ST), computed with sc.pp.calculate_qc_metrics(log1p=False, inplace=True). Instantiate a SOmodel with n_genes equal to the ST gene count, lr = 0.002, alpha = 0.8 (MMD loss weight), mmdbatch = 1000, sigma set to half the ST spot adjacent distance (the empirically best setting), and the computed dp, then run train_wholedata for 400 epochs on CUDA while saving the loss curve. After training, switch to modeleval, compute SC and ST embeddings with netE, and build the row-normalized softmax mapping matrices netst2sc (spot-to-cell) and netsc2st (cell-to-spot) as softmax of the embedding dot products. Recover per-cell spatial coordinates via all_coord applied to netsc2st and the spot coordinates, stored in adata.obsm['spatialDA'].

## Related tools

- **STEM / scSTEM** (Core model: SOmodel with train_wholedata, netE encoder, and all_coord used to learn spatially aware embeddings and infer SC-ST mappings) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Deep-learning backend used to train SOmodel on CUDA for 400 epochs and compute softmax embedding dot products)
- **scanpy** (Preprocessing: sc.pp.normalize_total, sc.pp.log1p, and sc.pp.calculate_qc_metrics for computing the dropout-rate domain gap)

## Examples

```
from STEM.model import SOmodel; testmodel = SOmodel(opt); loss_curve = testmodel.train_wholedata(400, torch.tensor(scdata.values).float(), torch.tensor(stdata.values).float(), torch.tensor(spcoor.values).float())
```

## Evaluation signals

- Loss curve from train_wholedata decreases smoothly over the 400 epochs (expected shape shown in the README loss.png).
- Rows of netst2sc and netsc2st each sum to 1 (softmax over embedding dot products).
- Reconstructed single-cell coordinates in adata.obsm['spatialDA'] preserve the original topology of all single cells, unlike competing methods (e.g., no compression into L1/L2/3 or offsetting to deeper cortical layers in the MTG test).
- Downstream task validation passes: e.g., L1–L6 cortical-depth distributions align with true layer boundaries, neighborhood-enrichment heatmap shows diagonal enrichment for excitatory neurons, and landmark zonation markers (Cyp2f2 periportal, Alb midlobular, Glul pericentral) are recovered.
- The computed dropout rate dp falls in a plausible range for the dataset (e.g., 0.5837 for the SeqFISH E1z2 demo).

## Limitations

- Application of STEM to SC and ST data from different donors in the same disease state is untested and explicitly deferred to future work.
- Performance depends on empirically chosen parameters — sigma must be set to half the ST spot adjacent distance; mis-specification degrades mapping.
- Identifying SDGs of cancer cells or disease-related cell types as potential drug targets is only proposed, not demonstrated.
- Supplementary Data 1–4 are listed by name only with no description of contents, limiting reproducibility of some supplementary results.

## Evidence

- [readme] Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance.: "Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
- [readme] The matrix netst2sc and netsc2st are ST-SC and SC-ST mapping matrices, respectively.: "In the ST-SC mapping matrix, the probability of one spot to other cells summarizes to 1. In the SC-ST mapping matrix, the probability of one cell to other spots summarizes to 1."
- [readme] The processed data and trained models used for reproducing the results are deposited in Figshare.: "The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812)."
- [abstract] It uses deep transfer learning to encode both ST and SC data into a unified spatially aware embedding space: "It uses deep transfer learning to encode both ST and SC data into a unified spatially aware embedding space"
- [methods] Next we calculate the ratio between the median total counts value of SC and ST data as the dropout rate.: "Next we calculate the ratio between the median total counts value of SC and ST data as the dropout rate."
