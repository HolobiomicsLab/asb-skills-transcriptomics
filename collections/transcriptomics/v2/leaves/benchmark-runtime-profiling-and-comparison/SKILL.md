---
name: benchmark-runtime-profiling-and-comparison
description: Use when a claim about an optimal-transport or sample-level single-cell
  method's scalability to millions of cells must be validated by measuring and comparing
  end-to-end wall-clock runtimes across competing pipelines.
when_to_use_negative:
- Input is a single sample rather than multiple patient samples — sample-level distance
  methods (PILOT, PhEMD) have nothing to compare, so runtime benchmarking is meaningless.
- The goal is comparing methods' biological accuracy (clustering/trajectory performance
  across the 12 public datasets) rather than computational scalability; use a performance
  benchmark instead.
- Data is not multi-sample single-cell or pathomics data (e.g. bulk RNA-seq or a feature
  table), where the per-sample Wasserstein-distance framing does not apply.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3307
tools:
- name: PILOT
  role: 'Method under test: computes Wasserstein distance between individual single-cell
    samples end-to-end; timed on the full Lupus PBMC dataset'
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Provides the emd2 function underlying PILOT's Wasserstein distance computation
    between sample cluster distributions
- name: PhEMD
  role: Competing EMD-based method timed end-to-end as the slow baseline for runtime
    comparison
- name: Monocle 2
  role: Required inside the PhEMD pipeline to perform normalization, dimension reduction,
    and clustering of cells before timing
- name: scanpy
  role: Loads the downloaded .h5ad Anndata file for processing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:cf2a11792609435764f1d5b230db993ff80607b8ddbb180a8506bb56cfcd23d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - Monocle 2
  - PILOT
  - POT library
  - PhEMD
  - Seurat
  - scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# benchmark-runtime-profiling-and-comparison

## Summary

Wall-clock runtime benchmarking of PILOT against sample-level competitors (pseudo-bulk baseline, PhEMD) on a large multi-patient single-cell dataset, using Python's time module around end-to-end pipeline executions. It grounds scalability claims — PILOT's POT/emd2-based Wasserstein distances — in measured runtimes rather than qualitative statements.

## When to use

Use when a claim about an optimal-transport or sample-level single-cell method's scalability to millions of cells must be validated by measuring and comparing end-to-end wall-clock runtimes across competing pipelines.

## When NOT to use

- Input is a single sample rather than multiple patient samples — sample-level distance methods (PILOT, PhEMD) have nothing to compare, so runtime benchmarking is meaningless.
- The goal is comparing methods' biological accuracy (clustering/trajectory performance across the 12 public datasets) rather than computational scalability; use a performance benchmark instead.
- Data is not multi-sample single-cell or pathomics data (e.g. bulk RNA-seq or a feature table), where the per-sample Wasserstein-distance framing does not apply.

## Inputs

- Lupus PBMC Anndata file GSE174188_CLUES1_adjusted.h5ad.gz from GEO accession GSE174188 (~1,263,676 cells)
- Installed PILOT (pilotpy) environment with the POT Python library
- PhEMD pipeline with Monocle 2 for normalization, dimension reduction, and clustering
- Pseudo-bulk baseline aggregating per-patient cell-type proportions

## Outputs

- lupus_runtime_benchmark.csv: table with one runtime row per method (PILOT, pseudo-bulk, PhEMD)
- Measured wall-clock runtimes per method (e.g. PILOT 58.12 s, pseudo-bulk 9.03 s, PhEMD 16.50 min on 1.26M cells)

## How to apply

Download the large target dataset (Lupus PBMC, GSE174188_CLUES1_adjusted.h5ad.gz, ~1.26M cells) from GEO and load it with scanpy. Run PILOT end-to-end, which computes the Wasserstein distance between individual patient samples via the emd2 function of the POT library, timing the whole run with Python's time module. Run the pseudo-bulk baseline, which aggregates cell-type proportions per patient before computing sample-level distances, and record its wall-clock time. Run PhEMD end-to-end — it requires Monocle 2 for normalization, dimension reduction, and clustering of cells — and record its time. Assemble the measured (method, runtime) pairs into a table (e.g. lupus_runtime_benchmark.csv) and validate that each measured runtime falls within the same order of magnitude as the value reported in the paper's Results section.

## Related tools

- **PILOT** (Method under test: computes Wasserstein distance between individual single-cell samples end-to-end; timed on the full Lupus PBMC dataset) — https://github.com/CostaLab/PILOT
- **POT library** (Provides the emd2 function underlying PILOT's Wasserstein distance computation between sample cluster distributions)
- **PhEMD** (Competing EMD-based method timed end-to-end as the slow baseline for runtime comparison)
- **Monocle 2** (Required inside the PhEMD pipeline to perform normalization, dimension reduction, and clustering of cells before timing)
- **scanpy** (Loads the downloaded .h5ad Anndata file for processing)

## Examples

```
import scanpy as sc, time; adata = sc.read_h5ad('GSE174188_CLUES1_adjusted.h5ad.gz'); t0 = time.time(); # run PILOT end-to-end via ot.emd2 between sample distributions; runtime = time.time() - t0
```

## Evaluation signals

- The benchmark table contains exactly one runtime row for each of PILOT, pseudo-bulk, and PhEMD.
- Each measured runtime falls within the same order of magnitude as the corresponding value reported in the paper's Results for the 1.26M-cell Lupus dataset.
- Each method was run end-to-end (PILOT with emd2 distances, PhEMD including Monocle 2 preprocessing), not partially, before timing.
- Timing was captured with Python's time module around the full pipeline, and the cell count of the processed dataset (~1,263,676) matches the expected scale.

## Limitations

- Runtimes are hardware- and implementation-dependent; validation is by order of magnitude, not exact reproduction of 58.12 s / 9.03 s / 16.50 min.
- PhEMD depends on Monocle 2 preprocessing, so its measured time includes normalization, dimension reduction, and clustering, which inflates the comparison relative to a pure distance computation.
- The paper notes software/package versions were not specified, so exact runtime reproducibility across environments is not guaranteed.

## Evidence

- [discussion] PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011).: "PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011)."
- [discussion] PhEMD required the use of Monocle 2 (Trapnell et al, 2014) to perform normalization, dimension reduction, and clustering of cells.: "PhEMD required the use of Monocle 2 (Trapnell et al, 2014) to perform normalization, dimension reduction, and clustering of cells."
- [discussion] provided in Gene Expression Omibus (GEO; GSE174188; GSE174188_CLUES1_adjusted.h5ad.gz): "provided in Gene Expression Omibus (GEO; GSE174188; GSE174188_CLUES1_adjusted.h5ad.gz)"
- [readme] The easiest way to install PILOT and the required packages is using the following way:: [evidence span withheld — anchor status: external_doc]
- [abstract] We evaluate PILOT and competing approaches in single-cell genomics or pathomics studies involving various human diseases with up to 600 samples/patients and millions of cells: "We evaluate PILOT and competing approaches in single-cell genomics or pathomics studies involving various human diseases with up to 600 samples/patients"
