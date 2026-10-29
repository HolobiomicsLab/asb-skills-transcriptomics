---
name: count-data-log-transformation
description: Use when raw simulated count matrices must be transformed before correlation-
  or clustering-based validation, e.g. prior to Pearson/Kendall Tb regulator-gene
  correlations, Spearman-distance k-medoids clustering, or PCA in MOSim workflows.
when_to_use_negative:
- The data are already log-transformed or z-scaled (double transformation would distort
  the correlations used for validation)
- Downstream analysis requires raw integer counts (e.g. count-based simulation input
  such as SPARSim or count-aware DE models) rather than transformed profiles
- The task is simulation itself (running mosim()/sc_mosim()) rather than post-simulation
  validation of simulated profiles
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MOSim
  role: Source of the simulated bulk and single-cell count matrices that undergo the
    log(x+1) transformation before validation
  repo: github.com/ConesaLab/MOSim
- name: SPARSim
  role: Generates the scRNA-seq counts underlying sc_mosim output that are aggregated
    per cell type and log-transformed
- name: R
  role: Implementation language for the log(x+1) transformation and downstream Pearson/Kendall
    Tb/Spearman analyses
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:dd5dd1d31ab847e2931ee1ec510ee198d1fc03ac2d721e0876d93ed54610f1ac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  - SPARSim
  - Seurat
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# count-data-log-transformation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the log(x+1) transformation to raw simulated omics count matrices (bulk RNA-seq/ATAC-seq from mosim(), scRNA-seq/scATAC-seq from sc_mosim()) so that downstream Pearson, Kendall Tb, and Spearman-distance analyses on MOSim validation outputs are computed on a variance-stabilized scale.

## When to use

Use when raw simulated count matrices must be transformed before correlation- or clustering-based validation, e.g. prior to Pearson/Kendall Tb regulator-gene correlations, Spearman-distance k-medoids clustering, or PCA in MOSim workflows.

## When NOT to use

- The data are already log-transformed or z-scaled (double transformation would distort the correlations used for validation)
- Downstream analysis requires raw integer counts (e.g. count-based simulation input such as SPARSim or count-aware DE models) rather than transformed profiles
- The task is simulation itself (running mosim()/sc_mosim()) rather than post-simulation validation of simulated profiles

## Inputs

- Simulated bulk count matrices from mosim() (RNA-seq, ATAC-seq, miRNA-seq layers)
- Simulated single-cell count matrices from sc_mosim() (scRNA-seq, scATAC-seq)
- Per-cell-type aggregated average count matrix (single-cell validation)
- Ground-truth regulator-gene interaction list from the MOSim object

## Outputs

- log(x+1)-transformed count matrix for correlation and distance computations
- Row-wise z-scaled log-transformed per-group mean profiles for heatmap rendering
- Transformed feature profiles feeding regulator_correlation_validation.csv and kendall_validation_report.csv

## How to apply

Load the simulated count matrices from the MOSim or sc_mosim output object and, for single-cell data, first aggregate the matrix to the average count per cell type. Apply the log(x+1) transformation to the (aggregated) counts exactly as specified in the MOSim methods, so zero counts map to zero and large counts are compressed for downstream comparisons. Use the transformed profiles as inputs to Pearson's correlation for bulk data, Kendall's Tb correlation for single-cell data, and Spearman's distance (1 − Spearman's correlation) with k-medoids (k=10) clustering of across-cell-type profiles. For heatmaps, follow log(x+1) with row-wise z-score scaling of per-group mean expression. Rationale: count-scale dispersions distort correlation and distance metrics; the article's validation thresholds (e.g. 99.4% of modelled bulk interactions with |correlation| > 0.9 and 79.5% of single-cell interactions with Kendall Tb > 0.7) were established on log(x+1)-transformed data, so applying it is required to reproduce them.

## Related tools

- **MOSim** (Source of the simulated bulk and single-cell count matrices that undergo the log(x+1) transformation before validation) — github.com/ConesaLab/MOSim
- **SPARSim** (Generates the scRNA-seq counts underlying sc_mosim output that are aggregated per cell type and log-transformed)
- **R** (Implementation language for the log(x+1) transformation and downstream Pearson/Kendall Tb/Spearman analyses)

## Examples

```
log_counts <- log(simulated_counts + 1)
```

## Evaluation signals

- Zero counts map to exactly 0 after log(x+1); transformation is monotonic with no negative values
- Bulk Pearson validation on transformed data reproduces ~99.4% of modelled activator/repressor interactions with |correlation| > 0.9 and 0% of no-effect interactions above threshold
- Single-cell Kendall Tb validation on transformed aggregated data reproduces ~79.5% of modelled interactions above 0.7 and a no-effect centre near 0.32
- K-medoids clustering (Spearman distance, k=10) of transformed across-cell-type profiles shows ~3.3% incorrect cluster assignments, confirmed by PCA alignment with simulation settings

## Limitations

- Single-cell simulation is restricted to scRNA-seq and scATAC-seq only, so the transform is only exercised on those single-cell layers
- MOSim output is intended for regulatory benchmarking; the transform supports correlation/clustering validation but not spatial or proteomics/metabolomics layers, which MOSim does not simulate
- The article does not report a changelog for the source repository, so parameter defaults for sc_mosim-derived counts may differ across developer-version installs

## Evidence

- [methods] log transformation applied to data before validation: [evidence span withheld — anchor status: not_found]
