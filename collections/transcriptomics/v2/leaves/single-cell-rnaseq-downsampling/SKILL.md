---
name: single-cell-rnaseq-downsampling
description: Use when benchmarking computational performance (runtime, peak memory)
  of a scoring method across dataset sizes and you need controlled, identically sized
  single-cell expression matrices rather than testing only on the full million-cell
  atlas.
when_to_use_negative:
- The goal is biological discovery (multicellular pathway modules via MOFA), not performance
  measurement — downsampling cells would discard signal; the paper notes downsampling
  strategies (e.g. SCPA's default 500 cells per condition) may lose information on
  large datasets
- The study has fewer samples than scPAFA's requirement of at least 15 samples (not
  cells) for MOFA-based module identification — downsampling cells does not address
  the sample-count constraint
- The input is already a small dataset at the target size, making downsampling a no-op
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scPAFA
  role: Provides fast_ucell and fast_score_genes, the methods whose scaling with downsampled
    cell counts is measured (10 cores, and 20 cores on the 14,300-pathway C5 collection)
  repo: github.com/ZhuoliHuang/scPAFA
- name: UCell
  role: Baseline PAS method run with 10 cores on each downsampled dataset for runtime/memory
    comparison
- name: AUCell
  role: Baseline PAS method run with 10 cores on each downsampled dataset for runtime/memory
    comparison
- name: Scanpy
  role: Provides score_genes (equivalent to AddModuleScore), run with 1 core as a
    baseline on downsampled datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:cf228a57e9eb8e1b21e27225b710e4ae2c7726af84cccd0689d748b931905250
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AUCell
  - AddModuleScore
  - Python
  - Scanpy
  - UCell
  - scPAFA
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# single-cell-rnaseq-downsampling

## Summary

Generate downsampled versions of single-cell gene expression matrices spanning a range of cell counts (e.g. 10,000 to 1,263,676 cells from the lupus atlas) so that runtime and peak-memory scaling of pathway activity scoring methods can be benchmarked on identically sized inputs.

## When to use

Use when benchmarking computational performance (runtime, peak memory) of a scoring method across dataset sizes and you need controlled, identically sized single-cell expression matrices rather than testing only on the full million-cell atlas.

## When NOT to use

- The goal is biological discovery (multicellular pathway modules via MOFA), not performance measurement — downsampling cells would discard signal; the paper notes downsampling strategies (e.g. SCPA's default 500 cells per condition) may lose information on large datasets
- The study has fewer samples than scPAFA's requirement of at least 15 samples (not cells) for MOFA-based module identification — downsampling cells does not address the sample-count constraint
- The input is already a small dataset at the target size, making downsampling a no-op

## Inputs

- Single-cell gene expression matrices (e.g. CRC datasets and the large-scale lupus PBMC atlas with 1,263,676 cells from 99 healthy controls and 162 SLE cases)
- Pathway dictionaries (Python dict: pathway name -> gene list) built from MsigDB (1629 pathways for CRC) and NCATS BioPlanet (1383 pathways for lupus)
- Target cell-count grid for downsampling (e.g. 10,000, 50,000, ... up to 1,263,676 cells)

## Outputs

- Downsampled single-cell expression matrices at each sampled cell count
- Per-combination measurements of elapsed runtime and peak memory usage
- Aggregated benchmark table (runtime_memory_benchmark.csv)

## How to apply

Load the single-cell gene expression matrices (e.g. the CRC datasets and the large-scale lupus atlas of 1,263,676 cells) together with the pathway dictionaries (1629 pathways for CRC, 1383 for lupus, built from MsigDB and NCATS BioPlanet). Generate downsampled versions of each dataset at a series of sampled cell counts spanning 10,000 cells up to the full 1,263,676-cell dataset, holding the pathway collection constant so that runtime differences reflect cell number only. Feed each downsampled matrix through the methods under comparison (scPAFA's fast_ucell and fast_score_genes with 10 cores; UCell and AUCell with 10 cores; Scanpy score_genes with 1 core), recording elapsed runtime and peak memory for every method-dataset-cell-count combination. The downsampling design directly supports the paper's claims that scPAFA computes PAS for 1383 pathways in million-cell-level data within 30 min (fast_ucell 27.05 min, fast_score_genes 29.9 min on the full lupus atlas), versus UCell 21.4 h, score_genes 9.3 h and AUCell 5.1 h.

## Related tools

- **scPAFA** (Provides fast_ucell and fast_score_genes, the methods whose scaling with downsampled cell counts is measured (10 cores, and 20 cores on the 14,300-pathway C5 collection)) — github.com/ZhuoliHuang/scPAFA
- **UCell** (Baseline PAS method run with 10 cores on each downsampled dataset for runtime/memory comparison)
- **AUCell** (Baseline PAS method run with 10 cores on each downsampled dataset for runtime/memory comparison)
- **Scanpy** (Provides score_genes (equivalent to AddModuleScore), run with 1 core as a baseline on downsampled datasets)

## Examples

```
python -c "import scanpy as sc; adata = sc.read_h5ad('lupus.h5ad'); sc.pp.subsample(adata, n_obs=10000, random_state=0); adata.write('lupus_10k.h5ad')"
```

## Evaluation signals

- The downsampled matrix at each target size contains exactly the sampled number of cells (10k through 1,263,676), with the full lupus dataset as the largest point
- The same pathway dictionary (1629 CRC / 1383 lupus pathways) is applied to every downsampled size, so runtime scales reflect cell number only
- Runtime and peak memory are recorded for every method-dataset-cell-count combination and aggregated into runtime_memory_benchmark.csv
- Results on the full lupus atlas reproduce the reported baselines: fast_ucell 27.05 min, fast_score_genes 29.9 min vs UCell 21.4 h, score_genes 9.3 h, AUCell 5.1 h (~3.8–47.4x faster than original implementations)
- Runtime should scale monotonically (roughly linearly) with cell count for each method across the downsampled series

## Limitations

- Downsampling loses information on large datasets — the paper criticizes SCPA's default 500-cells-per-condition downsampling strategy for exactly this reason, and increasing downsampled cell numbers significantly affects computational efficiency
- Downsampling supports runtime/memory scaling benchmarks only; it does not test scaling with pathway number (that required the separate 14,300-pathway C5 MsigDB run with 20 cores)
- Scanpy score_genes is benchmarked with only 1 core versus 10 cores for the other methods, so cross-method runtime comparisons partly reflect core-count differences

## Evidence

- [other] Generate downsampled versions of each dataset at sampled cell counts spanning 10k to 1.26M cells.: [evidence span withheld — anchor status: not_found]
- [intro] Claimed performance: PAS for 1383 pathways in million-cell data within 30 min: "which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min"
- [intro] Downsampling may lose information on large datasets: "it uses a downsampling strategy (selecting 500 cells per condition as default) that may lose information on large datasets"
- [abstract] Lupus atlas over 1.2 million cells: "large-scale lupus atlas over 1.2 million cells"
- [readme] README tutorial notebooks for PAS computation: "single-cell gene expression matrix and collection of pathways are used to compute PAS by ‘fast_Ucell'"
- [readme] fast functions are efficient re-implementations of UCell and score_genes: "These functions are more computationally efficient implementation of UCell and AddModuleScore (also known as ‘score_genes' in Scanpy)"
