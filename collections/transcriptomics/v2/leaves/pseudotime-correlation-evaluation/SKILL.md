---
name: pseudotime-correlation-evaluation
description: Use when benchmarking trajectory inference methods on simulated single-cell
  RNA-seq data with known gold-standard pseudotime and branch labels, especially when
  quantifying how technical noise (sigma) or capture efficiency (alpha) affects method
  performance.
when_to_use_negative:
- Data are discrete populations suited to clustering benchmarks rather than continuous
  trajectories; SymSim's clustering benchmarks (PCA, scVI, SIMLR) are the relevant
  evaluation instead
- No gold-standard pseudotime exists (real experimental data without simulation),
  since cor_pseudotime and cal_KNNP require simulated ground truth
- The goal is experimental design (how many cells to sequence to detect a rare subpopulation)
  rather than trajectory method comparison
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: Simulates true transcript counts (SimulateTrueCounts) and observed non-UMI
    counts (True2ObservedCounts) for the (Sigma, alpha) grid, and provides gold-standard
    trajectory labels via getTrajectoryGenes and the Phyla5() tree
  repo: github.com/YosefLab/SymSim
- name: Monocle
  role: Trajectory inference method under benchmark (version 2.6.4), scored by cor_pseudotime
    and cal_KNNP
- name: Slingshot
  role: Trajectory inference method under benchmark (version 0.99.12); the top-performing
    method in the reference comparison
- name: dynmethods
  role: Provides the dynverse minimum-spanning-tree trajectory inference implementation
    (via dynwrap 0.1.0) used as the third benchmarked method
  repo: github.com/dynverse/dynmethods
- name: dynwrap
  role: Wraps trajectory inference input/output into a common trajectory model so
    the MST method's pseudotime can be extracted and scored
  repo: github.com/dynverse/dynwrap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:1fa6b6c2266805feaad57c8b70d434350a01de467b77a022089a02bec0edfb78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle
  - Slingshot
  - SymSim
  - dynmethods
  - dynwrap
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Pseudotime correlation evaluation

## Summary

Benchmarks trajectory inference methods (Monocle 2.6.4, Slingshot 0.99.12, dynverse MST via dynmethods 0.1.0) on SymSim-simulated continuous cell populations, scoring inferred trajectories against gold-standard pseudotime with Spearman correlation (cor_pseudotime) and k-nearest-neighbor purity (cal_KNNP) while varying within-population variability (sigma) and capture efficiency (alpha).

## When to use

Use when benchmarking trajectory inference methods on simulated single-cell RNA-seq data with known gold-standard pseudotime and branch labels, especially when quantifying how technical noise (sigma) or capture efficiency (alpha) affects method performance.

## When NOT to use

- Data are discrete populations suited to clustering benchmarks rather than continuous trajectories; SymSim's clustering benchmarks (PCA, scVI, SIMLR) are the relevant evaluation instead
- No gold-standard pseudotime exists (real experimental data without simulation), since cor_pseudotime and cal_KNNP require simulated ground truth
- The goal is experimental design (how many cells to sequence to detect a rare subpopulation) rather than trajectory method comparison

## Inputs

- SymSim-simulated observed counts from True2ObservedCounts (protocol='nonUMI', alpha_mean set per grid point, gene lengths from gene_len_pool)
- Gold-standard pseudotime (distance from root) and branch labels from getTrajectoryGenes on simulated cell metadata
- Five-leaf trajectory tree from Phyla5()
- Inferred trajectory outputs from Monocle 2.6.4, Slingshot 0.99.12, and the dynmethods MST implementation

## Outputs

- cor_pseudotime: Spearman correlation between inferred and true pseudotime per method and per (Sigma, alpha) grid setting
- cal_KNNP: k-nearest-neighbor purity per method and per grid setting
- Method ranking across the Sigma/alpha grid comparable to Fig. 7a-b of the SymSim paper

## How to apply

