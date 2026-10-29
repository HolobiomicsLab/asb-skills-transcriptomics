---
name: runtime-memory-benchmarking
description: Use when choosing or justifying a metacell construction tool for large
  scRNA-seq datasets — you need CPU time and peak memory as a function of cell count
  across methods such as MC2, SuperCell, and SEACells.
when_to_use_negative:
- The goal is metacell quality assessment (purity, compactness, separation, INV, separation)
  rather than computational scaling — use the Toolkit QC functions instead.
- You only have a single small dataset where all candidate methods are already known
  to be tractable and runtime is not a decision criterion.
- You are evaluating downstream metacell analysis tools (e.g. MCView shiny exploration)
  rather than the metacell construction step itself.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: MetacellAnalysisToolkit
  role: integrated conda/pipeline environment used to run metacell construction with
    multiple methods for the benchmark
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: MC2 (tanaylab/metacells)
  role: metacell construction method benchmarked at graining level 75, including its
    divide-and-conquer variant for the largest subsets
  repo: https://github.com/tanaylab/metacells
- name: SuperCell
  role: metacell construction method benchmarked at graining level 75, plus the projection-based
    variant
  repo: https://github.com/GfellerLab/SuperCell
- name: SEACells
  role: metacell construction method benchmarked on the subset sizes tractable for
    that method
  repo: https://github.com/dpeerlab/SEACells
- name: ggplot2
  role: plots CPU time and peak memory versus number of cells for each method and
    variant
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:b5bfecbc58cd69d116c71837d2f2c0f568ed4bffad48a1271ab206fcb44a6b8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/MetacellAnalysisToolkit
  tools:
  - MC2 (tanaylab/metacells)
  - MetacellAnalysisToolkit
  - SEACells
  - Seurat
  - SuperCell
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# runtime-memory-benchmarking

## Summary

Benchmark the computational cost of metacell construction by instrumenting MC2, SuperCell, and SEACells runs for elapsed CPU time and peak memory across size-stratified single-cell datasets, then fitting the scaling curves that reveal which methods remain tractable at atlas scale (e.g. 900,000 cells).

## When to use

Use when choosing or justifying a metacell construction tool for large scRNA-seq datasets — you need CPU time and peak memory as a function of cell count across methods such as MC2, SuperCell, and SEACells.

## When NOT to use

- The goal is metacell quality assessment (purity, compactness, separation, INV, separation) rather than computational scaling — use the Toolkit QC functions instead.
- You only have a single small dataset where all candidate methods are already known to be tractable and runtime is not a decision criterion.
- You are evaluating downstream metacell analysis tools (e.g. MCView shiny exploration) rather than the metacell construction step itself.

## Inputs

- scRNA-seq count matrices from GEO accession GSE114953 (MOCA, Cao et al 2019)
- size-stratified cell subsets spanning 10,000 to 900,000 cells
- graining level parameter (75) for metacell construction

## Outputs

- scaling_benchmark.csv with elapsed CPU time and peak memory per method x variant x dataset size
- fig5c_scaling_curves.png plotting CPU time and peak memory versus number of cells for each method and variant (ggplot2)

## How to apply

Prepare size-stratified subsets of the input count matrix (here the MOCA mouse organogenesis atlas, GEO GSE114953, spanning 10,000 to 900,000 cells). Build metacells with each method at a fixed graining level of 75 using the MetacellAnalysisToolkit environment, adding the divide-and-conquer MC2 variant and the projection-based SuperCell variant on the largest subsets, and running SEACells only on the subset sizes tractable for it. Instrument every construction run to record elapsed CPU time and peak memory per method, variant, and dataset size, including the per-sample parallel MOCA run. Assemble measurements into a table (scaling_benchmark.csv) and plot CPU time and peak memory versus number of cells per method/variant with ggplot2 (fig5c_scaling_curves.png). The rationale is that, except for the divide-and-conquer MC2 method, metacell construction is time- and memory-consuming on large datasets, so the scaling curves determine which method or variant extends to the largest data.

## Related tools

- **MetacellAnalysisToolkit** (integrated conda/pipeline environment used to run metacell construction with multiple methods for the benchmark) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **MC2 (tanaylab/metacells)** (metacell construction method benchmarked at graining level 75, including its divide-and-conquer variant for the largest subsets) — https://github.com/tanaylab/metacells
- **SuperCell** (metacell construction method benchmarked at graining level 75, plus the projection-based variant) — https://github.com/GfellerLab/SuperCell
- **SEACells** (metacell construction method benchmarked on the subset sizes tractable for that method) — https://github.com/dpeerlab/SEACells
- **ggplot2** (plots CPU time and peak memory versus number of cells for each method and variant)

## Examples

```
conda env create -f env/MetacellAnalysisToolkit_env.yml
```

## Evaluation signals

- The benchmark table covers every method x variant x dataset-size combination, including the divide-and-conquer MC2 and projection-based SuperCell variants on the 900k-cell dataset.
- Plotted CPU-time and memory curves reproduce the Fig 5C scaling behavior for each method.
- Every construction run has recorded elapsed CPU time and peak memory values for its method, variant, and dataset size (no missing instrumented runs).
- SEACells entries appear only for the subset sizes that method could tractably handle, and the largest subsets show divide-and-conquer MC2 completing where standard MC2 does not.
- The per-sample parallel MOCA run is represented as a distinct variant in the benchmark table.

## Limitations

- Except for the divide-and-conquer MC2 method, building metacells is time- and memory-consuming on large datasets, so non-divide-and-conquer methods may not run to completion on the largest subsets.
- SEACells is only tractable on a limited range of subset sizes and must be run on those sizes only.
- tanaylab/metacells notes API breaks across versions (0.8 to 0.9) and ongoing migration from AnnData to daf, so version pinning matters for reproducible timing.

## Evidence

- [abstract] Metacells reduce the size and complexity of single-cell genomics data while preserving biologically relevant information and improving interpretability.: "metacells have been introduced to reduce the size and complexity of single-cell genomics data while preserving biologically relevant information and improving interpretability"
- [abstract] Toolkit pipeline scope: "a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods"
- [readme] Metacells python package scalability: [evidence span withheld — anchor status: external_doc]
- [readme] Metacell dimension reduction benefit for large data: [evidence span withheld — anchor status: external_doc]
- [readme] Toolkit environment setup for benchmark runs: [evidence span withheld — anchor status: external_doc]
