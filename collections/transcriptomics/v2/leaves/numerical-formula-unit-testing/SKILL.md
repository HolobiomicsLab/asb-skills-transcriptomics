---
name: numerical-formula-unit-testing
description: Use when a paper describes mathematical transformations (e.g. sigmoid
  filters, convex weight optimization, norm-based matrix fusion) that you must re-implement
  in code and you need unit tests to confirm boundary and ordering properties before
  applying them to real data.
when_to_use_negative:
- The formulas are already provided by a tested library (e.g. Seurat 4's WNN integration)
  and no re-implementation is needed
- The dataset is massive (e.g. >100K cells with many ADT features) — LinQ-View's quadratic
  pairwise-distance computation fails on such data, so validating its formulas offers
  no benefit over Seurat 4
- The goal is statistical validation on real data rather than correctness of formula
  implementation on synthetic inputs
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2229
tools:
- name: R
  role: Implementation and unit-testing environment for the vectorized sigmoid filter,
    alpha optimization via optimize(), and L∞ max-fusion of distance matrices
  repo: https://www.r-project.org/
- name: LinQ-View
  role: The CITE-seq toolkit whose distance-fusion formulas (sigmoid filter, linear
    rescaling, L∞ norm) are being implemented and tested
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: RStudio v1.2.1335
  role: IDE used for developing and running the R validation scripts
  repo: https://www.rstudio.com/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:8d78bbf3c58098f5a7a04bb0b36ba6f6c30870c34dac7c59556ffdb75e81a044
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LinQ-View
  - R
  - RStudio v1.2.1335
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# numerical-formula-unit-testing

## Summary

A verification skill for validating that newly implemented numerical formulas (here, LinQ-View's sigmoid ADT-distance filter, alpha-weighted linear rescaling, and L∞ norm distance fusion) behave correctly on small synthetic inputs before they are trusted on real CITE-seq data.

## When to use

Use when a paper describes mathematical transformations (e.g. sigmoid filters, convex weight optimization, norm-based matrix fusion) that you must re-implement in code and you need unit tests to confirm boundary and ordering properties before applying them to real data.

## When NOT to use

- The formulas are already provided by a tested library (e.g. Seurat 4's WNN integration) and no re-implementation is needed
- The dataset is massive (e.g. >100K cells with many ADT features) — LinQ-View's quadratic pairwise-distance computation fails on such data, so validating its formulas offers no benefit over Seurat 4
- The goal is statistical validation on real data rather than correctness of formula implementation on synthetic inputs

## Inputs

- Small synthetic symmetric pairwise distance matrices D_RNA and D_ADT (e.g. R matrices)
- Sigmoid filter parameters (n=10, k=0.5 from the LinQ-View paper)
- Scalar objective function for α optimization over the closed interval [0,1]

## Outputs

- Self-contained R script (e.g. linq_view_distance_modules.R) implementing the three formula components
- Passing unit-test assertions on synthetic matrices (sigmoid midpoint value, α boundary reduction, D_joint ≥ rescaled matrices elementwise)
- A validated joint distance matrix generator usable on real CITE-seq modality distances

## How to apply

Implement each formula as a self-contained, vectorized function — for LinQ-View, the sigmoid filter F(x)=x/(1+e^{-n(x-k)}) with n=10 and k=0.5 applied elementwise to the ADT Euclidean cell-cell distance matrix, the linear rescaling D_RNA'=αD_RNA and D_ADT'=(1−α)D_ADT with α solved by univariate convex minimization over [0,1] (e.g. R's optimize() on a scalar objective such as fused-distance dispersion), and the elementwise-maximum fusion D_joint=max(D_RNA', D_ADT'). Then write unit tests on small synthetic symmetric distance matrices: check the sigmoid's midpoint behavior at x=k=0.5, check the α=0 and α=1 boundary cases reduce to pure-ADT and pure-RNA distances, and assert D_joint is elementwise ≥ both rescaled matrices. Pass criteria are that the script runs without errors, the source file contains the three formula components, and all assertions pass.

## Related tools

- **R** (Implementation and unit-testing environment for the vectorized sigmoid filter, alpha optimization via optimize(), and L∞ max-fusion of distance matrices) — https://www.r-project.org/
- **LinQ-View** (The CITE-seq toolkit whose distance-fusion formulas (sigmoid filter, linear rescaling, L∞ norm) are being implemented and tested) — https://github.com/WilsonImmunologyLab/LinQView
- **RStudio v1.2.1335** (IDE used for developing and running the R validation scripts) — https://www.rstudio.com/

## Examples

```
Rscript test_linq_view_distance_modules.R  # runs unit tests: sigmoid midpoint at k=0.5, alpha boundary cases (α=0,1), and D_joint >= elementwise max check on synthetic matrices
```

## Evaluation signals

- script_runs: the R script executes without errors
- contains_substring: linq_view_distance_modules.R contains the sigmoid formula (n=10, k=0.5), the α-weighting terms, and the max-fusion expression
- Sigmoid unit test: F(k)=F(0.5) returns the expected midpoint behavior on a synthetic ADT distance matrix
- Boundary test: setting α=0 and α=1 recovers pure-ADT and pure-RNA rescaled distances respectively
- Ordering invariant: D_joint is elementwise ≥ both D_RNA' and D_ADT' on synthetic symmetric matrices

## Limitations

- LinQ-View's pairwise distance computation scales quadratically with cell number, limiting time efficiency on large datasets (e.g. it failed on 170K cells with 228 ADT features)
- LinQ-View is best suited for large datasets with a limited number of ADT features (<50)
- Tests on synthetic matrices verify formula correctness but not biological validity on real CITE-seq data

## Evidence

- [discussion] we introduced a sigmoid filter into the distance calculation of the ADT modality: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [intro] LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [discussion] calculating cell-cell pair-wise distances for each modality, joint distance calculation, cell population clustering, non-linear dimension reduction and visualization: "calculating cell-cell pair-wise distances for each modality, joint distance calculation, cell population clustering, non-linear dimension reduction and visualization"
- [readme] The primary idea behind LinQ-View for CITE-seq is to calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression) and then combine the dissimilarities from each modality.: "The primary idea behind LinQ-View for CITE-seq is to calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression) and then combine"
- [discussion] LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50): "LinQ-View is the best option on large datasets with a limited number of ADT features (e.g., <50)"
