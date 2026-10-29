---
name: bcell-tcell-marker-contamination-assessment
description: Use when validating a segmentation method's biological accuracy on non-Xenium
  SST platforms (NanoString CosMx lung, Vizgen MERSCOPE melanoma) by quantifying cross-cell-type
  marker contamination and cell-type homogeneity.
when_to_use_negative:
- The input is a raw image or transcript file that has not yet been segmented — this
  skill evaluates segmentation output, it does not produce it.
- You need a general Xenium-based benchmark (e.g. Xenium-BreastCancer1 CellSPA metrics)
  rather than a cross-platform generalisability assessment on CosMx/MERSCOPE.
- The tissue or platform has no available single-cell reference and positive/negative
  marker definitions (e.g. no TISCH2/HLCA-derived marker files).
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0625
tools:
- name: BIDCell
  role: Segmentation framework whose CosMx-Lung and MERSCOPE-Melanoma outputs (expr_mat.csv,
    *_connected.tif) are assessed for B-cell marker contamination and entropy-versus-CV
    profiles; provides get_example_config('cosmx')/'merscope', stitch_nuclei(), segment_nuclei(),
    make_cell_gene_mat()
  repo: https://github.com/SydneyBioX/BIDCell
- name: NanoString CosMx
  role: SST platform supplying the Lung5-1 lung dataset on which B-cell contamination
    by T-cell markers is quantified
- name: Vizgen MERSCOPE
  role: SST platform supplying the melanoma patient-2 dataset on which cell-type entropy
    versus coefficient of variation is computed
- name: TISCH2
  role: Public repository from which single-cell references and positive/negative
    marker files (TISCH-NSCLC, TISCH-SKCM) are downloaded
- name: Human Cell Atlas
  role: Public repository for single-cell reference data, including HLCA for the lung
    analysis
- name: SydneyBioX/scClassify
  role: Source of example single-cell reference data for cell-type annotation used
    in BIDCell runs
  repo: https://github.com/SydneyBioX/scClassify
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:3489882595ca2072009947749b6240dc6b825b74db009887585184b9196c8774
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - BIDCell
  - Human Cell Atlas
  - NanoString CosMx
  - SydneyBioX/scClassify
  - TISCH2
  - Vizgen MERSCOPE
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bcell-tcell-marker-contamination-assessment

## Summary

Benchmark spatial-transcriptomics cell segmentation accuracy by measuring the percentage of B cells expressing negative (contamination-indicating) markers — e.g. T-cell markers misassigned to B cells — in BIDCell versus comparator segmentations on CosMx-Lung data, complemented by cell-type entropy versus coefficient-of-variation profiles on MERSCOPE-Melanoma.

## When to use

Use when validating a segmentation method's biological accuracy on non-Xenium SST platforms (NanoString CosMx lung, Vizgen MERSCOPE melanoma) by quantifying cross-cell-type marker contamination and cell-type homogeneity.

## When NOT to use

- The input is a raw image or transcript file that has not yet been segmented — this skill evaluates segmentation output, it does not produce it.
- You need a general Xenium-based benchmark (e.g. Xenium-BreastCancer1 CellSPA metrics) rather than a cross-platform generalisability assessment on CosMx/MERSCOPE.
- The tissue or platform has no available single-cell reference and positive/negative marker definitions (e.g. no TISCH2/HLCA-derived marker files).

## Inputs

- CosMx NSCLC Lung5-1 transcripts and FOV-separated DAPI/nuclei images
- MERSCOPE melanoma patient-2 transcripts and nuclei images
- Vendor .yaml configuration files (cosmx, merscope) with transcripts, image paths, and cpus edited
- Single-cell reference and positive/negative marker CSVs (HLCA/TISCH-NSCLC for lung, TISCH-SKCM for melanoma)
- expr_mat.csv cell-gene matrices per segmentation method

## Outputs

- Percentage of B cells expressing negative (contamination-indicating) markers per segmentation method (CosMx-Lung)
- Cell-type entropy versus coefficient of variation profiles (MERSCOPE-Melanoma)
- Fig. 4a-f generalisability benchmark panels
- *_connected.tif segmentation files and expr_mat.csv cell-gene matrices per platform

## How to apply

