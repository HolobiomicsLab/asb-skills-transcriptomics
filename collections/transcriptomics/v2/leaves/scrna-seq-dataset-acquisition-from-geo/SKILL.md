---
name: scrna-seq-dataset-acquisition-from-geo
description: Use when a benchmark task requires the seven annotated real scRNA-seq
  datasets referenced by GEO accessions GSE84133 (Baron pancreas) and GSE65525 (Klein
  mouse ES) plus human PBMC, kidney, hypothalamus, mouse kidney, and turtle brain,
  each with ground-truth cell-type labels for ARI computation.
when_to_use_negative:
- The task is to run or evaluate clustering (e.g., with run_scace) on data already
  loaded locally — use the scAce execution skill instead of re-acquiring data.
- A dataset lacks provided cell-type annotations, since the benchmark's ARI-vs-annotations
  comparison cannot be computed.
- The analysis targets simulated rather than the seven annotated real scRNA-seq datasets.
edam_topics: []
tools:
- name: scanpy
  role: Wraps each retrieved GEO dataset as sc.AnnData and performs the standard preprocessing
    (filtering, normalize_per_cell, log1p, scale) applied in the benchmark
- name: scAce
  role: 'Downstream consumer: the acquired annotated datasets are the inputs scAce
    and nine competing methods cluster to produce ARI values'
  repo: https://github.com/sldyns/scAce
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:225c2eff7c103a76f5fea03cf4040bb20dd6315173e4a20a324d85c72ecec792
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# scrna-seq-dataset-acquisition-from-geo

## Summary

Retrieves the annotated scRNA-seq datasets used by the scAce benchmark (human pancreas, mouse ES, human PBMC, human kidney, mouse hypothalamus, mouse kidney, turtle brain) from GEO using the accession IDs listed in Supplementary Table S4, so that clustering methods can be evaluated against provided cell-type annotations with mean ARI.

## When to use

Use when a benchmark task requires the seven annotated real scRNA-seq datasets referenced by GEO accessions GSE84133 (Baron pancreas) and GSE65525 (Klein mouse ES) plus human PBMC, kidney, hypothalamus, mouse kidney, and turtle brain, each with ground-truth cell-type labels for ARI computation.

## When NOT to use

- The task is to run or evaluate clustering (e.g., with run_scace) on data already loaded locally — use the scAce execution skill instead of re-acquiring data.
- A dataset lacks provided cell-type annotations, since the benchmark's ARI-vs-annotations comparison cannot be computed.
- The analysis targets simulated rather than the seven annotated real scRNA-seq datasets.

## Inputs

- GEO accession IDs from Supplementary Table S4 (e.g., GSE84133, GSE65525)
- GEO supplementary count-matrix files and cell-type annotation metadata for the seven datasets

## Outputs

- Seven AnnData objects (raw count matrices plus cell-type annotations) for human pancreas, mouse ES, human PBMC, human kidney, mouse hypothalamus, mouse kidney, and turtle brain
- A dataset manifest mapping each dataset to its GEO accession and annotation source

## How to apply

Consult the paper's Supplementary Table S4 to enumerate the seven annotated datasets and their GEO accession IDs (e.g., human pancreas, Baron et al. 2016, GSE84133; mouse ES, Klein et al. 2015, GSE65525). Download each dataset's count matrix and cell-type annotation from its GEO entry, verifying that both expression values and per-cell labels are obtainable. Assemble each dataset as an AnnData object (sc.AnnData) so the downstream scanpy preprocessing chain (filter_genes min_cells=3, filter_cells min_genes=200, normalize_per_cell, log1p, scale) can be applied uniformly. Confirm dataset completeness before benchmarking: all seven must be present so that per-dataset ARI values can be averaged into a mean-ARI ranking across the ten methods.

## Related tools

- **scanpy** (Wraps each retrieved GEO dataset as sc.AnnData and performs the standard preprocessing (filtering, normalize_per_cell, log1p, scale) applied in the benchmark)
- **scAce** (Downstream consumer: the acquired annotated datasets are the inputs scAce and nine competing methods cluster to produce ARI values) — https://github.com/sldyns/scAce

## Examples

```
accessions = {'pancreas': 'GSE84133', 'mouse_ES': 'GSE65525'}  # + PBMC, kidney, hypothalamus, mouse kidney, turtle brain per Supplementary Table S4
```

## Evaluation signals

- All seven datasets from Supplementary Table S4 (pancreas, mouse ES, PBMC, human kidney, hypothalamus, mouse kidney, turtle brain) are downloaded and loadable.
- GEO accessions match the paper's citations: Baron et al. 2016 = GSE84133; Klein et al. 2015 = GSE65525.
- Each AnnData object contains both an expression matrix and per-cell ground-truth labels usable for ARI computation.
- Datasets survive the preprocessing chain (filter_genes min_cells=3, filter_cells min_genes=200) without losing required cell-type annotations.
- Per-dataset ARI values can be averaged across all seven datasets to yield a mean-ARI row for each of the ten methods.

## Limitations

- Dataset availability depends on GEO entries remaining accessible and containing both raw counts and annotations as described in Supplementary Table S4.
- The benchmark assumes provided cell-type annotations exist; datasets without annotations cannot contribute to the mean-ARI ranking.
- Human pancreas and mouse ES datasets are pinned to specific studies (Baron et al. 2016; Klein et al. 2015); substituting other versions may change ARI results.

## Evidence

- [supplementary] Download the seven annotated scRNA-seq datasets listed in Supplementary Table S4: human pancreas (Baron et al. 2016, GSE84133), mouse ES (Klein et al. 2015, GSE65525): [evidence span withheld — anchor status: not_found]
- [other] Compute the Adjusted Rand Index of each method's predicted labels against the provided cell-type annotations for every dataset: [evidence span withheld — anchor status: not_found]
- [readme] Load the data to be analyzed: import scanpy as sc, adata = sc.AnnData(data): "adata = sc.AnnData(data)"
