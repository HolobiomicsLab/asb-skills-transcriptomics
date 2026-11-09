---
name: pcr-amplification-fold-quantification
description: Use when quantifying isoform expression from single-cell or spatially
  barcoded Nanopore reads where PCR amplification scattering inflates UMI counts and
  fixed-edit-distance collapsing would overestimate UMIs under high sequencing error.
when_to_use_negative:
- When input reads are short Illumina reads with low per-base error, where UMI collapse
  by fixed edit distance suffices and the r-based pruning tail assumption is unnecessary
- When UMI sequences are whitelist-anchored like cell barcodes; UMIs are shorter and
  randomly generated, so the skill's error model (estimated from 12mer adapter Needleman-Wunsch
  scores) assumes no UMI whitelist is available
- When the goal is consensus alignment correction or isoform discovery rather than
  correcting amplification-scattered UMI counts — the pipeline handles those in separate
  steps
edam_operation: http://edamontology.org/operation_3227
tools:
- name: Longcell
  role: R package implementing the r-based scatter-reduction pruning of within-isoform
    UMI clusters and isoform quantification; installed via devtools from GitHub
  repo: github.com/yuntianf/Longcell
- name: Nanopore
  role: Long-read sequencing platform producing the high-error-rate reads whose UMI
    clusters require adaptive pruning before quantification
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:37d91f936ae92d7d67873e3454b041f6abb78fab9a90c90fa18a9ea55fecc605
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pcr-amplification-fold-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Adaptive UMI cluster-size pruning in Longcell that corrects PCR amplification scattering in Nanopore single-cell isoform quantification by removing error-driven small clusters from the left tail of each within-isoform cluster-size distribution, with stability validated against GC-dependent amplification folds.

## When to use

Use when quantifying isoform expression from single-cell or spatially barcoded Nanopore reads where PCR amplification scattering inflates UMI counts and fixed-edit-distance collapsing would overestimate UMIs under high sequencing error.

## When NOT to use

- When input reads are short Illumina reads with low per-base error, where UMI collapse by fixed edit distance suffices and the r-based pruning tail assumption is unnecessary
- When UMI sequences are whitelist-anchored like cell barcodes; UMIs are shorter and randomly generated, so the skill's error model (estimated from 12mer adapter Needleman-Wunsch scores) assumes no UMI whitelist is available
- When the goal is consensus alignment correction or isoform discovery rather than correcting amplification-scattered UMI counts — the pipeline handles those in separate steps

## Inputs

- Nanopore long reads with recovered cell barcodes and UMIs (Longcell-pre output: k-mer dictionary (k=6) barcode alignment, Needleman-Wunsch UMI quality check, minimap2 spliced alignment against GRCh38, Louvain UMI clustering with edge cutoff l/2+2)
- 12mer adapter sequences upstream of confidently identified cell barcodes (for estimating the singleton ratio r)
- UMI cluster sizes per isoform per cell (clusters built by Louvain clustering, min_cut > 3, stop when minimum edge weight exceeds l/2+4)
- Matched Illumina UMI cluster-size distribution of a single-isoform gene (for validation)

## Outputs

- Pruned per-cell per-isoform UMI counts (m-p after scatter-reduction pruning)
- Estimated error-driven singleton ratio r
- Plot of pruned UMI counts vs. GC-dependent PCR amplification fold A = k*sigmoid(0.5*(G-50)) + 2 across meta-isoform groups
- Comparison plot of pruned Nanopore vs. Illumina UMI cluster-size distributions

## How to apply

First estimate the error-driven singleton ratio r from the Needleman-Wunsch score distribution of the 12mer adapters upstream of confidently identified cell barcodes, counting adapters scoring below k = l/2+2. Then, for each isoform within each cell, rank UMI cluster sizes s1...sm increasingly and iteratively remove the smallest cluster until the cumulative removed size satisfies the balance condition removed_sum >= r * remaining_sum / (1-r); report the pruned UMI count as m-p. The rationale is that within-isoform UMI cluster sizes should be comparable, and small clusters (e.g., singletons) in the left tail of the empirical distribution are likely sequencing-error artifacts rather than true transcripts. Validate by plotting pruned UMI counts against the GC-dependent PCR amplification fold A = k*sigmoid(0.5*(G-50)) + 2 (k=20) across meta-isoform groups to confirm quantification stability, and by comparing the pruned Nanopore cluster-size distribution of a single-isoform gene against the matched Illumina UMI cluster-size distribution.

## Related tools

- **Longcell** (R package implementing the r-based scatter-reduction pruning of within-isoform UMI clusters and isoform quantification; installed via devtools from GitHub) — github.com/yuntianf/Longcell
- **Nanopore** (Long-read sequencing platform producing the high-error-rate reads whose UMI clusters require adaptive pruning before quantification)

## Examples

```
devtools::install_github("yuntianf/Longcell",dependencies=TRUE)
```

## Evaluation signals

- The pruning loop terminates only when the balance condition removed_sum >= r * remaining_sum / (1-r) is satisfied for every isoform-cell pair
- Pruned Nanopore UMI cluster-size distribution of a single-isoform gene matches the matched Illumina distribution (Supplementary Fig. 5C)
- Pruned UMI counts remain stable across PCR amplification folds when plotted against A = k*sigmoid(0.5*(G-50)) + 2 with k=20 (Fig. 1B)
- Pruned counts are consistently lower than or equal to raw UMI cluster counts, with the smallest (singleton-dominated) clusters removed first
- The recorded singleton ratio r is reproducible across the Needleman-Wunsch score distribution of 12mer adapters, with the count of adapters below k = l/2+2 reported in output files

## Limitations

- Nanopore per-base sequencing accuracy of 85–99% makes UMI recovery inherently vulnerable; the pruning quality depends on how well the error-driven singleton ratio r is estimated from adapter scores
- Fixed-edit-distance UMI collapsing methods (Sicelore2, FLAMES) overestimate UMIs under high sequencing error, so the pruning must replace, not supplement, such collapsing
- The assumption that within-isoform UMI cluster sizes are comparable may fail when true biological UMI spread is large, risking over-pruning of genuine low-count clusters

## Evidence

- [other] error-driven small clusters are noise: [evidence span withheld — anchor status: not_found]
- [discussion] consensus corrects PCR duplicates: [evidence span withheld — anchor status: not_found]
