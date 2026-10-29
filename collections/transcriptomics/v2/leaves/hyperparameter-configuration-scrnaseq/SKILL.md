---
name: hyperparameter-configuration-scrnaseq
description: Reference for setting MarkerCount/HiCAT hyper-parameters (N, fth, score
  exponent, reassignment cutoff) when running reference-based or marker-based cell-type
  identification on scRNA-seq count matrices.
when_to_use_negative:
- The marker sets or thresholds are already provided (e.g. saved from a previous training
  run in a markers_and_thresholds file) — re-running hyper-parameter selection would
  be redundant
- The task is hierarchical marker-based annotation with an existing taxonomy tree;
  HiCAT's marker DB format (tissue/cell_type_major/cell_type_minor/cell_type_subset/exp/markers
  columns) governs configuration instead
- The goal is evaluating identification accuracy on new datasets rather than configuring
  the model
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3307
tools:
- name: MarkerCount
  role: Python3 (or R via reticulate) cell-type identification toolkit whose reference-based
    function MarkerCount_Ref and marker-based function MarkerCount consume the hyper-parameters
    being configured
  repo: https://github.com/combio-dku/MarkerCount
- name: HiCAT
  role: Improved hierarchical marker-based successor to MarkerCount; shares the MarkerCount
    hyper-parameter configuration and is installed via the same pip package
  repo: https://github.com/combio-dku/HiCAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:b4cbabf18c51b3c996331e59b4686402ba63cb8abc32168704842dbbc7ff3cb0
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# hyperparameter-configuration-scRNAseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configures and justifies the hyper-parameters of the MarkerCount cell-type identification pipeline for scRNA-seq data: the marker count N=18, occurrence frequency threshold fth=0.9, marker score exponent n=2, per-cell-type penalty weight, and cluster-reassignment trigger threshold. These values were tuned empirically in the MarkerCount paper and reused by downstream annotation workflows.

## When to use

Reference for setting MarkerCount/HiCAT hyper-parameters (N, fth, score exponent, reassignment cutoff) when running reference-based or marker-based cell-type identification on scRNA-seq count matrices.

## When NOT to use

- The marker sets or thresholds are already provided (e.g. saved from a previous training run in a markers_and_thresholds file) — re-running hyper-parameter selection would be redundant
- The task is hierarchical marker-based annotation with an existing taxonomy tree; HiCAT's marker DB format (tissue/cell_type_major/cell_type_minor/cell_type_subset/exp/markers columns) governs configuration instead
- The goal is evaluating identification accuracy on new datasets rather than configuring the model

## Inputs

- Pre-annotated reference single-cell gene expression matrix (e.g. .h5ad) with cell type labels
- Cell type annotation vector for the reference dataset
- Optional existing marker lists (TSV) for marker-based mode

## Outputs

- Per-cell-type marker sets (up to N=18 markers each, overlaps allowed)
- Configured hyper-parameter values (N=18, fth=0.9, n=2, penalty weight w per cell type)
- Rejection thresholds tm on normalized marker counts derived from the configured model

## How to apply

Binarize the gene expression matrix (bij=1 if xij>0) and pre-filter candidate markers per annotated cell type using an occurrence frequency cutoff fth=0.9, a heuristic choice from the paper. Compute the marker score sm,j = (1/Cm)*sum(bij) * (1 - (1/Cbar_m)*sum(bij))^n with the exponent hyper-parameter n=2, sort scores descending per cell type, and select the top N=18 markers per type (overlaps allowed); N=18 was fixed because it showed the best performance on the paper's datasets (pancreas 2K/8K, CBMC 8K, PBMC 68K, lung, melanoma, head/neck). When cell types have uneven marker numbers, apply the per-cell-type penalty weight w = 1+3e^(-(n-1)/2)-1 based on the number of markers. During identification, if the portion of unassigned cells falls below the cluster-reassignment threshold (say 0.8, best performance observed in the range 0.1-0.9 at 0.1 steps), trigger cluster-basis reassignment; tune this value in 0.1 steps if adapting to new datasets.

## Related tools

- **MarkerCount** (Python3 (or R via reticulate) cell-type identification toolkit whose reference-based function MarkerCount_Ref and marker-based function MarkerCount consume the hyper-parameters being configured) — https://github.com/combio-dku/MarkerCount
- **HiCAT** (Improved hierarchical marker-based successor to MarkerCount; shares the MarkerCount hyper-parameter configuration and is installed via the same pip package) — https://github.com/combio-dku/HiCAT

## Examples

```
pip install MarkerCount; python -c "from MarkerCount.marker_count import MarkerCount_Ref, MarkerCount"
```

## Evaluation signals

- Marker sets contain up to 18 genes per cell type and candidate genes were pre-filtered by occurrence frequency >= fth=0.9 in target cells
- Every rejection threshold tm falls within the [0,1] range of normalized marker counts
- Reassignment is only triggered when the portion of unassigned cells is below the configured threshold (0.8 by default; validated range 0.1-0.9)
- Penalty weight per cell type equals 1+3e^(-(n-1)/2)-1 with n equal to that cell type's marker count, compensating for uneven marker numbers
- Dependency check: MarkerCount runs with Numpy, Pandas, sklearn and scipy (scanpy/plotly only for visualization)

## Limitations

- N=18 and fth=0.9 were chosen as empirical/heuristic values from the paper's datasets; they may not be optimal for other tissues or technologies
- Handling not-clearly-separable sub-populations, such as monocytes, macrophage, and dendritic cells, requires further studies
- Proper identification requires well-clustered cell populations, which is not always the case
- No standardized cell type taxonomy exists, so researchers must prepare their own marker lists for marker-based mode

## Evidence
