---
name: pairwise-distance-matrix-construction
description: Use when you have a normalized scRNA-seq matrix with cell-type, condition,
  and patient metadata and need a cell-type-by-cell-type distance matrix (e.g., to
  cluster cell types by transcriptomic perturbation magnitude or lineage similarity).
when_to_use_negative:
- You only need differential abundance (proportion changes) between conditions — use
  DA-seq or Milo instead, as scDist targets differential state, not abundance
- Input is raw (unnormalized) counts — scDist cannot currently be applied directly
  to raw counts; normalize first (scTransform)
- Cell types are heavily over-clustered — the article shows over-clustering drives
  median distance toward 0, producing false negatives and downward-biased distances
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: Estimates the distance between condition means in gene-expression space for
    each pair of cell types via a linear mixed-effects model on PC scores
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-principal-component linear mixed-effects model underlying the
    scDist distance estimate
- name: lmerTest
  role: Provides Satterthwaite's approximation for degrees of freedom used in scDist's
    inference
- name: scTransform
  role: Recommended normalization (Pearson residuals) producing the normalized expression
    matrix input to scDist
- name: ggplot2
  role: Renders the dendrogram from hierarchical clustering of the pairwise distance
    matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:2d94700e80a86c974452d3def4b406faf3ffcc695b836d9fe9435b81596d0cae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ggplot2
  - lme4
  - lmerTest
  - scDist
  - scTransform
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pairwise-distance-matrix-construction

## Summary

Construct a symmetric matrix of pairwise between-cell-type distances by running scDist's linear mixed-effects model on PC scores for every pair of annotated cell types in a scRNA-seq dataset, then use the matrix for downstream analyses such as hierarchical clustering that recapitulates immune cell lineage relationships.

## When to use

Use when you have a normalized scRNA-seq matrix with cell-type, condition, and patient metadata and need a cell-type-by-cell-type distance matrix (e.g., to cluster cell types by transcriptomic perturbation magnitude or lineage similarity).

## When NOT to use

- You only need differential abundance (proportion changes) between conditions — use DA-seq or Milo instead, as scDist targets differential state, not abundance
- Input is raw (unnormalized) counts — scDist cannot currently be applied directly to raw counts; normalize first (scTransform)
- Cell types are heavily over-clustered — the article shows over-clustering drives median distance toward 0, producing false negatives and downward-biased distances

## Inputs

- Normalized scRNA-seq expression matrix (scTransform Pearson residuals recommended), e.g., genes x cells count matrix
- Cell-level metadata data frame with columns for condition (fixed effect), patient (random effect), and cell-type cluster annotation
- Vector or factor of annotated cell-type labels (cell types present in >10% of samples)

## Outputs

- Symmetric pairwise cell-type distance matrix (e.g., pairwise_distances.csv)
- Hierarchical clustering dendrogram of cell types (e.g., dendrogram.png)
- Per-pair scDist results (estimated distance, 95% CI, p-value) for each cell-type pair

## How to apply

Start from an annotated, normalized scRNA-seq matrix (scTransform Pearson residuals recommended for scDist) with per-cell metadata for condition, patient, and cluster label, excluding any cell type not present in more than 10% of samples. For every pair of annotated cell types, call scDist with fixed.effects = disease condition, random.effects = patient, and clusters = cell-type annotation; scDist uses PCA loadings as a plug-in estimator for U and fits per-PC mixed models with lme4 (Satterthwaite degrees of freedom via lmerTest), returning an estimated distance between condition means in gene-expression space. Assemble all pairwise distances into a symmetric matrix (e.g., pairwise_distances.csv), treating each cell type in turn as one 'condition' group. Apply hierarchical clustering (e.g., stats::hclust with complete linkage) to the matrix and render the dendrogram (e.g., ggplot2). Validate that the topology recovers known biology — lymphoid T/NK cells together, B cells apart, myeloid DC/monocytes/neutrophils grouped, matching Fig. 3D of the paper.

## Related tools

- **scDist** (Estimates the distance between condition means in gene-expression space for each pair of cell types via a linear mixed-effects model on PC scores) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-principal-component linear mixed-effects model underlying the scDist distance estimate)
- **lmerTest** (Provides Satterthwaite's approximation for degrees of freedom used in scDist's inference)
- **scTransform** (Recommended normalization (Pearson residuals) producing the normalized expression matrix input to scDist)
- **ggplot2** (Renders the dendrogram from hierarchical clustering of the pairwise distance matrix)

## Examples

```
out <- scDist(Y, meta.data, fixed.effects="condition", random.effects="patient", clusters="celltype"); D[i,j] <- out$results$Dist.
```

## Evaluation signals

- The distance matrix is square, symmetric, and has a defined entry for every pair of retained cell types
- The dendrogram topology reproduces the reported structure: T and NK (lymphoid) cells cluster together, B cells fall apart, and DC/monocytes/neutrophils (myeloid) cells group together, matching Fig. 3D
- All cell types included are present in more than 10% of samples (the paper's exclusion filter)
- Per-pair scDist output includes estimated distance with 95% CI and p-value (e.g., out$results columns 'Dist.', '95% CI (low)', '95% CI (upper)', 'p.val')
- Expression matrix is normalized (scTransform Pearson residuals) before model fitting, not raw counts

## Limitations

- Results are sensitive to the choice of the number of principal components K used to estimate the distance; no adaptive selection procedure exists (fixed heuristic of 20 or 50)
- Condition-specific effects combined with double-dipping of expression data for annotation and distance estimation can cause over-clustering and downward bias in estimated distances
- Violations of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests
- Performance may degrade if the input is normalized by a sub-optimal approach other than Pearson residuals; raw counts cannot be used directly
- Significant clustering/annotation errors cause unavoidable bias; a cluster-free extension of scDist does not yet exist

## Evidence

- [methods] scDist call on pairwise cell types: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
- [discussion] Estimate via PC plug-in and lme4: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Recommended normalization: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
- [discussion] Cell-type filter threshold: "Subsequently, we excluded any cell types not present in more than 10% of the samples from further analysis."
- [readme] Results data frame contents: "The results data frame gives a summary of the estimated distance and uncertainty for each cell type"
- [discussion] Sensitivity to number of PCs: "Another point of sensitivity for scDist is the choice of the number of principal components used to estimate the distance."
