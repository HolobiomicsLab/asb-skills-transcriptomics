---
name: illumina-nanopore-cross-platform-umi-comparison
description: Use when validating Nanopore single-cell isoform quantification after
  adaptive UMI cluster pruning (e.g., Longcell step 4) by checking that pruned cluster-size
  distributions reproduce a matched short-read Illumina reference distribution.
when_to_use_negative:
- The sample lacks a matched Illumina (short-read) dataset for the same gene/cell
  population, so no reference UMI cluster-size distribution exists for comparison
- The gene under inspection has multiple isoforms, making it impossible to attribute
  Nanopore UMI clusters cleanly to a single isoform as the Fig. 1C comparison requires
- Quantification was produced by fixed-edit-distance UMI collapsing (e.g., Sicelore2
  or FLAMES) rather than Longcell's adaptive pruning, in which case distributions
  reflect error-inflated UMI counts
tools:
- name: Longcell
  role: Recovers Nanopore cell barcodes/UMIs, builds Louvain UMI clusters, and performs
    the adaptive pruning whose output distribution is compared to the Illumina reference
  repo: github.com/yuntianf/Longcell
- name: Nanopore
  role: Long-read sequencing platform producing the UMI cluster-size distribution
    under validation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:1ff343452890b97ff621496a2ac3732991430b81b116fe3571150da78b76bb49
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# illumina-nanopore-cross-platform-umi-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validation skill for Longcell's UMI-scatter-reduction pruning: compare the pruned Nanopore UMI cluster-size distribution of a single-isoform gene against the matched Illumina UMI cluster-size distribution to confirm that error-driven small clusters were correctly removed rather than true transcripts.

## When to use

Use when validating Nanopore single-cell isoform quantification after adaptive UMI cluster pruning (e.g., Longcell step 4) by checking that pruned cluster-size distributions reproduce a matched short-read Illumina reference distribution.

## When NOT to use

- The sample lacks a matched Illumina (short-read) dataset for the same gene/cell population, so no reference UMI cluster-size distribution exists for comparison
- The gene under inspection has multiple isoforms, making it impossible to attribute Nanopore UMI clusters cleanly to a single isoform as the Fig. 1C comparison requires
- Quantification was produced by fixed-edit-distance UMI collapsing (e.g., Sicelore2 or FLAMES) rather than Longcell's adaptive pruning, in which case distributions reflect error-inflated UMI counts

## Inputs

- Pruned Nanopore UMI cluster-size distribution per isoform/cell from Longcell (post step-4 scatter-reduction pruning)
- Matched Illumina short-read UMI cluster-size distribution for the same gene(s) and sample
- Estimated error-driven singleton ratio r from the Needleman-Wunsch score distribution of 12mer adapters upstream of confidently identified cell barcodes (adapters scoring below k = l/2+2)
- Read-to-isoform assignments from minimap2 spliced alignment against GRCh38

## Outputs

- Overlay plot of pruned Nanopore vs. matched Illumina UMI cluster-size distributions for a single-isoform gene (Fig. 1C-style)
- Plot of pruned UMI counts vs. GC-dependent PCR amplification fold across meta-isoform groups (Fig. 1B-style)
- Recorded singleton ratio r and pruned UMI counts per isoform/cell

## How to apply

After recovering cell barcodes and UMIs from Nanopore reads with Longcell (k-mer dictionary barcode alignment, Needleman-Wunsch UMI checks, Louvain UMI clustering with edge cutoff l/2+2) and pruning UMI clusters per isoform using the error-driven singleton ratio r (remove smallest clusters until removed sum >= r * remaining sum / (1-r)), select a single-isoform gene with matched Illumina data. Plot the pruned Nanopore UMI cluster-size distribution and overlay the matched Illumina UMI cluster-size distribution for the same gene/cell population. The two distributions should be concordant (Supplementary Fig. 5C-style comparison), because small Nanopore clusters in the left tail are attributed to sequencing-error-driven 'scatter' rather than true molecules. Complement this with the Fig. 1B check that pruned UMI counts remain stable across the GC-dependent PCR amplification fold (A = k*sigmoid(0.5*(G-50)) + 2, k=20) across meta-isoform groups; record r, pruned counts, and both comparison plots.

## Related tools

- **Longcell** (Recovers Nanopore cell barcodes/UMIs, builds Louvain UMI clusters, and performs the adaptive pruning whose output distribution is compared to the Illumina reference) — github.com/yuntianf/Longcell
- **Nanopore** (Long-read sequencing platform producing the UMI cluster-size distribution under validation)

## Evaluation signals

- The pruned Nanopore UMI cluster-size distribution of the single-isoform gene matches the Illumina counterpart (Supplementary Fig. 5C criterion)
- Pruned UMI counts remain stable across GC-dependent PCR amplification folds (A = k*sigmoid(0.5*(G-50)) + 2, k=20) across meta-isoform groups
- The removed cumulative cluster size satisfies the balance condition: removed sum >= r * remaining sum / (1-r) at the stopping point
- The singleton ratio r is recorded and derived from adapters scoring below k = l/2+2 in the Needleman-Wunsch 12mer adapter score distribution
- No pruned clusters remain in the extreme left tail (e.g., singletons) that would indicate residual error-driven scatter

## Limitations

- UMI recovery on Nanopore is more error-prone than cell barcode recovery because UMIs are shorter, randomly generated, and not whitelist-backed, so the comparison depends on a reliably estimated r
- The Illumina-match validation is demonstrated for single-isoform genes; multi-isoform genes confound cluster-to-isoform attribution
- Existing fixed-edit-distance UMI collapsing methods (Sicelore2, FLAMES) overestimate UMI counts under high sequencing error, so their distributions cannot serve as the pruned reference

## Evidence

- [other] workflow step 5: [evidence span withheld — anchor status: not_found]
- [other] workflow step 6: [evidence span withheld — anchor status: not_found]
- [readme] Longcell installation: [evidence span withheld — anchor status: external_doc]
