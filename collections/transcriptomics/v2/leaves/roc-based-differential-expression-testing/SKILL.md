---
name: roc-based-differential-expression-testing
description: 'Use when a Seurat WNN joint mRNA-ADT clustering needs biological validation:
  after identifying a protein-defined cluster (e.g. CD14+ CD86-high monocytes) and
  you must confirm its identity via differentially expressed transcripts such as inflammatory
  and activation genes.'
when_to_use_negative:
- Input is not a clustered single-cell object (e.g. bulk RNA-seq or already-summarized
  feature counts) — ROC testing on cluster identities is not defined
- The goal is quantifying ambient antibody noise itself rather than validating cluster
  identity — use the CLR-vs-98th-percentile empty-droplet correlation instead
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Provides FindMultiModalNeighbors/FindClusters for WNN clustering and the ROC-test
    differential expression (test.use='roc') on cluster identities
  repo: https://github.com/satijalab/seurat-data
- name: dsb
  role: Supplies the denoised ADT values (DSBNormalizeProtein) whose WNN clusters
    are annotated and tested for differential transcript expression
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: Language in which the whole workflow (normalization, WNN clustering, ROC-test
    DE) is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:a2edd9410a68fcfd93ef46acab3c1385728f85b6f9b81653ab53a99b55390efa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat-data
  tools:
  - R
  - Seurat
  - dsb
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# roc-based-differential-expression-testing

## Summary

Differential expression testing between a Seurat WNN-derived cell cluster (e.g. CD14+ CD86-high monocyte cluster 3) and all other clusters using the ROC-test (test.use = 'roc'), which ranks genes by the power of each transcript to subsume the cluster identity, to validate cell-type annotations in CITE-seq mRNA-protein normalization comparisons.

## When to use

Use when a Seurat WNN joint mRNA-ADT clustering needs biological validation: after identifying a protein-defined cluster (e.g. CD14+ CD86-high monocytes) and you must confirm its identity via differentially expressed transcripts such as inflammatory and activation genes.

## When NOT to use

- Input is not a clustered single-cell object (e.g. bulk RNA-seq or already-summarized feature counts) — ROC testing on cluster identities is not defined
- The goal is quantifying ambient antibody noise itself rather than validating cluster identity — use the CLR-vs-98th-percentile empty-droplet correlation instead

## Inputs

- Seurat object with WNN neighbors/clusters computed from 30 mRNA PCs plus dsb- or CLR-normalized ADT values
- Cluster identity assignments from FindClusters (algorithm = 3) with marker-protein annotation (CD14, CD86)

## Outputs

- DE gene table for the target cluster (ROC-test power/AUC ranking of transcripts, e.g. inflammatory and activation genes)
- Normalized per-protein median-value tables (dsb and CLR) and the CLR-vs-empty-droplet correlation statistic, written to output files

## How to apply

After clustering PBMC CITE-seq cells with Seurat FindMultiModalNeighbors (30 mRNA principal components combined with dsb- or CLR-normalized ADT values) and FindClusters (algorithm = 3), identify the target cluster by marker protein expression (cluster 3 = CD14+ monocytes with high CD86 under dsb). Run ROC-test differential expression comparing that cluster's identities against all remaining cells, producing per-gene power/AUC statistics. Inspect the resulting gene table for known biology — inflammatory and activation genes are expected to be enriched in the CD86-high monocyte cluster, confirming the annotation. Write the DE gene table to output alongside the per-protein median dsb/CLR value tables and the CLR-vs-empty-droplet correlation statistic (R = 0.67, p = 3.1e-10).

## Related tools

- **Seurat** (Provides FindMultiModalNeighbors/FindClusters for WNN clustering and the ROC-test differential expression (test.use='roc') on cluster identities) — https://github.com/satijalab/seurat-data
- **dsb** (Supplies the denoised ADT values (DSBNormalizeProtein) whose WNN clusters are annotated and tested for differential transcript expression) — https://cran.r-project.org/package=dsb
- **R** (Language in which the whole workflow (normalization, WNN clustering, ROC-test DE) is executed)

## Examples

```
markers_cluster3 <- FindMarkers(seu, ident.1 = '3', test.use = 'roc')
```

## Evaluation signals

- DE gene table for cluster 3 contains inflammatory and activation genes consistent with the CD86-high monocyte phenotype
- Per-protein median CLR values for cluster 3 correlate with the 98th percentile of empty-droplet expression (R = 0.67, p = 3.1e-10), while dsb medians do not — the DE annotation is on the correct cluster only if this benchmark reproduces
- All output files (median-value tables, correlation statistic, DE table) are present and consistent with the same Seurat object used for WNN clustering
- Cluster 3 under dsb is unambiguously CD14+ with high CD86 median normalized protein values

## Limitations

- Defining background droplets for dsb normalization (which defines the clusters being DE-tested) requires care; dsb values are sensitive to how background droplets are defined
- Software version numbers for the tools are not stated, so exact ROC-test results may not be bit-reproducible
- Caveats for datasets without isotype control antibody measurements are deferred to the Supplementary Note and affect the dsb input values feeding the clustering

## Evidence

- [results] ROC-test DE for cluster 3 and write the normalized median-value tables: [evidence span withheld — anchor status: not_found]
- [discussion] the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [results] differentially expressed transcripts in cluster 3 included inflammatory and activation genes: [evidence span withheld — anchor status: not_found]
- [abstract] interfaces easily with existing single cell software platforms such as Seurat, Bioconductor, and Scanpy: "interfaces easily with existing single cell software platforms such as Seurat, Bioconductor, and Scanpy"
