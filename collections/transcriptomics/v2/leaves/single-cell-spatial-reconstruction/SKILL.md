---
name: single-cell-spatial-reconstruction
description: Use when scRNA-seq data must be mapped onto 2D/3D tissue coordinates
  without marker genes or a reference atlas, e.g. BDTNP dge.txt/geometry.txt pairs
  where spatial fidelity should be validated against FISH.
when_to_use_negative:
- Space geometry is complex (e.g. tissue cross-section) or gene expression count measurements
  are inaccurate, so perfect reconstruction may not hold
- Dataset contains data points from only one class (no contrast), where CCSF works
  similarly to kernel PCA and offers only speed/linearization benefits over PCA
- A marker-gene-based novoSpaRc reconstruction is acceptable and preferred for interpretability
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: novoSpaRc
  role: Provides subsample_dge for aligning cells to geometry and is the spatial-mapping
    backbone executed on CCSF components via cSpaRc
  repo: github.com/xinglab-ai/ccsf
- name: ccsf
  role: Supplies the CCSF class (metric_learning, cSpaRc) that learns the correlation
    metric and cell-cell similarity representation feeding the spatial reconstruction
  repo: github.com/xinglab-ai/ccsf
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2023.100840
    title: Leveraging cell-cell similarity for high-performance spatial and temporal
      cellular mappings from gene expression data
  source_content_hash: sha256:9d5b9b1b3cef86c2d6ca0821680e29b8326030f2537c85846ca21b270460cd5f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinglab-ai/ccsf
  tools:
  - ccsf
  - novoSpaRc
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-spatial-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Markerless spatial reconstruction of single-cell gene expression onto tissue coordinates by running novoSpaRc on CCSF-transformed data (cSpaRc), achieving near-perfect correlation with ground-truth FISH maps on the Drosophila embryo BDTNP dataset without marker genes.

## When to use

Use when scRNA-seq data must be mapped onto 2D/3D tissue coordinates without marker genes or a reference atlas, e.g. BDTNP dge.txt/geometry.txt pairs where spatial fidelity should be validated against FISH.

## When NOT to use

- Space geometry is complex (e.g. tissue cross-section) or gene expression count measurements are inaccurate, so perfect reconstruction may not hold
- Dataset contains data points from only one class (no contrast), where CCSF works similarly to kernel PCA and offers only speed/linearization benefits over PCA
- A marker-gene-based novoSpaRc reconstruction is acceptable and preferred for interpretability

## Inputs

- dge.txt: scRNA-seq expression matrix (84 transcription factors, BDTNP Drosophila embryo)
- geometry.txt: 3D embryo coordinates (use columns 0 and 2 as 2D locations)
- gene_names row (first line of dge.txt, parsed via np.genfromtxt dtype='str', max_rows=1)
- Gene name to reconstruct, e.g. 'ftz' or 'sna'

## Outputs

- Spatial reconstruction matrix (embedding_CSPARC) of gene expression per location
- Per-gene spatial map, e.g. ftz_spatial_reconstruction.png scatter plot of locations colored by reconstructed expression
- Correlation with ground-truth FISH spatial map

## How to apply

Load the expression matrix (dge.txt, usecols=range(84), skiprows=1) and embryo geometry (geometry.txt, usecols=range(3), skiprows=1, columns 0 and 2 as 2D locations), then align cells to coordinates with novosparc.pp.subsample_dge(dge, 3039, 3040), subsetting locations to cells_selected. Randomly sample 2,000 cells (rngState=0) and z-score them with scipy.stats.zscore(axis=0, ddof=1) for metric learning; the learned metric is 'correlation' with numClass=7 (num_comp=numClass-1). Instantiate CCSF(n_clusters=7, num_comp=6, metric='correlation') and call manifolder.cSpaRc(data=feed_data, locations=locations) to obtain reconstructed spatial expression maps. Extract a gene row (e.g. 'ftz' via np.argwhere(gene_names == gene)) and scatter-plot locations[:,0] vs locations[:,1] colored by reconstructed expression. Rationale: CCSF's cell-cell similarity low-dimensional representation removes the need for marker genes, which the article identifies as the key limitation of markerless novoSpaRc alone.

## Related tools

- **novoSpaRc** (Provides subsample_dge for aligning cells to geometry and is the spatial-mapping backbone executed on CCSF components via cSpaRc) — github.com/xinglab-ai/ccsf
- **ccsf** (Supplies the CCSF class (metric_learning, cSpaRc) that learns the correlation metric and cell-cell similarity representation feeding the spatial reconstruction) — github.com/xinglab-ai/ccsf

## Examples

```
manifolder = CCSF(n_clusters=7, num_comp=6, metric='correlation'); embedding_CSPARC = manifolder.cSpaRc(data={'data': dge}, locations=locations)
```

## Evaluation signals

- Saved ftz_spatial_reconstruction.png exists and shows the expected embryonic ftz stripe pattern
- Reconstruction correlates near-perfectly (~1) with the ground-truth FISH spatial map when ≥2 CCSF components are used; a single component should still yield correlation ≈0.91
- metric_learning on 2,000 z-scored sampled cells returns metric='correlation'
- locations array length equals dge rows after subsample_dge (3,039 cells), confirming cells and coordinates stayed aligned
- numClass=7 yields num_comp=6 CCSF components used for the reconstruction

## Limitations

- Perfect spatial reconstruction may not hold when space geometry is complex (such as tissue cross-section) and the acquired data contain inaccurate measurements of gene expression count
- With no contrast between data points (single-class dataset), CCSF works similarly to kernel PCA and its only benefits over PCA are data linearization and computational speed
- No accession numbers or external repository links are provided for the scRNA-seq datasets; sample data must be fetched from the ccsf Google Drive folder
- Python implementation version is not specified in the paper (only MATLAB 2019a is stated); README pins scikit-learn==1.1.1

## Evidence

- [results] CCSF-novoSpaRc achieved a perfect correlation with the ground-truth FISH spatial map when two or more CCSF components were used: [evidence span withheld — anchor status: not_found]
