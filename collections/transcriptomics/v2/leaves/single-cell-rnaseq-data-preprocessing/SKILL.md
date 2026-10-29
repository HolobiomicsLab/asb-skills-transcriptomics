---
name: single-cell-rnaseq-data-preprocessing
description: Use when a raw single-cell gene expression matrix needs filtering (min_cells=3,
  min_genes=200), total-count normalization, log1p and scaling before feeding into
  scAce or computing ARI/NMI clustering benchmarks.
when_to_use_negative:
- Input is already normalized, scaled, or a reduced-dimension embedding rather than
  raw per-cell gene counts
- Data are not single-cell gene expression (e.g. bulk RNA-seq, spatial transcriptomics
  at spot resolution, or proteomics) where scanpy min_cells/min_genes filtering and
  total-count normalization are inappropriate
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: AnnData container and preprocessing functions (filter_genes, filter_cells,
    normalize_per_cell, log1p, scale) used for all filtering and normalization steps
  repo: https://github.com/sldyns/scAce
- name: scAce
  role: Downstream consumer of the preprocessed AnnData; run_scace adds scace_cluster
    labels and scace_emb embeddings
  repo: https://github.com/sldyns/scAce
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:91b7558eb73454c3bb55d2efd6f1f28a3049b62c3c52ac8125982a85b71e744c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-rnaseq-data-preprocessing

## Summary

Standard scRNA-seq preprocessing pipeline (scanpy-based filtering, total-count normalization, log1p, scaling) that prepares a gene-expression AnnData for scAce's VAE pre-training and adaptive clustering, following the exact thresholds used in the scAce benchmark.

## When to use

Use when a raw single-cell gene expression matrix needs filtering (min_cells=3, min_genes=200), total-count normalization, log1p and scaling before feeding into scAce or computing ARI/NMI clustering benchmarks.

## When NOT to use

- Input is already normalized, scaled, or a reduced-dimension embedding rather than raw per-cell gene counts
- Data are not single-cell gene expression (e.g. bulk RNA-seq, spatial transcriptomics at spot resolution, or proteomics) where scanpy min_cells/min_genes filtering and total-count normalization are inappropriate

## Inputs

- Raw single-cell gene expression count matrix loadable into a scanpy AnnData object
- Optional ground-truth cell-type labels for ARI/NMI evaluation

## Outputs

- Preprocessed AnnData with filtered genes/cells, normalized and scaled expression, stored raw counts, and a scale_factor column in adata.obs
- AnnData ready for run_scace input

## How to apply

Load the expression data into a scanpy AnnData object (adata = sc.AnnData(data)). Apply basic filtering with sc.pp.filter_genes(adata, min_cells=3) and sc.pp.filter_cells(adata, min_genes=200), then store a copy in adata.raw for later use. Total-count normalize with sc.pp.normalize_per_cell, recording the scale factor as adata.obs['scale_factor'] = adata.obs.n_counts / adata.obs.n_counts.median() (equivalently n_counts/median(n_counts) as in the simulated-data benchmark workflow), then log1p-transform with sc.pp.log1p and scale with sc.pp.scale. This preprocessed AnnData is the direct input to run_scace, which returns cluster labels in adata.obs['scace_cluster'] and cell embeddings in adata.obsm['scace_emb']; downstream ARI/NMI comparisons against ground-truth cell-type labels (e.g. on the five-cell-type simulated dataset) depend on using identical preprocessing across all ten methods.

## Related tools

- **scanpy** (AnnData container and preprocessing functions (filter_genes, filter_cells, normalize_per_cell, log1p, scale) used for all filtering and normalization steps) — https://github.com/sldyns/scAce
- **scAce** (Downstream consumer of the preprocessed AnnData; run_scace adds scace_cluster labels and scace_emb embeddings) — https://github.com/sldyns/scAce

## Examples

```
import scanpy as sc; adata = sc.AnnData(data); sc.pp.filter_genes(adata, min_cells=3); sc.pp.filter_cells(adata, min_genes=200); adata.raw = adata.copy(); sc.pp.normalize_per_cell(adata); sc.pp.log1p(adata); sc.pp.scale(adata)
```

## Evaluation signals

- Genes detected in fewer than 3 cells and cells expressing fewer than 200 genes are removed (min_cells=3, min_genes=200 filters applied)
- adata.raw holds a copy of the filtered-but-unnormalized counts; adata.obs['scale_factor'] equals adata.obs.n_counts / adata.obs.n_counts.median()
- Expression matrix after sc.pp.scale is log1p-transformed and z-scored across cells
- The same preprocessed AnnData, when passed to run_scace, yields adata.obs['scace_cluster'] and adata.obsm['scace_emb'] without pre-specifying cluster number
- On the simulated five-cell-type dataset, downstream scAce clustering reaches ARI = 1 and recovers the correct cluster number

## Limitations

- Thresholds min_cells=3 and min_genes=200 are the scAce benchmark defaults; other data modalities or sparse/low-depth datasets may need adjusted cutoffs
- sc.pp.scale operates on the log1p-normalized matrix, so re-running the pipeline out of order (scaling before normalization) will not reproduce the reported results
- Methods requiring a predetermined cluster number (e.g. the nine comparison methods in the simulated benchmark) need the true cell-type count passed separately; only scAce infers it adaptively

## Evidence

- [readme] Basic filtering and normalization thresholds: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [readme] Normalization and scale factor recording: "sc.pp.normalize_per_cell(adata)
adata.obs['scale_factor'] = adata.obs.n_counts / adata.obs.n_counts.median()"
- [supplementary] Scale factor formula from benchmark workflow: [evidence span withheld — anchor status: not_found]
- [readme] Output structure of run_scace: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`"
