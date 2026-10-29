---
name: marker-gene-selection-per-cell-type
description: Use when you have a pre-annotated reference scRNA-seq dataset and need
  to learn cell-type marker lists (with rejection thresholds) for downstream reference-based
  cell type identification instead of using a curated marker DB.
when_to_use_negative:
- You already have a curated marker list/database (e.g. Panglao DB or CellMarker DB)
  and want marker-based identification — use MarkerCount() marker-based mode or HiCAT
  instead of training from a reference
- No cell type annotations are available for the input dataset — this skill requires
  a pre-annotated reference to compute the gene-cell type association score
- The goal is test-phase cell identification of new cells rather than constructing
  the reference training model
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MarkerCount
  role: Provides the reference-based training workflow (binarization, marker scoring
    with sm,j, top-N selection, threshold learning) implemented in marker_count.py's
    MarkerCount_Ref function
  repo: https://github.com/combio-dku/MarkerCount
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:57047cb438fa6f25f6fbfb03dcb1b7db46009fbc13b4054f1f78f90db713b08d
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
  - MarkerCount
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# marker-gene-selection-per-cell-type

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives per-cell-type marker gene sets from a pre-annotated single-cell RNA-seq reference by binarizing expression, computing MarkerCount's gene-cell type association score, and selecting the top N=18 markers per type. This training step replaces curated marker databases with data-driven markers before test-phase identification.

## When to use

Use when you have a pre-annotated reference scRNA-seq dataset and need to learn cell-type marker lists (with rejection thresholds) for downstream reference-based cell type identification instead of using a curated marker DB.

## When NOT to use

- You already have a curated marker list/database (e.g. Panglao DB or CellMarker DB) and want marker-based identification — use MarkerCount() marker-based mode or HiCAT instead of training from a reference
- No cell type annotations are available for the input dataset — this skill requires a pre-annotated reference to compute the gene-cell type association score
- The goal is test-phase cell identification of new cells rather than constructing the reference training model

## Inputs

- Pre-annotated reference single-cell RNA-seq gene expression matrix (.h5ad or similar count matrix)
- Per-cell cell type annotations for the reference dataset
- Hyper-parameters: occurrence frequency threshold fth=0.9, score exponent n=2, number of markers N=18, target FPR p

## Outputs

- markers_and_thresholds file containing per-cell-type marker lists (up to 18 genes each)
- Per-gene target/non-target occurrence frequencies for each cell type
- Learned rejection threshold tm per cell type (within [0,1] range of normalized marker counts)

## How to apply

Load the pre-annotated reference gene expression matrix and binarize it per the MarkerCount rule (bij=1 if xij>0, else 0), ignoring expression levels to suppress noise and batch effects. For each annotated cell type m, pre-filter candidate genes by requiring occurrence frequency (1/Cm)∑i∈Cm bij >= fth=0.9 (heuristic choice), then compute the marker score sm,j = (1/Cm)∑i∈Cm bij * (1-(1/C̄m)∑i∈C̄m bij)^n with hyper-parameter n=2, which rewards genes expressed in target cells and penalizes off-target expression. Sort scores sm,j per cell type in descending order and take the top N=18 genes as the marker set (18 was chosen empirically as best-performing; overlaps between cell types are allowed). On the reference cells, compute normalized marker counts yi,m and sort them descending to find the minimum rejection threshold tm such that FPR <= p. Persist the per-cell-type marker lists, target/non-target occurrence frequencies, and thresholds to the markers_and_thresholds output.

## Related tools

- **MarkerCount** (Provides the reference-based training workflow (binarization, marker scoring with sm,j, top-N selection, threshold learning) implemented in marker_count.py's MarkerCount_Ref function) — https://github.com/combio-dku/MarkerCount

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref; df_res = MarkerCount_Ref(reference_data, ref_labels, ...)
```

## Evaluation signals

- The markers_and_thresholds output file exists and contains an entry for every annotated reference cell type
- Each cell type's marker set contains up to N=18 genes
- Every learned rejection threshold tm lies within the [0,1] range of normalized marker counts
- Every selected marker passed the pre-filter: occurrence frequency in target cells >= fth=0.9
- Marker sets may overlap between cell types; verify scores were computed with exponent n=2 using both target occurrence (1/Cm)∑bij and non-target penalty (1-(1/C̄m)∑bij)^n

## Limitations

- For certain cell types (beta/delta/epsilon/gamma cells in pancreas, dendritic cells in blood, B cells in tumors, T cells in lung), target and non-target cells are not clearly separable by normalized marker counts alone, so identification yields considerable errors
- Marker selection quality depends on the reference annotation; handling not-clearly-separable sub-populations such as monocytes, macrophages, and dendritic cells requires further studies
- Whether binarization's batch-effect suppression actually explains MarkerCount's superior performance is unclear per the authors

## Evidence
