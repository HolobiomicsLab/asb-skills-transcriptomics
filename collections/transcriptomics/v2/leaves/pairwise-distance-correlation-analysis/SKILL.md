---
name: pairwise-distance-correlation-analysis
description: Use when a pseudo-space has been reconstructed from scRNA-seq data using
  an ST reference (e.g., 10X Visium DLPFC slice 151674) and the geometric fidelity
  of the reconstruction must be quantified before downstream interpretation.
when_to_use_negative:
- The target tissue has low spatial coherence (e.g., Patients 5 and 9 of SCC in the
  paper), where scSpace's spatial reconstruction and thus distance correlations are
  unreliable
- The spatial reference architecture does not match the target tissue (e.g., using
  DLPFC as reference for agranular dACC, which lacks cortical layer 4) without prior
  appropriateness assessment
- The data already carry measured spatial coordinates, making pseudo-space reconstruction
  and its distance-correlation validation unnecessary
tools:
- name: scSpace
  role: Produces the pseudo-space coordinates (TCA shared features + MLP projection)
    that are validated by the distance correlation
  repo: https://github.com/ZJUFanLab/scSpace
- name: scipy
  role: Computes pairwise-distance matrices and the correlation statistics between
    pseudo-space and reference distances
- name: pandas
  role: Loads reference/target matrices and stores coordinate/metric results (dlpfc_pseudospace_results.csv)
- name: matplotlib
  role: Visualizes reconstructed pseudo-space, layer ordering, and marker-gene patterns
    for the validation panel
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38121-4
    title: Reconstruction of the cell pseudo-space from single-cell RNA sequencing
      data with scSpace
  source_content_hash: sha256:8cb1e16b9ede4dd8107555dd3c836936a0696577cacb2ae04210d99cdd7dddaf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUFanLab/scSpace
  tools:
  - Python
  - Visium
  - matplotlib
  - pandas
  - scSpace
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pairwise-distance-correlation-analysis

## Summary

Quantitative validation of scSpace pseudo-space reconstructions by computing pairwise-distance correlations between reconstructed pseudo-space positions and the spatial transcriptomics reference positions, alongside ordering and marker-gene checks (e.g., human DLPFC cortical layers).

## When to use

Use when a pseudo-space has been reconstructed from scRNA-seq data using an ST reference (e.g., 10X Visium DLPFC slice 151674) and the geometric fidelity of the reconstruction must be quantified before downstream interpretation.

## When NOT to use

- The target tissue has low spatial coherence (e.g., Patients 5 and 9 of SCC in the paper), where scSpace's spatial reconstruction and thus distance correlations are unreliable
- The spatial reference architecture does not match the target tissue (e.g., using DLPFC as reference for agranular dACC, which lacks cortical layer 4) without prior appropriateness assessment
- The data already carry measured spatial coordinates, making pseudo-space reconstruction and its distance-correlation validation unnecessary

## Inputs

- Pseudo-space coordinate matrix (cells x 2D coordinates) from the scSpace MLP projection
- Reference spatial transcriptomics spot coordinates (e.g., 10X Visium DLPFC slice 151674 spatial positions)
- Preprocessed/normalized expression matrices for reference and target (pandas .csv inputs as used in the DLPFC vignette)
- Layer annotations and layer-specific marker-gene expression values (scipy/pandas)

## Outputs

- Pairwise-distance correlation coefficient and significance between pseudo-space and reference distances
- Coordinate/metric table saved as dlpfc_pseudospace_results.csv
- Validation panel figure (figure2_ad_validation.png) showing reconstructed pseudo-space, layer ordering, and marker-gene patterns

## How to apply

After scSpace projects target cells into pseudo-space coordinates via its transfer component analysis (TCA) shared feature extraction and multilayer perceptron (MLP) model, compute pairwise-distance matrices for both the pseudo-space coordinates and the reference spatial transcriptomics positions using scipy, then correlate the two distance vectors (e.g., Pearson correlation with scipy.stats / pandas). A significant positive correlation indicates the reconstruction preserves relative positions and the neighbor system of spots. Complement the correlation with a check that the expected cortical layer hierarchy (DLPFC layers) is ordered correctly in pseudo-space and that layer-specific marker-gene expression patterns match the reference. Summarize coordinates and metrics in a table (e.g., dlpfc_pseudospace_results.csv) and visualize the reconstructed layers with matplotlib.

## Related tools

- **scSpace** (Produces the pseudo-space coordinates (TCA shared features + MLP projection) that are validated by the distance correlation) — https://github.com/ZJUFanLab/scSpace
- **scipy** (Computes pairwise-distance matrices and the correlation statistics between pseudo-space and reference distances)
- **pandas** (Loads reference/target matrices and stores coordinate/metric results (dlpfc_pseudospace_results.csv))
- **matplotlib** (Visualizes reconstructed pseudo-space, layer ordering, and marker-gene patterns for the validation panel)

## Examples

```
from scipy.stats import pearsonr; from scipy.spatial.distance import pdist; r, p = pearsonr(pdist(ref_coords), pdist(pseudo_coords))
```

## Evaluation signals

- Pairwise-distance correlation between pseudo-space and reference spatial positions is significantly positive
- Reconstructed pseudo-space reproduces the expected cortical layer ordering and the neighbor system of spots
- Layer-specific marker-gene expression in pseudo-space matches patterns reported in Fig. 2a-d of the paper
- Relative positions between layers are preserved when compared against the Visium reference (slice 151674)
- Output files dlpfc_pseudospace_results.csv and figure2_ad_validation.png are produced with expected schema (coordinates + metrics)

## Limitations

- The choice of spatial reference directly influences the spatial reconstruction; appropriateness assessment or pre-processing of the reference is required (e.g., dACC is agranular and lacks cortical layer 4)
- Intrinsic differences in spatial homogeneity between tissues may influence the accuracy of scSpace and hence distance-correlation validation
- scSpace should be applied cautiously to tissues with low spatial coherence, such as SCC Patients 5 and 9, where distance-based validation may be misleading

## Evidence

- [abstract] scSpace shows promising performance on revealing the pairwise cellular spatial association within single-cell data.: "scSpace shows promising performance on revealing the pairwise cellular spatial association within single-cell data."
- [methods] project single cells into a pseudo space via an MLP model: "project single cells into a pseudo space via a [multiple layer perceptron (MLP)](https://en.wikipedia.org/wiki/Multilayer_perceptron) model"
- [discussion] choice of spatial reference influences the reconstruction: "Since the choice of spatial reference would directly influence the spatial reconstruction"
- [readme] scSpace workflow combining TCA and MLP: "project single cells into a pseudo space via a [multiple layer perceptron (MLP)](https://en.wikipedia.org/wiki/Multilayer_perceptron) model, so that gene expression and spatial weight of cells can be"
