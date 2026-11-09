---
name: gaussian-mixture-rejection-threshold-fitting
description: Use when binarized scRNA-seq marker counts per cell type show a bimodal
  distribution and a defensible cutoff is needed to label clusters as 'unassigned'
  (unknown) rather than force an annotation.
when_to_use_negative:
- Cells cannot be reasonably clustered first — the article notes 'for proper identification,
  each cell populations should be well clustered, even though this is not always the
  case'
- The task is reference-based annotation with a pre-annotated training dataset and
  no marker count distributions to threshold — use MarkerCount_Ref's reference-based
  path instead
- Marker counts per cell type are clearly unimodal, so a k=2 Gaussian mixture would
  not identify a meaningful separation between target and non-target cells
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MarkerCount
  role: Marker-based identifier that consumes the fitted rejection thresholds, then
    reassigns cell types and corrects at cluster level
  repo: https://github.com/combio-dku/MarkerCount
- name: sklearn
  role: Fits the parametric Gaussian mixture (EM, k=2) on normalized marker counts
    and supports PCA/clustering used by the reassignment step
- name: SCANPY
  role: Loads the scRNA-seq datasets (e.g. GSE84133, GSE103322) as AnnData objects
    prior to binarization and marker counting
- name: Numpy
  role: Vectorized binarization and normalized marker count computation
- name: scipy
  role: Statistical support for the mixture fitting and downstream computations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:441f1be794697b54a1a387df4db3bbdfc30dc0db0783681885fcdfd78172b722
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
  - Numpy
  - SCANPY
  - scipy
  - sklearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# Gaussian mixture rejection threshold fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits a bimodal (k=2) Gaussian mixture model to per-cell-type normalized marker counts to derive a conservative rejection threshold that separates confidently assigned cells from unknowns in MarkerCount's count-based cell type identification.

## When to use

Use when binarized scRNA-seq marker counts per cell type show a bimodal distribution and a defensible cutoff is needed to label clusters as 'unassigned' (unknown) rather than force an annotation.

## When NOT to use

- Cells cannot be reasonably clustered first — the article notes 'for proper identification, each cell populations should be well clustered, even though this is not always the case'
- The task is reference-based annotation with a pre-annotated training dataset and no marker count distributions to threshold — use MarkerCount_Ref's reference-based path instead
- Marker counts per cell type are clearly unimodal, so a k=2 Gaussian mixture would not identify a meaningful separation between target and non-target cells

## Inputs

- Binarized (0/1) single-cell expression matrix from an .h5ad AnnData object (e.g. GSE84133, GSE72056)
- Normalized marker counts yi,m per cell for each curated cell type from Panglao DB / CellMarkerDB marker lists
- Per-cell-type marker counts n for computing the penalty weight w

## Outputs

- Per-cell-type rejection thresholds on normalized marker counts
- Assignment labels per cell including an 'unassigned' (unknown) class
- Performance measures (C, E, EA, CUA, EUA) when compared to manual annotations, e.g. rows in marker_size_sensitivity.csv

## How to apply

After binarizing the test dataset's expression matrix (bij=1 if xij>0) and computing normalized marker counts per cell type, fit a two-component Gaussian mixture (EM, e.g. via sklearn.mixture.GaussianMixture with k=2) to the distribution of normalized counts for each cell type and take the fitted separation as the rejection threshold. Apply the threshold conservatively so uncertain cells are first left unassigned; MarkerCount then performs cluster-basis reassignment and correction using within-cluster expression similarity to recover unnecessarily unassigned cells. When marker lists have uneven sizes across cell types, weight counts per cell type with the penalty w=1+3e-(n-1)/2-1, where n is the number of markers, before thresholding. Rejection is expected to leave many unassigned cells initially (the unassigned fraction is compared against a reassignment trigger such as 0.8), so judge correctness by the downstream C/E/EA/CUA/EUA metrics against manual annotations rather than by a low unassigned fraction alone.

## Related tools

- **MarkerCount** (Marker-based identifier that consumes the fitted rejection thresholds, then reassigns cell types and corrects at cluster level) — https://github.com/combio-dku/MarkerCount
- **sklearn** (Fits the parametric Gaussian mixture (EM, k=2) on normalized marker counts and supports PCA/clustering used by the reassignment step)
- **SCANPY** (Loads the scRNA-seq datasets (e.g. GSE84133, GSE103322) as AnnData objects prior to binarization and marker counting)
- **Numpy** (Vectorized binarization and normalized marker count computation)
- **scipy** (Statistical support for the mixture fitting and downstream computations)

## Examples

```
from MarkerCount.marker_count import MarkerCount, MarkerCount_Ref; df_res = MarkerCount(..arguments..)
```

## Evaluation signals

- The fitted mixture must separate counts into two components with target cells concentrated above the threshold and non-target cells below, matching the paper's expectation that most cell types are largely separable by normalized marker counts alone
- After cluster-basis reassignment, the unassigned portion should remain below the reassignment trigger (chosen in the 0.1–0.9 range, e.g. 0.8) without inflating erroneously assigned (EA) cells
- Performance measured with the five metrics (C, E, EA, CUA, EUA) against manual annotations should reproduce the paper's pattern that MarkerCount best handles both known and unknown populations
- When marker counts are uneven across cell types, verify the penalty w=1+3e-(n-1)/2-1 is applied per cell type before thresholding
- If marker list size n is swept, the best C/lowest E+EUA+EA should occur around n=18, the value fixed for most datasets

## Limitations

- For certain cell types (beta, delta, epsilon, gamma cells in pancreas, dendritic cells in blood, B cells in tumors, T cells in lung), identification via normalized marker counts alone yields considerable errors even with thresholding
- Proper identification requires well-clustered cell populations, which is not always the case
- Handling not-clearly-separable sub-populations such as monocytes, macrophages, and dendritic cells requires further study
- Cell type taxonomy is not standardized, so threshold quality depends on the user-curated marker lists supplied

## Evidence

- [intro] We set a conservative rejection threshold in the normalized marker counts, which potentially yields many "unassigned" cells: [evidence span withheld — anchor status: not_found]
