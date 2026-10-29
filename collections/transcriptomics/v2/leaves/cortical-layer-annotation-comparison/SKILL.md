---
name: cortical-layer-annotation-comparison
description: Use when evaluating spatial transcriptomics methods on the 12 human DLPFC
  Visium slices with manually annotated cortical layers (1-6, WM) as ground truth,
  e.g. comparing SPACEL modules against PRECAST, STAligner, or other deconvolution/domain-identification
  baselines.
when_to_use_negative:
- The dataset is a single-slice ST dataset without manually annotated cortical-layer
  ground truth, so no layer-recovery benchmarking is possible
- Seq-based ST data where no cell type deconvolution (e.g. Spoint) has been run first,
  since Splane requires cell type composition as input for 10X Visium data
- The analysis goal is 3D alignment/stacking of consecutive slices (use Scube-based
  skills) rather than annotation-accuracy benchmarking
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: SPACEL
  role: Toolkit providing the Spoint and Splane modules whose annotations are benchmarked
    against DLPFC cortical-layer ground truth
  repo: https://github.com/QuKunLab/SPACEL
- name: Splane
  role: Identifies spatial domains across the 12 slices via GCN with adversarial learning;
    domain DEGs are tested for SVG recovery
  repo: https://github.com/QuKunLab/SPACEL
- name: Spoint
  role: Deconvolves cell type composition per spot from the Allen Brain Map scRNA-seq
    reference for accuracy-score evaluation
  repo: https://github.com/QuKunLab/SPACEL
- name: Python
  role: Implementation environment for anndata loading, scanpy rank_genes_groups Wilcoxon
    testing, PyTorch training, and metric computation
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:3f4c493fa662b1de139e021b63b55671602d478b7d5ab5127f25f2d7a994f7f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QuKunLab/SPACEL
  tools:
  - PyTorch
  - Python
  - SPACEL
  - Splane
  - Spoint
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cortical-layer-annotation-comparison

## Summary

Benchmarks how accurately method-derived annotations (Splane spatial domains or Spoint deconvolution compositions) recover manually annotated cortical layers 1-6 and white matter across the 12 human DLPFC 10X Visium slices, using SVG recovery ROC AUC and per-cell-type accuracy scores.

## When to use

Use when evaluating spatial transcriptomics methods on the 12 human DLPFC Visium slices with manually annotated cortical layers (1-6, WM) as ground truth, e.g. comparing SPACEL modules against PRECAST, STAligner, or other deconvolution/domain-identification baselines.

## When NOT to use

- The dataset is a single-slice ST dataset without manually annotated cortical-layer ground truth, so no layer-recovery benchmarking is possible
- Seq-based ST data where no cell type deconvolution (e.g. Spoint) has been run first, since Splane requires cell type composition as input for 10X Visium data
- The analysis goal is 3D alignment/stacking of consecutive slices (use Scube-based skills) rather than annotation-accuracy benchmarking

## Inputs

- 12 human DLPFC 10X Visium slice datasets (spot expression matrices with manual cortical-layer 1-6/WM annotations)
- Layer-annotated ground-truth SVG list for DLPFC cortical layers and white matter
- Allen Brain Map human brain scRNA-seq reference dataset (as anndata object for Spoint)
- Splane-identified spatial domain labels per spot across the 12 slices

## Outputs

- svg_recovery_metrics.csv containing DEG membership table and SVG recovery/ROC AUC summary per method
- spoint_dlpfc_deconvolution_results.csv with accuracy scores per cell type (56 cell types x 12 slices), excitatory L3/4 accuracy, and average Wilcoxon rank-sum P-values
- ROC AUC comparison table (Splane 0.90 vs PRECAST 0.89, STAligner 0.88, others < 0.83)

## How to apply

Load the 12 DLPFC Visium slices and, for deconvolution benchmarks, the Allen Brain Map human brain scRNA-seq reference into anndata objects. For domain-based evaluation, run Splane's GCN with adversarial learning on cell-type composition plus spatial coordinates, then use scanpy's rank_genes_groups Wilcoxon rank-sum test per domain, filtering DEGs at logFC > 0.5 and P < 0.01 to obtain recovered SVG candidates; intersect these with the ground-truth layer-annotated SVG list (expected recovery 1714 of 1917) and compute ROC AUC alongside PRECAST (0.89) and STAligner (0.88) baselines. For deconvolution evaluation, train SpointModel on the single-cell reference via Spoint.base_model.SpointModel.train (PyTorch, GPU if available), run deconv_spatial per slice, and compare predicted compositions against manual cortical-layer labels using accuracy scores per cell type and average Wilcoxon rank-sum P-values. A correct run reproduces Splane's SVG-recovery ROC AUC of 0.90 and Spoint's excitatory L3/4 accuracy score of 0.85 with average AS = 0.60 over 56 cell types.

## Related tools

- **SPACEL** (Toolkit providing the Spoint and Splane modules whose annotations are benchmarked against DLPFC cortical-layer ground truth) — https://github.com/QuKunLab/SPACEL
- **Splane** (Identifies spatial domains across the 12 slices via GCN with adversarial learning; domain DEGs are tested for SVG recovery) — https://github.com/QuKunLab/SPACEL
- **Spoint** (Deconvolves cell type composition per spot from the Allen Brain Map scRNA-seq reference for accuracy-score evaluation) — https://github.com/QuKunLab/SPACEL
- **Python** (Implementation environment for anndata loading, scanpy rank_genes_groups Wilcoxon testing, PyTorch training, and metric computation)

## Examples

```
sp_model = Spoint.base_model.SpointModel(init_model='MLP'); sp_model.train(ad_sc_ref); sp_model.deconv_spatial(ad_st_dlpfc)
```

## Evaluation signals

- Splane SVG-recovery ROC AUC reaches 0.90, exceeding PRECAST (0.89), STAligner (0.88), and all other methods (AUC < 0.83)
- SVG recovery count from domain DEGs intersected with the ground-truth list is 1714 of 1917
- Spoint excitatory L3/4 neuron accuracy score reaches 0.85 versus 0.29-0.64 for competing deconvolution methods
- Average accuracy score across all 56 cell types and 12 slices is 0.60 (vs 0.30-0.48 for other methods) and average Wilcoxon rank-sum P-value is 0.01
- Output CSVs (svg_recovery_metrics.csv, spoint_dlpfc_deconvolution_results.csv) contain the expected columns and the benchmark-matching values

## Limitations

- Splane depends on cell type composition input, so a deconvolution step (e.g., Spoint) must precede it for seq-based ST datasets like 10X Visium
- Adding a new slice requires retraining the entire deep learning model from scratch, limiting computational efficiency for large-scale datasets
- Accuracy scores depend on the quality and granularity of the manually annotated cortical layers and the external scRNA-seq reference (Allen Brain Map)

## Evidence

- [other] Splane achieved the highest SVG-identification accuracy (ROC AUC = 0.90), outperforming PRECAST (0.89), STAligner (0.88): [evidence span withheld — anchor status: not_found]
- [other] Spoint achieved the highest accuracy score (AS = 0.85) for excitatory L3/4 neurons: [evidence span withheld — anchor status: not_found]
- [abstract] Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains"
- [discussion] one has to employ a cell type deconvolution algorithm such as Spoint: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [methods] Deconvolution of cell types compostion on human brain Visium dataset: "Deconvolution of cell types compostion on human brain Visium dataset <tutorials/Visium_human_DLPFC_Spoint>"
