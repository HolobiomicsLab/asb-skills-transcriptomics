---
name: gcn-hyperparameter-optimization
description: Use when a semi-supervised GCN (e.g. scSHARP EdgeConv label propagation)
  must be tuned on scRNA-seq data with a set of confident consensus labels and you
  need the -validation-accuracy architecture recorded per dataset.
when_to_use_negative:
- No confident consensus labels exist yet — the validation split is built from confidently
  labeled cells produced by the upstream consensus step, so run that first
- You only need the final annotation outputs and already have a validated fixed architecture;
  sweeping hyperparameters is unnecessary recompute
- A dataset has too few confident labels for a 20% hold-out to yield a meaningful
  validation set
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: PyTorch
  role: Framework in which the scSHARP GCNModel with EdgeConv layers is implemented
    and trained during the hyperparameter sweep
  repo: https://github.com/lewinsohndp/scSHARP
- name: pytorch-cluster
  role: Provides neighbor sampling used to build the cell-neighbor graph for each
    candidate GCN configuration
  repo: https://github.com/lewinsohndp/scSHARP
- name: scSHARP
  role: Provides the GCNModel and pipeline whose batch size b, neighbors k, EdgeConv
    layers l, and embedding size e are swept
  repo: https://github.com/lewinsohndp/scSHARP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:da9a28cb5e132ec159362bfd4d8a76c557eb9508062edf89401d4f4da5aa9cca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lewinsohndp/scSHARP
  tools:
  - PyTorch
  - pytorch-cluster
  - scSHARP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gcn-hyperparameter-optimization

## Summary

Sweep the scSHARP GCNModel architecture hyperparameters (batch size b, neighbors k, EdgeConv layer count l, embedding size e) on confident-consensus-labeled cells and select the highest-validation-accuracy configuration per dataset.

## When to use

Use when a semi-supervised GCN (e.g. scSHARP EdgeConv label propagation) must be tuned on scRNA-seq data with a set of confident consensus labels and you need the best-validation-accuracy architecture recorded per dataset.

## When NOT to use

- No confident consensus labels exist yet — the validation split is built from confidently labeled cells produced by the upstream consensus step, so run that first
- You only need the final annotation outputs and already have a validated fixed architecture; sweeping hyperparameters is unnecessary recompute
- A dataset has too few confident labels for a 20% hold-out to yield a meaningful validation set

## Inputs

- Confident cell type labels from the upstream consensus step (task_001) for each dataset
- sim-0.7 and sim-0.8-sim simulated scRNA-seq datasets
- Testis scRNA-seq dataset (jung)
- PBMC scRNA-seq dataset

## Outputs

- selected_hyperparameters.csv with the highest-validation-accuracy GCN configuration per dataset
- Validation accuracy scores for each candidate configuration

## How to apply

Load the confident cell type labels from the upstream consensus step for each dataset (sim-0.7, sim-0.8-sim, testis, PBMC), and hold out 20% of confidently labeled cells as a validation split, subsampling PBMC to 5% of its confident cells. For each candidate configuration, build the cell-neighbor graph with pytorch-cluster neighbor sampling in PyTorch and train the scSHARP GCNModel with EdgeConv layers, sweeping batch size b, neighbors k, EdgeConv layer count l, and embedding size e. Score each configuration by validation accuracy on the held-out confidently labeled cells and keep the per-dataset optimum. In the published search the optima were batch size 20/20/50, k=2, 2 EdgeConv layers, and embedding size 25/40/25 for sim-0.7/sim-0.8-sim/testis/PBMC respectively. Write the selected hyperparameters per dataset to selected_hyperparameters.csv for reproducibility.

## Related tools

- **PyTorch** (Framework in which the scSHARP GCNModel with EdgeConv layers is implemented and trained during the hyperparameter sweep) — https://github.com/lewinsohndp/scSHARP
- **pytorch-cluster** (Provides neighbor sampling used to build the cell-neighbor graph for each candidate GCN configuration) — https://github.com/lewinsohndp/scSHARP
- **scSHARP** (Provides the GCNModel and pipeline whose batch size b, neighbors k, EdgeConv layers l, and embedding size e are swept) — https://github.com/lewinsohndp/scSHARP

## Evaluation signals

- Selected optima reproduce the reported values: batch size 20/20/50, k=2, 2 EdgeConv layers, embedding size 25/40/25 for sim-0.7/sim-0.8-sim/testis/PBMC
- Validation accuracy is computed on a 20% hold-out of confidently labeled cells (PBMC subsampled to 5% of confident cells)
- selected_hyperparameters.csv contains exactly one optimal configuration row per dataset
- Each selected configuration's validation accuracy exceeds all other swept (b, k, l, e) combinations for that dataset

## Limitations

- Optimal hyperparameters are dataset-specific (embedding size 25 vs 40 vs 25 across datasets), so a single configuration should not be assumed transferable
- Validation depends on the quality of the upstream confident consensus labels; noisy consensus labels will bias the selection
- Version numbers for scSHARP and its dependencies are not pinned in the paper's installation instructions

## Evidence

- [abstract] using state-of-the-art tools to find cells with highly confident label assignments through consensus: "using state-of-the-art tools to find cells with highly confident label assignments through consensus"
- [methods] conda install pytorch-cluster -c pyg: "``conda install pytorch-cluster -c pyg``"
- [readme] simulation_test_results.ipynb - Main results for both simulated data sets.: [evidence span withheld — anchor status: external_doc]
- [abstract] Our code for conducting the experiments in this paper and using our model is available at https://github.com/lewinsohndp/scSHARP.: "Our code for conducting the experiments in this paper and using our model is available at https://github.com/lewinsohndp/scSHARP."
