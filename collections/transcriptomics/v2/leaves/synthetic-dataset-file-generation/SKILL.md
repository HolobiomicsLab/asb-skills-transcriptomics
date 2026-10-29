---
name: synthetic-dataset-file-generation
description: Use when a benchmark of batch-integration tools needs controlled two-batch
  scRNA-seq count matrices where batch magnitude, dropout rate, batch-size imbalance,
  or cell-type composition must be varied systematically.
when_to_use_negative:
- The goal is integration of real experimental scRNA-seq datasets (e.g., the Muraro
  CEL-Seq2 pancreatic dataset, Segerstolpe Smart-Seq2 dataset, 10X PBMC 8k, or 10X
  Pan T cells) rather than controlled simulation
- Only a single batch/dataset exists and no contrast between batches can be simulated
  or evaluated
- The downstream tool only handles two datasets — note the current BATMAN implementation
  works only with two datasets, so simulated files must contain exactly two batches
tools:
- name: Splatter
  role: Simulator that generates the two-batch scRNA-seq count data under the gamma-Poisson
    model for each of the five scenarios
- name: BATMAN
  role: Downstream integration method benchmarked on the simulated files; its repository
    provides the scenario scripts
  repo: https://github.com/mandricigor/batman
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:ea6ef42e249322143bf54e1b213fcbd65b25017240dbff6c172f27e11cce6b38
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mandricigor/batman
  repo_url: https://github.com/mandricigor/batman
  tools:
  - BATMAN
  - Splatter
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# synthetic-dataset-file-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates simulated two-batch single-cell RNA-seq count-matrix files with Splatter under the gamma-Poisson model, reproducing the five benchmark scenarios (LB, LB-DR, LB-UB, SB, LB-CT) used to evaluate BATMAN against Seurat V3.0, MNNcorrect, and Scanorama.

## When to use

Use when a benchmark of batch-integration tools needs controlled two-batch scRNA-seq count matrices where batch magnitude, dropout rate, batch-size imbalance, or cell-type composition must be varied systematically.

## When NOT to use

- The goal is integration of real experimental scRNA-seq datasets (e.g., the Muraro CEL-Seq2 pancreatic dataset, Segerstolpe Smart-Seq2 dataset, 10X PBMC 8k, or 10X Pan T cells) rather than controlled simulation
- Only a single batch/dataset exists and no contrast between batches can be simulated or evaluated
- The downstream tool only handles two datasets — note the current BATMAN implementation works only with two datasets, so simulated files must contain exactly two batches

## Inputs

- Scenario configuration (LB, LB-DR, LB-UB, SB, or LB-CT) selecting batch-effect magnitude, dropout rate, batch-size balance, or cell-type structure
- Splatter simulation parameters: 1,000 genes, two batches, batch.facLoc/batch.facScale (0.5 for large, 0.001 for small batch effects), batch sizes (200 and 1,000 cells for the unequal-size scenario)

## Outputs

- One simulated two-batch dataset file per scenario (CSV/TSV count matrices), 100 datasets per scenario
- Simulated gamma-Poisson count matrices suitable for downstream integration benchmarking with BATMAN, Seurat V3.0, MNNcorrect, and Scanorama

## How to apply

Load the scenario scripts from the BATMAN repository (github.com/mandricigor/batman) and run Splatter to simulate two-batch count data under the gamma-Poisson model with 1,000 genes. Configure one of five scenario variants: large batch effects (batch.facLoc/batch.facScale = 0.5), large dropout rate, unequal batch sizes (200 vs. 1,000 cells), small batch effects (batch.facLoc/batch.facScale = 0.001), or a two-cell-type variant. Per the paper's setup, simulate 100 datasets per scenario and save each simulated batch pair as one dataset file per scenario (e.g., CSV/TSV count matrices). These files then serve as inputs to the integration benchmark, where Seurat V3.0 pre-processing is used to obtain log-counts before running BATMAN and the other tools.

## Related tools

- **Splatter** (Simulator that generates the two-batch scRNA-seq count data under the gamma-Poisson model for each of the five scenarios)
- **BATMAN** (Downstream integration method benchmarked on the simulated files; its repository provides the scenario scripts) — https://github.com/mandricigor/batman

## Examples

```
sim <- splatSimulate(nGenes = 1000, batchCells = c(200, 1000), batch.facLoc = 0.5, batch.facScale = 0.5)
```

## Evaluation signals

- Each scenario file exists, contains exactly two batches, and has 1,000 genes
- 100 simulated datasets are produced per scenario, matching the paper's setup
- Qualitative QC: the designed condition is reproducible in the file — elevated dropout in LB-DR, strong batch shift in LB (batch.facLoc/facScale 0.5) vs. negligible shift in SB (0.001), and unequal cell counts (200 vs. 1,000) in LB-UB
- Downstream benchmarking runs: mean iLISI on the original LB data is ~1.82, so a mis-specified simulation should be detectable when compared against reported baselines (BATMAN 1.84, Seurat V3.0 1.00, MNNcorrect 1.00, Scanorama 1.01)

## Limitations

- Only two datasets per simulated pair are supported by the downstream BATMAN implementation; multi-dataset integration is not implemented
- A simple qualitative QC check of scenario conditions (e.g., elevated dropout or batch shift) is a coarse validation rather than a statistical test
- Simulations use the gamma-Poisson model and 1,000 genes, which may not capture all real-dataset characteristics

## Evidence

- [other] The simulations comprise five scenarios (LB, LB-DR, LB-UB, SB, LB-CT) with 1,000 genes each; large batch effects use batch.facLoc/batch.facScale of 0.5, small batch effects use 0.001: [evidence span withheld — anchor status: not_found]
