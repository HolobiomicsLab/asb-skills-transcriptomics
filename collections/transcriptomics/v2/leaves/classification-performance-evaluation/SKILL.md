---
name: classification-performance-evaluation
description: Use when comparing cell-type identifiers (MarkerCount, SingleR, scmap,
  CaSTLe, CHETAH) across annotated scRNA-seq dataset pairs where some test populations
  are absent from the reference, so both assignment accuracy and unknown-population
  rejection must be scored.
when_to_use_negative:
- The test data contains only cell types also present in the reference and no unknown-population
  rejection needs to be measured — plain accuracy metrics suffice without the CUA/EUA
  rejection measures
- Cell-type annotations of the two datasets cannot be reconciled into a shared taxonomy,
  so 'correct' cannot be defined per split
- No manual ground-truth annotations are available for the test dataset, making computation
  of C, E, EA, CUA, EUA impossible
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: MarkerCount
  role: Identifier under evaluation in both marker-based (MarkerCount) and reference-based
    (MarkerCount_Ref) modes, with N=18 markers per cell type and FPR-based rejection
    thresholds
  repo: https://github.com/combio-dku/MarkerCount
- name: SCANPY
  role: Preprocessing and loading of scRNA-seq datasets for the baseline identifiers
    and the circular cross-evaluation
- name: scmap
  role: Baseline identifier (scmap cell/cluster modes) run on the same circular splits
    for comparison
- name: CHETAH
  role: Baseline hierarchical identifier run on the same circular splits for comparison
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:e9e0fcd734396a6bca1b5efd593734753e12c2ceab469d5903bc9ab67acd52e8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/combio-dku/MarkerCount
  repo_url: https://github.com/combio-dku/MarkerCount
  tools:
  - CHETAH
  - MarkerCount
  - SCANPY
  - numpy
  - pandas
  - scikit-learn
  - scipy
  - scmap
  - sklearn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# classification-performance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates scRNA-seq cell-type identifiers with the five cross-evaluation measures used in the MarkerCount paper — correct (C), error (E), erroneously assigned (EA), correctly unassigned (CUA), and erroneously unassigned (EUA) — on circular dataset splits to quantify both identification of known populations and rejection of unknown ones.

## When to use

Use when comparing cell-type identifiers (MarkerCount, SingleR, scmap, CaSTLe, CHETAH) across annotated scRNA-seq dataset pairs where some test populations are absent from the reference, so both assignment accuracy and unknown-population rejection must be scored.

## When NOT to use

- The test data contains only cell types also present in the reference and no unknown-population rejection needs to be measured — plain accuracy metrics suffice without the CUA/EUA rejection measures
- Cell-type annotations of the two datasets cannot be reconciled into a shared taxonomy, so 'correct' cannot be defined per split
- No manual ground-truth annotations are available for the test dataset, making computation of C, E, EA, CUA, EUA impossible

## Inputs

- Manually annotated scRNA-seq datasets in .h5ad (AnnData) format, e.g. GSE84133, GSE85241, GSE100866, GSE93421, ERP114453, SRP218543, GSE72056, GSE103322
- Prediction tables mapping each cell to a called label or 'unassigned' for each identifier and circular split
- Shared broad cell-type taxonomy renaming per dataset pair

## Outputs

- Per-dataset, per-method table of the five measures C, E, EA, CUA, EUA (values between 0 and 1), e.g. tumor_cross_evaluation_metrics.csv
- Cross-evaluation benchmark table comparing reference- and marker-based MarkerCount against SingleR, scmap(cell/cluster), CaSTLE, and CHETAH

## How to apply

For each circular split (one dataset as test, the other as reference), run each identifier — e.g. train MarkerCount_Ref with N=18 markers per cell type, fth=0.9 occurrence frequency, FPR-based rejection thresholds, and cluster-basis reassignment when the unassigned fraction is below 0.8 — and score predictions against manual annotations with the five measures C, E, EA, CUA, and EUA, each a fraction between 0 and 1. Preprocess datasets consistently (e.g. with Scanpy) and rename cell-type annotations to a shared broad taxonomy per dataset pair, since annotation granularity differs between reference and test data (e.g. pancreas GSE84133/GSE85241, blood GSE100866/GSE93421, lung ERP114453/SRP218543, tumors GSE72056/GSE103322). For unknown-population tests (tumor cells scored against a blood reference), treat tumor cells as correctly unassigned so high CUA with low EA approximates the ideal rejection pattern. Aggregate per-dataset, per-method measures into a table and cross-check against the deposited results on figshare (MarkerCount_cell_type_identification_results).

## Related tools

- **MarkerCount** (Identifier under evaluation in both marker-based (MarkerCount) and reference-based (MarkerCount_Ref) modes, with N=18 markers per cell type and FPR-based rejection thresholds) — https://github.com/combio-dku/MarkerCount
- **SCANPY** (Preprocessing and loading of scRNA-seq datasets for the baseline identifiers and the circular cross-evaluation)
- **scmap** (Baseline identifier (scmap cell/cluster modes) run on the same circular splits for comparison)
- **CHETAH** (Baseline hierarchical identifier run on the same circular splits for comparison)

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref, MarkerCount; df_res = MarkerCount_Ref(data, markers_per_type=18)
```

## Evaluation signals

- Output table contains columns C, E, EA, CUA, EUA per dataset-method pair with all values between 0 and 1
- Reproduced measures match the deposited figshare results (MarkerCount_cell_type_identification_results) within tolerance
- For tumor-vs-blood evaluations (GSE72056, GSE103322 with GSE100866/GSE93421 reference), CUA is high while EA is small, matching the reported ideal rejection pattern
- Reproduced ranking is consistent with the paper: MarkerCount best for blood and tumor datasets; SingleR best for pancreas and lung with MarkerCount close behind
- Marker-list-size sweep centers on N=18, with 18 yielding the best C / lowest E+EA+EUA for most datasets

## Limitations

- Existing rejection functions involve a tradeoff between erroneously unassigned and erroneously assigned measures, so no single identifier dominates on all criteria
- No standardized cell type taxonomy exists; researchers must prepare their own marker lists / taxonomy renaming per study
- Proper identification requires well-clustered cell populations, which is not always the case; not-clearly-separable sub-populations such as monocytes, macrophages, and dendritic cells remain an open issue
- It is unclear whether MarkerCount's superior performance in Fig. 3(b), (e), (f) stems from binarized gene expression suppressing batch effect

## Evidence

- [abstract] The results show that MarkerCount performs better in the identification of known and unknown cell populations than other reference- and marker-based cell type identifiers for most datasets analyzed: [evidence span withheld — anchor status: not_found]
