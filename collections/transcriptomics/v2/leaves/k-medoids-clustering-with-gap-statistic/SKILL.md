---
name: k-medoids-clustering-with-gap-statistic
description: Use when comparing normalization strategies (e.g., dsb vs CLR) on protein-only
  ADT data via clustering coherence, or when selecting the number of protein-defined
  clusters in a PBMC CITE-seq matrix.
when_to_use_negative:
- Input is raw ADT UMI counts without prior normalization — clustering coherence would
  reflect noise rather than normalization quality (low-signal proteins with max UMI
  < 5 must first be removed and data dsb/CLR normalized).
- The goal is joint mRNA+protein clustering (WNN) — this skill deliberately clusters
  on protein expression data alone; multimodal WNN analysis combining 30 mRNA PCs
  is a different workflow.
- Datasets without empty-droplet background or isotype controls — dsb normalization
  prerequisites may be missing, invalidating the dsb-vs-CLR comparison.
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: dsb
  role: Provides DSBNormalizeProtein to generate the dsb-normalized protein matrix
    being compared; also supplies the PBMC test data (cells_citeseq_mtx, empty_drop_citeseq_mtx)
  repo: https://cran.r-project.org/package=dsb
- name: Seurat
  role: Provides NormalizeData with margin=2 for CLR baseline normalization of the
    ADT matrix
- name: mclust
  role: Gaussian mixture model engine used inside dsb's denoising step (fit per cell
    with k=2 mixture components) that produces the normalized values fed to clustering
- name: limma
  role: Its removeBatchEffect() function is used by dsb for robust matrix decomposition
    of the technical component prior to clustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:33e72aaa0a35b84309c0f25982058b13515a216a616f471618575ef2a8e00135
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# k-medoids-clustering-with-gap-statistic

## Summary

Applies k-medoids clustering to normalized ADT (protein) matrices from CITE-seq data for k = 1..20 and computes the Gap-Statistic — a measure of within-cluster coherence — to compare how well normalization methods (dsb vs CLR) unmask biologically meaningful cell populations.

## When to use

Use when comparing normalization strategies (e.g., dsb vs CLR) on protein-only ADT data via clustering coherence, or when selecting the number of protein-defined clusters in a PBMC CITE-seq matrix.

## When NOT to use

- Input is raw ADT UMI counts without prior normalization — clustering coherence would reflect noise rather than normalization quality (low-signal proteins with max UMI < 5 must first be removed and data dsb/CLR normalized).
- The goal is joint mRNA+protein clustering (WNN) — this skill deliberately clusters on protein expression data alone; multimodal WNN analysis combining 30 mRNA PCs is a different workflow.
- Datasets without empty-droplet background or isotype controls — dsb normalization prerequisites may be missing, invalidating the dsb-vs-CLR comparison.

## Inputs

- dsb-normalized ADT matrix (cells x proteins, e.g., from DSBNormalizeProtein with quantile.clipping)
- CLR-normalized ADT matrix (Seurat::NormalizeData, margin=2)
- cluster identity assignments per cell for cluster-level identity labeling

## Outputs

- gap_statistic_table.csv (one row per k = 1..20, columns for dsb and CLR Gap-Statistics)
- fold_change_comparison.csv (lineage-marker fold changes, e.g., CD56, CD8, CD4, per normalization method)
- k-medoids cluster assignments per cell for each k and each normalization method

## How to apply

First remove proteins with maximum raw UMI count < 5 across all cells, then produce two normalized protein matrices in parallel: dsb via DSBNormalizeProtein (denoise.counts=TRUE, isotype controls, quantile.clipping=TRUE with quantile.clip=c(0.001, 0.9995)) and CLR via Seurat::NormalizeData with margin=2. Run k-medoids clustering on each matrix independently for k = 1..20 using the normalized protein values alone (no mRNA PCs). For each k and method, compute the Gap-Statistic by comparing within-cluster dispersion against a null reference distribution sampled from the data. Higher Gap-Statistics indicate tighter within-cluster protein coherence; downstream, assign cluster-level cell identities and compute lineage-marker fold changes (e.g., CD56 for NK cells, CD8/CD4 for T cells) to confirm biological interpretability.

## Related tools

- **dsb** (Provides DSBNormalizeProtein to generate the dsb-normalized protein matrix being compared; also supplies the PBMC test data (cells_citeseq_mtx, empty_drop_citeseq_mtx)) — https://cran.r-project.org/package=dsb
- **Seurat** (Provides NormalizeData with margin=2 for CLR baseline normalization of the ADT matrix)
- **mclust** (Gaussian mixture model engine used inside dsb's denoising step (fit per cell with k=2 mixture components) that produces the normalized values fed to clustering)
- **limma** (Its removeBatchEffect() function is used by dsb for robust matrix decomposition of the technical component prior to clustering)

## Examples

```
dsb_rescaled <- DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE, use.isotype.control = TRUE, isotype.control.name.vec = rownames(cells_citeseq_mtx)[67:70], quantile.clipping = TRUE, quantile.clip = c(0.001, 0.9995)); for (k in 1:20) pam_fit <- cluster::pam(t(dsb_rescaled), k)
```

## Evaluation signals

- Gap-Statistics for dsb-normalized data should be consistently higher than for CLR across k = 1..20 (paper's Supplementary Fig. 3f,g finding).
- gap_statistic_table.csv must contain exactly 20 rows (one per k) with both dsb and CLR columns populated.
- Lineage-marker fold changes (CD56 for NK cells, CD8/CD4 for T cells) should be sharper under dsb than CLR for the corresponding assigned clusters.
- Proteins with maximum raw UMI count < 5 across all cells (e.g., CD34) must be excluded from the clustering input.
- Cluster assignments must come from protein-only matrices — verify no mRNA principal components leaked into the k-medoids input.

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; 'defining a reasonable subset of barcodes as background droplets still requires care'.
- Assessment details on robustness of noise estimation, empty-droplet definition sensitivity, and isotype caveats are in the Supplementary Note, not the main text; caveats for datasets without isotype controls are deferred there.
- The workflow's gap statistic was validated on the >50,000-cell 87-antibody PBMC CITE-seq dataset; software version numbers for the tools used are not stated in the paper.

## Evidence

- [abstract] Compared to existing normalization methods, dsb improves downstream analyses by better unmasking biologically meaningful cell populations: "Compared to existing normalization methods, our approach improves downstream analyses by better unmasking biologically meaningful cell populations"
- [discussion] low-signal protein filter before normalization/clustering: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization, resulting in removal of the CD34 protein"
- [discussion] dsb vs CLR protein data used for clustering comparisons: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [discussion] mclust GMM inside dsb producing normalized input values: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package to the transformed count of each cell from step 1 with k = 2 mixture components"
- [readme] dsb output is a standard matrix usable with clustering tools: [evidence span withheld — anchor status: external_doc]
