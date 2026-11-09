---
name: runtime-memory-profiling
description: Use when validating that a per-cell metric computed over scRNA data (e.g.
  TEI via orthomap2tei.get_tei) remains tractable on datasets tiled to millions of
  cells, or when producing runtime/memory scaling tables for a sparse-matrix implementation.
when_to_use_negative:
- The goal is biological interpretation of TEI/phylostrata patterns rather than verifying
  computational scalability of the implementation.
- The expression data is small (e.g. a handful of bulk-RNA stages) and runtime is
  trivially bounded, so a scaling benchmark adds no information.
- The data cannot be kept in sparse format — densifying a million-cell matrix defeats
  the purpose and will inflate memory usage beyond what the skill is meant to measure.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: oggmap
  role: provides orthomap2tei.get_tei, the sparse-matrix TEI implementation being
    benchmarked on tiled AnnData
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: holds and manipulates the sparse scRNA AnnData objects that are tiled to >=1e6
    cells
- name: Python
  role: environment (v3.10) in which wall-clock timing and benchmark CSV recording
    are implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:707ab0cc65fe955cc2fdaa4f7625b378c1fa94c79e05f37b67b311510a286e3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - Python
  - Scanpy
  - oggmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# runtime-memory-profiling

## Summary

Benchmark the wall-clock runtime and peak memory of oggmap's sparse-matrix TEI calculation as AnnData objects are tiled from ~90,000 nematode scRNA cells up to >=1,000,000 cells, to demonstrate computational tractability at scale.

## When to use

Use when validating that a per-cell metric computed over scRNA data (e.g. TEI via orthomap2tei.get_tei) remains tractable on datasets tiled to millions of cells, or when producing runtime/memory scaling tables for a sparse-matrix implementation.

## When NOT to use

- The goal is biological interpretation of TEI/phylostrata patterns rather than verifying computational scalability of the implementation.
- The expression data is small (e.g. a handful of bulk-RNA stages) and runtime is trivially bounded, so a scaling benchmark adds no information.
- The data cannot be kept in sparse format — densifying a million-cell matrix defeats the purpose and will inflate memory usage beyond what the skill is meant to measure.

## Inputs

- nematode scRNA-seq AnnData object (~90,000 cells) with sparse (CSR) expression matrix
- gene age assignment (orthomap) per gene, e.g. from Sun et al. 2021
- tile sizes defining the cell counts to benchmark (e.g. up to 1,000,000 cells)

## Outputs

- tei_runtime_benchmark.csv — per-tile-size wall-clock time and peak memory usage
- TEI values per cell for each tiled AnnData

## How to apply

Load the nematode single-cell RNA AnnData object (~90,000 cells) together with the Sun et al. 2021 gene age (orthomap) assignment into an oggmap/Scanpy environment, keeping the expression matrix in sparse CSR format. Tile or replicate the sparse AnnData to reach at least 1,000,000 cells, then run orthomap2tei.get_tei — which computes TEI as the expression-weighted mean of phylostrata per cell following Domazet-Lošo and Tautz 2010 — on each tiled AnnData. Time each execution with a wall-clock timer and record peak memory usage. The rationale is that per-cell TEI on large scRNA datasets is computationally heavy, and the authors state the sparse-matrix implementation scales to millions of cells; the benchmark empirically verifies that claim rather than accepting it. Record per-tile-size wall-clock time and peak memory into a CSV table (tei_runtime_benchmark.csv) so scaling behavior can be checked with range assertions.

## Related tools

- **oggmap** (provides orthomap2tei.get_tei, the sparse-matrix TEI implementation being benchmarked on tiled AnnData) — https://github.com/kullrich/oggmap
- **Scanpy** (holds and manipulates the sparse scRNA AnnData objects that are tiled to >=1e6 cells)
- **Python** (environment (v3.10) in which wall-clock timing and benchmark CSV recording are implemented)

## Examples

```
import time; from oggmap import orthomap2tei; t0=time.time(); tei = orthomap2tei.get_tei(adata, gene_age_dict, obs_id='cell', gene_id='gene'); print(time.time()-t0)
```

## Evaluation signals

- The benchmark script runs to completion on a tiled AnnData with >=1e6 cells without densifying the matrix.
- Recorded wall-clock times in tei_runtime_benchmark.csv pass a value_in_range check confirming tractability at the largest tile size.
- The expression matrix remains sparse (CSR) throughout — a conversion to dense signals a failure mode.
- TEI values match the expected definition: expression-weighted mean of phylostrata per cell (Domazet-Lošo and Tautz 2010).
- Runtime/memory scale roughly linearly (or tractably) with tile size rather than exploding.

## Limitations

- Runtime claims rest on the authors' statement that the implementation scales to millions of cells; actual tractability depends on hardware and must be re-verified per environment.
- Tiling/replicating a real ~90,000-cell dataset to 1,000,000 cells duplicates cells rather than sampling new biological data, so the benchmark measures compute cost, not statistical behavior.
- The article's discussion section provides no versioned dependency information for reproducing the exact benchmark environment (oggmap v0.0.2, Python 3.10 stated in methods only).

## Evidence

- [methods] calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets.: "calculate and visualize transcriptome evolutionary index (TEI) on scRNA data sets."
- [abstract] can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit: "can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit"
- [readme] Oggmap results (gene ages per orthologous group) can be further used to calculate and visualize weighted expression: [evidence span withheld — anchor status: external_doc]
