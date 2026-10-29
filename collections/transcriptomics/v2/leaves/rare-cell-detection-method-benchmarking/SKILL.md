---
name: rare-cell-detection-method-benchmarking
description: Use when you need to quantify and compare rare cell detection accuracy
  of scCAD against FiRE, GapClust, and GiniClust3, e.g. reproducing the DE-gene robustness/sensitivity
  analysis (Supplementary Fig. 11) on artificial or 1%Jurkat.h5 data.
when_to_use_negative:
- The input is not a cell-by-gene scRNA-seq count matrix with known rare cell ground
  truth — the F1 benchmark requires labeled rare cells
- The goal is detection of rare cells in a new dataset rather than comparing/benchmarking
  method performance
- The rare cell population is not defined by differential expression signals — all
  methods failed when too few DE genes were available
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCAD
  role: 'Primary method under test: cluster decomposition-based anomaly detection
    with rare_h=0.01 and independence score threshold I=0.7'
  repo: github.com/xuyp-csu/scCAD
- name: FiRE
  role: Competing rare cell identification method run with default parameters on each
    substituted expression matrix
  repo: github.com/princethewinner/FiRE
- name: GapClust
  role: Competing rare cell identification method run with default parameters on each
    substituted expression matrix
  repo: github.com/fabotao/GapClust
- name: GiniClust3
  role: Competing rare cell identification method run with default parameters; stabilizes
    near F1 ~ 0.6 on Jurkat in the DE-gene sensitivity analysis
  repo: github.com/rdong08/GiniClust3
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:9fbdc8303463ae8c26e1be34a6fe8b78e4ac0fe4e972cd34df485cac32587fb0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  tools:
  - FiRE
  - GapClust
  - GiniClust
  - GiniClust3
  - scCAD
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rare-cell-detection-method-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark rare cell identification methods (scCAD vs FiRE, GapClust, GiniClust3) by running each on controlled scRNA-seq inputs — including artificial splatter data and the packaged 1%Jurkat.h5 subsample — and comparing F1 scores of predicted versus true rare cells, e.g. across varying numbers of substituted DE genes.

## When to use

Use when you need to quantify and compare rare cell detection accuracy of scCAD against FiRE, GapClust, and GiniClust3, e.g. reproducing the DE-gene robustness/sensitivity analysis (Supplementary Fig. 11) on artificial or 1%Jurkat.h5 data.

## When NOT to use

- The input is not a cell-by-gene scRNA-seq count matrix with known rare cell ground truth — the F1 benchmark requires labeled rare cells
- The goal is detection of rare cells in a new dataset rather than comparing/benchmarking method performance
- The rare cell population is not defined by differential expression signals — all methods failed when too few DE genes were available

## Inputs

- Artificial scRNA-seq expression matrix generated with the splatter R package (2500 cells, 5000 genes, 1% rare group)
- Packaged 1%Jurkat.h5 subsampled dataset (1540 293T + 16 Jurkat cells, 32,738 genes) from the scCAD repository
- Lists of pre-identified DE genes and non-DE genes (Wilcoxon rank sum test, FDR 0.05; |FC| > 1.5 artificial / > 1 Jurkat)
- Ground-truth rare cell labels for F1 score computation

## Outputs

- Average F1 score per method per DE gene count (averaged over 10 random substitution iterations)
- Score table of F1 scores for scCAD, FiRE, GapClust, and GiniClust3
- Line plot of average F1 score versus number of DE genes, reproducing Supplementary Fig. 11

## How to apply

Generate the artificial scRNA-seq dataset with splatter (group.prob = c(0.99, 0.01), batchCells = 2500, nGenes = 5000, de.prob = c(0.4, 0.4), de.facLoc = 0.4, de.facScale = 0.8, seed = 2023) and load the packaged 1%Jurkat.h5 file (1540 293T + 16 Jurkat cells). Identify pre-identified DE genes via a two-sided Wilcoxon rank sum test (FDR < 0.05; |fold-change| > 1.5 artificial, > 1 Jurkat), remove them, then for each tested DE gene count substitute an equal number of non-DE genes with random pre-identified DE genes, repeating 10 times per count. Run scCAD with defaults (rare_h = 0.01, merge threshold via median nearest-neighbor center distance, independence score threshold I = 0.7) plus FiRE, GapClust, and GiniClust3 with their defaults on each substituted matrix. Compute F1 scores of predicted versus true rare cells, average across the 10 iterations per count, and plot average F1 versus DE gene count. Judge success by whether the curves reproduce Supplementary Fig. 11: all methods improve with more DE genes and scCAD reaches higher F1 than FiRE, GapClust, and GiniClust3 (GiniClust3 stabilizing near F1 ~ 0.6 on Jurkat).

## Related tools

- **scCAD** (Primary method under test: cluster decomposition-based anomaly detection with rare_h=0.01 and independence score threshold I=0.7) — github.com/xuyp-csu/scCAD
- **FiRE** (Competing rare cell identification method run with default parameters on each substituted expression matrix) — github.com/princethewinner/FiRE
- **GapClust** (Competing rare cell identification method run with default parameters on each substituted expression matrix) — github.com/fabotao/GapClust
- **GiniClust3** (Competing rare cell identification method run with default parameters; stabilizes near F1 ~ 0.6 on Jurkat in the DE-gene sensitivity analysis) — github.com/rdong08/GiniClust3

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- Per-count average F1 curves qualitatively reproduce Supplementary Fig. 11: all methods improve as more DE genes are added
- scCAD attains higher average F1 than FiRE, GapClust, and GiniClust3 in weak differential expression scenarios
- GiniClust3's average F1 stabilizes near ~ 0.6 on the 1% Jurkat dataset
- Each DE gene count is evaluated over exactly 10 random substitution iterations before averaging
- scCAD's identified rare clusters overlap the known 16 Jurkat cells (verifiable via the returned result and labels counter)

## Limitations

- All methods, including scCAD, failed when very few DE genes were present; performance depends on the number of available DE genes
- Exact version numbers for the comparison packages (FiRE, GapClust, GiniClust3, etc.) are not stated in the article section
- scCAD requires a standard computer with enough RAM for in-memory operations; supported on Linux (Ubuntu 16.04.7) and macOS (Monterey 12.6.3) with Python 3.7.13

## Evidence

- [discussion] scCAD default independence threshold: [evidence span withheld — anchor status: not_found]
- [discussion] Iterative cluster decomposition threshold: [evidence span withheld — anchor status: not_found]
