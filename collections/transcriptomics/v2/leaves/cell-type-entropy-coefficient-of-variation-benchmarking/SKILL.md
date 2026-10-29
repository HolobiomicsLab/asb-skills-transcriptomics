---
name: cell-type-entropy-coefficient-of-variation-benchmarking
description: Use when comparing segmentation methods on spatial transcriptomics data
  (e.g., MERSCOPE melanoma, CosMx lung) where no pixel-level ground truth exists and
  you need reference-based homogeneity vs cell-size consistency metrics.
when_to_use_negative:
- Datasets with pixel-level ground truth annotations exist — use direct segmentation
  accuracy metrics instead of entropy/CV surrogates
- Input is already a cell-by-cell-type annotated matrix with no segmentation step
  to compare (nothing to benchmark)
- MERSCOPE tissues dominated by elongated melanocytes/fibroblasts where ~25% of cells
  lack nuclei — interpret entropy metrics cautiously given the documented morphology
  caveat
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0625
tools:
- name: BIDCell
  role: Segmentation method under benchmark; provides expr_mat.csv via make_cell_gene_mat()
    for entropy computation
  repo: https://github.com/SydneyBioX/BIDCell
- name: scClassify
  role: Hierarchical single-cell classification source for reference data used to
    assign cell types to segmented cells
  repo: https://github.com/SydneyBioX/scClassify
- name: Vizgen MERSCOPE
  role: Platform generating the melanoma patient-2 dataset on which entropy-vs-CV
    benchmarking is performed
- name: NanoString CosMx
  role: Platform generating the NSCLC Lung5-1 dataset for the complementary B-cell
    contamination benchmark
- name: TISCH2
  role: Public repository for downloading single-cell reference and marker files (TISCH-SKCM,
    TISCH-NSCLC)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:975b7709ef615c6c56a0bf9246ed8ca36a627b821bc816a0a057ad6351610b1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - BIDCell
  - Human Cell Atlas
  - NanoString CosMx
  - TISCH2
  - Vizgen MERSCOPE
  - scClassify
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Cell-type entropy versus coefficient of variation benchmarking

## Summary

Benchmark SST segmentation methods on non-Xenium platforms by computing cell-type entropy (homogeneity of per-cell transcriptional composition from single-cell reference annotation) against the coefficient of variation of cell size, plotting the results as the Fig. 4 generalisability panels (e.g., MERSCOPE-Melanoma).

## When to use

Use when comparing segmentation methods on spatial transcriptomics data (e.g., MERSCOPE melanoma, CosMx lung) where no pixel-level ground truth exists and you need reference-based homogeneity vs cell-size consistency metrics.

## When NOT to use

- Datasets with pixel-level ground truth annotations exist — use direct segmentation accuracy metrics instead of entropy/CV surrogates
- Input is already a cell-by-cell-type annotated matrix with no segmentation step to compare (nothing to benchmark)
- MERSCOPE tissues dominated by elongated melanocytes/fibroblasts where ~25% of cells lack nuclei — interpret entropy metrics cautiously given the documented morphology caveat

## Inputs

- expr_mat.csv cell-gene expression matrix from model.make_cell_gene_mat() per segmentation method and platform
- *_connected.tif segmentation mask with cell IDs (BIDCell output)
- Single-cell reference expression data (e.g., TISCH-SKCM for melanoma, HLCA/TISCH-NSCLC for lung from TISCH2 / Human Cell Atlas)
- Positive/negative marker files (e.g., sc_breast_markers_pos.csv / sc_breast_markers_neg.csv as template)

## Outputs

- Per-cell cell-type entropy scores per segmentation method
- Coefficient of variation (cell size) per segmentation method
- Entropy-versus-CV benchmark panels reproducing Fig. 4a-f
- B-cell contamination statistics (B cells expressing negative/T-cell markers) for the CosMx-Lung arm

## How to apply

Run each segmentation method (BIDCell, plus comparators) on the same platform dataset and extract the per-cell gene expression matrix with model.make_cell_gene_mat() to obtain expr_mat.csv. Annotate each segmented cell to cell types using a single-cell reference (e.g., TISCH-SKCM for melanoma, HLCA/TISCH-NSCLC for lung) via scClassify-style hierarchical classification, then compute cell-type entropy per cell — low entropy indicates a transcriptionally homogeneous cell body consistent with a single reference cell type. In parallel, compute the coefficient of variation of cell size; plot entropy versus CV so methods that achieve homogeneity simply by arbitrary over-segmentation of nuclei are exposed, since 'achieving more homogenous cell bodies ... can also result from the arbitrary over-segmentation of nuclei, motivating use of multiple metrics'. Validate that each run produced a *_connected.tif segmentation and an expr_mat.csv, and that BIDCell shows competitive entropy-versus-CV profiles while also maintaining low B-cell negative-marker contamination on CosMx-Lung.

## Related tools

- **BIDCell** (Segmentation method under benchmark; provides expr_mat.csv via make_cell_gene_mat() for entropy computation) — https://github.com/SydneyBioX/BIDCell
- **scClassify** (Hierarchical single-cell classification source for reference data used to assign cell types to segmented cells) — https://github.com/SydneyBioX/scClassify
- **Vizgen MERSCOPE** (Platform generating the melanoma patient-2 dataset on which entropy-vs-CV benchmarking is performed)
- **NanoString CosMx** (Platform generating the NSCLC Lung5-1 dataset for the complementary B-cell contamination benchmark)
- **TISCH2** (Public repository for downloading single-cell reference and marker files (TISCH-SKCM, TISCH-NSCLC))

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel('merscope_example_config.yaml'); model.run_pipeline(); model.make_cell_gene_mat(is_cell=False)
```

## Evaluation signals

- Each segmentation run yields a *_connected.tif file whose pixel values correspond to cell IDs, plus an expr_mat.csv cell-gene matrix
- Entropy-versus-CV panels reproduce the reported Fig. 4a-f behaviour: BIDCell maintains competitive (low) entropy and CV relative to comparator methods
- On CosMx-Lung, BIDCell shows a lower percentage of B cells expressing negative (contamination-indicating) markers than comparator methods
- Methods that only lower entropy by over-segmenting nuclei are flagged via their CV, per the multi-metric rationale

## Limitations

- Achieving more homogeneous cell bodies can result from arbitrary over-segmentation of nuclei, so entropy alone is insufficient and must be paired with CV
- In MERSCOPE data with cell membrane markers, ~25% of cells lack nuclei in their segmentation (elongated melanocytes/fibroblasts), confounding entropy estimates
- Benchmarking is time-consuming due to software versioning, operating system differences, and varying ease of adapting comparator code

## Evidence

- [discussion] multi-metric rationale: "While achieving more homogenous cell bodies is desired, it can also result from the arbitrary over-segmentation of nuclei."
- [readme] pipeline output contract: ".tif file of segmented cells, where the value corresponds to cell IDs. File name ends in `_connected.tif`"
- [methods] reference data sourcing: "BIDCell uses single-cell reference data for improved results. These can be downloaded from public repositories such as TISCH2, Allen Brain Map, and the Human Cell Atlas."
- [readme] cell-gene matrix extraction: "model.make_cell_gene_mat(is_cell=False)"
- [discussion] MERSCOPE morphology caveat: "In MERSCOPE data that display cell membrane markers, there is a percentage (25%) of cells that lack nuclei in their segmentation"
