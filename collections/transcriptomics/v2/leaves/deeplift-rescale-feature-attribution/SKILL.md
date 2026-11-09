---
name: deeplift-rescale-feature-attribution
description: Use when a trained scSHARP GCN model checkpoint exists and you need to
  identify which genes drive cell type classification (e.g., reproducing Figure 2
  gene rankings for testis and PBMC datasets).
when_to_use_negative:
- The GCN has not yet been trained — run the scSHARP consensus label propagation training
  (task_003 checkpoints) before attempting attribution.
- You need expression data that is not raw counts; the scSHARP README specifies 'Please
  input raw counts.'
- The model is not a graph convolutional network with per-cell-type output neurons
  — DeepLIFT targets here are the per-tool output neurons of the scSHARP GCN, not
  generic model heads.
tools:
- name: scSHARP
  role: Provides the trained GCNModel (PyTorch) whose per-tool output neurons are
    targeted by DeepLIFT for per-gene importance scoring
  repo: https://github.com/lewinsohndp/scSHARP
- name: PyTorch
  role: Framework in which the scSHARP GCN pipeline and DeepLIFT attribution are implemented
- name: pytorch-cluster
  role: PyTorch Geometric dependency that must be manually installed for the scSHARP
    GCN to run before attribution
- name: Captum
  role: Provides the DeepLIFT implementation with the Rescale rule used to compute
    attributions against the zero reference matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:c44961c46f84a1ca5cd835180bc74cb2d5bd387bb990790b98fdd6820541ceb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lewinsohndp/scSHARP
  tools:
  - Captum
  - PyTorch
  - pytorch-cluster
  - scSHARP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# deeplift-rescale-feature-attribution

## Summary

Interprets the trained scSHARP graph convolutional network by running DeepLIFT (Rescale rule, via Captum) with a zero reference matrix and per-cell-type target neurons, yielding per-gene importance scores for cell type classification.

## When to use

Use when a trained scSHARP GCN model checkpoint exists and you need to identify which genes drive cell type classification (e.g., reproducing Figure 2 gene rankings for testis and PBMC datasets).

## When NOT to use

- The GCN has not yet been trained — run the scSHARP consensus label propagation training (task_003 checkpoints) before attempting attribution.
- You need expression data that is not raw counts; the scSHARP README specifies 'Please input raw counts.'
- The model is not a graph convolutional network with per-cell-type output neurons — DeepLIFT targets here are the per-tool output neurons of the scSHARP GCN, not generic model heads.

## Inputs

- Trained scSHARP GCNModel PyTorch checkpoints (from the scSHARP_tool package)
- Testis single-cell gene expression matrix (raw counts)
- PBMC single-cell gene expression matrix (raw counts)
- Mapping of consensus cell types to GCN output neurons (per-tool output neurons)

## Outputs

- gene_importance_scores.csv for the testis dataset (per-gene DeepLIFT importance per cell type)
- gene_importance_scores.csv for the PBMC dataset (per-gene DeepLIFT importance per cell type)
- Ranked gene lists per cell type (e.g., CD8B top for CD8 T Cells in PBMCs)

## How to apply

Load the trained GCN model checkpoints from the scSHARP GCNModel class (PyTorch, with pytorch-cluster installed via `conda install pytorch-cluster -c pyg`), then load the testis and PBMC single-cell expression matrices from raw counts. Construct an all-zero reference input matrix for the gene expression features as the DeepLIFT baseline. Run DeepLIFT with the Rescale rule (Captum) on the GCN, targeting the per-tool output neurons for each consensus cell type. Aggregate attributions into per-gene importance scores per cell type and write gene_importance_scores.csv for the testis and PBMC datasets. The rationale is that scSHARP's GCN propagates confident consensus labels from component tools, so feature attribution on the GCN reveals genes supporting each label — including genes such as Ncl, Ldhc, and Tnp1 that were not explicitly provided as marker genes.

## Related tools

- **scSHARP** (Provides the trained GCNModel (PyTorch) whose per-tool output neurons are targeted by DeepLIFT for per-gene importance scoring) — https://github.com/lewinsohndp/scSHARP
- **PyTorch** (Framework in which the scSHARP GCN pipeline and DeepLIFT attribution are implemented)
- **pytorch-cluster** (PyTorch Geometric dependency that must be manually installed for the scSHARP GCN to run before attribution)
- **Captum** (Provides the DeepLIFT implementation with the Rescale rule used to compute attributions against the zero reference matrix)

## Examples

```
from captum.attr import DeepLift; dl = DeepLift(gcn_model); gene_scores = dl.attribute(expr_matrix, baselines=torch.zeros_like(expr_matrix), target=cell_type_idx)
```

## Evaluation signals

- CD8B ranks as the most important gene for CD8 T Cell classification in PBMCs (reproduces Figure 2).
- Ncl, Ldhc, and Tnp1 appear among top-ranked genes for testis cell types even though they were not explicitly included as marker genes.
- Tsga8 and Acrv1 appear among top testis genes; CD74/FTH1/GNLY and CD79A/CD3D/CD3E appear among top PBMC genes.
- Output file gene_importance_scores.csv exists for both testis and PBMC with per-gene, per-cell-type importance scores.
- Reference matrix is all zeros — verify baselines were not set to random or mean expression values, which would change Rescale-rule attributions.

## Limitations

- Attributions were validated only on the paper's testis and PBMC datasets; top-gene reproduction on other datasets is not guaranteed.
- The method interprets the GCN's consensus classification, so attribution quality depends on the component tools' confident labels propagated by the GCN.
- PyTorch Geometric dependencies such as torch-cluster may need to be manually installed depending on the system (per the README).
- Version numbers for scSHARP and its dependencies are not specified in the article's methods.

## Evidence

- [abstract] We then demonstrate that our GCN method allows for feature interpretation, identifying important genes for cell type classification.: "We then demonstrate that our GCN method allows for feature interpretation, identifying important genes for cell type classification."
- [methods] Our package utilizes a suite of cutting edge single cell classification tools which we have compiled as an R package: "Our package utilizes a suite of cutting edge single cell classification tools which we have compiled as an R package"
- [methods] ``conda install pytorch-cluster -c pyg``: "``conda install pytorch-cluster -c pyg``"
- [readme] Installation of torch and torch geometric required prior to pip install.: [evidence span withheld — anchor status: external_doc]
