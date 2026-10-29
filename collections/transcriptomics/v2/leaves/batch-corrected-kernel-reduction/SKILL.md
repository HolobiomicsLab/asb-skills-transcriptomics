---
name: batch-corrected-kernel-reduction
description: Use when merging multiple scRNA-seq count matrices from independent studies
  and you need a shared low-dimensional cell embedding in which batch/study-of-origin
  effects are removed before clustering or label propagation.
when_to_use_negative:
- Input is a single homogeneous dataset with no batch/study structure — plain reduce_kernel
  without correction is sufficient
- Data have not been filtered and normalized (filter_adata with min_cells_per_feature=0.01,
  min_features_per_cell=1000, then normalize_total and log1p must precede the reduction)
- The goal requires preserving batch-specific technical signals rather than removing
  them (e.g., QC auditing of per-study effects)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: ACTIONet
  role: Provides the randomized-SVD reduce_kernel reduction step with Harmony-style
    batch correction that generates the corrected kernel for network construction
  repo: https://github.com/shmohammadi86/ACTIONet
- name: ACTIONet (R package)
  role: R interface to the reduction and downstream ACTIONet pipeline (install from
    R-release branch via devtools)
  repo: https://github.com/shmohammadi86/ACTIONet/tree/R-release
- name: R
  role: Environment in which the batch-corrected reduction and downstream cortex integration
    are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:ec2d868af5b6d4d46332878428cef4482496db0996b45dbfde9a6a5ae81f18f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (R package)
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# batch-corrected-kernel-reduction

## Summary

Low-rank cell-space reduction (randomized SVD) performed with Harmony-style batch correction so that downstream ACTIONet archetypal analysis and network construction group cells by cell type rather than by study of origin. This is the integration entry point used to merge the Velmeshev, Mathys, and Schirmer human prefrontal cortex datasets (105,099 cells).

## When to use

Use when merging multiple scRNA-seq count matrices from independent studies and you need a shared low-dimensional cell embedding in which batch/study-of-origin effects are removed before clustering or label propagation.

## When NOT to use

- Input is a single homogeneous dataset with no batch/study structure — plain reduce_kernel without correction is sufficient
- Data have not been filtered and normalized (filter_adata with min_cells_per_feature=0.01, min_features_per_cell=1000, then normalize_total and log1p must precede the reduction)
- The goal requires preserving batch-specific technical signals rather than removing them (e.g., QC auditing of per-study effects)

## Inputs

- Merged gene-by-cell count matrix from multiple independent scRNA-seq studies (e.g., 105,099 human prefrontal cortex cells from three studies)
- Per-dataset raw 10x count matrices (e.g., .h5 files read via read_10x_h5)
- Batch/study-of-origin label vector for each cell

## Outputs

- Batch-corrected low-rank cell embedding (randomized-SVD reduction) usable as the ACTIONet input kernel
- Joint ACTIONet network where clusters mix cells from all studies
- 2D UMAP SGD embedding colored by cell type and by batch for validation

## How to apply

Merge the independent count matrices into a single matrix (e.g., Velmeshev 52,556 + Mathys 35,140 + Schirmer 17,403 control cells = 105,099 cells), preprocess each dataset with ACTIONet's filter_adata (min_cells_per_feature=0.01, min_features_per_cell=1000), normalize_total library-size normalization, and log1p transformation. Then apply ACTIONet's randomized-SVD-based reduce_kernel step with Harmony-style batch correction over the merged data, which produces a low-rank cell-to-cell similarity kernel in which batch variation is regressed out. This corrected reduction feeds the adaptive k*-NN multiresolution ACTIONet network and subsequent UMAP SGD layout and Leiden clustering. The decision point: if the reduction is working, network groups should correspond to cell types (neuronal vs glial) rather than to the study of origin, which is verified in the 2D embedding before proceeding to label propagation and consensus signature extraction.

## Related tools

- **ACTIONet** (Provides the randomized-SVD reduce_kernel reduction step with Harmony-style batch correction that generates the corrected kernel for network construction) — https://github.com/shmohammadi86/ACTIONet
- **ACTIONet (R package)** (R interface to the reduction and downstream ACTIONet pipeline (install from R-release branch via devtools)) — https://github.com/shmohammadi86/ACTIONet/tree/R-release
- **R** (Environment in which the batch-corrected reduction and downstream cortex integration are executed)

## Examples

```
an.pp.reduce_kernel(adata)  # randomized-SVD reduction with Harmony-style batch correction on the merged 105,099-cell cortex matrix
```

## Evaluation signals

- In the joint 2D UMAP embedding, annotated groups mix cells from all three studies (Velmeshev, Mathys, Schirmer) rather than separating by batch (Fig. 5a check)
- Leiden clusters on the corrected network correspond to biological cell types — neuronal vs glial — not to study of origin
- Network-based label propagation transfers cell-type labels coherently across the three datasets (Figs. 5b–d)
- Each consolidated consensus signature (e.g., LINGO2, CUX2, SST for neuronal; SLC1A2, MBP, CLDN5 for glial) discriminates its assigned cell type/subtype in the joint embedding
- The reduction scales to ~105k cells, reflecting the randomized-SVD efficiency claim of the reduction step

## Limitations

- No software/tool versions are stated for ACTIONet, R, or the Harmony-kernel implementation (only CMake 3.19+ is documented), so reproduction requires picking a branch (R-release or python-devel)
- Source cortex datasets are cited only by reference numbers without accessions or download links
- Batch correction is baked into the reduction step; if mis-specified, residual batch structure can masquerade as subtypes in downstream Leiden clusters

## Evidence

- [discussion] Harmony-kernel reduction groups by cell type: [evidence span withheld — anchor status: not_found]
- [discussion] Randomized SVD reduction step: "an efficient randomized SVD-based low-rank approximation (reduction step) that enables scaling with input data size"
- [full_text] Merge of three cortex studies: "Data for the joint analysis of the human prefrontal cortex was obtained from three independent studies29–31 and includes in total 105,099 cells"
- [methods] filter_adata thresholds before reduction: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [methods] reduce_kernel invocation: "an.pp.reduce_kernel(adata)"
- [discussion] Multiresolution network construction from the reduced space: "an adaptive nearest-neighbor algorithm to build a multiresolution network with automatic neighbor size selection"
