---
name: hyperparameter-sweep-analysis
description: Use when a method has a tunable hyper-parameter (e.g. number of markers
  per cell type) whose value must be chosen empirically by sweeping candidate values
  and comparing performance measures across datasets.
when_to_use_negative:
- The hyper-parameter already has a theoretically or empirically justified fixed value
  and only needs to be applied, not re-tuned
- Only a single dataset is available, so 'best for most datasets' cannot be assessed
  via cross-evaluation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MarkerCount
  role: Marker-selection and reference-based identification tool whose MarkerCount_Ref
    function is run for each candidate N during the sweep
  repo: github.com/combio-dku/MarkerCount
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:992ec99580bd059d3ae3c9fb9cc197feafc3721bc21589066c81c3c7edaacab0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/combio-dku/MarkerCount
  tools:
  - MarkerCount
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hyperparameter-sweep-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematic performance-vs-hyperparameter evaluation, exemplified by MarkerCount's marker-count sweep that fixed N=18 markers per cell type because 18 showed the best performance for most of the eight evaluated scRNA-seq datasets.

## When to use

Use when a method has a tunable hyper-parameter (e.g. number of markers per cell type) whose value must be chosen empirically by sweeping candidate values and comparing performance measures across datasets.

## When NOT to use

- The hyper-parameter already has a theoretically or empirically justified fixed value and only needs to be applied, not re-tuned
- Only a single dataset is available, so 'best for most datasets' cannot be assessed via cross-evaluation

## Inputs

- Eight annotated scRNA-seq datasets (GSE84133, GSE85241, GSE100866, GSE93421, ERP114453, SRP218543, GSE72056, GSE103322) with cell-type labels
- Candidate values of the hyper-parameter N (number of markers per cell type)
- Fixed method settings: fth=0.9, relative-weight exponent n=2, PCA 15 components, 0.8 unassigned-portion reassignment threshold

## Outputs

- marker_count_sweep.csv — performance-vs-N table with the five measures (C, E, EA, CUA, EUA) per dataset and candidate N
- Selected fixed hyper-parameter value (N=18 markers per cell type) with supporting evidence across datasets

## How to apply

Sweep candidate values of the hyper-parameter while holding other settings fixed. For MarkerCount's marker-count sweep: binarize expression (bij=1 if xij>0) for eight annotated datasets (GSE84133, GSE85241, GSE100866, GSE93421, ERP114453, SRP218543, GSE72056, GSE103322); for each candidate N compute the marker score sm,j using occurrence-frequency threshold fth=0.9 and relative-weight exponent n=2, selecting the top-N genes per cell type. Run MarkerCount_Ref in reference-based mode via circular cross-evaluation (one dataset as test, the rest as reference), obtaining rejection thresholds from reference data per cell type, then compute the five performance measures (C, E, EA, CUA, EUA) on test predictions applying cluster-basis reassignment with PCA (15 components), GMM clustering, and the 0.8 unassigned-portion reassignment threshold. Assemble the sweep into a table (e.g. marker_count_sweep.csv) and choose the value that performs best for most datasets — here N=18.

## Related tools

- **MarkerCount** (Marker-selection and reference-based identification tool whose MarkerCount_Ref function is run for each candidate N during the sweep) — github.com/combio-dku/MarkerCount

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref, MarkerCount
```

## Evaluation signals

- The sweep table shows N=18 yielding the best performance for most of the eight datasets, consistent with the reported fixed hyper-parameter choice
- All five measures (C, E, EA, CUA, EUA) are reported for every candidate N in the sweep
- Cross-evaluation roles rotate circularly so each of the eight datasets serves as the test set exactly once per N
- Reassignment settings remain fixed across the sweep (PCA 15 components, GMM clustering, 0.8 unassigned-portion threshold), isolating the effect of N

## Limitations

- The chosen value (N=18) is dataset-dependent — it was best 'for most datasets used in this work', not guaranteed optimal for new tissues or assays
- Handling not-clearly-separable sub-populations (monocytes, macrophage, dendritic cells) remains an open issue that affects sweep conclusions for those types
- Proper identification requires well-clustered cell populations, which is not always the case

## Evidence
