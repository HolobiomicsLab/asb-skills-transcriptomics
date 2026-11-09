---
name: spatial-cell-type-proportion-visualization
description: Use when you have LDA-derived per-pixel cell-type proportions (theta)
  and pixel positions from multi-cellular pixel-resolution ST data and need to inspect
  where each inferred cell type localizes spatially, e.
when_to_use_negative:
- Input ST data is at sub-cellular pixel resolution — the paper states STdeconvolve
  may not be best suited for such data
- Cell-type proportions are homogeneous/uniform across pixels so genes lack spatial
  variability, or the tissue is a thin structure covering only a few pixels — deconvolution
  and hence spatial proportion maps are unreliable
- A suitable scRNA-seq reference exists and a reference-based method (SPOTlight, RCTD,
  spatialDWLS) is preferred for supervised deconvolution
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: STdeconvolve
  role: Reference-free LDA deconvolution of pixel ST data and spatial visualization
    of deconvolved cell-type proportions (vizAllTopics)
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Execution environment for the STdeconvolve open-source R package
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Installs STdeconvolve from Bioconductor (R >= 4.2 for Bioconductor release
    3.15)
- name: remotes
  role: Alternative installer via remotes::install_github('JEFworks-Lab/STdeconvolve')
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:57a91ec4c79913d260bd4c9077b02093c0bc2025ad1faf5929df5c580ff4c732
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spatial-cell-type-proportion-visualization

## Summary

Visualize per-pixel cell-type proportions from STdeconvolve's reference-free LDA deconvolution overlaid on spatial transcriptomics pixel coordinates, enabling assessment of spatial organization of inferred cell types such as granule cell layer or rostral migratory stream populations in the mouse main olfactory bulb (mOB) ST data.

## When to use

Use when you have LDA-derived per-pixel cell-type proportions (theta) and pixel positions from multi-cellular pixel-resolution ST data and need to inspect where each inferred cell type localizes spatially, e.g. confirming a cell type maps to the granule cell layer or an expected structure like the RMS.

## When NOT to use

- Input ST data is at sub-cellular pixel resolution — the paper states STdeconvolve may not be best suited for such data
- Cell-type proportions are homogeneous/uniform across pixels so genes lack spatial variability, or the tissue is a thin structure covering only a few pixels — deconvolution and hence spatial proportion maps are unreliable
- A suitable scRNA-seq reference exists and a reference-based method (SPOTlight, RCTD, spatialDWLS) is preferred for supervised deconvolution

## Inputs

- STdeconvolve deconvolved cell-type proportion matrix (theta/deconProp: pixels x cell types) from getBetaTheta
- Pixel spatial position matrix (pos, e.g. mOB$pos)
- Optional annotation labels per pixel (e.g. mOB$annot layer assignments)
- Cell-type transcriptional profiles (beta/deconGexp) for extracting top genes of a cell type

## Outputs

- Spatial plot of per-pixel cell-type proportions (vizAllTopics output) overlaid with manual annotation groups
- Identified spatially localized cell type (e.g. X7 overlapping granule cell layer / RMS)
- List of top upregulated marker genes for the cell type (e.g. Sox11, Nrep, Dcx)
- Saved fitted LDA model, proportion matrix, and gene list (e.g. mob_k12_deconvolution.rds)

## How to apply

Run STdeconvolve's LDA (e.g. fitLDA over Ks, select the optimal model, extract beta/theta via getBetaTheta with perc.filt = 0.05 and betaScale = 1000), then plot the per-pixel proportions (theta) against the pixel coordinate matrix (pos) with vizAllTopics, optionally overlaying manual annotation layers (e.g. mOB layers) via groups and group_cols to check whether a cell type's mass overlaps known layers. For the K=12 mOB deconvolution, verify that cell type X7's proportion map localizes over the granule cell layer at the expected rostral migratory stream location rather than dispersing uniformly. Extract the cell type's top upregulated genes (X7: Sox11, Nrep, Dcx) and cross-check their spatial expression against Allen Brain Atlas in situ hybridization to support the spatial assignment.

## Related tools

- **STdeconvolve** (Reference-free LDA deconvolution of pixel ST data and spatial visualization of deconvolved cell-type proportions (vizAllTopics)) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Execution environment for the STdeconvolve open-source R package) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Installs STdeconvolve from Bioconductor (R >= 4.2 for Bioconductor release 3.15))
- **remotes** (Alternative installer via remotes::install_github('JEFworks-Lab/STdeconvolve'))

## Examples

```
results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000); vizAllTopics(results$theta, pos, groups = annot, group_cols = rainbow(length(levels(annot))), r = 0.4)
```

## Evaluation signals

- Per-pixel proportions (theta) are valid fractions (non-negative, summing to 1 per pixel) with one column per inferred cell type
- The proportion map of the cell type of interest (e.g. X7 in the K=12 mOB fit) spatially overlaps the expected layer — granule cell layer at the RMS location — when overlaid with annot groups in vizAllTopics
- Top upregulated genes of the cell type (X7: Sox11, Nrep, Dcx) show matching spatial patterns to Allen Brain Atlas in situ hybridization
- The deconvolution resolution (K) was chosen via STdeconvolve's data-driven metrics, and the spatial structure is finer than prior transcriptional clustering (e.g. RMS resolved)
- Preprocessing filters were applied before visualization: cleanCounts with min.lib.size = 100 and restrictCorpus feature selection of significantly overdispersed genes (removeAbove=1.0, removeBelow=0.05)

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and genes; assumptions not satisfied limit deconvolution quality
- Thin structures or tissue slivers covering only a few pixels are challenging to deconvolve
- If cell-type proportions are homogeneous across pixels, STdeconvolve may fail to deconvolve transcriptionally distinct cell types
- Sparse ST data with high stochastic drop-out rates may hinder LDA from identifying distinct co-expressed gene groups

## Evidence

- [intro] LDA infers cell-type proportions plotted spatially: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [readme] README example: visualize deconvolved proportions with vizAllTopics: "## visualize deconvolved cell-type proportions
vizAllTopics(deconProp, pos, groups = annot, group_cols = rainbow(length(levels(annot))), r=0.4)"
- [readme] Extraction of proportions and transcriptional profiles from the optimal LDA model: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)
deconProp <- results$theta
deconGexp <- results$beta"
- [intro] Overdispersed gene feature selection precedes LDA: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels"
- [discussion] Homogeneous proportions cause failure mode: [evidence span withheld — anchor status: not_found]
