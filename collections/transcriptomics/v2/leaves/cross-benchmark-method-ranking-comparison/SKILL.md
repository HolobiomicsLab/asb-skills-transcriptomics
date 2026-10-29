---
name: cross-benchmark-method-ranking-comparison
description: Use when ranking GRN inference algorithms against a known ground-truth
  GRN on simulated scRNA-seq data (BoneMarrow, PBMC) and checking whether the resulting
  method ordering reproduces BEELINE's experimental-benchmark results.
when_to_use_negative:
- Benchmarking on the multi-cell-type PBMC-All dataset, where the paper states the
  benchmarking results are not conclusive and follow-up studies are required
- Comprehensively evaluating cell-specific GRN inference methods such as CeSpGRN —
  the paper explicitly notes no comprehensive study of this method class has been
  performed
- Judging simulated-data quality rather than GRN inference performance — use GRouNdGAN's
  --evaluate command instead of --benchmark_grn
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GRouNdGAN
  role: Generates the simulated BoneMarrow/PBMC expression data with an imposed causal
    ground-truth GRN and provides the --benchmark_grn command that evaluates each
    inferred network against it
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: BEELINE
  role: Reference curated-benchmark framework whose method ranking (from experimental
    data) is compared against the GRouNdGAN-simulated ranking
  repo: https://github.com/Murali-group/Beeline
- name: GRNBoost2
  role: Infers regulatory edges from the reference dataset that are imposed (top 15
    TFs per gene) as the ground-truth GRN used in simulation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:e69b354ded9400e1e814517a86a6ea5ef4af3e3241a6fafe497b968b79280d93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - BEELINE
  - GRNBoost2
  - GRouNdGAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-benchmark-method-ranking-comparison

## Summary

Benchmarks eight GRN inference methods (GRNBoost2, GENIE3, PIDC, PPCOR, LEAP, SCODE, SINCERITIES, CeSpGRN) on GRouNdGAN-simulated BoneMarrow and PBMC data and aggregates precision-at-k and AUPRC into a single ranking table that is validated against BEELINE's curated-benchmark method ordering.

## When to use

Use when ranking GRN inference algorithms against a known ground-truth GRN on simulated scRNA-seq data (BoneMarrow, PBMC) and checking whether the resulting method ordering reproduces BEELINE's experimental-benchmark results.

## When NOT to use

- Benchmarking on the multi-cell-type PBMC-All dataset, where the paper states the benchmarking results are not conclusive and follow-up studies are required
- Comprehensively evaluating cell-specific GRN inference methods such as CeSpGRN — the paper explicitly notes no comprehensive study of this method class has been performed
- Judging simulated-data quality rather than GRN inference performance — use GRouNdGAN's --evaluate command instead of --benchmark_grn

## Inputs

- GRouNdGAN-simulated scRNA-seq expression matrices (100k cells × 1000 genes) for BoneMarrow and PBMC reference datasets
- Exported ground-truth GRN CSV (causal graph imposed for simulation, each gene regulated by its top 15 GRNBoost2-identified TFs), saved via [GRN Benchmarking]/ground truth save path
- Inferred GRN files from the eight methods, formatted as tab-separated CSV with columns TF, Gene, importance

## Outputs

- ranking_table.csv aggregating precision@k (k=15) and AUPRC across both datasets into a single method ranking
- Per-gene precision-at-k (k=15) values for each inferred network
- Global Precision-Recall curves with AUPRC, including the random-predictor baseline AUPRC

## How to apply

Run each of the eight inference methods on the GRouNdGAN-simulated expression matrices (100k cells × 1000 genes for BoneMarrow and PBMC), formatting every inferred network as a tab-separated CSV with columns TF, Gene, importance (headers optional). For each network, run GRouNdGAN's --benchmark_grn command, which evaluates the reconstructed GRN against the exported ground-truth GRN (the causal graph imposed at simulation time, built from the top 15 GRNBoost2-identified TFs per gene); specify the ground truth export via [GRN Benchmarking]/ground truth save path. Compute precision-at-k per gene with k = 15 (matching the imposed out-degree) and a global Precision-Recall curve with AUPRC, always including the random-predictor baseline AUPRC as a floor. Aggregate the precision@k and AUPRC metrics across both datasets into a single method ranking table saved as ranking_table.csv. Validate the result by confirming PIDC ranks first (followed by GENIE3 and GRNBoost2) and that the ordering aligns with BEELINE's curated benchmarks, with the only expected difference being LEAP and SCODE exchanging ranks.

## Related tools

- **GRouNdGAN** (Generates the simulated BoneMarrow/PBMC expression data with an imposed causal ground-truth GRN and provides the --benchmark_grn command that evaluates each inferred network against it) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **BEELINE** (Reference curated-benchmark framework whose method ranking (from experimental data) is compared against the GRouNdGAN-simulated ranking) — https://github.com/Murali-group/Beeline
- **GRNBoost2** (Infers regulatory edges from the reference dataset that are imposed (top 15 TFs per gene) as the ground-truth GRN used in simulation)

## Examples

```
bash main.py --benchmark_grn
```

## Evaluation signals

- PIDC ranks first in ranking_table.csv on both BoneMarrow and PBMC, followed by GENIE3 and GRNBoost2
- Method ordering matches BEELINE's curated experimental-benchmark results with only LEAP and SCODE exchanged in rank
- Every inferred network is a tab-separated CSV with columns in the order TF, Gene, importance (headers optional)
- Precision-at-k uses k = 15, matching the imposed top-15-GRNBoost2-TF out-degree of the ground-truth GRN
- AUPRC is reported alongside the random-predictor baseline AUPRC, which all methods must exceed

## Limitations

- The imposed causal ground-truth graph is not necessarily unique for an observational dataset; verifying it is the most probable graph is beyond the study's scope
- Benchmarking results on PBMC-All (multiple distinct cell types) are not conclusive and require follow-up studies
- No comprehensive evaluation of cell-specific GRN inference methods (e.g., CeSpGRN) has been performed

## Evidence

- [intro] Objective: benchmark GRN inference methods using GRouNdGAN and compare with BEELINE's experimental results: "Using GRouNdGAN, we benchmark eight GRN inference methods and find the results to be aligned with BEELINE’s experimental results19."
- [methods] --benchmark_grn evaluates inferred GRN against ground truth: "This command evaluates the reconstructed GRN (from GRouNdGAN-simulate data) against the ground truth GRN (used for simulation)."
- [methods] Ground-truth GRN export path: "If you specify ``[GRN Benchmarking]/ground truth save path``, the ground truth GRN will be exported as a CSV:"
- [discussion] Ground truth GRN built from GRNBoost2-imposed edges: "we recommend first finding a set of regulatory edges from the reference dataset (e.g., using GRNBoost2 or even other data modalities) and imposing a selected subset of those in the simulated data"
