---
name: single-cell-spatial-coordinate-mapping
description: Use when you have a paired single-cell RNA-seq matrix and ST spot-level
  count matrix with spot coordinates (e.g. Visium, SeqFISH-derived ST) and need to
  assign each single cell pseudo-spatial coordinates or infer SC-SC spatial adjacency.
when_to_use_negative:
- SC and ST data come from different donors or different tissue regions — the article
  states applying STEM across donors in the same disease state remains untested future
  work
- The goal is spot-level deconvolution into a fixed number of cell types rather than
  single-cell-level mapping; STEM is designed to link SC and ST at the single-cell
  level instead
- Input lacks spot spatial coordinates for the ST data — the Gaussian spatial-variance
  training (sigma) and all_coord coordinate assignment both require spot coordinates
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STEM
  role: 'Core model: SOmodel trains spatially aware embeddings (netE) and derives
    SC-ST / ST-SC softmax mapping matrices; installable via pip install scSTEM'
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Deep-learning backend for SOmodel training on CUDA GPU and embedding computation
- name: scanpy
  role: 'Preprocessing: sc.pp.normalize_total, sc.pp.log1p, sc.pp.calculate_qc_metrics
    for dropout-rate estimation; AnnData container for obsm[''spatialDA'']'
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:d5fcd6c966afb5c09291c190aff6cfecc7a26724c7d60de57b5ee3da55dd5142
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - CellTrek
  - PyTorch
  - STEM
  - Seurat
  - Spaotsc
  - Tangram
  - pandas
  - scSTEM
  - scSpace
  - scanpy
  - seaborn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# single-cell-spatial-coordinate-mapping

## Summary

Maps single-cell RNA-seq cells to spatial coordinates of a matched spatial transcriptomics (ST) section using STEM's deep transfer learning: SC and ST data are encoded into a shared spatially aware embedding space, from which an SC-ST mapping matrix assigns each cell pseudo-spatial coordinates and predicts SC-SC spatial adjacency.

## When to use

Use when you have a paired single-cell RNA-seq matrix and ST spot-level count matrix with spot coordinates (e.g. Visium, SeqFISH-derived ST) and need to assign each single cell pseudo-spatial coordinates or infer SC-SC spatial adjacency.

## When NOT to use

- SC and ST data come from different donors or different tissue regions — the article states applying STEM across donors in the same disease state remains untested future work
- The goal is spot-level deconvolution into a fixed number of cell types rather than single-cell-level mapping; STEM is designed to link SC and ST at the single-cell level instead
- Input lacks spot spatial coordinates for the ST data — the Gaussian spatial-variance training (sigma) and all_coord coordinate assignment both require spot coordinates

## Inputs

- SC (single-cell RNA-seq) count matrix, e.g. raw SeqFISH CSV transposed to cells x genes (data/mousedata_2020/E1z2/simu_sc_counts.csv)
- ST (spatial transcriptomics) spot-level count matrix with spot spatial coordinates (e.g. simu_st_counts.csv and simu_st_metadata.csv, or Visium sections)
- SC cell metadata with x_global/y_global coordinates where available (metadata.csv)
- Optional spot-level ground-truth cell types or scores to transfer (e.g. simu_st_celltype.csv, zonation scores)

## Outputs

- Row-normalized SC-ST mapping matrix netsc2st (each cell's probability distribution over spots) and ST-SC matrix netst2sc
- Pseudo-spatial coordinates per single cell stored in adata.obsm['spatialDA']
- Predicted SC-SC spatial adjacency supporting downstream analyses (hit-number@K, neighborhood enrichment, per-cell-type spatial DE)

## How to apply

Normalize both SC and ST count matrices with sc.pp.normalize_total and sc.pp.log1p, then compute the dropout rate dp = 1 - median(SC n_genes_by_counts)/median(ST n_genes_by_counts) via sc.pp.calculate_qc_metrics (e.g. dp ≈ 0.584 for E1z2). Train the STEM SOmodel on GPU for 400 epochs with lr=0.002, sigma set to half the ST spot adjacent distance, MMD loss weight alpha=0.8, mmdbatch=1000, and seed_all(2022); dp is used as the dropout rate for ST data. After training (modeleval), extract SC and ST embeddings with netE and build row-normalized softmax mapping matrices netst2sc = softmax(stE·scE^T) and netsc2st = softmax(scE·stE^T), each row summing to 1. Assign every single cell spatial coordinates with all_coord applied to netsc2st and the ST spot coordinates (stored in adata.obsm['spatialDA']); netsc2st also supports transferring spot-level scores (e.g. zonation) to cells. Sigma choice matters empirically: half the ST spot adjacent distance gives the best performance.

## Related tools

- **STEM** (Core model: SOmodel trains spatially aware embeddings (netE) and derives SC-ST / ST-SC softmax mapping matrices; installable via pip install scSTEM) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Deep-learning backend for SOmodel training on CUDA GPU and embedding computation)
- **scanpy** (Preprocessing: sc.pp.normalize_total, sc.pp.log1p, sc.pp.calculate_qc_metrics for dropout-rate estimation; AnnData container for obsm['spatialDA'])

## Examples

```
testmodel = SOmodel(opt); testmodel.togpu(); testmodel.train_wholedata(400, torch.tensor(scdata.values).float(), torch.tensor(stdata.values).float(), torch.tensor(spcoor.values).float()); adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st, index=adata.obs_names, columns=stadata.obs_names), spcoor)
```

## Evaluation signals

- Row sums of netst2sc and netsc2st equal 1 after softmax normalization (each spot-to-cells and cell-to-spots distribution sums to 1)
- On semi-simulation E1z2, mapped coordinates should achieve the lowest MAE versus ground truth and roughly two-fold higher hit-number@K=200 than the second-best of CellTrek, scSpace, Seurat, Spaotsc, Tangram
- Loss curve over the 400-epoch whole-data training converges as in the README's loss.png reference
- Dropout rate dp computed from median n_genes_by_counts matches the expected ratio (e.g. 0.5837 for E1z2)
- Transferred spot-level signals are consistent with known biology (e.g. Cyp2f2 periportal, Alb midlobular, Glul pericentral along the PV-CV axis)

## Limitations

- Applying STEM to SC and ST data from different donors in the same disease state remains untested future work
- Performance depends on empirically setting sigma to half the ST spot adjacent distance; mis-specified sigma degrades mapping
- Identifying spatially dominant genes (SDGs) of cancer cells as drug-target insights is proposed but not demonstrated in the article

## Evidence

- [abstract] unified spatially aware embedding space: [evidence span withheld — anchor status: not_found]
- [methods] dropout rate from median genes-by-counts: "Next we calculate the ratio between the median total counts value of SC and ST data as the dropout rate."
- [readme] softmax mapping matrices: [evidence span withheld — anchor status: not_found]
- [readme] sigma as half spot adjacent distance: "Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
- [discussion] untested across donors: "In the future, we will try to apply STEM to SC and ST data from different donors in the same disease state."
