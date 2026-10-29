---
name: pathway-coordination-testing-kld
description: Use when scRNA-seq data (Seurat object) must be screened to determine
  which curated gene sets (KEGG, BIOCARTA, GO:BP) are genuinely heterogeneous across
  cells prior to pathway activity scoring or binarization, without pre-clustering
  the cells.
when_to_use_negative:
- Data are targeted gene-panel scRNA-seq with limited numbers of genes — GSDensity
  is not suitable for such data
- The gene sets are intended to be used directly without testing; the article recommends
  binarization only on gene sets that pass the coordination test
- Input lacks whole-transcriptome coverage needed to estimate an all-gene background
  density in the MCA space
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GSDensity
  role: Provides compute.mca and compute.kld for MCA co-embedding and KL-divergence
    coordination testing of gene sets
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Underlying implementation of the MCA cell-gene co-embedding used by compute.mca
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: msigdbr
  role: Supplies curated MSigDB gene sets (KEGG, BIOCARTA, GO:BP) converted to a named
    list as test input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:ff78908acf9ea9dce6842b22cdd3e06fee02c7c8c3a107529869896d6fda146e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - RANN
  - Seurat
  - igraph
  - monocle3
  - msigdbr
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pathway-coordination-testing-kld

## Summary

Tests whether a curated gene set/pathway shows heterogeneous, coordinated activity among cells in single-cell RNA-seq data, by co-embedding cells and genes with MCA and comparing the pathway-gene density to the all-gene background density via KL-divergence permutation testing in GSDensity.

## When to use

Use when scRNA-seq data (Seurat object) must be screened to determine which curated gene sets (KEGG, BIOCARTA, GO:BP) are genuinely heterogeneous across cells prior to pathway activity scoring or binarization, without pre-clustering the cells.

## When NOT to use

- Data are targeted gene-panel scRNA-seq with limited numbers of genes — GSDensity is not suitable for such data
- The gene sets are intended to be used directly without testing; the article recommends binarization only on gene sets that pass the coordination test
- Input lacks whole-transcriptome coverage needed to estimate an all-gene background density in the MCA space

## Inputs

- Normalized Seurat single-cell RNA-seq object (e.g. pbmc3k from SeuratData)
- Named list of gene sets from msigdbr (KEGG, BIOCARTA, GO:BP categories)
- Cell-by-gene co-embedding from compute.mca (MCA latent space)

## Outputs

- Data frame (res) with per-gene-set KL-divergence test statistics, raw p-values, and adjusted p-values (p.adj)
- Vector of coordinated gene sets passing p.adj < 0.05 (gene.set.deviated)

## How to apply

First co-embed cells and genes into the same latent space with compute.mca on a normalized Seurat object (e.g. pbmc3k). Then call compute.kld with genes.use = intersect(rownames(coembed), rownames(object)), n.grids = 100 and n.times = 100 permutations, passing the gene sets as a named list built with msigdbr; compute.kld estimates the density of each pathway's genes in the MCA space and compares it to the all-gene background density using KL-divergence, assessing significance by permutation. Retain gene sets with adjusted p-value < 0.05 (res[res$p.adj < 0.05, ]$gene.set) as coordinated pathways; the article recommends performing downstream steps such as PAL binarization only on gene sets that pass this coordination test. Parameters such as n.grids and n.times control the density-grid resolution and permutation count, and results can be applied to pathway collections (97 KEGG/BIOCARTA sets in the mouse brain development analysis).

## Related tools

- **GSDensity** (Provides compute.mca and compute.kld for MCA co-embedding and KL-divergence coordination testing of gene sets) — https://github.com/qingnanl/gsdensity
- **CelliD** (Underlying implementation of the MCA cell-gene co-embedding used by compute.mca) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **msigdbr** (Supplies curated MSigDB gene sets (KEGG, BIOCARTA, GO:BP) converted to a named list as test input)

## Examples

```
ce <- compute.mca(object = pbmc3k); res <- compute.kld(coembed = ce, genes.use = intersect(rownames(ce), rownames(pbmc3k)), n.grids = 100, gene.set.list = gene.set.list, gene.set.cutoff = 3, n.times = 100)
```

## Evaluation signals

- Adjusted p-value (p.adj) is present for every tested gene set and coordinated sets satisfy p.adj < 0.05
- Reproducing the mouse brain development analysis must yield 97 significant KEGG/BIOCARTA pathways at adjusted p < 0.05
- genes.use passed to compute.kld must be the intersection of coembed rownames and object rownames so only genes, not cells, are used
- n.grids = 100 and n.times = 100 permutation parameters match the reported settings
- Output is a data frame res from which gene.set.deviated <- res[res$p.adj < 0.05, ]$gene.set can be extracted

## Limitations

- GSDensity's MCA embedding step is memory-intensive (~25 Gb for a 40,000-cell dataset like hcabm40k)
- gsdensity version number is not stated and the CRAN install was temporarily unavailable, complicating reproducibility
- Improper batch correction (over- or under-correction) may introduce bias before pathway analysis

## Evidence

- [intro] quantifies the overall variation of pathway activity levels across cells by estimating the density of the pathway genes in the latent space: "quantifies the overall variation of pathway activity levels across cells by estimating the density of the pathway genes in the latent space"
- [methods] compute.kld call with parameters: "res <- compute.kld(coembed = ce, genes.use = intersect(rownames(ce), rownames(pbmc3k)),"
- [readme] extraction of significant gene sets: "gene.set.deviated <- res[res$p.adj < 0.05, ]$gene.set"
- [readme] gene set density rationale in MCA space: "when a gene set shows a very different 'density' in the MCA space compared with the background, it indicates that there does exist some 'affinity' between the gene set and some cell subpopulation"
- [discussion] CelliD origin of MCA embedding: "The application of MCA embedding in scRNA-seq studies was first reported in the CelliD12 method"
