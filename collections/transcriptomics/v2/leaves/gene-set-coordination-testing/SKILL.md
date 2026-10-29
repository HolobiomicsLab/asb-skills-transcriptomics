---
name: gene-set-coordination-testing
description: Use when you have a normalized cell-by-gene scRNA-seq or ST matrix and
  a list of gene sets, and need to decide which pathways are truly coordinated among
  cells without clustering, before pathway scoring or binarization.
when_to_use_negative:
- Targeted gene panel data with limited numbers of genes — the article states GSDensity
  'is not suitable for targeted gene data having limited numbers of genes'
- If downstream goal is cell-type specificity scoring of already-annotated spots rather
  than coordination testing — compute.spec is the appropriate tool
- If pathway activity per cell (not coordination) is already established and only
  scoring speed matters — AUCell may be faster on small datasets
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GSDensity
  role: Provides compute.mca for MCA co-embedding and compute.kld for density-based
    coordination testing with permutation nulls
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Dependency supplying the MCA co-embedding implementation; requires the updated
    Bioconductor version
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: Seurat
  role: Loads and normalizes input datasets (e.g. pbmc3k, stxBrain Visium) and supplies
    rownames for genes.use intersection
- name: msigdbr
  role: Supplies GO:BP / MSigDB gene set collections converted to a named list for
    testing
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:062e28ccea88da96c51ca547de294bdbece0a956ab12e8343c702dae7ca05076
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - Seurat
  - SeuratData
  - future.apply
  - igraph
  - msigdbr
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Gene set coordination testing with GSDensity MCA and KLD

## Summary

Cluster-free statistical test of whether a gene set/pathway is heterogeneous (coordinated) among cells: cells and genes are co-embedded by multiple correspondence analysis (MCA) and the density of pathway genes in the latent space is compared against the all-gene background with KL-divergence and size-matched random-set permutation nulls.

## When to use

Use when you have a normalized cell-by-gene scRNA-seq or ST matrix and a list of gene sets, and need to decide which pathways are truly coordinated among cells without clustering, before pathway scoring or binarization.

## When NOT to use

- Targeted gene panel data with limited numbers of genes — the article states GSDensity 'is not suitable for targeted gene data having limited numbers of genes'
- If downstream goal is cell-type specificity scoring of already-annotated spots rather than coordination testing — compute.spec is the appropriate tool
- If pathway activity per cell (not coordination) is already established and only scoring speed matters — AUCell may be faster on small datasets

## Inputs

- Normalized cell-by-gene expression matrix (Seurat object, e.g. pbmc3k from SeuratData)
- Gene set list (e.g. GO:BP sets from msigdbr converted to a named list of gene symbols; marker, mixed, or random sets)
- MCA co-embedding of cells and genes produced by compute.mca

## Outputs

- Data frame res with per-gene-set observed KLD statistics and empirical adjusted p-values (p.adj)
- Vector of deviated/coordinated gene sets: res[res$p.adj < 0.05, ]$gene.set

## How to apply

Normalize the count matrix (e.g. Seurat NormalizeData) and co-embed cells and genes into a shared latent space with gsdensity::compute.mca, which uses MCA following the CelliD approach. Run compute.kld on the co-embedding with genes.use restricted to measured genes, n.grids = 100, gene.set.cutoff = 3, and n.times = 100 size-matched random gene set permutations to obtain observed KLD statistics and empirical adjusted p-values per gene set. Extract significant (deviated/coordinated) sets with res[res$p.adj < 0.05, ]$gene.set; these are the sets for which the density of pathway genes in the MCA space differs from the background all-gene density. Rationale: this coordination test distinguishes true coordinated marker sets from mixed marker/random and purely random sets (verified on pbmc3k, bmcite.small, pancreas, liver.immune, spleen.immune, hcortex, lung, heart), and the authors recommend passing only these sets downstream to run.rwr-based activity scoring, spatial relevance testing, or PAL binarization.

## Related tools

- **GSDensity** (Provides compute.mca for MCA co-embedding and compute.kld for density-based coordination testing with permutation nulls) — https://github.com/qingnanl/gsdensity
- **CelliD** (Dependency supplying the MCA co-embedding implementation; requires the updated Bioconductor version) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **Seurat** (Loads and normalizes input datasets (e.g. pbmc3k, stxBrain Visium) and supplies rownames for genes.use intersection)
- **msigdbr** (Supplies GO:BP / MSigDB gene set collections converted to a named list for testing)

## Examples

```
ce <- compute.mca(object = pbmc3k); res <- compute.kld(coembed = ce, genes.use = intersect(rownames(ce), rownames(pbmc3k)), n.grids = 100, gene.set.list = gene.set.list, gene.set.cutoff = 3, n.times = 100)
```

## Evaluation signals

- Marker gene sets receive the highest coordination significance (p.adj < 0.05) while purely random gene sets show almost no significance, validated across pbmc3k, bmcite.small, pancreas, liver.immune, spleen.immune, hcortex, lung, and heart
- compute.kld was run with genes.use = intersect(rownames(ce), rownames(pbmc3k)), n.grids = 100, gene.set.cutoff = 3, n.times = 100; deviating from these without cause changes empirical p-values
- Significant (deviated) gene sets are exactly those with res$p.adj < 0.05; the returned set should be a strict subset of input gene sets with at least gene.set.cutoff = 3 genes
- In the PBMC/GO:BP setting, over 98% of coordinated pathways reported by the authors, giving a sanity anchor on the fraction of sets expected to pass
- For pan-cancer Visium analysis, coordination pre-filtering should retain the pathway pool whose 6-dataset intersection at adjusted p < 0.05 contains exactly 34 pathways

## Limitations

- MCA embedding is memory-intensive: about 25 Gb for the 40,000-cell hcabm40k SeuratData dataset
- Installation fails on macOS with M1 chips due to a CelliD/scuttle dependency; the 'anticlust' dependency must be an older version (e.g. 0.6.1) and the CRAN gsdensity version was temporarily unavailable
- Proper batch correction is required — over- or under-correction may introduce bias in downstream pathway analysis
- gsdensity version number is not stated in the methods

## Evidence

- [intro] quantifies the overall variation of pathway activity levels across cells by estimating the density of the pathway genes in the latent space: "quantifies the overall variation of pathway activity levels across cells by estimating the density of the pathway genes in the latent space"
- [methods] compute.kld on the co-embedding: "res <- compute.kld(coembed = ce, 
                   genes.use = intersect(rownames(ce), rownames(pbmc3k)),"
- [readme] README quick-start parameters: "                   n.grids = 100, 
                   gene.set.list = gene.set.list,
                   gene.set.cutoff = 3,
                   n.times = 100)"
- [discussion] coordination test precedes binarization: "we recommend performing binarization on only gene sets that pass the coordination test"
