---
name: kl-divergence-null-hypothesis-testing
description: Use when you must test whether a gene set/pathway is truly coordinated
  (heterogeneous) among cells in scRNA-seq data without pre-clustering, e.g. validating
  marker, mixed, or random gene sets across datasets.
when_to_use_negative:
- Data are targeted gene panels with limited numbers of genes — GSDensity is 'not
  suitable for targeted gene data having limited numbers of genes' per the discussion
- You need cluster/annotation-free per-cell pathway activity scores rather than a
  gene set coordination p-value — use run.rwr/compute.cell.label instead
- Input gene sets are smaller than gene.set.cutoff = 3 genes
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GSDensity
  role: Provides compute.mca() for MCA co-embedding and compute.kld() for KL-divergence-based
    coordination testing against size-matched random gene set nulls
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Underlying package for the MCA co-embedding used by GSDensity; requires the
    updated Bioconductor version
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: Seurat
  role: Loads and normalizes the scRNA-seq datasets (e.g., pbmc3k from SeuratData)
    prior to MCA embedding
- name: msigdbr
  role: Supplies GO:BP gene set collections converted to a named list format for compute.kld()
- name: future.apply
  role: Parallel computing backend for scaling the KLD null resampling across many
    gene sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:6588f68a6e84936be6e074eef00cf651f6ca20100da6147930cd7c34ac4f8c77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - Seurat
  - future.apply
  - msigdbr
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# KL-divergence null hypothesis testing

## Summary

GSDensity's cluster-free gene set coordination test: co-embed cells and genes via MCA, compute the KL-divergence (KLD) between a gene set's density and the background density in the latent space, and calibrate significance against size-matched random gene sets (n.times = 100) with adjusted p-values at alpha 0.05.

## When to use

Use when you must test whether a gene set/pathway is truly coordinated (heterogeneous) among cells in scRNA-seq data without pre-clustering, e.g. validating marker, mixed, or random gene sets across datasets.

## When NOT to use

- Data are targeted gene panels with limited numbers of genes — GSDensity is 'not suitable for targeted gene data having limited numbers of genes' per the discussion
- You need cluster/annotation-free per-cell pathway activity scores rather than a gene set coordination p-value — use run.rwr/compute.cell.label instead
- Input gene sets are smaller than gene.set.cutoff = 3 genes

## Inputs

- Normalized cell-by-gene scRNA-seq matrix as a Seurat object (e.g., pbmc3k via SeuratData)
- Gene set list (marker, mixed marker/random, or random sets; e.g., GO:BP sets from msigdbr)
- MCA co-embedding of cells and genes produced by compute.mca()

## Outputs

- Data frame res with columns gene.set, KLD statistic, and p.adj (empirical adjusted p-value)
- Deviated gene set list: res[res$p.adj < 0.05, ]$gene.set
- Per-dataset ranking of gene set categories (marker > mixed > random) by coordination significance

## How to apply

First normalize the Seurat object and co-embed cells and genes into a shared MCA latent space with compute.mca(). Then run compute.kld() with genes.use restricted to genes (intersecting coembed rownames with the data rownames), n.grids = 100, gene.set.cutoff = 3, and n.times = 100 size-matched random gene set nulls to obtain observed KLD statistics and empirical adjusted p-values for each gene set. Extract deviated (significant) gene sets with res[res$p.adj < 0.05, ]$gene.set and rank gene set categories by significance. The rationale is that a gene set whose density in the MCA space differs from the background density of all genes indicates affinity to a cell subpopulation, without requiring clustering or manual annotation. In the manuscript benchmark, curated marker sets across 8 real-world datasets (pbmc3k, bmcite.small, pancreas, liver.immune, spleen.immune, hcortex, lung, heart) received the highest coordination significance, mixed marker/random sets showed lower significance, and random sets showed almost none.

## Related tools

- **GSDensity** (Provides compute.mca() for MCA co-embedding and compute.kld() for KL-divergence-based coordination testing against size-matched random gene set nulls) — https://github.com/qingnanl/gsdensity
- **CelliD** (Underlying package for the MCA co-embedding used by GSDensity; requires the updated Bioconductor version) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **Seurat** (Loads and normalizes the scRNA-seq datasets (e.g., pbmc3k from SeuratData) prior to MCA embedding)
- **msigdbr** (Supplies GO:BP gene set collections converted to a named list format for compute.kld())
- **future.apply** (Parallel computing backend for scaling the KLD null resampling across many gene sets)

## Examples

```
ce <- compute.mca(object = pbmc3k); res <- compute.kld(coembed = ce, genes.use = intersect(rownames(ce), rownames(pbmc3k)), n.grids = 100, gene.set.list = gene.set.list, gene.set.cutoff = 3, n.times = 100); gene.set.deviated <- res[res$p.adj < 0.05, ]$gene.set
```

## Evaluation signals

- All tested marker gene sets in each of the 8 benchmark datasets yield p.adj < 0.05, while random gene sets show almost no significance (p.adj >= 0.05)
- Ranking invariant: marker sets > mixed marker/random sets > random sets by coordination significance within every dataset (pbmc3k, bmcite.small, pancreas, liver.immune, spleen.immune, hcortex, lung, heart)
- compute.kld() call uses the documented parameters: n.grids = 100, gene.set.cutoff = 3, n.times = 100
- genes.use contains only genes (intersect(rownames(ce), rownames(pbmc3k))), excluding cell identifiers
- Reproducibility: the same benchmark can be rerun via benchmark.real(fname = 'pbmc3k') with 'pbmc3k.rds' and 'pbmc3k.markers.rds' from the GSDensity_manuscript_code repository

## Limitations

- MCA embedding step requires large memory (~25 Gb for a 40,000-cell dataset)
- Not suitable for targeted gene data with limited numbers of genes; imputation of unmeasured genes is suggested but not implemented
- Over- or under-correction of batch effects may introduce bias, requiring careful evaluation before pathway analysis
- Installation is unsupported on macOS with M1 chips due to a CelliD/scuttle dependency; the 'anticlust' dependency must be an older version (e.g., 0.6.1)
- gsdensity version number is not stated; the CRAN stable version was temporarily unavailable

## Evidence

- [intro] quantifies the overall variation of pathway activity levels across cells by estimating the density of the pathway genes in the latent space: "quantifies the overall variation of pathway activity levels across cells by estimating the density of the pathway genes in the latent space"
- [methods] res <- compute.kld(coembed = ce, genes.use = intersect(rownames(ce), rownames(pbmc3k)),: "res <- compute.kld(coembed = ce, 
                   genes.use = intersect(rownames(ce), rownames(pbmc3k)),"
- [intro] it should distinguish whether a pathway, in the format of a gene set, is truly heterogenous among the cells without clustering them: "it should distinguish whether a pathway, in the format of a gene set, is truly heterogenous among the cells without clustering them"
- [discussion] we recommend performing binarization on only gene sets that pass the coordination test: "we recommend performing binarization on only gene sets that pass the coordination test"
- [readme] The core idea of gsdensity is that we use multiple correspondence analsysis (MCA) to co-embed cells and genes (thanks to the CelliD package): "The core idea of gsdensity is that we use multiple correspondence analsysis (MCA) to co-embed cells and genes"
