---
name: nk-cell-surface-marker-phenotyping
description: Use when CITE-seq data shows protein (ADT) evidence of cell sub-populations,
  e.g. CD16/CD56 NK markers, that is hidden in the matching RNA counts by dropout,
  and you need to quantify whether denoising recovers it.
when_to_use_negative:
- The data has no paired antibody-derived tag (protein) measurements, so no protein-derived
  ground-truth sub-population labels can be fitted with mclust
- The population is not one where surface-marker protein levels indicate sub-structure
  that RNA dropout obscures (e.g. single-marker, no known CD16/CD56-like dichotomy)
- The goal is general clustering of the whole dataset rather than validating a denoising
  step against protein-derived labels in a small marker-gene space
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DCA (deep count autoencoder network)
  role: Produces the denoised NCAM1/FCGR3A RNA matrix (replacing counts with predicted
    NB means) used as the third silhouette space
  repo: https://github.com/theislab/dca
- name: mclust
  role: Fits the two-component Gaussian mixture model on CD16/CD56 ADT protein levels
    to generate the reference sub-population labels
- name: cluster
  role: Provides the silhouette() function used to compute average silhouette widths
    of protein labels in each expression space
- name: Scanpy
  role: Hosts the dca method for producing the denoised expression used in the silhouette
    comparison
  repo: https://github.com/theislab/dca
- name: anndata
  role: Container for the h5ad-formatted RNA/ADT count matrix used as DCA input
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:1ab5d89a6ea1d53b6ac5b6d0bf3a9e79ce4be644a124e9ded10514cf5abcfa97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA (deep count autoencoder network)
  - Scanpy
  - anndata
  - cluster
  - mclust
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# NK-cell surface marker phenotyping

## Summary

Quantifies whether DCA-denoised RNA recovers a sub-population structure visible in CITE-seq surface-protein (ADT) levels but obscured by dropout in raw counts, using mclust-derived protein labels and silhouette widths in CD16/CD56 space.

## When to use

Use when CITE-seq data shows protein (ADT) evidence of cell sub-populations, e.g. CD16/CD56 NK markers, that is hidden in the matching RNA counts by dropout, and you need to quantify whether denoising recovers it.

## When NOT to use

- The data has no paired antibody-derived tag (protein) measurements, so no protein-derived ground-truth sub-population labels can be fitted with mclust
- The population is not one where surface-marker protein levels indicate sub-structure that RNA dropout obscures (e.g. single-marker, no known CD16/CD56-like dichotomy)
- The goal is general clustering of the whole dataset rather than validating a denoising step against protein-derived labels in a small marker-gene space

## Inputs

- Stoeckius et al. CITE-seq cord blood mononuclear cell dataset (GSE100866, 8,005 human cells) with RNA count matrix and ADT protein matrix
- Annotated NK-cell population subset (906 cells)
- CD16 (FCGR3A) and CD56 (NCAM1) antibody-derived tag protein levels
- Original RNA counts for NCAM1/FCGR3A
- DCA-denoised RNA matrix (predicted negative binomial means) for NCAM1/FCGR3A

## Outputs

- Protein-derived sub-population labels (CD56bright/CD56dim) from the mclust two-component GMM
- Three average silhouette widths (protein ~0.47, original RNA ~0.17, DCA-denoised RNA ~0.58)
- silhouette_scores.csv comparison table

## How to apply

Load the Stoeckius et al. CITE-seq cord blood dataset (GSE100866, subset to 8,005 human cells) and extract the annotated NK-cell population (906 cells). Fit a two-component Gaussian mixture model with the R mclust package on the CD16 (FCGR3A) and CD56 (NCAM1) antibody-derived tag protein levels to obtain protein-derived cluster labels, since protein space is trusted over sparse RNA. Compute average silhouette widths of these labels in three spaces with cluster::silhouette: protein expression (~0.47 expected), original NCAM1/FCGR3A RNA (~0.17), and DCA-denoised NCAM1/FCGR3A RNA (~0.58), where the denoised matrix comes from the deep count autoencoder (ZINB/NB loss). Assemble the three values into a comparison table and write silhouette_scores.csv. The rationale is that DCA's negative-binomial reconstruction replaces dropout counts with predicted NB means, restoring RNA-protein concordance.

## Related tools

- **DCA (deep count autoencoder network)** (Produces the denoised NCAM1/FCGR3A RNA matrix (replacing counts with predicted NB means) used as the third silhouette space) — https://github.com/theislab/dca
- **mclust** (Fits the two-component Gaussian mixture model on CD16/CD56 ADT protein levels to generate the reference sub-population labels)
- **cluster** (Provides the silhouette() function used to compute average silhouette widths of protein labels in each expression space)
- **Scanpy** (Hosts the dca method for producing the denoised expression used in the silhouette comparison) — https://github.com/theislab/dca
- **anndata** (Container for the h5ad-formatted RNA/ADT count matrix used as DCA input) — https://github.com/theislab/anndata

## Examples

```
library(mclust); fit <- Mclust(cbind(FCGR3A_ADT, NCAM1_ADT), G = 2); sil <- cluster::silhouette(fit$classification, dist(expression_matrix))
```

## Evaluation signals

- Ordering check: average silhouette width for DCA-denoised RNA (~0.58) must exceed protein space (~0.47), which must exceed original RNA (~0.17)
- Absolute-value check: DCA-denoised silhouette ≈ 0.58, protein ≈ 0.47, original RNA ≈ 0.17 against the paper's reported values
- Cell-count check: exactly 906 annotated NK cells are extracted from the 8,005-cell human subset
- Silhouette is computed in each space against the same protein-derived mclust labels (two components), so the only varying factor is the expression space
- Output file silhouette_scores.csv contains all three silhouette values in a comparison table

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data; evaluation is hard because ground truth for real data is challenging to define
- Overimputation can occur with inadequate hyperparameter choices, e.g. a too low-dimensional bottleneck layer and data manifold
- Hyperparameters were not fine-tuned for any of the manuscript analyses, so reported silhouette values may not transfer to retuned DCA runs

## Evidence

- [results] Protein vs RNA vs denoised silhouette ordering: [evidence span withheld — anchor status: not_found]
- [discussion] Denoised matrix definition: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [discussion] CITE-seq dataset subset definition: [evidence span withheld — anchor status: not_found]
- [discussion] Human-cell filtering threshold: "removing cells with less than 90% human UMI counts"
- [intro] Scanpy API for DCA: [evidence span withheld — anchor status: not_found]
- [results] mclust two-component GMM on ADT: [evidence span withheld — anchor status: not_found]
