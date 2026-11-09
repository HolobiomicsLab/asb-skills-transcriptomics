---
name: adjusted-rand-index-evaluation
description: Use when comparing clustering assignments from single-cell RNA-seq methods
  against known cell-type labels, or aggregating ARI across repeated runs to benchmark
  a clustering method against alternatives.
when_to_use_negative:
- The datasets lack ground-truth cell-type labels, making an external clustering metric
  inapplicable
- Evaluating downstream CCI inference outputs rather than cluster assignments — that
  requires interaction-level validation, not ARI
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: DeepCCI
  role: Cluster model whose clustering outputs are scored by ARI against ground truth
  repo: https://github.com/JiangBioLab/DeepCCI
- name: Seurat
  role: R-based baseline clustering (Louvain and Leiden) whose results are also scored
    with ARI
- name: Python
  role: Environment in which ARI is computed for DeepCCI and most baselines
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:649f01b494c05c7784b3bc50a84ea376bbe48ac7c50a7aa9200590c7e71b40b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiangBioLab/DeepCCI
  tools:
  - DeepCCI
  - Python
  - R
  - Seurat
  - conda
  - git
  - igraph
  - pip
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# adjusted-rand-index-evaluation

## Summary

Scores single-cell clustering results against ground-truth cell-type labels using the Adjusted Rand Index (ARI), as used in DeepCCI's benchmark of its cluster model against 13 baselines (scTAG, scGNN, scDCC, Seurat, etc.) on 12 public scRNA-seq datasets.

## When to use

Use when comparing clustering assignments from single-cell RNA-seq methods against known cell-type labels, or aggregating ARI across repeated runs to benchmark a clustering method against alternatives.

## When NOT to use

- The datasets lack ground-truth cell-type labels, making an external clustering metric inapplicable
- Evaluating downstream CCI inference outputs rather than cluster assignments — that requires interaction-level validation, not ARI

## Inputs

- Clustering label assignments from DeepCCI's Cluster.py or from baseline methods (scTAG, scGNN, scDCC, Seurat, K-means, etc.)
- Ground-truth cell-type labels for the 12 public scRNA-seq datasets (e.g. Yan / GSE36552 in CSV format, Quake_10x_Limb_Muscle / GSE109774 in h5 format)
- Per-run seeds from 10 repeated runs per method per dataset

## Outputs

- ARI value per clustering result against ground-truth labels
- Mean ARI per method per dataset aggregated over 10 repeats
- Benchmark table matching Supplementary Data S4

## How to apply

Run each clustering method on the same labeled datasets — in the DeepCCI benchmark, the cluster model (python Cluster.py, e.g. with --pretain True --pretrain_epoch 50 --device cuda) and 13 baselines (scTAG, Graph-sc, scGNN, scGAE, GraphSCC, scziDesk, scDCC, DCA, DEC, K-means, Spectral clustering, and Seurat-based Louvain/Leiden in R) are each repeated 10 runs per dataset to control seed variance. Score every clustering result against the ground-truth cell-type labels with the Adjusted Rand Index. Aggregate mean ARI per method per dataset into a comparison table (matching Supplementary Data S4). A method is judged superior when it attains the highest ARI across all datasets — DeepCCI reached the best ARI on all 12 datasets, including 0.9848 on Quake_10x_Limb_Muscle (GSE109774).

## Related tools

- **DeepCCI** (Cluster model whose clustering outputs are scored by ARI against ground truth) — https://github.com/JiangBioLab/DeepCCI
- **Seurat** (R-based baseline clustering (Louvain and Leiden) whose results are also scored with ARI)
- **Python** (Environment in which ARI is computed for DeepCCI and most baselines)

## Examples

```
python Cluster.py --name Quake_10x_Limb_Muscle --pretain False --device cuda
```

## Evaluation signals

- ARI must be computed for every method on all 12 datasets, each with 10 repeated runs to control seed variance
- DeepCCI must achieve the highest ARI among all 14 methods on all 12 datasets for the benchmark to validate
- Reported ARI of 0.9848 on Quake_10x_Limb_Muscle (GSE109774) must be reproduced
- Aggregated mean ARI table should match Supplementary Data S4

## Limitations

- Seed variance affects clustering results, so single runs are unreliable — 10 repeats per method per dataset are needed
- Validation depends on datasets having trusted ground-truth cell-type labels

## Evidence

- [supplementary] task_004 finding: [evidence span withheld — anchor status: not_found]
- [methods] task_004 workflow: [evidence span withheld — anchor status: not_found]
- [readme] readme: "python Cluster.py --name Yan --pretain True --pretrain_epoch 50 --device cuda"
