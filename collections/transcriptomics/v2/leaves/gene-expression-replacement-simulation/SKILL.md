---
name: gene-expression-replacement-simulation
description: Use when quantifying how much transcriptional identity (number of DE
  genes) a rare cell type needs before a detection method can still recover it, via
  controlled replacement of DE genes in subsampled scRNA-seq data.
when_to_use_negative:
- The goal is rare-cell detection on a single real dataset rather than benchmarking
  sensitivity against synthetic degradation of rare-cell identity
- No annotated abundant population (e.g., B cells) and rare population (e.g., monocytes)
  exist to define the replacement experiment
- The rare population is large enough (not 2–10 cells) that existing tools already
  discriminate it reliably, making the replacement-sensitivity design uninformative
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GapClust
  role: Primary method under test; detects the rare monocyte population in each synthetic
    replicate
  repo: https://github.com/fabotao/GapClust
- name: RaceID
  role: Comparison method; baseline rare-cell detection AUC (~0.63 at 50 DE genes)
- name: GiniClust
  role: Comparison method; baseline rare-cell detection AUC (~0.5 at 50 DE genes)
- name: FiRE
  role: Comparison method; baseline rare-cell detection AUC (~0.5 at 50 DE genes)
- name: Seurat
  role: Preprocessing/subsetting of the PBMC expression matrix and cell-type annotations
    (README prerequisite Seurat >= 3.1.0)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:973e6b0c6555f2e618fcda6f10f1daaf744917e81237bef041ae49856b476d15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabotao/GapClust
  tools:
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-expression-replacement-simulation

## Summary

Benchmarking skill for rare-cell detection sensitivity: build synthetic scRNA-seq replicates by replacing differentially expressed genes (1–144 DE genes) of a rare population within an abundant background (68k PBMC B cells vs CD14+ monocytes), then score detection accuracy by AUC across methods (GapClust, RaceID, GiniClust, FiRE), reproducing Fig. 3b.

## When to use

Use when quantifying how much transcriptional identity (number of DE genes) a rare cell type needs before a detection method can still recover it, via controlled replacement of DE genes in subsampled scRNA-seq data.

## When NOT to use

- The goal is rare-cell detection on a single real dataset rather than benchmarking sensitivity against synthetic degradation of rare-cell identity
- No annotated abundant population (e.g., B cells) and rare population (e.g., monocytes) exist to define the replacement experiment
- The rare population is large enough (not 2–10 cells) that existing tools already discriminate it reliably, making the replacement-sensitivity design uninformative

## Inputs

- 68k PBMC scRNA-seq expression matrix
- Cell-type annotations identifying CD19+ B cells and CD14+ monocytes
- List of differentially expressed genes (1–144) distinguishing monocytes from B cells

## Outputs

- 100 synthetic replicate expression matrices per monocyte count (2, 5, 10) and per DE-gene count
- Per-method, per-replicate AUC values for rare-cell detection
- Average-AUC curves as a function of DE gene number (Fig. 3b replication)

## How to apply

Load the 68k PBMC scRNA-seq expression matrix and subset 500 CD19+ B cells as the abundant background together with a rare population of 2, 5, or 10 CD14+ monocytes. For each monocyte count, generate 100 replicate datasets in which 1 to 144 differentially expressed genes are replaced following the paper's replacement-experiment design, spanning the sensitivity range. Run GapClust, RaceID, GiniClust, and FiRE on every synthetic replicate to detect the rare monocyte population, then compute the AUC of rare-cell detection per method per replicate and average over the 100 replicates. Plot average-AUC curves as a function of the number of DE genes to replicate Fig. 3b; the key decision point is the 2-monocyte / 50-DE-gene setting used for validation against the published benchmarks. The rationale is that replacing DE genes erodes the inter-cluster gap that GapClust exploits, so the AUC-vs-DE-genes curve directly measures each method's tolerance of diminishing rare-cell identity.

## Related tools

- **GapClust** (Primary method under test; detects the rare monocyte population in each synthetic replicate) — https://github.com/fabotao/GapClust
- **RaceID** (Comparison method; baseline rare-cell detection AUC (~0.63 at 50 DE genes))
- **GiniClust** (Comparison method; baseline rare-cell detection AUC (~0.5 at 50 DE genes))
- **FiRE** (Comparison method; baseline rare-cell detection AUC (~0.5 at 50 DE genes))
- **Seurat** (Preprocessing/subsetting of the PBMC expression matrix and cell-type annotations (README prerequisite Seurat >= 3.1.0))

## Examples

```
install.packages("devtools"); devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- In the 2-monocyte setting at 50 DE genes, GapClust's average AUC across 100 replicates should be ~0.89
- RaceID's average AUC at 50 DE genes (2 monocytes) should be ~0.63
- GiniClust and FiRE should plateau near ~0.5 AUC at 50 DE genes (2 monocytes)
- Average-AUC curves should rise with the number of DE genes (1 to 144) for all methods, with GapClust consistently highest
- Each replicate must contain exactly 500 B cells plus the specified monocyte count (2, 5, or 10) with the designated DE genes replaced

## Limitations

- GiniClust was reported to fail in processing very large datasets, and RaceID incurs unrealistic running time on datasets with thousands of cells, constraining their use in this benchmark
- FiRE does not discriminate outliers from minor cell types, limiting its detection AUC in the replacement design
- No version numbers for the compared tools (GiniClust, RaceID, CellSIUS, FiRE) are reported, so exact reproducibility of baselines may vary

## Evidence

- [results] In the 2-monocyte setting, with 50 DE genes present GapClust achieved an average AUC of 0.89, far exceeding RaceID (0.63), GiniClust (0.5) and FiRE (0.5), demonstrating superior sensitivity to cell-type identity.: [evidence span withheld — anchor status: not_found]
- [abstract] Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets: "Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets"
- [intro] Intro listing of comparison methods: "eminent among these are rare cell-type identification (RaceID)13, GiniClust14, cell subtype identification from up-regulated gene sets (CellSIUS)15 and finder of rare entities (FiRE)16"
- [readme] README on GapClust performance metric: "In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F<sub>1</sub> score."
