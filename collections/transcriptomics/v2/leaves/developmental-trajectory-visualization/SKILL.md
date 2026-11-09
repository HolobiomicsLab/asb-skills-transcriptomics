---
name: developmental-trajectory-visualization
description: Use when you have a dropout-affected scRNA-seq matrix from a differentiating
  or developing cell population (e.g., zebrafish embryos, embryoid bodies) and need
  to recover pseudotime ordering and state transitions that imputation may distort.
when_to_use_negative:
- 'The input is small genomic data: SERM learns its distribution from a small fraction
  of the data, so the estimated distribution may diverge from the actual distribution
  and yield inaccurate imputation that would mislead trajectory inference'
- The dataset contains strong batch effects requiring explicit correction — batch
  effects are not considered explicitly in the SERM formulation (only z-score integration
  across batches is offered)
- The biological question is clustering/taxonomy rather than continuous trajectory
  recovery — the article's clustering benchmarks (k-means accuracy, NMI, AR, Hubert)
  cover that case instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Imputes the dropout-affected matrix (adaptive histogram equalization, ROI
    = half matrix dimensions, 100 bins, bilinear resampling) before trajectory inference
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: TSCAN
  role: Pseudotime inference applied to reference, observed, and imputed matrices
    with the authors' default settings
- name: Slingshot
  role: Pseudotime inference applied to reference, observed, and imputed matrices
    with the authors' default settings
- name: PHATE
  role: 2D embedding (2 dims, 5 NN, alpha=40, metric MDS, Euclidean, gamma=0.5; 100
    PCs zebrafish / 50 PCs EB) to visualize the developmental trajectory
- name: MAGIC
  role: Competing imputation baseline run with default configurations to demonstrate
    manifold distortion
  repo: https://github.com/pkathail/magic
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:f068548f3dfc79ed1c979980c543c1522c3beeeb253e627b7d9189a9f1427124
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - MAGIC
  - PHATE
  - Slingshot
  - TSCAN
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# developmental-trajectory-visualization

## Summary

Recover and visualize continuous developmental trajectories from dropout-affected scRNA-seq data by imputing with SERM, then running TSCAN/Slingshot pseudotime inference and PHATE 2D embedding, and validating against ground-truth stage labels (e.g., 12 zebrafish developmental stages) via Pearson correlation.

## When to use

Use when you have a dropout-affected scRNA-seq matrix from a differentiating or developing cell population (e.g., zebrafish embryos, embryoid bodies) and need to recover pseudotime ordering and state transitions that imputation may distort.

## When NOT to use

- The input is small genomic data: SERM learns its distribution from a small fraction of the data, so the estimated distribution may diverge from the actual distribution and yield inaccurate imputation that would mislead trajectory inference
- The dataset contains strong batch effects requiring explicit correction — batch effects are not considered explicitly in the SERM formulation (only z-score integration across batches is offered)
- The biological question is clustering/taxonomy rather than continuous trajectory recovery — the article's clustering benchmarks (k-means accuracy, NMI, AR, Hubert) cover that case instead

## Inputs

- Reference (dropout-free) scRNA-seq expression matrix (cells x genes), e.g., zebrafish development data (38,731 cells, 694 embryos, 12 stages) or human ES-to-embryoid-body differentiation data
- Dropout-affected observed expression matrix simulated at a given sampling efficiency (e.g., 10% efficiency via rgamma/Poisson downsampling following Huang et al.)
- Ground-truth stage labels for each cell (e.g., zebrafish developmental stage annotations)

## Outputs

- Pseudotime orderings per cell from TSCAN and Slingshot for reference, observed, and SERM-imputed matrices
- PHATE 2D embeddings of reference, observed, and imputed data visualizing the continuous developmental trajectory
- Pearson correlations between pseudotime (monocle) and ground-truth stage labels, and reference-to-imputed Pearson coefficients, used to judge trajectory fidelity

## How to apply

First impute the observed (dropout-affected) expression matrix with SERM using adaptive histogram equalization (ROI length/width = half the matrix dimensions, sliding step = window size, 100 histogram bins, bilinear resampling between ROIs). Next, apply TSCAN and Slingshot pseudotime inference to the reference, observed, and SERM-imputed matrices with default settings. Visualize reference, observed, and imputed data with PHATE using 2 embedding dimensions, 5 nearest neighbors, alpha = 40, metric MDS, Euclidean distance, gamma = 0.5, and 100 principal components for the zebrafish data or 50 PCs for the EB data. The rationale is that SERM reproduces the continuous developmental manifold of the reference, whereas competing methods such as MAGIC visibly distort it; validation compares pseudotime against ground-truth stage labels by Pearson correlation (monocle) across sampling efficiencies.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Imputes the dropout-affected matrix (adaptive histogram equalization, ROI = half matrix dimensions, 100 bins, bilinear resampling) before trajectory inference) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **TSCAN** (Pseudotime inference applied to reference, observed, and imputed matrices with the authors' default settings)
- **Slingshot** (Pseudotime inference applied to reference, observed, and imputed matrices with the authors' default settings)
- **PHATE** (2D embedding (2 dims, 5 NN, alpha=40, metric MDS, Euclidean, gamma=0.5; 100 PCs zebrafish / 50 PCs EB) to visualize the developmental trajectory)
- **MAGIC** (Competing imputation baseline run with default configurations to demonstrate manifold distortion) — https://github.com/pkathail/magic

## Examples

```
pip install serm; out = serm.serm(dataObs, ROIsize=[N_cells//2, N_genes//2], percOL=0.5, randomize=False)
```

## Evaluation signals

- SERM-imputed data reproduces the continuous developmental trajectory of the reference (Fig. 5 pattern) without the visible manifold distortion caused by competing methods
- Pearson correlation between monocle pseudotime and ground-truth stage labels is consistently higher for SERM than for all competing imputation methods across sampling efficiencies
- SERM achieves the highest reference-to-imputed Pearson coefficients for both zebrafish and EB differentiation datasets
- PHATE embedding of SERM-imputed data shows a continuous state-transition manifold matching the reference, whereas MAGIC-degraded data show distorted geometry

## Limitations

- SERM learns the gene expression model from a small fraction of the data, posing a challenge for small genomic datasets where the estimated distribution may diverge from the actual data distribution
- Reference evaluation data were created by choosing only cells and genes with high expression, and dropouts were simulated rather than observed experimentally
- SERM performance assumes ROI coverage is sufficiently large that the captured ROI distribution is representative of the entire expression matrix

## Evidence

- [methods] PHATE visualization parameters: [evidence span withheld — anchor status: not_found]
- [results] acceptance criterion: [evidence span withheld — anchor status: not_found]
- [methods] MAGIC download: "MAGIC was downloaded from https://github.com/pkathail/magic."
- [results] dataset scale: "profiling 38,731 cells from 694 embryos across 12 stages of early zebrafish development"
- [intro] ROIs and consistency: "In the last step, all the ROIs are interpolated using bilinear resampling to achieve the global consistency of the expression values"
- [discussion] ROI representative assumption: "An implicit assumption in our formulation is that the coverage of ROI is sufficiently large so that the captured data distribution in the ROI is representative of the entire matrix"
