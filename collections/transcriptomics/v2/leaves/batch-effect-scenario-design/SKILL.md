---
name: batch-effect-scenario-design
description: Use when benchmarking batch integration/correction methods requires controlled
  simulated scRNA-seq count matrices with tunable batch-effect magnitude, dropout,
  batch-size imbalance, or cell-type composition.
when_to_use_negative:
- The goal is correction of real experimental scRNA-seq data (e.g., Muraro CEL-Seq2
  vs. Segerstolpe Smart-Seq2 pancreatic datasets) rather than controlled benchmarking
  of method behavior per distortion mechanism
- The integration task involves more than two batches/datasets — BATMAN's current
  implementation supports only two datasets
- The downstream method requires raw UMI distributions incompatible with the gamma-Poisson
  Splatter model assumptions
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Splatter
  role: Simulates two-batch scRNA-seq count data under the gamma-Poisson model for
    each of the five batch-effect scenarios (1,000 genes, 100 datasets per scenario)
- name: BATMAN
  role: Repository hosts the scenario simulation configuration; simulated datasets
    are used to benchmark BATMAN's minimum-weight matching correction
  repo: https://github.com/mandricigor/batman
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:948cdf544371e8ce8254a88c98b5bf97ac641b1dc31afb1c81c03a611b4dfabe
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
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# batch-effect-scenario-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design and generate simulated two-batch scRNA-seq benchmark datasets using Splatter under five controlled batch-effect scenarios (LB, LB-DR, LB-UB, SB, LB-CT), each with 1,000 genes and 100 replicate datasets, to stress-test batch integration tools like BATMAN.

## When to use

Use when benchmarking batch integration/correction methods requires controlled simulated scRNA-seq count matrices with tunable batch-effect magnitude, dropout, batch-size imbalance, or cell-type composition.

## When NOT to use

- The goal is correction of real experimental scRNA-seq data (e.g., Muraro CEL-Seq2 vs. Segerstolpe Smart-Seq2 pancreatic datasets) rather than controlled benchmarking of method behavior per distortion mechanism
- The integration task involves more than two batches/datasets — BATMAN's current implementation supports only two datasets
- The downstream method requires raw UMI distributions incompatible with the gamma-Poisson Splatter model assumptions

## Inputs

- Splatter simulation configuration/scripts from github.com/mandricigor/batman
- Scenario parameter sets: batch.facLoc/batch.facScale (0.5 for LB-type, 0.001 for SB), batch size settings (equal vs. 200/1000 cells), dropout settings, cell-type variant settings
- Gene count setting (1,000 genes per simulated dataset)

## Outputs

- One simulated two-batch dataset per replicate per scenario (e.g., CSV/TSV count matrices with 1,000 genes), five scenario files/families (LB, LB-DR, LB-UB, SB, LB-CT) with 100 datasets each
- Per-scenario benchmark inputs for integration tools yielding mean iLISI and mean 50-RNN scores

## How to apply

Load the scenario configuration scripts from the BATMAN repository (github.com/mandricigor/batman) and use Splatter to simulate two-batch scRNA-seq count data under the gamma-Poisson model with 1,000 genes per dataset. Configure the five scenarios: LB (large batch effects via batch.facLoc=0.5 and batch.facScale=0.5), LB-DR (large batch effects plus elevated dropout rate), LB-UB (large batch effects with unequal batch sizes of 200 and 1,000 cells), SB (small batch effects via batch.facLoc=0.001 and batch.facScale=0.001), and LB-CT (a two-cell-type variant). Follow the paper's setup of simulating 100 datasets per scenario so each integration method can be run over replicate pairs. Save the simulated batch pairs as count-matrix files (e.g., CSV/TSV), one per scenario, and run the correction tools (e.g., BATMAN, Seurat V3.0, MNNcorrect, Scanorama) on these log-normalized inputs. Rationale: simulations isolate specific distortion mechanisms (batch shift, dropout, imbalance, cell-type composition) that real datasets confound, allowing per-scenario scoring of methods on metrics like mean iLISI and mean 50-RNN.

## Related tools

- **Splatter** (Simulates two-batch scRNA-seq count data under the gamma-Poisson model for each of the five batch-effect scenarios (1,000 genes, 100 datasets per scenario))
- **BATMAN** (Repository hosts the scenario simulation configuration; simulated datasets are used to benchmark BATMAN's minimum-weight matching correction) — https://github.com/mandricigor/batman

## Examples

```
Rscript -e "splatSimulate(group.prob=NULL, batchCells=c(1000,1000), nGenes=1000, batch.facLoc=0.5, batch.facScale=0.5)"
```

## Evaluation signals

- Each scenario file exists and contains exactly two batches (equal-sized for LB/LB-DR/SB/LB-CT; 200 vs. 1,000 cells for LB-UB)
- Each dataset contains 1,000 genes and 100 replicate datasets are present per scenario
- Qualitative QC: the designed condition is reproducible, e.g., elevated dropout in LB-DR, small batch shift in SB (facLoc/facScale=0.001) vs. large shift in LB (0.5)
- Original (uncorrected) LB data should yield a mean iLISI near 1.82, so corrected outputs can be compared against that baseline
- Simulated counts should be consistent with the gamma-Poisson generative model (nonnegative integer counts with scenario-appropriate zero inflation)

## Limitations

- BATMAN's current implementation works only with two datasets, so simulated benchmarks are restricted to pairwise batch correction
- Five scenarios may not capture the full space of real batch-effect mechanisms; results are scenario-dependent
- Simulations use 1,000 genes, which is far smaller than typical genome-wide scRNA-seq datasets, limiting transferability of conclusions

## Evidence

- [results] The simulations comprise five scenarios (LB, LB-DR, LB-UB, SB, LB-CT) with 1,000 genes each; large batch effects use batch.facLoc/batch.facScale of 0.5, small batch effects use 0.001, and unequal-size batches contain 200 and 1,000 cells.: [evidence span withheld — anchor status: not_found]
- [results] For each scenario, we simulated 100 datasets and ran BATMAN alongside the other three tools.: "For each scenario, we simulated 100 datasets and ran BATMAN alongside the other three tools."
- [abstract] Splatter: simulation of single-cell RNA sequencing data: "Splatter: simulation of single-cell RNA sequencing data"
- [results] MetricOriginalBATMANSeurat V3.0MNNcorrectScanoramaMean iLISI1.821.841.001.001.01: "MetricOriginalBATMANSeurat V3.0MNNcorrectScanoramaMean iLISI1.821.841.001.001.01"
