---
name: gamma-poisson-count-modeling
description: Use when benchmarking scRNA-seq batch-integration tools requires synthetic
  paired-batch count matrices with controlled batch strength, dropout rate, or batch-size
  imbalance rather than real data.
when_to_use_negative:
- The task is integrating real experimental datasets (e.g., Muraro CEL-Seq2, Segerstolpe
  Smart-Seq2, 10X PBMC 8k) rather than controlled simulation-based benchmarking.
- The benchmark requires more than two batches — the current BATMAN implementation
  only supports two-dataset integration.
- The goal is expression-analysis of an existing count matrix rather than generating
  synthetic data for a benchmark.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Splatter
  role: Gamma-Poisson simulator used to generate two-batch scRNA-seq count data under
    the five scenario configurations (1,000 genes; 100 datasets per scenario)
- name: BATMAN
  role: Batch-integration method whose benchmark consumes the simulated scenario datasets
  repo: https://github.com/mandricigor/batman
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:08426a0e600d2616d74fb01e8fe1460273c59627566380e2eda2cdd4ec0d4755
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gamma-poisson-count-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates two-batch scRNA-seq count data with Splatter's gamma-Poisson model to generate controlled batch-effect benchmarks (five scenarios: LB, LB-DR, LB-UB, SB, LB-CT) used to evaluate integration methods such as BATMAN against Seurat V3.0, MNNcorrect, and Scanorama.

## When to use

Use when benchmarking scRNA-seq batch-integration tools requires synthetic paired-batch count matrices with controlled batch strength, dropout rate, or batch-size imbalance rather than real data.

## When NOT to use

- The task is integrating real experimental datasets (e.g., Muraro CEL-Seq2, Segerstolpe Smart-Seq2, 10X PBMC 8k) rather than controlled simulation-based benchmarking.
- The benchmark requires more than two batches — the current BATMAN implementation only supports two-dataset integration.
- The goal is expression-analysis of an existing count matrix rather than generating synthetic data for a benchmark.

## Inputs

- Simulation parameter configuration (Splatter: nGenes=1000, batch.facLoc, batch.facScale, batchCells)
- Scenario specification from the BATMAN repository scripts (LB, LB-DR, LB-UB, SB, LB-CT)
- Number of replicates per scenario (100 datasets per scenario, per the paper's setup)

## Outputs

- One simulated two-batch count-matrix file per scenario (CSV/TSV), 1,000 genes, gamma-Poisson counts
- 100 datasets per scenario suitable for integration benchmarking with BATMAN, Seurat V3.0, MNNcorrect, and Scanorama

## How to apply

Load the simulation configuration and scenario scripts from the BATMAN repository (github.com/mandricigor/batman), then run Splatter with 1,000 genes under the gamma-Poisson model, simulating 100 datasets per scenario. Configure scenario-specific parameters: large batch effects use batch.facLoc/batch.facScale of 0.5, small batch effects use 0.001, the large-dropout (LB-DR) scenario elevates dropout, unequal-size batches (LB-UB) contain 200 and 1,000 cells, and LB-CT is a two-cell-type variant. Save each simulated batch pair as one count-matrix file per scenario for downstream integration benchmarking. Before feeding to integration tools, apply the same Seurat V3.0 pre-processing used in all BATMAN experiments to obtain log-counts. Validate qualitatively that each scenario file exists, contains two batches, and reproduces its designed condition (e.g., elevated dropout for LB-DR or batch shift for LB/SB).

## Related tools

- **Splatter** (Gamma-Poisson simulator used to generate two-batch scRNA-seq count data under the five scenario configurations (1,000 genes; 100 datasets per scenario))
- **BATMAN** (Batch-integration method whose benchmark consumes the simulated scenario datasets) — https://github.com/mandricigor/batman

## Examples

```
library(splatter); params <- newSplatParams(nGenes = 1000, batch.facLoc = 0.5, batch.facScale = 0.5); sim <- splatSimulate(params, batchCells = c(1000, 1000))
```

## Evaluation signals

- Each scenario file exists, contains exactly two batches, and has 1,000 genes.
- LB and SB scenarios differ in batch shift magnitude (batch.facLoc/batch.facScale 0.5 vs 0.001), verifiable by comparing per-batch mean expression shifts.
- LB-DR shows visibly elevated dropout relative to the LB scenario.
- LB-UB batch pair has unequal sizes of 200 and 1,000 cells.
- Downstream sanity check: on the original (uncorrected) LB data the reported mean iLISI is 1.82, and BATMAN achieves mean iLISI 1.84 and mean 50-RNN 0.92 after integration.

## Limitations

- The paper's validation is a qualitative QC check (elevated dropout or batch shift per scenario); no quantitative simulation-fidelity test is specified.
- The simulation scale is fixed by the paper's setup — 1,000 genes and 100 datasets per scenario — which may not transfer to genome-scale or many-batch benchmarks.
- BATMAN's current implementation works only with two datasets, so multi-batch simulations cannot be round-tripped through it.

## Evidence

- [other] Scenario parameterization: [evidence span withheld — anchor status: not_found]
