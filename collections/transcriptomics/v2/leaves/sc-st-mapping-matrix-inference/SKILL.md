---
name: sc-st-mapping-matrix-inference
description: Use when you have normalized SC and ST datasets and need to map every
  single cell to a spatial coordinate on ST spots — e.g. to build a single-cell level
  spatial transcriptomic landscape or run cell-type-specific spatial DE along a tissue
  axis.
when_to_use_negative:
- When the goal is spot-level deconvolution into a fixed number of cell types — STEM's
  single-cell mapping is explicitly positioned against deconvolution approaches that
  treat a spot as a fixed mixture.
- When SC and ST data come from different donors or disease states — the article notes
  this remains untested future work (the mouse liver application relied on same-region,
  similar cell-type-proportion data).
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STEM
  role: Trains the spatially aware embedding model (SOmodel) whose netE embeddings
    and softmax dot products yield the SC-ST / ST-SC mapping matrices; all_coord maps
    cells to coordinates
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Backend for SOmodel training and GPU embedding computation (netE forward passes
    on cuda tensors)
- name: scanpy
  role: Normalization (sc.pp.normalize_total, sc.pp.log1p) and QC metrics used to
    compute the dropout rate before mapping
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:1d8016646f5963ea17213698df30bf470e3066340e7a4891f71a37c912c57920
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - PyTorch
  - STEM
  - pandas
  - scanpy
  - seaborn
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# sc-st-mapping-matrix-inference

## Summary

Infers SC-ST and ST-SC mapping matrices by encoding single-cell RNA-seq and spatial transcriptomics data into STEM's unified spatially aware embedding space, then taking softmax of embedding dot products so each row sums to 1. This assigns single cells spatial coordinates and transfers spot-level spatial information (e.g. zonation scores) to individual cells.

## When to use

Use when you have normalized SC and ST datasets and need to map every single cell to a spatial coordinate on ST spots — e.g. to build a single-cell level spatial transcriptomic landscape or run cell-type-specific spatial DE along a tissue axis.

## When NOT to use

- When the goal is spot-level deconvolution into a fixed number of cell types — STEM's single-cell mapping is explicitly positioned against deconvolution approaches that treat a spot as a fixed mixture.
- When SC and ST data come from different donors or disease states — the article notes this remains untested future work (the mouse liver application relied on same-region, similar cell-type-proportion data).

## Inputs

- Normalized, log1p-transformed SC count matrix (genes x cells, e.g. from scRNA-seq or SeqFISH CSV)
- Normalized, log1p-transformed ST count matrix with spot spatial coordinates (e.g. Visium ST sections)
- Spot spatial coordinates metadata table (spcoor)

## Outputs

- netst2sc: ST-SC mapping matrix (rows = spots, softmax over cells, each row sums to 1)
- netsc2st: SC-ST mapping matrix (rows = cells, softmax over spots, each row sums to 1)
- Per-cell spatial coordinates in adata.obsm['spatialDA'] via all_coord
- Spot-to-cell transferred values (e.g. zonation scores) for downstream DE testing

## How to apply

First normalize both datasets with sc.pp.normalize_total and sc.pp.log1p, and compute the dropout rate as 1 minus the ratio of median genes-by-counts between SC and ST data. Train the STEM SOmodel (GPU, lr=0.002, MMD loss weight alpha=0.8, sigma set to half the ST spot adjacent distance) for ~400 epochs, then compute SC and ST embeddings with netE. Build the mapping matrices as softmax of the embedding dot products: netst2sc = softmax(st_embedding @ sc_embedding.T, dim=1) and netsc2st = softmax(sc_embedding @ st_embedding.T, dim=1), where rows of netst2sc (spot-to-cells) and netsc2st (cell-to-spots) each sum to 1. Assign every single cell a spatial coordinate via all_coord applied to the netsc2st matrix and the spot coordinates, storing it in adata.obsm['spatialDA']; downstream zonation scores or spot-level values can be transferred to cells via the same mapping matrix.

## Related tools

- **STEM** (Trains the spatially aware embedding model (SOmodel) whose netE embeddings and softmax dot products yield the SC-ST / ST-SC mapping matrices; all_coord maps cells to coordinates) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Backend for SOmodel training and GPU embedding computation (netE forward passes on cuda tensors))
- **scanpy** (Normalization (sc.pp.normalize_total, sc.pp.log1p) and QC metrics used to compute the dropout rate before mapping)

## Examples

```
netsc2st = F.softmax(scembedding.mm(stembedding.t()),dim=1).detach().cpu().numpy(); adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)
```

## Evaluation signals

- Each row of netst2sc and netsc2st must sum to 1 (softmax over the embedding dot products).
- Every single cell receives a finite spatial coordinate in adata.obsm['spatialDA'] after all_coord.
- The SC-ST dropout rate computed as 1 - median(SC n_genes_by_counts)/median(ST n_genes_by_counts) falls in a plausible range (e.g. ~0.58 in the SeqFISH demo).
- Mapping quality check: STEM should preserve the original topology structure of single cells when compared against ground-truth spatial layouts (e.g. simulated E1z2 data).
- Downstream validation: transferring zonation scores through netsc2st recovers reported spatial marker profiles (e.g. Cyp2f2 periportal, Glul pericentral in hepatocytes).

## Limitations

- STEM assumes ST and SC data share spatial gene-expression relationships; applying it to SC and ST data from different donors in the same disease state is explicitly untested future work.
- Model performance depends on setting sigma to half of the ST spot adjacent distance; mis-specified spatial variance can degrade mapping.
- The mapping is probabilistic (softmax), so cells are assigned soft coordinates — interpretation of hard assignments requires care.

## Evidence

- [abstract] It uses deep transfer learning to encode both ST and SC data into a unified spatially aware embedding space: "It uses deep transfer learning to encode both ST and SC data into a unified spatially aware embedding space"
- [readme] mapping matrix construction: "netst2sc = F.softmax(stembedding.mm(scembedding.t()),dim=1).detach().cpu().numpy()"
- [readme] row normalization semantics: "In the SC-ST mapping matrix, the probability of one cell to other spots summarizes to 1."
- [methods] coordinate assignment: "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
- [readme] dropout rate: "Next we calculate the ratio between the median total counts value of SC and ST data as the dropout rate."
