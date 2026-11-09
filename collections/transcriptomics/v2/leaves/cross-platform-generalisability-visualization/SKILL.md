---
name: cross-platform-generalisability-visualization
description: Use when testing whether a segmentation method trained on Xenium transfers
  to other SST platforms (CosMx, MERSCOPE) or tissue types, and when plotting cross-platform
  benchmark panels.
when_to_use_negative:
- Data is from a single platform only (e.g. Xenium breast cancer demo) and no cross-platform
  transfer question is being asked
- The goal is raw segmentation output rather than comparative benchmarking/visualisation
  against other methods
- Single-cell reference data for the target tissue is unavailable, since BIDCell relies
  on sc references and positive/negative marker files for its biologically-informed
  losses
edam_operation: http://edamontology.org/operation_0558
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: BIDCell
  role: Segmentation framework whose run_pipeline (stitch_nuclei, segment_nuclei,
    generate_expression_maps, generate_patches, train, predict, make_cell_gene_mat)
    produces the cell-gene matrices benchmarked across platforms
  repo: https://github.com/SydneyBioX/BIDCell
- name: NanoString CosMx
  role: Non-Xenium SST platform providing the Lung5-1 lung cancer dataset used to
    test generalisability
- name: Vizgen MERSCOPE
  role: Non-Xenium SST platform providing the melanoma patient-2 dataset used to test
    generalisability
- name: TISCH2
  role: Public repository from which single-cell references and marker files (TISCH-NSCLC,
    TISCH-SKCM) are downloaded
- name: Human Cell Atlas
  role: Alternative public repository for single-cell reference data used in the biologically-informed
    loss functions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:831993e2caaab46dafb8a3a602651a085fe46e1a838269c015f3f16722df263f
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
  edam_operation: http://edamontology.org/operation_0558
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# cross-platform-generalisability-visualization

## Summary

Reproduces the BIDCell cross-platform generalisability benchmark (Fig. 4a-f) by running the BIDCell pipeline on NanoString CosMx lung (Lung5-1) and Vizgen MERSCOPE melanoma (patient-2) data and visualising contamination and cell-type homogeneity metrics against comparator segmentation methods.

## When to use

Use when testing whether a segmentation method trained on Xenium transfers to other SST platforms (CosMx, MERSCOPE) or tissue types, and when plotting cross-platform benchmark panels.

## When NOT to use

- Data is from a single platform only (e.g. Xenium breast cancer demo) and no cross-platform transfer question is being asked
- The goal is raw segmentation output rather than comparative benchmarking/visualisation against other methods
- Single-cell reference data for the target tissue is unavailable, since BIDCell relies on sc references and positive/negative marker files for its biologically-informed losses

## Inputs

- CosMx NSCLC Lung5-1 transcript and FOV-separated nuclei image files
- MERSCOPE melanoma patient-2 transcripts and nuclei images
- Single-cell reference csv (e.g. sc_breast.csv-style average expression per cell type)
- Positive/negative marker csv files per cell type (e.g. sc_breast_markers_pos.csv, sc_breast_markers_neg.csv)
- Vendor-specific .yaml config files (cosmx_example_config.yaml, merscope_example_config.yaml)

## Outputs

- *_connected.tif segmentation masks with cell IDs per platform
- expr_mat.csv per-cell gene expression matrix for each platform
- Fig. 4a-f style benchmark panels: B-cell contamination and entropy-versus-CV profiles comparing BIDCell to other segmentation methods

## How to apply

Download the public CosMx NSCLC Lung5-1 and MERSCOPE melanoma patient-2 datasets plus single-cell references and positive/negative marker files (HLCA/TISCH-NSCLC for lung, TISCH-SKCM for melanoma) from TISCH2 or the Human Cell Atlas. Generate vendor-specific configs with BIDCellModel.get_example_config('cosmx') and get_example_config('merscope'), edit file paths (transcripts, DAPI/nuclei images) and the cpus parameter. For CosMx FOV-separated nuclei images run model.stitch_nuclei() before segment_nuclei(), then generate_expression_maps(), generate_patches(), train (~4,000 steps, UNet3+ default) and predict on a GPU with at least 12GB VRAM, and extract expression with make_cell_gene_mat(). Assess generalisability by quantifying B-cell contamination by negative markers on CosMx-Lung and cell-type entropy versus coefficient of variation on MERSCOPE-Melanoma, plotting the results as the Fig. 4a-f benchmark panels.

## Related tools

- **BIDCell** (Segmentation framework whose run_pipeline (stitch_nuclei, segment_nuclei, generate_expression_maps, generate_patches, train, predict, make_cell_gene_mat) produces the cell-gene matrices benchmarked across platforms) — https://github.com/SydneyBioX/BIDCell
- **NanoString CosMx** (Non-Xenium SST platform providing the Lung5-1 lung cancer dataset used to test generalisability)
- **Vizgen MERSCOPE** (Non-Xenium SST platform providing the melanoma patient-2 dataset used to test generalisability)
- **TISCH2** (Public repository from which single-cell references and marker files (TISCH-NSCLC, TISCH-SKCM) are downloaded)
- **Human Cell Atlas** (Alternative public repository for single-cell reference data used in the biologically-informed loss functions)

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel('cosmx_example_config.yaml'); model.stitch_nuclei(); model.segment_nuclei(); model.generate_expression_maps(); model.generate_patches(); model.train(); model.predict(); model.make_cell_gene_mat(is_cell=False)
```

## Evaluation signals

- Each run yields a *_connected.tif segmentation file whose values correspond to cell IDs
- Each run yields an expr_mat.csv cell-gene matrix per platform
- On CosMx-Lung, BIDCell shows a lower percentage of B cells expressing negative (contamination-indicating) markers than comparator methods
- On MERSCOPE-Melanoma, BIDCell's cell-type entropy-versus-coefficient-of-variation profile is competitive with other methods in the reproduced Fig. 4 panels

## Limitations

- Requires a GPU with at least 12GB VRAM for training/prediction and 32GB RAM for data processing; high cpus settings may freeze the system
- In MERSCOPE data with cell membrane markers, ~25% of cells lack nuclei in their segmentation (elongated melanocytes or fibroblasts without a nucleus in the section), complicating whole-cell boundary assessment
- Benchmarking is time-consuming due to software versioning, operating system differences, and differing ease of adapting comparator code

## Evidence

- [results] BIDCell generalises to non-Xenium platforms; on CosMx-Lung it had a lower percentage of B cells expressing negative (contamination-indicating) markers: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline stages for FOV-separated nuclei and cell-gene extraction: "# model.stitch_nuclei() # for when nuclei images are separated into FOVs (e.g., CosMx)"
- [readme] Hardware requirements for the deep learning stages: "A GPU with at least 12GB VRAM is strongly recommended for the deep learning component, and 32GB RAM for data processing."
- [abstract] Cross-platform performance claim motivating the benchmark: "we demonstrate that BIDCell outperforms other state-of-the-art methods according to many metrics across a variety of tissue types and technology platforms"
- [methods] Single-cell reference sources for the biologically-informed losses: "These can be downloaded from public repositories such as TISCH2, Allen Brain Map, and the Human Cell Atlas."
