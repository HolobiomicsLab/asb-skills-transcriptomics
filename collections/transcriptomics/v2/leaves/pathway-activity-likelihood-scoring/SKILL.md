---
name: pathway-activity-likelihood-scoring
description: Use when you need single-cell pathway activity scores (PAL) for a gene
  set in scRNA-seq or spatial transcriptomics data without clustering, e.g., to fetch
  relevant cells or weight cells for spatial relevance tests.
when_to_use_negative:
- Input is targeted gene-panel data with limited numbers of genes — the article states
  GSDensity 'is not suitable for targeted gene data having limited numbers of genes'
- The goal is differential-density testing of gene sets in the MCA space alone (use
  compute.kld) rather than per-cell activity scoring/fetching of relevant cells
- Running on macOS with an M1 chip, where installation fails due to a CelliD/scuttle
  dependency issue
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: GSDensity
  role: Provides compute.mca, compute.nn.edges, run.rwr/run.rwr.list, and compute.cell.label
    to compute per-cell pathway activity likelihood scores via random walk with restart
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Source of the MCA co-embedding approach and parallel benchmarking method for
    per-cell gene-set ranking
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:7c728c09a63fc8b20fdeb6bb07e4aead201873988c2efc28a3b1f691ac1d07e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# pathway-activity-likelihood-scoring

## Summary

Score per-cell pathway activity likelihood (PAL) with GSDensity by running random walk with restart (network propagation) on a nearest-neighbor cell-gene graph built in the MCA co-embedding, using pathway genes as seeds; optionally benchmarked against CelliD's MCA-based ranking.

## When to use

Use when you need single-cell pathway activity scores (PAL) for a gene set in scRNA-seq or spatial transcriptomics data without clustering, e.g., to fetch relevant cells or weight cells for spatial relevance tests.

## When NOT to use

- Input is targeted gene-panel data with limited numbers of genes — the article states GSDensity 'is not suitable for targeted gene data having limited numbers of genes'
- The goal is differential-density testing of gene sets in the MCA space alone (use compute.kld) rather than per-cell activity scoring/fetching of relevant cells
- Running on macOS with an M1 chip, where installation fails due to a CelliD/scuttle dependency issue

## Inputs

- Normalized cell-by-gene count matrix as a Seurat object (e.g., pbmc3k after NormalizeData)
- Gene sets as a named list of gene symbols (e.g., MSigDB GO:BP sets from msigdbr)
- MCA co-embedding (compute.mca output) and nearest-neighbor edge list (compute.nn.edges output, nn.use = 300)

## Outputs

- Per-cell PAL scores (label-propagation probabilities) per gene set, keyed by cell barcode
- Binarized positive/negative cell labels per gene set via compute.cell.label
- Activity-weighted cell densities usable as input to compute.spatial.kld for spatial relevance testing

## How to apply

First co-embed cells and genes into a shared latent space with compute.mca (MCA, as in CelliD), then build a nearest-neighbor cell-gene graph with compute.nn.edges using nn.use = 300. Normalize the input Seurat object with NormalizeData before MCA. Run run.rwr (or run.rwr.list for many gene sets, with parallelization) with the default restart probability of 0.75, passing pathway genes as seeds, to obtain a label-propagation probability (PAL score) per cell; compute.cell.label can binarize cells into positive/negative labels for the gene set. For spatial data, the resulting PAL scores serve as cell weights for the 2D activity-weighted density used by compute.spatial.kld. Rationale: propagation on the shared MCA graph alleviates single-cell sparsity and avoids clustering, and GSDensity's network propagation estimates pathway relevance at single-cell level where CelliD uses a hypergeometric test instead.

## Related tools

- **GSDensity** (Provides compute.mca, compute.nn.edges, run.rwr/run.rwr.list, and compute.cell.label to compute per-cell pathway activity likelihood scores via random walk with restart) — https://github.com/qingnanl/gsdensity
- **CelliD** (Source of the MCA co-embedding approach and parallel benchmarking method for per-cell gene-set ranking) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html

## Examples

```
ce <- compute.mca(object = pbmc3k); el <- compute.nn.edges(coembed = ce, nn.use = 300); cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells)
```

## Evaluation signals

- PAL score vector has one entry per cell barcode and values in [0, 1] (label-propagation probability)
- Fetched positive-labeled cells (compute.cell.label) should map to a coherent subpopulation, e.g., GOBP_B_CELL_ACTIVATION PAL peaks in B-cell clusters of pbmc3k
- Graph construction uses nn.use = 300 and rwr uses restart = 0.75 (default); deviations must be documented
- In spatial benchmarking (SRTsim hotspot/streak/gradient modes), ground-truth spatially relevant gene sets must show significantly higher spatial KLD than random sets (adjusted p < 0.05)
- gsdensity installation must succeed with anticlust 0.6.1 and the updated CelliD Bioconductor version

## Limitations

- Neighborhood size is an important parameter when constructing the cell-gene graph; results are sensitive to it
- MCA embedding is memory-hungry (~25 Gb for a 40,000-cell dataset like hcabm40k)
- PAL-based binarization can yield more than two modalities for some pathways; manual visual inspection of PAL distributions is recommended
- Proper batch correction improves performance, but over- or under-correction may introduce bias
- run.rwr may fail with a Matrix 1.5-0 deprecation error; fix with igraph::igraph.options(sparsematrices = FALSE)

## Evidence

- [intro] Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks: "Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks"
- [discussion] GSDensity uses network propagation to estimate the pathway relevance at single-cell level, while CelliD performs hypergeometric test: "GSDensity uses network propagation to estimate the pathway relevance at single-cell level, while CelliD performs hypergeometric test"
- [readme] cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells): "cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells)"
- [readme] el <- compute.nn.edges(coembed = ce, nn.use = 300): "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
- [discussion] such as what we did in treating the pathway activity as cell weights for spatial relevant pathway identification: "such as what we did in treating the pathway activity as cell weights for spatial relevant pathway identification"
