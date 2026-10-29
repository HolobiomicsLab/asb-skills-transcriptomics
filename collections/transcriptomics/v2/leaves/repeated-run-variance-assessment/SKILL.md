---
name: repeated-run-variance-assessment
description: Use when benchmarking a stochastic clustering model (e.g., DeepCCI's
  GCN cluster model with random initialization) against baselines on labeled scRNA-seq
  datasets and you must control seed variance by averaging ARI over multiple repeats.
when_to_use_negative:
- The clustering pipeline is fully deterministic with a fixed output — repeated runs
  provide no information about seed variance
- Ground-truth cell-type labels are unavailable for the dataset, so ARI cannot be
  computed per run
- You only need a single qualitative clustering result, not a benchmark comparison
  against other methods
edam_operation: http://edamontology.org/operation_2948
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DeepCCI
  role: Stochastic GCN-based clustering model whose run-to-run ARI variance is assessed
    via 10 repeats per dataset using Cluster.py
  repo: https://github.com/JiangBioLab/DeepCCI
- name: Seurat
  role: R baseline clustering method (Louvain and Leiden) run with 10 repeats on the
    same 12 datasets for variance-controlled comparison
- name: Python 3.7.4
  role: Runtime environment (conda deepcciEnv) executing Cluster.py repeats on GPU
    for the target model and most baselines
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:f2b007aea11013bce258cae1985ecfd3758bf88fd8f67dc85cafe5ce58eee277
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiangBioLab/DeepCCI
  tools:
  - DeepCCI
  - Python
  - Python 3.7.4
  - R
  - Seurat
  - conda
  - git
  - igraph
  - pip
  edam_operation: http://edamontology.org/operation_2948
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# repeated-run-variance-assessment

## Summary

Assess run-to-run (seed) variance of a stochastic clustering pipeline by repeating each clustering run 10 times per dataset and aggregating mean ARI against ground-truth cell-type labels, as done in the DeepCCI vs 13-baseline benchmark on 12 scRNA-seq datasets.

## When to use

Use when benchmarking a stochastic clustering model (e.g., DeepCCI's GCN cluster model with random initialization) against baselines on labeled scRNA-seq datasets and you must control seed variance by averaging ARI over multiple repeats.

## When NOT to use

- The clustering pipeline is fully deterministic with a fixed output — repeated runs provide no information about seed variance
- Ground-truth cell-type labels are unavailable for the dataset, so ARI cannot be computed per run
- You only need a single qualitative clustering result, not a benchmark comparison against other methods

## Inputs

- scRNA-seq datasets with ground-truth cell-type labels (CSV or h5 files, e.g., Yan GSE36552, Quake_10x_Limb_Muscle GSE109774)
- Clustering predictions from each individual run of the model under test
- Ground-truth cell-type label files for ARI computation

## Outputs

- Per-run ARI scores for each method on each dataset (10 runs per method-dataset pair)
- Aggregated mean ARI table per method per dataset (Supplementary Data S4 style)
- Run-to-run variance/spread estimate per method-dataset pair

## How to apply

Run the stochastic clustering pipeline (e.g., DeepCCI's Cluster.py with random initialization) 10 times per dataset, repeating the same protocol for every baseline (scTAG, Graph-sc, scGNN, scGAE, GraphSCC, scziDesk, scDCC, DCA, DEC, K-means, Spectral clustering, and Seurat-based Louvain and Leiden in R) on each of the 12 GEO scRNA-seq datasets (CSV or h5 format, e.g., Yan GSE36552 and Quake_10x_Limb_Muscle GSE109774). Keep the configuration identical across repeats — either --pretain True --pretrain_epoch 50 on GPU, or --pretain False using the provided pretain_model folder files to cut runtime. Score every run's clustering against the ground-truth labels using the Adjusted Rand Index, then aggregate mean ARI per method per dataset into a summary table matching Supplementary Data S4. Judge correctness by confirming per-run ARI was recorded before aggregation (so variance is controlled, not hidden) and that the averaged results reproduce the reported benchmark, e.g., ARI 0.9848 for DeepCCI on Quake_10x_Limb_Muscle.

## Related tools

- **DeepCCI** (Stochastic GCN-based clustering model whose run-to-run ARI variance is assessed via 10 repeats per dataset using Cluster.py) — https://github.com/JiangBioLab/DeepCCI
- **Seurat** (R baseline clustering method (Louvain and Leiden) run with 10 repeats on the same 12 datasets for variance-controlled comparison)
- **Python 3.7.4** (Runtime environment (conda deepcciEnv) executing Cluster.py repeats on GPU for the target model and most baselines)

## Examples

```
python Cluster.py --name Quake_10x_Limb_Muscle --pretain False --device cuda
```

## Evaluation signals

- Exactly 10 runs per method per dataset were executed, with per-run ARI recorded before aggregation
- Mean ARI per method per dataset reproduces the pattern of Supplementary Data S4
- DeepCCI attains the highest ARI among all 14 methods on all 12 datasets
- ARI of 0.9848 is reproduced on Quake_10x_Limb_Muscle (GSE109774) when averaging repeats
- Per-run ARI spread is nonzero and captured, confirming the model is stochastic and variance is controlled rather than cherry-picked from a single run

## Limitations

- Requires ground-truth labels; ARI cannot be computed for unlabeled datasets
- Compute cost scales with 10 repeats × 14 methods × 12 datasets; pretraining (~25 minutes with --pretain True) is slower than reusing pretain_model files (~5 minutes)
- Preprocessing each dataset takes about 5-10 minutes depending on file size, adding to total benchmark runtime
- Single-run clustering results may not be reproducible due to random initialization; only repeated-run means are trustworthy for reporting

## Evidence

- [other] repeating 10 runs per dataset to control seed variance: [evidence span withheld — anchor status: not_found]
- [other] DeepCCI best ARI on all 12 datasets: [evidence span withheld — anchor status: not_found]
- [methods] Cluster.py pretrain invocation: "python Cluster.py --name Yan --pretain True --pretrain_epoch 50 --device cuda"
- [readme] Repeatable clustering command on Quake_10x_Limb_Muscle: "python Cluster.py --name Quake_10x_Limb_Muscle --pretain False --device cuda"
- [other] ARI scoring against ground truth: [evidence span withheld — anchor status: not_found]
