---
name: single-cell-reference-cell-type-annotation
description: Use when segmenting subcellular spatial transcriptomics data (e.g. Xenium
  breast cancer) with BIDCell and a single-cell reference plus marker files are needed
  to pre-annotate cells for biologically-informed training or cross-replicate cell
  type proportion comparison.
when_to_use_negative:
- The spatial dataset is from a tissue with no matching single-cell reference — the
  reference csv must cover the tissue/cell types of the spatial data (e.g. do not
  apply the breast cancer reference to brain tissue)
- Only nuclear segmentation is needed without cell type information — the marker-driven
  loss terms are optional inputs, not required to produce the *_connected.tif mask
  alone
- Reference genes do not overlap the spatial panel — the reference csv must contain
  average expressions for genes present in the spatial transcriptomic dataset (fewer
  than 1,000 genes suffice for segmentation)
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: BIDCell
  role: Runs the preannotate step that consumes the single-cell reference and marker
    files to label pixels/cells for biologically-informed loss functions during segmentation
    training
  repo: https://github.com/SydneyBioX/BIDCell
- name: scClassify
  role: Source of example single-cell reference data / pretrained models that may
    be used to build cell type references for annotation
  repo: https://github.com/SydneyBioX/scClassify
- name: 10x Genomics Xenium
  role: Platform generating the spatial transcriptomics data (Xenium breast cancer
    replicates) whose segmented cells are annotated using the breast cancer reference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:d680bfaafb622e8ff4879d3950bffbc2aa6789f1976204cb0664187642662f61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - scClassify
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-reference-cell-type-annotation

## Summary

Annotates predicted cells from BIDCell spatial transcriptomics segmentation with cell types using a single-cell sequencing reference (average expression per cell type) plus positive/negative marker gene lists, which feed BIDCell's biologically-informed loss functions and the preannotate step.

## When to use

Use when segmenting subcellular spatial transcriptomics data (e.g. Xenium breast cancer) with BIDCell and a single-cell reference plus marker files are needed to pre-annotate cells for biologically-informed training or cross-replicate cell type proportion comparison.

## When NOT to use

- The spatial dataset is from a tissue with no matching single-cell reference — the reference csv must cover the tissue/cell types of the spatial data (e.g. do not apply the breast cancer reference to brain tissue)
- Only nuclear segmentation is needed without cell type information — the marker-driven loss terms are optional inputs, not required to produce the *_connected.tif mask alone
- Reference genes do not overlap the spatial panel — the reference csv must contain average expressions for genes present in the spatial transcriptomic dataset (fewer than 1,000 genes suffice for segmentation)

## Inputs

- sc_breast.csv — single-cell reference with average expression per gene for each breast cancer cell type
- sc_breast_markers_pos.csv — positive marker genes (top 10 percentile expression per cell type)
- sc_breast_markers_neg.csv — negative marker genes (bottom 10 percentile expression per cell type)
- BIDCell yaml config (e.g. xenium_example_config.yaml) paths under files
- Cell-gene expression matrix expr_mat.csv from segmented cells (for downstream cell type proportion comparison)

## Outputs

- Pre-annotated cells/pixels with cell type labels feeding BIDCell training (via model.preannotate())
- Cell type proportion estimates per replicate for cross-replicate agreement metrics (Fig. 3i-j, Supplementary Fig. 16)

## How to apply

Prepare three files: a reference CSV (e.g. sc_breast.csv) containing average expressions for all genes in the spatial dataset per cell type, and positive/negative marker CSVs (sc_breast_markers_pos.csv, sc_breast_markers_neg.csv) built from genes in the highest and lowest 10 percentile expression for each cell type. Remove positive markers common to at least a third of cell types in the dataset; note that using a larger number of positive markers tends to increase predicted cell size. References can be sourced from public repositories such as TISCH2, Allen Brain Map, the Human Cell Atlas, or SydneyBioX/scClassify pretrained models. Supply these paths in the .yaml config (files section) and run model.preannotate() as the final preprocessing step before model.train(); for the Xenium breast cancer replicability task, annotate both Xenium-BreastCancer1 and replicate 2 cells with the same breast cancer reference to compare cell type proportions.

## Related tools

- **BIDCell** (Runs the preannotate step that consumes the single-cell reference and marker files to label pixels/cells for biologically-informed loss functions during segmentation training) — https://github.com/SydneyBioX/BIDCell
- **scClassify** (Source of example single-cell reference data / pretrained models that may be used to build cell type references for annotation) — https://github.com/SydneyBioX/scClassify
- **10x Genomics Xenium** (Platform generating the spatial transcriptomics data (Xenium breast cancer replicates) whose segmented cells are annotated using the breast cancer reference)

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel('xenium_example_config.yaml'); model.segment_nuclei(); model.generate_expression_maps(); model.generate_patches(); model.make_cell_gene_mat(is_cell=False); model.preannotate()
```

## Evaluation signals

- The reference csv contains average expressions for all genes in the spatial transcriptomic dataset across the chosen cell types
- Positive/negative marker files derive from the highest/lowest 10 percentile expression per cell type, and positive markers common to ≥1/3 of cell types have been removed
- expr_mat.csv for replicate 2 and the *_connected.tif mask exist before annotation-dependent analyses
- Cell type proportion comparisons between Xenium-BreastCancer1 and replicate 2 reproduce the replicate-concordance patterns in Fig. 3i-j and Supplementary Fig. 16
- Altering the number of positive markers changes predicted cell size (larger marker sets → larger cells), so marker count is a tunable decision point

## Limitations

- Using a larger number of positive markers tends to increase the size of predicted cells; manual curation or alternative marker selection approaches may be required
- Removing positive markers common to at least a third of cell types was found appropriate across various datasets but is a heuristic, not a guarantee
- Only <1,000 genes are needed for segmentation, but the reference must still be matched to the tissue and the spatial gene panel; heterogeneous tissues may need a custom reference

## Evidence

- [readme] The reference csv file contains average expressions for all of the genes in the spatial transcriptomic dataset of different cell types.: "The reference csv file contains average expressions for all of the genes in the spatial transcriptomic dataset of different cell types."
- [readme] Positive and negative markers from top/bottom 10 percentile: "The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample."
- [methods] Breast cancer reference files named in methods: "Please see the provided breast cancer single-cell reference and positive/negative marker files (`sc_breast.csv`, `sc_breast_markers_pos.csv`, and `sc_breast_markers_neg.csv`)"
- [readme] Positive markers removed if common across cell types: "We found that removing positive markers that were common to at least a third of cell types in each dataset was appropriate across various datasets."
