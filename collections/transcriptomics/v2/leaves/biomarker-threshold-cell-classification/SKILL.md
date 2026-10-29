---
name: biomarker-threshold-cell-classification
description: 'Use when evaluating a marker-gene thresholding baseline for cell-type
  mapping: target cells must be assigned to reference clusters using top cluster markers
  alone, and the Classified/Ambiguous/Orphan fractions must be quantified.'
when_to_use_negative:
- When the goal is accurate cross-dataset cell-type mapping rather than benchmarking
  the baseline — the article shows this approach 'does not work effectively owing
  to the dropout problem', so use scID's full scid_multiclass pipeline instead
- When data are log-transformed — scID's normalization requires raw or library-depth
  (TPM/CPM) normalized values, not log-transformed data
- When the reference lacks well-defined cluster-specific gene signatures — scID is
  'not a recommended approach' in that case, and the top-two-marker baseline would
  be even less reliable
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scID
  role: Provides find_markers to extract cluster-specific gene signatures (logFC =
    0.6 minimum) from reference clusters that feed the thresholding baseline
  repo: https://github.com/BatadaLab/scID
- name: R
  role: Implementation environment for the baseline, data loading, thresholding, and
    plotting
- name: ggplot2
  role: Renders the assignment-fraction bar plots reproducing Figure 2C and Figure
    3C
- name: data.table
  role: Fast reading/tabulation of the normalized expression matrices and per-cell
    labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:cd3c1e012a38512fe184c7806a6a28c69ce70a7384d9ec2e0be04d9c9aacd7d3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BatadaLab/scID
  repo_url: https://github.com/BatadaLab/scID
  tools:
  - R
  - data.table
  - ggplot2
  - scID
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# biomarker-threshold-cell-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the paper's biomarker-based baseline that assigns target scRNA-seq cells to reference clusters using only the top two marker genes per cluster with per-gene expression thresholds, categorizing cells as Classified, Ambiguous, or Orphan to quantify how dropout limits simple marker-threshold classification.

## When to use

Use when evaluating a marker-gene thresholding baseline for cell-type mapping: target cells must be assigned to reference clusters using top cluster markers alone, and the Classified/Ambiguous/Orphan fractions must be quantified.

## When NOT to use

- When the goal is accurate cross-dataset cell-type mapping rather than benchmarking the baseline — the article shows this approach 'does not work effectively owing to the dropout problem', so use scID's full scid_multiclass pipeline instead
- When data are log-transformed — scID's normalization requires raw or library-depth (TPM/CPM) normalized values, not log-transformed data
- When the reference lacks well-defined cluster-specific gene signatures — scID is 'not a recommended approach' in that case, and the top-two-marker baseline would be even less reliable

## Inputs

- Raw 10X gene-expression count matrices for reference and target datasets (e.g. neuron_9k single cells and nuclei_900 single nuclei from E18 mouse brain)
- Pre-computed TPM-normalized gene expression matrices from github.com/BatadaLab/scID ExampleData folder
- Reference cluster labels for the reference cells
- Pre-computed marker file (e.g. markers.rds from ExampleData) if signatures are not re-derived

## Outputs

- Per-cell label vector with Classified / Ambiguous / Orphan assignments for each target cell
- Table of the fraction of target cells in each category (Classified, Ambiguous, Orphan)
- ggplot2 bar plots of assignment fractions reproducing Figure 2C (retinal bipolar) and Figure 3C (mouse brain)

## How to apply

Load the 10X E18 mouse brain single-cell (neuron_9k) and single-nuclei (nuclei_900) datasets, or the pre-computed TPM-normalized data from the scID GitHub ExampleData folder, and normalize by library depth (TPM/CPM), avoiding log-transformed values since scID requires non-log data. Extract cluster-specific gene signatures from the reference clusters with scID::find_markers using a minimum log-fold-change threshold of logFC = 0.6. Select the top two markers per reference cluster and apply per-gene expression thresholds on the normalized values to assign each target cell, labeling each cell Classified, Ambiguous, or Orphan (unassigned when marker evidence is absent due to dropout). Tabulate the fraction of target cells per category, save per-cell labels, and render bar plots with ggplot2 reproducing Figure 2C (retinal bipolar) and Figure 3C (mouse brain). The expected outcome is a large Orphan fraction on both datasets, demonstrating that a simple biomarker-threshold approach fails to assign most cells because of scRNA-seq dropout.

## Related tools

- **scID** (Provides find_markers to extract cluster-specific gene signatures (logFC = 0.6 minimum) from reference clusters that feed the thresholding baseline) — https://github.com/BatadaLab/scID
- **R** (Implementation environment for the baseline, data loading, thresholding, and plotting)
- **ggplot2** (Renders the assignment-fraction bar plots reproducing Figure 2C and Figure 3C)
- **data.table** (Fast reading/tabulation of the normalized expression matrices and per-cell labels)

## Examples

```
markers <- scID::find_markers(reference_gem, reference_clusters, logFC = 0.6, only.pos = TRUE); labels <- classify_by_top2_thresholds(target_gem, markers)
```

## Evaluation signals

- The reproduced bar plots must show a large Orphan fraction on both the retinal bipolar and mouse brain datasets, qualitatively matching the paper's reported result
- Only a small proportion of target cells should receive unambiguous assignments despite the target data having higher genes-per-cell depth
- Marker extraction must use logFC = 0.6 as the minimum log-fold-change and exactly the top two markers per reference cluster
- Expression values must be library-depth normalized (TPM/CPM), not log-transformed — verify scales before thresholding
- Per-cell labels must be saved and category fractions must sum to 1 across Classified + Ambiguous + Orphan

## Limitations

- The biomarker-based approach fails to assign most target cells because of the dropout problem in scRNA-seq data
- Even with higher genes-per-cell depth in the target data, only a small proportion of cells receive unambiguous assignment
- scID is not recommended if neither dataset is well clustered and contains good gene signatures
- No software version is stated for scID in the article; installation is via devtools::install_github('BatadaLab/scID') (v2.0.0 as of May 17, 2019)

## Evidence

- [results] Despite higher genes-per-cell depth in the target data, the biomarker approach using top reference-cluster markers achieved unambiguous assignment of only a small proportion of cells (Figure 2C).: [evidence span withheld — anchor status: not_found]
