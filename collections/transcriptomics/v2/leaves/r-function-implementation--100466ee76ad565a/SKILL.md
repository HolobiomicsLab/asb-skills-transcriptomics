---
name: r-function-implementation
description: Use when a paper defines explicit mathematical formulas (e.g., F(x)=x/(1+e^{-n(x-k)}),
  α-weighted distance scaling, elementwise max fusion) that must be translated into
  vectorized R functions with synthetic-matrix unit tests.
when_to_use_negative:
- Datasets are massive with many ADT features (e.g., 170K cells with 228 ADT features),
  where the article reports LinQ-View fails and Seurat 4 is recommended
- The input is raw expression data rather than per-modality pairwise distance matrices
  — pre-processing (QC, normalization, scaling, PCA) and per-modality distance calculation
  must happen first
- The task requires a different fusion model than the L∞ norm (elementwise maximum),
  e.g., learned joint embeddings from Seurat 4 or MOFA2
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3572
tools:
- name: LinQ-View
  role: Toolkit whose joint-distance modules (sigmoid filter, linear rescaling, L∞
    fusion) are being reimplemented as R functions
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: R v3.6.3
  role: Implementation language for the vectorized sigmoid, α-optimization via optimize(),
    and max-fusion functions
  repo: https://www.r-project.org/
- name: RStudio v1.2.1335
  role: Development environment used with R for the LinQ-View implementation
  repo: https://www.rstudio.com/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:72dbcc1449e023fa5dc5636afeebb5f5e3c1f1a847b4820a245d97c244624da0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - R
  - R v3.6.3
  - RStudio v1.2.1335
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# r-function-implementation

## Summary

Implement paper-specified computational modules as self-contained, unit-tested R functions — exemplified by LinQ-View's joint distance computation (sigmoid-filtered ADT distances, linear α-rescaling, L∞ norm fusion) in a single script (linq_view_distance_modules.R).

## When to use

Use when a paper defines explicit mathematical formulas (e.g., F(x)=x/(1+e^{-n(x-k)}), α-weighted distance scaling, elementwise max fusion) that must be translated into vectorized R functions with synthetic-matrix unit tests.

## When NOT to use

- Datasets are massive with many ADT features (e.g., 170K cells with 228 ADT features), where the article reports LinQ-View fails and Seurat 4 is recommended
- The input is raw expression data rather than per-modality pairwise distance matrices — pre-processing (QC, normalization, scaling, PCA) and per-modality distance calculation must happen first
- The task requires a different fusion model than the L∞ norm (elementwise maximum), e.g., learned joint embeddings from Seurat 4 or MOFA2

## Inputs

- Symmetric pairwise cell-cell distance matrix from the mRNA modality (DRNA)
- Symmetric pairwise cell-cell distance matrix from the ADT (surface protein) modality (DADT)
- Sigmoid filter parameters (n=10, k=0.5 as specified by LinQ-View)
- Scalar α ∈ [0,1] or a scalar objective for convex minimization over [0,1]

## Outputs

- Sigmoid-filtered ADT distance matrix
- Rescaled distance matrices DRNA' = α·DRNA and DADT' = (1-α)·DADT
- Joint distance matrix Djoint = elementwise max(DRNA', DADT') for downstream clustering and t-SNE/UMAP visualization
- Self-contained R script (e.g., linq_view_distance_modules.R) with passing unit tests

## How to apply

Translate each formula into a vectorized R function: (1) a sigmoid filter F(x)=x/(1+e^{-n(x-k)}) with the LinQ-View parameters n=10 and k=0.5, applied elementwise to the ADT Euclidean cell-cell distance matrix; (2) a linear scaling step DRNA'=α·DRNA, DADT'=(1-α)·DADT, solving for α by univariate convex minimization over [0,1] (e.g., R's optimize() on a scalar objective such as fused-distance dispersion); (3) an L∞ fusion step Djoint=max(DRNA', DADT') taken elementwise over the two rescaled matrices. Write self-contained unit tests on small synthetic symmetric DRNA and DADT matrices, checking that the sigmoid gives the k-midpoint behavior at x=k=0.5, that α=0 and α=1 boundary cases reduce to pure-ADT and pure-RNA distances, and that Djoint is elementwise ≥ both rescaled matrices. The rationale, grounded in the LinQ-View paper, is that fusing pairwise distance matrices with an L∞ norm integrates transcriptome and cell-surface protein expression information while preventing variation in a single surface protein from dominating the joint representation.

## Related tools

- **LinQ-View** (Toolkit whose joint-distance modules (sigmoid filter, linear rescaling, L∞ fusion) are being reimplemented as R functions) — https://github.com/WilsonImmunologyLab/LinQView
- **R v3.6.3** (Implementation language for the vectorized sigmoid, α-optimization via optimize(), and max-fusion functions) — https://www.r-project.org/
- **RStudio v1.2.1335** (Development environment used with R for the LinQ-View implementation) — https://www.rstudio.com/

## Examples

```
Rscript linq_view_distance_modules.R
```

## Evaluation signals

- The R script runs without errors (script_runs) and the file linq_view_distance_modules.R exists
- The script contains the three formula components as substrings: the sigmoid term, the α-weighting terms (α·DRNA, (1-α)·DADT), and the max-fusion term
- Unit test: sigmoid output at x=k=0.5 equals the 0.5·k-midpoint behavior with n=10, k=0.5
- Unit test: α=0 and α=1 boundary cases reduce exactly to pure-ADT and pure-RNA distance matrices
- Invariant: Djoint is elementwise ≥ both α·DRNA and (1-α)·DADT on synthetic symmetric matrices

## Limitations

- Pairwise distance computation scales quadratically with cell number, limiting time and memory efficiency as dataset size increases
- LinQ-View failed on a massive dataset of 170K cells with 228 ADT features, whereas Seurat 4 succeeded
- LinQ-View is best suited for large datasets with a limited number of ADT features (e.g., <50)

## Evidence

- [intro] LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [discussion] we introduced a sigmoid filter into the distance calculation of the ADT modality: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [discussion] we introduced an L∞ norm model to calculate the joint distance: "we introduced an L∞ norm model to calculate the joint distance"
- [discussion] calculating cell-cell pair-wise distances for each modality, joint distance calculation, cell population clustering, non-linear dimension reduction and visualization: "calculating cell-cell pair-wise distances for each modality, joint distance calculation, cell population clustering, non-linear dimension reduction and visualization"
- [readme] The primary idea behind LinQ-View for CITE-seq is to calculate the dissimilarity representations for each single cell profiling modality: "The primary idea behind LinQ-View for CITE-seq is to calculate the dissimilarity representations for each single cell profiling modality"
