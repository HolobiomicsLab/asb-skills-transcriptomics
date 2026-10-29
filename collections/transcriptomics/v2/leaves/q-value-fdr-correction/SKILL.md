---
name: q-value-fdr-correction
description: Use when converting bootstrap-derived p-values from pseudotime-based
  curve fitting (complete vs. flat-line null models) into q-values to call time-dependent
  DEGs and cell subclusters at a chosen fdr_level (0.01 in the paper).
when_to_use_negative:
- When the analysis goal is static (single-time-point) differential expression, e.g.
  simple two-condition DE tests that do not involve pseudotime curve fitting or a
  bootstrap null distribution
- When trajectory inference has not been performed — scTITANS' q-value step depends
  on Monocle3-derived pseudotime for genes and cells
- When a raw matrix with no trajectory results is available and no cell annotation
  is needed only for upstream preprocessing — use the full mode's filtering/normalization
  instead, not the q-value correction in isolation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scTITANS
  role: Implements the curve fitting, bootstrap null, and Storey-style q-value computation
    at the user-specified fdr_level; writes SigGenes and SigClusters outputs
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: Upstream supplier of pseudotime used as the time-dependent covariate for the
    F-statistic models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:842f83f3a5279df2bcbee67ce8daac5540bafe49f8bebf4c418af5c64e2074fe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scTITANS
  repo_url: https://github.com/ZJUFanLab/scTITANS
  tools:
  - Monocle3
  - scTITANS
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# q-value-fdr-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scTITANS' multiple-testing correction step for time-series scRNA-seq: per-gene/per-cluster p-values from an F-statistic bootstrap null are converted to Storey-style q-values (with pi0) and filtered at an FDR level of 0.01 to call significant genes and cell subclusters.

## When to use

Use when converting bootstrap-derived p-values from pseudotime-based curve fitting (complete vs. flat-line null models) into q-values to call time-dependent DEGs and cell subclusters at a chosen fdr_level (0.01 in the paper).

## When NOT to use

- When the analysis goal is static (single-time-point) differential expression, e.g. simple two-condition DE tests that do not involve pseudotime curve fitting or a bootstrap null distribution
- When trajectory inference has not been performed — scTITANS' q-value step depends on Monocle3-derived pseudotime for genes and cells
- When a raw matrix with no trajectory results is available and no cell annotation is needed only for upstream preprocessing — use the full mode's filtering/normalization instead, not the q-value correction in isolation

## Inputs

- Pseudotime-ordered gene expression matrix (genes in rows, cells in columns) from upstream Monocle3 trajectory inference
- Cell metadata file (row cells) containing cell clusters and/or cell types
- Gene metadata file (row genes, header must contain gene_short_name)
- fdr_level parameter (0.01 in the paper)

## Outputs

- SigGenes.fdr0.01.txt — significant genes with q-values below the 0.01 fdr level
- SigClusters.fdr0.01.txt — significant cell subclusters with q-values below the 0.01 fdr level
- Error message instead of SigClusters file when cluster significance calling fails

## How to apply

After Monocle3 trajectory inference, fit each gene's relative-abundance curve along pseudotime with a p-dimensional linear basis (basis dimension chosen by SVD) against a flat-line null, and compute the F statistic from the residual sums of squares (equations 3-4). Build the null distribution by bootstrapping residuals from the alternative fit, adding them back to the null fit, and deriving each p-value from the frequency that bootstrap F statistics exceed the observed statistic (equation 5). Convert p-values to q-values with a Storey-style estimator that estimates the pi0 proportion of non-differentially expressed genes (equation 6), and retain entries with q-value below the fdr_level (0.01). Apply the identical F/bootstrap/q-value procedure to cell counts per subcluster binned along pseudotime to call significant clusters. Write surviving genes and clusters to SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt; emit an error message if cluster significance calling fails.

## Related tools

- **scTITANS** (Implements the curve fitting, bootstrap null, and Storey-style q-value computation at the user-specified fdr_level; writes SigGenes and SigClusters outputs) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (Upstream supplier of pseudotime used as the time-dependent covariate for the F-statistic models)

## Examples

```
result = scTITANS_partial(data_file,cell_meta,gene_meta,"none",0.01)
```

## Evaluation signals

- Both SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt files exist after the run
- Every gene/cluster entry in the output files carries a q-value below the 0.01 fdr level (smaller q-value = more significant)
- q-values are monotonically consistent with p-values (q >= p per gene), as expected from Storey-style pi0 estimation
- If cluster significance calling fails, an explicit error message is emitted rather than an empty/invalid clusters file
- Changing fdr_level (e.g. from 0.01) proportionally changes which genes/clusters are retained in the output txt files

## Limitations

- Requires completed trajectory inference — in 'partial' mode the user must supply pseudotime-ordered data, cell_metadata, and gene_metadata; in 'full' mode scTITANS performs TI itself
- No software version numbers for scTITANS, Monocle3, or scran are stated in the article, and the repository has no changelog
- Cluster significance calling can fail, in which case only an error message (not SigClusters.fdr0.01.txt) is produced

## Evidence
