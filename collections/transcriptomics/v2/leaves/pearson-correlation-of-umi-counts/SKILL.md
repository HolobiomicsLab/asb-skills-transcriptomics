---
name: pearson-correlation-of-umi-counts
description: Use when two aligned gene-barcode matrices (same barcodes, zero-padded
  genes) from different processing pipelines must be quantitatively compared at the
  UMI-count level to assess processing concordance.
when_to_use_negative:
- The GBM pair has not been adjusted — barcodes not shared between the two matrices
  still present, or genes missing from one matrix have not been zero-padded
- The goal is to compare clustering outcomes rather than raw UMI-count concordance
  — use the Adjusted Rand Index on Seurat clustering results instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: stats
  role: R package (version 4.1.2) used to compute the Pearson correlation between
    GBM pairs
- name: clues
  role: R package (version 0.6.2.2) used in parallel to compute the Adjusted Rand
    Index between clustering outcomes
- name: UniverSC
  role: Universal pipeline that generated one GBM of each pair for comparison
  repo: https://github.com/minoda-lab/universc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:04684efa9387fecd1d70d2f72d9ffdf1e81fa580ca8a4fe52f0731fd13fa2699
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/minoda-lab/universc
  tools:
  - R
  - UniverSC
  - clues
  - stats
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pearson-correlation-of-umi-counts

## Summary

Computes the Pearson correlation between paired gene-barcode matrices (GBMs) from UniverSC and platform-specific pipelines, treating each non-zero gene-by-cell UMI-count pair as one data point, as a supporting concordance metric alongside the Adjusted Rand Index.

## When to use

Use when two aligned gene-barcode matrices (same barcodes, zero-padded genes) from different processing pipelines must be quantitatively compared at the UMI-count level to assess processing concordance.

## When NOT to use

- The GBM pair has not been adjusted — barcodes not shared between the two matrices still present, or genes missing from one matrix have not been zero-padded
- The goal is to compare clustering outcomes rather than raw UMI-count concordance — use the Adjusted Rand Index on Seurat clustering results instead

## Inputs

- Adjusted gene-barcode matrix (GBM) from UniverSC processing (matching barcodes, zero-padded genes)
- Adjusted gene-barcode matrix (GBM) from the platform-specific counterpart pipeline (Cell Ranger, dropSeqPipe, CogentAP, or zUMIs)

## Outputs

- Pearson correlation value per platform GBM pair (Chromium, Drop-seq, ICELL8, SmartSeq3)
- Results table combining ARI values and Pearson correlations, written to ari_results.csv

## How to apply

Load the adjusted GBM pairs produced by the alignment step (barcodes kept only if present in both matrices; genes missing from one matrix padded with 0 UMIs) for each of the four platforms — Chromium, Drop-seq, ICELL8, and SmartSeq3. For each pair, compute the Pearson correlation across all gene-by-cell entries, counting each gene-by-cell UMI-count pair as one data point unless both counts are zero. Assemble the correlations as supporting metrics alongside the ARI values in a single results table and write it to ari_results.csv. High correlations, together with ARI values of 1 (Chromium), 0.78 (Drop-seq), 0.87 (ICELL8), and 0.78 (SmartSeq3), confirm that UniverSC processing reproduces the counterpart pipelines' outputs.

## Related tools

- **stats** (R package (version 4.1.2) used to compute the Pearson correlation between GBM pairs)
- **clues** (R package (version 0.6.2.2) used in parallel to compute the Adjusted Rand Index between clustering outcomes)
- **UniverSC** (Universal pipeline that generated one GBM of each pair for comparison) — https://github.com/minoda-lab/universc

## Examples

```
cor(as.vector(gbm_universc), as.vector(gbm_counterpart), method="pearson")  # computed over entries where not both UMI counts are zero
```

## Evaluation signals

- Each Pearson correlation must be computed only over gene-by-cell entries where at least one of the two counts is non-zero (pairs with both counts zero are excluded)
- Correlation values should be high (close to 1) for all four platform pairs, consistent with the reported high similarity of UniverSC vs. counterpart-pipeline GBMs
- The computed ARI values should approximate the reported benchmarks: ~1.0 (Chromium), 0.78 (Drop-seq), 0.87 (ICELL8), 0.78 (SmartSeq3) as in Fig. 2
- The output ari_results.csv must contain one row per platform comparison with both ARI and Pearson correlation assembled into a single table

## Limitations

- The comparison relies on GBM pairs pre-adjusted to matching barcodes and genes; unadjusted matrices with differing barcode/gene sets would bias the correlation
- Correlation is a supporting metric only — clustering concordance (ARI) is the primary benchmark in the paper
- UniverSC uses Cell Ranger version 3.0.2 due to licensing, which may affect comparability with newer pipeline versions

## Evidence

- [methods] the Pearson correlation between the GBMs and ARI between the two clustering outcomes were calculated: "the Pearson correlation between the GBMs and ARI between the two clustering outcomes were calculated"
- [methods] The pair of GBMs was adjusted to have matching sets of barcodes and genes: "The pair of GBMs was adjusted to have matching sets of barcodes and genes: only barcodes found in both GBMs were kept, and genes only found in one GBM were added to the other with 0 UMIs assigned."
- [methods] The adjusted pair of GBMs was then used to carry out clustering analysis with an R package Seurat: "The adjusted pair of GBMs was then used to carry out clustering analysis with an R package Seurat"
