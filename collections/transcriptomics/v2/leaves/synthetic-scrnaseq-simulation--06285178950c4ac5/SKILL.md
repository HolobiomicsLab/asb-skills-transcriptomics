---
name: synthetic-scrnaseq-simulation
description: Use when benchmarking rare-cell detection methods needs ground-truth
  controls — e.g. an artificial ~1% rare population (splatter, group.prob c(0.99,
  0.01)) or DE-gene titration on a subsampled dataset like 1%Jurkat.h5.
when_to_use_negative:
- The goal is real biological discovery of rare cell types rather than controlled
  benchmarking — use the primary scCAD pipeline on real datasets instead of simulated/substituted
  matrices
- The dataset has no reliable ground-truth rare-cell labels, making F1-based validation
  of substituted matrices impossible
- Only the abundance of rare cells (not DE gene availability/signal strength) is the
  factor under study — DE-gene substitution does not vary rare population frequency
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: splatter
  role: R package used to generate the artificial scRNA-seq dataset with a 1% rare
    group and controlled DE gene parameters
- name: scCAD
  role: Method under evaluation on the artificial and DE-substituted matrices; run
    with rare_h = 0.01 and independence score threshold I = 0.7
  repo: https://github.com/xuyp-csu/scCAD
- name: FiRE
  role: Competing rare-cell identification method run with default parameters on each
    substituted expression matrix
  repo: https://github.com/princethewinner/FiRE
- name: GapClust
  role: Competing rare-cell identification method run with default parameters on each
    substituted expression matrix
  repo: https://github.com/fabotao/GapClust
- name: GiniClust3
  role: Competing rare-cell identification method run with default parameters on each
    substituted expression matrix
  repo: https://github.com/rdong08/GiniClust3
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:26a1793fca9091adab19ff5c1e7dc80399dbc1760201e02002901c5d1f820670
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  repo_url: https://github.com/xuyp-csu/scCAD
  tools:
  - FiRE
  - GapClust
  - GiniClust
  - GiniClust3
  - scCAD
  - splatter
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# synthetic-scrnaseq-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate artificial scRNA-seq datasets with known rare-cell ground truth using splatter, and substitute pre-identified DE genes into non-DE positions to control the strength of differential signal for robustness/sensitivity testing of rare cell identification methods.

## When to use

Use when benchmarking rare-cell detection methods needs ground-truth controls — e.g. an artificial ~1% rare population (splatter, group.prob c(0.99, 0.01)) or DE-gene titration on a subsampled dataset like 1%Jurkat.h5.

## When NOT to use

- The goal is real biological discovery of rare cell types rather than controlled benchmarking — use the primary scCAD pipeline on real datasets instead of simulated/substituted matrices
- The dataset has no reliable ground-truth rare-cell labels, making F1-based validation of substituted matrices impossible
- Only the abundance of rare cells (not DE gene availability/signal strength) is the factor under study — DE-gene substitution does not vary rare population frequency

## Inputs

- splatter simulation parameters (group.prob = c(0.99, 0.01), batchCells = 2500, nGenes = 5000, de.prob = c(0.4, 0.4), de.facLoc = 0.4, de.facScale = 0.8, seed = 2023)
- Packaged 1%Jurkat.h5 subsampled dataset (1540 293T + 16 Jurkat cells, 32,738 genes, cells-in-rows/genes-in-columns matrix with X, Y, gn, cn fields)
- Pre-identified DE gene lists from two-sided Wilcoxon rank sum test (FDR 0.05; |fold-change| cutoff 1.5 artificial / 1 Jurkat)
- True rare-cell labels for F1 computation

## Outputs

- Artificial scRNA-seq count matrix with known rare population and DE/non-DE gene partition
- Per-count, per-iteration F1 scores of predicted versus true rare cells for scCAD, FiRE, GapClust, and GiniClust3
- Average F1 versus DE gene count table and plot (qualitatively reproducing Supplementary Fig. 11)

## How to apply

