---
name: pbmc-dataset-loading-with-scanpy
description: Use when benchmarking single-cell clustering on the 10x Genomics PBMC
  dataset (Zheng et al. 2017) or generating scDesign2 simulations fitted to PBMC data;
  input is raw gene-by-cell counts.
when_to_use_negative:
- Input data is not single-cell gene expression counts (e.g. bulk RNA-seq, proteomics,
  or already-clustered label tables)
- The analysis target is a different tissue/dataset rather than the PBMC benchmark
  used to fit scDesign2 parameters for the scAce evaluation
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3316
tools:
- name: scanpy
  role: Constructs the AnnData object and performs basic gene/cell filtering (min_cells=3,
    min_genes=200) on the PBMC count data
  repo: https://github.com/sldyns/scAce
- name: scDesign2
  role: Fits per-cell-type gene expression parameters to the loaded PBMC data to simulate
    the synthetic benchmark dataset
- name: Python 3.8
  role: Runtime environment for the scanpy-based loading and filtering workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:a1a30d8a685316a619a7a4a50c6114dba3904dac33539061e686112e237f90ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Python 3.8
  - scDesign2
  - scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# pbmc-dataset-loading-with-scanpy

## Summary

Load the public 10x Genomics PBMC gene expression count data (Zheng et al. 2017) used to benchmark scAce into a scanpy AnnData object, from which simulated scDesign2 datasets and clustering runs are derived.

## When to use

Use when benchmarking single-cell clustering on the 10x Genomics PBMC dataset (Zheng et al. 2017) or generating scDesign2 simulations fitted to PBMC data; input is raw gene-by-cell counts.

## When NOT to use

- Input data is not single-cell gene expression counts (e.g. bulk RNA-seq, proteomics, or already-clustered label tables)
- The analysis target is a different tissue/dataset rather than the PBMC benchmark used to fit scDesign2 parameters for the scAce evaluation

## Inputs

- 10x Genomics PBMC single-cell RNA-seq gene expression count matrix (Zheng et al. 2017)

## Outputs

- scanpy AnnData object containing the PBMC count matrix
- Filtered AnnData with genes retained at min_cells=3 and cells retained at min_genes=200
- Simulated scDesign2 count matrix of 16653 genes x 1150 cells across 5 cell types with ground-truth labels (e.g. scdesign2_pbmc_simulated_counts.csv)

## How to apply

Retrieve the public 10x Genomics PBMC gene expression counts (Zheng et al. 2017) and construct a scanpy AnnData object from the count matrix (sc.AnnData(data)). Before scAce clustering or scDesign2 parameter fitting, apply the paper's basic filtering: sc.pp.filter_genes(adata, min_cells=3) to keep informative genes and sc.pp.filter_cells(adata, min_genes=200). Keep a copy of the filtered data in adata.raw since the pre-processing retains counts for downstream use. When generating the scDesign2 simulation, fit the model per cell type to the PBMC data and simulate 5 cell types at 600/200/200/100/50 cells (1150 cells total, rarest type <5%).

## Related tools

- **scanpy** (Constructs the AnnData object and performs basic gene/cell filtering (min_cells=3, min_genes=200) on the PBMC count data) — https://github.com/sldyns/scAce
- **scDesign2** (Fits per-cell-type gene expression parameters to the loaded PBMC data to simulate the synthetic benchmark dataset)
- **Python 3.8** (Runtime environment for the scanpy-based loading and filtering workflow)

## Examples

```
import scanpy as sc; adata = sc.AnnData(data); sc.pp.filter_genes(adata, min_cells=3); sc.pp.filter_cells(adata, min_genes=200)
```

## Evaluation signals

- The AnnData object is constructed successfully with a genes-by-cells count matrix from the PBMC source
- After filtering with min_cells=3 and min_genes=200, no gene is observed in fewer than 3 cells and no cell expresses fewer than 200 genes
- adata.raw contains a copy of the filtered data prior to normalization/scaling
- For the simulation path: output contains exactly 1150 cells (600+200+200+100+50) with a label column, and the rarest cell type is <5% of cells
- Cluster labels from downstream runs land in adata.obs['scace_cluster'] and embeddings in adata.obsm['scace_emb']

## Limitations

- Filtering parameters (min_cells=3, min_genes=200) are tuned for the PBMC benchmark; other datasets may need different thresholds
- The simulated dataset (16653 genes x 1150 cells, 5 types with a rare type of 50 cells) is fitted to PBMC marginals, so it reflects only that data's expression structure

## Evidence

- [methods] AnnData construction from count matrix: "import scanpy as sc

adata = sc.AnnData(data)"
- [methods] Basic filtering thresholds: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [readme] README pre-processing including raw copy: "# Basic filtering
sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)

adata.raw = adata.copy()"
- [results] Simulated dataset specifications from benchmarking card: [evidence span withheld — anchor status: not_found]
