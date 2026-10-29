---
name: single-cell-rna-seq-data-subsetting
description: Use when building synthetic rare-cell detection benchmarks by subsampling
  known cell types from a large scRNA-seq expression matrix, or when probing how few
  DE genes preserve rare-cell identity.
when_to_use_negative:
- The goal is identifying novel rare populations in a real, unlabelled dataset rather
  than benchmarking a detector on controlled synthetic mixtures
- The dataset is too small to support an abundant population of ~500 cells plus a
  genuinely rare subpopulation
- Cell-type identities or marker genes are unknown, since the subsetting depends on
  picking defined populations (CD19+ B cells, CD14+ monocytes)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GapClust
  role: Rare-cell detector run on the synthetic replicate subsets to measure rare-cell
    detection AUC
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Baseline rare-cell detector run on each synthetic replicate for AUC comparison
- name: GiniClust
  role: Baseline rare-cell detector run on each synthetic replicate for AUC comparison
- name: FiRE
  role: Baseline rare-cell detector run on each synthetic replicate for AUC comparison
- name: Seurat
  role: scRNA-seq toolkit used for handling and preprocessing the expression matrix
    during subsetting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:e65c67ea137ad306c609bb205d44ca6f2b864e4dd12aeebb73fa78c7f33e6135
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  - Seurat
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# single-cell-rna-seq-data-subsetting

## Summary

Subsample cell populations from a large scRNA-seq matrix (e.g., 500 CD19+ B cells plus 2, 5, or 10 CD14+ monocytes drawn from the 68k PBMC dataset) to construct synthetic rare-cell benchmarks, optionally with DE gene replacement, for evaluating rare-cell detectors like GapClust.

## When to use

Use when building synthetic rare-cell detection benchmarks by subsampling known cell types from a large scRNA-seq expression matrix, or when probing how few DE genes preserve rare-cell identity.

## When NOT to use

- The goal is identifying novel rare populations in a real, unlabelled dataset rather than benchmarking a detector on controlled synthetic mixtures
- The dataset is too small to support an abundant population of ~500 cells plus a genuinely rare subpopulation
- Cell-type identities or marker genes are unknown, since the subsetting depends on picking defined populations (CD19+ B cells, CD14+ monocytes)

## Inputs

- 68k PBMC scRNA-seq expression matrix (cell-by-gene counts)
- Cell-type labels or marker gene sets to identify CD19+ B cells and CD14+ monocytes
- Replacement-experiment parameters (rare-cell counts 2/5/10; DE gene replacement range 1–144; 100 replicates per condition)

## Outputs

- Synthetic replicate expression matrices containing 500 B cells plus a rare monocyte subpopulation with substituted DE genes
- Per-method, per-replicate AUC values for rare-cell detection
- Average-AUC curves as a function of number of DE genes (replicating Fig. 3b)

## How to apply

Load the 68k PBMC scRNA-seq expression matrix and subset an abundant population (500 CD19+ B cells) together with a rare population of CD14+ monocytes at controlled sizes of 2, 5, and 10 cells. For each rare-cell count, generate 100 replicate datasets in which 1 to 144 differentially expressed genes are replaced following the paper's replacement-experiment design to degrade rare-cell identity. Run detectors (GapClust, RaceID, GiniClust, FiRE) on each replicate and compute average AUC of rare-cell detection across replicates. Rationale: the replacement design directly controls the number of distinguishing DE genes, so the resulting AUC-vs-DE-genes curves quantify each method's sensitivity to rare-cell identity, as in the paper's Fig. 3b.

## Related tools

- **GapClust** (Rare-cell detector run on the synthetic replicate subsets to measure rare-cell detection AUC) — github.com/fabotao/GapClust
- **RaceID** (Baseline rare-cell detector run on each synthetic replicate for AUC comparison)
- **GiniClust** (Baseline rare-cell detector run on each synthetic replicate for AUC comparison)
- **FiRE** (Baseline rare-cell detector run on each synthetic replicate for AUC comparison)
- **Seurat** (scRNA-seq toolkit used for handling and preprocessing the expression matrix during subsetting)

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- In the 2-monocyte setting with 50 DE genes, GapClust should reach an average AUC of ~0.89, versus RaceID ~0.63 and GiniClust/FiRE ~0.5
- Each experimental condition must contain 100 replicates spanning 1 to 144 replaced DE genes
- Abundant population size must be fixed at 500 B cells across all replicates so rare-cell fraction varies only via the 2/5/10 monocyte counts
- Average-AUC curves should increase monotonically toward 1 as the number of DE genes approaches 144 for a sensitive detector
- Resulting curves should qualitatively reproduce Fig. 3b of the GapClust paper

## Limitations

- GiniClust and FiRE plateau near AUC 0.5 in the 2-monocyte setting, indicating chance-level detection when few DE genes distinguish the rare population
- Results depend on the replacement-experiment design; altered DE gene pools or population sizes change the achievable AUCs
- GapClust requires R >= 3.4.0 with Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3 and e1071 >= 1.7-0.1 as prerequisites

## Evidence

- [results] In the 2-monocyte setting, with 50 DE genes present GapClust achieved an average AUC of 0.89, far exceeding RaceID (0.63), GiniClust (0.5) and FiRE (0.5): [evidence span withheld — anchor status: not_found]
- [abstract] GapClust is a lightweight rare-cell detector: "Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets"
- [abstract] Benchmarking shows superior performance: "Benchmarking on diverse experimental datasets demonstrates the superior performance of GapClust compared to other recently proposed methods"
- [readme] README installation prerequisites: "R users  can easily install GapClust by running following code in R console. 
``n# Install devtools first if it has not been installed on your R environment."
- [intro] Design based on inter-cluster gap: "the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to"
