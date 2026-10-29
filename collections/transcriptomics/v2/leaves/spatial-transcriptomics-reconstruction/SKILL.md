---
name: spatial-transcriptomics-reconstruction
description: Use when reconstructing 2D spatial gene expression maps from single-cell
  expression data (e.g., mouse cerebellum slide-seq, 46,293 locations) without marker
  genes, and when benchmarking reconstruction accuracy vs number of CCSF components.
when_to_use_negative:
- 'Dataset contains data points from only one class (no contrast): CCSF works similarly
  to kernel PCA, with only linearization and speed benefits over PCA.'
- Space geometry is complex (e.g., tissue cross-section) or gene expression count
  measurements are inaccurate, so perfect spatial reconstruction cannot be expected.
- Input is already a dimensionality-reduced embedding rather than raw gene expression
  — CCSF is a PCA replacement applied to the expression matrix itself.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ccsf
  role: Provides the CCSF class whose metric_learning routine learns the 'correlation'
    cell-cell similarity metric and whose cSpaRc method performs the spatial mapping
    with a chosen number of CCSF components
  repo: github.com/xinglab-ai/ccsf
- name: novoSpaRc
  role: Optimal-transport spatial mapping engine wrapped by cSpaRc; also run directly
    as the markerless baseline (no marker genes) for comparison
- name: pandas
  role: Used to load tabular expression data and write the spatial_correlations.csv
    results table
- name: numpy
  role: Loads dge.txt/geometry.txt arrays, sets num_comp arrays, and supports the
    correlation computations
- name: scipy
  role: stats.zscore normalizes sampled and full expression data (axis=0, ddof=1)
    before metric learning and mapping
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2023.100840
    title: Leveraging cell-cell similarity for high-performance spatial and temporal
      cellular mappings from gene expression data
  source_content_hash: sha256:0284eb070ab6a14c888aa1aa7a4f5525dff4c13750303f5a724ff4fa71d00cb7
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
  - matplotlib
  - novoSpaRc
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# spatial-transcriptomics-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Markerless spatial reconstruction of gene expression from scRNA-seq coordinates using CCSF-novoSpaRc, where cell-cell similarity features replace PCA before optimal transport mapping. Reproduces the paper's mouse cerebellum slide-seq result that correlations rise with the number of CCSF components (0.90 at 4, 0.99 at 16, 1 at 32).

## When to use

Use when reconstructing 2D spatial gene expression maps from single-cell expression data (e.g., mouse cerebellum slide-seq, 46,293 locations) without marker genes, and when benchmarking reconstruction accuracy vs number of CCSF components.

## When NOT to use

- Dataset contains data points from only one class (no contrast): CCSF works similarly to kernel PCA, with only linearization and speed benefits over PCA.
- Space geometry is complex (e.g., tissue cross-section) or gene expression count measurements are inaccurate, so perfect spatial reconstruction cannot be expected.
- Input is already a dimensionality-reduced embedding rather than raw gene expression — CCSF is a PCA replacement applied to the expression matrix itself.

## Inputs

- Mouse cerebellum slide-seq expression matrix (sagittal sections, 46,293 locations, 19,782 genes)
- Spatial locations/geometry file (e.g., geometry.txt with x,y columns)
- Z-score normalized coarse-grained expression data (3,900 locations)
- Randomly sampled 2,000-point subset for CCSF metric learning

## Outputs

- Reconstructed spatial expression maps per gene from cSpaRc for 4, 16, and 32 CCSF components
- Markerless novoSpaRc baseline reconstruction for comparison
- spatial_correlations.csv with columns for component count, method, and Pearson correlation
- Learned metric name (expected: 'correlation') and number of data classes from metric_learning

## How to apply

Load the mouse cerebellum slide-seq dataset (46,293 sagittal locations, 19,782 genes; 7,704 cells analyzed) and coarse-grain it to 3,900 locations as in the paper. Z-score normalize the expression matrix (scipy.stats.zscore, axis=0, ddof=1) and, since the dataset exceeds 2,000 points, learn the cell-cell similarity metric on 2,000 randomly sampled points via manifolder.metric_learning; expect the learned metric to be 'correlation'. Run CCSF-novoSpaRc via manifolder.cSpaRc(data=feed_data, locations=locations) three times with metric='correlation' and num_comp set to 4, 16, and 32 CCSF components. Run the markerless novoSpaRc baseline (no marker genes) on the same coarse-grained data for comparison. Compute the Pearson correlation between each reconstructed spatial map and the slide-seq ground-truth locations, and write component count, method, and correlation to spatial_correlations.csv.

## Related tools

- **ccsf** (Provides the CCSF class whose metric_learning routine learns the 'correlation' cell-cell similarity metric and whose cSpaRc method performs the spatial mapping with a chosen number of CCSF components) — github.com/xinglab-ai/ccsf
- **novoSpaRc** (Optimal-transport spatial mapping engine wrapped by cSpaRc; also run directly as the markerless baseline (no marker genes) for comparison)
- **pandas** (Used to load tabular expression data and write the spatial_correlations.csv results table)
- **numpy** (Loads dge.txt/geometry.txt arrays, sets num_comp arrays, and supports the correlation computations)
- **scipy** (stats.zscore normalizes sampled and full expression data (axis=0, ddof=1) before metric learning and mapping)

## Examples

```
manifolder = CCSF(n_clusters=numClass,num_comp=4,metric='correlation'); embedding_CSPARC = manifolder.cSpaRc(data=feed_data,locations=locations)
```

## Evaluation signals

- metric_learning on the z-scored 2,000-point sample returns 'correlation' as the learned metric
- Pearson correlation of CCSF-novoSpaRc reconstruction vs slide-seq ground truth is approximately 0.90 with 4 CCSF components, 0.99 with 16, and 1 with 32
- The markerless novoSpaRc baseline (no marker genes) shows lower accuracy than CCSF-novoSpaRc on the same coarse-grained data
- Data dimensions match the paper's setup: 46,293 locations coarse-grained to 3,900, with 7,704 cells analyzed
- spatial_correlations.csv contains one row per (component count, method) with valid numeric correlation values

## Limitations

- With no contrast between data points (single-class dataset), CCSF works similarly to kernel PCA and its only benefits over PCA are data linearization and computational speed.
- Perfect spatial reconstruction may not hold when space geometry is complex (such as a tissue cross-section) or when gene expression count measurements are inaccurate.
- Reproducibility gaps: no accession numbers or repository links are provided for the cerebellum scRNA-seq data, the Python implementation version is unspecified, and the Leiden clustering resolution is ambiguously reported as '138'.

## Evidence
