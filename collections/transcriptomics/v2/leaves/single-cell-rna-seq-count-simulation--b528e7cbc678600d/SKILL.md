---
name: single-cell-rna-seq-count-simulation
description: Use when you need controlled synthetic two-batch scRNA-seq benchmarks
  with known batch-effect magnitude, dropout rates, batch sizes, or cell-type composition
  for evaluating integration or batch-correction methods.
when_to_use_negative:
- Your goal is to evaluate integration on real experimental data (e.g. the Muraro
  CEL-Seq2 and Segerstolpe Smart-Seq2 pancreatic datasets) rather than on synthetic
  data with known ground-truth batch structure
- You need simulated data with more than two batches — the BATMAN setup and its current
  implementation target two-dataset integration only
- The downstream method requires raw UMI counts with real biological structure; splatter's
  gamma-Poisson simulation may not capture your dataset's specific characteristics
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Splatter
  role: Simulator used to generate two-batch scRNA-seq count data under the gamma-Poisson
    model for each benchmark scenario
- name: BATMAN
  role: Downstream integration method benchmarked on the simulated batch pairs; repository
    also holds the simulation configuration/scenario scripts
  repo: https://github.com/mandricigor/batman
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:59007717eb7d4cac49f773b09b1098c031e551e0bd89066aa57cc414d28e8430
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# single-cell-rna-seq-count-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate two-batch scRNA-seq count datasets with Splatter under a gamma-Poisson model to generate controlled benchmarks for batch-integration methods, following the BATMAN paper's five scenarios (LB, LB-DR, LB-UB, SB, LB-CT) with 1,000 genes per dataset.

## When to use

Use when you need controlled synthetic two-batch scRNA-seq benchmarks with known batch-effect magnitude, dropout rates, batch sizes, or cell-type composition for evaluating integration or batch-correction methods.

## When NOT to use

- Your goal is to evaluate integration on real experimental data (e.g. the Muraro CEL-Seq2 and Segerstolpe Smart-Seq2 pancreatic datasets) rather than on synthetic data with known ground-truth batch structure
- You need simulated data with more than two batches — the BATMAN setup and its current implementation target two-dataset integration only
- The downstream method requires raw UMI counts with real biological structure; splatter's gamma-Poisson simulation may not capture your dataset's specific characteristics

## Inputs

- Simulation configuration/scenario scripts from the BATMAN repository (github.com/mandricigor/batman)
- Splatter scenario parameters (batch.facLoc, batch.facScale, gene count = 1,000, batch sizes, dropout settings, cell-type labels)

## Outputs

- One dataset file per scenario (e.g. CSV/TSV count matrices) each containing two simulated batches of scRNA-seq counts
- 100 simulated datasets per scenario (LB, LB-DR, LB-UB, SB, LB-CT) ready for integration benchmarking

## How to apply

Use Splatter to simulate two-batch scRNA-seq count data under the gamma-Poisson model with 1,000 genes per dataset, generating 100 simulated datasets per scenario as in the BATMAN benchmark. Configure five scenarios via Splatter parameters: (a) large batch effects with batch.facLoc/batch.facScale of 0.5, (b) a large dropout rate, (c) unequal batch sizes of 200 and 1,000 cells, (d) small batch effects with batch.facLoc/batch.facScale of 0.001, and (e) a two-cell-type variant (LB-CT). Save each simulated batch pair as one dataset file per scenario (e.g. CSV/TSV count matrices) so integration tools such as BATMAN, Seurat V3.0, MNNcorrect, and Scanorama can be run on identical inputs. Rationale: simulated data provide ground-truth batch structure so mixing metrics like iLISI and structure-retention metrics like 50-RNN can be interpreted against known conditions.

## Related tools

- **Splatter** (Simulator used to generate two-batch scRNA-seq count data under the gamma-Poisson model for each benchmark scenario)
- **BATMAN** (Downstream integration method benchmarked on the simulated batch pairs; repository also holds the simulation configuration/scenario scripts) — https://github.com/mandricigor/batman

## Examples

```
python3.7 batman.py -data scenario_LB_batch1.csv,scenario_LB_batch2.csv -output scenario_LB_corrected.csv
```

## Evaluation signals

- Each scenario output file exists and contains exactly two batches
- Each scenario reproduces its designed condition as a qualitative QC check (e.g. elevated dropout in LB-DR, batch shift in LB with batch.facLoc/batch.facScale = 0.5, unequal 200 vs 1,000 cell batch sizes in LB-UB, minimal batch shift in SB with 0.001)
- Each simulated dataset contains 1,000 genes as per the paper's setup
- 100 datasets are generated per scenario, matching the paper's benchmark scale

## Limitations

- Scenario conditions are verified only qualitatively (e.g. elevated dropout or batch shift), not with formal quantitative assertions
- The simulated setup covers two-batch cases; the current BATMAN implementation only works with two datasets, so the simulation design mirrors that scope
- Simulation parameters (batch.facLoc/batch.facScale, cell counts) must be manually configured per scenario; no single wrapper script across all five scenarios is documented

## Evidence
