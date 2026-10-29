---
name: python-script-orchestration
description: Use when reproducing or extending scTopoGAN results from the AkashCiel/scTopoGAN
  repository, i.e., when running manifold alignment of single-cell datasets whose
  benchmark scripts are provided as standalone Python entry points.
when_to_use_negative:
- Datasets with overlapping features or cells that standard integration methods can
  link directly — the article notes current integration methods rely on such overlap,
  and this repository targets the non-overlapping setting
- Raw, unprocessed single-cell count matrices — the scripts take the pre-processed
  data in the Data folder, not raw inputs
- Manifold projection (not alignment) comparisons handled by a dedicated path — TopoAE_benchmark.py
  vs. regular Autoencoder, Variational Autoencoder and UMAP is its own script
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scTopoGAN
  role: Core method executed via Run_scTopoGAN.py and the scTopoGAN_Functions.py functions
    (get_TopoAE_Embeddings, run_scTopoGAN)
  repo: github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Obtains the low-dimensional manifold of each dataset separately inside get_TopoAE_Embeddings
  repo: github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Benchmark method applied via the UnionCom.py script
  repo: github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Benchmark method applied via the manifold_align_mmd_pytorch.py script
  repo: github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:3c4af8bc03708d8bea9595719b04c0171223e096615f2037f0ca68df610f7344
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MMD-MA
  - Python
  - Topological Autoencoder
  - UnionCom
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# python-script-orchestration

## Summary

Sequences the scTopoGAN repository's Python scripts (TopoAE_benchmark.py, scTopoGAN_results.py, UnionCom.py, manifold_align_mmd_pytorch.py) to execute the two-stage topoAE + topology-guided GAN pipeline and its UnionCom/MMD-MA benchmarks on the pre-processed Data-folder inputs.

## When to use

Use when reproducing or extending scTopoGAN results from the AkashCiel/scTopoGAN repository, i.e., when running manifold alignment of single-cell datasets whose benchmark scripts are provided as standalone Python entry points.

## When NOT to use

- Datasets with overlapping features or cells that standard integration methods can link directly — the article notes current integration methods rely on such overlap, and this repository targets the non-overlapping setting
- Raw, unprocessed single-cell count matrices — the scripts take the pre-processed data in the Data folder, not raw inputs
- Manifold projection (not alignment) comparisons handled by a dedicated path — TopoAE_benchmark.py vs. regular Autoencoder, Variational Autoencoder and UMAP is its own script

## Inputs

- Pre-processed single-cell datasets from the repository's Data folder (e.g., Full PBMC, Partial PBMC, BM)
- Repository scripts: Run_scTopoGAN.py, TopoAE_benchmark.py, scTopoGAN_results.py, UnionCom.py, manifold_align_mmd_pytorch.py
- scTopoGAN_Functions.py exposing get_TopoAE_Embeddings and run_scTopoGAN

## Outputs

- Low-dimensional manifold/embedding per modality from Topological Autoencoder
- Aligned source-to-target embedding from the topology-guided GAN
- Benchmark alignment results for UnionCom and MMD-MA
- Aggregated resource-usage table (peak memory and runtime per method per dataset), e.g., resource_usage.csv

## How to apply

Run the repository scripts in the order given by the README: first get_TopoAE_Embeddings (from scTopoGAN_Functions.py) computes the low-dimensional manifold of each modality with Topological Autoencoder, then run_scTopoGAN aligns the source manifold to the target manifold with the topology-guided GAN via Run_scTopoGAN.py. Use the pre-processed datasets shipped in the Data folder as inputs to these scripts rather than re-preprocessing raw counts. For benchmarking, execute UnionCom.py and manifold_align_mmd_pytorch.py on the same datasets, and use TopoAE_benchmark.py when comparing Topological Autoencoder against regular Autoencoder, Variational Autoencoder and UMAP. Aggregate per-method, per-dataset outputs (e.g., aligned embeddings and, for resource comparisons, peak memory and runtime) into a single table such as resource_usage.csv.

## Related tools

- **scTopoGAN** (Core method executed via Run_scTopoGAN.py and the scTopoGAN_Functions.py functions (get_TopoAE_Embeddings, run_scTopoGAN)) — github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Obtains the low-dimensional manifold of each dataset separately inside get_TopoAE_Embeddings) — github.com/AkashCiel/scTopoGAN
- **UnionCom** (Benchmark method applied via the UnionCom.py script) — github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Benchmark method applied via the manifold_align_mmd_pytorch.py script) — github.com/AkashCiel/scTopoGAN

## Examples

```
python Run_scTopoGAN.py
```

## Evaluation signals

- Run_scTopoGAN.py completes the two-step flow: get_TopoAE_Embeddings output exists for both modalities before run_scTopoGAN is invoked
- Alignment outputs are reproduced consistently with the scTopoGAN_results.py script on the same Data-folder datasets
- UnionCom.py and manifold_align_mmd_pytorch.py run on the identical pre-processed inputs used for scTopoGAN, so benchmark comparisons are apples-to-apples
- For the resource comparison, resource_usage.csv shows scTopoGAN with the lowest peak memory and SCOT with the lowest runtime across Full PBMC, Partial PBMC and BM datasets

## Limitations

- The scripts depend on the pre-processed data provided in the Data folder; new datasets require adaptation via Run_scTopoGAN.py as the worked example
- UnionCom and MMD-MA are executed as separate standalone benchmark scripts, so orchestration must ensure consistent configurations across methods
- No changelog is present in the repository, so script versions must be pinned externally

## Evidence

- [methods] Run_scTopoGAN.py example entry point: "The ```Run_scTopoGAN.py``` script provides an example for running scTopoGAN on a new data."
- [readme] two core functions: "1) ```get_TopoAE_Embeddings``` which obtains the low-dimensional manifold of each dataset separately using Topological Autoencoder, and 2) ```run_scTopoGAN``` which aligns the source"
- [readme] benchmark scripts: "The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively."
- [readme] pre-processed Data folder input: "this script takes as input the pre-processed data provided in the ```Data``` folder."
