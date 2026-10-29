---
name: hepatocyte-zonation-landmark-gene-interpretation
description: Use when single cells have been assigned pseudo-spatial coordinates along
  the hepatic lobule PV-CV axis (e.g. via STEM's netsc2st mapping and all_coord) and
  you need to identify regionally expressed genes per cell type and confirm zonation
  biology with landmark markers.
when_to_use_negative:
- ST and SC data come from different donors or different disease states — the article
  states this application 'remains untested future work' (the mouse liver demonstration
  required same healthy region with similar cell type proportions)
- The goal is spot-level deconvolution into a fixed number of cell types rather than
  single-cell continuous spatial variation — the article notes deconvolution 'cannot
  link the SC and ST at the single cell level'
- Tissue lacks a known continuous spatial axis (e.g. dissociated cultures without
  spatial coordinates) — the zonation-axis DE test requires ordered pseudo-spatial
  positions
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STEM
  role: Provides the spatially aware embeddings, netsc2st mapping matrix, and all_coord
    function used to place single cells along the PV-CV axis before zonation DE testing
  repo: https://github.com/WhirlFirst/STEM
- name: scanpy
  role: Normalization (sc.pp.normalize_total), log1p transformation, QC metrics, and
    downstream differential-expression testing on AnnData objects
- name: seaborn
  role: Plotting the spatial expression curves for landmark zonation markers
- name: pandas
  role: Handling the netsc2st mapping DataFrame indexed by cell obs_names and columned
    by spot obs_names, and writing per-cell-type DE CSV tables
- name: PyTorch
  role: Runs the SOmodel embedding computations (netE) on GPU that feed the mapping
    matrices
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:2be247d57080b134dfbc288b3c2f6930b7f188b4e7d59cdc7a609dc2c38bcf51
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hepatocyte-zonation-landmark-gene-interpretation

## Summary

Interprets STEM-mapped single-cell spatial landscapes of the mouse liver by testing differential expression along the portal vein–central vein (PV-CV) zonation axis and validating cell-type-specific spatial patterns against landmark markers (Cyp2f2, Alb, Glul for hepatocytes; Gja5/Adgrg6 vs Ntn4/Lgals1 for endothelial; Sparc/Tagln vs Rspo3/DCN/Apoe for fibroblasts).

## When to use

Use when single cells have been assigned pseudo-spatial coordinates along the hepatic lobule PV-CV axis (e.g. via STEM's netsc2st mapping and all_coord) and you need to identify regionally expressed genes per cell type and confirm zonation biology with landmark markers.

## When NOT to use

- ST and SC data come from different donors or different disease states — the article states this application 'remains untested future work' (the mouse liver demonstration required same healthy region with similar cell type proportions)
- The goal is spot-level deconvolution into a fixed number of cell types rather than single-cell continuous spatial variation — the article notes deconvolution 'cannot link the SC and ST at the single cell level'
- Tissue lacks a known continuous spatial axis (e.g. dissociated cultures without spatial coordinates) — the zonation-axis DE test requires ordered pseudo-spatial positions

## Inputs

- STEM-trained SOmodel embeddings (netE) and netsc2st SC-ST mapping matrix (rows sum to 1)
- ST spot spatial coordinates (spcoor) from the mouse liver Visium sections
- Normalized, log1p-transformed SC and ST count matrices (scanpy AnnData)
- Cell-type annotations for hepatocytes, endothelial cells, and fibroblasts from the matched scRNA-seq data

## Outputs

- Per-cell-type CSV tables of zonation-axis differential-expression entries (FDR < 0.05)
- Spatial expression-curve plots for landmark markers (e.g. Cyp2f2/Alb/Glul, Gja5/Adgrg6/Ntn4/Lgals1, Sparc/Tagln/Rspo3/DCN/Apoe)
- Hepatocyte subregion assignments (10 spatial subregions along the zonation axis)

## How to apply

After STEM assigns each single cell a spatial coordinate via all_coord applied to the netsc2st softmax mapping matrix and spot coordinates, transfer zonation scores from spots to cells through the same mapping matrix, then bin cells of each type (e.g. hepatocytes into 10 subregions) along the PV-CV axis and run differential-expression testing, keeping genes with FDR < 0.05 (yielding ~819 significant regionally expressed hepatocyte genes, 126 periportal and 105 pericentral fibroblast genes). Fit and plot spatial expression curves per gene with seaborn/pandas to check the expected landmark profiles: Cyp2f2 periportal-enriched, Alb midlobular, Glul pericentral for hepatocytes; Gja5/Adgrg6 (PV) vs Ntn4/Lgals1 (CV) for endothelial cells; Sparc/Tagln (PV) vs Rspo3/DCN/Apoe (CV) for fibroblasts. Correct application is judged when output CSV tables contain per-cell-type DE entries and the fitted curves reproduce the reported PV/middle/CV landmark profiles.

## Related tools

- **STEM** (Provides the spatially aware embeddings, netsc2st mapping matrix, and all_coord function used to place single cells along the PV-CV axis before zonation DE testing) — https://github.com/WhirlFirst/STEM
- **scanpy** (Normalization (sc.pp.normalize_total), log1p transformation, QC metrics, and downstream differential-expression testing on AnnData objects)
- **seaborn** (Plotting the spatial expression curves for landmark zonation markers)
- **pandas** (Handling the netsc2st mapping DataFrame indexed by cell obs_names and columned by spot obs_names, and writing per-cell-type DE CSV tables)
- **PyTorch** (Runs the SOmodel embedding computations (netE) on GPU that feed the mapping matrices)

## Examples

```
adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)
```

## Evaluation signals

- Output CSV files exist with per-cell-type differential-expression entries
- ~819 statistically significant regionally highly expressed hepatocyte genes recovered across 10 subregions at FDR < 0.05
- 126 periportal and 105 pericentral significant fibroblast genes recovered
- Fitted curves show Cyp2f2 periportal-enriched, Alb midlobular, and Glul pericentral hepatocyte profiles
- Endothelial curves show Gja5/Adgrg6 (PV) vs Ntn4/Lgals1 (CV); fibroblast curves show Sparc/Tagln (PV) vs Rspo3/DCN/Apoe (CV)

## Limitations

- STEM was only demonstrated for ST and SC data from the same healthy region with similar cell type proportions (mouse liver ST and SC were not from the same donor); application to different donors in the same disease state is untested future work
- Landmark profile validation depends on the sigma parameter being set to half of the ST spot adjacent distance for correct embedding-based mapping
- Interpretation is restricted to cell types represented in the matched scRNA-seq reference (hepatocytes, endothelial cells, fibroblasts in this analysis)

## Evidence

- [other] 819 significant regionally highly expressed genes: [evidence span withheld — anchor status: not_found]
- [abstract] STEM mapping rationale: "We apply STEM to human squamous cell carcinoma and hepatic lobule datasets to uncover the localization of rare cell types and reveal cell-type-specific gene expression variation along a spatial axis"
- [methods] all_coord usage: "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
- [intro] deconvolution limitation: [evidence span withheld — anchor status: not_found]
- [methods] dropout rate computation: "Next we calculate the ratio between the median total counts value of SC and ST data as the dropout rate."
- [readme] sigma setting rationale: "Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
