---
name: marker-based-cell-type-identification
description: Use when annotating scRNA-seq cells or clusters with only external marker
  lists (no annotated reference), and you need conservative rejection of unknown cell
  types plus reassignment of uncertain cells.
when_to_use_negative:
- A pre-annotated reference dataset exists and reference-based identification is preferred
  (use MarkerCount_Ref() instead)
- Cell populations are not well clustered — the cluster-basis reassignment step requires
  well-clustered populations to work properly
- Marker lists cannot be prepared because cell types/taxonomy for the specific study
  are undefined (no standardized taxonomy exists)
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MarkerCount
  role: 'Implements the marker-based identification (MarkerCount() function): binarization,
    marker counting, EM rejection thresholds, and cluster-basis reassignment'
  repo: https://github.com/combio-dku/MarkerCount
- name: HiCAT
  role: Improved marker-based, hierarchical successor of MarkerCount using multi-level
    taxonomy marker files (.tsv with pos/neg/sec markers)
  repo: https://github.com/combio-dku/HiCAT
- name: SCANPY
  role: Loads/preprocesses the .h5ad scRNA-seq datasets; required only to show results,
    not for MarkerCount itself
- name: Panglao DB
  role: Source of externally curated per-cell-type marker lists for marker-based mode
- name: CellMarker DB
  role: Source of externally curated per-cell-type marker lists for marker-based mode
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:25f3ea5966ed34974502e1be527cdd7f801baffb07c1f8570d91b199bbddfc4c
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
  - CellMarker DB
  - HiCAT
  - MarkerCount
  - Panglao DB
  - SCANPY
  - scikit-learn
  - scipy
  - scmap
  - sklearn
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# marker-based-cell-type-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MarkerCount's marker-based cell type identification assigns cell types by counting expressed markers (binarized expression) from user-supplied marker lists (e.g., Panglao DB / CellMarker DB, ~18 markers per cell type), applying rejection thresholds and cluster-basis reassignment to flag unknown populations without requiring a pre-annotated reference.

## When to use

Use when annotating scRNA-seq cells or clusters with only external marker lists (no annotated reference), and you need conservative rejection of unknown cell types plus reassignment of uncertain cells.

## When NOT to use

- A pre-annotated reference dataset exists and reference-based identification is preferred (use MarkerCount_Ref() instead)
- Cell populations are not well clustered — the cluster-basis reassignment step requires well-clustered populations to work properly
- Marker lists cannot be prepared because cell types/taxonomy for the specific study are undefined (no standardized taxonomy exists)

## Inputs

- scRNA-seq expression matrix (.h5ad) without annotations
- Per-cell-type marker list (e.g., curated from Panglao DB or CellMarker DB, ~18 markers per cell type, .csv/.tsv marker matrix)
- Number of markers per cell type (N=18 hyper-parameter)

## Outputs

- Per-cell cell type assignments with 'unassigned' labels for rejected/unknown cells
- Normalized marker count matrix per cell type
- Performance measures (C, E, EA, CUA, EUA) when manual annotations are available

## How to apply

Binarize the test expression matrix (bij=1 if xij>0) and compute normalized marker counts yi,m per cell for each marker list. Apply the cell-type-specific rejection thresholds obtained by bimodal EM (2-component Gaussian mixture) fitting of the normalized marker counts, using penalty weight w=1+3e-(n-1)/2-1 for uneven marker numbers (N=18 per cell type was chosen as best-performing). Reduce dimensionality with 15-component PCA, cluster with a Gaussian mixture model at Ncell/2 clusters, and reassign unassigned cells by weighted Mahalanobis distance (regularization rho=0.1) when the unassigned fraction is below 0.8 (best value in a 0.1–0.9 sweep). Run the MarkerCount() function (Python, from the MarkerCount/HiCAT package; also callable from R via reticulate) on .h5ad data with marker matrices in .csv/.tsv format.

## Related tools

- **MarkerCount** (Implements the marker-based identification (MarkerCount() function): binarization, marker counting, EM rejection thresholds, and cluster-basis reassignment) — https://github.com/combio-dku/MarkerCount
- **HiCAT** (Improved marker-based, hierarchical successor of MarkerCount using multi-level taxonomy marker files (.tsv with pos/neg/sec markers)) — https://github.com/combio-dku/HiCAT
- **SCANPY** (Loads/preprocesses the .h5ad scRNA-seq datasets; required only to show results, not for MarkerCount itself)
- **Panglao DB** (Source of externally curated per-cell-type marker lists for marker-based mode)
- **CellMarker DB** (Source of externally curated per-cell-type marker lists for marker-based mode)

## Examples

```
from MarkerCount.marker_count import MarkerCount; df_res = MarkerCount(<arguments>)
```

## Evaluation signals

- Identification accuracy assessed with the five measures C, E, EA, CUA, EUA against manual annotations; best list sizes centered on 18 markers per cell type should yield lowest E+EUA+EA
- Reassignment threshold 0.8 should outperform other values in the 0.1–0.9 range at 0.1 steps
- Reassignment should only trigger when the portion of unassigned cells is below the threshold (applied when unassigned fraction < 0.8)
- Per-cell-type rejection thresholds from bimodal EM fitting (k=2 Gaussian mixture) should separably fit the normalized marker count distributions
- Cross-check reproduced results against the deposited performance table on figshare (MarkerCount_cell_type_identification_results)

## Limitations

- For some cell types (beta/delta/epsilon/gamma cells in pancreas, dendritic cells in blood, B cells in tumors, T cells in lung), identification via normalized marker counts alone yields considerable errors
- Handling not-clearly-separable sub-populations such as monocytes, macrophage, and dendritic cells requires further studies
- No standardized cell type taxonomy exists, so researchers must prepare their own marker lists for the specific study and objective
- Proper identification requires well-clustered cell populations, which is not always the case
- It is unclear whether superior performance stems from binarized gene expression suppressing batch effect

## Evidence
