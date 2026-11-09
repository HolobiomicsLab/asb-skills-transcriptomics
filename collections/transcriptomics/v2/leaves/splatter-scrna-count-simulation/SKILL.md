---
name: splatter-scrna-count-simulation
description: Use when a benchmark requires simulated scRNA-Seq counts with known ground
  truth (true counts before noise, DE gene labels, group assignments) so imputation
  quality can be scored against a reference.
when_to_use_negative:
- The analysis goal requires real biological scRNA-Seq data (e.g., the Baron dataset)
  where a simulator's generative assumptions would not reflect the actual tissue or
  protocol
- You already have paired true/observed count data from a controlled mixture experiment
  (e.g., the sc_mixology cell-line mixtures)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: Generates the simulated scRNA-Seq count matrices (Dataset 1, Dataset 8) with
    true counts, group structure, and DE gene labels used to benchmark imputation
    methods
  repo: https://github.com/datapplab/AutoClass
- name: AutoClass
  role: Downstream imputation method applied to the dropout-containing simulated matrix
    (AutoClassImpute, dropout_rate=0.3) whose performance is scored against simulation
    ground truth
  repo: https://github.com/datapplab/AutoClass
- name: SAVER
  role: Control imputation method run on the simulated data for method comparison
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Control imputation method run on the simulated data for method comparison
  repo: https://github.com/Vivianstats/scImpute
- name: DCA
  role: Control deep count autoencoder imputation method run on the simulated data
    for method comparison
  repo: https://github.com/theislab/dca
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:039c0e5f3d0a7cc0f00abffc4908f6d57783ad4676820db8420b53a0c5e03005
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - Python
  - SAVER
  - Splatter
  - numpy
  - scImpute
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# splatter-scrna-count-simulation

## Summary

Simulates single-cell RNA-Seq count matrices with known ground truth using Splatter — as in the AutoClass paper, which generated simulated datasets (e.g., 1000 genes × 500 cells) with true pre-noise counts, group structure, and differentially expressed gene labels — to benchmark denoising/imputation methods like AutoClass against SAVER, scImpute, and DCA.

## When to use

Use when a benchmark requires simulated scRNA-Seq counts with known ground truth (true counts before noise, DE gene labels, group assignments) so imputation quality can be scored against a reference.

## When NOT to use

- The analysis goal requires real biological scRNA-Seq data (e.g., the Baron dataset) where a simulator's generative assumptions would not reflect the actual tissue or protocol
- You already have paired true/observed count data from a controlled mixture experiment (e.g., the sc_mixology cell-line mixtures)

## Inputs

- Simulation parameters (nGenes, nCells, number of groups, DE gene count, dropout rate) e.g., 1000 genes, 500 cells, 2 groups, 161 DE genes, dropout_rate=0.3
- Random seed for reproducibility

## Outputs

- Simulated true (pre-noise) count matrix (e.g., truecounts.csv)
- Observed count matrix with dropout noise added (e.g., Dataset_raw.csv)
- DE gene labels for the 161 truly differentially expressed genes
- Group/cell-type assignments for each simulated cell

## How to apply

Use Splatter to simulate a count matrix with controlled structure — in the AutoClass paper, Dataset 8 used 1000 genes, 500 cells, and two groups with 161 truly differentially expressed genes, while Dataset 1 used 1000 genes and 500 cells in six groups. Retain the true (pre-noise) counts from the simulator and add dropout noise to the true counts to obtain the observed count matrix that serves as the raw input for downstream methods. Keep the DE gene labels so per-gene two-sample T-tests between groups can be run on the raw and imputed matrices (e.g., AutoClassImpute with dropout_rate=0.3, SAVER, scImpute, DCA), the median t-statistic across truly DE genes computed, and ROC/sensitivity-at-specificity metrics derived. The rationale is that the simulator provides paired true/observed data, which real scRNA-Seq data cannot, enabling direct quantification of method ordering — e.g., median DE t-statistic recovering from 2.11 (raw) to 5.86 (AutoClass) versus the true-data value of 5.79.

## Related tools

- **Splatter** (Generates the simulated scRNA-Seq count matrices (Dataset 1, Dataset 8) with true counts, group structure, and DE gene labels used to benchmark imputation methods) — https://github.com/datapplab/AutoClass
- **AutoClass** (Downstream imputation method applied to the dropout-containing simulated matrix (AutoClassImpute, dropout_rate=0.3) whose performance is scored against simulation ground truth) — https://github.com/datapplab/AutoClass
- **SAVER** (Control imputation method run on the simulated data for method comparison) — https://github.com/mohuangx/SAVER
- **scImpute** (Control imputation method run on the simulated data for method comparison) — https://github.com/Vivianstats/scImpute
- **DCA** (Control deep count autoencoder imputation method run on the simulated data for method comparison) — https://github.com/theislab/dca

## Examples

```
library(splatter); sim <- splatSimulate(nGenes = 1000, batchCells = 500, group.prob = c(0.5, 0.5), method = "groups", de.prob = 161/1000, seed = 8); truecounts <- counts(sim, altExp("Spliced"))
```

## Evaluation signals

- Median t-statistic across the truly DE genes on the raw (dropout) matrix should drop relative to the true data (e.g., 5.79 true → 2.11 raw in the paper's Dataset 8)
- AutoClass imputation should recover the median DE t-statistic to approximately the true-data level (~5.86 vs 5.79), exceeding SAVER, scImpute, and DCA
- Sensitivity at specificity = 0.90 computed from per-gene t-statistic ROC curves should reach ~0.61 for AutoClass vs ~0.30 for raw data, reproducing the reported method ordering
- True (pre-noise) counts and DE labels must be retained from the simulator output and kept disjoint from the dropout-added observed matrix
- Simulated dimensions should match the specification (1000 genes, 500 cells, 161 DE genes in two groups for Dataset 8)

## Limitations

- Simulation fidelity depends on Splatter's generative model and its parameters (Supplementary Tables 1–3 contain simulation parameters but are not included in the article text); conclusions only generalize to dropout-type noise as modeled
- Simulated data may not capture all noise types the paper identifies (general profiling noise, cell-cycle variation, batch effects), limiting extrapolation to real scRNA-Seq data
- Exactly reproducible numeric targets (e.g., median t-statistic 5.86, sensitivity ~0.61) depend on seeds and version pinning that are not fully specified in the article text

## Evidence

- [intro] Splatter provides controlled simulation parameters: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
- [results] Simulated data with dropout serve as the raw benchmark input: "X = expr.values #Splash generated scRNA-seq data with dropout"
- [results] AutoClassImpute with dropout_rate=0.3 is run on the simulated matrix: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [results] True counts are retained separately from the simulated data: "expr_true = pd.read_csv('/Dataset1/truecounts.csv',index_col=0)"
- [intro] Imputation methods are benchmarked against state-of-the-art alternatives on simulated data: "In this study, we extensively evaluated AutoClass against existing methods using multiple simulated and real datasets."
