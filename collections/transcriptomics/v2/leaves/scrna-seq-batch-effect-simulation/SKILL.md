---
name: scrna-seq-batch-effect-simulation
description: Use when you need a controlled scRNA-seq integration benchmark with simulated
  large batch effects, or to verify reported LB-scenario iLISI/50-RNN values for BATMAN,
  Seurat V3.0, MNNcorrect, and Scanorama.
when_to_use_negative:
- The task is to integrate more than two datasets — BATMAN's current implementation
  works only with two datasets (multi-dataset integration is only possible 'in principle')
- The goal is batch correction of a real biological dataset rather than a controlled
  simulation-based benchmark with ground-truth batch structure
- You need to distinguish cell subtypes hidden by batch effects with deep transfer
  learning (e.g., BERMUDA-style analysis) rather than benchmark matching-based correction
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: Simulates the LB-scenario scRNA-seq dataset pair (100 datasets per scenario,
    1,000 genes each) used for the benchmark
- name: BATMAN
  role: Performs batch correction via minimum-weight matching of anchors across the
    two simulated datasets; benchmarked as the target method
  repo: https://github.com/mandricigor/batman
- name: Seurat V3.0
  role: Provides the pre-processing steps to obtain log-counts for each dataset, and
    serves as a comparison integration method
- name: MNNcorrect
  role: MNN-based comparison method run on the same preprocessed simulated pair
- name: Scanorama
  role: MNN-based comparison method run on the same preprocessed simulated pair
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:e2c304611ceed77047b4bdfd5dfe1c0b5293347847526843c4f6befd5e44d64e
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
  - MNNcorrect
  - Scanorama
  - Seurat V3.0
  - Splatter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# scrna-seq-batch-effect-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the BATMAN paper's large-batch-effects (LB) simulation benchmark by generating Splatter-simulated dataset pairs (1,000 genes, 100 datasets per scenario) and comparing BATMAN, Seurat V3.0, MNNcorrect, and Scanorama against the uncorrected baseline using mean iLISI (batch mixing) and mean 50-RNN (local structure preservation).

## When to use

Use when you need a controlled scRNA-seq integration benchmark with simulated large batch effects, or to verify reported LB-scenario iLISI/50-RNN values for BATMAN, Seurat V3.0, MNNcorrect, and Scanorama.

## When NOT to use

- The task is to integrate more than two datasets — BATMAN's current implementation works only with two datasets (multi-dataset integration is only possible 'in principle')
- The goal is batch correction of a real biological dataset rather than a controlled simulation-based benchmark with ground-truth batch structure
- You need to distinguish cell subtypes hidden by batch effects with deep transfer learning (e.g., BERMUDA-style analysis) rather than benchmark matching-based correction

## Inputs

- Splatter simulation configuration for the large-batch-effects (LB) scenario (1,000 genes per simulated dataset)
- Log-count matrices for the two simulated datasets produced by Seurat V3.0 pre-processing
- Anchor count (-anchors) and Pearson correlation filtering threshold (-filter-cor) parameters for BATMAN

## Outputs

- lb_integration_metrics.csv: table of mean iLISI and mean 50-RNN for Original, BATMAN, Seurat V3.0, MNNcorrect, and Scanorama
- Batch-corrected expression matrix from BATMAN (output of batman.py)

## How to apply

Simulate the LB-scenario dataset pair with Splatter using 1,000 genes per simulated dataset and 100 datasets per scenario, as in the paper's simulation design. Apply Seurat V3.0 pre-processing steps to obtain log-counts for each simulated dataset, then run BATMAN: build a weighted bipartite graph whose partitions correspond to the two datasets with edges weighted by Euclidean distance between cells, find the minimum-weight matching (exact blossom-based algorithm or the greedy approximation with ratio 2, the latter recommended for extra large datasets via the -greedy flag), determine local shifting vectors, and apply batch correction. Run Seurat V3.0, MNNcorrect, and Scanorama on the same preprocessed pair, keeping the uncorrected concatenation as the Original baseline. Compute mean iLISI and mean 50-RNN for Original, BATMAN, Seurat V3.0, MNNcorrect, and Scanorama, and tabulate both metrics into lb_integration_metrics.csv.

## Related tools

- **Splatter** (Simulates the LB-scenario scRNA-seq dataset pair (100 datasets per scenario, 1,000 genes each) used for the benchmark)
- **BATMAN** (Performs batch correction via minimum-weight matching of anchors across the two simulated datasets; benchmarked as the target method) — https://github.com/mandricigor/batman
- **Seurat V3.0** (Provides the pre-processing steps to obtain log-counts for each dataset, and serves as a comparison integration method)
- **MNNcorrect** (MNN-based comparison method run on the same preprocessed simulated pair)
- **Scanorama** (MNN-based comparison method run on the same preprocessed simulated pair)

## Examples

```
python3.7 batman.py -data lb_sim1.csv,lb_sim2.csv -output batman_corrected.csv -greedy
```

## Evaluation signals

- Mean iLISI across methods should approximate the reported LB values: 1.82 (Original), 1.84 (BATMAN), 1.00 (Seurat V3.0), 1.00 (MNNcorrect), 1.01 (Scanorama), within reasonable simulation variance
- Mean 50-RNN across methods should approximate the reported LB values: 1.00 (Original), 0.92 (BATMAN), 0.58 (Seurat V3.0), 0.51 (MNNcorrect), 0.14 (Scanorama)
- BATMAN should preserve the highest 50-RNN among corrected methods, confirming local structure retention
- Each scenario must be evaluated over 100 simulated datasets with 1,000 genes, matching the paper's simulation design
- Output table lb_integration_metrics.csv must contain exactly the five methods (Original, BATMAN, Seurat V3.0, MNNcorrect, Scanorama) with both metric columns

## Limitations

- The current implementation of BATMAN works only with two datasets; multi-dataset integration is not implemented (only possible 'in principle')
- The greedy matching is an approximation with approximation ratio 2, recommended only for extra large datasets, so results may differ from the exact blossom-based matching
- Simulated-benchmark results are subject to reasonable simulation variance; individual runs may deviate from the reported means

## Evidence
