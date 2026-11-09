---
name: spatial-axis-differential-expression-analysis
description: Use when you have mapped single cells to spatial coordinates on a ST
  section (e.g. via STEM's all_coord) and need to identify genes differentially expressed
  along a spatial axis within one cell type, or fit spatial expression curves for
  landmark markers.
when_to_use_negative:
- When the goal is spot-level deconvolution into a fixed number of cell types rather
  than single-cell spatial variation — the article notes deconvolution 'cannot link
  the SC and ST at the single cell level'
- When SC and ST data come from different donors or different disease states — the
  article states applying STEM to SC and ST from different donors in the same disease
  state 'remains untested future work'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: STEM
  role: Provides the spatially aware embeddings and SC-ST mapping matrix (netsc2st)
    used to assign cells spatial coordinates and transfer zonation scores before DE
    testing; installed via pip install scSTEM
  repo: https://github.com/WhirlFirst/STEM
- name: scanpy
  role: Normalize total counts (sc.pp.normalize_total) and log1p-transform SC/ST data;
    compute QC metrics (n_genes_by_counts medians) used for the dropout rate
- name: PyTorch
  role: Trains the SOmodel and computes netE embeddings whose softmax dot products
    form the mapping matrices used for coordinate transfer
- name: seaborn
  role: Plotting the fitted spatial expression-curve figures for landmark markers
- name: pandas
  role: Store and validate per-cell-type DE output tables as CSV files
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:3a89af83acc17925dbe23e4608ab5527ceb2b13915429a767f87345aacc826a0
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spatial-axis-differential-expression-analysis

## Summary

Performs cell-type-specific differential expression and expression-curve fitting along a spatial axis (e.g. the hepatic lobule PV-CV zonation axis) after transferring spatial coordinates from ST spots to single cells via STEM's SC-ST mapping matrix. This recovers regionally expressed genes within a cell type that deconvolution approaches cannot resolve.

## When to use

Use when you have mapped single cells to spatial coordinates on a ST section (e.g. via STEM's all_coord) and need to identify genes differentially expressed along a spatial axis within one cell type, or fit spatial expression curves for landmark markers.

## When NOT to use

- When the goal is spot-level deconvolution into a fixed number of cell types rather than single-cell spatial variation — the article notes deconvolution 'cannot link the SC and ST at the single cell level'
- When SC and ST data come from different donors or different disease states — the article states applying STEM to SC and ST from different donors in the same disease state 'remains untested future work'

## Inputs

- Normalized, log1p-transformed SC and ST count matrices (scanpy AnnData)
- SC-ST mapping matrix netsc2st (softmax of scembedding.mm(stembedding.t()), rows sum to 1)
- ST spot spatial coordinates (spcoor / adata.obsm['spatial'])
- Per-cell type annotations (e.g. hepatocytes, endothelial cells, fibroblasts)
- Spot-level zonation scores to transfer to cells

## Outputs

- Per-cell-type CSV tables of differentially expressed / regionally expressed genes with FDR values
- Spatial coordinates for every single cell (adata.obsm['spatialDA'])
- Spatial expression-curve plots for landmark markers per cell type
- Subregion-partitioned gene lists (e.g. 10 hepatocyte subregions along the PV-CV axis)

## How to apply

After training the STEM SOmodel (400 epochs, lr 0.002, MMD loss weight alpha 0.8, sigma set to half the ST spot adjacent distance) and computing the SC-ST mapping matrix netsc2st as softmax of the SC/ST embedding dot product, assign each single cell a spatial coordinate with all_coord applied to netsc2st and the spot coordinates. Transfer zonation scores from ST spots to cells through the same mapping matrix, then subset cells by type (hepatocytes, endothelial cells, fibroblasts) and run differential-expression testing along the axis for each type separately. In the mouse liver application this recovered 819 significant regionally highly expressed hepatocyte genes across 10 spatial subregions (FDR < 0.05), plus 126 periportal / 105 pericentral significant fibroblast genes. Finally fit and plot spatial expression curves for landmark markers (hepatocyte Cyp2f2/Alb/Glul; endothelial Gja5/Adgrg6 vs Ntn4/Lgals1; fibroblast Sparc/Tagln vs Rspo3/DCN/Apoe) to confirm zonation profiles.

## Related tools

- **STEM** (Provides the spatially aware embeddings and SC-ST mapping matrix (netsc2st) used to assign cells spatial coordinates and transfer zonation scores before DE testing; installed via pip install scSTEM) — https://github.com/WhirlFirst/STEM
- **scanpy** (Normalize total counts (sc.pp.normalize_total) and log1p-transform SC/ST data; compute QC metrics (n_genes_by_counts medians) used for the dropout rate)
- **PyTorch** (Trains the SOmodel and computes netE embeddings whose softmax dot products form the mapping matrices used for coordinate transfer)
- **seaborn** (Plotting the fitted spatial expression-curve figures for landmark markers)
- **pandas** (Store and validate per-cell-type DE output tables as CSV files)

## Examples

```
adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)
```

## Evaluation signals

- Output tables exist as CSV files with per-cell-type DE entries and FDR < 0.05 significance calls
- Recovery of ~819 significant regionally expressed hepatocyte genes across 10 subregions (FDR < 0.05) in the mouse liver dataset
- Recovery of 126 periportal and 105 pericentral significant fibroblast genes
- Fitted curves reproduce reported zonation landmark profiles: Cyp2f2 periportal-enriched, Alb midlobular, Glul pericentral; Gja5/Adgrg6 vs Ntn4/Lgals1 for endothelial PV/CV; Sparc/Tagln vs Rspo3/DCN/Apoe for fibroblast PV/CV
- Each row of the SC-ST mapping matrix (netsc2st) sums to 1 before coordinate assignment

## Limitations

- STEM was validated on datasets where ST and SC came from the same healthy region with similar cell type proportions; cross-donor same-disease application is untested
- Using spatially dominant genes (SDGs) of cancer or disease-related cell types as drug-target insights is proposed but not demonstrated
- Continuous spatial gene expression variation within a cell type requires single-cell mapping; fixed-mixture deconvolution representations cannot recover sub-clusters or continuous gradients

## Evidence

- [other] Assign every single cell a spatial coordinate: "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
- [abstract] STEM resolves cell-type-specific spatial variation: "We apply STEM to human squamous cell carcinoma and hepatic lobule datasets to uncover the localization of rare cell types and reveal cell-type-specific gene expression variation along a spatial axis."
- [intro] Deconvolution cannot capture within-type spatial variation: [evidence span withheld — anchor status: not_found]
- [readme] Softmax mapping matrices: "netst2sc = F.softmax(stembedding.mm(scembedding.t()),dim=1).detach().cpu().numpy()"
- [readme] Mapping matrix row sums: "In the SC-ST mapping matrix, the probability of one cell to other spots summarizes to 1."
