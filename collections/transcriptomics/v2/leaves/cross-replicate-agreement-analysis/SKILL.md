---
name: cross-replicate-agreement-analysis
description: Use when two or more replicate sections from the same platform (e.g.,
  Xenium-BreastCancer1 and Xenium-BreastCancer2) have been segmented with BIDCell
  and you need to quantify whether cell segmentation and cell-type annotation results
  are concordant across replicates.
when_to_use_negative:
- You are comparing segmentation methods on a single section (use the full CellSPA
  method-comparison framework instead of cross-replicate agreement)
- Only one replicate section exists, so no cross-replicate concordance can be computed
- Input data is not subcellular spatial transcriptomics (e.g., bulk or single-cell-only
  data), since segmentation replicability is not defined
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BIDCell
  role: Self-supervised segmentation pipeline run per replicate to produce cell-ID
    masks and cell-gene expression matrices for agreement analysis
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source platform providing the replicate in situ output bundles (Xenium-BreastCancer1
    and replicate 2) being compared
- name: CellSPA
  role: Evaluation framework whose metric categories (cell characteristics, cell type
    proportions, embedding mixing, morphology) structure the cross-replicate agreement
    analysis
  repo: https://github.com/SydneyBioX/BIDCell
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:fbed4ff02bee89257d75c27649a9a559649bf69d8819a857da681cfaa62d6982
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - CellSPA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cross-replicate-agreement-analysis

## Summary

Assesses whether BIDCell's subcellular segmentation of spatial transcriptomics data is replicable across replicate tissue sections by running the full BIDCell pipeline on each replicate and comparing agreement metrics following the CellSPA evaluation framework (e.g., Fig. 3i-j and Supplementary Fig. 16 of the BIDCell paper).

## When to use

Use when two or more replicate sections from the same platform (e.g., Xenium-BreastCancer1 and Xenium-BreastCancer2) have been segmented with BIDCell and you need to quantify whether cell segmentation and cell-type annotation results are concordant across replicates.

## When NOT to use

- You are comparing segmentation methods on a single section (use the full CellSPA method-comparison framework instead of cross-replicate agreement)
- Only one replicate section exists, so no cross-replicate concordance can be computed
- Input data is not subcellular spatial transcriptomics (e.g., bulk or single-cell-only data), since segmentation replicability is not defined

## Inputs

- Xenium Output Bundle for each replicate (transcripts.csv.gz, morphology_mip.ome.tif)
- BIDCell config YAML per replicate (e.g., xenium_example_config.yaml)
- Breast cancer single-cell reference: sc_breast.csv
- Positive/negative marker files: sc_breast_markers_pos.csv, sc_breast_markers_neg.csv

## Outputs

- *_connected.tif cell-ID segmentation mask per replicate
- expr_mat.csv cell-gene expression matrix per replicate
- Cross-replicate agreement metrics: baseline characteristic correlations, cell type proportion comparisons, UMAP/tSNE replicate-label mixing, morphology metric correlations

## How to apply

Run the BIDCell pipeline (preprocess, train, predict) independently on each replicate section — for the second Xenium breast cancer replicate, download the Xenium Output Bundle from the 10x Genomics preview-dataset page and set paths for transcripts.csv.gz and morphology_mip.ome.tif in xenium_example_config.yaml, tuning the cpus parameter for your machine. Train the default UNet3+ model for ~4,000 steps (about 10 min on a >=12GB GPU) per replicate and extract the cell-gene expression matrix expr_mat.csv from the *_connected.tif cell-ID segmentation mask. Annotate cell types in both replicates using the breast cancer single-cell reference and marker files (sc_breast.csv, sc_breast_markers_pos.csv, sc_breast_markers_neg.csv). Then compute cross-replicate agreement following the CellSPA categories: correlations of baseline cell characteristics, comparison of cell type proportions, UMAP/tSNE embedding mixing of replicate labels, and correlations of morphology metrics. Replicability is demonstrated when replicate labels are well-mixed in the embedding and proportion/morphology correlations reproduce the concordance patterns reported in Fig. 3i-j and Supplementary Fig. 16.

## Related tools

- **BIDCell** (Self-supervised segmentation pipeline run per replicate to produce cell-ID masks and cell-gene expression matrices for agreement analysis) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source platform providing the replicate in situ output bundles (Xenium-BreastCancer1 and replicate 2) being compared)
- **CellSPA** (Evaluation framework whose metric categories (cell characteristics, cell type proportions, embedding mixing, morphology) structure the cross-replicate agreement analysis) — https://github.com/SydneyBioX/BIDCell

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel("xenium_example_config.yaml"); model.preprocess(); model.train(); model.predict()
```

## Evaluation signals

- expr_mat.csv and the *_connected.tif mask exist for each replicate
- tSNE/UMAP embedding shows a well-mixed population of cells between replicates (replicate labels not separated)
- Cell type proportions and baseline cell characteristic correlations between replicates match the concordance patterns in Fig. 3i-j and Supplementary Fig. 16
- Morphology metric correlations between replicates are consistent with the reported replicate-concordance results

## Limitations

- Training requires a GPU with at least 12GB VRAM and ~32GB RAM for data processing; higher cpus settings speed up runs but may freeze the system
- Manual annotation of SST data is unattainable, so replicability relies on indirect agreement metrics rather than ground-truth boundaries
- 2D segmentation may achieve limited representation of cells' 3D structure, and elongated morphologies (melanocytes, fibroblasts) without nearby nuclei are challenging to segment consistently

## Evidence

- [intro] CellSPA evaluation framework motivation: "we created a comprehensive evaluation framework for cell segmentation, CellSPA, that assesses five complementary categories of criteria for identifying the optimal segmentation strategies"
- [readme] Xenium replicate data source: "The full dataset (Xenium Output Bundle In Situ Replicate 1) may be downloaded from https://www.10xgenomics.com/products/xenium-in-situ/preview-dataset-human-breast"
- [methods] Single-cell reference files used for annotation: "Please see the provided breast cancer single-cell reference and positive/negative marker files (`sc_breast.csv`, `sc_breast_markers_pos.csv`, and `sc_breast_markers_neg.csv`)"
- [readme] CellSPA metric categories described in README: "We also introduce CellSPA, a comprehensive evaluation framework consisting of metrics in five complementary categories for cell segmentation performance."
- [readme] Expected segmentation output format: ".tif file of segmented cells, where the value corresponds to cell IDs. File name ends in `_connected.tif`"
