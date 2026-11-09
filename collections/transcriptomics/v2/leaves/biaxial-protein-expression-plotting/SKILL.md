---
name: biaxial-protein-expression-plotting
description: Use when dsb-normalized protein values (and cell cluster labels) are
  available and you need cytometry-style biaxial plots to verify that clusters and
  UMAP reproduce expected immune cell population structure.
when_to_use_negative:
- Protein data have not been dsb-normalized (e.g., raw ADT UMIs or CLR-normalized
  values), since biaxial separation of populations depends on the background-denoised
  scaling
- The goal is RNA-based visualization of cell populations rather than validation of
  protein (ADT) marker separation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: dsb
  role: Produces the denoised, background-scaled protein values that serve as the
    axes of the biaxial plots
  repo: https://cran.r-project.org/package=dsb
- name: Seurat
  role: Hosts the dsb-normalized ADT data, cluster assignments, and UMAP embedding
    used to color/annotate the biaxial plots
  repo: https://github.com/satijalab/seurat-data
- name: R
  role: Implementation language for plotting the biaxial scatter from the normalized
    matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:25abc0dd06b13fba8fe96de3546292b86d7feee4533a9d02e7d40f2a6c8257fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dsb
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Biaxial protein expression plotting

## Summary

Generates flow-cytometry-style biaxial scatter plots (e.g., CD14 vs CD4) from dsb-normalized ADT values to visually validate that protein-based clusters recover the population structure shown in the paper's figures (Fig. 2a-h).

## When to use

Use when dsb-normalized protein values (and cell cluster labels) are available and you need cytometry-style biaxial plots to verify that clusters and UMAP reproduce expected immune cell population structure.

## When NOT to use

- Protein data have not been dsb-normalized (e.g., raw ADT UMIs or CLR-normalized values), since biaxial separation of populations depends on the background-denoised scaling
- The goal is RNA-based visualization of cell populations rather than validation of protein (ADT) marker separation

## Inputs

- dsb-normalized ADT expression matrix (DSBNormalizeProtein output)
- Cell cluster assignments from clustering on dsb-normalized protein values
- UMAP embedding coordinates (RunUMAP on dsb-normalized values)
- Protein/ADT marker names to plot (e.g., CD14, CD4)

## Outputs

- Biaxial scatter plot of CD14 vs CD4 on dsb-normalized values, colored by cluster
- Plots demonstrating agreement with the population structure in Fig. 2a-h

## How to apply

First run dsb normalization (DSBNormalizeProtein with default parameters: log(cell + 10) transformation, background mu_N/sigma_N subtraction from empty droplets, mclust k=2 Gaussian mixture per cell, PC1-based technical component removal via limma::removeBatchEffect). Then extract the dsb-normalized ADT matrix (e.g., into a Seurat object metadata or data frame) along with cluster assignments and plot pairs of marker proteins — in the reproduction this was CD14 vs CD4 — as biaxial scatter plots with cells colored by cluster. Rationale: biaxial plots are the standard cytometry visualization; on correctly normalized data, expected populations (e.g., CD14+ monocytes vs CD4+ T cells) should resolve into discrete gates analogous to flow plots. Judge correctness by whether the biaxial plots, together with the UMAP and clusters, reproduce the population structure shown in Fig. 2a-h of the paper.

## Related tools

- **dsb** (Produces the denoised, background-scaled protein values that serve as the axes of the biaxial plots) — https://cran.r-project.org/package=dsb
- **Seurat** (Hosts the dsb-normalized ADT data, cluster assignments, and UMAP embedding used to color/annotate the biaxial plots) — https://github.com/satijalab/seurat-data
- **R** (Implementation language for plotting the biaxial scatter from the normalized matrix)

## Examples

```
ggplot(pData, aes(x = CD14, y = CD4, color = cluster)) + geom_point()
```

## Evaluation signals

- CD14 vs CD4 biaxial plot shows discrete, interpretable population gates (e.g., CD14+ monocyte-like cells separated from CD4+ T cell-like cells) on dsb-normalized values
- Population structure in the biaxial plots is concordant with clusters and the UMAP embedding built on the same dsb-normalized values
- Biaxial plots visually reproduce the population structure shown in Fig. 2a-h of the dsb paper
- Axes reflect the dsb residual scale (ambient-background-corrected, denoised values, typically near-centered around 0 after technical component removal)

## Limitations

- Interpretability of biaxial plots depends on the prior dsb normalization steps, including careful definition of background droplets ('defining a reasonable subset of barcodes as background droplets still requires care')
- Biaxial plots alone do not quantify correctness; they must be judged jointly with cluster assignments and UMAP against the reference figure

## Evidence

- [other] Biaxial plot step in the reproduction workflow: [evidence span withheld — anchor status: not_found]
- [other] Validation criterion for the plots: [evidence span withheld — anchor status: not_found]
- [discussion] dsb normalization that precedes plotting: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