Generate an artificial dataset with splatter using group.prob = c(0.99, 0.01), batchCells = 2500, nGenes = 5000, de.prob = c(0.4, 0.4), de.facLoc = 0.4, de.facScale = 0.8, and seed = 2023; for real-data titration, load the packaged 1%Jurkat.h5 file (1540 293T + 16 Jurkat cells, 32,738 genes). Identify pre-identified DE genes with a two-sided Wilcoxon rank sum test at FDR 0.05 and absolute fold-change cutoff 1.5 (artificial) or 1 (Jurkat), yielding 220 DE / 3226 non-DE genes (artificial) and 108 DE / 30,479 non-DE genes (Jurkat), and remove the DE genes from the matrices. For each tested DE gene count, substitute an equal number of non-DE genes with randomly selected pre-identified DE genes, repeating the random substitution 10 times per count. Run each method (scCAD with rare_h=0.01, merge via median nearest-neighbor center distance, independence threshold I=0.7; plus FiRE, GapClust, GiniClust3) with default parameters on every substituted matrix, compute the F1 score against the true rare cells, and average across the 10 iterations per DE gene count to plot average F1 versus DE gene count.

## Related tools

- **splatter** (R package used to generate the artificial scRNA-seq dataset with a 1% rare group and controlled DE gene parameters)
- **scCAD** (Method under evaluation on the artificial and DE-substituted matrices; run with rare_h = 0.01 and independence score threshold I = 0.7) — https://github.com/xuyp-csu/scCAD
- **FiRE** (Competing rare-cell identification method run with default parameters on each substituted expression matrix) — https://github.com/princethewinner/FiRE
- **GapClust** (Competing rare-cell identification method run with default parameters on each substituted expression matrix) — https://github.com/fabotao/GapClust
- **GiniClust3** (Competing rare-cell identification method run with default parameters on each substituted expression matrix) — https://github.com/rdong08/GiniClust3

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- The Wilcoxon-based DE call must yield 220 reserved DE genes and 3226 non-DE genes for the artificial dataset, and 108 DE and 30,479 non-DE genes for the Jurkat dataset
- Each tested DE gene count must have exactly 10 random substitution replicates before F1 averaging
- All four methods should improve in average F1 as more DE genes are substituted, and scCAD should reach higher F1 than FiRE, GapClust, and GiniClust3 in weak differential expression scenarios
- On the Jurkat titration, GiniClust3 should stabilize near F1 ~ 0.6, and the curves should qualitatively match Supplementary Fig. 11
- The scCAD demo run on 1%Jurkat.h5 completes in approximately 40 seconds and recovers the 16 Jurkat cells as the rare sub-cluster

## Limitations

- All methods fail when very few DE genes are available; the artificial and substituted datasets only test DE-gene-count sensitivity, not other perturbations
- Simulation parameters (splatter seed 2023, group.prob c(0.99,0.01), de.prob c(0.4,0.4)) fix a single rare-fraction scenario; results may not generalize to other rare-cell frequencies
- The substituted DE genes are randomly re-selected each replicate, so per-run F1 scores vary and require 10-iteration averaging

## Evidence

- [supplementary] Generate the artificial scRNA-seq dataset with the splatter R package using group.prob = c(0.99, 0.01), batchCells = 2500, nGenes = 5000, de.prob = c(0.4, 0.4), de.facLoc = 0.4, de.facScale = 0.8, seed = 2023: [evidence span withheld — anchor status: not_found]
- [supplementary] DE genes identified by Wilcoxon test with FDR and fold-change cutoffs: [evidence span withheld — anchor status: not_found]
- [supplementary] substitute an equivalent number of non-DE genes with randomly selected pre-identified DE genes, repeating the random substitution 10 times per count: [evidence span withheld — anchor status: not_found]
- [discussion] Default independence score threshold for calling rare clusters: [evidence span withheld — anchor status: not_found]
- [supplementary] Robustness finding: all methods improve with more DE genes and scCAD outperforms competitors: [evidence span withheld — anchor status: not_found]
