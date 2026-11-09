---
name: spot-to-cell-zonation-score-transfer
description: Use when you have trained STEM embeddings for paired scRNA-seq and Visium
  ST data and need to assign each single cell a pseudo-spatial coordinate or transfer
  spot-level scores (e.g., PV-CV zonation) to cells for cell-type-specific spatial
  DE testing.
when_to_use_negative:
- When SC and ST data come from different donors or different disease states — STEM
  is untested in that setting per the article's discussion
- When the goal is spot-level deconvolution into a fixed number of cell types rather
  than single-cell-level spatial mapping — the article notes deconvolution 'cannot
  link the SC and ST at the single cell level'
- When input ST data lacks usable spatial coordinates, since all_coord requires spot
  coordinate arrays (spcoor) to place cells
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STEM
  role: Trains the spatially-aware embedding model and supplies netE, the softmax
    mapping matrices, and the all_coord function used to transfer spot coordinates
    and scores to cells
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Backend for SOmodel training and netE embedding computation on GPU
  repo: https://github.com/WhirlFirst/STEM
- name: scanpy
  role: Loads/preprocesses SC and ST data (sc.pp.normalize_total, sc.pp.log1p, sc.pp.calculate_qc_metrics
    for the dropout-rate computation)
- name: pandas
  role: Builds the netsc2st DataFrame indexed by adata.obs_names with stadata.obs_names
    columns passed to all_coord
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:2203ad13ec6d0b9231db9f693fcd3641f05b0e634704daddb2a117551ccbb347
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# spot-to-cell-zonation-score-transfer

## Summary

Transfers spatial information (coordinates and zonation/axis scores) from spatial transcriptomics spots onto individual single cells using STEM's learned SC-ST mapping matrix (netsc2st), enabling cell-type-specific spatial variation analyses such as PV-CV zonation in the mouse liver lobule.

## When to use

Use when you have trained STEM embeddings for paired scRNA-seq and Visium ST data and need to assign each single cell a pseudo-spatial coordinate or transfer spot-level scores (e.g., PV-CV zonation) to cells for cell-type-specific spatial DE testing.

## When NOT to use

- When SC and ST data come from different donors or different disease states — STEM is untested in that setting per the article's discussion
- When the goal is spot-level deconvolution into a fixed number of cell types rather than single-cell-level spatial mapping — the article notes deconvolution 'cannot link the SC and ST at the single cell level'
- When input ST data lacks usable spatial coordinates, since all_coord requires spot coordinate arrays (spcoor) to place cells

## Inputs

- Trained STEM SOmodel with netE encoder (PyTorch GPU checkpoint)
- Normalized, log1p-transformed scRNA-seq count matrix (e.g., mouse liver cell atlas, scanpy AnnData)
- Normalized, log1p-transformed Visium ST spot count matrix with spot spatial coordinates (spcoor)
- Spot-level spatial scores to transfer (e.g., PV-CV zonation scores)
- Computed SC-ST dropout rate: 1 minus the ratio of median n_genes_by_counts between SC and ST data

## Outputs

- Per-cell pseudo-spatial coordinates stored in adata.obsm['spatialDA'] via all_coord(netsc2st, spcoor)
- SC-ST mapping matrix (netsc2st) with rows summing to 1 over spots
- Per-cell transferred zonation scores
- Cell-type-specific spatially variable gene tables (e.g., 819 significant hepatocyte genes across 10 PV-CV subregions, FDR < 0.05)
- Fitted spatial expression curves for landmark markers (Cyp2f2/Alb/Glul, Gja5/Adgrg6 vs Ntn4/Lgals1, Sparc/Tagln vs Rspo3/DCN/Apoe)

## How to apply

After training the STEM SOmodel (e.g., 400 epochs, lr=0.002, alpha=0.8 MMD weight, sigma = half the ST spot adjacent distance) and computing netE embeddings, build the SC-ST mapping matrix as netsc2st = softmax(sc_embedding @ st_embedding.T), so each cell's row sums to 1 over spots. Assign every single cell a spatial coordinate by applying the all_coord function to the netsc2st DataFrame (rows = adata.obs_names, columns = stadata.obs_names) together with the spot coordinates (spcoor), storing the result in adata.obsm['spatialDA']. Transfer continuous spatial scores (e.g., zonation axis scores) from spots to cells through the same mapping matrix, then perform differential-expression testing along the axis per cell type — e.g., dividing hepatocytes into 10 subregions along the PV-CV axis. Validation relies on recovering known landmark profiles: Cyp2f2 periportal, Alb midlobular, Glul pericentral in hepatocytes.

## Related tools

- **STEM** (Trains the spatially-aware embedding model and supplies netE, the softmax mapping matrices, and the all_coord function used to transfer spot coordinates and scores to cells) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Backend for SOmodel training and netE embedding computation on GPU) — https://github.com/WhirlFirst/STEM
- **scanpy** (Loads/preprocesses SC and ST data (sc.pp.normalize_total, sc.pp.log1p, sc.pp.calculate_qc_metrics for the dropout-rate computation))
- **pandas** (Builds the netsc2st DataFrame indexed by adata.obs_names with stadata.obs_names columns passed to all_coord)

## Examples

```
netsc2st = F.softmax(scembedding.mm(stembedding.t()),dim=1).detach().cpu().numpy(); adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)
```

## Evaluation signals

- Row-wise sums of netsc2st equal 1 (softmax over spots for each cell)
- adata.obsm['spatialDA'] exists for every cell in adata.obs_names and falls within the convex hull of the original spot coordinates
- Transferred zonation scores reproduce reported landmark profiles: Cyp2f2 periportal-enriched, Alb midlobular, Glul pericentral; Gja5/Adgrg6 (endothelial PV) vs Ntn4/Lgals1 (CV); Sparc/Tagln (fibroblast PV) vs Rspo3/DCN/Apoe (CV)
- Per-cell-type DE tables recover the expected counts: ~819 significant hepatocyte genes across 10 PV-CV subregions at FDR < 0.05, and 126 periportal / 105 pericentral significant fibroblast genes
- Dropout rate computed as 1 - median(n_genes_by_counts SC)/median(n_genes_by_counts ST) matches a plausible value (e.g., ~0.58 in the demo)

## Limitations

- STEM was demonstrated only when ST and SC data share the same healthy region with similar cell-type proportions; application to SC and ST from different donors in the same disease state is stated as untested future work
- Sigma must be set empirically to half of the ST spot adjacent distance for best performance
- Mapping quality depends on eliminating the SC-ST domain gap; the dropout-rate calculation (median genes-by-counts ratio) is the mechanism used, and misestimating it degrades the mapping

## Evidence

- [methods] Assign every single cell a spatial coordinate using the all_coord function applied to the SC-ST mapping matrix and the spot coordinates: "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
- [readme] Mapping matrices are softmax of embedding dot products, rows sum to 1: "In the SC-ST mapping matrix, the probability of one cell to other spots summarizes to 1."
- [readme] Dropout rate computation for SC-ST domain gap correction: "Next we calculate the ratio between the median total counts value of SC and ST data as the dropout rate."
- [readme] Sigma parameter guidance for model configuration: "Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
- [abstract] Biological validation via cell-type-specific spatial gene variation on the hepatic lobule: "We apply STEM to human squamous cell carcinoma and hepatic lobule datasets to uncover the localization of rare cell types and reveal cell-type-specific gene expression variation along a spatial axis."
