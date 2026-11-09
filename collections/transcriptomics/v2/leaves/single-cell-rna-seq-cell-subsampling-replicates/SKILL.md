---
name: single-cell-rna-seq-cell-subsampling-replicates
description: Use when comparing the wall-clock runtime of cell-cell interaction inference
  methods (dsCellNet, NATMI, CellPhoneDB, SingleCellSignalR) on identically sized
  scRNA-seq inputs, and the original dataset cell count differs from or exceeds the
  target benchmark size.
when_to_use_negative:
- When the analysis goal is biological inference of dynamic cell-cell interactions
  rather than runtime benchmarking — use the full un-subsampled dataset through the
  full dsCellNet pipeline (preparepairs, singlenet, developcorrelation) instead
- When the dataset already contains the target benchmark cell count or fewer cells,
  so subsampling would discard data rather than equalize inputs
- When comparing methods on accuracy or interaction sensitivity (e.g. detection of
  EC interactions or autocrine signaling) rather than computational speed
tools:
- name: dsCellNet
  role: Method under benchmark; its singlenet call is wrapped in a wall-clock timer
    and run on each seeded replicate subsample
  repo: github.com/songzh523/dsCellNet
- name: R
  role: Runtime environment for dsCellNet (R>=4.0.3), the subsampling with fixed seed,
    and the timing wrapper
- name: NATMI
  role: Comparison method timed on the identical three subsampled inputs with default
    parameters
- name: CellPhoneDB
  role: Comparison method timed on the identical three subsampled inputs with default
    parameters
- name: SingleCellSignalR
  role: Slowest comparison method (2-3 h for 10,000 cells), timed on the identical
    three subsampled inputs with default parameters
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:44faa153ede8557917af69acd1282695728658b34d6f4552fd6891a3b6359ccb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  tools:
  - CellPhoneDB
  - NATMI
  - R
  - SingleCellSignalR
  - dsCellNet
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-rna-seq-cell-subsampling-replicates

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for fairly timing cell-cell communication tools on single-cell RNA-seq data by randomly subsampling cells to a fixed count (e.g. 10,000 cells from the 11,670-cell E15.5 mouse embryonic cerebral cortex dataset GSE153164), generating independent replicate subsamples with a fixed seed, and wrapping each method's core call in a wall-clock timer.

## When to use

Use when comparing the wall-clock runtime of cell-cell interaction inference methods (dsCellNet, NATMI, CellPhoneDB, SingleCellSignalR) on identically sized scRNA-seq inputs, and the original dataset cell count differs from or exceeds the target benchmark size.

## When NOT to use

- When the analysis goal is biological inference of dynamic cell-cell interactions rather than runtime benchmarking — use the full un-subsampled dataset through the full dsCellNet pipeline (preparepairs, singlenet, developcorrelation) instead
- When the dataset already contains the target benchmark cell count or fewer cells, so subsampling would discard data rather than equalize inputs
- When comparing methods on accuracy or interaction sensitivity (e.g. detection of EC interactions or autocrine signaling) rather than computational speed

## Inputs

- GSE153164 E15.5 mouse embryonic cerebral cortex scRNA-seq expression matrix (11,670 cells) in dsCellNet datainput format
- Cell-type label file and time-point file required by dsCellNet's datainput interface
- Fixed random seed for reproducible subsampling
- Target subsample size (10,000 cells) and number of replicates (3)

## Outputs

- Runtime table with columns (method, replicate, minutes) for dsCellNet, NATMI, CellPhoneDB and SingleCellSignalR
- Wilcoxon signed-rank test result comparing per-replicate runtimes of dsCellNet vs each other method (significance at p < 0.01)

## How to apply

Load the source scRNA-seq expression matrix, cell-type labels and time-point files (here GSE153164 E15.5 mouse embryonic cerebral cortex, 11,670 cells) through the method's data input interface, then randomly subsample cells to the fixed benchmark size (10,000) with a fixed seed, generating three independent replicate subsamples for reproducibility. Run each method with default parameters on the identical replicate inputs, timing only the core inference call (e.g. dsCellNet's singlenet) to minute resolution. Compile runtimes into a table keyed by (method, replicate, minutes). Rationale: identical subsampled inputs isolate runtime differences from input-size differences, and triplicate replicates enable a non-parametric paired significance test rather than a single timed run.

## Related tools

- **dsCellNet** (Method under benchmark; its singlenet call is wrapped in a wall-clock timer and run on each seeded replicate subsample) — github.com/songzh523/dsCellNet
- **R** (Runtime environment for dsCellNet (R>=4.0.3), the subsampling with fixed seed, and the timing wrapper)
- **NATMI** (Comparison method timed on the identical three subsampled inputs with default parameters)
- **CellPhoneDB** (Comparison method timed on the identical three subsampled inputs with default parameters)
- **SingleCellSignalR** (Slowest comparison method (2-3 h for 10,000 cells), timed on the identical three subsampled inputs with default parameters)

## Examples

```
system.time(singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab))
```

## Evaluation signals

- dsCellNet runtime reproduces the reported ~12 minutes per 10,000-cell replicate
- SingleCellSignalR runtimes fall in the reported 2-3 h range for 10,000 cells, confirming the benchmark setup matches the published conditions
- Wilcoxon signed-rank test comparing per-replicate runtimes across methods yields p < 0.01
- The three replicate subsamples are bit-identical across re-runs because the seed is fixed (reproducibility check)
- Runtime table has complete rows: 4 methods x 3 replicates, each timed to minute resolution

## Limitations

- Runtime results are demonstrated on a single dataset (GSE153164 E15.5 mouse embryonic cerebral cortex); timings may not transfer to other tissues or species
- dsCellNet requires installing an R source package with pinned dependencies (igraph>=1.2.6, homologene>=1.4.68, Hmisc>=4.5.0, ggpubr>=0.4.0, pheatmap>=1.0.12, reshape2>=1.4.4, ggalluvial>=0.12.3, tseries>=0.10.48, magrittr>=2.0.1), and the dsCellNet version number is not stated
- Timing comparisons assume all methods run with default parameters; default-parameter behavior differs (e.g. SingleCellSignalR ignores EC interactions and cannot distinguish autocrine interactions), which affects runtime but not fairness of the speed comparison itself

## Evidence

- [other] Randomly subsample the cells to 10,000, generating three independent replicate subsamples with a fixed seed for reproducibility.: [evidence span withheld — anchor status: not_found]
- [results] dsCellNet 12-minute runtime finding: [evidence span withheld — anchor status: not_found]
