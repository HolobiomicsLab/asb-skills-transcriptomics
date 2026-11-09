---
name: single-cell-gene-set-density-analysis
description: Use when you have a cell-by-gene scRNA-seq matrix (Seurat object) and
  curated gene sets, and need to determine whether specific gene sets are heterogeneously
  coordinated across cells without clustering.
when_to_use_negative:
- Input is targeted gene data with limited numbers of genes — GSDensity is 'not suitable
  for targeted gene data having limited numbers of genes'
- Gene set size is below the gene.set.cutoff = 3 minimum (small sets cannot support
  density estimation)
- The goal is per-cell pathway activity scoring on spatial data rather than gene set
  coordination testing — use the spatial/relevance workflow (compute.spatial.kld)
  instead
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GSDensity (gsdensity R package)
  role: Provides compute.mca() for MCA cell-gene co-embedding and compute.kld() for
    coordination testing with random gene set nulls
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Inspires and supplies the MCA embedding used to co-project cells and genes
    into the same low-dimensional space
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: Seurat / SeuratData
  role: Loads and normalizes input scRNA-seq objects (e.g. pbmc3k via SeuratData)
- name: msigdbr
  role: Supplies GO:BP gene set collections converted to a named list of gene symbols
- name: future.apply
  role: Parallel computing support when testing many gene sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:b314edbb45f01dc1b86fe8c7b84249cc47465e86d97a4fdaac1952e162d8114e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - GSDensity (gsdensity R package)
  - Seurat
  - Seurat / SeuratData
  - future.apply
  - msigdbr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# single-cell-gene-set-density-analysis

## Summary

Cluster-free gene set coordination testing with GSDensity: co-embed cells and genes via multiple correspondence analysis (MCA), compare pathway-gene density against all-gene density using KL-divergence, and call gene sets 'coordinated' (deviated) at adjusted p < 0.05 using size-matched random gene set nulls.

## When to use

Use when you have a cell-by-gene scRNA-seq matrix (Seurat object) and curated gene sets, and need to determine whether specific gene sets are heterogeneously coordinated across cells without clustering.

## When NOT to use

- Input is targeted gene data with limited numbers of genes — GSDensity is 'not suitable for targeted gene data having limited numbers of genes'
- Gene set size is below the gene.set.cutoff = 3 minimum (small sets cannot support density estimation)
- The goal is per-cell pathway activity scoring on spatial data rather than gene set coordination testing — use the spatial/relevance workflow (compute.spatial.kld) instead

## Inputs

- Normalized Seurat object (cell x gene matrix), e.g. pbmc3k from SeuratData
- Gene sets as a named list of gene symbols (e.g. from msigdbr GO:BP category or curated marker lists)
- Marker gene lists for real-world datasets (.rds marker files, e.g. Cell_marker_Human.xlsx, PanglaoDB_markers_27_Mar_2020.tsv.gz)

## Outputs

- Data frame of per-gene-set KLD statistics and empirical adjusted p-values (columns: gene.set, p.adj)
- List of deviated gene sets with p.adj < 0.05
- Aggregated per-dataset results table comparing marker, mixed, and random gene set categories

## How to apply

Normalize the Seurat object and co-embed cells and genes into a shared MCA latent space with gsdensity's compute.mca(). Run compute.kld() with genes.use restricted to genes, n.grids = 100, gene.set.cutoff = 3, and n.times = 100 size-matched random gene set nulls to obtain observed KLD statistics and empirical adjusted p-values for every gene set. Extract deviated gene sets via res[res$p.adj < 0.05, ]$gene.set. The rationale is that a gene set whose density in the MCA space differs from the background indicates 'affinity' between the gene set and a cell subpopulation, without requiring clustering or manual annotation. In the benchmarking (Fig. 2a-e), curated marker sets scored highest, mixed marker/random sets lower, and purely random sets almost no significance across 8 real-world datasets (pbmc3k, bmcite.small, pancreas, liver.immune, spleen.immune, hcortex, lung, heart).

## Related tools

- **GSDensity (gsdensity R package)** (Provides compute.mca() for MCA cell-gene co-embedding and compute.kld() for coordination testing with random gene set nulls) — https://github.com/qingnanl/gsdensity
- **CelliD** (Inspires and supplies the MCA embedding used to co-project cells and genes into the same low-dimensional space) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **Seurat / SeuratData** (Loads and normalizes input scRNA-seq objects (e.g. pbmc3k via SeuratData))
- **msigdbr** (Supplies GO:BP gene set collections converted to a named list of gene symbols)
- **future.apply** (Parallel computing support when testing many gene sets)

## Examples

```
ce <- compute.mca(object = pbmc3k); res <- compute.kld(coembed = ce, genes.use = intersect(rownames(ce), rownames(pbmc3k)), n.grids = 100, gene.set.list = gene.set.list, gene.set.cutoff = 3, n.times = 100)
```

## Evaluation signals

- Marker gene sets receive the highest coordination significance (p.adj < 0.05) relative to mixed marker/random and purely random sets across all eight benchmark datasets (pbmc3k, bmcite.small, pancreas, liver.immune, spleen.immune, hcortex, lung, heart)
- Random-gene sets show almost no significance, so any random set with p.adj < 0.05 flags a possible calibration problem
- compute.kld must use n.grids = 100, gene.set.cutoff = 3, and n.times = 100 size-matched random nulls; deviations from these parameters invalidate comparability
- genes.use must be restricted to genes only (e.g. intersect of coembed rownames and the original matrix rownames) so cells are not treated as background items
- The 'anticlust' dependency must be an older version (e.g. 0.6.1) or installation/behavior may be off

## Limitations

- Not suitable for targeted gene data with limited numbers of genes; imputation of unmeasured genes is suggested but not implemented
- MCA embedding step is memory-hungry (~25 Gb for a 40,000-cell dataset like hcabm40k)
- Installation fails on macOS with M1 chips due to a CelliD/scuttle dependency; 'anticlust' must be pinned to an older version (e.g. 0.6.1)
- Over- or under-correction of batch effects may introduce bias before pathway analysis
- AUCell is faster than GSDensity when the number of cells is small, though GSDensity is more scalable

## Evidence

- [methods] compute.kld on genes with nulls and grids: "res <- compute.kld(coembed = ce, genes.use = intersect(rownames(ce), rownames(pbmc3k)),"
- [intro] coordination test purpose: "it should distinguish whether a pathway, in the format of a gene set, is truly heterogenous among the cells without clustering them"
- [discussion] recommend coordination test before binarization: "we recommend performing binarization on only gene sets that pass the coordination test"
- [readme] README quick-start parameters: "gene.set.deviated <- res[res$p.adj < 0.05, ]$gene.set"
- [readme] README on density interpretation: "when a gene set shows a very different 'density' in the MCA space compared with the background, it indicates that there does exist some 'affinity' between the gene set and some cell subpopulation"
