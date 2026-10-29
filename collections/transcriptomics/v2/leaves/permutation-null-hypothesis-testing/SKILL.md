---
name: permutation-null-hypothesis-testing
description: Use when you must demonstrate that an observed cell-to-cell similarity/distance
  metric (e.g., ACTIONet JSD similarity) exceeds random expectation, by comparing
  it against a label-permuted null and a competing baseline metric (e.g., Euclidean
  distance on HVG+PCA coordinates).
when_to_use_negative:
- The input lacks cell-type labels or a marker panel that can assign per-cell labels,
  since the permutation null requires shuffling labels repeatedly.
- The data has not been reduced into a cell-similarity space (no archetypal/kernel
  embedding or PCA coordinates), because the test compares metric values rather than
  raw counts.
- The goal is a parametric significance test (e.g., t-test or ANOVA on metric values)
  rather than an empirical label-shuffled null of deviation from random expectation.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ACTIONet
  role: Python package providing filter_adata, reduce_kernel, run_ACTIONet, JSD-based
    cell distances in the archetypal space, and marker-based cell annotation (load_markers,
    annotate_cells_using_markers)
  repo: https://github.com/shmohammadi86/ACTIONet
- name: scanpy
  role: Reads the 10x h5 matrix (sc.read_10x_h5), makes variable names unique, and
    applies normalize_total and log1p preprocessing
- name: Python
  role: Execution environment for the permutation workflow and plotting of Fig. 4e
    deviation distributions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:d34b0a6b4bbf2a531eec0a58c4e3e4ac3913ba67b38426369e221e878c27f0c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - Python
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# permutation-null-hypothesis-testing

## Summary

Builds label-permutation null models to test whether ACTIONet's Jensen-Shannon-divergence (JSD)-based cell similarity separates same-type cells more strongly than random expectation — and more strongly than a Seurat-style HVG + PCA + Euclidean-distance baseline — as reported in Fig. 4e of the ACTIONet paper.

## When to use

Use when you must demonstrate that an observed cell-to-cell similarity/distance metric (e.g., ACTIONet JSD similarity) exceeds random expectation, by comparing it against a label-permuted null and a competing baseline metric (e.g., Euclidean distance on HVG+PCA coordinates).

## When NOT to use

- The input lacks cell-type labels or a marker panel that can assign per-cell labels, since the permutation null requires shuffling labels repeatedly.
- The data has not been reduced into a cell-similarity space (no archetypal/kernel embedding or PCA coordinates), because the test compares metric values rather than raw counts.
- The goal is a parametric significance test (e.g., t-test or ANOVA on metric values) rather than an empirical label-shuffled null of deviation from random expectation.

## Inputs

- 10x Genomics PBMC count matrix (pbmc_10k_v3.h5, read via scanpy)
- Cell-type marker panel (PBMC_Monaco2019_12celltypes) loaded through an.tl.load_markers
- JSD-based cell similarity/distance matrix from the ACTIONet archetypal space
- Seurat-style HVG + PCA Euclidean distance matrix (baseline)

## Outputs

- Per-cell-type observed-vs-null deviation distributions for both metrics
- Side-by-side comparison plot saved as fig4e_deviation_distributions.png (Fig. 4e analogue)
- Conclusion that ACTIONet's JSD-based similarity shows systematically larger positive deviation from the permutation null than the Euclidean baseline

## How to apply

Load the 10x PBMC matrix with `sc.read_10x_h5('pbmc_10k_v3.h5')`, make variable names unique with join='.', then filter with `an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)` and apply `sc.pp.normalize_total` followed by `sc.pp.log1p`. Run `an.pp.reduce_kernel` (randomized SVD low-rank reduction) and `an.run_ACTIONet` to build the archetypal metric cell space, then annotate per-cell type labels using the PBMC_Monaco2019_12celltypes panel loaded with `an.tl.load_markers` and assigned via `an.tl.annotate_cells_using_markers`. Compute intra-cell-type similarity using ACTIONet's JSD-based cell distances in the archetypal space and, in parallel, baseline intra-cell-type Euclidean distances on Seurat-style HVG+PCA coordinates. Permute cell-type labels repeatedly to build null distributions per method and per cell type, then compute the observed-vs-null deviation from random expectation. Plot the deviation distributions for ACTIONet JSD similarity versus the Euclidean baseline side by side as in Fig. 4e and save to fig4e_deviation_distributions.png.

## Related tools

- **ACTIONet** (Python package providing filter_adata, reduce_kernel, run_ACTIONet, JSD-based cell distances in the archetypal space, and marker-based cell annotation (load_markers, annotate_cells_using_markers)) — https://github.com/shmohammadi86/ACTIONet
- **scanpy** (Reads the 10x h5 matrix (sc.read_10x_h5), makes variable names unique, and applies normalize_total and log1p preprocessing)
- **Python** (Execution environment for the permutation workflow and plotting of Fig. 4e deviation distributions)

## Examples

```
adata = sc.read_10x_h5('pbmc_10k_v3.h5'); adata.var_names_make_unique(join='.'); an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)
```

## Evaluation signals

- ACTIONet's JSD-based similarity distribution shows systematically larger positive deviation from the permutation null than the HVG+PCA Euclidean baseline, qualitatively matching the published Fig. 4e pattern.
- Per-cell-type deviation is computed as observed metric value versus the label-permuted null; deviation for same-type cells should be positive and far from zero.
- Both metrics are evaluated over identical cell-type labels and identical permutation repetitions, so the comparison is matched.
- Filtering and preprocessing parameters match the published methods: min_cells_per_feature=0.01, min_features_per_cell=1000, sc.pp.normalize_total then sc.pp.log1p.
- Output figure fig4e_deviation_distributions.png displays the two deviation distributions side by side.

## Limitations

- The finding is qualitative ('a deviation remarkably larger than Seurat's HVG + PCA + Euclidean distance measure') rather than attached to a stated p-value or effect-size threshold.
- No software/tool versions are stated for ACTIONet, R, Python, or the permutation procedure, so exact reproducibility of the null distributions is not guaranteed.
- Validation depends on the PBMC_Monaco2019_12celltypes marker panel quality; annotation errors in per-cell labels propagate into both the observed and null distributions.

## Evidence

- [results] Using permutation tests, ACTIONet's JSD-based cell similarity scores are significantly higher for cells of the same type than random expectation, with a deviation remarkably larger than Seurat's HVG + PCA + Euclidean distance measure.: [evidence span withheld — anchor status: not_found]
- [methods] an.pp.filter_adata parameters and preprocessing chain: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [methods] Marker panel loading for cell-type annotation: "markers, directions, names = an.tl.load_markers('PBMC_Monaco2019_12celltypes')"
- [methods] 10x data loading from the paper's methods: "adata = sc.read_10x_h5('pbmc_10k_v3.h5')"
- [abstract] Paper's overall performance claim supporting the superiority framing: "Using multiple synthetic and real data sets, we demonstrate ACTIONet's superior performance relative to existing alternatives"
