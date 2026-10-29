---
name: transformer-model-training-single-cell-rnaseq
description: Use when a labeled single-cell reference AnnData must be turned into
  a cell type annotation model (e.g., training TOSICA with GMT masks on mouse/human
  data), or when comparing convergence of knowledge-based masks vs random masks as
  in Supplementary Fig. 8.
when_to_use_negative:
- The input is an unlabeled query dataset with no reference training labels — use
  TOSICA.pre for prediction instead of training
- You need a general-purpose cell clustering or deconvolution tool rather than supervised
  label transfer from a reference dataset
- The query and reference var_names (genes) differ or are not in the same order and
  cannot be reconciled — predictions will be invalid
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: TOSICA
  role: Python package providing TOSICA.train to fit the masked multi-head self-attention
    model and TOSICA.pre to predict labels on query data
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Loads and preprocesses the reference/query AnnData single-cell objects
- name: torch
  role: Deep learning backend (torch 1.7.1) for the Transformer attention layers,
    SGD optimizer, and cross-entropy loss
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:fc8b46cd6e705f231a239a6982769a07c4bdcf99a9d68fa9d9dc936ad4540cea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# transformer-model-training-single-cell-rnaseq

## Summary

Trains TOSICA, a Transformer-based multi-head self-attention model for interpretable cell type annotation on scRNA-seq reference data, using gene-set-masked attention tokens, and supports controlled ablations such as knowledge-based versus random masks.

## When to use

Use when a labeled single-cell reference AnnData must be turned into a cell type annotation model (e.g., training TOSICA with GMT masks on mouse/human data), or when comparing convergence of knowledge-based masks vs random masks as in Supplementary Fig. 8.

## When NOT to use

- The input is an unlabeled query dataset with no reference training labels — use TOSICA.pre for prediction instead of training
- You need a general-purpose cell clustering or deconvolution tool rather than supervised label transfer from a reference dataset
- The query and reference var_names (genes) differ or are not in the same order and cannot be reconciled — predictions will be invalid

## Inputs

- Reference AnnData object (ref_adata) with a cell type label column in ref_adata.obs
- Gene set GMT files mapping genes to pathways/regulons (e.g., m_GO_bp.gmt, m_reactome.gmt, m_TF.gmt) or built-in mask keywords (mouse_gobp, mouse_reactome, mouse_tf, human_gobp, human_reactome, human_tf, human_immune)
- Random mask matrices reserving 1% and 5% of gene-to-token connections, parameter-matched to the knowledge-based mask

## Outputs

- ./my_project/mask.npy : Mask matrix
- ./my_project/pathway.csv : Gene set list
- ./my_project/label_dictionary.csv : Label list
- ./my_project/model-n.pth : Trained model weights
- Per-epoch validation accuracy traces per mask condition
- convergence_comparison.csv summarizing epochs-to-convergence and converged accuracy for knowledge-based vs random masks

## How to apply

Load the reference AnnData with scanpy and confirm the label column in ref_adata.obs; ensure query var_names match reference gene order via query_adata = query_adata[:,ref_adata.var_names]. Call TOSICA.train(ref_adata, gmt_path, project=<my_project>, label_name=<label_key>) with mouse masks (m_GO_bp.gmt, m_reactome.gmt, m_TF.gmt) or human masks (GO_bp.gmt, reactome.gmt, TF.gmt, immune.gmt); training uses SGD with cosine learning rate decay and cross-entropy loss, holding out 30% of the training set as validation. For the mask ablation, construct two random mask matrices reserving 1% and 5% of gene-to-token connections, matched in parameter count to the knowledge-based mask, and train a model for each. Record validation accuracy per epoch for each condition to determine epochs-to-convergence and converged accuracy (on mPancreas, the random mask converges lower than the knowledge-based mask). Summarize the three conditions and save as convergence_comparison.csv; artifacts written per project folder are mask.npy, pathway.csv, label_dictionary.csv, and model-n.pth.

## Related tools

- **TOSICA** (Python package providing TOSICA.train to fit the masked multi-head self-attention model and TOSICA.pre to predict labels on query data) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Loads and preprocesses the reference/query AnnData single-cell objects)
- **torch** (Deep learning backend (torch 1.7.1) for the Transformer attention layers, SGD optimizer, and cross-entropy loss)

## Examples

```
TOSICA.train(ref_adata, 'm_GO_bp.gmt', project='mouse_gobp_mPancreas', label_name='celltype')
```

## Evaluation signals

- Trained model weights (model-n.pth), mask.npy, pathway.csv, and label_dictionary.csv exist for all three mask conditions (knowledge-based, 1% random, 5% random)
- convergence_comparison.csv shows random-mask models approaching knowledge-mask accuracy on mPancreas but requiring more epochs and/or reaching lower converged accuracy, consistent with Supplementary Fig. 8
- Query and reference var_names are verified identical and in the same order before prediction
- 30% of the training set is held out as validation and per-epoch validation accuracy is recorded for convergence assessment
- TOSICA.pre on a query AnnData returns new_adata.obs['Prediction'], 'Probability', and the attention matrix with pathway_index columns

## Limitations

- Accuracy depends on mask choice in some datasets: random masks usually match knowledge-based accuracy but converge lower on mPancreas
- The learned attention depends on tokens masked by biologically relevant pathways or regulons, so arbitrary masks can reduce interpretability
- ref_adata and query_adata var_names must be consistent and in the same order, otherwise results are invalid
- Python >= 3.8 with torch == 1.7.1 is required; the paper does not report the number of attention heads, learning rate, batch size, or random seed

## Evidence

- [abstract] TOSICA achieves fast and accurate one-stop annotation and batch-insensitive integration: "TOSICA achieves fast and accurate one-stop annotation and batch-insensitive integration while providing biologically interpretable insights"
- [discussion] Training details: "Stochastic gradient descent (SGD) is chosen as optimizer, and we use cosine learning rate decay to avoid too large steps in late stage of training"
- [discussion] Validation split: [evidence span withheld — anchor status: not_found]
- [discussion] Mask construction: "we map the input genes to selected gene sets (gmt files)"
- [readme] TOSICA.train usage: "TOSICA.train(ref_adata, gmt_path,project=<my_project>,label_name=<label_key>)"
- [readme] Gene order requirement: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [discussion] Attribution to masks: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
