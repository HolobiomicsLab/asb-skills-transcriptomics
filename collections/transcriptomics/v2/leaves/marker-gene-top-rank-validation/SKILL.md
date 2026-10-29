---
name: marker-gene-top-rank-validation
description: Use when interpreting a trained single-cell classifier (e.g., scSHARP
  GCN) with DeepLIFT and you need to confirm the per-gene importance rankings recover
  known cell-type markers.
when_to_use_negative:
- You only need cell type labels and not model interpretability — run the scSHARP
  consensus/GCN classification pipeline instead of attribution.
- The trained model or its checkpoints are unavailable; DeepLIFT attributions require
  the fitted GCN, not just predictions.
- Input expression data is not raw counts; the scSHARP README explicitly instructs
  to 'Please input raw counts.'
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scSHARP
  role: Provides the GCNModel class and per-tool output neurons whose attributions
    are interpreted
  repo: https://github.com/lewinsohndp/scSHARP
- name: PyTorch
  role: Framework hosting the trained GCN checkpoints on which DeepLIFT runs
  repo: https://github.com/lewinsohndp/scSHARP
- name: pytorch-cluster
  role: PyTorch Geometric dependency required to load and run the scSHARP GCN
- name: Captum
  role: Implements DeepLIFT with the Rescale rule used to compute per-gene attributions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:c4f5a70fe9167f231512ead2882b2b8b667b6b362a5a69b11dda38d584c2ab91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lewinsohndp/scSHARP
  tools:
  - Captum
  - PyTorch
  - pytorch-cluster
  - scSHARP
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Marker gene top-rank validation

## Summary

Validate GCN feature attributions by checking whether biologically canonical marker genes rank at the top of per-cell-type gene importance scores (e.g., CD8B for CD8 T cells, Tnp1/Ldhc for testis), reproducing Figure 2 of the scSHARP paper.

## When to use

Use when interpreting a trained single-cell classifier (e.g., scSHARP GCN) with DeepLIFT and you need to confirm the per-gene importance rankings recover known cell-type markers.

## When NOT to use

- You only need cell type labels and not model interpretability — run the scSHARP consensus/GCN classification pipeline instead of attribution.
- The trained model or its checkpoints are unavailable; DeepLIFT attributions require the fitted GCN, not just predictions.
- Input expression data is not raw counts; the scSHARP README explicitly instructs to 'Please input raw counts.'

## Inputs

- Trained scSHARP GCN model checkpoints (PyTorch, GCNModel class)
- Single-cell raw counts expression matrices for the testis (Jung) and PBMC datasets
- Consensus cell type labels from the scSHARP classification step
- Captum DeepLIFT (Rescale rule) attribution configuration

## Outputs

- gene_importance_scores.csv for the testis dataset (per-gene importance per cell type)
- gene_importance_scores.csv for the PBMC dataset (per-gene importance per cell type)
- Ranked top-gene lists per cell type for comparison against Figure 2

## How to apply

Load the trained GCN checkpoints into scSHARP's GCNModel class (PyTorch with pytorch-cluster installed) and run DeepLIFT with the Rescale rule via Captum, using an all-zero reference expression matrix and targeting the per-tool output neuron for each consensus cell type. Aggregate per-cell attributions into per-gene importance scores per cell type and export them as gene_importance_scores.csv for each dataset (testis, PBMC). Then inspect the top-ranked genes: for PBMCs, CD8B should rank first for CD8 T Cell classification, with CD74/FTH1/GNLY and CD79A/CD3D/CD3E among top PBMC genes; for testis, Ncl, Ldhc, Tnp1 plus Tsga8/Acrv1 should appear among top genes. Crucially, some top genes (Ncl, Ldhc, Tnp1) were not explicitly supplied as marker genes, so their emergence is evidence the GCN learned classification-relevant features rather than echoing inputs.

## Related tools

- **scSHARP** (Provides the GCNModel class and per-tool output neurons whose attributions are interpreted) — https://github.com/lewinsohndp/scSHARP
- **PyTorch** (Framework hosting the trained GCN checkpoints on which DeepLIFT runs) — https://github.com/lewinsohndp/scSHARP
- **pytorch-cluster** (PyTorch Geometric dependency required to load and run the scSHARP GCN)
- **Captum** (Implements DeepLIFT with the Rescale rule used to compute per-gene attributions)

## Examples

```
from captum.attr import DeepLift; dl = DeepLift(gcn_model); attributions = dl.attribute(inputs=expr_matrix, baselines=zero_ref, target=cell_type_neuron)
```

## Evaluation signals

- CD8B ranks as the single most important gene for CD8 T Cell classification in the PBMC gene_importance_scores.csv
- Ncl, Ldhc, and Tnp1 appear among top-ranked testis genes despite not being explicitly included as marker genes
- Tsga8 and Acrv1 appear among top testis genes, and CD74/FTH1/GNLY plus CD79A/CD3D/CD3E among top PBMC genes, matching Figure 2
- Attribution is computed against an all-zero reference matrix and the correct per-tool output neuron for each consensus cell type
- Output files are well-formed CSVs with per-gene importance scores per cell type for both testis and PBMC datasets

## Limitations

- Attributions depend on the choice of reference input; this workflow uses an all-zero reference matrix, and other references could shift rankings.
- Validation is qualitative marker recovery (Figure 2 reproduction), not a quantitative accuracy metric.
- torch-cluster and other PyTorch Geometric dependencies may need manual installation; missing them will prevent loading the GCN model.
- Environment setup is version-sensitive: Python 3.9 conda environment with torch and torch geometric installed prior to pip install of scSHARP.

## Evidence

- [other] CD8B ranks as the most important gene for CD8 T Cell classification in PBMCs, and Ncl, Ldhc, and Tnp1 are identified for testis cell types even though they were not explicitly included as marker genes.: [evidence span withheld — anchor status: not_found]
- [abstract] We then demonstrate that our GCN method allows for feature interpretation, identifying important genes for cell type classification.: "We then demonstrate that our GCN method allows for feature interpretation, identifying important genes for cell type classification."
- [other] Load the trained GCN model checkpoints from task_003 into scSHARP's GCNModel class (PyTorch, with pytorch-cluster installed).: [evidence span withheld — anchor status: not_found]
- [other] Run DeepLIFT with the Rescale rule (Captum) on the GCN, targeting the per-tool output neurons for each consensus cell type.: [evidence span withheld — anchor status: not_found]
- [other] Our code for conducting the experiments in this paper and using our model is available at https://github.com/lewinsohndp/scSHARP.: "Our code for conducting the experiments in this paper and using our model is available at https://github.com/lewinsohndp/scSHARP."
- [readme] Please input raw counts.: [evidence span withheld — anchor status: external_doc]
