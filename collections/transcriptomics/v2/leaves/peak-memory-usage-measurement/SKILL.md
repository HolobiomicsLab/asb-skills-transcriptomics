---
name: peak-memory-usage-measurement
description: Use when benchmarking manifold alignment or single-cell integration methods
  for scalability, specifically to compare peak memory demand and runtime across multiple
  methods on the same datasets.
when_to_use_negative:
- The goal is to assess alignment quality (e.g. label transfer accuracy or topology
  preservation) rather than computational cost — use the alignment benchmarking workflow
  instead.
- Datasets lack overlapping-free single-modality structure or are not the pre-processed
  PBMC/BM inputs, making runtime/memory comparisons against the paper's baselines
  not directly interpretable.
tools:
- name: scTopoGAN
  role: 'Method under profiling: get_TopoAE_Embeddings obtains per-modality manifolds
    and run_scTopoGAN aligns source to target; expected to be least memory-demanding'
  repo: github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Benchmark method applied via the UnionCom.py script while profiling the same
    resource metrics
  repo: github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Benchmark method applied via the manifold_align_mmd_pytorch.py script while
    profiling the same resource metrics
  repo: github.com/AkashCiel/scTopoGAN
- name: SCOT
  role: Benchmark method run under equivalent configurations; expected to be the fastest
    method
- name: Python
  role: Implementation language for all profiling and benchmark scripts
  repo: github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:7f9a473d3024d7f38576081ad1a08bd26eba2e63ef685a274e0bd3dfd81854e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CaSee
  - CopyKAT
  - MMD-MA
  - Python
  - SCEVAN
  - SCOT
  - UnionCom
  - ikarus
  - scTopoGAN
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# peak-memory-usage-measurement

## Summary

Profiles peak memory usage (and total runtime) of manifold alignment methods — scTopoGAN, UnionCom, MMD-MA, and SCOT — across the Full PBMC, Partial PBMC, and BM single-cell datasets to reproduce the computational resource comparison in Supplementary Table S1.

## When to use

Use when benchmarking manifold alignment or single-cell integration methods for scalability, specifically to compare peak memory demand and runtime across multiple methods on the same datasets.

## When NOT to use

- The goal is to assess alignment quality (e.g. label transfer accuracy or topology preservation) rather than computational cost — use the alignment benchmarking workflow instead.
- Datasets lack overlapping-free single-modality structure or are not the pre-processed PBMC/BM inputs, making runtime/memory comparisons against the paper's baselines not directly interpretable.

## Inputs

- Pre-processed Full PBMC dataset (from the scTopoGAN Data folder)
- Pre-processed Partial PBMC dataset (from the scTopoGAN Data folder)
- Pre-processed BM (bone marrow) dataset (from the scTopoGAN Data folder)
- Benchmark scripts: UnionCom.py, manifold_align_mmd_pytorch.py, scTopoGAN_results.py

## Outputs

- resource_usage.csv — peak memory usage and total runtime per method (scTopoGAN, UnionCom, MMD-MA, SCOT) per dataset (Full PBMC, Partial PBMC, BM)
- Supplementary Table S1-style comparison of memory and runtime

## How to apply

Run each alignment method on identical inputs — the pre-processed Full PBMC, Partial PBMC, and BM datasets from the scTopoGAN package's Data folder — profiling peak memory and total wall-clock time per run. For scTopoGAN, first obtain per-modality low-dimensional manifolds with get_TopoAE_Embeddings (Topological Autoencoder), then align source to target with run_scTopoGAN (topology-guided GAN). Run UnionCom via the UnionCom.py benchmark script, MMD-MA via manifold_align_mmd_pytorch.py, and SCOT under equivalent configurations. Aggregate peak memory and runtime per method per dataset into a single resource_usage.csv and check that scTopoGAN shows the lowest peak memory demand while SCOT shows the lowest runtime, matching Supplementary Table S1.

## Related tools

- **scTopoGAN** (Method under profiling: get_TopoAE_Embeddings obtains per-modality manifolds and run_scTopoGAN aligns source to target; expected to be least memory-demanding) — github.com/AkashCiel/scTopoGAN
- **UnionCom** (Benchmark method applied via the UnionCom.py script while profiling the same resource metrics) — github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Benchmark method applied via the manifold_align_mmd_pytorch.py script while profiling the same resource metrics) — github.com/AkashCiel/scTopoGAN
- **SCOT** (Benchmark method run under equivalent configurations; expected to be the fastest method)
- **Python** (Implementation language for all profiling and benchmark scripts) — github.com/AkashCiel/scTopoGAN

## Examples

```
python UnionCom.py  # run per dataset under the paper's configurations while profiling peak memory and runtime
```

## Evaluation signals

- resource_usage.csv contains a peak-memory and runtime value for every method-dataset pair (4 methods x 3 datasets)
- scTopoGAN shows the lowest peak memory usage across the three datasets, matching the reported finding
- SCOT shows the lowest total runtime across the three datasets, matching the reported finding
- All methods were run on the same pre-processed inputs from the Data folder under equivalent configurations, so resource metrics are comparable

## Limitations

- The resource comparison is tied to the specific Full PBMC, Partial PBMC, and BM datasets and the paper's configurations; results may differ on other data sizes or hardware.
- No changelog found in the scTopoGAN repository, so script or configuration drift over time cannot be tracked from the repo alone.

## Evidence

- [supplementary] Across the three datasets, scTopoGAN was the least memory-demanding method, while SCOT was the fastest method in terms of computation time.: [evidence span withheld — anchor status: not_found]
- [readme] this script takes as input the pre-processed data provided in the Data folder: "Please note, this script takes as input the pre-processed data provided in the ```Data``` folder."
- [methods] run_scTopoGAN which aligns the source manifold to the target manifold using the topology-guided GAN model: "2) ```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model."
- [readme] The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively.: "The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively."
