---
name: graph-based-batch-correction-bbknn
description: Use when integrating scRNA-seq matrices from multiple batches (e.g.,
  GSE70630 oligodendroglioma samples MGH36-MGH97) and you need a fast, graph-based
  alternative that builds connections between analogous cells across batches without
  altering counts or PCA space.
when_to_use_negative:
- Input is gene-centric scaled data rather than raw UMI/FPKM/TPM counts — the README
  warns matrices 'should not be gene-centric scaled data'
- There is only a single batch or no BATCH label vector, so no per-batch neighbor
  identification is possible
- 'You need actual expression-value correction rather than a neighbor graph: BBKNN
  creates connections between cells across batches ''without altering the counts or
  PCA space'''
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BBKNN
  role: 'Batch-effect removal enhancement: builds the batch-balanced KNN graph used
    for UMAP generation in the BEER pipeline benchmark'
  repo: https://github.com/Teichlab/bbknn
- name: BEER
  role: 'Host pipeline: BEER.R provides BEER.combat (ComBat PC adjustment) and BEER.bbknn
    (R wrapper calling BBKNN with NB and NT parameters)'
  repo: https://github.com/jumphone/BEER
- name: ComBat
  role: Adjusts PCs via BEER.combat before the BBKNN graph step
  repo: https://github.com/jumphone/BEER
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41421-019-0114-x
    title: A novel approach to remove the batch effect of single-cell data
  source_content_hash: sha256:ab762070b36a209ef4742ce3c884f27603c32130d9d9133c9363be121f6961fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Teichlab/bbknn
  tools:
  - BBKNN
  - BEER
  - ComBat
  - fastMNN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# Graph-based batch correction with BBKNN

## Summary

Applies BBKNN (batch balanced KNN), a graph-based batch-effect removal tool from the scanpy ecosystem, to reduce batch effects in single-cell data by identifying nearest neighbors within each batch separately and merging them into a batch-balanced neighbor graph, used in the BEER workflow as a 'batch-effect removal enhancement' step after ComBat-adjusted PCs.

## When to use

Use when integrating scRNA-seq matrices from multiple batches (e.g., GSE70630 oligodendroglioma samples MGH36-MGH97) and you need a fast, graph-based alternative that builds connections between analogous cells across batches without altering counts or PCA space.

## When NOT to use

- Input is gene-centric scaled data rather than raw UMI/FPKM/TPM counts — the README warns matrices 'should not be gene-centric scaled data'
- There is only a single batch or no BATCH label vector, so no per-batch neighbor identification is possible
- You need actual expression-value correction rather than a neighbor graph: BBKNN creates connections between cells across batches 'without altering the counts or PCA space'

## Inputs

- Combined UMI count matrix (genes x cells) with rownames as gene names and unique colnames per batch, e.g. after .simple_combine of GSE70630 batch matrices
- BATCH character vector with length ncol(DATA) labeling each cell's batch
- BEER output object (mybeer) providing the Seurat object mybeer$seurat and selected PCs PCUSE=mybeer$select
- ComBat-adjusted PCs from BEER.combat(pbmc)

## Outputs

- Batch-balanced neighbor graph and UMAP cell embeddings (assigned to pbmc@reductions$umap@cell.embeddings)
- UMAP/DimPlot visualizations colored by batch for qualitative batch-mixing assessment
- Per-method batch-mixing and cell-type-clustering scores for the comparison table (e.g., benchmark_table.csv)

## How to apply

Install BBKNN (pip3 install bbknn or conda install -c bioconda bbknn) and set up Python via reticulate in R. In the BEER workflow, BBKNN is used for batch-effect removal enhancement: first adjust PCs with ComBat via BEER.combat, then run BEER.bbknn(pbmc, PCUSE, NB=3, NT=10) on the PC subspace selected from mybeer$cor vs mybeer$lcor. BBKNN takes each cell and finds a smaller k nearest neighbors in each batch separately (rather than the dataset as a whole), then merges these per-batch neighbors into a final neighbor list, which is why it helps with cell-type imbalanced data — connections between analogous cells in different batches are created without changing counts or PCA space. The returned UMAP embeddings overwrite pbmc@reductions$umap@cell.embeddings for downstream DimPlot visualization and clustering-based scoring.

## Related tools

- **BBKNN** (Batch-effect removal enhancement: builds the batch-balanced KNN graph used for UMAP generation in the BEER pipeline benchmark) — https://github.com/Teichlab/bbknn
- **BEER** (Host pipeline: BEER.R provides BEER.combat (ComBat PC adjustment) and BEER.bbknn (R wrapper calling BBKNN with NB and NT parameters)) — https://github.com/jumphone/BEER
- **ComBat** (Adjusts PCs via BEER.combat before the BBKNN graph step) — https://github.com/jumphone/BEER

## Examples

```
pbmc=BEER.combat(pbmc); umap=BEER.bbknn(pbmc, PCUSE, NB=3, NT=10); pbmc@reductions$umap@cell.embeddings=umap
```

## Evaluation signals

- UMAP colored by batch shows mixed batches rather than batch-separated islands (DimPlot group.by='batch')
- Cell-type structure is preserved: kmeans clusters (e.g., centers=30) on the UMAP embeddings separate cell types, enabling cell-type-clustering scores
- Neighbors are balanced per batch: each cell receives NB neighbors from each batch (NB=3 in the BEER call), not dominated by one batch
- Benchmark comparison table ranks BBKNN against BEER, ComBat, Seurat CCA, and fastMNN on identical combined data and QC-filtered cells (500-4000 sequenced genes)
- BEER.bbknn returns a UMAP embedding matrix that successfully overwrites pbmc@reductions$umap@cell.embeddings without errors

## Limitations

- BBKNN is a graph-based method: it does not alter counts or the PCA space, so it cannot correct expression values themselves
- BBKNN depends on external Python packages (Cython, numpy, scipy, annoy, pynndescent, umap-learn, scikit-learn) and requires reticulate/Python setup in the R BEER workflow
- In the cell-type imbalanced benchmark of GSE70630 oligodendroglioma data, BBKNN underperformed BEER for batch-effect removal

## Evidence

- [abstract] A graph-based method named batch balanced KNN (BBKNN) reduces batch-effect by creating connections between analogous cells in different batches: [evidence span withheld — anchor status: not_found]
- [methods] BEER.bbknn invocation with parameters: "umap=BEER.bbknn(pbmc, PCUSE, NB=3, NT=10)"
- [readme] BBKNN finds neighbors per batch separately without altering counts or PCA space: [evidence span withheld — anchor status: external_doc]
- [readme] BBKNN installation instruction for BEER enhancement: "For batch-effect removal enhancement, please install BBKNN: https://github.com/Teichlab/bbknn"
