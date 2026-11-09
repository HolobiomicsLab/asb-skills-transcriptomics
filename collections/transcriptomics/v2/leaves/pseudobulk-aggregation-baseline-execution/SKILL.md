---
name: pseudobulk-aggregation-baseline-execution
description: 'Use when benchmarking sample-level single-cell methods on runtime or
  scalability: you need the cheaper aggregation-based baseline (cell-type proportion
  aggregation per patient) to time against PILOT/PhEMD on a large dataset such as
  the 1.26M-cell Lupus PBMC data.'
when_to_use_negative:
- The goal is unsupervised sample-level analysis of full single-cell distributions
  rather than a timing baseline — use PILOT, which computes Wasserstein distances
  between individual single-cell samples
- The analysis requires per-cluster gene-expression markers or trajectory statistics
  (Huber regression, Wald tests) that only PILOT's optimal-transport plan provides
- The dataset is pathomics (histological tissue structures) where PhEMD's Monocle
  2-based pipeline, not pseudo-bulk aggregation, is the competing baseline
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: PILOT
  role: The method being benchmarked against the pseudo-bulk baseline; computes Wasserstein
    distances between individual samples using optimal transport
  repo: https://github.com/CostaLab/PILOT
- name: PhEMD
  role: Third competing method timed in the same benchmark; requires Monocle 2 for
    normalization, dimension reduction, and clustering of cells
- name: POT library
  role: Provides the emd2 function PILOT uses for the optimal-transport distances
    that the pseudo-bulk baseline is compared against on runtime
- name: scanpy
  role: Loads and preprocesses the .h5ad single-cell dataset before aggregation and
    timing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:cc07d1886a59f838dd091a47d8904e3e0e457e5be794dab02cfcc1bd56b0320e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - PhEMD
  - Seurat
  - scanpy
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# pseudobulk-aggregation-baseline-execution

## Summary

Runs the pseudo-bulk baseline from the PILOT benchmark, which aggregates cell-type proportions per patient/sample before computing sample-level distances, and records its wall-clock runtime to compare against PILOT's optimal-transport-based Wasserstein distances and PhEMD on the same data.

## When to use

Use when benchmarking sample-level single-cell methods on runtime or scalability: you need the cheaper aggregation-based baseline (cell-type proportion aggregation per patient) to time against PILOT/PhEMD on a large dataset such as the 1.26M-cell Lupus PBMC data.

## When NOT to use

- The goal is unsupervised sample-level analysis of full single-cell distributions rather than a timing baseline — use PILOT, which computes Wasserstein distances between individual single-cell samples
- The analysis requires per-cluster gene-expression markers or trajectory statistics (Huber regression, Wald tests) that only PILOT's optimal-transport plan provides
- The dataset is pathomics (histological tissue structures) where PhEMD's Monocle 2-based pipeline, not pseudo-bulk aggregation, is the competing baseline

## Inputs

- Single-cell AnnData .h5ad file (e.g., GSE174188_CLUES1_adjusted.h5ad.gz, ~1.26M Lupus PBMC cells from GEO accession GSE174188)
- Per-cell cluster/cell-type label assignments per sample

## Outputs

- Sample-level distance matrix computed from aggregated per-patient cell-type proportions
- Benchmark table (method, runtime) saved as lupus_runtime_benchmark.csv

## How to apply

Load the target dataset (e.g., GSE174188_CLUES1_adjusted.h5ad.gz from GEO, ~1,263,676 cells) with scanpy, then aggregate cell-type proportions per patient/sample so each sample is summarized by its cell-type composition rather than single-cell resolution. Compute sample-level distances on these aggregated proportions (this avoids the per-cell optimal transport cost that PILOT incurs via POT's emd2). Time the end-to-end run with Python's time module and assemble the runtime into a comparison table (lupus_runtime_benchmark.csv) alongside PILOT (58.12 s) and PhEMD (16.50 min), checking that each measured runtime falls within the same order of magnitude as the values reported in the Results section.

## Related tools

- **PILOT** (The method being benchmarked against the pseudo-bulk baseline; computes Wasserstein distances between individual samples using optimal transport) — https://github.com/CostaLab/PILOT
- **PhEMD** (Third competing method timed in the same benchmark; requires Monocle 2 for normalization, dimension reduction, and clustering of cells)
- **POT library** (Provides the emd2 function PILOT uses for the optimal-transport distances that the pseudo-bulk baseline is compared against on runtime)
- **scanpy** (Loads and preprocesses the .h5ad single-cell dataset before aggregation and timing)

## Examples

```
import scanpy as sc, time; adata = sc.read_h5ad('GSE174188_CLUES1_adjusted.h5ad.gz'); t0 = time.time(); # aggregate cell-type proportions per patient, compute sample distances; print(time.time() - t0)
```

## Evaluation signals

- The benchmark table contains one runtime row for each of PILOT, pseudo-bulk, and PhEMD
- Each measured runtime falls within the same order of magnitude as the corresponding reported value (PILOT 58.12 s, pseudo-bulk 9.03 s, PhEMD 16.50 min) for the 1.26M-cell Lupus dataset
- The pseudo-bulk runtime should be substantially lower than PILOT's, since aggregating cell-type proportions per patient avoids per-cell optimal-transport computation
- The input AnnData is confirmed to be the Lupus PBMC dataset (~1,263,676 cells) downloaded from GEO GSE174188

## Limitations

- Runtimes are hardware- and implementation-dependent, so absolute times may differ; validation is order-of-magnitude, not exact
- The pseudo-bulk baseline discards per-cell information by aggregating proportions per patient, so it cannot reproduce PILOT's trajectory or cluster-level statistical outputs
- PILOT itself does not explicitly correct for potential technical artefacts and assumes tissue samples were collected uniformly (similar tissue areas and same technology)

## Evidence

- [results] pseudo-bulk runtime result: [evidence span withheld — anchor status: not_found]
- [abstract] PILOT optimal transport basis: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
- [discussion] POT emd2 implementation: "PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011)."
- [discussion] PhEMD pipeline requirement: "PhEMD required the use of Monocle 2 (Trapnell et al, 2014) to perform normalization, dimension reduction, and clustering of cells."
- [discussion] Lupus dataset source: "provided in Gene Expression Omibus (GEO; GSE174188; GSE174188_CLUES1_adjusted.h5ad.gz)"
