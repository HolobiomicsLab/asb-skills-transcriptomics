---
name: precision-recall-curve-analysis
description: Use when inferred GRN edge lists (TF, Gene, importance from PIDC, GENIE3,
  GRNBoost2, etc.) must be evaluated against a ground-truth GRN to rank inference
  methods by precision@k and AUPRC.
when_to_use_negative:
- You need to assess the quality of the simulated cells themselves (trajectory, pseudo-time,
  noise) rather than GRN inference performance — use GRouNdGAN's --evaluate data-quality
  step instead.
- The dataset has no imposed ground-truth GRN; this skill only evaluates a reconstructed
  GRN against the GRN used for simulation.
- You are working with raw unprocessed scRNA-seq data — a GRN must first be inferred
  (e.g., --create_grn with GRNBoost2) before benchmarking is possible.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GRouNdGAN
  role: Provides the --benchmark_grn evaluation command, the imposed causal ground-truth
    GRN, and ground-truth CSV export used for precision-at-k/AUPRC benchmarking.
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the regulatory edges from the preprocessed reference data that become
    the ground truth (top 15 TFs per gene) and produces candidate inferred networks
    for benchmarking.
- name: BEELINE
  role: Curated experimental benchmark whose method ordering serves as the external
    validation target for the GRouNdGAN-based ranking.
  repo: github.com/Murali-group/Beeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:aa25bac1033c9fee546bfb4652a496818b8e582a78d53374b8827c940854d574
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BEELINE
  - GRNBoost2
  - GRouNdGAN
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# precision-recall-curve-analysis

## Summary

Scores inferred gene regulatory networks against a GRouNdGAN-imposed ground-truth GRN by computing per-gene precision-at-k (k = 15) and a global precision–recall curve with AUPRC, including the random-predictor baseline, to rank GRN inference methods.

## When to use

Use when inferred GRN edge lists (TF, Gene, importance from PIDC, GENIE3, GRNBoost2, etc.) must be evaluated against a ground-truth GRN to rank inference methods by precision@k and AUPRC.

## When NOT to use

- You need to assess the quality of the simulated cells themselves (trajectory, pseudo-time, noise) rather than GRN inference performance — use GRouNdGAN's --evaluate data-quality step instead.
- The dataset has no imposed ground-truth GRN; this skill only evaluates a reconstructed GRN against the GRN used for simulation.
- You are working with raw unprocessed scRNA-seq data — a GRN must first be inferred (e.g., --create_grn with GRNBoost2) before benchmarking is possible.

## Inputs

- Inferred GRN as a tab-separated CSV with columns TF, Gene, importance (headers optional)
- Ground-truth GRN exported as CSV via the [GRN Benchmarking]/ground truth save path config field
- GRouNdGAN-simulated expression matrices (e.g., 100k cells x 1000 genes for BoneMarrow and PBMC references)

## Outputs

- Per-gene precision-at-k values (k = 15)
- Global precision–recall curve with AUPRC
- Random-predictor baseline AUPRC
- Aggregated method ranking table (ranking_table.csv)

## How to apply

Format every inferred network (e.g., from GRNBoost2, GENIE3, PIDC, PPCOR, LEAP, SCODE, SINCERITIES, CeSpGRN) as a tab-separated CSV with columns in the order TF, Gene, importance (headers optional). Export the ground-truth GRN by setting [GRN Benchmarking]/ground truth save path in the GRouNdGAN config, then run the --benchmark_grn command, which evaluates the reconstructed GRN from GRouNdGAN-simulated data against the GRN used for simulation. Compute per-gene precision-at-k with k = 15 — matching the top 15 GRNBoost2-identified TFs per gene imposed as ground truth — plus a global precision–recall curve with AUPRC and the random-predictor baseline AUPRC for calibration. Aggregate precision@k and AUPRC across datasets (e.g., BoneMarrow and PBMC) into a single ranking_table.csv and verify the method ordering reproduces BEELINE's curated experimental benchmark (PIDC first, followed by GENIE3 and GRNBoost2, with only LEAP and SCODE swapped).

## Related tools

- **GRouNdGAN** (Provides the --benchmark_grn evaluation command, the imposed causal ground-truth GRN, and ground-truth CSV export used for precision-at-k/AUPRC benchmarking.) — github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the regulatory edges from the preprocessed reference data that become the ground truth (top 15 TFs per gene) and produces candidate inferred networks for benchmarking.)
- **BEELINE** (Curated experimental benchmark whose method ordering serves as the external validation target for the GRouNdGAN-based ranking.) — github.com/Murali-group/Beeline

## Examples

```
docker pull yazdanz/groundgan:4b98686  # then run the container with --benchmark_grn to evaluate the inferred GRN against the exported ground truth
```

## Evaluation signals

- Precision-at-k is computed per gene with k = 15, matching the top-15 GRNBoost2-identified TFs per gene imposed as the ground-truth GRN.
- The PR evaluation reports AUPRC alongside the random-predictor baseline AUPRC so method scores are interpretable relative to chance.
- The aggregated ranking_table.csv places PIDC first (followed by GENIE3 and GRNBoost2) and matches BEELINE's curated ordering except for a LEAP/SCODE rank swap.
- Every inferred-network file is a tab-separated CSV with column order TF, Gene, importance (headers optional) before benchmarking.
- A ground-truth GRN CSV exists at the configured [GRN Benchmarking]/ground truth save path prior to running the evaluation.

## Limitations

- GRouNdGAN only supports imposing bipartite GRNs (no multi-layer or TF-gene-phenotype causal graphs), which constrains the ground truth that can be benchmarked.
- Benchmarking on multi-cell-type data such as PBMC-All is not conclusive; the effect of distinct cell types on GRN inference performance requires follow-up studies.
- No comprehensive evaluation of cell-specific GRN inference methods (e.g., CeSpGRN) has been performed from different perspectives.

## Evidence

- [methods] This command evaluates the reconstructed GRN (from GRouNdGAN-simulate data) against the ground truth GRN (used for simulation).: "This command evaluates the reconstructed GRN (from GRouNdGAN-simulate data) against the ground truth GRN (used for simulation)."
- [abstract] GRouNdGAN effectively bridges the existing gap between simulated and biological data benchmarks of GRN inference algorithms, providing gold standard ground truth GRNs: "GRouNdGAN effectively bridges the existing gap between simulated and biological data benchmarks of GRN inference algorithms, providing gold standard ground truth GRNs"
- [intro] Using GRouNdGAN, we benchmark eight GRN inference methods and find the results to be aligned with BEELINE’s experimental results19.: "Using GRouNdGAN, we benchmark eight GRN inference methods and find the results to be aligned with BEELINE’s experimental results19."
- [discussion] The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN, an assumption that we plan to relax: "The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN, an assumption that we plan to relax"
- [methods] If you specify ``[GRN Benchmarking]/ground truth save path``, the ground truth GRN will be exported as a CSV:: "If you specify ``[GRN Benchmarking]/ground truth save path``, the ground truth GRN will be exported as a CSV:"
