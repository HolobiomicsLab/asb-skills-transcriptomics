---
name: gene-age-weighted-expression-analysis
description: Use when single-cell RNA-seq AnnData objects need an evolutionary age
  metric (TEI or partial TEI profiles) per cell, especially for large datasets (~90,000+
  up to 1e6 cells) where per-cell computation must stay tractable.
when_to_use_negative:
- Input expression data is dense or not stored as a sparse CSR AnnData matrix — the
  scalability claim rests on sparse-matrix calculation
- No gene age / orthomap is available or extractable for the query species (TEI weights
  require phylostrata per gene)
- The goal is bulk-RNA stage-level analysis with only a small number of samples rather
  than per-cell TEI across large scRNA datasets
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0084
tools:
- name: oggmap
  role: Provides orthomap2tei.get_tei, which computes per-cell TEI as the expression-weighted
    mean of phylostrata on sparse AnnData; also supplies of2orthomap to extract gene
    ages from OrthoFinder results
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Toolkit whose AnnData scRNA objects and environment host the TEI calculation
    and downstream visualization
- name: OrthoFinder
  role: Upstream orthogroup inference whose results are converted into the orthomap
    (gene ages per orthogroup) used as TEI weights
  repo: https://github.com/davidemms/OrthoFinder
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:29e83a81b9af0cd5a4f2b40ffbf5bff91b61dba70717e60e253d2de1d2d09c1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - OrthoFinder
  - Python
  - Scanpy
  - oggmap
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-age-weighted-expression-analysis

## Summary

Computes the transcriptome evolutionary index (TEI) — an expression-weighted mean of gene phylostrata per cell following Domazet-Lošo and Tautz 2010 — on single-cell RNA data using oggmap's sparse-matrix get_tei implementation, which scales to millions of cells.

## When to use

Use when single-cell RNA-seq AnnData objects need an evolutionary age metric (TEI or partial TEI profiles) per cell, especially for large datasets (~90,000+ up to 1e6 cells) where per-cell computation must stay tractable.

## When NOT to use

- Input expression data is dense or not stored as a sparse CSR AnnData matrix — the scalability claim rests on sparse-matrix calculation
- No gene age / orthomap is available or extractable for the query species (TEI weights require phylostrata per gene)
- The goal is bulk-RNA stage-level analysis with only a small number of samples rather than per-cell TEI across large scRNA datasets

## Inputs

- Scanpy AnnData object with a sparse (CSR) scRNA expression matrix (e.g. ~90,000 nematode cells, or tiled up to >=1,000,000 cells)
- orthomap: gene age (phylostratum) assignment per orthogroup/gene for the query species (e.g. Sun et al. 2021 nematode gene ages, or oggmap-extracted from OrthoFinder/eggNOG results)

## Outputs

- Per-cell transcriptome evolutionary index (TEI) values added to the AnnData object
- Partial TEI profiles per cell
- Mean/relative expression per gene age class
- Optional runtime benchmark table (e.g. tei_runtime_benchmark.csv) with per-tile-size wall-clock time and peak memory

## How to apply

First obtain an orthomap (gene ages per orthogroup) for the query species, e.g. via oggmap's of2orthomap from OrthoFinder results or from a published gene-age table such as Sun et al. 2021 for nematode. Match gene/transcript IDs and their gene age classes to the scRNA dataset so each gene carries a phylostratum value. Keep the expression matrix in sparse (CSR) format inside the Scanpy AnnData object, then run orthomap2tei.get_tei, which computes TEI as the expression-weighted mean of phylostrata per cell; the sparse-matrix implementation is what makes per-cell TEI tractable at scale. If the dataset is small, it can be tiled/replicated to benchmark tractability, recording wall-clock time and peak memory per tile size (e.g. a tei_runtime_benchmark.csv). Partial TEI profiles and per-gene-age-class mean/relative expression can be computed and visualized with the same Scanpy-coupled oggmap functions.

## Related tools

- **oggmap** (Provides orthomap2tei.get_tei, which computes per-cell TEI as the expression-weighted mean of phylostrata on sparse AnnData; also supplies of2orthomap to extract gene ages from OrthoFinder results) — https://github.com/kullrich/oggmap
- **Scanpy** (Toolkit whose AnnData scRNA objects and environment host the TEI calculation and downstream visualization)
- **OrthoFinder** (Upstream orthogroup inference whose results are converted into the orthomap (gene ages per orthogroup) used as TEI weights) — https://github.com/davidemms/OrthoFinder

## Examples

```
from oggmap.datasets import scRNA; adata = scRNA.nematode(); adata.layers['counts'] = adata.X; from oggmap import orthomap2tei; gene_sets, orthomap, gene_names = orthomap2tei.get_orthomap(adata, qt='6238'); adata.obs['tei'], adata.layers['tei'] = orthomap2tei.get_tei(adata, gene_sets, gene_id_col='index', gene_name_col='gene_name')
```

## Evaluation signals

- TEI per cell equals the expression-weighted mean of assigned phylostrata, consistent with Domazet-Lošo and Tautz 2010 — spot-check weighted-mean arithmetic on a few cells
- Every gene contributing to TEI has a matched gene age class; unmatched gene/transcript IDs are excluded from the weighted mean
- Benchmark runs to completion and recorded wall-clock times remain within an acceptable tractable range at >=1e6 cells (checked via value_in_range), with matrix kept sparse (CSR) throughout
- Peak memory usage scales with tile size rather than exploding, confirming sparse-matrix tractability
- TEI values fall within the numeric range of the assigned phylostrata (min <= TEI <= max phylostratum per cell)

## Limitations

- oggmap v0.0.2 was developed with Python 3.10 and does not support Python 2.7x or Python <=3.10, so environment setup matters
- The scalability claim is grounded in the authors' statement that the implementation is built on sparse-matrix calculation; tractability at a given scale depends on wall-clock time and peak memory observed in benchmarking, not on a guaranteed runtime bound
- TEI depends on the quality and completeness of the upstream orthomap; extraction is tied to orthology resources such as OrthoFinder, eggNOG, or PLAZA for the query species

## Evidence

- [abstract] can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit: "can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit"
- [methods] calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets: "calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets"
- [full_text] TEI as the expression-weighted mean of phylostrata per cell following Domazet-Lošo and Tautz 2010: [evidence span withheld — anchor status: not_found]
- [results] oggmap's TEI implementation is built on sparse-matrix calculation, which the authors state scales to millions of cells: [evidence span withheld — anchor status: not_found]
- [methods] oggmap v0.0.2 was developed using Python `3.10`. We do not support Python `2.7x` or Python `<=3.10`.: "oggmap v0.0.2 was developed using Python `3.10`. We do not support Python `2.7x` or Python `<=3.10`."
- [methods] match gene and transcript IDs and their corresponding gene age class with scRNA data sets: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
