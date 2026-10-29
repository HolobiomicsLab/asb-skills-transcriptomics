---
name: scrna-seq-malignant-cell-classification
description: Use when annotating patient-derived cancer scRNA-seq or spatial transcriptomics
  data to label malignant cells within an already-identified cancerous lineage and
  to obtain a continuous malignancy probability per cell or spot.
when_to_use_negative:
- The dataset has not been pre-annotated for basic cell types — the tool relies on
  prerequisite identification of basic cell types and is designed to identify malignant
  cells from cancerous lineages only, not to distinguish malignant cells from all
  other non-malignant cell types
- The cancer type has unknown or controversial origins (e.g., synovial sarcoma or
  certain brain tumors), where the article states capabilities require further testing
- Input is normal (non-cancer) tissue data where no malignant cells are expected and
  no prior lineage annotation has been performed
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMalignantFinder
  role: 'Primary classifier: loads pretrained model.joblib + ordered_feature.tsv and
    predicts Normal/Malignant with malignancy_probability per cell'
  repo: https://github.com/Jonyyqn/scMalignantFinder
- name: PreCanCell
  role: Baseline ensemble k-NN classifier for accuracy/balanced-accuracy and sensitivity/specificity
    comparison
  repo: https://github.com/WangX-Lab/PreCanCell
- name: ikarus
  role: Baseline logistic classifier distinguishing tumor from normal cells for benchmark
    comparison
  repo: https://github.com/BIMSBbioinfo/ikarus
- name: Cancer-Finder
  role: Baseline domain-generalization cancer cell annotation method for benchmark
    comparison
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: CopyKAT
  role: Baseline CNV-inference-based tumor/normal separation method for benchmark
    comparison
  repo: https://github.com/navinlabcode/copykat
- name: scATOMIC
  role: Optional upstream pan-cancer cell type annotation integrated into the current
    version of scMalignantFinder
  repo: https://github.com/abelson-lab/scATOMIC
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:23bc4e0b954bc22b3eaa385263198da3c7015350c0492cfb5724ae441129a604
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  repo_url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - Cancer-Finder
  - CopyKAT
  - PreCanCell
  - ikarus
  - scATOMIC
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# scrna-seq-malignant-cell-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies malignant versus normal cells in cancer scRNA-seq (and spatial transcriptomics) data using scMalignantFinder, a machine-learning classifier trained on a signature-calibrated training set with a union of DEG features, returning a per-cell 'Normal'/'Malignant' prediction plus a malignancy_probability.

## When to use

Use when annotating patient-derived cancer scRNA-seq or spatial transcriptomics data to label malignant cells within an already-identified cancerous lineage and to obtain a continuous malignancy probability per cell or spot.

## When NOT to use

- The dataset has not been pre-annotated for basic cell types — the tool relies on prerequisite identification of basic cell types and is designed to identify malignant cells from cancerous lineages only, not to distinguish malignant cells from all other non-malignant cell types
- The cancer type has unknown or controversial origins (e.g., synovial sarcoma or certain brain tumors), where the article states capabilities require further testing
- Input is normal (non-cancer) tissue data where no malignant cells are expected and no prior lineage annotation has been performed

## Inputs

- Patient-derived cancer scRNA-seq datasets as AnnData/.h5ad with raw counts (or test_cancerCellLine.h5ad / test_TabulaSapiens.h5ad gold-standard test sets from Zenodo 17888140)
- Pretrained model file model.joblib from Zenodo record 17888140
- Ordered feature list ordered_feature.tsv from Zenodo record 17888140
- Ground-truth malignant/normal cell labels (for evaluation against baselines)

## Outputs

- Per-cell scMalignantFinder_prediction ('Normal'/'Malignant') stored in the result AnnData
- Per-cell malignancy_probability (predicted probability of being malignant)
- benchmark_metrics_comparison.csv with accuracy and balanced accuracy per method (when benchmarking against PreCanCell, ikarus, Cancer-Finder, CopyKAT)
- Malignant region labels for spatial transcriptomics spots (when run in ST mode)

