---
name: curve-fitting-along-pseudotime
description: Use when time-series scRNA-seq cells have been ordered along pseudotime
  (e.g. by Monocle3) and the goal is to identify genes or cell subclusters whose relative
  abundance changes significantly over the trajectory rather than across discrete
  time points.
when_to_use_negative:
- Data are discrete-time-point bulk or pseudobulk comparisons without a trajectory-inferred
  pseudotime — the time-dependent covariate requires cells ordered along pseudotime
- Data have not been preprocessed (filtered/normalized) and you cannot run scTITANS
  'full' mode with indication 'none' — the statistical test assumes appropriately
  filtered and scran-normalized expression
- The scientific question is differential expression between predefined conditions/groups
  rather than temporal variation along a trajectory — standard DE methods are the
  wrong model here
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTITANS
  role: Implements the curve-fitting time-dependent covariate test (F statistic, bootstrap
    null, q-values) for genes and subclusters along pseudotime; provides scTITANS_full
    and scTITANS_partial modes
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: Upstream trajectory construction and pseudotime calculation that supplies
    the time-dependent covariate used by the curve-fitting step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:f3c16ba34e30334ca420d8e2888c7079e50cfd9a1cd5f03cc34ef147178ce38c
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
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# curve-fitting-along-pseudotime

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scTITANS' time-dependent covariate analysis: after Monocle3 pseudotime ordering, fit a curve for each gene's (or subcluster's cell-count) trajectory along pseudotime and test it against a flat-line null with an F statistic, bootstrapped p-values, and Storey-style q-values at FDR 0.01 to call time-varying genes and cell subclusters.

## When to use

Use when time-series scRNA-seq cells have been ordered along pseudotime (e.g. by Monocle3) and the goal is to identify genes or cell subclusters whose relative abundance changes significantly over the trajectory rather than across discrete time points.

## When NOT to use

- Data are discrete-time-point bulk or pseudobulk comparisons without a trajectory-inferred pseudotime — the time-dependent covariate requires cells ordered along pseudotime
- Data have not been preprocessed (filtered/normalized) and you cannot run scTITANS 'full' mode with indication 'none' — the statistical test assumes appropriately filtered and scran-normalized expression
- The scientific question is differential expression between predefined conditions/groups rather than temporal variation along a trajectory — standard DE methods are the wrong model here

## Inputs

- Gene expression matrix from time-series scRNA-seq (genes in rows, cells in columns), pseudotime-ordered via Monocle3 trajectory inference
- Cell metadata file (row cells) containing cell cluster and/or cell type annotations
- Gene metadata file (row genes, header must contain gene_short_name)

## Outputs

- SigGenes.fdr0.01.txt — significant time-varying genes with q-values below the 0.01 FDR level
- SigClusters.fdr0.01.txt — significant cell subclusters with q-values below the 0.01 FDR level (or an error message if cluster significance calling fails)

## How to apply

Load the pseudotime-ordered expression matrix (genes in rows, cells in columns) and cell metadata (clusters/cell types) from the upstream Monocle3 trajectory inference. For each gene, fit a complete model along pseudotime using a p-dimensional linear basis for the fitted mean expression (basis dimension chosen by singular value decomposition) and a flat-line null model, then compute the F statistic from the sums of squared residuals of both models (equations 3-4). Build each F statistic's null distribution by bootstrap: resample residuals from the alternative-model fit, add them back to the null-model fit, and derive per-gene p-values from how often bootstrap statistics exceed the observed statistic (equation 5). Convert p-values to q-values with the Storey-style pi0 estimator (equation 6) at an FDR level of 0.01. Separately bin pseudotime, count cells of each subcluster per bin, apply the same complete-vs-flat fit and F/bootstrap/q-value procedure to call significant subclusters at fdr 0.01, and write SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt (or an error if cluster calling fails).

## Related tools

- **scTITANS** (Implements the curve-fitting time-dependent covariate test (F statistic, bootstrap null, q-values) for genes and subclusters along pseudotime; provides scTITANS_full and scTITANS_partial modes) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (Upstream trajectory construction and pseudotime calculation that supplies the time-dependent covariate used by the curve-fitting step)

## Examples

```
result = scTITANS_partial(data_file, cell_meta, gene_meta, "none", 0.01)
```

## Evaluation signals

- Both SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt files exist and are non-empty after the run
- Every gene and cluster entry in the output files carries a q-value below the 0.01 FDR level (smaller q-value = more significant difference between complete and null models)
- Genes called significant show fitted curves along pseudotime that deviate detectably from a flat line under the F statistic computed from the residuals of the complete and null models
- Subcluster calling is verified by binned pseudotime cell counts producing fitted curves significantly different from the flat null; if significance fails, an error message is emitted instead of the cluster file
- Input matrices conform to expected layout (genes in rows, cells in columns; gene metadata header contains gene_short_name)

## Limitations

- The basis dimension for the fitted mean expression is chosen by singular value decomposition; results depend on trajectory quality — only Monocle, Monocle3, GPfates, and Mpath successfully reconstructed two branches in the HSMM benchmark, so a poor TI choice upstream degrades the test
- Cluster significance calling can fail outright, in which case an error message is returned instead of SigClusters.fdr0.01.txt
- No software version numbers for scTITANS, Monocle3, or scran are stated in the article, and no changelog is available for the repository, complicating reproducibility
- Cell annotation support (scCATCH) is limited to Human or Mouse (parameter sp must be Human or Mouse), so non-model organisms need annotation='F'

## Evidence

- [other] After trajectory inference, scTITANS fits a curve for each gene's relative abundance along pseudotime and evaluates the significance of the difference between the fitted curve and a flat line, quantifying it with a q-value.: [evidence span withheld — anchor status: not_found]
