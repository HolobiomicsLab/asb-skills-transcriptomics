---
name: gaussian-mixture-clustering-of-surface-protein-markers
description: Use when CITE-seq data provides surface-protein ADT levels for markers
  (e.g., CD16/CD56) whose RNA is dropout-obscured, and you need protein-derived sub-population
  labels to quantify or benchmark RNA-based (or denoised) recovery of that structure.
when_to_use_negative:
- Only scRNA-seq counts are available with no surface-protein ADT measurements — the
  mixture model needs protein levels because RNA alone is dropout-obscured (silhouette
  drops to ~0.17 on original RNA)
- The marker protein distributions are not compatible with a two-component mixture
  (e.g., continuous gradients rather than CD56bright/CD56dim bimodality) — mclust's
  G = 2 assumption would be unjustified
- The goal is denoising itself rather than protein-guided labeling or denoising benchmarking
  — apply DCA directly instead of this evaluation workflow
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: mclust
  role: R package used to fit the two-component Gaussian mixture model on CD16/CD56
    ADT protein levels to obtain protein-derived NK sub-population labels
- name: cluster
  role: R package providing the silhouette computation (cluster::silhouette) used
    to score the protein labels against protein, original RNA, and denoised RNA spaces
- name: DCA
  role: Deep count autoencoder producing the denoised NCAM1/FCGR3A RNA matrix (negative
    binomial means) whose silhouette width (~0.58) is compared against the protein
    labels
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Python API exposing DCA via the 'dca' method in the preprocessing package
    for generating the denoised RNA input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:87fb506943b266ee58816357a59c7f85f7d8e962122eb326a6372a2c1fc5a54e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Scanpy
  - cluster
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gaussian-mixture-clustering-of-surface-protein-markers

## Summary

Fits a two-component Gaussian mixture model on CITE-seq antibody-derived tag (ADT) levels of CD16/FCGR3A and CD56/NCAM1 to define CD56bright/CD56dim NK sub-population labels, then benchmarks those labels via average silhouette widths against protein, original RNA, and DCA-denoised RNA expression.

## When to use

Use when CITE-seq data provides surface-protein ADT levels for markers (e.g., CD16/CD56) whose RNA is dropout-obscured, and you need protein-derived sub-population labels to quantify or benchmark RNA-based (or denoised) recovery of that structure.

## When NOT to use

- Only scRNA-seq counts are available with no surface-protein ADT measurements — the mixture model needs protein levels because RNA alone is dropout-obscured (silhouette drops to ~0.17 on original RNA)
- The marker protein distributions are not compatible with a two-component mixture (e.g., continuous gradients rather than CD56bright/CD56dim bimodality) — mclust's G = 2 assumption would be unjustified
- The goal is denoising itself rather than protein-guided labeling or denoising benchmarking — apply DCA directly instead of this evaluation workflow

## Inputs

- CITE-seq ADT protein count matrix (CD16/FCGR3A and CD56/NCAM1 antibody-derived tags) for the 906-cell annotated NK population from the Stoeckius et al. cord blood dataset (GSE100866, 8,005 human cells)
- Original RNA count matrix restricted to NCAM1 and FCGR3A for the same 906 NK cells
- DCA-denoised NCAM1/FCGR3A RNA expression matrix (predicted negative binomial means from the deep count autoencoder)

## Outputs

- Two-component (CD56bright/CD56dim) protein-derived cluster labels for the 906 NK cells
- Three average silhouette widths (protein space ~0.47, original RNA ~0.17, DCA-denoised RNA ~0.58)
- silhouette_scores.csv comparison table

## How to apply

Load the Stoeckius et al. CITE-seq cord blood mononuclear cell dataset (GSE100866, subset to 8,005 human cells by the <90% human UMI filter) and extract the annotated NK cell population of 906 cells. Fit a two-component Gaussian mixture model with the R mclust package (G = 2) on the CD16 (FCGR3A) and CD56 (NCAM1) ADT protein levels to obtain protein-derived cluster labels. Compute average silhouette widths with cluster::silhouette for these labels against three expression spaces: the protein ADT space (expected ~0.47), the original NCAM1/FCGR3A RNA counts (expected ~0.17, showing dropout obscures the structure), and the DCA-denoised RNA matrix, whose values are the predicted negative binomial means (expected ~0.58). Assemble the three silhouette values into a comparison table written to silhouette_scores.csv. Accept the result only if the ordering DCA-denoised RNA > protein > original RNA is reproduced.

## Related tools

- **mclust** (R package used to fit the two-component Gaussian mixture model on CD16/CD56 ADT protein levels to obtain protein-derived NK sub-population labels)
- **cluster** (R package providing the silhouette computation (cluster::silhouette) used to score the protein labels against protein, original RNA, and denoised RNA spaces)
- **DCA** (Deep count autoencoder producing the denoised NCAM1/FCGR3A RNA matrix (negative binomial means) whose silhouette width (~0.58) is compared against the protein labels) — https://github.com/theislab/dca
- **Scanpy** (Python API exposing DCA via the 'dca' method in the preprocessing package for generating the denoised RNA input)

## Examples

```
library(mclust); fit <- Mclust(nk_adt[, c("FCGR3A", "NCAM1")], G = 2); mean(cluster::silhouette(fit$classification, dist(nk_adt[, c("FCGR3A", "NCAM1")]))[, 3])
```

## Evaluation signals

- mclust on the 906-cell NK subset yields exactly two clusters interpretable as CD56bright and CD56dim NK cells
- Average silhouette width of protein labels in the protein expression space is ~0.47
- Average silhouette width of the same labels against original NCAM1/FCGR3A RNA is ~0.17, well below protein space
- Average silhouette width against DCA-denoised RNA is ~0.58, exceeding both protein (~0.47) and original RNA (~0.17)
- Accept only if the ordering denoised RNA > protein > original RNA holds in silhouette_scores.csv

## Limitations

- Silhouette values (~0.47 / ~0.17 / ~0.58) are specific to this dataset and NK-cell subset; different preprocessing or cell filters may shift them
- Denoising evaluation lacks ground truth for real data, so agreement with protein labels is only a proxy; inadequate DCA hyperparameters (e.g., too low-dimensional bottleneck) can cause overimputation that inflates silhouette scores
- The two-component mixture is an assumption; if marker protein distributions are not bimodal, mclust labels will not reflect true sub-populations

## Evidence

- [methods] Fit a two-component Gaussian mixture model on the CD16 (FCGR3A) and CD56 (NCAM1) antibody-derived tag protein levels using the R mclust package to obtain protein-derived cluster labels.: [evidence span withheld — anchor status: not_found]
- [methods] Compute the average silhouette width of the protein labels against the DCA-denoised NCAM1/FCGR3A RNA expression (expected ~0.58): [evidence span withheld — anchor status: not_found]
- [discussion] The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [discussion] data were subset to 8,005 human cells by removing cells with less than 90% human UMI counts.: "data were subset to 8,005 human cells by removing cells with less than 90% human UMI counts."
