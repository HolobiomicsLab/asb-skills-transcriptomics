---
name: single-cell-rna-seq-data-simulation
description: Use when you must benchmark cross-timepoint cell annotation methods under
  controlled scenarios (shifted compositions, disappearing or emerging cell types)
  with ground-truth labels and Monte Carlo replicates.
when_to_use_negative:
- Input is a single-timepoint (cross-sectional) scRNA-seq dataset with no t0/t1 structure
  to simulate or borrow information across
- The goal is real-data evaluation without ground truth — CASi's own limitation is
  that a dramatically different later timepoint (or a transcriptomically similar novel
  cell) breaks the neural network/novel-cell detection
- You only need novel-cell-type detection or tDEG identification on real data rather
  than a controlled annotation benchmark
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: CASi
  role: R package providing CreateCASiList and the Keras/TensorFlow CrossTimeAnnotation
    classifier used to annotate simulated t1 cells and produce pbmc_demo-style inputs
  repo: github.com/yizhuo-wang/CASi
- name: keras
  role: TensorFlow backend required before running CASi's neural network cross-timepoint
    annotation on simulated data
- name: scmap
  role: Competing supervised annotation method run on the identical simulated t0/t1
    splits as a baseline
- name: CHETAH
  role: Competing supervised annotation method run on the identical simulated t0/t1
    splits as a baseline
- name: scPred
  role: Competing supervised annotation method run on the identical simulated t0/t1
    splits as a baseline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:91809cdc75751f26134cdac5615a35d87f924acfb9cd83b7e52611f36804f218
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - CHETAH
  - GENIE3
  - PIDC
  - R (>=)
  - keras
  - sc-compReg
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-rna-seq-data-simulation

## Summary

Generates simulated longitudinal (t0/t1) single-cell RNA-seq datasets from a real PBMC pool of sorted cells to benchmark cross-timepoint cell annotation methods (e.g., CASi vs. scmap, CHETAH, scPred) by accuracy and adjusted Rand index (ARI).

## When to use

Use when you must benchmark cross-timepoint cell annotation methods under controlled scenarios (shifted compositions, disappearing or emerging cell types) with ground-truth labels and Monte Carlo replicates.

## When NOT to use

- Input is a single-timepoint (cross-sectional) scRNA-seq dataset with no t0/t1 structure to simulate or borrow information across
- The goal is real-data evaluation without ground truth — CASi's own limitation is that a dramatically different later timepoint (or a transcriptomically similar novel cell) breaks the neural network/novel-cell detection
- You only need novel-cell-type detection or tDEG identification on real data rather than a controlled annotation benchmark

## Inputs

- Public PBMC scRNA-seq count matrix (peripheral blood mononuclear cells, >60,000 sorted cells from eight immune cell types)
- Randomly drawn cell pool from five of the eight PBMC cell types (as used for pbmc_demo)
- Scenario specification: shifted compositions, disappearing cell type, or novel emerging cell type
- Ground-truth t1 cell labels per replicate for accuracy/ARI computation

## Outputs

- Simulated paired t0/t1 gene expression count matrices with t0 labels per replicate
- Per-replicate per-method annotation accuracy across the three scenarios
- Adjusted Rand index (ARI) comparisons per scenario (cf. supplementary Figure 11)
- Aggregated benchmark table (e.g., annotation_benchmark_results.csv)

## How to apply

Start from a public PBMC dataset of more than 60,000 sorted cells from eight immune cell types and randomly extract cells from five cell types as the simulation pool (as done for the pbmc_demo data shipped with CASi). Generate simulated t0/t1 pairs under three scenarios: (1) same cell types with shifted compositions, (2) a disappearing cell type, and (3) a novel emerging cell type. For each replicate, build a CASiList with CreateCASiList (t0 counts, t1 counts, t0 labels) and train the Keras/TensorFlow neural network via CrossTimeAnnotation to annotate t1 cells; run competing supervised methods (scmap, CHETAH, scPred) on identical splits. Repeat for 200 Monte Carlo repetitions, record per-cell accuracy and compute ARI per scenario (ARI results appear in supplementary Figure 11), then aggregate into a results table such as annotation_benchmark_results.csv.

## Related tools

- **CASi** (R package providing CreateCASiList and the Keras/TensorFlow CrossTimeAnnotation classifier used to annotate simulated t1 cells and produce pbmc_demo-style inputs) — github.com/yizhuo-wang/CASi
- **keras** (TensorFlow backend required before running CASi's neural network cross-timepoint annotation on simulated data)
- **scmap** (Competing supervised annotation method run on the identical simulated t0/t1 splits as a baseline)
- **CHETAH** (Competing supervised annotation method run on the identical simulated t0/t1 splits as a baseline)
- **scPred** (Competing supervised annotation method run on the identical simulated t0/t1 splits as a baseline)

## Examples

```
require(keras); require(CASi); casilist <- CreateCASiList(t0_counts, t1_counts, t0_labels); annot <- CrossTimeAnnotation(casilist)
```

## Evaluation signals

- Per-replicate accuracy is recorded for every method on the identical t0/t1 splits; expected ordering: CASi achieves the highest accuracy and ARI in scenarios 1 and 2
- In scenario 3 (novel emerging cell type), accuracy and ARI should rise significantly only when the DetectNovelCells pipeline is combined with CrossTimeAnnotation
- ARI per scenario must be reproduced against the paper's supplementary Figure 11 values
- Simulation pool must contain exactly five cell types drawn randomly from the eight PBMC immune cell types, matching the pbmc_demo construction
- Final output is a CSV table aggregating 200 Monte Carlo repetitions by method and scenario (annotation_benchmark_results.csv)

## Limitations

- The neural network fails to borrow useful information when the initial timepoint's data is dramatically different from the later timepoint's data
- Novel cell detection may not work well when the novel cell is transcriptomically similar to known cell types
- Cell type annotation pools data from all samples together, which may be inappropriate when cell populations differ across conditions
- ARI comparison results are only available in supplementary Figure 11, not in the main text

## Evidence

- [abstract] Simulation studies and real multi-timepoint single cell dataset applications showed robust and favorable performance versus existing methods with similar purposes: [evidence span withheld — anchor status: not_found]
- [methods] PBMC source dataset: "We obtain a publicly available dataset of peripheral blood mononuclear cells (PBMC)[1], containing more than 60,000 sorted cells from eight immune cell types."
- [discussion] Simulation scenario settings referenced only in supplementary: "Using the same scenario settings of the simulation, we compare ARI of unsupervised clustering"
- [intro] Neural network classifier rationale: "Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting."
- [readme] CASi package capabilities: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data, which provides users with: (1) cross-time points cell annotation"
- [readme] TensorFlow prerequisite: [evidence span withheld — anchor status: external_doc]
