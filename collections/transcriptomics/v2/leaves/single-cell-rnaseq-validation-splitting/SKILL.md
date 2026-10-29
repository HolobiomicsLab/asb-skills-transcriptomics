---
name: single-cell-rnaseq-validation-splitting
description: Use when tuning the scSHARP GCNModel (batch size, neighbor count k, EdgeConv
  layer count l, embedding size e) on scRNA-seq data with upstream consensus confident
  labels, and you need held-out cells to score candidate configurations.
when_to_use_negative:
- The dataset has no confidently labeled cells from the upstream consensus step —
  there is nothing to hold out for validation scoring
- The goal is final cell type annotation rather than hyperparameter selection — use
  the selected configuration to run the full scSHARP GCN pipeline instead
- Inputs are already optimal hyperparameters for the target dataset (e.g. the reported
  optima for sim-0.7/sim-0.8-sim/testis/PBMC) and no new dataset is being tuned
tools:
- name: scSHARP
  role: Provides the GCNModel with EdgeConv layers whose configurations are scored
    on the validation splits
  repo: https://github.com/mperozek11/scSHARP_tool
- name: pytorch-cluster
  role: Neighbor sampling used to build the cell-neighbor graph for training and validation
- name: PyTorch
  role: Framework in which the GCN training and validation-accuracy evaluation are
    implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:7c7c8433b0609253b33d822b06685bcc0824b315f3f4328a85570d30c3bd4b43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mperozek11/scSHARP_tool
  tools:
  - PyTorch
  - pytorch-cluster
  - scSHARP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# single-cell-rnaseq-validation-splitting

## Summary

Constructs held-out validation splits from confidently labeled single cells to score scSHARP GCN configurations during hyperparameter optimization (batch size b, neighbors k, EdgeConv layers l, embedding size e). Validation accuracy on the held-out confident cells is the selection criterion.

## When to use

Use when tuning the scSHARP GCNModel (batch size, neighbor count k, EdgeConv layer count l, embedding size e) on scRNA-seq data with upstream consensus confident labels, and you need held-out cells to score candidate configurations.

## When NOT to use

- The dataset has no confidently labeled cells from the upstream consensus step — there is nothing to hold out for validation scoring
- The goal is final cell type annotation rather than hyperparameter selection — use the selected configuration to run the full scSHARP GCN pipeline instead
- Inputs are already optimal hyperparameters for the target dataset (e.g. the reported optima for sim-0.7/sim-0.8-sim/testis/PBMC) and no new dataset is being tuned

## Inputs

- Confident cell type labels from the upstream consensus step (task_001)
- scRNA-seq datasets: sim-0.7, sim-0.8-sim, testis (jung), and PBMC (raw counts)
- Candidate GCN configurations: batch size b, neighbor count k, EdgeConv layer count l, embedding size e

## Outputs

- Per-configuration validation accuracy scores on held-out confident cells
- selected_hyperparameters.csv with the highest-validation-accuracy configuration per dataset

## How to apply

Load the confident cell type labels produced by the upstream consensus step (task_001) together with the sim-0.7, sim-0.8-sim, testis, and PBMC scRNA-seq datasets. Build validation splits by holding out 20% of confidently labeled cells per dataset, and subsample PBMC to 5% of its confident cells because of its size. For each candidate configuration, build the cell-neighbor graph (using pytorch-cluster neighbor sampling in PyTorch) and train the scSHARP GCNModel with EdgeConv layers, sweeping batch size b, neighbors k, EdgeConv layer count l, and embedding size e. Score each configuration by validation accuracy computed only on the held-out confidently labeled cells, then select the configuration with the highest validation accuracy per dataset and write the selections to selected_hyperparameters.csv. The rationale is to keep the labeled-cell budget focused on semi-supervised propagation while reserving confident cells as an unbiased accuracy signal.

## Related tools

- **scSHARP** (Provides the GCNModel with EdgeConv layers whose configurations are scored on the validation splits) — https://github.com/mperozek11/scSHARP_tool
- **pytorch-cluster** (Neighbor sampling used to build the cell-neighbor graph for training and validation)
- **PyTorch** (Framework in which the GCN training and validation-accuracy evaluation are implemented)

## Examples

```
conda install pytorch-cluster -c pyg
```

## Evaluation signals

- The selected optima match the reported values: batch 20/20/50, k=2, 2 EdgeConv layers, embedding size 25/40/25 for sim-0.7/sim-0.8-sim/testis/PBMC respectively
- selected_hyperparameters.csv is produced with one selected configuration per dataset (sim-0.7, sim-0.8-sim, testis, PBMC)
- Validation accuracy is computed only on the held-out confidently labeled cells (20% holdout; PBMC subsampled to 5% of confident cells)
- Every swept configuration reports a finite validation accuracy, with no leakage of held-out confident cells into training

## Limitations

- PBMC must be subsampled to 5% of its confident cells, which may reduce the representativeness of the validation split for rare cell types
- Requires the upstream consensus step (task_001) to already have produced confident cell type labels; the split quality depends on that consensus's confidence
- Version numbers for scSHARP and its dependencies are not documented in the article, so reproducing exact validation accuracies may be sensitive to the installed versions

## Evidence

- [other] 20% holdout of confident cells with PBMC subsampling: [evidence span withheld — anchor status: not_found]
- [other] Validation accuracy as the selection criterion: [evidence span withheld — anchor status: not_found]
- [other] Expected selected hyperparameters: [evidence span withheld — anchor status: not_found]
- [methods] pytorch-cluster dependency for graph construction: "``conda install pytorch-cluster -c pyg``"
- [readme] GCN training environment setup: "conda create -n <env name> python=3.9
conda activate <env name>"
