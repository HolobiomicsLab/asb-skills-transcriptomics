---
name: wall-clock-runtime-profiling
description: Use when comparing computational cost (wall-clock minutes) of cell-cell
  communication inference tools on the same scRNA-seq input, e.g. justifying dsCellNet's
  efficiency on 10,000-cell developmental datasets.
when_to_use_negative:
- The analysis goal is accuracy or interaction-detection performance rather than computational
  cost — the article benchmarks accuracy separately on simulated and real data
- The scRNA-seq data are not developmental series data with time-point information,
  so dsCellNet's developmental pipeline (DTW + fuzzy clustering) does not apply
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: dsCellNet
  role: Method under test; R package installed from source, singlenet call timed on
    10,000-cell subsamples
  repo: github.com/songzh523/dsCellNet
- name: NATMI
  role: Comparison method run with default parameters on identical subsampled inputs
- name: CellPhoneDB
  role: Comparison method run with default parameters on identical subsampled inputs
- name: SingleCellSignalR
  role: Comparison method (slowest, 2-3 h for 10,000 cells) run with default parameters
- name: R
  role: Environment for installing dsCellNet and wrapping singlenet in a wall-clock
    timer (R >= 4.0.3)
  repo: github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:9b81d61c9f461ad6b868f41432be30b4b63f0fea2ac54eadbd5825b75e921d33
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# wall-clock-runtime-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the wall-clock runtime of dsCellNet against NATMI, CellPhoneDB and SingleCellSignalR on identically subsampled E15.5 mouse cortex scRNA-seq data (10,000 cells, three seeded replicates), demonstrating dsCellNet's ~12 min runtime versus SingleCellSignalR's 2-3 h.

## When to use

Use when comparing computational cost (wall-clock minutes) of cell-cell communication inference tools on the same scRNA-seq input, e.g. justifying dsCellNet's efficiency on 10,000-cell developmental datasets.

## When NOT to use

- The analysis goal is accuracy or interaction-detection performance rather than computational cost — the article benchmarks accuracy separately on simulated and real data
- The scRNA-seq data are not developmental series data with time-point information, so dsCellNet's developmental pipeline (DTW + fuzzy clustering) does not apply

## Inputs

- E15.5 mouse embryonic cerebral cortex scRNA-seq expression matrix (GSE153164, 11,670 cells)
- Cell-type label file required by dsCellNet's datainput
- Time-point file required by dsCellNet's datainput
- Three fixed-seed random subsamples of 10,000 cells shared across all four methods

## Outputs

- Runtime table with columns (method, replicate, minutes) for dsCellNet, NATMI, CellPhoneDB and SingleCellSignalR
- Wilcoxon signed-rank test p-value comparing dsCellNet runtime against the other methods

## How to apply

Load the E15.5 mouse embryonic cerebral cortex scRNA-seq data (GSE153164, 11,670 cells) into dsCellNet via its datainput interface with expression, cell-type label and time-point files. Randomly subsample cells to 10,000, generating three independent replicate subsamples with a fixed seed for reproducibility. Install dsCellNet from source (requires igraph, homologene, Hmisc, ggpubr, pheatmap, reshape2, ggalluvial, tseries, magrittr; R >= 4.0.3) and wrap the singlenet call in a wall-clock timer for each replicate. Run NATMI, CellPhoneDB and SingleCellSignalR with default parameters on the identical three subsampled inputs, timing each execution to minutes. Compile per-method, per-replicate runtimes into a table (method, replicate, minutes) and test method differences with a Wilcoxon signed-rank test at p < 0.01.

## Related tools

- **dsCellNet** (Method under test; R package installed from source, singlenet call timed on 10,000-cell subsamples) — github.com/songzh523/dsCellNet
- **NATMI** (Comparison method run with default parameters on identical subsampled inputs)
- **CellPhoneDB** (Comparison method run with default parameters on identical subsampled inputs)
- **SingleCellSignalR** (Comparison method (slowest, 2-3 h for 10,000 cells) run with default parameters)
- **R** (Environment for installing dsCellNet and wrapping singlenet in a wall-clock timer (R >= 4.0.3)) — github.com/songzh523/dsCellNet

## Examples

```
start_time <- Sys.time(); memsercpros<-preparepairs(inpexp,species,dbloc,mainlab); singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab); runtime_min <- as.numeric(difftime(Sys.time(), start_time, units="mins"))
```

## Evaluation signals

- dsCellNet runtime per replicate is approximately 12 minutes for 10,000 cells
- Wilcoxon signed-rank test comparing dsCellNet against the other methods yields p < 0.01
- All four methods receive byte-identical subsampled inputs (same three fixed-seed 10,000-cell replicates)
- SingleCellSignalR replicates take up to 2-3 h, and NATMI is the fastest among the top-performing comparison methods

## Limitations

- Runtime depends on sample size; the ~12 min figure is specific to 10,000-cell subsamples of the E15.5 cortex data
- Default-parameter runs may under-represent comparison methods' capabilities (e.g., SingleCellSignalR cannot distinguish autocrine interactions and ignored EC interactions with default parameters)
- dsCellNet's version number is not stated in the article; the README references dsCellNetv1_1.0.tar.gz

## Evidence

- [results] SingleCellSignalR takes up to 2-3 h to finish for samples with 10,000 single cells; among the top two performing methods NATMI takes the least time, but dsCellNet only requires 12 min for 10,000 single cells.: [evidence span withheld — anchor status: not_found]
