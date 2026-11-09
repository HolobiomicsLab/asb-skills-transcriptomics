---
name: differential-gene-expression-analysis-of-novel-cluster
description: Use when a Cell BLAST query leaves query cells unassigned (rejected under
  the default hit cutoff), indicating a putative novel cell type that must be validated
  by differential expression of marker genes.
when_to_use_negative:
- All query cells receive confident reference annotations under the default cutoff
  — there is no rejected cluster to characterize
- The reference dataset is too small for the DIRECTi model (fewer than ~1000 cells
  may suffer from under-training), making rejection calls and downstream DE unreliable
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Cell BLAST
  role: Queries the dataset against the curated reference and identifies rejected
    cells forming the putative novel cluster; its documented public functions perform
    the downstream differential expression analysis
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model trained on the reference (via Cell_BLAST.directi.fit_DIRECTi)
    to learn low-dimensional cell embeddings used by BLAST to score cell-to-cell similarity
    as posterior-distribution distance
  repo: https://github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:6c52ca83f5e383c4b6dfbf88229571e7c952579d3570733fa3df27ab824c0567
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - Cell BLAST
  - DIRECTi
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# differential-gene-expression-analysis-of-novel-cluster

## Summary

Characterizes an unassigned ('novel') query cell cluster left over after Cell BLAST querying — cells rejected under the default cutoff because no suitable reference annotation exists — by running differential expression on the rejected cells to confirm their identity via marker gene enrichment (e.g. Cftr, Foxi1, Ascl3 for pulmonary ionocytes).

## When to use

Use when a Cell BLAST query leaves query cells unassigned (rejected under the default hit cutoff), indicating a putative novel cell type that must be validated by differential expression of marker genes.

## When NOT to use

- All query cells receive confident reference annotations under the default cutoff — there is no rejected cluster to characterize
- The reference dataset is too small for the DIRECTi model (fewer than ~1000 cells may suffer from under-training), making rejection calls and downstream DE unreliable

## Inputs

- Query scRNA-seq dataset in h5ad format (e.g. Plasschaert airway epithelium)
- Curated reference scRNA-seq dataset in h5ad format with the target cell type excluded (e.g. ionocyte-removed Montoro airway epithelium)
- Fitted DIRECTi model and Cell_BLAST.blast.BLAST object with rejected (unassigned) query cells identified under the default cutoff

## Outputs

- Differential expression table for the rejected novel cluster versus remaining cells
- Set of enriched marker genes confirming cluster identity (e.g. Cftr, Foxi1, Ascl3)
- Validation metrics: marker-based accuracy (MBA) of transferred annotations and count of rejected cells

## How to apply

First query the labeled cells (e.g. the Plasschaert airway epithelium dataset) against a curated reference (the Montoro airway epithelium panel with the cell type of interest removed) using a DIRECTi model trained via Cell_BLAST.directi.fit_DIRECTi and a Cell_BLAST.blast.BLAST object, which scores cell-to-cell similarity as the distance between posterior distributions. Collect the query cells rejected under the default cutoff — in the ionocyte rediscovery case, 14 of 19 Plasschaert ionocytes were rejected, yielding an MBA of 0.873 — as the putative novel cluster. Then apply the documented Cell_BLAST public differential expression functions to this rejected cluster versus the remaining query cells. Validate by checking that the DE table is enriched for the expected markers (Cftr, Foxi1, Ascl3 for ionocytes), confirming the rejected cluster is the known-but-omitted cell type rather than an artifact.

## Related tools

- **Cell BLAST** (Queries the dataset against the curated reference and identifies rejected cells forming the putative novel cluster; its documented public functions perform the downstream differential expression analysis) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model trained on the reference (via Cell_BLAST.directi.fit_DIRECTi) to learn low-dimensional cell embeddings used by BLAST to score cell-to-cell similarity as posterior-distribution distance) — https://github.com/gao-lab/Cell_BLAST

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST && python -c "import Cell_BLAST as cb; print(cb.__version__)"
```

## Evaluation signals

- Marker-based accuracy (MBA) of transferred annotations reproduces the reported value of 0.873 for the ionocyte-removed reference
- Rejected (unassigned) cell count matches the expected 14 of 19 Plasschaert ionocytes under the default cutoff
- DE table for the rejected cluster contains Cftr, Foxi1, and Ascl3 as enriched marker genes, confirming ionocyte identity
- import Cell_BLAST as cb succeeds and cb.__version__ prints, confirming the pip-installed package is usable
- Input datasets are in h5ad format (legacy h5 files must first pass through cb.data.h5_to_h5ad)

## Limitations

- The Cell BLAST model is over-parameterized and requires a reasonable number of samples to train properly; datasets with fewer than 1000 cells may suffer from under-training issues, while a cell number of 3000 is generally sufficient
- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition, which could distort cluster rejection and DE results
- Differential expression here relies on the documented Cell_BLAST public functions; conversion of legacy h5 reference data to h5ad may be required first

## Evidence

- [abstract] Herein, we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric.: "Herein, we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric."
- [intro] we demonstrate the effectiveness of Cell BLAST in annotating discrete cell types and continuous cell differentiation potential, as well as identifying novel cell types: "we demonstrate the effectiveness of Cell BLAST in annotating discrete cell types and continuous cell differentiation potential, as well as identifying novel cell types"
- [methods] cb.data.h5_to_h5ad(inputfilename, outputfilename): "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [readme] Annotations in reference cells, e.g. cell type, can then be transfered to query cells based on cell-to-cell similarities.: "Annotations in reference cells, e.g. cell type, can then be transfered to query cells based on cell-to-cell similarities."
- [discussion] Cell BLAST is an accurate and robust querying algorithm for heterogeneous single-cell transcriptome datasets.: "Cell BLAST is an accurate and robust querying algorithm for heterogeneous single-cell transcriptome datasets."
