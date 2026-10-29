---
name: pseudotime-based-differential-expression-testing
description: Use when time-series scRNA-seq data already have (or need) Monocle3 pseudotime
  and the goal is to identify genes or cell subclusters whose abundance changes significantly
  along pseudotime.
when_to_use_negative:
- The data are not time-series scRNA-seq and no temporal/pseudotime ordering of cells
  is relevant to the biological question.
- Only static differential expression between labeled conditions is needed — standard
  DE tools apply, since scTITANS tests variation along pseudotime rather than between
  groups.
- 'Trajectory reconstruction itself is not feasible: the paper shows Monocle3 failed
  on some topologies (e.g. only Mpath reconstructed the eight CMP branches), so pseudotime
  inputs may be unreliable.'
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTITANS
  role: 'Core method: fits complete vs. flat-null curves for gene expression and subcluster
    cell counts along pseudotime and calls significance via bootstrap q-values'
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: 'Upstream trajectory inference: constructs the trajectory graph and calculates
    pseudotime for each cell used as the time-dependent covariate'
- name: scran
  role: Normalization of the expression matrix (pool and deconvolute algorithm) before
    downstream analysis in full mode
- name: scCATCH
  role: Optional cell type annotation when running scTITANS_full with annotation=T
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:059ec6649ff038b143766158c1061341f0bbfd25621d016ccfd1c515f337bbfa
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
  - scCATCH
  - scTITANS
  - scran
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pseudotime-based-differential-expression-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scTITANS converts Monocle3-derived pseudotime into a time-dependent covariate statistical test, fitting per-gene and per-subcluster curves along pseudotime and calling differential genes/subclusters via F-statistics with bootstrap q-values at FDR 0.01.

## When to use

Use when time-series scRNA-seq data already have (or need) Monocle3 pseudotime and the goal is to identify genes or cell subclusters whose abundance changes significantly along pseudotime.

## When NOT to use

- The data are not time-series scRNA-seq and no temporal/pseudotime ordering of cells is relevant to the biological question.
- Only static differential expression between labeled conditions is needed — standard DE tools apply, since scTITANS tests variation along pseudotime rather than between groups.
- Trajectory reconstruction itself is not feasible: the paper shows Monocle3 failed on some topologies (e.g. only Mpath reconstructed the eight CMP branches), so pseudotime inputs may be unreliable.

## Inputs

- gene expression matrix file (rows = genes, columns = cells), optionally raw/filtered/normalized
- cell metadata file (rows = cells; clusters and/or cell types required for partial mode)
- gene metadata file (rows = genes; header must contain gene_short_name)
- indication flag of matrix status: 'none' (raw), 'filtered', or 'normalized'
- fdr level (0.01 in the paper)
- optional root cluster name and annotation options (annotation T/F, species Human/Mouse, tissue)

## Outputs

- SigGenes.fdr0.01.txt — significant genes with q-values below the 0.01 FDR level
- SigClusters.fdr0.01.txt — significant cell subclusters with q-values below the 0.01 FDR level (error message emitted if cluster calling fails)

## How to apply

Run scTITANS in 'full' mode from a raw gene-expression matrix (genes in rows, cells in columns) with 'indication' set to none/filtered/normalized, or in 'partial' mode when Monocle3 trajectory inference is already done, supplying cell_metadata (row cells, with clusters/cell types) and gene_metadata (header must contain gene_short_name). scTITANS uses Monocle3 to construct the trajectory and compute pseudotime, then for each gene fits a complete model along pseudotime using a p-dimensional linear basis (basis dimension chosen by singular value decomposition) and a flat-line null model, computing an F statistic from the sum of squared residuals (equations 3-4). Per-gene p-values come from a bootstrap null distribution: residuals under the alternative fit are resampled and added back to the null fit (equation 5), then converted to q-values with a Storey-style pi0 estimator (equation 6). Separately, bin pseudotime, count cells per subcluster per bin, and repeat the same complete-vs-flat model F-statistic/bootstrap/q-value procedure to call significant subclusters. Use fdr_level 0.01 (as in the paper's calls); outputs are the two text files SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt.

## Related tools

- **scTITANS** (Core method: fits complete vs. flat-null curves for gene expression and subcluster cell counts along pseudotime and calls significance via bootstrap q-values) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (Upstream trajectory inference: constructs the trajectory graph and calculates pseudotime for each cell used as the time-dependent covariate)
- **scran** (Normalization of the expression matrix (pool and deconvolute algorithm) before downstream analysis in full mode)
- **scCATCH** (Optional cell type annotation when running scTITANS_full with annotation=T)

## Examples

```
result = scTITANS_partial(data_file,cell_meta,gene_meta,"none",0.01)
```

## Evaluation signals

- Both SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt exist after the run (or a documented error message appears if cluster calling fails).
- Every gene/cluster entry in the output files carries a q-value below the 0.01 fdr level (smaller q-value = more significant).
- Per-gene F statistics derive from comparing sums of squared residuals of the complete p-dimensional basis fit against the flat-line null model (equations 3-4).
- P-values derive from the bootstrap frequency that null statistics exceed the observed statistic (equation 5), then Storey-style q-values with pi0 (equation 6).
- In partial mode, cell_metadata rows align with matrix columns and gene_metadata header contains gene_short_name.

## Limitations

- Accuracy of the calls depends on the upstream trajectory: only Monocle, Monocle3, GPfates, and Mpath reconstructed the two HSMM branches, and only Monocle3 reconstructed the three MEF branches, while only Mpath handled the eight CMP branches.
- If cluster significance calling fails, scTITANS returns an error message instead of SigClusters.fdr0.01.txt.
- No software version numbers for Monocle3, scran, or scTITANS are stated in the article, and no changelog is available for the repository.

## Evidence

- [full_text] After trajectory inference, scTITANS fits a curve for each gene's relative abundance along pseudotime.: [evidence span withheld — anchor status: not_found]