Generate the five-leaf reference tree with SymSim's Phyla5() and, for each (Sigma, alpha) grid point, call SimulateTrueCounts with evf_type='continuous', ncells_total=500, ngenes=500, nevf=20, n_de_evf=12, vary='s', and a fixed randseed so runs are reproducible. Convert true transcript counts to observed non-UMI counts with True2ObservedCounts, setting alpha_mean from the alpha grid value and sampling gene lengths from the human-transcript-length pool gene_len_pool. Extract gold-standard cell pseudotime (distance from root) and branch labels with getTrajectoryGenes on the simulated cell metadata. Run Monocle 2.6.4, Slingshot 0.99.12, and the dynmethods (dynwrap 0.1.0) minimum-spanning-tree implementation on each observed-count matrix, then score each with cor_pseudotime (Spearman correlation of inferred vs. true pseudotime) and cal_KNNP (k-nearest-neighbor purity). The rationale is that SymSim explicitly models extrinsic cell-state variation and technical factors, so performance degradation across the grid is interpretable as a response to specific physical processes (mRNA capture efficiency, transcriptional noise).

## Related tools

- **SymSim** (Simulates true transcript counts (SimulateTrueCounts) and observed non-UMI counts (True2ObservedCounts) for the (Sigma, alpha) grid, and provides gold-standard trajectory labels via getTrajectoryGenes and the Phyla5() tree) — github.com/YosefLab/SymSim
- **Monocle** (Trajectory inference method under benchmark (version 2.6.4), scored by cor_pseudotime and cal_KNNP)
- **Slingshot** (Trajectory inference method under benchmark (version 0.99.12); the top-performing method in the reference comparison)
- **dynmethods** (Provides the dynverse minimum-spanning-tree trajectory inference implementation (via dynwrap 0.1.0) used as the third benchmarked method) — github.com/dynverse/dynmethods
- **dynwrap** (Wraps trajectory inference input/output into a common trajectory model so the MST method's pseudotime can be extracted and scored) — github.com/dynverse/dynwrap

## Examples

```
true_counts <- SimulateTrueCounts(ncells_total=500, ngenes=500, nevf=20, n_de_evf=12, evf_type='continuous', vary='s', phyla=Phyla5(), randseed=1); obs_counts <- True2ObservedCounts(true_counts[[1]], meta=true_counts[[3]], protocol='nonUMI', alpha_mean=0.1, gene_len_pool=gene_len_pool)
```

## Evaluation signals

- Spearman correlation (cor_pseudotime) and KNN purity (cal_KNNP) decrease monotonically as sigma increases when alpha=0.1
- Both metrics increase as alpha increases when sigma=0.6
- Slingshot's scores are consistently higher than Monocle 2.6.4 and the dynverse MST across the grid, matching the Fig. 7a-b trends and magnitudes
- Identical (Sigma, alpha) runs with a fixed randseed reproduce the same true-count matrices and gold-standard pseudotime
- Each scored dataset contains 500 cells and 500 genes simulated with nevf=20 and n_de_evf=12 on the Phyla5() tree

## Limitations

- The SymSim simulation pipeline omits several library-preparation steps, including reverse transcription and library cleaning up, so observed-count properties may not fully match all protocols
- Captured dynamics depend on the non-UMI protocol modeling via alpha_mean; scVI-based imputation is not applicable to small datasets (e.g., the 130-cell Th17 scale), limiting some preprocessing options
- Parameter estimates such as capture efficiency are reported in Supplementary Tables 2-5, which may not be included in the main text

## Evidence

- [methods] SimulateTrueCounts generates true transcript counts: "SimulateTrueCounts generates true transcript counts for the given number of genes and cells"
- [methods] True2ObservedCounts conversion: "True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts"
- [methods] Gold-standard trajectory extraction: "Function getTrajectoryGenes( ) returns the simulated trajectory information of cells."
- [abstract] Trajectory inference benchmarking use case: "We demonstrate how SymSim can be used for benchmarking methods for clustering, differential expression and trajectory inference"
- [readme] SymSim package installation: "SymSim (Synthetic model of multiple variability factors for Simulation) is an R package for simulation of single cell RNA-Seq data."
- [discussion] dynmethods wrapping of trajectory methods: "We use the R packages dynwrap (https://github.com/dynverse/dynwrap, version 0.1.0)"
