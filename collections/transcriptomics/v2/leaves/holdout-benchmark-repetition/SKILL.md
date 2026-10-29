---
name: holdout-benchmark-repetition
description: Use when quantifying whether adding supervision (pairwise must-link/cannot-link
  constraints from a labeled hold-out) consistently improves scRNA-seq clustering
  across incrementally varied parameter levels.
when_to_use_negative:
- The dataset has no partial cell-type labels from which must-link/cannot-link constraints
  can be built
- The goal is purely unsupervised clustering without evaluating the benefit of integrating
  prior/domain knowledge
- Only a single training run per setting is affordable — single-run scores are unreliable
  versus the ten-repeat mean protocol
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scDCC
  role: Deep constrained clustering model trained per constraint level; pairwise constraints
    are added as soft terms in its loss function
  repo: https://github.com/ttgump/scDCC
- name: PyTorch
  role: Framework (version 1.5.1+cu101) in which scDCC.py is implemented and trained
  repo: https://pytorch.org
- name: Scanpy
  role: Preprocessing library (version 1.0.4) for the scRNA-seq inputs
  repo: https://scanpy.readthedocs.io/en/stable
- name: Nvidia Tesla P100
  role: GPU hardware used to train the scDCC model for each benchmark setting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:414708ba02bd9dbca79e1d93471356d3ccff8a1db333a41d9ec98033603e49be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - Nvidia Tesla P100
  - PyTorch
  - Python
  - Scanpy
  - scDCC
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# holdout-benchmark-repetition

## Summary

Benchmark a clustering method by holding out a labeled fraction of cells to generate supervision signals, sweeping a parameter (e.g., number of pairwise constraints from 0 to 6000) and repeating over ten random splits to report mean NMI, CA, and ARI.

## When to use

Use when quantifying whether adding supervision (pairwise must-link/cannot-link constraints from a labeled hold-out) consistently improves scRNA-seq clustering across incrementally varied parameter levels.

## When NOT to use

- The dataset has no partial cell-type labels from which must-link/cannot-link constraints can be built
- The goal is purely unsupervised clustering without evaluating the benefit of integrating prior/domain knowledge
- Only a single training run per setting is affordable — single-run scores are unreliable versus the ten-repeat mean protocol

## Inputs

- scRNA-seq count matrices with cell-type labels for at least 10% of cells (10X PBMC, mouse bladder, worm neuron, human kidney from the scDCC figshare deposit)
- Script parameters: n_pairwise sweep values (0–6000), n_clusters per dataset, gamma, ml_weight, cl_weight

## Outputs

- Mean NMI, CA, and ARI per dataset per constraint level over ten random splits
- Aggregated table of mean clustering metrics ordered by number of pairwise constraints, reproducing the Figure 2 trend

## How to apply

Download the study datasets (e.g., 10X PBMC, mouse bladder, worm neuron, human kidney) from the figshare deposit, and hold out 10% of cells with known labels as the constraint source. Generate must-link and cannot-link pairwise constraints at incremental values of n_pairwise from 0 up to 6000 using the --n_pairwise argument of the scDCC scripts. Train scDCC (scDCC.py in PyTorch 1.5.1+cu101 on an Nvidia Tesla P100) with n_clusters set per dataset, gamma weighting the clustering loss, and ml_weight/cl_weight weighting the constraint loss terms. Evaluate clustering on the remaining 90% of cells with NMI, CA, and ARI, repeating each setting over ten random splits/repeats, then aggregate per-dataset, per-constraint-level mean metrics ordered by constraint count. Validation: the trend should show metrics rising with constraint count, with most values exceeding 0.8 at 6000 constraints.

## Related tools

- **scDCC** (Deep constrained clustering model trained per constraint level; pairwise constraints are added as soft terms in its loss function) — https://github.com/ttgump/scDCC
- **PyTorch** (Framework (version 1.5.1+cu101) in which scDCC.py is implemented and trained) — https://pytorch.org
- **Scanpy** (Preprocessing library (version 1.0.4) for the scRNA-seq inputs) — https://scanpy.readthedocs.io/en/stable
- **Nvidia Tesla P100** (GPU hardware used to train the scDCC model for each benchmark setting)

## Examples

```
python scDCC_pairwise_CITE_PBMC.py --n_pairwise 6000
```

## Evaluation signals

- Each clustering setting is repeated over ten random splits/repeats and the mean NMI/CA/ARI is recorded, not a single run
- Clustering metrics improve monotonically (or near-monotonically) as n_pairwise increases from 0 to 6000
- At 6000 pairwise constraints most NMI/CA/ARI values exceed 0.8 on all four datasets
- Evaluation is performed on the held-out 90% of cells, disjoint from the 10% label source used to build constraints

## Limitations

- Requires a labeled subset (10% here) to generate constraints; unlabeled datasets cannot seed the hold-out benchmark
- Performance gains are demonstrated only on the four small study datasets (10X PBMC, mouse bladder, worm neuron, human kidney)
- Discussion section content is absent from the provided document, so caveats about generalization are not available

## Evidence

- [abstract] Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance: "Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance"
- [methods] n_pairwise argument: "--n_pairwise: number of pairwise constraints want to generate"
- [methods] CITE PBMC benchmark script: "python scDCC_pairwise_CITE_PBMC.py"
- [methods] Datasets availability: "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
- [readme] Unsupervised motivation: "purely unsupervised clustering methods may not produce biologically interpretable clusters, which complicates cell type assignment"
