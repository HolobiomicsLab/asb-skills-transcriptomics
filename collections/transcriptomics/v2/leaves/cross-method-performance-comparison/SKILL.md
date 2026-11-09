---
name: cross-method-performance-comparison
description: Use when comparing the computational cost (peak memory and total runtime)
  of unsupervised single-cell manifold alignment methods such as scTopoGAN, UnionCom,
  MMD-MA, and SCOT on one or more pre-processed single-modality datasets.
when_to_use_negative:
- When the goal is to compare alignment quality (e.g., label-transfer or topology-preservation
  accuracy) rather than computational resource usage — use the alignment benchmark
  scripts instead.
- When datasets have overlapping features or cells, since methods that exploit overlaps
  are outside the complete unsupervised alignment setting this comparison targets.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scTopoGAN
  role: Method under benchmark; run get_TopoAE_Embeddings then run_scTopoGAN while
    profiling peak memory and runtime
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Produces the per-modality low-dimensional manifold inside scTopoGAN's resource
    profile
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Benchmarked competitor applied via the UnionCom.py script with resource profiling
  repo: https://github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Benchmarked competitor applied via the manifold_align_mmd_pytorch.py script
    with resource profiling
  repo: https://github.com/AkashCiel/scTopoGAN
- name: SCOT
  role: Benchmarked competitor run under equivalent configurations; identified as
    the fastest method
- name: Python
  role: Implementation and profiling environment for all benchmark scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:754adc1d5eb5e8187c2c916a6d2378ef9ccd30fa2c588c4fa2ce01d67c4fd828
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - MMD-MA
  - Python
  - SCOT
  - Topological Autoencoder
  - UnionCom
  - scTopoGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-method-performance-comparison

## Summary

Benchmarks the computational resource profile (peak memory usage and total runtime) of multiple unsupervised manifold alignment methods — scTopoGAN, UnionCom, MMD-MA, and SCOT — across several single-cell datasets, producing a unified resource-usage table as in Supplementary Table S1.

## When to use

Use when comparing the computational cost (peak memory and total runtime) of unsupervised single-cell manifold alignment methods such as scTopoGAN, UnionCom, MMD-MA, and SCOT on one or more pre-processed single-modality datasets.

## When NOT to use

- When the goal is to compare alignment quality (e.g., label-transfer or topology-preservation accuracy) rather than computational resource usage — use the alignment benchmark scripts instead.
- When datasets have overlapping features or cells, since methods that exploit overlaps are outside the complete unsupervised alignment setting this comparison targets.

## Inputs

- Pre-processed single-cell single-modality datasets from the Data folder (e.g., Full PBMC, Partial PBMC, BM)
- Benchmark scripts: scTopoGAN (Run_scTopoGAN.py / scTopoGAN_Functions.py), UnionCom.py, manifold_align_mmd_pytorch.py, and SCOT runs under equivalent configurations

## Outputs

- resource_usage.csv — peak memory usage per method per dataset and total runtime per method per dataset
- Comparative summary mirroring Supplementary Table S1 (memory vs. runtime ranking across methods)

## How to apply

Load the pre-processed datasets (e.g., Full PBMC, Partial PBMC, BM) from the Data folder of the scTopoGAN repository. For each dataset, run each method under equivalent configurations while profiling peak memory usage and total runtime: scTopoGAN via get_TopoAE_Embeddings (Topological Autoencoder per modality) followed by run_scTopoGAN (topology-guided GAN), UnionCom via UnionCom.py, MMD-MA via manifold_align_mmd_pytorch.py, and SCOT under equivalent settings. Keep profiling instrumentation identical across methods so memory and timing are comparable. Aggregate peak memory and total runtime per method per dataset into a single resource_usage.csv table and check the expected ordering: scTopoGAN least memory-demanding, SCOT fastest.

## Related tools

- **scTopoGAN** (Method under benchmark; run get_TopoAE_Embeddings then run_scTopoGAN while profiling peak memory and runtime) — https://github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Produces the per-modality low-dimensional manifold inside scTopoGAN's resource profile) — https://github.com/AkashCiel/scTopoGAN
- **UnionCom** (Benchmarked competitor applied via the UnionCom.py script with resource profiling) — https://github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Benchmarked competitor applied via the manifold_align_mmd_pytorch.py script with resource profiling) — https://github.com/AkashCiel/scTopoGAN
- **SCOT** (Benchmarked competitor run under equivalent configurations; identified as the fastest method)
- **Python** (Implementation and profiling environment for all benchmark scripts)

## Examples

```
python Run_scTopoGAN.py
```

## Evaluation signals

- resource_usage.csv contains one peak-memory and one runtime value for each of the 4 methods (scTopoGAN, UnionCom, MMD-MA, SCOT) across all 3 datasets (Full PBMC, Partial PBMC, BM)
- scTopoGAN shows the lowest peak memory demand across all datasets compared to the other three methods
- SCOT shows the lowest total runtime across all datasets compared to the other three methods
- Each method was run on identical pre-processed inputs from the Data folder under equivalent configurations, so memory/runtime differences are attributable to the methods themselves

## Limitations

- Results are hardware- and implementation-dependent; absolute runtimes and memory peaks may differ on other machines even if the method ordering holds.
- SCOT must be configured 'under equivalent configurations' for a fair comparison; mismatched parameter budgets invalidate the runtime comparison.
- The resource comparison does not measure alignment quality — a memory- or time-efficient method is not necessarily the most accurate aligner.

## Evidence

- [methods] UnionCom.py and manifold_align_mmd_pytorch.py: "The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively."
- [methods] run_scTopoGAN alignment function: "```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model."
- [methods] get_TopoAE_Embeddings function: "```get_TopoAE_Embeddings``` which obtains the low-dimensional manifold of each dataset separately using Topological Autoencoder"
- [readme] scTopoGAN_results.py reproduction script: "The ```scTopoGAN_results.py``` script reproduce the alignment results of scTopoGAN using different datasets."
- [readme] pre-processed Data folder input: "this script takes as input the pre-processed data provided in the ```Data``` folder"
- [abstract] unsupervised benchmarking scope: "We show that scTopoGAN outperforms state-of-the-art manifold alignment methods in complete unsupervised settings."
