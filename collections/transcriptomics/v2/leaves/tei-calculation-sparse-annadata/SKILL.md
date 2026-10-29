---
name: tei-calculation-sparse-annadata
description: Use when you have gene ages (an orthomap from OrthoFinder or eggNOG)
  matched to a scRNA-seq AnnData object and need a per-cell transcriptome evolutionary
  index, especially for very large datasets (thousands to millions of cells) where
  dense per-cell computation would be intractable.
when_to_use_negative:
- The expression matrix is stored as a dense array and the dataset is small enough
  that dense per-cell computation is fine — converting to sparse is unnecessary overhead
- No gene-age (orthomap/phylostrata) assignment exists for the species and generating
  one (of2orthomap, plaza2orthomap, etc.) is a separate task
- The goal is orthogroup/gene-age extraction itself (cds2aa, of2orthomap, qlin, ncbitax)
  rather than TEI computation on existing scRNA data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: oggmap
  role: Provides orthomap2tei.get_tei, which computes TEI as the expression-weighted
    mean of phylostrata per cell on sparse AnnData and scales to millions of cells
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Python toolkit used to handle the scRNA-seq AnnData objects and visualize
    the calculated TEI on scRNA datasets
- name: Python
  role: Runtime environment (oggmap v0.0.2 developed with Python 3.10; Python 2.7x
    and <=3.10 not supported per package warnings)
- name: OrthoFinder
  role: Source of orthologous groups from which the gene ages (orthomap) used as TEI
    weights are extracted via of2orthomap
  repo: https://github.com/davidemms/OrthoFinder
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:270fc0b32ca5f0e8cac7372d3c2c556abb98978abbf512706ea4e5d5de88e723
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# tei-calculation-sparse-annadata

## Summary

Compute the transcriptome evolutionary index (TEI) — the expression-weighted mean of gene phylostrata per cell, following Domazet-Lošo and Tautz 2010 — on scRNA-seq AnnData objects using oggmap's orthomap2tei.get_tei, which is built on sparse-matrix calculation so it scales to millions of cells.

## When to use

Use when you have gene ages (an orthomap from OrthoFinder or eggNOG) matched to a scRNA-seq AnnData object and need a per-cell transcriptome evolutionary index, especially for very large datasets (thousands to millions of cells) where dense per-cell computation would be intractable.

## When NOT to use

- The expression matrix is stored as a dense array and the dataset is small enough that dense per-cell computation is fine — converting to sparse is unnecessary overhead
- No gene-age (orthomap/phylostrata) assignment exists for the species and generating one (of2orthomap, plaza2orthomap, etc.) is a separate task
- The goal is orthogroup/gene-age extraction itself (cds2aa, of2orthomap, qlin, ncbitax) rather than TEI computation on existing scRNA data

## Inputs

- Sparse (CSR) AnnData scRNA-seq expression matrix (e.g. ~90,000-cell nematode single-cell RNA object)
- Orthomap gene-age assignment (gene ages per orthogroup from OrthoFinder or eggNOG results, e.g. Sun et al. 2021 gene age classes)
- Gene/transcript ID table matched between the orthomap and the scRNA dataset (e.g. from gtf2t2g)

## Outputs

- Per-cell transcriptome evolutionary index (TEI) values stored on the AnnData object
- Partial TEI profiles per gene age class
- Benchmark table (tei_runtime_benchmark.csv) of wall-clock time and peak memory per tile size (for scalability runs)

## How to apply

First extract an orthomap (gene ages per orthogroup via LCA assignment from OrthoFinder or eggNOG results) and match gene/transcript IDs to the gene age classes of the target scRNA dataset. Keep the expression matrix in sparse (CSR) format within the AnnData object and load it into an oggmap/Scanpy Python 3.10 environment. Then run orthomap2tei.get_tei, which computes TEI as the expression-weighted mean of phylostrata per cell following Domazet-Lošo and Tautz 2010; because the implementation is sparse-matrix based, very large datasets (e.g. ~90,000 nematode cells) can be tiled or replicated to at least 1,000,000 cells and processed without leaving sparse format. For scalability benchmarking, time each per-tile execution with a wall-clock timer and record wall-clock time and peak memory usage into a CSV table (tei_runtime_benchmark.csv). TEI values and partial TEI profiles can subsequently be visualized in a Scanpy context to interpret scRNA-based cell type assignments in an evolutionary framework.

## Related tools

- **oggmap** (Provides orthomap2tei.get_tei, which computes TEI as the expression-weighted mean of phylostrata per cell on sparse AnnData and scales to millions of cells) — https://github.com/kullrich/oggmap
- **Scanpy** (Python toolkit used to handle the scRNA-seq AnnData objects and visualize the calculated TEI on scRNA datasets)
- **Python** (Runtime environment (oggmap v0.0.2 developed with Python 3.10; Python 2.7x and <=3.10 not supported per package warnings))
- **OrthoFinder** (Source of orthologous groups from which the gene ages (orthomap) used as TEI weights are extracted via of2orthomap) — https://github.com/davidemms/OrthoFinder

## Examples

```
import anndata, scanpy; from oggmap import orthomap2tei; adata = anndata.read_h5ad('nematode_90k.h5ad'); tei = orthomap2tei.get_tei(adata=adata, gene_id='orthomap_gene', gene_age='ps')
```

## Evaluation signals

- The benchmark script runs to completion on a tiled AnnData of at least 1e6 cells while the expression matrix stays sparse (CSR)
- Recorded wall-clock times per tile size (checked via value_in_range) confirm get_tei remains computationally tractable at >=1e6 cells
- Per-cell TEI values equal the expression-weighted mean of assigned phylostrata following Domazet-Lošo and Tautz 2010 (spot-check on a few cells)
- Every gene in the AnnData var axis is matched to a gene age class from the orthomap before TEI computation (no unmatched IDs silently dropped)
- Peak memory usage scales with tile size and is recorded in tei_runtime_benchmark.csv

## Limitations

- oggmap v0.0.2 was developed with Python 3.10 and does not support Python 2.7x or Python <=3.10
- TEI requires an orthomap for the query species, which depends on upstream orthology inference quality (OrthoFinder/eggNOG/PLAZA results)
- The claim of scaling to millions of cells rests on the authors' sparse-matrix implementation statement; wall-clock and memory behavior should be benchmarked on the target hardware

## Evidence

- [methods] calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets: "calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets."
- [other] TEI as expression-weighted mean of phylostrata (Domazet-Lošo and Tautz 2010): [evidence span withheld — anchor status: not_found]
- [results] sparse-matrix TEI scales to millions of cells: [evidence span withheld — anchor status: not_found]
- [abstract] Scanpy toolkit for gene age-weighted expression data: "can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit"
- [methods] match gene and transcript IDs with scRNA datasets: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
