---
name: marker-vs-random-gene-set-benchmarking
description: 'Use when validating a gene-set coordination or enrichment method''s
  specificity: curated marker, mixed marker/random, and size-matched random gene sets
  are tested on real scRNA-seq data to confirm true sets rank highest.'
when_to_use_negative:
- Input is targeted gene expression data with limited numbers of genes — GSDensity
  is not suitable there and imputation of unmeasured genes is suggested but not implemented
- The goal is to interpret novel pathway activity in a single dataset rather than
  to benchmark coordination-test specificity against null gene sets
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: GSDensity
  role: Provides compute.mca() for MCA co-embedding and compute.kld() for the coordination
    significance test with size-matched random nulls
  repo: https://github.com/qingnanl/gsdensity
- name: Seurat
  role: Loads and normalizes scRNA-seq objects (e.g., pbmc3k via SeuratData with NormalizeData)
    prior to MCA embedding
- name: msigdbr
  role: Source of curated gene set collections used to assemble gene set lists for
    testing
- name: future.apply
  role: Parallel computing backend for scaling KLD computation across gene sets and
    datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:b661b866c5f6c14369f32817d704a8d48b8f84e94d720bbc3d6acb34859c5a84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - GSDensity
  - Seurat
  - future.apply
  - msigdbr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# marker-vs-random-gene-set-benchmarking

## Summary

Benchmark GSDensity's cluster-free gene set coordination test by comparing curated cell-type marker sets against mixed marker/random and purely random gene sets across eight real-world scRNA-seq datasets, verifying that true coordinated sets receive the highest significance at alpha 0.05.

## When to use

Use when validating a gene-set coordination or enrichment method's specificity: curated marker, mixed marker/random, and size-matched random gene sets are tested on real scRNA-seq data to confirm true sets rank highest.

## When NOT to use

- Input is targeted gene expression data with limited numbers of genes — GSDensity is not suitable there and imputation of unmeasured genes is suggested but not implemented
- The goal is to interpret novel pathway activity in a single dataset rather than to benchmark coordination-test specificity against null gene sets

## Inputs

- Eight scRNA-seq datasets: pbmc3k (SeuratData), bmcite.small, pancreas, liver.immune, spleen.immune, hcortex, lung, heart (.rds from qingnanl/GSDensity_manuscript_code)
- Curated cell-type marker gene set lists (Cell_marker_Human.xlsx, Cell_marker_Mouse.xlsx, st4.csv, PanglaoDB_markers_27_Mar_2020.tsv.gz curated via curate_data.R)
- Mixed marker/random gene set lists
- Random gene set lists

## Outputs

- Per-dataset table of KLD statistics and empirical adjusted p-values (p.adj) for marker, mixed, and random gene sets
- Vector of significant (deviated) gene sets per dataset via res[res$p.adj < 0.05, ]$gene.set
- Aggregated results table showing ranking of the three gene set categories across all eight datasets

## How to apply

Load each of the eight benchmarking datasets (pbmc3k via SeuratData; bmcite.small, pancreas, liver.immune, spleen.immune, hcortex, lung, heart from the qingnanl/GSDensity_manuscript_code repository) with its marker list, then co-embed cells and genes into the shared MCA latent space using GSDensity's compute.mca(). Run compute.kld() with genes.use restricted to genes, n.grids = 100, gene.set.cutoff = 3, and n.times = 100 size-matched random gene set nulls to obtain observed KLD statistics and empirical adjusted p-values per gene set. Extract deviated sets with res[res$p.adj < 0.05, ]$gene.set and rank marker, mixed, and random categories within each dataset. The benchmark passes when marker sets receive the highest coordination significance relative to mixed sets, while random sets show almost no significance, across all eight datasets.

## Related tools

- **GSDensity** (Provides compute.mca() for MCA co-embedding and compute.kld() for the coordination significance test with size-matched random nulls) — https://github.com/qingnanl/gsdensity
- **Seurat** (Loads and normalizes scRNA-seq objects (e.g., pbmc3k via SeuratData with NormalizeData) prior to MCA embedding)
- **msigdbr** (Source of curated gene set collections used to assemble gene set lists for testing)
- **future.apply** (Parallel computing backend for scaling KLD computation across gene sets and datasets)

## Examples

```
ce <- compute.mca(object = pbmc3k); res <- compute.kld(coembed = ce, genes.use = intersect(rownames(ce), rownames(pbmc3k)), n.grids = 100, gene.set.cutoff = 3, n.times = 100); deviated <- res[res$p.adj < 0.05, ]$gene.set
```

## Evaluation signals

- Marker gene sets achieve adjusted p-values < 0.05 in all eight benchmarking datasets
- Within each dataset, marker sets rank above mixed marker/random sets, and random-gene sets show almost no coordination significance
- compute.kld() runs with the specified parameters return both observed KLD statistics and empirical adjusted p-values for every gene set
- Aggregated per-dataset results table is complete for pbmc3k, bmcite.small, pancreas, liver.immune, spleen.immune, hcortex, lung, and heart

## Limitations

- MCA embedding is memory-intensive (~25 Gb for a 40,000-cell dataset)
- gsdensity version number is not stated; the CRAN stable version was temporarily unavailable and installation via remotes from GitHub is recommended
- Installation fails on macOS with M1 chips due to a CelliD/scuttle dependency issue
- The 'anticlust' dependency must be an older version (e.g., 0.6.1)

## Evidence

- [results] Across all 8 real-world benchmarking datasets (pbmc3k, bmcite.small, pancreas, liver.immune, spleen.immune, hcortex, lung, heart), marker sets received the highest coordination significance at alpha 0.05, mixed sets showed lower significance, and random-gene sets showed almost no significance.: [evidence span withheld — anchor status: not_found]
- [readme] gene.set.deviated <- res[res$p.adj < 0.05, ]$gene.set: "gene.set.deviated <- res[res$p.adj < 0.05, ]$gene.set"
- [readme] compute.kld with n.grids = 100, gene.set.cutoff = 3, n.times = 100: "res <- compute.kld(coembed = ce, genes.use = intersect(rownames(ce), rownames(pbmc3k)), n.grids = 100"
- [intro] GSDensity first projects cells and genes in scRNA-seq data into the same low dimensional space using MCA: "GSDensity first projects cells and genes in scRNA-seq data into the same low dimensional space using MCA"
- [readme] run benchmark.real(fname = "pbmc3k") with "pbmc3k.rds" and "pbmc3k.markers.rds" in the dir: [evidence span withheld — anchor status: external_doc]
