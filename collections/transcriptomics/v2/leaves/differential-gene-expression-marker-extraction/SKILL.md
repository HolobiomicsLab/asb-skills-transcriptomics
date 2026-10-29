---
name: differential-gene-expression-marker-extraction
description: Use when a reference scRNA-seq dataset with known cluster labels must
  yield per-cluster marker gene sets to score or assign target cells, e.g. before
  scid_multiclass mapping or a top-N-marker thresholding baseline.
when_to_use_negative:
- Input is log-transformed expression data — scID explicitly requires library-depth
  normalized (TPM/CPM) values, not log-transformed data
- Neither reference nor target data is well clustered or contains good gene signatures
  — the paper states scID (and hence marker-based identification) is not a recommended
  approach in that case
- The goal is batch correction/integration of the full datasets rather than marker-based
  cell-type identification, since the paper shows the biomarker approach fails for
  most cells due to scRNA-seq dropout
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scID
  role: Provides find_markers to extract cluster-specific gene signatures and scid_multiclass
    to consume them for assigning target cells
  repo: https://github.com/BatadaLab/scID
- name: R
  role: Implementation environment for the scID package and the marker-thresholding
    baseline
- name: ggplot2
  role: Renders the assignment-fraction bar plots (Figure 2C / Figure 3C reproductions)
- name: Seurat
  role: Listed as an Imports dependency of the scID R package
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:87d6c5b2c570185f04cfcd1f7d019c116c01548c416a10f491c25f67896e578c
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
  - Seurat
  - data.table
  - ggplot2
  - scID
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# differential-gene-expression-marker-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts cluster-specific gene signatures from a reference scRNA-seq dataset using scID's find_markers, which selects genes differentially expressed in each cluster above a minimum log-fold-change threshold (e.g. logFC = 0.6). These signatures support downstream cell-type mapping and the biomarker-threshold baseline that classifies, marks ambiguous, or orphans target cells.

## When to use

Use when a reference scRNA-seq dataset with known cluster labels must yield per-cluster marker gene sets to score or assign target cells, e.g. before scid_multiclass mapping or a top-N-marker thresholding baseline.

## When NOT to use

- Input is log-transformed expression data — scID explicitly requires library-depth normalized (TPM/CPM) values, not log-transformed data
- Neither reference nor target data is well clustered or contains good gene signatures — the paper states scID (and hence marker-based identification) is not a recommended approach in that case
- The goal is batch correction/integration of the full datasets rather than marker-based cell-type identification, since the paper shows the biomarker approach fails for most cells due to scRNA-seq dropout

## Inputs

- Reference gene-expression matrix (genes x cells), library-depth (TPM/CPM) normalized, non-log-transformed, e.g. 10X neuron_9k or nuclei_900 data or pre-computed TPM data from the scID ExampleData folder
- Named list/vector of reference cluster labels for the reference cells
- logFC threshold for cluster-specific gene selection (paper/tutorial value: 0.6)
- only.pos flag controlling whether only positive markers are extracted

## Outputs

- Data frame of signature genes per reference cluster (equivalent to scID_output$markers; ExampleData provides markers.rds as a reference format)
- Per-cell target labels (Classified / Ambiguous / Orphan) when used in the biomarker-threshold baseline
- Fractions of target cells in each category for reproducing Figure 2C / Figure 3C bar plots
- Optional: gene weights per cluster signature and per-cell scores if the full scID pipeline is run

## How to apply

Load the reference gene-expression matrix (e.g. 10X E18 mouse brain neuron_9k or retinal bipolar data) with cell names as columns and genes as rows, and a list of reference cluster labels per cell. Normalize by library depth (TPM or CPM) via counts_to_cpm; scID requires depth-normalized, not log-transformed, values. Call scID::find_markers(reference_gem, reference_clusters, logFC, only.pos, normalize_reference) with logFC = 0.6 to select genes differentially expressed in each reference cluster; for the biomarker baseline, take the top two markers per cluster and apply per-gene thresholds on normalized expression to label each target cell as Classified, Ambiguous, or Orphan. Alternatively, pass the marker table directly into scid_multiclass, which estimates discriminative Fisher-LDA-like weights (optionally from the target when estimate_weights_from_target is TRUE) and scores cells. Output the markers data frame (and weights/scores if running the full scID pipeline) plus per-cell labels for tabulating assignment fractions.

## Related tools

- **scID** (Provides find_markers to extract cluster-specific gene signatures and scid_multiclass to consume them for assigning target cells) — https://github.com/BatadaLab/scID
- **R** (Implementation environment for the scID package and the marker-thresholding baseline)
- **ggplot2** (Renders the assignment-fraction bar plots (Figure 2C / Figure 3C reproductions))
- **Seurat** (Listed as an Imports dependency of the scID R package)

## Examples

```
markers <- scID::find_markers(reference_gem = reference_gem, reference_clusters = reference_clusters, logFC = 0.6, only.pos = TRUE, normalize_reference = TRUE)
```

## Evaluation signals

- Extracted markers must be genes with log-fold-change >= the chosen threshold (logFC = 0.6) within each reference cluster
- In the biomarker-threshold baseline on retinal bipolar and mouse brain (neuron_9k/nuclei_900) data, the Orphan fraction must be large, qualitatively matching Figure 2C/3C where only a small proportion of cells are unambiguously classified
- Output marker table schema must match scID_output$markers (cluster-to-signature-gene mapping), comparable to the ExampleData markers.rds format
- Expression values fed to find_markers must be TPM/CPM (column) normalized and not log-transformed
- Per-cell labels must cover every target cell with exactly one of the three categories: Classified, Ambiguous, or Orphan

## Limitations

- The biomarker approach using top reference-cluster markers achieves unambiguous assignment of only a small proportion of cells because of the dropout problem in scRNA-seq data, even when the target data has higher genes-per-cell depth
- Distance/naive marker metrics in high-dimensional space may not work as expected for reference-based mapping (Aggarwal et al., 2001), motivating scID's weighted approach
- scID v2.0.0 (released May 17, 2019) uses negative markers together with positive markers; older marker sets may lack negative markers, reducing discrimination among very similar cell types
- No software version for scID is stated in the discussion section; imputation for shallow target data is deferred to future work and not implemented

## Evidence

- [readme] scID v2.0 negative markers: [evidence span withheld — anchor status: external_doc]
