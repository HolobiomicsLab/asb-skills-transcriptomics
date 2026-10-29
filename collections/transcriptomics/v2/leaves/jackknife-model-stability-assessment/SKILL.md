---
name: jackknife-model-stability-assessment
description: Use when fitting per-gene PLS1 regression models on multi-omics GRN data
  (e.g., voom-transformed RNA-seq response against miRNA-seq and TF predictor layers)
  and you need to separate regulations stable across resampled fits from spurious
  single-fit associations.
when_to_use_negative:
- 'Single-cell data: MOSim validates single-cell gene-regulator relationships with
  Kendall''s Tb correlation, not Pearson on bulk profiles, so a bulk PLS1 + Pearson
  stability workflow is the wrong template'
- Data simulated from omics layers MOSim does not cover (proteomics, metabolomics,
  spatially resolved GRNs), where no ground-truth regulatory network exists for stability-based
  inference
- Datasets without a two-group simulated design or ground-truth regulatory pairs —
  Jack-Knife stability can flag coefficients, but sensitivity/F1 benchmarking against
  random feature pairs requires the simulated true positives/negatives
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: MOSim
  role: Simulates the bulk multi-omics layers (RNA-seq, miRNA-seq, TF) with ground-truth
    regulations that the Jack-Knife-stable MORE models are validated against
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Runtime environment for the MOSim package, voom/limma preprocessing, and PLS1
    regression with Jack-Knife resampling
  repo: github.com/ConesaLab/MOSim
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:3fcf4e35cfce6a31fc25729da1e928c812bffdafc5908a921f8d2b0750b4637d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Jack-Knife model stability assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Per-gene regression models (PLS1 in the MORE GRN-inference benchmark) are refitted under Jack-Knife resampling so that regulator coefficients surviving resampling are called stable/significant regulations rather than artifacts of a single fit.

## When to use

Use when fitting per-gene PLS1 regression models on multi-omics GRN data (e.g., voom-transformed RNA-seq response against miRNA-seq and TF predictor layers) and you need to separate regulations stable across resampled fits from spurious single-fit associations.

## When NOT to use

- Single-cell data: MOSim validates single-cell gene-regulator relationships with Kendall's Tb correlation, not Pearson on bulk profiles, so a bulk PLS1 + Pearson stability workflow is the wrong template
- Data simulated from omics layers MOSim does not cover (proteomics, metabolomics, spatially resolved GRNs), where no ground-truth regulatory network exists for stability-based inference
- Datasets without a two-group simulated design or ground-truth regulatory pairs — Jack-Knife stability can flag coefficients, but sensitivity/F1 benchmarking against random feature pairs requires the simulated true positives/negatives

## Inputs

- Voom-transformed bulk RNA-seq count matrix (response layer, after NOISeq filtering/normalization)
- Simulated miRNA-seq and TF regulatory layers (predictor matrices)
- Group labels for the two-group simulated design (fixed seed, 30M reads, 20 time points, one replicate, 50% DEGs, 60% significant regulations)

## Outputs

- Per-gene fitted PLS1 models (expected 5,573 fitted models)
- Enumerated gene–regulator interactions with significance per group (370,566 potential; 174,051 and 174,067 significant in groups 1 and 2)
- Benchmark metrics file more_evaluation_metrics.csv (DEG count, fitted models, potential/significant interactions, sensitivity, F1-score)

## How to apply

In the MORE benchmark on MOSim-simulated bulk data, first preprocess the simulated count matrices (NOISeq filtering/normalization, voom transformation for linear modeling, limma DE calling with an expected 10,593 DEGs). Then run MORE inference using PLS1 regression with Jack-Knife resampling to fit one model per gene — expected 5,573 fitted models — where miRNA-seq and TF layers act as regulators of gene expression. Coefficients that remain stable across the Jack-Knife leave-one-out refits are enumerated as gene–regulator interactions (370,566 potential; 174,051 and 174,067 significant in groups 1 and 2, run separately per group). Validate stability calls with Pearson correlation on bulk data, comparing inferred regulations against the MOSim ground truth and against 20,000 random feature pairs with no simulated regulatory effect, then compute sensitivity (expected 85.5%) and F1-score (expected 62.9%). Write reproduced metrics (DEG count, fitted models, potential/significant interactions, sensitivity, F1) to more_evaluation_metrics.csv.

## Related tools

- **MOSim** (Simulates the bulk multi-omics layers (RNA-seq, miRNA-seq, TF) with ground-truth regulations that the Jack-Knife-stable MORE models are validated against) — github.com/ConesaLab/MOSim
- **R** (Runtime environment for the MOSim package, voom/limma preprocessing, and PLS1 regression with Jack-Knife resampling) — github.com/ConesaLab/MOSim

## Examples

```
BiocManager::install("MOSim")
```

## Evaluation signals

- Fitted-model count matches the expected 5,573 per-gene PLS1 models after Jack-Knife resampling
- Significant interaction counts land near the expected per-group values (174,051 in group 1 and 174,067 in group 2, out of 370,566 potential pairs)
- Benchmark metrics reproduce sensitivity ≈ 85.5% and F1-score ≈ 62.9% against the MOSim-simulated ground truth
- Pearson correlations of inferred regulations separate from the distribution of 20,000 random feature pairs with no simulated regulatory effect
- more_evaluation_metrics.csv is populated with DEG count (expected 10,593), fitted models, interaction counts, sensitivity, and F1

## Limitations

- The reference design uses a single replicate and two groups (STATegra-derived configuration, 20 time points), limiting resampling diversity
- MOSim ground truth is restricted to scRNA-seq and scATAC-seq for single cell, so this bulk-style Jack-Knife stability workflow cannot be transplanted directly to single-cell layers
- MOSim does not simulate GRNs with spatial resolution, nor proteomics/metabolomics layers, bounding the biological scope of validated regulations

## Evidence

- [other] Run MORE inference using PLS1 regression with Jack-Knife resampling to fit per-gene models (expected: 5,573 fitted models): [evidence span withheld — anchor status: not_found]