Run the segmentation pipeline (BIDCell or comparator) on the CosMx NSCLC Lung5-1 and MERSCOPE melanoma patient-2 datasets using vendor-specific .yaml configs (BIDCellModel.get_example_config('cosmx') / ('merscope')), including model.stitch_nuclei() for CosMx FOV-separated nuclei images before model.segment_nuclei(). Extract per-cell expression with model.make_cell_gene_mat() to produce expr_mat.csv per platform, with single-cell references and positive/negative marker files drawn from TISCH2 (TISCH-NSCLC, TISCH-SKCM) and the Human Cell Atlas (HLCA for lung). On the CosMx-Lung output, annotate cell types and compute the percentage of B cells expressing negative markers (e.g. T-cell markers), where lower contamination indicates more accurate cell segmentation; on MERSCOPE-Melanoma, compute cell-type entropy versus coefficient of variation to yield the Fig. 4a-f generalisability panels. Validate that each run yields a *_connected.tif segmentation file and an expr_mat.csv cell-gene matrix, and that BIDCell maintains low B-cell marker contamination and competitive entropy-versus-CV profiles relative to other methods.

## Related tools

- **BIDCell** (Segmentation framework whose CosMx-Lung and MERSCOPE-Melanoma outputs (expr_mat.csv, *_connected.tif) are assessed for B-cell marker contamination and entropy-versus-CV profiles; provides get_example_config('cosmx')/'merscope', stitch_nuclei(), segment_nuclei(), make_cell_gene_mat()) — https://github.com/SydneyBioX/BIDCell
- **NanoString CosMx** (SST platform supplying the Lung5-1 lung dataset on which B-cell contamination by T-cell markers is quantified)
- **Vizgen MERSCOPE** (SST platform supplying the melanoma patient-2 dataset on which cell-type entropy versus coefficient of variation is computed)
- **TISCH2** (Public repository from which single-cell references and positive/negative marker files (TISCH-NSCLC, TISCH-SKCM) are downloaded)
- **Human Cell Atlas** (Public repository for single-cell reference data, including HLCA for the lung analysis)
- **SydneyBioX/scClassify** (Source of example single-cell reference data for cell-type annotation used in BIDCell runs) — https://github.com/SydneyBioX/scClassify

## Examples

```
from bidcell import BIDCellModel; BIDCellModel.get_example_config('cosmx'); model = BIDCellModel('cosmx_example_config.yaml'); model.stitch_nuclei(); model.segment_nuclei(); model.generate_expression_maps(); model.generate_patches(); model.train(); model.predict(); model.make_cell_gene_mat()
```

## Evaluation signals

- Each pipeline run produces a *_connected.tif segmentation file (values = cell IDs) and an expr_mat.csv cell-gene matrix for the platform.
- On CosMx-Lung, BIDCell shows a lower percentage of B cells expressing negative (T-cell) markers than comparator segmentation methods, matching the reported finding.
- On MERSCOPE-Melanoma, entropy-versus-coefficient-of-variation plots reproduce the competitive profile reported in Fig. 4a-f.
- The .yaml configs are the vendor-specific examples (cosmx, merscope) with file paths and the cpus parameter edited for the host system.
- Training/prediction runs complete on a GPU with at least 12GB VRAM and ~4,000 training steps with the default UNet3+ architecture.

## Limitations

- A GPU with at least 12GB VRAM is strongly recommended for the deep learning component, and 32GB RAM for data processing; higher cpus settings may freeze the system or raise pickle.UnpicklingError: pickle data was truncated.
- In MERSCOPE data with cell membrane markers, 25% of cells lack nuclei in their segmentation (elongated melanocytes/fibroblasts), complicating boundary identification.
- Achieving more homogenous cell bodies can also result from arbitrary over-segmentation of nuclei, so multiple metrics (contamination and entropy-vs-CV) should be used together.
- Manual curation of positive/negative marker lists is advised; a larger number of positive markers tends to increase predicted cell size.

## Evidence

- [results] BIDCell generalises to non-Xenium platforms; on CosMx-Lung it had a lower percentage of B cells expressing negative (contamination-indicating) markers than comparator methods: [evidence span withheld — anchor status: not_found]
- [readme] model.stitch_nuclei() # for when nuclei images are separated into FOVs (e.g., CosMx): "model.stitch_nuclei() # for when nuclei images are separated into FOVs (e.g., CosMx)"
- [readme] These can be downloaded from public repositories such as TISCH2, Allen Brain Map, and the Human Cell Atlas.: "BIDCell uses single-cell reference data for improved results. These can be downloaded from public repositories such as TISCH2, Allen Brain Map, and the Human Cell Atlas."
- [readme] A GPU with at least 12GB VRAM is strongly recommended for the deep learning component, and 32GB RAM for data processing.: "A GPU with at least 12GB VRAM is strongly recommended for the deep learning component, and 32GB RAM for data processing."
- [discussion] While achieving more homogenous cell bodies is desired, it can also result from the arbitrary over-segmentation of nuclei.: "While achieving more homogenous cell bodies is desired, it can also result from the arbitrary over-segmentation of nuclei."
