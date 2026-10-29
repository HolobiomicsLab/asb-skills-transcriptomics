---
name: batch-lisi-computation
description: Use when you have aligned joint embeddings from multiple single-cell
  modalities (e.g., Full PBMC, Partial PBMC, BM datasets aligned by scTopoGAN, UnionCom,
  MMD-MA, or SCOTv2) and need to quantify modality mixing for benchmarking.
when_to_use_negative:
- When cells from the modalities share overlapping features or cells and a supervised/feature-linked
  integration method is more appropriate — batch LISI is for evaluating unsupervised
  manifold alignment of non-overlapping modalities
- When the goal is to evaluate topology preservation of individual-modality low-dimensional
  representations (e.g., comparing topoAE against Autoencoder, VAE, or UMAP) rather
  than cross-modality mixing
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTopoGAN
  role: Produces the aligned embeddings whose modality mixing is scored by batch LISI;
    achieved the highest batch LISI (optimal 2) on Full PBMC, Partial PBMC, and BM
    datasets
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Baseline alignment method applied via UnionCom.py; its aligned embeddings
    are scored with batch LISI for benchmark comparison
- name: MMD-MA
  role: Baseline alignment method applied via manifold_align_mmd_pytorch.py with hyperparameter
    sweeps from Methods 2.7; scored with batch LISI for benchmark comparison
- name: SCOTv2
  role: Baseline alignment method run with Methods-2.7 settings; its aligned embeddings
    are scored with batch LISI
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:f96601ad2a6820d5b11892cdfbcc3b71e52f42d386bf422ef60816bd97ea2642
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - MMD-MA
  - SCOTv2
  - UnionCom
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# batch-lisi-computation

## Summary

Computes the batch Local Inverse Simpson Index (batch LISI) on aligned joint single-cell embeddings to quantify how well modalities (e.g., scRNA-seq and scATAC-seq) mix after manifold alignment; the optimal value is 2, indicating complete modality mixing. It is the key integration metric used to benchmark scTopoGAN against UnionCom, MMD-MA, and SCOT.

## When to use

Use when you have aligned joint embeddings from multiple single-cell modalities (e.g., Full PBMC, Partial PBMC, BM datasets aligned by scTopoGAN, UnionCom, MMD-MA, or SCOTv2) and need to quantify modality mixing for benchmarking.

## When NOT to use

- When cells from the modalities share overlapping features or cells and a supervised/feature-linked integration method is more appropriate — batch LISI is for evaluating unsupervised manifold alignment of non-overlapping modalities
- When the goal is to evaluate topology preservation of individual-modality low-dimensional representations (e.g., comparing topoAE against Autoencoder, VAE, or UMAP) rather than cross-modality mixing

## Inputs

- Aligned joint embedding matrix (e.g., .npy file) where cells from source and target modalities occupy a common space
- Per-cell batch/modality label vector identifying which modality each cell originates from
- Optionally per-cell Celltype and Subcelltype label vectors for companion matching metrics

## Outputs

- Batch LISI score per method-dataset combination (optimal value 2)
- Rows aggregated into a Table-2-equivalent CSV containing Celltype matching, Subcelltype matching, and batch LISI per method-dataset pair

## How to apply

After each alignment method (scTopoGAN via run_scTopoGAN, or the UnionCom.py and manifold_align_mmd_pytorch.py baselines, or SCOTv2 with the Methods 2.7 settings) produces aligned embeddings for a dataset, compute batch LISI on the joint embedding where each cell is labeled by its source modality/batch. Interpret the score against the optimal value of 2: a higher score means better mixing of the modalities. In the scTopoGAN benchmark, batch LISI is computed alongside Celltype matching and Subcelltype matching for every method-dataset combination (Full PBMC, Partial PBMC, BM), and all metrics are aggregated into a single Table-2-equivalent CSV with the aligned embeddings stored as .npy files. scTopoGAN achieved the highest batch LISI across all three datasets, which is the expected reference result when reproducing the benchmark.

## Related tools

- **scTopoGAN** (Produces the aligned embeddings whose modality mixing is scored by batch LISI; achieved the highest batch LISI (optimal 2) on Full PBMC, Partial PBMC, and BM datasets) — https://github.com/AkashCiel/scTopoGAN
- **UnionCom** (Baseline alignment method applied via UnionCom.py; its aligned embeddings are scored with batch LISI for benchmark comparison)
- **MMD-MA** (Baseline alignment method applied via manifold_align_mmd_pytorch.py with hyperparameter sweeps from Methods 2.7; scored with batch LISI for benchmark comparison)
- **SCOTv2** (Baseline alignment method run with Methods-2.7 settings; its aligned embeddings are scored with batch LISI)

## Examples

```
import numpy as np; emb = np.load('aligned_sctopogan_full_pbmc.npy'); batches = np.load('batches.npy'); # compute batch LISI on emb with batch labels, optimal score = 2
```

## Evaluation signals

- The output CSV contains exactly one batch LISI entry (plus Celltype matching and Subcelltype matching) per method-dataset combination
- Batch LISI values should reproduce the corresponding Table 2 results reported in the scTopoGAN preprint
- Batch LISI scores fall in the range up to the optimal value of 2, with higher values indicating better integration
- When reproducing the benchmark, scTopoGAN should show the highest batch LISI across all three datasets (Full PBMC, Partial PBMC, BM) compared to UnionCom, MMD-MA, and SCOT
- Aligned embeddings for each method-dataset pair are saved as .npy files alongside the metric table

## Limitations

- Batch LISI alone measures modality mixing and must be paired with Celltype matching and Subcelltype matching scores, since good mixing can occur at the cost of biological alignment
- Alignment quality can vary widely across GAN initializations — Celltype matching over 40 GANs was 37.1 ± 16.5% — so metric values depend on which candidate model is selected
- The benchmark relies on pre-processed data provided in the Data folder; different preprocessing may change LISI values

## Evidence

- [other] Batch LISI optimal value and scTopoGAN superiority: [evidence span withheld — anchor status: not_found]
- [other] Metric computation on aligned joint embeddings: [evidence span withheld — anchor status: not_found]
- [methods] Benchmark against UnionCom and MMD-MA: "The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively. Both methods were used to benchmark the performance of scTopoGAN."
- [abstract] Purpose of scTopoGAN alignment: "A topology-guided Generative Adversarial Network then aligns these latent representations into a common space."
