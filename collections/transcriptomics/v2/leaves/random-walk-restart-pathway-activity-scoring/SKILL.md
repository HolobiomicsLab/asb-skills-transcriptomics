---
name: random-walk-restart-pathway-activity-scoring
description: Use when a gene set's spatial/density relevance has been established
  (compute.kld p.adj < 0.05) and per-cell pathway activity scores or positive/negative
  cell labels are needed for downstream spatial or classification analysis.
when_to_use_negative:
- Data are targeted-panel spatial transcriptomics with limited gene numbers — the
  article states GSDensity 'is not suitable for targeted gene data having limited
  numbers of genes'
- Gene sets that failed the coordination pre-filter (compute.kld adjusted p >= 0.05),
  since over 98% of non-coordinated gene sets yield spurious binarizations
- Non-spatial single-cell clustering workflows where only cluster-level enrichment
  is needed and per-cell labels are not required
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GSDensity
  role: Provides compute.nn.edges, run.rwr, run.rwr.list, and compute.cell.label for
    graph construction and RWR pathway activity scoring
  repo: https://github.com/qingnanl/gsdensity
- name: igraph
  role: Underlying graph engine for the nearest-neighbor graph and random walk propagation;
    igraph.options(sparsematrices = FALSE) is a documented workaround for Matrix deprecation
    errors
- name: future.apply
  role: Parallel backend for run.rwr.list when scoring many gene sets across the six
    cancer Visium datasets
- name: Seurat
  role: Loads and normalizes the 10X Visium / scRNA-seq count matrices that supply
    cells and genes to the co-embedding and RWR steps
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:5c7591f793e1639c4f2606086d4c66a6845840c168cb0a2ab63a3e0efd555b42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - GSDensity
  - Seurat
  - future.apply
  - igraph
  - msigdbr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Random-walk-restart pathway activity scoring

## Summary

GSDensity's single-cell pathway activity scoring step: a random walk with restart (RWR) is propagated over a nearest-neighbor cell-gene graph built in the MCA latent space, using pathway genes as seeds, to yield per-cell pathway relevance probabilities without clustering.

## When to use

Use when a gene set's spatial/density relevance has been established (compute.kld p.adj < 0.05) and per-cell pathway activity scores or positive/negative cell labels are needed for downstream spatial or classification analysis.

## When NOT to use

- Data are targeted-panel spatial transcriptomics with limited gene numbers — the article states GSDensity 'is not suitable for targeted gene data having limited numbers of genes'
- Gene sets that failed the coordination pre-filter (compute.kld adjusted p >= 0.05), since over 98% of non-coordinated gene sets yield spurious binarizations
- Non-spatial single-cell clustering workflows where only cluster-level enrichment is needed and per-cell labels are not required

## Inputs

- MCA co-embedding (cells and genes) from compute.mca
- Nearest-neighbor cell-gene edge list from compute.nn.edges (nn.use = 300)
- Named list of gene sets (e.g., msigdbr GO_BP, KEGG, BIOCARTA gene symbols)
- Vector of cell names from the normalized Seurat object

## Outputs

- Named vector of per-cell label-propagation probabilities for each gene set
- Binarized positive/negative cell labels via compute.cell.label
- Pathway activity weight matrix (per gene set × per spot) for compute.spatial.kld.df spatial testing

## How to apply

First co-embed cells and genes with compute.mca, then build the cell-gene nearest-neighbor graph with compute.nn.edges using nn.use = 300 (neighborhood size is an important parameter per the discussion). For a single gene set, run run.rwr with the gene set as seeds over the edge list; for many gene sets (e.g., MSigDB GO_BP, KEGG, BIOCARTA collections), use run.rwr.list with restart = 0.75 and parallel execution via future.apply. Convert the returned propagation probabilities to binary positive/negative cell labels with compute.cell.label, or keep the continuous scores as cell weights for the weighted-KDE spatial relevance test (compute.spatial.kld.df). The rationale is that network propagation through the shared latent graph alleviates single-cell sparsity and evaluates pathway relevance at single-cell resolution without pre-clustering, unlike CelliD's hypergeometric test or ssGSEA/AUCell scores.

## Related tools

- **GSDensity** (Provides compute.nn.edges, run.rwr, run.rwr.list, and compute.cell.label for graph construction and RWR pathway activity scoring) — https://github.com/qingnanl/gsdensity
- **igraph** (Underlying graph engine for the nearest-neighbor graph and random walk propagation; igraph.options(sparsematrices = FALSE) is a documented workaround for Matrix deprecation errors)
- **future.apply** (Parallel backend for run.rwr.list when scoring many gene sets across the six cancer Visium datasets)
- **Seurat** (Loads and normalizes the 10X Visium / scRNA-seq count matrices that supply cells and genes to the co-embedding and RWR steps)

## Examples

```
cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells)
```

## Evaluation signals

- The edge list from compute.nn.edges must be built with nn.use = 300 on the compute.mca co-embedding
- run.rwr.list must use restart = 0.75 and return a named probability vector for every cell supplied
- compute.cell.label on the probabilities should partition cells into positive and negative labels; visual inspection of the binarized labels on UMAP (DimPlot) or spatial map should align with known pathway-relevant populations (e.g., B-cell activation labeling B-cell clusters in pbmc3k)
- Downstream compute.spatial.kld.df at adjusted p < 0.05 should only be run on gene sets that passed the compute.kld coordination filter (p.adj < 0.05, gene.set.cutoff = 3, n.times = 100)
- For the pan-cancer task, the cross-dataset intersection of spatially significant pathways must contain exactly 34 pathways

## Limitations

- Neighborhood size (nn.use) is an important parameter when constructing the cell-gene graph; poor choices degrade label propagation
- MCA embedding step requires large memory (~25 Gb for a 40,000-cell dataset like hcabm40k)
- AUCell is faster than GSDensity when the number of cells is small, though GSDensity scales better
- Improper batch correction (over- or under-correction) can bias the embedding used for the graph and thus the RWR scores
- Not suitable for targeted gene data with limited numbers of genes; imputation is suggested but not implemented
- MacOS M1 installation may fail due to a CelliD/scuttle dependency; anticlust must be pinned to an older version (e.g., 0.6.1)

## Evidence

- [intro] Pathway activity for each cell via network propagation: "Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks"
- [methods] RWR graph construction and scoring calls: "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
- [methods] Single gene set RWR invocation: "cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells)"
- [discussion] RWR scalability vs CelliD: "GSDensity uses network propagation to estimate the pathway relevance at single-cell level, while CelliD performs hypergeometric test"
- [readme] README quick-start RWR workflow: "# get label propagation probability for each cell of a gene set 'GOBP_B_CELL_ACTIVATION'
cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells)"
