---
name: auc-curve-computation-across-replicates
description: Use when benchmarking rare-cell detection sensitivity on scRNA-seq data
  by sweeping the number of DE genes (1–144) separating a known rare population from
  an abundant one, computing per-replicate AUCs and averaging across ~100 synthetic
  replicates.
when_to_use_negative:
- The goal is de novo rare-cell detection on a real dataset rather than controlled
  sensitivity benchmarking via DE gene replacement
- The input is not a scRNA-seq expression matrix with known labels for both an abundant
  population and a planted rare population (e.g. already an aggregated feature table)
- Only a single dataset (no replicate generation) is required — this skill's validity
  rests on averaging AUCs across ~100 replicate datasets
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: GapClust
  role: Primary rare-cell detector whose DE-gene sensitivity is benchmarked via per-replicate
    AUC (target average AUC ~0.89 at 50 DE genes, 2 monocytes)
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Comparison rare-cell-type identification method run on every synthetic replicate
    (target average AUC ~0.63 at 50 DE genes)
- name: GiniClust
  role: Comparison rare-cell detector run on every synthetic replicate; expected to
    stay near chance-level AUC (~0.5) in the 2-monocyte setting
- name: FiRE
  role: Comparison finder-of-rare-entities method run on every synthetic replicate;
    expected to stay near chance-level AUC (~0.5) in the 2-monocyte setting
- name: Seurat
  role: R toolkit used to load/subset the 68k PBMC expression matrix into the B-cell
    + monocyte replicate datasets (>= 3.1.0 required by GapClust)
  repo: github.com/fabotao/GapClust
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:e7675604b19c10930e701ffb3dbdacf23ebee61848ae0dec90f0fb825c4285dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# auc-curve-computation-across-replicates

## Summary

A benchmarking skill for quantifying how sensitive rare-cell detection methods are to the identity of a rare cell type: DE genes (1–144) distinguishing the rare population are progressively replaced in replicate synthetic datasets, rare-cell detection AUC is computed per replicate, and average-AUC curves are plotted against DE gene count.

## When to use

Use when benchmarking rare-cell detection sensitivity on scRNA-seq data by sweeping the number of DE genes (1–144) separating a known rare population from an abundant one, computing per-replicate AUCs and averaging across ~100 synthetic replicates.

## When NOT to use

- The goal is de novo rare-cell detection on a real dataset rather than controlled sensitivity benchmarking via DE gene replacement
- The input is not a scRNA-seq expression matrix with known labels for both an abundant population and a planted rare population (e.g. already an aggregated feature table)
- Only a single dataset (no replicate generation) is required — this skill's validity rests on averaging AUCs across ~100 replicate datasets

## Inputs

- scRNA-seq expression matrix (e.g. 68k PBMC count matrix)
- Cell-type labels identifying the abundant population (500 CD19+ B cells) and the rare population (2, 5, or 10 CD14+ monocytes)
- List of differentially expressed genes (1–144) distinguishing the rare cell type
- Rare-cell detection method implementations: GapClust, RaceID, GiniClust, FiRE

## Outputs

- Per-replicate AUC values for each method on each synthetic dataset
- Average-AUC curves as a function of the number of DE genes (Fig. 3b analogue)
- Comparative sensitivity summary across methods (e.g. GapClust 0.89 vs RaceID 0.63 vs GiniClust/FiRE 0.5 at 50 DE genes, 2 monocytes)

## How to apply

Subset an expression matrix (e.g. 68k PBMC) to an abundant population such as 500 CD19+ B cells plus a small rare population of 2, 5, or 10 CD14+ monocytes, and generate 100 replicate datasets per rare-cell count in which 1 to 144 DE genes are replaced following the paper's replacement-experiment design. Run each detector — GapClust, RaceID, GiniClust, FiRE — on every replicate to score the rare monocyte population, then compute the AUC of rare-cell detection per replicate and average across the 100 replicates. Plot average AUC as a function of DE gene number to reproduce Fig. 3b. Decision points: an average AUC near 0.5 indicates chance-level detection (the expected result for GiniClust and FiRE regardless of DE gene count in the hardest 2-monocyte setting), while rising AUC with DE gene count indicates DE-gene-driven sensitivity; validation targets are GapClust ~0.89 and RaceID ~0.63 at 50 DE genes with 2 monocytes.

## Related tools

- **GapClust** (Primary rare-cell detector whose DE-gene sensitivity is benchmarked via per-replicate AUC (target average AUC ~0.89 at 50 DE genes, 2 monocytes)) — github.com/fabotao/GapClust
- **RaceID** (Comparison rare-cell-type identification method run on every synthetic replicate (target average AUC ~0.63 at 50 DE genes))
- **GiniClust** (Comparison rare-cell detector run on every synthetic replicate; expected to stay near chance-level AUC (~0.5) in the 2-monocyte setting)
- **FiRE** (Comparison finder-of-rare-entities method run on every synthetic replicate; expected to stay near chance-level AUC (~0.5) in the 2-monocyte setting)
- **Seurat** (R toolkit used to load/subset the 68k PBMC expression matrix into the B-cell + monocyte replicate datasets (>= 3.1.0 required by GapClust)) — github.com/fabotao/GapClust

## Examples

```
devtools::install_github("fabotao/GapClust"); library(GapClust)  # R >= 3.4.0 with Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1
```

## Evaluation signals

- In the 2-monocyte setting at 50 DE genes, GapClust's average AUC over 100 replicates is ~0.89
- At the same setting, RaceID averages ~0.63 and GiniClust/FiRE average ~0.5 (chance level)
- Average AUC is a monotone non-decreasing function of the number of retained DE genes (1–144) for GapClust
- AUC computation is validated by averaging across exactly 100 replicate datasets per rare-cell count (2, 5, and 10 monocytes)
- The resulting average-AUC curves visually reproduce Fig. 3b of the GapClust paper

## Limitations

- AUC at chance level (~0.5) means the method cannot discriminate the rare population — methods like GiniClust and FiRE fail even with 144 DE genes in the hardest setting
- Results depend on the paper's specific DE gene replacement-experiment design; other replacement schemes may yield different sensitivity rankings
- Version numbers for the compared tools (GiniClust, RaceID, CellSIUS, FiRE, EDGE) are not stated, limiting exact reproducibility

## Evidence

- [results] GapClust sensitivity benchmark via DE gene replacement: [evidence span withheld — anchor status: not_found]
- [intro] Comparison methods named in the article: "eminent among these are rare cell-type identification (RaceID)13, GiniClust14, cell subtype identification from up-regulated gene sets (CellSIUS)15 and finder of rare entities (FiRE)16"
- [abstract] Benchmarking claim supporting the AUC-based comparison: "Benchmarking on diverse experimental datasets demonstrates the superior performance of GapClust compared to other recently proposed methods"
- [readme] README performance-evaluation framing: "In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F<sub>1</sub> score."
