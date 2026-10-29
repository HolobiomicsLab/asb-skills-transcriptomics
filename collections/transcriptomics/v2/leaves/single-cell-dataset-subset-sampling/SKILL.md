---
name: single-cell-dataset-subset-sampling
description: 'Use when benchmarking execution time of single-cell methods across growing
  sample sizes: subset a raw cell-by-gene count matrix at fixed increments and record
  single-core wall-clock time and failures per method per size.'
when_to_use_negative:
- The goal is rare-cell detection accuracy on a single fixed dataset, not runtime
  scaling — use FiRE scoring directly with IQR thresholding (th = q3 + 1.5*iqr) instead
  of subsetting
- The dataset is small (hundreds to ~1k cells) where scaling differences between methods
  are not meaningful
- Input is already a per-method runtime table and only needs comparison, not re-subsetting
  of raw data
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FiRE
  role: Fast benchmarked method whose runtime is measured per subset; installed in
    the Python environment via ./INSTALL --py and instantiated with L=100, M=50, H=1017881,
    seed=5489
  repo: github.com/princethewinner/FiRE
- name: RaceID
  role: R-based rare-cell method benchmarked as the slow baseline (took ~79 h for
    just 5k cells)
- name: GiniClust
  role: R-based rare-cell method benchmarked as a scaling-failure case (runtime error
    beyond ~45k profiles)
- name: Python
  role: Environment for running FiRE and LOF single-core on each subset and recording
    wall-clock times
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:b3266e89ead3e67ca0bb72b72a4453267503e1502e19850123210eb8059e48a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - GiniClust
  - Python
  - RaceID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-dataset-subset-sampling

## Summary

Constructs nested cell-count subsets (1k, 5k, 10k, ... up to full ~68k profiles) from a scRNA-seq expression matrix and times each method on every subset to produce a runtime-scaling comparison table. Used to quantify how algorithms like FiRE, LOF, RaceID and GiniClust scale with sample size.

## When to use

Use when benchmarking execution time of single-cell methods across growing sample sizes: subset a raw cell-by-gene count matrix at fixed increments and record single-core wall-clock time and failures per method per size.

## When NOT to use

- The goal is rare-cell detection accuracy on a single fixed dataset, not runtime scaling — use FiRE scoring directly with IQR thresholding (th = q3 + 1.5*iqr) instead of subsetting
- The dataset is small (hundreds to ~1k cells) where scaling differences between methods are not meaningful
- Input is already a per-method runtime table and only needs comparison, not re-subsetting of raw data

## Inputs

- ~68k-cell human PBMC scRNA-seq count matrix from GEO accession GSE94820 (cell x gene, e.g. .txt/.txt.gz as used by FiRE's loader)
- List of target subset sizes (1k, 5k, 10k, 15k, ... ~68k cells)
- Installed methods: FiRE (Python/C++), LOF (Python), RaceID (R), GiniClust (R)

## Outputs

- Nested expression-matrix subsets of increasing cell count (cells x top-1000 variable genes)
- Runtime table: one row per method per subset size with wall-clock seconds
- Failure log per method per size (e.g. GiniClust error beyond ~45k profiles)

## How to apply

Download the ~68k-cell human PBMC scRNA-seq dataset (GEO GSE94820) and load it as a cell-by-gene matrix, since FiRE's Python module expects rows as cells and columns as genes. Build nested subsets at fixed cell counts (1k, 5k, 10k, 15k, ... up to the full ~68k profiles) by slicing rows of the expression matrix, applying the same preprocessing (e.g. ranger_preprocess selecting the top 1000 variable genes) to each subset. Run each method single-core on every subset, timing wall-clock execution per run: FiRE via its Python/C++ module (e.g. FiRE.FiRE(L=100, M=50, H=1017881, seed=5489)), LOF in Python, and RaceID/GiniClust in R. Log both completion times and failures, such as GiniClust terminating beyond ~45k profiles. Compile all times into one comparison table with one row per method per subset size, then validate against reported values: FiRE ~26 s (paper reports ~31 s class) on ~68k cells, GiniClust failing past ~45k cells, and RaceID taking ~79 h at only 5k cells.

## Related tools

- **FiRE** (Fast benchmarked method whose runtime is measured per subset; installed in the Python environment via ./INSTALL --py and instantiated with L=100, M=50, H=1017881, seed=5489) — github.com/princethewinner/FiRE
- **RaceID** (R-based rare-cell method benchmarked as the slow baseline (took ~79 h for just 5k cells))
- **GiniClust** (R-based rare-cell method benchmarked as a scaling-failure case (runtime error beyond ~45k profiles))
- **Python** (Environment for running FiRE and LOF single-core on each subset and recording wall-clock times)

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489); import time; t0=time.time(); model.fit(subset); print(time.time()-t0)
```

## Evaluation signals

- FiRE completes the full ~68k-cell PBMC subset in ~26 s wall-clock time (paper reports ~31 s class runtimes), matching Supplementary Table 1 within tolerance
- GiniClust throws a runtime error or terminates beyond ~45k profiles rather than completing larger subsets
- RaceID fails to complete in reasonable time at 5k cells (reported ~79 h), reproducing the paper's scaling result
- Every subset is preprocessed identically (e.g. top-1000 variable genes via ranger_preprocess) so runtimes are comparable across sizes
- Comparison table has exactly one row per (method, subset size) pair with no missing timing or failure entry

## Limitations

- The benchmark presupposes a machine comparable to the paper's setup (single-core timing); results differ on other hardware
- GiniClust cannot be timed at large sample sizes because it terminates beyond ~45k profiles
- FiRE does not discriminate between an outlier and cells representing minor cell types, so the scaling benchmark does not evaluate detection quality, only runtime
- Loading large data (e.g. the full jurkat/PBMC .txt.gz) can require large amounts of RAM, separate from method runtime itself

## Evidence

- [discussion] FiRE ~31 s on ~68k profiles: "FiRE took ~31 s to analyze a scRNA-seq dataset containing ~68 k expression profiles."
- [intro] ~68k human blood profiles demonstration: "We also demonstrated the efficacy of FiRE in delineating human blood dendritic cell sub-types using ~68 k single-cell expression profiles of human blood cells."
- [abstract] Existing methods fail at tens of thousands: "Existing algorithms to find rare cells scale unbearably slowly or terminate, as the sample size grows to the order of tens of thousands."
- [readme] Data orientation and preprocessing: [evidence span withheld — anchor status: external_doc]
- [readme] Model parameters: [evidence span withheld — anchor status: external_doc]
