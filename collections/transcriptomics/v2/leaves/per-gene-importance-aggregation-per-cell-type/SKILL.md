---
name: per-gene-importance-aggregation-per-cell-type
description: Use when a trained scSHARP GCN model must be interpreted to identify
  which genes drive cell type classification, e.g. reproducing Figure 2 gene rankings
  for testis and PBMC cell types.
when_to_use_negative:
- When the goal is cell type annotation itself (label propagation), not feature interpretation
  — use the scSHARP GCN labeling workflow instead.
- When no trained GCN checkpoint exists — DeepLIFT attributions require the task_003
  model checkpoints, not raw expression data alone.
- When the input is a gene expression matrix that has not been provided as raw counts
  as scSHARP expects.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scSHARP
  role: Provides the GCNModel class whose per-tool output neurons are targeted for
    DeepLIFT attribution
  repo: https://github.com/lewinsohndp/scSHARP
- name: PyTorch
  role: Framework in which the completed pipeline, including interpretation of GCN
    classifications, is written
- name: pytorch-cluster
  role: PyTorch Geometric dependency required to run the scSHARP GCN
- name: Captum
  role: Runs DeepLIFT with the Rescale rule to compute per-gene attributions against
    the zero reference matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:24f1ae9366317e3ae44500f8fe5a1b594f1e36068ca5b98a26f45f831a24927e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lewinsohndp/scSHARP
  tools:
  - Captum
  - PyTorch
  - pytorch-cluster
  - scSHARP
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Per-gene importance aggregation per cell type

## Summary

Aggregate DeepLIFT (Captum, Rescale rule) attributions from the scSHARP GCN, targeting the per-tool output neurons for each consensus cell type, into per-gene importance scores (gene_importance_scores.csv) for the testis and PBMC datasets.

## When to use

Use when a trained scSHARP GCN model must be interpreted to identify which genes drive cell type classification, e.g. reproducing Figure 2 gene rankings for testis and PBMC cell types.

## When NOT to use

- When the goal is cell type annotation itself (label propagation), not feature interpretation — use the scSHARP GCN labeling workflow instead.
- When no trained GCN checkpoint exists — DeepLIFT attributions require the task_003 model checkpoints, not raw expression data alone.
- When the input is a gene expression matrix that has not been provided as raw counts as scSHARP expects.

## Inputs

- Trained scSHARP GCNModel checkpoints (PyTorch) from task_003
- Testis single-cell gene expression matrix
- PBMC single-cell gene expression matrix
- All-zero reference input matrix for gene expression features
- Per-tool output neurons and consensus cell type labels

## Outputs

- gene_importance_scores.csv for the testis dataset
- gene_importance_scores.csv for the PBMC dataset
- Per-gene importance rankings per cell type (e.g. CD8B top for CD8 T Cells in PBMCs; Ncl, Ldhc, Tnp1 for testis)

## How to apply

Load the trained GCN checkpoints from task_003 into scSHARP's GCNModel class (PyTorch with pytorch-cluster installed, installed via `conda install pytorch-cluster -c pyg`). Load the testis and PBMC single-cell expression matrices and construct an all-zero reference input matrix for the gene expression features. Run DeepLIFT with the Rescale rule via Captum, targeting the per-tool output neurons for each consensus cell type. Aggregate the resulting attributions into per-gene importance scores per cell type and write gene_importance_scores.csv for each dataset. The rationale is that the GCN 'allows for feature interpretation, identifying important genes for cell type classification' beyond simple consensus labeling.

## Related tools

- **scSHARP** (Provides the GCNModel class whose per-tool output neurons are targeted for DeepLIFT attribution) — https://github.com/lewinsohndp/scSHARP
- **PyTorch** (Framework in which the completed pipeline, including interpretation of GCN classifications, is written)
- **pytorch-cluster** (PyTorch Geometric dependency required to run the scSHARP GCN)
- **Captum** (Runs DeepLIFT with the Rescale rule to compute per-gene attributions against the zero reference matrix)

## Examples

```
conda install pytorch-cluster -c pyg
```

## Evaluation signals

- CD8B ranks as the most important gene for CD8 T Cell classification in PBMCs
- Ncl, Ldhc, and Tnp1 appear among top testis genes even though they were not explicitly included as marker genes
- CD74/FTH1/GNLY and CD79A/CD3D/CD3E appear among top PBMC genes (Figure 2 reproduction check)
- Tsga8/Acrv1 appear among top testis genes
- Attributions aggregated per cell type and written as gene_importance_scores.csv for both testis and PBMC datasets

## Limitations

- Attributions depend on the trained GCN checkpoints; without them, no importance scores can be computed.
- A zero reference matrix is used; alternative references may change per-gene rankings.
- The GCN approach shows only comparable results to a nonparametric neighbor majority approach, so importance scores should be interpreted alongside such baselines.
- Additional PyTorch Geometric dependencies such as torch-cluster may need to be manually installed depending on the system.

## Evidence

- [abstract] We then demonstrate that our GCN method allows for feature interpretation, identifying important genes for cell type classification.: "We then demonstrate that our GCN method allows for feature interpretation, identifying important genes for cell type classification."
- [results] DeepLIFT attribution on the GCN identifies biologically meaningful genes: CD8B ranks as the most important gene for CD8 T Cell classification in PBMCs.: [evidence span withheld — anchor status: not_found]
- [methods] GCN model class propagates labels from component tool predictions: "GCN model class propagates labels from component tool predictions"
- [methods] conda install pytorch-cluster -c pyg: "``conda install pytorch-cluster -c pyg``"
- [readme] Please input raw counts.: "See demo.ipynb for example work flow.
Please input raw counts."
