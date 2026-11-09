---
name: marker-list-curation-from-public-databases
description: Use when running marker-based cell type identification without a pre-annotated
  reference dataset and you must assemble cell-type marker lists from Panglao DB or
  CellMarker DB, choosing how many markers per type.
when_to_use_negative:
- A pre-annotated reference dataset is available and reference-based MarkerCount_Ref()
  can derive markers via marker score sm,j instead of database curation
- Cell types of interest are not covered by Panglao DB or CellMarker DB and no expert-curated
  markers exist (the paper notes taxonomy is not standardized and researchers must
  prepare their own lists)
- The analysis uses HiCAT's hierarchical taxonomy directly against its bundled R&D
  Systems-derived marker file (cell_markers_rndsystems_rev.tsv), which already provides
  curated markers
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: MarkerCount
  role: Marker-based cell type identifier whose MarkerCount() function consumes the
    curated marker lists; marker list size N is its hyper-parameter fixed at 18
  repo: https://github.com/combio-dku/MarkerCount
- name: HiCAT
  role: Improved marker-based (hierarchical) successor of MarkerCount; accepts user-prepared
    marker DBs in a 6-column .tsv format
  repo: https://github.com/combio-dku/HiCAT
- name: SCANPY
  role: Loads the eight scRNA-seq datasets as AnnData/.h5ad objects prior to binarization
    and marker counting
- name: Numpy
  role: Numerical computation for binarization (bij=1 if xij>0), normalized marker
    counts, and penalty-weight arithmetic
- name: sklearn
  role: Bimodal EM (Gaussian mixture, k=2) fitting of normalized marker counts to
    set rejection thresholds
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:875c18fbe9a2a60a3963cd828b42a9b1258bd48a68cad5ce000207653e5b20ab
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
  - HiCAT
  - MarkerCount
  - Numpy
  - SCANPY
  - scipy
  - sklearn
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# marker-list-curation-from-public-databases

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curate per-cell-type marker gene lists from public marker databases (Panglao DB, CellMarker DB), sized around ~18 markers per cell type, for use in marker-based cell type identification with MarkerCount. List size is a sensitivity-relevant hyper-parameter since MarkerCount fixes N=18 as the marker count per cell type.

## When to use

Use when running marker-based cell type identification without a pre-annotated reference dataset and you must assemble cell-type marker lists from Panglao DB or CellMarker DB, choosing how many markers per type.

## When NOT to use

- A pre-annotated reference dataset is available and reference-based MarkerCount_Ref() can derive markers via marker score sm,j instead of database curation
- Cell types of interest are not covered by Panglao DB or CellMarker DB and no expert-curated markers exist (the paper notes taxonomy is not standardized and researchers must prepare their own lists)
- The analysis uses HiCAT's hierarchical taxonomy directly against its bundled R&D Systems-derived marker file (cell_markers_rndsystems_rev.tsv), which already provides curated markers

## Inputs

- Cell type names for the target tissue(s) from manual annotations or study design
- Marker gene entries retrieved from Panglao DB and CellMarker DB
- Test scRNA-seq expression matrices (.h5ad / AnnData) from GSE84133, GSE85241, GSE100866, GSE93421, ERP114453, SRP218543, GSE72056, GSE103322
- Optional HiCAT marker DB as .tsv with columns tissue, cell_type_major, cell_type_minor, cell_type_subset, exp, markers

## Outputs

- Per-cell-type marker lists of varying sizes n centered on ~18 markers
- Marker matrix / .tsv marker DB formatted for MarkerCount or HiCAT
- marker_size_sensitivity.csv aggregating C, E, EA, CUA, EUA across list sizes and datasets

## How to apply

Download marker sets for each target cell type from Panglao DB and CellMarker DB and assemble a per-cell-type list whose size n is swept around the guideline value of 18, since MarkerCount fixed N=18 because it 'showed the best performance for most of the datasets evaluated'. Format lists for the tool at hand: MarkerCount's marker-based mode consumes a marker matrix of gene symbols per cell type, while HiCAT requires a tab-separated file with columns 'tissue', 'cell_type_major', 'cell_type_minor', 'cell_type_subset', 'exp' ('pos', 'neg', or 'sec') and comma-separated 'markers'. For each candidate list size, binarize the test dataset's expression matrix (bij=1 if xij>0), compute normalized marker counts, apply the penalty weight w=1+3e-(n-1)/2-1 for uneven marker counts, obtain rejection thresholds via bimodal EM (k=2 Gaussian mixture) fitting, and run the MarkerCount() function. Evaluate each size with the five performance measures (C, E, EA, CUA, EUA) against manual annotations across the eight datasets (GSE84133, GSE85241, GSE100866, GSE93421, ERP114453, SRP218543, GSE72056, GSE103322) and select the size yielding best C / lowest E+EUA+EA.

## Related tools

- **MarkerCount** (Marker-based cell type identifier whose MarkerCount() function consumes the curated marker lists; marker list size N is its hyper-parameter fixed at 18) — https://github.com/combio-dku/MarkerCount
- **HiCAT** (Improved marker-based (hierarchical) successor of MarkerCount; accepts user-prepared marker DBs in a 6-column .tsv format) — https://github.com/combio-dku/HiCAT
- **SCANPY** (Loads the eight scRNA-seq datasets as AnnData/.h5ad objects prior to binarization and marker counting)
- **Numpy** (Numerical computation for binarization (bij=1 if xij>0), normalized marker counts, and penalty-weight arithmetic)
- **sklearn** (Bimodal EM (Gaussian mixture, k=2) fitting of normalized marker counts to set rejection thresholds)

## Examples

```
from MarkerCount.marker_count import MarkerCount; df_res = MarkerCount(adata, markers_dict, tissue_list)
```

## Evaluation signals

- The swept list sizes span multiple values centered on 18 markers per cell type
- The size yielding best C (correct assignment) and lowest E+EUA+EA matches the paper's report that 18 markers showed the best performance for most datasets
- Each curated list contains only gene symbols resolvable in the test dataset's AnnData var_names, with per-cell-type counts equal to n
- Penalty weight w=1+3e-(n-1)/2-1 is applied per cell type according to its marker count, and occurrence frequency threshold fth=0.9 is respected when filtering markers
- Output table marker_size_sensitivity.csv contains all five measures (C, E, EA, CUA, EUA) per list size per dataset

## Limitations

- Cell types and taxonomy are not standardized, so researchers must prepare and adapt their own marker lists for the specific study and objective
- For hard-to-separate populations (beta/delta/epsilon/gamma pancreas cells, dendritic cells, B cells in tumors, T cells in lung, monocytes/macrophages), normalized marker counts alone yield considerable errors regardless of list size
- The N=18 optimum was chosen empirically from the eight datasets evaluated and may not transfer to all tissues or platforms
- Proper identification additionally requires well-clustered cell populations, which is not always the case

## Evidence

- [readme] Marker file must be a tap-separated-value file (.tsv) with 6 columns, "tissue", "cell_type_major", "cell_type_minor", "cell_type_subset", "exp" and "markers".: [evidence span withheld — anchor status: external_doc]
