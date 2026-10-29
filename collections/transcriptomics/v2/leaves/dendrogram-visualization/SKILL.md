---
name: dendrogram-visualization
description: Use when you have a symmetric pairwise distance matrix (e.g., scDist
  distances between cell types) and need to cluster it hierarchically and render a
  dendrogram to check whether distances recapitulate known biological relationships
  such as immune cell lineages.
when_to_use_negative:
- Distances were computed with a naive approach subject to individual-to-individual/cohort-to-cohort
  variability (e.g., without a mixed-effects model), since resulting dendrogram topology
  may reflect false-positive perturbations rather than true lineage structure.
- Cell-type clusters are over-clustered relative to condition-specific effects — scDist
  distances decay toward 0 with increasing cluster number, which would compress the
  dendrogram and mask lineage separation.
- The goal is differential state testing (which cell types change between conditions)
  rather than visualizing between-cell-type relationships; use scDist's DistPlot/p-values
  instead.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scDist
  role: Estimates the pairwise distances between cell types (mixed-effects model on
    PC scores) that form the input distance matrix for clustering
  repo: https://github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-principal-component linear mixed-effects models underlying scDist's
    distance estimates
- name: lmerTest
  role: Provides Satterthwaite's approximation for degrees of freedom used in scDist's
    uncertainty estimates
- name: ggplot2
  role: Renders the hierarchical-clustering dendrogram, saved as dendrogram.png
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:d4b35de0c3435d77790e341d97496fbac63392d19ca2afa15c5adae9107c555a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/phillipnicol/scDist
  tools:
  - ggplot2
  - lme4
  - lmerTest
  - scDist
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# dendrogram-visualization

## Summary

Apply hierarchical clustering (R stats::hclust, complete linkage) to a symmetric pairwise distance matrix — here the pairwise scDist distances between cell types from the COVID-19 dataset — and render the resulting dendrogram with ggplot2 so cell-type lineage structure (lymphoid vs myeloid grouping) can be visually validated.

## When to use

Use when you have a symmetric pairwise distance matrix (e.g., scDist distances between cell types) and need to cluster it hierarchically and render a dendrogram to check whether distances recapitulate known biological relationships such as immune cell lineages.

## When NOT to use

- Distances were computed with a naive approach subject to individual-to-individual/cohort-to-cohort variability (e.g., without a mixed-effects model), since resulting dendrogram topology may reflect false-positive perturbations rather than true lineage structure.
- Cell-type clusters are over-clustered relative to condition-specific effects — scDist distances decay toward 0 with increasing cluster number, which would compress the dendrogram and mask lineage separation.
- The goal is differential state testing (which cell types change between conditions) rather than visualizing between-cell-type relationships; use scDist's DistPlot/p-values instead.

## Inputs

- Symmetric pairwise distance matrix between cell types (e.g., pairwise_distances.csv of scDist-estimated distances)
- Cell-type annotations and sample-level metadata (condition, patient) from an annotated scRNA-seq dataset (e.g., COVID-19 data from www.covid19cellatlas.org)

## Outputs

- hclust hierarchical clustering object over cell types
- Dendrogram plot rendered with ggplot2 and saved as dendrogram.png

## How to apply

Assemble pairwise cell-type distances (e.g., scDist fitted per pair with fixed.effects = disease condition, random.effects = patient, clusters = cell-type annotation on PC scores) into a symmetric distance matrix such as pairwise_distances.csv. Pass the matrix to R stats::hclust using complete linkage to build the clustering, then render the dendrogram with ggplot2 and save it as dendrogram.png. Judge correctness by dendrogram topology: in the COVID-19 case it should reproduce the known lineage structure reported in Fig. 3D of the paper — lymphoid T and NK cells clustering together, B cells falling further apart, and myeloid DC, monocytes, and neutrophils grouping together. Note that the upstream distances depend on scDist's normalization (scTransform Pearson residuals recommended), choice of number of PCs, and avoidance of over-clustering, all of which propagate into the dendrogram.

## Related tools

- **scDist** (Estimates the pairwise distances between cell types (mixed-effects model on PC scores) that form the input distance matrix for clustering) — https://github.com/phillipnicol/scDist
- **lme4** (Fits the per-principal-component linear mixed-effects models underlying scDist's distance estimates)
- **lmerTest** (Provides Satterthwaite's approximation for degrees of freedom used in scDist's uncertainty estimates)
- **ggplot2** (Renders the hierarchical-clustering dendrogram, saved as dendrogram.png)

## Examples

```
hc <- hclust(as.dist(pairwise_distances), method = "complete")
```

## Evaluation signals

- Dendrogram topology check: T and NK (lymphoid) cells cluster together; B cells fall apart from the T/NK branch.
- Myeloid check: DC, monocytes, and neutrophils group together on a shared branch, matching Fig. 3D of the paper.
- Distance matrix invariant: pairwise_distances.csv is symmetric with zero diagonal.
- Pairwise scDist values are consistent with the results data frame (Dist., 95% CI low/upper, p.val) reported per cell type by scDist.
- Clustering uses a defined linkage (complete linkage via stats::hclust) and the output dendrogram.png is produced reproducibly from the saved matrix.

## Limitations

- Distances are sensitive to the choice of the number of principal components (fixed heuristic of 20 or 50; no adaptive selection), which can change dendrogram topology.
- Double dipping (using expression data for both annotation and distance estimation) can bias distances downward via over-clustering, compressing the dendrogram.
- scDist cannot be applied directly to raw counts; a sub-optimal normalization (anything other than scTransform Pearson residuals) could affect the distances being clustered.
- Violations of the spherical-normal assumption could lead to miscalibrated standard errors, and significant clustering/annotation errors cause unavoidable bias; a cluster-free scDist extension does not yet exist.

## Evidence

- [results] Hierarchical clustering of pairwise scDist distances recapitulates immune lineages: [evidence span withheld — anchor status: not_found]
- [methods] Exclusion filter and pairwise scDist fitting: [evidence span withheld — anchor status: not_found]
- [discussion] Recommended normalization upstream of clustering: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
- [discussion] Over-clustering failure mode affecting distances: "As the number of clusters increases, the median distance decays towards 0, which demonstrates that scDist can produce false negatives when the data is over-clustered"
- [readme] scDist invocation pattern from README: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
