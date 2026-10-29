---
name: random-walk-restart-label-propagation
description: Use when scoring per-cell pathway activity in normalized scRNA-seq data
  (e.g. Seurat objects) to fetch the cells most relevant to a curated gene set (GO,
  Hallmarks, KEGG, BIOCARTA) without clustering.
when_to_use_negative:
- Data are targeted-gene panels with limited gene numbers; GSDensity is 'not suitable
  for targeted gene data having limited numbers of genes'
- The gene set fails the coordination (KLD) test — binarization is recommended only
  for coordinated gene sets
- A fast score for a small cell number is needed, where AUCell is faster than GSDensity
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: gsdensity
  role: Provides compute.mca, compute.nn.edges, run.rwr/run.rwr.list, and compute.cell.label
    for RWR-based pathway activity scoring and binarization
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Source of the MCA co-embedding approach used to build the cell-gene graph
    prior to RWR
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: Seurat
  role: Normalization (NormalizeData) of input scRNA-seq data and downstream visualization
    (FeaturePlot/DimPlot) of RWR probabilities and labels
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:57ba2536a53ca8582faf090c8d1dc9bd507fddeb8c8a993e7b6b42bfeec5dee8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - AUCell
  - CelliD
  - GSDensity
  - Seurat
  - VAM
  - gsdensity
  - ssGSEA
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# random-walk-restart-label-propagation

## Summary

GSDensity's single-cell pathway activity scoring step: after MCA co-embedding of cells and genes and nearest-neighbor graph construction, pathway genes are used as seeds for random walk with restart (RWR) on the cell-gene graph, yielding per-cell pathway activity (label propagation) probabilities that can be binarized to fetch the most relevant cells.

## When to use

Use when scoring per-cell pathway activity in normalized scRNA-seq data (e.g. Seurat objects) to fetch the cells most relevant to a curated gene set (GO, Hallmarks, KEGG, BIOCARTA) without clustering.

## When NOT to use

- Data are targeted-gene panels with limited gene numbers; GSDensity is 'not suitable for targeted gene data having limited numbers of genes'
- The gene set fails the coordination (KLD) test — binarization is recommended only for coordinated gene sets
- A fast score for a small cell number is needed, where AUCell is faster than GSDensity

## Inputs

- Normalized cell-by-gene matrix or Seurat object (e.g. pbmc3k after NormalizeData)
- MCA co-embedding of cells and genes (output of compute.mca)
- Nearest-neighbor cell-gene edge list from compute.nn.edges (nn.use = 300)
- Gene set(s) as named lists of gene symbols (e.g. GOBP_B_CELL_ACTIVATION from msigdbr GO:BP)

## Outputs

- Named vector of per-cell RWR label-propagation probabilities (cv)
- Binarized positive/negative cell labels per gene set (cl via compute.cell.label)
- Aggregated pathway activity score matrix across gene set collections (GO, Hallmarks, KEGG, BIOCARTA)

## How to apply

First co-embed cells and genes into a shared latent space with compute.mca (MCA, as in CelliD), then build the nearest-neighbor cell-gene graph with compute.nn.edges using nn.use = 300. Run random walk with restart via run.rwr (or run.rwr.list for many sets) with default restart probability 0.75, seeding the walk with the pathway's genes. Convert the resulting per-cell propagation probabilities into positive/negative labels with compute.cell.label, i.e. PAL-based binarization. Per the article, binarize only gene sets that pass the coordination (KLD) test, and visually inspect PAL distributions since some pathways show more than two modalities. For TNBC-style applications (e.g. G2M checkpoint scoring to define dividing TYRO3-high tumor cells), the binarized labels feed downstream tests such as a Wilcoxon rank-sum comparison.

## Related tools

- **gsdensity** (Provides compute.mca, compute.nn.edges, run.rwr/run.rwr.list, and compute.cell.label for RWR-based pathway activity scoring and binarization) — https://github.com/qingnanl/gsdensity
- **CelliD** (Source of the MCA co-embedding approach used to build the cell-gene graph prior to RWR) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **Seurat** (Normalization (NormalizeData) of input scRNA-seq data and downstream visualization (FeaturePlot/DimPlot) of RWR probabilities and labels)

## Examples

```
ce <- compute.mca(object = pbmc3k); el <- compute.nn.edges(coembed = ce, nn.use = 300); cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells); cl <- compute.cell.label(cv)
```

## Evaluation signals

- Binarized labels reproduce known biology, e.g. G2M checkpoint-positive labels overlap dividing TNBC tumor cells with Wilcoxon TYRO3 p = 3.18e-8 in TNBC-5 and p = 0.039 for GSM4909284_TN-MH0114-T2
- In benchmarking, cell-type recovery AUC and accuracy across 27 SERGIO-simulated and 8 real datasets rank GSDensity first and CelliD typically second
- PAL distributions of key pathways are visually inspected and show the expected two-modality separation after compute.cell.label
- Only gene sets passing the coordination test (res$p.adj < 0.05 in compute.kld output) are binarized
- The same graph parameters (nn.use = 300, restart = 0.75) are used consistently so scores are comparable across gene sets

## Limitations

- Neighborhood size (nn.use) is an important parameter when constructing the cell-gene graph and affects results
- Some pathways show more than two modalities in PAL distribution, so automatic binarization should be visually validated and corrected as necessary
- MCA embedding is memory-intensive (~25 Gb for a 40,000-cell dataset like hcabm40k)
- RWR can raise a Matrix deprecation error ('as(<dsCMatrix>, "dgCMatrix") is deprecated since Matrix 1.5-0'); fix with igraph::igraph.options(sparsematrices = FALSE)
- The 'anticlust' dependency requires an older version (e.g. 0.6.1), and installation fails on macOS M1 due to a CelliD/scuttle dependency

## Evidence

- [methods] Pathway activity for each cell via network propagation on a nearest-neighbor graph with pathway genes as seeds: "Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks"
- [readme] RWR and binarization calls with nn.use = 300: "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
- [readme] run.rwr usage with a GOBP gene set: "cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells)"
- [discussion] Binarize only coordinated gene sets: "we recommend performing binarization on only gene sets that pass the coordination test"
- [discussion] PAL-based binarization demonstrated on TNBC data: "we also offer an option of PAL-based cell binarization for downstream analysis, as was demonstrated in the TNBC data analysis"
- [discussion] GSDensity vs CelliD mechanism distinction: "GSDensity uses network propagation to estimate the pathway relevance at single-cell level, while CelliD performs hypergeometric test"
