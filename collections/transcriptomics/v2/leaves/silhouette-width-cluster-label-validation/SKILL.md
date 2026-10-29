---
name: silhouette-width-cluster-label-validation
description: Use when you have external cluster labels (e.g., from a different modality
  or a GMM fit) and need to quantify how well-separated those clusters are in an expression
  matrix, such as testing whether denoising recovers structure lost to dropout.
when_to_use_negative:
- You have no pre-defined cluster labels and are still discovering clusters — silhouette
  here only scores externally supplied labels, it does not determine them.
- The denoised output may suffer overimputation (e.g., inadequate hyperparameters
  such as too low-dimensional a bottleneck), which can inflate silhouette widths by
  artificially sharpening structure rather than recovering true biology.
- The features underlying the labels are absent from the expression matrix you are
  scoring against (e.g., NCAM1/FCGR3A not measured).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DCA (deep count autoencoder network)
  role: Produces the denoised RNA matrix whose silhouette width against protein-derived
    labels is compared with the original RNA and protein spaces
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Python API hosting the 'dca' method in the preprocessing package; Scanpy preprocessing
    (e.g., filter_genes_dispersion) used for gene filtering steps around the analysis
- name: mclust
  role: R package used to fit the two-component Gaussian mixture model on FCGR3A/NCAM1
    protein levels to derive the cluster labels being validated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:916dbf1f1abc05608d0a979636f2ce11c9c0b2de774cb02cbc2267a6b2e41e82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA (deep count autoencoder network)
  - Scanpy
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# silhouette-width-cluster-label-validation

## Summary

Validate that cluster labels (e.g., protein- or RNA-derived cell types) are consistent with a given expression space by computing average silhouette widths, as in the DCA paper where CD56bright/CD56dim NK sub-population labels were scored against protein, original RNA, and DCA-denoised RNA expression of NCAM1/FCGR3A.

## When to use

Use when you have external cluster labels (e.g., from a different modality or a GMM fit) and need to quantify how well-separated those clusters are in an expression matrix, such as testing whether denoising recovers structure lost to dropout.

## When NOT to use

- You have no pre-defined cluster labels and are still discovering clusters — silhouette here only scores externally supplied labels, it does not determine them.
- The denoised output may suffer overimputation (e.g., inadequate hyperparameters such as too low-dimensional a bottleneck), which can inflate silhouette widths by artificially sharpening structure rather than recovering true biology.
- The features underlying the labels are absent from the expression matrix you are scoring against (e.g., NCAM1/FCGR3A not measured).

## Inputs

- Cell-by-feature expression matrix per modality (protein ADT levels, raw scRNA-seq counts, DCA-denoised RNA matrix)
- Cluster labels per cell (e.g., mclust two-component GMM labels from FCGR3A/NCAM1 protein levels)
- Subset of cells of interest (e.g., 906 annotated NK cells from the 8,005-cell CITE-seq cord blood dataset, GSE100866)

## Outputs

- Average silhouette width per (label set, expression space) pair
- Comparison table of silhouette values (e.g., silhouette_scores.csv) with expected ordering: denoised RNA > protein > original RNA

## How to apply

Obtain a set of cluster labels for the cells — here, fit a two-component Gaussian mixture model on the CD16 (FCGR3A) and CD56 (NCAM1) antibody-derived tag protein levels with the R mclust package to label 906 NK cells as CD56bright/CD56dim. Then, for each candidate expression space (protein space, original RNA counts, denoised RNA), compute the average silhouette width of the fixed labels against that space using standard silhouette computation (e.g., cluster::silhouette in R) restricted to the relevant features (NCAM1/FCGR3A). Compare the average widths across spaces: a higher width means the labels are better separated in that space. In the DCA paper this yielded protein 0.47, original RNA 0.17, and DCA-denoised RNA 0.58 — accept the denoising result if the denoised width exceeds both the original RNA and the protein-derived values. Write the three values into a comparison table (e.g., silhouette_scores.csv).

## Related tools

- **DCA (deep count autoencoder network)** (Produces the denoised RNA matrix whose silhouette width against protein-derived labels is compared with the original RNA and protein spaces) — https://github.com/theislab/dca
- **Scanpy** (Python API hosting the 'dca' method in the preprocessing package; Scanpy preprocessing (e.g., filter_genes_dispersion) used for gene filtering steps around the analysis)
- **mclust** (R package used to fit the two-component Gaussian mixture model on FCGR3A/NCAM1 protein levels to derive the cluster labels being validated)

## Examples

```
library(cluster); sil <- silhouette(labels, dist(expr[, c('NCAM1','FCGR3A')])); mean(sil[, 'sil_width'])
```

## Evaluation signals

- Average silhouette width of protein-derived labels against protein space should be ~0.47.
- Average silhouette width against original NCAM1/FCGR3A RNA should be ~0.17.
- Average silhouette width against DCA-denoised RNA should be ~0.58, exceeding both the original RNA (~0.17) and protein (~0.47) values.
- Ordering check: silhouette_width(denoised RNA) > silhouette_width(protein) > silhouette_width(original RNA).
- Comparison table (silhouette_scores.csv) contains exactly the three modality-specific silhouette values for the same fixed label set and same 906-cell NK subset.

## Limitations

- Evaluation of denoising is difficult because defining a ground truth for real data is challenging; silhouette width against external labels is one proxy, not a definitive measure.
- It may be difficult to determine in general when denoising improves scRNA-seq data.
- Overimputation can occur with inadequate hyperparameter choices (e.g., too low-dimensional bottleneck layer), which could distort silhouette-based comparisons; hyperparameters were not fine-tuned for any of the manuscript's analyses.

## Evidence

- [results] DCA-denoised RNA shows higher agreement with the protein-derived CD56bright/CD56dim NK sub-population labels (average silhouette width 0.58) than the original RNA data (0.17), and even exceeds the protein data itself (0.47).: [evidence span withheld — anchor status: not_found]
- [methods] Fit a two-component Gaussian mixture model on the CD16 (FCGR3A) and CD56 (NCAM1) antibody-derived tag protein levels using the R mclust package: [evidence span withheld — anchor status: not_found]
- [discussion] The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [intro] Scanpy users can directly use the "dca" method in the preprocessing package: [evidence span withheld — anchor status: not_found]
- [discussion] data were subset to 8,005 human cells by removing cells with less than 90% human UMI counts: "data were subset to 8,005 human cells by removing cells with less than 90% human UMI counts"
