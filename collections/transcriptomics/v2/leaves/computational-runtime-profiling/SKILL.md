---
name: computational-runtime-profiling
description: Use when comparing computational cost (peak memory and runtime) of single-cell
  manifold-alignment methods — scTopoGAN, UnionCom, MMD-MA, SCOT — on the Full PBMC,
  Partial PBMC, or BM datasets.
when_to_use_negative:
- The goal is alignment-quality comparison (e.g. topology preservation or integration
  accuracy) rather than resource cost — use the TopoAE_benchmark.py / scTopoGAN_results.py
  comparison workflow instead
- The method under evaluation has no benchmark entry point in the scTopoGAN repository
  (e.g. a custom pipeline without a script equivalent to UnionCom.py or manifold_align_mmd_pytorch.py),
  so resource metrics would not be comparable
tools:
- name: scTopoGAN
  role: Method being profiled; run via get_TopoAE_Embeddings then run_scTopoGAN while
    capturing memory and runtime
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Produces the per-modality low-dimensional manifold whose runtime is part of
    scTopoGAN's total cost
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Benchmark method run via UnionCom.py to profile memory and runtime
  repo: https://github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Benchmark method run via manifold_align_mmd_pytorch.py to profile memory and
    runtime
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Python
  role: Implementation language for all benchmark scripts and profiling wrapper
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:324feb5772e68eff344c2dbbb5e116a60aa7b0b33dc39a9bf916c4698df626aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - MMD-MA
  - Python
  - Topological Autoencoder
  - UnionCom
  - scTopoGAN
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# computational-runtime-profiling

## Summary

Profiles peak memory usage and total runtime of scTopoGAN against UnionCom, MMD-MA, and SCOT across the Full PBMC, Partial PBMC, and BM datasets, reproducing the Supplementary Table S1 resource comparison.

## When to use

Use when comparing computational cost (peak memory and runtime) of single-cell manifold-alignment methods — scTopoGAN, UnionCom, MMD-MA, SCOT — on the Full PBMC, Partial PBMC, or BM datasets.

## When NOT to use

- The goal is alignment-quality comparison (e.g. topology preservation or integration accuracy) rather than resource cost — use the TopoAE_benchmark.py / scTopoGAN_results.py comparison workflow instead
- The method under evaluation has no benchmark entry point in the scTopoGAN repository (e.g. a custom pipeline without a script equivalent to UnionCom.py or manifold_align_mmd_pytorch.py), so resource metrics would not be comparable

## Inputs

- Pre-processed Full PBMC, Partial PBMC, and BM single-cell datasets from the repository's Data folder
- scTopoGAN benchmark scripts (scTopoGAN_results.py, UnionCom.py, manifold_align_mmd_pytorch.py) and the SCOT benchmark configuration

## Outputs

- resource_usage.csv containing peak memory usage and total runtime per method (scTopoGAN, UnionCom, MMD-MA, SCOT) per dataset (Full PBMC, Partial PBMC, BM), matching Supplementary Table S1

## How to apply

Load the pre-processed Full PBMC, Partial PBMC, and BM datasets provided in the repository's ```Data``` folder. For scTopoGAN, first obtain per-modality low-dimensional manifolds with ```get_TopoAE_Embeddings``` (Topological Autoencoder), then align source to target with ```run_scTopoGAN``` (topology-guided GAN), wrapping each run with memory/runtime instrumentation. Run UnionCom via the ```UnionCom.py``` benchmark script and MMD-MA via ```manifold_align_mmd_pytorch.py``` under the paper's configurations, and run SCOT under equivalent settings, profiling the same metrics. Record peak memory and total wall-clock time per method per dataset, aggregate into a single CSV (resource_usage.csv), and sanity-check the expected pattern: scTopoGAN shows the lowest peak memory demand while SCOT is the fastest in computation time.

## Related tools

- **scTopoGAN** (Method being profiled; run via get_TopoAE_Embeddings then run_scTopoGAN while capturing memory and runtime) — https://github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Produces the per-modality low-dimensional manifold whose runtime is part of scTopoGAN's total cost) — https://github.com/AkashCiel/scTopoGAN
- **UnionCom** (Benchmark method run via UnionCom.py to profile memory and runtime) — https://github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Benchmark method run via manifold_align_mmd_pytorch.py to profile memory and runtime) — https://github.com/AkashCiel/scTopoGAN
- **Python** (Implementation language for all benchmark scripts and profiling wrapper) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
python UnionCom.py  # run under the paper's configuration while profiling peak memory and total runtime on Full PBMC
```

## Evaluation signals

- resource_usage.csv contains 12 rows (4 methods x 3 datasets) each with a peak-memory value and a total-runtime value
- scTopoGAN has the lowest peak memory of the four methods on each of the Full PBMC, Partial PBMC, and BM datasets
- SCOT has the lowest runtime of the four methods on each dataset, matching the Supplementary Table S1 pattern
- All methods were run on the same pre-processed inputs from the Data folder under equivalent configurations, so differences reflect the methods rather than data preprocessing

## Limitations

- Resource figures are specific to the Full PBMC, Partial PBMC, and BM datasets and the paper's configurations; they do not generalize to other datasets or hardware
- The scTopoGAN repository contains no changelog, so version-specific performance differences between runs may be untraceable
- Runtime ordering (SCOT fastest) is independent of the memory ordering (scTopoGAN least memory-demanding); a fast method is not necessarily memory-efficient and vice versa

## Evidence

- [supplementary] Across the three datasets, scTopoGAN was the least memory-demanding method, while SCOT was the fastest method in terms of computation time.: [evidence span withheld — anchor status: not_found]
- [methods] benchmark scripts: "The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively."
- [readme] pre-processed data location: "Please note, this script takes as input the pre-processed data provided in the ```Data``` folder."
- [methods] scTopoGAN run steps: "```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model."
