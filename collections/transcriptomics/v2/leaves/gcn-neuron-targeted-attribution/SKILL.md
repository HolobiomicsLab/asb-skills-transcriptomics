---
name: gcn-neuron-targeted-attribution
description: Use when a trained scSHARP GCN model must be interpreted at the feature
  level — e.g. to recover which genes drive classification of specific cell types
  (CD8 T Cells, testis spermatogenic stages) after label propagation.
when_to_use_negative:
- The goal is to train or re-run the GCN label propagation itself rather than interpret
  an already-trained model's feature attributions.
- The model is not the scSHARP GCN (e.g. a plain consensus or neighbor-majority labeling
  approach) — there are no per-tool output neurons to target with DeepLIFT.
- Input is normalized/processed expression data instead of the raw counts the scSHARP
  pipeline expects.
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scSHARP
  role: Provides the trained GCNModel class whose per-tool output neurons are targeted
    for DeepLIFT attribution
  repo: https://github.com/mperozek11/scSHARP_tool
- name: PyTorch
  role: Framework in which the GCN model is loaded and run for attribution
- name: pytorch-cluster
  role: Dependency required to run the scSHARP GCN model in PyTorch
- name: Captum
  role: Implements DeepLIFT with the Rescale rule used to compute attributions against
    target neurons
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:ea199c3d014b222b17c3c0d283b9a3c390b9e968f71d2b6e7fb9a0bd4785206e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mperozek11/scSHARP_tool
  tools:
  - Captum
  - PyTorch
  - pytorch-cluster
  - scSHARP
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gcn-neuron-targeted-attribution

## Summary

Interpret a trained scSHARP semi-supervised GCN by running DeepLIFT (Rescale rule, via Captum) on the network with a zero reference matrix and per-cell-type target output neurons, yielding per-gene importance scores for cell type classification.

## When to use

Use when a trained scSHARP GCN model must be interpreted at the feature level — e.g. to recover which genes drive classification of specific cell types (CD8 T Cells, testis spermatogenic stages) after label propagation.

## When NOT to use

- The goal is to train or re-run the GCN label propagation itself rather than interpret an already-trained model's feature attributions.
- The model is not the scSHARP GCN (e.g. a plain consensus or neighbor-majority labeling approach) — there are no per-tool output neurons to target with DeepLIFT.
- Input is normalized/processed expression data instead of the raw counts the scSHARP pipeline expects.

## Inputs

- Trained scSHARP GCN model checkpoints (PyTorch, GCNModel class)
- Single-cell RNA-seq raw counts expression matrix for testis or PBMC data
- Consensus cell type labels per cell (from scSHARP consensus/GCN prediction stage)
- All-zero reference input matrix over the gene expression features

## Outputs

- gene_importance_scores.csv with per-gene importance scores per cell type (separate files for testis and PBMC datasets)
- Ranked gene lists per cell type (e.g. top genes for CD8 T Cell classification)

## How to apply

Load the trained GCN checkpoints (from the consensus label propagation stage, task_003) into scSHARP's GCNModel class in PyTorch, ensuring pytorch-cluster is installed. Load the single-cell raw counts expression matrices for the dataset of interest (testis from the Jung dataset, or PBMC), and construct an all-zero reference input matrix for the gene expression features. Run DeepLIFT with the Rescale rule through Captum, targeting the per-tool output neurons corresponding to each consensus cell type label rather than a single aggregated output. Aggregate neuron attributions into per-gene importance scores per cell type and write them as gene_importance_scores.csv. The rationale is that targeting individual output neurons maps attributions to specific cell type decisions, and DeepLIFT's Rescale rule with a zero reference propagates credit back through the graph convolutional layers to gene features.

## Related tools

- **scSHARP** (Provides the trained GCNModel class whose per-tool output neurons are targeted for DeepLIFT attribution) — https://github.com/mperozek11/scSHARP_tool
- **PyTorch** (Framework in which the GCN model is loaded and run for attribution)
- **pytorch-cluster** (Dependency required to run the scSHARP GCN model in PyTorch)
- **Captum** (Implements DeepLIFT with the Rescale rule used to compute attributions against target neurons)

## Examples

```
conda install pytorch-cluster -c pyg
```

## Evaluation signals

- CD8B ranks as the most important gene for CD8 T Cell classification in the PBMC gene_importance_scores.csv (reproducing Figure 2).
- Ncl, Ldhc, and Tnp1 appear among top-ranked testis genes even though they were not explicitly included as marker genes.
- Tsga8/Acrv1 are among top testis genes and CD74/FTH1/GNLY and CD79A/CD3D/CD3E among top PBMC genes.
- Attribution run completes with pytorch-cluster installed and the zero reference matrix shaped to match the gene expression feature dimension.
- Output file is a per-gene importance table with one score vector per consensus cell type / target neuron.

## Limitations

- Attributions require a trained GCN checkpoint; version numbers for scSHARP and its dependencies are not documented, so environment reproducibility depends on the README's conda/pip instructions.
- DeepLIFT attributions depend on the chosen zero reference input; results reflect credit relative to that reference.
- R tools are necessary for running scSHARP end-to-end, so the full pipeline has an R dependency even though attribution itself is Python-side.

## Evidence

- [abstract] We then demonstrate that our GCN method allows for feature interpretation, identifying important genes for cell type classification.: "We then demonstrate that our GCN method allows for feature interpretation, identifying important genes for cell type classification."
- [methods] GCN label propagation from component tool predictions: "GCN model class propagates labels from component tool predictions"
- [readme] torch-cluster dependency: [evidence span withheld — anchor status: external_doc]
- [readme] raw counts input requirement: [evidence span withheld — anchor status: external_doc]
- [readme] jung_results.ipynb covers testis dataset: [evidence span withheld — anchor status: external_doc]
