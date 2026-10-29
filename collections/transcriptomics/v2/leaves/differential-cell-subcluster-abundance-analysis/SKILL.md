---
name: differential-cell-subcluster-abundance-analysis
description: Use when you have pseudotime-ordered single-cell data from trajectory
  inference (e.g., Monocle3) with cluster/cell-type annotations, and need to detect
  which cell subclusters expand or deplete significantly over pseudotime rather than
  at discrete time points.
when_to_use_negative:
- When the goal is differential gene expression without testing cell subcluster abundance
  dynamics — although scTITANS co-outputs both, the subcluster test requires cell
  cluster/type annotations in cell_metadata.
- When no trajectory/pseudotime information exists and none can be inferred — the
  skill presupposes upstream Monocle3 (or equivalent TI) pseudotime; scTITANS is a
  trajectory inference-based approach.
- When subclusters are defined only at discrete sampling timepoints rather than along
  a continuous pseudotime axis — the method corrects cell asynchrony by introducing
  a time-dependent covariate along pseudotime, not by per-timepoint comparisons.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scTITANS
  role: 'primary method: bins pseudotime, fits complete vs. flat null curves of per-bin
    cell counts, computes F statistics with bootstrap null distributions and Storey
    q-values to call significant subclusters at FDR 0.01'
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: 'upstream trajectory inference: constructs the trajectory and calculates the
    pseudotime for each cell, providing the ordered axis over which subcluster abundances
    are tested'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:7fb4678c3f45144cdc8d173de6880a6270d8401cad765ce8fade872a6868145a
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# differential-cell-subcluster-abundance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify cell subclusters whose abundances vary significantly along pseudotime in time-series scRNA-seq data using scTITANS, which fits complete versus flat-line null curves of cell counts across pseudotime bins and calls significance via bootstrap F-statistics with Storey-style q-values at FDR 0.01.

## When to use

Use when you have pseudotime-ordered single-cell data from trajectory inference (e.g., Monocle3) with cluster/cell-type annotations, and need to detect which cell subclusters expand or deplete significantly over pseudotime rather than at discrete time points.

## When NOT to use

- When the goal is differential gene expression without testing cell subcluster abundance dynamics — although scTITANS co-outputs both, the subcluster test requires cell cluster/type annotations in cell_metadata.
- When no trajectory/pseudotime information exists and none can be inferred — the skill presupposes upstream Monocle3 (or equivalent TI) pseudotime; scTITANS is a trajectory inference-based approach.
- When subclusters are defined only at discrete sampling timepoints rather than along a continuous pseudotime axis — the method corrects cell asynchrony by introducing a time-dependent covariate along pseudotime, not by per-timepoint comparisons.

## Inputs

- pseudotime-ordered gene expression matrix (genes in rows, cells in columns)
- cell metadata file (row cells) containing cell cluster and/or cell type assignments from upstream trajectory inference
- gene metadata file (row genes, header must contain gene_short_name)
- root cluster name (optional; provide "none" if unknown)
- FDR level (e.g., 0.01)

## Outputs

- SigClusters.fdr0.01.txt — significant cell subclusters with q-values below the 0.01 FDR level
- SigGenes.fdr0.01.txt — significant differentially expressed genes (co-produced by the same scTITANS run)
- error message if significant cluster identification fails

## How to apply

After Monocle3 trajectory inference produces pseudotime and cell metadata (clusters/cell types), bin pseudotime and count cells of each subcluster per bin. For each subcluster, fit a complete model along pseudotime using a p-dimensional linear basis for fitted mean cell number (basis dimension chosen by singular value decomposition) and a flat-line null model, then compute an F statistic from the sum of squared residuals of both models (equations 3-4). Generate a null distribution by bootstrapping residuals from the alternative fit, adding them back to the null fit, and deriving the per-subcluster p-value from the frequency that bootstrap statistics exceed the observed statistic (equation 5); convert p-values to q-values with the Storey-style pi0 estimator (equation 6) at FDR 0.01. This is the same time-dependent covariate framework scTITANS applies to gene relative abundances, so the subcluster test typically runs alongside the DEG test in one scTITANS_full or scTITANS_partial call, writing SigClusters.fdr0.01.txt (or an error message if cluster significance fails).

## Related tools

- **scTITANS** (primary method: bins pseudotime, fits complete vs. flat null curves of per-bin cell counts, computes F statistics with bootstrap null distributions and Storey q-values to call significant subclusters at FDR 0.01) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (upstream trajectory inference: constructs the trajectory and calculates the pseudotime for each cell, providing the ordered axis over which subcluster abundances are tested)

## Examples

```
result = scTITANS_partial(data_file, cell_meta, gene_meta, "none", 0.01)
```

## Evaluation signals

- SigClusters.fdr0.01.txt exists after the run and lists each significant cluster entry with a q-value below 0.01
- Each reported cluster's complete (p-dimensional basis) model has a significantly smaller sum of squared residuals than the flat-line null model, i.e., smaller q-value = more significant
- If cluster identification fails, scTITANS emits an error message instead of an output file — absence of both the file and the error indicates the step did not run
- Bootstrap p-values are derived from residual resampling under the alternative fit and converted via Storey-style pi0 estimator at FDR 0.01 (equations 5-6); verify the q-value column reflects this transformation
- The co-produced SigGenes.fdr0.01.txt should be non-empty for the same run, confirming the shared time-dependent covariate framework executed end-to-end

## Limitations

- If something goes wrong in identifying significant clusters, scTITANS displays an error information rather than writing SigClusters.fdr0.01.txt; the failure mode must be checked explicitly.
- The skill depends on accurate upstream trajectory inference — in benchmarks only Monocle, Monocle3, GPfates, and Mpath reconstructed two branches in HSMM, and only Monocle3 reconstructed the three branches in MEF, so TI method choice directly affects subcluster calls.
- Software version numbers for scTITANS, Monocle3, and scran are not stated in the article, and no changelog is available for the scTITANS repository.
- The method requires cluster/cell-type annotations (cell metadata) to count subclusters per pseudotime bin; without annotation the subcluster test is not applicable.

## Evidence

- [methods] Separately bin pseudotime, count cells of each subcluster per bin, fit complete versus flat null curves for cell numbers along pseudotime, and repeat the same F-statistic/bootstrap/q-value procedure to call significant cell subclusters at fdr 0.01: [evidence span withheld — anchor status: not_found]