## How to apply

Install scMalignantFinder (pip) and download the pretrained model.joblib and ordered_feature.tsv from Zenodo record 17888140 into a pretrained_model directory. Normalize each input AnnData (.h5ad) with sc.pp.normalize_total(target_sum=1e4) — set norm_type=True and skip log-transformation, since the model expects total-count-normalized values — then set pretrain_dir so the model and ordered feature list load automatically, call model.load() and model.predict(). The prediction returns two per-cell outputs: scMalignantFinder_prediction ('Normal'/'Malignant') and malignancy_probability (predicted probability of being malignant), the latter useful for capturing dynamic characteristics during tumor progression. For benchmarking, run baselines PreCanCell, ikarus, Cancer-Finder, and CopyKAT on the same datasets and compute accuracy/balanced accuracy against ground-truth labels; the article reports scMalignantFinder at 0.824 accuracy / 0.732 balanced accuracy across 13 patient-derived datasets, and 1.000 sensitivity / 0.786 specificity on gold-standard cancer cell line (53,513 cells) and non-cancer donor (104,148 cells) datasets. For spatial data, extend the same classifier without retraining and refine malignant regions via multi-feature clustering combining malignancy_probability, Malignant_up features, and image_score, optionally with spatial_nn=True to refine region labels using spatial neighborhood information.

## Related tools

- **scMalignantFinder** (Primary classifier: loads pretrained model.joblib + ordered_feature.tsv and predicts Normal/Malignant with malignancy_probability per cell) — https://github.com/Jonyyqn/scMalignantFinder
- **PreCanCell** (Baseline ensemble k-NN classifier for accuracy/balanced-accuracy and sensitivity/specificity comparison) — https://github.com/WangX-Lab/PreCanCell
- **ikarus** (Baseline logistic classifier distinguishing tumor from normal cells for benchmark comparison) — https://github.com/BIMSBbioinfo/ikarus
- **Cancer-Finder** (Baseline domain-generalization cancer cell annotation method for benchmark comparison) — https://github.com/Patchouli-M/SequencingCancerFinder
- **CopyKAT** (Baseline CNV-inference-based tumor/normal separation method for benchmark comparison) — https://github.com/navinlabcode/copykat
- **scATOMIC** (Optional upstream pan-cancer cell type annotation integrated into the current version of scMalignantFinder) — https://github.com/abelson-lab/scATOMIC

## Examples

```
pip install scMalignantFinder; adata_norm = sc.pp.normalize_total(adata, target_sum=1e4); model = scMalignantFinder(pretrain_dir='pretrained_model'); model.load(); result_adata = model.predict()
```

## Evaluation signals

- Reproduced accuracy and balanced accuracy across the 13 patient-derived test datasets should match reported values (scMalignantFinder 0.824/0.732; PreCanCell 0.713/0.613; ikarus 0.446/0.533; Cancer-Finder 0.654/0.531; CopyKAT 0.427/0.571)
- On test_cancerCellLine.h5ad, sensitivity across the 53,513 gold-standard malignant cells should reach 1.000 (baselines: PreCanCell 0.996, ikarus 0.834, Cancer-Finder 1.000, CopyKAT 0.594)
- On test_TabulaSapiens.h5ad, specificity across the 104,148 gold-standard normal epithelial cells should reach 0.786 (baselines: PreCanCell 0.503, ikarus 0.642, Cancer-Finder 0.022, CopyKAT 0.397)
- Verify the input was normalized with sc.pp.normalize_total(target_sum=1e4) and NOT log-transformed before model.predict(), matching the norm_type=True contract
- Check that every cell in the result AnnData carries both scMalignantFinder_prediction and a malignancy_probability in [0, 1]

## Limitations

- Requires prerequisite identification of basic cell types; malignant cell identification is restricted to cancerous lineages
- Capabilities untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors
- ST malignant-region identification depends on continuous accumulation of expert-curated ST data, which is not yet available, and additional spatial features (e.g., spatial distribution probability of malignant cells) are not yet incorporated

## Evidence
