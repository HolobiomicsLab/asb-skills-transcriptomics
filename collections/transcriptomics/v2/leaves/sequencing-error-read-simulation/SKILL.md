---
name: sequencing-error-read-simulation
description: Use when benchmarking or validating UMI recovery/collapse methods (e.g.,
  Longcell iterative clustering vs DBSCAN eps=2) where you need error-laden simulated
  reads with known ground-truth UMI counts across amplification levels.
when_to_use_negative:
- Input is real Nanopore sequencing data with no ground-truth UMI counts — simulation-based
  validation is unnecessary and ground truth cannot be verified
- The downstream method collapses UMIs by whitelist-based cell-barcode-style matching
  rather than clustering, where error-tolerance behavior is already known
- Sequencing error is negligible (e.g., short-read Illumina UMI protocols), since
  inflation under high error is the phenomenon being modeled
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Longcell
  role: Supplies the iterative error-tolerant UMI clustering evaluated on the simulated
    samples (meta-isoform-group Needleman-Wunsch graph + Louvain via igraph v1.5.1)
  repo: github.com/yuntianf/Longcell
- name: igraph
  role: R package (v1.5.1) implementing the iterative Louvain algorithm used to cluster
    simulated UMIs within subgraphs
- name: dbscan
  role: Fixed-edit-distance baseline (eps=2) applied to the same simulated samples
    to quantify UMI-count inflation
- name: Nanopore
  role: Source of the sequencing error profile (85-99% per-base accuracy) emulated
    in the simulated reads
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:58df2dbe7fa622fab8fddce147ad63e01abbf148f8aec8ddc7346302a87eaaab
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yuntianf/Longcell
  tools:
  - Longcell
  - Nanopore
  - dbscan
  - igraph
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# sequencing-error-read-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates simulated UMI-bearing read samples (e.g., adapter 10-mers) with Nanopore-like sequencing errors across PCR amplification folds from 2x to 50x, retaining ground-truth UMI counts, so that UMI-collapse/clustering methods can be benchmarked for count inflation under high error rates.

## When to use

Use when benchmarking or validating UMI recovery/collapse methods (e.g., Longcell iterative clustering vs DBSCAN eps=2) where you need error-laden simulated reads with known ground-truth UMI counts across amplification levels.

## When NOT to use

- Input is real Nanopore sequencing data with no ground-truth UMI counts — simulation-based validation is unnecessary and ground truth cannot be verified
- The downstream method collapses UMIs by whitelist-based cell-barcode-style matching rather than clustering, where error-tolerance behavior is already known
- Sequencing error is negligible (e.g., short-read Illumina UMI protocols), since inflation under high error is the phenomenon being modeled

## Inputs

- Ground-truth UMI sequences (e.g., adapter 10-mers) with per-sample true UMI counts
- Target amplification fold range (2 to 50) for sampling depth
- Nanopore sequencing error model (per-base accuracy 85-99%)

## Outputs

- Simulated UMI read samples with sequencing errors at each amplification fold
- umi_clustering_benchmark.csv: estimated UMI counts per amplification fold for Longcell and DBSCAN (eps=2) versus ground truth

## How to apply

Generate simulated adapter 10-mer UMI read samples per cell with sequencing errors drawn at the Nanopore per-base accuracy range, sampling each ground-truth UMI at amplification folds from 2 to 50 and recording the true UMI count per sample. Use the simulated samples to drive downstream UMI-grouping comparisons: within each meta-isoform group, compute Needleman-Wunsch scores between UMIs as similarity, build a graph with edges filtered at weight l/2+2 (l = UMI length), and iteratively apply Louvain (igraph v1.5.1), splitting subgraphs recursively until min_cut > 3 or minimum edge weight exceeds l/2+4. Run the fixed-edit-distance baseline (DBSCAN, eps=2) on the same simulated samples. Tabulate estimated UMI counts per amplification fold for both methods against ground truth to expose UMI-scattering-driven inflation at high error/amplification.

## Related tools

- **Longcell** (Supplies the iterative error-tolerant UMI clustering evaluated on the simulated samples (meta-isoform-group Needleman-Wunsch graph + Louvain via igraph v1.5.1)) — github.com/yuntianf/Longcell
- **igraph** (R package (v1.5.1) implementing the iterative Louvain algorithm used to cluster simulated UMIs within subgraphs)
- **dbscan** (Fixed-edit-distance baseline (eps=2) applied to the same simulated samples to quantify UMI-count inflation)
- **Nanopore** (Source of the sequencing error profile (85-99% per-base accuracy) emulated in the simulated reads)

## Evaluation signals

- Longcell's estimated UMI counts remain stable across amplification folds 2-50, tracking ground truth without inflation
- DBSCAN (eps=2) baseline overestimates UMI numbers at high sequencing error on the same samples, reproducing the Fig. 1B trend
- Graph edge weights below l/2+2 (for UMI length l) are excluded, and recursive Louvain splits stop when min_cut > 3 or minimum edge weight exceeds l/2+4
- umi_clustering_benchmark.csv contains per-fold estimates for both methods plus ground-truth counts with no missing fold values

## Limitations

- Simulation uses adapter 10-mer UMIs, so results may not generalize to all UMI lengths or random-sequence compositions
- Nanopore per-base accuracy varies widely (85-99%), and read truncation and mapping errors additionally distort isoform quantification beyond UMI-level simulation
- Existing fixed-edit-distance methods (Sicelore2, FLAMES) are known to overestimate UMIs under high error, so baselines must be run on identical simulated input for a fair comparison

## Evidence

- [other] Within each cell, Longcell first groups putative UMIs at the meta-isoform-group level and applies an iterative clustering procedure that tolerates sequencing errors: [evidence span withheld — anchor status: not_found]
- [other] Simulation spans 2-50x amplification folds with ground truth retained: [evidence span withheld — anchor status: not_found]
