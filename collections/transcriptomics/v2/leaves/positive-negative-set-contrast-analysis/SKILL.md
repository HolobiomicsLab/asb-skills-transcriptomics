---
name: positive-negative-set-contrast-analysis
description: Use when you have two balanced sets of centered, fixed-length nucleotide
  fragments (e.g., 5,298 vs 5,298 A. thaliana; 4,563 vs 4,563 M. musculus) and need
  to test whether the positive class shows position-specific residue enrichment or
  motifs relative to the negative class.
when_to_use_negative:
- You need a predictive model of m5C sites rather than a descriptive motif contrast
  (use the Staem5 stacked-ensemble predictor instead)
- Positive and negative fragment sets are not balanced or fragments are not aligned
  to a common centered reference position, so per-position contrasts are confounded
- The analysis goal is feature-based classification performance (e.g., F-score feature
  selection, SVM/XGBoost evaluation), not visual nucleotide preference inspection
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Two Sample Logo
  role: Generates the two-sample sequence logos by computing per-position nucleotide
    frequency differences between m5C-positive and m5C-negative fragments
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:23e0e1c473b62be0d06fa09e04917d9d550e713b94efd3494ca60bb69eb51442
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Two Sample Logo
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# positive-negative-set-contrast-analysis

## Summary

Contrasts nucleotide preferences between balanced positive (m5C-containing) and negative (non-m5C) 41-bp RNA fragments using the Two Sample Logo approach, rendering per-position two-sample sequence logos to reveal whether m5C sites carry distinctive flanking motifs.

## When to use

Use when you have two balanced sets of centered, fixed-length nucleotide fragments (e.g., 5,298 vs 5,298 A. thaliana; 4,563 vs 4,563 M. musculus) and need to test whether the positive class shows position-specific residue enrichment or motifs relative to the negative class.

## When NOT to use

- You need a predictive model of m5C sites rather than a descriptive motif contrast (use the Staem5 stacked-ensemble predictor instead)
- Positive and negative fragment sets are not balanced or fragments are not aligned to a common centered reference position, so per-position contrasts are confounded
- The analysis goal is feature-based classification performance (e.g., F-score feature selection, SVM/XGBoost evaluation), not visual nucleotide preference inspection

## Inputs

- Balanced FASTA fragment sets of m5C-positive sequences (A. thaliana: 5,298; M. musculus: 4,563)
- Balanced FASTA fragment sets of m5C-negative sequences (A. thaliana: 5,298; M. musculus: 4,563)
- 41-bp fragments with the candidate m5C cytosine centered at position 0

## Outputs

- Two-sample sequence logo image per species (A. thaliana and M. musculus)
- Per-position nucleotide frequency/enrichment table (nucleotide_frequencies.csv) covering every aligned position for both species

## How to apply

Load the balanced m5C-positive and m5C-negative training fragments for each species and align the 41-bp fragments with the candidate cytosine centered so upstream positions (-18 to -1) and downstream positions can be indexed per nucleotide. Compute per-position nucleotide frequency differences between the positive and negative sets with the Two Sample Logo approach, which highlights statistically enriched residues at each aligned position. Render the two-sample logo image per species and export the per-position nucleotide frequency/enrichment table to nucleotide_frequencies.csv. Validate qualitatively against the published pattern (e.g., C enrichment in upstream positions of A. thaliana fragments) and confirm the frequency table covers every aligned position for both species.

## Related tools

- **Two Sample Logo** (Generates the two-sample sequence logos by computing per-position nucleotide frequency differences between m5C-positive and m5C-negative fragments)

## Evaluation signals

- The rendered logo qualitatively reproduces the reported pattern (e.g., C enrichment in upstream positions of A. thaliana fragments) with no strong overall motifs distinguishing m5C from non-m5C sites
- nucleotide_frequencies.csv covers every aligned position (-18..+18 relative to the centered cytosine, plus center) for both A. thaliana and M. musculus
- Positive and negative set sizes are equal per species (5,298 vs 5,298 for A. thaliana; 4,563 vs 4,563 for M. musculus) so frequency contrasts are unconfounded
- Each fragment is 41 bp with the candidate cytosine centered, verified before position indexing
- Conclusion matches the paper's finding: m5C sites in A. thaliana and M. musculus do not show notable sequence motifs versus non-m5C sites

## Limitations

- The finding is qualitative: no strong sequence motifs were detected, so this analysis cannot alone justify sequence-based m5C discrimination
- Contrasts are species-specific; patterns in A. thaliana and M. musculus must be assessed separately, and no general cross-species motif conclusion should be drawn

## Evidence
