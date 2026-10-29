---
name: cell-type-neighborhood-enrichment-analysis
description: Use when single-cell data have been pseudo-spatially mapped onto ST spots
  (e.g., via STEM's netsc2st mapping) and the goal is to quantify spatial proximity/colocalization
  between rare cell types such as TSKs, endothelial cells, or pDCs.
when_to_use_negative:
- ST data have not been integrated with SC data and only spot-level deconvolution
  is needed — deconvolution treats a spot as a mixture of a fixed number of cell types
  and cannot link SC and ST at the single-cell level
- SC and ST data come from different donors; the article notes applying STEM across
  donors in the same disease state remains untested future work
- Input is a spot-level cell-type composition matrix rather than single-cell pseudo-spatial
  assignments; the enrichment analysis requires per-cell mapped coordinates
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STEM
  role: Deep transfer learning model that provides the netE embeddings and softmax
    SC-ST mapping matrices (netsc2st) from which pseudo-spatial coordinates for the
    neighborhood-enrichment analysis are derived
  repo: https://github.com/WhirlFirst/STEM
- name: scSTEM
  role: PyPI-installable distribution of STEM used to run the SC-ST mapping pipeline
    that feeds the enrichment analysis
- name: scanpy
  role: Used to load data into AnnData, normalize (sc.pp.normalize_total), log-transform
    (sc.pp.log1p), compute QC metrics (sc.pp.calculate_qc_metrics), and attach spatial
    coordinates via adata.obsm
- name: PyTorch
  role: Backend for training the STEM SOmodel (CUDA) and computing softmax embedding
    dot-product mapping matrices
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:e82ab59a11fcbb52c131fe0347343b86d0e9a3a4031df126805ae70c89ff29e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - PyTorch
  - STEM
  - scSTEM
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cell-type-neighborhood-enrichment-analysis

## Summary

Computes row-normalized neighborhood-enrichment matrices among mapped cell-type subpopulations (e.g., keratinocyte subtypes including tumor-specific keratinocytes) in pseudo-spatial coordinates produced by STEM, to test whether specific cell types colocalize or self-aggregate (e.g., TSKs vs. tumor cycling KCs, TSKs vs. endothelial cells at the tumor leading edge).

## When to use

Use when single-cell data have been pseudo-spatially mapped onto ST spots (e.g., via STEM's netsc2st mapping) and the goal is to quantify spatial proximity/colocalization between rare cell types such as TSKs, endothelial cells, or pDCs.

## When NOT to use

- ST data have not been integrated with SC data and only spot-level deconvolution is needed — deconvolution treats a spot as a mixture of a fixed number of cell types and cannot link SC and ST at the single-cell level
- SC and ST data come from different donors; the article notes applying STEM across donors in the same disease state remains untested future work
- Input is a spot-level cell-type composition matrix rather than single-cell pseudo-spatial assignments; the enrichment analysis requires per-cell mapped coordinates

## Inputs

- SC and ST count matrices for hSCC donors 2 and 10 with cell-type labels and spot spatial coordinates (Figshare deposit DOI 10.6084/m9.figshare.24452812)
- Row-normalized SC-ST mapping matrix netsc2st from a trained STEM model
- ST spot spatial coordinates (spcoor)
- Cell-type/subtype annotations for mapped single cells (e.g., TSKs, tumor cycling KCs, endothelial cells, pDCs)

## Outputs

- Row-normalized neighborhood-enrichment matrix among cell-type subtypes (saved as CSV)
- Mapped spatial cell-type table with pseudo-spatial coordinates in adata.obsm['spatialDA']
- Colocalization test results (e.g., TSK–endothelial cell colocalization at the top tumor leading edge)
- Marker-gene comparison table for pDC-localized vs. other spots (BST2, NRP1, JCHAIN; P < 0.01)

## How to apply

First generate pseudo-spatial coordinates for every single cell by applying all_coord to the row-normalized SC-ST softmax mapping matrix netsc2st (built from STEM's netE embeddings via F.softmax(scembedding.mm(stembedding.t()),dim=1)) together with the ST spot spatial coordinates. Then compute the row-normalized neighborhood-enrichment matrix among the cell-type subtypes present in the mapped data — in the hSCC reproduction this included keratinocyte subtypes, verifying that tumor-specific keratinocytes (TSKs) self-aggregate away from tumor cycling KCs. Finally, test colocalization of a target cell type with a spatially defined region or partner type, e.g., TSK colocalization with endothelial cells at the top tumor leading edge in both donors, and export the enrichment matrix as CSV. Marker-gene comparisons (e.g., BST2, NRP1, JCHAIN between pDC-localized and other spots, significance P < 0.01) can corroborate enrichment calls.

## Related tools

- **STEM** (Deep transfer learning model that provides the netE embeddings and softmax SC-ST mapping matrices (netsc2st) from which pseudo-spatial coordinates for the neighborhood-enrichment analysis are derived) — https://github.com/WhirlFirst/STEM
- **scSTEM** (PyPI-installable distribution of STEM used to run the SC-ST mapping pipeline that feeds the enrichment analysis)
- **scanpy** (Used to load data into AnnData, normalize (sc.pp.normalize_total), log-transform (sc.pp.log1p), compute QC metrics (sc.pp.calculate_qc_metrics), and attach spatial coordinates via adata.obsm)
- **PyTorch** (Backend for training the STEM SOmodel (CUDA) and computing softmax embedding dot-product mapping matrices)

## Examples

```
testmodel.modeleval(); scembedding = testmodel.netE(torch.tensor(scdata.values,dtype=torch.float32).cuda()); netsc2st = F.softmax(scembedding.mm(stembedding.t()),dim=1).detach().cpu().numpy(); adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)
```

## Evaluation signals

- Row sums of netst2sc (per spot) and netsc2st (per cell) equal 1, since softmax mapping matrices are row-normalized
- Enrichment matrix is row-normalized and shows TSKs self-aggregating away from tumor cycling KCs
- TSK colocalization with endothelial cells at the top tumor leading edge reproduces in both hSCC donors (2 and 10), matching the original signature-based study without manual gene selection
- Loss curve from train_wholedata over 400 epochs converges (as shown in the README's expected loss.png)
- pDC/immune marker genes BST2, NRP1, JCHAIN are significantly higher in pDC-localized spots (P < 0.01)

## Limitations

- STEM has not yet been tested on SC and ST data from different donors in the same disease state
- Identifying SDGs of cancer cells or disease-related cell types as potential drug targets is only proposed, not demonstrated
- Enrichment conclusions depend on the quality of the SC-ST mapping; sigma should be set to half the ST spot adjacent distance and dropout rate dp estimated as 1 minus the ratio of median n_genes_by_counts between SC and ST data

## Evidence

- [intro] Such a single-cell level spatial landscape would greatly help investigate the spatial proximity of different cell types: "Such a single-cell level spatial landscape would greatly help investigate the spatial proximity of different cell types"
- [readme] In the ST-SC mapping matrix, the probability of one spot to other cells summarizes to 1.: "In the ST-SC mapping matrix, the probability of one spot to other cells summarizes to 1. In the SC-ST mapping matrix, the probability of one cell to other spots summarizes to 1."
- [methods] adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor): "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
- [discussion] The analysis on the MTG and hSCC datasets showed the ability of STEM to integrate SC data with different types (in-situ or sequenced based) of ST data.: "The analysis on the MTG and hSCC datasets showed the ability of STEM to integrate SC data with different types (in-situ or sequenced based) of ST data."
- [readme] The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812).: "The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812)."
